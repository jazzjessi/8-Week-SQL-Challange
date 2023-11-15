### How many unique nodes are there on the Data Bank system?
``` sql
SELECT COUNT(DISTINCT node_id) AS unique_nodes
FROM data_bank.customer_nodes;
```
| unique_nodes |
| ------------ |
| 5            |
---
### What is the number of nodes per region?
``` sql
SELECT region_id, region_name, COUNT(DISTINCT node_id) AS nodes_per_region
FROM data_bank.customer_nodes
JOIN data_bank.regions USING(region_id)
GROUP BY region_id, region_name
ORDER BY region_id;
```
| region_id | region_name | nodes_per_region |
| --------- | ----------- | ---------------- |
| 1         | Australia   | 5                |
| 2         | America     | 5                |
| 3         | Africa      | 5                |
| 4         | Asia        | 5                |
| 5         | Europe      | 5                |

---
### How many customers are allocated to each region?
``` sql
SELECT region_id, region_name, COUNT(customer_id) AS customer_per_region
FROM data_bank.customer_nodes
JOIN data_bank.regions USING(region_id)
GROUP BY region_id, region_name
ORDER BY region_id;
```
| region_id | region_name | customer_per_region |
| --------- | ----------- | ------------------- |
| 1         | Australia   | 770                 |
| 2         | America     | 735                 |
| 3         | Africa      | 714                 |
| 4         | Asia        | 665                 |
| 5         | Europe      | 616                 |

---
### How many days on average are customers reallocated to a different node?
#### I started by looking for the previous and next nodes, so I can see, if they are reallocated to a different node or to the same node again. For customer 1 'cte_previous_next' looks like that
| customer_id | region_id | node_id | original_start           | original_end             | prev_node | prev_start               | next_node | next_end                 |
| ----------- | --------- | ------- | ------------------------ | ------------------------ | --------- | ------------------------ | --------- | ------------------------ |
| 1           | 3         | 4       | 2020-01-02T00:00:00.000Z | 2020-01-03T00:00:00.000Z |           |                          | 4         | 2020-01-14T00:00:00.000Z |
| 1           | 3         | 4       | 2020-01-04T00:00:00.000Z | 2020-01-14T00:00:00.000Z | 4         | 2020-01-02T00:00:00.000Z | 2         | 2020-01-16T00:00:00.000Z |
| 1           | 3         | 2       | 2020-01-15T00:00:00.000Z | 2020-01-16T00:00:00.000Z | 4         | 2020-01-04T00:00:00.000Z | 5         | 2020-01-28T00:00:00.000Z |
| 1           | 3         | 5       | 2020-01-17T00:00:00.000Z | 2020-01-28T00:00:00.000Z | 2         | 2020-01-15T00:00:00.000Z | 3         | 2020-02-18T00:00:00.000Z |
| 1           | 3         | 3       | 2020-01-29T00:00:00.000Z | 2020-02-18T00:00:00.000Z | 5         | 2020-01-17T00:00:00.000Z | 2         | 2020-03-16T00:00:00.000Z |
| 1           | 3         | 2       | 2020-02-19T00:00:00.000Z | 2020-03-16T00:00:00.000Z | 3         | 2020-01-29T00:00:00.000Z | 2         | 9999-12-31T00:00:00.000Z |
| 1           | 3         | 2       | 2020-03-17T00:00:00.000Z | 9999-12-31T00:00:00.000Z | 2         | 2020-02-19T00:00:00.000Z |           |                          |

#### If it is the latter case, we want to change the start and end dates with 'cte_different_customer_nodes'
| customer_id | region_id | node_id | start_date               | end_date                 |
| ----------- | --------- | ------- | ------------------------ | ------------------------ |
| 1           | 3         | 4       | 2020-01-02T00:00:00.000Z | 2020-01-14T00:00:00.000Z |
| 1           | 3         | 4       | 2020-01-02T00:00:00.000Z | 2020-01-14T00:00:00.000Z |
| 1           | 3         | 2       | 2020-01-15T00:00:00.000Z | 2020-01-16T00:00:00.000Z |
| 1           | 3         | 5       | 2020-01-17T00:00:00.000Z | 2020-01-28T00:00:00.000Z |
| 1           | 3         | 3       | 2020-01-29T00:00:00.000Z | 2020-02-18T00:00:00.000Z |
| 1           | 3         | 2       | 2020-02-19T00:00:00.000Z | 9999-12-31T00:00:00.000Z |
| 1           | 3         | 2       | 2020-02-19T00:00:00.000Z | 9999-12-31T00:00:00.000Z |

