---
title: "PostgreSQL International Debt Analysis"
date: 2023-06-15
tags: [SQL, Data Analysis, Query Writing]
excerpt: "Analyze international debt data collected by The World Bank using PostgreSQL querying"
mathjax: true
---

## Background

In this project, I analyzed international debt data collected by The World Bank using PostgreSQL querying, gaining valuable insights into the economic landscape of developing countries.

I began exploring this data set by determining which countries and distinct debt indicators were included in a particular table retrieved from the World Bank database. Using standard grouping and ordering techniques, I constructed a query with a 'total debt' field that calculated a sum of debts (combining values each debt indicator) for each country and sorted them.


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
