### What are the top 3 products by total revenue before discount?
```sql
SELECT product_name, to_char(SUM(qty*s.price), '999,999,999') AS total_revenue
FROM balanced_tree.sales AS s
JOIN balanced_tree.product_details ON prod_id = product_id
GROUP BY product_name
ORDER BY total_revenue desc LIMIT 3;
```
| product_name                 | total_revenue |
| ---------------------------- | ------------- |
| Blue Polo Shirt - Mens       | 217,683       |
| Grey Fashion Jacket - Womens | 209,304       |
| White Tee Shirt - Mens       | 152,000       |

---
### What is the total quantity, revenue and discount for each segment?
```sql
SELECT segment_name,
	to_char(SUM(qty), '999,999,999') AS total_quantity,
	to_char(SUM(qty*s.price), '999,999,999') AS total_revenue,
	to_char(SUM(qty*discount), '999,999,999') AS total_discount
FROM balanced_tree.sales AS s
JOIN balanced_tree.product_details ON prod_id = product_id
GROUP BY segment_name;
```
| segment_name | total_quantity | total_revenue | total_discount |
| ------------ | -------------- | ------------- | -------------- |
| Shirt        | 11,265         | 406,143       | 136,971        |
| Jeans        | 11,349         | 208,350       | 137,909        |
| Jacket       | 11,385         | 366,983       | 137,044        |
| Socks        | 11,217         | 307,977       | 134,507        |

---
### What is the top selling product for each segment?
```sql
SELECT DISTINCT ON(segment_name) segment_name, product_name
FROM balanced_tree.sales AS s
JOIN balanced_tree.product_details ON prod_id = product_id
GROUP BY segment_name, product_name
ORDER BY segment_name, COUNT(qty) desc;
```
| segment_name | product_name                  |
| ------------ | ----------------------------- |
| Jacket       | Grey Fashion Jacket - Womens  |
| Jeans        | Navy Oversized Jeans - Womens |
| Shirt        | Blue Polo Shirt - Mens        |
| Socks        | Navy Solid Socks - Mens       |

---
### What is the total quantity, revenue and discount for each category?
```sql
SELECT category_name,
	to_char(SUM(qty), '999,999,999') AS total_quantity,
	to_char(SUM(qty*s.price), '999,999,999') AS total_revenue,
	to_char(SUM(qty*discount), '999,999,999') AS total_discount
FROM balanced_tree.sales AS s
JOIN balanced_tree.product_details ON prod_id = product_id
GROUP BY category_name;
```
| category_name | total_quantity | total_revenue | total_discount |
| ------------- | -------------- | ------------- | -------------- |
| Mens          | 22,482         | 714,120       | 271,478        |
| Womens        | 22,734         | 575,333       | 274,953        |

---
### What is the top selling product for each category?
```sql
SELECT DISTINCT ON(category_name) category_name, product_name 
FROM balanced_tree.sales AS s
JOIN balanced_tree.product_details ON prod_id = product_id
GROUP BY category_name, product_name
ORDER BY category_name, COUNT(qty) desc;
```
| category_name | product_name                 |
| ------------- | ---------------------------- |
| Mens          | Navy Solid Socks - Mens      |
| Womens        | Grey Fashion Jacket - Womens |

