---
title: "Upload in Progress..."
date: 2000-08-04
tags: [R, Time Series Analysis, Forecasting]
excerpt: "Upload in Progress 8/4/24"
---

## Background
There are plenty of methods that can be used to forecast time series data. The characteristics of your data can help guide your choice, but it's important to acknowledge that every model will have strengths and limitations.

Traditional models like ARIMA and exponential smoothing have been widely used for decades due to their robustness and simplicity. However, new advancements in modeling can offer fresh perspectives by simplifying implementation and improving performance on complex data.

Using daily sales from a [Superstore](https://www.kaggle.com/datasets/vivek468/superstore-dataset-final) dataset, we'll explore traditional time series forecasting methods and compare them to Facebook's 2017 Prophet model. We will assess model diagnostics and accuracy between these approaches while considering the potential drawbacks of different models. Lastly, we'll take a deep dive into the features of the Facebook Prophet model package in RStudio.

## Traditional Time Series Modeling
Let's start creating some forecasts! First, we'll load the superstore data into RStudio and format our time series dataframe.

```R
# Load in Superstore data from Kaggle
library(readr)
superstore <- read_csv("R Studio Projects/R Datasets/Sample - Superstore.csv")

# Install libraries for aggregating sales & profit data
library(dplyr)
library(lubridate)

# Convert "Order Date" to date format
superstore$`Order Date` = as.Date(superstore$`Order Date`, format="%m/%d/%Y")

# Group by "Order Date" and aggregate sales and profit
sales_profit = superstore %>%
  group_by(`Order Date`) %>%
  summarize(Total_Sales = sum(Sales, na.rm = TRUE),
            Total_Profit = sum(Profit, na.rm = TRUE))

# Change variable name
names(sales_profit)[1]="Order_Date"

# Display the first few rows of the new dataframe
head(sales_profit)
```

| Order_Date | Total_Sales | Total_Profit |
| --- | --: | --: |
| 2014-01-03 | 16.4 | 5.55 |
| 2014-01-04 | 288.0 | -66.0 |
| 2014-01-05 | 19.5 | 4.88 |
| 2014-01-06 | 4407.0 | 1358.0 |
| 2014-01-07 | 87.2 | -72.0 |
| 2014-01-09 | 40.5 | 10.9 |

Our time series ranges from January 2014 to December 2017. For each day in the series, we have *total sales* and *total profit* values. We will only focus on *total sales* for now. Let's investigate further to see if any seasonality or trends are present in the data.

```R
# Plot sales time series
library(ggplot2)
ggplot(sales_profit, aes(x = Order_Date, y = Total_Sales)) +
  geom_line(color = "blue") +
  labs(title = "Time Series of Total Sales",
       x = "Order Date",
       y = "Total Sales") +
  theme_minimal()
```

![]({{ site.url }}{{ site.baseurl }}/images/Sales Forecasting/Raw Sales Plot.png)

Looking at our *total sales* plot, we see a slight positive trend over time — something a store owner would definitely hope for! We also observe signs of seasonality in the data. This can be expected as shopping patterns generally change throughout the year.

With the presence of an apparent trend, we will need to test the stationarity of our data before constructing a forecasting model. If the data is not stationary, we will take...


