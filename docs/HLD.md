# High-Level Design (HLD)

---

# Project Information

| Item | Details |
|------|---------|
| Project Name | Healthcare Enterprise Data Platform (HEDP) |
| Client | HealthFirst Analytics — Fictional Portfolio Organization |
| Industry | Healthcare |
| Document Type | High-Level Design |
| Prepared By | Sravan Vaddepudi |
| Role | Data Engineer |
| Version | 1.0 |
| Status | Draft |
| Methodology | Agile Scrum |
| Start Date | August 2026 |

---

# Table of Contents

1. Purpose
2. Business Context
3. Architecture Objectives
4. Solution Overview
5. High-Level Architecture
6. Source Systems
7. Ingestion Layer
8. Storage Layers
9. Validation and Data Quality
10. Processing Layer
11. Metadata Management
12. Workflow Orchestration
13. Data Warehouse
14. Reporting and Consumption
15. Security and Privacy
16. Logging, Monitoring, and Alerting
17. Deployment Environments
18. Scalability and Performance
19. Reliability and Fault Tolerance
20. Technology Stack
21. Key Design Decisions
22. Assumptions and Constraints
23. Future Enhancements
24. Approval

---

# 1. Purpose

This document defines the high-level technical design for the Healthcare Enterprise Data Platform.

The platform will ingest healthcare-related data from multiple source systems, validate incoming files and records, preserve raw data, perform technical and business transformations, and publish trusted datasets for analytics, reporting, and future machine learning workloads.

This document describes the major platform components and their interactions without defining every implementation-level class, function, or database column.

---

# 2. Business Context

HealthFirst Analytics receives operational, financial, clinical, provider, pharmacy, and laboratory data from multiple healthcare organizations.

The incoming data may arrive through:

- Secure file transfers
- Cloud object storage
- REST APIs
- Relational databases
- Streaming platforms
- External healthcare systems

The existing process lacks a common ingestion framework, centralized validation, consistent monitoring, and reusable transformation logic.

The proposed platform will standardize data movement from source systems to business-ready analytical datasets.

---

# 3. Architecture Objectives

The platform is designed to:

- Support batch and streaming ingestion.
- Ingest CSV, JSON, and Parquet files.
- Support REST API ingestion.
- Support Kafka-based streaming in a later release.
- Preserve original source data.
- Validate filenames, schemas, formats, and business rules.
- Quarantine invalid files.
- Capture metadata for every execution.
- Support Landing, Raw, Bronze, Silver, and Gold layers.
- Perform distributed processing using PySpark.
- Orchestrate workflows using Apache Airflow.
- Load curated data into Snowflake.
- Support local, development, QA, and production environments.
- Provide logging, monitoring, auditing, and alerting.
- Protect sensitive healthcare data.
- Support repeatable deployments using Docker and Terraform.

---

# 4. Solution Overview

The Healthcare Enterprise Data Platform will follow a layered architecture.

```text
Healthcare Source Systems
          |
          v
Ingestion Framework
          |
          v
Landing Zone
          |
          v
Validation and Data Quality
       /       \
      /         \
Valid Data    Invalid Data
    |             |
    v             v
Raw Layer     Quarantine
    |
    v
Bronze Layer
    |
    v
Silver Layer
    |
    v
Gold Layer
    |
    v
Snowflake Data Warehouse
    |
    v
Power BI / Analytics / Machine Learning
```

Supporting services include:

- Apache Airflow for orchestration
- PostgreSQL for operational metadata
- Python logging for application logs
- Monitoring and alerting services
- GitHub Actions for CI/CD
- Docker for local containerized execution
- Terraform for infrastructure provisioning

---

# 5. High-Level Architecture

