### What is the unique count and total amount for each transaction type?
``` sql
SELECT txn_type, sum(txn_amount), count(txn_type)
FROM data_bank.customer_transactions
GROUP BY txn_type;
```
| txn_type   | sum     | count |
| ---------- | ------- | ----- |
| purchase   | 806537  | 1617  |
| deposit    | 1359168 | 2671  |
| withdrawal | 793003  | 1580  |
---
### What is the average total historical deposit counts and amounts for all customers?
``` sql
SELECT count(txn_type) AS total_deposit_counts, ROUND(avg(txn_amount),2) AS avg_amount
FROM data_bank.customer_transactions
WHERE txn_type = 'deposit';
```
| total_deposit_counts | avg_amount |
| -------------------- | ---------- |
| 2671                 | 508.86     |
---
### For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
#### I started by pivoting the table. For the first five customers it looks like this
| customer_id | month_number | month     | total_deposit_counts | total_purchase_counts | total_withdrawal_counts |
| ----------- | ------------ | --------- | -------------------- | --------------------- | ----------------------- |
| 1           | 1            | january   | 1                    | 0                     | 0                       |
| 1           | 3            | march     | 1                    | 2                     | 0                       |
| 2           | 1            | january   | 1                    | 0                     | 0                       |
| 2           | 3            | march     | 1                    | 0                     | 0                       |
| 3           | 1            | january   | 1                    | 0                     | 0                       |
| 3           | 2            | february  | 0                    | 1                     | 0                       |
| 3           | 3            | march     | 0                    | 0                     | 2                       |
| 3           | 4            | april     | 1                    | 0                     | 0                       |
| 4           | 1            | january   | 2                    | 0                     | 0                       |
| 4           | 3            | march     | 0                    | 1                     | 0                       |
| 5           | 1            | january   | 2                    | 0                     | 1                       |
| 5           | 3            | march     | 2                    | 3                     | 2                       |
| 5           | 4            | april     | 0                    | 0                     | 1                       |
#### With that it was easier to calculate
``` sql
WITH cte_txn_types AS
    (SELECT customer_id, DATE_PART('month', txn_date) AS month_number, TO_CHAR(txn_date, 'month') AS month,  
     COUNT(txn_type) FILTER (WHERE txn_type = 'deposit') AS total_deposit_counts,
     COUNT(txn_type) FILTER (WHERE txn_type = 'purchase') AS total_purchase_counts,
     COUNT(txn_type) FILTER (WHERE txn_type = 'withdrawal') AS total_withdrawal_counts 
     FROM data_bank.customer_transactions
     GROUP BY customer_id, month_number, month
    )
SELECT month, COUNT(customer_id) AS more_than_one_deposit_and_either_one_purchase_or_one_withdrawal
FROM cte_txn_types
WHERE (total_deposit_counts > 1)
    AND((total_purchase_counts = 1) OR (total_withdrawal_counts = 1))
GROUP BY month_number, month
ORDER BY month_number;
```
| month     | more_than_one_deposit_and_either_one_purchase_or_one_withdrawal |
| --------- | --------------------------------------------------------------- |
| january   | 115                                                             |
| february  | 108                                                             |
| march     | 113                                                             |
| april     | 50                                                              |
---
### What is the closing balance for each customer at the end of the month?
#### I pivoted the table again and countet the increase/decrease for each month
| customer_id | month | balance |
| ----------- | ----- | ------- |
| 1           | 1     | 312     |
| 1           | 3     | -952    |
| 2           | 1     | 549     |
| 2           | 3     | 61      |
| 3           | 1     | 144     |
| 3           | 2     | -965    |
| 3           | 3     | -401    |
| 3           | 4     | 493     |
#### With that I could calculate the closing balance for each month
| customer_id | month | balance |
| ----------- | ----- | ------- |
| 1           | 1     | 312     |
| 1           | 3     | -640    |
| 2           | 1     | 549     |
| 2           | 3     | 610     |
| 3           | 1     | 144     |
| 3           | 2     | -821    |
| 3           | 3     | -1222   |
| 3           | 4     | -729    |
#### I did't really understand what exactly they wanted to have as a solustion but I ended up with showing the first balance and the closing balance for the last month (maybe the step before that was asked but I needed this one for the next question anyway)
``` sql
WITH cte_pvt AS
    (SELECT customer_id, DATE_PART('month', txn_date) AS month,
    	SUM(txn_amount) FILTER (WHERE txn_type = 'deposit') AS deposit,
    	SUM(txn_amount) FILTER (WHERE txn_type = 'purchase') AS purchase,
    	SUM(txn_amount) FILTER (WHERE txn_type = 'withdrawal') AS withdrawal
     FROM data_bank.customer_transactions
     GROUP BY customer_id, month
    ),
    cte_balance_month AS
    (SELECT customer_id, month, (coalesce(deposit,0) - coalesce(purchase,0) - coalesce(withdrawal,0)) AS balance
     FROM cte_pvt
    ),
    cte_balance AS
    (SELECT customer_id, month, SUM(balance) OVER(PARTITION BY customer_id ORDER BY month) AS balance
     FROM cte_balance_month
    )
SELECT DISTINCT customer_id,
    first_value(balance) OVER (PARTITION BY customer_id) AS first_balance, 
    last_value(balance) OVER (PARTITION BY customer_id) AS closing_balance
FROM cte_balance
ORDER BY customer_id limit 50;
```
| customer_id | first_balance | closing_balance |
| ----------- | ------------- | --------------- |
| 1           | 312           | -640            |
| 2           | 549           | 610             |
| 3           | 144           | -729            |
| 4           | 848           | 655             |
| 5           | 954           | -2413           |
| 6           | 733           | 340             |
| 7           | 964           | 2623            |
| 8           | 587           | -1029           |
| 9           | 849           | 862             |
| 10          | -1622         | -5090           |
| 11          | -1744         | -2416           |
| 12          | 92            | 295             |
| 13          | 780           | 1405            |
| 14          | 205           | 989             |
| 15          | 379           | 1102            |
| 16          | -1341         | -3422           |
| 17          | 465           | -892            |
| 18          | 757           | -815            |
| 19          | -12           | 42              |
| 20          | 465           | 776             |
| 21          | -204          | -3253           |
| 22          | 235           | -1358           |
| 23          | 94            | -678            |
| 24          | 615           | 254             |
| 25          | 174           | -304            |
| 26          | 638           | -1870           |
| 27          | -1189         | -3116           |
| 28          | 451           | 272             |
| 29          | -138          | -548            |
| 30          | 33            | 508             |
| 31          | 83            | -141            |
| 32          | -89           | -1001           |
| 33          | 473           | 989             |
| 34          | 976           | -185            |
| 35          | 507           | -1163           |
| 36          | 149           | 427             |
| 37          | 85            | -959            |
| 38          | 367           | -1246           |
| 39          | 1429          | 2516            |
| 40          | 347           | -208            |
| 41          | -46           | 2525            |
| 42          | 447           | -1886           |
| 43          | -201          | 545             |
| 44          | -690          | -339            |
| 45          | 940           | 584             |
| 46          | 522           | 104             |
| 47          | -1153         | -3169           |
| 48          | -2368         | -3745           |
| 49          | -397          | -2556           |
| 50          | 931           | 450             |
---
### What is the percentage of customers who increase their closing balance by more than 5%?
#### I started like I did in the last question but I needed to filter for positive differences because elsewise I would get positive percentages if I start with a negative amount and get even more negative
``` sql
WITH cte_pvt AS
    (SELECT customer_id, DATE_PART('month', txn_date) AS month,
    	SUM(txn_amount) FILTER (WHERE txn_type = 'deposit') AS deposit,
    	SUM(txn_amount) FILTER (WHERE txn_type = 'purchase') AS purchase,
    	SUM(txn_amount) FILTER (WHERE txn_type = 'withdrawal') AS withdrawal
     FROM data_bank.customer_transactions
     GROUP BY customer_id, month
    ),
    cte_balance_month AS
    (SELECT customer_id, month, (coalesce(deposit,0) - coalesce(purchase,0) - coalesce(withdrawal,0)) AS balance
     FROM cte_pvt
    ),
    cte_balance AS
    (SELECT customer_id, month, SUM(balance) OVER(PARTITION BY customer_id ORDER BY month) AS balance
     FROM cte_balance_month
    ),
    cte_closing_balance AS
    (SELECT DISTINCT customer_id, first_value(balance) OVER (PARTITION BY customer_id) AS first_balance, 
     last_value(balance) OVER (PARTITION BY customer_id) AS closing_balance, 
     last_value(balance) OVER (PARTITION BY customer_id) - first_value(balance) OVER (PARTITION BY customer_id) AS diff
     FROM cte_balance
     ORDER BY customer_id 
    ),
    cte_percentage AS
    (SELECT customer_id, ROUND(diff / first_balance * 100,2) AS percentage
     FROM cte_closing_balance
     WHERE diff > 0
    )
SELECT ROUND((CAST(COUNT(DISTINCT customer_id) FILTER (WHERE percentage > 5) AS numeric))/COUNT(DISTINCT customer_id) *100,2) AS count_customer_percentage
FROM cte_percentage
RIGHT JOIN data_bank.customer_transactions USING(customer_id);
```
| count_customer_percentage |
| ------------------------- |
| 23.40                     |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2GtQz4wZtuNNu7zXH5HtV4/3)
