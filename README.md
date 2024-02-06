# bellabeat-casestudy
This repo analyzes smart device fitness data to reveal growth opportunities for Bellabeat, a wellness tech firm. We aim to identify device usage trends and align them with Bellabeat's marketing strategies, enhancing customer engagement.

# BELLABEAT: 
## Case Study 2: How Can a Wellness Technology Company Play It Smart?

Considering this is a pilot project, I was inspired and followed the guidance of Ahmad Nawaz on LinkedIn. However, I went on and used SQL Server instead of MySQL, as well as R programming to conduct the processing and analysis.

### Statement of the Business Task:
Analyze smart device fitness data and unlock new growth opportunities for the company (Bellabeat). Identify trends and investigate how these trends could apply to Bellabeat customers. Give recommendations based on findings to improve marketing strategy.

### Ask
1. What are some trends in smart device usage?
2. How could these trends apply to Bellabeat customers?
3. How could these trends help influence Bellabeat marketing strategy?

### Prepare

The dataset under review comprises a 31-day activity log from 30 distinct users, sourced from FitBit, a prominent American company specializing in health and fitness products. The data is segregated across 18 Excel files, collated from an open-source survey and available in CSV format. Notably, while the majority of these files record the activities of the full cohort, a few contain data for a subset of 24 or even as few as 7 users, presenting potential outliers within the dataset.
A preliminary assessment of the data has revealed a lack of demographic information for the participants, which may introduce an element of bias. Without details such as age, gender, geographic location, or physical characteristics, the ability to generalize the findings or apply them to broader populations is limited.
For the purposes of this analysis, a focused approach has been adopted, with the selection of four key tables: daily activity, sleep patterns, hourly steps, and hourly calories. These tables were chosen with the intent to construct a comprehensive picture of the users' physical activity, sleep quality, and caloric expenditure, all of which are pivotal metrics for understanding lifestyle impacts on health.
The analysis will employ rigorous data visualization techniques, primarily utilizing the ggplot2 package within the R programming environment, to elucidate trends and behaviors within the user data. This will be complemented by statistical analysis to explore correlations and other relationships within the dataset.
As we proceed, it is imperative to maintain a critical view of the insights derived from this data, given the noted potential for bias and the specific subset of data files chosen for this examination.

## Process
This section outlines the SQL Server code used in the analysis, focusing on identifying unique users, checking for duplicates, and cleaning the data.

### Unique Users
Queries to check the number of unique users in different tables:

```sql
-- Checking number of Users in dailyactivity Table
SELECT count(DISTINCT(Id)) FROM dailyactivity;

-- Checking number of Users in sleepday Table
SELECT count(DISTINCT(Id)) FROM sleepday;

-- Checking number of Users in hourlysteps Table
SELECT count(DISTINCT(Id)) FROM hourlysteps;

-- Checking number of Users in hourlycalories Table
SELECT count(distinct(id)) FROM hourlycalories;

-- Checking Duplicates in dailyactivity Table
SELECT Id, ActivityDate, TotalSteps, Count(*) FROM dailyactivity
GROUP BY id, ActivityDate, TotalSteps
HAVING Count(*) > 1;

-- Checking Duplicates in sleepday Table
SELECT Id, SleepDay, TotalSleepRecords, Count(*) AS duplicates FROM sleepday
GROUP BY id, SleepDay, TotalSleepRecords
HAVING Count(*) > 1;

-- Checking Duplicates in hourlysteps Table
SELECT Id, ActivityHour, StepTotal, count(*) FROM hourlysteps
GROUP BY Id, ActivityHour, StepTotal
HAVING Count(*) > 1;

-- Checking Duplicates in hourlycalories Table
SELECT Id, ActivityHour, Calories, count(*) FROM hourlycalories
GROUP BY Id, ActivityHour, Calories
Having Count(*) > 1;

-- Copying distinct values into a new table for sleepday
SELECT DISTINCT * INTO sleepdayv2 FROM sleepday;

-- Drop old sleepday table
USE master;
DROP TABLE sleepday;

-- Rename sleepdayv2 to sleepday as the updated table
EXEC sp_rename 'Bellabeat2.dbo.sleepdayv2', 'sleepday';


### Adding Day of Week Column
To enhance our analysis, we added a 'Day_of_Week' column to our tables to categorize data by the day of the week. Here's how:

```sql
-- Step 1: Add a new 'Day_of_Week' column to the dailyactivity table
ALTER TABLE dailyactivity ADD Day_of_Week VARCHAR(10);

