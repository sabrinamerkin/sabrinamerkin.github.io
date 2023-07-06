---
title: "PostgreSQL International Debt Analysis"
date: 2023-06-15
tags: [SQL, Data Analysis, Query Writing]
excerpt: "Analyze international debt data collected by The World Bank using PostgreSQL querying"
mathjax: true
---

## Background

In this project, I analyzed international debt data collected by The World Bank using PostgreSQL querying. I first completed this assignment on DataCamp upon finishing the prerequisite 'Intermediate SQL` course. After completing the 'Data Manipulation in SQL' course on DataCamp, I returned back to this project to incorporate new skills. This analysis was performed using Jupyter Notebook. Access to the international_debt database was provided by DataCamp.

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
![]({{ site.url }}{{ site.baseurl }}/images/World Bank/snip1.png)

This reveals the amount of debt owed by Afghanistan in 10 different debt indicators. However, the amount of distinct countries in the table remains unknown.

``` sql
%%sql
SELECT COUNT(DISTINCT country_name) AS total_distinct_countries
FROM international_debt;
```

![]({{ site.url }}{{ site.baseurl }}/images/World Bank/snip2.png)

We can see the table holds a total of 124 distinct countries. As we saw earlier, there is a column called indicator_name that describes the purpose of taking the debt. Just beside that column, there is another column called indicator_code which symbolizes the category of these debts. Knowing about these various debt indicators will help us to understand the areas in which a country can possibly be indebted to.

``` sql
%%sql
SELECT DISTINCT indicator_code AS distinct_debt_indicators
FROM international_debt
ORDER BY distinct_debt_indicators
```

![]({{ site.url }}{{ site.baseurl }}/images/World Bank/snip3.png)

### Debt Analysis

To gain a sense of the overall global economy, we can find the total amount of debt owed in US dollars amongst all countries in the table. To make this result fathomable, we divide the grand total by 1 million and round to the nearest 2 decimal places.

``` sql
%%sql
SELECT 
    ROUND(SUM(debt)/1000000, 2) AS total_debt
FROM international_debt; 
```

![]({{ site.url }}{{ site.baseurl }}/images/World Bank/snip4.png)

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
| --- | --- |
| China | 285793494734.20 |

We see that china owns the highest amount of debt amongst all countries.


