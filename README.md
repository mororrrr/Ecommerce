# 🛒 E-Commerce Data Warehouse & Business Intelligence Analytics

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue?logo=python&logoColor=white)](https://www.python.org/)
[![Pandas](https://img.shields.io/badge/Pandas-2.0%2B-150458?logo=pandas&logoColor=white)](https://pandas.pydata.org/)
[![Matplotlib](https://img.shields.io/badge/Matplotlib-Visualization-orange?logo=matplotlib&logoColor=white)](https://matplotlib.org/)
[![Data Warehouse](https://img.shields.io/badge/Architecture-Star--Schema-success)](#-data-architecture--dimensional-model)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

An end-to-end data analytics and dimensional modeling project engineered to transform raw, noisy e-commerce transactional data into an enterprise-ready Data Warehouse (Star Schema). The project delivers diagnostic and exploratory business insights across logistics fulfillment, regional profitability, product category demand, discount elasticity, and marketing ROI.

---

## 📌 Table of Contents
- [Project Overview](#-project-overview)
- [Data Architecture & Dimensional Model](#-data-architecture--dimensional-model)
- [ETL & Data Quality Engineering](#-etl--data-quality-engineering)
- [Executive Business Insights & Visualizations](#-executive-business-insights--visualizations)
  - [1. Logistics Impact: Delivery Speed vs. Customer Rating & Refunds](#1-logistics-impact-delivery-speed-vs-customer-rating--refunds)
  - [2. Regional Profitability Hotspots (Top 10 Cities)](#2-regional-profitability-hotspots-top-10-cities)
  - [3. Product Catalog Performance (Subcategory Net Sales)](#3-product-catalog-performance-subcategory-net-sales)
  - [4. Price Elasticity: Discount Tiers vs. Volume](#4-price-elasticity-discount-tiers-vs-volume)
  - [5. Marketing Capital Allocation: Campaign ROI Efficiency](#5-marketing-capital-allocation-campaign-roi-efficiency)
- [Strategic Business Recommendations](#-strategic-business-recommendations)
- [Repository Structure](#-repository-structure)
- [Getting Started & Reproduction](#-getting-started--reproduction)
- [Interview & Technical Deep-Dive Guide](#-interview--technical-deep-dive-guide)

---

## 🎯 Project Overview

In e-commerce operations, raw data ingested from transactional systems (OLTP) is frequently plagued by duplicates, missing financial fields, negative fulfillment intervals, and unaligned dimensional keys. 

This project establishes:
1. **A Reliable ETL Cleaning Pipeline**: Cleanses, standardizes, and validates transactional order logs and dimension tables to guarantee relational database and SQL loading integrity (`DECIMAL`, `PRIMARY KEY`, `FOREIGN KEY`).
2. **Actionable Business Intelligence (BI)**: Diagnoses key margin drivers, logistical bottlenecks, promotional efficiency, and marketing spend returns through rigorous analytical aggregations and visualizations.

---

## 🏗 Data Architecture & Dimensional Model

The pipeline prepares data for ingestion into a Kimball-style **Star Schema** Data Warehouse:

```
                  +------------------------+
                  |    dim_order_status    |
                  +------------------------+
                  | PK | Order_Status_Key  |
                  |    | Order_Status      |
                  +-----------+------------+
                              |
                              | 1:N
                              v
+-----------------------+   +-------------------+   +----------------------+
|     dim_geography     |   |    fact_orders    |   |    dim_marketing     |
+-----------------------+   +-------------------+   +----------------------+
| PK | Geography_Key    |<--| PK | Order_ID     |-->| PK | Marketing_Key   |
|    | City             |   | FK | Status_Key   |   |    | Campaign        |
|    | Country          |   | FK | Geogr_Key    |   +----------------------+
+-----------------------+   | FK | Product_Key  |
                            | FK | Market_Key   |   +----------------------+
                            |    | Delivery_Days|   |     dim_product      |
                            |    | Gross_Sales  |   +----------------------+
                            |    | Net_Sales    |<--| PK | Product_Key     |
                            |    | Profit       |   |    | Category        |
                            |    | Shipping_Cost|   |    | Subcategory     |
                            |    | Refund_Amount|   +----------------------+
                            |    | Rating, etc. |
                            +-------------------+
```

---

## ⚙️ ETL & Data Quality Engineering

Data cleaning logic was specifically designed to resolve operational database constraints prior to downstream data warehouse ingestion:

| Challenge | Issue in Raw Data | Engineering Solution & Impact |
| :--- | :--- | :--- |
| **Primary Key Violation** | Duplicate `Order_ID` records | Applied `.drop_duplicates(subset=['Order_ID'])` to guarantee unique entity constraints in SQL tables. |
| **Domain Logic Anomaly** | Negative delivery durations (e.g. `-2` days) | Filtered via boolean masking (`Delivery_Days >= 0`) to eliminate corrupted fulfillment timestamps. |
| **Financial Null Values** | Missing entries in `Gross_Sales`, `COGS`, `Profit`, etc. | Imputed nulls with `0.00` to prevent `NULL` calculation errors in aggregation metrics. |
| **Data Type Precision** | High floating-point variances | Applied `.round(2)` to comply strictly with database `DECIMAL(12, 2)` financial types. |
| **Dimension Discrepancies**| Missing status and campaign labels | Imputed `'Unknown'` for order status and `'No Campaign'` for organic acquisition. |

---

## 📊 Executive Business Insights & Visualizations

### 1. Logistics Impact: Delivery Speed vs. Customer Rating & Refunds
* **Objective**: Evaluate how delivery duration affects customer satisfaction and reverse logistics expense.
* **Finding**: As delivery days increase, customer review ratings experience an immediate drop, alongside a spike in refund values.
* **Business Impact**: Shows that logistics speed is directly correlated with customer lifetime value (LTV) and margin protection.

### 2. Regional Profitability Hotspots (Top 10 Cities)
* **Objective**: Identify the geographical centers contributing the highest net profit.
* **Finding**: The top 10 cities generate the vast majority of cumulative profit, highlighting clear regional density.
* **Business Impact**: Allows supply chain teams to position fulfillment hubs closer to high-margin hubs and reallocate local marketing spend.

### 3. Product Catalog Performance (Subcategory Net Sales)
* **Objective**: Break down gross volume across merchandise categories and subcategories.
* **Finding**: A select cluster of subcategories drives over 60% of total net sales volume.
* **Business Impact**: Directs inventory holding prioritization, working capital allocation, and supplier contract negotiations.

### 4. Price Elasticity: Discount Tiers vs. Volume
* **Objective**: Assess the impact of discounting on purchase volumes (`No Discount`, `Low 1-10%`, `Medium 11-20%`, `High >20%`).
* **Finding**: While volume increases with higher discount tiers, aggressive discounting (>20%) significantly compresses profit margins without proportional demand growth.
* **Business Impact**: Informs strategic guardrails against margin cannibalization.

### 5. Marketing Capital Allocation: Campaign ROI Efficiency
* **Objective**: Measure true campaign efficacy by computing Return on Investment:
  $$	ext{ROI (\%)} = rac{	ext{Total Profit} - 	ext{Total Marketing Cost}}{	ext{Total Marketing Cost}} 	imes 100$$
* **Finding**: Several campaigns yield outstanding triple-digit ROI, whereas others operate at negative returns.
* **Business Impact**: Immediate termination or restructuring of negative-ROI ad channels to preserve EBITDA.

---

## 💡 Strategic Business Recommendations

1. **Service Level Agreement (SLA) Tightening**: Implement an alert threshold for delivery timelines exceeding 4 days to preemptively curb customer dissatisfaction and returns.
2. **Promotional Discipline**: Cap automated discounting at 15% for high-velocity SKUs and substitute deep discounts with bundle/loyalty incentives.
3. **Targeted Growth Reallocation**: Shift marketing capital from underperforming channels to top-performing regional clusters and high-ROI campaigns.

---

## 📂 Repository Structure

```text
├── Ecommerce_DW.ipynb          # Jupyter / Google Colab pipeline (ETL + EDA + Charts)
├── sample_data/               # Raw and transformed CSV datasets
│   ├── fact_orders.csv
│   ├── dim_order_status.csv
│   ├── dim_marketing.csv
│   ├── clean_fact_orders.csv
│   └── clean_dim_marketing.csv
├── requirements.txt           # Python dependencies
├── LICENSE                    # MIT License
└── README.md                  # Project documentation & business brief
```

---

## 🚀 Getting Started & Reproduction

### Prerequisites
* Python 3.10+
* Jupyter Notebook or Google Colab

### Installation
```bash
# 1. Clone this repository
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>

# 2. Install dependencies
pip install -r requirements.txt

# 3. Launch Jupyter Notebook
jupyter notebook Ecommerce_DW.ipynb
```

---

## 💼 Interview & Technical Deep-Dive Guide

When presenting this project in a data analyst or analytics engineering interview, structure your narrative using the **STAR Method**:

### 🎯 Project Story (STAR Framework)
* **Situation**: E-commerce transactional datasets had inconsistent logging, duplicate order records, and fragmented dimensional tables, hindering accurate financial reporting.
* **Task**: Build a robust Python ETL pipeline to sanitize the data for SQL Data Warehouse ingestion and extract diagnostic analytics on logistics, margins, and marketing returns.
* **Action**:
  - Implemented deduplication on unique business keys to protect SQL primary key integrity.
  - Imputed missing financial values and standardized decimal precision.
  - Built segmented aggregations analyzing the interaction between fulfillment delays, product performance, discount tiers, and campaign ROI.
* **Result**: Clean data ready for data warehousing and five prioritized strategic levers that directly improve operating margins and fulfillment efficiency.

### ❓ Sample Interview Questions & Model Answers

<details>
<summary><b>Q1: Why did you perform data cleaning in Python instead of directly inside the SQL database?</b></summary>
<br>
<i>"Handling preliminary data hygiene (like dropping duplicates on primary keys, filtering invalid delivery records, and handling floating-point nulls) prior to database loading prevents bulk staging job failures, avoids constraint violation rollbacks, and reduces processing load on production databases."</i>
</details>

<details>
<summary><b>Q2: How did you evaluate marketing performance beyond just looking at revenue?</b></summary>
<br>
<i>"Revenue can be misleading if ad spend or cost of goods sold is high. I calculated true campaign ROI using <code>(Profit - Marketing Cost) / Marketing Cost * 100</code>. This revealed campaigns that were generating revenue but destroying margin, separating genuine profit drivers from vanity metrics."</i>
</details>

<details>
<summary><b>Q3: What would be the next step to scale this project to production?</b></summary>
<br>
<i>"I would containerize the pipeline, orchestrate data ingestion with tools like dbt or Airflow, automate daily data quality tests (e.g., Great Expectations), and connect the resulting clean tables to an interactive BI dashboard (Power BI or Tableau) for real-time stakeholder reporting."</i>
</details>

---

## 👤 Author
* **Portfolio / GitHub**: [@YourUsername](https://github.com/YourUsername)
* **LinkedIn**: [Connect on LinkedIn](https://linkedin.com/in/YourProfile)
