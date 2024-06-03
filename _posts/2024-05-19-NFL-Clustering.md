---
title: "Upload in progress..."
date: 2000-5-19
tags: [Python, K-means Clustering, Regression Analysis]
excerpt: "Apply K-means clustering to identify correlated performance metrics across NFL Combine history"
---

## Background
While players in the National Football League are undoubtably athletic, their physical traits can vary significantly by position. Each year, the NFL Combine records physical and performance-based metrics for new athletes entering the NFL draft. These combine measurements allow scouts to further evaluate player prospects before drafting them. In this project, I explore the effectiveness of K-means clustering to identify player positions using NFL Combine data from 2000-2018. I also perform several tests to investigate an annual trend in the Cone Drill.  

## Exploratory Analysis
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

# View the new dataframe
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

Let's take one last look at a summary of the new dataframe.

```python
df.describe()
```

|  | Height | Weight | Forty | Vert | Bench | Jump | Cone | Shuttle | Year | Round | Pick |
| --- | --: | --: | --: | --: | --: | --: | --: | --: | --: | --: | --: |
| count | 2885.000000 | 2885.000000 | 2885.000000 | 2885.000000 | 2885.000000 | 2885.000000 | 2885.000000 | 2885.000000 | 2885.000000 | 2885.000000 | 2885.000000 |
| mean | 73.992721 | 252.087348 | 4.811584 | 32.598267 | 21.091854 | 113.190295 | 7.305719 | 4.408548 | 2009.374350 | 2.508492 | 74.869324 |
| std | 2.675779 | 45.882206 | 0.321231 | 4.270008 | 6.395231 | 9.491045 | 0.431420 | 0.269871 | 5.253415 | 2.442618 | 79.269856 |
| min | 65.000000 | 166.000000 | 4.260000 | 19.500000 | 2.000000 | 82.000000 | 6.280000 | 3.750000 | 2000.000000 | 0.000000 | 0.000000 |
| 25% | 72.000000 | 210.000000 | 4.550000 | 29.500000 | 17.000000 | 107.000000 | 6.970000 | 4.200000 | 2005.000000 | 0.000000 | 0.000000 |
| 50% | 74.000000 | 247.000000 | 4.730000 | 33.000000 | 21.000000 | 114.000000 | 7.220000 | 4.370000 | 2010.000000 | 2.000000 | 53.000000 |
| 75% | 76.000000 | 300.000000 | 5.060000 | 35.500000 | 25.000000 | 120.000000 | 7.590000 | 4.590000 | 2014.000000 | 5.000000 | 137.000000 |
| max | 81.000000 | 370.000000 | 6.000000 | 45.500000 | 45.000000 | 139.000000 | 9.040000 | 5.560000 | 2018.000000 | 7.000000 | 259.000000 |

We are left with 2885 player records to evaluate.

## Variable Correlation

Next, we'll create a correlation heatmap to investigate connections between numeric variables in the dataset.

```python
# Create a copy of the DataFrame
df_numeric = df.copy()

# Drop the original 'team' and 'position' columns
df_numeric.drop(['Team', 'Position'], axis=1, inplace=True)

# Calculate the correlation matrix
correlation_matrix = df_numeric.corr()

# Create the heatmap
plt.figure(figsize=(10, 8))
sns.heatmap(correlation_matrix, annot=True, cmap=sns.color_palette("vlag", as_cmap=True), fmt=".2f", square=True)
plt.title('Correlation Heatmap')
plt.show()
```

![]({{ site.url }}{{ site.baseurl }}/images/NFL/Correlation Heatmap.png)<!-- -->

Many correlations make intuitive sense. We'd expect a player's weight to be positively correlated with their 40-Yard Dash time, which we see with a correlation coefficient r of 0.89 (i.e., heavier players often take longer to complete this drill). It's also understandable that a player's standing jump would be negatively correlated with their Forty-Yard Dash time. Players who can complete the drill in less time (faster) are more likely to jump higher than heavier, slower players. We will circle back to these correlations later for deeper analysis.

## Cluster Analysis

