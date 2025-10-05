# Credit Default Prediction - Data Pipeline# Credit Default Prediction - Data Pipeline# Credit Default Prediction - Data Pipeline



## 📋 Executive Summary



A production-ready data engineering pipeline implementing the **Medallion Architecture** (Bronze → Silver → Gold) for credit risk modeling. This pipeline processes raw customer and loan data into ML-ready features for predicting loan defaults at 6-month observation period (MOB=6) using only application-time (MOB=0) features.## Executive Summary



### Key Metrics

- **Processing Period**: 24 monthly snapshots (Jan 2023 - Dec 2024)

- **Data Sources**: 4 distinct operational systems  A production-ready data engineering pipeline implementing the **Medallion Architecture** (Bronze → Silver → Gold) for credit risk modeling. This pipeline processes raw customer and loan data into ML-ready features for predicting loan defaults at 6-month observation period (MOB=6) using only application-time (MOB=0) features.A production-ready data engineering pipeline implementing the **Medallion Architecture** (Bronze → Silver → Gold) for credit risk modeling. This pipeline processes raw customer and loan data into ML-ready features for predicting loan defaults at 6-month observation period (MOB=6) using only application-time (MOB=0) features.

- **Features Engineered**: 40+ derived features

- **Target Definition**: Default (DPD ≥ 30 days at MOB=6)

- **Temporal Design**: Application-time features (MOB=0) → 6-month labels (MOB=6)

### Key Metrics### Key Metrics

---

- **Processing Period**: 24 monthly snapshots (Jan 2023 - Dec 2024)- **Processing Period**: 24 monthly snapshots (Jan 2023 - Dec 2024)

## 🎯 Business Problem

- **Data Sources**: 4 distinct operational systems  - **Data Sources**: 4 distinct operational systems

**Objective**: Predict loan default risk at the time of application to support credit approval decisions.

- **Features Engineered**: 40+ derived features- **Features Engineered**: 40+ derived features

**Challenge**: At loan application (MOB=0), the bank must decide whether to approve the loan using only information available at that moment. The success of this decision can only be evaluated 6 months later (MOB=6) by observing whether the customer defaulted.

- **Target Definition**: Default (DPD ≥ 30 days at MOB=6)- **Target Definition**: Default (DPD ≥ 30 days at MOB=6)

**Solution**: Build a temporal ML pipeline that:

1. Uses **only MOB=0 features** (application time) for prediction- **Temporal Design**: Application-time features (MOB=0) → 6-month labels (MOB=6)- **Temporal Design**: Application-time features (MOB=0) → 6-month labels (MOB=6)

2. Labels outcomes using **MOB=6 performance** (6-month observation)

3. Prevents temporal leakage by excluding future loan performance data



---------



## 🏗️ Architecture Overview



### Medallion Architecture## 🎯 Business Problem## 🏗️ Architecture Overview



```

┌──────────────────────────────────────────────────────────────────┐

│                         RAW DATA SOURCES                          │**Objective**: Predict loan default risk at the time of application to support credit approval decisions.### High-Level Architecture

│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │

│  │  Loan    │  │ Customer │  │Financial │  │ Digital  │        │

│  │Management│  │Attributes│  │  Data    │  │Behavior  │        │

│  └──────────┘  └──────────┘  └──────────┘  └──────────┘        │**Challenge**: At loan application (MOB=0), the bank must decide whether to approve the loan using only information available at that moment. The success of this decision can only be evaluated 6 months later (MOB=6) by observing whether the customer defaulted.```

└──────────────────────────────────────────────────────────────────┘

                               │┌─────────────────────────────────────────────────────────────────┐

                               ▼

┌──────────────────────────────────────────────────────────────────┐**Solution**: Build a temporal ML pipeline that:│                        DATA SOURCES                              │

│                    🥉 BRONZE LAYER (Raw)                         │

│              ✓ No Transformation  ✓ Full Lineage                 │1. Uses **only MOB=0 features** (application time) for prediction│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │

│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │

│  │loan_daily│  │attributes│  │financials│  │clickstream│        │2. Labels outcomes using **MOB=6 performance** (6-month observation)│  │loan_daily│  │attributes│  │financials│  │clickstream│       │

│  │  (.csv)  │  │  (.csv)  │  │  (.csv)  │  │  (.csv)  │        │

│  └──────────┘  └──────────┘  └──────────┘  └──────────┘        │3. Prevents temporal leakage by excluding future loan performance data│  └──────────┘  └──────────┘  └──────────┘  └──────────┘       │

└──────────────────────────────────────────────────────────────────┘

                               │└─────────────────────────────────────────────────────────────────┘

                               ▼

┌──────────────────────────────────────────────────────────────────┐---                              │

│              🥈 SILVER LAYER (Cleaned & Enriched)                │

│    ✓ Type Casting  ✓ Data Quality  ✓ Feature Engineering        │                              ▼

│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │

│  │loan_daily│  │attributes│  │financials│  │clickstream│        │## 🏗️ Architecture Overview┌─────────────────────────────────────────────────────────────────┐

│  │(.parquet)│  │(.parquet)│  │(.parquet)│  │(.parquet)│        │

│  └──────────┘  └──────────┘  └──────────┘  └──────────┘        ││                    BRONZE LAYER (Raw Data)                       │

└──────────────────────────────────────────────────────────────────┘

                               │### Medallion Architecture│              Schema Validation | Data Ingestion                  │

                               ▼

┌──────────────────────────────────────────────────────────────────┐│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │

│                🥇 GOLD LAYER (ML-Ready)                          │

