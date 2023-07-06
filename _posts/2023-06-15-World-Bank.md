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

``` sql
```
country_name	country_code	indicator_name	indicator_code	debt
Afghanistan	AFG	Disbursements on external debt, long-term (DIS, current US$)	DT.DIS.DLXF.CD	72894453.700000003
Afghanistan	AFG	Interest payments on external debt, long-term (INT, current US$)	DT.INT.DLXF.CD	53239440.100000001
Afghanistan	AFG	PPG, bilateral (AMT, current US$)	DT.AMT.BLAT.CD	61739336.899999999
Afghanistan	AFG	PPG, bilateral (DIS, current US$)	DT.DIS.BLAT.CD	49114729.399999999
Afghanistan	AFG	PPG, bilateral (INT, current US$)	DT.INT.BLAT.CD	39903620.100000001
Afghanistan	AFG	PPG, multilateral (AMT, current US$)	DT.AMT.MLAT.CD	39107845
Afghanistan	AFG	PPG, multilateral (DIS, current US$)	DT.DIS.MLAT.CD	23779724.300000001
Afghanistan	AFG	PPG, multilateral (INT, current US$)	DT.INT.MLAT.CD	13335820
Afghanistan	AFG	PPG, official creditors (AMT, current US$)	DT.AMT.OFFT.CD	100847181.900000006
Afghanistan	AFG	PPG, official creditors (DIS, current US$)	DT.DIS.OFFT.CD	72894453.700000003
