## 📌 Project Title

**E-commerce Event-Driven Incremental Data Pipeline using Databricks & GCP**

---

## 🧠 Project Overview

This project implements an **event-driven, incremental data pipeline** for an e-commerce platform using **Databricks, Apache Spark, Delta Lake, and Google Cloud Storage (GCS)**.

The pipeline is automatically triggered when new files arrive in cloud storage and processes data end-to-end from raw ingestion to analytics-ready curated tables.

---

## 🛠 Tech Stack

* **Cloud**: Google Cloud Platform (GCS)
* **Processing**: Databricks, PySpark
* **Storage**: Delta Lake
* **Governance**: Unity Catalog
* **Orchestration**: Databricks Jobs (File Arrival Trigger)
* **Data Modeling**: SCD Type-2
* **Format**: CSV → Delta Tables

---

## 🏗 Architecture Overview

**Event-driven Medallion Architecture**

* **Bronze (Staging)**
  Raw incremental ingestion with schema enforcement and data quality checks

* **Silver (Validated & Enriched)**
  Referential validation and business enrichment

* **Gold (Curated)**
  Analytics-ready tables with historical tracking using SCD Type-2

---

## 🔁 Incremental Load Strategy

* Pipeline is triggered on **file arrival**
* Each file is processed **exactly once**
* Processed files are moved to an **archive folder**
* Metadata such as batch_id and processed_timestamp are added

---

## ✅ Data Quality & Validation

* Null checks
* Range validations
* Referential integrity checks
* Invalid records stored in separate error tables

---

## 🔄 Slowly Changing Dimension (SCD Type-2)

* Implemented using **Delta MERGE**
* Tracks historical changes
* Columns:

  * effective_date
  * expiry_date
  * is_current

---

## 📊 Business Metrics Derived

* Profit Margin
* Customer Lifetime Value (CLV)
* Order Seasonality
* Weekend vs Weekday orders
* Time-based behavior metrics

---

## 🚀 Outcome

* Scalable, production-grade pipeline
* Analytics-ready datasets
* Enterprise-level Delta Lake implementation

---


# 4️⃣ ARCHITECTURE STORY (HOW TO EXPLAIN THE DIAGRAM)

Say this **step-by-step**:

> “Data lands in GCS from upstream systems.
> Databricks Jobs are configured with file-arrival triggers.
>
> The staging layer ingests raw data incrementally using schema enforcement and validation.
>
> After validation, the data is enriched by joining multiple domain datasets like orders, customers, products, inventory, and shipping.
>
> Finally, enriched data is merged into curated Delta tables using SCD Type-2 logic to preserve history and support analytics.”

Interviewers LOVE this clarity.

---

## 1️⃣ Notebook-wise Brief (What each notebook does & why)

You can literally copy-paste these into **Notion** or your **GitHub README**.

---

### 📘 **01_orders_stage_load.ipynb**

**Purpose:** Ingest raw orders data incrementally into a staging layer.

**What happens:**

* Reads order CSV files from **GCS external volume** (Unity Catalog).
* Applies **explicit schema** to avoid schema drift.
* Adds **metadata columns**:

  * `processed_timestamp`
  * `batch_id`
  * `source_system`
* Performs **data quality checks**:

  * Null `order_id`
  * Invalid `order_amount`
* Splits data into:

  * **Valid records → orders_stage (Delta)**
  * **Invalid records → orders_errors**
* Moves processed files to **archive folder** (ensures incremental behavior).

**Why it matters:**

* Prevents reprocessing of old files.
* Ensures only clean data moves forward.
* Supports **file-arrival based triggering**.

---

### 📘 **02_customers_stage_load.ipynb**

**Purpose:** Load customer master data incrementally.

**What happens:**

* Reads customer data from GCS using schema enforcement.
* Adds ingestion metadata.
* Validates:

  * Null `customer_id`
  * Invalid email / lifecycle fields
* Routes bad data to error table.
* Writes clean data to `customers_stage` (Delta).

**Why it matters:**

* Customer data is a **dimension table** → must be clean and consistent.
* Enables joins during enrichment.

---

### 📘 **03_products_stage_load.ipynb**

**Purpose:** Load product catalog data.