-- Step 2: Populate the 'Day_of_Week' column with day names
UPDATE dailyactivity SET Day_of_Week = DATENAME(WEEKDAY, ActivityDate);

-- Repeat the process for the sleepday table
ALTER TABLE Bellabeat2.dbo.sleepday ADD Day_of_Week char(10);
UPDATE sleepday SET Day_of_Week = DATENAME(WEEKDAY, SleepDay);

-- Verify the changes
SELECT * FROM dailyactivity;
SELECT * FROM sleepday;
```

### Creating Daily_Activity_Sleep Table
To consolidate our data for in-depth analysis, we created a new table that joins daily activity with sleep data:

```sql
-- Creating a new table to combine daily activity and sleep data
CREATE TABLE Daily_Activity_Sleep AS
SELECT t1.*, t2.totalsleeprecords, t2.totalminutesasleep, t2.totaltimeinbed
INTO Daily_Activity_Sleep
FROM dailyactivity t1
INNER JOIN sleepdaytable t2 ON t1.id = t2.id AND t1.ActivityDate = t2.SleepDay;

-- Review the newly created table
SELECT * FROM Daily_Activity_Sleep;
```

### Updating and Reviewing Data
Final steps involved updating the 'ActivityDate' format for consistency and reviewing the prepared tables:

```sql
-- Update 'ActivityDate' to DATE format for uniformity
UPDATE Daily_Activity_Sleep SET ActivityDate = CAST(ActivityDate AS DATE);
UPDATE Daily_Activity_Sleep SET ActivityDate = CONVERT(DATE, ActivityDate);

-- Review the updates
SELECT * FROM Daily_Activity_Sleep;
SELECT * FROM Bellabeat2.dbo.Daily_Activity_Sleep;
SELECT * FROM Bellabeat2.dbo.sleepday;
SELECT * FROM Bellabeat2.dbo.Daily_Use;
```


Continuing with the structured approach for your GitHub README.md, here's how you can include the analysis phase using SQL queries to analyze total data by users, daily averages, and classify users based on their activity levels:


## Analyze
This section delves into the analysis of the consolidated Daily_Activity_Sleep data, focusing on total and average metrics, and classifying users by activity levels.

### Total Data by Users and Day of the Week
Analysis of total distance, steps, sleep, and calories by user ID and day of the week:

```sql
-- Total data by users
SELECT Id, 
       ROUND(SUM(TotalDistance),2) AS total_distance,
       ROUND(SUM(TotalSteps),2) AS total_steps,
       ROUND(SUM(TotalMinutesAsleep),2) AS total_minutes_sleep,
       ROUND(SUM(Calories),2) AS total_calories
FROM Daily_Activity_Sleep
GROUP BY Id;

-- Total data by day of the week
SELECT Day_of_Week, 
       ROUND(SUM(TotalDistance),2) AS total_distance,
       ROUND(SUM(TotalSteps),2) AS total_steps,
       ROUND(SUM(TotalMinutesAsleep),2) AS total_minutes_sleep,
       ROUND(SUM(Calories),2) AS total_calories
FROM Daily_Activity_Sleep
GROUP BY Day_of_Week
ORDER BY CASE Day_of_Week
         WHEN 'Monday' THEN 1
         WHEN 'Tuesday' THEN 2
         WHEN 'Wednesday' THEN 3
         WHEN 'Thursday' THEN 4
         WHEN 'Friday' THEN 5
         WHEN 'Saturday' THEN 6
         WHEN 'Sunday' THEN 7
         END;