│           ✓ Join Operations  ✓ Temporal Separation               │```│  │  .csv    │  │  .csv    │  │  .csv    │  │  .csv    │       │

│                                                                   │

│         ┌──────────────────┐    ┌──────────────────┐            │┌──────────────────────────────────────────────────────────────────┐│  └──────────┘  └──────────┘  └──────────┘  └──────────┘       │

│         │  Feature Store   │    │   Label Store    │            │

│         │    (MOB=0)       │    │    (MOB=6)       │            ││                         RAW DATA SOURCES                          │└─────────────────────────────────────────────────────────────────┘

│         │  Application     │    │  6-Month         │            │

│         │  Time Features   │    │  Observation     │            ││  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │                              │

│         └──────────────────┘    └──────────────────┘            │

└──────────────────────────────────────────────────────────────────┘│  │  Loan    │  │ Customer │  │Financial │  │ Digital  │        │                              ▼

                               │

                               ▼│  │Management│  │Attributes│  │  Data    │  │Behavior  │        │┌─────────────────────────────────────────────────────────────────┐

                   ┌────────────────────────┐

                   │  ML Model Training     ││  └──────────┘  └──────────┘  └──────────┘  └──────────┘        ││              SILVER LAYER (Cleaned & Enriched)                   │

                   │  (Temporal Validation) │

                   └────────────────────────┘└──────────────────────────────────────────────────────────────────┘│        Type Casting | Feature Engineering | Validation           │

```

                               ││  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │

---

                               ▼│  │ .parquet │  │ .parquet │  │ .parquet │  │ .parquet │       │

## 🔄 Pipeline Layers

┌──────────────────────────────────────────────────────────────────┐│  └──────────┘  └──────────┘  └──────────┘  └──────────┘       │

### 1️⃣ Bronze Layer: Raw Data Ingestion

│                    🥉 BRONZE LAYER (Raw)                         │└─────────────────────────────────────────────────────────────────┘

**Purpose**: Capture data exactly as received from source systems with full lineage

│              ✓ No Transformation  ✓ Full Lineage                 │                              │

#### Design Decisions

│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │                              ▼

| Decision | Rationale |

|----------|-----------|│  │loan_daily│  │attributes│  │financials│  │clickstream│        │┌─────────────────────────────────────────────────────────────────┐

| **Format: CSV** | Human-readable, debugging friendly, industry standard |

| **Partitioning: Date-based** | Enables incremental processing and backfilling |│  │  (.csv)  │  │  (.csv)  │  │  (.csv)  │  │  (.csv)  │        ││               GOLD LAYER (ML-Ready Features)                     │

| **No Transformation** | Maintains data lineage and audit trail |

| **Idempotent Writes** | Supports reprocessing without side effects |│  └──────────┘  └──────────┘  └──────────┘  └──────────┘        ││                  Join Operations | Aggregation                   │



#### Data Sources└──────────────────────────────────────────────────────────────────┘│         ┌──────────────────┐    ┌──────────────────┐           │



| Source | Records/Month | Key Identifier | Business Context |                               ││         │  Feature Store   │    │   Label Store    │           │

|--------|---------------|----------------|------------------|

| **loan_daily** | ~1,000 | loan_id + snapshot_date | Daily loan performance tracking |                               ▼│         │   (Features)     │    │   (Targets)      │           │

| **attributes** | ~800 | Customer_ID | Customer demographics (monthly) |

| **financials** | ~800 | Customer_ID | Financial profile (monthly) |┌──────────────────────────────────────────────────────────────────┐│         └──────────────────┘    └──────────────────┘           │

| **clickstream** | ~800 | Customer_ID | Digital engagement features |

│              🥈 SILVER LAYER (Cleaned & Enriched)                │└─────────────────────────────────────────────────────────────────┘

---

│    ✓ Type Casting  ✓ Data Quality  ✓ Feature Engineering        │                              │

### 2️⃣ Silver Layer: Cleaned & Enriched Data

│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │                              ▼

**Purpose**: Apply business logic, data quality rules, and feature engineering

│  │loan_daily│  │attributes│  │financials│  │clickstream│        │                   ┌────────────────────┐

#### Data Quality Operations

│  │(.parquet)│  │(.parquet)│  │(.parquet)│  │(.parquet)│        │                   │   ML Model Training │

##### 1. Data Cleaning

- **Underscore Removal**: Strip trailing/wrapped underscores from numeric fields│  └──────────┘  └──────────┘  └──────────┘  └──────────┘        │                   └────────────────────┘

  - Example: `'40_'` → `'40'`, `'__10000__'` → `'10000'`

  - Affected columns: Age, Annual_Income, Num_of_Loan, Outstanding_Debt, etc.└──────────────────────────────────────────────────────────────────┘```



##### 2. Type Casting                               │

```python

# Strict schema enforcement                               ▼---

Age: IntegerType()

Annual_Income: FloatType()┌──────────────────────────────────────────────────────────────────┐

loan_amt: FloatType()

snapshot_date: DateType()│                🥇 GOLD LAYER (ML-Ready)                          │## 🔄 Pipeline Layers

```

│           ✓ Join Operations  ✓ Temporal Separation               │

##### 3. Data Validation

- **Age Validation**: Set to `null` if Age > 150 or Age < 0│                                                                   │### 1️⃣ Bronze Layer: Raw Data Ingestion

- **Balance Validation**: Set to `null` if |Monthly_Balance| > $1M (data quality markers)

- **Rationale**: `null` preserves data quality signals; `0` implies false information│         ┌──────────────────┐    ┌──────────────────┐            │



##### 4. Feature Engineering│         │  Feature Store   │    │   Label Store    │            │**Purpose**: Capture data exactly as received from source systems



**Loan Performance Features** (in loan_daily):│         │    (MOB=0)       │    │    (MOB=6)       │            │

- `mob`: Month on book (installment_num)

- `dpd`: Days past due (calculated from overdue amount and installment dates)│         │  Application     │    │  6-Month         │            │#### Design Decisions

- `installments_missed`: Count of missed installments

│         │  Time Features   │    │  Observation     │            │- ✅ **Format**: CSV (human-readable, easy debugging)

**Customer Demographic Features** (in attributes):

- `age_group`: Categorical age buckets (18-24, 25-34, ..., 65+)│         └──────────────────┘    └──────────────────┘            │- ✅ **Partitioning**: By snapshot_date (YYYY_MM_DD)

- `occupation_category`: Grouped occupations (Technical, Professional, Management, Education, Other)

└──────────────────────────────────────────────────────────────────┘- ✅ **No Transformation**: Maintain data lineage and auditability

**Financial Health Features** (in financials):

- `debt_to_income_ratio`: (Total_EMI_per_month / Monthly_Inhand_Salary) × 100                               │- ✅ **Idempotent**: Overwrite mode for reprocessing capability

- `income_category`: Income brackets (Low, Medium, High, Very High)

                               ▼

#### Technical Decisions

                   ┌────────────────────────┐#### Data Sources

| Decision | Rationale |

|----------|-----------|                   │  ML Model Training     │

| **Format: Parquet** | Columnar storage, compression, schema evolution |

| **Null Handling** | Preserve data quality signals vs. imputation |                   │  (Temporal Validation) │| Source | Records | Key Fields | Update Frequency |

| **Feature Naming** | Snake_case for consistency |

| **Overwrite Mode** | Enable reprocessing for data corrections |                   └────────────────────────┘|--------|---------|------------|------------------|



---```| **loan_daily** | ~1000/month | loan_id, installment_num, amounts | Daily snapshot |