```text
+--------------------------------------------------------------------+
|                         SOURCE SYSTEMS                              |
|--------------------------------------------------------------------|
| EHR | Claims | Pharmacy | Laboratory | Provider | REST API | Kafka |
+-------------------------------+------------------------------------+
                                |
                                v
+--------------------------------------------------------------------+
|                         INGESTION LAYER                            |
|--------------------------------------------------------------------|
| File Ingestion | API Ingestion | Streaming Ingestion              |
+-------------------------------+------------------------------------+
                                |
                                v
+--------------------------------------------------------------------+
|                          LANDING ZONE                              |
|--------------------------------------------------------------------|
| Original files | Arrival timestamps | Source identification       |
+-------------------------------+------------------------------------+
                                |
                                v
+--------------------------------------------------------------------+
|                  VALIDATION AND DATA QUALITY                       |
|--------------------------------------------------------------------|
| Filename | Extension | Size | Schema | Required Fields | DQ Rules |
+--------------------+------------------------------+----------------+
                     |                              |
                 Valid Data                    Invalid Data
                     |                              |
                     v                              v
+----------------------------------+    +-----------------------------+
|            RAW LAYER             |    |       QUARANTINE            |
|----------------------------------|    |-----------------------------|
| Source-preserved validated data  |    | Rejected files and reasons  |
+----------------+-----------------+    +-----------------------------+
                 |
                 v
+--------------------------------------------------------------------+
|                          BRONZE LAYER                              |
|--------------------------------------------------------------------|
| Technical standardization | Metadata columns | Type conversion     |
+-------------------------------+------------------------------------+
                                |
                                v
+--------------------------------------------------------------------+
|                          SILVER LAYER                              |
|--------------------------------------------------------------------|
| Cleansing | Deduplication | Conformance | Business validation     |
+-------------------------------+------------------------------------+
                                |
                                v
+--------------------------------------------------------------------+
|                           GOLD LAYER                               |
|--------------------------------------------------------------------|
| Fact tables | Dimensions | Aggregates | Business-ready datasets   |
+-------------------------------+------------------------------------+
                                |
                                v
+--------------------------------------------------------------------+
|                     SNOWFLAKE WAREHOUSE                            |
|--------------------------------------------------------------------|
| Curated schemas | Secure views | Analytical models                |
+-------------------------------+------------------------------------+
                                |
                                v
+--------------------------------------------------------------------+
|                     DATA CONSUMPTION                               |
|--------------------------------------------------------------------|
| Power BI | Analysts | Reporting | Data Science | Machine Learning |
+--------------------------------------------------------------------+
```

---

# 6. Source Systems

## 6.1 Electronic Health Record Systems

Potential datasets include:

- Patient demographics
- Encounters
- Diagnoses
- Procedures
- Medication orders
- Provider assignments
- Admission and discharge events

Initial supported delivery formats:

- CSV
- JSON
- Parquet
- REST API responses

Future versions may support:

- HL7
- FHIR
- Clinical document formats

## 6.2 Insurance Claims Systems

Claims datasets may include:

- Claim header
- Claim line
- Member identifier
- Provider identifier
- Diagnosis codes
- Procedure codes
- Claim status
- Submitted amount
- Allowed amount
- Paid amount

## 6.3 Pharmacy Systems

Pharmacy datasets may include:

- Prescription identifier
- Medication code
- Patient identifier
- Provider identifier
- Dispense date
- Quantity
- Days supplied
- Pharmacy identifier

## 6.4 Laboratory Systems

Laboratory datasets may include:

- Lab order identifier
- Patient identifier
- Test code
- Test name
- Result value
- Unit of measure
- Reference range
- Abnormal indicator
- Collected timestamp
- Result timestamp

## 6.5 Provider Systems

Provider datasets may include:

- Provider identifier
- National Provider Identifier
- Provider name
- Specialty
- Facility
- Address
- Active status

---

# 7. Ingestion Layer

The ingestion layer will provide reusable, configuration-driven ingestion components.

## 7.1 File Ingestion

The file-ingestion framework will:

- Discover files in the Landing Zone.
- Identify the source entity.
- Validate the filename.
- Validate the file extension.
- Check that the file is not empty.
- Validate maximum file size.
- Check whether the file was processed previously.
- Read supported formats.
- Capture file metadata.
- Route valid files to the Raw layer.
- Route invalid files to Quarantine.

## 7.2 API Ingestion

The API-ingestion framework will:

- Read endpoint configuration.
- Authenticate securely.
- Submit HTTP requests.
- Handle pagination.
- Handle rate limits.
- Apply retries for temporary failures.
- Store raw API responses.
- Capture request and response metadata.
- Log failures.

## 7.3 Kafka Ingestion

The streaming framework will later:

- Subscribe to configured topics.
- Deserialize incoming messages.
- Validate event schemas.
- Track offsets.
- Use checkpointing.
- Route invalid events.
- Store raw event data.
- Monitor consumer lag.

---

# 8. Storage Layers

## 8.1 Landing Zone

The Landing Zone contains newly arrived source files.

Responsibilities:

- Preserve original incoming files.
- Capture arrival time.
- Separate files by source.
- Support replay.
- Support investigation.
- Prevent direct downstream consumption.

Example:

```text
data/landing/patient/patient_20260806.csv
```

Future AWS example:

```text
s3://healthcare-data-platform/landing/patient/
```

## 8.2 Raw Layer

The Raw layer contains files that passed initial file and schema validation.

Responsibilities:

- Preserve original source values.
- Add minimal technical metadata.
- Support replay and recovery.
- Organize files by entity and load date.

