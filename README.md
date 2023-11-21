# Case-Study: Cyclistic Bike-Share Analysis

##### Author: Hanzian Ngoran.

##### Date: November 21th, 2023.

##### [Tableau Dashboard](https://public.tableau.com/app/profile/hanzian.ngoran/viz/CyclisticBike_17000644833820/Dashboard7#1).

## Analysis Overview
In 2016, Cyclistic launched a successful bike-share offering. The company’s future success depends on maximizing the number of annual memberships. Therefore, your team wants to understand how casual riders and annual members use Cyclistic bikes differently. From these insights, your team will design a new marketing strategy to convert casual riders into annual members.

## Analysis Process

This analysis follows 6 data analysis steps process:

### Step 1 : Ask

**Business Task**: Analyse the data to design marketing strategies aimed at converting casual riders into annual members.
**Stakeholders**: Lily Moreno: The director of marketing / Cyclistic marketing analytics team

### Step 2: Prepare

Data Source: 6 Month (Jan to Jun 2022) of Cyclistic trip Data from Motivate International Inc: [data source link](https://divvy-tripdata.s3.amazonaws.com/index.html) with [license](https://www.divvybikes.com/data-license-agreement).

The dataset has 6 CSV, 13 columns and 2,278,732 rows. The data also follow a ROCCC approach:

- Reliability: the data includes complete and accurate ride data from Divvy. Divvy is program of the Chicago Department of Transportation (CDOT), which owns the city’s bikes, stations and vehicles
- Original: the data is from Motivate International Inc, which operates the City of Chicago’s Divvy bicycle sharing service.
- Comprehensive: The data incudes type of bikes, start and end station name, start and end time, station ID, station longtitude and latitude, membership types.
- Current: data is up to date to January 2022
- Cited: the data is cited and under current [license](https://www.divvybikes.com/data-license-agreement) agreement.

The dataset has some limitations. We have a lot of NA values: after checking `sum(is.na(bike_data))`, we see the dataset has 1,386,388 NA values, such as in starting_station_id, end_station_id. Further investigation we noticed the NA values are mostly under rideable type: electric bike. Future investigations may be needed by the station names are not entered for electric bike. 

### Step 3: Process

Load the packages
```
install.packages('tidyverse')
install.packages('skimr')
library(tidyverse) #wrangle data
library(dplyr) #clean data
library(lubridate)  #wrangle date attributes
library(skimr) #get summary data
library(ggplot2) #visualize data
library(readr)
```

Prepare the data and combine them in one data frame.
#Combine the data from Jan 2022 to Jun 2022 into one data frame.
```
bike_data <- rbind(
  read_csv("202201-divvy-tripdata.csv"),
  read_csv("202202-divvy-tripdata.csv"),
  read_csv("202203-divvy-tripdata.csv"),
  read_csv("202204-divvy-tripdata.csv"),
  read_csv("202205-divvy-tripdata.csv"),
  read_csv("202206-divvy-tripdata.csv"))
```
Examine the data frame

```
head(bike_data)
dim(bike_data)
colnames(bike_data)
summary(bike_data)
```

Drop columns (start_lat, start_lng, end_lat, end_lng)

```
bike_data <- bike_data %>% select(-c(start_lat, start_lng, end_lat, end_lng))
```

 Add column “ride_length"

```
bike_data <- bike_data %>% mutate(ride_length = ended_at - started_at)
```

Add column "Week_day"
```
bike_data <- bike_data %>% mutate(day_of_week = weekdays(as.Date(bike_data$started_at)))

colnames(bike_data)
```
Convert ride_length from from seconds into minutes

```
bike_data$ride_length <- as.numeric(bike_data$ride_length)
bike_data$ride_length <- as.numeric(bike_data$ride_length/60)
head(bike_data)
```

Delete data with negative ride length

```
bike_data <- bike_data[bike_data$ride_length>0,]
```

### Step 4: Analysis

Check min, max, mean, median and any outlier on the ride length. 

```
summary(bike_data$ride_length)
```
Compare members and casual users

```
aggregate(bike_data$ride_length ~ bike_data$member_casual, FUN = mean)
aggregate(bike_data$ride_length ~ bike_data$member_casual, FUN = median)
aggregate(bike_data$ride_length ~ bike_data$member_casual, FUN = max)
aggregate(bike_data$ride_length ~ bike_data$member_casual, FUN = min)
```

Let's analyse the number of rides by rider type 
```
bike_data %>% 
  mutate(weekday = wday(started_at, label = TRUE)) %>% 
  group_by(member_casual, weekday) %>% 
  summarise(number_of_rides = n() ,average_duration = mean(ride_length)) %>% 
  arrange(member_casual, weekday)
```
![number of rides by rider type](https://github.com/Hanzian/Cyclistic-Bike-Share-Analysis-/blob/main/number_ride_by_rider.png)


Analyse the average duration
```
bike_data %>% 
  mutate(weekday = wday(started_at, label = TRUE)) %>% 
  group_by(member_casual, weekday) %>% 
  summarise(number_of_rides = n()
            ,average_duration = mean(ride_length)) %>% 
  arrange(member_casual, weekday)  
```
![average duration](https://github.com/Hanzian/Cyclistic-Bike-Share-Analysis-/blob/main/average_duration.png)


