## CUSTOMER_ORDERS
``` sql
UPDATE pizza_runner.customer_orders
SET exclusions = NULL
WHERE exclusions IN ('null', '', 'NaN');

UPDATE pizza_runner.customer_orders
SET extras = NULL
WHERE extras IN ('null', '', 'NaN');

ALTER TABLE pizza_runner.customer_orders 
ADD COLUMN IF NOT EXISTS unique_pizza_id SERIAL PRIMARY KEY;

DROP TABLE IF EXISTS pizza_runner.pizza_exclusions;
CREATE TABLE pizza_runner.pizza_exclusions AS 
    SELECT unique_pizza_id, UNNEST(string_to_array(exclusions, ',')) AS exclusions
    FROM pizza_runner.customer_orders;
  
DROP TABLE IF EXISTS pizza_runner.pizza_extras;
CREATE TABLE pizza_runner.pizza_extras AS 
    SELECT unique_pizza_id, UNNEST(string_to_array(extras, ',')) AS extras
    FROM pizza_runner.customer_orders;
    
ALTER TABLE pizza_runner.pizza_exclusions
ALTER exclusions TYPE int USING exclusions::int;

ALTER TABLE pizza_runner.pizza_extras
ALTER extras TYPE int USING extras::int;

ALTER TABLE customer_orders 
DROP COLUMN IF EXISTS exclusions,
DROP COLUMN IF EXISTS extras;
``` 
---
## RUNNER_ORDERS 
``` sql
UPDATE pizza_runner.runner_orders
SET duration = null
WHERE duration in('', 'null');

UPDATE pizza_runner.runner_orders
SET distance = null
WHERE distance in('', 'null');

UPDATE pizza_runner.runner_orders
SET pickup_time = null
WHERE pickup_time in('', 'null');

UPDATE pizza_runner.runner_orders
SET cancellation = NULL
WHERE cancellation IN ('null', '');

ALTER TABLE pizza_runner.runner_orders
ALTER pickup_time TYPE timestamp USING pickup_time::timestamp,
ALTER distance TYPE numeric USING 
    regexp_replace(distance, '[^0-9\.]', '', 'g')::numeric,
ALTER duration TYPE int USING 
    regexp_replace(duration, '[^0-9]', '', 'g')::int;
``` 
---
## PIZZA_RECIPES 
``` sql
DROP TABLE IF EXISTS pizza_runner.pizza_recipes_old;
DROP TABLE IF EXISTS pizza_runner.pizza_recipes_new;

CREATE TABLE pizza_runner.pizza_recipes_new AS
    SELECT pizza_id, UNNEST(string_to_array(toppings, ','))  AS topping_id
    FROM pizza_runner.pizza_recipes;
    
DROP TABLE pizza_runner.pizza_recipes;
ALTER TABLE pizza_runner.pizza_recipes_new RENAME TO pizza_recipes;

ALTER TABLE pizza_runner.pizza_recipes
ALTER COLUMN topping_id TYPE INT USING topping_id::integer;
``` 
