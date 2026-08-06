# Low-Level Design (LLD)

---

# Project Information

| Item | Details |
|------|---------|
| Project Name | Healthcare Enterprise Data Platform |
| Client | HealthFirst Analytics — Fictional Portfolio Organization |
| Document Type | Low-Level Design |
| Prepared By | Sravan Vaddepudi |
| Role | Data Engineer |
| Version | 1.0 |
| Status | Draft |
| Date | August 2026 |

---

# Table of Contents

1. Purpose
2. Application Structure
3. Coding Standards
4. Configuration Design
5. Ingestion Workflow
6. Module Design
7. Validation Design
8. Metadata Database Design
9. Logging Design
10. Exception Handling
11. Retry Design
12. Idempotency
13. Data Layer Design
14. Transformation Design
15. Airflow Design
16. Snowflake Design
17. Testing Design
18. Security Controls
19. Performance Considerations
20. Exit Codes
21. Definition of Done
22. Approval

---

# 1. Purpose

This document defines the implementation-level design for the Healthcare Enterprise Data Platform.

It describes:

- Repository structure
- Python modules
- Classes and functions
- Configuration files
- Processing workflows
- Validation rules
- Metadata tables
- Logging behavior
- Error handling
- Data transformations
- Orchestration logic
- Testing requirements

---

# 2. Application Structure

```text
healthcare-data-platform/
│
├── configs/
│   ├── ingestion_config.yaml
│   ├── logging_config.yaml
│   ├── quality_rules.yaml
│   └── schemas/
│       ├── patient_schema.json
│       ├── provider_schema.json
│       ├── claims_schema.json
│       ├── pharmacy_schema.json
│       └── lab_results_schema.json
│
├── data/
│   ├── landing/
│   ├── raw/
│   ├── quarantine/
│   ├── bronze/
│   ├── silver/
│   └── gold/
│
├── ingestion/
│   ├── __init__.py
│   ├── file_ingestion/
│   │   ├── __init__.py
│   │   ├── main.py
│   │   ├── config_loader.py
│   │   ├── file_discovery.py
│   │   ├── source_resolver.py
│   │   ├── file_reader.py
│   │   ├── file_validator.py
│   │   ├── file_mover.py
│   │   ├── metadata_logger.py
│   │   ├── logger.py
│   │   ├── checksum.py
│   │   ├── retry.py
│   │   ├── exceptions.py
│   │   └── constants.py
│   ├── api_ingestion/
│   └── kafka_ingestion/
│
├── validation/
│   ├── __init__.py
│   ├── schema_validator.py
│   ├── data_quality_validator.py
│   ├── duplicate_validator.py
│   ├── business_rule_validator.py
│   └── validation_result.py
│
├── transformation/
│   ├── __init__.py
│   ├── bronze_transformer.py
│   ├── silver_transformer.py
│   ├── gold_transformer.py
│   ├── patient_transformer.py
│   ├── provider_transformer.py
│   ├── claims_transformer.py
│   ├── pharmacy_transformer.py
│   └── lab_transformer.py
│
├── orchestration/
│   └── airflow_dags/
│       ├── healthcare_file_ingestion_dag.py
│       ├── patient_pipeline_dag.py
│       ├── claims_pipeline_dag.py
│       └── provider_pipeline_dag.py
│
├── warehouse/
│   └── snowflake/
│       ├── ddl/
│       ├── dml/
│       ├── stages/
│       ├── views/
│       ├── tasks/
│       └── procedures/
│
├── monitoring/
│   ├── metrics_collector.py
│   ├── alert_manager.py
│   └── health_check.py
│
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── end_to_end/
│   └── test_data/
│
├── scripts/
├── logs/
├── docs/
├── .env.example
├── .gitignore
├── requirements.txt
├── docker-compose.yml
└── README.md
```

---

# 3. Coding Standards

All Python code must use:

- PEP 8
- Type hints
- Docstrings
- Small reusable functions
- Meaningful names
- Custom exceptions
- Structured logging
- No hardcoded secrets
- No hardcoded absolute paths
- Unit tests
- Clear separation of concerns

Example:

```python
def validate_file_name(
    file_name: str,
    expected_pattern: str
) -> bool:
    """Return True when the filename matches the configured pattern."""
    pass
```

