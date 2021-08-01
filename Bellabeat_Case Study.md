---
title: "Bellabeat Case Study"
author: "Zaryn Ooi"
date: "7/29/2021"
output: github_document
---
---


# Phase 1 : Ask 
### About the Company 
Bellabeat is a high tech company that manufactures health-focused smart products. Since it was founded in 2013, Bellabeat has grown rapidly and positioned itself as a tech-driven wellness company for women. 

### Key Stakeholders 
#### Primary stakeholders:

 - Urška Sršen, cofounder and Chief Creative Officer
 - Sando Mur, Mathematician and Bellabeat’s cofounder

#### Secondary stakeholders: 

 - Bellabeat marketing analytics team

### Business task 
To identify trends in smart devices usage to help Bellabeat make marketing strategy improvement. 



# Phase 2 : Prepare
### Data Source
Fitbit Fitness Tracker Data (CC0: Public Domain, dataset made available through Mobius)

<https://www.kaggle.com/arashnic/fitbit>

This data set contains personal fitness tracker from thirty Fitbit users. Thirty eligible Fitbit users consented to the submission of personal tracker data, including minute-level output for physical activity, heart rate, and sleep monitoring. It includes information about daily activity, steps, and heart rate that can be used to explore users’ habits.

### Bias/ Credibility (ROCCC Analysis) 
  - Reliability = YES
  - Original = YES
  - Comprehensive = YES 
  - Current = NO
  - Cited = YES

### Limitations
  - Data is not current as it was generated between 03.12.2016 - 05.12.2016.

  - Sample size of 30 is too small to represent the population. 

  - The data does not show the demographic information of the respondent. Hence, the data cannot ensure that the respondent is a representative of female users, given that the target audience for Bellabeat is female.

  - The data were collected by different types of Fitbit trackers and individual tracking behaviors / preferences. However, the data does not indicate which devices were used. 

### Data Selection 
For this analysis, i have selected 2 datasets that provided daily activity and the sleep time of the user because there may be a correlation between the sleep behaviours and the overall activity of the users, which are helpful for me to identify trends in smart devices usage. 

#### Files selected: 
  - dailyActivity_merged.csv
  - sleepDay_merged.csv



# Phase 3 : Process

## Install and Load Packages

I have installed necessary pacakges and dataasets for analysis in R.
```{r}
install.packages('tidyverse')
install.packages('dplyr')
install.packages('ggplot2')
install.packages('lubridate')
install.packages('skimr')

library(tidyverse)
library(dplyr)
library(ggplot2)
library(lubridate)
library(skimr)
```


## Importing data

I will create a dataframe named 'daily_activity' for the daily activity data and 'sleep_day' for the sleep data.

```{r}
daily_activity <- read.csv("dailyActivity_merged.csv")
```

Let's use the `head()` function to display the first 6 rows present in the 'daily_activity' data frame.

```{r}
head(daily_activity)
```

```{r}
sleep_day <- read.csv("sleepDay_merged.csv")
```

```{r}
head(sleep_day)
```


## Explore the imported data

```{r}
head(daily_activity)
```

```{r}
head(sleep_day)
```

```{r}
skim_without_charts(daily_activity)
skim_without_charts(sleep_day)
```


## Fixing Formatting

After exploring the datasets, I have noticed that there are some problems with the timestamp data. Before analysis, I will use the lubridate library's `mdy()` function to convert date strings as the date elements in daily_activity data are ordered as month, day and year.

```{r}
daily_activity <-
  daily_activity %>% 
  mutate(ActivityDate = mdy(ActivityDate))
```

Take a look and recheck the data.
```{r}
head(daily_activity)
```

I will use the `mdy_hms()` function to convert date strings as the date elements in sleep_day data are ordered as month, day, year, hour, minute, and second. 
```{r}
sleep_day <-
  sleep_day %>% 
  mutate(SleepDay = mdy_hms(SleepDay))
```

let's take a look and recheck the data. 
```{r}
head(sleep_day)
```


Next,I will rename both 'ActivityDate' and 'SleepDay' column to 'Date'.
```{r}
daily_activity <- 
  daily_activity %>%
rename(Date = ActivityDate)

sleep_day <- 
  sleep_day %>% 
rename(Date = SleepDay)
```

Now that i have finsihed formatting, I can start exploring the data. 



## Summarize the data

I will use `n_distinct()` function to find out the number of respondents in each dataset. 
```{r}
n_distinct(daily_activity$Id)
n_distinct(sleep_day$Id)
```
There are 33 participants in the daily_activity dataset, and only 24 in the sleep-day dataset. This shows that not all users have sleep data. 


The, I will use the `summary()` function to view the summary statistics of each dataset.

### Summary Statistics of daily_activity Data:

```{r}
daily_activity %>% 
  select(TotalSteps,
         TotalDistance,
         SedentaryMinutes, 
         Calories) %>% 
  summary()
```
Based on the summary, I have discovered that the average total steps of the respondents is 7638 per day, the average total distance is 5.490, and the average sedentary time is 991 minutes (16 hours). The goal of 10,000 steps is the recommended daily step target for healthy adults to achieve health benefits.

### Summary Statistics of sleep_day Data:

```{r}
sleep_day %>% 
  select(TotalSleepRecords,
         TotalMinutesAsleep,
         TotalTimeInBed) %>% 
  summary()
```
From this summary, the average time asleep of the respondents is 419.5 minutes (7 hours). National Sleep Foundation guidelines advise that healthy adults need between 7 and 9 hours of sleep per night.



## Merged the data

To identify the relationship between both datasets, I need to merge(inner join) both datasets using `inner_join()` function by "Id" and "Date". 
```{r}
combined_data <- inner_join(daily_activity, sleep_day, by = c("Id", "Date"))
```

```{r}
n_distinct(combined_data$Id)
```
Now that I have merged the two datasets, there will only be 24 respondents. 



# Phase 4 & Phase5 : Analyze & Share 

### Visualization 1 - Total Steps vs Calories

First, I would like to identify the relationship beween total steps taken in a day and the calories burn.
```{r}
ggplot(data = combined_data) +
  geom_point(mapping = aes(x = TotalSteps, y = Calories, color = Calories)) + labs(title = "Total Steps vs Calories")
```

The visualization shows that there is a positive correlation between total steps taken in a day and the calories burn. The more steps we taken, the more calories we burn. The marketing analytics team can position this as a way to motivate users who plan to lose weight to walk more.



### Visualization 2 - Total Minutes Asleep

Next, I would like to explore the sleep behaviors of the respondents. 

```{r}
ggplot(combined_data, aes(x=TotalMinutesAsleep)) + 
  geom_histogram(aes(y=..density..), binwidth=30, alpha=0.6) +
  geom_density(alpha=0.2, fill="green") +
  geom_vline(aes(xintercept=mean(TotalMinutesAsleep)), color="blue",  linetype="dashed") + 
  annotate(geom="text", x= 419.5 - 150, y = 0.004, size = 5, label = "mean = 419.5") +
  labs(title = "Total Minutes Asleep", x= "Total Minutes Asleep", y="Density")
```

The histogram shows that most respondents sleep around 419.5 minutes, which is 7 hours a day. This means that most respondents get enough hours of sleep and do not have bad sleeping habits. 


### Visualization 3 - Time Awake in Bed

I will identify the time awake in bed to recheck if the respondents have bad sleeping habits. Before plotting the visualization, I will add a new column called "TotalAwakeInBed" with the `mutate()` function to the data frame to identify if there are respondents who have bad sleeping habits like insomnia or sleep in. 

```{r}
combined_data <- 
  combined_data %>% 
    mutate(TimeAwakeInBed = TotalTimeInBed - TotalMinutesAsleep)
```


Now, I can use the "TimeAwakeInBed" column to plot the visualization to examine if the respondents are spending too much time awake in bed. I will use the `mean()`function to find the average time spent awake in bed. 

```{r}
mean(combined_data$TimeAwakeInBed)
```

```{r}
ggplot(combined_data, aes(x=TimeAwakeInBed)) + 
  geom_histogram(aes(y=..density..), binwidth=30, alpha=0.6)+
  geom_density(alpha=0.2, fill="green") +
  geom_vline(aes(xintercept=mean(TimeAwakeInBed)), color="blue", linetype="dashed") +
  geom_vline(aes(xintercept=max(TimeAwakeInBed)), color="blue", linetype="dashed") +
  annotate(geom="text", x=39 + 55, y = 0.02, size = 5, label = "mean = 39") +
  annotate(geom="text", x=371 - 50, y = 0.02, size = 5, label = "max = 371") +
  labs(title = "Time Awake In Bed", x= "Time Awake in Bed", y="Density")
```


From the histogram, I discovered that the average time of respondents awake in bed is 39 minutes, which shows that most of the respondents do not have bad sleeping habits. Interesting to note, there is a small amount of respondents who spent 371 minutes time awake in bed, which signals that some of the respondents might suffered from insomnia. 



### Visualization 4 - Total Steps vs Time Awake In Bed

From this analysis, I will construct a visualization to find out if there is a relationship between activity and the sleep quality. I will use  "TotalSteps" data to represent the activity and "TimeAwakeInBed" data to represent the sleep quality. 

```{r}
ggplot(data = combined_data) +
    geom_point(mapping = aes(x = TotalSteps, y = TimeAwakeInBed, color = TimeAwakeInBed)) + annotate("rect", xmin = 0, xmax = 5000, ymin = 300, ymax = 390, alpha = 0.2) + labs(title = "Total Steps vs Time Awake In Bed")
```


