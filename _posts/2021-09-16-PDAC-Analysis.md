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

With a lack of screening biomarkers for PDAC, this low neoplastic cellularity in tumor samples makes the disease difficult to diagnose in its early stages.
The goal of this study is to identify the presence of differentially expressed (DE) genes between the healthy and cancerous cells that make up PDAC tumor samples. 
