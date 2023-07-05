---
title: "R: Differential Expression of Genes in PDAC Tumor Samples"
date: 2022-04-14
tags: [R, Rstudio, ggplot2, tweedie]
excerpt: "Identifying DE genes using R"
---

## Background

The purpose of this project was to identify differentially expressed genes from an over-dispersed count data set comprised of 148 PDAC tumor samples. Each tumor sample held a neoplastic cellularity percentage and count values corresponding to one of 17394 genes per sample. This project began with an analysis of the Poisson and negative binomial distributions (two common distributions used to model count data) and their effectiveness in modeling count data exhibiting overdispersion. It was determined that neither of these distributions could meet the desired accuracy of modeling this PDAC data set. Normal Quantile-Quantile plots were also constructed to visualize these inaccuracies. The Poisson-Tweedie distribution (from the 'tweedie' CRAN R package) demonstrated suitability in modeling the PDAC data set after conducting 12000+ simulated experiments testing the goodness-of-fit of estimated model parameters on over-dispersed data. An additional series of simulation studies were performed to test the empirical power of a differential-expression-test function on distinct Poisson-Tweedie distributions. This function proved to be powerful and effective in detecting differential expression between subsets in the PDAC data set. The PDAC data was split into two subsets for the first analysis. All tumor samples with low neoplastic cellularity levels (less than 0.5) were assigned to group A. The remaining tumor samples with high neoplastic cellularity levels (greater than or equal to 0.5) were assigned to group B. The second analysis focused on the explicit neoplastic cellularity values of each tumor sample. Linear regression modeling was performed on both analyses to successfully detect 1403 consensus genes from the original data supporting known prognostic markers in human cancer.

## Description

Pancreatic Ductal Adenocarcinoma (often referred to as PDAC) is a type of pancreatic cancer that is the 4th leading cause of cancer-related deaths around the world today. This disease is projected to become the 2nd leading cause of cancer deaths by 2030. There are several challenges in treating PDAC. Due to the lack of effective treatment options, the median survival of PDAC is 6 months, and the 5-year survival rate is less than 10% after diagnosis. Neoplastic cellularity is the term used to define the proportion of cancerous cells to healthy cells within a bulk tumor sample. PDAC tumor samples tend to have low neoplastic cellularity, meaning they are comprised of significantly fewer cancerous cells than normal cells.

![]({{ site.url }}{{ site.baseurl }}/images/PDAC Images/Neo_cell.png)<!-- -->

With a lack of screening biomarkers for PDAC, this low neoplastic cellularity in tumor samples makes the disease difficult to diagnose in its early stages.
The goal of this study is to identify the presence of differentially expressed (DE) genes between the healthy and cancerous cells that make up PDAC tumor samples.

Modeling gene expression for this study requires a probability model that can measure genetic count data from PDAC tumor samples. The following table depicts a simple example of how the count data is formatted. Both the Poisson and Negative Binomial distributions are two common count-data probability models that were initially considered. It is important to note the presence of overdispersion in certain data sets. Overdispersion is a property that occurs in a data set when the variance of the data is greater than the expected value of the data, denoted as µ. This particular study of DE genes requires a probability distribution that can model over-dispersed count data. When considering the Poisson model, it was important to recognize the expected value of the distribution is always equal to the variance of the distribution. Therefore, a Poisson model cannot account for overdispersion in a count data set, and thus was not the best choice to model the data. The Negative Binomial model, on the other hand, has the ability to model overdispersion in count data. However, it is still rare for a Negative Binomial distribution to model a count data set with the desired accuracy for this study. The Poisson-Tweedie (PT) model utilizes more parameters than both previous distributions and consequently offers more accuracy in modeling an over-dispersed count data set. These parameters include µ, the expected value; D, the dispersion index; and α, the shape parameter of the distribution. By modifying the shape parameter α, a PT model can mimic both the Poisson and Negative Binomial distributions, providing the most flexible model for this data study (this will also be seen in the following slides). Multiple simulation studies were conducted to further analyze the benefits of using the PT model.

Four simulation studies were conducted to evaluate estimated PT distribution parameters. These simulations tested the accuracy of a PT-parameter estimating function. Each scenario was tested using 3 different sample sizes (100, 200, and 500). This was done to investigate the effectiveness of the function when increasing the sample size of the simulated data. 


![]({{ site.url }}{{ site.baseurl }}/images/PDAC Images/sim_trials_1.png)<!-- -->


Here is a standard procedure for one of the simulations in scenario II:


``` r
# rtweedie Scenario II

library(tweedie)
library(tweeDEseq)
set.seed(123)
mu.res <- d.res <- a.res <- NB.pvalues <- Pois.pvalues <- c(1:1000) #Create five vectors of size 1000

for (i in 1:1000){
  
y <- rPT(n=200, mu=20, D=5, a=-1)
thetahat <- mlePoissonTweedie(y)

mu.res[i] <- getParam(thetahat)[1]
d.res[i] <- getParam(thetahat)[2]
a.res [i] <- getParam(thetahat)[3]


#Test fit to Negative Binomial Distribution
NB.pval <- testShapePT(thetahat, a=0)
NB.pvalues[i] <- NB.pval$pvalue

#Test fit to Poisson Distribution
Pois.pval <- testShapePT(thetahat, a=1)
Pois.pvalues[i] <- Pois.pval$pvalue


}

mean(mu.res) #Average mu value
mean(d.res) #Average D value
mean(a.res) #Average a value

NB.pvalues <- sort(NB.pvalues, decreasing = FALSE) #Sorts p-values from lowest to highst
which(NB.pvalues<0.05) #Lists indicies of significant p.values lower than 0.05

Pois.pvalues <- sort(Pois.pvalues, decreasing = FALSE) #Sorts p-values from lowest to highst
which(Pois.pvalues<0.05) #Lists indicies of significant p.values lower than 0.05

hist(mu.res)
hist(d.res)
hist(a.res)
```


![]({{ site.url }}{{ site.baseurl }}/images/PDAC Images/sim_plots_1.png)<!-- -->


By repeating similar simulations that meet the sample size requirements for each scenario, we obtain the following visual:


![]({{ site.url }}{{ site.baseurl }}/images/PDAC Images/sim_tables_1.png)<!-- -->


The results show that the PT-parameter estimating function provides good estimates of the PT parameters µ and D regardless of sample size. As for the PT parameter α, increasing the sample size of the simulated data improved the parameter estimates.

The same four simulated scenarios were also used to test the empirical power of the PT-goodness-of-fit function. These simulations tested whether or not a particular scenario’s distribution shape deviated from that of the Negative Binomial or Poisson distribution. The shape of each distribution from the four scenarios were compared to the Negative Binomial and Poisson distributions holding PT parameter values α=0 and α=1, respectively. This shape test was performed on each of the 1000 replicates for every simulation.


