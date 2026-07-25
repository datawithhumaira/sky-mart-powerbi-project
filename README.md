# SkyMart Power BI Dashboard

An end-to-end Business Intelligence project built in **Power BI Desktop**, using **Power Query** for data cleaning and **DAX** for analysis. Simulates a real BI analyst engagement for a fictional retailer, SkyMart, consolidating messy Excel/CSV exports into one trusted, interactive dashboard.

---

## 📌 Business Problem

SkyMart management needed one trusted BI dashboard because sales, returns, inventory, shipments, survey, and web traffic data all came from different, messy source files. This project acts as the BI analyst work of cleaning, transforming, modeling, and visualizing that data into a decision-ready report.

---

## ❓ Business Questions Answered

| # | Question | Report Page |
|---|---|---|
| 1 | Are sales and profit improving month over month? | Executive Overview, Sales Analysis |
| 2 | Which products, categories, stores, and regions drive revenue? | Sales Analysis, Sales Drilldown |
| 3 | Which products have high return rates and why? | Returns Analysis |
| 4 | Which items are understocked or overstocked? | Inventory Dashboard |
| 5 | Are shipments being delivered on time? | Shipment Operations |
| 6 | How does customer satisfaction differ by region? | Customer Satisfaction |
| 7 | Are actuals meeting budget targets? | Budget vs Actual |

---

## 📊 Report Pages

1. **Executive Overview** — Company-wide KPIs: Total Sales, Profit, Margin, Return Rate, On-Time Delivery.
2. **Sales Analysis** — Monthly trend, MoM % table with conditional formatting, sales by store.
3. **Sales Drilldown** — Interactive decomposition tree (Region → Category → Product → Store).
4. **Returns Analysis** — Return rate by category/brand, return reasons, item condition.
5. **Inventory Dashboard** — Understocked/overstocked counts, per-SKU stock status table.
6. **Shipment Operations** — On-time delivery %, average delivery days, carrier performance.
7. **Customer Satisfaction** — Satisfaction score by province and by survey question.
8. **Budget vs Actual** — Budget variance by month, region, and category.

*(Screenshots of every page are in `/screenshots`.)*

---

## 🔑 Key Insights

- **Sales declined Jan → Apr 2025, then recovered in May–Jun**, ending the period down overall for profit MoM.
- **East region** drives the most revenue ($23.7M), with **Home & Kitchen** the top category.
- **Furniture** has the highest return rate (10.9%); **Prairiepro** is the highest-return brand.
- **~63% of SKU–store combinations are below reorder threshold** — a significant restocking signal.
- **88.2% of shipments arrive on time** (defined as within 5 days of ship date — no promised-date field existed in source data).
- **Customer satisfaction is consistently high (~3.8–3.9/5)** across all provinces and survey categories.
- **Actual profit exceeded budget targets by ~398%** across all months/regions/categories — flagged as a planning-assumption finding, not a data error (see `docs/data_quality_notes.md`).

---

## 🛠️ Tools & Techniques

- **Power Query:** folder-based append, merges/joins across 10+ tables, text cleaning (trim/clean/casing), mixed-date-format parsing, duplicate/blank handling, column splitting, unpivoting survey data.
- **Data Modeling:** star schema, Date table with hierarchy, many-to-many relationship with `USERELATIONSHIP()` for cross-fact filtering (Budget ↔ Sales).
- **DAX:** 25+ measures covering revenue, profit, margin, MoM trend, return rate, stock status, delivery performance, satisfaction, and budget variance. Full list in `DAX_measures.md`.
- **Report Design:** custom theme, consistent KPI/chart/table layout across pages, decomposition tree, conditional formatting, page navigation buttons.

---

## 📁 Repo Structure

```
skymart-powerbi-dashboard/
├── SkyMart_Dashboard.pbix
├── README.md
├── screenshots/
├── DAX_measures.md
├── data_quality_notes.md

---

## 🚀 How to Use

Download `SkyMart_Power_BI_Project.pbix`.



---

## 📄 License

MIT — feel free to reference the structure/approach for your own learning projects.
