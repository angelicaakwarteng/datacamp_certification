# Datacamp_certification 2
This is a Sql project as part of datacamp data analyst certification. Certifications are a great way to showcase to employers that you can apply problem-solving using the required tool-in ny case, sql.

-- Display the first few rows
SELECT * FROM product_sales LIMIT 5;

-- Get the number of rows and columns
SELECT COUNT(*) AS total_rows FROM product_sales;

-- Display table schema
DESCRIBE product_sales;

-- Summary statistics
SELECT 
    MIN(revenue) AS min_revenue,
    MAX(revenue) AS max_revenue,
    AVG(revenue) AS avg_revenue,
    MEDIAN(revenue) AS median_revenue
FROM product_sales;

-- Get unique values in sales_method column
SELECT DISTINCT sales_method FROM product_sales;

-- Count NULL values in sales_method and week columns
SELECT COUNT(*) AS null_sales_method FROM product_sales WHERE sales_method IS NULL;
SELECT COUNT(*) AS null_week FROM product_sales WHERE week IS NULL;

-- Updating data type
ALTER TABLE product_sales MODIFY sales_method VARCHAR(50);
ALTER TABLE product_sales MODIFY customer_id VARCHAR(50);
ALTER TABLE product_sales MODIFY state VARCHAR(50);

-- Updating sales_method values
UPDATE product_sales 
SET sales_method = 'Email' 
WHERE sales_method = 'email';

UPDATE product_sales 
SET sales_method = 'Email + Call' 
WHERE sales_method = 'em + call';

-- Count total rows
SELECT COUNT(*) FROM product_sales;

-- Count NULL values in revenue column
SELECT COUNT(*) AS null_revenue FROM product_sales WHERE revenue IS NULL;

-- Compute threshold (5% of total rows)
WITH total_rows AS (
    SELECT COUNT(*) AS total FROM product_sales
)
SELECT 0.05 * total AS threshold FROM total_rows;

-- Fill NULL values in revenue with median revenue
WITH median_value AS (
    SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY revenue) AS median_revenue FROM product_sales
)
UPDATE product_sales 
SET revenue = (SELECT median_revenue FROM median_value)
WHERE revenue IS NULL;

-- Determine whether to drop or impute based on threshold
WITH total_rows AS (
    SELECT COUNT(*) AS total FROM product_sales
),
threshold AS (
    SELECT 0.05 * total AS thresh FROM total_rows
),
null_count AS (
    SELECT COUNT(*) AS null_revenue FROM product_sales WHERE revenue IS NULL
)
SELECT 
    CASE 
        WHEN null_revenue < (SELECT thresh FROM threshold) THEN 'drop'
        ELSE 'impute'
    END AS action
FROM null_count;

SELECT sales_method, COUNT(*) AS method_count
FROM product_sales
GROUP BY sales_method
ORDER BY method_count DESC;

SELECT sales_method, COUNT(customer_id) AS customer_count
FROM product_sales
GROUP BY sales_method;

SELECT revenue FROM product_sales;

SELECT sales_method, SUM(revenue) AS total_revenue
FROM product_sales
GROUP BY sales_method;

SELECT sales_method, COUNT(nb_sold) AS total_items_sold
FROM product_sales
GROUP BY sales_method;

SELECT week, SUM(revenue) AS total_revenue
FROM product_sales
GROUP BY week
ORDER BY week;

WITH weekly_revenue AS (
    SELECT week, SUM(revenue) AS total_revenue
    FROM product_sales
    GROUP BY week
),
lagged_revenue AS (
    SELECT week, 
           total_revenue, 
           LAG(total_revenue, 1, 0) OVER (ORDER BY week) AS prev_week_revenue
    FROM weekly_revenue
)
SELECT week, 
       total_revenue, 
       (total_revenue - prev_week_revenue) AS revenue_difference
FROM lagged_revenue;
