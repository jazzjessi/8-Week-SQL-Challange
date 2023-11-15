###  How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
``` sql
SELECT DATE_PART('week', registration_date) AS week,
	COUNT(runner_id) AS amount_runners_per_week
FROM pizza_runner.runners
GROUP BY week;
```
| week | amount_runners_per_week |
| ---- | ----------------------- |
| 53   | 2                       |
| 1    | 1                       |
| 2    | 1                       |

---
### What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
``` sql
SELECT runner_id, 
	DATE_PART('minute',AVG(CASE WHEN pickup_time IS NOT null THEN pickup_time::timestamp - order_time::timestamp END)) AS avg_time_per_runner_in_minutes
FROM pizza_runner.customer_orders
JOIN pizza_runner.runner_orders USING(order_id)
GROUP BY runner_id
ORDER BY runner_id;
```
| runner_id | avg_time_per_runner_in_minutes |
| --------- | ------------------------------ |
| 1         | 15                             |
| 2         | 23                             |
| 3         | 10                             |

---
### What was the average distance travelled for each customer?
``` sql
SELECT customer_id, 
	ROUND(AVG(distance),2) AS avg_distance_to_customer
FROM pizza_runner.customer_orders
JOIN pizza_runner.runner_orders USING(order_id)
WHERE distance IS NOT null
GROUP BY customer_id
ORDER BY customer_id;
```
| customer_id | avg_distance_to_customer |
| ----------- | ------------------------ |
| 101         | 20.00                    |
| 102         | 16.73                    |
| 103         | 23.40                    |
| 104         | 10.00                    |
| 105         | 25.00                    |

---
### Is there any relationship between the number of pizzas and how long the order takes to prepare?
---
### What was the difference between the longest and shortest delivery times for all orders?
``` sql
SELECT MAX(duration) - MIN(duration) AS diff_longest_shortest_delivery_time
FROM pizza_runner.customer_orders
JOIN pizza_runner.runner_orders USING(order_id)
WHERE duration IS NOT null;
```
| diff_longest_shortest_delivery_time |
| ----------------------------------- |
| 30                                  |

---
### What was the average speed for each runner for each delivery and do you notice any trend for these values?
``` sql
SELECT runner_id, order_id, ROUND(distance/duration *60,2) AS km_per_hour
FROM pizza_runner.runner_orders
ORDER BY runner_id, order_id;
```
| runner_id | order_id | km_per_hour |
| --------- | -------- | ----------- |
| 1         | 1        | 37.50       |
| 1         | 2        | 44.44       |
| 1         | 3        | 40.20       |
| 1         | 10       | 60.00       |
| 2         | 4        | 35.10       |
| 2         | 7        | 60.00       |
| 2         | 8        | 93.60       |
| 2         | 9        |             |
| 3         | 5        | 40.00       |
| 3         | 6        |             |

---
### What is the successful delivery percentage for each runner?
``` sql
SELECT runner_id, 
    ROUND(COUNT(CASE WHEN distance IS NOT null THEN order_id END)/COUNT(order_id)::numeric * 100,2)
AS successful_delivery_percentage
FROM pizza_runner.runner_orders
GROUP BY runner_id;
```
| runner_id | successful_delivery_percentage |
| --------- | ------------------------------ |
| 3         | 50.00                          |
| 2         | 75.00                          |
| 1         | 100.00                         |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)
