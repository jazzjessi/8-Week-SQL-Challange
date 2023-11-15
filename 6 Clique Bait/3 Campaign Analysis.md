### Generate a table that has 1 single row for every unique visit_id record and has the following columns:

- user_id
- visit_id
- visit_start_time: the earliest event_time for each visit
- page_views: count of page views for each visit
- cart_adds: count of product cart add events for each visit
- purchase: 1/0 flag if a purchase event exists for each visit
- campaign_name: map the visit to a campaign if the visit_start_time falls between the start_date and end_date
- impression: count of ad impressions for each visit
- click: count of ad clicks for each visit
- (Optional column) cart_products: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number)

```sql
CREATE TABLE clique_bait.campaign_analysis AS
(WITH cte_cart AS
    (SELECT visit_id, string_agg(page_name, ', ' ORDER BY sequence_number) AS cart_products
     FROM clique_bait.events
     JOIN clique_bait.page_hierarchy USING(page_id)
     WHERE event_type = '2'
     GROUP BY visit_id
    )
SELECT 
    user_id, 
    visit_id,
    MIN(event_time) AS visit_start_time,
    COUNT(event_type) FILTER(WHERE event_type = '1') AS page_views,
    COUNT(event_type) FILTER(WHERE event_type = '2') AS cart_adds,
    CASE WHEN COUNT(event_type) FILTER(WHERE event_type = '3') > 0 THEN '1' ELSE '0' END AS purchase,
    campaign_name,
    COUNT(event_type) FILTER(WHERE event_type = '4') AS impression,
    COUNT(event_type) FILTER(WHERE event_type = '5') AS click,
    cart_products
FROM clique_bait.users
JOIN clique_bait.events USING(cookie_id)
JOIN clique_bait.page_hierarchy USING(page_id)
JOIN cte_cart USING(visit_id)
JOIN clique_bait.campaign_identifier AS c
    ON c.start_date < event_time AND event_time < end_date
GROUP BY user_id, visit_id, campaign_name, cart_products
ORDER BY user_id, visit_start_time
);
```
#### I only showed the first 20 rows.
```sql
SELECT * FROM clique_bait.campaign_analysis LIMIT 20;
```
| user_id | visit_id | visit_start_time         | page_views | cart_adds | purchase | campaign_name                     | impression | click | cart_products                                                                         |
| ------- | -------- | ------------------------ | ---------- | --------- | -------- | --------------------------------- | ---------- | ----- | ------------------------------------------------------------------------------------- |
| 1       | 0fc437   | 2020-02-04T17:49:49.602Z | 10         | 6         | 1        | Half Off - Treat Your Shellf(ish) | 1          | 1     | Tuna, Russian Caviar, Black Truffle, Abalone, Crab, Oyster                            |
| 1       | ccf365   | 2020-02-04T19:16:09.182Z | 7          | 3         | 1        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Lobster, Crab, Oyster                                                                 |
| 1       | f7c798   | 2020-03-15T02:23:26.312Z | 9          | 3         | 1        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Russian Caviar, Crab, Oyster                                                          |
| 1       | 30b94d   | 2020-03-15T13:12:54.023Z | 9          | 7         | 1        | Half Off - Treat Your Shellf(ish) | 1          | 1     | Salmon, Kingfish, Tuna, Russian Caviar, Abalone, Lobster, Crab                        |
| 1       | 41355d   | 2020-03-25T00:11:17.860Z | 6          | 1         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Lobster                                                                               |
| 1       | eaffde   | 2020-03-25T20:06:32.342Z | 10         | 8         | 1        | Half Off - Treat Your Shellf(ish) | 1          | 1     | Salmon, Tuna, Russian Caviar, Black Truffle, Abalone, Lobster, Crab, Oyster           |
| 2       | 3b5871   | 2020-01-18T10:16:32.158Z | 9          | 6         | 1        | 25% Off - Living The Lux Life     | 1          | 1     | Salmon, Kingfish, Russian Caviar, Black Truffle, Lobster, Oyster                      |
| 2       | e26a84   | 2020-01-18T16:06:40.907Z | 6          | 2         | 1        | 25% Off - Living The Lux Life     | 0          | 0     | Salmon, Oyster                                                                        |
| 2       | d58cbd   | 2020-01-18T23:40:54.761Z | 8          | 4         | 0        | 25% Off - Living The Lux Life     | 0          | 0     | Kingfish, Tuna, Abalone, Crab                                                         |
| 2       | 910d9a   | 2020-02-01T10:40:46.875Z | 8          | 1         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Abalone                                                                               |
| 2       | 49d73d   | 2020-02-16T06:21:27.138Z | 11         | 9         | 1        | Half Off - Treat Your Shellf(ish) | 1          | 1     | Salmon, Kingfish, Tuna, Russian Caviar, Black Truffle, Abalone, Lobster, Crab, Oyster |
| 2       | 0635fb   | 2020-02-16T06:42:42.735Z | 9          | 4         | 1        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Salmon, Kingfish, Abalone, Crab                                                       |
| 3       | 9a2f24   | 2020-02-21T03:19:10.032Z | 6          | 2         | 1        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Kingfish, Black Truffle                                                               |
| 3       | bf200a   | 2020-03-11T04:10:26.708Z | 7          | 2         | 1        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Salmon, Crab                                                                          |
| 4       | 7caba5   | 2020-02-22T17:49:37.646Z | 5          | 2         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Tuna, Lobster                                                                         |
| 4       | b90e25   | 2020-03-19T11:01:58.182Z | 9          | 4         | 1        | Half Off - Treat Your Shellf(ish) | 1          | 1     | Tuna, Black Truffle, Lobster, Crab                                                    |
| 5       | f61ed7   | 2020-02-01T06:30:39.766Z | 8          | 2         | 1        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Lobster, Crab                                                                         |
| 5       | 580bf6   | 2020-02-11T04:05:42.307Z | 8          | 6         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Salmon, Tuna, Russian Caviar, Black Truffle, Abalone, Lobster                         |
| 5       | 05c52a   | 2020-02-11T12:30:33.479Z | 9          | 8         | 0        | Half Off - Treat Your Shellf(ish) | 1          | 1     | Salmon, Kingfish, Tuna, Russian Caviar, Black Truffle, Abalone, Lobster, Crab         |
| 5       | 4bffe1   | 2020-02-26T16:03:10.377Z | 8          | 1         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Russian Caviar                                                                        |

