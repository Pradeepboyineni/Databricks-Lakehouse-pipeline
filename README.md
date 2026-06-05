# Novacart E-Commerce Lakehouse Project

## Project Summary

This project demonstrates an end-to-end production-style Data Engineering pipeline using **SQL Server, Databricks, Delta Lake, PySpark, and Power BI**.

The pipeline ingests e-commerce data from SQL Server tables such as `products`, `orders`, and `payments` into Databricks using incremental loading. The data is processed through a **Medallion Architecture**: Bronze, Silver, and Gold layers.

The project focuses on real-world data engineering concepts:

- Incremental ingestion using watermark logic
- Bronze/Silver/Gold Lakehouse architecture
- Delta Lake tables
- Data cleaning and standardization
- Deduplication using PySpark window functions
- Data quality validation
- Quarantine handling for bad records
- Control tables for pipeline tracking
- Gold-level fact, dimension, and summary tables
- Business dashboards and pipeline monitoring

---

## Architecture Diagram

![Novacart E-Commerce Lakehouse Architecture](./images/novacart-architecture.png)

---

## Project Architecture

```text
SQL Server Source
        |
        | Incremental Load using Timestamp + Primary Key Watermark
        v
Bronze Layer
        |
        | Raw data + audit columns + ingestion control
        v
Silver Layer
        |
        | Cleaning + deduplication + validation
        |
        |---- Bad Records ----> Quarantine Tables
        |
        v
Gold Layer
        |
        | Business-ready fact, dimension, and summary tables
        v
Power BI / Databricks SQL Dashboard