By looking at this scatterplot, we’re able to identify that users who suffered from insomnia are those who have taken less than 5000 steps a day. We’re also able to see that users who took more than 15000 steps do not spent more than 100 minutes awake in bed, which is less than an hour.This further suggests that these users do not develop bad sleeping habits. 

In short, daily step count influences sleep quality, and that higher daily step count leads to better sleep quality. 



### Visualization 5 - Most Frequent Types of Activity

I will construct a data frame and use the `mean()` function to calculate the mean for each types of Active Minutes to find out the most frequent types of activity among the respondents.  

```{r}
TypesofActivityByDate <- 
  combined_data %>% 
    group_by(Date) %>% 
    summarize(MeanVeryActiveMinutes = mean(VeryActiveMinutes), 
              MeanFairlyActiveMinutes = mean(FairlyActiveMinutes),
              MeanLightlyActiveMinutes = mean(LightlyActiveMinutes),
              MeanSedentaryMinutes = mean(SedentaryMinutes))

colors <- c("Very" = "green", "Fairly" = "orange", "Lightly" = "red", "Sedentary" = "blue")
ggplot(data = TypesofActivityByDate, aes(x = Date)) +
    geom_line(aes(y = MeanVeryActiveMinutes, color = "Very"), size = 1) +
    geom_line(aes(y = MeanFairlyActiveMinutes, color="Fairly"), size = 1) + geom_line(aes(y = MeanLightlyActiveMinutes, color="Lightly"), size = 1) + geom_line(aes(y = MeanSedentaryMinutes, color="Sedentary"), size = 1) +
labs(title = "Most Frequent Types of Activity", x = "Date", y = "Minutes (mean)", color = "Activity") +
    scale_color_manual(values = colors)
```


 By looking at this graph, we can see that the blue line appears to be top among the other lines. This means that sedentary activity is the most frequent type of activity carried out by the users, followed by 'Lightly' activity, 'Very' activity, while the least activity type is 'Fairly' activity. 
 
Given that “Sedentary Activity” is the most frequent types of activity carried out by the users, this means that most of the users are working adults who spent long hours sitting in front of the computer/ meeting (Sedentary). And they also spent some time doing some house chores after work (Lightly).



# Phase 6 : Act 
### Business Task : Identify Trends in Smart Device Usage

I have gathered 3 trends from my analysis: 

#### 1) Target Audience 

By analysing the most frequent types of activity as well as the sleep behavior data, I am able to to identify how does the target audience looks like. This can help Bellabeat identify the characteristic of a typical consumer. 

Target Audience of Bellabeat: 

 - Female working adults who spent most of the time doing sedentary activities, and rarely exercises.
 
 - Have a relatively good amount of sleep but may spend some time awake in bed.
 

#### 2)	Sleep Behavior Analysis

Given the fact that out of 33 respondents, only 24 of them record sleep data, this means that not every user will wear Bellabeat devices while sleeping. 

Reason:
•	Do not know that Bellabeat app can track sleep data
•	Do not like wearing Bellabeat devices while sleeping 
•	Do not know the importance of tracking sleep data

#### 3) Daily steps count improves overall well being.

By analysing the "TotalSteps" data and the "Time Spent Awake in Bed" data,  I found out that higher daily step count leads to better sleep quality. 

By analysing the daily step data and the calorie data, I've aslo discovered that higher daily step count leads to more calories burn. 


### Recommendations for Bellabeat App
After understanding the trends that I have discovered, Bellabeat should start:

1) Target potential customer 

Bellabeat can consider to target the potential customer. This can be done by crafting marketing messages in marketing campaigns/ Ads campaign to target people who have a similar activity pattern. 

2) Utilize the identified trend to improve user experience 

The next strategy that Bellabeat could bring into effective action, is to utilise the trends that I’ve discovered to improve the user experience of their Bellabeat App.   

Bellebeat can do this by first tracking the personal metrics of the users. For example, based on the personal metrics,Bellebeat identified that user A, Jamie is someone who suffer from insomnia, Bellebeat can send notifications to Jamie to remind her to go for a run, go to gym or to increase her frequency of higher intensive activities in order to improve her sleep quality. Additionally, these app notifications should combine with health knowledge such higher daily step counts leads to better sleep quality in order to motivate Jamie. 

For users who wants to lose weight, the app can send them notifications to remind these users to do more exercise to burn their calories. 

For users who do not record sleep data, Bellabeat App should send notifications to users to remind them to wear Bellabeat devices while sleeping in order to track their sleep data, combined with knowledge of the importance of tracking sleeping data. In addition, Bellabeat can allow users who used Bellabeat app to track their sleep data to earn and accrue reward points, which are then redeemable from wide range of gifts. 









