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
```
---

## Tech Stack

| Tool / Technology | Purpose |
|---|---|
| SQL Server | Source relational database |
| Databricks | Lakehouse platform for data engineering |
| PySpark | Data cleaning, transformation, and processing |
| Delta Lake | Reliable storage with ACID transactions |
| Lakehouse Federation / JDBC | Reading SQL Server source data |
| Databricks Workflows | Pipeline orchestration and scheduling |
| Power BI / Databricks SQL | Reporting and dashboarding |
| GitHub | Version control and project documentation |

---

## Source Tables

The source system contains three SQL Server tables:

```text
dbo.products
dbo.orders
dbo.payments
```
Example messy data handled in this project:
```
Blank status values
Null customer IDs
Unknown payment status
Negative order amounts
Price values with $ symbols
Comma-based decimal values
Category typo such as ELECTRINICS
Extra spaces in text columns
```

##Bronze Layer

The Bronze layer stores raw incremental data from SQL Server.
```
bronze_schema.products_raw
bronze_schema.orders_raw
bronze_schema.payments_raw
```
Bronze Responsibilities

* Read new records incrementally from SQL Server
* Preserve raw source data
* Add ingestion audit columns
* Track ingestion status using a control table
* Maintain raw history for debugging and replay

##Silver Layer
```
Bronze Raw Tables
        |
        | Read latest processed Bronze watermark
        v
Filter only new Bronze records
        |
        v
Apply cleaning and standardization rules
        |
        v
Deduplicate records using window functions
        |
        v
Apply data quality validation rules
        |
        |---------------- Bad Records ----------------|
        v                                             v
Valid Records                                  Quarantine Tables
        |
        v
MERGE / Upsert into Silver Clean Tables
        |
        v
Update Silver Processing Control Table
        |
        v
Silver data ready for Gold processing
```
Silver Responsibilities

The Silver layer handles the following responsibilities:

* Read only new Bronze records
* Clean messy string values
* Standardize status and category values
* Convert string amounts to decimal values
* Fix known source data issues
* Remove duplicate records
* Apply data quality rules
* Separate invalid records into quarantine tables
* Merge clean records into Silver Delta tables
* Update Silver processing control table after successful completion

### Silver Control Table

The Silver control table tracks incremental processing for each entity. It stores the latest processed Bronze timestamp, Silver run ID, rows merged, run status, and update time. This makes the Silver layer incremental, restartable, and auditable.

![Silver Control Table](./images/Silver_layer_control_table.png)

![Silver Control Table](./images/Silver_layer_control_table_2.png)

---
### Silver Orders Cleaned Output

The `orders_clean` table contains cleaned, standardized, deduplicated, and validated order records from the Bronze layer.

In this step, order data is processed by trimming spaces, standardizing order status values, converting order amounts into decimal format, removing invalid records, and keeping only the latest valid record for each `order_id`.

![Silver Orders Cleaned Output](./images/Silver_Orders_cleaned.png)

## Quarantine Handling - Data Quality Exception Management

The Quarantine layer stores records that fail Silver-level validation rules. Instead of dropping bad records silently, the pipeline captures them separately with a clear verification reason for debugging, monitoring, and business review.

This approach helps maintain trusted Silver and Gold tables while still preserving invalid records for investigation.

---

### Why Quarantine Is Needed

Real-world source data can contain issues such as:

```text
Null customer IDs
Blank order statuses
Unknown payment statuses
Negative order amounts
Invalid product prices
Missing product categories
Amount fields containing $ symbols or comma values
```
![Quarantine Table Output](./images/Quarantine_table_1.png)
![Quarantine Table Output](./images/Quarantine_table_2.png)




