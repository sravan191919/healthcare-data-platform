# High-Level Design (HLD)

---

# Project Information

| Item | Details |
|------|----------|
| Project Name | Healthcare Enterprise Data Platform |
| Client | HealthFirst Analytics (Fictional) |
| Document Type | High-Level Design |
| Prepared By | Sravan Vaddepudi |
| Role | Data Engineer |
| Version | 1.0 |
| Status | Draft |
| Date | August 2026 |

---

# 1. Purpose

This High-Level Design document defines the overall architecture, major platform components, technology stack, data movement, security approach, and operational design for the Healthcare Enterprise Data Platform.

The platform will ingest healthcare data from multiple sources, validate and transform it through layered processing stages, and publish trusted datasets for reporting, analytics, and machine learning.

---

# 2. Architecture Goals

The platform is designed to:

- Support batch and streaming ingestion.
- Process CSV, JSON, and Parquet data.
- Provide reusable and configurable pipelines.
- Detect invalid files and schema changes.
- Capture metadata for every pipeline execution.
- Support Bronze, Silver, and Gold data layers.
- Provide centralized logging and monitoring.
- Protect sensitive healthcare information.
- Scale from local development to AWS.
- Load curated data into Snowflake.

---

# 3. High-Level Architecture

```text
Source Systems
    |
    |-- Hospitals
    |-- EHR Systems
    |-- Claims Systems
    |-- Laboratories
    |-- Pharmacies
    |-- Provider Systems
    |-- REST APIs
    |-- Kafka Events
    |
    v
Ingestion Layer
    |
    |-- File Ingestion
    |-- API Ingestion
    |-- Kafka Ingestion
    |
    v
Landing Zone
    |
    v
Validation Layer
    |
    |-- File Name Validation
    |-- File Type Validation
    |-- Schema Validation
    |-- Data Quality Validation
    |
    |---- Invalid Data ----> Quarantine
    |
    v
Raw Layer
    |
    v
Bronze Layer
    |
    |-- Basic Standardization
    |-- Technical Metadata
    |-- Source Preservation
    |
    v
Silver Layer
    |
    |-- Cleansing
    |-- Deduplication
    |-- Standardization
    |-- Business Rule Validation
    |
    v
Gold Layer
    |
    |-- Aggregations
    |-- Fact Tables
    |-- Dimension Tables
    |-- Business-Ready Datasets
    |
    v
Snowflake Data Warehouse
    |
    v
Power BI / Analytics / Machine Learning


4. Major Components
4.1 Source Systems

The platform will receive healthcare data from:

Electronic Health Record systems
Insurance claims systems
Pharmacy systems
Laboratory systems
Provider systems
Hospital operational systems
REST APIs
Secure file transfers
Kafka event streams

Supported formats include:

CSV
JSON
Parquet
API responses
Streaming events
4.2 Ingestion Layer

The ingestion layer will provide reusable components for:

File-based ingestion
REST API ingestion
Kafka streaming ingestion
File discovery
File movement
Retry handling
Metadata capture

The framework will be configuration-driven using YAML files.

4.3 Landing Zone

The Landing Zone is the first storage location for incoming data.

Responsibilities:

Receive source files without modification.
Preserve the original source content.
Record arrival timestamps.
Support replay and troubleshooting.
Separate valid and invalid data processing.

Local development location:

data/landing/

Future AWS location:

s3://healthcare-data-platform/landing/
4.4 Validation Layer

The validation layer will perform:

File naming convention checks
Supported file-type validation
File-size checks
Empty-file checks
Required-column validation
Schema validation
Data-type validation
Duplicate-file detection
Business-rule validation

Invalid files will be moved to:

data/quarantine/
4.5 Raw Layer

The Raw Layer stores valid source data with minimal modification.

Responsibilities:

Preserve source values.
Add ingestion metadata.
Maintain auditability.
Support reprocessing.
Separate source systems and load dates.

Example structure:

data/raw/patient/load_date=2026-08-06/
data/raw/claims/load_date=2026-08-06/
4.6 Bronze Layer

The Bronze Layer contains technically standardized data.

Processing includes:

Standard column naming
Data-type conversion
Timestamp standardization
Technical metadata columns
Basic malformed-record handling
Source-system identification
4.7 Silver Layer

The Silver Layer contains cleaned and conformed data.

Processing includes:

Null handling
Duplicate removal
Code standardization
Referential-integrity checks
Patient and provider identifier validation
Date and timestamp cleansing
Business-rule enforcement
4.8 Gold Layer

The Gold Layer contains business-ready datasets.

Examples:

Patient encounter facts
Claims facts
Provider dimensions
Hospital dimensions
Date dimensions
Pharmacy summaries
Laboratory summaries
Executive reporting aggregates
4.9 Metadata Repository

PostgreSQL will store pipeline metadata including:

Execution ID
Pipeline name
Source system
Source file
Start time
End time
Execution status
Record counts
Rejected record counts
Error messages
Processing duration

Primary metadata table:

pipeline_execution_log
4.10 Orchestration Layer

Apache Airflow will orchestrate:

File detection
Validation
Ingestion
Transformation
Data-quality checks
Snowflake loading
Notifications
Retry execution

Example workflow:

detect_files
    |
validate_files
    |
move_to_raw
    |
process_bronze
    |
process_silver
    |
process_gold
    |
load_snowflake
    |
publish_metrics
4.11 Snowflake Data Warehouse

Snowflake will store curated analytical datasets.

Planned objects:

Databases
Schemas
Warehouses
Stages
File formats
Fact tables
Dimension tables
Views
Streams
Tasks
Secure views
4.12 Monitoring and Logging

The platform will provide:

Centralized application logs
Error logs
Audit logs
Pipeline metrics
Record-count reconciliation
Execution-duration tracking
Failure alerts
SLA monitoring
Data-freshness monitoring
5. Technology Stack
Layer	Technology
Programming	Python
Distributed Processing	PySpark / Apache Spark
Batch Ingestion	Python, Pandas, PySpark
Streaming	Apache Kafka
Configuration	YAML
Local Storage	File system
Cloud Storage	AWS S3
Metadata Database	PostgreSQL
Data Warehouse	Snowflake
Orchestration	Apache Airflow
Logging	Python Logging
Testing	pytest
Version Control	Git and GitHub
Containers	Docker and Docker Compose
Infrastructure	Terraform
Cloud Platform	AWS
Reporting	Power BI
Documentation	Markdown and diagrams.net
6. Deployment Environments

The project will use the following environments:

Environment	Purpose
Local	Developer coding and testing
Development	Integrated pipeline testing
QA	Quality-assurance validation
Production	Business-facing workloads

Configuration and credentials will remain separate for each environment.

7. Security Design

The platform will follow these principles:

No passwords stored in source code.
Secrets loaded from environment variables.
.env excluded from Git.
Role-based access control.
Least-privilege permissions.
Encryption in transit.
Encryption at rest.
Sensitive-data masking.
Audit logging.
Restricted access to patient-level data.

Only synthetic or de-identified healthcare data will be stored in the public GitHub repository.

8. Data Privacy

The platform will follow HIPAA-aware engineering practices.

The portfolio implementation will not use real Protected Health Information.

Data used in the project will be:

Synthetic
Publicly available
De-identified
Generated specifically for testing

Examples of sensitive fields requiring protection include:

Patient name
Date of birth
Medical record number
Address
Phone number
Insurance identifier
Diagnosis information
9. Scalability

The system will support scalability through:

Partitioned data storage
Parallel file processing
Distributed PySpark transformations
Configurable batch sizes
Incremental processing
Stateless ingestion components
AWS S3 storage
Snowflake elastic compute
Airflow task parallelism
10. Fault Tolerance

The platform will handle failures through:

Retry policies
Idempotent processing
Quarantine handling
Execution metadata
Structured exception handling
Checkpointing for streaming
Atomic file movement
Reprocessing support
Alert notifications
11. Data Partitioning Strategy

Data will primarily be partitioned by:

Source system
Entity type
Load date
Event date

Example:

data/bronze/claims/source_system=hospital_a/load_date=2026-08-06/

Partitioning will improve query performance and simplify incremental processing.

12. Naming Standards

Examples:

patient_20260806.csv
provider_20260806.csv
claims_20260806.parquet
pharmacy_20260806.json
lab_results_20260806.csv

Database objects will use lowercase snake case.

Examples:

fact_claims
dim_patient
dim_provider
pipeline_execution_log
13. Configuration Strategy

Pipeline settings will be stored in YAML configuration files.

Configuration will define:

Source name
File type
Filename pattern
Required columns
Landing path
Raw path
Quarantine path
Retry count
Logging level
Metadata settings

This avoids hardcoded logic and allows pipelines to be reused across source systems.

14. Testing Strategy

The solution will include:

Unit testing
Configuration testing
Schema-validation testing
File-format testing
Data-quality testing
Integration testing
Failure-scenario testing
Record-count reconciliation
End-to-end pipeline testing
15. CI/CD Strategy

GitHub Actions will eventually automate:

Code-quality checks
Unit tests
Configuration validation
Security checks
Docker image builds
Deployment validation
16. Key Design Decisions
Decision	Reason
Configuration-driven pipelines	Reduce hardcoded logic
Medallion architecture	Separate raw and curated processing stages
PostgreSQL metadata repository	Centralized pipeline auditing
PySpark processing	Support larger healthcare datasets
Airflow orchestration	Manage scheduling and dependencies
Snowflake warehouse	Scalable analytical storage
Quarantine area	Protect downstream systems from invalid data
Synthetic healthcare data	Protect privacy in a public portfolio
17. Future Enhancements

Future versions may include:

Change Data Capture
HL7 and FHIR ingestion
AWS Glue Data Catalog
AWS Lake Formation
Great Expectations
Data lineage
Schema registry
Real-time fraud detection
Machine learning feature pipelines
Kubernetes deployment
Infrastructure automation with Terraform
18. HLD Approval
Role	Status
Product Owner	Pending
Data Architect	Pending
Technical Lead	Pending
Security Team	Pending

Then save the file and run these commands in the VS Code terminal:

```bash
git status
git add docs/HLD.md
git commit -m "Sprint 0: Add high-level architecture design"
git push origin main

