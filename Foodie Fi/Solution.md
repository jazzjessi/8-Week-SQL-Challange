# A. Customer Journey

### Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey. Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!
``` sql
SELECT customer_id, plan_name, start_date 
FROM foodie_fi.plans
JOIN foodie_fi.subscriptions USING(plan_id)
WHERE customer_id between 1 and 8
ORDER BY customer_id, start_date;
```
| customer_id | plan_name     | start_date               |
| ----------- | ------------- | ------------------------ |
| 1           | trial         | 2020-08-01T00:00:00.000Z |
| 1           | basic monthly | 2020-08-08T00:00:00.000Z |
| 2           | trial         | 2020-09-20T00:00:00.000Z |
| 2           | pro annual    | 2020-09-27T00:00:00.000Z |
| 3           | trial         | 2020-01-13T00:00:00.000Z |
| 3           | basic monthly | 2020-01-20T00:00:00.000Z |
| 4           | trial         | 2020-01-17T00:00:00.000Z |
| 4           | basic monthly | 2020-01-24T00:00:00.000Z |
| 4           | churn         | 2020-04-21T00:00:00.000Z |
| 5           | trial         | 2020-08-03T00:00:00.000Z |
| 5           | basic monthly | 2020-08-10T00:00:00.000Z |
| 6           | trial         | 2020-12-23T00:00:00.000Z |
| 6           | basic monthly | 2020-12-30T00:00:00.000Z |
| 6           | churn         | 2021-02-26T00:00:00.000Z |
| 7           | trial         | 2020-02-05T00:00:00.000Z |
| 7           | basic monthly | 2020-02-12T00:00:00.000Z |
| 7           | pro monthly   | 2020-05-22T00:00:00.000Z |
| 8           | trial         | 2020-06-11T00:00:00.000Z |
| 8           | basic monthly | 2020-06-18T00:00:00.000Z |
| 8           | pro monthly   | 2020-08-03T00:00:00.000Z |

---
---
# B. Data Analysis Questions
### How many customers has Foodie-Fi ever had?
``` sql
SELECT count(DISTINCT customer_id) AS amount_customers
FROM foodie_fi.subscriptions;
```
| amount_customers |
| ---------------- |
| 1000             |

---
### What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value (also distributes months per year, was not asked in the question, no one startet a trial after 2020)
``` sql
SELECT DISTINCT ON(date_part('year',start_date), date_part('month',start_date))
    date_part('year',start_date) AS year,
    to_char(start_date,'month') AS month, 
    COUNT(start_date) AS new_trial_per_month
FROM foodie_fi.subscriptions
WHERE plan_id = 0
GROUP BY date_part('month',start_date), date_part('year',start_date), to_char(start_date,'month')
ORDER BY date_part('year',start_date), date_part('month',start_date);
```
| year | month     | new_trial_per_month |
| ---- | --------- | ------------------- |
| 2020 | january   | 88                  |
| 2020 | february  | 68                  |
| 2020 | march     | 94                  |
| 2020 | april     | 81                  |
| 2020 | may       | 88                  |
| 2020 | june      | 79                  |
| 2020 | july      | 89                  |
| 2020 | august    | 88                  |
| 2020 | september | 87                  |
| 2020 | october   | 79                  |
| 2020 | november  | 75                  |
| 2020 | december  | 84                  |

---
### What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name
``` sql
SELECT plan_name, count(plan_id) AS amount
FROM foodie_fi.plans
JOIN foodie_fi.subscriptions USING(plan_id)
WHERE date_part('year',start_date) > 2020
GROUP BY plan_id, plan_name
ORDER BY plan_id;
```
| plan_name     | amount |
| ------------- | ------ |
| basic monthly | 8      |
| pro monthly   | 60     |
| pro annual    | 63     |
| churn         | 71     |

---
### What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
``` sql
SELECT COUNT(customer_id) AS customer_count, 
    ROUND(COUNT(customer_id) / (SELECT COUNT(DISTINCT customer_id)::numeric FROM foodie_fi.subscriptions) * 100, 1) AS percentage
FROM foodie_fi.subscriptions 
WHERE plan_id = 4;
```
| customer_count | percentage |
| -------------- | ---------- |
| 307            | 30.7       |

