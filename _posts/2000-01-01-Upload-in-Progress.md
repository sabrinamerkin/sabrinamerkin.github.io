---
title: "Upload in progress..."
date: "2000-01-01"
tags: [Python, Machine Learning, Matplotlib, Data Visualization]
excerpt: "Utilize machine learning models to estimate salary from employee metrics"
mathjax: true
---

## Background
The dataset in this project explores factors that influence employee performance and satisfaction. I investigate how different variables influence employee salary. I then create and test machine learning models to predict employee salary from the data. The *Employee Productivity and Satisfaction HR Data* was imported from Kaggle. All Python code was run in a Jupyter Notebook environment.

## Exploratory Analysis
We begin by importing the required libraries and dataset.

```python
# Import required libraries
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import missingno as msno
import plotly.express as px
from sklearn import preprocessing
from sklearn.preprocessing import StandardScaler

# Load dataset
df = pd.read_csv("C:/Users/ethan/OneDrive/Documents/Data Glacier/datasets/hr_data.csv")
df.shape
```

```python
(200, 11)
```
The dataset contains 200 employee records with 11 informative variables. We can print the first 6 rows of the dataset to get a better sense of these variables.

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

All variables are relatively intuitive to understand. It is important to note that `Projects Completed` represents the total number of projects completed out of 25. We are left to assume that all projects hold the same amount of difficulty, regardless of employee position. Next, we will check to see if there are any missing values in the dataset.

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

There are no missing values in any of the columns. Further, we can look at the data type of each column in the dataset.

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

To provide more information, we will look at summary statistics for each column in the dataset.

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

**`Name`** is not an important variable for this analysis, so we can remove it from the data frame.

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
```
![]({{ site.url }}{{ site.baseurl }}/images/Salary/EDA_plots_1.png)<!-- -->
![]({{ site.url }}{{ site.baseurl }}/images/Salary/EDA_plots_2.png)<!-- -->

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
df['Joining Date'] = pd.Series(join_year).astype(int)

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

```python
px.box(df,x='Position',y='Salary',color='Gender')
```

![]({{ site.url }}{{ site.baseurl }}/images/Salary/gender_salary_position.png)<!-- -->

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

Age and Projects Completed are highly correlated with salary (> 0.8). This can be further illustrated by scatter plots.

```python
px.scatter(df, x='Age', y='Salary', trendline='ols')
```

![]({{ site.url }}{{ site.baseurl }}/images/Salary/scatter_age_salary.png)<!-- -->

```python
px.scatter(df, x='Projects Completed', y='Salary', trendline='ols', color_discrete_sequence=['Orange'])
```

![]({{ site.url }}{{ site.baseurl }}/images/Salary/scatter_projects_salary.png)<!-- -->

We can see how average salary fluxuates by position and department.

```python
pos = df.groupby('Position').mean(numeric_only=True)['Salary'].reset_index()
fig = px.bar(pos, x='Position', y='Salary', color='Position', color_discrete_sequence=px.colors.sequential.Rainbow)
fig.update_layout(title_text='Average Salary by Position', xaxis_title='Position', yaxis_title='Salary')
fig.show()
```

![]({{ site.url }}{{ site.baseurl }}/images/Salary/salary_by_position.png)<!-- -->

```python
dept = df.groupby('Department').mean(numeric_only=True)['Salary'].reset_index()
fig = px.bar(dept, x='Department', y='Salary', color='Department', color_discrete_sequence=px.colors.sequential.Rainbow)
fig.update_layout(title_text='Average Salary by Department', xaxis_title='Department', yaxis_title='Salary')
fig.show()
```

![]({{ site.url }}{{ site.baseurl }}/images/Salary/salary_by_department.png)<!-- -->

```python
px.box(df,x='Position',y='Salary',color='Department')
```

![]({{ site.url }}{{ site.baseurl }}/images/Salary/department_salary_position.png)<!-- -->

## Predicting Salary

We will use four machine learning models to predict employee salary using information contained in this dataset. We will begin by importing the following libraries.

```python
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn.ensemble import RandomForestRegressor
from sklearn.tree import DecisionTreeRegressor
from xgboost import XGBRegressor
from sklearn import metrics
```

Prior to data modeling, it will be helpful to rescale salary values in the dataset. Standardizing this data will aid in the interpretability of the following models.

```python
# Create scaling object from sklearn.preprocessing
scaler = StandardScaler()

# Standardize Salary values
df[['Salary']] = scaler.fit_transform(df[['Salary']])

# Show updated salary column
print(df['Salary'])
```

```python
0     -0.482083
1      1.329684
2     -0.382285
3     -1.418358
4      0.907429
         ...   
195   -0.983481
196   -1.110783
197   -1.614956
198    1.021553
199    1.026180
Name: Salary, Length: 200, dtype: float64
```

We can clearly see that salary values have been rescaled to fit a standard normal distribution. This will be greatly beneficial when assessing model performance metrics. Next, we will assign test and training data for the machine learning models to use.

```python
# Initialize independent and dependent variables
X = df.drop(['Salary'], axis=1)
y = df['Salary']

# Allocate test and training data
X_train, X_test, y_train, y_test = train_test_split(X, y, train_size = 0.8, random_state = 123)
```

We will start by creating a linear regression model from our training data. Linear regression provides coefficient values that can be used to understand the impact of each feature on the target variable `Salary`. It is important to note that this model will assume a linear relationship between the predictor variables and `Salary`.

```python
# Fit a linear regression model
LR = LinearRegression()
LR.fit(X_train, y_train)

# Predict salary from test data
y_pred_lr = LR.predict(X_test)
```

A random forest regression model will be able to handle non-linearity between predictor variables and `Salary`. It is a generally robust model and is less prone to overfit the data.

```python
# Fit a random forest model
RF = RandomForestRegressor(n_estimators=500, random_state=123)
RF.fit(X_train, y_train)

