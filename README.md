# 📊 Project Title: E-Commerce Sales Data Transformation & Relational Data Modeling
A Power BI data preparation, transformation, and relational modeling solution that consolidates order headers, line-item details, and monthly category targets to uncover profit margins, loss-making products, and regional sales performance.

---

## 📑 Table of Contents
- [Project Overview](#-project-overview)
- [Data Sources & Architecture](#-data-sources--architecture)
- [Data Transformation (ETL)](#-data-transformation-etl)
- [Data Model & DAX](#-data-model--dax)
- [Key Insights](#-key-insights)
- [How To Use](#-how-to-use)

---

## 🎯 Project Overview
- **Business Problem:** Raw operational sales exports (`List of Orders`, `Order Details`, and `Sales target`) exist as disconnected flat files with inconsistent customer name capitalization, split city/state fields, and no line-item profitability indicators—preventing accurate tracking of profit margins, loss-making orders, and monthly sales targets.
- **Objective:** Build a clean, auditable, and relationally linked Power BI data model that classifies order profitability (`Profit`, `Loss`, `Break-even`), aggregates category and sub-category performance, and aligns actual sales against monthly revenue targets.
- **Target Audience:** E-Commerce Operations Managers, Category Managers, Regional Sales Leads, and Business Analytics Teams.

---

## 🗃️ Data Sources & Architecture
- **Source Systems:** Local CSV files (`List of Orders.csv`, `Order Details.csv`, and `Sales target.csv`).
- **Data Volume:** 
  - **List of Orders:** 500 order header records (restricted via *Keep Top Rows*).
  - **Order Details:** 1,500 product line-item records across 3 categories (`Electronics`, `Clothing`, `Furniture`) and 17 sub-categories.
  - **Sales Target:** 36 monthly category target records spanning a 12-month cycle (`Apr 2018 – Mar 2019`).
- **Storage Mode:** Import Mode (ingested and transformed via Power Query Editor).

---

## ⚙️ Data Transformation (ETL)
- **Tool Used:** Power Query Editor (Microsoft Power BI Desktop).
- **Key Cleanups:**
  - **Row Limiting & Schema Enforcement:** Restricted `List of Orders` to the top 500 rows; standardized `Order Date` and `Month of Order Date` to **Date**, `Amount` and `Target` to **Fixed Decimal / Currency (`$`)**, and `Profit` and `Quantity` to **Whole Number**.
  - **Text Standardization:** Applied *Capitalize Each Word* (`Text.Proper`) on `CustomerName` for uniform capitalization.
  - **Column Creation & Merging:** Merged `City` and `State` with a comma delimiter into a unified `Location` column (`City,State`); created `Profit Margin` (`[Profit] / [Amount]`) formatted as a percentage.
  - **Conditional Logic:** Created `Profit Status` to categorize transactions into **Loss** (`Profit < 0`), **Break-even** (`Profit = 0`), and **Profit** (`Profit > 0`).
  - **Table Joins & Aggregations:** Merged `List of Orders` and `Order Details` on `Order ID` into a consolidated query (`Orders Data`); filtered `Location` for `Chennai,Tamil Nadu` sorted descending by `Order Date`; built *Group By* summary tables for order counts, average profit by category, total sales by sub-category, and monthly targets.
- **Custom Functions / M Code Scripts:**
  - `Table.CombineColumns(#"Capitalized Each Word", {"City", "State"}, Combiner.CombineTextByDelimiter(",", QuoteStyle.None), "Location")`
  - `Table.AddColumn(#"Changed column type", "Profit Margin", each [Profit] / [Amount])`
  - `Table.AddColumn(#"Added custom", "Profit Status", each if [Profit Margin] < 0 then "Loss" else if [Profit Margin] = 0 then "Break-even" else if [Profit Margin] > 0 then "Profit" else "")`

---

## 🧠 Data Model & DAX
- **Model Type:** Relational Schema (One-to-Many `1:*` relationships linking order headers to line-item detail tables).
- **Fact Tables:** 
  - `Order Details` (line-item amounts, quantities, profits, and margins)
  - `Orders Data` (merged order-and-detail analytical table)
  - `Sales target` (monthly category revenue benchmarks)
- **Dimension / Lookup Tables:** 
  - `List of Orders` (unique `Order ID` header table with `CustomerName`, `Location`, and `Order Date`)
- **Key Measures:**
  ```dax
  Total Sales Amount = SUM('Order Details'[Amount])
  Total Profit = SUM('Order Details'[Profit])
  Overall Profit Margin % = DIVIDE([Total Profit], [Total Sales Amount], 0)
  Total Orders = DISTINCTCOUNT('List of Orders'[Order ID])
  Total Target = SUM('Sales target'[Target])
  Target Variance = [Total Sales Amount] - [Total Target]
💡 Key Insights
Trend A (Category Profitability Leader): Electronics delivers the highest average profit per order line at $34.07 (outperforming Clothing at $11.76 and Furniture at $9.46), while Printers ($58,252), Bookcases ($56,861), and Saree ($53,511) lead sub-category sales volume.

Trend B (Margin Erosion in Heavy Furniture): Despite high top-line revenue in Bookcases, Furniture suffers from frequent negative-margin orders (such as -90.04% on Order B-25601 and -136.66% on Order B-25608 in Tables), dragging down overall category profitability.

Recommendation: Enforce minimum margin thresholds and review discount/shipping structures on loss-making Furniture sub-categories (Bookcases and Tables), while prioritizing inventory and promotional campaigns in high-margin Electronics and Clothing sub-categories to meet the +39.5% monthly target ramp ($31,400 to $43,800).

🚀 How To Use
Prerequisites: Ensure you have the latest version of Microsoft Power BI Desktop installed.

File Formats: Clone or download this repository to access PowerBI-Assignment.pbix, the raw CSV datasets (List of Orders.csv, Order Details.csv, Sales target.csv), the transformation documentation (Data_Transformation_and_Data_Modeling_PowerBI.pdf), and the screenshots/ folder.

Data Refresh: Open PowerBI-Assignment.pbix in Power BI Desktop, navigate to Transform data → Data source settings, update the local file paths to point to your downloaded CSV files, and click Refresh to load the data model.
