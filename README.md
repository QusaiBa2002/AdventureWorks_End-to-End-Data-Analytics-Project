# AdventureWorks_End-to-End-Data-Analytics-Project
End-to-end data analytics project using Power BI featuring advanced DAX (RFM, BCG, ABC Pareto), Data Modeling, and Market Basket Analysis.
# 📊 AdventureWorks Executive Sales & Customer Analytics (Power BI)

![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-Advanced-blue?style=for-the-badge)
![Data Modeling](https://img.shields.io/badge/Data_Modeling-Star_Schema-green?style=for-the-badge)

## 📌 Business Overview
This project presents an end-to-end Power BI analytics solution built on the **AdventureWorks** enterprise dataset. The primary objective is to transform raw sales, product, and customer transactional data into actionable strategic insights through advanced analytics, DAX modeling, and interactive reporting.

---

## 🔑 Key Analytical Models & Features

### 1. Executive Overview & Time Intelligence
- Tracks core KPIs including **Total Revenue**, **Profit Margins**, and **Total Orders**.
- Dynamic Year-over-Year (YoY) performance and growth rate evaluation.

### 2. Product Portfolio Analytics & Pareto Logic
- **ABC / Pareto Product Classification:** Categorizes products into **A, B, C** classes based on cumulative profit contribution (80/15/5 rule) while dynamically isolating **Dead Stock**.
- **BCG Growth-Share Matrix:** Classifies active products into **Stars, Cash Cows, Question Marks, and Dogs** relative to total company growth and market share lines.
- **Market Basket Analysis:** Evaluates co-purchasing patterns using disconnected product comparison tables (`Table_Product_Compare`) to support cross-selling strategies.
- **Inventory Metrics:** Tracks **Slow-Moving Products Rate %** and **Active Products Rate %**.

### 3. Advanced Customer Behavior & RFM Segmentation
- **RFM Score Model:** Implements statistical percentile logic (`PERCENTILE.INC`) to assign granular **Recency, Frequency, and Monetary (RFM)** scores (1–3), segmenting customers into actionable behavioral groups (*Top Customers, Churn Risk, Loyal, etc.*).
- **Dynamic Churn Tracking:** Identifies lost customers using relative 365-day vs. 90-day purchase windows as of the active date axis.
- **Dynamic Customer Lifetime Value (CLV):** Computes time-decayed CLV by factoring average revenue per customer against dynamic churn rates (`Avg Revenue Per Customer / Category Churn Rate %`).

---

## 🛠️ Data Architecture & Modeling

- **Data Schema:** Designed a high-performance **Star Schema** with `Fact_Sales` linked to dimensions (`DimProduct`, `DimCustomer`, `Date_Table`, `Table_Product_Compare`).
- **DAX Techniques:** Utilizes context transition, dynamic filtering (`CALCULATE`, `FILTER`, `ALLEXCEPT`, `ALLSELECTED`), statistical percentiles, and string concatenation for composite scores.

---

## 📂 Repository Structure

```text
├── 📄 README.md                 <-- Project Overview & Documentation
├── 📄 DAX_Documentation.md       <-- Detailed DAX Measures & Logic Breakdown
├── 📊 AdventureWorks_Report.pbix <-- Power BI Report File
└── 📁 Images/                   <-- Screenshots of Dashboard Pages
