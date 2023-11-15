### In a single query, perform the following operations and generate a new table in the data_mart schema named clean_weekly_sales:

- Convert the week_date to a DATE format

- Add a week_number as the second column for each week_date value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc

- Add a month_number with the calendar month for each week_date value as the 3rd column

- Add a calendar_year column as the 4th column containing either 2018, 2019 or 2020 values

- Add a new column called age_band after the original segment column using the following mapping on the number inside the segment value

|segment |age_band    |
| ------ | ---------- |
|1	 |Young Adults|
|2	 |Middle Aged |
|3 or 4	 |Retirees    |

-Add a new demographic column using the following mapping for the first letter in the segment values:
|segment|demographic|
| ----- | --------- |
|C      |Couples    |
|F      |Families   |

- Ensure all null string values with an "unknown" string value in the original segment column as well as the new age_band and demographic columns

- Generate a new avg_transaction column as the sales value divided by transactions rounded to 2 decimal places for each record

```sql
DROP TABLE IF EXISTS data_mart.clean_weekly_sales;
CREATE TABLE data_mart.clean_weekly_sales AS
SELECT 
	to_date(week_date, 'DD/MM/YY') AS week_date, 
	to_char(to_date(week_date, 'DD/MM/YY'), 'IW') AS week_number, 
	to_char(to_date(week_date, 'DD/MM/YY'), 'MM') AS month_number, 
	to_char(to_date(week_date, 'DD/MM/YY'), 'IYYY') AS year_number,
	region, 
    platform, 
    CASE WHEN segment = 'null' THEN 'unknown'
    	 ELSE segment 
         END AS segment, 
    CASE WHEN segment like '%1' THEN 'Young Adults'
         WHEN segment like '%2' THEN 'Middle Age'
         WHEN segment like '%3' OR segment like '%4' THEN 'Retirees'
         ELSE 'unknown' 
         END AS age_band,
    CASE WHEN segment like 'C%' THEN 'Couples'
         WHEN segment like 'F%' THEN 'Families'
         ELSE 'unknown' 
         END AS demographic,
    customer_type, 
    transactions, 
    ROUND(sales/transactions,2) AS avg_transactions,
    sales
FROM data_mart.weekly_sales;

SELECT * FROM data_mart.clean_weekly_sales limit 10;
```
### For the first 10 rows it looks like this

| week_date                | week_number | month_number | year_number | region | platform | segment | age_band     | demographic | customer_type | transactions | avg_transactions | sales    |
| ------------------------ | ----------- | ------------ | ----------- | ------ | -------- | ------- | ------------ | ----------- | ------------- | ------------ | ---------------- | -------- |
| 2020-08-31T00:00:00.000Z | 36          | 08           | 2020        | ASIA   | Retail   | C3      | Retirees     | Couples     | New           | 120631       | 30.00            | 3656163  |
| 2020-08-31T00:00:00.000Z | 36          | 08           | 2020        | ASIA   | Retail   | F1      | Young Adults | Families    | New           | 31574        | 31.00            | 996575   |
| 2020-08-31T00:00:00.000Z | 36          | 08           | 2020        | USA    | Retail   | unknown | unknown      | unknown     | Guest         | 529151       | 31.00            | 16509610 |
| 2020-08-31T00:00:00.000Z | 36          | 08           | 2020        | EUROPE | Retail   | C1      | Young Adults | Couples     | New           | 4517         | 31.00            | 141942   |
| 2020-08-31T00:00:00.000Z | 36          | 08           | 2020        | AFRICA | Retail   | C2      | Middle Age   | Couples     | New           | 58046        | 30.00            | 1758388  |
| 2020-08-31T00:00:00.000Z | 36          | 08           | 2020        | CANADA | Shopify  | F2      | Middle Age   | Families    | Existing      | 1336         | 182.00           | 243878   |
| 2020-08-31T00:00:00.000Z | 36          | 08           | 2020        | AFRICA | Shopify  | F3      | Retirees     | Families    | Existing      | 2514         | 206.00           | 519502   |
| 2020-08-31T00:00:00.000Z | 36          | 08           | 2020        | ASIA   | Shopify  | F1      | Young Adults | Families    | Existing      | 2158         | 172.00           | 371417   |
| 2020-08-31T00:00:00.000Z | 36          | 08           | 2020        | AFRICA | Shopify  | F2      | Middle Age   | Families    | New           | 318          | 155.00           | 49557    |
| 2020-08-31T00:00:00.000Z | 36          | 08           | 2020        | AFRICA | Retail   | C3      | Retirees     | Couples     | New           | 111032       | 35.00            | 3888162  |

---
