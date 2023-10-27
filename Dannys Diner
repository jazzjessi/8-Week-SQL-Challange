# 🍜 Case Study #1: Danny's Diner

# Solutions



### 1. What is the total amount each customer spent at the restaurant?
``` sql
SELECT s.customer_id, SUM(m.price) AS total_amount_per_customer
FROM dannys_diner.sales AS s JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
GROUP BY s.customer_id;
```
| customer_id | total_amount_per_customer |
| ----------- | ------------------------- |
| B           | 74                        |
| C           | 36                        |
| A           | 76                        |



###  2. How many days has each customer visited the restaurant?
``` sql
    SELECT s.customer_id, COUNT(DISTINCT s.order_date) AS Days_visited 
    FROM dannys_diner.sales AS s
    GROUP BY s.customer_id;
```
| customer_id | days_visited |
| ----------- | ------------ |
| A           | 4            |
| B           | 6            |
| C           | 2            |

---
### 3. What was the first item from the menu purchased by each customer?
``` sql
    SELECT DISTINCT ON(s.customer_id) s.customer_id, m.product_name
    FROM dannys_diner.sales AS s JOIN dannys_diner.menu AS m ON s.product_id = m.product_id;
```
| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| B           | sushi        |
| C           | ramen        |

---
### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
``` sql
    SELECT m.product_name AS most_purchased, COUNT(s.product_id) AS times_purchsed
    FROM dannys_diner.sales AS s JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
    GROUP BY m.product_name
    ORDER BY times_purchsed desc limit 1;
```
| most_purchased | times_purchsed |
| -------------- | -------------- |
| ramen          | 8              |

---
### 5. Which item was the most popular for each customer?
``` sql
    SELECT DISTINCT ON(s.customer_id) s.customer_id, m.product_name AS most_popular
    FROM dannys_diner.sales AS s JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
    GROUP BY s.customer_id, m.product_name
    ORDER BY s.customer_id, COUNT(s.product_id) desc;
```
| customer_id | most_popular |
| ----------- | ------------ |
| A           | ramen        |
| B           | ramen        |
| C           | ramen        |

---
### 6. Which item was purchased first by the customer after they became a member?
``` sql
    SELECT DISTINCT ON(s.customer_id) s.customer_id, m.product_name
    FROM dannys_diner.sales AS s 
    JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
    JOIN dannys_diner.members AS mem ON s.customer_id = mem.customer_id
    WHERE s.order_date >= mem.join_date;
```
| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| B           | sushi        |

---
### 7. Which item was purchased just before the customer became a member?
``` sql
    SELECT DISTINCT ON(s.customer_id) s.customer_id, m.product_name
    FROM dannys_diner.sales AS s 
    JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
    JOIN dannys_diner.members AS mem ON s.customer_id = mem.customer_id
    WHERE s.order_date < mem.join_date 
    ORDER BY s.customer_id, s.order_date desc, m.product_name;
```
| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| B           | sushi        |

---
### 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
``` sql
    SELECT s.customer_id, COUNT(m.product_id) AS total_items, SUM(m.price) AS total_amount
    FROM dannys_diner.sales AS s 
    JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
    JOIN dannys_diner.members AS mem ON s.customer_id = mem.customer_id
    WHERE s.order_date < mem.join_date
    GROUP BY s.customer_id;
```
| customer_id | total_items | total_amount |
| ----------- | ----------- | ------------ |
| B           | 3           | 40           |
| A           | 2           | 25           |

---
### 8. What is the total items and amount spent for each member before they became a member?
``` sql
    SELECT s.customer_id, SUM(CASE WHEN m.product_id = 1 THEN m.price*20 ELSE m.price*10 END) AS points
    FROM dannys_diner.sales AS s JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
    GROUP BY s.customer_id;
```
| customer_id | points |
| ----------- | ------ |
| B           | 940    |
| C           | 360    |
| A           | 860    |

---
### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
``` sql
    SELECT s.customer_id, 
    		SUM(CASE 
                WHEN m.product_id = 1 OR s.order_date BETWEEN mem.join_date AND mem.join_date + 7 
                THEN m.price*20 
                ELSE m.price*10 END) AS points
    FROM dannys_diner.sales AS s 
    JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
    JOIN dannys_diner.members AS mem ON s.customer_id = mem.customer_id
    WHERE s.order_date < '2021-02-01'::date
    GROUP BY s.customer_id;
```
| customer_id | points |
| ----------- | ------ |
| A           | 1370   |
| B           | 940    |

---
### BONUS
``` sql
    SELECT customer_id, order_date, product_name, price, CASE WHEN order_date < join_date OR join_date IS null THEN 'N' ELSE 'Y' END AS member
    FROM dannys_diner.sales  
    JOIN dannys_diner.menu  USING(product_id)
    LEFT JOIN dannys_diner.members  USING(customer_id)
    ORDER BY customer_id, order_date;
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
