## We will continue the following Exercises with data_mart.clean_weekly_sales
### What day of the week is used for each week_date value?
```sql
SELECT DISTINCT to_char(week_date, 'dy') AS day_of_the_week
FROM data_mart.clean_weekly_sales;
```
| day_of_the_week |
| --------------- |
| mon             |

---
### What range of week numbers are missing from the dataset?
```sql
SELECT concat(MAX(week_number)::numeric + 1,' - ', MIN(week_number)::numeric - 1) AS missing_weeks
    ROM data_mart.clean_weekly_sales;
```
| missing_weeks |
| ------------- |
| 37 - 12       |

---
### How many total transactions were there for each year in the dataset?
```sql
SELECT year_number, to_char(SUM(transactions), '999,999,999') AS trasactions_per_year
FROM data_mart.clean_weekly_sales
GROUP BY year_number;
```
| year_number | trasactions_per_year |
| ----------- | -------------------- |
| 2018        |  346,406,460         |
| 2019        |  365,639,285         |
| 2020        |  375,813,651         |

---
### What is the total sales for each region for each month?
```sql
SELECT region, month_number, to_char(SUM(sales), '999,999,999,999') AS sales_per_region_per_month
FROM data_mart.clean_weekly_sales
GROUP BY region, month_number
ORDER BY region, month_number;
```
| region        | month_number | sales_per_region_per_month |
| ------------- | ------------ | -------------------------- |
| AFRICA        | 03           |      567,767,480           |
| AFRICA        | 04           |    1,911,783,504           |
| AFRICA        | 05           |    1,647,244,738           |
| AFRICA        | 06           |    1,767,559,760           |
| AFRICA        | 07           |    1,960,219,710           |
| AFRICA        | 08           |    1,809,596,890           |
| AFRICA        | 09           |      276,320,987           |
| ASIA          | 03           |      529,770,793           |
| ASIA          | 04           |    1,804,628,707           |
| ASIA          | 05           |    1,526,285,399           |
| ASIA          | 06           |    1,619,482,889           |
| ASIA          | 07           |    1,768,844,756           |
| ASIA          | 08           |    1,663,320,609           |
| ASIA          | 09           |      252,836,807           |
| CANADA        | 03           |      144,634,329           |
| CANADA        | 04           |      484,552,594           |
| CANADA        | 05           |      412,378,365           |
| CANADA        | 06           |      443,846,698           |
| CANADA        | 07           |      477,134,947           |
| CANADA        | 08           |      447,073,019           |
| CANADA        | 09           |       69,067,959           |
| EUROPE        | 03           |       35,337,093           |
| EUROPE        | 04           |      127,334,255           |
| EUROPE        | 05           |      109,338,389           |
| EUROPE        | 06           |      122,813,826           |
| EUROPE        | 07           |      136,757,466           |
| EUROPE        | 08           |      122,102,995           |
| EUROPE        | 09           |       18,877,433           |
| OCEANIA       | 03           |      783,282,888           |
| OCEANIA       | 04           |    2,599,767,620           |
| OCEANIA       | 05           |    2,215,657,304           |
| OCEANIA       | 06           |    2,371,884,744           |
| OCEANIA       | 07           |    2,563,459,400           |
| OCEANIA       | 08           |    2,432,313,652           |
| OCEANIA       | 09           |      372,465,518           |
| SOUTH AMERICA | 03           |       71,023,109           |
| SOUTH AMERICA | 04           |      238,451,531           |
| SOUTH AMERICA | 05           |      201,391,809           |
| SOUTH AMERICA | 06           |      218,247,455           |
| SOUTH AMERICA | 07           |      235,582,776           |
| SOUTH AMERICA | 08           |      221,166,052           |
| SOUTH AMERICA | 09           |       34,175,583           |
| USA           | 03           |      225,353,043           |
| USA           | 04           |      759,786,323           |
| USA           | 05           |      655,967,121           |
| USA           | 06           |      703,878,990           |
| USA           | 07           |      760,331,754           |
| USA           | 08           |      712,002,790           |
| USA           | 09           |      110,532,368           |

#### Here we have a cleaner way to show the results
```sql
SELECT month_number AS month, 
     to_char(SUM(sales) FILTER(WHERE region = 'AFRICA'), '999,999,999,999') AS sales_Africa,
     to_char(SUM(sales) FILTER(WHERE region = 'ASIA'), '999,999,999,999') AS sales_ASIA,
     to_char(SUM(sales) FILTER(WHERE region = 'CANADA'), '999,999,999,999') AS sales_CANADA,
     to_char(SUM(sales) FILTER(WHERE region = 'EUROPE'), '999,999,999,999') AS sales_EUROPE,
     to_char(SUM(sales) FILTER(WHERE region = 'OCEANIA'), '999,999,999,999') AS sales_OCEANIA,
     to_char(SUM(sales) FILTER(WHERE region = 'SOUTH AMERICA'), '999,999,999,999') AS sales_SOUTH_AMERICA,
     to_char(SUM(sales) FILTER(WHERE region = 'USA'), '999,999,999,999') AS sales_USA                  
FROM data_mart.clean_weekly_sales
GROUP BY month_number
ORDER BY month_number;
```
| month | sales_africa     | sales_asia       | sales_canada     | sales_europe     | sales_oceania    | sales_south_america | sales_usa        |
| ----- | ---------------- | ---------------- | ---------------- | ---------------- | ---------------- | ------------------- | ---------------- |
| 03    |      567,767,480 |      529,770,793 |      144,634,329 |       35,337,093 |      783,282,888 |       71,023,109    |      225,353,043 |
| 04    |    1,911,783,504 |    1,804,628,707 |      484,552,594 |      127,334,255 |    2,599,767,620 |      238,451,531    |      759,786,323 |
| 05    |    1,647,244,738 |    1,526,285,399 |      412,378,365 |      109,338,389 |    2,215,657,304 |      201,391,809    |      655,967,121 |
| 06    |    1,767,559,760 |    1,619,482,889 |      443,846,698 |      122,813,826 |    2,371,884,744 |      218,247,455    |      703,878,990 |
| 07    |    1,960,219,710 |    1,768,844,756 |      477,134,947 |      136,757,466 |    2,563,459,400 |      235,582,776    |      760,331,754 |
| 08    |    1,809,596,890 |    1,663,320,609 |      447,073,019 |      122,102,995 |    2,432,313,652 |      221,166,052    |      712,002,790 |
| 09    |      276,320,987 |      252,836,807 |       69,067,959 |       18,877,433 |      372,465,518 |       34,175,583    |      110,532,368 |

