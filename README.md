# 🛒 Walmart Sales Analysis
### End-to-End Data Analytics Project | Python · SQL · Power BI

<div align="center">

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![PowerBI](https://img.shields.io/badge/Power_BI-Dashboard-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![Pandas](https://img.shields.io/badge/Pandas-Data_Analysis-150458?style=for-the-badge&logo=pandas&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-2ea44f?style=for-the-badge)

</div>

---

<div align="center">
  <img src="Image/Dashboard 1.png" alt="Walmart Executive Sales Overview" width="900">
</div>

---

## 📌 Repository Name
```
Walmart-sales-analysis
```

## 📝 Repository Description
```
End-to-end Walmart sales analysis | Python EDA → MySQL KPI Queries → Power BI Dashboards | Seasonality, Holiday Impact & Store Risk Analysis
```

---

## 🧠 Business Problem

> **Walmart operates 45 stores across different regions.**
> **What drives sales performance — and what is the real impact of holidays, seasonality, and economic conditions like unemployment and inflation?**

As a retail chain, Walmart faces four critical business questions every quarter:

| # | Business Question | Why It Matters |
|---|---|---|
| 1 | When do sales peak and when do they slow down? | Drives inventory ordering schedules |
| 2 | Do holidays genuinely boost sales — and by how much? | Decides staffing levels and stock volumes |
| 3 | Which stores are top performers and which carry risk? | Guides resource allocation decisions |
| 4 | How do unemployment and fuel prices affect revenue? | Helps predict and respond to economic downturns |

This project answers all four questions with data.

---

## 📊 Key Results at a Glance

| Metric | Result |
|---|---|
| 💰 Total Revenue (2010–2012) | **$6,737.22M** |
| 📅 Avg Weekly Sales per Store | **$1.05M** |
| 🎄 Overall Holiday Sales Lift | **+8%** |
| 🔥 Highest Holiday Month | **November → +42% lift** (Thanksgiving + Black Friday) |
| ⚠️ December Anomaly | **-29% lift** — customers front-load shopping in November |
| 🏆 Top Performing Store | **Store 20 → $301M** total revenue |
| 🔴 Highest Risk Stores | **Store 12 & 28** (13.12% unemployment rate) |
| 📉 Sharpest YoY Decline | **2012 → -12.3%** vs 2011 (unemployment-linked) |

---

## 🗂️ Project Structure

```
walmart-sales-analysis/
│
├── 📁 data/
│   ├── Walmart.csv                     ← Raw dataset (original, unmodified)
│   └── walmart_cleaned.csv             ← Cleaned & feature-engineered (Python output)
│
├── 📁 sql/
│   └── walmart_analysis.sql            ← All SQL queries + views
│
├── 📁 dashboards/
│   ├── Dashboard_1.png                 ← Executive Sales Overview
│   ├── Dashboard_2.png                 ← Seasonality & Holiday Analysis
│   └── Dashboard_3.png                 ← External Market & Economic Impact
│
├── walmart_sales_analysis.ipynb        ← Python EDA notebook
├── requirements.txt                    ← Python dependencies
└── README.md                           ← This file
```

---

## 🔧 Three-Layer Analytics Workflow

```
┌─────────────────────────────────────────────────────────┐
│  LAYER 1 — Python (Jupyter Notebook)                    │
│  Load → Validate → Clean → Feature Engineer → EDA       │
└────────────────────────┬────────────────────────────────┘
                         │  exports walmart_cleaned.csv
                         ▼
┌─────────────────────────────────────────────────────────┐
│  LAYER 2 — MySQL                                        │
│  Load → Validate → KPI Queries → Views for Power BI     │
└────────────────────────┬────────────────────────────────┘
                         │  connects via SQL Views
                         ▼
┌─────────────────────────────────────────────────────────┐
│  LAYER 3 — Power BI                                     │
│  3 Interactive Dashboards → Business Recommendations    │
└─────────────────────────────────────────────────────────┘
```

> **Why this three-layer flow:**
> Python handles messy raw data and deep exploration.
> SQL handles structured aggregations that refresh consistently.
> Power BI handles visual storytelling for non-technical stakeholders.
> Each tool is used for what it does best — which is how real analytics teams operate.

---

## 🔍 Project Walkthrough

---

### 🐍 Layer 1 — Python: Data Cleaning & EDA

#### Step 1 — Data Loading & Validation

```python
df = pd.read_csv(DATA_PATH)

df.head(10)            # Preview structure
df.shape               # (6435, 8) — rows and columns
df.isnull().sum()      # Check for missing values per column
df.duplicated().sum()  # Check for duplicate records
df.info()              # Data types and memory usage
df.describe()          # Statistical summary (min, max, mean, std)
```

> **Why this step exists:**
> Raw data is never trusted without inspection first.
> In this dataset the `Date` column was stored as a plain string —
> meaning Python could not sort, filter, or group it by time without conversion.
> Catching issues like this at the start prevents incorrect results from
> flowing into every chart and query built afterwards.

---

#### Step 2 — Data Cleaning

```python
# Convert Date from string to proper datetime
df['Date'] = pd.to_datetime(df['Date'], format='%d-%m-%Y', errors='coerce')

# Drop rows where critical fields are null
before_rows = len(df)
df = df.dropna(subset=["Date", "Weekly_Sales", "Store", "Holiday_Flag"])
after_rows = len(df)
print(f"Dropped {before_rows - after_rows} rows due to invalid critical values")

# Enforce business rules
df = df[df["Weekly_Sales"] > 0]           # No zero or negative sales
df = df[df["Holiday_Flag"].isin([0, 1])]  # Only valid flag values (0 or 1)

# Sort chronologically for time-based analysis
df = df.sort_values(by='Date').reset_index(drop=True)
```

> **Why convert Date:**
> When stored as a string like `"05-02-2010"`, Python treats it as plain text.
> Sorting it alphabetically gives wrong order — `"12-01-2010"` would appear before `"05-02-2010"`.
> Converting to `datetime` makes all chronological operations correct.
>
> **Why enforce business rules:**
> A `Weekly_Sales` value of 0 or negative makes no business sense.
> These rows would pull down averages and distort trend lines if kept in the analysis.

---

#### Step 3 — Feature Engineering

```python
df["Year"]       = df["Date"].dt.year
df["Month"]      = df["Date"].dt.month
df["Month_Name"] = df["Date"].dt.month_name()
df["Week"]       = df["Date"].dt.isocalendar().week.astype(int)
```

> **Why extract these columns:**
> A single `Date` column cannot be used in `GROUP BY Month` or `GROUP BY Year` in SQL.
> Extracting them as separate columns makes time-based grouping possible everywhere.
> Since these columns are included in the exported CSV, SQL and Power BI
> use the same pre-computed values — no repeated logic, no risk of inconsistency.

---

#### Step 4 — Exploratory Data Analysis

**Yearly Sales Trend**
```python
yearly_sales = df.groupby('Year')['Weekly_Sales'].sum()
# Result → 2010: $2.34B | 2011: $2.42B (+6.1%) | 2012: $1.98B (-12.3%)
```

**Monthly Seasonality Pattern**
```python
monthly_sales = df.groupby('Month')['Weekly_Sales'].mean()
# Peak months  → July ($1.03M), November ($1.14M), December ($1.28M)
# Slowest month → January ($923K)
```

**Sales Heatmap — Year × Month**
```python
pivot = df.pivot_table(values="Weekly_Sales", index="Year",
                       columns="Month", aggfunc="mean")
sns.heatmap(pivot, cmap="YlGnBu")
plt.title("Sales Heatmap (Year vs Month)")
```
> This heatmap is a cross-validation tool.
> If the same seasonal pattern appears across all three years,
> it is a real structural pattern — not a data error or outlier.

**Weekly Sales Trend**
```python
weekly_sales = df.groupby('Date')['Weekly_Sales'].sum()
plt.plot(weekly_sales)
plt.title("Overall Weekly Sales Trend")
```
> The spikes on this chart align exactly with Thanksgiving weeks —
> confirming that the `Holiday_Flag` column is correctly encoded in the dataset.

**Holiday vs Non-Holiday**
```python
df.groupby('Holiday_Flag')['Weekly_Sales'].mean()
# Holiday weeks  → $1,122,888 avg
# Regular weeks  → $1,041,256 avg
# Lift           → +8%
```

---

#### Step 5 — Export Cleaned Dataset

```python
df.to_csv(OUTPUT_PATH, index=False, date_format="%Y-%m-%d")
```

> **Why export once and share downstream:**
> Python produces one validated, cleaned file.
> SQL and Power BI both read from this same file.
> If each tool did its own cleaning independently, small logic differences
> would cause numbers to disagree between dashboards and queries —
> a common and avoidable problem. One source of truth solves it.

---

### 🗃️ Layer 2 — MySQL: KPI Analysis & Views

#### Database & Table Setup

```sql
CREATE DATABASE walmart_analysis;
USE walmart_analysis;

CREATE TABLE walmart_sales (
    Store        INT NOT NULL,
    Date         DATE NOT NULL,
    Weekly_Sales DECIMAL(12,2) NOT NULL,
    Holiday_Flag TINYINT NOT NULL,
    Temperature  FLOAT,
    Fuel_Price   FLOAT,
    CPI          FLOAT,
    Unemployment FLOAT,
    Year         INT,
    Month        INT,
    Month_Name   VARCHAR(15),
    Week         INT,
    PRIMARY KEY (Store, Date)
);
```

> **Why PRIMARY KEY (Store, Date):**
> Each row represents one store's sales for one specific week.
> Store 1 cannot have two entries for the same date — it is physically impossible.
> The composite primary key enforces this uniqueness rule at the database level
> and prevents duplicate data from corrupting any aggregation.

---

#### Data Validation in SQL

```sql
-- Confirm row count and date range loaded correctly
SELECT COUNT(*) AS total_rows, MIN(Date) AS start_date, MAX(Date) AS end_date
FROM walmart_sales;

-- Confirm Holiday_Flag only contains 0 or 1
SELECT Holiday_Flag, COUNT(*) AS records
FROM walmart_sales GROUP BY Holiday_Flag;

-- Confirm no zero or negative sales survived the cleaning step
SELECT * FROM walmart_sales WHERE Weekly_Sales <= 0;
```

> **Why validate in SQL after Python already cleaned the data:**
> The CSV export and `LOAD DATA INFILE` step can introduce problems —
> encoding issues, truncated rows, or wrong delimiters.
> SQL validation is a second checkpoint.
> This is called **end-to-end data integrity verification** and is
> standard practice in any professional analytics workflow.

---

#### Core Business Queries

| Query | SQL Technique | Business Purpose |
|---|---|---|
| Yearly Sales Trend | `GROUP BY Year` | YoY performance comparison |
| Monthly Seasonality | `GROUP BY Month` | Seasonal pattern identification |
| Holiday vs Regular | `AVG` split by `Holiday_Flag` | Holiday impact baseline |
| Holiday Lift % | Conditional `AVG` + `NULLIF` | Lift % with zero-division protection |
| Top 10 Stores | `ORDER BY SUM DESC LIMIT 10` | Revenue leader identification |
| Store Rankings | `RANK() OVER (ORDER BY SUM DESC)` | Full 45-store performance ranking |
| Store-Wise Holiday Lift | Per-store conditional `AVG` + `HAVING` | Which stores benefit most from holidays |
| YoY Growth | `LAG()` window function + CTE | Year-over-year growth rate |
| Store Risk Scoring | `CASE WHEN` + `STDDEV` | Composite economic risk categorization |
| Seasonality Index | `CROSS JOIN` single-row CTE | Above/below-average month labeling |
| Holiday Lift by Month | Self-join on CTE | November vs December comparison |

---

#### Key SQL Patterns Explained

**Year-over-Year Growth using LAG()**
```sql
WITH yearly AS (
    SELECT Year, SUM(Weekly_Sales) AS total_sales
    FROM walmart_sales GROUP BY Year
)
SELECT Year, total_sales,
    LAG(total_sales) OVER (ORDER BY Year) AS prev_year_sales,
    ROUND(((total_sales - LAG(total_sales) OVER (ORDER BY Year)) /
            LAG(total_sales) OVER (ORDER BY Year)) * 100, 2) AS growth_pct
FROM yearly;
```
> `LAG()` fetches the value from the previous row in the result.
> Ordered by Year, it returns last year's total — so growth percentage
> is calculated in one pass without joining the table to itself.
> **Result: 2010→2011: +6.1% | 2011→2012: -12.3%**

---

**Holiday Lift % using NULLIF**
```sql
SELECT ROUND(
    ((AVG(CASE WHEN Holiday_Flag = 1 THEN Weekly_Sales END) -
      AVG(CASE WHEN Holiday_Flag = 0 THEN Weekly_Sales END)) /
     NULLIF(AVG(CASE WHEN Holiday_Flag = 0 THEN Weekly_Sales END), 0)
    ) * 100, 2) AS holiday_lift_pct
FROM walmart_sales;
```
> `NULLIF(expression, 0)` returns NULL instead of 0 when the denominator is zero.
> This prevents a division-by-zero error that would crash the query
> if non-holiday data were ever missing from the table.
> **Result: +8% overall holiday lift**

---

**Holiday Lift by Month — Self Join on CTE**
```sql
WITH monthly_sales AS (
    SELECT Month, Holiday_Flag, AVG(Weekly_Sales) AS avg_sales
    FROM walmart_sales GROUP BY Month, Holiday_Flag
)
SELECT h.Month,
    ROUND(h.avg_sales, 0) AS holiday_avg,
    ROUND(r.avg_sales, 0) AS regular_avg,
    ROUND(((h.avg_sales - r.avg_sales) / r.avg_sales) * 100, 1) AS lift_pct
FROM monthly_sales h
JOIN monthly_sales r ON h.Month = r.Month
WHERE h.Holiday_Flag = 1 AND r.Holiday_Flag = 0
ORDER BY lift_pct DESC;
```
> The CTE contains both holiday rows and regular rows.
> Joining it to itself — alias `h` for holiday, alias `r` for regular —
> with `ON h.Month = r.Month` ensures November holidays are compared
> only against November regular weeks, not the overall average.
> **Result: November +42% | December -29%**

---

**Store Risk Scoring**
```sql
WITH store_metrics AS (
    SELECT Store,
        SUM(Weekly_Sales)    AS total_sales,
        AVG(Unemployment)    AS avg_unemployment,
        STDDEV(Weekly_Sales) AS volatility
    FROM walmart_sales GROUP BY Store
)
SELECT Store,
    ROUND(total_sales, 0)      AS total_sales,
    ROUND(avg_unemployment, 2) AS avg_unemployment,
    ROUND(volatility, 0)       AS volatility,
    CASE
        WHEN avg_unemployment >= 10 AND total_sales < 15000000 THEN 'Critical Risk'
        WHEN avg_unemployment >= 9  OR  total_sales < 10000000 THEN 'High Risk'
        WHEN avg_unemployment >= 7.5                           THEN 'Medium Risk'
        ELSE 'Low Risk'
    END AS risk_category
FROM store_metrics
ORDER BY
    CASE WHEN risk_category = 'Critical Risk' THEN 1
         WHEN risk_category = 'High Risk'     THEN 2
         ELSE 3 END, total_sales;
```
> `STDDEV` measures how much a store's sales swing week to week.
> High standard deviation means one week it sells a lot, the next very little —
> a sign of supply chain, staffing, or local economic instability.
> Combining volatility + unemployment + revenue threshold produces
> a single prioritized intervention list for management.

---

**Store-Wise Holiday Lift with HAVING**
```sql
SELECT Store,
    ROUND(((AVG(CASE WHEN Holiday_Flag = 1 THEN Weekly_Sales END) -
             AVG(CASE WHEN Holiday_Flag = 0 THEN Weekly_Sales END)) /
             AVG(CASE WHEN Holiday_Flag = 0 THEN Weekly_Sales END)) * 100, 2)
    AS holiday_lift_pct
FROM walmart_sales
GROUP BY Store
HAVING
    COUNT(CASE WHEN Holiday_Flag = 1 THEN 1 END) >= 2
AND COUNT(CASE WHEN Holiday_Flag = 0 THEN 1 END) >= 2
ORDER BY holiday_lift_pct DESC;
```
> `WHERE` filters rows before grouping.
> `HAVING` filters after grouping — allowing aggregate conditions.
> Without this guard, a store with only 1 holiday week would produce
> a misleading average based on a single data point.
> Requiring at least 2 holiday weeks and 2 regular weeks ensures reliable results.

---

**Seasonality Index using CROSS JOIN**
```sql
WITH monthly AS (
    SELECT Month, AVG(Weekly_Sales) AS avg_sales
    FROM walmart_sales GROUP BY Month
),
overall AS (SELECT AVG(Weekly_Sales) AS overall_avg FROM walmart_sales)
SELECT m.Month,
    ROUND(m.avg_sales, 0)                          AS month_avg,
    ROUND(o.overall_avg, 0)                        AS annual_avg,
    ROUND((m.avg_sales / o.overall_avg) * 100, 1) AS seasonality_index,
    CASE
        WHEN (m.avg_sales / o.overall_avg) * 100 >= 110 THEN 'Peak Season'
        WHEN (m.avg_sales / o.overall_avg) * 100 <= 90  THEN 'Slow Season'
        ELSE 'Normal'
    END AS season_label
FROM monthly m CROSS JOIN overall o
ORDER BY m.Month;
```
> The `overall` CTE returns exactly one row — the grand average.
> `CROSS JOIN` multiplies every row in `monthly` (12 rows) with that single row,
> making the overall average available in every monthly row for division.
> Index > 100 → above average → increase inventory.
> Index < 100 → below average → reduce stock and plan clearance.

---

#### SQL Views for Power BI

```sql
CREATE OR REPLACE VIEW vw_yearly_sales AS
SELECT Year, SUM(Weekly_Sales) AS total_sales
FROM walmart_sales GROUP BY Year;

CREATE OR REPLACE VIEW vw_monthly_seasonality AS
SELECT Month, Month_Name, AVG(Weekly_Sales) AS avg_weekly_sales
FROM walmart_sales GROUP BY Month, Month_Name ORDER BY Month;

CREATE OR REPLACE VIEW vw_store_performance AS
SELECT Store, SUM(Weekly_Sales) AS total_sales,
       RANK() OVER (ORDER BY SUM(Weekly_Sales) DESC) AS store_rank
FROM walmart_sales GROUP BY Store;

CREATE VIEW vw_dim_date AS
SELECT DISTINCT Date, Year, Month, Month_Name, Week,
       QUARTER(Date) AS Quarter,
       CASE WHEN Holiday_Flag = 1 THEN 'Holiday' ELSE 'Regular' END AS Date_Type
FROM walmart_sales;
```

> **Why Views and not exported tables:**
> Views are virtual tables — they always reflect the current state of `walmart_sales`.
> When Power BI connects to a view, refreshing the dashboard pulls the latest data automatically.
> No manual re-exports needed.
> This is the standard production approach: one source of truth, no duplicated logic.

---

### 📊 Layer 3 — Power BI: Interactive Dashboards

---

#### Dashboard 1 — Walmart Executive Sales Overview

![Executive Dashboard](Image/Dashboard 1.png)


**KPI Cards:** Total Sales · Avg Weekly Sales · Avg Holiday Sales · Avg Non-Holiday Sales · Holiday Lift %

**Visuals:**
- Annual Sales Growth & Weekly Performance Trends (line chart)
- Revenue Contribution: Holiday vs Regular Weeks (donut → 92.5% Regular, 7.5% Holiday)
- Monthly Sales Seasonality: Total Revenue by Month (bar chart)
- Top 10 Stores by Total Revenue (horizontal bar)

**Slicers:** Year (2010 / 2011 / 2012) · Store

> **What this answers:**
> Which year was strongest? → 2011
> Which store leads? → Store 20 at $301M
> What share of revenue comes from holidays? → Only 7.5% — regular weeks drive the business

---

#### Dashboard 2 — Seasonality & Holiday Performance Analysis

![Seasonality Dashboard](Image/Dashboard 2.png)

**Visuals:**
- Matrix: Avg Weekly Sales by Month × Year (conditional formatting highlights peaks)
- Multi-year monthly trend: 2010 vs 2011 vs 2012 overlay (line chart)
- Total Sales Trend: Holiday vs Regular across all weeks
- Holiday Sales Impact by Month — Lift % bar chart:
  - 🟢 November: **+42%** (Thanksgiving + Black Friday)
  - 🟢 September: **+7%** (Labor Day)
  - 🟢 February: **+3%** (Super Bowl)
  - 🔴 December: **-29%** (post-Thanksgiving demand pull-forward)

**Slicers:** Year · Store · Date Type (Holiday / Regular)

> **What this answers:**
> Why does December underperform despite being Christmas season?
> Customers complete holiday shopping during Black Friday in November.
> This single insight prevents unnecessary and costly December overstocking.

---

#### Dashboard 3 — External Market & Economic Impact Analysis

![Economic Dashboard](Image/Dashboard 3.png)

**Visuals:**
- Scatter: Sales vs Unemployment (risk zone highlighted at unemployment > 8%)
- Scatter: Sales vs Regional Fuel Price Fluctuations
- Table: Store Economic Exposure Ranking (color-coded by unemployment level)
- Dual-axis chart: Total Sales Stability vs CPI (inflation) Trend

**Slicers:** Year · Store

> **What this answers:**
> Which stores are economically vulnerable?
> Stores 12, 28, and 38 operate in 13.12% unemployment zones — the highest risk cluster.
> These stores need cost controls and targeted promotions, or deeper intervention planning.

---

## 💡 Key Business Insights

```
🔴  CRITICAL  — November holiday lift = +42%
               Increase inventory 40% before October 15th every year

🔴  CRITICAL  — December = -29% lift vs November
               Do NOT replicate November stock levels for December
               Start clearance sales in the first week of December

🟡  WARNING   — Stores 12, 28, 38 in 13.12% unemployment zones
               High economic risk + low revenue = intervention needed now

🟡  WARNING   — 2012 showed -12.3% YoY decline
               Linked to rising unemployment — a leading indicator to monitor

🟢  STRENGTH  — Store 20 leads at $301M total revenue
               Document its practices and replicate to mid-tier stores

🟢  STRENGTH  — Sales are stable relative to CPI movement
               Walmart customers are largely inflation-resilient

🟢  STRENGTH  — July and April are strongest non-holiday months
               Good windows for mid-year promotional campaigns
```

---

## 🛠️ Tech Stack

| Tool | Version | Used For |
|---|---|---|
| Python | 3.10+ | Data cleaning, EDA, statistical testing, visualizations |
| Pandas | 1.5+ | Data manipulation and aggregation |
| NumPy | 1.23+ | Numerical operations |
| Matplotlib | 3.6+ | Base visualizations |
| Seaborn | 0.12+ | Statistical plots and heatmaps |
| SciPy | 1.9+ | T-test for holiday statistical significance |
| MySQL | 8.0 | KPI queries, window functions, views |
| Power BI Desktop | Latest | Interactive dashboards |

---

## 🎯 What This Project Demonstrates

| Skill | Evidence |
|---|---|
| Data Cleaning | Handled type errors, missing values, and business rule violations in Python |
| Feature Engineering | Extracted time columns used consistently across Python, SQL, and Power BI |
| Statistical Testing | T-test confirmed holiday lift is statistically significant (p < 0.001) |
| SQL Window Functions | `RANK()`, `LAG()`, `NTILE()`, `STDDEV()` across 11 business queries |
| SQL Query Design | CTEs, self-joins, CROSS JOINs, HAVING guards, NULLIF protection |
| Dashboard Design | 3 Power BI pages — KPI cards, slicers, scatter plots, matrix tables |
| Business Thinking | Every chart answers a specific business question with a clear recommendation |

---

## 🚀 How to Run This Project

**Step 1 — Clone the repository**
```bash
git clone https://github.com/Gudurupavankumarreddy/walmart-sales-analysis.git
cd walmart-sales-analysis
```

**Step 2 — Install Python dependencies**
```bash
pip install -r requirements.txt
```

**Step 3 — Download the dataset**

📥 [Walmart Store Sales — Kaggle](https://www.kaggle.com/datasets/yasserh/walmart-dataset)

Place it at: `data/Walmart.csv`

**Step 4 — Run the Python notebook**
```bash
jupyter notebook walmart_sales_analysis.ipynb
```
> Kernel → Restart & Run All
> Output: `data/walmart_cleaned.csv`

**Step 5 — Load into MySQL**
```sql
-- Open sql/walmart_analysis.sql in MySQL Workbench
-- Update the LOAD DATA INFILE path to match your system
-- Run the full SQL file top to bottom
```

**Step 6 — Open Power BI**
```
1. Open dashboards/walmart_dashboard.pbix
2. Home → Transform Data → Data Source Settings
3. Update MySQL connection or point to walmart_cleaned.csv
4. Click Refresh All
```

---

## 📦 Requirements

```txt
pandas>=1.5.0
numpy>=1.23.0
matplotlib>=3.6.0
seaborn>=0.12.0
scipy>=1.9.0
jupyter>=1.0.0
```

---

## 👤 Author

**G. Pavan Kumar Reddy**
Aspiring Data Analyst | Python · SQL · Power BI

<div align="center">

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/pavankumar0415/)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Gudurupavankumarreddy)
[![Email](https://img.shields.io/badge/Email-Contact-EA4335?style=for-the-badge&logo=gmail&logoColor=white)](mailto:gudurupavanpavankumarreddy@gmail.com)

</div>

---

<div align="center">

*⭐ If this project helped you, give it a star — it helps others find it too!*

</div>
