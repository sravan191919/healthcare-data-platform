# Data Dictionary

---

# Project Information

| Item | Details |
|------|---------|
| Project | Healthcare Enterprise Data Platform |
| Version | 1.0 |
| Status | Draft |

---

# Purpose

The Data Dictionary defines every field used in the platform.

---

# Patient Table

Table Name

dim_patient

| Column | Type | Description |
|---------|------|------------|
| patient_key | BIGINT | Surrogate Key |
| patient_id | STRING | Business Key |
| first_name | STRING | Patient First Name |
| last_name | STRING | Patient Last Name |
| gender | STRING | Gender |
| birth_date | DATE | Date of Birth |
| city | STRING | City |
| state | STRING | State |
| zip_code | STRING | ZIP Code |

---

# Provider Table

Table Name

dim_provider

| Column | Type | Description |
|---------|------|------------|
| provider_key | BIGINT | Surrogate Key |
| provider_id | STRING | Business Key |
| provider_name | STRING | Provider Name |
| specialty | STRING | Provider Specialty |
| hospital_name | STRING | Hospital |

---

# Claims Table

Table Name

fact_claims

| Column | Type | Description |
|---------|------|------------|
| claim_key | BIGINT | Surrogate Key |
| claim_id | STRING | Claim Number |
| patient_key | BIGINT | FK Patient |
| provider_key | BIGINT | FK Provider |
| claim_amount | DECIMAL | Claim Amount |
| paid_amount | DECIMAL | Paid Amount |
| diagnosis_code | STRING | ICD Code |
| procedure_code | STRING | CPT Code |

---

# Pharmacy Table

fact_pharmacy_transactions

| Column | Type | Description |
|---------|------|------------|
| prescription_id | STRING | Prescription Number |
| medication_name | STRING | Medication |
| quantity | INTEGER | Quantity |
| dispense_date | DATE | Dispense Date |

---

# Laboratory Table

fact_lab_results

| Column | Type | Description |
|---------|------|------------|
| lab_result_id | STRING | Lab Identifier |
| patient_key | BIGINT | FK Patient |
| test_name | STRING | Laboratory Test |
| result_value | DECIMAL | Numeric Result |
| result_date | DATE | Date of Test |

---

# Audit Columns

Every table contains:

| Column | Description |
|---------|-------------|
| execution_id | Pipeline Execution |
| source_system | Source System |
| source_file | Source File |
| ingestion_timestamp | Timestamp |
| load_date | Load Date |
| record_hash | Duplicate Detection |

---

# Naming Standards

Tables

dim_patient

dim_provider

fact_claims

fact_lab_results

fact_pharmacy_transactions

Columns

snake_case

lower_case

No spaces

No special characters

---

# Approval

| Role | Status |
|------|--------|
| Data Architect | Pending |
| Technical Lead | Pending |