---
What is the total count of transactions for each platform?
```sql
SELECT platform, to_char(SUM(transactions), '999,999,999,999') AS trasactions_per_platform
FROM data_mart.clean_weekly_sales
GROUP BY platform;
```
| platform | trasactions_per_platform |
| -------- | ------------------------ |
| Shopify  | 5,925,169                |
| Retail   | 1,081,934,227            |

---
### What is the percentage of sales for Retail vs Shopify for each month?
```sql
WITH cte_sales AS
    (SELECT month_number, 
     SUM(sales) FILTER (WHERE platform = 'Retail') AS Retail_sales,
     SUM(sales) FILTER (WHERE platform = 'Shopify') AS Shopify_sales,
     SUM(sales)::numeric AS total_sales
     FROM data_mart.clean_weekly_sales 
     GROUP BY month_number
     )
SELECT month_number, 
    ROUND((retail_sales/total_sales)*100,2) AS retail_sales_in_percentage,
    ROUND((shopify_sales/total_sales)*100,2) AS shopify_sales_in_percentage
FROM cte_sales
ORDER BY month_number;
```
| month_number | retail_sales_in_percentage | shopify_sales_in_percentage |
| ------------ | -------------------------- | --------------------------- |
| 03           | 97.54                      | 2.46                        |
| 04           | 97.59                      | 2.41                        |
| 05           | 97.30                      | 2.70                        |
| 06           | 97.27                      | 2.73                        |
| 07           | 97.29                      | 2.71                        |
| 08           | 97.08                      | 2.92                        |
| 09           | 97.38                      | 2.62                        |

---
### What is the percentage of sales by demographic for each year in the dataset?
```sql
WITH cte_sales AS
    (SELECT year_number, 
     SUM(sales) FILTER (WHERE demographic = 'Couples') AS Couples_sales,
     SUM(sales) FILTER (WHERE demographic = 'Families') AS Families_sales,
     SUM(sales) FILTER (WHERE demographic = 'unknown') AS unknown_sales,
     SUM(sales)::numeric AS total_sales
     FROM data_mart.clean_weekly_sales 
     GROUP BY year_number
     )
SELECT year_number, 
    ROUND((Couples_sales/total_sales)*100,2) AS Couples_sales_in_percentage,
    ROUND((Families_sales/total_sales)*100,2) AS Families_sales_in_percentage,
    ROUND((unknown_sales/total_sales)*100,2) AS unknown_sales_in_percentage
FROM cte_sales
ORDER BY year_number;
```
| year_number | couples_sales_in_percentage | families_sales_in_percentage | unknown_sales_in_percentage |
| ----------- | --------------------------- | ---------------------------- | --------------------------- |
| 2018        | 26.38                       | 31.99                        | 41.63                       |
| 2019        | 27.28                       | 32.47                        | 40.25                       |
| 2020        | 28.72                       | 32.73                        | 38.55                       |

---
### Which age_band and demographic values contribute the most to Retail sales?
#### Except for the unknown the Retirees Families and Couples contribute the most
```sql
SELECT age_band, demographic, to_char(SUM(sales), '999,999,999,999') AS segment_sales
FROM data_mart.clean_weekly_sales
GROUP BY age_band, demographic
ORDER BY segment_sales desc;
```
| age_band     | demographic | segment_sales    |
| ------------ | ----------- | ---------------- |
| unknown      | unknown     |   16,338,612,234 |
| Retirees     | Families    |    6,750,457,132 |
| Retirees     | Couples     |    6,531,115,070 |
| Middle Age   | Families    |    4,556,141,618 |
| Young Adults | Couples     |    2,679,593,130 |
| Middle Age   | Couples     |    1,990,499,351 |
| Young Adults | Families    |    1,897,215,692 |

---
### Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?
#### No, the column avg_transaction shows us the average sales per transaction
```sql
SELECT year_number AS year,
   to_char(ROUND(AVG(transactions) FILTER (WHERE platform = 'Retail')), '999,999') AS avg_Retail_transactions,
   to_char(ROUND(AVG(transactions) FILTER (WHERE platform = 'Shopify')), '999,999') AS avg_Shopify_transactions
FROM data_mart.clean_weekly_sales
GROUP BY year;
```
| year | avg_retail_transactions | avg_shopify_transactions |
| ---- | ----------------------- | ------------------------ |
| 2018 |  120,770                |      523                 |
| 2019 |  127,360                |      666                 |
| 2020 |  130,698                |      889                 |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/8)
