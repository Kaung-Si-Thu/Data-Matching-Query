# Automated Customer Identity (KYC) Matching Pipeline & Reconciliation System (BigQuery SQL)

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
- `BUSINESS_MARTS.CUSTOMER_PROFILE_COMBI`
- `RISK_MARTS.CUSTOMER_PROFILE_COMBI`
- Staging table for incoming customer data (`staging_customers`)

## Data Processing Workflow

### 1. Data Loading
Extract customer data from multiple source systems, filtering only active, valid, non-test records.

### 2. Data Cleaning & Standardization
- Trim and normalize text fields (names, IDs).
- Standardize NRC format using regex:
  ```sql
  REGEXP_REPLACE(NRC, r'\([^)]*\)', '()')