---
### How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
``` sql
SELECT COUNT(customer_id) AS churn_after_trial, 
    ROUND(COUNT(customer_id)/ (SELECT COUNT(DISTINCT customer_id)::numeric  FROM foodie_fi.subscriptions) * 100) AS percentage
FROM foodie_fi.subscriptions 
WHERE plan_id = 4
AND customer_id NOT IN
    (SELECT customer_id 
     FROM foodie_fi.subscriptions
     WHERE plan_id IN(1,2,3)
    );
```
| churn_after_trial | percentage |
| ----------------- | ---------- |
| 92                | 9          |

---
### What is the number and percentage of customer plans after their initial free trial?
``` sql
SELECT plan_name, COUNT(customer_id) AS customer_plans, 
    ROUND(COUNT(customer_id) / (SELECT COUNT(DISTINCT customer_id)::numeric FROM foodie_fi.subscriptions) * 100, 1) AS percentage
FROM foodie_fi.subscriptions 
JOIN foodie_fi.plans USING(plan_id)
WHERE (customer_id, start_date) IN
    (SELECT customer_id, start_date + 7
     FROM foodie_fi.subscriptions
     WHERE plan_id = 0)
GROUP BY plan_id, plan_name
ORDER BY plan_id;
```
| plan_name     | customer_plans | percentage |
| ------------- | -------------- | ---------- |
| basic monthly | 546            | 54.6       |
| pro monthly   | 325            | 32.5       |
| pro annual    | 37             | 3.7        |
| churn         | 92             | 9.2        |

---
### What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
``` sql
WITH cte_next_date AS (
    SELECT *,
           LEAD(start_date, 1) OVER(PARTITION BY customer_id ORDER BY start_date) AS next_date
    FROM foodie_fi.subscriptions
    WHERE start_date <= '2020-12-31'
    ),
    cte_plans_breakdown AS (
    SELECT plan_id,
           COUNT(DISTINCT customer_id)::numeric) AS total,
           (SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions) AS total_all
    FROM cte_next_date c
    WHERE next_date IS NULL
    GROUP BY plan_id
    )
    SELECT plan_name, 
           total, 
           ROUND(total / total_all * 100, 1) AS percentage
    FROM cte_plans_breakdown 
    LEFT JOIN foodie_fi.plans USING(plan_id)
    ORDER BY plan_id;
```
| plan_name     | total | percentage |
| ------------- | ----- | ---------- |
| trial         | 19    | 1.9        |
| basic monthly | 224   | 22.4       |
| pro monthly   | 326   | 32.6       |
| pro annual    | 195   | 19.5       |
| churn         | 236   | 23.6       |

---
### How many customers have upgraded to an annual plan in 2020?
``` sql
SELECT COUNT(customer_id) AS upgraded_customers
FROM foodie_fi.subscriptions
WHERE start_date BETWEEN '2020-01-01' AND '2020-12-31'
AND plan_id = 3;
```
| upgraded_customers |
| ------------------ |
| 195                |

---
### How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
``` sql
WITH cte_annual_start AS
    (SELECT customer_id, start_date AS annual_start
     FROM foodie_fi.subscriptions
     WHERE plan_id = 3
    ),
    cte_first_join AS
    (SELECT customer_id, start_date AS first_join
     FROM foodie_fi.subscriptions
     WHERE plan_id = 0
    )
SELECT ROUND(AVG(annual_start - first_join)) AS avg_days_to_upgrade
FROM cte_annual_start
JOIN cte_first_join USING(customer_id);
```
| avg_days_to_upgrade |
| ------------------- |
| 105                 |

---
### Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
``` sql
WITH cte_annual_start AS
    (SELECT customer_id, start_date AS annual_start
     FROM foodie_fi.subscriptions
     WHERE plan_id = 3
    ),
    cte_first_join AS
    (SELECT customer_id, start_date AS first_join
     FROM foodie_fi.subscriptions
     WHERE plan_id = 0
    )
SELECT
    CONCAT(FLOOR((annual_start - first_join)/30)*30, ' - ', FLOOR((annual_start - first_join)/30)*30 + 30, ' days') AS period,
    COUNT(customer_id) AS total_amount_of_customers,
    ROUND(AVG(annual_start - first_join)) AS avg_days_to_upgrade
FROM cte_annual_start
JOIN cte_first_join USING(customer_id)
GROUP BY FLOOR((annual_start - first_join)/30)
ORDER BY FLOOR((annual_start - first_join)/30);
```
| period         | total_amount_of_customers | avg_days_to_upgrade |
| -------------- | ------------------------- | ------------------- |
| 0 - 30 days    | 48                        | 10                  |
| 30 - 60 days   | 25                        | 42                  |
| 60 - 90 days   | 33                        | 71                  |
| 90 - 120 days  | 35                        | 100                 |
| 120 - 150 days | 43                        | 133                 |
| 150 - 180 days | 35                        | 162                 |
| 180 - 210 days | 27                        | 190                 |
| 210 - 240 days | 4                         | 224                 |
| 240 - 270 days | 5                         | 257                 |
| 270 - 300 days | 1                         | 285                 |
| 300 - 330 days | 1                         | 327                 |
| 330 - 360 days | 1                         | 346                 |