---
### What is the percentage split of revenue by product for each segment?
```sql
WITH cte_revenue AS
    (SELECT segment_name, product_name, SUM(s.qty*s.price) AS revenue
     FROM balanced_tree.sales AS s
     JOIN balanced_tree.product_details ON prod_id = product_id
     GROUP BY segment_name, product_name
    ),
    cte_total AS
    (SELECT segment_name, SUM(s.qty*s.price)::numeric AS total_revenue
     FROM balanced_tree.sales AS s
     JOIN balanced_tree.product_details ON prod_id = product_id
     GROUP BY segment_name
    )
SELECT segment_name, product_name, ROUND(revenue/total_revenue*100,2) AS "revenue[%]"
FROM cte_revenue
JOIN cte_total USING(segment_name)
ORDER BY segment_name;
```
| segment_name | product_name                     | revenue[%] |
| ------------ | -------------------------------- | ---------- |
| Jacket       | Indigo Rain Jacket - Womens      | 19.45      |
| Jacket       | Khaki Suit Jacket - Womens       | 23.51      |
| Jacket       | Grey Fashion Jacket - Womens     | 57.03      |
| Jeans        | Navy Oversized Jeans - Womens    | 24.06      |
| Jeans        | Black Straight Jeans - Womens    | 58.15      |
| Jeans        | Cream Relaxed Jeans - Womens     | 17.79      |
| Shirt        | White Tee Shirt - Mens           | 37.43      |
| Shirt        | Blue Polo Shirt - Mens           | 53.60      |
| Shirt        | Teal Button Up Shirt - Mens      | 8.98       |
| Socks        | Navy Solid Socks - Mens          | 44.33      |
| Socks        | White Striped Socks - Mens       | 20.18      |
| Socks        | Pink Fluro Polkadot Socks - Mens | 35.50      |

---
### What is the percentage split of revenue by segment for each category?
```sql
WITH cte_revenue AS
    (SELECT segment_name,  
     	SUM(qty*s.price) FILTER(WHERE category_name = 'Mens') AS revenue_men,
     	SUM(qty*s.price) FILTER(WHERE category_name = 'Womens') AS revenue_women
     FROM balanced_tree.sales AS s
     JOIN balanced_tree.product_details ON prod_id = product_id
     GROUP BY segment_name, category_name
    ),
    cte_total AS
    (SELECT SUM(price)::numeric AS total_revenue
     FROM balanced_tree.sales 
    )
SELECT segment_name, ROUND(revenue_men/total_revenue*100,2) AS "revenue[%] Men",
	ROUND(revenue_women/total_revenue*100,2) AS "revenue[%] Women"
FROM cte_revenue, cte_total;
```
| segment_name | revenue[%] Men | revenue[%] Women |
| ------------ | -------------- | ---------------- |
| Shirt        | 94.61          |                  |
| Jacket       |                | 85.49            |
| Jeans        |                | 48.53            |
| Socks        | 71.74          |                  |

---
### What is the percentage split of total revenue by category?
```sql
WITH cte_revenue AS
    (SELECT category_name, SUM(qty*s.price) AS revenue
     FROM balanced_tree.sales AS s
     JOIN balanced_tree.product_details ON prod_id = product_id
     GROUP BY category_name
    ),
    cte_total AS
    (SELECT SUM(qty*price)::numeric AS total_revenue
     FROM balanced_tree.sales 
    )
SELECT category_name, ROUND(revenue/total_revenue*100,2) AS "revenue[%]"
FROM cte_revenue, cte_total;
```
| category_name | revenue[%] |
| ------------- | ---------- |
| Mens          | 55.38      |
| Womens        | 44.62      |

---
### What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)
```sql
WITH cte_total AS
    (SELECT COUNT(DISTINCT txn_id)::numeric AS total_txn
     FROM balanced_tree.sales
    ),
    cte_txn AS
    (SELECT product_name, COUNT(txn_id) AS txn_count
     FROM balanced_tree.sales AS s
     JOIN balanced_tree.product_details ON prod_id = product_id
     GROUP BY product_name
     )
SELECT product_name, ROUND(txn_count/total_txn*100,2) AS "penetration[%]"
FROM cte_txn, cte_total;
```
| product_name                     | penetration[%] |
| -------------------------------- | -------------- |
| White Tee Shirt - Mens           | 50.72          |
| Navy Solid Socks - Mens          | 51.24          |
| Grey Fashion Jacket - Womens     | 51.00          |
| Navy Oversized Jeans - Womens    | 50.96          |
| Pink Fluro Polkadot Socks - Mens | 50.32          |
| Khaki Suit Jacket - Womens       | 49.88          |
| Black Straight Jeans - Womens    | 49.84          |
| White Striped Socks - Mens       | 49.72          |
| Blue Polo Shirt - Mens           | 50.72          |
| Indigo Rain Jacket - Womens      | 50.00          |
| Cream Relaxed Jeans - Womens     | 49.72          |
| Teal Button Up Shirt - Mens      | 49.68          |

