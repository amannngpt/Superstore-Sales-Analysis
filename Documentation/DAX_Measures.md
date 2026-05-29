# DAX Measures & Calculated Columns Documentation

This file documents every custom measure and calculated column created in Power BI for the Superstore Sales Analysis project. All items are organized under a **DAX display folder** in the Power BI data model, each with a written description visible on hover in the Data pane.

---

## Measures
*Measures are calculated dynamically based on the current filter context (slicers, page filters, visual-level filters). They are NOT stored row by row — they recalculate every time a filter changes.*

---

### 1. Overall Margin %

**Formula:**
```dax
Overall Margin % =
DIVIDE(
    SUM('Superstore_Sales_Cleaned'[Profit]),
    SUM('Superstore_Sales_Cleaned'[Sales])
)
```

**Purpose:** Revenue-weighted overall business profit margin.

**Result (unfiltered):** 12.47%

**Why DIVIDE instead of `/`:** The `DIVIDE` function handles division-by-zero gracefully — it returns BLANK instead of an error when the denominator is zero (e.g., when a slicer filters to a sub-set with no sales). The `/` operator would crash the visual.

**How it differs from Average Profit Margin %:**
This measure weights each transaction by its revenue. A ₹1,000 order with 15% margin has 20× more influence on this result than a ₹50 order with 40% margin. This is appropriate for executive-level business performance reporting.

Average Profit Margin % (the column average) treats every transaction equally — a ₹50 order counts the same as a ₹1,000 order. This is appropriate for understanding typical per-transaction profitability.

**Result difference:** Overall Margin % = 12.47%, Avg Profit Margin % = 12.03%. The 0.44% gap (Delta) indicates that higher-value orders in this dataset tend to carry slightly better margins.

**Used in:** KPI card (Page 1), Sub-Category Performance Metrics table (Page 2), Map tooltip (Page 1).

---

### 2. Avg Order Value

**Formula:**
```dax
Avg Order Value =
DIVIDE(
    SUM('Superstore_Sales_Cleaned'[Sales]),
    COUNT('Superstore_Sales_Cleaned'[Row ID])
)
```

**Purpose:** Average revenue per transaction (order line).

**Result (unfiltered):** approximately ₹230 per order line

**Why COUNT(Row ID) not DISTINCTCOUNT(Order ID):** Each Order ID can contain multiple product lines (Row IDs). We want the average revenue per product line ordered, not per unique order.

**Used in:** Top 10 Customers by Revenue table (Page 4), Top 10 Customers by Orders table (Page 5).

---

### 3. Avg Profit Value

**Formula:**
```dax
Avg Profit Value =
DIVIDE(
    SUM('Superstore_Sales_Cleaned'[Profit]),
    COUNT('Superstore_Sales_Cleaned'[Row ID])
)
```

**Purpose:** Average profit per transaction (order line).

**Used in:** Top 10 Customers by Orders table (Page 5), Top 10 Customers by Profit table (Page 5).

**Insight it enables:** Comparing Avg Order Value with Avg Profit Value for the same customer reveals their average margin per transaction — useful for identifying customers who order frequently but at low or negative margins.

---

### 4. Median Sales by SubCategory

**Formula:**
```dax
Median Sales by SubCategory =
MEDIANX(
    VALUES('Superstore_Sales_Cleaned'[Sub-Category]),
    CALCULATE(SUM('Superstore_Sales_Cleaned'[Sales]))
)
```

**Purpose:** Calculates the median of total sales values across all 17 sub-categories. Used as the dynamic X-axis threshold line in the BCG Product Performance Matrix.

**Result:** ₹1,14,880

**Why MEDIANX and not MEDIAN:**
`MEDIAN([Sales])` would calculate the median of all 9,994 individual transaction sales values (a typical single order of ~₹100–₹200). That is not what we want.

`MEDIANX` is an iterator — it loops over the table provided in the first argument, executes the expression in the second argument for each row, then calculates the median of those results.

Step-by-step:
1. `VALUES([Sub-Category])` creates a 17-row virtual table (one row per unique sub-category)
2. For each sub-category, `CALCULATE(SUM([Sales]))` computes total sales (e.g., Copiers = ₹1,49,528, Phones = ₹3,30,007, etc.)
3. `MEDIANX` finds the middle value of those 17 totals → ₹1,14,880

**Why median and not average (mean):**
Phones (₹3,30,007) and Chairs (₹3,28,449) are significantly higher than most sub-categories. Using the mean would pull the threshold upward, causing more products to appear as "low volume" than is representative. The median is robust to these outliers.

**Interviewer answer:** "The X-axis threshold is data-driven using MEDIANX — it calculates the median total sales across all 17 sub-categories, giving ₹1,14,880. This is resistant to outliers, dynamically updates if the dataset changes, and can be explained with a clear statistical rationale."

**Used in:** BCG Matrix X-axis constant line (Page 4). The Y-axis constant line is hardcoded to 0% (break-even) — a universal business standard, not a guess.

---

### 5. Unique Customers

**Formula:**
```dax
Unique Customers =
DISTINCTCOUNT('Superstore_Sales_Cleaned'[Customer ID])
```

**Purpose:** Count of unique customers in the current filter context.

**Result (unfiltered):** 793 unique customers

**Why DISTINCTCOUNT not COUNT:**
`COUNT([Customer ID])` would return 9,994 (one per row, including duplicates for repeat customers).
`DISTINCTCOUNT([Customer ID])` counts each Customer ID only once regardless of how many orders they placed.

**Business insight it enables:** 9,994 orders ÷ 793 unique customers = avg 12.6 orders per customer — a strong retention indicator. Visible immediately as a KPI card on Page 1.

