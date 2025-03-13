# Zomato SQL Project


# Analysis & Reports


### Q.1 Write a query to find the top 5 most frequently ordered dishes by customer called "Arjun Mehta" in the last 1 year.
''' sql 
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
'''
