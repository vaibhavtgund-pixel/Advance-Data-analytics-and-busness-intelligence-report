# Advanced SQL Data Analytics & Business Intelligence Project

Exploratory and analytical SQL queries built on a retail sales data warehouse (`datawarehouseanalytics`), covering trend analysis, cumulative metrics, product performance, segmentation, and a reusable customer reporting view.

## Dataset
Star-schema data warehouse with:
- `fact_sales` — order-level sales transactions (order_date, sales_amount, quantity, customer_key, product_key)
- `dim_products` — product attributes (product_name, category, cost)
- `dim_customers` — customer attributes (name, birthdate, customer_number)

## What's inside

| # | Analysis | What it answers |
|---|----------|------------------|
| 1 | **Change Over Time** | Monthly sales, customer count, and quantity trends |
| 2 | **Cumulative Analysis** | Running total and moving average of sales by year — is the business growing? |
| 3 | **Performance Analysis** | Each product's sales vs. its own average and vs. the prior year (YoY), using `LAG()` |
| 4 | **Part-to-Whole Analysis** | Each product category's % contribution to total sales |
| 5 | **Data Segmentation** | Buckets products into cost ranges and counts products per range |
| 6 | **Customer Segmentation** | Classifies customers as VIP / Regular / New based on spend and lifespan |
| 7 | **Customer Report (View)** | `report_customers2` — a reusable view consolidating customer demographics, order metrics, segment, recency, average order value, and average monthly spend |

## Key SQL techniques used
- Window functions: `SUM() OVER()`, `AVG() OVER()`, `LAG()`
- CTEs (including multi-CTE chains) and subqueries
- Conditional aggregation with `CASE WHEN`
- Date functions: `TIMESTAMPDIFF`, `YEAR()`, `MONTH()`
- View creation for repeatable reporting

## How to use
1. Load the `datawarehouseanalytics` schema (fact/dim tables above) into MySQL.
2. Run `advanced_sql_analytics.sql` top to bottom, or execute individual sections independently — each analysis is self-contained.
3. The final section creates a view (`report_customers2`) that can be queried directly for customer-level reporting.

## Tech
MySQL

---
*Part of a portfolio of data analytics projects — see also: Power BI Data Architecture, S&OP Analytics Dashboard.*
