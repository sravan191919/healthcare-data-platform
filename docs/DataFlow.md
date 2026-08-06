# Data Flow Design

---

# Project Information

| Item | Details |
|------|---------|
| Project Name | Healthcare Enterprise Data Platform |
| Document Type | Data Flow Design |
| Prepared By | Sravan Vaddepudi |
| Role | Data Engineer |
| Version | 1.0 |
| Status | Draft |
| Date | August 2026 |

---

# 1. Purpose

This document describes how healthcare data moves through the Healthcare Enterprise Data Platform.

It covers:

- Source-to-Landing flow
- Landing-to-Raw flow
- Raw-to-Bronze flow
- Bronze-to-Silver flow
- Silver-to-Gold flow
- Gold-to-Snowflake flow
- Metadata flow
- Error flow
- Monitoring flow
- Reprocessing flow

---

# 2. End-to-End Data Flow

```text
Source Systems
      |
      v
Landing Zone
      |
      v
File Validation
      |
      +----------- Invalid ----------> Quarantine
      |
     Valid
      |
      v
Raw Layer
      |
      v
Bronze Transformation
      |
      v
Silver Transformation
      |
      v
Gold Transformation
      |
      v
Snowflake Warehouse
      |
      v
Power BI / Analytics / Machine Learning
```

---

# 3. Source-to-Landing Flow

## Input Sources

- Hospitals
- EHR systems
- Claims systems
- Pharmacies
- Laboratories
- Provider systems
- REST APIs
- Kafka topics

## Process

1. Source system creates or publishes data.
2. Data is transferred to the Landing Zone.
3. Arrival timestamp is captured.
4. Source identifier is recorded.
5. File remains unmodified.
6. Ingestion framework discovers the file.

## Example

```text
Hospital A
   |
patient_20260806.csv
   |
data/landing/patient/
```

---

# 4. Landing-to-Raw Flow

```text
Landing File
     |
     v
Resolve Source
     |
     v
Validate Filename
     |
     v
Validate Extension
     |
     v
Validate File Size
     |
     v
Check Empty File
     |
     v
Generate Checksum
     |
     v
Check Processing History
     |
     v
Read File
     |
     v
Validate Schema
     |
     +------ Failure ------> Quarantine
     |
   Success
     |
     v
Move File to Raw
     |
     v
Update Metadata
```

---

# 5. Validation Failure Flow

```text
Invalid File
     |
     v
Capture Failure Reason
     |
     v
Move to Quarantine
     |
     v
Write Failed Metadata
     |
     v
Write Error Log
     |
     v
Continue Processing Remaining Files
```

Possible reasons:

- Invalid filename
- Unsupported extension
- Empty file
- Oversized file
- Corrupted file
- Missing columns
- Invalid data type
- Duplicate file
- Unrecognized source

---

# 6. Raw-to-Bronze Flow

## Input

Validated raw source files.

## Processing

- Read raw source data.
- Convert column names to snake case.
- Cast configured data types.
- Normalize timestamps.
- Add technical metadata.
- Add record hash.
- Capture malformed rows.
- Write partitioned Bronze data.

## Technical Columns

- execution_id
- source_system
- source_file
- ingestion_timestamp
- load_date
- record_hash

## Example

```text
data/raw/claims/load_date=2026-08-06/
             |
             v
Bronze Transformer
             |
             v
data/bronze/claims/source_system=hospital_a/load_date=2026-08-06/
```

---

# 7. Bronze-to-Silver Flow

## Input

Technically standardized Bronze data.

## Processing

- Remove duplicate records.
- Apply null-handling rules.
- Standardize dates.
- Standardize codes.
- Validate identifiers.
- Apply business rules.
- Check referential integrity.
- Produce conformed entities.
- Write rejected records separately when required.

## Examples

### Patient

- Standardize gender values.
- Validate date of birth.
- Remove duplicate patient identifiers.
- Normalize state codes.

### Claims

- Validate claim amount.
- Validate claim status.
- Check provider identifier.
- Check patient identifier.
- Standardize diagnosis and procedure codes.

### Laboratory

- Standardize test units.
- Validate result timestamps.
- Standardize abnormal indicators.

---

# 8. Silver-to-Gold Flow

## Input

Cleaned and conformed Silver data.

## Processing

- Build dimensions.
- Build facts.
- Generate surrogate keys.
- Aggregate business measures.
- Apply reporting logic.
- Publish business-ready datasets.

## Planned Gold Objects

