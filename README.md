# Bellabeat-Case-Study
##### Author: Christine Pang
##### Date: Nov 17, 2024



## Introduction
Bellabeat, a high-tech company founded in 2013, specializes in creating health-focused smart products for women. By collecting data on activity, sleep, stress, and reproductive health, Bellabeat empowers women to gain valuable insights into their own well-being and daily habits. Over the years, Bellabeat has grown rapidly, establishing itself as a tech-driven wellness brand for women. Our team has been tasked with analyzing smart device fitness data to understand how consumers use these devices. The insights gained from this analysis will help uncover new growth opportunities for Bellabeat and guide the company's future strategies.

## ASK
### Business Task
The business task was to analyse smart device usage data in order to gain insights into how consumers use non-Bellabeat smart devices. Then selecting one Bellabeat product to apply these insights to in the presentation. 
The business questions are:
1. What are some trends in smart device usage? 
2. How could these trends apply to Bellabeat customers? 
3. How could these trends help in influence Bellabeat marketing strategy

### Key Stakeholders
•	Urška Sršen: Bellabeat’s co-founder and Chief Creative Officer

•	Sando Mur: Mathematician and Bellabeat’s co-founder; key member of the Bellabeat executive team

•	Bellabeat marketing analytics team: A team of data analysts responsible for collecting, analyzing, and reporting data that helps guide Bellabeat’s marketing strategy

## PREPARE
I used the Public Data Set: FitBit Fitness Tracker Data (CC0: Public Domain, dataset made available through Mobius): This Kaggle data set contains personal fitness tracker from thirty fitbit users. Thirty eligible Fitbit users consented to the submission of personal tracker data, including minute-level output for physical activity, heart rate, and sleep monitoring. It includes information about daily activity, steps, and heart rate that can be used to explore users’ habits.

### Data Integrity and Credibility
This data, collected in 2016 from 30 participants, may no longer reflect current trends, especially given the technological advancements since then. With such a small sample size, the findings might not represent the broader female population accurately. Additionally, key details like participants' age, lifestyle, health concerns, job, and location are missing; these factors could be invaluable for identifying specific customer segments if we were to target a particular market.

**Reliability**: 30 eligible Fitbit users consented to the submission of personal tracker data  
**Original**:  Third party data provider: Amazon Mechanical Turk.   
**Comprehensive**: Has data that consist of minute-level output for physical activity, heart rate, and sleep monitoring.    
**Current**: The data is from March 2016 to May 2016 which is not current and is 8 years outdated.    
**Cited**: Data obtained from Amazon Mechanical Turk.

## PROCESS
--Load Packages--
``` r
install.packages('tidyverse')
install.packages('skimr')
install.packages('cowplot')
install.packages('plotly')
install.packages('lm.beta')
```
``` r
library(plotly)
library(tidyverse) #wrangle data
library(dplyr) #clean data
library(lubridate) #wrangle date attributes
library(skimr) #get summary data
library(ggplot2) #visualize data
library(cowplot) #grid the plot
library(readr) #save csv 
library(plotly) #pie chart
library(lm.beta) 
```
--Load and View the CSV files--
``` r
setwd("/cloud/project")

daily_activity <- read.csv("dailyActivity_merged.csv")
sleep_day <- read.csv("sleepDay_merged.csv")
weight <- read.csv("weightLogInfo_merged.csv")
hourly_step <- read.csv("hourlySteps_merged.csv")
```
``` r
head(daily_activity)
```

``` r
head(sleep_day)
```

    ##           Id              SleepDay TotalSleepRecords TotalMinutesAsleep TotalTimeInBed
    ## 1 1503960366 4/12/2016 12:00:00 AM                 1                327            346
    ## 2 1503960366 4/13/2016 12:00:00 AM                 2                384            407
    ## 3 1503960366 4/15/2016 12:00:00 AM                 1                412            442
    ## 4 1503960366 4/16/2016 12:00:00 AM                 2                340            367
    ## 5 1503960366 4/17/2016 12:00:00 AM                 1                700            712
    ## 6 1503960366 4/19/2016 12:00:00 AM                 1                304            320

``` r
**Notice how the 12:00:00 AM timestamp on each observation is unnecessary, so it should be removed to simplify the data for easier anaysis.** 
``` r
head(weight)
```
