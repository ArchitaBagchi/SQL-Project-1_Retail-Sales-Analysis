# SQL-Project-1_Retail-Sales-Analysis

## Project Title: Retail Sales Analysis

**Skill Level:** Beginner

**Database Environment:** PostgreSQL 

**Project Description:** This project serves as a comprehensive introduction to SQL-driven data analytics within the retail sector. Designed for aspiring data analysts, the project walks through the entire lifecycle of data analysis-from database schema creation and rigorous data cleaning to exploratory data analysis (EDA) and strategic business problem-solving. By writing structured SQL queries to extract meaningful insights from transaction records, this project builds a strong fundamental foundation in relational database management and data analysis.

## Objectives
**1. Database Setup:** Build the retail database schema and successfully populate the sales transaction records.

**2. Data Cleansing:** Detect and purge records containing missing or null values to secure data integrity.

**3. Exploratory Analysis (EDA):** Run foundational queries to assess dataset distributions, metrics, and key variables.

**4. Business Intelligence:** Formulate advanced SQL scripts to resolve target commercial challenges and pull strategic retail insights.

## Project Structure

### 1. Database Setup
- **Database Creation:** Initialized the database named 'sql_project_p2'.

- **Table Schema:** Created a structured table named 'retail_sales' to store critical operational variables (Transaction ID, Date, Time, Customer Demographics, Product Category, Quantity, Price, COGS, and Total Sale).

```sql
-- SQL Retail Sales Analysis - P1
CREATE DATABASE sql_project_p2;


-- Create TABLE
DROP TABLE IF EXISTS retail_sales;
CREATE TABLE retail_sales
            (
                transactions_id INT PRIMARY KEY,	
                sale_date DATE,	 
                sale_time TIME,	
                customer_id	INT,
                gender	VARCHAR(15),
                age	INT,
                category VARCHAR(15),	
                quantity	INT,
                price_per_unit FLOAT,	
                cogs	FLOAT,
                total_sale FLOAT
            );

-- Streamlines the importation of raw transactional records from a flat CSV file into the PostgreSQL schema.
COPY retail_sales
FROM 'C:\Users\tushu\Downloads\proj1\Retail-Sales-Analysis-SQL-Project--P1\Book1.csv'
DELIMITER ','
CSV HEADER;

-- Runs a baseline inspection query
SELECT * FROM retail_sales;
```

### 2. Baseline Profiling & Cleansing
- **Volume Metrics:** Evaluated total transaction count and distinct customer footprint.

- **Product Auditing:** Isolated unique product categories across inventory.

- **Data Quality Control:** Detected and systematically removed incomplete or null records to ensure dataset integrity.

```sql
SELECT 
    COUNT(*) 
FROM retail_sales


-- Data Cleaning
SELECT * FROM retail_sales
WHERE transactions_id IS NULL

SELECT * FROM retail_sales
WHERE sale_date IS NULL

SELECT * FROM retail_sales
WHERE sale_time IS NULL

SELECT * FROM retail_sales
WHERE 
    transactions_id IS NULL
    OR
    sale_date IS NULL
    OR 
    sale_time IS NULL
    OR
    gender IS NULL
    OR
    category IS NULL
    OR
    quantity IS NULL
    OR
    cogs IS NULL
    OR
    total_sale IS NULL;
    
-- 
DELETE FROM retail_sales
WHERE 
    transactions_id IS NULL
    OR
    sale_date IS NULL
    OR 
    sale_time IS NULL
    OR
    gender IS NULL
    OR
    category IS NULL
    OR
    quantity IS NULL
    OR
    cogs IS NULL
    OR
    total_sale IS NULL;
    
```
### 3. Data Analysis & Findings

The following SQL queries were developed to answer specific business questions:

Q.1 **Write a SQL query to retrieve all columns for sales made on '2022-11-05**
```sql
SELECT *
FROM retail_sales
WHERE sale_date = '2022-11-05';
```

Q.2 **Write a SQL query to retrieve all transactions where the category is 'Clothing' and the quantity sold is more than 4 in the month of Nov-2022**
```sql
SELECT 
  *
FROM retail_sales
WHERE 
    category = 'Clothing'
    AND 
    TO_CHAR(sale_date, 'YYYY-MM') = '2022-11'
    AND
    quantity >= 4
```

Q.3 **Write a SQL query to calculate the total sales (total_sale) for each category.**
```sql
SELECT 
    category,
    SUM(total_sale) as net_sale,
    COUNT(*) as total_orders
FROM retail_sales
GROUP BY 1
```

