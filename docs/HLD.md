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


