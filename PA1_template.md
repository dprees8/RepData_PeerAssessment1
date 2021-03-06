---
title: "Reproducible Research - Project 1"
author: "Dan Rees"
date: "8/15/2020"
output: 
  html_document: 
    keep_md: yes
---



This is Project 1 for the Reproducible Research course.

### Step 1: Loading and preprocessing data

Load data: 


```r
activity_data<-read.csv(file="activity.csv")
```

### Step 2: What is mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in the dataset.

1. Calculate the total number of steps taken per day


```r
Steps_by_day <- activity_data %>% group_by(date) %>% summarize(Steps=sum(steps, na.rm = TRUE))

Steps_by_day
```

```
## # A tibble: 61 x 2
##    date       Steps
##    <fct>      <int>
##  1 2012-10-01     0
##  2 2012-10-02   126
##  3 2012-10-03 11352
##  4 2012-10-04 12116
##  5 2012-10-05 13294
##  6 2012-10-06 15420
##  7 2012-10-07 11015
##  8 2012-10-08     0
##  9 2012-10-09 12811
## 10 2012-10-10  9900
## # ... with 51 more rows
```

2. Make a histogram of the total number of steps taken each day


```r
Steps_by_day %>% 
  ggplot(aes(Steps)) + 
  geom_histogram(binwidth = 1000) + 
  labs(title="Histogram of Steps per Day", y = "# of Occurences", x = "Steps per Day")
```

![](PA1_template_files/figure-html/histogram-1.png)<!-- -->

3. Calculate and report the mean and median of the total number of steps taken per day.


```r
mean(Steps_by_day$Steps)
```

```
## [1] 9354.23
```

```r
median(Steps_by_day$Steps)
```

```
## [1] 10395
```

### Step 3: What is the average daily activity pattern?

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
Steps_by_interval <- activity_data %>% group_by(interval) %>% summarize(Steps=mean(steps, na.rm = TRUE))

Steps_by_interval %>%
  ggplot(aes(x=interval, y=Steps)) +
  geom_line() + 
  labs(title="Average # of Steps by 5 min Interval", y = "Avg # of Steps", x = "5 min Interval")
```

![](PA1_template_files/figure-html/time series plot-1.png)<!-- -->


2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
Steps_by_interval_order <- Steps_by_interval %>% arrange(desc(Steps))
Steps_by_interval_order[1,]
```

```
## # A tibble: 1 x 2
##   interval Steps
##      <int> <dbl>
## 1      835  206.
```


### Step 4: Imputing missing values

Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
sum(!complete.cases(activity_data))
```

```
## [1] 2304
```


2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

Approach - Replace missing number of steps based on mean for 5 minute interval.

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
activity_data_replace <- activity_data

activity_data_replace <- activity_data_replace %>% mutate(steps = ifelse(is.na(steps), 
                                                                      Steps_by_interval$Steps[
                                                                        which(Steps_by_interval$interval==interval)], 
                                                                         steps))
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
Steps_by_day_replace <- activity_data_replace %>% group_by(date) %>% summarize(Steps=sum(steps, na.rm = TRUE))

Steps_by_day %>% 
  ggplot(aes(Steps)) + 
  geom_histogram(binwidth = 1000) + 
  labs(title="Histogram of Steps per Day", y = "# of Occurences", x = "Steps per Day")
```

![](PA1_template_files/figure-html/new-1.png)<!-- -->

```r
mean(Steps_by_day_replace$Steps)
```

```
## [1] 9530.724
```

```r
mean(Steps_by_day$Steps)
```

```
## [1] 9354.23
```

```r
median(Steps_by_day_replace$Steps)
```

```
## [1] 10439
```

```r
median(Steps_by_day$Steps)
```

```
## [1] 10395
```

### Step 5: Are there differences in activity patterns between weekdays and weekends?

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.


```r
activity_data_replace <- activity_data_replace %>% mutate(date_ymd = ymd(date))

activity_data_replace <- activity_data_replace %>% mutate(weekdays = weekdays(date_ymd))

activity_data_replace <- activity_data_replace %>% 
          mutate(weekday_weekend = factor(ifelse(weekdays %in% c("Saturday", "Sunday"),
                                                 "weekend", 
                                                 "weekday")))
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 


```r
Steps_by_interval_weekday <- activity_data_replace %>% 
          group_by(interval, weekday_weekend) %>% 
          summarize(Steps=mean(steps, na.rm = TRUE))

Steps_by_interval_weekday %>%
  ggplot(aes(x=interval, y=Steps)) +
  geom_line() + 
  facet_grid(weekday_weekend~.) +
  labs(title="Average # of Steps by 5 min Interval", y = "Avg # of Steps", x = "5 min Interval")
```

![](PA1_template_files/figure-html/time series weekday-1.png)<!-- -->