Q.4 **Write a SQL query to find the average age of customers who purchased items from the 'Beauty' category.**
```sql
SELECT
    ROUND(AVG(age), 2) as avg_age
FROM retail_sales
WHERE category = 'Beauty'
```

Q.5 **Write a SQL query to find all transactions where the total_sale is greater than 1000.**
```sql
SELECT * FROM retail_sales
WHERE total_sale > 1000
```

Q.6 **Write a SQL query to find the total number of transactions (transaction_id) made by each gender in each category.**
```sql
SELECT 
    category,
    gender,
    COUNT(*) as total_trans
FROM retail_sales
GROUP 
    BY 
    category,
    gender
ORDER BY 1
```

Q.7 **Write a SQL query to calculate the average sale for each month. Find out best selling month in each year.**
```sql
SELECT 
       year,
       month,
    avg_sale
FROM 
(    
SELECT 
    EXTRACT(YEAR FROM sale_date) as year,
    EXTRACT(MONTH FROM sale_date) as month,
    AVG(total_sale) as avg_sale,
    RANK() OVER(PARTITION BY EXTRACT(YEAR FROM sale_date) ORDER BY AVG(total_sale) DESC) as rank
FROM retail_sales
GROUP BY 1, 2
) as t1
WHERE rank = 1
    
-- ORDER BY 1, 3 DESC
```

Q.8 **Write a SQL query to find the top 5 customers based on the highest total sales.**
```sql
SELECT 
    customer_id,
    SUM(total_sale) as total_sales
FROM retail_sales
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5
```

Q.9 **Write a SQL query to find the number of unique customers who purchased items from each category.**
```sql
SELECT 
    category,    
    COUNT(DISTINCT customer_id) as cnt_unique_cs
FROM retail_sales
GROUP BY category
```

Q.10 **Write a SQL query to create each shift and number of orders (Example Morning <12, Afternoon Between 12 & 17, Evening >17)**
```sql
WITH hourly_sale
AS
(
SELECT *,
    CASE
        WHEN EXTRACT(HOUR FROM sale_time) < 12 THEN 'Morning'
        WHEN EXTRACT(HOUR FROM sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
        ELSE 'Evening'
    END as shift
FROM retail_sales
)
SELECT 
    shift,
    COUNT(*) as total_orders    
FROM hourly_sale
GROUP BY shift
```

Q.11 **Calculate total revenue, total COGS, net profit, and the net profit margin percentage for each product category.**
```sql
SELECT 
    category,
    SUM(total_sale) AS total_revenue,
    SUM(cogs) AS total_cogs,
    SUM(total_sale) - SUM(cogs) AS net_profit,
    ROUND(((SUM(total_sale) - SUM(cogs)) / SUM(total_sale) * 100)::numeric, 2) AS profit_margin_pct
FROM retail_sales
GROUP BY category
ORDER BY net_profit DESC;
```

Q.12 **Segment customers into age groups ('Under 25', '25-40', '41-60', '60+') and find the total orders and revenue for each group.**
```sql
SELECT 
    CASE 
        WHEN age < 25 THEN 'Under 25'
        WHEN age BETWEEN 25 AND 40 THEN '25-40'
        WHEN age BETWEEN 41 AND 60 THEN '41-60'
        ELSE 'Above 60'
    END AS age_group,
    COUNT(*) AS total_orders,
    SUM(total_sale) AS total_sales
FROM retail_sales
WHERE age IS NOT NULL
GROUP BY age_group
ORDER BY total_sales DESC;
```

Q.13 **Calculate the Month-over-Month (MoM) percentage growth or decline in total revenue across all years.**
```sql
WITH monthly_sales AS (
    SELECT 
        EXTRACT(YEAR FROM sale_date) AS year,
        EXTRACT(MONTH FROM sale_date) AS month,
        SUM(total_sale) AS total_revenue
    FROM retail_sales
    GROUP BY 1, 2
)
SELECT 
    year,
    month,
    total_revenue,
    LAG(total_revenue) OVER (ORDER BY year, month) AS previous_month_revenue,
    ROUND(
        ((total_revenue - LAG(total_revenue) OVER (ORDER BY year, month)) / 
        LAG(total_revenue) OVER (ORDER BY year, month) * 100)::numeric, 2
    ) AS mom_growth_pct
FROM monthly_sales;
```

Q.14 **Find the percentage revenue contribution of each product category to the company's overall total sales.**
```sql
SELECT 
    category,
    SUM(total_sale) AS category_sales,
    ROUND((SUM(total_sale) / SUM(SUM(total_sale)) OVER () * 100)::numeric, 2) AS percentage_contribution
FROM retail_sales
GROUP BY category
ORDER BY percentage_contribution DESC;
```