---

# 4. Configuration Design

The ingestion framework will be controlled by YAML configuration.

Example:

```yaml
application:
  name: healthcare-data-platform
  environment: development
  log_level: INFO

paths:
  landing: data/landing
  raw: data/raw
  quarantine: data/quarantine
  bronze: data/bronze
  silver: data/silver
  gold: data/gold

ingestion:
  supported_file_types:
    - csv
    - json
    - parquet
  maximum_file_size_mb: 500
  retry_count: 3
  retry_delay_seconds: 5
  duplicate_check_enabled: true

metadata:
  enabled: true
  execution_table: pipeline_execution_log
  file_history_table: file_processing_history
  quality_table: data_quality_result

sources:
  patient:
    file_pattern: "^patient_[0-9]{8}\\.csv$"
    file_type: csv
    schema_file: configs/schemas/patient_schema.json
    key_columns:
      - patient_id

  provider:
    file_pattern: "^provider_[0-9]{8}\\.csv$"
    file_type: csv
    schema_file: configs/schemas/provider_schema.json
    key_columns:
      - provider_id

  claims:
    file_pattern: "^claims_[0-9]{8}\\.parquet$"
    file_type: parquet
    schema_file: configs/schemas/claims_schema.json
    key_columns:
      - claim_id

  pharmacy:
    file_pattern: "^pharmacy_[0-9]{8}\\.json$"
    file_type: json
    schema_file: configs/schemas/pharmacy_schema.json
    key_columns:
      - prescription_id

  lab_results:
    file_pattern: "^lab_results_[0-9]{8}\\.csv$"
    file_type: csv
    schema_file: configs/schemas/lab_results_schema.json
    key_columns:
      - lab_result_id
```

---

# 5. Ingestion Workflow

```text
1. Load application configuration.
2. Validate required configuration sections.
3. Initialize logging.
4. Generate an execution ID.
5. Discover files in the Landing Zone.
6. Resolve each file to a configured source.
7. Validate the filename.
8. Validate the extension.
9. Validate file size.
10. Validate that the file is not empty.
11. Generate a checksum.
12. Check file-processing history.
13. Read the file.
14. Validate schema.
15. Execute configured data-quality checks.
16. Capture record counts.
17. Move valid files to the Raw layer.
18. Move invalid files to Quarantine.
19. Write execution metadata.
20. Write data-quality metadata.
21. Return final status.
```

---

# 6. Module Design

## 6.1 `config_loader.py`

```python
class ConfigLoader:
    def __init__(self, config_path: str) -> None:
        self.config_path = config_path

    def load(self) -> dict:
        pass

    def validate_required_sections(self, config: dict) -> None:
        pass

    def get_source_config(
        self,
        config: dict,
        source_name: str
    ) -> dict:
        pass
```

Required sections:

- application
- paths
- ingestion
- metadata
- sources

Missing sections will raise `ConfigurationError`.

## 6.2 `file_discovery.py`

```python
class FileDiscovery:
    def __init__(self, landing_path: str) -> None:
        self.landing_path = landing_path

    def discover_files(self) -> list[str]:
        pass

    def filter_supported_files(
        self,
        files: list[str],
        supported_extensions: list[str]
    ) -> list[str]:
        pass
```

Rules:

- Ignore directories.
- Ignore hidden files.
- Ignore temporary files.
- Ignore unsupported formats.
- Return files in modification-time order.

## 6.3 `source_resolver.py`

```python
class SourceResolver:
    def resolve_source(
        self,
        file_name: str,
        source_configurations: dict
    ) -> str:
        pass
```

The resolver compares a filename against configured regular-expression patterns.

## 6.4 `file_reader.py`

```python
class FileReader:
    def read(
        self,
        file_path: str,
        file_type: str
    ):
        pass

    def read_csv(self, file_path: str):
        pass

    def read_json(self, file_path: str):
        pass

    def read_parquet(self, file_path: str):
        pass
```

Unsupported formats will raise `UnsupportedFileTypeError`.

## 6.5 `file_validator.py`