---
### Use the subsequent dataset to generate at least 5 insights for the Clique Bait team - bonus: prepare a single A4 infographic that the team can use for their management reporting sessions, be sure to emphasise the most important points from your findings.

### Some ideas you might want to investigate further include:

- Identifying users who have received impressions during each campaign period and comparing each metric with other users who did not have an impression event
- Does clicking on an impression lead to higher purchase rates?
- What is the uplift in purchase rate when comparing users who click on a campaign impression versus users who do not receive an impression? What if we compare them with users who just an impression but do not click?
- What metrics can you use to quantify the success or failure of each campaign compared to eachother?
---
#### I compared the purchase rates for clicking on an add, getting an impression on an add without clicking on it and getteing no add. We can see that clicking on an add seems to have a high impact on wether the users purchase the items or abandon it.
```sql
WITH cte_adds AS
    (SELECT purchase,
     COUNT(purchase) FILTER(WHERE impression > 0 AND click > 0) AS click,
     COUNT(purchase) FILTER(WHERE impression > 0 AND click = 0) AS impression,
     COUNT(purchase) FILTER(WHERE impression = 0 AND click = 0) AS no_add,
     COUNT(purchase)::numeric AS total
     FROM clique_bait.campaign_analysis
     GROUP BY purchase
     ORDER BY purchase desc
    )
SELECT 
    CASE WHEN purchase = '1' THEN 'successful purchase'
    WHEN purchase = '0' THEN 'abandoned purchase' END AS purchase,
    ROUND(click/total*100,2) AS click,
    ROUND(impression/total*100,2) AS impression,
    ROUND(no_add/total*100,2) AS no_add
FROM cte_adds;
```
| purchase            | click | impression | no_add |
| ------------------- | ----- | ---------- | ------ |
| successful purchase | 35.59 | 6.49       | 57.92  |
| abandoned purchase  | 9.79  | 6.48       | 83.73  |

---
#### I did the same as above but for the different campaigns. There isn't really that much of a difference between the different events.
```sql
WITH cte_adds AS
    (SELECT campaign_name, purchase,
     COUNT(purchase) FILTER(WHERE impression > 0 AND click > 0) AS click,
     COUNT(purchase) FILTER(WHERE impression > 0 AND click = 0) AS impression,
     COUNT(purchase) FILTER(WHERE impression = 0 AND click = 0) AS no_add,
     COUNT(purchase)::numeric AS total
     FROM clique_bait.campaign_analysis
     GROUP BY campaign_name, purchase
     ORDER BY campaign_name, purchase desc
    )
SELECT campaign_name,
    CASE WHEN purchase = '1' THEN 'successful purchase'
    WHEN purchase = '0' THEN 'abandoned purchase' END AS purchase,
    ROUND(click/total*100,2) AS click,
    ROUND(impression/total*100,2) AS impression,
    ROUND(no_add/total*100,2) AS no_add
FROM cte_adds;
```

