# 🚕 NYC Taxi Analysis — Azure Data Engineering Project

## 📋 Project Overview

An end-to-end Azure Data Engineering project that ingests, processes, and analyzes **NYC Green Taxi trip data for 2023** using the Microsoft Azure ecosystem. The project implements a full **Medallion Architecture (Bronze → Silver → Gold)** with Delta Lake tables for reliable, versioned data storage.

---

## 🏗️ Architecture

```
NYC Taxi Data (Web/HTTP Source)
            ↓
  Azure Data Factory (ADF)
  [ForEach + If Condition Pipeline]
            ↓
  ADLS Gen2 — Bronze Layer
  (Raw Parquet files — 12 months)
            ↓
  Azure Databricks — Silver Notebook
  (Cleaned + Transformed data)
            ↓
  Azure Databricks — Gold Notebook
  (Delta Lake Tables for Analytics)
            ↓
  SQL Analytics on Gold Delta Tables
```

---

## 🛠️ Technology Stack

| Tool | Purpose |
|---|---|
| **Azure Data Factory (ADF)** | Data ingestion & pipeline orchestration |
| **Azure Data Lake Storage Gen2 (ADLS)** | Cloud storage — Bronze/Silver/Gold layers |
| **Azure Databricks** | PySpark data transformation & Delta Lake |
| **Delta Lake** | Versioned, ACID-compliant Gold tables |
| **PySpark** | Data cleaning & transformation |
| **SQL** | Analytics on Gold Delta tables |

---

## 📁 Repository Structure

```
NYC-Taxi-Analysis/
  ├── adf_pipelines/
  │     ├── nycWebtoDL.json          ← ADF Pipeline (ForEach + If Condition)
  │     ├── DataLakeStorage.json     ← ADLS Linked Service
  │     └── nyc_web.json             ← HTTP Web Source Linked Service
  ├── notebooks/
  │     ├── silver_notebook.dbc      ← Databricks Silver transformation notebook
  │     └── gold_notebook.dbc        ← Databricks Gold Delta Lake notebook
  └── README.md
```

---

## 🔄 ADF Pipeline — nycWebtoDL

### Pipeline Logic:
The pipeline dynamically ingests **12 months** of NYC Taxi data using:

- **ForEach Activity** → loops through months 1 to 12 (`@range(1,12)`)
- **If Condition Activity** → handles different URL formats:
  - Months 1–9 → single digit month format (e.g., `01`, `02`)
  - Months 10–12 → double digit month format (e.g., `10`, `11`, `12`)
- **Copy Activity** → downloads Parquet files from NYC TLC web source → saves to ADLS Bronze

### Linked Services:
- `nyc_web` → HTTP connection to `https://d37ci6vzurychx.cloudfront.net` (NYC TLC data source)
- `DataLakeStorage` → ADLS Gen2 connection to `nyctaxistoragemk`

---

## 🥉 Bronze Layer — Raw Data

**Storage:** `abfss://bronze@nyctaxistoragemk.dfs.core.windows.net`

| Folder | Content |
|---|---|
| `trips2023data/` | Raw NYC Green Taxi trip parquet files (12 months) |
| `trip_type/` | Trip type reference CSV |
| `trip_zone/` | Taxi zone lookup CSV |

**Rules:** No transformation, no joins — raw data preserved exactly as received.

---

## 🥈 Silver Layer — Cleaned & Transformed Data

**Storage:** `abfss://silver@nyctaxistoragemk.dfs.core.windows.net`

### Transformations applied in Silver Notebook:

**Trip Data (`trips2023data`):**
- Defined explicit schema using `StructType` for all 20 columns
- Added derived columns:
  - `trip_date` → extracted from `lpep_pickup_datetime`
  - `trip_year` → year extracted
  - `trip_month` → month extracted
- Selected key columns: `VendorID`, `PULocationID`, `DOLocationID`, `fare_amount`, `total_amount`
- Saved as **Parquet** format

**Trip Zone:**
- Split `Zone` column into `Zone1` and `Zone2` using `/` delimiter
- Saved as **Parquet** format

**Trip Type:**
- Renamed column `trip description` → `trip_description`
- Saved as **Parquet** format

---

## 🥇 Gold Layer — Delta Lake Tables

**Storage:** `abfss://gold@nyctaxistoragemk.dfs.core.windows.net`

### Delta Tables created in Gold Notebook:

| Table | Description |
|---|---|
| `gold.trips2023data` | Full cleaned trip data as Delta table |
| `gold.trip_zone` | Zone lookup as Delta table |
| `gold.trip_type` | Trip type reference as Delta table |

### Delta Lake Features used:
- **ACID transactions** — reliable reads and writes
- **Versioning** — full history tracking with `DESCRIBE HISTORY`
- **Time Travel** — restore to previous versions using `RESTORE TABLE TO VERSION AS OF 0`
- **DML operations** — `UPDATE`, `DELETE` on Delta tables

---

## 📊 SQL Analytics (Gold Layer)

```sql
-- Total trips by vendor
SELECT VendorID, MAX(total_amount) 
FROM gold.trips2023data 
GROUP BY VendorID;

-- Zone lookup
SELECT * FROM gold.trip_zone WHERE Borough = 'EWR';

-- Trip types
SELECT * FROM gold.trip_type;
```

---

## 🔐 Authentication

Databricks connects to ADLS using **OAuth 2.0 (Service Principal)**:
- `client_id` → App Registration Client ID
- `tenant_id` → Azure Tenant ID
- `client_secret` → App Registration Secret

---

## 🚀 How to Run This Project

1. **Set up Azure resources:**
   - Create Resource Group
   - Create ADLS Gen2 with Bronze/Silver/Gold containers
   - Create Azure Data Factory
   - Create Azure Databricks workspace

2. **Import ADF Pipeline:**
   - Import `nycWebtoDL.json` into your ADF instance
   - Configure linked services using your storage account

3. **Run ADF Pipeline:**
   - Trigger `nycWebtoDL` pipeline → downloads 12 months of data to Bronze

4. **Run Databricks Notebooks:**
   - Import `silver_notebook.dbc` → run to process Bronze → Silver
   - Import `gold_notebook.dbc` → run to create Gold Delta tables

5. **Run SQL Analytics:**
   - Query Gold Delta tables directly in Databricks SQL

---

## 📈 Dataset

- **Source:** NYC Taxi & Limousine Commission (TLC) — Green Taxi Trip Records 2023
- **Format:** Parquet
- **Volume:** 12 months of trip data
- **Key columns:** VendorID, pickup/dropoff datetime, location IDs, fare amount, total amount, trip type

---

## 👤 Author

**Mayur Kumbhar**
- GitHub: [Mayurkumbhar456](https://github.com/Mayurkumbhar456)
- Project: Azure Data Engineer Learning Path