# Predict salary from test data
y_pred_rf = RF.predict(X_test)
```

A decision tree regression model can handle non-linearity and interactions between features. A single decision tree might not generalize well to unseen data, but default parameters will prevent this issue.

```python
# Fit a decision tree regression model
DT = DecisionTreeRegressor(random_state=123)
DT.fit(X_train, y_train)

# Predict salary from testing data
y_pred_dt = DT.predict(X_test)
```

Lastly, we will deploy an XGBoost regression model that utilizes gradient boosting. XGBoost provies a powerful model that works well with a variety of data types. This model may require hyperparameter tuning for improvement. We will set the number of gradient boosted trees to 500.

```python
# Fit an XGBoost Regression Model
XGB = XGBRegressor(n_estimators=500, random_state=22)
XGB.fit(X_train, y_train)

# Predict salary from testing data
y_pred_xgb = XGB.predict(X_test)
```

It is time to take a look at the accuracy of these models by comparing predicted salaries with the test data. 

```python
# Define dictionaries to identify models
key = {0:'lr', 1:'rf', 2:'dt', 3:'xgb'}
name = {0:'Linear Regression', 1:'Random Forest', 2:'Decision Tree', 3:'XGBoost'}

# Format plot
plt.figure(figsize=(10, 10))
plt.subplots_adjust(hspace=0.4, wspace=0.4)

# Plot predicted salary vs actual salary
with plt.rc_context({'xtick.color':'grey','ytick.color':'grey'}):
    for i in range(4):
        plt.subplot(2,2,(i+1))
        sns.scatterplot(x=y_test, y=globals().get('y_pred_' + key[i]))
        sns.regplot(x=y_test, y=globals().get('y_pred_' + key[i]), scatter=True, scatter_kws = {"color": "g"}, line_kws = {"color": "orange", "alpha":0.5})
        plt.xlabel('Actual Salary')
        plt.ylabel('Predicted Salary')
        plt.title(name[i], fontsize=16)
```

![]({{ site.url }}{{ site.baseurl }}/images/Salary/prediction_accuracy_plots.png)<!-- -->

All four models have a clear positive correlation between the predicted and actual values for `Salary`. A simple regression line is included in each scatter plot to highlight this trend. The random forest model appears to have the best prediction values for salary. Additionally, we can create density plots to visualize the fit of model values for `Salary`. 

 ```python
plt.figure(figsize=(10, 10))

plt.subplots_adjust(hspace=0.4, wspace=0.4)

with plt.rc_context({'xtick.color':'grey','ytick.color':'grey'}):
    for i in range(4):
        plt.subplot(2,2,(i+1))
        ax = sns.kdeplot(y_test, color="g", label="Actual Salary")
        sns.kdeplot(globals().get('y_pred_' + key[i]), color="orange", label="Fitted Values", ax=ax)
        plt.legend()
        plt.title(name[i], fontsize=16)
```

![]({{ site.url }}{{ site.baseurl }}/images/Salary/model_fit_plots.png)<!-- -->

Finally, we will construct a table that allows us to compare performance metrics for each model.

```python
# Initialize Table Columns
results = {'Model':[], 'Mean Squared Error':[], 'Mean Absolute Error':[], 'R-squared':[], 'Adjusted R-squared':[]}

# Store models in a list
models = [LR, RF, DT, XGB]

# Fill in table values
for i in range(4):
    results['Model'].append(name[i])

for mod in models:
    # Fit model
    mod.fit(X_train, y_train)
    y_pred = mod.predict(X_test)
    
    # Metric calculations
    mse = metrics.mean_squared_error(y_test, y_pred)
    mae = metrics.mean_absolute_error(y_test, y_pred)
    r2 = metrics.r2_score(y_test, y_pred)
    adj_r2 = 1-(1-r2)*(len(y_train)-1)/(len(y_train)-X_train.shape[1]-1)
    
    results['Mean Squared Error'].append(mse)
    results['Mean Absolute Error'].append(mae)
    results['R-squared'].append(r2)
    results['Adjusted R-squared'].append(adj_r2)
    
    
table = pd.DataFrame(results)   
```

| Model	| Mean Squared Error	| Mean Absolute Error |	R-squared |	Adjusted R-squared |
| :-- | --- | --- | --- | --- |
| Linear Regression |	0.127851	| 0.279753	| 0.878521 |	0.871232 |
| Random Forest	| 0.033916	| 0.144999	| 0.967774	| 0.965840 |
| Decision Tree	| 0.054168	| 0.172992	| 0.948531	| 0.945443 |
| XGBoost	| 0.056552	| 0.186158	| 0.946266 |	0.943042 |

The **Mean Squared Error** measures an average of the squared differences between the estimated values and actual values of `Salary`. A lower mean squared error indicates that model predictions are closer to the actual values on average, which is a measure of accuracy.

The **Mean Absolute Error** is another measure of model error between these paired observations. As it follows, a lower mean absolute error indicates better accuracy in predicting `Salary`.

The **Coefficient of Determination (R-squared)** measures the proportion of variance in `Salary` that is explained by the model. These values usually range from 0 to 1. A model with a higher coefficient of determination provides a better fit to the data.

The **Adjusted Coefficient of Determination (Adjusted R-squared)** accounts for overfitting by penalizing the addition of independent variables to a model. Similar to R-squared, an adjusted R-squared value close to 1 indicates a good model fit.

The random forest model is the best of the four models. It has the lowest mean squared error and lowest mean absolute error of the models. The random forest model also has the greatest standard and adjusted coefficients of determination (R-squared and Adjusted R-squared). 








