# MySQL_project

create database pizzahut;

create table orders (
order_id int not null,
order_date date not null,
order_time time not null,
primary key (order_id));
select * from orders;

create table order_details (
order_details_id int not null,
order_id int not null,
pizza_id VARCHAR(50) NOT NULL,
quantity int not null,
primary key (order_details_id));

select * from order_details;

-- Total number of orders placed.

select count(order_id) 
as total_orders from orders;


-- Calculate the total revenue generated from pizza sales.
SELECT 
    ROUND(SUM(order_details.quantity * pizzas.price), 2) 
    AS total_revenue
FROM  order_details JOIN  
	pizzas ON pizzas.pizza_id = order_details.pizza_id;
    
    -- Identify the highest-priced pizza.

SELECT
    pizza_types.name, pizzas.price
FROM pizza_types JOIN pizzas ON 
	pizza_types.pizza_type_id = pizzas.pizza_type_id
ORDER BY pizzas.price DESC LIMIT 1;

-- Identify the most common pizza size ordered.

SELECT 
    pizzas.size, COUNT(order_details.order_details_id) AS 
    order_count FROM pizzas JOIN order_details ON 
    pizzas.pizza_id = order_details.pizza_id
GROUP BY pizzas.size ORDER BY order_count DESC;

-- List the top 5 most ordered pizza 
-- types along with their quantities.

SELECT 
    pizza_types.name, 
    SUM(order_details.quantity) AS quantity
FROM pizza_types JOIN pizzas ON 
	pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY quantity DESC
	LIMIT 5;
    
-- Join the necessary tables to find the
-- total quantity of each pizza category ordered.

SELECT 
    pizza_types.category, 
    SUM(order_details.quantity) AS total_quantity
FROM pizza_types JOIN pizzas 
ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category
	ORDER BY total_quantity DESC;
    
-- Determine the distribution of orders by hour of the day.

SELECT
    HOUR(order_time) AS hour,
COUNT(order_id) AS order_count
FROM orders GROUP BY HOUR(order_time);

-- Join relevant tables to find the
-- category-wise distribution of pizzas.

SELECT 
category, COUNT(name) as pizza_count
	FROM pizza_types
		GROUP BY category;
        
-- Group the orders by date and calculate the 
-- average number of pizzas ordered per day.

SELECT ROUND(AVG(quantity), 0)
FROM ( SELECT orders.order_date, SUM(order_details.quantity)
 AS quantity FROM orders JOIN order_details ON 
 orders.order_id = order_details.order_id 
 GROUP BY orders.order_date) AS order_quantity;
 
-- Determine the top 3 most ordered
-- pizza types based on revenue.

SELECT 
    pizza_types.name, SUM(order_details.quantity * pizzas.price) 
    AS revenue FROM pizza_types JOIN pizzas
    ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN order_details
    ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name ORDER BY revenue DESC
LIMIT 3;

-- Calculate the percentage contribution
-- of each pizza type to total revenue.

SELECT 
    pizza_types.category,
    CONCAT(ROUND(SUM(order_details.quantity * pizzas.price) / 
		(SELECT SUM(order_details.quantity * pizzas.price)
		FROM order_details 
		JOIN pizzas ON pizzas.pizza_id = order_details.pizza_id )
		* 100, 2), '%') AS revenue_percentage
FROM pizza_types 
JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category 
ORDER BY revenue_percentage DESC;

-- Analyze the cumulative revenue generated over time.

SELECT order_date,
       ROUND(SUM(revenue) OVER (ORDER BY order_date),2) AS cum_revenue
FROM
( SELECT orders.order_date, 
	ROUND (SUM(order_details.quantity * pizzas.price),2) AS revenue
    FROM order_details
    JOIN pizzas ON order_details.pizza_id = pizzas.pizza_id
    JOIN orders ON orders.order_id = order_details.order_id
    GROUP BY orders.order_date
	) AS sales;
    
-- Determine the top 3 most ordered pizza types 
-- based on revenue for each pizza category.

SELECT name, ROUND(revenue,2) AS revenue
FROM ( SELECT category, name, revenue,
	RANK() OVER (PARTITION BY category ORDER BY revenue DESC) AS rn
    FROM ( SELECT pizza_types.category, pizza_types.name,
		SUM(order_details.quantity * pizzas.price) AS revenue
	FROM pizza_types JOIN pizzas 
		ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN order_details 
            ON order_details.pizza_id = pizzas.pizza_id
        GROUP BY pizza_types.category, pizza_types.name ) AS a ) AS b
		WHERE rn <= 3;