```

### Averages by Users and Day of the Week
Calculating average distance, steps, sleep, and calories:

```sql
-- Average data by users
SELECT Id, 
       ROUND(AVG(TotalDistance),2) AS avg_distance,
       ROUND(AVG(TotalSteps),2) AS avg_steps,
       ROUND(AVG(TotalMinutesAsleep),2) AS avg_sleep,
       ROUND(AVG(Calories),2) AS avg_calories
FROM Daily_Activity_Sleep
GROUP BY Id;

-- Average data by day of the week
SELECT Day_of_Week,
       ROUND(AVG(TotalDistance),2) AS avg_distance,
       ROUND(AVG(TotalSteps),2) AS avg_steps,
       ROUND(AVG(TotalMinutesAsleep),2) AS avg_sleep,
       ROUND(AVG(Calories),2) AS avg_calories
FROM Daily_Activity_Sleep
GROUP BY Day_of_Week
ORDER BY CASE Day_of_Week
         WHEN 'Monday' THEN 1
         WHEN 'Tuesday' THEN 2
         WHEN 'Wednesday' THEN 3
         WHEN 'Thursday' THEN 4
         WHEN 'Friday' THEN 5
         WHEN 'Saturday' THEN 6
         WHEN 'Sunday' THEN 7
         END;
```

### Classifying Users by Average Daily Steps
Creating a new table for average metrics and classifying users based on their activity:

```sql
-- Creating a new table for averages
SELECT Id, 
       ROUND(AVG(TotalDistance), 2) AS avg_distance,
       ROUND(AVG(TotalSteps), 2) AS avg_steps,
       ROUND(AVG(TotalMinutesAsleep), 2) AS avg_sleep,
       ROUND(AVG(Calories), 2) AS avg_calories
INTO average_by_users
FROM Daily_Activity_Sleep
GROUP BY Id;

-- Renaming column for clarity
EXEC sp_rename 'average_by_users.avg_steps', 'avg_daily_steps', 'COLUMN';

-- Classifying users based on average daily steps
SELECT Id, 
       avg_daily_steps,
       CASE
           WHEN avg_daily_steps < 5000 THEN 'Sedentary'
           WHEN avg_daily_steps >= 5000 AND avg_daily_steps < 7499 THEN 'Lightly Active'
           WHEN avg_daily_steps >= 7500 AND avg_daily_steps < 9999 THEN 'Fairly Active'
           WHEN avg_daily_steps >= 10000 THEN 'Very Active'
       END AS User_Type
INTO user_classification_by_avg_daily_steps
FROM average_by_users;

-- Determining the percentage of user types
SELECT User_Type,
       COUNT(Id) AS total_users,
       CAST(COUNT(Id) * 100.0 / 24 AS DECIMAL(5, 1)) AS user_percentage
FROM user_classification_by_avg_daily_steps
GROUP BY User_Type
ORDER BY user_percentage DESC;
```

Adding to the structured GitHub README.md documentation, here's how to include the analysis regarding average hourly steps throughout the day, calculating average calories, user engagement in days, and classifying users based on usage in a clear and formatted way:

```markdown
## Calculating Average Hourly Steps Throughout the Day
To gain insights into hourly activity patterns, we've calculated the average number of steps taken by users throughout the day.

```sql
-- Adding Time of the Day column to hourlysteps table
ALTER TABLE hourlysteps ADD Time_of_Day TIME;
UPDATE hourlysteps SET Time_of_Day = CAST(ActivityHour AS TIME);

-- Adding Date column for further analysis
ALTER TABLE hourlysteps ADD Date_Column DATE;
UPDATE hourlysteps SET Date_Column = CAST(ActivityHour AS DATE);

-- Calculating average steps by time of day
SELECT Time_of_Day, ROUND(AVG(StepTotal), 2) AS Average_Steps FROM hourlysteps GROUP BY Time_of_Day ORDER BY Average_Steps DESC;

-- Formatting the time for clarity
SELECT DATEPART(HOUR, Time_of_Day) AS Time_of_the_Day, ROUND(AVG(StepTotal), 2) AS Average_Steps FROM hourlysteps GROUP BY DATEPART(HOUR, Time_of_Day) ORDER BY Average_Steps DESC;
```