---
### How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
``` sql
WITH cte_pro_monthly AS
    (SELECT customer_id, start_date
     FROM foodie_fi.subscriptions
     WHERE plan_id = 2 AND DATE_PART('year',start_date) = '2020'
    ),
     cte_basic_monthly AS
    (SELECT customer_id, start_date
     FROM foodie_fi.subscriptions
     WHERE plan_id = 1 AND DATE_PART('year',start_date) = '2020'
    )
SELECT COUNT(customer_id) AS downgrade_from_pro_to_basic
FROM cte_pro_monthly AS pro
JOIN cte_basic_monthly AS basic USING(customer_id)
WHERE pro.start_date < basic.start_date;
```
| downgrade_from_pro_to_basic |
| --------------------------- |
| 0                           |

---
---
# C. Challenge Payment Question
### The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements:
- monthly payments always occur on the same day of month as the original start_date of any monthly paid plan
- upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
- upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
- once a customer churns they will no longer make payments
``` sql
WITH cte_end_dates AS 
    (SELECT customer_id, plan_id, start_date AS subscription_start,
     LEAD(plan_id) OVER(PARTITION BY customer_id ORDER BY customer_id, start_date) AS next_plan, 
     LEAD(start_date) OVER(PARTITION BY customer_id ORDER BY customer_id, start_date) AS end_date,
     LAG(start_date) OVER(PARTITION BY customer_id ORDER BY customer_id, start_date) AS prev_start
     FROM foodie_fi.subscriptions
    ),
    cte AS
    (SELECT customer_id, plan_id, plan_name, subscription_start,
     CASE WHEN plan_id = 3 THEN prev_start + ((start_date - prev_start)/30 + 1) * interval '1 month'
     	  ELSE start_date
     	  END AS start_date,
     CASE WHEN end_date IS not null THEN end_date  
     	  ELSE '2020-12-31' 
      	  END AS end_date,
     CASE WHEN plan_id = 3 THEN interval '1 year' 
     	  ELSE interval '1 month' 
     	  END AS period,
     price AS amount
     FROM foodie_fi.subscriptions
     JOIN foodie_fi.plans USING(plan_id)
     JOIN cte_end_dates USING(customer_id, plan_id)
    ),
    cte_payment AS
    (SELECT customer_id, plan_id, plan_name, 
     generate_series(start_date, end_date, period) AS payment_date,
     amount 
     FROM cte
     ORDER BY customer_id, start_date
    )
SELECT customer_id, plan_id, plan_name, payment_date, amount, 
     RANK() OVER(PARTITION BY customer_id ORDER BY customer_id, payment_date) AS payment_order
FROM cte_payment
WHERE plan_id NOT IN (0,4)
AND customer_id BETWEEN 1 AND 20;
```
#### I only showed the first 20 customers

