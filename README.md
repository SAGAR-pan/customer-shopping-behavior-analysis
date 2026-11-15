# Customer Shopping Behavior Analysis

**End-to-end retail analytics using Python, PostgreSQL, SQL, and Power BI.**

An analysis of 3,900 synthetic retail-customer records to understand revenue, product demand, discount usage, subscriptions, and customer loyalty — with interactive dashboards and business recommendations.

> **Project note:** This repository is an enhanced adaptation of an existing tutorial-based project. The original dataset and tutorial are credited in [Credits & Acknowledgements](#credits--acknowledgements). The additional dashboard page, measures, validation, and documentation updates are identified as project enhancements.

---

## Dashboard Preview

The Power BI report contains two pages:

1. **Customer Behavior Dashboard** — overall KPIs, subscription share, revenue and customer counts by category and age group, with slicers for subscription status, gender, category, and shipping type.
2. **Customer Insights** — customer segmentation, top five products by discount rate, subscriber comparisons, and business recommendations.

Open `powerbi/Customer_Intelligence_Analytics.pbix` in [Power BI Desktop](https://www.microsoft.com/en-us/power-platform/products/power-bi/desktop) to explore the report. The PBIX file requires a local PostgreSQL connection; see [How to Reproduce](#how-to-reproduce).

To add preview images to this README, place screenshots in `assets/` (for example, `dashboard_overview.png` and `dashboard_insights.png`) and uncomment/update the image links below:

![Customer Behavior Dashboard](assets/Dashboard%20Overview.png)

![Customer Insights Dashboard](assets/Dashboard%20Insights.png)
---

## Business Problem

A retail business wants to understand how customers shop and where it should focus marketing, merchandising, and retention efforts.

The analysis addresses ten business questions:

1. How does total revenue compare by gender?
2. Which customers used a discount and spent above the average purchase amount?
3. Which products have the highest average review ratings?
4. How do average purchase amounts compare between Standard and Express shipping?
5. How do subscriber and non-subscriber customer counts, average spend, and total revenue compare?
6. Which products have the highest percentage of purchases with discounts?
7. How many customers fall into New, Returning, and Loyal segments?
8. What are the top three purchased products within each category?
9. What share of repeat buyers are subscribers?
10. How is revenue distributed across age groups?

## Dataset and Methodology

- **Dataset:** Customer Shopping Trends Dataset by Sourav Banerjee, sourced from Kaggle.
- **Size:** 3,900 rows and 18 original columns.
- **Coverage:** customer demographics, purchase details, product categories, discounts, subscription status, shipping, payment methods, and purchase frequency.
- **Data type:** synthetic customer-level snapshot data; each row represents a customer record, not a timestamped transaction.

### Data preparation

The Python notebook uses pandas to:

- Impute 37 missing review-rating values using the median rating for the corresponding product category.
- Standardize column names to `snake_case`.
- Create `age_group` categories:
  - Young Adult: 18–31
  - Adult: 32–44
  - Middle-aged: 45–57
  - Senior: 58–70
- Convert stated purchase frequency into an approximate `purchase_frequency_days` value.
- Drop `promo_code_used` after checking that it duplicates `discount_applied`.

The cleaned dataset contains 19 columns after feature engineering and removal of the redundant field.

### Limitations

The dataset is synthetic and does not contain transaction dates. Results demonstrate an analytical workflow and support hypotheses to test on real transactional data; they should not be interpreted as market-wide facts. Because there are no dates, time-series or month-over-month trend claims are out of scope.

---

## Key Findings

The following values were checked against the local PostgreSQL `customer` table during the project workflow.

| # | Finding | Result |
|---|---|---|
| 1 | Overall dataset size and value | 3,900 customer records; total purchase amount $233,081; average purchase amount $59.76; average review rating 3.75/5 |
| 2 | Revenue by gender | Male: $157,890; Female: $75,191. Average spend per customer is close: $59.54 and $60.25 respectively |
| 3 | Revenue by age group | Young Adult: $62,143; Middle-aged: $59,197; Adult: $55,978; Senior: $55,763 |
| 4 | Subscription economics | Subscribers: 1,053 records, $59.49 average purchase, $62,645 total revenue. Non-subscribers: 2,847 records, $59.87 average purchase, $170,436 total revenue |
| 5 | Discount usage | 839 discounted purchases were above the overall average purchase amount in the executed query |
| 6 | Products with highest discount rates | Hat: 50.00%; Sneakers: 49.66%; Coat: 49.07%; Sweater: 48.17%; Pants: 47.37% |
| 7 | Customer segments | Loyal: 3,116 (79.9%); Returning: 701 (18.0%); New: 83 (2.1%) |
| 8 | Repeat buyers and subscriptions | Among records with more than five previous purchases: 2,518 non-subscribers (72.44%) and 958 subscribers (27.56%) |
| 9 | Shipping comparison | Express average purchase: $60.48; Standard: $58.46 |
| 10 | Highest-rated products | Gloves: 3.86; Sandals: 3.84; Boots: 3.82. The fifth-place rating is tied/ordering-sensitive at 3.78; verify the tied products with a tie-aware query before publishing a definitive top-five list |

### Interpretation

- Clothing was the largest revenue category in the original analysis, contributing $104,264 (44.7%) of total revenue.
- The gender revenue gap is largely explained by the different customer counts; average spend per customer is similar.
- Subscribers do not have a higher average purchase amount in this dataset. The subscriber programme may need to be evaluated on retention and purchase frequency rather than basket value alone.
- Hats, Sneakers, Coats, Sweaters, and Pants have the highest discount attachment rates in the executed analysis. This identifies products for further promotion and margin review; discount rate alone does not establish profitability.
- The customer base is heavily weighted toward the Loyal segment under the project's chosen definition, while the New segment is small.
- Express-shipping purchases have a higher average purchase amount than Standard-shipping purchases in this dataset. This is an association, not evidence that shipping choice causes higher spending.

---

## Project Enhancements

In addition to reproducing the base analysis, the local implementation includes the following enhancements:

### 1. Validated local data pipeline

The notebook cleans the CSV and loads the resulting DataFrame into PostgreSQL database `customer_behavior`, table `customer`. A verification query returned:

- Total rows: 3,900
- Average purchase amount: $59.76
- Average review rating: 3.75
- Total purchase amount: $233,081

### 2. Additional Power BI page: Customer Insights

The second report page adds:

- Customer segmentation calculated column using DAX.
- KPI measures for total customers, loyal customers, returning customers, and loyal-customer share.
- Top-five products by discount rate.
- Customer-count comparison by subscription status.
- Average purchase amount by subscription status.
- Total revenue by subscription status.
- A business-insights and recommendations text panel.

### 3. Explicit calculations

Examples of the added DAX logic:

```dax
Customer Segment =
VAR Purchases = 'public customer'[previous_purchases]
RETURN
    SWITCH(
        TRUE(),
        ISBLANK(Purchases), "Unknown",
        Purchases <= 1, "New",
        Purchases <= 10, "Returning",
        "Loyal"
    )
```

```dax
Discount Rate =
DIVIDE(
    CALCULATE(
        COUNTROWS('public customer'),
        'public customer'[discount_applied] = "Yes"
    ),
    COUNTROWS('public customer'),
    0
)
```

```dax
Loyal Customer Share =
DIVIDE(
    [Loyal Customers],
    [Customer Count],
    0
)
```

The segmentation thresholds are project-specific definitions: 0–1 previous purchases = New, 2–10 = Returning, and more than 10 = Loyal. The source SQL and DAX should use the same thresholds when results are compared.

---

## Technology Stack

| Layer | Technology | Use |
|---|---|---|
| Data preparation | Python, pandas | Cleaning, exploration, feature engineering |
| Database | PostgreSQL | Store the cleaned customer table |
| Python database access | SQLAlchemy, psycopg2 | Load data from Python into PostgreSQL |
| Analysis | SQL | Aggregations, subqueries, CTEs, CASE expressions, window functions |
| Visualization | Power BI Desktop | Two-page interactive dashboard |
| Measures and calculated columns | DAX | Customer KPIs, segmentation, discount rate |
| Documentation | PDF, PowerPoint, Markdown | Report findings and communicate recommendations |

## Repository Structure

The repository should contain the following files. Keep the original files as well as the enhanced deliverables where appropriate.

```text
Customer Shopping Behavior Analysis/
├── README.md
├── LICENSE
├── requirements.txt
├── .gitignore
├── assets/
│   ├── dashboard_screenshot.jpg
│   ├── dashboard_overview.png              # add final screenshot
│   └── dashboard_insights.png              # add final screenshot
├── data/
│   └── customer_shopping_behavior.csv
├── notebooks/
│   └── customer_shopping_behavior_analysis.ipynb
├── sql/
│   └── customer_behavior_sql_queries.sql
├── powerbi/
│   ├── customer_behavior_dashboard.pbix     # original report copy
│   └── Customer_Intelligence_Analytics.pbix # enhanced two-page report
└── report/
    ├── Customer_Shopping_Behavior_Report.pdf
    ├── Customer_Shopping_Behavior_Report_Sagar_BS_October_2026.pdf
    ├── Customer_Shopping_Behavior_Presentation.pptx
    └── Customer_Shopping_Behavior_Presentation_Enhanced.pptx
```

If the original or enhanced filenames differ in your local folder, update this tree to match the actual filenames before committing.

---

## How to Reproduce

### Prerequisites

- Python 3.10 or later
- PostgreSQL installed and running
- Power BI Desktop
- The project CSV dataset

### 1. Clone the repository and install dependencies

```bash
git clone <your-repository-url>
cd <repository-folder>

python -m venv .venv
```

Activate the environment:

**Windows PowerShell**
```powershell
.venv\Scripts\Activate.ps1
```

**Windows Command Prompt**
```cmd
.venv\Scripts\activate.bat
```

**macOS/Linux**
```bash
source .venv/bin/activate
```

Install the Python dependencies:

```bash
python -m pip install -r requirements.txt
python -m pip install ipykernel
```

### 2. Create the PostgreSQL database

In pgAdmin, create a database named:

```text
customer_behavior
```

The notebook connects with the PostgreSQL user `postgres` on `localhost:5432`. It prompts for the password using Python's `getpass`; the password should not be hardcoded into the notebook or committed to Git.

### 3. Run the notebook

Open:

```text
notebooks/customer_shopping_behavior_analysis.ipynb
```

Select the project's `.venv` Python interpreter as the Jupyter kernel, then run the notebook cells in order. The notebook cleans the CSV and loads the `customer` table into the `customer_behavior` database.

### 4. Run the SQL analysis

Open `sql/customer_behavior_sql_queries.sql` in pgAdmin Query Tool connected to `customer_behavior`, then execute the queries. Compare the results with the findings in this README and the report.

### 5. Open and refresh Power BI

Open `powerbi/Customer_Intelligence_Analytics.pbix` in Power BI Desktop.

In **Transform data → Data source settings**, check the PostgreSQL connection and credentials. Use the local database:

- Server: `localhost:5432`
- Database: `customer_behavior`
- Schema: `public`
- Table: `customer`
- Username: `postgres`

Refresh the report after confirming the source and credentials. Do not commit database passwords or other credentials.

### Sanity checks

Run this query in PostgreSQL:

```sql
SELECT
    COUNT(*) AS total_customers,
    ROUND(AVG(purchase_amount), 2) AS avg_purchase,
    ROUND(AVG(review_rating::numeric), 2) AS avg_rating,
    SUM(purchase_amount) AS total_revenue
FROM customer;
```

Expected result:

| Total customers | Average purchase | Average rating | Total purchase amount |
|---:|---:|---:|---:|
| 3,900 | 59.76 | 3.75 | 233,081 |

---

## Credits & Acknowledgements

- **Dataset:** [Customer Shopping Trends Dataset](https://www.kaggle.com/datasets/iamsouravbanerjee/customer-shopping-trends-dataset) by Sourav Banerjee (Kaggle). The dataset contains synthetic retail-customer records.
- **Tutorial and base project:** The base project structure, cleaning workflow, SQL questions, and original dashboard layout are adapted from [Amlan Mohanty's end-to-end tutorial](https://youtu.be/5PrZvPeUw60) and its [GitHub repository](https://github.com/amlanmohanty1/customer-trends-data-analysis-SQL-Python-PowerBI), released under the MIT License. The original copyright and license notices are retained.
- **Enhancements documented here:** local environment and database setup, execution and verification of the analysis, the additional Customer Insights Power BI page, DAX segmentation and KPI measures, discount-rate visualization, subscription comparisons, and updated project documentation.

## License

This repository is released under the MIT License in `LICENSE`. Components adapted from the tutorial remain subject to the original license and attribution requirements.