## Calculating Average Calories Throughout the Day
Understanding energy expenditure patterns is crucial for wellness insights.

```sql
-- Preparing the hourlycalories table for analysis
ALTER TABLE hourlycalories ADD Time_of_Day TIME;
UPDATE hourlycalories SET Time_of_Day = CAST(ActivityHour AS TIME);

-- Analyzing average calorie burn by hour
SELECT DATEPART(HOUR, Time_of_Day) AS Time_of_the_Day, ROUND(AVG(Calories), 2) AS Calories FROM hourlycalories GROUP BY DATEPART(HOUR, Time_of_Day) ORDER BY Calories DESC;
```

## Calculating Usage in Days by Users
Evaluating user engagement over the dataset's timeframe provides insight into device utilization.

```sql
-- Identifying total usage days for each user
SELECT id, COUNT(id) AS Total_Usage_in_Days INTO frequency_of_usage FROM daily_activity_sleep GROUP BY id;
SELECT * FROM frequency_of_usage ORDER BY Total_Usage_in_Days DESC;

-- Renaming table for consistency
EXEC sp_rename 'Frequency_of_Usage', 'frequency_of_usage';

-- Classifying users by app usage frequency
SELECT id, Total_Usage_in_Days, CASE WHEN Total_Usage_in_Days >= 1 AND Total_Usage_in_Days <= 10 THEN 'Low Use' WHEN Total_Usage_in_Days >= 11 AND Total_Usage_in_Days <= 20 THEN 'Moderate Use' WHEN Total_Usage_in_Days > 20 THEN 'High Use' END AS Type_of_Usage INTO user_classification_by_usage_in_days FROM frequency_of_usage;
SELECT * FROM user_classification_by_usage_in_days ORDER BY Total_Usage_in_Days DESC;

-- Determining percentage of users by type of usage
SELECT Type_of_Usage, ROUND(COUNT(id)*100.0/24, 2) AS Percentage FROM user_classification_by_usage_in_days GROUP BY Type_of_Usage;
```

## Activity Time Analysis
A deep dive into how users spend their time on various activities throughout a typical day.

```sql
-- Analyzing total time spent on each activity type per day
SELECT Day_of_Week, SUM(SedentaryMinutes) AS Sedentary_Mins, SUM(LightlyActiveMinutes) AS LightlyActive_Mins, SUM(FairlyActiveMinutes) AS FairlyActive_Mins, SUM(VeryActiveMinutes) AS VeryActive_Mins FROM dailyactivity GROUP BY Day_of_Week ORDER BY CASE Day_of_Week WHEN 'Monday' THEN 1 WHEN 'Tuesday' THEN 2 ELSE Day_of_Week END;

-- Calculating the percentage of total time spent on each activity
SELECT Day_of_Week, CONCAT(ROUND(SUM(VeryActiveMinutes) / SUM(VeryActiveMinutes + FairlyActiveMinutes + LightlyActiveMinutes + SedentaryMinutes) * 100, 2), '%') AS VAM FROM dailyactivity GROUP BY Day_of_Week;
```

To include the additional analysis focusing on the percentage of total time spent on each activity per day, user classification by day, and the calculation of wear time percentage in your GitHub README.md using SQL Server code snippets, here's how you can format it with Markdown for clear documentation:

```markdown
## Advanced Activity Analysis

This section covers advanced analysis techniques, including calculating the percentage of total time spent on various activities throughout the day and classifying users based on their daily activity patterns and device usage.

### Calculating Percentage of Time Spent on Each Activity

```sql
-- Calculating percentage of total time spent on each activity per day
SELECT Day_of_Week,
       CONCAT(CAST(ROUND(CAST(SUM(VeryActiveMinutes) AS DECIMAL) / SUM(VeryActiveMinutes + FairlyActiveMinutes + LightlyActiveMinutes + SedentaryMinutes) * 100, 2) AS DECIMAL(10,2)), '%') AS VAM,
       CONCAT(CAST(ROUND(CAST(SUM(FairlyActiveMinutes) AS DECIMAL) / SUM(VeryActiveMinutes + FairlyActiveMinutes + LightlyActiveMinutes + SedentaryMinutes) * 100, 2) AS DECIMAL(10,2)), '%') AS FAM,
       CONCAT(CAST(ROUND(CAST(SUM(LightlyActiveMinutes) AS DECIMAL) / SUM(VeryActiveMinutes + FairlyActiveMinutes + LightlyActiveMinutes + SedentaryMinutes) * 100, 2) AS DECIMAL(10,2)), '%') AS LAM,
       CONCAT(CAST(ROUND(CAST(SUM(SedentaryMinutes) AS DECIMAL) / SUM(VeryActiveMinutes + FairlyActiveMinutes + LightlyActiveMinutes + SedentaryMinutes) * 100, 2) AS DECIMAL(10,2)), '%') AS SM
FROM dailyactivity
GROUP BY Day_of_Week;

-- Analysis by user ID to understand individual patterns
SELECT id, Day_of_Week,
       CONCAT(CAST(ROUND(CAST(SUM(VeryActiveMinutes) AS DECIMAL) / NULLIF(SUM(VeryActiveMinutes + FairlyActiveMinutes + LightlyActiveMinutes + SedentaryMinutes), 0) * 100, 2) AS DECIMAL(10,2)), '%') AS VAM,
       CONCAT(CAST(ROUND(CAST(SUM(FairlyActiveMinutes) AS DECIMAL) / NULLIF(SUM(VeryActiveMinutes + FairlyActiveMinutes + LightlyActiveMinutes + SedentaryMinutes), 0) * 100, 2) AS DECIMAL(10,2)), '%') AS FAM,
       CONCAT(CAST(ROUND(CAST(SUM(LightlyActiveMinutes) AS DECIMAL) / NULLIF(SUM(VeryActiveMinutes + FairlyActiveMinutes + LightlyActiveMinutes + SedentaryMinutes), 0) * 100, 2) AS DECIMAL(10,2)), '%') AS LAM,
       CONCAT(CAST(ROUND(CAST(SUM(SedentaryMinutes) AS DECIMAL) / NULLIF(SUM(VeryActiveMinutes + FairlyActiveMinutes + LightlyActiveMinutes + SedentaryMinutes), 0) * 100, 2) AS DECIMAL(10,2)), '%') AS SM
FROM dailyactivity
GROUP BY id, Day_of_Week
ORDER BY Id;
```

### Device Wear Time Analysis

```sql
-- Creating a table to analyze how users wear their devices throughout the day
SELECT t1.*, t2.Total_Usage_in_Days, t2.Type_of_Usage INTO Daily_Use FROM dailyactivity t1 INNER JOIN user_classification_by_usage_in_days t2 ON t1.id = t2.id;

-- Adding and updating columns to classify wear time
ALTER TABLE Daily_Use ADD Total_Minutes_Worn bigint;
UPDATE Daily_Use SET Total_Minutes_Worn = VeryActiveMinutes + FairlyActiveMinutes + LightlyActiveMinutes + SedentaryMinutes;

ALTER TABLE Daily_Use ADD Worn_Type VARCHAR(50);
UPDATE Daily_Use SET Worn_Type = CASE WHEN Total_Minutes_Worn IS NULL THEN NULL WHEN Total_Minutes_Worn = 1440 THEN 'ALL Day' WHEN Total_Minutes_Worn >= 720 THEN 'More Than Half Day' WHEN Total_Minutes_Worn > 0 THEN 'Less Than Half Day' END;