Example:

```text
data/raw/patient/load_date=2026-08-06/patient_20260806.csv
```

## 8.3 Quarantine Layer

The Quarantine layer stores rejected files.

Possible rejection reasons:

- Invalid filename
- Unsupported extension
- Empty file
- Invalid schema
- Missing required columns
- Duplicate file
- Corrupted content

Example:

```text
data/quarantine/patient/schema_failure/patient_20260806.csv
```

## 8.4 Bronze Layer

The Bronze layer contains technically standardized records.

Processing includes:

- Column-name standardization
- Data-type conversion
- Timestamp normalization
- Technical metadata
- Source identification
- Malformed-row handling

Technical columns may include:

- execution_id
- source_system
- source_file
- ingestion_timestamp
- load_date
- record_hash

## 8.5 Silver Layer

The Silver layer contains cleaned and conformed records.

Processing includes:

- Duplicate removal
- Null handling
- Code standardization
- Date normalization
- Identifier validation
- Referential-integrity checks
- Business-rule validation
- Entity conformance

## 8.6 Gold Layer

The Gold layer contains business-ready datasets.

Planned datasets include:

- fact_claims
- fact_encounters
- fact_lab_results
- fact_pharmacy_transactions
- dim_patient
- dim_provider
- dim_hospital
- dim_date
- claims_summary
- hospital_performance_summary

---

# 9. Validation and Data Quality

The validation framework will support several levels of checks.

## 9.1 File-Level Validation

- Filename pattern
- File extension
- File size
- Empty-file detection
- Duplicate-file detection
- Source identification

## 9.2 Schema Validation

- Required columns
- Unexpected columns
- Data types
- Column order when required
- Schema version
- Nullable and non-nullable fields

## 9.3 Record-Level Data Quality

- Null checks
- Duplicate checks
- Primary-key uniqueness
- Accepted-value validation
- Date-range validation
- Numeric-range validation
- Referential-integrity validation
- Data-freshness validation

## 9.4 Business Rules

Examples:

- Paid amount cannot be negative.
- Discharge date cannot be before admission date.
- Patient date of birth cannot be in the future.
- Provider identifier must exist in the provider reference data.
- Claim status must be from an approved list.
- Lab result timestamp cannot precede collection timestamp.

---

# 10. Processing Layer

Python and PySpark will be used for data processing.

Python will handle:

- Configuration loading
- File discovery
- File validation
- Metadata logging
- Small-file ingestion
- Utility functions
- Operational workflows

PySpark will handle:

- Larger datasets
- Distributed transformations
- Large joins
- Aggregations
- Deduplication
- Partitioned writes
- Bronze-to-Silver processing
- Silver-to-Gold processing

---

# 11. Metadata Management

PostgreSQL will serve as the operational metadata repository.

Planned tables:

- pipeline_execution_log
- file_processing_history
- data_quality_result
- pipeline_configuration
- schema_version_history

Metadata will include:

- Execution identifier
- Pipeline name
- Source system
- Source filename
- Start time
- End time
- Status
- Record counts
- Rejected record counts
- Error message
- File checksum
- File size
- Processing duration
- Data-quality results

---

# 12. Workflow Orchestration

Apache Airflow will orchestrate pipeline execution.

Example workflow:

```text
start
  |
detect_files
  |
validate_file_metadata
  |
check_duplicate_file
  |
read_source_data
  |
validate_schema
  |
run_data_quality_checks
  |
move_valid_file_to_raw
  |
run_bronze_processing
  |
run_silver_processing
  |
run_gold_processing
  |
load_snowflake
  |
publish_pipeline_metrics
  |
send_notification
  |
end
```

Airflow responsibilities:

- Scheduling
- Task dependency management
- Retries
- Backfills
- Failure handling
- SLA tracking
- Manual reruns
- Operational visibility

---

# 13. Data Warehouse

Snowflake will store curated analytical data.

Planned logical structure:

```text
HEDP_DATABASE
├── RAW_SCHEMA
├── CURATED_SCHEMA
├── ANALYTICS_SCHEMA
└── AUDIT_SCHEMA
```

Planned Snowflake components:

- Virtual warehouses
- Databases
- Schemas
- Stages
- File formats
- Fact tables
- Dimension tables
- Views
- Secure views
- Streams
- Tasks

---

# 14. Reporting and Consumption

Data consumers may include:

- Data analysts
- BI developers
- Operational reporting teams
- Finance teams
- Compliance teams
- Data scientists
- Machine learning engineers

Power BI will be the initial reporting tool.

Potential dashboards include:

- Claims trends
- Hospital utilization
- Provider performance
- Laboratory turnaround time
- Pharmacy activity
- Pipeline monitoring
- Data-quality metrics

