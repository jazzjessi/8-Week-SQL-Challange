### What are the standard ingredients for each pizza?
``` sql
    SELECT pizza_name, string_agg(topping_name, ', ') AS standard_ingredients
    FROM pizza_runner.pizza_names
    JOIN pizza_runner.pizza_recipes USING(pizza_id)
    JOIN pizza_runner.pizza_toppings USING(topping_id)
    GROUP BY pizza_name;
```
| pizza_name | standard_ingredients                                                  |
| ---------- | --------------------------------------------------------------------- |
| Meatlovers | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| Vegetarian | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce            |

---
### What was the most commonly added extra?
``` sql
    SELECT DISTINCT ON(COUNT(topping_id)) topping_name AS most_commonly_extra
    FROM pizza_runner.pizza_extras 
    JOIN pizza_runner.pizza_toppings ON extras = topping_id
    GROUP BY topping_name
    ORDER BY COUNT(topping_id) DESC LIMIT 1;
```
| most_commonly_extra |
| ------------------- |
| Bacon               |

---
### What was the most common exclusion?
``` sql
    SELECT DISTINCT ON(COUNT(topping_id)) topping_name AS most_commonly_exclusion
    FROM pizza_runner.pizza_exclusions
    JOIN pizza_runner.pizza_toppings ON exclusions = topping_id
    GROUP BY topping_name
    ORDER BY COUNT(topping_id) DESC LIMIT 1;
```
| most_commonly_exclusion |
| ----------------------- |
| Cheese                  |

---
###  Generate an order item for each record in the customers_orders table in the format of one of the following:
- 	Meat Lovers
- 	Meat Lovers - Exclude Beef
- 	Meat Lovers - Extra Bacon
- 	Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers
``` sql
    WITH order_infos AS
    (SELECT DISTINCT order_id, customer_id, pizza_id, unique_pizza_id, pizza_name, plus.extra, minus.exclusion
    	FROM pizza_runner.customer_orders
    	JOIN pizza_runner.pizza_names USING(pizza_id)
    	JOIN pizza_runner.pizza_recipes USING(pizza_id)
    	LEFT JOIN 
    		(SELECT unique_pizza_id, string_agg(topping_name, ', ') AS extra
    		FROM pizza_runner.pizza_toppings
    		JOIN pizza_runner.pizza_extras ON extras = topping_id
        	GROUP BY unique_pizza_id
            ) AS plus
    	USING(unique_pizza_id)
    	LEFT JOIN 
    		(SELECT unique_pizza_id, string_agg(topping_name, ', ') AS exclusion
    		FROM pizza_runner.pizza_toppings
    		JOIN pizza_runner.pizza_exclusions ON exclusions = topping_id
        	GROUP BY unique_pizza_id
            ) AS minus
    	USING(unique_pizza_id)
    )
    SELECT order_id, customer_id, pizza_id, unique_pizza_id, 
    	   string_agg(pizza_name || concat( 
                      CASE WHEN extra IS NOT null THEN concat(' - EXTRA ', extra) END,
                      CASE WHEN exclusion IS NOT null THEN concat(' - EXCLUDE ', exclusion) END), '') AS order_item
    FROM order_infos
    GROUP BY order_id, customer_id, pizza_id, unique_pizza_id
    ORDER BY order_id;
```
| order_id | customer_id | pizza_id | unique_pizza_id | order_item                                                      |
| -------- | ----------- | -------- | --------------- | --------------------------------------------------------------- |
| 1        | 101         | 1        | 9               | Meatlovers                                                      |
| 2        | 101         | 1        | 10              | Meatlovers                                                      |
| 3        | 102         | 1        | 11              | Meatlovers                                                      |
| 3        | 102         | 2        | 3               | Vegetarian                                                      |
| 4        | 103         | 2        | 8               | Vegetarian - EXCLUDE Cheese                                     |
| 4        | 103         | 1        | 7               | Meatlovers - EXCLUDE Cheese                                     |
| 4        | 103         | 1        | 6               | Meatlovers - EXCLUDE Cheese                                     |
| 5        | 104         | 1        | 4               | Meatlovers - EXTRA Bacon                                        |
| 6        | 101         | 2        | 12              | Vegetarian                                                      |
| 7        | 105         | 2        | 5               | Vegetarian - EXTRA Bacon                                        |
| 8        | 102         | 1        | 13              | Meatlovers                                                      |
| 9        | 103         | 1        | 1               | Meatlovers - EXTRA Bacon, Chicken - EXCLUDE Cheese              |
| 10       | 104         | 1        | 14              | Meatlovers                                                      |
| 10       | 104         | 1        | 2               | Meatlovers - EXTRA Bacon, Cheese - EXCLUDE BBQ Sauce, Mushrooms |

