The data was Pizzahut, India sales for the year 2015

The following questions were tackeled using SQL in this project

1. Retrieve the total number of orders placed.
2. Calculate the total revenue generated from pizza sales.
3. Identify the highest-priced pizza.
4. Identify the most common pizza size ordered.
5. List the top 5 most ordered pizza types along with their quantities.
6. Join the necessary tables to find the total quantity of each pizza category ordered.
7. Determine the distribution of orders by hour of the day.
8. Join relevant tables to find the category-wise distribution of pizzas.
9. Group the orders by date and calculate the average number of pizzas ordered per day.
10. Determine the top 3 most ordered pizza types based on revenue.
11. Calculate the percentage contribution of each pizza type to total revenue.
12. Analyze the cumulative revenue generated over time.
13. Determine the top 3 most ordered pizza types based on revenue for each pizza category.

# Below is the Code:

## -- Retrieve the total number of orders placed.

select count(order_id) as total_orders

from orders;

## -- Calculate the total revenue generated from pizza sales.

SELECT 

    ROUND(SUM(order_details.quantity * pizzas.price),
    
            2) AS total_revenue
FROM

    order_details
        
        JOIN
   
    pizzas ON order_details.pizza_id = pizzas.pizza_id;
    
## -- Identify the highest-priced pizza.

SELECT 
   
    pizza_types.name, pizzas.price

FROM
   
    pizza_types
    
        JOIN
   
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id

ORDER BY pizzas.price DESC

LIMIT 1;

## -- List the top 5 most ordered pizza types along with their quantities.

SELECT 
   
    pizza_types.name, SUM(order_details.quantity) AS quantity

FROM

    pizza_types
    
        JOIN
    
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
    
        JOIN
    
    order_details ON order_details.pizza_id = pizzas.pizza_id

GROUP BY pizza_types.name

ORDER BY quantity DESC

LIMIT 5; 

## -- Join the necessary tables to find the total quantity of each pizza category ordered.

SELECT 

    pizza_types.category,
    
    SUM(order_details.quantity) AS quantity

FROM
   
    pizza_types
    
        JOIN
    
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
    
        JOIN
    
    order_details ON order_details.pizza_id = pizzas.pizza_id

GROUP BY pizza_types.category

ORDER BY quantity DESC;

## -- Determine the distribution of orders by hour of the day.

SELECT 

    HOUR(order_time), COUNT(order_id) AS Count_of_orders

FROM

    orders

GROUP BY HOUR(order_time);

## -- Join relevant tables to find the category-wise distribution of pizzas.

select category, count(name)

 from pizza_types
 
 group by category;
 
##  -- Group the orders by date and calculate the average number of pizzas ordered per day

SELECT 
    
    ROUND(AVG(quantity), 0) AS avg_quantity_per_day

FROM

    (SELECT 
    
        orders.order_date, SUM(order_details.quantity) AS quantity
    
    FROM
        
        orders
    
    JOIN order_details ON orders.order_id = order_details.order_id
    
    GROUP BY orders.order_date) AS order_quantity;

## -- Determine the top 3 most ordered pizza types based on revenue.

select pizza_types.name,

sum(order_details.quantity * pizzas.price) as revenue

from pizza_types join pizzas

on pizza_types.pizza_type_id = pizzas.pizza_type_id

join order_details

on order_details.pizza_id = pizzas.pizza_id

group by pizza_types.name order by revenue desc limit 3;

## -- Calculate the percentage contribution of each pizza type to total revenue.

select pizza_types.category,

CONCAT(round(sum(order_details.quantity * pizzas.price)/ (SELECT 

    ROUND(SUM(order_details.quantity * pizzas.price),
    
            2) AS total_revenue

FROM

    order_details
    
        JOIN
    
    pizzas ON order_details.pizza_id = pizzas.pizza_id)*100, 2) , '%') as revenue 

from pizza_types join pizzas

on pizza_types.pizza_type_id = pizzas.pizza_type_id

join order_details

on order_details.pizza_id = pizzas.pizza_id

group by pizza_types.category order by revenue desc limit 3;

## -- Analyze the cumulative revenue generated over time.

select order_date,

sum(revenue) over(order by order_date) as cumulative_revenue

from

(select orders.order_date,

sum(order_details.quantity * pizzas.price) as revenue

from order_details join pizzas

on order_details.pizza_id = pizzas.pizza_id

join orders

on orders.order_id = order_details.order_id

group by orders.order_date) as sales;

## -- Determine the top 3 most ordered pizza types based on revenue for each pizza category.

select category, name, revenue

from

(select category, name , revenue,

rank() over (partition by category order by revenue desc) as rn

from

(select pizza_types.category, pizza_types.name,

round(sum(order_details.quantity * pizzas.price),2) as revenue

from pizza_types join pizzas

on pizza_types.pizza_type_id = pizzas.pizza_type_id

join order_details

on order_details.pizza_id = pizzas.pizza_id

group by pizza_types.category, pizza_types.name) as a) as b

where rn <= 3;
