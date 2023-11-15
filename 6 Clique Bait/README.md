# Case-Study-6---Clique-Bait
https://8weeksqlchallenge.com/case-study-6/
#### I like to see what the tables look like so here is the sql for the first table
```sql
    SELECT COLUMN_NAME AS event_identifier, DATA_TYPE 
    FROM INFORMATION_SCHEMA.COLUMNS 
    WHERE TABLE_NAME = 'event_identifier';
```
| event_identifier | data_type         |
| ---------------- | ----------------- |
| event_type       | integer           |
| event_name       | character varying |

---

| campaign_identifier | data_type                   |
| ------------------- | --------------------------- |
| campaign_id         | integer                     |
| start_date          | timestamp without time zone |
| end_date            | timestamp without time zone |
| products            | character varying           |
| campaign_name       | character varying           |

---

| page_hierarchy   | data_type         |
| ---------------- | ----------------- |
| page_id          | integer           |
| product_id       | integer           |
| page_name        | character varying |
| product_category | character varying |

---

| users      | data_type                   |
| ---------- | --------------------------- |
| user_id    | integer                     |
| start_date | timestamp without time zone |
| cookie_id  | character varying           |

---

| events          | data_type                   |
| --------------- | --------------------------- |
| page_id         | integer                     |
| event_type      | integer                     |
| sequence_number | integer                     |
| event_time      | timestamp without time zone |
| visit_id        | character varying           |
| cookie_id       | character varying           |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/17)
