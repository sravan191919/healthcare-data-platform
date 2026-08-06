# Source to Target Mapping Document

---

# Project Information

| Item | Details |
|------|---------|
| Project Name | Healthcare Enterprise Data Platform |
| Client | HealthFirst Analytics (Portfolio Project) |
| Document | Source to Target Mapping |
| Prepared By | Sravan Vaddepudi |
| Version | 1.0 |
| Status | Draft |

---

# Purpose

This document defines how every source column is transformed and loaded into the target warehouse.

It serves as the implementation guide for ETL developers.

---

# Source Systems

1. Patient
2. Provider
3. Claims
4. Pharmacy
5. Laboratory

---

# Patient Source → Target Mapping

Source File

patient_YYYYMMDD.csv

Target Table

dim_patient

| Source Column | Target Column | Data Type | Transformation | Nullable |
|--------------|---------------|-----------|----------------|----------|
| patient_id | patient_id | STRING | Direct | No |
| first_name | first_name | STRING | Trim() | No |
| last_name | last_name | STRING | Trim() | No |
| gender | gender | STRING | Upper Case | No |
| dob | birth_date | DATE | Convert to YYYY-MM-DD | No |
| city | city | STRING | Trim() | Yes |
| state | state | STRING | Upper Case | Yes |
| zip | zip_code | STRING | Remove Spaces | Yes |
| phone | phone | STRING | Remove Special Characters | Yes |
| created_date | created_date | DATE | Convert Date | No |

---

# Provider Source → Target Mapping

Target Table

dim_provider

| Source | Target | Transformation |
|---------|---------|---------------|
| provider_id | provider_id | Direct |
| provider_name | provider_name | Trim |
| specialty | specialty | Upper |
| hospital | hospital_name | Trim |
| city | city | Trim |
| state | state | Upper |

---

# Claims Source → Target Mapping

Target Table

fact_claims

| Source | Target | Transformation |
|---------|---------|---------------|
| claim_id | claim_id | Direct |
| patient_id | patient_key | Lookup |
| provider_id | provider_key | Lookup |
| claim_amount | claim_amount | Decimal |
| paid_amount | paid_amount | Decimal |
| diagnosis_code | diagnosis_code | Upper |
| procedure_code | procedure_code | Upper |
| claim_date | claim_date | Date |

---

# Pharmacy Mapping

Target Table

fact_pharmacy_transactions

| Source | Target | Transformation |
|---------|---------|---------------|
| prescription_id | prescription_id | Direct |
| patient_id | patient_key | Lookup |
| medication | medication_name | Trim |
| quantity | quantity | Integer |
| dispense_date | dispense_date | Date |

---

# Laboratory Mapping

Target Table

fact_lab_results

| Source | Target | Transformation |
|---------|---------|---------------|
| lab_result_id | lab_result_id | Direct |
| patient_id | patient_key | Lookup |
| test_name | test_name | Upper |
| result_value | result_value | Decimal |
| result_date | result_date | Date |

---

# Standard Transformations

- Trim spaces
- Convert NULL values
- Upper case codes
- Standardize dates
- Remove duplicate records
- Remove invalid characters
- Validate primary keys
- Validate foreign keys
- Generate surrogate keys
- Generate audit columns

---

# Audit Columns Added

Every table contains:

execution_id

load_date

ingestion_timestamp

source_system

source_file

record_hash

created_by

updated_timestamp

---

# Approval

| Role | Status |
|------|--------|
| Data Architect | Pending |
| ETL Lead | Pending |
| QA Lead | Pending |