**What happens:**

* Reads product files incrementally.
* Validates:

  * `product_id` not null
  * `price > 0`
  * `stock_quantity >= 0`
  * `launch_date` not in future
* Separates valid vs invalid records.
* Writes valid data into `products_stage`.

**Why it matters:**

* Prevents downstream metric corruption (pricing, stock).
* Protects analytics and revenue calculations.

---

### 📘 **04_inventory_stage_load.ipynb**

**Purpose:** Track inventory availability across warehouses.

**What happens:**

* Reads inventory data with strong schema.
* Performs checks on:

  * Negative stock
  * Reserved quantity
  * Available quantity
* Adds batch & processing metadata.
* Writes:

  * Valid → `inventory_stage`
  * Invalid → `inventory_errors`

**Why it matters:**

* Inventory drives fulfillment & availability.
* Bad inventory data = failed orders.

---

### 📘 **05_shipping_stage_load.ipynb**

**Purpose:** Ingest shipping and logistics data.

**What happens:**

* Reads shipping details from source.
* Validates:

  * Shipping cost ≥ 0
  * Package weight > 0
  * Insurance value ≥ 0
* Tracks delivery dates and carrier info.
* Writes valid records to `shipping_stage`.

**Why it matters:**

* Enables delivery tracking & logistics analytics.
* Used later for **order-level enrichment**.

---

### 📘 **06_data_validation.ipynb**

**Purpose:** Cross-table validation before enrichment.

**What happens:**

* Reads **all staging tables**.
* Performs **referential integrity checks**:

  * Orders without customers
  * Customers without orders
* Business rule validation:

  * Order amount range sanity checks
* Stores validation results in a dedicated table.

**Why it matters:**

* Catches **logical data issues**, not just schema issues.
* Prevents bad joins & misleading analytics.

---

### 📘 **07_data_enrichment.ipynb**

**Purpose:** Create analytics-ready enriched datasets.

**What happens:**

* Joins:

  * Orders + Customers + Products + Inventory + Shipping
* Renames conflicting columns (very important 🔥).
* Adds **derived business metrics**:

  * Profit margin
  * Estimated Customer Lifetime Value (CLV)
  * Seasonality
  * Weekend flag
  * Time-of-day indicators
* Writes:

  * `enriched_orders`
  * `customer_analytics`
  * `product_analytics`

**Why it matters:**

* This is where **raw data becomes business data**.
* Enables dashboards, ML models, and reporting.

---

### 📘 **08_final_merge_operation.ipynb**

**Purpose:** Load curated data into final tables using Delta Lake features.

**What happens:**

* Reads enriched datasets.
* Performs **SCD Type-2 merge** for orders:

  * Maintains history
  * Tracks `effective_date`, `expiry_date`
  * Uses `is_current` flag
* Creates analytics summary tables.
* Uses **Delta MERGE / UPSERT** logic.

**Why it matters:**

* Preserves historical changes.
* Enterprise-grade warehouse design.
* Interviewers LOVE SCD-2.

---

## 3️⃣ 🔥 Interview-Ready Story (MOST IMPORTANT)

You should say this **confidently and smoothly**:

> “I built an **event-driven incremental data pipeline for an e-commerce platform** using **Databricks, Delta Lake, and GCP**.
>
> The pipeline is triggered automatically when new files arrive in a GCS bucket. Databricks Jobs detect file arrival and execute a sequence of notebooks.
>
> I implemented a **multi-layer architecture**:
>
> * **Staging layer** for raw ingestion with schema enforcement
> * **Validation layer** for referential and business checks
> * **Enrichment layer** where I joined orders, customers, products, inventory, and shipping data
> * **Curated layer** where I applied **SCD Type-2 merges** using Delta Lake
>
> Each ingestion notebook performs **data quality checks**, captures invalid records into error tables, and archives processed files to support true incremental loading.
>
> During enrichment, I derived business metrics like profit margin, customer lifetime value, seasonality, and order behavior indicators.
>
> Finally, I used **Delta Lake MERGE operations** to maintain historical data and enable analytics-ready tables.
>
> This project helped me understand **event-driven ingestion, Delta Lake internals, SCD-2 design, and production-grade data engineering practices**.”

---

