# Health Smart Device Usage Business Analysis

This is Google Professional Certificate Final Capstone Project (Unguided), on Wellness Tech Company, Bellabeat. Taking the role as Marketing Data Analyst to answer some business questions to increase the sales and optimize the marketing strategy.


**Scenario**:
Bellabeat's founder and Chief Creative Officer aware that analysis of Bellabeat's users data would reveal more opportunities for growth, hence asking Marketing Data Analyst to help analyze trends from users usage. Dataset source: [here](https://www.kaggle.com/datasets/arashnic/fitbit)

**Business task** : Analyze smart device usage data to gain insight into how users use their smart devices and make recommendations for developing Bellabeat marketing strategies.

Questions that arise to help the analysis process:
1. What are the trends in the use of smart devices?
2. How can this trend be applied to Bellabeat subscribers?
3. How can this trend help influence Bellabeat's marketing strategy?


## Prepare

### Dataset information

There are 18 files in CSV format that available. What user informations are available?
- Activity (daily)
- Activity intensity (daily, hourly, per minute)
- Total calories burned (daily, hourly, minute)
- Number of heart rate (seconds)
- Steps (daily, minutes, hourly)
- Weight
- Sleeps (daily, minutes)
- MET (minutes)



### Importing Library
```{r}
library(tidyverse)
library(tidyr)
library(lubridate)
library(ggplot2)
library(janitor)
```


### Importing the dataset
```{r}
daily_activity <- read.csv("dailyActivity_merged.csv")
head(daily_activity)

daily_sleep <- read.csv("sleepDay_merged.csv")
head(daily_sleep)

hourly_sleep <- read.csv("minuteSleep_merged.csv")
head(hourly_sleep)

hourly_calories <- read.csv("hourlyCalories_merged.csv")
head(hourly_calories)

hourly_steps <- read.csv("hourlySteps_merged.csv")
head(hourly_steps)

heart_rate <- read.csv("heartrate_seconds_merged.csv")
head(heart_rate)
```


## Data Cleaning

### Checking unique ID each dataset
```{r}
n_distinct(daily_activity$Id)
n_distinct(daily_sleep$Id)
n_distinct(hourly_sleep$Id)
n_distinct(hourly_steps$Id)
n_distinct(hourly_calories$Id)
n_distinct(heart_rate$Id)
```
Heart rate dataset is removed because only contains 14 unique user ids and not representative enough

### Check duplicates
```{r}
sum(duplicated(daily_activity))
sum(duplicated(daily_sleep))
sum(duplicated(hourly_sleep))
sum(duplicated(hourly_calories))
sum(duplicated(hourly_steps))
```

### Make field names uniform and no duplicates
```{r}
daily_activity <- daily_activity %>%
  clean_names() %>%
  unique()

daily_sleep <- daily_sleep %>%
  clean_names() %>%
  unique()
sum(duplicated(daily_sleep))      # re-check the amount of NA

hourly_sleep <- hourly_sleep %>%
  clean_names() %>%
  unique()
sum(duplicated(hourly_sleep))     # re-check the amount of NA

hourly_steps <- hourly_steps %>%
  clean_names() %>%
  unique()

hourly_calories <- hourly_calories %>%
  clean_names() %>%
  unique()
```

### Check NA

```{r}
sum(is.na(daily_activity))
sum(is.na(daily_sleep))
sum(is.na(hourly_sleep))
sum(is.na(hourly_steps))
sum(is.na(hourly_calories))
```
### Check Data Structures

```{r}
glimpse(daily_activity)
```
```{r}
glimpse(daily_sleep)
glimpse(hourly_sleep)
```
```{r}
glimpse(hourly_steps)
```
```{r}
glimpse(hourly_calories)
```

### Correcting date format

```{r}
daily_activity <- daily_activity %>%
  rename(date = activity_date) %>%
  mutate(date = as_date(date, format = "%m/%d/%Y"))

glimpse(daily_activity)
```

```{r}
daily_sleep <- daily_sleep %>%
  rename(date = sleep_day) %>%
  mutate(date = as_date(date, format = "%m/%d/%Y"))
glimpse(daily_sleep)

hourly_sleep <- hourly_sleep %>%
  rename(date_time = date) %>%
  mutate(date_time = as.POSIXct(date_time, format = "%m/%d/%Y %I:%M:%S %p", tz = Sys.timezone()))
glimpse(hourly_sleep)
```

### Correcting data time format with as.POSIXct

```{r}
hourly_steps <- hourly_steps %>%
  rename(date_time = activity_hour) %>%
  mutate(date_time = as.POSIXct(date_time, format = "%m/%d/%Y %I:%M:%S %p", tz = Sys.timezone()))

glimpse(hourly_steps)
```

```{r}
hourly_calories <- hourly_calories %>%
  rename(date_time = activity_hour) %>%
  mutate(date_time = as.POSIXct(date_time, format = "%m/%d/%Y %I:%M:%S %p", tz = Sys.timezone()))

glimpse(hourly_calories)
```

## Exploratory Data Analysis

### Descriptive Statistics
```{r}
daily_activity %>% summary()
daily_sleep %>% summary()
hourly_sleep %>% summary()
hourly_steps %>% summary()
hourly_calories %>% summary()
```

### Plotting users habit: Application log 

```{r}
daily_activity %>%
  mutate(day = weekdays(date)) %>%
  group_by(day) %>%
  summarise(app_log = n()) %>%
  ggplot(aes(day, app_log, fill = day)) + 
  geom_bar(stat = "identity") + 
  labs(x=NULL,y="num of logs", title="Application log")
```
![image](https://user-images.githubusercontent.com/104981673/199954592-2dc74d76-bff9-4231-8c37-1b1147bad815.png)

Bellabeat users did not use the application every day. Most of them only use the application / wearing the gadget in Tuesday - Thursday.


### Plotting User activities category distribution using a pie chart

```{r fig.height=8, fig.width=10}
library(ggrepel)              # importing another library to make visualization labels

daily_activity %>%
  summarise(sedentary_mins = sum(sedentary_minutes), lightly_active_mins = sum(lightly_active_minutes), fairly_active_mins = sum(fairly_active_minutes), very_active_mins = sum(very_active_minutes)) %>%
  pivot_longer(cols =1:4, names_to="category",values_to='total_mins') %>%
  mutate(percentage = total_mins/sum(total_mins)) %>%
  mutate(percentage = scales::percent(percentage)) %>%
  ggplot(aes(x = "", y = total_mins, fill = category)) +
  geom_col() +
  labs(title="Activity", x=NULL, y=NULL) +
  theme(plot.title = element_text(hjust = 0.5)) +
  geom_label_repel(aes(label = percentage),
            position = position_stack(vjust = 0.5), show.legend=F) +
  coord_polar(theta = "y")
```
![image](https://user-images.githubusercontent.com/104981673/199954661-875ec325-ba4a-4037-8ed8-b736cbed6334.png)

The graph shows that users do sedentary activities the most.

## Data Analysis and Visualization

### Merging datasets
Merging datasets needed to observe correlation between two variables.

## 1. Daily activity and daily sleep record including 24 unique user ids.

```{r}
daily_activity_sleep <- left_join(daily_activity, daily_sleep, by=c("id", "date")) %>% drop_na()

glimpse(daily_activity_sleep)
head(daily_activity_sleep)
sum(is.na(daily_activity_sleep))
n_distinct(daily_activity_sleep$id)

# Modifying dataset by dividing dates into days of the week

daily_activity_sleep <- daily_activity_sleep %>%
  mutate(day = weekdays(date)) %>%
  select(id, day, date, everything())
head(daily_activity_sleep)
sum(is.na(daily_activity_sleep))
```

### Calculating users habit

#### Sleeping log

```{r}
daily_activity_sleep %>%
  group_by(day) %>%
  summarise(app_log = n()) %>%
  ggplot(aes(day, app_log, fill = day)) + 
  geom_bar(stat = "identity") + 
  labs(x=NULL,title="Application log (sleeping habit)")
```
![image](https://user-images.githubusercontent.com/104981673/199954801-67014963-aeb0-49a1-8339-621a1346959b.png)

It is known that the number of users who enter their sleep data is only 24 people out of 33 participants. The graph shows that they do not do it every day, but most often between Tuesdays and Thursdays.


#### Grouping users into activity intensity categories

```{r}
user_intensities <- daily_activity_sleep %>% 
  group_by(id) %>%
  rename(sedentary = sedentary_minutes, light = lightly_active_minutes, fair = fairly_active_minutes, very = very_active_minutes) %>%
  mutate(
    intensities = case_when (
      light < mean(light) & fair < mean(fair) & very < mean(very) ~ "Sedentary",
      light >= mean(light) & fair < mean(fair) & very < mean(very) ~ "Light",
      fair >= mean(fair) & very < mean(very) ~ "Fair",
      very >= mean(very) ~ "Very Active"))

head(user_intensities)
sum(is.na(user_intensities$intensities))

user_intensities %>%
  select(sedentary, light, fair, very) %>%
  summary()

user_intensities %>%   
  ggplot(aes(intensities, fill = intensities)) + 
  geom_bar() + 
  labs(title="User intensities", x=NULL, y="users")
```
![image](https://user-images.githubusercontent.com/104981673/199954929-ce848977-9b7a-4031-b675-0fd83849964b.png)

Based on the dataset, the majority of Bellabeat users who enter their activity data completely (including their sleep activity) are the ones who are **active**.


#### User intensities per Day of the week

```{r fig.height=10, fig.width=12}
user_intensities %>%   
  ggplot(aes(day, fill = day)) + 
  geom_bar() + 
  labs(title="Intensities per Day", x=NULL) +
  facet_wrap(~ intensities)
```
![image](https://user-images.githubusercontent.com/104981673/199955184-c0b1e809-1263-42e4-902b-095963b43f27.png)

Most of users do sedentary activities in **Sunday** and heavy activities in **Tuesday.**


#### Plotting the correlation between activity intensities and calories burned

```{r}
ggplot(user_intensities, aes(x=sedentary, y= calories)) + 
geom_point() + geom_smooth() +
  labs(title="Sedentary Activity vs. Calories burned", x="sedentary activity minutes")

ggplot(user_intensities, aes(x=light, y= calories)) + 
geom_point() + geom_smooth() +
  labs(title="Light Activity vs. Calories burned", x="light activity minutes")

ggplot(user_intensities, aes(x=fair, y= calories)) + 
geom_point() + geom_smooth() +
  labs(title="Fairly Active Activity vs. Calories burned", x="fairly active activity minutes")

ggplot(user_intensities, aes(x=very, y= calories)) + 
geom_point() + geom_smooth() +
  labs(title="Heavy Activity vs. Calories burned", x="heavy activity minutes")
  
user_intensities %>%
  mutate(total_activity = sedentary+light+fair+very) %>%
  ggplot(aes(x=total_activity, y= calories)) + 
  geom_point() + geom_smooth() +
  labs(title="Total Activity vs. Calories burned", x="total activity minutes")
```
![image](https://user-images.githubusercontent.com/104981673/199955608-08524a96-bb75-4270-b640-9b7e49c7fe84.png)
![activity vs calories burned](https://user-images.githubusercontent.com/104981673/199956876-81510407-dbe8-4862-b48c-011152fb8f1c.jpg)

Heavy activity becomes the only type of activities that have positive correlation with amount of calories burned. The heavier the activity, the more calories burned.

#### Dividing users sleeping habit into categories

Sleep category per hours divided based on https://www.cdc.gov/sleep/about_sleep/how_much_sleep.html

```{r}
user_sleep_steps <- daily_activity_sleep %>% 
  group_by(id) %>%
  mutate(mean_sleep = mean(total_minutes_asleep), mean_steps = mean(total_steps)) %>%
  mutate(
    sleep_quality = factor(case_when (
    mean(total_minutes_asleep) < 420 ~ "Not Enough Sleep", 
    mean(total_minutes_asleep) >= 420 & mean(total_minutes_asleep) <= 540 ~"Ideal", 
    mean(total_minutes_asleep) >540 ~ 'Too Much Sleep'),
    levels = c("Not Enough Sleep","Ideal","Too Much Sleep"))) %>%
  select(id, mean_sleep, sleep_quality, mean_steps) %>%
  unique()

head(user_sleep_steps)
glimpse(user_sleep_steps)
```
<img width="455" alt="image" src="https://user-images.githubusercontent.com/104981673/199957068-96940438-070c-427d-9d9c-1dc626d6e1c1.png">


#### Calculating number of users with their sleeping habit

```{r}
ggplot(user_sleep_steps, aes(sleep_quality, fill = sleep_quality)) +
  geom_bar() +
labs(x=NULL, fill="Users sleep quality")
```
![image](https://user-images.githubusercontent.com/104981673/199957123-e7e0998c-9a5c-4f3e-8e6f-d8d5919dac23.png)

#### Comparing the mean of steps taken each user with their sleep quality

```{r}
ggplot(user_sleep_steps, aes(sleep_quality, mean_steps, fill = sleep_quality)) +
geom_bar(position = "dodge", stat = "identity") +
labs(x=NULL, fill="Steps and sleep quality")
```
![image](https://user-images.githubusercontent.com/104981673/199957152-853f33e8-fea3-4fa4-a32e-5f0845690abb.png)

#### Plotting correlation between total steps taken and total sleeping minutes

```{r}
ggplot(daily_activity_sleep, aes(x=total_steps, y= total_minutes_asleep)) + 
geom_point() + geom_smooth() +
  labs(title="Total steps vs. Sleep Duration", x="Total steps", y="Sleep duration") 
```
![image](https://user-images.githubusercontent.com/104981673/199957205-0a7503b2-e51c-4b8a-948c-126f4d31c8ff.png)

The graph shows no correlation between total steps taken and sleep duration.


#### Plotting correlation between total activites throughout the day and total sleeping minutes

Heavy activity is chosen to check the correlation

```{r}
user_intensities %>%
  mutate(total_activity = sedentary+light+fair+very) %>%
  ggplot(aes(x=total_activity, y= total_minutes_asleep)) + 
  geom_point() + geom_smooth() +
  labs(title="Total active time vs. Sleep duration")

ggplot(user_intensities, aes(y= total_minutes_asleep, x=very)) + 
geom_point() + geom_smooth() +
  labs(title="Heavy activity time vs. Sleep duration", y="Total minutes asleep", x="Heavy activity (minutes)")
```
![image](https://user-images.githubusercontent.com/104981673/199957264-6ce872b7-64e4-43b6-9a8b-0a6d4f385e43.png)

The graph above shows that there is a negative correlation between total activity time and sleep duration. The longer activity time in a day, the less the sleep duration.


## 2. Hourly steps and hourly calories including 33 unique user ids.

### Merging datasets
```{r}
hourly_steps_cal <- left_join(hourly_steps, hourly_calories, by=c("id", "date_time"))

glimpse(hourly_steps_cal)
head(hourly_steps_cal)
sum(is.na(hourly_steps_cal))
n_distinct(hourly_steps_cal$id)
```

### Dividing time of the Day

```{r}
breaks <- hour(hm("00:00", "6:00", "12:00", "18:00", "23:59"))

labels <- c("Night", "Morning", "Afternoon", "Evening")

hourly_steps_cal %>%
  mutate(time_of_day = cut(x=hour(date_time), breaks = breaks, labels = labels, include.lowest=TRUE))
```
<img width="539" alt="image" src="https://user-images.githubusercontent.com/104981673/199957395-b54b8bd3-becb-4a0c-954a-089c9b6a8966.png">

### Plotting amount of calories burned based on time of the day

```{r fig.height=10, fig.width=12}
hourly_steps_cal %>% 
  separate(date_time, sep = " ", into = c("date","time")) %>% 
  mutate(date = as_date(date), time = format(parse_date_time(as.character(time), "HMS"), format = "%H:%M")) %>%
  ggplot(aes(time, calories, fill=time)) +
  geom_bar(position = "dodge", stat = "identity") +
  labs(x=NULL, fill="Hour")

hourly_steps_cal %>%
  mutate(time_of_day = cut(x=hour(date_time), breaks = breaks, labels = labels, include.lowest=TRUE)) %>%
  ggplot(aes(time_of_day, calories, fill=time_of_day)) +
geom_bar(position = "dodge", stat = "identity") +
labs(x=NULL, fill="Time of the day")
```
![amount of calories burned based on time of the day](https://user-images.githubusercontent.com/104981673/199957887-eb7e9235-2ef7-4679-892a-5df8e8a1990e.jpg)

```{r}
# modifying dataset to separate date and time column

hourly_steps_cal <- hourly_steps_cal %>% 
  separate(date_time, sep = " ", into = c("date","time")) %>% 
  mutate(date = as_date(date), time = format(parse_date_time(as.character(time), "HMS"), format = "%H:%M")) %>%
  mutate(day = weekdays(date)) %>%
  select(id, day, date, everything())
         
head(hourly_steps_cal)
n_distinct(hourly_steps_cal$id)
```

### Plotting amount of calories burned based on Day of the week

```{r}
ggplot(hourly_steps_cal, aes(day, calories, fill = day)) +
geom_bar(position = "dodge", stat = "identity") +
labs(x=NULL, title="Calories burned per day", fill="Day")
```
![image](https://user-images.githubusercontent.com/104981673/199958029-dd530a92-bf6c-4478-baa4-685d9c8fc8d3.png)

### Plotting amount of total steps taken each Day

```{r}
ggplot(hourly_steps_cal, aes(day, step_total, fill = day)) +
geom_bar(position = "dodge", stat = "identity") +
labs(x=NULL, y="Total steps", title="Total steps per Day", fill="Day")
```
![image](https://user-images.githubusercontent.com/104981673/199958063-701aeca1-6d9a-4b57-a920-41ceb65d85a7.png)

### Plotting correlation between total steps taken and amount of calories burned

```{r}
ggplot(hourly_steps_cal, aes(x=step_total, y= calories)) + 
geom_point() + geom_smooth() +
  labs(title="Total steps vs. Calories Burned", x="Total steps", y="Calories burned")
```
![image](https://user-images.githubusercontent.com/104981673/199958095-69218010-cce4-40fb-8166-b7aa0bf0e95f.png)

The graph shows positive correlations between total steps taken each day and amount of calories burned


# Conclusion



# Recommendation



