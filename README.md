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

