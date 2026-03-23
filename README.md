# Automated Customer KYC Matching Pipeline & Reconciliation System (Google BigQuery)

## Overview
This project implements a **scalable data matching pipeline** designed to identify and reconcile customer records across multiple datasets. By leveraging National ID (NRC) and mobile numbers, the pipeline standardizes incoming data, applies a multi-level matching strategy, and ensures incremental updates to maintain an accurate, deduplicated customer profile.

## Problem Statement
Customer data is often fragmented across multiple systems, leading to:
- Duplicate customer records
- Inconsistent NRC/ID formatting
- Multiple mobile numbers per user
- Missing or partially filled attributes

These issues cause inaccurate reporting, poor customer insights, and operational inefficiencies.

## Solution Overview
The pipeline addresses these challenges by:
1. **Standardizing and cleaning** incoming customer data.
2. **Matching records** using defined criteria with confidence levels.
3. **Classifying** match status (`Matched Customers`, `Partially Matched (Mobile)`, `Unmatched Customers`).
4. **Updating datasets incrementally** via `MERGE` operations, enabling continuous processing.

## Data Sources
- `DATA_MARTS_1.CUSTOMER_PROFILE`
- `DATA_MARTS_2.CUSTOMER_PROFILE`
- Staging table for incoming customer data (`staging_customers`)

## Data Processing Workflow

### 1. Data Loading
Extract customer data from multiple source systems, filtering only active, valid, non-test records.

### 2. Data Cleaning & Standardization
- Trim and normalize text fields (names, IDs).
- Standardize NRC format using regex:
  ```sql
  REGEXP_REPLACE(NRC, r'\([^)]*\)', '()')

### 3. Matching Strategy
| Level | Match Type                | Criteria                     | Confidence | Match Status               |
|-------|---------------------------|------------------------------|------------|----------------------------|
| 1     | Exact Match               | NRC + Mobile Number          | High       | `Matched Customers`        |
| 2     | Mobile-Based Match        | Mobile Number only           | Medium     | `Partially Matched (Mobile)` |
| 3     | No Match                  | No match criteria met        | Low        | `Unmatched Customers`      |

### 4. Data Tagging
Each record is tagged with a `Match_Status` based on the matching level.

### 5. Incremental Update (MERGE)
Daily updates are performed using `MERGE` to avoid duplicates and ensure only new or changed records are processed:

```sql
MERGE target_table AS target
USING source_table AS source
ON target.Contact_Number = source.Contact_Number
   AND target.NRC = source.NRC
WHEN MATCHED THEN UPDATE
SET target.Match_Status = source.Match_Status;
```
### 6. Output Extraction
- **Fully matched records** (high confidence) for downstream use.
- **Partially matched records** for manual review.
- **Unmatched records** for further investigation.

### Automation Strategy
- Designed for **daily execution**.
- Supports scheduling via **BigQuery Scheduled Queries**, **Apache Airflow**, or **Cron jobs**.
- Implements **incremental matching** to process only new or unprocessed records.

### Business Impact
- **Improved customer data accuracy** and consistency.
- **Reduced duplicate customer profiles**.
- **Enhanced KYC validation** processes.
- **Better decision-making** for analytics, marketing, and risk management.

### Key Features
- **Multi-level matching** (Exact + Partial)
- **Data normalization** using regex
- **Efficient mobile number joins** with `UNNEST`
- **Incremental updates** via `MERGE`
- **Scalable** for large datasets

### Tech Stack
- **SQL (BigQuery)**
- Data Warehousing Concepts
- Data Cleaning & Transformation
- Incremental Data Processing

### Skills Demonstrated
- Data Engineering (ETL/ELT pipeline design)
- Customer Identity Resolution
- Data Quality Management
- Query Optimization
- Real-world business logic implementation

### Future Improvements
- Implement **fuzzy matching** for name similarity (Levenshtein distance).
- Add **scoring model** for match confidence.
- Build **dashboard** for match monitoring and visualization.
- Integrate **real-time streaming data**.
- Automate **reporting & notifications** (Excel/CSV exports, email summaries).
