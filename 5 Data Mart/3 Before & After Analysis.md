### This technique is usually used when we inspect an important event and want to inspect the impact before and after a certain point in time. Taking the week_date value of 2020-06-15 as the baseline week where the Data Mart sustainable packaging changes came into effect. We would include all week_date values for 2020-06-15 as the start of the period after the change and the previous week_date values would be before
### Using this analysis approach - answer the following questions:
---
### What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?
#### Comparing the 4 weeks before and after the change we lost 1.15% of the total sales.
```sql
WITH cte_sales AS
    (SELECT
     SUM(sales) FILTER(WHERE week_date >= '2020-05-18' AND week_date < '2020-06-15')::numeric AS sales_before_20200615,
     SUM(sales) FILTER(WHERE week_date >= '2020-06-15' AND week_date < '2020-07-13' ) AS sales_after_20200615
     FROM data_mart.clean_weekly_sales
    )
SELECT
    to_char(sales_before_20200615, '999,999,999,999') AS sales_before_20200615,
    to_char(sales_after_20200615, '999,999,999,999') AS sales_after_20200615,  
    ROUND((sales_after_20200615-sales_before_20200615)/sales_before_20200615*100,2) AS growth_ratio
FROM cte_sales;
```
| sales_before_20200615 | sales_after_20200615 | growth_ratio |
| --------------------- | -------------------- | ------------ |
| 2,345,878,357         | 2,318,994,169        | -1.15        |

---
#### Here we show the percentage of the sales before and after for the 4 week periods
```sql
WITH cte_sales AS
    (SELECT
     SUM(sales) FILTER(WHERE week_date >= '2020-05-18' AND week_date < '2020-06-15') AS sales_before_20200615,
     SUM(sales) FILTER(WHERE week_date >= '2020-06-15' AND week_date < '2020-07-13') AS sales_after_20200615,
     SUM(sales) FILTER(WHERE week_date >= '2020-05-18' AND week_date < '2020-07-13')::numeric AS total_sales
     FROM data_mart.clean_weekly_sales
    )
SELECT
    ROUND((sales_before_20200615/total_sales)*100,2) AS sales_before_20200615,
    ROUND((sales_after_20200615/total_sales)*100,2) AS sales_after_20200615
FROM cte_sales;
```
| sales_before_20200615 | sales_after_20200615 |
| --------------------- | -------------------- |
| 50.29                 | 49.71                |

---
### What about the entire 12 weeks before and after?
#### Comparing the 12 weeks before and after the change we lost 2.14% of the total sales
```sql
WITH cte_sales AS
    (SELECT
     SUM(sales) FILTER(WHERE week_date >= '2020-03-23' AND week_date < '2020-06-15')::numeric AS sales_before_20200615,
     SUM(sales) FILTER(WHERE week_date >= '2020-06-15' AND week_date <= '2020-08-31') AS sales_after_20200615
     FROM data_mart.clean_weekly_sales
    )
SELECT
    to_char(sales_before_20200615, '999,999,999,999') AS sales_before_20200615,
    to_char(sales_after_20200615, '999,999,999,999') AS sales_after_20200615,  
    ROUND((sales_after_20200615-sales_before_20200615)/sales_before_20200615*100,2) AS growth_ratio
FROM cte_sales;
```
| sales_before_20200615 | sales_after_20200615 | growth_ratio |
| --------------------- | -------------------- | ------------ |
| 7,126,273,147         | 6,973,947,753        | -2.14        |

---
#### Here we show the percentage of the sales before and after for the 12 week periods
```sql
WITH cte_sales AS
    (SELECT
     SUM(sales) FILTER(WHERE week_date >= '2020-03-23' AND week_date < '2020-06-15') AS sales_before_20200615,
     SUM(sales) FILTER(WHERE week_date >= '2020-06-15' AND week_date <= '2020-08-31') AS sales_after_20200615,
     SUM(sales) FILTER(WHERE week_date >= '2020-03-23' AND week_date <= '2020-08-31')::numeric AS total_sales
     FROM data_mart.clean_weekly_sales
    )
SELECT
    ROUND((sales_before_20200615/total_sales)*100,2) AS sales_before_20200615,
    ROUND((sales_after_20200615/total_sales)*100,2) AS sales_after_20200615
FROM cte_sales;
```
| sales_before_20200615 | sales_after_20200615 |
| --------------------- | -------------------- |
| 50.54                 | 49.46                |

