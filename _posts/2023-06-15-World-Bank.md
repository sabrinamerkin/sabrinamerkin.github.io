---
title: "PostgreSQL: International Debt Analysis"
date: 2023-06-15
tags: [SQL, Data Analysis, Query Writing]
excerpt: "Analyze international debt data collected by The World Bank using PostgreSQL querying"
mathjax: true
---

## Background

In this project, I analyzed international debt data collected by The World Bank using PostgreSQL querying. I completed this assignment on DataCamp upon finishing the prerequisite 'Intermediate SQL` course. After completing the 'Data Manipulation in SQL' course on DataCamp, I returned back to this project to incorporate new skills. This analysis was performed using Jupyter Notebook. Access to the international_debt database was provided by DataCamp.

**SQL Concepts Implemented**

  - The Big 5: `SELECT`–\>`FROM`–\>`WHERE`–\>`GROUP
    BY`–\>`ORDER BY`
  - Aggregation (`COUNT`, `SUM`, `AVG`, ect.)
  - Table Joins
  - Common Table Expressions and Subqueries
  - Window Functions
  - Case Statements

## Project

Key questions to answer:
- What is the total amount of debt that is owed by the countries listed in the dataset?
- Which country owns the maximum amount of debt and what does that amount look like?
- What is the average amount of debt owed by countries across different debt indicators?

### Data Exploration

We first begin by selecting all of the columns from the international_debt table and outputting the first 10 rows. 

``` sql
%%sql
postgresql:///international_debt
    
SELECT * FROM international_debt LIMIT 10; 
```

| country_name |	country_code | indicator_name | indicator_code | debt |
| --- | --: | --: | --: | --: |
| Afghanistan	| AFG	| Disbursements on external debt, long-term (DIS, current US$)	| DT.DIS.DLXF.CD |	72894453.700000003
| Afghanistan	| AFG	| Interest payments on external debt, long-term (INT, current US$)	| DT.INT.DLXF.CD |	53239440.100000001
| Afghanistan	| AFG	| PPG, bilateral (AMT, current US$)	| DT.AMT.BLAT.CD	| 61739336.899999999
| Afghanistan	| AFG	| PPG, bilateral (DIS, current US$)	| DT.DIS.BLAT.CD	| 49114729.399999999
| Afghanistan	| AFG	| PPG, bilateral (INT, current US$)	| DT.INT.BLAT.CD	| 39903620.100000001
| Afghanistan	| AFG	| PPG, multilateral (AMT, current US$)	| DT.AMT.MLAT.CD	| 39107845
| Afghanistan	| AFG	| PPG, multilateral (DIS, current US$)	| DT.DIS.MLAT.CD	| 23779724.300000001
| Afghanistan	| AFG	| PPG, multilateral (INT, current US$)	| DT.INT.MLAT.CD	| 13335820
| Afghanistan	| AFG	| PPG, official creditors (AMT, current US$)	| DT.AMT.OFFT.CD	| 100847181.900000006
| Afghanistan	| AFG	| PPG, official creditors (DIS, current US$)	| DT.DIS.OFFT.CD	| 72894453.700000003


This reveals the amount of debt owed by Afghanistan in 10 different debt indicators. However, the amount of distinct countries in the table remains unknown.

``` sql
%%sql
SELECT COUNT(DISTINCT country_name) AS total_distinct_countries
FROM international_debt;
```

| total_distinct_countries |
| --- |
| 124 |

We can see the table holds a total of 124 distinct countries. As we saw earlier, there is a column called indicator_name that describes the purpose of taking the debt. Just beside that column, there is another column called indicator_code which symbolizes the category of these debts. Knowing about these various debt indicators will help us to understand the areas in which a country can possibly be indebted to.

``` sql
%%sql
SELECT DISTINCT indicator_code AS distinct_debt_indicators
FROM international_debt
ORDER BY distinct_debt_indicators
LIMIT 10;
```

| distinct_debt_indicators |
| --- |
| DT.AMT.BLAT.CD |
| DT.AMT.DLXF.CD |
| DT.AMT.DPNG.CD |
| DT.AMT.MLAT.CD |
| DT.AMT.OFFT.CD |
| DT.AMT.PBND.CD |
| DT.AMT.PCBK.CD |
| DT.AMT.PROP.CD |
| DT.AMT.PRVT.CD |
| DT.DIS.BLAT.CD |

### Debt Analysis

To gain a sense of the overall global economy, we can find the total amount of debt owed in US dollars amongst all countries in the table. To make this result fathomable, we divide the grand total by 1 million and round to the nearest 2 decimal places.

``` sql
%%sql
SELECT 
    ROUND(SUM(debt)/1000000, 2) AS total_debt
FROM international_debt; 
```

| total_debt |
| --- |
| 3079734.49 |

This value exceeds 3 million **million** US dollars, which is slightly easier (but still quite difficult) for one to comprehend. Next, we can find which country owns the highest amount of debt along with the amount. **Note** that this debt is the sum of all debts owed by a country.

