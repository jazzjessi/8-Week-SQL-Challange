### How many unique transactions were there?
```sql
SELECT COUNT(DISTINCT txn_id) AS amount_transactions
FROM balanced_tree.sales;
```
| amount_transactions |
| ------------------- |
| 2500                |

---
### What is the average unique products purchased in each transaction?
```sql
WITH cte_products AS
    (SELECT txn_id, COUNT(DISTINCT prod_id) AS count_products
     FROM balanced_tree.sales
     GROUP BY txn_id
    )
SELECT ROUND(AVG(count_products)) AS avg_unique_products_per_transaction
FROM cte_products;
```
| avg_unique_products_per_transaction |
| ----------------------------------- |
| 6                                   |

---
### What are the 25th, 50th and 75th percentile values for the revenue per transaction?
```sql
WITH cte_revenue AS
    (SELECT txn_id, 
     ROW_NUMBER() OVER(ORDER BY SUM(price * (1-discount/100))) AS id,
     ROUND(SUM(price * (1-discount/100)),2) AS revenue_per_transaction
     FROM balanced_tree.sales
     GROUP BY txn_id
     ORDER BY revenue_per_transaction
    ), cte_count AS
    (SELECT COUNT(DISTINCT txn_id)::numeric AS amount_transactions
     FROM balanced_tree.sales
     )
SELECT 
    MAX(revenue_per_transaction) FILTER(WHERE id = amount_transactions*0.25) AS "25th percentile",
    MAX(revenue_per_transaction) FILTER(WHERE id = amount_transactions*0.5) AS "50th percentile",
    MAX(revenue_per_transaction) FILTER(WHERE id = amount_transactions*0.75) AS "75th percentile"
FROM cte_revenue, cte_count;
```
| 25th percentile | 50th percentile | 75th percentile |
| --------------- | --------------- | --------------- |
| 135.00          | 173.00          | 209.00          |

---
### What is the average discount value per transaction?
```sql
WITH cte_discount AS
    (SELECT txn_id, SUM(price*discount/100) AS discount_value_per_transaction
     FROM balanced_tree.sales
     GROUP BY txn_id
    )
SELECT ROUND(AVG(discount_value_per_transaction),2) AS avg_discount_value_per_transaction
FROM cte_discount;
```
| avg_discount_value_per_transaction |
| ---------------------------------- |
| 18.05                              |

---
### What is the percentage split of all transactions for members vs non-members?
```sql
WITH cte_member AS
    (SELECT member, COUNT(DISTINCT txn_id) AS count_members
     FROM balanced_tree.sales
     GROUP BY member
    ), cte_count AS
    (SELECT COUNT(DISTINCT txn_id)::numeric AS amount_transactions
     FROM balanced_tree.sales
     )
SELECT member, ROUND(count_members/amount_transactions*100,2) AS percentage_transactions
FROM cte_member, cte_count;
```
| member | percentage_transactions |
| ------ | ----------------------- |
| false  | 39.80                   |
| true   | 60.20                   |

---
### What is the average revenue for member transactions and non-member transactions?
```sql
WITH cte_discount AS
    (SELECT member, txn_id, SUM(price*discount/100) AS discount_value_per_transaction
     FROM balanced_tree.sales
     GROUP BY member, txn_id
    )
SELECT member, ROUND(AVG(discount_value_per_transaction),2) AS avg_revenue
FROM cte_discount
GROUP BY member;
```
| member | avg_revenue |
| ------ | ----------- |
| false  | 18.17       |
| true   | 17.96       |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/dkhULDEjGib3K58MvDjYJr/8)
