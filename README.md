# First_sql_project


-- Retrieve the total number of orders place 

SELECT 
    COUNT(order_id) AS total_orders
FROM
    orders;


-- Calculate the total revenue generated from pizza sales.

SELECT 
    ROUND(SUM(order_details.quantity * pizzas.price),
            2) AS total_sales
FROM
    order_details
        JOIN
    pizzas ON pizzas.pizza_id = order_details.pizza_id;


-- Identify the highest-priced pizza.

SELECT 
    pizza_types.name, pizzas.price
FROM
    pizza_types
        JOIN
    pizzas ON pizzas.pizza_type_id = pizza_types.pizza_type_id
ORDER BY pizzas.price DESC
LIMIT 1;


-- Identify the most common pizza size ordered.

SELECT 
    pizzas.size, SUM(order_details.quantity) as total_quantity
FROM
    order_details
        JOIN
    pizzas ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizzas.size
order by  SUM(order_details.quantity) desc limit 1 ;


-- List the top 5 most ordered pizza types along with their quantities.

SELECT 
    pizza_types.name,
    SUM(order_details.quantity) AS total_quantity
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizza_types.name
ORDER BY SUM(order_details.quantity) DESC
LIMIT 5;


-- Join the necessary tables to find the total quantity of each pizza category ordered.

SELECT 
    pizza_types.category,
    SUM(order_details.quantity) AS total_quantity
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizza_types.category
ORDER BY total_quantity DESC;


-- Determine the distribution of orders by hour of the day.

SELECT 
    HOUR(order_time), COUNT(order_id) AS ordered
FROM
    orders
GROUP BY HOUR(order_time)
ORDER BY HOUR(order_time);


-- Join relevant tables to find the category-wise distribution of pizzas.

SELECT 
    category, COUNT(name)
FROM
    pizza_types
GROUP BY category;


-- Group the orders by date and calculate the average number of pizzas ordered per day.

SELECT 
    ROUND(AVG(quantity_ordered),2) AS AVG_Pizza_Ordered
FROM
    (SELECT 
        orders.order_date,
            SUM(order_details.quantity) AS quantity_ordered
    FROM
        orders
    JOIN order_details ON orders.order_id = order_details.order_id
    GROUP BY orders.order_date
    ORDER BY orders.order_date) AS quantity_ordered;


-- Determine the top 3 most ordered pizza types based on revenue.

SELECT 
    pizza_types.name,
    SUM(order_details.quantity * pizzas.price) AS revenue
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizza_types.name
ORDER BY revenue DESC
LIMIT 3;


-- Calculate the percentage contribution of each pizza type to total revenue.

SELECT 
    pizza_types.category,
    ROUND(
        SUM(pizzas.price * order_details.quantity) / 
        (
            SELECT 
                ROUND(SUM(order_details.quantity * pizzas.price), 2) 
            FROM 
                order_details
            JOIN pizzas ON pizzas.pizza_id = order_details.pizza_id
        ) * 100, 
        2
    ) AS total_sales_percentage
FROM 
    pizza_types
JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY 
    pizza_types.category;



-- Analyze the cumulative revenue generated over first 10 days

SELECT 
    order_date, 
    ROUND(SUM(revenue) OVER (ORDER BY order_date), 2) AS cumulative_sales
FROM (
    SELECT 
        orders.order_date, 
        SUM(order_details.quantity * pizzas.price) AS revenue
    FROM 
        order_details 
    JOIN pizzas ON order_details.pizza_id = pizzas.pizza_id
    JOIN orders ON orders.order_id = order_details.order_id
    GROUP BY orders.order_date
) AS sales limit 10;



-- Determine the top 3 most ordered pizza types based on revenue for each pizza category.

SELECT 
    category, 
    name AS pizza_name, 
    revenue
FROM (
    SELECT 
        pizza_types.category,
        pizza_types.name, 
        SUM(order_details.quantity * pizzas.price) AS revenue,
        RANK() OVER (PARTITION BY pizza_types.category 
        ORDER BY SUM(order_details.quantity * pizzas.price)DESC)AS rank_position
    FROM 
        pizza_types 
    JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
    JOIN order_details ON order_details.pizza_id = pizzas.pizza_id
    GROUP BY pizza_types.category, pizza_types.name
) AS ranked_pizzas
WHERE rank_position <= 3
ORDER BY category, rank_position;