| campaign_name                     | purchase            | click | impression | no_add |
| --------------------------------- | ------------------- | ----- | ---------- | ------ |
| 25% Off - Living The Lux Life     | successful purchase | 35.15 | 7.92       | 56.93  |
| 25% Off - Living The Lux Life     | abandoned purchase  | 11.76 | 8.24       | 80.00  |
| BOGOF - Fishing For Compliments   | successful purchase | 39.37 | 3.94       | 56.69  |
| BOGOF - Fishing For Compliments   | abandoned purchase  | 9.43  | 9.43       | 81.13  |
| Half Off - Treat Your Shellf(ish) | successful purchase | 35.25 | 6.53       | 58.22  |
| Half Off - Treat Your Shellf(ish) | abandoned purchase  | 9.49  | 5.86       | 84.65  |

---
#### Here I looked for the users who got an impression for every event. There are only three but I didn't check if it is because they never visited the website anymore or because they didn't get impressions anymore. From these three users there is only one who abandoned his purchase once.
#### I named the subquery x because every subquery needs a name. We could have also just used a cte again.
```sql
SELECT user_id, purchased, added_to_cart FROM
    (SELECT user_id, COUNT(DISTINCT campaign_name), SUM(purchase::int) AS purchased, COUNT(purchase) AS added_to_cart
     FROM clique_bait.campaign_analysis
     WHERE impression > 0
     GROUP BY user_id
     HAVING COUNT(DISTINCT campaign_name) = 3)x;
```
| user_id | purchased | added_to_cart |
| ------- | --------- | ------------- |
| 288     | 3         | 3             |
| 405     | 3         | 3             |
| 498     | 2         | 3             |
---
#### We can do more than that. I looked up the users who visited the website for every event and calculated how many visits they had in total, how many impressions they got (note: that doesn't mean they got impressions for every event, see the table above) and how many times they purchased something durig their visit.
```sql
    SELECT user_id, visits, impression, purchased FROM
        (SELECT user_id,
         COUNT(visit_id) AS visits,
         SUM(impression) AS impression, 
         COUNT(DISTINCT campaign_name), 
         SUM(purchase::int) AS purchased 
         FROM clique_bait.campaign_analysis
         GROUP BY user_id
         HAVING COUNT(DISTINCT campaign_name) = 3)x
    ORDER BY impression desc, visits desc;
```
| user_id | visits | impression | purchased |
| ------- | ------ | ---------- | --------- |
| 430     | 8      | 4          | 6         |
| 187     | 8      | 3          | 6         |
| 498     | 7      | 3          | 4         |
| 226     | 7      | 3          | 3         |
| 405     | 7      | 3          | 5         |
| 288     | 6      | 3          | 5         |
| 227     | 6      | 2          | 4         |
| 245     | 6      | 2          | 5         |
| 103     | 6      | 2          | 5         |
| 133     | 6      | 2          | 6         |
| 326     | 6      | 2          | 5         |
| 379     | 6      | 2          | 2         |
| 217     | 6      | 2          | 3         |
| 109     | 5      | 2          | 3         |
| 127     | 5      | 2          | 5         |
| 200     | 5      | 2          | 4         |
| 69      | 5      | 2          | 4         |
| 117     | 5      | 2          | 4         |
| 39      | 6      | 1          | 6         |
| 392     | 5      | 1          | 4         |
| 312     | 4      | 1          | 4         |
| 365     | 4      | 1          | 2         |
| 432     | 4      | 1          | 3         |
| 465     | 4      | 1          | 2         |
| 82      | 3      | 1          | 2         |
| 126     | 3      | 0          | 2         |
| 121     | 3      | 0          | 2         |

---
#### Here we have the 27 users who visitet the website for every event and compare them to all the users
```sql
SELECT COUNT(user_id) AS count_user,
    ROUND(AVG(visits),2) AS avg_visits, 
    ROUND(AVG(impression),2) AS avg_impressions, 
    ROUND(AVG(purchased),2) AS avg_purchased FROM
        (SELECT user_id,
         COUNT(visit_id) AS visits,
         SUM(impression) AS impression, 
         COUNT(DISTINCT campaign_name), 
         SUM(purchase::int) AS purchased 
         FROM clique_bait.campaign_analysis
         GROUP BY user_id
         HAVING COUNT(DISTINCT campaign_name) = 3)x;
```
| count_user | avg_visits | avg_impressions | avg_purchased |
| ---------- | ---------- | --------------- | ------------- |
| 27         | 5.41       | 1.85            | 3.93          |


