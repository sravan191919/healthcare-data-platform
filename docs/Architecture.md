# System Architecture

---

# Project Information

| Item | Details |
|------|---------|
| Project Name | Healthcare Enterprise Data Platform |
| Client | HealthFirst Analytics — Fictional Portfolio Organization |
| Document Type | Architecture Design |
| Prepared By | Sravan Vaddepudi |
| Role | Data Engineer |
| Version | 1.0 |
| Status | Draft |
| Date | August 2026 |

---

# 1. Architecture Overview

The Healthcare Enterprise Data Platform uses a layered architecture to ingest, validate, process, store, and publish healthcare datasets.

The design separates:

- Source integration
- Ingestion
- Validation
- Raw storage
- Technical standardization
- Business cleansing
- Analytical modeling
- Consumption
- Monitoring
- Metadata
- Security
- Deployment

---

# 2. Complete Logical Architecture

```text
+====================================================================+
|                         SOURCE SYSTEMS                              |
+====================================================================+
| Electronic Health Records                                          |
| Insurance Claims                                                   |
| Pharmacy Systems                                                   |
| Laboratory Systems                                                 |
| Provider Systems                                                   |
| Hospital Operational Systems                                       |
| REST APIs                                                          |
| Kafka Topics                                                       |
+=================================+==================================+
                                  |
                                  v
+====================================================================+
|                         INGESTION LAYER                             |
+====================================================================+
| File Ingestion     | API Ingestion       | Kafka Ingestion          |
| File Discovery     | Authentication      | Topic Subscription       |
| Source Resolution  | Pagination          | Offset Management        |
| Retry Handling     | Rate Limit Handling | Checkpointing            |
+=================================+==================================+
                                  |
                                  v
+====================================================================+
|                          LANDING ZONE                               |
+====================================================================+
| Original Files | Arrival Time | Source | File Metadata             |
+=================================+==================================+
                                  |
                                  v
+====================================================================+
|                  VALIDATION AND DATA QUALITY                       |
+====================================================================+
| Filename | Extension | Empty File | Size | Schema | Business Rules |
+===================+=================================+===============+
                    |                                 |
                 Valid Data                       Invalid Data
                    |                                 |
                    v                                 v
+=====================================+   +===========================+
|             RAW LAYER               |   |        QUARANTINE         |
+=====================================+   +===========================+
| Valid source-preserved records      |   | Rejected files            |
| Technical arrival metadata          |   | Rejection reasons         |
| Replay support                       |   | Investigation support      |
+===================+=================+   +===========================+
                    |
                    v
+====================================================================+
|                          BRONZE LAYER                               |
+====================================================================+
| Standardized columns | Data types | Technical metadata             |
+=================================+==================================+
                                  |
                                  v
+====================================================================+
|                          SILVER LAYER                               |
+====================================================================+
| Cleansing | Deduplication | Conformance | Business validation      |
+=================================+==================================+
                                  |
                                  v
+====================================================================+
|                           GOLD LAYER                                |
+====================================================================+
| Facts | Dimensions | Aggregations | Business-ready datasets        |
+=================================+==================================+
                                  |
                                  v
+====================================================================+
|                     SNOWFLAKE WAREHOUSE                             |
+====================================================================+
| Raw Schema | Curated Schema | Analytics Schema | Audit Schema      |
+=================================+==================================+
                                  |
                                  v
+====================================================================+
|                       DATA CONSUMPTION                              |
+====================================================================+
| Power BI | Analysts | Compliance | Data Science | ML Workloads      |
+====================================================================+
```

---

# 3. Supporting Services

```text
                         +------------------------+
                         | Apache Airflow         |
                         | Scheduling             |
                         | Dependencies           |
                         | Retries                |
                         | Backfills              |
                         +-----------+------------+
                                     |
                                     v
+------------------------+  +------------------------+  +----------------------+
| PostgreSQL Metadata    |  | Data Pipelines         |  | Centralized Logging  |
| Execution history      |<->| Python / PySpark       |<->| Audit and Errors     |
| File history           |  | Validation / Transform |  | Operational events   |
| DQ results             |  +-----------+------------+  +----------------------+
+------------------------+              |
                                        v
                           +---------------------------+
                           | Monitoring and Alerting   |
                           | SLA                       |
                           | Record counts             |
                           | Failures                  |
                           | Data freshness            |
                           +---------------------------+
```

---

# 4. Local Development Architecture

```text
Windows Development Computer
│
├── Visual Studio Code
├── Python Virtual Environment
├── PostgreSQL
├── Local File-System Data Lake
├── Docker Desktop
├── Git
├── GitHub
└── Power BI Desktop
```

Local data flow:

```text
data/landing
      |
      v
Python Ingestion
      |
      v
data/raw
      |
      v
PySpark Processing
      |
      v
data/bronze
      |
      v
data/silver
      |
      v
data/gold
```

---

# 5. Docker-Based Architecture

```text
Docker Compose
│
├── PostgreSQL Container
├── Airflow Webserver
├── Airflow Scheduler
├── Airflow Worker
├── Redis
├── Kafka
├── Zookeeper or KRaft
└── Optional MinIO Storage
```