We will now explore various player groups within the dataset using K-means clustering. To decide on the optimal number of clusters, we'll develop a function that visualizes inertia across different values of k. An *elbow point* on this plot will represent a significant decrease in inertia relative to increasing k values, indicating the optimal number of clusters where adding more clusters does not significantly reduce inertia.

```python
# Create a function to plot the inertias of k for k-means clustering
def optimize_k(data, max_k):
    means=[]
    inertias=[]

    for k in range(1, max_k):
        kmeans=KMeans(n_clusters=k)
        kmeans.fit(data)

        means.append(k)
        inertias.append(kmeans.inertia_)

    # Elbow plot to identify optimal k-value
    fig=plt.subplots(figsize=(10,5))
    plt.plot(means, inertias, "o-")
    plt.xlabel("Number of Clusters")
    plt.ylabel("Inertia")
    plt.grid(True)
    plt.show()

```

Now that we've initialized the function, we can test it by clustering all numerical fields in the dataset. This process will effectively create k player clusters characterized by similar performance metrics among their members.

```python
# Determine k for clustering on all numerical fields in the data
optimize_k(df[["Height", "Weight", "Forty", "Vert", "Bench", "Jump", "Cone", "Shuttle"]], 10)
```

![]({{ site.url }}{{ site.baseurl }}/images/NFL/All-Numeric Elbow Plot.png)<!-- -->

Based on the elbow plot above, we will select k=3 for our first K-means cluster on all numerical fields.

```python
# Let k=3 and cluster on all numerical fields
kmeans = KMeans(n_clusters=3)
kmeans.fit(df[["Height", "Weight", "Forty", "Vert", "Bench", "Jump", "Cone", "Shuttle"]])
df["kmeans_3_all"]=kmeans.labels_

# Split up clusters for plotting
cluster_1 = df[df["kmeans_3_all"]==1]
cluster_2 = df[df["kmeans_3_all"]==2]
cluster_3 = df[df["kmeans_3_all"]==0]

# Plot the distribution of player positions within clusters
max_count = max(cluster_1['Position'].value_counts().max(), cluster_2['Position'].value_counts().max(), cluster_3['Position'].value_counts().max())
y_axis_limit = max_count + 50
fig, axes = plt.subplots(1, 3, figsize=(15, 5), sharey=True)

cluster_1['Position'].value_counts().plot(kind='bar', color='firebrick', ax=axes[0], width=0.50)
axes[0].set_title('Cluster 1')
axes[0].set_xlabel('Position')
axes[0].set_ylabel('Count')
axes[0].set_ylim(0, y_axis_limit)
axes[0].tick_params(axis='x', rotation=45)

cluster_2['Position'].value_counts().plot(kind='bar', color='olivedrab', ax=axes[1], width=0.35)
axes[1].set_title('Cluster 2')
axes[1].set_xlabel('Position')
axes[1].set_ylabel('Count')
axes[1].set_ylim(0, y_axis_limit)
axes[1].tick_params(axis='x', rotation=45)

cluster_3['Position'].value_counts().plot(kind='bar', color='royalblue', ax=axes[2], width=0.30)
axes[2].set_title('Cluster 3')
axes[2].set_xlabel('Position')
axes[2].set_ylabel('Count')
axes[2].set_ylim(0, y_axis_limit)
axes[2].tick_params(axis='x', rotation=45)

plt.tight_layout()
plt.show()
```
![]({{ site.url }}{{ site.baseurl }}/images/NFL/All-Numeric Cluster Plot.png)<!-- -->

As we specified, the 2285 player-records have been clustered using all numeric fields available to them (Height, Weight, Forty, Bench...) into three groups. We then plotted the distribution of player positions in each cluster.

Cluster 1 contains mostly offensive and defensive linemen. This cluster makes up for the strong, hefty, and powerful players in our dataset.

Cluster 2 contains lots of skill players like recievers, defensive backs, and running backs. This cluster contains lighter, faster, and more explosive player positions.

Cluster 3 contains hybrid-style players that share similarities between the first two clusters. These players may be a bit weaker than Cluster 1 and slightly slower than Cluster 2, but are agile and powerful nonetheless. 
