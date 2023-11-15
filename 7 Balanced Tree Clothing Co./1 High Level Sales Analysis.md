### What was the total quantity sold for all products?
```sql 
SELECT to_char(SUM(qty), '999,999') AS total_quantity
FROM balanced_tree.sales;
```
| total_quantity |
| -------------- |
| 45,216         |

---
### What is the total generated revenue for all products before discounts?
```sql
SELECT to_char(SUM(price), '999,999') AS total_revenue
FROM balanced_tree.sales;
```
| total_revenue |
| ------------- |
| 429,290       |

---
### What was the total discount amount for all products?
```sql
SELECT to_char(SUM(price * discount / 100), '999,999') AS total_discount
FROM balanced_tree.sales;
```
| total_discount |
| -------------- |
| 45,113         |
