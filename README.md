# Retail Orders Data Analysis

## Project Overview
This project involves analyzing retail order data to extract valuable insights such as top revenue-generating products, top-selling products by region, month-over-month sales growth, and the highest-growing subcategories. The dataset is sourced from Kaggle and processed using Python and SQL.

## Dataset
- Source: Kaggle dataset ankitbansal06/retail-orders
- File: orders.csv
- Columns:
  - order_id, order_date, ship_mode, region, product_id, category, sub_category, list_price, cost_price, discount_percent, etc.

## Setup Instructions
### Prerequisites
Ensure the following dependencies are installed:
pip install pandas sqlalchemy pyodbc kaggle

### Steps to Run
1. Download the Dataset
      kaggle datasets download ankitbansal06/retail-orders -f orders.csv --path D:/Python/DataAnalyticsProject/Data
   
2. Load and Process Data
   - Read the dataset and handle missing values.
   - Standardize column names.
   - Compute additional metrics (discount, sale_price, profit).
   - Convert order_date to datetime format.

3. Database Setup
   - Ensure Microsoft ODBC Driver 17 for SQL Server is installed.
   - Modify the connection string in sqlalchemy.create_engine() with your SQL Server details.
   - Load the processed data into the SQL database.

## SQL Queries for Analysis
### 1. Find Top 10 Highest Revenue-Generating Products
SELECT TOP 10 product_id, SUM(sale_price) AS sales
FROM df_orders
GROUP BY product_id
ORDER BY sales DESC;

### 2. Find Top 5 Highest Selling Products in Each Region
WITH cte AS (
    SELECT region, product_id, SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY region, product_id
)
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY region ORDER BY sales DESC) AS rn
    FROM cte
) A
WHERE rn <= 5;

### 3. Month-over-Month Growth Comparison for 2022 and 2023 Sales
WITH cte AS (
    SELECT YEAR(order_date) AS order_year, MONTH(order_date) AS order_month,
           SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY YEAR(order_date), MONTH(order_date)
)
SELECT order_month,
       SUM(CASE WHEN order_year = 2022 THEN sales ELSE 0 END) AS sales_2022,
       SUM(CASE WHEN order_year = 2023 THEN sales ELSE 0 END) AS sales_2023
FROM cte
GROUP BY order_month
ORDER BY order_month;

### 4. Find the Month with the Highest Sales for Each Category
WITH cte AS (
    SELECT category, FORMAT(order_date, 'yyyyMM') AS order_year_month,
           SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY category, FORMAT(order_date, 'yyyyMM')
)
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY category ORDER BY sales DESC) AS rn
    FROM cte
) a
WHERE rn = 1;

### 5. Find the Subcategory with the Highest Growth by Profit in 2023 vs 2022
WITH cte AS (
    SELECT sub_category, YEAR(order_date) AS order_year, SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY sub_category, YEAR(order_date)
),
cte2 AS (
    SELECT sub_category,
           SUM(CASE WHEN order_year = 2022 THEN sales ELSE 0 END) AS sales_2022,
           SUM(CASE WHEN order_year = 2023 THEN sales ELSE 0 END) AS sales_2023
    FROM cte
    GROUP BY sub_category
)
SELECT TOP 1 *, (sales_2023 - sales_2022) AS growth
FROM cte2
ORDER BY growth DESC;

## Conclusion
This project provides insights into retail order data by analyzing revenue, sales trends, and growth patterns. By using Python and SQL, we efficiently clean, process, and analyze the data to derive meaningful business insights.

## Future Enhancements
- Implement data visualization using Matplotlib/Seaborn.
- Automate data ingestion and analysis using Airflow.
- Deploy insights as a Power BI dashboard.

## Author
Shantanu Suvarna