---
### Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients. For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"
``` sql
    WITH info_ingredients AS
    (
      (SELECT order_id, customer_id, pizza_id, unique_pizza_id, topping_id
    	FROM pizza_runner.customer_orders 
    	JOIN pizza_runner.pizza_recipes USING(pizza_id)
    	EXCEPT
    		(SELECT order_id, customer_id, pizza_id, unique_pizza_id, exclusions 
         	FROM pizza_runner.customer_orders
         	JOIN pizza_runner.pizza_exclusions USING(unique_pizza_id)
            )
      )
      UNION ALL
    	(SELECT order_id, customer_id, pizza_id, unique_pizza_id, extras 
         FROM pizza_runner.customer_orders
         JOIN pizza_runner.pizza_extras USING(unique_pizza_id)
        )
      ORDER BY order_id, unique_pizza_id, topping_id
    ),
    duplicates AS
    (
      SELECT unique_pizza_id, topping_id 
      FROM info_ingredients
      GROUP BY unique_pizza_id, topping_id
      HAVING count(*) = 2
    ),
    info_unique_ingredients AS
    (
      SELECT DISTINCT * FROM info_ingredients
    )
    SELECT order_id, customer_id, pizza_id, unique_pizza_id, 
    	   string_agg(
             CASE WHEN (unique_pizza_id, topping_id) IN (SELECT unique_pizza_id, topping_id FROM duplicates)
                  THEN concat(' 2x',topping_name) 
             	  ELSE concat(topping_name) END
             , ', ' ORDER BY topping_name) AS ingredient_list
    FROM info_unique_ingredients
    JOIN pizza_runner.pizza_toppings USING(topping_id)
    GROUP BY order_id, customer_id, pizza_id, unique_pizza_id
    ORDER BY order_id;
```
| order_id | customer_id | pizza_id | unique_pizza_id | ingredient_list                                                          |
| -------- | ----------- | -------- | --------------- | ------------------------------------------------------------------------ |
| 1        | 101         | 1        | 9               | BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 2        | 101         | 1        | 10              | BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 3        | 102         | 1        | 11              | BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 3        | 102         | 2        | 3               | Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes               |
| 4        | 103         | 1        | 6               | BBQ Sauce, Bacon, Beef, Chicken, Mushrooms, Pepperoni, Salami            |
| 4        | 103         | 1        | 7               | BBQ Sauce, Bacon, Beef, Chicken, Mushrooms, Pepperoni, Salami            |
| 4        | 103         | 2        | 8               | Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes                       |
| 5        | 104         | 1        | 4               | BBQ Sauce,  2xBacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| 6        | 101         | 2        | 12              | Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes               |
| 7        | 105         | 2        | 5               | Bacon, Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes        |
| 8        | 102         | 1        | 13              | BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 9        | 103         | 1        | 1               | BBQ Sauce,  2xBacon, Beef,  2xChicken, Mushrooms, Pepperoni, Salami      |
| 10       | 104         | 1        | 2               |  2xBacon, Beef,  2xCheese, Chicken, Pepperoni, Salami                    |
| 10       | 104         | 1        | 14              | BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |

---
### What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?
``` sql
    WITH info_ingredients AS
    (
      (SELECT order_id, customer_id, pizza_id, unique_pizza_id, topping_id
    	FROM pizza_runner.customer_orders 
    	JOIN pizza_runner.pizza_recipes USING(pizza_id)
    	EXCEPT
    		(SELECT order_id, customer_id, pizza_id, unique_pizza_id, exclusions 
         	FROM pizza_runner.customer_orders
         	JOIN pizza_runner.pizza_exclusions USING(unique_pizza_id)
            )
      )
      UNION ALL
    	(SELECT order_id, customer_id, pizza_id, unique_pizza_id, extras 
         FROM pizza_runner.customer_orders
         JOIN pizza_runner.pizza_extras USING(unique_pizza_id)
        )
      ORDER BY order_id, unique_pizza_id, topping_id
    )
    SELECT topping_name, COUNT(topping_id) AS total_quantity_of_each_ingredient
    FROM info_ingredients
    JOIN pizza_runner.pizza_toppings USING(topping_id)
    JOIN pizza_runner.runner_orders USING(order_id)
    WHERE cancellation IS null
    GROUP BY topping_name
    ORDER BY COUNT(topping_id) DESC;
```
| topping_name | total_quantity_of_each_ingredient |
| ------------ | ----- |
| Bacon        | 12    |
| Mushrooms    | 11    |
| Cheese       | 10    |
| Pepperoni    | 9     |
| Salami       | 9     |
| Chicken      | 9     |
| Beef         | 9     |
| BBQ Sauce    | 8     |
| Tomatoes     | 3     |
| Onions       | 3     |
| Peppers      | 3     |
| Tomato Sauce | 3     |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)
