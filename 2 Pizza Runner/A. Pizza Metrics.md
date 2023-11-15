### How many pizzas were ordered?
``` sql
    SELECT COUNT(*) AS amount_of_pizzas_ordered 
    FROM pizza_runner.customer_orders;
```
| amount_of_pizzas_ordered |
| ------------------------ |
| 14                       |

---
### How many unique customer orders were made?
``` sql
    SELECT COUNT(DISTINCT order_id) AS unique_costumer_orders
    FROM pizza_runner.customer_orders;
```
| unique_costumer_orders |
| ---------------------- |
| 10                     |

---
### How many successful orders were delivered by each runner?
``` sql
    SELECT runner_id, COUNT(duration) AS number_of_successful_orders
    FROM pizza_runner.runner_orders
    GROUP BY runner_id
    ORDER BY runner_id;
```
| runner_id | number_of_successful_orders |
| --------- | --------------------------- |
| 1         | 4                           |
| 2         | 3                           |
| 3         | 1                           |

---
### How many of each type of pizza was delivered?
``` sql
    SELECT pizza_name, COUNT(duration) AS amount_delivered
    FROM pizza_runner.customer_orders
    JOIN pizza_runner.pizza_names USING(pizza_id)
    JOIN pizza_runner.runner_orders USING(order_id)
    GROUP BY pizza_name;
```
| pizza_name | amount_delivered |
| ---------- | ---------------- |
| Meatlovers | 9                |
| Vegetarian | 3                |

---
### How many Vegetarian and Meatlovers were ordered by each customer?
``` sql
    SELECT customer_id, pizza_name, COUNT(pizza_id) AS amount_ordered
    FROM pizza_runner.customer_orders
    JOIN pizza_runner.pizza_names USING(pizza_id)
    GROUP BY customer_id, pizza_name
    ORDER BY customer_id, pizza_name;
```
| customer_id | pizza_name | amount_ordered |
| ----------- | ---------- | -------------- |
| 101         | Meatlovers | 2              |
| 101         | Vegetarian | 1              |
| 102         | Meatlovers | 2              |
| 102         | Vegetarian | 1              |
| 103         | Meatlovers | 3              |
| 103         | Vegetarian | 1              |
| 104         | Meatlovers | 3              |
| 105         | Vegetarian | 1              |

---
### What was the maximum number of pizzas delivered in a single order?
``` sql
    SELECT COUNT(duration) AS maximum_number_of_pizzas_delivered_in_a_single_order
    FROM pizza_runner.customer_orders
    JOIN pizza_runner.runner_orders USING(order_id)
    GROUP BY order_id
    ORDER BY maximum_number_of_pizzas_delivered_in_a_single_order desc limit 1;
```
| maximum_number_of_pizzas_delivered_in_a_single_order |
| ---------------------------------------------------- |
| 3                                                    |

---
### What was the maximum number of pizzas delivered in a single order?
``` sql
    SELECT customer_id, 
    COUNT(CASE WHEN exclusions IS null AND extras IS null THEN pizza_id END) AS no_changes, 
    COUNT(CASE WHEN exclusions IS NOT null OR extras IS NOT null THEN pizza_id END) AS at_least_one_change
    FROM pizza_runner.customer_orders
    JOIN pizza_runner.runner_orders USING(order_id)
    LEFT JOIN pizza_runner.pizza_exclusions USING(unique_pizza_id)
    LEFT JOIN pizza_runner.pizza_extras USING(unique_pizza_id)
    WHERE duration IS NOT null
    GROUP BY customer_id
    ORDER BY customer_id;
```
| customer_id | no_changes | at_least_one_change |
| ----------- | ---------- | ------------------- |
| 101         | 2          | 0                   |
| 102         | 3          | 0                   |
| 103         | 0          | 3                   |
| 104         | 1          | 5                   |
| 105         | 0          | 1                   |

---
### How many pizzas were delivered that had both exclusions and extras?
``` sql
    SELECT COUNT(DISTINCT CASE WHEN exclusions IS NOT null AND extras IS NOT null THEN pizza_id END) AS delivery_with_exclusions_and_extras
    FROM pizza_runner.customer_orders
    JOIN pizza_runner.runner_orders USING(order_id)
    JOIN pizza_runner.pizza_exclusions USING(unique_pizza_id)
    JOIN pizza_runner.pizza_extras USING(unique_pizza_id)
    WHERE duration IS NOT null;
```
| delivery_with_exclusions_and_extras |
| ----------------------------------- |
| 1                                   |

---
### What was the total volume of pizzas ordered for each hour of the day?
``` sql
    SELECT DATE_PART('hour', order_time) AS hours, COUNT(order_id) AS voume_of_pizzas_ordered_per_hour
    FROM pizza_runner.customer_orders
    GROUP BY hours
    ORDER BY hours;
```
| hours | voume_of_pizzas_ordered_per_hour |
| ----- | -------------------------------- |
| 11    | 1                                |
| 13    | 3                                |
| 18    | 3                                |
| 19    | 1                                |
| 21    | 3                                |
| 23    | 3                                |

---
### What was the volume of orders for each day of the week?
``` sql
    SELECT to_char(order_time, 'day') AS weekday, COUNT(DISTINCT order_id) AS volume_of_orders_per_weekday
    FROM pizza_runner.customer_orders
    GROUP BY weekday;
``` 
| weekday   | volume_of_orders_per_weekday |
| --------- | ---------------------------- |
| friday    | 1                            |
| saturday  | 2                            |
| thursday  | 2                            |
| wednesday | 5                            |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)