### 3️⃣ Gold Layer: ML-Ready Feature & Label Stores| **attributes** | ~800/month | Customer_ID, demographics | Monthly snapshot |



**Purpose**: Create temporally correct feature-label pairs for ML training---| **financials** | ~800/month | Customer_ID, income, debts | Monthly snapshot |



#### Critical Design: Temporal Separation| **clickstream** | ~800/month | Customer_ID, 20 features | Daily aggregation |



```## 🔄 Pipeline Layers

Timeline:

│#### Directory Structure

├─ T=0 (MOB=0) ──────────────────────> T=6 months (MOB=6)

│  Application Time                    Observation Time### 1️⃣ Bronze Layer: Raw Data Ingestion```

│  ├─ Feature Collection               ├─ Label Creation

│  │  • Customer demographics          │  • Did customer default?datamart/bronze/

│  │  • Financial profile              │  • DPD >= 30 days?

│  │  • Clickstream behavior           │  • Label: 0 or 1**Purpose**: Capture data exactly as received from source systems with full lineage├── bronze_loan_daily/

│  │  • Loan application details       │

│  │                                   ││   ├── bronze_loan_daily_2023_01_01.csv

│  └─ Features → Feature Store         └─ Labels → Label Store

```#### Design Decisions│   └── bronze_loan_daily_2024_12_01.csv



#### ⚠️ Preventing Temporal Leakage├── bronze_attributes/



**Problem**: Using features from MOB 1-6 would create data leakage, as loan performance data wouldn't be available at application time in production.| Decision | Rationale |├── bronze_financials/



**Solution**:|----------|-----------|└── bronze_clickstream/

1. **Feature Store**: Filter `loan_daily` for `mob == 0` only

2. **Label Store**: Filter `loan_daily` for `mob == 6` to observe outcomes| **Format: CSV** | Human-readable, debugging friendly, industry standard |```

3. **Excluded Features** (not available at MOB=0):

   - `installment_num`, `due_amt`, `paid_amt`, `overdue_amt`, `balance`, `dpd`| **Partitioning: Date-based** | Enables incremental processing and backfilling |

   - These represent future loan performance

| **No Transformation** | Maintains data lineage and audit trail |---

**Business Justification**:

> *"At the time a customer applies for a loan, can we predict if they will default at MOB=6?"*| **Idempotent Writes** | Supports reprocessing without side effects |



The loan officer must make an approval decision **today** (MOB=0) using only current information:### 2️⃣ Silver Layer: Cleaned & Enriched Data

- Customer demographics and credit history

- Financial position and debt ratios#### Data Sources

- Digital engagement behavior

- Loan amount and tenure requested**Purpose**: Apply business logic, data quality rules, and feature engineering



The correctness of this decision is evaluated **6 months later** (MOB=6) by checking if the customer defaulted (DPD ≥ 30 days).| Source | Records/Month | Key Identifier | Business Context |



#### Feature Store Structure|--------|---------------|----------------|------------------|#### Design Decisions



**Dimensions**: ~800 rows × 40+ columns per snapshot| **loan_daily** | ~1,000 | loan_id + snapshot_date | Daily loan performance tracking |- ✅ **Format**: Parquet (columnar, compressed, efficient)



**Feature Categories**:| **attributes** | ~800 | Customer_ID | Customer demographics (monthly) |- ✅ **Schema Enforcement**: Explicit type casting for data quality



| Category | Features | Source Table || **financials** | ~800 | Customer_ID | Financial profile (monthly) |- ✅ **Feature Engineering**: Domain-specific calculated fields

|----------|----------|--------------|

| **Loan Application** | loan_id, loan_amt, tenure, loan_start_date | loan_daily (MOB=0) || **clickstream** | ~800 | Customer_ID | Digital engagement features |- ✅ **Validation**: Null handling, outlier detection

| **Demographics** | customer_age, age_group, occupation_category | attributes |

| **Financial Profile** | Annual_Income, Monthly_Inhand_Salary, debt_to_income_ratio, income_category | financials |

| **Credit Behavior** | Num_of_Loan, Outstanding_Debt, Credit_Utilization_Ratio | financials |

| **Digital Engagement** | fe_1 through fe_20 (clickstream features) | clickstream |---#### Transformations by Table



#### Label Store Structure



**Dimensions**: ~800 rows × 5 columns per snapshot### 2️⃣ Silver Layer: Cleaned & Enriched Data##### 📊 Loan Daily



**Schema**:| Feature | Calculation | Business Logic |

- `loan_id`: Loan identifier

- `Customer_ID`: Customer identifier**Purpose**: Apply business logic, data quality rules, and feature engineering|---------|-------------|----------------|

- `label`: Binary (0=No Default, 1=Default)

- `label_def`: Label definition ("30dpd_6mob")| **mob** | installment_num | Month on Book |

- `snapshot_date`: Observation date

#### Data Quality Operations| **dpd** | Days between snapshot and first missed payment | Days Past Due |

**Label Definition**: 

```python| **installments_missed** | CEIL(overdue_amt / due_amt) | Number of missed payments |

label = 1 if DPD >= 30 at MOB=6 else 0

```##### 1. **Data Cleaning**



---- **Underscore Removal**: Strip trailing/wrapped underscores from numeric fields##### 👤 Attributes



## 📊 Data Transformation Flow  - Example: `'40_'` → `'40'`, `'__10000__'` → `'10000'`| Feature | Calculation | Business Logic |



