---
title: "Upload in progress..."
date: "2000-01-01"
tags: [Python, Machine Learning, Matplotlib, Data Visualization]
excerpt: "Utilize machine learning models to estimate employee salary from performance metrics"
mathjax: true
---

## Background
The dataset used in this analysis was imported from Kaggle. It was created to explore typical elements that influence employee performance and satisfaction in the workplace. This dataset contains information for 200 unique employees working at an organization.

## Analysis
We begin the exploration of this dataset by importing the required libraries and CSV file for analysis.

```python
# Import required libraries
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import missingno as msno
import plotly.express as px

# Load dataset
df = pd.read_csv("C:/Users/ethan/OneDrive/Documents/Data Glacier/datasets/hr_data.csv")
df.shape
```

```python
(200, 11)
```
The dataset contains 200 employee records with 11 informative variables.

```python
df.head(6)
```

|  | Name	| Age |	Gender | Projects Completed |	Productivity (%) |	Satisfaction Rate (%) |	Feedback Score |	Department |	Position |	Joining Date |	Salary |
| --- | --: | --: | --: | --: | --: | --: | --: | --: | --: | --: | --: |
| 0	| Douglas Lindsey |	25	| Male |	11	| 57 |	25	| 4.7 |	Marketing	| Analyst |	Jan-20 |	63596 |
| 1 |	Anthony Roberson	| 59 |	Female | 19 |	55 |	76 |	2.8 |	IT |	Manager |	Jan-99 |	112540 |
| 2	| Thomas Miller	| 30	| Male	| 8 |	87 |	10	| 2.4	| IT |	Analyst	| Jan-17 |	66292 |
| 3	| Joshua Lewis	| 26	| Female	| 1	| 53	| 4	| 1.4	| Marketing	| Intern |	Jan-22 |	38303 |
| 4	| Stephanie Bailey |	43 |	Male	| 14 |	3 |	9 |	4.5 |	IT	| Team Lead |	Jan-05	| 101133 |
| 5	| Jonathan King	| 24 |	Male	| 5	| 63 |	33 |	4.2 |	Sales |	Junior Developer |	Jan-21 |	48740 |

Next, we check to see if there are any missing values in the dataset.

```python
df.isnull().sum()
```

```python
Name                     0
Age                      0
Gender                   0
Projects Completed       0
Productivity (%)         0
Satisfaction Rate (%)    0
Feedback Score           0
Department               0
Position                 0
Joining Date             0
Salary                   0
dtype: int64
```

There are no missing values. Next, we can see the type of each variable in the dataset.

```python
df.info()
```

```python
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 200 entries, 0 to 199
Data columns (total 11 columns):
 #   Column                 Non-Null Count  Dtype  
---  ------                 --------------  -----  
 0   Name                   200 non-null    object 
 1   Age                    200 non-null    int64  
 2   Gender                 200 non-null    object 
 3   Projects Completed     200 non-null    int64  
 4   Productivity (%)       200 non-null    int64  
 5   Satisfaction Rate (%)  200 non-null    int64  
 6   Feedback Score         200 non-null    float64
 7   Department             200 non-null    object 
 8   Position               200 non-null    object 
 9   Joining Date           200 non-null    object 
 10  Salary                 200 non-null    int64  
dtypes: float64(1), int64(5), object(5)
memory usage: 17.3+ KB
```
We can see summary statistics of the data with the following commands.

```python
df.describe()
```

|  | Age	| Projects Completed	| Productivity (%)	| Satisfaction Rate (%)	| Feedback Score	| Salary |
| --- | --: | --: | --: | --: | --: | --: |
| count	| 200.000000	| 200.000000	| 200.000000	| 200.000000	| 200.000000	| 200.000000 |
| mean	| 34.650000	| 11.455000	| 46.755000	| 49.935000	| 2.883000	| 76619.245000 |
| std	| 9.797318	| 6.408849	| 28.530068	| 28.934353	| 1.123263	| 27082.299202 |
| min	| 22.000000	| 0.000000	| 0.000000 |	0.000000	| 1.000000	| 30231.000000 | 
| 25%	| 26.000000	| 6.000000	| 23.000000	| 25.750000	| 1.900000	| 53080.500000 |
| 50%	| 32.000000	| 11.000000	| 45.000000	| 50.500000	| 2.800000	| 80540.000000 |
| 75%	| 41.000000	| 17.000000	| 70.000000	| 75.250000	| 3.900000	| 101108.250000 |
| max	| 60.000000	| 25.000000	| 98.000000	| 100.000000	| 4.900000	| 119895.000000 |

```python
df.describe(include='object')
```

|  | Name	| Gender |	Department	| Position |	Joining Date |
| --- | --: | --: | --: | --: | --: |
| count	| 200	| 200	| 200	| 200	| 200 |
| unique	| 200	| 2 |	5	| 6	| 25 |
| top	| Douglas Lindsey	| Male |	Sales |	Manager	| Jan-18 |
| freq	| 1	| 100 |	47 |	40 |	23 |

*Name* is not an important variable to this analysis, so we can remove it from the data frame.

```python
df=df.drop(['Name'],axis=1)
df.shape
```

```python
(200, 10)
```
We now have 200 employee records with 10 informative variables.




