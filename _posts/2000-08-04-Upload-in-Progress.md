---
title: "Upload in Progress..."
date: 2000-08-04
tags: [R, Time Series Analysis, Forecasting]
excerpt: "Upload in Progress 8/4/24"
---

## Background
There are plenty of methods that can be used to forecast time series data. The characteristics of your data can help guide your choice, but it's important to acknowledge that every model will have strengths and limitations. Traditional models like ARIMA and exponential smoothing have been widely used for decades due to their robustness and simplicity. However, new advancements in modeling can offer a fresh perspective by simplifying implementation and improving performance on complex data.

Using daily sales from a [Superstore](https://www.kaggle.com/datasets/vivek468/superstore-dataset-final), we'll evaluate traditional time series forecasting methods and explore Facebook's 2017 Prophet model package in R. We will assess model diagnostics and accuracy between these approaches while considering the potential drawbacks of different models.

## Traditional Time Series Modeling
We will begin by loading our superstore data into RStudio and formatting a time series dataframe.

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

Our data ranges from January 2014 to December 2017. For each day in the series, we have *total sales* and *total profit* values. We will only focus on *total sales* for now. Let's investigate if any seasonality or trends are present in the data.

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

Our *total sales* plot exhibits a slight positive trend over time. We can observe signs of seasonality in our data with sales spiking at various times of the year. This can be expected as shopping habits typically shift with holidays, promotonal events, and seasons.

A time series is considered *stationary* if its mean and variance remain constant over time. When traditional models are used to forecast a non-stationary time series, their predictions often lack reliability. This is because they struggle to detect underlying patterns in the data, like trends and seasonality. For this reason, it's quite likely our time series is non-stationary. We will use an *Autocorrelation Function* (ACF) and *Partial Autocorrelation Function* (PACF) to test for stationarity in the data. The ACF measures how data points in our time series are correlated with each other over different lag times. The PACF measures the correlation of the time series with its own lagged values, excluding the effects of intermediate lags. Typically, graphing the ACF and PACF of a stationary time series will show a rapid decline in correlation as the lag increases. This rapid decline indicates that past values have little influence on future values beyond a certain point.

Let's take a look at the ACF and PACF plots of our raw timeseries.

```r
# Check Stationarity by plotting ACF and PACF
library(forecast)
par(mfrow = c(2, 1))
acf(sales_profit$Total_Sales, main = "ACF of Total Sales")
pacf(sales_profit$Total_Sales, main = "PACF of Total Sales")
```

![]({{ site.url }}{{ site.baseurl }}/images/Sales Forecasting/ACF & PACF 1.png)

The ACF does not decline quickly to zero as it has significant spikes beyond lag 5. The PACF also has several lags after lag 5 that are significantly different from zero. These plots further indicate that our time series is non-stationary.

The most common way to achieve stationarity is through a method called differencing. *Differencing* involves subtracting the previous observation from the current observation in an effort to stabilize the mean of the time series. This is calculated using the following formula:

**ΔY<sub>t</sub> = Y<sub>t</sub> - Y<sub>t-1</sub>**

Where **ΔY<sub>t</sub>** is the first difference, **Y<sub>t</sub>** is the current observation, and **Y<sub>t-1</sub>** is the previous observation. This transformation can help stabilize the mean of the time series.

We will use the *diff* function in R to take the first difference of our time series.

```r
# Calculate the first difference of the Total Sales
diff_total_sales <- diff(sales_profit$Total_Sales, differences = 1)

# Create a new data frame for the differenced series
diff_sales_profit <- sales_profit[-1, ] # removes first row to match first difference dimensions
diff_sales_profit$Diff_Total_Sales <- diff_total_sales

# Plot the differenced time series for total sales
ggplot(diff_sales_profit, aes(x = Order_Date, y = Diff_Total_Sales)) +
  geom_line(color = "blue") +
  labs(title = "Differenced Time Series of Total Sales",
       x = "Order Date",
       y = "Differenced Total Sales") +
  theme_minimal()
```

![]({{ site.url }}{{ site.baseurl }}/images/Sales Forecasting/First Difference Plot.png)







