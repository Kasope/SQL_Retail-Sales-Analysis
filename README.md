# SQL Retail Sales Analysis Project (PostgreSQL)

## Project Overview

**Project Title**: Retail Sales Analysis  
**Database**: `p1_retail_db`, sourced From Zero Analyst 

This project is designed to demonstrate SQL skills and techniques typically used by data analysts to explore, clean, and analyze retail sales data. The project involves setting up a retail sales database, performing exploratory data analysis (EDA), and answering specific business questions through SQL queries. This project is ideal for those who are starting their journey in data analysis and want to build a solid foundation in SQL.

This project focuses on analyzing a retail transaction database using PostgreSQL to extract actionable business insights. My goal was to bridge the gap between raw database schemas and business questions about customer behavior, sales trends, and operational efficiency. 

As a data analyst, I used SQL to explore, clean, and analyze retail sales data. The project involves setting up a retail sales database, performing exploratory data analysis (EDA), and answering specific business questions through SQL queries. 

## Objectives

1. **Set up a retail sales database**: Create and populate a retail sales database with the provided sales data.
2. **Data Cleaning**: Identify and remove any records with missing or null values.
3. **Exploratory Data Analysis (EDA)**: Perform basic exploratory data analysis to understand the dataset.
4. **Business Analysis**: Use SQL to answer specific business questions and derive insights from the sales data.

## Project Structure

### 1. Database Setup

- **Database Creation**: The project starts by creating a database named `p1_retail_db`.
- **Table Creation**: A table named `retail_sales` is created to store the sales data. The table structure includes columns for transaction ID, sale date, sale time, customer ID, gender, age, product category, quantity sold, price per unit, cost of goods sold (COGS), and total sale amount.

```sql
CREATE DATABASE p1_retail_db;

CREATE TABLE retail_sales
(
  transaction_id INT PRIMARY KEY,
	sale_date DATE,
	sale_time TIME,
	customer_id INT,
	gender VARCHAR(15),
	age INT,
	category VARCHAR(15),
	quantity INT,
	price_per_unit FLOAT,
	cogs FLOAT,
	total_sale FLOAT
);
```

### 2. Data Exploration and Cleaning

- **Record Count**: Determine the total number of records in the dataset.
- **Customer Count**: Find out how many unique customers are in the dataset.
- **Category Count**: Identify all unique product categories in the dataset.
- **Null Value Check**: Check for any null values in the dataset and delete records with missing data.

```sql
SELECT * FROM retail_sales
LIMIT 10;

-- Checking duplicate with Group by
SELECT transaction_id,
	COUNT(*) as no_duplicate_count
FROM retail_sales
GROUP BY transaction_id
HAVING COUNT(*) > 1;

-- Checking duplcate with ROW_NUMBER, window function
SELECT * FROM
	(
		SELECT 
			transaction_id,
			customer_id,
			sale_date,
			ROW_NUMBER() OVER(PARTITION BY transaction_id ORDER BY sale_date) AS row_num
		FROM retail_sales
	) as duplicate_check
	WHERE row_num > 1;

-- Another way to check for duplicates if the data does not have a unique identifier
-- step 1: create a uniqueID
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY transaction_id, sale_date) AS row_num
FROM retail_sales;

-- step 2: use the code in step 1 to create a CTE
WITH duplicate_cte AS
(
	SELECT *,
	ROW_NUMBER() OVER(
	PARTITION BY transaction_id, sale_date) AS row_num
	FROM retail_sales
)
SELECT * FROM duplicate_cte WHERE row_num > 1

**-- Data Exploration**
-- 1. Record Count: Determine the total number of records in the dataset.
SELECT COUNT(*) FROM retail_sales;
-- ANS: 2000

-- 2. Customer Count: Find out how many unique customers are in the dataset.
SELECT COUNT(DISTINCT customer_id) AS unique_customers FROM retail_sales;
-- ANS: 155

-- 3. Category Count: Identify all unique product categories in the dataset.
SELECT DISTINCT(category) FROM retail_sales;
-- ANS: Electronics, Clothing, Beauty

-- 4. Null Value Check: Check for any null values in the dataset and delete records with missing data.
-- use this method

SELECT * FROM retail_sales
	WHERE transaction_id IS NULL OR sale_date IS NULL OR sale_time IS NULL OR customer_id IS NULL
	OR gender IS NULL OR age IS NULL OR category IS NULL OR quantity IS NULL 
	OR price_per_unit IS NULL OR cogs IS NULL OR total_sale IS NULL;

-- Or use this method
SELECT 
	COUNT(*),
	COUNT(transaction_id) transact_id,
	COUNT(sale_date) sale_date,
	COUNT(sale_time) sale_time,
	COUNT(customer_id) customer_id,
	COUNT(gender) gender,
	COUNT(category) category,
	COUNT(age) age,
	COUNT(quantity) quantity,
	COUNT(price_per_unit) price_per_unit,
	COUNT(cogs)cogs,
	COUNT(total_sale) total_sale
FROM retail_sales;

-- All these methods require me to write the column name one by one so those methods are not effective, Python will help better

-- To fix the null issue in this data, From the null value, I could see that age has 
-- null and 3 transactions at the end has null for all the transaction 
-- details which are the ones I will delete and change null of age to zero instead

-- changing null of age to zero
UPDATE retail_sales
SET age = 0
WHERE age IS NULL;

-- Deleting the last three data that has null in all the transaction details

DELETE FROM retail_sales
WHERE quantity IS NULL
AND price_per_unit IS NULL
AND cogs IS NULL
AND total_sale IS NULL;
```

### 3. Data Analysis and Findings

The following SQL queries were developed to answer specific business questions:

