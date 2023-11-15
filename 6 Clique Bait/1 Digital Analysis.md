### Using the available datasets - answer the following questions using a single query for each one:
---
### How many users are there?
```sql
    SELECT COUNT(DISTINCT user_id) AS user_count
    FROM clique_bait.users;
```
| user_count |
| ---------- |
| 500        |

---
### How many cookies does each user have on average?
```sql
    WITH cte_cookie AS
    (SELECT user_id, COUNT(DISTINCT cookie_id) AS cookie_count
     FROM clique_bait.users
     GROUP BY user_id
    )
    SELECT ROUND(avg(cookie_count),2) AS avg_cookie
    FROM cte_cookie;
```
| avg_cookie |
| ---------- |
| 3.56       |

---
### What is the unique number of visits by all users per month?
```sql
    SELECT DISTINCT ON(date_part('month',event_time)) 
    to_char(event_time,'month') AS month, COUNT(event_name) AS visit_count
    FROM clique_bait.event_identifier
    JOIN clique_bait.events USING(event_type)
    WHERE event_name = 'Page View'
    GROUP BY date_part('month',event_time), month
    ORDER BY date_part('month',event_time);
```
| month     | visit_count |
| --------- | ----------- |
| january   | 5185        |
| february  | 8713        |
| march     | 5298        |
| april     | 1484        |
| may       | 248         |

---
### What is the number of events for each event type?
```sql
    SELECT event_name, COUNT(event_type) AS number_of_events
    FROM clique_bait.event_identifier
    JOIN clique_bait.events USING(event_type)
    GROUP BY event_name
    ORDER BY number_of_events;
```
| event_name    | number_of_events |
| ------------- | ---------------- |
| Ad Click      | 702              |
| Ad Impression | 876              |
| Purchase      | 1777             |
| Add to Cart   | 8451             |
| Page View     | 20928            |

---
### What is the percentage of visits which have a purchase event?
```sql
    WITH cte_events AS
    (SELECT 
     COUNT(event_type) FILTER(WHERE event_type = '3')::numeric AS purchase,
     COUNT(event_type) FILTER(WHERE event_type = '1')::numeric AS page_view
     FROM clique_bait.events
    )
    SELECT ROUND(purchase/page_view*100,2) AS visits_with_purchase
    FROM cte_events;
```
| visits_with_purchase |
| -------------------- |
| 8.49                 |

---
### What is the percentage of visits which view the checkout page but do not have a purchase event?
```sql
    WITH cte_events AS
    (SELECT 
     COUNT(event_type) FILTER(WHERE event_type = '3')::numeric AS purchase,
     COUNT(event_type) FILTER(WHERE page_id = '12')::numeric AS page_checkout
     FROM clique_bait.events
    )
    SELECT 100 - ROUND(purchase/page_checkout*100,2) AS checkout_without_purchase
    FROM cte_events;
```
| checkout_without_purchase |
| ------------------------- |
| 15.50                     |

---
### What are the top 3 pages by number of views?
```sql
    SELECT page_name, COUNT(event_type) AS number_of_views
    FROM clique_bait.events
    JOIN clique_bait.page_hierarchy USING(page_id)
    WHERE event_type = '1'
    GROUP BY page_name
    ORDER BY number_of_views desc limit 3;
```
| page_name    | number_of_views |
| ------------ | --------------- |
| All Products | 3174            |
| Checkout     | 2103            |
| Home Page    | 1782            |

---
### What is the number of views and cart adds for each product category?
```sql
    SELECT product_category,
    COUNT(event_type) FILTER(WHERE event_type = '1') AS number_of_views,
    COUNT(event_type) FILTER(WHERE event_type = '2') AS cart_adds
    FROM clique_bait.events
    JOIN clique_bait.page_hierarchy USING(page_id)
    WHERE product_category != 'null'
    GROUP BY product_category;
```
| product_category | number_of_views | cart_adds |
| ---------------- | --------------- | --------- |
| Luxury           | 3032            | 1870      |
| Shellfish        | 6204            | 3792      |
| Fish             | 4633            | 2789      |

---
### What are the top 3 products by purchases?
```sql
    SELECT page_name AS product, COUNT(event_type) AS number_of_purchases
    FROM clique_bait.events
    JOIN clique_bait.page_hierarchy USING(page_id)
    WHERE visit_id IN
    (SELECT visit_id
     FROM clique_bait.events
     WHERE event_type = '3')
    AND product_id IS NOT null
    GROUP BY page_name
    ORDER BY number_of_purchases desc limit 3;
```
| product | number_of_purchases |
| ------- | ------------------- |
| Lobster | 1867                |
| Oyster  | 1862                |
| Crab    | 1827                |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/17)