```sql
SELECT COUNT(user_id) AS count_user,
    ROUND(AVG(visits),2) AS avg_visits, 
    ROUND(AVG(impression),2) AS avg_impressions, 
    ROUND(AVG(purchased),2) AS avg_purchased FROM
        (SELECT user_id,
         COUNT(visit_id) AS visits,
         SUM(impression) AS impression,  
         SUM(purchase::int) AS purchased 
         FROM clique_bait.campaign_analysis
         GROUP BY user_id)x;
```
| count_user | avg_visits | avg_impressions | avg_purchased |
| ---------- | ---------- | --------------- | ------------- |
| 494        | 4.34       | 1.49            | 3.05          |

---
#### I wanted to compare the results for the top 50, top 100 ect users by visits. Still looking for a better way to do it.
```sql 
WITH cte_visit AS
    (SELECT user_id,
         COUNT(visit_id) AS visits,
         SUM(impression) AS impression, 
         SUM(purchase::int) AS purchased 
         FROM clique_bait.campaign_analysis
         GROUP BY user_id
         ORDER BY visits desc limit 50
    ), cte_avg AS
    (SELECT COUNT(user_id) AS top_user_count,
     ROUND(AVG(visits),2) AS avg_visits, 
     ROUND(AVG(impression),2) AS avg_impressions, 
     ROUND(AVG(purchased),2) AS avg_purchased 
     FROM cte_visit
    )
SELECT top_user_count, avg_visits, avg_impressions, 
    ROUND(avg_impressions/avg_visits*100,2) AS impressions_per_visit,
    avg_purchased,
    ROUND(avg_purchased/avg_visits*100,2) AS purchase_per_visit
    FROM cte_avg;
```
| top_user_count | avg_visits | avg_impressions | impressions_per_visit | avg_purchased | purchase_per_visit |
| -------------- | ---------- | --------------- | --------------------- | ------------- | ------------------ |
| 50             | 7.62       | 3.06            | 40.16                 | 5.68          | 74.54              |

---
#### Maybe something like that would be interesting? I compared the impressions per visit and the purchases per visit dependent on the amount of visits. E.g. 15 Users visitet the website eight times and 40.88% of the visits they got add impressions and 74.13% of the time the visit leads to a purchase.
```sql
WITH cte_visit AS
    (SELECT user_id,
         COUNT(visit_id) AS visits,
         SUM(impression) AS impression, 
         SUM(purchase::int) AS purchased 
         FROM clique_bait.campaign_analysis
         GROUP BY user_id
    	 ORDER BY visits desc
     ), cte_avg AS
     (SELECT
         COUNT(user_id) OVER(PARTITION BY visits ORDER BY visits desc) AS top_user_count,
         ROUND(AVG(visits) OVER(PARTITION BY visits ORDER BY visits desc),2) AS avg_visits, 
         ROUND(AVG(impression) OVER(PARTITION BY visits ORDER BY visits desc),2) AS avg_impressions, 
         ROUND(AVG(purchased) OVER(PARTITION BY visits ORDER BY visits desc),2) AS avg_purchased 
     FROM cte_visit
   )
SELECT DISTINCT
    top_user_count,
    avg_visits,
    avg_impressions, 
    ROUND(avg_impressions/avg_visits*100,2) AS impressions_per_visit,
    avg_purchased,
    ROUND(avg_purchased/avg_visits*100,2) AS purchase_per_visit
FROM cte_avg
ORDER BY avg_visits desc;
```
| top_user_count | avg_visits | avg_impressions | impressions_per_visit | avg_purchased | purchase_per_visit |
| -------------- | ---------- | --------------- | --------------------- | ------------- | ------------------ |
| 2              | 10.00      | 4.00            | 40.00                 | 8.00          | 80.00              |
| 5              | 9.00       | 3.80            | 42.22                 | 6.80          | 75.56              |
| 15             | 8.00       | 3.27            | 40.88                 | 5.93          | 74.13              |
| 38             | 7.00       | 2.74            | 39.14                 | 5.11          | 73.00              |
| 69             | 6.00       | 2.30            | 38.33                 | 4.36          | 72.67              |
| 88             | 5.00       | 1.85            | 37.00                 | 3.51          | 70.20              |
| 107            | 4.00       | 1.32            | 33.00                 | 2.72          | 68.00              |
| 91             | 3.00       | 0.74            | 24.67                 | 2.10          | 70.00              |
| 57             | 2.00       | 0.44            | 22.00                 | 1.23          | 61.50              |
| 22             | 1.00       | 0.14            | 14.00                 | 0.64          | 64.00              |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/17)