-- Calculating wear type percentage
SELECT Worn_Type, COUNT(Id) AS Total_Users, CONCAT(CAST(ROUND(COUNT(Id) * 100.0 / 713, 2) AS decimal(10, 2)), '%') AS Worn_Type_Percentage FROM Daily_Use GROUP BY Worn_Type;
```

These advanced analytical steps enable a deeper understanding of user engagement and activity patterns, offering valuable insights for personalized health and wellness strategies.
```

## Correlation Analysis

Exploring correlations within our dataset to understand relationships between different health metrics.

### Initial Data Review

```sql
-- Reviewing the Daily_Activity_Sleep data structure
SELECT * FROM Daily_Activity_Sleep;

-- Reviewing the sleepday data structure
SELECT * FROM sleepday;
```

### Correlation Between Daily Steps and Minutes Asleep

Investigating the relationship between the number of steps taken each day and the total minutes asleep:

```sql
-- Analyzing correlation between daily steps and minutes asleep
SELECT TotalSteps AS Steps, totalminutesasleep FROM Daily_Activity_Sleep;
```

### Correlation Between Daily Steps and Calories Burned

Assessing how daily steps correlate with the number of calories burned:

```sql
-- Exploring the relationship between steps and calories burned
SELECT TotalSteps, Calories FROM Daily_Activity_Sleep;
```

### Visualization and Export

The results from the above queries were exported into a CSV file for advanced visualization, using Tableau to create detailed charts and graphs that illustrate these correlations. This visual analysis helps in understanding the impact of physical activity on sleep quality and energy expenditure, providing valuable insights for health and wellness recommendations.

These SQL analyses and the subsequent visualizations in Tableau offer a comprehensive view of user behavior and health trends, vital for enhancing wellness technology solutions like Bellabeat.
```

```
# Using R Programming and RStudio for Processing and Analysis

**Author:** Fisnik  
**Date:** 2023-12-27  

## Upload your CSV files to R

