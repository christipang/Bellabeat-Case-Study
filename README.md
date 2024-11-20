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
### Load Packages
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
### Load CSV files
``` r
setwd("/cloud/project")

sleep_day <- read.csv("sleepDay_merged.csv")
daily_activity <- read.csv("dailyActivity_merged.csv")
weight <- read.csv("weightLogInfo_merged.csv")
hourly_steps <- read.csv("hourlySteps_merged.csv")
hourly_calories <- read.csv("hourlyCalories_merged.csv")
```
### Count the Distinct IDs in Each Dataset to Determine the Number of Participants
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

```r
n_distinct(hourly_calories$Id)
```
    ## [1] 33
*hourly_calories has 33 user data*

### View, Clean, and Merge Data Set
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
##### *The 12:00:00 AM timestamp is unnecessary and will be removed to simplify the data. The SleepDay column will also be renamed to Date for consistency, and a Weekday column will be added to indicate the day of the week.*
```r
# Convert SleepDay to Date format (removes time part) 
sleep_day$SleepDay <- as.Date(sleep_day$SleepDay, format="%m/%d/%Y")

# Format the SleepDay to "MM/DD/YYYY" format
sleep_day$SleepDay <- format(sleep_day$SleepDay, "%m/%d/%Y")

# Rename the column 'SleepDay' to 'Date'
colnames(sleep_day)[colnames(sleep_day) == "SleepDay"] <- "Date"

# Add a Weekday column
sleep_day$Weekday <- weekdays(as.Date(sleep_day$Date, format = "%m/%d/%Y"))

# Check the result
head(sleep_day)
```
    ##           Id       Date TotalSleepRecords TotalMinutesAsleep TotalTimeInBed   Weekday
    ## 1 1503960366 04/12/2016                 1                327            346   Tuesday
    ## 2 1503960366 04/13/2016                 2                384            407 Wednesday
    ## 3 1503960366 04/15/2016                 1                412            442    Friday
    ## 4 1503960366 04/16/2016                 2                340            367  Saturday
    ## 5 1503960366 04/17/2016                 1                700            712    Sunday
    ## 6 1503960366 04/19/2016                 1                304            320   Tuesday

```r
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
##### *Seperating the time from the date and adding Weekday Column*    
```r
# splitting ActivityHour into Date and Hour 
hourly_steps <- hourly_steps %>% separate(ActivityHour, c("Date", "Hour"), sep = "^\\S*\\K")

# Convert the 'Date' column to Date format
hourly_steps$Date <- mdy(hourly_steps$Date)  

# Add the Weekday column based on the Date column
hourly_steps$Weekday <- weekdays(hourly_steps$Date)

# Check the result
head(hourly_steps)
```
    ##           Id       Date         Hour StepTotal Weekday
    ## 1 1503960366 2016-04-12  12:00:00 AM       373 Tuesday
    ## 2 1503960366 2016-04-12   1:00:00 AM       160 Tuesday
    ## 3 1503960366 2016-04-12   2:00:00 AM       151 Tuesday
    ## 4 1503960366 2016-04-12   3:00:00 AM         0 Tuesday
    ## 5 1503960366 2016-04-12   4:00:00 AM         0 Tuesday
    ## 6 1503960366 2016-04-12   5:00:00 AM         0 Tuesday

```r
head(hourly_calories)
```
    ##           Id          ActivityHour Calories
    ## 1 1503960366 4/12/2016 12:00:00 AM       81
    ## 2 1503960366  4/12/2016 1:00:00 AM       61
    ## 3 1503960366  4/12/2016 2:00:00 AM       59
    ## 4 1503960366  4/12/2016 3:00:00 AM       47
    ## 5 1503960366  4/12/2016 4:00:00 AM       48
    ## 6 1503960366  4/12/2016 5:00:00 AM       48
##### *Seperating the time from the date*   
```r
# splitting ActivityHour into Date and Hour 
hourly_calories <- hourly_calories %>% separate(ActivityHour, c("Date", "Hour"), sep = "^\\S*\\K")

# Convert the 'Date' column to Date format
hourly_calories$Date <- mdy(hourly_calories$Date)  

