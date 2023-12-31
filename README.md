---
title: "Bellabeat Report"
author: "Akanksha Panwar"

output: html_document
---

# Business Task:

-   To analyze trends in smart device data to see how users use them.
-   Bellabeat is a company that tracks health data for women. It was founded by Urška Sršen: Bellabeat's cofounder and Chief Creative Officer and Sando Mur: Mathematician and Bellabeat's cofounder.
-   The purpose of analyzing smart device usage is to find out how these devices are used and give recommendations for one of Bellabeat's products to influence Bellabeat's marketing strategy.

# Preparing the Data:

-   Cofounder Sršen has asked for a specific dataset called [FitBit Fitness Tracker Data](https://www.kaggle.com/arashnic/fitbit) to be used in the analysis.
-   This dataset has been made available by a dataset expert who goes by the name [Mobius](https://www.kaggle.com/arashnic) on Kaggle. This dataset contains the fitness data of 30 Fitbit users who consented to make their personal fitness data public.
-   Out of all the files in the data, only minute-level data for calories, intensities, and steps are provided in both long and wide formats. The rest of the files; daily activities, heart rate, METs, and hourly data for calories, intensities, and steps, are all in long format.
-   The Data needs a lot of cleaning and that will be done in the next phase.

## Loading Packages:

```{r}
library(tidyverse)
library(lubridate)
library(dplyr)
library(ggplot2)
library(janitor)
library(reshape2)
```

## Importing Datasets:

```{r}
daily_activity <- read_csv("/kaggle/input/fitbit/Fitabase Data 4.12.16-5.12.16/dailyActivity_merged.csv")
weight_log <- read_csv("/kaggle/input/fitbit/Fitabase Data 4.12.16-5.12.16/weightLogInfo_merged.csv")
sleep_log <- read_csv("/kaggle/input/fitbit/Fitabase Data 4.12.16-5.12.16/sleepDay_merged.csv")
mets <- read_csv("/kaggle/input/fitbit/Fitabase Data 4.12.16-5.12.16/minuteMETsNarrow_merged.csv")
```

# Cleaning Process:

```{r}
head(daily_activity)
```

![head(daily_activity)](https://github.com/AkankshaPanwar/R-Projects/assets/78524003/e80b5770-8726-4c1c-b130-088d72b67fe4)


```{r}
glimpse(daily_activity)
```
![glimpse(daily_activity)](https://github.com/AkankshaPanwar/R-Projects/assets/78524003/153870ed-b841-4afe-8b7d-057ab744551e)


```{r}
daily_activity <- clean_names(daily_activity)
daily_activity$date <- as.POSIXct(daily_activity$activity_date, format="%m/%d/%Y", tz=Sys.timezone())
daily_activity$date <- as.Date(daily_activity$date, "%d/%m/%Y", tz=Sys.timezone())
daily_activity <- daily_activity[, -2]
```
```{r}
glimpse(mets)
```
![glimpse(mets)](https://github.com/AkankshaPanwar/R-Projects/assets/78524003/36b27169-0f6f-4eb4-8109-b292ee12c9d4)


```{r}
mets_log <- rename_with(mets, tolower)
mets_log$date <- as.Date(mets_log$activityminute, "%d/%m/%Y")
mets_log$activity_minute <- parse_date_time(mets_log$activityminute, "%m/%d/%Y %I:%M:%S %p")
mets_log <- mets_log[, -2]
```
```{r}
glimpse(sleep_log)
```
![glimpse(sleep_log)](https://github.com/AkankshaPanwar/R-Projects/assets/78524003/90dc6073-c891-47bf-ad27-5b4eb836335b)

```{r}
sleep_log <- clean_names(sleep_log)
sleep_log$date <- as.POSIXct(sleep_log$sleep_day, format="%m/%d/%Y", tz=Sys.timezone())
sleep_log$date <- as.Date(sleep_log$date, "%d/%m/%Y", tz=Sys.timezone())
sleep_log <- sleep_log[, -2]
```
```{r}
glimpse(weight_log)
```
![glimpse(weight_log)](https://github.com/AkankshaPanwar/R-Projects/assets/78524003/2f1a61ae-9279-4126-b5df-fe256cc259d7)

```{r}
weight_log$date <- as.Date(weight_log$Date, "%d/%m/%Y", tz=Sys.timezone())
weight_log <- clean_names(weight_log)
weight_log <- weight_log[, -2]
```

-   Added underscores between the words in column names and changed all column names to lowercase.
-   change activity_date, activityminute, sleep_day, Date columns from string format to date time format.
-   Added date column to all the datasets in date format for merging later.
-   Removed columns that were no longer required for the analysis process.

```{r}
n_distinct(daily_activity$id)
n_distinct(mets_log$id)
n_distinct(sleep_log$id)
n_distinct(weight_log$id)
```
![n_distinct](https://github.com/AkankshaPanwar/R-Projects/assets/78524003/71da00c5-51e0-4b60-80ad-672b5f4bf0f2)


-   Number of participants in weight_log data frame is 8, which is not a significant number to conduct data analysis on the data frame. Therefore, I will use daily_activity, mets_log, and sleep_log data frames for the analysis.

# Analyse Data

-   Merged the data frames "daily_activity" and "sleep_log" according to "id" and "date" columns, not including some columns that are not required.

```{r}
merge_1 <- merge(daily_activity, sleep_log, by=c('id', 'date')) %>% select(id, date, total_steps, total_distance, very_active_minutes, fairly_active_minutes, lightly_active_minutes, sedentary_minutes, calories, total_minutes_asleep)

merge_1 %>% select(-id, -date) %>% summary()
```
![merge_1 summary()](https://github.com/AkankshaPanwar/R-Projects/assets/78524003/140fb39a-9731-4390-a253-51fc82966c7d)

-   According to the summary statistics of the dataset "merge_1", the average sedentary minutes is 712.2 minutes, i.e., approximately 12 hours. This is too much inactivity in one day.
-   Average sleep duration is 419.5 minutes, i.e., 7 hours, which is not bad as it should be around 6-8 hours per day.
-   Participants are mostly lightly active and barely spend time either, vigorously active or moderately active.

## Visualizing Data:

```{r}
head(merge_1)
```
![head(merge_1)](https://github.com/AkankshaPanwar/R-Projects/assets/78524003/57ea91cb-9632-44ec-b9da-f306a8503758)


```{r}
activity_type_data <- merge_1 %>%
  summarise(
  activity_type = factor(case_when(
  sedentary_minutes > mean(sedentary_minutes) & lightly_active_minutes < mean(lightly_active_minutes) & fairly_active_minutes < mean(fairly_active_minutes) & very_active_minutes < mean(very_active_minutes) ~ "sedentary",
  sedentary_minutes < mean(sedentary_minutes) & lightly_active_minutes > mean(lightly_active_minutes) & fairly_active_minutes < mean(fairly_active_minutes) & very_active_minutes < mean(very_active_minutes) ~ "lightly_active", 
  sedentary_minutes < mean(sedentary_minutes) & lightly_active_minutes < mean(lightly_active_minutes) & fairly_active_minutes > mean(fairly_active_minutes) & very_active_minutes < mean(very_active_minutes) ~ "fairly_active", 
  sedentary_minutes < mean(sedentary_minutes) & lightly_active_minutes < mean(lightly_active_minutes) & fairly_active_minutes < mean(fairly_active_minutes) & very_active_minutes > mean(very_active_minutes) ~ "very_active"), 
  levels = c("sedentary", "lightly_active", "fairly_active", "very_active")), 
  calories, total_minutes_asleep, .groups = id) %>% 
  drop_na()
  
activity_type_data %>% 
  group_by(activity_type) %>% 
  summarise(
  total = n()) %>% 
  mutate(totals = sum(total)) %>% 
  group_by(activity_type) %>% 
  summarise(
    participants_percent = total/totals) %>% 
  ggplot(
    aes(activity_type, y = participants_percent, fill=activity_type)) + 
  geom_col() +  
  scale_y_continuous(labels = scales::percent) + 
  labs(title = "Activity Type Distribution" , x= NULL)
```
![Activity Type Distribution](https://github.com/AkankshaPanwar/R-Projects/assets/78524003/e4fcac46-4bf8-4399-846e-0d386f550fd0)


-   The "Activity Type Distribution" graph shows that the majority of the participants spent time being sedentary or lightly active.

```{r}
activity_type_data %>%    ggplot(aes(activity_type, calories, fill = activity_type)) +    geom_boxplot() +    labs(title = "Calories Burned during each Activity Type", x=NULL)
``` 
![Calories burned by each activity type](https://github.com/AkankshaPanwar/R-Projects/assets/78524003/cadf1bab-a3c8-477c-b179-d2a2a4aa52ab)


-   The Box Plot shows that the participants who are very active burn the most amount of calories.

```{r}
sleep_type_data <- merge_1 %>% 
  group_by(id) %>% 
  summarise(
  activity_type = factor(case_when(
  sedentary_minutes > mean(sedentary_minutes) & lightly_active_minutes < mean(lightly_active_minutes) & fairly_active_minutes < mean(fairly_active_minutes) & very_active_minutes < mean(very_active_minutes) ~ "sedentary", 
  sedentary_minutes < mean(sedentary_minutes) & lightly_active_minutes > mean(lightly_active_minutes) & fairly_active_minutes < mean(fairly_active_minutes) & very_active_minutes < mean(very_active_minutes) ~ "lightly_active", 
  sedentary_minutes < mean(sedentary_minutes) & lightly_active_minutes < mean(lightly_active_minutes) & fairly_active_minutes > mean(fairly_active_minutes) & very_active_minutes < mean(very_active_minutes) ~ "fairly_active", 
  sedentary_minutes < mean(sedentary_minutes) & lightly_active_minutes < mean(lightly_active_minutes) & fairly_active_minutes < mean(fairly_active_minutes) & very_active_minutes > mean(very_active_minutes) ~ "very_active"), 
  levels = c("sedentary", "lightly_active", "fairly_active", "very_active")), 
  sleep_type = factor(case_when(
    mean(total_minutes_asleep) < 360 ~ "Poor Sleep", 
    mean(total_minutes_asleep) > 360 & mean(total_minutes_asleep) <= 480 ~ "Normal Sleep",
    mean(total_minutes_asleep) > 480 ~ "Oversleep"), 
    levels = c("Poor Sleep", "Normal Sleep", "Oversleep")), 
  total_sleep = sum(total_minutes_asleep), .groups = "drop") %>% 
  drop_na() %>% 
  group_by(activity_type) %>% 
  summarise(
    bad_sleepers = sum(sleep_type == "Poor Sleep"), 
    normal_sleepers = sum(sleep_type == "Normal Sleep"), 
    oversleepers = sum(sleep_type == "Oversleep"), 
    total = n(), .groups= "drop") %>% 
  group_by(activity_type) %>% 
  summarise(
    bad_sleepers = bad_sleepers/total, 
    normal_sleepers = normal_sleepers/total, 
    oversleepers = oversleepers/total, .groups = "drop")
```

```{r}
sleep_type_data_melted <- melt(sleep_type_data, id.vars = "activity_type")

ggplot(sleep_type_data_melted, aes(activity_type, value, fill= variable)) + geom_bar(position = "dodge", stat = "identity") + scale_y_continuous(labels = scales::percent) + labs(title = "Sleep Quality according Activity Type", x=NULL, fill= "Sleep Type")
```
![Sleep Quality according to Activity Type](https://github.com/AkankshaPanwar/R-Projects/assets/78524003/d2c39499-c738-41dc-8c28-55a626045cd3)


-   The Bar graph for comparing sleep quality with activity type shows that people who are fairly active or very active, tend to not oversleep at all. Although around 30% of them don't seem to get enough sleep.
-   Of the people who are mostly sedentary or lightly active, around 20% each, tend to oversleep.

# Conclusions: 
  According to the observations made above, I have the following insights for the Bellabeat App:

-   Since the majority of the participants are sedentary or lightly active, the app can send frequent notifications as reminders to take a break and move and rehydrate for the user. 
-   The app could have a feature for setting daily or weekly goals, like calories burned or total steps, that the user can set for themselves and aim to achieve in incremental stages. 
-   Daily motivational quotes or even health-specific articles, on a separate tab in the app, might also encourage the user to make healthy changes to their lifestyle. 
-   A weekly report on the user's sleep cycle could bring light to any changes the user might want to make to their sleeping habits. The app can enlighten the user about the risks of sleeping too little or oversleeping.
-   An animated yearly report at the end of the year, like a rewind, showing every user their stats and goals set and achieved throughout the past year is also a good idea to make the user realize how much difference one year can make, even with small changes in lifestyle. Thereby, encouraging the user to do even better in the upcoming year. 
-   Maybe the company can offer incentives to the user like posting about the user's progress on the company's official social media pages.

### Thank you for reading!