1a. **Write a SQL query to retrieve all columns for sales made on '2022-11-05**:
```sql
SELECT * FROM retail_sales
WHERE sale_date = '2022-11-05'
ORDER BY total_sale ASC;
```

1b. **Write a SQL query to retrieve total sales made on '2022-11-05:**
```sql
SELECT SUM(total_sale)
FROM retail_sales
WHERE sale_date = '2022-11-05';
```

2. **Write a SQL query to retrieve all transactions where the category is 'Clothing' and the quantity sold is more than 4 in the month of Nov-2022**:
```sql
SELECT * FROM retail_sales
	WHERE category = 'Clothing' 
	AND quantity > 4 
	AND TO_CHAR(sale_date, 'YYYY-MM') = 'Nov 2022';
```

3. **Write a SQL query to calculate the total sales (total_sale) for each category.**:
```sql
SELECT category, 
	SUM(total_sale) FROM retail_sales
	GROUP BY  category; 
```

4. **Write a SQL query to find the average age of customers who purchased items from the 'Beauty' category.**:
```sql
SELECT 
	ROUND(AVG(age), 2) avg_age
FROM retail_sales
WHERE category = 'Beauty';
```

5. **Write a SQL query to find all transactions where the total_sale is greater than 1000.**:
```sql
SELECT * FROM retail_sales
WHERE total_sale > 1000;
```

6. **Write a SQL query to find the total number of transactions (transaction_id) made by each gender in each category.**:
```sql
SELECT 
	COUNT(transaction_id), 
	gender, 
	category 
FROM retail_sales
GROUP BY gender, category;
```

7. **Write a SQL query to calculate the average sale for each month. Find out best selling month in each year**:
```sql
SELECT TO_CHAR(sale_date, 'Month') AS months, ROUND(AVG(total_sale)::numeric, 2) AS avg_sale FROM retail_sales
GROUP BY TO_CHAR(sale_date, 'Month');

-- complete solution
WITH monthly_metrics AS
(
	SELECT 
		EXTRACT(YEAR FROM sale_date) AS sale_year,
		TO_CHAR(sale_date, 'Month') AS sale_month,
		ROUND(AVG(total_sale)::numeric, 2) AS avg_sale,
		SUM(total_sale)AS total_revenue,
		RANK() OVER(PARTITION BY EXTRACT(YEAR FROM sale_date) ORDER BY SUM(total_sale) DESC) AS sales_rank
	FROM retail_sales
	GROUP BY 
		EXTRACT(YEAR FROM sale_date),
		TO_CHAR(sale_date, 'Month')
)
SELECT 
	sale_year,
	TRIM(sale_month) AS best_selling_month,
	avg_sale,
	total_revenue AS month_total_evenue
FROM monthly_metrics
WHERE sales_rank = 1
ORDER BY sale_year;
```

8. **Write a SQL query to find the top 5 customers based on the highest total sales **:
```sql
SELECT 
	customer_id,
	SUM(total_sale) AS total_sales
FROM retail_sales
GROUP BY customer_id
ORDER BY SUM(total_sale) DESC 
LIMIT 5;
```

9. **Write a SQL query to find the number of unique customers who purchased items from each category.**:
```sql
SELECT 
	 COUNT(DISTINCT customer_id) AS unique_customers,
	category
FROM retail_sales
WHERE total_sale > 0
GROUP BY category;
```

10. **Write a SQL query to create each shift and number of orders (Example Morning <12, Afternoon Between 12 & 17, Evening >17)**:
```sql
SELECT Count(total_sale),
	CASE 
		WHEN EXTRACT(HOUR FROM sale_time) < 12 THEN 'Morning'
		WHEN EXTRACT(HOUR FROM sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
		ELSE 'Evening'
	END AS shift
FROM retail_sales
GROUP BY 
	CASE 
		WHEN EXTRACT(HOUR FROM sale_time) < 12 THEN 'Morning'
		WHEN EXTRACT(HOUR FROM sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
		ELSE 'Evening'
	END

-- Using CTE
WITH shift_order AS
(
	SELECT total_sale,
		CASE 
			WHEN EXTRACT(HOUR FROM sale_time) < 12 THEN 'Morning'
			WHEN EXTRACT(HOUR FROM sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
			ELSE 'Evening'
		END AS shift_name
	FROM retail_sales
)
SELECT 
	shift_name, 
	COUNT(total_sale) 
FROM shift_order
GROUP BY shift_name;
```

## Findings

- **Customer Demographics**: The dataset includes customers from various age groups, with sales distributed across different categories such as Clothing and Beauty.
- **High-Value Transactions**: Several transactions had a total sale amount greater than 1000, indicating premium purchases.
- **Sales Trends**: Monthly analysis shows variations in sales, helping identify peak seasons.
- **Customer Insights**: The analysis identifies the top-spending customers and the most popular product categories.

## Conclusion

This project covers database setup, data cleaning, exploratory data analysis, and business-driven SQL queries. The findings from this project can help drive business decisions by understanding sales patterns, customer behavior, and product performance. SQL techniques used are:
* **Conditional Logic:** Utilizing `CASE WHEN` to dynamically group continuous time data into operational categories.
* **Window Functions:** Writing analytical operations like `DENSE_RANK() OVER (PARTITION BY ... ORDER BY ...)` to determine performance tiers without losing raw row detail.
* **Common Table Expressions (CTEs):** Structuring complex queries using `WITH` statements to write modular, readable, and highly maintainable code.
* **Date and Time Extraction:** Isolating years, months, and hours from transactional timestamps using `EXTRACT` and `TO_CHAR` for accurate chronological reporting.
* **Aggregations:** Applying grouping and filtration techniques using `SUM`, `AVG`, `COUNT(DISTINCT)`, and `GROUP BY`.