```text
dim_patient
dim_provider
dim_hospital
dim_date
fact_claims
fact_encounters
fact_lab_results
fact_pharmacy_transactions
claims_summary
hospital_performance_summary
```

---

# 9. Gold-to-Snowflake Flow

```text
Gold Dataset
     |
     v
Write Export File
     |
     v
Upload to Snowflake Stage
     |
     v
COPY INTO Target Table
     |
     v
Validate Row Counts
     |
     v
Update Audit Metadata
     |
     v
Publish Views
```

Validation includes:

- Source count
- Target count
- Rejected count
- Duplicate count
- Null threshold
- Load status

---

# 10. Metadata Flow

```text
Pipeline Starts
     |
     v
Generate Execution ID
     |
     v
Insert RUNNING Status
     |
     v
Process File
     |
     v
Capture Counts
     |
     v
Capture Quality Results
     |
     v
Update SUCCESS / FAILED Status
     |
     v
Publish Metrics
```

Metadata tables:

- pipeline_execution_log
- file_processing_history
- data_quality_result
- schema_version_history

---

# 11. Data Quality Flow

```text
Input Data
    |
    v
Execute Configured Rules
    |
    +-- Null Check
    +-- Duplicate Check
    +-- Accepted Values
    +-- Range Check
    +-- Date Check
    +-- Referential Integrity
    |
    v
Generate Rule Results
    |
    v
Compare Failure Threshold
    |
    +--- Within Threshold ---> Continue
    |
    +--- Above Threshold ----> Reject or Stop
```

---

# 12. Error Flow

## Validation Error

```text
Validation Failure
      |
      v
Quarantine File
      |
      v
Write Error Metadata
      |
      v
Continue Remaining Files
```

## Temporary System Error

```text
System Failure
      |
      v
Retry
      |
      v
Retry Successful?
   /       \
 Yes       No
 |          |
Continue   Mark Failed
            |
            v
          Alert
```

## Critical Failure

```text
Configuration or Platform Failure
              |
              v
          Stop Pipeline
              |
              v
        Log Critical Error
              |
              v
          Send Alert
```

---

# 13. Reprocessing Flow

```text
Quarantined or Failed File
          |
          v
Issue Corrected
          |
          v
Approved for Reprocessing
          |
          v
Return to Landing Zone
          |
          v
Generate New Execution ID
          |
          v
Process Through Standard Flow
```

The original failure history will remain available.

---

# 14. Late-File Flow

```text
Expected File Time Reached
          |
          v
File Available?
     /           \
   Yes           No
    |             |
Process       Record Missing
                  |
                  v
               Alert
                  |
                  v
          Continue Monitoring
```

---

# 15. Incremental Processing Flow

Incremental loads may use:

- Load date
- Last-modified timestamp
- Event timestamp
- Watermark
- File checksum
- Change Data Capture

```text
Read Last Successful Watermark
          |
          v
Select New or Changed Data
          |
          v
Process Increment
          |
          v
Validate Completion
          |
          v
Update Watermark
```

---

# 16. Record Count Reconciliation

```text
Source Record Count
        |
        v
Raw Record Count
        |
        v
Bronze Record Count
        |
        v
Silver Accepted + Rejected
        |
        v
Gold Record Count
        |
        v
Snowflake Target Count
```

The platform must explain any difference between layers.

---

# 17. Security Data Flow

```text
Sensitive Source Data
        |
        v
Restricted Landing Access
        |
        v
Encrypted Storage
        |
        v
Mask or Tokenize Sensitive Fields
        |
        v
Curated Access-Controlled Data
        |
        v
Secure Views for Consumers
```

Only synthetic or de-identified data will be stored in the public project.

---

# 18. Monitoring Data Flow

```text
Pipeline Components
       |
       v
Logs and Metrics
       |
       v
Metadata Repository
       |
       v
Monitoring Dashboard
       |
       v
Alerts and Operational Review
```

---

# 19. File Naming Flow

Expected patterns:

```text
patient_YYYYMMDD.csv
provider_YYYYMMDD.csv
claims_YYYYMMDD.parquet
pharmacy_YYYYMMDD.json
lab_results_YYYYMMDD.csv
```

Example:

```text
patient_20260806.csv
```

The date portion represents the source delivery or business date.

---

# 20. Data Flow Approval

| Role | Status |
|------|--------|
| Data Architect | Pending |
| Technical Lead | Pending |
| Data Engineering Lead | Pending |
| QA Lead | Pending |
