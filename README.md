# SQL Project: Data Analysis for Zomato – A Food Delivery Company

## Project Overview
This project demonstrates my SQL skills through the analysis of data for **Zomato**, a leading food delivery company in India. It involves setting up a database, importing data, ensuring data integrity, and solving a series of business problems using complex SQL queries. The goal is to derive meaningful insights from data and solve real-world business challenges using SQL.

## Project Structure

### 1. **Database Setup**
- Created a `zomato_db` database, including essential tables such as restaurants, orders, customers, etc.
- Established relationships between tables using primary and foreign keys to ensure referential integrity.
- 
```sql 
CREATE DATABASE zomato_db;
```
```sql
CREATE TABLE customers
	(
		customer_id INT PRIMARY KEY,
		customer_name VARCHAR(30),
		reg_date DATE
	); 
	
CREATE TABLE restuarents
	(
		restaurant_id INT PRIMARY KEY,
		restaurant_name VARCHAR(55),
		city VARCHAR(20) ,
		opening_hours VARCHAR(55)
	);

CREATE TABLE orders
	(
		order_id INT PRIMARY KEY,
		customer_id INT, --Comming from customer table
		restaurant_id INT, -- comming from restuarent table
		order_item VARCHAR(50),
		order_date DATE,
		order_time TIME,
		order_status VARCHAR(55),
		total_amount FLOAT
	);

-- adding fk contraints
ALTER TABLE orders
ADD CONSTRAINT fk_customers
FOREIGN KEY (customer_id)
REFERENCES customers(customer_id);

	
-- adding fk contraints
ALTER TABLE orders
ADD CONSTRAINT fk_restuarent
FOREIGN KEY (restaurant_id)
REFERENCES restuarents(restaurant_id);

CREATE TABLE riders
	(
		rider_id INT PRIMARY KEY,
		rider_name VARCHAR(55),
		sign_up DATE
	);

DROP TABLE IF EXISTS diliveries;

CREATE TABLE diliveries
	(
		delivery_id INT PRIMARY KEY,
		order_id INT,   --coming from order table
		delivery_status VARCHAR(35) ,
		delivery_time TIME,
		rider_id INT,  -- coming from riders table
		CONSTRAINT fk_orders FOREIGN KEY (order_id) REFERENCES orders(order_id),
		CONSTRAINT fk_riders FOREIGN KEY (rider_id) REFERENCES riders(rider_id)
	);

```
### 2. **Data Import**
- Imported a sample dataset that includes business-related information like restaurant details, menu items, customer profiles, and order history.
```sql
SELECT * FROM TABLE_NAME;
```
```sql
SELECT * FROM customers;
SELECT * FROM restuarents;
SELECT * FROM orders;
SELECT * FROM riders;
SELECT * FROM deliveries;
```
### 3. **Data Cleaning & Transformation**
- Handled missing or null values to ensure data quality.
- Standardized and transformed data for analysis.

### 4. **Business Problem Solving**
- Solved 18 business problems related to Zomato’s operations using advanced SQL queries. Example problems include:
  - Identifying popular restaurants by rating, location, or cuisine.
  - Analyzing customer ordering behavior and trends.
  - Estimating delivery times to optimize operations.
  - Generating insights for improving marketing strategies or customer engagement.

## Key Technologies
- SQL (Structured Query Language)
- Database Design and Normalization
- Data Cleaning and Transformation

## Key Learning Outcomes
- Proficient in creating and managing SQL databases.
- Mastery of complex SQL queries for data analysis, aggregation, and reporting.
- Understanding of database relationships, normalization, and data integrity.
- Practical application of SQL to solve business challenges.


# Analysis & Reports


### 1. To find the top 5 most frequently ordered dishes by customer called "Arjun Mehta" in the last 1 year.
```sql
SELECT 
	customer_name,
	dishes,
	total_orders
FROM
	(SELECT 
		c.customer_id,
		c.customer_name,
		o.order_item as dishes,
		COUNT(*) as total_orders,
		DENSE_RANK() OVER(ORDER BY COUNT(*) DESC) as rank
	FROM orders as o
	JOIN
	customers as c
	ON c.customer_id = o.customer_id
	WHERE 
 	c.customer_name = 'Arjun Mehta'
	GROUP BY 1, 2, 3
	ORDER BY 1, 4 DESC) as t1
WHERE rank <= 5   

```
### 2. Identifing the time slots during which most orders are placed, based on 2-hour intervals.
#### Approach 1
```sql
SELECT
    CASE
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 0 AND 1 THEN '00:00 - 02:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 2 AND 3 THEN '02:00 - 04:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 4 AND 5 THEN '04:00 - 06:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 6 AND 7 THEN '06:00 - 08:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 8 AND 9 THEN '08:00 - 10:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 10 AND 11 THEN '10:00 - 12:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 12 AND 13 THEN '12:00 - 14:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 14 AND 15 THEN '14:00 - 16:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 16 AND 17 THEN '16:00 - 18:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 18 AND 19 THEN '18:00 - 20:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 20 AND 21 THEN '20:00 - 22:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 22 AND 23 THEN '22:00 - 00:00'
    END AS time_slot,
    COUNT(order_id) AS order_count
FROM Orders
GROUP BY time_slot
ORDER BY order_count DESC;
```
#### Approach 2
```sql
SELECT 
	FLOOR(EXTRACT(HOUR FROM order_time)/2)*2 as start_time,
	FLOOR(EXTRACT(HOUR FROM order_time)/2)*2 + 2 as end_time,
	COUNT(*) as total_orders
FROM orders
GROUP BY 1, 2
ORDER BY 3 DESC

```
### 3. Find the average order value per customer who has placed more than 750 orders.
```sql
SELECT 
	c.customer_name, 
	AVG(o.total_amount) as aov

FROM orders as o
JOIN customers as c ON c.customer_id = o.customer_id
GROUP BY 1
HAVING  COUNT(order_id) > 750;
```
### 4. List the customers who have spent more than 100K in total on food orders.
```sql
SELECT 
c.customer_name,
c.customer_id,
SUM(o.total_amount) as Total_Spent

FROM orders as o 

JOIN customers as c ON c.customer_id = o.customer_id

GROUP BY 1,2

HAVING SUM(o.total_amount) > 100000

```