#### In clonclusin we can say that the change didn't really make any difference to the sales.

---
---

### How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?
#### I did't really know what I should show but here you go.
- 4 week period for 15-06-2018
```sql
WITH cte_sales AS
    (SELECT 
     SUM(sales) FILTER(WHERE week_date >= '2018-05-18' AND week_date < '2018-06-15')::numeric AS sales_before_20180615,
     SUM(sales) FILTER(WHERE week_date >= '2018-06-15' AND week_date < '2018-07-13' ) AS sales_after_20180615
     FROM data_mart.clean_weekly_sales
    )
SELECT
    to_char(sales_before_20200615, '999,999,999,999') AS sales_before_20200615,
    to_char(sales_after_20200615, '999,999,999,999') AS sales_after_20200615,  
    ROUND((sales_after_20180615-sales_before_20180615)/sales_before_20180615*100,2) AS growth_ratio_4_weeks
FROM cte_sales;
```
| sales_before_20180615 | sales_after_20180615 | growth_ratio_4_weeks |
| --------------------- | -------------------- | -------------------- |
| 2,125,140,809         | 2,129,242,914        | 0.19                 |

---
- 12 week period for 15-06-2018
```sql
WITH cte_sales AS
    (SELECT 
     SUM(sales) FILTER(WHERE week_date >= '2018-03-23' AND week_date < '2018-06-15')::numeric AS sales_before_20180615,
     SUM(sales) FILTER(WHERE week_date >= '2018-06-15' AND week_date <= '2018-08-31') AS sales_after_20180615
     FROM data_mart.clean_weekly_sales
    )
SELECT
    to_char(sales_before_20200615, '999,999,999,999') AS sales_before_20200615,
    to_char(sales_after_20200615, '999,999,999,999') AS sales_after_20200615,  
    ROUND((sales_after_20180615-sales_before_20180615)/sales_before_20180615*100,2) AS growth_ratio_12_weeks
FROM cte_sales;
```
| sales_before_20180615 | sales_after_20180615 | growth_ratio_12_weeks |
| --------------------- | -------------------- | --------------------- |
| 6,396,562,317         | 5,947,847,148        | -7.01                 |

---
- 4 week period for 15-06-2019
```sql
WITH cte_sales AS
    (SELECT 
     SUM(sales) FILTER(WHERE week_date >= '2019-05-18' AND week_date < '2019-06-15')::numeric AS sales_before_20190615,
     SUM(sales) FILTER(WHERE week_date >= '2019-06-15' AND week_date < '2019-07-13' ) AS sales_after_20190615
     FROM data_mart.clean_weekly_sales
    )
SELECT
    to_char(sales_before_20200615, '999,999,999,999') AS sales_before_20200615,
    to_char(sales_after_20200615, '999,999,999,999') AS sales_after_20200615,  
    ROUND((sales_after_20190615-sales_before_20190615)/sales_before_20190615*100,2) AS growth_ratio_4_weeks
FROM cte_sales;
```
| sales_before_20190615 | sales_after_20190615 | growth_ratio_4_weeks |
| --------------------- | -------------------- | -------------------- |
| 2,249,989,796         | 2,252,326,390        | 0.10                 |

---
- 12 week period for 15-06-2019
```sql
WITH cte_sales AS
    (SELECT 
     SUM(sales) FILTER(WHERE week_date >= '2019-03-23' AND week_date < '2019-06-15')::numeric AS sales_before_20190615,
     SUM(sales) FILTER(WHERE week_date >= '2019-06-15' AND week_date <= '2019-08-31') AS sales_after_20190615
     FROM data_mart.clean_weekly_sales
    )
SELECT
    to_char(sales_before_20200615, '999,999,999,999') AS sales_before_20200615,
    to_char(sales_after_20200615, '999,999,999,999') AS sales_after_20200615,  
    ROUND((sales_after_20190615-sales_before_20190615)/sales_before_20190615*100,2) AS growth_ratio_12_weeks
FROM cte_sales;
```
| sales_before_20190615 | sales_after_20190615 | growth_ratio_12_weeks |
| --------------------- | -------------------- | --------------------- |
| 6,883,386,397         | 6,281,340,810        | -8.75                 |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/8)