### Example: Customer Journey  - Affected columns: Age, Annual_Income, Num_of_Loan, Outstanding_Debt, etc.|---------|-------------|----------------|



```python| **age_group** | Age bucketing | "18-24", "25-34", "35-44", "45-54", "55-64", "65+" |

# Raw Data (Bronze Layer)

Customer_ID: "CUST_001"##### 2. **Type Casting**| **occupation_category** | Group similar occupations | "Technical", "Professional", "Management", "Education", "Other" |

Age: "40_"                          # Issue: Trailing underscore

Annual_Income: "52312.68_"          # Issue: Trailing underscore```python

Monthly_Balance: "__-333333333__"   # Issue: Data quality marker

# Strict schema enforcement##### 💰 Financials

# Cleaned Data (Silver Layer)

Customer_ID: "CUST_001"Age: IntegerType()| Feature | Calculation | Business Logic |

Age: 40                             # ✓ Cleaned & cast to int

Annual_Income: 52312.68             # ✓ Cleaned & cast to floatAnnual_Income: FloatType()|---------|-------------|----------------|

Monthly_Balance: null               # ✓ Invalid value set to null

age_group: "35-44"                  # ✓ Derived featureloan_amt: FloatType()| **debt_to_income_ratio** | (Total_EMI / Monthly_Salary) × 100 | DTI percentage |

debt_to_income_ratio: 45.2          # ✓ Calculated feature

snapshot_date: DateType()| **savings_rate** | (Amount_invested / Monthly_Salary) × 100 | Savings percentage |

# ML-Ready Features (Gold Layer - MOB=0)

Customer_ID: "CUST_001"```| **income_category** | Annual income bucketing | "Low", "Medium", "High", "Very High" |

loan_amt: 50000.0

tenure: 36| **credit_health_score** | Based on delayed payments | 100 (none) to 40 (>5 delays) |

customer_age: 40

age_group: "35-44"##### 3. **Data Validation**

Annual_Income: 52312.68

debt_to_income_ratio: 45.2- **Age Validation**: Set to `null` if Age > 150 or Age < 0##### 🖱️ Clickstream

# ... + 30 more features

- **Balance Validation**: Set to `null` if |Monthly_Balance| > $1M (data quality markers)| Feature | Calculation | Business Logic |

# Label (Gold Layer - MOB=6)

loan_id: "LOAN_001"- **Rationale**: `null` preserves data quality signals; `0` implies false information|---------|-------------|----------------|

Customer_ID: "CUST_001"

label: 0                            # No default at 6 months| **total_activity_score** | SUM(fe_1 to fe_20) | Overall digital engagement |

label_def: "30dpd_6mob"

```##### 4. **Feature Engineering**| **avg_feature_value** | total_activity_score / 20 | Average engagement per feature |



---| **engagement_level** | Score bucketing | "Low", "Medium", "High", "Very High" |



## 🚀 Getting Started**Loan Performance Features** (in loan_daily):| **active_feature_count** | COUNT(non-zero features) | Breadth of engagement |



### Prerequisites- `mob`: Month on book (installment_num)



```bash- `dpd`: Days past due (calculated from overdue amount and installment dates)#### Directory Structure

# Required software

Python 3.8+- `installments_missed`: Count of missed installments```

PySpark 3.x

Java 8 or 11 (for Spark)datamart/silver/

```

**Customer Demographic Features** (in attributes):├── silver_loan_daily/

### Installation

- `age_group`: Categorical age buckets (18-24, 25-34, ..., 65+)│   └── silver_loan_daily_2023_01_01.parquet

```bash

# Navigate to project directory- `occupation_category`: Grouped occupations (Technical, Professional, Management, Education, Other)├── silver_attributes/

cd assignment_1

├── silver_financials/

# Install dependencies

pip install -r requirements.txt**Financial Health Features** (in financials):└── silver_clickstream/

```

- `debt_to_income_ratio`: (Total_EMI_per_month / Monthly_Inhand_Salary) × 100```

### Running the Pipeline

- `income_category`: Income brackets (Low, Medium, High, Very High)

```bash

# Execute full pipeline (Bronze → Silver → Gold)---

python main.py

#### Technical Decisions

# Processing summary:

# - Bronze: Ingests 24 monthly snapshots### 3️⃣ Gold Layer: ML-Ready Feature & Label Stores

# - Silver: Cleans and enriches 4 tables × 24 months

# - Gold: Creates feature store (MOB=0) and label store (MOB=6)| Decision | Rationale |

```

|----------|-----------|**Purpose**: Create unified, model-ready datasets for ML pipelines

### Expected Output

| **Format: Parquet** | Columnar storage, compression, schema evolution |

```

Credit Default Prediction - Data Pipeline| **Null Handling** | Preserve data quality signals vs. imputation |#### Design Decisions

======================================================================

Processing 24 snapshots from 2023-01-01 to 2024-12-01| **Feature Naming** | Snake_case for consistency |- ✅ **Star Schema**: Denormalized for ML performance

======================================================================

| **Overwrite Mode** | Enable reprocessing for data corrections |- ✅ **Feature Store**: All predictors in one table

######################################################################

# BRONZE LAYER PROCESSING- ✅ **Label Store**: Separate target variables

######################################################################

[1/24] Processing 2023-01-01...---- ✅ **Join Key**: Customer_ID + loan_id + snapshot_date

...



######################################################################

# SILVER LAYER PROCESSING### 3️⃣ Gold Layer: ML-Ready Feature & Label Stores#### Gold Layer Components

######################################################################

[1/24] Processing 2023-01-01...

...

**Purpose**: Create temporally correct feature-label pairs for ML training```

######################################################################

# GOLD LAYER PROCESSING┌─────────────────────────────────────────────────────────┐

######################################################################

[1/24] Processing 2023-01-01...#### Critical Design: Temporal Separation│                    FEATURE STORE                         │

Feature Store: MOB=0 (Application Time)

Label Store: MOB=6, DPD>=30 (6-month Default Observation)│  ┌────────────┐ ┌────────────┐ ┌────────────┐          │

...

