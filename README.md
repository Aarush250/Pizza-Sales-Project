# Pizza Sales Analysis - SQL Project

## Project Overview
This project analyzes pizza sales data using SQL queries to derive key business insights. 
The dataset includes order details, pizza types, and sales transactions. 
The queries cover a variety of analytical tasks, such as revenue calculation, popular pizza sizes, and order distribution.

---

## SQL Queries and Solutions

### 1. Retrieve the total number of orders placed.
```sql
SELECT 
    COUNT(order_id) AS total_orders
FROM
    orders;
```

### 2. Calculate the total revenue generated from pizza sales.
```sql
SELECT 
    ROUND(SUM(order_details.quantity * pizzas.price), 2) AS Total_Revenue
FROM
    order_details
        JOIN
    pizzas ON order_details.pizza_id = pizzas.pizza_id;
```

### 3. Identify the highest-priced pizza.
```sql
SELECT 
    pizza_types.name, pizzas.price
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
ORDER BY pizzas.price DESC
LIMIT 1;
```

### 4. Identify the most common pizza size ordered.
```sql
SELECT 
    pizzas.size,
    COUNT(order_details.order_details_id) AS order_count
FROM
    pizzas
        JOIN
    order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY size
ORDER BY order_count DESC
LIMIT 1;
```

### 5. List the top 5 most ordered pizza types along with their quantities.
```sql
SELECT 
    pizza_types.name, SUM(order_details.quantity) AS Quantity
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY Quantity DESC
LIMIT 5;
```

### 6. Join the necessary tables to find the total quantity of each pizza category ordered.
```sql
SELECT 
    pizza_types.category,
    SUM(order_details.quantity) AS Quantity
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category
ORDER BY Quantity DESC;
```

### 7. Determine the distribution of orders by hour of the day.
```sql
SELECT 
    HOUR(order_time) AS hour, COUNT(order_id) AS orders_count
FROM
    orders
GROUP BY hour
ORDER BY orders_count DESC;
```

### 8. Join relevant tables to find the category-wise distribution of pizzas.
```sql
SELECT 
    category, COUNT(name) AS Total_count
FROM
    pizza_types
GROUP BY category;
```

### 9. Group the orders by date and calculate the average number of pizzas ordered per day.
```sql
SELECT 
    ROUND(AVG(quantity), 0) AS average
FROM
    (SELECT 
        orders.order_date, SUM(order_details.quantity) AS quantity
    FROM
        orders
    JOIN order_details ON orders.order_id = order_details.order_id
    GROUP BY orders.order_date) AS order_quantity;
```

### 10. Determine the top 3 most ordered pizza types based on revenue.
```sql
SELECT 
    pizza_types.name, ROUND(SUM(quantity * price), 0) AS revenue
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY name
ORDER BY revenue DESC
LIMIT 3;
```

### 11. Calculate the percentage contribution of each pizza type to total revenue.
```sql
SELECT 
    category,
    ROUND(SUM(quantity * price) / (SELECT 
                    ROUND(SUM(quantity * price), 2) AS Total_Revenue
                FROM
                    order_details od
                        JOIN
                    pizzas p ON od.pizza_id = p.pizza_id) * 100,
            2) AS revenue
FROM
    pizza_types pt
        JOIN
    pizzas p ON pt.pizza_type_id = p.pizza_type_id
        JOIN
    order_details od ON od.pizza_id = p.pizza_id
GROUP BY category
ORDER BY revenue DESC;
```

### 12. Analyze the cumulative revenue generated over time.
```sql
SELECT Order_date, SUM(revenue) 
OVER(ORDER BY order_date) AS cum_revenue
FROM 
(SELECT orders.order_date,
SUM(order_details.quantity * pizzas.price) AS revenue
FROM order_details 
JOIN pizzas ON order_details.pizza_id = pizzas.pizza_id
JOIN orders ON orders.order_id = order_details.order_id
GROUP BY orders.order_date) AS sales;
```

### 13. Determine the top 3 most ordered pizza types based on revenue for each pizza category.
```sql
SELECT name, revenue 
FROM (
    SELECT category, name, revenue,
    RANK() OVER(PARTITION BY category ORDER BY revenue DESC) AS rn 
    FROM (
        SELECT pizza_types.category, pizza_types.name,
        SUM(order_details.quantity * pizzas.price) AS revenue
        FROM pizza_types 
        JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN order_details ON order_details.pizza_id = pizzas.pizza_id 
        GROUP BY pizza_types.category, pizza_types.name
    ) AS a
) AS b 
WHERE rn <= 3;
```

### Conclusion
This SQL project provides a detailed analysis of pizza sales, highlighting popular items, revenue distribution, and order patterns. 
These insights can help businesses optimize their inventory, pricing, and marketing strategies.