```python
class FileValidator:
    def validate_file_name(
        self,
        file_name: str,
        expected_pattern: str
    ) -> bool:
        pass

    def validate_extension(
        self,
        file_path: str,
        expected_extension: str
    ) -> bool:
        pass

    def validate_not_empty(self, file_path: str) -> bool:
        pass

    def validate_file_size(
        self,
        file_path: str,
        maximum_size_mb: int
    ) -> bool:
        pass
```

## 6.6 `checksum.py`

```python
class ChecksumGenerator:
    def generate_sha256(self, file_path: str) -> str:
        pass
```

The checksum will help prevent duplicate processing.

## 6.7 `file_mover.py`

```python
class FileMover:
    def move_to_raw(
        self,
        source_path: str,
        raw_root: str,
        source_name: str,
        load_date: str
    ) -> str:
        pass

    def move_to_quarantine(
        self,
        source_path: str,
        quarantine_root: str,
        source_name: str,
        reason: str
    ) -> str:
        pass
```

## 6.8 `metadata_logger.py`

```python
class MetadataLogger:
    def __init__(self, connection_string: str) -> None:
        self.connection_string = connection_string

    def start_execution(self, metadata: dict) -> None:
        pass

    def complete_execution(
        self,
        execution_id: str,
        status: str,
        counts: dict
    ) -> None:
        pass

    def log_failure(
        self,
        execution_id: str,
        error_message: str
    ) -> None:
        pass

    def file_was_processed(
        self,
        source_system: str,
        source_file: str,
        checksum: str
    ) -> bool:
        pass
```

## 6.9 `retry.py`

```python
def retry_operation(
    maximum_attempts: int,
    delay_seconds: int,
    backoff_multiplier: int
):
    pass
```

Retryable failures:

- Database connection failure
- Temporary file lock
- Network timeout
- API timeout

Non-retryable failures:

- Invalid filename
- Missing required column
- Unsupported file type
- Permanent schema failure

## 6.10 `main.py`

```python
def run_ingestion(config_path: str) -> int:
    pass
```

Responsibilities:

- Coordinate all modules.
- Process multiple files independently.
- Continue when one source file fails validation.
- Stop only for critical platform failures.
- Return a documented exit code.

---

# 7. Validation Design

## 7.1 Schema Validator

```python
class SchemaValidator:
    def load_expected_schema(self, schema_path: str) -> dict:
        pass

    def validate_required_columns(
        self,
        actual_columns: list[str],
        required_columns: list[str]
    ) -> list[str]:
        pass

    def validate_unexpected_columns(
        self,
        actual_columns: list[str],
        expected_columns: list[str]
    ) -> list[str]:
        pass

    def validate_data_types(
        self,
        dataframe,
        expected_schema: dict
    ) -> list[str]:
        pass
```

## 7.2 Data Quality Validator

```python
class DataQualityValidator:
    def check_nulls(
        self,
        dataframe,
        required_columns: list[str]
    ) -> dict:
        pass

    def check_duplicates(
        self,
        dataframe,
        key_columns: list[str]
    ) -> dict:
        pass

    def check_accepted_values(
        self,
        dataframe,
        column_name: str,
        accepted_values: list[str]
    ) -> dict:
        pass

    def check_numeric_range(
        self,
        dataframe,
        column_name: str,
        minimum,
        maximum
    ) -> dict:
        pass

    def check_date_range(
        self,
        dataframe,
        column_name: str,
        minimum_date,
        maximum_date
    ) -> dict:
        pass
```

## 7.3 Validation Result

```python
from dataclasses import dataclass
from typing import Any


@dataclass
class ValidationResult:
    rule_name: str
    passed: bool
    records_checked: int
    records_failed: int
    message: str
    details: Any | None = None
```

---

# 8. Metadata Database Design

## 8.1 `pipeline_execution_log`

