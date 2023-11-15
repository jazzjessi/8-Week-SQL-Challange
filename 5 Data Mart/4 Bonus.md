### Which areas of the business have the highest negative impact in sales metrics performance in 2020 for the 12 week before and after period?
---
- region
```sql
WITH cte_sales AS
    (SELECT region,
     SUM(sales) FILTER(WHERE week_date >= '2020-03-23' AND week_date < '2020-06-15')::numeric AS sales_before_20200615,
     SUM(sales) FILTER(WHERE week_date >= '2020-06-15' AND week_date <= '2020-08-31') AS sales_after_20200615
     FROM data_mart.clean_weekly_sales
     GROUP BY region
    )
SELECT
    region,
    to_char(sales_before_20200615, '999,999,999,999') AS sales_before_20200615,
    to_char(sales_after_20200615, '999,999,999,999') AS sales_after_20200615,  
    ROUND((sales_after_20200615-sales_before_20200615)/sales_before_20200615*100,2) AS growth_ratio
FROM cte_sales
GROUP BY region, sales_before_20200615, sales_after_20200615
ORDER BY growth_ratio;
```
| region        | sales_before_20200615 | sales_after_20200615 | growth_ratio |
| ------------- | --------------------- | -------------------- | ------------ |
| ASIA          |    1,637,244,466      |    1,583,807,621     | -3.26        |
| OCEANIA       |    2,354,116,790      |    2,282,795,690     | -3.03        |
| SOUTH AMERICA |      213,036,207      |      208,452,033     | -2.15        |
| CANADA        |      426,438,454      |      418,264,441     | -1.92        |
| USA           |      677,013,558      |      666,198,715     | -1.60        |
| AFRICA        |    1,709,537,105      |    1,700,390,294     | -0.54        |
| EUROPE        |      108,886,567      |      114,038,959     | 4.73         |

---
- platform
```sql
WITH cte_sales AS
    (SELECT platform,
     SUM(sales) FILTER(WHERE week_date >= '2020-03-23' AND week_date < '2020-06-15')::numeric AS sales_before_20200615,
     SUM(sales) FILTER(WHERE week_date >= '2020-06-15' AND week_date <= '2020-08-31') AS sales_after_20200615
     FROM data_mart.clean_weekly_sales
     GROUP BY platform
    )
SELECT
    platform,
    to_char(sales_before_20200615, '999,999,999,999') AS sales_before_20200615,
    to_char(sales_after_20200615, '999,999,999,999') AS sales_after_20200615,  
    ROUND((sales_after_20200615-sales_before_20200615)/sales_before_20200615*100,2) AS growth_ratio
FROM cte_sales
GROUP BY platform, sales_before_20200615, sales_after_20200615
ORDER BY growth_ratio;
```
| platform | sales_before_20200615 | sales_after_20200615 | growth_ratio |
| -------- | --------------------- | -------------------- | ------------ |
| Retail   |    6,906,861,113      |    6,738,777,279     | -2.43        |
| Shopify  |      219,412,034      |      235,170,474     | 7.18         |

---
- age_band
```sql
WITH cte_sales AS
    (SELECT age_band,
     SUM(sales) FILTER(WHERE week_date >= '2020-03-23' AND week_date < '2020-06-15')::numeric AS sales_before_20200615,
     SUM(sales) FILTER(WHERE week_date >= '2020-06-15' AND week_date <= '2020-08-31') AS sales_after_20200615
     FROM data_mart.clean_weekly_sales
     GROUP BY age_band
    )
SELECT
    age_band,
    to_char(sales_before_20200615, '999,999,999,999') AS sales_before_20200615,
    to_char(sales_after_20200615, '999,999,999,999') AS sales_after_20200615, 
    ROUND((sales_after_20200615-sales_before_20200615)/sales_before_20200615*100,2) AS growth_ratio
FROM cte_sales
GROUP BY age_band, sales_before_20200615, sales_after_20200615
ORDER BY growth_ratio;
```
| age_band     | sales_before_20200615 | sales_after_20200615 | growth_ratio |
| ------------ | --------------------- | -------------------- | ------------ |
| unknown      |    2,764,354,464      |    2,671,961,443     | -3.34        |
| Middle Age   |    1,164,847,640      |    1,141,853,348     | -1.97        |
| Retirees     |    2,395,264,515      |    2,365,714,994     | -1.23        |
| Young Adults |      801,806,528      |      794,417,968     | -0.92        |

---
- demographic
```sql
WITH cte_sales AS
    (SELECT demographic,
     SUM(sales) FILTER(WHERE week_date >= '2020-03-23' AND week_date < '2020-06-15')::numeric AS sales_before_20200615,
     SUM(sales) FILTER(WHERE week_date >= '2020-06-15' AND week_date <= '2020-08-31') AS sales_after_20200615
     FROM data_mart.clean_weekly_sales
     GROUP BY demographic
    )
SELECT
    demographic,
    to_char(sales_before_20200615, '999,999,999,999') AS sales_before_20200615,
    to_char(sales_after_20200615, '999,999,999,999') AS sales_after_20200615, 
    ROUND((sales_after_20200615-sales_before_20200615)/sales_before_20200615*100,2) AS growth_ratio
FROM cte_sales
GROUP BY demographic, sales_before_20200615, sales_after_20200615
ORDER BY growth_ratio;
```
| demographic | sales_before_20200615 | sales_after_20200615 | growth_ratio |
| ----------- | --------------------- | -------------------- | ------------ |
| unknown     |    2,764,354,464      |    2,671,961,443     | -3.34        |
| Families    |    2,328,329,040      |    2,286,009,025     | -1.82        |
| Couples     |    2,033,589,643      |    2,015,977,285     | -0.87        |

---
- customer_type
```sql
WITH cte_sales AS
    (SELECT customer_type,
     SUM(sales) FILTER(WHERE week_date >= '2020-03-23' AND week_date < '2020-06-15')::numeric AS sales_before_20200615,
     SUM(sales) FILTER(WHERE week_date >= '2020-06-15' AND week_date <= '2020-08-31') AS sales_after_20200615
     FROM data_mart.clean_weekly_sales
     GROUP BY customer_type
    )
SELECT
    customer_type,
    to_char(sales_before_20200615, '999,999,999,999') AS sales_before_20200615,
    to_char(sales_after_20200615, '999,999,999,999') AS sales_after_20200615,  
    ROUND((sales_after_20200615-sales_before_20200615)/sales_before_20200615*100,2) AS growth_ratio
FROM cte_sales
GROUP BY customer_type, sales_before_20200615, sales_after_20200615
ORDER BY growth_ratio;
```
| customer_type | sales_before_20200615 | sales_after_20200615 | growth_ratio |
| ------------- | --------------------- | -------------------- | ------------ |
| Guest         |    2,573,436,301      |    2,496,233,635     | -3.00        |
| Existing      |    3,690,116,427      |    3,606,243,454     | -2.27        |
| New           |      862,720,419      |      871,470,664     | 1.01         |

---

### Do you have any further recommendations for Danny’s team at Data Mart or any interesting insights based off this analysis?
#### We can see that the change to charge plastic bags had a positive effect only in europe. We can also see, that the sales in shopify increased, so it could be that the customers didn't change their place to shop but only changed to online shopping. The sales also increased for new customers.

[View on DB Fiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/8)
