# Low-Level Design (LLD)

---

# Project Information

| Item | Details |
|------|---------|
| Project Name | Healthcare Enterprise Data Platform |
| Client | HealthFirst Analytics (Fictional) |
| Document Type | Low-Level Design |
| Prepared By | Sravan Vaddepudi |
| Role | Data Engineer |
| Version | 1.0 |
| Status | Draft |
| Date | August 2026 |

---

# 1. Purpose

This Low-Level Design document describes the detailed implementation of the Healthcare Enterprise Data Platform.

It defines:

- Python modules
- Classes and functions
- Configuration structure
- File-processing workflow
- Validation rules
- Metadata tables
- Logging standards
- Exception handling
- Data-layer processing
- Testing approach

The LLD converts the High-Level Design into specific technical components that developers can implement.

---

# 2. Application Structure

```text
healthcare-data-platform/
│
├── configs/
│   ├── ingestion_config.yaml
│   ├── logging_config.yaml
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
│   │   ├── file_reader.py
│   │   ├── file_validator.py
│   │   ├── file_mover.py
│   │   ├── metadata_logger.py
│   │   ├── logger.py
│   │   ├── exceptions.py
│   │   └── constants.py
│   │
│   ├── api_ingestion/
│   └── kafka_ingestion/
│
├── validation/
│   ├── schema_validator.py
│   ├── data_quality_validator.py
│   ├── duplicate_validator.py
│   ├── business_rule_validator.py
│   └── validation_result.py
│
├── transformation/
│   ├── bronze_transformer.py
│   ├── silver_transformer.py
│   ├── gold_transformer.py
│   ├── patient_transformer.py
│   ├── provider_transformer.py
│   └── claims_transformer.py
│
├── orchestration/
│   └── airflow_dags/
│       ├── healthcare_file_ingestion_dag.py
│       ├── claims_pipeline_dag.py
│       └── patient_pipeline_dag.py
│
├── warehouse/
│   └── snowflake/
│       ├── ddl/
│       ├── dml/
│       ├── views/
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
│   └── test_data/
│
├── logs/
├── docs/
├── scripts/
├── .env.example
├── .gitignore
├── requirements.txt
└── README.md