| customer_id | plan_id | plan_name     | payment_date             | amount | payment_order |
| ----------- | ------- | ------------- | ------------------------ | ------ | ------------- |
| 1           | 1       | basic monthly | 2020-08-08T00:00:00.000Z | 9.90   | 1             |
| 1           | 1       | basic monthly | 2020-09-08T00:00:00.000Z | 9.90   | 2             |
| 1           | 1       | basic monthly | 2020-10-08T00:00:00.000Z | 9.90   | 3             |
| 1           | 1       | basic monthly | 2020-11-08T00:00:00.000Z | 9.90   | 4             |
| 1           | 1       | basic monthly | 2020-12-08T00:00:00.000Z | 9.90   | 5             |
| 2           | 3       | pro annual    | 2020-10-20T00:00:00.000Z | 199.00 | 1             |
| 3           | 1       | basic monthly | 2020-01-20T00:00:00.000Z | 9.90   | 1             |
| 3           | 1       | basic monthly | 2020-02-20T00:00:00.000Z | 9.90   | 2             |
| 3           | 1       | basic monthly | 2020-03-20T00:00:00.000Z | 9.90   | 3             |
| 3           | 1       | basic monthly | 2020-04-20T00:00:00.000Z | 9.90   | 4             |
| 3           | 1       | basic monthly | 2020-05-20T00:00:00.000Z | 9.90   | 5             |
| 3           | 1       | basic monthly | 2020-06-20T00:00:00.000Z | 9.90   | 6             |
| 3           | 1       | basic monthly | 2020-07-20T00:00:00.000Z | 9.90   | 7             |
| 3           | 1       | basic monthly | 2020-08-20T00:00:00.000Z | 9.90   | 8             |
| 3           | 1       | basic monthly | 2020-09-20T00:00:00.000Z | 9.90   | 9             |
| 3           | 1       | basic monthly | 2020-10-20T00:00:00.000Z | 9.90   | 10            |
| 3           | 1       | basic monthly | 2020-11-20T00:00:00.000Z | 9.90   | 11            |
| 3           | 1       | basic monthly | 2020-12-20T00:00:00.000Z | 9.90   | 12            |
| 4           | 1       | basic monthly | 2020-01-24T00:00:00.000Z | 9.90   | 1             |
| 4           | 1       | basic monthly | 2020-02-24T00:00:00.000Z | 9.90   | 2             |
| 4           | 1       | basic monthly | 2020-03-24T00:00:00.000Z | 9.90   | 3             |
| 5           | 1       | basic monthly | 2020-08-10T00:00:00.000Z | 9.90   | 1             |
| 5           | 1       | basic monthly | 2020-09-10T00:00:00.000Z | 9.90   | 2             |
| 5           | 1       | basic monthly | 2020-10-10T00:00:00.000Z | 9.90   | 3             |
| 5           | 1       | basic monthly | 2020-11-10T00:00:00.000Z | 9.90   | 4             |
| 5           | 1       | basic monthly | 2020-12-10T00:00:00.000Z | 9.90   | 5             |
| 6           | 1       | basic monthly | 2020-12-30T00:00:00.000Z | 9.90   | 1             |
| 6           | 1       | basic monthly | 2021-01-30T00:00:00.000Z | 9.90   | 2             |
| 7           | 1       | basic monthly | 2020-02-12T00:00:00.000Z | 9.90   | 1             |
| 7           | 1       | basic monthly | 2020-03-12T00:00:00.000Z | 9.90   | 2             |
| 7           | 1       | basic monthly | 2020-04-12T00:00:00.000Z | 9.90   | 3             |
| 7           | 1       | basic monthly | 2020-05-12T00:00:00.000Z | 9.90   | 4             |
| 7           | 2       | pro monthly   | 2020-05-22T00:00:00.000Z | 19.90  | 5             |
| 7           | 2       | pro monthly   | 2020-06-22T00:00:00.000Z | 19.90  | 6             |
| 7           | 2       | pro monthly   | 2020-07-22T00:00:00.000Z | 19.90  | 7             |
| 7           | 2       | pro monthly   | 2020-08-22T00:00:00.000Z | 19.90  | 8             |
| 7           | 2       | pro monthly   | 2020-09-22T00:00:00.000Z | 19.90  | 9             |
| 7           | 2       | pro monthly   | 2020-10-22T00:00:00.000Z | 19.90  | 10            |
| 7           | 2       | pro monthly   | 2020-11-22T00:00:00.000Z | 19.90  | 11            |
| 7           | 2       | pro monthly   | 2020-12-22T00:00:00.000Z | 19.90  | 12            |
| 8           | 1       | basic monthly | 2020-06-18T00:00:00.000Z | 9.90   | 1             |
| 8           | 1       | basic monthly | 2020-07-18T00:00:00.000Z | 9.90   | 2             |
| 8           | 2       | pro monthly   | 2020-08-03T00:00:00.000Z | 19.90  | 3             |
| 8           | 2       | pro monthly   | 2020-09-03T00:00:00.000Z | 19.90  | 4             |
| 8           | 2       | pro monthly   | 2020-10-03T00:00:00.000Z | 19.90  | 5             |
| 8           | 2       | pro monthly   | 2020-11-03T00:00:00.000Z | 19.90  | 6             |
| 8           | 2       | pro monthly   | 2020-12-03T00:00:00.000Z | 19.90  | 7             |
| 10          | 2       | pro monthly   | 2020-09-26T00:00:00.000Z | 19.90  | 1             |
| 10          | 2       | pro monthly   | 2020-10-26T00:00:00.000Z | 19.90  | 2             |
| 10          | 2       | pro monthly   | 2020-11-26T00:00:00.000Z | 19.90  | 3             |
| 10          | 2       | pro monthly   | 2020-12-26T00:00:00.000Z | 19.90  | 4             |
| 12          | 1       | basic monthly | 2020-09-29T00:00:00.000Z | 9.90   | 1             |
| 12          | 1       | basic monthly | 2020-10-29T00:00:00.000Z | 9.90   | 2             |
| 12          | 1       | basic monthly | 2020-11-29T00:00:00.000Z | 9.90   | 3             |
| 12          | 1       | basic monthly | 2020-12-29T00:00:00.000Z | 9.90   | 4             |
| 13          | 1       | basic monthly | 2020-12-22T00:00:00.000Z | 9.90   | 1             |
| 13          | 1       | basic monthly | 2021-01-22T00:00:00.000Z | 9.90   | 2             |
| 13          | 1       | basic monthly | 2021-02-22T00:00:00.000Z | 9.90   | 3             |
| 13          | 1       | basic monthly | 2021-03-22T00:00:00.000Z | 9.90   | 4             |
| 14          | 1       | basic monthly | 2020-09-29T00:00:00.000Z | 9.90   | 1             |
| 14          | 1       | basic monthly | 2020-10-29T00:00:00.000Z | 9.90   | 2             |
| 14          | 1       | basic monthly | 2020-11-29T00:00:00.000Z | 9.90   | 3             |
| 14          | 1       | basic monthly | 2020-12-29T00:00:00.000Z | 9.90   | 4             |
| 15          | 2       | pro monthly   | 2020-03-24T00:00:00.000Z | 19.90  | 1             |
| 15          | 2       | pro monthly   | 2020-04-24T00:00:00.000Z | 19.90  | 2             |
| 16          | 1       | basic monthly | 2020-06-07T00:00:00.000Z | 9.90   | 1             |
| 16          | 1       | basic monthly | 2020-07-07T00:00:00.000Z | 9.90   | 2             |
| 16          | 1       | basic monthly | 2020-08-07T00:00:00.000Z | 9.90   | 3             |
| 16          | 1       | basic monthly | 2020-09-07T00:00:00.000Z | 9.90   | 4             |
| 16          | 1       | basic monthly | 2020-10-07T00:00:00.000Z | 9.90   | 5             |
| 16          | 3       | pro annual    | 2020-11-07T00:00:00.000Z | 199.00 | 6             |
| 17          | 1       | basic monthly | 2020-08-03T00:00:00.000Z | 9.90   | 1             |
| 17          | 1       | basic monthly | 2020-09-03T00:00:00.000Z | 9.90   | 2             |
| 17          | 1       | basic monthly | 2020-10-03T00:00:00.000Z | 9.90   | 3             |
| 17          | 1       | basic monthly | 2020-11-03T00:00:00.000Z | 9.90   | 4             |
| 17          | 1       | basic monthly | 2020-12-03T00:00:00.000Z | 9.90   | 5             |
| 18          | 2       | pro monthly   | 2020-07-13T00:00:00.000Z | 19.90  | 1             |
| 18          | 2       | pro monthly   | 2020-08-13T00:00:00.000Z | 19.90  | 2             |
| 18          | 2       | pro monthly   | 2020-09-13T00:00:00.000Z | 19.90  | 3             |
| 18          | 2       | pro monthly   | 2020-10-13T00:00:00.000Z | 19.90  | 4             |
| 18          | 2       | pro monthly   | 2020-11-13T00:00:00.000Z | 19.90  | 5             |
| 18          | 2       | pro monthly   | 2020-12-13T00:00:00.000Z | 19.90  | 6             |
| 19          | 2       | pro monthly   | 2020-06-29T00:00:00.000Z | 19.90  | 1             |
| 19          | 2       | pro monthly   | 2020-07-29T00:00:00.000Z | 19.90  | 2             |
| 19          | 2       | pro monthly   | 2020-08-29T00:00:00.000Z | 19.90  | 3             |
| 19          | 3       | pro annual    | 2020-09-29T00:00:00.000Z | 199.00 | 4             |
| 20          | 1       | basic monthly | 2020-04-15T00:00:00.000Z | 9.90   | 1             |
| 20          | 1       | basic monthly | 2020-05-15T00:00:00.000Z | 9.90   | 2             |
| 20          | 3       | pro annual    | 2020-06-15T00:00:00.000Z | 199.00 | 3             |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/rHJhRrXy5hbVBNJ6F6b9gJ/16)