``` sql
%%sql
SELECT 
    country_name, 
    ROUND(SUM(debt),2) AS total_debt
FROM international_debt
GROUP BY country_name
ORDER BY total_debt DESC
LIMIT 1;
```

| country_name | total_debt |
| :-- | :-- |
| China | 285793494734.20 |

We now see that china owns the highest amount of debt amongst all countries.

Window functions can be used to pass running aggregate values along rows. We can use a window functuion to create a running_debt field that tracks the contribution of each debt owed by China. 

``` sql
%%sql
WITH china_table AS (SELECT * FROM international_debt WHERE country_name = 'China')

SELECT indicator_name, SUM(debt) OVER(ORDER BY indicator_name ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_debt
FROM china_table;
```

| indicator_name |	running_debt |
| --: | --: |
| Disbursements on external debt, long-term (DIS, current US$) |	15692563746.100000381 |
| Interest payments on external debt, long-term (INT, current US$) |	33559112397.500001907 |
| Interest payments on external debt, private nonguaranteed (PNG) (INT, current US$) |	47701831149.100002288 |
| PPG, bilateral (AMT, current US$) |	54234277591.000001907 |
| PPG, bilateral (INT, current US$) |	54749175998.100001931 |
| PPG, bonds (AMT, current US$)	| 64583852998.100001931 |
| PPG, bonds (INT, current US$)	| 65808101998.100001931 |
| PPG, commercial banks (AMT, current US$) |	69854345296.600001931 |
| PPG, commercial banks (DIS, current US$)	| 73631395569.900002122 |
| PPG, commercial banks (INT, current US$) |	74601328659.900002122 |
| PPG, multilateral (AMT, current US$) |	77217052374.000002027 |
| PPG, multilateral (DIS, current US$) |	80296553646.100001932 |
| PPG, multilateral (INT, current US$) |	81154960620.900001884 |
| PPG, official creditors (AMT, current US$) |	90303130776.900001884 |
| PPG, official creditors (DIS, current US$) |	93382632049.000001789 |
| PPG, official creditors (INT, current US$) |	94755937430.900001884 |
| PPG, other private creditors (AMT, current US$) |	95552481598.300001860 |
| PPG, other private creditors (DIS, current US$) |	95886493799.000001848 |
| PPG, other private creditors (INT, current US$) |	96042836226.900001854 |
| PPG, private creditors (AMT, current US$) |	110720300692.800001473 |
| PPG, private creditors (DIS, current US$) |	114831363166.800001473 |
| PPG, private creditors (INT, current US$) |	117181887684.700001568 |
| Principal repayments on external debt, long-term (AMT, current US$) |	213400508520.399998516 |
| Principal repayments on external debt, private nonguaranteed (PNG) (AMT, current US$) |	285793494734.200001568 |


This running_debt field provides an interesting insight on how each debt indicator contributes to the growth of China's total debt.

Next, we use case-statements and a common table expression to label the debt severity of each country. Countries with total debt less than $95 billion are labeled as having `low` debt severity. Countries with total debt above $95 billion and less than $190 billion are labeled as having `medium` debt severity. Countries with total debt exceeding $95 billion are have `high` debt severity. The results are sorted by debt severity in decending order and limited to 10 rows.


``` sql
%%sql
WITH debt_table AS
    (SELECT country_name, SUM(debt) AS total_debt FROM international_debt GROUP BY country_name ORDER BY total_debt DESC)

SELECT country_name, total_debt, (CASE WHEN total_debt < 9.5E+10 THEN 'low'
                                 WHEN total_debt >= 9.5E+10 AND total_debt < 1.9E+11 THEN 'medium'
                                 ELSE 'high' END) AS debt_severity
FROM debt_table
GROUP BY country_name, total_debt
ORDER BY total_debt DESC
LIMIT 10;
```

| country_name |	total_debt |	debt_severity |
| --: | --: | --: |
| China	| 285793494734.200001568	| high |
| Brazil |	280623966140.800007581	| high |
| South Asia |	247608723990.600003211	| high |
| Least developed countries: UN classification |	212880992791.900000988 |	high |
| Russian Federation |	191289057259.200001943	| high |
| IDA only |	179048127207.299999298	| medium |
| Turkey |	151125758035.300003616	| medium |
| India	| 133627060958.399997148	| medium |
| Mexico	| 124596786217.300001668	| medium |
| Indonesia	| 113435696693.499999149	| medium |


To experiment a bit with table joins, we demonstrate two ways to query a list of all countries possessing any individual debts that fall between $100 thousand - $10 million. The first (and more complicated) method involves creating two common table expressions that meet the constraints of this problem. We then intersect these two common table expressions.

``` sql
%%sql

with greater_debt as (SELECT country_name, debt FROM international_debt WHERE debt > 1.0E+5),
smaller_debt as (SELECT country_name, debt FROM international_debt WHERE debt < 1.0E+7)

SELECT * FROM greater_debt

INTERSECT

SELECT * FROM smaller_debt

ORDER BY debt
LIMIT 10;
```