Remember to upload your CSV files to your project from the relevant data source: [Fitbit Dataset on Kaggle](https://www.kaggle.com/arashnic/fitbit)

### Uploading CSV Files

#### Specify File Path:

```r
# Specify file path and read CSV file
dailyActivity_merged <- read.csv("/path/to/dailyActivity_merged.csv")
# Display the first few rows of the data frame
head(dailyActivity_merged)

# Specify file path and read another CSV file
sleepDay_merged <- read.csv("/path/to/sleepDay_merged.csv")
head(sleepDay_merged)
```

## Installing and loading common packages and libraries

```r
install.packages('tidyverse')
library(tidyverse)
```

## Loading your CSV files

```r
daily_activity <- read.csv("dailyActivity_merged.csv")
sleep_day <- read.csv("sleepDay_merged.csv")
hourly_steps <- read.csv("hourlySteps_merged.csv")
hourly_calories <- read.csv("hourlyCalories_merged.csv")
```

## Exploring a few key tables

```r
# Take a look at the daily_activity data
head(daily_activity)
# Identify all the columns in the daily_activity data
colnames(daily_activity)

# Take a look at the sleep_day data
head(sleep_day)
# Identify all the columns in the sleep_day data
colnames(sleep_day)
```

## Understanding some summary statistics

```r
# How many unique participants are there?
n_distinct(daily_activity$Id)
n_distinct(sleep_day$Id)

# How many observations are there?
nrow(daily_activity)
nrow(sleep_day)
```

### Summary statistics for daily activity and sleep dataframes

```r
# For the daily activity dataframe
daily_activity %>% select(TotalSteps, TotalDistance, SedentaryMinutes) %>% summary()

# For the sleep dataframe
sleep_day %>% select(TotalSleepRecords, TotalMinutesAsleep, TotalTimeInBed) %>% summary()
```

## Plotting a few explorations

```r
# Relationship between steps and sedentary minutes
ggplot(data=daily_activity, aes(x=TotalSteps, y=SedentaryMinutes)) + geom_point()

# Relationship between minutes asleep and time in bed
ggplot(data=sleep_day, aes(x=TotalMinutesAsleep, y=TotalTimeInBed)) + geom_point()
```

## Process and Analysis BB 

## Processing the Data

### Load the dplyr package

```r
library(dplyr)
```

Assuming `dailyactivity` and `sleepday` data frames are already loaded in your R session.

### Count Distinct IDs in Daily Activity

```r
distinct_count_dailyactivity <- daily_activity %>%
  summarise(count_distinct_id = n_distinct(Id))
```

### Count Distinct IDs in Sleep Day

```r
distinct_count_sleepday <- sleep_day %>%
  summarise(count_distinct_id = n_distinct(Id))
```

### Display the results

```r
distinct_count_dailyactivity
distinct_count_sleepday
```

## Distinct Values in Hourly Steps and Hourly Calories

```r
n_distinct(hourly_steps$Id)
n_distinct(hourly_calories$Id)
```

## Checking for Duplicates on Daily Activity and Sleep Day

### Daily Activity

```r
duplicates_da <- daily_activity[duplicated(daily_activity[c("Id", "ActivityDate", "TotalSteps")]), ]
head(duplicates_da)
```

### Sleep Day

```r
duplicates_sd <- sleep_day[duplicated(sleep_day[c("Id", "SleepDay", "TotalSleepRecords")]), ]
head(duplicates_sd)
```

### Count the number of duplicate rows in Sleep Day

```r
num_duplicates <- sum(duplicated(sleep_day))
num_duplicates
```

## Deleting Duplicate Records from sleepday

```r
sleep_day <- sleep_day %>% distinct()
head(sleep_day)
```

## Upload Lubridate and Dplyr for Date and Time Manipulation

```r
library(lubridate)
library(dplyr)
```

### Formatting Date and Time in Hourly Steps and Hourly Calories

#### Hourly Steps

```r
hourly_steps <- hourly_steps %>%
  mutate(
    Date = format(as.Date(mdy_hms(ActivityHour)), "%m/%d/%Y"),
    Time = format(mdy_hms(ActivityHour), "%H:%M:%S")
  ) %>%
  select(-ActivityHour) %>%
  select(Id, Date, Time, StepTotal)
```

#### Hourly Calories

```r
hourly_calories <- hourly_calories %>%
  mutate(
    Date = format(as.Date(mdy_hms(ActivityHour)), "%m/%d/%Y"),
    Time = format(mdy_hms(ActivityHour), "%H:%M:%S")
  ) %>%
  select(-ActivityHour) %>%
  select(Id, Date, Time, Calories)
```

### Formatting the Date on Sleep Day Table

```r
sleep_day <- sleep_day %>%
  mutate(
    SleepDay = as.Date(SleepDay, format="%m/%d/%Y %I:%M:%S %p", tz="UTC"),
    Date = as.Date(SleepDay),
    Time = format(SleepDay, "%H:%M:%S")
  ) %>%
  select(-c(SleepDay, Time)) %>%
  rename(SleepDay = Date)
```

### Update Date Formats Across All Tables to Match R Standards

```r
daily_activity$ActivityDate <- as.Date(daily_activity$ActivityDate, format = "%m/%d/%Y")
daily_activity$Day_of_Week <- weekdays(daily_activity$ActivityDate)

sleep_day$SleepDay <- as.Date(sleep_day$SleepDay)
sleep_day$Day_of_Week <- weekdays(sleep_day$SleepDay)

hourly_steps$Date <- as.Date(hourly_steps$Date, format = "%m/%d/%Y")
hourly_calories$Date <- as.Date(hourly_calories$Date, format = "%m/%d/%Y")
```