```│  │ Loan Data  │ │  Customer  │ │  Digital   │          │

======================================================================

PIPELINE EXECUTION COMPLETETimeline:│  │  Features  │+│  Financial │+│ Engagement │          │

======================================================================

Feature Store: ~19,200 rows (24 months × 800 loans)││  │            │ │  Features  │ │  Features  │          │

Label Store: ~19,200 rows (24 months × 800 loans)

```├─ T=0 (MOB=0) ──────────────────────> T=6 months (MOB=6)│  └────────────┘ └────────────┘ └────────────┘          │



---│  Application Time                    Observation Time│                                                          │



## 📁 Project Structure│  ├─ Feature Collection               ├─ Label Creation│  35+ Features | Left Join on Customer_ID                │



```│  │  • Customer demographics          │  • Did customer default?└─────────────────────────────────────────────────────────┘

assignment_1/

├── data/                           # Source data files│  │  • Financial profile              │  • DPD >= 30 days?

│   ├── lms_loan_daily.csv

│   ├── features_attributes.csv│  │  • Clickstream behavior           │  • Label: 0 or 1┌─────────────────────────────────────────────────────────┐

│   ├── features_financials.csv

│   └── feature_clickstream.csv│  │  • Loan application details       ││                     LABEL STORE                          │

│

├── datamart/                       # Processed data layers│  │                                   ││                                                          │

│   ├── bronze/                     # Raw data (CSV)

│   │   ├── bronze_loan_daily/│  └─ Features → Feature Store         └─ Labels → Label Store│  loan_id | Customer_ID | label | label_def | snapshot   │

│   │   ├── bronze_attributes/

│   │   ├── bronze_financials/```│  ─────────────────────────────────────────────────────  │

│   │   └── bronze_clickstream/

│   ││  L001    | C123        | 1     | 30dpd_6mob| 2023-06-01│

│   ├── silver/                     # Cleaned data (Parquet)

│   │   ├── silver_loan_daily/#### ⚠️ Preventing Temporal Leakage│                                                          │

│   │   ├── silver_attributes/

│   │   ├── silver_financials/│  Filter: MOB = 6 | Label: DPD >= 30 → Default (1)       │

│   │   └── silver_clickstream/

│   │**Problem**: Using features from MOB 1-6 would create data leakage, as loan performance data wouldn't be available at application time in production.└─────────────────────────────────────────────────────────┘

│   └── gold/                       # ML-ready data (Parquet)

│       ├── feature_store/          # MOB=0 features```

│       └── label_store/            # MOB=6 labels

│**Solution**:

├── utils/                          # Processing modules

│   ├── data_processing_bronze_table.py1. **Feature Store**: Filter `loan_daily` for `mob == 0` only#### Feature Store Schema

│   ├── data_processing_silver_table.py

│   └── data_processing_gold_table.py2. **Label Store**: Filter `loan_daily` for `mob == 6` to observe outcomes

│

├── main.py                         # Pipeline orchestration3. **Excluded Features** (not available at MOB=0):| Category | Features | Count |

├── data_quality_analysis.ipynb     # Data profiling notebook

├── requirements.txt                # Python dependencies   - `installment_num`, `due_amt`, `paid_amt`, `overdue_amt`, `balance`, `dpd`|----------|----------|-------|

└── README.md                       # This file

```   - These represent future loan performance| **Loan Metrics** | loan_amt, tenure, balance, dpd, mob | 7 |



---| **Demographics** | age, age_group, occupation_category | 3 |



## 🔧 Technical Implementation**Business Justification**:| **Financial Health** | income, DTI, savings_rate, credit_health_score | 8 |



### Key Technologies> *"At the time a customer applies for a loan, can we predict if they will default at MOB=6?"*| **Credit Behavior** | num_delayed_payments, credit_utilization, outstanding_debt | 5 |



- **PySpark**: Distributed data processing| **Digital Engagement** | activity_score, engagement_level, active_features | 4 |

- **Parquet**: Columnar storage format (Silver & Gold layers)

- **Pandas**: Data quality analysisThe loan officer must make an approval decision **today** (MOB=0) using only current information:| **Time Variables** | snapshot_date, loan_start_date | 2 |

- **Python**: Pipeline orchestration

- Customer demographics and credit history

### Processing Patterns

- Financial position and debt ratios**Total: 35+ Features**

#### 1. Idempotent Processing

```python- Digital engagement behavior

# All layers support reprocessing

df.write.mode("overwrite").parquet(filepath)- Loan amount and tenure requested#### Label Store Design

```



#### 2. Partition Strategy

```pythonThe correctness of this decision is evaluated **6 months later** (MOB=6) by checking if the customer defaulted (DPD ≥ 30 days).**Target Definition**: 

# Date-based partitioning enables:

# - Incremental processing- **Observation Point**: Month on Book (MOB) = 6

# - Historical backfilling

# - Point-in-time recovery#### Feature Store Structure- **Default Definition**: Days Past Due (DPD) ≥ 30

partition_name = f"table_name_{snapshot_date}.parquet"

```- **Binary Classification**: 1 = Default, 0 = Non-Default



#### 3. Schema Evolution**Dimensions**: ~800 rows × 40+ columns per snapshot

```python

# Parquet format supports schema changes:**Business Rationale**:

# - Adding new columns

# - Type corrections**Feature Categories**:- 6 MOB provides sufficient payment history

# - Backward compatibility

```- 30 DPD aligns with regulatory standards



---| Category | Features | Source Table |- Early warning system for intervention



## 📈 Data Quality Metrics|----------|----------|--------------|



### Bronze Layer Quality Checks| **Loan Application** | loan_id, loan_amt, tenure, loan_start_date | loan_daily (MOB=0) |#### Directory Structure

- ✅ Row counts per snapshot

- ✅ Schema validation| **Demographics** | customer_age, age_group, occupation_category | attributes |```

- ✅ Date range coverage

| **Financial Profile** | Annual_Income, Monthly_Inhand_Salary, debt_to_income_ratio, income_category | financials |datamart/gold/

### Silver Layer Quality Checks

- ✅ Null percentage < threshold| **Credit Behavior** | Num_of_Loan, Outstanding_Debt, Credit_Utilization_Ratio | financials |├── feature_store/

