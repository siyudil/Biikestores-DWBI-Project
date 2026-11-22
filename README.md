# BikeStores Data Warehouse & Business Intelligence Solution

End-to-end **Data Warehousing & BI** project built on the BikeStores retail dataset: a SQL Server **star-schema data warehouse**, an **SSAS multidimensional cube**, **OLAP analysis in Excel**, and interactive **Power BI** reports.

> Module: **IT3021 – Data Warehousing and Business Intelligence** · SLIIT Malabe Campus · Year 3 Semester 1/2 (2026)
> Author: **H.F.S.D. Fonseka** (IT23654280)

![SQL Server](https://img.shields.io/badge/SQL%20Server-DW-CC2927?logo=microsoftsqlserver&logoColor=white)
![SSIS](https://img.shields.io/badge/SSIS-ETL-blue)
![SSAS](https://img.shields.io/badge/SSAS-OLAP%20Cube-0078D4)
![Power BI](https://img.shields.io/badge/Power%20BI-Reports-F2C811?logo=powerbi&logoColor=black)
![Excel](https://img.shields.io/badge/Excel-PivotTables-217346?logo=microsoftexcel&logoColor=white)

---

## 📌 Project Overview

BikeStores is a bicycle retailer with several stores across the USA. Its operational (OLTP) database is great for recording orders, but poor for analysis. This project turns that data into an analytical platform that answers questions such as:

- How do sales trend by **year → quarter → month → day**?
- Which **stores, categories, brands and products** perform best?
- How much revenue is lost to **rejected** orders, and how large are **discounts**?
- How long does a transaction take to complete (accumulating fact)?

**Dataset:** [BikeStores Sample Database (Kaggle)](https://www.kaggle.com/datasets/dillonmyrick/bike-store-sample-database) – 9 normalised tables, orders from **Jan 2016 – Dec 2018**.

## 🏗️ Solution Architecture

```
 ┌──────────────┐   ┌───────────────┐   ┌──────────────┐   ┌────────────────┐   ┌────────────────────┐
 │ Data Sources │ → │ Staging DB    │ → │ ETL (SSIS)   │ → │ Data Warehouse │ → │ BI / Reporting     │
 │ SQL Server   │   │ BikeStores_   │   │ 5 packages   │   │ BikeStores_DW  │   │ SSAS · Excel ·     │
 │ CSV · TXT    │   │ Staging       │   │ SCD2, lookups│   │ Star schema    │   │ Power BI           │
 └──────────────┘   └───────────────┘   └──────────────┘   └────────────────┘   └────────────────────┘
```

Data is deliberately spread across **three source types** (SQL Server, CSV, pipe-delimited TXT) to mimic a real enterprise environment, and enriched with extra attributes (`customer_segment`, `region`, `job_title`) to support hierarchies.

## ⭐ Data Warehouse Design

**Business process:** bike sales transactions · **Grain:** one row per order line item

| Table | Type | Notes |
|---|---|---|
| `FactSales` | Fact (4,722 rows) | Quantity, UnitPrice, Discount, Gross/Discount/Net sales amounts, accumulating-fact columns (`accm_txn_complete_time`, `txn_process_time_hours`) |
| `DimDate` | Dimension | Generated 2016-01-01 → 2018-12-31 |
| `DimProduct` | Dimension | **SCD Type 2** (`EffectiveFrom`, `EffectiveTo`, `IsCurrent`) |
| `DimCustomer` | Dimension | Includes `CustomerSegment` (East Coast / West Coast / Central) |
| `DimStore` | Dimension | Includes `Region` (East / West / Central) |
| `DimStaff` | Dimension | Self-referencing manager hierarchy, `JobTitle` |
| `DimOrderStatus` | Dimension | Pending, Processing, Rejected, Completed |

**Hierarchies:** Date (Year→Quarter→Month→Day) · Product (Category→Brand→Product) · Customer (Segment→State→City→Customer) · Store (Region→State→City→Store) · Staff (JobTitle→Staff)

## 🔄 ETL (SSIS)

| Package | Purpose |
|---|---|
| `pkg_Extract_Sources` | Extract SQL Server, CSV and TXT sources into staging |
| `pkg_Load_Dimensions` | Load all dimensions (SCD Type 2 for `DimProduct`) |
| `pkg_Load_FactSales` | Surrogate-key lookups, derived columns, load fact table |
| `pkg_Update_AccumFact` | Read `OrderCompletions.csv`, update accumulating-fact columns |
| `pkg_Master_ETL` | Orchestrate everything with precedence constraints & error routing |

> The ETL design is documented in [`docs/Assignment1_DW_Design_and_ETL.pdf`](docs/Assignment1_DW_Design_and_ETL.pdf).

## 🧊 SSAS Cube (`ssas-cube/`)

Multidimensional cube **BikeStores Cube** with 6 dimensions and 7 measures (Net/Gross Sales Amount, Discount Amount, Quantity, Unit Price, Txn Process Time Hours, Fact Sales Count), all related via regular surrogate-key relationships. Built in Visual Studio with the Analysis Services Projects extension and deployed to SSAS.

## 📊 OLAP Operations in Excel (`excel-olap/`)

Excel PivotTables connected to the cube demonstrate the five classic OLAP operations: **Roll-up, Drill-down, Slice, Dice and Pivot**.

Key findings:
- Total net sales: **$7,689,109.71**
- **2017** was the strongest year (**$3,447,205.06**)
- Q4 / December consistently led sales
- Santa Cruz Bikes led completed sales in the West region

## 📈 Power BI Reports (`powerbi/`)

Connected to `BikeStores_DW` (Import mode) with DAX measures (*Total Sales, Total Quantity, Total Discount, Gross Sales, Avg Sale Value, Transaction Count, Discount %*).

1. **Matrix report** – Store × Category by Year with subtotals, conditional formatting and KPI cards
2. **Cascading slicers** – Region → Store filtering with bar, line and donut charts
3. **Drill-down report** – Year→Quarter→Month and Category→Brand→Product
4. **Drill-through report** – summary page linking to a detail page

## 📁 Repository Structure

```
bikestores-dwbi-project/
├── README.md
├── .gitignore
├── docs/                      # Assignment 1 (DW design & ETL) and Assignment 2 (cube, OLAP, Power BI) reports
├── data-warehouse/
│   └── BikeStores_DW.bak      # SQL Server backup of the data warehouse
├── ssas-cube/                 # Visual Studio SSAS multidimensional project
├── excel-olap/
│   └── BikeStores_OLAP.xlsx   # Excel PivotTables connected to the cube
└── powerbi/
    └── BikeStores_Reports.pbix
```

## 🚀 Getting Started

**Requirements:** SQL Server + SSMS, SQL Server Analysis Services (multidimensional mode), Visual Studio with *Analysis Services Projects* extension, Excel, Power BI Desktop.

1. **Restore the data warehouse** in SSMS: *Databases → Restore Database → Device →* `data-warehouse/BikeStores_DW.bak`. Or in T-SQL:
   ```sql
   RESTORE DATABASE BikeStores_DW
   FROM DISK = N'C:\path\to\BikeStores_DW.bak'
   WITH REPLACE;
   ```
2. **Verify** the load: `SELECT COUNT(*) FROM FactSales;` → expect **4,722**.
3. **Open the cube project:** open `ssas-cube/BikeStores_Cube.slnx` (or the `.dwproj`), then edit the data source `BikeStores_DW.ds` so the server name points to *your* SQL Server instance (the original was a local machine name).
4. **Deploy & process** the cube to your SSAS instance (set the target server in project properties).
5. **Excel:** open `excel-olap/BikeStores_OLAP.xlsx`; update the data connection to your SSAS server if you want live refresh.
6. **Power BI:** open `powerbi/BikeStores_Reports.pbix`, then *Transform data → Data source settings* and point it to your SQL Server.

## 🛠️ Tech Stack

SQL Server · SSIS · SSAS (Multidimensional) · Visual Studio · Excel PivotTables · Power BI / DAX · Star-schema dimensional modelling · SCD Type 2

## 👤 Author

**H.F.S.D. Fonseka** – IT23654280 | SLIIT Malabe Campus
