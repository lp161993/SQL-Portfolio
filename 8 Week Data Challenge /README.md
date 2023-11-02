# Project 1 - Danny Data Mart Challenge
Let us help Danny analyse how well Data Mart is contributing to his pockets after a major supply chain change!

### Introduction: 
https://8weeksqlchallenge.com/case-study-5/ is the link which provides the data challenge related details.
In short: Data Mart is a supermarket in Australia and Danny, the owner, has made major supply chain changes in 2020. Let's help Danny analyse the effects of this change in the sales performance from the sales data given. 

### Data Cleanup:

Tasks at hand: 
In a single query, perform the following operations and generate a new table in the data_mart schema named clean_weekly_sales:

- Convert the week_date to a DATE format

- Add a week_number as the second column for each week_date value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc

- Add a month_number with the calendar month for each week_date value as the 3rd column

- Add a calendar_year column as the 4th column containing either 2018, 2019 or 2020 values

- Add a new column called age_band after the original segment column using the following mapping on the number inside the segment value

- Segment	age_band
  - 1	(Young Adults)
  - 2	(Middle Aged)
  - 3 or 4	(Retirees)
- Add a new demographic column using the following mapping for the first letter in the segment values:
segment	demographic
  - C	(Couples)
  - F	(Families)
- Ensure all null string values with an "unknown" string value in the original segment column as well as the new age_band and demographic columns

- Generate a new avg_transaction column as the sales value divided by transactions rounded to 2 decimal places for each record

``` sql
CREATE TABLE data_mart.clean_weekly_sales AS (SELECT
 TO_DATE(week_date, 'DD-MM-YY') as week_date,
 DATE_PART('W', TO_DATE(week_date, 'DD-MM-YY')) as week_number,
 DATE_PART('MON', TO_DATE(week_date, 'DD-MM-YY')) as month_number,
 DATE_PART('Y', TO_DATE(week_date, 'DD-MM-YY')) as calendar_year,
 CASE
  WHEN RIGHT(segment,1)='1' THEN 'Young Adults'
  WHEN RIGHT(segment,1)='2' THEN 'Middle_Aged'
  WHEN RIGHT(segment,1) IN ('3','4') THEN 'Retirees'
  ELSE 'UNKNOWN'  
 END AS age_band,
 CASE
  WHEN LEFT(segment,1)='C' THEN 'couples'
  WHEN LEFT(segment,1)='F' THEN 'family'
  ELSE 'UNKNOWN'
 END AS demographic,
 CASE
   WHEN segment = 'null' THEN 'UNKNOWN'
   ELSE segment
 END AS segment,                                            
 region,
 platform,
 customer_type,
 transactions,
 sales,
 ROUND((sales*1.0)/transactions, 2) AS avg_transaction            
FROM data_mart.weekly_sales);
```
Here are a sample of 10 rows after the cleanup of the dataset

|week_date|week_number|month_number|calendar_year|age_band|demographic|segment|region|platform|customer_type|transactions|sales|avg_transaction|
|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|
|2020-08-31T00:00:00.000Z|36|8|2020|Retirees|couples|C3|ASIA|Retail|New|120631|3656163|30.31|
|2020-08-31T00:00:00.000Z|36|8|2020|Young Adults|family|F1|ASIA|Retail|New|31574|996575|31.56|
|2020-08-31T00:00:00.000Z|36|8|2020|UNKNOWN|UNKNOWN|UNKNOWN|USA|Retail|Guest|529151|16509610|31.20|
|2020-08-31T00:00:00.000Z|36|8|2020|Young Adults|couples|C1|EUROPE|Retail|New|4517|141942|31.42|
|2020-08-31T00:00:00.000Z|36|8|2020|Middle_Aged|couples|C2|AFRICA|Retail|New|58046|1758388|30.29|
|2020-08-31T00:00:00.000Z|36|8|2020|Middle_Aged|family|F2|CANADA|Shopify|Existing|1336|243878|182.54|
|2020-08-31T00:00:00.000Z|36|8|2020|Retirees|family|F3|AFRICA|Shopify|Existing|2514|519502|206.64|
|2020-08-31T00:00:00.000Z|36|8|2020|Young Adults|family|F1|ASIA|Shopify|Existing|2158|371417|172.11|
|2020-08-31T00:00:00.000Z|36|8|2020|Middle_Aged|family|F2|AFRICA|Shopify|New|318|49557|155.84|
|2020-08-31T00:00:00.000Z|36|8|2020|Retirees|couples|C3|AFRICA|Retail|New|111032|3888162|35.02|

### Data Exploration 
**1. What day of the week is used for each week_date value?**

``` sql
SELECT DISTINCT(TO_CHAR(week_date, 'day')) as Day_Used_for_Week_Date
FROM data_mart.clean_weekly_sales;
```
|day_used_for_week_date|
|:----|
|monday|

**2. What range of week numbers are missing from the dataset?**
``` sql
SELECT * from generate_series(1,52,1) as missing_week_numbers
WHERE missing_week_numbers NOT IN (SELECT DISTINCT(week_number) FROM data_mart.clean_weekly_sales);
```
There are a total of 28 rows, shown below are top 4 rows only.

|missing_week_numbers|
|:----|
|1|
|2|
|3|
|4|