- ✅ Type casting success rate

- ✅ Outlier detection (Age, Balances)| **Digital Engagement** | fe_1 through fe_20 (clickstream features) | clickstream |│   └── feature_store_2023_01_01.parquet

- ✅ Feature distribution analysis

└── label_store/

### Gold Layer Quality Checks

- ✅ Feature-label alignment#### Label Store Structure    └── label_store_2023_01_01.parquet

- ✅ No temporal leakage (MOB validation)

- ✅ Label distribution (class balance)```

- ✅ Join success rate

**Dimensions**: ~800 rows × 5 columns per snapshot

---

---

## 🎓 ML Pipeline Considerations

**Schema**:

### Temporal Validation Strategy

- `loan_id`: Loan identifier## 🛠️ Technical Stack

**Critical**: When building ML models, maintain temporal separation:

- `Customer_ID`: Customer identifier

```python

# Training data: Use snapshots 2023-01 to 2024-06- `label`: Binary (0=No Default, 1=Default)### Technologies

train_features = feature_store[snapshot_date <= "2024-06-01"]

train_labels = label_store[snapshot_date <= "2024-06-01"]- `label_def`: Label definition ("30dpd_6mob")- **Framework**: Apache PySpark 3.x



# Validation data: Use snapshots 2024-07 to 2024-09- `snapshot_date`: Observation date- **Language**: Python 3.8+

val_features = feature_store["2024-07-01" <= snapshot_date <= "2024-09-01"]

val_labels = label_store["2024-07-01" <= snapshot_date <= "2024-09-01"]- **Storage Format**: CSV → Parquet



# Test data: Use snapshots 2024-10 to 2024-12**Label Definition**: - **Orchestration**: Python scripts (production: Airflow/Databricks)

test_features = feature_store[snapshot_date >= "2024-10-01"]

test_labels = label_store[snapshot_date >= "2024-10-01"]```python

```

label = 1 if DPD >= 30 at MOB=6 else 0### Key Libraries

**Why Temporal Splits?**

- Prevents data leakage from future to past``````python

- Simulates real-world deployment

- Tests model's ability to generalize to new time periodspyspark.sql.functions  # Data transformations



### Important Note on Temporal Leakage Prevention---pyspark.sql.types      # Schema definitions



**For the ML pipeline, you should only use features available at the point of application (MOB=0).**datetime               # Date operations



The business scenario is: *"At the time a customer applies for a loan, can we predict if they will default at MOB=6?"* ## 📊 Data Transformation Flowos, glob               # File management



This means the bank needs to make an approval decision using only information available at application time (customer attributes, financials, clickstream behavior at MOB=0), and the label (whether they defaulted) comes from observing their loan performance at MOB=6. ```



Using any features from MOB=1 through MOB=6 would be temporal leakage. Think of it like a loan officer who must decide today (MOB=0) whether to approve a loan, but can only know if that decision was correct 6 months later (MOB=6).### Example: Customer Journey



### Feature Importance Analysis---



**Expected High-Impact Features**:```python

1. `debt_to_income_ratio`: Payment capacity indicator

2. `Credit_Utilization_Ratio`: Credit stress signal# Raw Data (Bronze Layer)## 📊 Data Quality & Validation

3. `Outstanding_Debt`: Existing obligations

4. `income_category`: Income stabilityCustomer_ID: "CUST_001"

5. `loan_amt`: Loan size relative to profile

Age: "40_"                          # Issue: Trailing underscore### Silver Layer Quality Checks

### Model Monitoring

Annual_Income: "52312.68_"          # Issue: Trailing underscore1. **Schema Validation**: Explicit type casting for all columns

**Post-Deployment Metrics**:

- Monthly default rate vs. predictionsMonthly_Balance: "__-333333333__"   # Issue: Data quality marker2. **Null Handling**: COALESCE for safe arithmetic operations

- Feature drift detection

- Model performance degradation alerts3. **Business Rules**: 

- Population stability index (PSI)

# Cleaned Data (Silver Layer)   - DTI ratio only calculated when salary > 0

---

Customer_ID: "CUST_001"   - DPD only for loans with overdue amounts

## 🔒 Data Privacy & Compliance

Age: 40                             # ✓ Cleaned & cast to int   - Feature counts exclude null/zero values

### PII Handling

- ✅ **Excluded from Gold Layer**: Name, SSNAnnual_Income: 52312.68             # ✓ Cleaned & cast to float

- ✅ **Pseudonymization**: Customer_ID, loan_id

- ✅ **Access Control**: Role-based permissionsMonthly_Balance: null               # ✓ Invalid value set to null### Gold Layer Quality Checks



### Data Retentionage_group: "35-44"                  # ✓ Derived feature1. **Join Validation**: Left joins preserve all loan records

- **Bronze Layer**: 7 years (regulatory compliance)

- **Silver Layer**: 5 years (audit trail)debt_to_income_ratio: 45.2          # ✓ Calculated feature2. **Label Integrity**: MOB filter before label creation

- **Gold Layer**: 3 years (model training)

3. **Feature Completeness**: Track missing value rates

---

# ML-Ready Features (Gold Layer - MOB=0)

## 📝 Best Practices Implemented

Customer_ID: "CUST_001"---

1. ✅ **Separation of Concerns**: Modular processing functions

2. ✅ **Idempotency**: Safe reprocessing without duplicatesloan_amt: 50000.0

3. ✅ **Schema Enforcement**: Strict type casting in Silver layer

4. ✅ **Temporal Correctness**: MOB-based feature-label separationtenure: 36## 🚀 Usage & Execution

5. ✅ **Data Lineage**: Bronze → Silver → Gold traceability

6. ✅ **Error Handling**: Null handling vs. error propagationcustomer_age: 40

7. ✅ **Documentation**: Inline comments and external docs

8. ✅ **Version Control**: Git-based code managementage_group: "35-44"### Running the Pipeline



---Annual_Income: 52312.68



## 🚦 Future Enhancementsdebt_to_income_ratio: 45.2```bash



### Phase 2 Roadmap# ... + 30 more features# Navigate to project directory

- [ ] **Incremental Processing**: Process only new snapshots

