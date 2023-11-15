# 🍜 Case Study #1: Danny's Diner

# Solutions

### 1. What is the total amount each customer spent at the restaurant?
``` sql
SELECT customer_id, SUM(price) AS total_amount_per_customer
FROM dannys_diner.sales  
JOIN dannys_diner.menu USING(product_id)
GROUP BY customer_id;
```
| customer_id | total_amount_per_customer |
| ----------- | ------------------------- |
| B           | 74                        |
| C           | 36                        |
| A           | 76                        |
---
###  2. How many days has each customer visited the restaurant?
``` sql
SELECT customer_id, COUNT(DISTINCT order_date) AS Days_visited 
FROM dannys_diner.sales
GROUP BY customer_id;

```
| customer_id | days_visited |
| ----------- | ------------ |
| A           | 4            |
| B           | 6            |
| C           | 2            |

---
### 3. What was the first item from the menu purchased by each customer?
``` sql
SELECT DISTINCT ON(customer_id) customer_id, product_name
FROM dannys_diner.sales  
JOIN dannys_diner.menu USING(product_id);
```
| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| B           | sushi        |
| C           | ramen        |

---
### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
``` sql
SELECT product_name AS most_purchased, COUNT(product_id) AS times_purchsed
FROM dannys_diner.sales
JOIN dannys_diner.menu USING(product_id)
GROUP BY product_name
ORDER BY times_purchsed desc limit 1;
```
| most_purchased | times_purchsed |
| -------------- | -------------- |
| ramen          | 8              |

---
### 5. Which item was the most popular for each customer?
``` sql
SELECT DISTINCT ON(customer_id) customer_id, product_name AS most_popular
FROM dannys_diner.sales
JOIN dannys_diner.menu USING(product_id)
GROUP BY customer_id, product_name
ORDER BY customer_id, COUNT(product_id) desc;
```
| customer_id | most_popular |
| ----------- | ------------ |
| A           | ramen        |
| B           | ramen        |
| C           | ramen        |

---
### 6. Which item was purchased first by the customer after they became a member?
``` sql
SELECT DISTINCT ON(customer_id) customer_id, product_name
FROM dannys_diner.sales
JOIN dannys_diner.menu USING(product_id)
JOIN dannys_diner.members USING(customer_id)
WHERE order_date >= join_date;
```
| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| B           | sushi        |

---
### 7. Which item was purchased just before the customer became a member?
``` sql
SELECT DISTINCT ON(customer_id) customer_id, product_name
FROM dannys_diner.sales
JOIN dannys_diner.menu USING(product_id)
JOIN dannys_diner.members USING(customer_id)
WHERE order_date < join_date 
ORDER BY customer_id, order_date desc, product_name;
```
| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| B           | sushi        |

---
### 8. What is the total items and amount spent for each member before they became a member?
``` sql
SELECT customer_id, COUNT(product_id) AS total_items, SUM(price) AS total_amount
FROM dannys_diner.sales
JOIN dannys_diner.menu USING(product_id)
JOIN dannys_diner.members USING(customer_id)
WHERE order_date < join_date
GROUP BY customer_id;
```
| customer_id | total_items | total_amount |
| ----------- | ----------- | ------------ |
| B           | 3           | 40           |
| A           | 2           | 25           |

---
### 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
``` sql
SELECT customer_id, SUM(CASE WHEN product_id = 1 THEN price*20 ELSE price*10 END) AS points
FROM dannys_diner.sales
JOIN dannys_diner.menu USING(product_id)
GROUP BY customer_id;
```
| customer_id | points |
| ----------- | ------ |
| B           | 940    |
| C           | 360    |
| A           | 860    |

---
### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
``` sql
SELECT customer_id, 
		SUM(CASE 
            WHEN product_id = 1 OR order_date BETWEEN join_date AND join_date + 7 
            THEN price*20 
            ELSE price*10 END) AS points
FROM dannys_diner.sales
JOIN dannys_diner.menu USING(product_id)
JOIN dannys_diner.members USING(customer_id)
WHERE order_date < '2021-02-01'::date
GROUP BY customer_id;
```
| customer_id | points |
| ----------- | ------ |
| A           | 1370   |
| B           | 940    |

---
### BONUS - Join All The Things
``` sql
SELECT customer_id, order_date, product_name, price, CASE WHEN order_date < join_date OR join_date IS null THEN 'N' ELSE 'Y' END AS member
FROM dannys_diner.sales  
JOIN dannys_diner.menu  USING(product_id)
LEFT JOIN dannys_diner.members  USING(customer_id)
ORDER BY customer_id, order_date
```
| customer_id | order_date               | product_name | price | member |
| ----------- | ------------------------ | ------------ | ----- | ------ |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 10    | N      |
| A           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| A           | 2021-01-07T00:00:00.000Z | curry        | 15    | Y      |
| A           | 2021-01-10T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      |
| B           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-02T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-04T00:00:00.000Z | sushi        | 10    | N      |
| B           | 2021-01-11T00:00:00.000Z | sushi        | 10    | Y      |
| B           | 2021-01-16T00:00:00.000Z | ramen        | 12    | Y      |
| B           | 2021-02-01T00:00:00.000Z | ramen        | 12    | Y      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |
| C           | 2021-01-07T00:00:00.000Z | ramen        | 12    | N      |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)