| country_name	| debt |
| --: | --: |
| Nicaragua	| 105260.3 |
| Central African Republic |	120000 |
| Madagascar |	120000 |
| Albania	| 120324.7 |
| Djibouti |	127000 |
| Bangladesh |	131000 |
| Dominica |	137037.1 |
| Comoros	| 154358.4 |
| Lesotho	| 157326.4 |
| Albania	| 170018.4 |



Next, we see a much simpler approach to this problem.


``` sql
%%sql

Select distinct country_name, debt
From international_debt
WHERE debt > 1.0E+5 AND debt < 1.0E+7
ORDER BY debt
Limit 10;
```

| country_name	| debt |
| --: | --: |
| Nicaragua	| 105260.3 |
| Central African Republic |	120000 |
| Madagascar |	120000 |
| Albania	| 120324.7 |
| Djibouti |	127000 |
| Bangladesh |	131000 |
| Dominica |	137037.1 |
| Comoros	| 154358.4 |
| Lesotho	| 157326.4 |
| Albania	| 170018.4 |

Although the first method may seem redundant, it is neat to observe that there are are more ways than one to obtain this query using SQL.

Now that we have a nice overview of the international_debt table, it is time to discover the average amount of debt owed for each debt indicator. This will provide a better sense of the distribution amongst all debt indicators.

``` sql
%%sql
SELECT 
    indicator_code AS debt_indicator, indicator_name,  AVG(debt) as average_debt
FROM international_debt
GROUP BY debt_indicator, indicator_name
ORDER BY average_debt DESC
LIMIT 10;
```

| debt_indicator	| indicator_name	| average_debt |
| --: | --: | --: |
| DT.AMT.DLXF.CD	| Principal repayments on external debt, long-term (AMT, current US$) |	5904868401.499193612 |
| DT.AMT.DPNG.CD	| Principal repayments on external debt, private nonguaranteed (PNG) (AMT, current US$) |	5161194333.812658349 |
| DT.DIS.DLXF.CD	| Disbursements on external debt, long-term (DIS, current US$)	| 2152041216.890243888 |
| DT.DIS.OFFT.CD	| PPG, official creditors (DIS, current US$)	| 1958983452.859836046 |
| DT.AMT.PRVT.CD	| PPG, private creditors (AMT, current US$)	| 1803694101.963265321 |
| DT.INT.DLXF.CD	| Interest payments on external debt, long-term (INT, current US$)	| 1644024067.650806481 |
| DT.DIS.BLAT.CD	| PPG, bilateral (DIS, current US$)	| 1223139290.398230108 |
| DT.INT.DPNG.CD	| Interest payments on external debt, private nonguaranteed (PNG) (INT, current US$) |	1220410844.421518983 |
| DT.AMT.OFFT.CD	| PPG, official creditors (AMT, current US$)	| 1191187963.083064523 |
| DT.AMT.PBND.CD	| PPG, bonds (AMT, current US$) |	1082623947.653623188 |

The indicator `DT.AMT.DLXF.CD` is the greatest debt on average. This category is representative of debts requiring long-term repayments. This makes sense since long-term debt often acquires the most capital over time.

We can observe from the above finding that there is a significant difference in average debts following the second highest indicator. This implies that the first two indicators might be the most severe debt categories amongst all countries. 

We can investigate this further to discover which country owes the highest debt in the long-term debt (DT.AMT.DLXF.CD) category.

``` sql
%%sql
SELECT 
    country_name, 
    indicator_name
FROM international_debt
WHERE debt = (SELECT 
                 max(debt)
             FROM international_debt
             where indicator_code = 'DT.AMT.DLXF.CD');
```

| country_name |	indicator_name |
| :-- | --: |
| China	| Principal repayments on external debt, long-term (AMT, current US$) |

China has the largest amount of debt owed in the long-term debt (DT.AMT.DLXF.CD) category. Again, we observed that long-term debt is the greatest debt on average, but we still do not know if **every** country owes a long-term debt. To see if China is inflating this value, we can check whether all remaining countries are also contributing to this average. We will proceed by identifying the top 10 most common debt indicators amongst all countries. If long-term debt (DT.AMT.DLXF.CD) is owed by every country, we can confirm that long-term debt is the most severe debt category amongst all countries. 

``` sql
%%sql
SELECT indicator_code, count(*) AS indicator_count
FROM international_debt
GROUP BY indicator_code
ORDER BY indicator_count DESC, indicator_code DESC
LIMIT 10;
```

| indicator_code	| indicator_count |
| :-- | --:|
| DT.INT.OFFT.CD |	124 |
| DT.INT.MLAT.CD | 124 |
| DT.INT.DLXF.CD | 124 |
| DT.AMT.OFFT.CD |	124 |
| DT.AMT.MLAT.CD |	124 |
| DT.AMT.DLXF.CD |	124 |
| DT.DIS.DLXF.CD |	123 |
| DT.INT.BLAT.CD |	122 |
| DT.DIS.OFFT.CD |	122 |
| DT.AMT.BLAT.CD |	122 |

Provided that there are 124 countries in the table, we clearly see that long-term debt (DT.AMT.DLXF.CD) is indeed the most severe debt category amongst all countries. 