- [ ] **Data Quality Dashboard**: Automated quality reportscd /Users/vishalmishra/MyDocuments/SMU_MITB/Term-4/MLE/Assignment/assignment_1

- [ ] **Feature Store Versioning**: Track feature evolution

- [ ] **A/B Testing Framework**: Compare feature sets# Label (Gold Layer - MOB=6)

- [ ] **Real-time Scoring**: Streaming prediction pipeline

- [ ] **Automated Retraining**: Scheduled model updatesloan_id: "LOAN_001"# Execute full pipeline

- [ ] **Model Explainability**: SHAP values for predictions

Customer_ID: "CUST_001"python main.py

---

label: 0                            # No default at 6 months```

## 📄 License

label_def: "30dpd_6mob"

Internal Use Only - Confidential

```### Configuration

---



**Last Updated**: October 2025  

**Version**: 1.0.0  ---Edit `main.py` to customize:

**Pipeline Status**: Production Ready ✅

```python

## 🚀 Getting StartedCONFIG = {

    'start_date': "2023-01-01",      # Start of backfill

### Prerequisites    'end_date': "2024-12-01",        # End of backfill

    'base_dir': "datamart",          # Output directory

```bash    'dpd_threshold': 30,             # Default definition (DPD)

# Required software    'mob_threshold': 6,              # Observation point (MOB)

Python 3.8+}

PySpark 3.x```

Java 8 or 11 (for Spark)

```### Output Summary



### Installation```

FEATURE STORE SUMMARY

```bash======================================================================

# Navigate to project directoryFeature Store row count: 24,000

cd assignment_1Feature Store columns: 35



# Install dependenciesLABEL STORE SUMMARY

pip install -r requirements.txt======================================================================

```Label Store row count: 3,200

Label distribution:

### Running the Pipeline+-----+-----+

|label|count|

```bash+-----+-----+

# Execute full pipeline (Bronze → Silver → Gold)|    0| 2800|

python main.py|    1|  400|

+-----+-----+

# Processing summary:```

# - Bronze: Ingests 24 monthly snapshots

# - Silver: Cleans and enriches 4 tables × 24 months---

# - Gold: Creates feature store (MOB=0) and label store (MOB=6)

```## 📈 Business Value & Use Cases



### Expected Output### Primary Use Cases

1. **Credit Risk Modeling**: Predict loan defaults at 6 MOB

```2. **Early Warning System**: Identify high-risk customers

Credit Default Prediction - Data Pipeline3. **Portfolio Analytics**: Monitor portfolio health metrics

======================================================================4. **Customer Segmentation**: Group customers by risk profile

Processing 24 snapshots from 2023-01-01 to 2024-12-01

======================================================================### Key Performance Indicators

- **Default Rate**: ~12.5% at 6 MOB

######################################################################- **Feature Coverage**: 100% of active loans

# BRONZE LAYER PROCESSING- **Data Freshness**: Monthly snapshots

######################################################################- **Processing Time**: <5 minutes for 24 months

[1/24] Processing 2023-01-01...

...---



######################################################################## 🔐 Data Governance

# SILVER LAYER PROCESSING

######################################################################### Privacy & Security

[1/24] Processing 2023-01-01...- ✅ **PII Exclusion**: Name and SSN removed in Gold layer

...- ✅ **Access Control**: Layer-based permissions

- ✅ **Audit Trail**: Snapshot dates for lineage tracking

######################################################################- ✅ **Encryption**: At-rest (Parquet) and in-transit

# GOLD LAYER PROCESSING

######################################################################### Data Retention

[1/24] Processing 2023-01-01...- **Bronze**: 2 years (compliance)

Feature Store: MOB=0 (Application Time)- **Silver**: 3 years (analytics)

Label Store: MOB=6, DPD>=30 (6-month Default Observation)- **Gold**: 5 years (ML model history)

...

---

======================================================================

PIPELINE EXECUTION COMPLETE## 🔄 Pipeline Monitoring

======================================================================

Feature Store: ~19,200 rows (24 months × 800 loans)### Success Metrics

Label Store: ~19,200 rows (24 months × 800 loans)```python

```✓ Bronze layer: 24 snapshots processed

✓ Silver layer: 24 snapshots processed  

---✓ Gold layer: 24 snapshots processed

```

## 📁 Project Structure

### Error Handling

```- File-level error isolation

assignment_1/- Graceful degradation on failures

├── data/                           # Source data files- Detailed logging for debugging

│   ├── lms_loan_daily.csv

│   ├── features_attributes.csv---

│   ├── features_financials.csv

│   └── feature_clickstream.csv## 🎯 Future Enhancements

│

├── datamart/                       # Processed data layers### Roadmap

│   ├── bronze/                     # Raw data (CSV)1. **Real-time Streaming**: Kafka integration for live updates

│   │   ├── bronze_loan_daily/2. **Advanced Features**: 

│   │   ├── bronze_attributes/   - Rolling window aggregations

│   │   ├── bronze_financials/   - Trend indicators (3-month, 6-month)

│   │   └── bronze_clickstream/   - Cross-customer features (peer comparison)

│   │3. **ML Integration**:

│   ├── silver/                     # Cleaned data (Parquet)   - Auto-feature selection

│   │   ├── silver_loan_daily/   - Feature importance tracking

│   │   ├── silver_attributes/   - Model drift detection

│   │   ├── silver_financials/4. **Orchestration**: Migrate to Airflow DAGs

│   │   └── silver_clickstream/5. **Cloud Deployment**: AWS/Azure/GCP integration

│   │

│   └── gold/                       # ML-ready data (Parquet)---

│       ├── feature_store/          # MOB=0 features

│       └── label_store/            # MOB=6 labels## 📚 Project Structure

│

├── utils/                          # Processing modules```

│   ├── data_processing_bronze_table.pyassignment_1/

│   ├── data_processing_silver_table.py├── main.py                          # Pipeline orchestration

│   └── data_processing_gold_table.py├── utils/

││   ├── data_processing_bronze_table.py

├── main.py                         # Pipeline orchestration│   ├── data_processing_silver_table.py

├── data_quality_analysis.ipynb     # Data profiling notebook│   └── data_processing_gold_table.py