```sql
CREATE TABLE IF NOT EXISTS pipeline_execution_log (
    execution_id UUID PRIMARY KEY,
    pipeline_name VARCHAR(150) NOT NULL,
    source_system VARCHAR(100),
    source_file VARCHAR(255),
    target_layer VARCHAR(30),
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP,
    status VARCHAR(30) NOT NULL,
    records_received BIGINT DEFAULT 0,
    records_processed BIGINT DEFAULT 0,
    records_rejected BIGINT DEFAULT 0,
    error_message TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## 8.2 `file_processing_history`

```sql
CREATE TABLE IF NOT EXISTS file_processing_history (
    file_id UUID PRIMARY KEY,
    source_system VARCHAR(100) NOT NULL,
    source_file VARCHAR(255) NOT NULL,
    file_checksum VARCHAR(128),
    file_size_bytes BIGINT,
    arrival_time TIMESTAMP,
    processed_time TIMESTAMP,
    execution_id UUID,
    status VARCHAR(30),
    quarantine_reason TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE (source_system, source_file, file_checksum)
);
```

## 8.3 `data_quality_result`

```sql
CREATE TABLE IF NOT EXISTS data_quality_result (
    result_id UUID PRIMARY KEY,
    execution_id UUID NOT NULL,
    rule_name VARCHAR(150) NOT NULL,
    rule_type VARCHAR(100),
    column_name VARCHAR(150),
    records_checked BIGINT DEFAULT 0,
    records_failed BIGINT DEFAULT 0,
    status VARCHAR(30),
    failure_details TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## 8.4 `schema_version_history`

```sql
CREATE TABLE IF NOT EXISTS schema_version_history (
    schema_version_id UUID PRIMARY KEY,
    source_system VARCHAR(100) NOT NULL,
    entity_name VARCHAR(100) NOT NULL,
    schema_version VARCHAR(30) NOT NULL,
    schema_definition TEXT NOT NULL,
    effective_from TIMESTAMP NOT NULL,
    effective_to TIMESTAMP,
    active_flag BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

# 9. Logging Design

Log levels:

- DEBUG
- INFO
- WARNING
- ERROR
- CRITICAL

Required log fields:

- timestamp
- log_level
- execution_id
- pipeline_name
- source_system
- source_file
- module
- message

Example:

```text
2026-08-06 02:00:05 | INFO |
execution_id=b7591a2d |
pipeline=patient_ingestion |
source=patient |
file=patient_20260806.csv |
module=file_validator |
message=Filename validation completed successfully
```

Logs must not include:

- Passwords
- Access keys
- Full patient records
- Sensitive identifiers
- Authentication tokens

---

# 10. Exception Handling

```python
class HealthcarePlatformError(Exception):
    """Base application exception."""


class ConfigurationError(HealthcarePlatformError):
    """Raised when configuration is invalid."""


class FileValidationError(HealthcarePlatformError):
    """Raised when file validation fails."""


class SchemaValidationError(HealthcarePlatformError):
    """Raised when schema validation fails."""


class DataQualityError(HealthcarePlatformError):
    """Raised when a critical quality rule fails."""


class UnsupportedFileTypeError(HealthcarePlatformError):
    """Raised for unsupported formats."""


class FileReadError(HealthcarePlatformError):
    """Raised when file reading fails."""


class MetadataLoggingError(HealthcarePlatformError):
    """Raised when metadata cannot be stored."""


class CriticalPlatformError(HealthcarePlatformError):
    """Raised when the pipeline cannot continue."""
```

---

# 11. Retry Design

```yaml
retry:
  maximum_attempts: 3
  delay_seconds: 5
  backoff_multiplier: 2
```

Example delays:

```text
Attempt 1: Immediate
Attempt 2: 5 seconds
Attempt 3: 10 seconds
```

Validation failures will not be retried.

---

# 12. Idempotency

The same file must not be processed more than once.

Duplicate detection will use:

- Source system
- Filename
- File checksum
- File size
- Processing history

Before processing, the framework will query `file_processing_history`.

---

# 13. Data Layer Design

## Landing

```text
data/landing/patient/patient_20260806.csv
```

## Raw

```text
data/raw/patient/load_date=2026-08-06/patient_20260806.csv
```

## Quarantine

```text
data/quarantine/patient/schema_failure/patient_20260806.csv
```

## Bronze Technical Columns

- execution_id
- source_system
- source_file
- ingestion_timestamp
- load_date
- record_hash

## Silver Operations

- Deduplicate
- Standardize text
- Normalize dates
- Validate identifiers
- Apply business rules
- Reconcile records

## Gold Objects

- fact_claims
- fact_encounters
- fact_lab_results
- fact_pharmacy_transactions
- dim_patient
- dim_provider
- dim_hospital
- dim_date

---

# 14. Transformation Design

## Bronze Transformer

```python
class BronzeTransformer:
    def standardize_column_names(self, dataframe):
        pass

    def cast_data_types(self, dataframe, schema: dict):
        pass

    def add_technical_metadata(
        self,
        dataframe,
        execution_id: str,
        source_system: str,
        source_file: str
    ):
        pass
```

## Silver Transformer

```python
class SilverTransformer:
    def remove_duplicates(
        self,
        dataframe,
        key_columns: list[str]
    ):
        pass

    def standardize_dates(self, dataframe, date_columns: list[str]):
        pass

    def standardize_codes(self, dataframe):
        pass

    def apply_business_rules(self, dataframe):
        pass
```

## Gold Transformer

```python
class GoldTransformer:
    def build_fact_claims(self, claims_dataframe):
        pass

    def build_dim_patient(self, patient_dataframe):
        pass

    def build_dim_provider(self, provider_dataframe):
        pass

    def build_aggregates(self, dataframes: dict):
        pass
```

---

# 15. Airflow Design

Planned DAG:

```text
healthcare_file_ingestion_dag
```

Tasks:

```text
start
detect_files
validate_files
check_duplicates
read_source_files
validate_schemas
run_quality_checks
move_files_to_raw
write_metadata
finish
```

DAG requirements:

- Daily schedule
- Configurable retries
- Failure callback
- Execution timeout
- Task-level logging
- Manual rerun support
- Catchup disabled initially

---

# 16. Snowflake Design

Logical structure:

```text
HEDP_DATABASE
├── RAW_SCHEMA
├── CURATED_SCHEMA
├── ANALYTICS_SCHEMA
└── AUDIT_SCHEMA
```

Naming standards:

```text
FACT_CLAIMS
FACT_ENCOUNTERS
DIM_PATIENT
DIM_PROVIDER
DIM_HOSPITAL
DIM_DATE
```

Planned loading approach:

1. Write Gold datasets.
2. Upload files to a Snowflake stage.
3. Execute `COPY INTO`.
4. Validate row counts.
5. Update metadata.
6. Publish secure views.

---

# 17. Testing Design

## Unit Tests

- test_config_loader.py
- test_file_discovery.py
- test_source_resolver.py
- test_file_reader.py
- test_file_validator.py
- test_checksum.py
- test_schema_validator.py
- test_data_quality_validator.py
- test_file_mover.py
- test_metadata_logger.py

## Integration Tests

- Valid CSV to Raw
- Valid JSON to Raw
- Valid Parquet to Raw
- Invalid filename to Quarantine
- Schema failure to Quarantine
- Duplicate-file rejection
- PostgreSQL metadata insertion
- Data-quality-result insertion

## End-to-End Tests

- Landing to Gold
- Airflow-triggered ingestion
- Snowflake load
- Record-count reconciliation
- Failure recovery

---

# 18. Security Controls

- Secrets loaded from environment variables.
- `.env` excluded from Git.
- Least-privilege database account.
- Synthetic data only.
- Sensitive fields excluded from logs.
- Separate credentials for each environment.
- Production secrets stored in a managed secret service.
- Access controlled by role.

---

# 19. Performance Considerations

Initial version:

- Sequential file processing
- Pandas for small files
- Local PostgreSQL
- Local file-system storage

Future optimization:

- Parallel file processing
- PySpark
- Partition pruning
- Incremental loading
- Airflow task parallelism
- Kafka partition scaling
- Snowflake warehouse scaling

---

# 20. Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Successful execution |
| 1 | Partial file-processing failure |
| 2 | Configuration failure |
| 3 | Database failure |
| 4 | Critical platform failure |

---

# 21. Definition of Done

The implementation design is ready when:

- Modules are defined.
- Configuration is documented.
- Database tables are defined.
- Validation behavior is documented.
- Logging is defined.
- Error handling is defined.
- Retry behavior is defined.
- Idempotency is defined.
- Testing requirements are documented.
- Security controls are documented.

---

# 22. Approval

| Role | Status |
|------|--------|
| Data Architect | Pending |
| Technical Lead | Pending |
| QA Lead | Pending |
| DevOps Lead | Pending |
