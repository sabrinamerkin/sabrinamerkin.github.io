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
df.head()
```