**3. How many total transactions were there for each year in the dataset?**
 
``` sql
SELECT calendar_year, sum(transactions) as Total_Transactions
FROM data_mart.clean_weekly_sales
GROUP BY calendar_year
ORDER BY calendar_year;
```

|calendar_year|total_transactions|
|:----|:----|
|2018|346406460|
|2019|365639285|
|2020|375813651|

**4. What is the total sales for each region for each month?**

``` sql
SELECT region, month_number, sum(sales) as Total_Sales
FROM data_mart.clean_weekly_sales
GROUP BY region, month_number
ORDER BY region, month_number;
```
Shown below is a sample of 7 rows, one for each region for the month of March. The original table has 49 rows in total.
|region|month_number|total_sales|
|:---:|:---:|:---:|
|AFRICA|3|567767480|
|ASIA|3|529770793|
|CANADA|3|144634329|
|EUROPE|3|35337093|
|OCEANIA|3|783282888|
|SOUTH AMERICA|3|71023109|
|USA|3|225353043|

**5. What is the total count of transactions for each platform?**
 
``` sql
SELECT platform, sum(transactions) as Total_Transactions
FROM data_mart.clean_weekly_sales
GROUP BY platform;
```
|platform|total_transactions|
|:----|:----|
|Shopify|5925169|
|Retail|1081934227|


**6. What is the percentage of sales for Retail vs Shopify for each month?**

Let's solve this by using windown function(PARTITION BY)
``` sql
WITH ALPHA AS (
SELECT DISTINCT calendar_year, month_number, platform, ROUND(100.0* SUM(sales) OVER (PARTITION BY calendar_year,month_number, platform)/ SUM(sales) OVER (PARTITION BY calendar_year,month_number), 2) as sales
FROM data_mart.clean_weekly_sales
ORDER BY calendar_year, month_number)

SELECT calendar_year, month_number, sales as Retail_Sales, (100.0-sales) as Shopify_sales
FROM alpha
WHERE platform='Retail';
```
Given below is a sample of 3 rows.

|calendar_year|month_number|retail_sales|shopify_sales|
|----:|----:|----:|----:|
|2018|3|97.92|2.08|
|2018|4|97.93|2.07|
|2018|5|97.73|2.27|


**7. What is the percentage of sales by demographic for each year in the dataset?**
 
``` sql
WITH Beta AS(
SELECT DISTINCT calendar_year, demographic, ROUND(100.0* SUM(sales) OVER (PARTITION BY calendar_year, demographic) / SUM(sales) OVER
(PARTITION BY calendar_year), 2) as Sales_by_Demographic
FROM data_mart.clean_weekly_sales
ORDER BY calendar_year, demographic)

SELECT B1.calendar_year,
B1.sales_by_demographic as sales_percent_unknown,
B2.sales_by_demographic as sales_percent_family,
B3.sales_by_demographic as sales_percent_couples
FROM BETA B1
JOIN BETA B2 ON B1.calendar_year = B2.calendar_year
JOIN BETA B3 ON B2.calendar_year = B3.calendar_year
WHERE B1.demographic='UNKNOWN' and  B2.demographic='family' and B3.demographic='couples';
```

|calendar_year|sales_percent_unknown|sales_percent_family|sales_percent_couples|
|:---:|:---:|:---:|:---:|
|2018|41.63|31.99|26.38|
|2019|40.25|32.47|27.28|
|2020|38.55|32.73|28.72|

**8. Which age_band and demographic values contribute the most to Retail sales?**
 
``` sql
SELECT D.age_band, D.demographic, sum(D.sales) as total_sales
from data_mart.clean_weekly_sales D
where D.platform='Retail'
group by D.age_band, D.demographic
ORDER BY TOTAL_SALES DESC
```
As seen below, the highest sales is not specific to a age_band or demographic, the second row shows that the retired family people have the second highest total sales.
|age_band|demographic|total_sales|
|:---:|:---:|:---:|
|UNKNOWN|UNKNOWN|16067285533|
|Retirees|family|6634686916|
|Retirees|couples|6370580014|
|Middle_Aged|family|4354091554|
|Young Adults|couples|2602922797|
|Middle_Aged|couples|1854160330|
|Young Adults|family|1770889293|

**9. Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?**

``` sql
SELECT CALENDAR_YEAR, PLATFORM, ROUND(SUM(SALES)*1.0/SUM(TRANSACTIONS), 2) AS AVG_TRANSACTIONS_GROUP, ROUND(AVG(AVG_TRANSACTION), 2) AS AVG_OF_AVG_TRANSACTION
FROM data_mart.clean_weekly_sales
GROUP BY CALENDAR_YEAR, PLATFORM
ORDER BY CALENDAR_YEAR, PLATFORM; 
```
In math, the average of averages is not accurate and this question us not out of bounds to this fact as shown below. The collective sales/transactions for every platform in a year, is a better approach and the results are shown below.
|calendar_year|platform|avg_transactions_group|avg_of_avg_transaction|
|:----|:----|:----|:----|
|2018|Retail|36.56|42.91|
|2018|Shopify|192.48|188.28|
|2019|Retail|36.83|41.97|
|2019|Shopify|183.36|177.56|
|2020|Retail|36.56|40.64|
|2020|Shopify|179.03|174.87|

