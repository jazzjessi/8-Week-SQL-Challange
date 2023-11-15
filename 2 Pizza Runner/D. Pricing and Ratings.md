### If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has  Pizza Runner made so far if there are no delivery fees?
``` sql
    WITH amount_pizza AS
    (
      SELECT pizza_id, pizza_name, COUNT(pizza_id) AS count_pizza
      FROM pizza_runner.customer_orders
      JOIN pizza_runner.pizza_names USING(pizza_id)
      JOIN pizza_runner.runner_orders USING(order_id)
      WHERE cancellation IS NULL
      GROUP BY pizza_id, pizza_name
    )
    SELECT SUM(CASE WHEN pizza_id = 1 THEN count_pizza *12 ELSE count_pizza *10 END) AS profit
    FROM amount_pizza;
```
| profit |
| ------ |
| 138    |

---
### What if there was an additional $1 charge for any pizza extras? 
- Add cheese is $1 extra
``` sql
    WITH amount_extras AS
    (
      SELECT unique_pizza_id, COUNT(extras) AS count_extra
      FROM pizza_runner.pizza_extras
      JOIN pizza_runner.customer_orders USING(unique_pizza_id)
      GROUP BY unique_pizza_id
    ),
    amount_pizza AS
    (
      SELECT pizza_id, pizza_name, COUNT(pizza_id) AS count_pizza
      FROM pizza_runner.customer_orders
      JOIN pizza_runner.pizza_names USING(pizza_id)
      JOIN pizza_runner.runner_orders USING(order_id)
      WHERE cancellation IS NULL
      GROUP BY pizza_id, pizza_name
    )
    SELECT 
    (SELECT SUM(CASE WHEN pizza_id = 1 THEN count_pizza *12 ELSE count_pizza *10 END) 
    FROM amount_pizza)
    +
    (SELECT SUM(count_extra) 
    FROM amount_extras) AS profit_with_additional_charge;
```
| profit_with_additional_charge |
| ----------------------------- |
| 144                           |

---
### The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
``` sql
    CREATE SCHEMA rating;

    SET search_path = rating;

    DROP TABLE IF EXISTS runner_ratings;

    CREATE TABLE runner_ratings AS
    	SELECT DISTINCT runner_id, customer_id, order_id
        FROM pizza_runner.runner_orders
        JOIN pizza_runner.customer_orders USING(order_id)
        WHERE cancellation IS null;

    ALTER TABLE runner_ratings
    ADD rating INTEGER;

    UPDATE rating.runner_ratings
    SET rating = CASE WHEN customer_id = 101 THEN  4
    		 			WHEN customer_id = 102 THEN  3
            			WHEN customer_id = 103 THEN  5
             			WHEN customer_id = 104 THEN  2
             			END;

```
---
### Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
- customer_id
- order_id
- runner_id
- rating
- order_time
- pickup_time
- Time between order and pickup
- Delivery duration
- Average speed
- Total number of pizzas
``` sql
    SELECT 
     customer_id, 
     order_id, 
     runner_id, 
     rating, 
     order_time, 
     pickup_time,
     DATE_PART('minute', pickup_time - order_time) AS Time_between_order_and_pickup,
     duration AS Delivery_duration,
     ROUND(distance/duration *60,2) AS Average_speed_in_km_per_hour,
     COUNT(unique_pizza_id) AS Total_number_of_pizzas
    FROM pizza_runner.customer_orders
    JOIN pizza_runner.runner_orders USING(order_id)
    JOIN rating.runner_ratings USING(order_id, customer_id, runner_id)
    GROUP BY customer_id, 
     order_id, 
     runner_id, 
     rating, 
     order_time, 
     pickup_time,
     Time_between_order_and_pickup,
     Delivery_duration,
     Average_speed_in_km_per_hour
    ORDER BY runner_id, customer_id, order_id;
```
| customer_id | order_id | runner_id | rating | order_time               | pickup_time              | time_between_order_and_pickup | delivery_duration | average_speed_in_km_per_hour | total_number_of_pizzas |
| ----------- | -------- | --------- | ------ | ------------------------ | ------------------------ | ----------------------------- | ----------------- | ---------------------------- | ---------------------- |
| 101         | 1        | 1         | 4      | 2020-01-01T18:05:02.000Z | 2020-01-01T18:15:34.000Z | 10                            | 32                | 37.50                        | 1                      |
| 101         | 2        | 1         | 4      | 2020-01-01T19:00:52.000Z | 2020-01-01T19:10:54.000Z | 10                            | 27                | 44.44                        | 1                      |
| 102         | 3        | 1         | 3      | 2020-01-02T23:51:23.000Z | 2020-01-03T00:12:37.000Z | 21                            | 20                | 40.20                        | 2                      |
| 104         | 10       | 1         | 2      | 2020-01-11T18:34:49.000Z | 2020-01-11T18:50:20.000Z | 15                            | 10                | 60.00                        | 2                      |
| 102         | 8        | 2         | 3      | 2020-01-09T23:54:33.000Z | 2020-01-10T00:15:02.000Z | 20                            | 15                | 93.60                        | 1                      |
| 103         | 4        | 2         | 5      | 2020-01-04T13:23:46.000Z | 2020-01-04T13:53:03.000Z | 29                            | 40                | 35.10                        | 3                      |
| 105         | 7        | 2         |        | 2020-01-08T21:20:29.000Z | 2020-01-08T21:30:45.000Z | 10                            | 25                | 60.00                        | 1                      |
| 104         | 5        | 3         | 2      | 2020-01-08T21:00:29.000Z | 2020-01-08T21:10:57.000Z | 10                            | 15                | 40.00                        | 1                      |

---
### If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
``` sql
    WITH amount_pizza AS
    (
      SELECT pizza_id, pizza_name, COUNT(pizza_id) AS count_pizza
      FROM pizza_runner.customer_orders
      JOIN pizza_runner.pizza_names USING(pizza_id)
      JOIN pizza_runner.runner_orders USING(order_id)
      WHERE cancellation IS NULL
      GROUP BY pizza_id, pizza_name
    ),
    payment AS
    (
      SELECT SUM(distance*0.3)
      FROM pizza_runner.runner_orders
    ),
    profit AS
    (
      SELECT SUM(CASE WHEN pizza_id = 1 THEN count_pizza *12 ELSE count_pizza *10 END)
      FROM amount_pizza
    )
    SELECT
    (SELECT * from profit)-(SELECT * from payment) AS final_profit;
```
| final_profit |
| ------------ |
| 94.44        |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)
