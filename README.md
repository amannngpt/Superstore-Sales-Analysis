# 🛒 Superstore Sales Performance & Profitability Analysis

![Dashboard Preview](Screenshots/PowerBI/Page1_Executive_Overview.png)

## 📋 Project Overview

An end-to-end business intelligence project analyzing **9,994 retail transactions** across 4 years (2014–2017), 17 product sub-categories, and 4 US regions. The analysis identifies a **₹70K–₹90K annual profit recovery opportunity** through strategic discount optimization and product portfolio restructuring.

**Tools Used:** Microsoft Excel (Power Query, Pivot Tables, Charts) | Power BI (DAX, Interactive Dashboards) 

**Skills Demonstrated:** Data Cleaning | ETL | EDA | DAX Measures | BCG Matrix Framework | Customer Segmentation | Business Recommendations

---

## 🎯 Business Problems Identified

| # | Problem | Impact |
|---|---------|--------|
| 1 | Overall profit margin of only 12.47% despite ₹22.97L revenue | Profitability crisis |
| 2 | Strong negative correlation (R²=0.61) between discount depth and profitability | Discount strategy misalignment |
| 3 | 3 out of 17 sub-categories are unprofitable (negative overall margin) | Product portfolio imbalance |
| 4 | South region contributes only 17% of revenue vs West at 32% | Regional performance gap |
| 5 | 52% of orders are discounted; top revenue customer (Sean Miller) is unprofitable | Customer profitability blind spot |

---

## 💡 Key Findings

- **Binders** is the most problematic product: 37.23% average discount, -19.96% profit margin, 1,186 out of 1,523 orders are discounted (77.9% discount rate)
- **Tables** loses ₹17,725 annually despite ₹2.06L in revenue — highest absolute loss
- **Copiers** and **Phones** are the top performers: high sales volume with positive margins
- **793 unique customers** placed 9,994 orders — avg 12.6 orders per customer showing strong retention
- **7,655 customers** placed 11+ orders — excellent loyalty base generating ₹2,31,143 in profit
- **Top revenue customer (Sean Miller)** had 13 of 15 orders discounted, resulting in a net loss of -₹1,981
- **Most profitable customer (Tamara Chand)** had only 4 of 12 orders discounted — low discounts = high profit (₹8,981)
- **California and New York** account for 33% of total revenue (₹7,69,000 combined)

---

## 📊 Dashboard Overview (5 Pages)

### Page 1: Executive Overview
![Executive Overview](Screenshots/PowerBI/Page1_Executive_Overview.png)

Key metrics at a glance: Revenue, Profit, Margin %, Orders, Units Sold, Unique Customers. Interactive slicers for Year, Region, Ship Mode, and Sub-Category.

---

### Page 2: Profitability Analysis
![Profitability Analysis](Screenshots/PowerBI/Page2_Profitability_Analysis.png)

Three-dimensional discount impact analysis (depth × frequency × profitability), sub-category performance metrics with Delta column, and profit breakdown by category.

**Bubble Chart Insight:** Products in the bottom-right quadrant (high discount, negative margin) — Binders, Tables, Machines — are the priority targets for intervention.

---

### Page 3: Operational Insights
![Operational Insights](Screenshots/PowerBI/Page3_Operational_Insights.png)

Monthly order volume trends, customer segment distribution, sales trend by product category, top 10 states by revenue, and shipping mode efficiency analysis.

---

### Page 4: Customer & Product Strategy
![Customer & Product Strategy](Screenshots/PowerBI/Page4_Customer_Product_Strategy.png)

Top 10 customers by revenue with discount behavior analysis, customer purchase frequency distribution, and BCG Product Performance Matrix.

**BCG Matrix:** X-axis threshold set at **Median Sales by Sub-Category (₹1,14,880)** using MEDIANX DAX function. Y-axis set at **0% (Break-Even)** as the universal profitability threshold.

---

### Page 5: Additional Tables
![Additional](Screenshots/PowerBI/Page5_Additional.png)

Top 10 customers by total orders and by profitability, with Discounted Orders column revealing discount dependency per customer.

---

## 🔧 Data Cleaning Process

### Raw Data Issues Found:
| Issue | Description | Fix Applied |
|-------|-------------|-------------|
| Date format | Order Date and Ship Date in mixed text formats (MM-DD-YYYY vs M/D/YYYY) | Data → Text to Columns (MDY format) |
| Postal Code | Leading zeros missing (e.g., 6789 instead of 06789) | Power Query custom column: `= Text.PadStart(Text.From([Postal Code]), 5, "0")` |
| Data types | Multiple columns with incorrect data types | Power Query type conversions |

