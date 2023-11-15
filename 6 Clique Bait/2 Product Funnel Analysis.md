### Using a single SQL query - create a new output table which has the following details:
- How many times was each product viewed?
- How many times was each product added to cart?
- How many times was each product added to a cart but not purchased (abandoned)?
- How many times was each product purchased?
```sql
CREATE TABLE clique_bait.product_details AS
    (SELECT product_id, page_name AS product,
     COUNT(product_id) FILTER(WHERE event_type = '1') AS viewed,
     COUNT(product_id) FILTER(WHERE event_type = '2') AS added_to_cart,
     COUNT(product_id) FILTER(WHERE event_type = '2' AND visit_id IN
                              (SELECT visit_id
                               FROM clique_bait.events
                               WHERE event_type = '3'))
                              AS purchased,
     COUNT(product_id) FILTER(WHERE event_type = '2' AND visit_id NOT IN
                              (SELECT visit_id
                               FROM clique_bait.events
                               WHERE event_type = '3'))
                              AS not_purchased
     FROM clique_bait.events
     JOIN clique_bait.page_hierarchy USING(page_id)
     WHERE product_id IS NOT null
     GROUP BY product_id, product
     ORDER BY product_id);
```
```sql 
SELECT * FROM clique_bait.product_details;
```
| product_id | product        | viewed | added_to_cart | purchased | not_purchased |
| ---------- | -------------- | ------ | ------------- | --------- | ------------- |
| 1          | Salmon         | 1559   | 938           | 711       | 227           |
| 2          | Kingfish       | 1559   | 920           | 707       | 213           |
| 3          | Tuna           | 1515   | 931           | 697       | 234           |
| 4          | Russian Caviar | 1563   | 946           | 697       | 249           |
| 5          | Black Truffle  | 1469   | 924           | 707       | 217           |
| 6          | Abalone        | 1525   | 932           | 699       | 233           |
| 7          | Lobster        | 1547   | 968           | 754       | 214           |
| 8          | Crab           | 1564   | 949           | 719       | 230           |
| 9          | Oyster         | 1568   | 943           | 726       | 217           |

---
### Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.
```sql
CREATE TABLE clique_bait.product_category_details AS
    (SELECT product_category, 
     SUM(viewed) AS viewed, 
     SUM(added_to_cart)AS added_to_cart,
     SUM(purchased)AS purchased,
     SUM(not_purchased)AS not_purchased
     FROM clique_bait.product_details
     JOIN clique_bait.page_hierarchy USING(product_id)
     GROUP BY product_category);
```
```sql 
SELECT * FROM clique_bait.product_category_details;
```
| product_category | viewed | added_to_cart | purchased | not_purchased |
| ---------------- | ------ | ------------- | --------- | ------------- |
| Luxury           | 3032   | 1870          | 1404      | 466           |
| Shellfish        | 6204   | 3792          | 2898      | 894           |
| Fish             | 4633   | 2789          | 2115      | 674           |

---
### Use your 2 new output tables - answer the following questions:
- Which product had the most views, cart adds and purchases?
- Which product was most likely to be abandoned?
#### I selected MAX(product) because we need an aggregate funcion to use FILTER, we could've also used MIN. This will only show one product if there are two with the most views for example.
```sql
SELECT 
MAX(product) FILTER(WHERE viewed = (SELECT MAX(viewed) FROM clique_bait.product_details)) AS most_viewed, 
MAX(product) FILTER(WHERE added_to_cart = (SELECT MAX(added_to_cart) FROM clique_bait.product_details)) AS most_added_to_cart,
MAX(product) FILTER(WHERE purchased = (SELECT MAX(purchased) FROM clique_bait.product_details)) AS most_purchased,
MAX(product) FILTER(WHERE not_purchased = (SELECT MAX(not_purchased) FROM clique_bait.product_details)) AS most_abandoned
FROM clique_bait.product_details;
```
| most_viewed | most_added_to_cart | most_purchased | most_abandoned |
| ----------- | ------------------ | -------------- | -------------- |
| Oyster      | Lobster            | Lobster        | Russian Caviar |

---
- Which product had the highest view to purchase percentage?
```sql
SELECT product, ROUND(purchased/viewed::numeric *100,2) AS highest_view_to_purchase_percentage
FROM clique_bait.product_details
ORDER BY highest_view_to_purchase_percentage desc LIMIT 1;
```
| product | highest_view_to_purchase_percentage |
| ------- | ----------------------------------- |
| Lobster | 48.74                               |

---
- What is the average conversion rate from view to cart add?
- What is the average conversion rate from cart add to purchase?
```sql
WITH cte_conversion AS
    (SELECT product, ROUND(added_to_cart/viewed::numeric *100,2) AS view_to_cart_add,
     ROUND(purchased/added_to_cart::numeric *100,2) AS cart_add_to_purchase
     FROM clique_bait.product_details
     )
SELECT 
ROUND(AVG(view_to_cart_add),2) AS avg_view_to_cart_add, 
ROUND(AVG(cart_add_to_purchase),2) AS avg_cart_add_to_purchase
FROM cte_conversion;
```
| avg_view_to_cart_add | avg_cart_add_to_purchase |
| -------------------- | ------------------------ |
| 60.95                | 75.93                    |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/17)
