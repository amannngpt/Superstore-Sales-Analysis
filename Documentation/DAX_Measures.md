# DAX Measures & Calculated Columns Documentation

## Measures (Calculated at report filter context level)

### 1. Overall Margin %
**Formula:**
DIVIDE(SUM('Superstore_Sales_Cleaned'[Profit]), 
       SUM('Superstore_Sales_Cleaned'[Sales]))

**Purpose:** Revenue-weighted overall business profit margin.
**Result:** 12.47%
**Why DIVIDE:** Handles division by zero gracefully 
(returns BLANK instead of error).
**Difference from Avg Profit Margin %:** This weights by revenue, 
giving larger orders more influence. A ₹1,000 order with 15% margin 
has more impact than a ₹50 order with 40% margin.

---

### 2. Avg Order Value
**Formula:**
DIVIDE(SUM('Superstore_Sales_Cleaned'[Sales]), 
       COUNT('Superstore_Sales_Cleaned'[Row ID]))

**Purpose:** Average revenue per transaction.
**Result (overall):** ₹230 per order
**Use case:** Appears in Top 10 Customers table to compare 
spending behavior across customers.

---

### 3. Median Sales by SubCategory
**Formula:**
MEDIANX(
    VALUES('Superstore_Sales_Cleaned'[Sub-Category]),
    CALCULATE(SUM('Superstore_Sales_Cleaned'[Sales]))
)

**Purpose:** Dynamic X-axis threshold for BCG Matrix scatter chart.
**Result:** ₹1,14,880
**Why MEDIANX not MEDIAN:** MEDIAN calculates median of individual 
transactions. MEDIANX iterates over each sub-category, calculates 
total sales per sub-category, then finds the median of those 17 totals.
**Why median not mean:** Median is robust to outliers — Phones 
(₹3,30,007) and Chairs (₹3,28,449) are significantly higher than 
most sub-categories, which would skew the mean upward.
**Interviewer answer:** "The threshold is data-driven using MEDIANX, 
splitting 17 sub-categories at their median sales value 
(₹1,14,880). This is robust to outliers unlike a simple average."

---

### 4. Unique Customers
**Formula:**
DISTINCTCOUNT('Superstore_Sales_Cleaned'[Customer ID])

**Purpose:** Count of unique customers in the dataset.
**Result:** 793 unique customers
**Why DISTINCTCOUNT not COUNT:** COUNT would count 9,994 
(one per row). DISTINCTCOUNT counts each Customer ID only once, 
regardless of how many orders they placed.

---

### 5. Avg Profit Value
**Formula:**
DIVIDE(SUM('Superstore_Sales_Cleaned'[Profit]), 
       COUNT('Superstore_Sales_Cleaned'[Row ID]))

**Purpose:** Average profit per transaction.
**Use case:** Appears in Additional Tables to compare profitability 
per order across top customers.

---

## Calculated Columns (Computed row by row, stored in data model)

### 6. Orders Per Customer
**Formula:**
CALCULATE(
    COUNT('Superstore_Sales_Cleaned'[Row ID]),
    ALLEXCEPT('Superstore_Sales_Cleaned', 
              'Superstore_Sales_Cleaned'[Customer ID])
)

**Purpose:** For every row, shows the total number of orders 
that customer has ever placed.
**Why ALLEXCEPT:** Removes all filters EXCEPT Customer ID, so 
the COUNT spans all years, regions, and categories for that customer.
**Example:** Every row for "Sean Miller" shows 15 
(his total lifetime orders).

---

### 7. Purchase Frequency Band
**Formula:**
SWITCH(
    TRUE(),
    'Superstore_Sales_Cleaned'[Orders Per Customer] = 1, "1 order",
    'Superstore_Sales_Cleaned'[Orders Per Customer] <= 3, "2-3 orders",
    'Superstore_Sales_Cleaned'[Orders Per Customer] <= 5, "4-5 orders",
    'Superstore_Sales_Cleaned'[Orders Per Customer] <= 10, "6-10 orders",
    "11+ orders"
)

**Purpose:** Groups customers into loyalty segments based on 
order frequency.
**How SWITCH works:** Evaluates conditions in order, returns 
result for first TRUE condition. Default (no match) returns "11+ orders".
**Result distribution:**
- 11+ orders: 7,655 rows
- 6-10 orders: 2,028 rows
- 4-5 orders: 232 rows
- 2-3 orders: 74 rows
- 1 order: 5 rows

---

## Delta Column (Excel Formula — NOT a DAX measure)

**Location:** Profitability Analysis tab, Column F
**Formula:** =D4-E4 (Overall Margin % minus Average Profit Margin %)
**Why not a pivot calculated field:** Excel calculated fields 
aggregate incorrectly for percentage subtraction — they calculate 
(Sum of D / Sum of E) rather than (D% - E%) at the row level.
**Why not a Power BI measure:** Delta was first identified 
during Excel EDA. The Visual Calculation in Power BI's 
Sub-Category Performance Metrics table replicates this.

**Interpretation guide:**
- Large positive delta: Sub-category's high-value orders are 
  significantly more profitable than average orders
- Large negative delta: Sub-category's high-value orders are 
  being heavily discounted, dragging overall margin DOWN
- Near-zero delta: Consistent margins regardless of order size

**Notable findings:**
- Supplies: -13.75% delta → large Supplies orders are 
  disproportionately discounted
- Copiers: +5.48% delta → large Copier orders are premium-priced
- Binders: +34.82% delta → the few large Binder orders are profitable 
  but 1,186 individual transactions lose money

**PowerBI:**
- Same as Excel
- Used New Visual Calculation > Custom in the Calculations tab under the Home Ribbon 
