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
--Load CSV files--
``` r
setwd("/cloud/project")

sleep_day <- read.csv("sleepDay_merged.csv")
daily_activity <- read.csv("dailyActivity_merged.csv")
weight <- read.csv("weightLogInfo_merged.csv")
hourly_steps <- read.csv("hourlySteps_merged.csv")
```
--Count the Distinct IDs in Each Dataset to Determine the Number of Participants--
```r
n_distinct(sleep_day$Id)
```
    ## [1] 24 
*sleep_day has 24 user data*
```r
n_distinct(daily_activity$Id)
```
    ## [1] 33 
*daily_activity has 33 user data*
```r
n_distinct(hourly_step$Id)
```
    ## [1] 33 
*hourly_steps has 33 user data*
```r
n_distinct(weight$Id)
```
    ## [1] 8 
*weight has 8 user data. Due to this limited sample size, it will be excluded.*

--View, Clean, and Merge Data Set--
``` r
head(sleep_day)
```

    ##       Id              SleepDay TotalSleepRecords TotalMinutesAsleep TotalTimeInBed
    ## 1 1503960366 4/12/2016 12:00:00 AM                 1                327            346
    ## 2 1503960366 4/13/2016 12:00:00 AM                 2                384            407
    ## 3 1503960366 4/15/2016 12:00:00 AM                 1                412            442
    ## 4 1503960366 4/16/2016 12:00:00 AM                 2                340            367
    ## 5 1503960366 4/17/2016 12:00:00 AM                 1                700            712
    ## 6 1503960366 4/19/2016 12:00:00 AM                 1                304            320
##### *The 12:00:00 AM timestamp is redundant and unnecessary. Removing it will simplify the data. Additionally, the SleepDay column will be renamed to Date for consistency.*
```r
# Convert SleepDay to Date format (removes time part) 
sleep_day$SleepDay <- as.Date(sleep_day$SleepDay, format="%m/%d/%Y")

# Format the SleepDay to "MM/DD/YYYY" format
sleep_day$SleepDay <- format(sleep_day$SleepDay, "%m/%d/%Y")

# Rename the column 'SleepDay' to 'Date'
colnames(sleep_day)[colnames(sleep_day) == "SleepDay"] <- "Date"

# Check the result
head(sleep_day)
```
    ##           Id       Date TotalSleepRecords TotalMinutesAsleep TotalTimeInBed
    ## 1 1503960366 04/12/2016                 1                327            346
    ## 2 1503960366 04/13/2016                 2                384            407
    ## 3 1503960366 04/15/2016                 1                412            442
    ## 4 1503960366 04/16/2016                 2                340            367
    ## 5 1503960366 04/17/2016                 1                700            712
    ## 6 1503960366 04/19/2016                 1                304            320

``` r
head(daily_activity)
```
    ##    Id ActivityDate TotalSteps TotalDistance TrackerDistance LoggedActivitiesDistance
    ## 1 1503960366    4/12/2016      13162          8.50            8.50                        0
    ## 2 1503960366    4/13/2016      10735          6.97            6.97                        0
    ## 3 1503960366    4/14/2016      10460          6.74            6.74                        0
    ## 4 1503960366    4/15/2016       9762          6.28            6.28                        0
    ## 5 1503960366    4/16/2016      12669          8.16            8.16                        0
    ## 6 1503960366    4/17/2016       9705          6.48            6.48                        0
    ##   VeryActiveDistance ModeratelyActiveDistance LightActiveDistance SedentaryActiveDistance
    ## 1               1.88                     0.55                6.06                       0
    ## 2               1.57                     0.69                4.71                       0
    ## 3               2.44                     0.40                3.91                       0
    ## 4               2.14                     1.26                2.83                       0
    ## 5               2.71                     0.41                5.04                       0
    ## 6               3.19                     0.78                2.51                       0
    ##   VeryActiveMinutes FairlyActiveMinutes LightlyActiveMinutes SedentaryMinutes Calories
    ## 1                25                  13                  328              728     1985
    ## 2                21                  19                  217              776     1797
    ## 3                30                  11                  181             1218     1776
    ## 4                29                  34                  209              726     1745
    ## 5                36                  10                  221              773     1863
    ## 6                38                  20                  164              539     1728