# Check the result
head(hourly_calories)
```
    ##           Id       Date         Hour Calories
    ## 1 1503960366 2016-04-12  12:00:00 AM       81
    ## 2 1503960366 2016-04-12   1:00:00 AM       61
    ## 3 1503960366 2016-04-12   2:00:00 AM       59
    ## 4 1503960366 2016-04-12   3:00:00 AM       47
    ## 5 1503960366 2016-04-12   4:00:00 AM       48
    ## 6 1503960366 2016-04-12   5:00:00 AM       48

Converting the Id column in the data frames to a character type to ensure consistency and avoid potential issues during data manipulation or merging.
```r
sleep_day$Id <- as.character(sleep_day$Id)
daily_activity$Id <- as.character(daily_activity$Id)
hourly_steps$Id <- as.character(hourly_steps$Id)
hourly_calories$Id <- as.character(hourly_calories$Id)
```
Count the duplicates and remove them if there are any.
```r
sum(duplicated(sleep_day))
```
    ## [1] 3 
*Sleep_day has 3 duplicates*
```r
# remove the duplicate
sleep_day <- sleep_day[!duplicated(sleep_day), ]

# Count the number of duplicate to ensure there are none
sum(duplicated(sleep_day))
```
```r
sum(duplicated(daily_activity))
```
    ## [1] 0
*daily_activity has 0 duplicates*
```r
sum(duplicated(hourly_steps))    
```
    ## [1] 0
*hourly_steps has 0 duplicates*

```r
sum(duplicated(hourly_calories))
```
    ## [1] 0
*hourly_calories has 0 duplicates*

## ANALYZE

### What Days are People Most Active

```r
# Calculate total TotalSteps by Weekday
total_steps_by_weekday <- daily_activity %>%
  group_by(Weekday) %>%
  summarise(TotalSteps = sum(TotalSteps))

