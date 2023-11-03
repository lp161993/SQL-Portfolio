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
In math, the average of averages is not accurate and this question is not out of bounds to this fact as shown below. The collective sales/transactions for every platform in a year, is a better approach and the results are shown below in 'avg_transactions_group'.
|calendar_year|platform|avg_transactions_group|avg_of_avg_transaction|
|:----|:----|:----|:----|
|2018|Retail|36.56|42.91|
|2018|Shopify|192.48|188.28|
|2019|Retail|36.83|41.97|
|2019|Shopify|183.36|177.56|
|2020|Retail|36.56|40.64|
|2020|Shopify|179.03|174.87|

### Before & After Analysis

Now to the actual question at hand. 2020-06-15 is the date when the supply chain changes were made. Hence, we analyse the sales before and after this specific date.

Approach:
- Pre and post sales are analysed on week basis. Hence, it is convinient to identify the week_number associated with cutoff date and base our further calculation on that number.
- The analysis required Pre and Post 4 weeks, 12 weeks sales performance in 2020 and the same for 2018, 2019. Hence, after multiple iterations of code, I have been able to calculate all the required numbers in a single table. 

```sql
/*Identify the week_number associated with the cutoff date*/
CREATE TABLE GAMMA AS
( SELECT DISTINCT WEEK_NUMBER AS CUTOFF
 FROM DATA_MART.CLEAN_WEEKLY_SALES
 WHERE WEEK_DATE ='2020-06-15');

/*Finding the rows which are 12 weeks before and after the cutoff date, this includes the 4 weeks sales as well(!Duh obviously)*/
CREATE TABLE HECTAR AS 
 (
   SELECT CALENDAR_YEAR, WEEK_NUMBER, SUM(SALES) AS SALES
   FROM DATA_MART.CLEAN_WEEKLY_SALES C, GAMMA G
   WHERE WEEK_NUMBER BETWEEN G.CUTOFF-12 AND G.CUTOFF+11
   GROUP BY CALENDAR_YEAR, WEEK_NUMBER
 );

/*Calculating the collective sales for Pre and Post 4 and 12 weeks for a given calendar year */
 WITH DIVINE AS (SELECT H.CALENDAR_YEAR,
    SUM(CASE WHEN WEEK_NUMBER BETWEEN G.CUTOFF-4 AND G.CUTOFF-1 THEN H.SALES END) AS PRE_FOUR_WEEK_SALES,
    SUM(CASE WHEN WEEK_NUMBER BETWEEN G.CUTOFF AND G.CUTOFF+3 THEN H.SALES END) AS POST_FOUR_WEEK_SALES,
    SUM(CASE WHEN WEEK_NUMBER BETWEEN G.CUTOFF-12 AND G.CUTOFF-1 THEN H.SALES END) AS PRE_12_WEEK_SALES,
    SUM(CASE WHEN WEEK_NUMBER BETWEEN G.CUTOFF AND G.CUTOFF+12 THEN H.SALES END) AS POST_12_WEEK_SALES
  FROM HECTAR H, GAMMA G
  GROUP BY H.CALENDAR_YEAR
  ORDER BY H.CALENDAR_YEAR)

/*Calculating the sales performance every year */
 SELECT D.CALENDAR_YEAR, 
 	(D.POST_FOUR_WEEK_SALES- D.PRE_FOUR_WEEK_SALES) AS DIFFERENCE_IN_SALES_4W, 
 	ROUND(((D.POST_FOUR_WEEK_SALES- D.PRE_FOUR_WEEK_SALES)*100.0)/D.PRE_FOUR_WEEK_SALES, 2) AS DIFFERENCE_PERCENT_4W,
    (D.POST_12_WEEK_SALES - D.PRE_12_WEEK_SALES) AS DIFFERENCE_IN_SALES_12W,
    ROUND(((D.POST_12_WEEK_SALES - D.PRE_12_WEEK_SALES)*100.0)/D.PRE_12_WEEK_SALES, 2) AS DIFFERENCE_PERCENT_12W
 FROM DIVINE D
 ORDER BY CALENDAR_YEAR;
```
Shown below is the sales performance on yearly basis. The pre and post performance of 4 week bandwidth and 12 week bandwidth are calculated based on the cutoff date '2020-06-15'

