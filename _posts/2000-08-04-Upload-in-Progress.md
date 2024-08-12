---
title: "Upload in Progress..."
date: 2000-08-04
tags: [R, Time Series Analysis, Forecasting]
excerpt: "Upload in Progress 8/4/24"
---

## Background
There are plenty of methods that can be used to forecast time series data. The characteristics of your data can help guide your choice, but it's important to acknowledge that every model will have strengths and limitations. Traditional models like ARIMA and exponential smoothing have been widely used for decades due to their robustness and simplicity. However, new advancements in modeling can offer a fresh perspective by simplifying implementation and improving performance on complex data.

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

Our *total sales* plot appears to exhibit a slight positive trend over time—something our store owner would surely hope for! We can observe signs of seasonality in our data with sales spiking at various times of the year. This is expected since shopping habits typically shift with the changing seasons.

A time series is considered *stationary* if it maintains a constant mean and variance over time. Oftentimes, attempting to forecast a non-stationary time series will result in inaccurate predictions. This is because the forecasting model will struggle to detect underlying patterns like trends and seasonality in the data. For the reasons mentioned above, the likelihood of our time series being non-stationary is quite high. We can use the *Autocorrelation Function* (ACF) and *Partial Autocorrelation Function* (PACF) to test for stationarity. The ACF will measure how data points in a time series are correlated with each other over different lag times. The PACF will measure the correlation of the time series with its own lagged values, excluding the effects of intermediate lags. In the graphs of the ACF and PACF, a stationary time series will typically show a rapid decline in correlation as the lag increases. This rapid decline indicates that past values have little influence on future values beyond a certain point.

Let's take a look at the ACF and PACF plots of our raw timeseries.

```r
# Check Stationarity by plotting ACF and PACF
library(forecast)
par(mfrow = c(2, 1))
acf(sales_profit$Total_Sales, main = "ACF of Total Sales")
pacf(sales_profit$Total_Sales, main = "PACF of Total Sales")
```

![]({{ site.url }}{{ site.baseurl }}/images/Sales Forecasting/ACF & PACF 1.png)

The ACF does not decline quickly to zero as it has significant spikes beyond lag 5. The PACF also has several lags after lag 5 that are significantly different from zero. This is sufficient evidence to believe that our data is indeed non-stationary.

One common method to achieve stationarity is through differencing. *Differencing* involves subtracting the previous observation from the current observation in an effort to stabilize the mean of the time series. This is calculated using the following formula:

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







