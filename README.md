# 🎬 CineAnalytics: Event-Driven Azure Data Lakehouse & BI Platform

## 📌 Project Overview & Business Value
In the modern entertainment industry, data is heavily siloed. Financial data, user demographics, and complex movie metadata (cast, crew, genres) often live in fragmented systems. **CineAnalytics** solves this by acting as a fully automated, zero-touch ETL pipeline and single source of truth. 

This project ingests disparate, messy data sources (MovieLens & TMDB datasets) and transforms them into a highly optimized **Star Schema**. 

**Business Impact:**
* **Automated Insights:** Data flows seamlessly from source to dashboard without manual intervention, enabling real-time strategic decisions.
* **Cost Efficiency:** By utilizing Databricks Auto Loader for incremental loading, compute costs are slashed as the pipeline only processes *new* data.
* **Historical Accuracy:** Implemented Slowly Changing Dimensions (SCD Type 2) to preserve the exact financial history and ratings of movies over time.

---

## 🏗️ Architecture & Tech Stack

<img width="2202" height="676" alt="diagram-export-3-1-2026-7_24_53-PM" src="https://github.com/user-attachments/assets/1ffa8b67-12c9-497e-bec3-24019af69d35" />

* **Cloud Provider:** Microsoft Azure
* **Data Storage:** Azure Data Lake Storage Gen2 (ADLS Gen2)
* **Compute & ETL:** Azure Databricks (PySpark, Spark SQL)
* **Table Format:** Delta Lake (for strict ACID compliance)
* **Orchestration:** Azure Data Factory (ADF), Databricks Workflows
* **Event-Driven Automation:** GitHub Webhooks, Azure Logic Apps
* **Business Intelligence:** Tableau (20+ Visualizations, 4 Dashboards)

---

## ⚙️ The Zero-Touch Automated Pipeline

This architecture bypasses rigid time-based scheduling in favor of a true **event-driven** chain reaction:

<img width="1996" height="651" alt="diagram-export-3-6-2026-7_55_40-PM" src="https://github.com/user-attachments/assets/94470c1d-6203-4c28-ba0e-0dd7ba30dbad" />

1. **The Trigger:** A new data file or update is pushed to this GitHub repository.
2. **The Listener:** A GitHub Webhook fires an HTTP payload to an **Azure Logic App**.
3. **The Orchestrator:** The Logic App authenticates and triggers an **Azure Data Factory (ADF)** pipeline.
4. **The Execution:** ADF securely extracts the data, lands it in the ADLS Gen2 `bronze` container, and makes an authenticated REST API call to trigger the Databricks ETL workflow.

---

## 🥇 The Medallion Data Engineering Flow

The Databricks PySpark pipeline progressively cleans and enriches the data through three distinct layers:

### 🥉 Bronze Layer (Raw & Incremental)
* Ingests complex data formats including `.dat` files (requiring custom `::` delimiter parsing) and nested JSON strings.
* Utilizes **Databricks Auto Loader (`cloudFiles`)** with PySpark Structured Streaming to incrementally process only newly landed files, saving compute time and costs.

### 🥈 Silver Layer (Cleansed & Conformed)
* **Data Cleansing:** Standardizes dates, handles nulls, and unpacks massive nested JSON arrays (TMDB Cast/Crew data) using `from_json` and `expr()` to extract specific roles without array-out-of-bounds exceptions.
* **SCD Type 2 (MERGE INTO):** Financial data is merged using Delta Lake. Instead of overwriting records, old records are flagged (`is_current = false`) and timestamped, preserving the complete financial history of the film.
* **One Big Table (OBT):** Merges ratings, users, themes, financials, and credits into a unified dataset.

### 🥇 Gold Layer (Business-Ready Star Schema)
* Decomposes the Silver table into a Kimball **Star Schema** optimized for BI tools.
* **Fact Table:** `fact_movie_performance` (Profit, Gross, Ratings).
* **Dimension Tables:** `dim_movies_genres`, `dim_users_enriched`, `dim_time`.

---

## 🛡️ Why Delta Lake over Standard Parquet?
To ensure enterprise-grade reliability, all tables are stored as Delta formats rather than standard Parquet, guaranteeing:
* **Atomicity & Consistency:** `MERGE INTO` operations either fully succeed or fail. No corrupted, half-written data.
* **Snapshot Isolation:** Concurrent reads and writes are perfectly handled via the `_delta_log`. The Tableau dashboards can query the Gold tables at the exact millisecond the pipeline is writing new data without crashing.
* **Durability (Time Travel):** Delta Lake automatically versions the data, allowing for instant rollbacks to previous states if required.

---

## 📊 Analytics & Visualization (Tableau)
The data engineering pipeline feeds directly into **Tableau**, powering 20 advanced visualizations across 4 interactive dashboards:

1. **Executive Financial Summary:** KPIs tracking Worldwide Gross vs. Production Budgets and top-performing studios/genres.
2. **Audience Demographics:** Heatmaps and bar charts breaking down user ratings by age, gender, and geographic regions.
3. **Content & Genre Trends:** Visualizing the popularity of specific musical themes and genres over decades.
4. **Director & Cast ROI:** Analyzing the Return on Investment of specific directors and top-billed actors to identify consistent box-office drivers.