### 5. To find orders that were placed but not delivered. Return each restaurant name, city and number of not delivered orders 
```sql
SELECT 
	r.restaurant_name,
	COUNT(o.order_id) as cnt_not_delivered_orders
FROM orders as o
LEFT JOIN 
restuarents as r
ON r.restaurant_id = o.restaurant_id
LEFT JOIN
diliveries as d
ON d.order_id = o.order_id
WHERE d.delivery_id IS NULL
GROUP BY 1
ORDER BY 2 DESC
```
### 6. Rank restaurants by their total revenue from the last year, including their name, total revenue, and rank within their city.
```sql
WITH ranking_table
AS
(
	SELECT 
		r.city,
		r.restaurant_name,
		SUM(o.total_amount) as revenue,
		RANK() OVER(PARTITION BY r.city ORDER BY SUM(o.total_amount) DESC) as rank
	FROM orders as o
	JOIN 
	restuarents as r
	ON r.restaurant_id = o.restaurant_id
	GROUP BY 1, 2
)
SELECT 
	*
FROM ranking_table
WHERE rank = 1
```

### 7.  Identify the most popular dish in each city based on the number of orders.
```sql
SELECT *
FROM
(SELECT 
	r.city,
	o.order_item as Dish,
	COUNT(o.order_id) as Total_orders,
	RANK() OVER(PARTITION BY r.city ORDER BY COUNT(o.order_id) DESC) as rank

FROM orders as o
JOIN restuarents as r ON r.restaurant_id = o.restaurant_id

GROUP BY 1,2) as t1

WHERE rank = 1

```

### 8. Find customers who haven’t placed an order in 2024 but did in 2023.
```sql

SELECT DISTINCT customer_id FROM orders
WHERE 
	EXTRACT(YEAR FROM order_date) = 2023
	AND
	customer_id NOT IN 
					(SELECT DISTINCT customer_id FROM orders
					WHERE EXTRACT(YEAR FROM order_date) = 2024)

```
### 9. Monthly Restaurant Growth Ratio: Calculate each restaurant's growth ratio based on the total number of delivered orders since its joining

```sql
WITH growth_ratio
AS
(
SELECT 
	o.restaurant_id,
	EXTRACT(YEAR FROM o.order_date) as year,
	EXTRACT(MONTH FROM o.order_date) as month,
	COUNT(o.order_id) as cr_month_orders,
	LAG(COUNT(o.order_id), 1) OVER(PARTITION BY o.restaurant_id ORDER BY EXTRACT(YEAR FROM o.order_date),
    EXTRACT(MONTH FROM o.order_date)) as prev_month_orders
FROM orders as o
JOIN
diliveries as d
ON o.order_id = d.order_id
WHERE d.delivery_status = 'Delivered'
GROUP BY 1, 2, 3
ORDER BY 1, 2
)
SELECT
	restaurant_id,
	month,
	prev_month_orders,
	cr_month_orders,
	ROUND(
	(cr_month_orders::numeric-prev_month_orders::numeric)/prev_month_orders::numeric * 100
	,2)
	as growth_ratio
FROM growth_ratio;

```
### 10. Customer Segmentation: Segment customers into 'Gold' or 'Silver' groups based on their total spending compared to the average order value (AOV). If a customer's total spending exceeds the AOV, label them as 'Gold'; otherwise, label them as 'Silver'. Write an SQL query to determine each segment's total number of orders and total revenue
```sql
SELECT 
	cx_category,
	SUM(total_orders) as total_orders,
	SUM(total_spent) as total_revenue
FROM

	(SELECT 
		customer_id,
		SUM(total_amount) as total_spent,
		COUNT(order_id) as total_orders,
		CASE 
			WHEN SUM(total_amount) > (SELECT AVG(total_amount) FROM orders) THEN 'Gold'
			ELSE 'silver'
		END as cx_category
	FROM orders
	group by 1
	) as t1
GROUP BY 1

SELECT AVG(total_amount) FROM orders

```
### 11. Rider Monthly Earnings: Calculate each rider's total monthly earnings, assuming they earn 8% of the order amount.
```sql
SELECT 
	d.rider_id,
	TO_CHAR(o.order_date, 'mm-yy') as month,
	SUM(total_amount) as revenue,
	SUM(total_amount)* 0.08 as riders_earning
FROM orders as o
JOIN diliveries as d
ON o.order_id = d.order_id
GROUP BY 1, 2
ORDER BY 1, 2
```