|calendar_year|difference_in_sales_4w|difference_percent_4w|difference_in_sales_12w|difference_percent_12w|
|:----|:----|:----|:----|:----|
|2018|4102105|0.19|104256193|1.63|
|2019|2336594|0.10|-20740294|-0.30|
|2020|-26884188|-1.15|-152325394|-2.14|

Consider row 3, where the supply chain change had been made. There has been a 1.15 percent difference in sales for 4 weeks after the change has been made, whilst the same period saw a positive growth in sales for 2018 and 2019.

The 12 week period post the supply chain change has seen a decrease of 2.14 percent. This number is very less in comparison with the numbers of previous years. 
Based on these facts, we can conclude maybe the supply chain change has negatively impacted the sales performance. If there was further historical data, then we can be more assured that this change has for sure negatively impacted the sales.

### Bonus Question

Which area(based on Region, Platform, Age_band, Demographic & Customer_Type) has been highly impacted owing to this change?

```sql
CREATE TABLE BONUS AS(
	SELECT C.*
	FROM DATA_MART.CLEAN_WEEKLY_SALES C, GAMMA G
	WHERE C.WEEK_NUMBER BETWEEN G.CUTOFF-12 AND G.CUTOFF+11 AND C.CALENDAR_YEAR='2020'
);

/*METRICS WITH HIGHEST NEGATIVE IMPACT */
WITH AREA_SALES AS (
  SELECT B.REGION, B.PLATFORM, B.AGE_BAND, B.DEMOGRAPHIC, B.CUSTOMER_TYPE, SUM(CASE WHEN B.WEEK_NUMBER BETWEEN G.CUTOFF-12 AND G.CUTOFF-1 THEN B.SALES END) AS PRE_12_WEEK_SALES,
  SUM(CASE WHEN WEEK_NUMBER BETWEEN G.CUTOFF AND G.CUTOFF+12 THEN B.SALES END) AS POST_12_WEEK_SALES
  FROM BONUS B, GAMMA G
  GROUP BY B.REGION, B.PLATFORM, B.AGE_BAND, B.DEMOGRAPHIC, B.CUSTOMER_TYPE)
  
SELECT REGION, PLATFORM, AGE_BAND, DEMOGRAPHIC, CUSTOMER_TYPE,
(POST_12_WEEK_SALES-PRE_12_WEEK_SALES) AS DIFFERENCE, 
ROUND((POST_12_WEEK_SALES-PRE_12_WEEK_SALES)*100.0/ PRE_12_WEEK_SALES, 2) AS PERCENTAGE_CHANGE_IN_SALES
FROM AREA_SALES
ORDER BY PERCENTAGE_CHANGE_IN_SALES ASC;
```
Shown below are only top 4 rows from the result. As we can see the first row with parameter 'Region- South America, Platform- Shopify, Customer Type- Existing, Unknown Age and Demographic details' has shown the maximum decline in sales of 42.23%.

|region|platform|age_band|demographic|customer_type|difference|percentage_change_in_sales|
|:----|:----|:----|:----|:----|:----|:----|
|SOUTH AMERICA|Shopify|UNKNOWN|UNKNOWN|Existing|-4977|-42.23|
|EUROPE|Shopify|Retirees|family|New|-2458|-33.71|
|EUROPE|Shopify|Young Adults|family|New|-4437|-27.97|
|SOUTH AMERICA|Retail|UNKNOWN|UNKNOWN|Existing|-29650|-23.20|

### Insights for Danny Team:

1. Collect more historical data for years before 2018 and compare sales performance around the same time period. This will provide surity if the supply chain change was the only reason for decline in sales.
2. If the sales and transaction data was available on a category/ sub-category level, then we can pinpoint if the decline has been over all category or in a specific category. If the decline was in a specific category, then the supply chain change was not the only reason why the sales decline was observed.

