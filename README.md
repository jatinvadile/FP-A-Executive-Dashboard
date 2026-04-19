# 📊 FP&A Dashboard - Financial Planning & Analysis Platform

> A full-stack financial planning and analysis solution integrating **SQL Server**, **Swagger API**, **Python**, and **Power BI** - consolidating operational and financial data into a single decision-ready reporting layer for management reviews, KPI tracking, and variance analysis.

---

## 📌 Table of Contents

- [Overview](#-overview)
- [Business Context](#-business-context)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Data Sources](#-data-sources)
- [End-to-End Pipeline](#-end-to-end-pipeline)
- [Swagger API Integration](#-swagger-api-integration)
- [SQL Server Layer](#-sql-server-layer)
- [Data Transformation](#-data-transformation)
- [Dashboard Views & KPIs](#-dashboard-views--kpis)
- [Business Impact](#-business-impact)
- [Challenges & Solutions](#-challenges--solutions)
- [Getting Started](#-getting-started)
- [Future Enhancements](#-future-enhancements)

---

## 📖 Overview

This project delivers a **centralized FP&A Dashboard** that brings together operational and financial data from SQL Server, Swagger APIs, and Excel into a unified analytical layer. It goes beyond standard dashboarding-combining data engineering, API integration, SQL staging, and Power BI modeling into a platform that supports:

- Monthly and periodic business reviews
- Actual vs. target variance tracking
- Trend and contribution analysis
- Operational volume monitoring
- Management-level planning discussions

---

## 🔴 Business Context

Before this solution, performance data was scattered across disconnected systems with no unified, decision-ready view available.

| Business Question | Previously |
|---|---|
| How is actual performance trending? | Manual compilation across files |
| How is volume moving across categories? | No consolidated view |
| Which areas are driving growth or variance? | Checked separately per system |
| How do operational movements affect planning? | Not linked to financial data |
| Can management get one reliable view? | No-multiple files, no single source |

---

## 🏗️ Architecture

```
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│  Swagger API     │  │   SQL Server    │  │     Excel       │
│  (Volume Data)   │  │  (Structured    │  │  (Maintained    │
│  Inventory Units │  │   Business Data)│  │   Inputs /      │
│  + Pagination    │  │                 │  │   Ref Tables)   │
└────────┬────────┘  └────────┬────────┘  └────────┬────────┘
         │                    │                     │
         ▼                    │                     │
┌─────────────────┐           │                     │
│  Python Script  │           │                     │
│  · API Auth     │           │                     │
│  · Pagination   │           │                     │
│  · JSON Flatten │           │                     │
│  · SQL Insert   │           │                     │
└────────┬────────┘           │                     │
         │                    │                     │
         ▼                    ▼                     ▼
┌──────────────────────────────────────────────────────┐
│              SQL Server-teagtlvolume DB             │
│              dbo.inventory_units (staging table)      │
└──────────────────────────┬───────────────────────────┘
                           │
                           ▼
              ┌────────────────────────┐
              │   Power BI             │
              │   · Data Modeling      │
              │   · DAX Measures       │
              │   · Business KPIs      │
              └────────────┬───────────┘
                           │
                           ▼
              ┌────────────────────────┐
              │   FP&A Dashboard       │
              │   · Summary KPIs       │
              │   · Trend Analysis     │
              │   · Variance Tracking  │
              │   · Category Breakdown │
              │   · Planning Views     │
              └────────────────────────┘
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| **Operational Data Source** | Swagger REST API (Inventory Units endpoint) |
| **Structured Data Source** | SQL Server (Windows Auth · ODBC Driver 17) |
| **Supplementary Source** | Excel (reference tables, maintained inputs) |
| **Ingestion & Transformation** | Python · Requests · Pandas · pyodbc |
| **Storage / Staging** | SQL Server-`teagtlvolume` database |
| **Data Modeling** | Power BI (dimensional model + DAX) |
| **Visualization** | Power BI Desktop / Service |

---

## 📂 Data Sources

### 1. Swagger API-Operational Volume Data
- **Endpoint:** Inventory Units (`/inventory-units`)
- Fetches real-time operational volume records with pagination support
- Extracts nested JSON fields: category, freightKind, transitState, visitState, equipment details, positions, routing, and timestamps
- Feeds the SQL Server staging layer for reporting

### 2. SQL Server-Core Structured Source
- Central storage for transformed, reporting-ready operational data
- Database: `teagtlvolume`
- Primary table: `dbo.inventory_units`
- Acts as both a staging and reporting repository

### 3. Excel-Maintained Business Inputs
- Supplementary reference tables and historical inputs
- Used where controlled manual or maintained data is required (e.g., exchange rate history, lookup tables)
- Appended with duplicate prevention logic

---

## 🔄 End-to-End Pipeline

The solution is built across five sequential stages:

### Stage 1-Data Extraction
Raw data collected from three source types simultaneously:
- Swagger API → operational volume records
- SQL Server → structured business data
- Excel → maintained reference inputs

### Stage 2-Ingestion & Storage (Python → SQL Server)
```python
# Pipeline workflow
# 1. Connect to Swagger API with auth token
# 2. Paginate through all inventory unit records
# 3. Parse and flatten nested JSON response fields
# 4. Map API fields to relational column structure
# 5. Create database + table if not exists (teagtlvolume / dbo.inventory_units)
# 6. Clear stale data, insert transformed records row-by-row
# 7. Confirm load with row count validation
```

**Key helper functions in the ingestion script:**

| Function | Purpose |
|---|---|
| Epoch converter | Converts API timestamp fields to readable datetime |
| Nested key extractor | Safely pulls values from deeply nested JSON |
| Boolean normalizer | Standardizes True/False/null API flag fields |
| JSON-to-tabular mapper | Flattens full API record into relational row |

### Stage 3-Data Transformation
- Standardized date formats across all sources
- Aligned business categories and dimensional labels
- Converted data types and handled nulls
- Created period fields: Month, Quarter, Year
- Built business-friendly dimension views

### Stage 4-Power BI Modeling
- Loaded SQL Server and Excel sources via Power Query
- Applied dimensional modeling with fact and dimension tables
- Built reusable DAX measures for KPIs, trends, and variance
- Configured relationships and filter context for cross-visual analysis

### Stage 5-Dashboard & Insights
Interactive reporting pages designed for management reviews, planning discussions, and operational monitoring.

---

## 🔌 Swagger API Integration

A key technical differentiator of this project is the live API integration for operational volume data.

**Integration Details:**

| Aspect | Implementation |
|---|---|
| Endpoint | Inventory Units REST endpoint |
| Authentication | Bearer token passed via request headers |
| Parameters | `operator`, `page`, `size` |
| Pagination | Iterative page-by-page fetch until exhausted |
| Response parsing | Handles `content`, `data`, and direct list formats |
| Field extraction | Deep nested JSON flattening to tabular rows |
| Output | Structured records inserted into SQL Server |

**Fields extracted from API response (sample):**

```
category · freightKind · transitState · visitState
equipment type · equipment size · position details
routing info · operator · timestamps · boolean flags
```

This demonstrates the dashboard was built on **live, dynamic operational data**-not static exports-which is a significant value-add in an FP&A context where planning decisions depend on current operational reality.

---

## 🗄️ SQL Server Layer

SQL Server was used as an active staging and reporting repository-not a passive source.

**Database setup handled by ingestion script:**

```sql
-- Auto-created by Python script on first run
CREATE DATABASE teagtlvolume;

-- Wide operational staging table
CREATE TABLE dbo.inventory_units (
    -- Flattened fields from API JSON response
    -- category, freightKind, transitState, visitState,
    -- equipment details, routing, timestamps, flags...
);
```

**Script logic summary:**

| Step | Action |
|---|---|
| Connection | ODBC Driver 17 · Windows Authentication |
| DB creation | Creates `teagtlvolume` if not exists |
| Table creation | Creates `dbo.inventory_units` if not exists |
| Data refresh | Clears old records before each load |
| Insert | Row-by-row insert of transformed API records |

---

## 📊 Dashboard Views & KPIs

### KPI Categories

| KPI Type | Examples |
|---|---|
| **Volume KPIs** | Total units, movement counts by category |
| **Trend Metrics** | Month-over-month movement, directional trend |
| **Category Analysis** | FreightKind, transitState, visitState splits |
| **Operational Breakdown** | Equipment type, routing, operator views |
| **Variance Indicators** | Actual vs. plan, period-over-period delta |
| **Contribution Analysis** | Source/segment-wise share of total |

### Dashboard Pages

| Page | Purpose |
|---|---|
| **Summary** | Overall KPIs-at-a-glance financial & operational position |
| **Trend Analysis** | Monthly movement, directional performance over time |
| **Category Breakdown** | Volume and value split by business category |
| **Operational View** | State-level breakdown-transit, visit, freight kind |
| **Variance & Comparison** | Actual vs. target, period comparisons |
| **Contribution Analysis** | Which segments drive growth, decline, or variance |
| **Planning Support** | Management review views for budgeting discussions |

---

## 💼 Business Impact

| Outcome | Detail |
|---|---|
| Unified reporting layer | SQL + API + Excel → one Power BI model |
| Faster business reviews | Dashboard-ready data instead of manual compilation |
| Improved reporting confidence | Single, validated source of truth |
| Operational + financial alignment | Volume movements linked to planning context |
| Reduced manual effort | Automated ingestion replaces periodic file exports |
| Scalable foundation | Architecture supports future forecasting modules |

---

## 🧩 Challenges & Solutions

| Challenge | Solution |
|---|---|
| Nested and paginated API responses | Python pagination loop + nested JSON extractor |
| Aligning operational data with planning structure | Field mapping layer before SQL insert |
| Inconsistent date formats across sources | Epoch converter + standardization in transform step |
| Null and incomplete API records | Boolean normalizer + null-safe key extractor |
| Multi-source schema alignment | Common dimensional structure defined before modeling |
| Designing scalable SQL schema | Wide staging table with all API fields normalized |
| Dashboard usability vs. technical complexity | Planning-oriented page structure, not technical display |

---

## 🚀 Getting Started

### Prerequisites
- Python 3.8+
- SQL Server (any edition) with ODBC Driver 17
- Power BI Desktop (latest)
- Swagger API access credentials (auth token)

### Python Setup

```bash
pip install requests pandas pyodbc openpyxl
```

### Configuration

```python
# config.py
API_BASE_URL   = "https://your-api-host/inventory-units"
AUTH_TOKEN     = "your_bearer_token_here"
OPERATOR       = "your_operator_code"
PAGE_SIZE      = 100

SQL_SERVER     = "your_server_name"
DATABASE       = "teagtlvolume"
```

### Run Ingestion Pipeline

```bash
python ingest_inventory_units.py
```

This will:
1. Connect to the Swagger API and paginate through all records
2. Flatten and transform each JSON record
3. Create the `teagtlvolume` database and `dbo.inventory_units` table if they don't exist
4. Clear old records and insert the latest dataset
5. Print row count confirmation on completion

### Power BI Setup

1. Open `fpa_dashboard.pbix` in Power BI Desktop
2. Go to **Transform Data → Data Source Settings**
3. Update the SQL Server connection to your server name
4. Update any Excel file paths if used for reference tables
5. Click **Refresh All**-all pages update from the latest SQL data

---

## 🔮 Future Enhancements

- [ ] **Forecasting module**-ML-based volume and revenue forecasting (Prophet / ARIMA)
- [ ] **Automated target uploads**-structured Excel template → SQL for plan vs. actual tracking
- [ ] **Scheduled ingestion**-Airflow or Windows Task Scheduler for daily automated API pulls
- [ ] **Row-level security (RLS)**-department or region-based access control in Power BI Service
- [ ] **Alert system**-notify stakeholders when KPIs breach defined thresholds
- [ ] **Power BI Service deployment**-cloud publish with scheduled refresh and shared workspaces
- [ ] **Additional API sources**-extend to financial transaction APIs for richer FP&A coverage

---

## 👤 Author

**Aerunkar Abhishek**
Data & Analytics-Caterpillar Signs Private Limited

---

> *From raw API feeds to boardroom planning views-engineering meets financial analysis.*