##### *The columns LoggedActivitiesDistance and SedentaryActiveDistance do not provide valuable information. Removing these columns will simplify the data.*
```r
# Remove the columns LoggedActivitiesDistance and SedentaryActiveDistance
daily_activity <- daily_activity %>% 
  select(-LoggedActivitiesDistance, -SedentaryActiveDistance)

# View the updated dataset
head(daily_activity)
```
    ##          Id ActivityDate TotalSteps TotalDistance TrackerDistance VeryActiveDistance
    ## 1 1503960366    4/12/2016      13162          8.50            8.50               1.88
    ## 2 1503960366    4/13/2016      10735          6.97            6.97               1.57
    ## 3 1503960366    4/14/2016      10460          6.74            6.74               2.44
    ## 4 1503960366    4/15/2016       9762          6.28            6.28               2.14
    ## 5 1503960366    4/16/2016      12669          8.16            8.16               2.71
    ## 6 1503960366    4/17/2016       9705          6.48            6.48               3.19
     ##  ModeratelyActiveDistance LightActiveDistance VeryActiveMinutes FairlyActiveMinutes
    ## 1                     0.55                6.06                25                  13
    ## 2                     0.69                4.71                21                  19
    ## 3                     0.40                3.91                30                  11
    ## 4                     1.26                2.83                29                  34
    ## 5                     0.41                5.04                36                  10
    ## 6                     0.78                2.51                38                  20
    ##   LightlyActiveMinutes SedentaryMinutes Calories
    ## 1                  328              728     1985
    ## 2                  217              776     1797
    ## 3                  181             1218     1776
    ## 4                  209              726     1745
    ## 5                  221              773     1863
    ## 6                  164              539     1728
##### *Adding a Weekday column to daily_activity to indicate the day of the week for each entry. Additionally, the ActivityDate column will be renamed to Date for consistency.*
```r
# Convert 'ActivityDate' to Date format, create 'Weekday' column, and rename 'ActivityDate' to 'Date'
daily_activity <- daily_activity %>%
  mutate(Weekday = weekdays(as.Date(ActivityDate, "%m/%d/%Y"))) %>%
  rename(Date = ActivityDate)

# View the updated dataset
head(daily_activity)
```
    ##           Id      Date TotalSteps TotalDistance TrackerDistance VeryActiveDistance
    ## 1 1503960366 4/12/2016      13162          8.50            8.50               1.88
    ## 2 1503960366 4/13/2016      10735          6.97            6.97               1.57
    ## 3 1503960366 4/14/2016      10460          6.74            6.74               2.44
    ## 4 1503960366 4/15/2016       9762          6.28            6.28               2.14
    ## 5 1503960366 4/16/2016      12669          8.16            8.16               2.71
    ## 6 1503960366 4/17/2016       9705          6.48            6.48               3.19
    ##   ModeratelyActiveDistance LightActiveDistance VeryActiveMinutes FairlyActiveMinutes
    ## 1                     0.55                6.06                25                  13
    ## 2                     0.69                4.71                21                  19
    ## 3                     0.40                3.91                30                  11
    ## 4                     1.26                2.83                29                  34
    ## 5                     0.41                5.04                36                  10
    ## 6                     0.78                2.51                38                  20
    ##   LightlyActiveMinutes SedentaryMinutes Calories   Weekday
    ## 1                  328              728     1985   Tuesday
    ## 2                  217              776     1797 Wednesday
    ## 3                  181             1218     1776  Thursday
    ## 4                  209              726     1745    Friday
    ## 5                  221              773     1863  Saturday
    ## 6                  164              539     1728    Sunday
##### *daily_activity and sleep_day together and order the data by Monday to Sunday*
```r

```

``` r
head(hourly_steps)
```
    ##           Id          ActivityHour StepTotal
    ## 1 1503960366 4/12/2016 12:00:00 AM       373
    ## 2 1503960366  4/12/2016 1:00:00 AM       160
    ## 3 1503960366  4/12/2016 2:00:00 AM       151
    ## 4 1503960366  4/12/2016 3:00:00 AM         0
    ## 5 1503960366  4/12/2016 4:00:00 AM         0
    ## 6 1503960366  4/12/2016 5:00:00 AM         0
##### *Seperating the time from the date*    
```r
# splitting ActivityHour into Date and Hour 
hourly_steps <- hourly_steps %>% separate(ActivityHour, c("Date", "Hour"), sep = "^\\S*\\K")

# Check the result
head(hourly_steps)
```
    ##           Id      Date         Hour StepTotal
    ## 1 1503960366 4/12/2016  12:00:00 AM       373
    ## 2 1503960366 4/12/2016   1:00:00 AM       160
    ## 3 1503960366 4/12/2016   2:00:00 AM       151
    ## 4 1503960366 4/12/2016   3:00:00 AM         0
    ## 5 1503960366 4/12/2016   4:00:00 AM         0
    ## 6 1503960366 4/12/2016   5:00:00 AM         0