Q.15 **Analyze weekly sales performance to determine which day of the week generates the highest volume of orders and revenue.**
```sql
SELECT 
    TO_CHAR(sale_date, 'Day') AS day_name,
    COUNT(*) AS total_orders,
    SUM(total_sale) AS total_sales
FROM retail_sales
GROUP BY day_name
ORDER BY total_sales DESC;
```

Q.16 **Identify loyal repeat customers who have placed more than 3 separate orders and rank them by total spending.**
```sql
SELECT 
    customer_id,
    COUNT(*) AS total_transactions,
    SUM(total_sale) AS total_spent
FROM retail_sales
GROUP BY customer_id
HAVING COUNT(*) > 3
ORDER BY total_spent DESC;
```

Q.17 **Retrieve the top 3 highest-value individual transactions for each product category using window functions.**
```sql
WITH ranked_sales AS (
    SELECT 
        transactions_id,
        category,
        total_sale,
        DENSE_RANK() OVER (PARTITION BY category ORDER BY total_sale DESC) AS rank
    FROM retail_sales
)
SELECT 
    transactions_id,
    category,
    total_sale,
    rank
FROM ranked_sales
WHERE rank <= 3;
```

Q.18 **Calculate the continuous running (cumulative) Year-to-Date (YTD) revenue total for each year chronologically.**
```sql
SELECT 
    sale_date,
    EXTRACT(YEAR FROM sale_date) AS sale_year,
    SUM(total_sale) AS daily_sales,
    SUM(SUM(total_sale)) OVER (
        PARTITION BY EXTRACT(YEAR FROM sale_date) 
        ORDER BY sale_date
    ) AS running_total_revenue
FROM retail_sales
GROUP BY sale_date
ORDER BY sale_date;
```

Q.19 **Identify the single peak business hour (highest number of transactions) for each individual product category.**
```sql
WITH hourly_category_sales AS (
    SELECT 
        category,
        EXTRACT(HOUR FROM sale_time) AS sale_hour,
        COUNT(*) AS total_orders,
        RANK() OVER (PARTITION BY category ORDER BY COUNT(*) DESC) AS rank
    FROM retail_sales
    GROUP BY category, sale_hour
)
SELECT 
    category,
    sale_hour,
    total_orders
FROM hourly_category_sales
WHERE rank = 1;
```

 Q.20 **Audit the database to find any records where total_sale does not match the product of quantity and price_per_unit.**
```sql
SELECT 
    transactions_id,
    quantity,
    price_per_unit,
    total_sale,
    (quantity * price_per_unit) AS expected_total_sale
FROM retail_sales
WHERE ABS(total_sale - (quantity * price_per_unit)) > 0.01;
```

---

### Key Findings & Analytical Insights

* **Customer Demographics:** Transacting audience spans diverse age brackets with strong balanced distributions across product segments like **Clothing** and **Beauty**.
* **High-Value Purchases:** Multiple premium basket exceptions exceeded **$1,000**, highlighting high-ticket transaction segments.
* **Seasonality & Trends:** Monthly tracking isolated clear performance variations, pin-pointing peak consumer demand periods.
* **Consumer Intelligence:** Formulated rankings highlighting top-tier high-spending cohorts and dominant categories.

---

### Core Reports Generated

* **Executive Sales Summary:** High-level overview detailing aggregate revenue, demographic distributions, and category-level KPIs.
* **Temporal Trend Analysis:** Granular performance deep-dives segmented by operational business shifts and monthly intervals.
* **Customer Distribution Report:** Strategic mapping of repeat loyalists and unique volume footprints per commercial sector.

---

### Project Conclusion

This project models an industry-standard SQL data analysis lifecycle, spanning robust schema construction, data cleansing, exploratory profiling, and corporate business intelligence querying. The generated outputs deliver actionable analytics to optimize retail resource allocation, target demographics, and product placement strategies.

---

### Setup & Implementation Guide

1. **Repository Ingestion:** Clone this project architecture from the GitHub source.
2. **Schema Deployment:** Execute the structural statements in `database_setup.sql` to initialize and populate the relational tables.
3. **Analytics Execution:** Run the production queries located in `analysis_queries.sql` to re-compile the portfolio reports.
4. **Ad-Hoc Customization:** Tailor or scale the source syntax to run additional diagnostics on the data pipeline.

---

**Author:** Zero Analyst

*This repository functions as a formal technical portfolio piece demonstrating relational logic, validation workflows, and business intelligence techniques required for core Data Analyst roles.*

---

### Professional Network & Community Space

* **LinkedIn:** www.linkedin.com/in/architabagchi5
* **Instagram:** 