### 12. Rider Ratings Analysis: Find the number of 5-star, 4-star, and 3-star ratings each rider has riders receive this rating based on delivery time.

```sql
SELECT 
	rider_id,
	stars,
	COUNT(*) as total_stars
FROM
(
	SELECT
		rider_id,
		delivery_took_time,
		CASE 
			WHEN delivery_took_time < 15 THEN '5 star'
			WHEN delivery_took_time BETWEEN 15 AND 20 THEN '4 star'
			ELSE '3 star'
		END as stars
		
	FROM
	(
		SELECT 
			o.order_id,
			o.order_time,
			d.delivery_time,
			EXTRACT(EPOCH FROM (d.delivery_time - o.order_time + 
			CASE WHEN d.delivery_time < o.order_time THEN INTERVAL '1 day' 
			ELSE INTERVAL '0 day' END
			))/60 as delivery_took_time,
			d.rider_id
		FROM orders as o
		JOIN diliveries as d
		ON o.order_id = d.order_id
		WHERE delivery_status = 'Delivered'
	) as t1
) as t2
GROUP BY 1, 2
ORDER BY 1, 3 DESC
```

### 13. Order Frequency by Day: Analyze order frequency per day of the week and identify the peak day for each restaurant.
```sql
SELECT * FROM
(
	SELECT 
		r.restaurant_name,
		-- o.order_date,
		TO_CHAR(o.order_date, 'Day') as day,
		COUNT(o.order_id) as total_orders,
		RANK() OVER(PARTITION BY r.restaurant_name ORDER BY COUNT(o.order_id)  DESC) as rank
	FROM orders as o
	JOIN
	restuarents as r
	ON o.restaurant_id = r.restaurant_id
	GROUP BY 1, 2
	ORDER BY 1, 3 DESC
	) as t1
WHERE rank = 1
```
### 14. Customer Lifetime Value (CLV): Calculate the total revenue generated by each customer over all their orders.
```sql
SELECT 
	o.customer_id,
	c.customer_name,
	SUM(o.total_amount) as CLV
FROM orders as o
JOIN customers as c
ON o.customer_id = c.customer_id
GROUP BY 1, 2
```


### 15. Monthly Sales Trends: Identify sales trends by comparing each month's total sales to the previous month.
```sql
SELECT 
	EXTRACT(YEAR FROM order_date) as year,
	EXTRACT(MONTH FROM order_date) as month,
	SUM(total_amount) as total_sale,
	LAG(SUM(total_amount), 1) OVER(ORDER BY EXTRACT(YEAR FROM order_date), EXTRACT(MONTH FROM order_date)) as prev_month_sale
FROM orders
GROUP BY 1, 2

```

### Q.16 Rider Efficiency: Evaluate rider efficiency by determining average delivery times and identifying those with the lowest and highest averages.
```sql

WITH new_table
AS
(
	SELECT 
		*,
		d.rider_id as riders_id,
		EXTRACT(EPOCH FROM (d.delivery_time - o.order_time + 
		CASE WHEN d.delivery_time < o.order_time THEN INTERVAL '1 day' ELSE
		INTERVAL '0 day' END))/60 as time_deliver
	FROM orders as o
	JOIN diliveries as d
	ON o.order_id = d.order_id
	WHERE d.delivery_status = 'Delivered'
),

riders_time
AS

(
	SELECT 
		riders_id,
		AVG(time_deliver) avg_time
	FROM new_table
	GROUP BY 1
)
SELECT 
	MIN(avg_time),
	MAX(avg_time)
FROM riders_time

```
### 17. Order Item Popularity: Track the popularity of specific order items over time and identify seasonal demand spikes.
```sql
SELECT 
	order_item,
	seasons,
	COUNT(order_id) as total_orders
FROM 
(
SELECT 
		*,
		EXTRACT(MONTH FROM order_date) as month,
		CASE 
			WHEN EXTRACT(MONTH FROM order_date) BETWEEN 4 AND 6 THEN 'Spring'
			WHEN EXTRACT(MONTH FROM order_date) > 6 AND 
			EXTRACT(MONTH FROM order_date) < 9 THEN 'Summer'
			ELSE 'Winter'
		END as seasons
	FROM orders
) as t1
GROUP BY 1, 2
ORDER BY 1, 3 DESC
```



### 18. Rank each city based on the total revenue for last year 2023 
```sql
SELECT 
	r.city,
	SUM(total_amount) as total_revenue,
	RANK() OVER(ORDER BY SUM(total_amount) DESC) as city_rank
FROM orders as o
JOIN
restuarents as r
ON o.restaurant_id = r.restaurant_id
GROUP BY 1
```