### Power Query Transformations:
![Power Query Steps](Screenshots/DataCleaning/PowerQuery_Applied_Steps.png)

Applied steps include: Source, Changed Type, Added Custom columns (×5), Removed Columns, Reordered Columns, Renamed Columns — fully documented in the Query Settings panel.

### Calculated Columns Added in Power Query:
| Column | Formula | Purpose |
|--------|---------|---------|
| Profit Margin % | `= [Profit] / [Sales]` | Per-transaction profitability |
| Order Year | `= Date.Year([Order Date])` | Time-based filtering |
| Order Month | `= Date.MonthName([Order Date])` | Seasonal analysis |
| Ship Duration (Days) | `= Duration.Days([Ship Date] - [Order Date])` | Shipping efficiency |
| Is Discounted | `= if [Discount] > 0 then 1 else 0` | Accurate discount frequency count |

---

## 📐 DAX Measures & Calculated Columns

All custom measures and calculated columns are organized under a **DAX folder** in the Power BI data model, each with a description for documentation.

| Name | Type | Formula | Purpose |
|------|------|---------|---------|
| Overall Margin % | Measure | `DIVIDE(SUM([Profit]), SUM([Sales]))` | Revenue-weighted business margin |
| Avg Order Value | Measure | `DIVIDE(SUM([Sales]), COUNT([Row ID]))` | Average transaction value |
| Avg Profit Value | Measure | `DIVIDE(SUM([Profit]), COUNT([Row ID]))` | Average profit per transaction |
| Median Sales by SubCategory | Measure | `MEDIANX(VALUES([Sub-Category]), CALCULATE(SUM([Sales])))` | Dynamic X-axis threshold for BCG matrix |
| Unique Customers | Measure | `DISTINCTCOUNT([Customer ID])` | Count of unique customers |
| Orders Per Customer | Calculated Column | `CALCULATE(COUNT([Row ID]), ALLEXCEPT(table, [Customer ID]))` | Total orders per customer (row-level) |
| Purchase Frequency Band | Calculated Column | `SWITCH(TRUE(), [Orders Per Customer]=1, "1 order", ...)` | Customer loyalty segmentation |

---

## 📈 Excel Analysis

### Profitability Analysis
![Excel Profitability](Screenshots/Excel/Excel_Profitability_Analysis.png)

Sub-category performance showing Overall Margin % vs Average Profit Margin % with Delta column, alongside discount effects analysis. Delta = Overall Margin % − Avg Profit Margin % (0.44% at total level, indicating high-value orders have slightly better margins).

### Discount Impact Scatter Plot
![Excel Scatter](Screenshots/Excel/Excel_Scatter_Trendline.png)

Trendline equation: **y = -1.8257x + 0.4014**, R² = **0.6107** — confirming 61% of profit margin variance is explained by discount depth.

### Corrected Discount Analysis
![Excel Discount](Screenshots/Excel/Excel_Discount_Strategy.png)

Corrected analysis using binary "Is Discounted" flag. Total discounted orders: **5,196 out of 9,994** (52%). Previous analysis incorrectly counted all 9,994 orders as discounted.

---

## 💼 Strategic Recommendations

### Immediate Actions (0-30 days):
1. **Cap discounts at 15%** for Binders, Machines, Tables, Bookcases
   - Expected impact: ₹70K–₹90K annual profit improvement
   - Binders alone: 1,186 discounted orders at avg 37.23% — cap to 15% = ~₹45K recovery

2. **Review Sean Miller account** — ₹25,043 revenue but -₹1,981 net loss
   - 13 of 15 orders heavily discounted
   - Switch to standard pricing or renegotiate terms

### Medium-Term Actions (1-6 months):
3. **BCG Portfolio Restructuring:**
   - STARS (Copiers, Phones, Accessories, Storage): Increase marketing investment
   - CASH COWS (Labels, Paper, Envelopes): Maintain, consider bundling with Stars
   - QUESTION MARKS (Binders, Tables, Machines): Fix pricing or discontinue
   - DOGS (Bookcases, Appliances): Phase out within 6 months

4. **South Region Growth Campaign:** Increase from 17% to 20% revenue share (+₹1.15L)

5. **Customer Loyalty Program:** Target 7,655 customers with 11+ orders
   - Increase average orders from 12.6 to 14 per customer = ~₹2.8L additional revenue

### Future Enhancements:
- Calculate COGS per unit to distinguish discount-driven losses from structural losses
- Build price elasticity model to determine optimal discount thresholds
- Add returns/refunds analysis when data becomes available

---

## 📁 Repository Structure
