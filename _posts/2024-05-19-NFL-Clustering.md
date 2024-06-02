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


We have the biometric & performance-based values for the players in our dataset, along with information on their draft outcome. Some players are missing information, and several fields that are not of interest to us...

First, we can clean up the number of player positons in the dataset, we can group similar positons together (i.e., a linebacker *LB* and an inside linebacker *ILB*). This will be useful later for our cluster analysis.

```python
# Create a dictonary to categorize player positions in the dataset
position_mapping = {
    "Quarterback": ["QB"],
    "Receiver": ["WR"],
    "Ball-Carrier": ["RB", "FB"],
    "Tight-End": ["TE"],
    "Secondary": ["S", "SS", "FS", "CB", "DB"],
    "Linebacker": ["ILB", "OLB", "Edge", "LB"],
    "D-Line": ["DE", "DT", "NT"],
    "O-Line": ["OT", "OG", "C", "G", "OL"],
    "Special": ["K", "P", "LS"]
}

# Create a function to map categorized positions back to players
def map_position(pos):
    for position, values in position_mapping.items():
        if pos in values:
            return position
    return "Unknown"

# Reformat Position
df["Position"] = df["Pos"].apply(map_position)
```

Now it's time to remove those useless fields.

```python
# Drop unnecessary fields
df.drop(columns=["Pos", "AV", "Pfr_ID"], inplace=True)
```

Upon investigating the data source, it is safe to assume players with NaN values for **Team**, **Round**, and **Pick** went undrafted. We will make these players easier to identify with some basic aliasing. As for the players with missing combine metrics, we will remove them altogether. 

```python
# Replace NaN values for undrafted players
df.loc[df["Team"].isna(), "Team"] = "Undrafted"
df.loc[df["Round"].isna(), "Round"] = 0
df.loc[df["Pick"].isna(), "Pick"] = 0

# Appropriate integer fields
df['Round'] = df['Round'].astype(int)
df['Pick'] = df['Pick'].astype(int)

# Drop players from dataset with missing combine metrics
df.dropna(inplace=True)

# Rename select fields
df = df.rename(columns={
    'Ht': 'Height',
    'Wt': 'Weight',
    'Vertical': 'Vert',
    'BenchReps': 'Bench',
    'BroadJump': 'Jump'
})

# View the new dataframe!
df.head(6)
```

| Player | Height | Weight | Forty | Vert | Bench | Jump | Cone | Shuttle | Year | Team | Round | Pick | Position |
| --- | --: | --: | --: | --: | --: | --: | --: | --: | --: | --: | --: | --: | --: |
| Darnell Alford | 76 | 334 | 5.56 | 25.0 | 23.0 | 94.0 | 8.48 | 4.98 | 2000 | Kansas City Chiefs | 6 | 188 | O-Line |
| Corey Atkins | 72 | 237 | 4.72 | 31.0 | 21.0 | 112.0 | 7.96 | 4.39 | 2000 | Undrafted | 0 | 0 | Linebacker |
| Reggie Austin | 69 | 175 | 4.44 | 35.0 | 17.0 | 119.0 | 7.03 | 4.14 | 2000 | Chicago Bears | 4 | 125 | Secondary |
| Mark Baniewicz | 78 | 312 | 5.34 | 28.0 | 20.0 | 96.0 | 7.72 | 4.73 | 2000 | Undrafted | 0 | 0 | O-Line |
| Rashidi Barnes | 72 | 208 | 4.62 | 35.0 | 10.0 | 114.0 | 6.92 | 4.32 | 2000 | Cleveland Browns | 7 | 225 | Secondary |
| David Barrett | 70 | 199 | 4.44 | 37.5 | 16.0 | 116.0 | 6.81 | 4.04 | 2000 | Arizona Cardinals | 4 | 102 | Secondary |



