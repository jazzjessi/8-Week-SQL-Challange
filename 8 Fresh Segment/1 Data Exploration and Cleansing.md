### Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month
```sql 
    ALTER TABLE fresh_segments.interest_metrics
    ALTER COLUMN month_year type date
    USING to_date(month_year, 'MM-YYYY');
```
---
### What is count of records in the fresh_segments.interest_metrics for each month_year value sorted in chronological order (earliest to latest) with the null values appearing first?
#### I don't know how to make the null value appear first.
```sql
    SELECT month_year, COUNT(*)
    FROM fresh_segments.interest_metrics
    GROUP BY month_year
    ORDER BY month_year asc;
```
| month_year               | count |
| ------------------------ | ----- |
| 2018-07-01T00:00:00.000Z | 729   |
| 2018-08-01T00:00:00.000Z | 767   |
| 2018-09-01T00:00:00.000Z | 780   |
| 2018-10-01T00:00:00.000Z | 857   |
| 2018-11-01T00:00:00.000Z | 928   |
| 2018-12-01T00:00:00.000Z | 995   |
| 2019-01-01T00:00:00.000Z | 973   |
| 2019-02-01T00:00:00.000Z | 1121  |
| 2019-03-01T00:00:00.000Z | 1136  |
| 2019-04-01T00:00:00.000Z | 1099  |
| 2019-05-01T00:00:00.000Z | 857   |
| 2019-06-01T00:00:00.000Z | 824   |
| 2019-07-01T00:00:00.000Z | 864   |
| 2019-08-01T00:00:00.000Z | 1149  |
|                          | 1194  |

---
### What do you think we should do with these null values in the fresh_segments.interest_metrics
#### We can see that except for one row all the interest_id's are null too. So we simply delete them.
```sql 
    SELECT *
    FROM fresh_segments.interest_metrics
    WHERE month_year IS null 
    ORDER BY interest_id LIMIT 10;
```
| _month | _year | month_year | interest_id | composition | index_value | ranking | percentile_ranking |
| ------ | ----- | ---------- | ----------- | ----------- | ----------- | ------- | ------------------ |
|        |       |            | 21246       | 1.61        | 0.68        | 1191    | 0.25               |
|        |       |            |             | 6.82        | 2.84        | 45      | 96.23              |
|        |       |            |             | 5.46        | 2.81        | 51      | 95.73              |
|        |       |            |             | 7.73        | 2.82        | 48      | 95.98              |
|        |       |            |             | 5.37        | 2.82        | 48      | 95.98              |
|        |       |            |             | 6.15        | 2.82        | 48      | 95.98              |
|        |       |            |             | 5.96        | 2.83        | 47      | 96.06              |
|        |       |            |             | 7.13        | 2.84        | 45      | 96.23              |
|        |       |            |             | 5.99        | 2.44        | 113     | 90.54              |
|        |       |            |             | 6.12        | 2.85        | 43      | 96.4               |


```sql 
    DELETE FROM fresh_segments.interest_metrics
    WHERE interest_id IS null;
```
---
### How many interest_id values exist in the fresh_segments.interest_metrics table but not in the fresh_segments.interest_map table? What about the other way around?
### Summarise the id values in the fresh_segments.interest_map by its total record count in this table
```sql 
    SELECT 
    	COUNT(DISTINCT id) AS map_id_count,
    	COUNT(DISTINCT interest_id) AS metrics_id_count,
    	SUM(CASE WHEN id IS NULL THEN 1 END) AS not_in_metric,
    	SUM(CASE WHEN interest_id is NULL THEN 1 END) AS not_in_map
    FROM fresh_segments.interest_metrics
    FULL JOIN fresh_segments.interest_map ON interest_id::INT = id;
```
| map_id_count | metrics_id_count | not_in_metric | not_in_map |
| ------------ | ---------------- | ------------- | ---------- |
| 1209         | 1202             |               | 7          |

---
### What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where interest_id = 21246 in your joined output and include all columns from fresh_segments.interest_metrics and all columns from fresh_segments.interest_map except from the id column.
```sql 
    SELECT 
    	_month,
      	_year,
      	month_year,
      	interest_id,
      	composition,
      	index_value,
      	ranking,
      	percentile_ranking,
    	interest_name,
     	interest_summary,
    	created_at,
     	last_modified
    FROM fresh_segments.interest_metrics
    JOIN fresh_segments.interest_map ON interest_id::INT = id
    WHERE interest_id = '21246';
```
| _month | _year | month_year               | interest_id | composition | index_value | ranking | percentile_ranking | interest_name                    | interest_summary                                      | created_at               | last_modified            |
| ------ | ----- | ------------------------ | ----------- | ----------- | ----------- | ------- | ------------------ | -------------------------------- | ----------------------------------------------------- | ------------------------ | ------------------------ |
|        |       |                          | 21246       | 1.61        | 0.68        | 1191    | 0.25               | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 4      | 2019  | 2019-04-01T00:00:00.000Z | 21246       | 1.58        | 0.63        | 1092    | 0.64               | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 3      | 2019  | 2019-03-01T00:00:00.000Z | 21246       | 1.75        | 0.67        | 1123    | 1.14               | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 2      | 2019  | 2019-02-01T00:00:00.000Z | 21246       | 1.84        | 0.68        | 1109    | 1.07               | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 1      | 2019  | 2019-01-01T00:00:00.000Z | 21246       | 2.05        | 0.76        | 954     | 1.95               | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 12     | 2018  | 2018-12-01T00:00:00.000Z | 21246       | 1.97        | 0.7         | 983     | 1.21               | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 11     | 2018  | 2018-11-01T00:00:00.000Z | 21246       | 2.25        | 0.78        | 908     | 2.16               | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 10     | 2018  | 2018-10-01T00:00:00.000Z | 21246       | 1.74        | 0.58        | 855     | 0.23               | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 9      | 2018  | 2018-09-01T00:00:00.000Z | 21246       | 2.06        | 0.61        | 774     | 0.77               | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 8      | 2018  | 2018-08-01T00:00:00.000Z | 21246       | 2.13        | 0.59        | 765     | 0.26               | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 7      | 2018  | 2018-07-01T00:00:00.000Z | 21246       | 2.26        | 0.65        | 722     | 0.96               | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |

---
### Are there any records in your joined table where the month_year value is before the created_at value from the fresh_segments.interest_map table? Do you think these values are valid and why?
#### Yes, we have 188 records where the month_year value is before the created_at value. In Step one we set month_year value to the first of every month by default, that's why I checked if we have any 'real' month_year value's before the created_at value. That's not the case so all the values are valid.
```sql 
    WITH cte_m1 AS 
    (SELECT COUNT(*) AS month_year_before_create
     FROM fresh_segments.interest_metrics
     JOIN fresh_segments.interest_map ON interest_id::INT = id
     WHERE month_year < created_at
    ),
     cte_m2 AS
     (SELECT COUNT(*) AS real_month_year_before_create
     FROM fresh_segments.interest_metrics
     JOIN fresh_segments.interest_map ON interest_id::INT = id
     WHERE (month_year + interval '1 month') < created_at
    )
    SELECT month_year_before_create, real_month_year_before_create
    FROM cte_m1, cte_m2;
```
| month_year_before_create | real_month_year_before_create |
| ------------------------ | ----------------------------- |
| 188                      | 0                             |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/iRdsT76vaus813crPP8Ma4/10)
