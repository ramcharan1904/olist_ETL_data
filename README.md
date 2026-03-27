# 🏗️ Olist E-Commerce — End-to-End Big Data Engineering Pipeline on Azure

> A production-grade, cloud-native ETL pipeline built on Azure using Medallion Architecture — ingesting raw e-commerce data from multiple sources, transforming it through Bronze → Silver → Gold layers, and serving analytics-ready datasets via Azure Synapse.

![Azure](https://img.shields.io/badge/Azure-Data%20Engineering-blue?logo=microsoftazure)
![Databricks](https://img.shields.io/badge/Databricks-PySpark-orange?logo=databricks)
![Synapse](https://img.shields.io/badge/Azure%20Synapse-Analytics-purple?logo=microsoftazure)
![MongoDB](https://img.shields.io/badge/MongoDB-NoSQL-green?logo=mongodb)
![License](https://img.shields.io/badge/License-MIT-yellow)

---

## 📌 Table of Contents

- [About](#-about)
- [Architecture](#-architecture)
- [Medallion Layers](#-medallion-layers)
- [Features](#-features)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Pipeline Walkthrough](#-pipeline-walkthrough)
- [Getting Started](#-getting-started)
- [Azure Services Setup](#-azure-services-setup)
- [Synapse SQL — Gold Layer](#-synapse-sql--gold-layer)
- [Screenshots](#-screenshots)
- [Dataset](#-dataset)
- [Contributing](#-contributing)
- [Author](#-author)

---

## 📖 About

This project simulates a real-world, industry-standard **Big Data Engineering pipeline** using the [Olist Brazilian E-Commerce dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) — a rich, multi-table dataset spanning orders, customers, products, payments, reviews, and sellers.

The pipeline ingests raw data from HTTPS sources (GitHub) and SQL databases into Azure Data Lake Storage Gen 2, transforms it using PySpark on Azure Databricks, enriches it with MongoDB data, and finally serves analytics-ready external tables via Azure Synapse Analytics — all following the **Medallion Architecture** pattern.

---

## 🏛️ Architecture

![Architecture Diagram](Architecture%20Diagram.png)

```
┌──────────────────────────────────────────────────────────────────────┐
│                        DATA SOURCES                                  │
│   HTTPS (GitHub)  │  SQL Database  │  MongoDB (NoSQL enrichment)     │
└─────────┬─────────┴───────┬────────┴──────────────┬──────────────────┘
          │                 │                        │
          ▼                 ▼                        │
┌─────────────────────────────────────┐             │
│     Azure Data Factory (ADF)        │             │
│  Parameterized pipelines, Lookup,   │             │
│  ForEach activities                 │             │
└─────────────────┬───────────────────┘             │
                  ▼                                  │
┌─────────────────────────────────────┐             │
│   ADLS Gen 2 — Bronze Layer         │             │
│   Raw CSV / JSON files               │             │
└─────────────────┬───────────────────┘             │
                  ▼                                  ▼
┌─────────────────────────────────────────────────────┐
│          Azure Databricks (PySpark)                 │
│   Clean → Rename → Filter → Join → Enrich (MongoDB) │
└─────────────────┬───────────────────────────────────┘
                  ▼
┌─────────────────────────────────────┐
│   ADLS Gen 2 — Silver Layer         │
│   Parquet files (cleaned & joined)  │
└─────────────────┬───────────────────┘
                  ▼
┌─────────────────────────────────────┐
│   Azure Synapse Analytics           │
│   Views → External Tables → Gold   │
└─────────────────┬───────────────────┘
                  ▼
┌─────────────────────────────────────┐
│   ADLS Gen 2 — Gold Layer           │
│   Analytics-ready External Tables   │
│   (BI tools, Data Scientists)       │
└─────────────────────────────────────┘
```

---

## 🥇 Medallion Layers

| Layer | Storage | Format | Description |
|---|---|---|---|
| **Bronze** | ADLS Gen 2 | CSV / JSON | Raw, unprocessed data as-is from sources |
| **Silver** | ADLS Gen 2 | Parquet | Cleaned, filtered, joined, and MongoDB-enriched data |
| **Gold** | ADLS Gen 2 | Parquet (Snappy) | Aggregated, business-ready external tables for analytics |

---

## ✨ Features

- **Multi-Source Ingestion** — Pulls data from HTTPS endpoints (GitHub), SQL databases, and MongoDB
- **Parameterized ADF Pipelines** — ForEach and Lookup activities dynamically process all dataset files without hardcoding
- **PySpark Transformations** — Scalable cleaning, filtering, renaming, and multi-dataset joins on Databricks
- **NoSQL Enrichment** — Integrates MongoDB data into the Silver layer for richer context
- **Medallion Architecture** — Strict Bronze → Silver → Gold quality progression
- **Synapse Analytics** — Creates views and compressed external tables (Snappy Parquet) in the Gold layer
- **Analytics-Ready Output** — Gold layer directly queryable by Power BI, data scientists, and analysts
- **Managed Identity Auth** — Secure, passwordless access between Azure services

---

## 🛠️ Tech Stack

| Service / Tool | Role |
|---|---|
| Azure Data Factory (ADF) | Orchestration & data ingestion |
| Azure Data Lake Storage Gen 2 | Storage for all three layers |
| Azure Databricks | PySpark-based data transformation |
| Azure Synapse Analytics | Gold layer SQL views & external tables |
| MongoDB | NoSQL source for data enrichment |
| PySpark | Distributed data processing |
| SQL (T-SQL) | Synapse queries, views, external tables |
| Python / Jupyter | Data ingestion scripting |
| Parquet + Snappy | Columnar storage format for Silver & Gold |

---

## 📁 Project Structure

```
olist_ETL_data/
│
├── Architecture Diagram.png          # End-to-end pipeline architecture diagram
├── DATA_INGESTION_TO_DB.ipynb        # Jupyter notebook: ingesting raw data to SQL/MongoDB
├── Databricks Code 2025-06-10.html   # Exported Databricks notebook (Bronze → Silver transformations)
├── ForEachInput.json                 # ADF ForEach activity input — list of dataset files to process
├── SQL_code_synapse.txt              # Synapse Analytics SQL: views, external tables, Gold layer setup
├── Screenshot 2025-06-12 224404.png  # Pipeline screenshot
├── Screenshot 2025-06-12 224623.png  # Databricks transformation screenshot
├── Screenshot 2025-06-12 224950.png  # Synapse Gold layer screenshot
└── README.md
```

---

## 🔍 Pipeline Walkthrough

### Phase 1 — Data Ingestion (Bronze Layer)

- Raw Olist CSV files are sourced from HTTPS (GitHub) and a SQL database
- **Azure Data Factory** uses a **Lookup activity** to read `ForEachInput.json`, which lists all dataset files
- A **ForEach activity** iterates over every file and copies it into ADLS Gen 2 (Bronze layer) — no hardcoding required
- Additional structured data is ingested via `DATA_INGESTION_TO_DB.ipynb` into SQL and MongoDB

### Phase 2 — Data Transformation (Silver Layer)

- Azure Databricks reads raw Parquet/CSV files from the Bronze layer
- PySpark performs:
  - **Cleaning** — handles nulls, type casting, deduplication
  - **Renaming** — standardizes column names across datasets
  - **Filtering** — removes invalid/incomplete records
  - **Joining** — merges orders, customers, products, payments, reviews, sellers
  - **Enrichment** — pulls additional fields from MongoDB and joins into the dataset
- Output is written as **Parquet files** to the Silver layer in ADLS Gen 2

### Phase 3 — Data Serving (Gold Layer)

- Azure Synapse Analytics reads Silver layer Parquet files using `OPENROWSET`
- Creates a **view** (`gold.final`) over the Silver data
- Provisions an **External Data Source** pointing to the Gold layer container
- Creates a compressed **External Table** (`gold.finaltable`) using Snappy Parquet format
- Gold layer is now directly queryable by Power BI, analysts, and data science teams

---

## 🚀 Getting Started

### Prerequisites

- Azure subscription with the following services provisioned:
  - Azure Data Factory
  - Azure Data Lake Storage Gen 2
  - Azure Databricks workspace
  - Azure Synapse Analytics workspace
  - MongoDB Atlas (or Azure Cosmos DB for MongoDB)
- Python 3.9+ (for the ingestion notebook)
- Azure CLI (optional, for setup automation)

### Steps

1. **Clone the repository**
   ```bash
   git clone https://github.com/ramcharan1904/olist_ETL_data.git
   cd olist_ETL_data
   ```

2. **Run the ingestion notebook** to load raw data into SQL and MongoDB:
   ```
   Open DATA_INGESTION_TO_DB.ipynb in Jupyter or Azure Databricks
   Update connection strings for your SQL and MongoDB instances
   Run all cells
   ```

3. **Configure ADF pipeline:**
   - Import `ForEachInput.json` as the lookup source in your ADF pipeline
   - Set linked services for HTTPS, SQL, and ADLS Gen 2
   - Trigger the pipeline to populate the Bronze layer

4. **Run the Databricks notebook** (see `Databricks Code 2025-06-10.html`):
   - Attach to a Databricks cluster
   - Update ADLS mount paths and MongoDB connection
   - Run to generate the Silver layer

5. **Execute Synapse SQL** (`SQL_code_synapse.txt`):
   - Open Azure Synapse Studio → SQL script
   - Run in order: OPENROWSET query → Create schema → Create view → Create external table

---

## ☁️ Azure Services Setup

### ADLS Gen 2 Container Structure
```
olistdata/
├── bronze/     ← Raw CSV files from ADF
├── silver/     ← Cleaned Parquet from Databricks
└── gold/       ← External tables from Synapse
```

### Managed Identity Authentication
The pipeline uses **Managed Identity** for secure, credential-free access between Synapse and ADLS Gen 2:
```sql
CREATE DATABASE SCOPED CREDENTIAL ramadmin
WITH IDENTITY = 'Managed Identity';
```

---

## 🗄️ Synapse SQL — Gold Layer

Key SQL operations in `SQL_code_synapse.txt`:

```sql
-- Query Silver layer via OPENROWSET
SELECT * FROM OPENROWSET(
    BULK 'https://<storage>.dfs.core.windows.net/olistdata/silver/',
    FORMAT = 'PARQUET'
) AS result1

-- Create Gold view
CREATE VIEW gold.final AS
SELECT * FROM OPENROWSET(...) AS result1

-- Create compressed External Table in Gold layer
CREATE EXTERNAL TABLE gold.finaltable WITH (
    LOCATION = 'Serving',
    DATA_SOURCE = goldlayer,
    FILE_FORMAT = extfileformat   -- Snappy Parquet
) AS SELECT * FROM gold.final;
```

---

## 📸 Screenshots

| ADF Pipeline | Databricks Transformation | Synapse Gold Layer |
|---|---|---|
| ![ADF](Screenshot%202025-06-12%20224404.png) | ![Databricks](Screenshot%202025-06-12%20224623.png) | ![Synapse](Screenshot%202025-06-12%20224950.png) |

---

## 📦 Dataset

**Olist Brazilian E-Commerce** — publicly available on Kaggle.

The dataset contains ~100k orders from 2016–2018 across multiple tables:

| Table | Description |
|---|---|
| `olist_orders` | Order headers with status and timestamps |
| `olist_customers` | Customer location and IDs |
| `olist_order_items` | Products per order with price/freight |
| `olist_products` | Product categories and dimensions |
| `olist_sellers` | Seller location data |
| `olist_order_payments` | Payment types and values |
| `olist_order_reviews` | Customer review scores and comments |
| `olist_geolocation` | Brazilian ZIP code lat/lng coordinates |

---

## 🤝 Contributing

Contributions are welcome! To contribute:

1. Fork the repository
2. Create a branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m "Add: your feature"`
4. Push: `git push origin feature/your-feature`
5. Open a Pull Request

---

## 👤 Author

**Ram Charan Gubbala**

AI Engineer | MS Data Science @ UAB | AWS Certified

- 🌐 [Portfolio](https://ram-portfolio-theta.vercel.app/)
- 💼 [LinkedIn](https://linkedin.com/in/ramcharangubbala)
- 🐙 [GitHub](https://github.com/ramcharan1904)
- 📧 ramcharangubbala7@gmail.com

---

*If this project helped you, please give it a ⭐ — it helps others discover it!*
