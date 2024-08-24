---
title: "Upload in Progress..."
date: 2000-08-04
tags: [R, Time Series Analysis, Forecasting]
excerpt: "Upload in Progress 8/4/24"
---

## Background
There are plenty of methods that can be used to forecast a time series. While data characteristics can help guide your approach to forecasting, it's important to acknowledge every model's strengths and limitations. Popular models like ARIMA and exponential smoothing have been used for decades due to their robustness and simplicity. In recent years, newer models have made implementation easier and boosted overall performance on complex datasets. But can we really say which one is superior?

Using daily sales from a [Superstore](https://www.kaggle.com/datasets/vivek468/superstore-dataset-final), we'll evaluate traditional time series forecasting methods and explore Facebook's 2017 Prophet model package in R. This data was sourced from Kaggle -- a platform that hosts a rich collection of real-world datasets.

## "Traditional" Time Series Modeling
Let's begin by loading our Superstore data into RStudio and formatting a time series dataframe.

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

Our data was collected from January 2014 to December 2017. For each day in the series, we have *total sales* and *total profit* values. We will only focus on *total sales* for now. Let's look for any seasonality or trends present in the data.

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

Our plot of *total sales* shows a slight positive trend over time. We can also observe signs of seasonality in the data with sales spiking at various times of the year. This can be expected given that shopping habits often change due to holidays, promotional events, market conditions, and other factors.

A time series is considered *stationary* if its expected value, autocorrelation, and variance remain constant over time. When models are used to forecast a non-stationary time series, their predictions often lack accuracy. This is because they struggle to detect underlying patterns in the data like trends and seasonality. For this exact reason, it's likely our time series is non-stationary. We will use an *Autocorrelation Function* (ACF) and *Partial Autocorrelation Function* (PACF) to test for stationarity in the data. The ACF measures how data points in our time series are correlated with each other over different past value times (lags). The PACF measures the correlation of our time series with its own lagged values, excluding the effects of intermediate lags. Typically, graphing the ACF and PACF of a stationary time series will show a rapid decline in correlation as the lag increases. This rapid decline indicates that past values have little influence on future values beyond a certain point.

Let's take a look at the ACF and PACF of our sales data to explore stationarity before forecasting.

```r
# Check stationarity by plotting ACF and PACF
library(forecast)
par(mfrow = c(2, 1))
acf(sales_profit$Total_Sales, main = "ACF of Total Sales")
pacf(sales_profit$Total_Sales, main = "PACF of Total Sales")
```

![]({{ site.url }}{{ site.baseurl }}/images/Sales Forecasting/ACF & PACF 1.png)

The blue dashed lines in both plots represent 95% confidence bounds. Values that fall outside these bounds indicate statistically significant correlations that may need to be accounted for in the model. In our plots, the ACF does not decline quickly to zero as it has significant spikes beyond lag 5. The PACF also has several lags after lag 5 that are significantly different from zero. These plots further indicate that our time series is non-stationary.

The most common way to achieve stationarity is through a method called *differencing*. Differencing involves subtracting the previous observation from the current observation in an effort to stabilize the mean and variance of the time series. This is calculated using the following formula:

**ΔY<sub>t</sub> = Y<sub>t</sub> - Y<sub>t-1</sub>**

Where **ΔY<sub>t</sub>** is the first difference, **Y<sub>t</sub>** is the current observation, and **Y<sub>t-1</sub>** is the previous observation.

We will use the *diff* function in R to take the first difference of our time series.

```r
# Calculate the first difference of Total Sales
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

The mean of this new plot appears to be around zero, indicating that differencing has removed most of the trend component. Compared to the first plot, the variation in the differenced series is slightly more consistent over time. This suggests that differencing also helped stabilize the variance as we had hoped!

Next, using the *Augmented Dicky-Fuller Test*, we will test for stationarity in the first-differenced time series. The ADF test evaluates a null hypothesis that a *unit root* is present in the time series. If the p-value of this test falls below 0.05, we can reject the null hypothesis. This suggests that the series is stationary.

```r
# Test stationarity using the Augmemted Dickey-Fuller Test
library(tseries)
adf.test(diff_sales_profit$Total_Sales) # Significant p-value suggests differenced series is stationary

 ------------------------------------------------------------
	Augmented Dickey-Fuller Test

data:  diff_sales_profit$Total_Sales
Dickey-Fuller = -8.1878, Lag order = 10, p-value = 0.01
alternative hypothesis: stationary
```

A p-value of 0.01 indicates that our differenced time series is stationary. Once more, we'll look at ACF and PACF plots of our differenced series.

```r
# Plot ACF and PACF of the differenced series
par(mfrow = c(2, 1))  # Set up the plotting area for two plots
acf(diff_sales_profit$Diff_Total_Sales, main = "ACF of Differenced Total Sales")
pacf(diff_sales_profit$Diff_Total_Sales, main = "PACF of Differenced Total Sales")
```

![]({{ site.url }}{{ site.baseurl }}/images/Sales Forecasting/ACF & PACF 2 (Differenced).png)

As we can see, the ACF and PACF show a rapid decline in correlation as lags increase. Again, this backs our statement that the differenced time series is stationary.

Like the title of this section suggests, we'll be looking at more "traditional" forecasting methods to predict future sales. These models have been used in time series forecasting for quite some time.

- **AR(*p*)**: The Autoregressive model uses a linear combination of lags to predict future values. The parameter value *p* represents the number of lagged observations used in the model formula (I will refrain from diving too deep into these calculations here). AR models are most beneficial when a time series has a strong correlation with its past values.

- **MA(*q*)**: The Moving Average model uses previous forecasting errors to make predictions. The parameter *q* indicates the number of lagged forecasting errors considered by the model. MA models are useful when forecasting errors are highly correlated over time.

- **ARIMA(*p*,*d*,*q*)**: The AutoRegressive Integrated Moving Average model combines elements of the AutoRegressive (AR) model, Moving Average (MA) model, and differencing. Of the three models discussed, ARIMA models are the most versatile for handling real-world data. This is because data is often non-stationary and requires some degree of differencing.

Because we already differenced our sales time series, we will focus our attention on the ARIMA(*p*,*d*,*q*) model with *d*=1. We can use the following rules to determine *p* and *q*.

- If the ACF declines quickly to zero as lags increase and the PACF has significant spikes at lags 1 to *p*, an AR(*p*) model should be considered.
- If the ACF has significant spikes at lags 1 to *q* and the PACF declines quickly to zero, an MA(*q*) model should be considered.

Looking back at our ACF and PACF plots, we see the ACF has a single significant spike at lag 1, and the PACF spikes decline quickly to zero. Thus, we determine a value of 1 for *q*. In summary, we will consider an ARIMA(0,1,1) model to forecast our original sales data.

```r
# Fit ARIMA(0,1,1) model
sales_arima_model <- arima(sales_profit$Total_Sales, c(0,1,1))

# Summarize the model
summary(sales_arima_model)

 ------------------------------------------------------------
Call:
arima(x = sales_profit$Total_Sales, order = c(0, 1, 1))

Coefficients:
          ma1
      -0.9641
s.e.   0.0098

sigma^2 estimated as 5107288:  log likelihood = -11300.87,  aic = 22605.75

Training set error measures:
                   ME     RMSE      MAE      MPE     MAPE      MASE       ACF1
Training set 52.00244 2259.017 1487.412 -866.918 897.8833 0.7558027 0.03421496
```

The model diagnostics calculated above using the *summary* function will be useful later when comparing the performance of alternative forecasting models. Next, we will check the residuals (error terms) of our ARIMA(0,1,1) model to see if they are random. We would like our residuals to follow a white noise distribution.

**ϵ<sub>t</sub> ∼ WN(0,σ<sup>2</sup>)**

*White noise* (WN) has a mean of zero, a constant variance, and no autocorrelation between residuals at different lags.

```r
# Plot residual diagnostics
checkresiduals(sales_arima_model)
```

![]({{ site.url }}{{ site.baseurl }}/images/Sales Forecasting/ARIMA Residuals.png)

In the top plot, we see our model residuals over time. Ideally, these residuals should fluctuate around zero with no clear pattern. While the residuals are mostly centered around zero, there are some large spikes. These spikes likely indicate outliers or periods where the model did not fit the data well. In the bottom left, an ACF plot of <u>residuals</u> reveals one significant autocorrelation at lag 18. While there could be a pattern in the residuals that the model has not captured, it's likely due to random variation. In the bottom right, we see a distribution of residuals that are mostly centered at zero (with the exception of a few large outliers). The ARIMA(0,1,1) model seems to have captured the general pattern of the data well.

Additionally, we can perform the *Ljung-Box Test* to check for significant autocorrelation in the residuals of the model. This test helps determine whether the residuals are independently distributed. A p-value less than 0.05 would indicate significant autocorrelation in residuals.

```r
# Ljung-Box Test
res = residuals(sales_arima_model)
box_ljung_test = Box.test(res, lag = 20, type = "Ljung-Box")
print(box_ljung_test)

------------------------------------------------------------
Box-Ljung test

data:  res
X-squared = 23.103, df = 20, p-value = 0.2838
```

With a p-value of 0.2838, we fail to reject the null hypothesis and assume there is no significant autocorrelation in the residuals.

Now that we've manually selected an ARIMA model, let's see how it compares to the model that R's auto.arima function identifies as the best fit. This function automatically selects ARIMA parameters from the original time series as we did.

```r
# Check if proposed model matches auto.arima() function
test_arima = auto.arima(sales_profit$Total_Sales)

summary(test_arima) # also proposes ARIMA(0,1,1)

------------------------------------------------------------
Series: sales_profit$Total_Sales 
ARIMA(0,1,1) 

Coefficients:
          ma1
      -0.9641
s.e.   0.0098

sigma^2 = 5111424:  log likelihood = -11300.87
AIC=22605.75   AICc=22605.76   BIC=22615.98

Training set error measures:
                   ME     RMSE      MAE      MPE     MAPE      MASE       ACF1
Training set 52.00244 2259.017 1487.412 -866.918 897.8833 0.7558027 0.03421496
```
