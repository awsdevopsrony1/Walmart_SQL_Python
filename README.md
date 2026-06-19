##  🛒 Walmart_Sales_Analysis | End-to-End Python + MySQL Project
- How can Walmart leverage its sales data to boost revenue and profit? This project seeks to answer that question by performing an exploratory data analysis (EDA) on historical transaction data.
## 📌 Project Overview

This project demonstrates an end-to-end Data Analytics workflow using Python and MySQL. The objective was to extract Walmart sales data from Kaggle, clean and transform the data using Pandas, load it into MySQL using SQLAlchemy, and generate business insights through SQL queries
## 📊 Project Workflow

## 📊 Project Workflow

![Walmart Sales Analysis Workflow](https://raw.githubusercontent.com/awsdevopsrony1/Walmart_SQL_Python/main/workflow.png)



## 🛠️ Tools & Technologies

- Python
- Pandas
- MySQL
- SQLAlchemy
- Kaggle API
- Jupyter Notebook



## SQL Topics Covered

- Group By
- Aggregate Functions
- Window Functions
- Date Functions

## 🚀 Workflow

1. Download Walmart sales dataset using Kaggle API
2. Perform data cleaning and preprocessing using Pandas
3. Load cleaned data into MySQL using SQLAlchemy
4. Analyze sales data using SQL queries
5. Generate business insights and recommendations

## 📈 Key Skills Demonstrated

- Data Cleaning & Transformation
- ETL Pipeline Development
- SQL Analysis
- Database Integration
- Business Insight Generation
- Python Programming

## Key Insights from Walmart Sales Analysis

### Payment Methods :-

- Three unique payment methods are used: Credit Card, E-Wallet, and Cash.

- Credit Card dominates with the highest number of transactions (4,260) and units sold (9,573), making it the top-performing payment method.

- E-Wallet follows closely with 3,911 transactions and 8,932 units sold, while Cash lags behind with 1,880 transactions and 5,077 units sold.

- On average, E-Wallet provides the highest profit margin per transaction, even though Credit Card leads in volume.

--------------------------

### Product Categories :-

- Home & Lifestyle and Fashion Accessories consistently generate the highest revenue and profit.

- Combined, these two categories contribute ~80% of the total profit, making them the key drivers of overall business performance.

- These categories also receive higher customer ratings compared to others.

- Food & Beverages, Health & Beauty, and Sports & Travel are purchased in higher quantities, suggesting strong demand but lower profitability compared to top-performing categories.

- Food & Beverages and Health & Beauty show strong profit margins, making them strategically important.

----------------------------

### Branch & City Performance :-

- There are 100 unique branches across all locations.

- E-Wallet is the most popular transaction method in branches, indicating customer trust in digital payments.

- Weslaco, Waxahachie, and Plano are the top-performing cities, contributing significantly to total sales.

-------------------------

### Time & Seasonality Trends :-

- Evening (3 to 9 PM) is the peak sales window, both in terms of revenue and transactions.

- Thursday and Sunday are the highest revenue-generating days, highlighting weekend and mid-week spikes.

- Months 11, 12, 1, 2, 3 see the highest sales volume, likely due to holiday shopping. marking it as the prime revenue period

--------------------

### Overall Summary :-

- Credit Card drives the highest transactions and units sold, while E-Wallet offers better profit margins.

- Prioritize and promote credit card payment options. Investigate the drivers behind eWallet's high margins for potential replication across other methods.

- Home & Lifestyle and Fashion Accessories dominate profitability and revenue, forming the core business categories.

- Evenings, Thursdays, Sundays, and the holiday season (Nov–Mar) are the most critical sales periods.

- Align marketing campaigns, staffing schedules, and inventory stocking to these peak periods. Schedule promotions and flash sales for Thursday and Sunday evenings.

- Conduct a deep-dive analysis into the operations, local marketing, and customer demographics of the top-performing cities (Weslaco, Waxahachie, Plano) to identify best practices that can be applied to lower-performing branches

-----------------------------------------

## Solving Questins in SQL :-

### Database Setup & Data Cleaning
- Create database
```sql
CREATE DATABASE walmart;
USE walmart;
```

- Check data types
```sql
DESCRIBE walmart;
```

- Remove $ sign from unit_price
```sql
SET SQL_SAFE_UPDATES = 0;
UPDATE walmart
SET unit_price = REPLACE(unit_price, '$', '');
SET SQL_SAFE_UPDATES = 1;
```

- Modify column data types
```sql
ALTER TABLE walmart 
MODIFY unit_price DECIMAL(10,2);

ALTER TABLE walmart 
MODIFY date DATE;

ALTER TABLE walmart 
MODIFY time TIME;
```

- Add total column (quantity * unit_price)
```sql
ALTER TABLE walmart
ADD COLUMN total DECIMAL(10,2);

SET SQL_SAFE_UPDATES = 0;
UPDATE walmart
SET total = unit_price * quantity;
SET SQL_SAFE_UPDATES = 1;
```

- Data cleaning - remove records with NULL values
```sql
SET SQL_SAFE_UPDATES = 0;
DELETE FROM walmart WHERE unit_price IS NULL;
DELETE FROM walmart WHERE quantity IS NULL;
SET SQL_SAFE_UPDATES = 1;
```

### 1. Payment Methods Analysis
#### Understand customer preferences for payment methods to optimize payment strategies.
- Get unique payment methods
```sql
SELECT DISTINCT(payment_method) FROM walmart;
```

- Transactions by payment method
```sql
SELECT 
    payment_method,
    COUNT(*) AS total_transactions
FROM walmart
GROUP BY payment_method
ORDER BY COUNT(*) DESC;
```

### 2. Highest-Rated Category in Each Branch
#### Recognize popular categories in specific branches to enhance customer satisfaction.
- Count unique branches
```sql
SELECT COUNT(DISTINCT(branch)) FROM walmart;
```

- Top rated categories by branch
```sql
WITH CTE AS (
    SELECT 
        Branch, 
        category, 
        rating,
        DENSE_RANK() OVER (PARTITION BY Branch ORDER BY rating DESC) AS rnk
    FROM walmart
)
SELECT Branch, category, rating
FROM CTE
WHERE rnk = 1;
```

### 3. Busiest Day for Each Branch
#### Optimize staffing and inventory management by identifying peak days.
```sql
WITH cte AS (
    SELECT 
        Branch,
        DAYNAME(date) AS day_of_week,
        COUNT(*) AS total_transactions,
        DENSE_RANK() OVER(PARTITION BY Branch ORDER BY COUNT(*) DESC) AS rnk
    FROM walmart
    GROUP BY Branch, day_of_week
)
SELECT Branch, day_of_week, total_transactions
FROM cte
WHERE rnk = 1;
```

### 4. Quantity Sold by Payment Method
#### Track sales volume by payment type to understand purchasing habits.
```sql
SELECT 
    payment_method,
    SUM(quantity) AS total_units_sold
FROM walmart
GROUP BY payment_method
ORDER BY COUNT(*) DESC, SUM(quantity) DESC;
```

### 5. Category Ratings by City
#### Guide city-level promotions by understanding regional preferences.
```sql
SELECT 
    City,
    category,
    MIN(rating) AS min_rating,
    MAX(rating) AS max_rating,
    ROUND(AVG(rating), 2) AS avg_rating
FROM walmart
GROUP BY City, category
ORDER BY City, category;
```
### 6. Total Profit by Category
#### Identify high-profit categories to focus expansion efforts.
```sql
SELECT
    category,
    SUM(total) AS total_revenue,
    ROUND(SUM(total * profit_margin), 2) AS profit
FROM walmart
GROUP BY category
ORDER BY ROUND(SUM(total * profit_margin), 2) DESC;
```
### 7. Most Common Payment Method per Branch
#### Understand branch-specific payment preferences to streamline payment processing.
```sql
WITH cte AS (
    SELECT 
        Branch,
        payment_method,
        COUNT(*) AS total_transactions,
        DENSE_RANK() OVER(PARTITION BY Branch ORDER BY COUNT(*) DESC) AS rnk
    FROM walmart
    GROUP BY Branch, payment_method
)
SELECT Branch, payment_method, total_transactions
FROM cte
WHERE rnk = 1;
```

### 8. Sales Shifts Throughout the Day
#### Manage staff shifts and stock replenishment during high-sales periods.
```sql
SELECT 
    Branch,
    CASE 
        WHEN HOUR(time) < 12 THEN 'Morning'
        WHEN HOUR(time) > 12 AND HOUR(time) < 17 THEN 'Afternoon'
        ELSE 'Evening'
    END AS day_time,
    COUNT(*) AS total_transactions
FROM walmart
GROUP BY Branch, day_time
ORDER BY Branch, total_transactions DESC;
```

### 9. Branches with Highest Revenue Decline Year-Over-Year
#### Detect branches with declining revenue to address local issues.
```sql
WITH yearly_revenue AS (
    SELECT 
        Branch,
        YEAR(date) AS year_,
        SUM(unit_price * quantity) AS revenue
    FROM walmart
    GROUP BY Branch, YEAR(date)
),
revenue_lag AS (
    SELECT 
        Branch,
        year_,
        revenue,
        LAG(revenue) OVER (PARTITION BY Branch ORDER BY year_) AS previous_revenue
    FROM yearly_revenue
),
yoy_growth AS (
    SELECT
        Branch,
        year_,
        revenue,
        previous_revenue,
        ROUND((revenue - previous_revenue)/previous_revenue, 2) AS YOY_growth
    FROM revenue_lag
)
SELECT *
FROM yoy_growth;
```

## Additional SQL Questions

### 10.Total number of invoices
```sql
SELECT COUNT(*) FROM walmart;
```

### 11. Total sales revenue
```sql
SELECT *, unit_price * quantity AS revenue FROM walmart;
```

### 12. Total quantity of items sold by category
```sql
SELECT 
    category,
    COUNT(*) AS total_units
FROM walmart
GROUP BY category
ORDER BY COUNT(*) DESC;
```

### 13. Unique product categories
```sql
SELECT DISTINCT(category) FROM walmart;
```

### 14. Total sales revenue by branch
```sql
SELECT 
    Branch,
    City,
    SUM(total) AS total_revenue
FROM walmart
GROUP BY Branch, City
ORDER BY SUM(total) DESC;
```

### 15. Maximum and minimum ratings
```sql
SELECT MAX(rating), MIN(rating) FROM walmart;
```

### 16. Total sales per weekday
```sql
SELECT 
    DAYNAME(date) AS weekday,
    SUM(total) AS total_sales
FROM walmart
GROUP BY DAYNAME(date)
ORDER BY SUM(total) DESC;
```

### 17. Total revenue per product category
```sql
SELECT 
    category,
    SUM(total) AS total_revenue
FROM walmart
GROUP BY category
ORDER BY SUM(total) DESC;
```

### 18. City with highest sales revenue
```sql
SELECT 
    city,
    SUM(total) AS total_revenue
FROM walmart
GROUP BY city
ORDER BY SUM(total) DESC;
```

### 19. Average basket size by category
```sql
SELECT 
    category,
    ROUND(AVG(quantity), 0) AS avg_quantity
FROM walmart
GROUP BY category
ORDER BY AVG(quantity) DESC;
```

### 20. Total sales by month
```sql
SELECT 
    YEAR(date) AS year,
    MONTH(date) AS month,
    SUM(total) AS total_sales
FROM walmart
GROUP BY YEAR(date), MONTH(date)
ORDER BY YEAR(date), MONTH(date);
```

### 21. Category with highest profit margin
```sql
SELECT 
    category,
    ROUND(AVG(profit_margin), 2) AS avg_profit_margin
FROM walmart
GROUP BY category
ORDER BY AVG(profit_margin) DESC;
```

### 22. Average rating per branch
```sql
SELECT 
    Branch,
    ROUND(AVG(rating), 0) AS avg_rating
FROM walmart
GROUP BY Branch;
```

### 23. Branch with highest average profit margin
```sql
SELECT 
    Branch,
    ROUND(AVG(profit_margin), 2) AS avg_profit_margin
FROM walmart
GROUP BY Branch
ORDER BY AVG(profit_margin) DESC;
```

### 24. Peak sales hour of the day
```sql
SELECT 
    HOUR(time) AS hour,
    SUM(total) AS total_sales
FROM walmart
GROUP BY HOUR(time)
ORDER BY SUM(total) DESC;
```

### 25. Top 3 performing branches each month by revenue
```sql
WITH cte AS (
    SELECT 
        Branch,
        MONTH(date) AS month,
        SUM(total) AS total_sales
    FROM walmart
    GROUP BY Branch, MONTH(date)
),
cte2 AS (
    SELECT 
        *,
        DENSE_RANK() OVER(PARTITION BY month ORDER BY total_sales DESC) AS rnk
    FROM cte
)
SELECT *
FROM cte2
WHERE rnk <= 3
ORDER BY month, rnk;
```

### 26. Category with highest month-over-month sales growth
```sql
WITH cte AS (
    SELECT 
        category,
        MONTH(date) AS month,
        SUM(total) AS total_sales
    FROM walmart
    GROUP BY category, MONTH(date)
),
cte2 AS (
    SELECT 
        *,
        DENSE_RANK() OVER(PARTITION BY category ORDER BY total_sales DESC) AS rnk
    FROM cte
)
SELECT *
FROM cte2
WHERE rnk = 1
ORDER BY category, month;
```

### 27. Average profit per transaction by payment method
```sql
SELECT 
    payment_method,
    ROUND(AVG(profit_margin), 2) AS avg_profit_margin
FROM walmart
GROUP BY payment_method;
```

### 28. Category profit contribution percentage
```sql
WITH profit_cte AS (
    SELECT 
        category,
        ROUND(SUM(unit_price * quantity * profit_margin), 2) AS profit
    FROM walmart
    GROUP BY category
),
total_cte AS (
    SELECT SUM(profit) AS total_profit
    FROM profit_cte
)
SELECT 
    p.category,
    p.profit,
    ROUND((p.profit / t.total_profit) * 100, 2) AS contribution_percent
FROM profit_cte p
CROSS JOIN total_cte t
ORDER BY contribution_percent DESC;
```

### 29. Rolling 7-day average of sales revenue
```sql
WITH daily_sales AS (
    SELECT 
        DATE(date) AS sales_date,
        SUM(total) AS daily_revenue
    FROM walmart
    GROUP BY DATE(date)
)
SELECT 
    sales_date,
    daily_revenue,
    ROUND(
        AVG(daily_revenue) OVER (
            ORDER BY sales_date
            ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
        ), 2
    ) AS rolling_7day_avg
FROM daily_sales
ORDER BY sales_date;
```

-------------------------

## Solving Questions in Pandas :-


### 1.Calculate the total quantity of items sold.
```python
quantity=df['quantity'].sum()
quantity
```


### 2.Find the average unit_price across all transactions.
```sql
avg_price=df['unit_price'].mean().round(2)
avg_price
```


### 3.Count how many unique product categories exist.

- it igves total unique count of products
```python
unique_product=df['category'].nunique()
unique_product
```

- it gives the unique product list
```python
unique_product=df['category'].unique()
print(sorted(unique_product))
```


### 4.Find the most common payment_method.
```python
payment_method = (
    df.groupby('payment_method')['invoice_id']
      .count()
      .reset_index()
      .rename(columns={'invoice_id': 'total_transactions'})
      .sort_values(by='total_transactions', ascending=False)
)

print(payment_method.head(1)) 
```


### 5.Show the total sales per Branch.
```python
sales_per_branch=df.groupby('Branch')['total'].sum().round(2).reset_index().rename(columns={'total':'total_sales'}).sort_values(by='Branch')
sales_per_branch
```


### 6.Find the highest rating given by a customer.
```python
maxi_rating=df['rating'].max()
maxi_rating
```


### 7.Show the average profit_margin across all sales.
```python
avg_profit_margin=df['profit_margin'].mean().round(2)
avg_profit_margin
```

### 8.Convert date to datetime and extract the day of the week each transaction occurred.
- First convert date & time separately to datetime64[ns]
```python
df['date'] = pd.to_datetime(df['date'], errors='coerce')
```

### If you want a full datetime column (recommended for time-series)
```python
df['datetime'] = pd.to_datetime(df['date'].astype(str) + ' ' + df['time'].astype(str))
```


### 9.Calculate the total revenue per product category.
```python
category_revenue=df.groupby('category')['total'].sum().reset_index().rename(columns={'total':'total_revenue'})
category_revenue
```

### 10.Find which city generated the highest sales revenue.
```python
city_revenue=df.groupby('City')['total'].sum().reset_index().rename(columns={'total':'total_revenue'})
city_revenue
```

### 11.Compare the sales distribution by payment method (e.g., cash vs card).
```python
payment_method_sales=df.groupby('payment_method')['total'].sum().reset_index().rename(columns={'total':'total_sales'})
payment_method_sales
```

