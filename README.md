# Credit Default Prediction - Data Pipeline

## Executive Summary

A production-ready data engineering pipeline implementing the **Medallion Architecture** (Bronze → Silver → Gold) for credit default prediction. This pipeline transforms raw operational data into ML-ready features while maintaining strict **temporal correctness** to prevent data leakage.

**Critical Business Scenario**: At the time a customer applies for a loan, can we predict if they will default at MOB=6? This requires:
- Using **ONLY** features available at application time (MOB=0)
- Observing outcomes 6 months later (MOB=6) to create labels
- **Zero tolerance** for temporal leakage (no future information)

### Key Metrics
- **Processing Period**: 24 monthly snapshots (Jan 2023 - Dec 2024)
- **Data Sources**: 4 distinct operational systems (loan_daily, attributes, financials, clickstream)
- **Features Engineered**: 40+ derived features (all available at MOB=0)
- **Target Definition**: Default = DPD ≥ 30 days at MOB=6
- **Temporal Design**: Features (MOB=0) → Labels (MOB=6) | 6-month separation
- **Prevent Leakage**: Excluded all payment history and performance metrics from MOB 1-6

---

## 🏗️ Architecture Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        DATA SOURCES                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │loan_daily│  │attributes│  │financials│  │clickstream│       │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    BRONZE LAYER (Raw Data)                       │
│              Schema Validation | Data Ingestion                  │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │  .csv    │  │  .csv    │  │  .csv    │  │  .csv    │       │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│              SILVER LAYER (Cleaned & Enriched)                   │
│        Type Casting | Feature Engineering | Validation           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │ .parquet │  │ .parquet │  │ .parquet │  │ .parquet │       │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│               GOLD LAYER (ML-Ready Features)                     │
│                  Join Operations | Aggregation                   │
│         ┌──────────────────┐    ┌──────────────────┐           │
│         │  Feature Store   │    │   Label Store    │           │
│         │   (Features)     │    │   (Targets)      │           │
│         └──────────────────┘    └──────────────────┘           │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
                   ┌────────────────────┐
                   │   ML Model Training │
                   └────────────────────┘
```

---

## ⏰ Temporal Design: Preventing Data Leakage

### 🎯 Business Scenario

**The Challenge**: A loan officer sits across from a customer on **January 1, 2024** (MOB=0). The customer applies for a $50,000 loan. The officer must decide in real-time: **Approve or Reject?**

**What Information is Available** (MOB=0):
- ✅ Customer demographics (age, occupation, marital status)
- ✅ Financial profile (income, existing debts, credit score)
- ✅ Credit history (number of previous loans, delayed payments)
- ✅ Digital behavior (website engagement, clicks, time spent)
- ✅ Requested loan details (amount, tenure, interest rate)

**What Information is NOT Available** (MOB 1-6):
- ❌ Whether they'll make timely payments next month
- ❌ If they'll miss installments in months 2-5
- ❌ Their account balance after 3 months
- ❌ Days past due status in the future
- ❌ Any payment behavior after application

**The Evaluation** (MOB=6):
> Six months later (**July 1, 2024**), we observe the outcome: Did the customer default (DPD ≥ 30 days)?

This is our **label** - but it was **unknown** at decision time.

---

### 🚫 What is Temporal Leakage?

**Definition**: Using information from the future to predict the past. This creates artificially high model performance in testing but catastrophic failures in production.

**Example of Leakage** ❌:

| Feature | Available at MOB=0? | Include in Model? | Reason |
|---------|---------------------|-------------------|--------|
| `customer_age` | ✅ Yes | ✅ Yes | Static attribute at application |
| `Annual_Income` | ✅ Yes | ✅ Yes | Known from application form |
| `loan_amt` | ✅ Yes | ✅ Yes | Part of loan request |
| `paid_amt_mob3` | ❌ No (MOB=3) | ❌ **NO** | **LEAKAGE**: Payment 3 months in future |
| `overdue_amt_mob4` | ❌ No (MOB=4) | ❌ **NO** | **LEAKAGE**: Overdue 4 months in future |
| `balance_mob5` | ❌ No (MOB=5) | ❌ **NO** | **LEAKAGE**: Balance 5 months in future |
| `dpd_mob6` | ❌ No (MOB=6) | ❌ **NO** | **LEAKAGE**: This IS the target! |

**Why Leakage is Dangerous**:

```
SCENARIO: Model trained WITH leakage
═══════════════════════════════════════════════════════════

