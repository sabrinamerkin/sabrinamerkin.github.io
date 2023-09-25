---
title: "Upload in progress..."
date: "2000-01-01"
tags: [Python, Machine Learning, Matplotlib, Data Visualization]
excerpt: "Utilize machine learning models to estimate salary from employee metrics"
mathjax: true
---

## Background
The dataset in this analysis was created to explore factors that influence employee performance and satisfaction in a typical organization. These variables include personal traits, perfromance metrics, and job details of employees. In this project, I explore variable relativity and deploy machine learning models to predict employee salary from training data. The Employee Productivity and Satisfaction HR Data was imported from Kaggle. 

## Exploratory Analysis
We begin this analysis by importing the required libraries and dataset.

```python
# Import required libraries
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import missingno as msno
import plotly.express as px
from sklearn import preprocessing

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

| Name	| Age |	Gender | Projects Completed |	Productivity (%) |	Satisfaction Rate (%) |	Feedback Score |	Department |	Position |	Joining Date |	Salary |
| --- | --: | --: | --: | --: | --: | --: | --: | --: | --: | --: |
| Douglas Lindsey |	25	| Male |	11	| 57 |	25	| 4.7 |	Marketing	| Analyst |	Jan-20 |	63596 |
|	Anthony Roberson	| 59 |	Female | 19 |	55 |	76 |	2.8 |	IT |	Manager |	Jan-99 |	112540 |
| Thomas Miller	| 30	| Male	| 8 |	87 |	10	| 2.4	| IT |	Analyst	| Jan-17 |	66292 |
| Joshua Lewis	| 26	| Female	| 1	| 53	| 4	| 1.4	| Marketing	| Intern |	Jan-22 |	38303 |
| Stephanie Bailey |	43 |	Male	| 14 |	3 |	9 |	4.5 |	IT	| Team Lead |	Jan-05	| 101133 |
| Jonathan King	| 24 |	Male	| 5	| 63 |	33 |	4.2 |	Sales |	Junior Developer |	Jan-21 |	48740 |

We can check to see if there are any missing values in the dataset.

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
We can also see summary statistics of the data with the following commands.

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

## Visualizing Variable Distributions

We can define a density plot function that will visualize the distributions of quantitative variables in the dataset.

```python
# Define a density plot function
def den_plot(data):
    sns.kdeplot(x=df[data], fill=True)
    plt.gcf().set_size_inches(10,4)
```

```python
plt.subplots(1,2,sharey=True)
plt.subplot(1,2,1)
den_plot('Age')
plt.subplot(1,2,2)
den_plot('Projects Completed')
plt.subplots(1,2, sharey=True)
plt.subplot(1,2,1)
den_plot('Productivity (%)')
plt.subplot(1,2,2)
den_plot('Satisfaction Rate (%)')
plt.subplots(1,1)
plt.subplot(1,1,1)
den_plot('Feedback Score')
plt.subplots(1,1)
plt.subplot(1,1,1)
den_plot('Salary')
```
![]({{ site.url }}{{ site.baseurl }}/images/Salary/EDA_plots_1.png)<!-- -->
![]({{ site.url }}{{ site.baseurl }}/images/Salary/EDA_plots_2.png)<!-- -->
![]({{ site.url }}{{ site.baseurl }}/images/Salary/EDA_plots_3.png)<!-- -->
![]({{ site.url }}{{ site.baseurl }}/images/Salary/EDA_plots_4.png)<!-- -->

All variable distributions look approximately normal. `Age` is slightly skewed right. We can use a boxplot to detect potential outliers in `Age`.

```python
fig = px.box(df, y = 'Age', points = 'outliers')
fig.update_layout(hovermode='x')
```

![]({{ site.url }}{{ site.baseurl }}/images/Salary/age_box.png)<!-- -->

There are no outliers in `Age`.

## Investigating Gender

First, we will look at gender distribution in the dataset.

```python
fig = px.pie(sex, values='Count', names='Gender', color_discrete_sequence=['blue', 'pink'])
fig.show()
```

![]({{ site.url }}{{ site.baseurl }}/images/Salary/gender_dist.png)<!-- -->

There are exactly 100 male and 100 female employees in the dataset.

```python
mean_salary = df.groupby('Gender').mean(numeric_only=True)['Salary'].reset_index()
mean_salary.columns = ['Gender', 'Average Salary']
px.bar(mean_salary, x = 'Gender', y = 'Average Salary', color='Gender', color_discrete_sequence=['pink', 'blue'])
```
![]({{ site.url }}{{ site.baseurl }}/images/Salary/gender_avg_salary.png)<!-- -->

The average salary for males and females is equivalent.

We will now identify the average annual salary for both genders.

```python
# Create a new list to extract the join year
join_year = []
for i in df['Joining Date'].str.split('-'):
    if int(i[1]) < 23:
        join_year.append('20'+str(i[1]))
    else:
        join_year.append('19'+str(i[1]))
        
# Reassign this list to df['Joining Date']
df['Joining Date'] = pd.Series(join_year)

# Calculate average salary for each join year
avg_annual_sal = df.groupby('Joining Date').mean(numeric_only=True)['Salary'].reset_index()
avg_annual_sal.columns = ['Year', 'Average Salary']

# Split average annual salary by gender
female_aas = avg_annual_sal[avg_annual_sal['Gender']=='Female']
female_aas = female_aas.drop(['Gender'], axis=1)
female_aas.columns = ['Year', 'Female']

male_aas = avg_annual_sal[avg_annual_sal['Gender']=='Male']
male_aas = male_aas.drop(['Gender'], axis=1)
male_aas.columns = ['Year', 'Male']

# Combine gendered salaries into a shared dataframe
gendered_aas = pd.merge(male_aas,female_aas,on='Year')

# Plot average annual salaries
fig = px.line(gendered_aas, x='Year', y=['Male', 'Female'], title='Average Salary by Year', labels={'value':'Average Salary'})
fig.update_xaxes(tickangle=45)
fig.show()
```

![]({{ site.url }}{{ site.baseurl }}/images/Salary/avg_annual_salary.png)<!-- -->

## Correlation

We can create a correlation heatmap for numeric variables in the dataset.

```python
LE = preprocessing.LabelEncoder()

# Assign a binary value for Gender
df['Gender'] = LE.fit_transform(df['Gender'])

# Assign unique numeric values for each Department
df['Department'] = LE.fit_transform(df['Department'])

# Assign unique numeric values for each Position
df['Position'] = LE.fit_transform(df['Position'])

# Plot the correlation heatmap
plt.figure(figsize=(10, 8))
sns.heatmap(df.corr(numeric_only=True), annot=True)
plt.title("Variable Correlation")
plt.show()
```

![]({{ site.url }}{{ site.baseurl }}/images/Salary/correlation_heatmap.png)<!-- -->