├── requirements.txt                # Python dependencies├── data/                            # Source data

└── README.md                       # This file│   ├── loan_daily.csv

```│   ├── attributes.csv

│   ├── financials.csv

---│   └── clickstream.csv

├── datamart/                        # Output layers

## 🔧 Technical Implementation│   ├── bronze/

│   ├── silver/

### Key Technologies│   └── gold/

└── README.md                        # This file

- **PySpark**: Distributed data processing```

- **Parquet**: Columnar storage format (Silver & Gold layers)

- **Pandas**: Data quality analysis---

- **Python**: Pipeline orchestration

## 👥 Team & Contact

### Processing Patterns

**Project**: SMU MITB - Machine Learning Engineering

#### 1. Idempotent Processing**Term**: 4 | **Assignment**: 1

```python**Author**: Vishal Mishra

# All layers support reprocessing

df.write.mode("overwrite").parquet(filepath)---

```

## 📝 License & Acknowledgments

#### 2. Partition Strategy

```pythonThis pipeline implements industry best practices from:

# Date-based partitioning enables:- **Databricks Medallion Architecture**

# - Incremental processing- **MLOps Principles** (Feature Store Pattern)

# - Historical backfilling- **Data Engineering Patterns** (ELT over ETL)

# - Point-in-time recovery

partition_name = f"table_name_{snapshot_date}.parquet"---

```

## 🔗 References

#### 3. Schema Evolution

```python1. [Medallion Architecture](https://www.databricks.com/glossary/medallion-architecture)

# Parquet format supports schema changes:2. [Feature Store Design](https://www.featurestore.org/)

# - Adding new columns3. [PySpark Documentation](https://spark.apache.org/docs/latest/api/python/)

# - Type corrections4. [Credit Risk Modeling Best Practices](https://www.risk.net/)

# - Backward compatibility

```---



---**Last Updated**: January 2025

**Pipeline Version**: 1.0.0

## 📈 Data Quality Metrics

### Bronze Layer Quality Checks
- ✅ Row counts per snapshot
- ✅ Schema validation
- ✅ Date range coverage

### Silver Layer Quality Checks
- ✅ Null percentage < threshold
- ✅ Type casting success rate
- ✅ Outlier detection (Age, Balances)
- ✅ Feature distribution analysis

### Gold Layer Quality Checks
- ✅ Feature-label alignment
- ✅ No temporal leakage (MOB validation)
- ✅ Label distribution (class balance)
- ✅ Join success rate

---

## 🎓 ML Pipeline Considerations

### Temporal Validation Strategy

**Critical**: When building ML models, maintain temporal separation:

```python
# Training data: Use snapshots 2023-01 to 2024-06
train_features = feature_store[snapshot_date <= "2024-06-01"]
train_labels = label_store[snapshot_date <= "2024-06-01"]

# Validation data: Use snapshots 2024-07 to 2024-09
val_features = feature_store["2024-07-01" <= snapshot_date <= "2024-09-01"]
val_labels = label_store["2024-07-01" <= snapshot_date <= "2024-09-01"]

# Test data: Use snapshots 2024-10 to 2024-12
test_features = feature_store[snapshot_date >= "2024-10-01"]
test_labels = label_store[snapshot_date >= "2024-10-01"]
```

**Why Temporal Splits?**
- Prevents data leakage from future to past
- Simulates real-world deployment
- Tests model's ability to generalize to new time periods

### Important Note on Temporal Leakage Prevention

**For the ML pipeline, you should only use features available at the point of application (MOB=0).**

The business scenario is: *"At the time a customer applies for a loan, can we predict if they will default at MOB=6?"* 

This means the bank needs to make an approval decision using only information available at application time (customer attributes, financials, clickstream behavior at MOB=0), and the label (whether they defaulted) comes from observing their loan performance at MOB=6. 

Using any features from MOB=1 through MOB=6 would be temporal leakage. Think of it like a loan officer who must decide today (MOB=0) whether to approve a loan, but can only know if that decision was correct 6 months later (MOB=6).

### Feature Importance Analysis

**Expected High-Impact Features**:
1. `debt_to_income_ratio`: Payment capacity indicator
2. `Credit_Utilization_Ratio`: Credit stress signal
3. `Outstanding_Debt`: Existing obligations
4. `income_category`: Income stability
5. `loan_amt`: Loan size relative to profile

### Model Monitoring

**Post-Deployment Metrics**:
- Monthly default rate vs. predictions
- Feature drift detection
- Model performance degradation alerts
- Population stability index (PSI)

---

## 🔒 Data Privacy & Compliance

### PII Handling
- ✅ **Excluded from Gold Layer**: Name, SSN
- ✅ **Pseudonymization**: Customer_ID, loan_id
- ✅ **Access Control**: Role-based permissions

### Data Retention
- **Bronze Layer**: 7 years (regulatory compliance)
- **Silver Layer**: 5 years (audit trail)
- **Gold Layer**: 3 years (model training)

---

## 📝 Best Practices Implemented

1. ✅ **Separation of Concerns**: Modular processing functions
2. ✅ **Idempotency**: Safe reprocessing without duplicates
3. ✅ **Schema Enforcement**: Strict type casting in Silver layer
4. ✅ **Temporal Correctness**: MOB-based feature-label separation
5. ✅ **Data Lineage**: Bronze → Silver → Gold traceability
6. ✅ **Error Handling**: Null handling vs. error propagation
7. ✅ **Documentation**: Inline comments and external docs
8. ✅ **Version Control**: Git-based code management

---

## 🚦 Future Enhancements

### Phase 2 Roadmap
- [ ] **Incremental Processing**: Process only new snapshots
- [ ] **Data Quality Dashboard**: Automated quality reports
- [ ] **Feature Store Versioning**: Track feature evolution
- [ ] **A/B Testing Framework**: Compare feature sets
- [ ] **Real-time Scoring**: Streaming prediction pipeline
- [ ] **Automated Retraining**: Scheduled model updates
- [ ] **Model Explainability**: SHAP values for predictions

---

## 📄 License

Internal Use Only - Confidential

---

**Last Updated**: October 2025  
**Version**: 1.0.0  
**Pipeline Status**: Production Ready ✅