#### Now we get rid of the duplicates and calculate the day difference with 'cte_day_diff'
| customer_id | region_id | node_id | start_date               | end_date                 | day_diff |
| ----------- | --------- | ------- | ------------------------ | ------------------------ | -------- |
| 1           | 3         | 4       | 2020-01-02T00:00:00.000Z | 2020-01-14T00:00:00.000Z | 13       |
| 1           | 3         | 2       | 2020-01-15T00:00:00.000Z | 2020-01-16T00:00:00.000Z | 2        |
| 1           | 3         | 5       | 2020-01-17T00:00:00.000Z | 2020-01-28T00:00:00.000Z | 12       |
| 1           | 3         | 3       | 2020-01-29T00:00:00.000Z | 2020-02-18T00:00:00.000Z | 21       |
#### Finally we can calculate the average difference
``` sql
WITH cte_previous_next AS
    (SELECT customer_id, region_id, node_id, start_date AS original_start, end_date AS original_end,
     LAG(node_id) OVER(PARTITION BY customer_id ORDER BY customer_id, start_date) AS prev_node,
     LAG(start_date) OVER(PARTITION BY customer_id ORDER BY customer_id, start_date) AS prev_start,
     LEAD(node_id) OVER(PARTITION BY customer_id ORDER BY customer_id, start_date) AS next_node,
     LEAD(end_date) OVER(PARTITION BY customer_id ORDER BY customer_id, start_date) AS next_end
     FROM data_bank.customer_nodes
     ORDER BY customer_id, start_date
    ),
    cte_different_customer_nodes AS
    (SELECT customer_id, region_id, node_id,
     CASE WHEN prev_node = node_id THEN prev_start ELSE original_start END AS start_date,
     CASE WHEN next_node = node_id THEN next_end ELSE original_end END AS end_date
     FROM cte_previous_next 
    ),
    cte_day_diff AS
    (SELECT DISTINCT * , end_date - start_date + 1 AS day_diff
     FROM cte_different_customer_nodes
     WHERE end_date != '9999-12-31' 
     ORDER BY customer_id, start_date
    )
SELECT ROUND(AVG(day_diff)) AS avg_reallocation
FROM cte_day_diff;
```
| avg_reallocation |
| ---------------- |
| 20               |

---
### What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
#### I didn't quite understand the question, maybe they wantet the average reallocation per customer in the previous question. With that we can do it.
```sql
WITH cte_previous_next AS
    (SELECT customer_id, region_id, node_id, start_date AS original_start, end_date AS original_end,
     LAG(node_id) OVER(PARTITION BY customer_id ORDER BY customer_id, start_date) AS prev_node,
     LAG(start_date) OVER(PARTITION BY customer_id ORDER BY customer_id, start_date) AS prev_start,
     LEAD(node_id) OVER(PARTITION BY customer_id ORDER BY customer_id, start_date) AS next_node,
     LEAD(end_date) OVER(PARTITION BY customer_id ORDER BY customer_id, start_date) AS next_end
     FROM data_bank.customer_nodes
     ORDER BY customer_id, start_date
    ),
    cte_different_customer_nodes AS
    (SELECT customer_id, region_id, node_id,
     CASE WHEN prev_node = node_id THEN prev_start ELSE original_start END AS start_date,
     CASE WHEN next_node = node_id THEN next_end ELSE original_end END AS end_date
     FROM cte_previous_next 
    ),
    cte_day_diff AS
    (SELECT DISTINCT * , end_date - start_date + 1 AS day_diff
     FROM cte_different_customer_nodes
     WHERE end_date != '9999-12-31' 
     ORDER BY customer_id, start_date
    ),
    cte_reallocation AS
    (SELECT 
     ROW_NUMBER() OVER(ORDER BY AVG(day_diff)),
     customer_id, ROUND(AVG(day_diff)) AS avg_reallocation
     FROM cte_day_diff
     GROUP BY customer_id
     ORDER BY avg_reallocation
    ),
    cte_count AS
    (SELECT COUNT(*) AS count_
     FROM cte_reallocation
    )
SELECT 
    MAX(avg_reallocation) FILTER(WHERE row_number = count_*0.5) AS median,
    MAX(avg_reallocation) FILTER(WHERE row_number = count_*0.8) AS "80th percentile",
    MAX(avg_reallocation) FILTER(WHERE row_number = count_*0.95) AS "95th percentile"
FROM cte_reallocation, cte_count;
```
| median | 80th percentile | 95th percentile |
| ------ | --------------- | --------------- |
| 19     | 24              | 33              |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2GtQz4wZtuNNu7zXH5HtV4/3)