---

# 15. Security and Privacy

The project will use only synthetic, public, or de-identified data.

Security controls include:

- No credentials stored in code.
- Environment variables for local secrets.
- Managed secret storage in cloud environments.
- `.env` excluded from Git.
- Role-based access control.
- Least-privilege permissions.
- Encryption in transit.
- Encryption at rest.
- Restricted access to patient-level datasets.
- Sensitive-field masking.
- Audit logging.
- Secure Snowflake views.

---

# 16. Logging, Monitoring, and Alerting

## 16.1 Logging

The platform will produce:

- Application logs
- Audit logs
- Error logs
- Data-quality logs
- Airflow task logs

Required log fields include:

- Timestamp
- Log level
- Execution ID
- Pipeline name
- Source file
- Module
- Message
- Exception details

## 16.2 Monitoring

The platform will monitor:

- Pipeline status
- Files received
- Record counts
- Rejected records
- Execution duration
- Data-quality failures
- Missing files
- Late files
- SLA violations
- Database connectivity
- Kafka consumer lag

## 16.3 Alerting

Alerts may be generated through:

- Email
- Slack
- Microsoft Teams
- Amazon SNS

---

# 17. Deployment Environments

| Environment | Purpose |
|-------------|---------|
| Local | Individual development and debugging |
| Development | Integrated component testing |
| QA | End-to-end quality validation |
| Production | Business-facing workloads |

Each environment will use separate:

- Configuration
- Credentials
- Storage locations
- Databases
- Logs
- Access permissions

---

# 18. Scalability and Performance

The platform will support scalability through:

- Partitioned storage
- Incremental processing
- Parallel file processing
- Distributed PySpark execution
- Configurable batch sizes
- Airflow task parallelism
- Kafka topic partitioning
- Snowflake elastic compute
- S3 object storage

---

# 19. Reliability and Fault Tolerance

Reliability features include:

- Configurable retries
- Exponential backoff
- Idempotent file processing
- Duplicate-file detection
- Quarantine handling
- Metadata-based recovery
- Checkpointing
- Atomic file movement
- Pipeline reruns
- Failure notifications

---

# 20. Technology Stack

| Area | Technology |
|------|------------|
| Programming | Python |
| Distributed Processing | PySpark / Apache Spark |
| Batch Ingestion | Python, Pandas, PySpark |
| Streaming | Apache Kafka |
| Configuration | YAML |
| Metadata Repository | PostgreSQL |
| Data Lake | Local File System / AWS S3 |
| Data Warehouse | Snowflake |
| Orchestration | Apache Airflow |
| Logging | Python Logging |
| Testing | pytest |
| Version Control | Git and GitHub |
| CI/CD | GitHub Actions |
| Containers | Docker and Docker Compose |
| Infrastructure | Terraform |
| Cloud | AWS |
| Reporting | Power BI |
| Documentation | Markdown and diagrams.net |

---

# 21. Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| Configuration-driven ingestion | Avoid source-specific hardcoding |
| Landing-to-Gold architecture | Separate ingestion and curation stages |
| PostgreSQL metadata repository | Centralize operational tracking |
| PySpark processing | Support larger datasets |
| Airflow orchestration | Manage scheduling and dependencies |
| Snowflake warehouse | Support scalable analytics |
| Quarantine storage | Protect downstream systems |
| Synthetic data | Protect healthcare privacy |
| Docker | Provide repeatable local environments |
| Terraform | Provide repeatable cloud infrastructure |

---

# 22. Assumptions and Constraints

## Assumptions

- Source schemas will be available.
- Source owners will follow naming conventions.
- Synthetic datasets will represent expected healthcare structures.
- Required local and cloud infrastructure will be available.
- Users will have access to required development tools.

## Constraints

- Local development hardware has limited memory.
- Some enterprise services will be simulated locally.
- No real Protected Health Information will be used.
- Cloud services may be limited to free-tier or trial usage.
- Streaming capabilities may be introduced in later sprints.

---

# 23. Future Enhancements

Potential future enhancements include:

- HL7 ingestion
- FHIR API integration
- Change Data Capture
- AWS Glue Data Catalog
- AWS Lake Formation
- Schema Registry
- Great Expectations
- OpenLineage
- Kubernetes deployment
- Data masking policies
- Machine learning feature pipelines
- Real-time fraud detection
- Automated data lineage

---

# 24. Approval

| Role | Status |
|------|--------|
| Product Owner | Pending |
| Data Architect | Pending |
| Technical Lead | Pending |
| Security Lead | Pending |
| DevOps Lead | Pending |
