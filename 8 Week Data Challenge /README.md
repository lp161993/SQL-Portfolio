# Project 1 - Danny Data Mart Challenge
Let us help Danny analyse how well Data Mart is contributing to his pockets after a major supply chain change!

##### Introduction: 
https://8weeksqlchallenge.com/case-study-5/ is the link which provides the data challenge related details.
In short: Data Mart is a supermarket in Australia and Danny, the owner, has made major supply chain changes in 2020. Let's help Danny analyse the effects of this change in the sales performance from the sales data given. 

##### Data Cleanup:

Tasks at hand: 
In a single query, perform the following operations and generate a new table in the data_mart schema named clean_weekly_sales:

- Convert the week_date to a DATE format

- Add a week_number as the second column for each week_date value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc

- Add a month_number with the calendar month for each week_date value as the 3rd column

- Add a calendar_year column as the 4th column containing either 2018, 2019 or 2020 values

- Add a new column called age_band after the original segment column using the following mapping on the number inside the segment value

- Segment	age_band
  - 1	Young Adults
  - 2	Middle Aged
  - 3 or 4	Retirees
- Add a new demographic column using the following mapping for the first letter in the segment values:
segment	demographic
  - C	Couples
  - F	Families
- Ensure all null string values with an "unknown" string value in the original segment column as well as the new age_band and demographic columns

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

- Generate a new avg_transaction column as the sales value divided by transactions rounded to 2 decimal places for each record