Docker provides:

- Repeatable environments
- Dependency isolation
- Easier setup
- Service networking
- Consistent local testing

---

# 6. AWS Target Architecture

```text
Source Systems
      |
      v
Amazon S3 Landing
      |
      v
AWS Glue / PySpark
      |
      v
Amazon S3 Raw
      |
      v
Amazon S3 Bronze
      |
      v
Amazon S3 Silver
      |
      v
Amazon S3 Gold
      |
      +--------------------+
      |                    |
      v                    v
Amazon Athena          Snowflake
      |                    |
      v                    v
Ad Hoc Queries         Power BI
```

Supporting AWS services:

- IAM
- CloudWatch
- Glue Data Catalog
- Lake Formation
- Lambda
- Step Functions
- SNS
- SQS
- Secrets Manager
- KMS

Not all services will be implemented in the first release.

---

# 7. Ingestion Architecture

## File Ingestion

```text
Hospital File
      |
      v
Landing Zone
      |
      v
File Discovery
      |
      v
Filename Validation
      |
      v
Schema Validation
      |
      +---------- Invalid ----------> Quarantine
      |
     Valid
      |
      v
Raw Layer
```

## API Ingestion

```text
REST API
   |
Authentication
   |
Pagination
   |
Retry and Rate Limit Handling
   |
Raw Response Storage
   |
Validation
   |
Raw Layer
```

## Kafka Ingestion

```text
Producer
   |
Kafka Topic
   |
Consumer
   |
Schema Validation
   |
Checkpoint
   |
Raw Streaming Storage
```

---

# 8. Data Lake Architecture

```text
healthcare-data-platform/
├── landing/
│   ├── patient/
│   ├── provider/
│   ├── claims/
│   ├── pharmacy/
│   └── lab_results/
├── raw/
├── quarantine/
├── bronze/
├── silver/
└── gold/
```

Example partitioning:

```text
bronze/
└── claims/
    └── source_system=hospital_a/
        └── load_date=2026-08-06/
```

---

# 9. Metadata Architecture

PostgreSQL stores operational metadata.

```text
PostgreSQL
├── pipeline_execution_log
├── file_processing_history
├── data_quality_result
└── schema_version_history
```

Metadata flow:

```text
Pipeline Starts
      |
Insert Running Execution
      |
Process File
      |
Capture Counts and Status
      |
Update Execution
      |
Publish Metrics
```

---

# 10. Airflow Architecture

```text
Airflow Scheduler
       |
       v
Healthcare Ingestion DAG
       |
       +-- detect_files
       +-- validate_files
       +-- check_duplicates
       +-- read_files
       +-- validate_schema
       +-- run_quality_checks
       +-- move_to_raw
       +-- process_bronze
       +-- process_silver
       +-- process_gold
       +-- load_snowflake
       +-- publish_metrics
       +-- notify
```

---

# 11. Snowflake Architecture

```text
HEDP_DATABASE
│
├── RAW_SCHEMA
│   └── Source-aligned tables
├── CURATED_SCHEMA
│   └── Conformed entities
├── ANALYTICS_SCHEMA
│   ├── Fact tables
│   ├── Dimension tables
│   └── Aggregates
└── AUDIT_SCHEMA
    ├── Load history
    └── Data-quality results
```

---

# 12. Security Architecture

```text
Identity and Access
├── Local environment variables
├── AWS IAM
├── Snowflake roles
└── PostgreSQL least-privilege users

Data Protection
├── Encryption at rest
├── Encryption in transit
├── Masking
├── Secure views
└── Synthetic data

Audit
├── Pipeline metadata
├── Access logs
├── Data-quality history
└── Execution logs
```

---

# 13. Monitoring Architecture

Metrics:

- Files received
- Files processed
- Files quarantined
- Records received
- Records accepted
- Records rejected
- Execution duration
- Task status
- SLA breaches
- Data freshness

Alert conditions:

- Missing file
- Late file
- Invalid schema
- Pipeline failure
- Database failure
- Quality threshold breach
- Unexpected volume change

---

# 14. CI/CD Architecture

```text
Developer Branch
      |
      v
Pull Request
      |
      v
Linting and Formatting
      |
      v
Unit Tests
      |
      v
Configuration Validation
      |
      v
Security Scan
      |
      v
Docker Build
      |
      v
Deployment Validation
      |
      v
Merge to Main
```

---

# 15. Reliability Architecture

Reliability patterns:

- Idempotency
- Retry with backoff
- Duplicate-file detection
- Quarantine routing
- Metadata checkpoints
- Atomic writes
- Replay support
- Task-level recovery
- Alerting

---

# 16. Architecture Principles

- Configuration over hardcoding
- Separation of concerns
- Modular design
- Reusability
- Testability
- Observability
- Scalability
- Fault tolerance
- Secure defaults
- Data privacy
- Auditability
- Incremental processing

---

# 17. Architecture Approval

| Role | Status |
|------|--------|
| Data Architect | Pending |
| Technical Lead | Pending |
| Security Lead | Pending |
| DevOps Lead | Pending |