Training Phase (Historical Data):
  Features: Age, Income, loan_amt, paid_amt_mob3, balance_mob5
  Model learns: "If balance_mob5 is low → likely to default"
  Test Accuracy: 95% ✨ (Looks amazing!)

Production Deployment:
  Real-time prediction at MOB=0
  Features available: Age, Income, loan_amt
  Missing features: paid_amt_mob3, balance_mob5 (Don't exist yet!)
  
  Result: Model crashes or uses random values
  Production Accuracy: 55% 💥 (Worse than random!)
  
  Business Impact:
    - Approving bad loans → $5M losses
    - Rejecting good customers → $3M lost revenue
    - Regulatory investigation → Brand damage
```

---

### ✅ Our Solution: Strict Temporal Separation

**Architecture Principle**: Features and Labels must be temporally separated

```
TIMELINE OF A LOAN APPLICATION
═══════════════════════════════════════════════════════════════════

 Month:     0         1         2         3         4         5         6
            │         │         │         │         │         │         │
            ▼                                                           ▼
      ┌──────────┐                                              ┌──────────┐
      │   LOAN   │                                              │  CHECK   │
      │  APPLIED │                                              │ DEFAULT? │
      └──────────┘                                              └──────────┘
            │                                                           │
            │                                                           │
   ┌────────▼────────────────────────────────┐              ┌──────────▼─────────┐
   │     FEATURE COLLECTION (MOB=0)          │              │  LABEL CREATION    │
   │                                         │              │     (MOB=6)        │
   │  ✅ Customer_age                        │              │                    │
   │  ✅ Annual_Income                       │              │  IF dpd >= 30:     │
   │  ✅ Occupation                          │              │    label = 1       │
   │  ✅ Num_Bank_Accounts                   │              │  ELSE:             │
   │  ✅ Credit_Score                        │              │    label = 0       │
   │  ✅ Existing_Debts                      │              │                    │
   │  ✅ Loan_Amount_Requested               │              │  label_def:        │
   │  ✅ Digital_Engagement_Features         │              │  "30dpd_6mob"      │
   │                                         │              │                    │
   │  ❌ EXCLUDED (Future Information):      │              └────────────────────┘
   │     • installment_num (MOB 1-6)        │
   │     • due_amt (MOB 1-6)                │              🔒 TEMPORAL SEPARATION
   │     • paid_amt (MOB 1-6)               │              ═══════════════════════
   │     • overdue_amt (MOB 1-6)            │              Features: T = 0 months
   │     • balance (MOB 1-6)                │              Labels:   T = 6 months
   │     • dpd (MOB 1-6)                    │              Gap:      6 months
   │     • mob (any value > 0)              │
   └─────────────────────────────────────────┘
```

**Implementation Strategy**:

| Layer | Action | Purpose |
|-------|--------|----------|
| **Bronze** | Ingest all historical loan data (MOB 0-12) | Complete audit trail |
| **Silver** | Calculate MOB and DPD for all periods | Enable temporal filtering |
| **Gold - Features** | **Filter: MOB = 0 only** | Simulate application time |
| **Gold - Labels** | **Filter: MOB = 6 only** | Observe 6-month outcome |
| **ML Training** | Join features (MOB=0) with labels (MOB=6) | Temporally valid training set |

**Key Design Decision**: The Gold layer performs temporal filtering **before** creating feature and label stores:

| Process | Filter Applied | Result |
|---------|----------------|--------|
| **Feature Store Creation** | `WHERE mob = 0` | Only application-time data |
| **Feature Selection** | Exclude: `due_amt`, `paid_amt`, `overdue_amt`, `balance`, `dpd`, `mob` | Remove all payment metrics |
| **Label Store Creation** | `WHERE mob = 6` | Only 6-month observations |
| **Label Definition** | `IF dpd >= 30 THEN 1 ELSE 0` | Binary default flag |

---

### 🎓 Think of It Like This:

> **Analogy**: Imagine you're a loan officer in January 2024. A customer walks in and requests a loan. You have their application form, credit report, and bank statements **from today**. You must decide: approve or reject?
> 
> **What you CAN see**: Their current job, income, existing debts, credit history up to today.
> 
> **What you CANNOT see**: Will they lose their job next month? Will they make payments on time? Will they default in 6 months?
> 
> **The Reality**: You can only know if your decision was correct 6 months later when you observe their actual payment behavior.
> 
> **The Pipeline**: Replicates this exact scenario using historical data. We "pretend" we're back at application time (MOB=0), use only information available then, and evaluate our predictions using outcomes observed 6 months later (MOB=6).

---

### 📊 Temporal Validation Results

**Validation Checklist** ✅:

| Check | Status | Evidence |
|-------|--------|----------|
| Features use only MOB=0 data | ✅ Pass | Filter applied in `process_feature_store()` |
| No payment history in features | ✅ Pass | Excluded: `due_amt`, `paid_amt`, `overdue_amt`, `balance` |
| No future MOB values in features | ✅ Pass | Excluded: `mob`, `dpd`, `installment_num` |
| Labels use MOB=6 observations | ✅ Pass | Filter applied in `process_label_store()` |
| Feature-Label temporal gap = 6 months | ✅ Pass | Same `snapshot_date`, different MOB filters |
| All features exist at application time | ✅ Pass | Demographics, financials, clickstream at MOB=0 |

**Sample Feature-Label Pair** (Temporally Valid):

```
Snapshot Date: 2024-01-01

FEATURE STORE (MOB=0):
  loan_id: LOAN_12345
  Customer_ID: CUST_001
  snapshot_date: 2024-01-01
  loan_amt: 50000
  tenure: 36
  customer_age: 35
  Annual_Income: 75000
  debt_to_income_ratio: 45.2
  Credit_Score: 720
  Num_of_Delayed_Payment: 2
  fe_1: 12.5, fe_2: 8.3, ... (clickstream features)
  
LABEL STORE (MOB=6):
  loan_id: LOAN_12345
  Customer_ID: CUST_001
  snapshot_date: 2024-01-01
  label: 1  (Customer defaulted at 6 months)
  label_def: "30dpd_6mob"
  
ML TRAINING SET (After Join):
  All features from MOB=0 + label from MOB=6
  This is VALID because:
    - Features were available when loan was approved (Jan 2024)
    - Label was observed 6 months later (July 2024)
    - No future information used in features
```

---

## 🔄 Pipeline Layers

### 1️⃣ Bronze Layer: Raw Data Ingestion

**Purpose**: Capture data exactly as received from source systems

#### Design Decisions
- ✅ **Format**: CSV (human-readable, easy debugging)
- ✅ **Partitioning**: By snapshot_date (YYYY_MM_DD)
- ✅ **No Transformation**: Maintain data lineage and auditability
- ✅ **Idempotent**: Overwrite mode for reprocessing capability

#### Data Sources

| Source | Records | Key Fields | Update Frequency |
|--------|---------|------------|------------------|
| **loan_daily** | ~1000/month | loan_id, installment_num, amounts | Daily snapshot |
| **attributes** | ~800/month | Customer_ID, demographics | Monthly snapshot |
| **financials** | ~800/month | Customer_ID, income, debts | Monthly snapshot |
| **clickstream** | ~800/month | Customer_ID, 20 features | Daily aggregation |

#### Directory Structure
```
datamart/bronze/
├── bronze_loan_daily/
│   ├── bronze_loan_daily_2023_01_01.csv
│   └── bronze_loan_daily_2024_12_01.csv
├── bronze_attributes/
├── bronze_financials/
└── bronze_clickstream/
```

---

### 2️⃣ Silver Layer: Cleaned & Enriched Data

**Purpose**: Apply business logic, data quality rules, and feature engineering

#### Design Decisions
- ✅ **Format**: Parquet (columnar, compressed, efficient)
- ✅ **Schema Enforcement**: Explicit type casting for data quality
- ✅ **Feature Engineering**: Domain-specific calculated fields
- ✅ **Data Cleaning**: Remove trailing underscores, validate ranges
- ✅ **Validation**: Null handling, outlier detection, business rule enforcement

#### Data Quality & Cleaning Rules

**Issue Detected**: Source data contained data quality issues:
- Trailing underscores in numeric fields (e.g., `'40_'`, `'52312.68_'`, `'__10000__'`)
- Invalid age values (e.g., `3843` years old)
- Extreme balance values (e.g., `-333333333333333333333333333`)

**Cleaning Strategy**:

| Issue | Detection Logic | Resolution | Rationale |
|-------|----------------|------------|------------|
| **Trailing Underscores** | Regex pattern match | `regexp_replace(col, "_", "")` before type cast | Remove contamination, enable type casting |
| **Invalid Age** | `Age > 150 OR Age < 0` | Set to `NULL` (not 0) | Preserve data quality signal for imputation |
| **Extreme Balances** | `abs(balance) > 1,000,000` | Set to `NULL` (not 0) | Flag extreme outliers for investigation |
| **Missing Values** | `IS NULL` check | Preserve as `NULL` | Enable downstream imputation strategies |
| **Negative Amounts** | Loan amounts, incomes < 0 | Set to `NULL` | Invalid for non-debt fields |

**Why NULL instead of 0?**
- ✅ `NULL` = "Unknown/Invalid" → Explicit data quality signal for ML pipeline
- ❌ `0` = "Zero value" → Misleading, hides data quality issues
- ✅ Enables informed decisions: impute, flag high-risk, or exclude from model

#### Transformations by Table

##### 📊 Loan Daily
| Feature | Calculation | Business Logic |
|---------|-------------|----------------|
| **mob** | installment_num | Month on Book |
| **dpd** | Days between snapshot and first missed payment | Days Past Due |
| **installments_missed** | CEIL(overdue_amt / due_amt) | Number of missed payments |

##### 👤 Attributes
| Feature | Calculation | Business Logic |
|---------|-------------|----------------|
| **age_group** | Age bucketing | "18-24", "25-34", "35-44", "45-54", "55-64", "65+" |
| **occupation_category** | Group similar occupations | "Technical", "Professional", "Management", "Education", "Other" |

##### 💰 Financials
| Feature | Calculation | Business Logic |
|---------|-------------|----------------|
| **debt_to_income_ratio** | (Total_EMI / Monthly_Salary) × 100 | DTI percentage |
| **savings_rate** | (Amount_invested / Monthly_Salary) × 100 | Savings percentage |
| **income_category** | Annual income bucketing | "Low", "Medium", "High", "Very High" |
| **credit_health_score** | Based on delayed payments | 100 (none) to 40 (>5 delays) |

##### 🖱️ Clickstream
| Feature | Calculation | Business Logic |
|---------|-------------|----------------|
| **total_activity_score** | SUM(fe_1 to fe_20) | Overall digital engagement |
| **avg_feature_value** | total_activity_score / 20 | Average engagement per feature |
| **engagement_level** | Score bucketing | "Low", "Medium", "High", "Very High" |
| **active_feature_count** | COUNT(non-zero features) | Breadth of engagement |

#### Directory Structure
```
datamart/silver/
├── silver_loan_daily/
│   └── silver_loan_daily_2023_01_01.parquet
├── silver_attributes/
├── silver_financials/
└── silver_clickstream/
```

---

### 3️⃣ Gold Layer: ML-Ready Feature & Label Stores

**Purpose**: Create unified, model-ready datasets for ML pipelines with strict temporal correctness

#### Design Decisions
- ✅ **Star Schema**: Denormalized for ML performance
- ✅ **Feature Store**: All predictors available at MOB=0
- ✅ **Label Store**: Separate target variables observed at MOB=6
- ✅ **Temporal Filtering**: MOB=0 for features, MOB=6 for labels
- ✅ **Leakage Prevention**: Exclude all payment history and future performance metrics
- ✅ **Join Key**: Customer_ID + loan_id + snapshot_date

#### Gold Layer Components

```
┌─────────────────────────────────────────────────────────────────┐
│                    FEATURE STORE (MOB=0 ONLY)                    │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌──────────────┐│
│  │ Loan Data  │ │  Customer  │ │  Financial │ │   Digital    ││
│  │ (MOB=0)    │+│ Attributes │+│  Profile   │+│  Engagement  ││
│  │ FILTERED   │ │            │ │            │ │  (MOB=0)     ││
│  └────────────┘ └────────────┘ └────────────┘ └──────────────┘│
│                                                                  │
│  🔒 TEMPORAL FILTER: WHERE mob = 0                               │
│  ❌ EXCLUDED: due_amt, paid_amt, overdue_amt, balance, dpd, mob │
│  ✅ INCLUDED: loan_amt, tenure, demographics, financials, clicks│
│  40+ Features | Left Join on Customer_ID                        │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                   LABEL STORE (MOB=6 ONLY)                       │
│                                                                  │
│  loan_id | Customer_ID | label | label_def | snapshot_date      │
│  ──────────────────────────────────────────────────────────────│
│  L001    | C123        | 1     | 30dpd_6mob| 2023-01-01        │
│  L002    | C124        | 0     | 30dpd_6mob| 2023-01-01        │
│                                                                  │
│  🔒 TEMPORAL FILTER: WHERE mob = 6                               │
│  📊 LABEL LOGIC: IF dpd >= 30 THEN 1 ELSE 0                     │
│  ⏱️ OBSERVATION POINT: 6 months after application                │
└─────────────────────────────────────────────────────────────────┘

                    ⚠️ TEMPORAL SEPARATION ENFORCED
              Features (T=0) ←─── 6 months ───→ Labels (T=6)
```

#### Feature Store Schema (MOB=0 Only)

| Category | Features | Count | Temporal Validity |
|----------|----------|-------|-------------------|
| **Loan Application** | loan_id, Customer_ID, loan_amt, tenure, loan_start_date | 5 | ✅ MOB=0 |
| **Demographics** | Age, age_group, occupation, occupation_category | 4 | ✅ MOB=0 |
| **Financial Profile** | Annual_Income, Monthly_Inhand_Salary, income_category, debt_to_income_ratio | 4 | ✅ MOB=0 |
| **Credit History** | Num_of_Loan, Outstanding_Debt, Credit_Utilization_Ratio, Num_of_Delayed_Payment, Credit_History_Age | 5 | ✅ MOB=0 |
| **Banking Relationship** | Num_Bank_Accounts, Num_Credit_Card, Interest_Rate, Changed_Credit_Limit | 4 | ✅ MOB=0 |
| **Digital Engagement** | fe_1, fe_2, ..., fe_20 (clickstream features at application time) | 20 | ✅ MOB=0 |
| **Time Variables** | snapshot_date | 1 | ✅ MOB=0 |

**Total: 40+ Features (All available at application time)**

**❌ EXCLUDED Features** (Would cause temporal leakage):
- `installment_num` - Future payment sequence
- `due_amt` - Future amounts due
- `paid_amt` - Future payment behavior
- `overdue_amt` - Future delinquency
- `balance` - Future account balance
- `mob` - Indicates time period (filtered to 0)
- `dpd` - Future days past due (this is what we're predicting!)

#### Label Store Design

**Target Definition**: 
- **Observation Point**: Month on Book (MOB) = 6
- **Default Definition**: Days Past Due (DPD) ≥ 30
- **Binary Classification**: 1 = Default, 0 = Non-Default

**Business Rationale**:
- 6 MOB provides sufficient payment history
- 30 DPD aligns with regulatory standards
- Early warning system for intervention

#### Directory Structure
```
datamart/gold/
├── feature_store/
│   └── feature_store_2023_01_01.parquet
└── label_store/
    └── label_store_2023_01_01.parquet
```

---

## 🛠️ Technical Stack

### Technologies
- **Framework**: Apache PySpark 3.x (Distributed data processing)
- **Language**: Python 3.8+
- **Storage Format**: CSV (Bronze) → Parquet (Silver, Gold)
- **Compression**: Snappy (Parquet default)
- **Orchestration**: Python scripts (Production-ready for Airflow/Databricks)
- **Version Control**: Git

### Key Capabilities
- **Data Transformations**: PySpark SQL functions (regexp_replace, when, coalesce)
- **Schema Definitions**: Explicit type casting (IntegerType, FloatType, DateType)
- **Date Operations**: Month-end snapshots, date arithmetic
- **File Management**: Pattern-based file discovery, partition management
- **Data Quality**: Validation rules, outlier detection, null handling

---

## 📊 Data Quality & Validation

### Silver Layer Quality Checks
1. **Schema Validation**: Explicit type casting for all columns
2. **Null Handling**: COALESCE for safe arithmetic operations
3. **Business Rules**: 
   - DTI ratio only calculated when salary > 0
   - DPD only for loans with overdue amounts
   - Feature counts exclude null/zero values

### Gold Layer Quality Checks
1. **Join Validation**: Left joins preserve all loan records
2. **Label Integrity**: MOB filter before label creation
3. **Feature Completeness**: Track missing value rates

---

## 🚀 Usage & Execution

### Running the Pipeline

**Execution**: Run the main pipeline orchestrator to process all 24 monthly snapshots through Bronze → Silver → Gold layers.

**Processing Flow**:
1. Bronze Layer: Ingest raw CSV files from source data
2. Silver Layer: Clean, validate, and engineer features
3. Gold Layer: Create feature store (MOB=0) and label store (MOB=6)

### Configuration

**Pipeline Parameters**:
- Start Date: `2023-01-01` (First snapshot)
- End Date: `2024-12-01` (Last snapshot)
- Output Directory: `datamart/` (Bronze, Silver, Gold subdirectories)
- DPD Threshold: `30 days` (Default definition)
- MOB Threshold: `6 months` (Observation point for labels)
- Feature MOB Filter: `0` (Application time only)
- Label MOB Filter: `6` (6-month observation)

### Output Summary

**Feature Store Statistics**:
- Total Rows: ~19,200 (24 snapshots × 800 loans/month)
- Total Columns: 40+ features
- All features available at MOB=0 (application time)
- Format: Parquet (partitioned by snapshot_date)

**Label Store Statistics**:
- Total Rows: ~19,200 (24 snapshots × 800 loans/month)
- Label Distribution: ~12-15% default rate (label=1)
- Observation Point: MOB=6 (6 months after application)
- Label Definition: DPD ≥ 30 days = Default (1), else Non-Default (0)
- Format: Parquet (partitioned by snapshot_date)

---

## 🎯 Future Enhancements

1. **ML Pipeline**:
   - feature engineering and selection
   - Feature importance tracking
   - Model selection
   - Model drift detection
2. **Orchestration**: Migrate to Airflow DAGs
3. **Cloud Deployment**: AWS/Azure/GCP integration

---

## 📚 Project Structure

```
assignment_1/
├── main.py                          # Pipeline orchestration
├── utils/
│   ├── data_processing_bronze_table.py
│   ├── data_processing_silver_table.py
│   └── data_processing_gold_table.py
├── data/                            # Source data
│   ├── loan_daily.csv
│   ├── attributes.csv
│   ├── financials.csv
│   └── clickstream.csv
├── datamart/                        # Output layers
│   ├── bronze/
│   ├── silver/
│   └── gold/
└── README.md                        # This file
```


---

## 🔗 References

1. [Medallion Architecture](https://www.databricks.com/glossary/medallion-architecture)
2. [Feature Store Design](https://www.featurestore.org/)
3. [PySpark Documentation](https://spark.apache.org/docs/latest/api/python/)
4. [Credit Risk Modeling Best Practices](https://www.risk.net/)

---