---
### What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?
```sql
-- Join every transaction id with every product id
WITH cte_all_products AS
(SELECT txn_id, product_id, product_name
 FROM balanced_tree.sales, balanced_tree.product_details
 ORDER BY txn_id, product_id 
),
-- Join this with balanced_tree.sales and specify if the product is in the transaction
cte_products AS
(SELECT DISTINCT cte.txn_id, cte.product_id, cte.product_name, 
 CASE WHEN cte.product_id = s.prod_id THEN 1 ELSE 0 END AS product_count
 FROM cte_all_products AS cte
 LEFT JOIN balanced_tree.sales AS s ON cte.txn_id = s.txn_id AND cte.product_id = s.prod_id
 ORDER BY txn_id, product_id
),
-- With that specify for every 220 (binomial(12,3)) possible product combination if it is in a transaction
cte_combos AS
(SELECT c1.txn_id, 
 	 c1.product_name ||' , '|| c2.product_name ||' , '|| c3.product_name AS combo_name,
	 c1.product_id ||' , '|| c2.product_id ||' , '|| c3.product_id AS combo_id,
	 CASE WHEN c1.product_count = 1 AND c2.product_count = 1 AND c3.product_count = 1 THEN 1 ELSE 0 END AS cnt
 FROM cte_products AS c1
 INNER JOIN cte_products AS c2 ON c1.txn_id = c2.txn_id AND c1.product_id < c2.product_id
 INNER JOIN cte_products AS c3 ON c2.txn_id = c3.txn_id AND c2.product_id < c3.product_id
 )
 -- Count all the combinations
 SELECT combo_id, combo_name, SUM(cnt) AS combo_count
 FROM cte_combos
 GROUP BY combo_id, combo_name
 ORDER BY SUM(cnt) DESC LIMIT 3;
```
| combo_id                 | combo_name                                                                            | combo_count |
| ------------------------ | ------------------------------------------------------------------------------------- | ----------- |
| 5d267b , 9ec847 , c8d436 | White Tee Shirt - Mens , Grey Fashion Jacket - Womens , Teal Button Up Shirt - Mens   | 352         |
| 72f5d4 , e83aa3 , f084eb | Indigo Rain Jacket - Womens , Black Straight Jeans - Womens , Navy Solid Socks - Mens | 349         |
| 2a2353 , 9ec847 , c8d436 | Blue Polo Shirt - Mens , Grey Fashion Jacket - Womens , Teal Button Up Shirt - Mens   | 347         |
---
## Reporting Challange
### Write a single SQL script that combines all of the previous questions into a scheduled report that the Balanced Tree team can run at the beginning of each month to calculate the previous month’s values. Imagine that the Chief Financial Officer (which is also Danny) has asked for all of these questions at the end of every month. He first wants you to generate the data for January only - but then he also wants you to demonstrate that you can easily run the samne analysis for February without many changes (if at all). Feel free to split up your final outputs into as many tables as you need - but be sure to explicitly reference which table outputs relate to which question for full marks :)

#### My idea would be to create a table from balanced_tree.sales for every month when you need it and name it balancd_tree.salesmonth. Then just change every balanced_tree.sales to balancd_tree.salesmonth in your queries and run them if you need them.
```sql
CREATE TABLE balanced_tree.salesmonth AS
(SELECT * FROM balanced_tree.sales
 WHERE date_part('month', start_txn_time) = 2);
```

[View on DB Fiddle](https://www.db-fiddle.com/f/dkhULDEjGib3K58MvDjYJr/67)