**Used in:** KPI card "Unique Cx" (Page 1).

---

## Calculated Columns
*Calculated columns are computed once per row when the data loads and stored in the data model. They do not change with filter context — they are fixed values on each row.*

---

### 6. Orders Per Customer

**Formula:**
```dax
Orders Per Customer =
CALCULATE(
    COUNT('Superstore_Sales_Cleaned'[Row ID]),
    ALLEXCEPT(
        'Superstore_Sales_Cleaned',
        'Superstore_Sales_Cleaned'[Customer ID]
    )
)
```

**Purpose:** For every row in the dataset, shows the total lifetime order count for that row's customer.

**How it works:**
- `CALCULATE(...)` modifies the filter context for the expression inside it
- `COUNT([Row ID])` counts rows (orders) in the modified context
- `ALLEXCEPT(table, [Customer ID])` removes ALL filters from the table except the Customer ID filter — this forces the count to span all years, regions, categories, and ship modes for that specific customer

**Example:** Every row where Customer ID = "SM-20320" (Sean Miller) shows 15, because Sean Miller placed 15 total orders across the dataset.

**Why a calculated column and not a measure:** We need this value available row-by-row so it can be used as the input for the `Purchase Frequency Band` calculated column (below). Measures don't work as inputs to other calculated columns.

**Used in:** Input for Purchase Frequency Band column. Also available in customer tables for direct analysis.

---

### 7. Purchase Frequency Band

**Formula:**
```dax
Purchase Frequency Band =
SWITCH(
    TRUE(),
    'Superstore_Sales_Cleaned'[Orders Per Customer] = 1,    "1 order",
    'Superstore_Sales_Cleaned'[Orders Per Customer] <= 3,   "2-3 orders",
    'Superstore_Sales_Cleaned'[Orders Per Customer] <= 5,   "4-5 orders",
    'Superstore_Sales_Cleaned'[Orders Per Customer] <= 10,  "6-10 orders",
    "11+ orders"
)
```

**Purpose:** Groups each row's customer into a loyalty band based on their total order count.

**How SWITCH(TRUE(), ...) works:**
- `SWITCH(TRUE(), condition1, result1, condition2, result2, ..., default)` evaluates each condition in order
- Returns the result for the **first condition that equals TRUE**
- If no condition matches, returns the default value ("11+ orders")
- Conditions are checked sequentially: = 1, then ≤ 3, then ≤ 5, then ≤ 10, then default

**Example walkthrough for a customer with 7 orders:**
1. Is 7 = 1? → FALSE, skip
2. Is 7 ≤ 3? → FALSE, skip
3. Is 7 ≤ 5? → FALSE, skip
4. Is 7 ≤ 10? → TRUE → result: "6-10 orders"

**Result distribution across 9,994 rows:**
| Band | Row Count | Interpretation |
|------|-----------|----------------|
| 11+ orders | 7,655 | Highly loyal customers |
| 6-10 orders | 2,028 | Regular customers |
| 4-5 orders | 232 | Occasional customers |
| 2-3 orders | 74 | Infrequent customers |
| 1 order | 5 | One-time buyers |

**Business insight:** 76.6% of all order rows belong to customers with 11+ orders — an exceptionally high loyalty rate. Only 5 rows represent one-time buyers, suggesting the business has strong repeat purchase behaviour.

**Used in:** Customer Purchase Frequency Distribution chart (Page 4).

---

## Delta Column

### In Excel (Profitability Analysis Tab)

**Location:** Column F in the Profitability Analysis tab, adjacent to the pivot table

**Formula:** `=D4-E4` (Overall Margin % minus Average Profit Margin %)

**Why not a Pivot Table Calculated Field:**
Excel calculated fields operate on aggregated values, not on the displayed percentage values in each cell. For percentage subtraction, a calculated field would compute `(Sum of Profit / Sum of Sales) - (Sum of Profit Margin %)` — which does not give the same result as subtracting the two displayed percentages row by row. A direct cell reference formula `=D4-E4` reads the formatted values exactly as displayed and subtracts them correctly.

**Why not a Power BI DAX measure (for the Excel column):**
The Delta analysis was first developed during Excel EDA (Exploratory Data Analysis). The Excel formula approach is simpler, transparent, and gives accurate row-level results without any aggregation ambiguity.

### In Power BI (Sub-Category Performance Metrics Table, Page 2)

**Method:** New Visual Calculation → Custom (under the Calculations tab in the Home ribbon)

**Formula:**
```
[Overall Margin %] - [Avg Margin %]
```

This is a **Visual Calculation** — a Power BI feature that computes values within the context of a specific visual's rows, similar to how Excel formulas reference adjacent cells. It does not appear in the Data pane or DAX folder because it is scoped to the visual, not the data model.

**Interpretation Guide:**

| Delta Value | Meaning | Example |
|-------------|---------|---------|
| Large positive (+) | Large orders are significantly more profitable than small orders | Copiers: +5.48% — bulk/corporate Copier orders command better pricing |
| Large negative (-) | Large orders are more heavily discounted than small orders, dragging overall margin below the per-order average | Supplies: -13.75% — large Supplies orders receive disproportionate discounts |
| Near zero | Consistent margins regardless of order size | Envelopes: -0.05% — margin is stable across all transaction sizes |
| Special case — Binders: +34.82% | Overall margin appears positive (14.86%) but avg margin is deeply negative (-19.96%) | The few large Binder orders happen to be profitable, but 1,186 individual transactions lose money — the Delta reveals this contradiction |

**Grand Total Delta:** 0.44% — confirming that across all 9,994 transactions, high-value orders in this dataset carry marginally better margins than the simple transaction average.
