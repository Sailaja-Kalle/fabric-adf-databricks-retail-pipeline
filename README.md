# 🛍️ Retailer Reviews Data Ingestion Project (Microsoft Fabric + ADF + Databricks)

---

## 📖 Business Scenario  
A retailer collects customer reviews and product/category data from an **on‑prem SQL Server**.  
- **Reviews** → New feedback is appended daily.  
- **Products** → Updated details are merged incrementally.  
- **Categories** → Entire table is overwritten when taxonomy changes.  

This project automates ingestion into **Azure Data Lake Gen2** using **Azure Data Factory pipelines** and **Databricks notebooks**, applying **medallion architecture (raw → bronze → silver → gold)** for clean, structured analytics.

---

## 🧠 What This Project Does  
- Ingests data from **on‑prem SQL Server** into **Azure Gen2 Lakehouse**  
- Maintains **incremental load metadata** in a control table (Azure SQL Server)  
- Automates workflows with **ADF pipeline activities**  
- Applies transformations in **Databricks PySpark notebooks**  
- Implements **Medallion Architecture** (raw, bronze, silver, gold zones)  
- Updates ingestion metadata via **stored procedures**  
- Schedules pipelines with **ADF triggers**  

---

## 🏗️ Architecture Flow  

**Components:**  
- **On‑prem SQL Server** → Source system  
- **Azure SQL Control Table** → Metadata rules (tables, load type, incremental column, last ingestion date)  
- **Azure Data Factory Pipeline** → Lookup, ForEach, Copy, Notebook, Stored Procedure  
- **Azure Data Lake Gen2** → Raw, Bronze, Silver, Gold zones  
- **Databricks (PySpark)** → Transformations and SCD logic  

---

## 📋 Control Table Logic  

| Table Name | Load Type          | Incremental Column   | Last Ingestion Date |
|------------|--------------------|----------------------|---------------------|
| reviews    | Incremental Append | reviewdate           | 2025‑08‑02          |
| products   | Incremental Merge  | lastmodifieddate     | 2025‑08‑01          |
| categories | Full Load          | –                    | –                   |

**Purpose:**  
- Defines which tables to ingest  
- Specifies load type (append, merge, full load)  
- Tracks incremental column and last ingestion date  

---

## 🔄 Pipeline Workflow (ADF)  

### 1. **Lookup Activity**  
- Connects to control table  
- Retrieves metadata (table name, load type, incremental column, ingestion date)  

### 2. **ForEach Activity**  
- Iterates over each control table entry  
- Passes metadata to downstream activities  

### 3. **Copy Activity**  
- **Source:** On‑prem SQL Server  
- **Sink:** Azure Gen2 Lake (raw zone, parquet format)  
- **Connection:** Self‑hosted Integration Runtime (IR)  
- **Linked Services/Datasets:** Configured for source and sink  
- **Dynamic Query:** Built from Lookup output to fetch only required data  

### 4. **Notebook Activity (Databricks)**  
- Connects to Gen2 Lake external location  
- Implements **Medallion Architecture**:  
  - **Raw → Bronze:** Create DataFrame, enforce schema  
  - **Bronze → Silver:** Apply transformations:  
    - Add new columns  
    - Date transformations  
    - Handle null values  
    - Remove duplicates  
    - Apply **SCD Type 1** (overwrite changes)  
    - Apply **SCD Type 2** (track history)  
  - **Silver → Gold:** Build reporting tables as per business requirements  

### 5. **Stored Procedure Activity**  
- Updates ingestion date in control table  
- Ensures next run picks up only new/changed data  

### 6. **Scheduling Triggers**  
- Pipelines scheduled via ADF triggers  
- Supports daily/periodic runs aligned with source availability  

---

## 🧱 Data Lake Zones  

| Zone   | Purpose                        | Example Tables        |
|--------|--------------------------------|------------------------|
| Raw    | Landing zone (as‑is parquet)   | reviews_raw, products_raw  
| Bronze | Schematized, deduplicated      | bronze_reviews, bronze_products  
| Silver | Business‑ready curated tables  | silver_reviews, silver_products  
| Gold   | Aggregated facts/dimensions    | gold_product_ratings  

---

## 🛠️ Technologies Used  

| Technology             | Purpose                                                                 |
|------------------------|-------------------------------------------------------------------------|
| **Microsoft Fabric**   | Unified platform for ingestion + analytics                              |
| **Azure Data Factory** | Pipeline orchestration (Lookup, ForEach, Copy, Stored Procedure)        |
| **Azure Databricks**   | PySpark notebooks for transformations (raw → bronze → silver → gold)    |
| **Azure SQL Server**   | Control table metadata management                                       |
| **Azure Data Lake Gen2** | Storage zones: raw, bronze, silver, gold                                |
| **Self‑hosted IR**     | Secure connection to on‑prem SQL Server                                 |

---

## 🚀 How to Run  

1. **Create Control Table**  
   - Define table_name, load_type, incremental_col, ingestion_dt  

2. **Configure ADF Pipeline**  
   - Add Lookup → ForEach → Copy → Notebook → Stored Procedure  

3. **Deploy Databricks Notebook Logic**  
   - Handle append, overwrite, merge, SCD Type 1 & 2  

4. **Schedule Pipeline**  
   - Daily trigger aligned with source availability  

5. **Monitor Execution**  
   - Check ADF run logs, Databricks job status, and control table updates  

---

## ✅ Final Summary  
This project demonstrates a **metadata‑driven incremental ingestion pipeline** using **Microsoft Fabric, Azure Data Factory, Databricks, and Azure Data Lake Gen2**.  
- **Reviews** are appended daily  
- **Products** are merged incrementally  
- **Categories** are overwritten when updated  
- Data flows through **raw → bronze → silver → gold** layers  
- **Stored procedures** ensure ingestion metadata is updated for the next run  

---