# Reorder the Weekday column to ensure the days are in the correct order
total_steps_by_weekday$Weekday <- factor(total_steps_by_weekday$Weekday, 
                                 levels = c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"))

# Create the bar chart
ggplot(total_steps_by_weekday, aes(x = Weekday, y = TotalSteps, fill = Weekday)) +
  geom_bar(stat = "identity") +  # Bar chart
  geom_text(aes(label = round(TotalSteps, 1)), vjust = -0.3) +  # Label on top of bars
  ylab("Total Steps") +
  ggtitle("Total Steps by Weekday") 

```
![image](https://github.com/user-attachments/assets/b5de8237-d8b0-441b-b1ea-431e810b0099)

The graph indicates that activity levels are highest on Tuesday, followed by Wednesday and Thursday.

### Average Total Steps Per Day
```r
# Calculate average TotalSteps by Weekday
average_steps_by_weekday <- daily_activity %>%
  group_by(Weekday) %>%
  summarise(AverageSteps = mean(TotalSteps))

# Create the bar chart
ggplot(average_steps_by_weekday, aes(x = Weekday, y = AverageSteps, fill = Weekday)) +
  geom_bar(stat = "identity") +  # Bar chart
  geom_text(aes(label = round(AverageSteps, 1)), vjust = -0.3) +  # Label on top of bars
  ylab("Average Total Steps") +
  ggtitle("Average Total Steps by Weekday") 
````
![image](https://github.com/user-attachments/assets/9471e2a1-ca1f-46f4-84d1-33f836bbab65)

On average, the highest number of steps are taken on Saturday, followed by Tuesday and Monday.

### At what hours are the users most active

```r
# Load the scales package
library(scales)

# Remove any leading/trailing spaces
hourly_steps$Hour <- trimws(hourly_steps$Hour)

# Reorder the Hour column to ensure the times are in the correct order
hourly_steps$Hour <- factor(hourly_steps$Hour, 
                                 levels = c("12:00:00 AM", "1:00:00 AM", "2:00:00 AM", "3:00:00 AM", 
                "4:00:00 AM", "5:00:00 AM", "6:00:00 AM", "7:00:00 AM", 
                "8:00:00 AM", "9:00:00 AM", "10:00:00 AM", "11:00:00 AM", 
                "12:00:00 PM", "1:00:00 PM", "2:00:00 PM", "3:00:00 PM", 
                "4:00:00 PM", "5:00:00 PM", "6:00:00 PM", "7:00:00 PM", 
                "8:00:00 PM", "9:00:00 PM", "10:00:00 PM", "11:00:00 PM"))

# Plot the total steps per hour with formatted y-axis labels
ggplot(data = hourly_steps, aes(x = Hour, y = StepTotal)) +
  geom_bar(stat = "identity", fill = "purple") +  
  labs(
    title = "Total Steps by Hour",
    x = "Hour of the Day",
    y = "Total Steps"
  ) +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 90, hjust = 1)) +  
  scale_y_continuous(labels = label_comma())  
```
![image](https://github.com/user-attachments/assets/b47ecd28-99ed-4c11-ad37-85e5fdcbfef9)

The graph indicates that users tend to be most active between 5 PM and 7 PM, as well as between 12 PM and 2 PM.

### At what hours do users burn the most calories
```r
# Load the scales package
library(scales)

# Remove any leading/trailing spaces
hourly_calories$Hour <- trimws(hourly_calories$Hour)

# Reorder the Hour column to ensure the times are in the correct order
hourly_calories$Hour <- factor(hourly_calories$Hour, 
                                 levels = c("12:00:00 AM", "1:00:00 AM", "2:00:00 AM", "3:00:00 AM", 
                "4:00:00 AM", "5:00:00 AM", "6:00:00 AM", "7:00:00 AM", 
                "8:00:00 AM", "9:00:00 AM", "10:00:00 AM", "11:00:00 AM", 
                "12:00:00 PM", "1:00:00 PM", "2:00:00 PM", "3:00:00 PM", 
                "4:00:00 PM", "5:00:00 PM", "6:00:00 PM", "7:00:00 PM", 
                "8:00:00 PM", "9:00:00 PM", "10:00:00 PM", "11:00:00 PM"))

# Plot the calories per hour with formatted y-axis labels
ggplot(data = hourly_calories, aes(x = Hour, y = Calories)) +
  geom_bar(stat = "identity", fill = "purple") +  
  labs(
    title = "Calories burn by Hour",
    x = "Hour of the Day",
    y = "Calories"
  ) +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 90, hjust = 1)) +  
  scale_y_continuous(labels = label_comma())  
```

![image](https://github.com/user-attachments/assets/0cf4a9f8-6dfa-474f-9688-711e69c7cb6c)

The graph indicates that users tend to burn the most calories between 5 PM and 7 PM, as well as between 12 PM and 2 PM.
It matches with the Total Steps by Hour graph.

### Exploring the Relationship Between Calories, Total Steps, and Total Distance
```r
# Create the scatter plot with a regression line
ggplot(data = daily_activity, aes(x = TotalSteps, y = Calories)) +
  geom_point(color = "pink", alpha = 0.6) +  # Scatter plot with purple points
  geom_smooth(method = "lm", se = FALSE, color = "black") +  # Add linear regression line
  labs(
    title = "Total Steps and Calories",
    x = "Total Steps",
    y = "Calories"
  ) +
  theme_minimal()
```
![image](https://github.com/user-attachments/assets/d73d10a4-5c0b-47b6-b194-d88240c70f9c)

```r
ggplot(data = daily_activity, aes(x = TotalDistance, y = Calories)) +
  geom_point(color = "pink", alpha = 0.6) +  # Scatter plot with purple points
  geom_smooth(method = "lm", se = FALSE, color = "black") +  # Add linear regression line
  labs(
    title = "Total Distance and Calories",
    x = "Total Distance",
    y = "Calories"
  ) +
  theme_minimal()
```
![image](https://github.com/user-attachments/assets/693f640d-a131-4fe7-b087-8ef9cfa0793e)

The two graphs show a positive correlation. As the number of steps and the distance traveled increase, calorie burn also rises.

### Average Calories Per Day 
```r
# Calculate average Calories by Weekday
average_calories_by_weekday <- daily_activity %>%
  group_by(Weekday) %>%
  summarise(AverageCalories = mean(Calories))

# Create the bar chart
ggplot(average_calories_by_weekday, aes(x = Weekday, y = AverageCalories, fill = Weekday)) +
  geom_bar(stat = "identity") +  # Bar chart
  geom_text(aes(label = round(AverageCalories, 1)), vjust = -0.3) +  # Label on top of bars
  ylab("Average Calories") +
  ggtitle("Average Calories by Weekday")
```
![image](https://github.com/user-attachments/assets/365d2388-bdcf-442d-9f87-a326aff031f5)

The graph shows that users burn an average of 2,100 to 2,300 calories per day. The WHO recommends that adults consume approximately 2,000 calories daily.

### What Days do Users Sleep the Most
```r
# Calculate total TotalMinutesAsleep by Weekday
total_sleep_by_weekday <- sleep_day %>%
  group_by(Weekday) %>%
  summarise(TotalMinutesAsleep = sum(TotalMinutesAsleep))

# Reorder the Weekday column to ensure the days are in the correct order
total_sleep_by_weekday$Weekday <- factor(total_sleep_by_weekday$Weekday, 
                                 levels = c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"))

# Create the bar chart
ggplot(total_sleep_by_weekday, aes(x = Weekday, y = TotalMinutesAsleep, fill = Weekday)) +
  geom_bar(stat = "identity") +  # Bar chart
  geom_text(aes(label = round(TotalMinutesAsleep, 1)), vjust = -0.3) +  # Label on top of bars
  ylab("Total Minutes Asleep") +
  ggtitle("Total Minutes Asleep by Weekday") 
```
![image](https://github.com/user-attachments/assets/242cbeb4-b198-4cbb-8ea5-1f2837ee4123)

Users sleep the most on Wednesday then Tuesday and Thursday. 

### On Average How Many Hours Do Users Sleep Per Day 

```r
# Convert minutes to hours and calculate the average hours per weekday
sleep_day_average <- sleep_day %>%
  mutate(HoursAsleep = TotalMinutesAsleep / 60) %>%
  group_by(Weekday) %>%
  summarise(AverageHoursAsleep = mean(HoursAsleep))

# Reorder the Weekday column to ensure the days are in the correct order
sleep_day_average$Weekday <- factor(sleep_day_average$Weekday, 
                                 levels = c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"))

# Create the bar chart
ggplot(sleep_day_average, aes(x = Weekday, y = AverageHoursAsleep, fill = Weekday)) +
  geom_bar(stat = "identity") +  # Bar chart
  geom_text(aes(label = round(AverageHoursAsleep, 1)), vjust = -0.3) +  # Add labels
  ylab("Average Hours Asleep") +
  ggtitle("Average Hours Asleep by Weekday") 
```
![image](https://github.com/user-attachments/assets/076b8ee1-ec11-4abb-acb4-7a8c56cdf966)

On average users sleep between 6.7 hours to 7.5 hrs. Users sleep more on Sunday then Wednesday.

### The Average Amount of Sleep User Gets
```r
# Calculate the average hours asleep per Id
average_sleep_per_id <- sleep_day %>%
  group_by(Id) %>%
  summarise(AverageHoursAsleep = mean(TotalMinutesAsleep) / 60)  # Convert minutes to hours

# Create the bar chart
ggplot(average_sleep_per_id, aes(x = Id, y = AverageHoursAsleep, fill = Id)) +
  geom_bar(stat = "identity",) +  # Bar chart
  geom_text(aes(label = round(AverageHoursAsleep, 1)), vjust = -0.3, size = 3) +  # Add labels
  ylab("Average Hours Asleep") +
  ggtitle("Average Hours Asleep Per User") +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 90, hjust = 1))  # Rotate x-axis labels
```
![image](https://github.com/user-attachments/assets/69466790-c062-40e7-8c7e-6e4a9fb061ee)

There are three users who only got on average 1, 1.1, and 2.1 hours of sleep. There is one user who gets 10 hours of sleep. 
```r
# Count the number of users getting less than 7 hours of sleep
average_sleep_per_id %>%
filter(AverageHoursAsleep < 7) %>%
summarise(Count = n())
```
    ##     13
*There are 13 users who are getting less than 7 hours of sleep on average*

```r
# Count the number of users getting between 7 to 9 hours of sleep
average_sleep_per_id %>%
filter(AverageHoursAsleep >= 7 & AverageHoursAsleep <= 9) %>%
summarise(Count = n())
```
    ##     10
*There are 10 users who are getting between 7 to 9 hours of sleep*

```r
# Count the number of users getting more than 9 hours of sleep
average_sleep_per_id %>%
filter(AverageHoursAsleep > 9) %>%
summarise(Count = n())
```
    ##     1
*There is 1 user who is getting more than 9 hours of sleep on average*

NIH reccomends adults should get between 7 to 9 hrs of sleep daily. Only 10 users are getting the apporiate amount of sleep.

### The Percentage Of Users Getting Different Hours of Sleep
```r
# Create categories for sleep hours
sleep_categories <- average_sleep_per_id %>%
  mutate(SleepCategory = case_when(
    AverageHoursAsleep < 7 ~ "Less than 7 hours",
    AverageHoursAsleep >= 7 & AverageHoursAsleep <= 9 ~ "7 to 9 hours",
    AverageHoursAsleep > 9 ~ "More than 9 hours"
  ))

# Summarize the counts by sleep category
sleep_summary <- sleep_categories %>%
  group_by(SleepCategory) %>%
  summarise(Count = n()) %>%
  mutate(Percentage = (Count / sum(Count)) * 100)

# Create the pie chart
ggplot(sleep_summary, aes(x = "", y = Percentage, fill = SleepCategory)) +
  geom_bar(stat = "identity", width = 1) +
  coord_polar(theta = "y") +  # Create the pie chart
  labs(title = "Percentage of Users Getting Different Hours of Sleep") +
  theme_void() +  # Remove axes and grid lines
  geom_text(aes(label = paste0(round(Percentage, 1), "%")), position = position_stack(vjust = 0.5))  # Add percentages as labels
```
![image](https://github.com/user-attachments/assets/7ae4f52c-843a-4fd2-80a8-e7b8dcd343ee)

41.7% of users are getting the recommended amount of sleep, while 54.2% are getting less than the recommended hours, which is more than half of the users.

### Summary of TotalSteps, TotalDistance, Calories, and TotalMinutesAsleep
```r
daily_activity %>%
  dplyr::select(Weekday,
         TotalSteps,
         TotalDistance,         
         Calories) %>%
  summary()

sleep_day %>%
  dplyr::select(TotalMinutesAsleep) %>%
  summary()   
```
    ##      Weekday      TotalSteps    TotalDistance       Calories   
    ##   Monday   :120   Min.   :    0   Min.   : 0.000   Min.   :   0  
    ##   Tuesday  :152   1st Qu.: 3790   1st Qu.: 2.620   1st Qu.:1828  
    ##   Wednesday:150   Median : 7406   Median : 5.245   Median :2134  
    ##   Thursday :147   Mean   : 7638   Mean   : 5.490   Mean   :2304  
    ##   Friday   :126   3rd Qu.:10727   3rd Qu.: 7.713   3rd Qu.:2793  
    ##   Saturday :124   Max.   :36019   Max.   :28.030   Max.   :4900  
    ##   Sunday   :121

    ##  TotalMinutesAsleep
    ##  Min.   : 58.0     
    ##  1st Qu.:361.0     
    ##  Median :432.5     
    ##  Mean   :419.2     
    ##  3rd Qu.:490.0     
    ##  Max.   :796.0   

### Summary of Very Active Minutes, Fairly Active Minutes, Lightly Active Minutes, and Sedentary Minutes

```r
daily_activity %>%
  dplyr::select(
         VeryActiveMinutes,
         FairlyActiveMinutes,
         LightlyActiveMinutes,
         SedentaryMinutes,         
         ) %>%
  summary()
```
    ##  VeryActiveMinutes FairlyActiveMinutes LightlyActiveMinutes SedentaryMinutes
    ##  Min.   :  0.00    Min.   :  0.00      Min.   :  0.0        Min.   :   0.0  
    ##  1st Qu.:  0.00    1st Qu.:  0.00      1st Qu.:127.0        1st Qu.: 729.8  
    ##  Median :  4.00    Median :  6.00      Median :199.0        Median :1057.5  
    ##  Mean   : 21.16    Mean   : 13.56      Mean   :192.8        Mean   : 991.2  
    ##  3rd Qu.: 32.00    3rd Qu.: 19.00      3rd Qu.:264.0        3rd Qu.:1229.5  
    ##  Max.   :210.00    Max.   :143.00      Max.   :518.0        Max.   :1440.0  

    
