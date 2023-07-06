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

I first begin by selecting all of the columns from the international_debt table and outputting the first 10 rows. 

``` sql
%%sql
postgresql:///international_debt
    
SELECT * FROM international_debt LIMIT 10; 
```
![]({{ site.url }}{{ site.baseurl }}/images/World Bank/snip1.png)


