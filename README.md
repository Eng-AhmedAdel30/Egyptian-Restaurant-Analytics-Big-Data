# 🍽️ Egyptian Restaurant Analytics — Big Data Pipeline

<div align="center">


[![Databricks](https://img.shields.io/badge/Databricks-FF3621?style=for-the-badge&logo=databricks&logoColor=white)](https://databricks.com/)
[![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)](https://powerbi.microsoft.com/)
[![Fivetran](https://img.shields.io/badge/Fivetran-0073E6?style=for-the-badge&logo=fivetran&logoColor=white)](https://fivetran.com/)
[![PySpark](https://img.shields.io/badge/PySpark-E25A1C?style=for-the-badge&logo=apachespark&logoColor=white)](https://spark.apache.org/docs/latest/api/python/)
[![Google Drive](https://img.shields.io/badge/Google_Drive-4285F4?style=for-the-badge&logo=googledrive&logoColor=white)](https://drive.google.com/)
[![SQL](https://img.shields.io/badge/SQL-CC2927?style=for-the-badge&logo=microsoftsqlserver&logoColor=white)](https://databricks.com/product/databricks-sql)

**An end-to-end Big Data Analytics pipeline analyzing 11M+ rows of Egyptian restaurant data — from raw ingestion to interactive business intelligence dashboards.**


</div>

---

## 📌 Overview

This project delivers a **production-grade Big Data analytics solution** for the Egyptian restaurant industry, processing over **11 million rows** of transactional data spanning **2020–2025**. It demonstrates a complete modern data engineering workflow — from automated cloud ingestion all the way to interactive executive dashboards.

| 📦 Dataset Size | 📁 Source Files | 🗓️ Time Span | 🏗️ Architecture |
|:-:|:-:|:-:|:-:|
| ~1.5 GB | 7x CSV + 2x JSON | 2020 – 2025 | Medallion (Bronze → Silver → Gold) |

**Business Goal:** Enable restaurant stakeholders to uncover revenue trends, sales performance patterns, and operational inefficiencies through a scalable, low-latency analytics pipeline.

---

## 🏛️ Architecture

<div align="center">

![Architecture Diagram](Project%20architecture.png)

</div>

This project implements the **Medallion Architecture** — a layered data design pattern that progressively refines data quality as it moves through the pipeline.


### 🥉 Bronze Layer — Raw Landing Zone
- Acts as the **immutable landing zone** for all ingested data.
- Data is loaded **as-is** with **full fidelity** — no transformations applied.
- Preserves the original source schema for auditability and replayability.

### 🥈 Silver Layer — Cleaned & Conformed Data
- Applies all **data cleaning and standardization** logic.
- Handles nulls, removes duplicates, filters invalid records (e.g., negative prices).
- Joins and enriches datasets to produce a consistent, analytics-ready view.

### 🥇 Gold Layer — Business-Ready Star Schema
- Hosts the final **Star Schema** optimized for BI consumption.
- Contains **Fact** and **Dimension** tables designed for high-performance analytical queries.
- Directly connected to Power BI for live reporting.

---

## 🔄 Data Pipeline

The pipeline is fully automated — **no local downloads, no manual transfers.**

```
┌─────────────────┐     Auto Sync      ┌──────────────────────┐
│   Google Drive  │ ─────────────────► │       Fivetran        │
│  7x CSV Files   │   (Scheduled,      │  No-Code Connector    │
│  2x JSON Files  │    No-Code)        │  GDrive → Databricks  │
│  ~1.5 GB | 11M+ │                   └──────────┬───────────┘
└─────────────────┘                              │ Load Raw
                                                 ▼
                                     ┌──────────────────────┐
                                     │  Databricks (Bronze) │
                                     │  Raw Layer / Landing │
                                     └──────────┬───────────┘
                                                │ Transform
                                                ▼
                                     ┌──────────────────────┐
                                     │  Databricks (Silver) │
                                     │  Clean & Standardize │
                                     └──────────┬───────────┘
                                                │ Transform
                                                ▼
                                     ┌──────────────────────┐
                                     │  Databricks (Gold)   │
                                     │  Star Schema / BI    │
                                     └──────────┬───────────┘
                                                │ Connect (Live)
                                                ▼
                                     ┌──────────────────────┐
                                     │       Power BI       │
                                     │  Interactive Dashbd. │
                                     └──────────────────────┘
```

---

## 🧹 Data Cleaning & Processing

All transformations were performed inside **Databricks SQL** at the Silver layer, ensuring a clean and consistent dataset before any modeling takes place.

| Issue | Resolution |
|---|---|
| **Null / Missing Values** | Identified and handled per-column using conditional imputation or exclusion |
| **Duplicate Records** | Detected and removed using window functions and deduplication logic |
| **Negative Price Values** | Filtered out as invalid transactions before Silver promotion |
| **Schema Standardization** | Enforced consistent data types, date formats, and column naming conventions across all source files |
| **Multi-format Sources** | Unified CSV and JSON schemas into a single conformed structure |

> All cleaning logic is written in **Databricks SQL**, making it reusable, auditable, and scalable to any volume of incoming data.

---

## 🗂️ Data Modeling — Star Schema

In the **Gold layer**, a **Star Schema** was designed to maximize query performance and simplify Power BI reporting.
```
  ┌──────────────────────┐                          ┌──────────────────────┐
  │      dim_date        │                          │    dim_order_type    │
  │──────────────────────│                          │──────────────────────│
  │ Date_PK        (PK)  │                          │ order_type_pk  (PK)  │
  │ order_date           │                          │ order_type           │
  │ Day, day_month       │                          └──────────┬───────────┘
  │ day_of_week, hour    │                                     │ 1
  │ Month, month_num     │                                     │
  │ is_weekend, segment  │                           ┌─────────▼──────────────────────────┐
  │ year                 │◄────────── 1 ─────────────│           fact_orders              │
  └──────────────────────┘                           │────────────────────────────────────│
                                                     │ order_id                           │
  ┌──────────────────────┐                           │ branch_FK,   customer_FK           │
  │     dim_product      │                           │ date_FK,     order_type_FK         │
  │──────────────────────│                           │ payment_FK,  product_FK            │
  │ item_pk        (PK)  │◄────────── 1 ─────────────│ price,       total_amount          │
  │ item_name            │                           │ quantity,    discount               │
  │ category             │                           │ rating,      is_weekend            │
  └──────────────────────┘                           └─────────┬──────────────────────────┘
                                                               │
          ┌────────────────┬───────────────┬───────────────────┘
          │ 1              │ 1             │ 1
          ▼                ▼               ▼
  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐
  │  dim_brach   │  │ dim_customer │  │     dim_payment      │
  │──────────────│  │──────────────│  │──────────────────────│
  │ branch_pk(PK)│  │ customer_pk  │  │ payment_pk     (PK)  │
  │ branch       │  │ customer_id  │  │ payment_method       │
  └──────────────┘  └──────────────┘  └──────────────────────┘
```
**Why Star Schema?**
- ✅ Optimized for **analytical read performance** with minimal joins
- ✅ Easy to extend with new dimensions without restructuring
- ✅ Native compatibility with Power BI's relationship model
- ✅ Reduces query complexity for business users

 
## 📊 Dashboard & Insights
 
The Power BI dashboard is connected **live** to the Databricks Gold layer, providing real-time data refresh without manual exports. It consists of **3 interactive pages**, each targeting a distinct analytical domain.
 
---
 
### 🗺️ Page 1 — Overview
 
> *Branch performance, monthly trends, and category distribution across all Egyptian cities.*
 
| KPI | Value |
|---|---|
| 📋 Avg Orders / Day | **5.05K** |
| 📦 Total Quantity Sold | **35M** |
| 💳 Avg Spend / Customer | **14.5K** |
| 🔁 Repeat Customer Rate | **100%** |
| 🧾 No. of Orders | **11M** |
 
**Visuals:**
- 📅 **Total Sales by Month** — Consistent monthly revenue ranging between **225M – 246M**, showing stable demand year-round
- 🏙️ **Total Sales by Branch** — Cairo leads at **1,014.5M**, followed by Giza (**579.9M**) and Alexandria (**579.4M**); Assiut is the smallest branch at **145.1M**
- 🍽️ **Quantity by Category** — Perfectly balanced distribution across all 5 categories: Grills, Stuffed, Casseroles, Appetizers, and Beverages — each at **7M** units
- 🔄 **Repeating vs One-Time Customers** — **100% repeating customers**, indicating very strong loyalty across all branches
---
 
### 💰 Page 2 — Product & Sales
 
> *Deep-dive into revenue performance, time-based patterns, and product-level analytics.*
 
| KPI | Value |
|---|---|
| 📋 Avg Orders / Day | **5.05K** |
| 📦 Total Quantity | **35M** |
| 💰 Avg Sales / Day | **1.32M** |
| 💵 Total Sales | **$2.9bn** |
| 🛒 No. of Items | **15** |
 
**Visuals:**
- 📊 **Sales by Year (with MoM % & YoY %)** — Drillable table showing annual and monthly performance from 2020–2025; total revenue stands at **$2,899.14M** with **+19.94% YoY** overall
- 📈 **Products by Quantity & Revenue** — Dual-axis chart comparing units sold vs. revenue per product; top sellers include Kebab, Tagin Ferakh, and Kofta
- 🕐 **Sales by Time Segment** — Afternoon drives the most revenue at **1,082.6M**, followed by Evening (**794M**), Night (**686.7M**), and Morning (**335.8M**)
- 📅 **Sales by Weekend** — Perfectly split at **50% weekday / 50% weekend**, revealing consistent traffic regardless of day type
---

### 👥 Page 3 — Customer & Operations
 
> *Customer behaviour, payment preferences, order types, and branch-level satisfaction.*
 
| KPI | Value |
|---|---|
| 👤 Total Customers | **200K** |
| ⭐ Avg Rating | **3.7** |
| 🏷️ Total Discount | **399.6K** |
| 💵 Total Sales | **$2.9bn** |
| 🧾 No. of Orders | **11M** |
 
**Visuals:**
- 💳 **Orders by Payment Method** — Cash dominates at **50%**, followed by Card (**30%**) and Wallet (**20%**)
- 📆 **Total Sales by Year** — Line chart from 2020–2025 showing a dip in 2022 (**482.1M**) and recovery peak in 2024 (**483.9M**)
- 🚗 **Orders by Order Type** — Dine-in leads at **40%**, Takeaway and Delivery are tied at **30%** each
- 🏆 **Top 5 Items** — Mango Juice (**1.31M orders**), Tabeekha (**1.11M**), Kebab (**1.11M**), Stuffed Grape Leaves (**0.89M**), Casserole Chicken (**0.89M**)
- 🗺️ **Avg Rating by Branch** — Cairo scores highest at **4.0**, while Assiut scores **3.1**; all branches within interactive filter scope
---


## 🚀 Scalability & Performance Considerations

This pipeline was built with growth and performance in mind from day one.

### Horizontal Scalability
- Databricks clusters **auto-scale** based on workload — the same pipeline handles 11M rows or 100M+ rows without architectural changes.
- Fivetran's incremental sync ensures only **new/changed records** are loaded on each run, avoiding full reloads.

### Query Performance
- The **Star Schema** in the Gold layer minimizes join complexity, reducing query execution time for analytical workloads.
- Databricks SQL uses **Delta Lake** under the hood — providing ACID transactions, data versioning, and optimized Parquet storage.
- **Z-ordering** and **data partitioning** strategies can be applied on high-cardinality columns (e.g., `date_id`, `restaurant_id`) for further query acceleration.

### Storage Efficiency
- Delta Lake's **columnar storage format** compresses data significantly compared to raw CSV/JSON.
- The Medallion layers act as **checkpoints**, preventing repeated reprocessing of already-cleaned data.

### Reliability
- Bronze layer's **full-fidelity landing** means data can always be reprocessed from the source without re-ingestion.
- Fivetran's **auto sync and scheduling** ensures pipeline resilience without manual intervention.

---



<div align="center">

**Ahmed Adel**
*Data Engineer|Analyst*

🎓 **Databricks Data Analysis — Certified** (DataCamp)

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/eng-ahmedadel30)
[![YouTube](https://img.shields.io/badge/YouTube-FF0000?style=for-the-badge&logo=youtube&logoColor=white)](https://youtu.be/JE_Wmt4087o)

</div>

---
