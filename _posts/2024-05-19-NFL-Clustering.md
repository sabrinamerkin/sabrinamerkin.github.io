---
title: "Upload in progress..."
date: 2000-5-19
tags: [Python, K-means Clustering, Regression Analysis]
excerpt: "Apply K-means clustering to identify correlated performance metrics across NFL Combine history"
---

## Background
While players in the National Football League are undoubtably athletic, their physical traits can vary significantly by position. Each year, the NFL Combine records physical and performance-based metrics for new athletes entering the NFL draft. These combine measurements allow scouts to further evaluate player prospects before drafting them. In this project, I explore the effectiveness of K-means clustering to identify player positions using NFL Combine data from 2000-2018. I also perform several tests to investigate an annual trend in the Cone Drill.  

## Data Analysis
Using a Python 3 kernel in Jupyter, we will load in the following libraries and dataset.

```python
# Import libraries
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans

# Load NFL Combine data from 2000-2018
df = pd.read_csv("C:/Users/ethan/OneDrive/Data Projects/Datasets/NFL_Combine.csv", index_col="Player")
```

First, we'll take a look at the raw dataframe.
 
```python
df.shape
```
(6218, 15)

We have 6218 player records and 15 data fields.

```python
# Print the first 6 rows
df.head(6)
```

| Player | Pos |	Ht |	Wt	| Forty |	Vertical	| BenchReps	| BroadJump	| Cone	|	Shuttle	| Year	|	Pfr_ID	|	AV	|	Team	|	Round	|	Pick |
| --- | --: | --: | --: | --: | --: | --: | --: | --: | --: | --: | --: | --: | --: | --: | --: |
| John Abraham |	OLB |	76	| 252	| 4.55	| NaN	| NaN	| NaN	| NaN |	NaN	| 2000	| AbraJo00	| 26 |	New York Jets	| 1.0	| 13.0 |
| Shaun Alexander |	RB |	72 |	218	| 4.58	| NaN |	NaN |	NaN |	NaN |	NaN |	2000 |	AlexSh00 |	26	| Seattle Seahawks	| 1.0 |	19.0 |
| Darnell Alford |	OT |	76 |	334 |	5.56 |	25.0 |	23.0 |	94.0 |	8.48 |	4.98 | 2000 |	AlfoDa20 |	0	| Kansas City Chiefs |	6.0	| 188.0 |
| Kyle Allamon |	TE |	74 |	253 |	4.97 |	29.0 |	NaN |	104.0 |	7.29 |	4.49 |	2000 |	NaN |	0 |	NaN |	NaN |	NaN |
| Rashard Anderson |	CB |	74 |	206 |	4.55 |	34.0 |	NaN	| 123.0 |	7.18 |	4.15 |	2000 |	AndeRa21 |	6	| Carolina Panthers |	1.0 |	23.0 |
| Jake Arians	| K |	70 |	202 |	NaN |	NaN |	NaN |	NaN |	NaN |	NaN |	2000 |	arianjak01 |	0 |	NaN |	NaN |	NaN |

We have the biometric & performance-based values for the players in our dataset, along with information on their draft outcome. Some players are missing information, and several fields that are not of interest to us.

```python
df['Pos'].value_counts()
```

Pos
WR      857
CB      630
RB      540
DE      487
DT      463
OT      460
OLB     424
OG      365
QB      350
TE      337
ILB     276
FS      229
SS      213
C       171
P       120
FB      117
K        85
S        27
EDGE     23
LS       20
G        14
NT        3
LB        3
DB        2
OL        2
Name: count, dtype: int64
