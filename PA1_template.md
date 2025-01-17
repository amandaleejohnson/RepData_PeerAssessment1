---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---

## Introduction

It is now possible to collect a large amount of data about personal movement using activity
monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are
part of the "quantified self" movement - a group of enthusiasts who take measurements about
themselves regularly to improve their health, to find patterns in their behavior, or because
they are tech geeks. But these data remain under-utilized both because the raw data are hard
to obtain and there is a lack of statistical methods and software for processing and
interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device
collects data at 5 minute intervals through out the day. The data consists of two months of
data from an anonymous individual collected during the months of October and November, 2012
and include the number of steps taken in 5 minute intervals each day.

## Loading and preprocessing the data

```
## 
## Attaching package: 'lubridate'
```

```
## The following object is masked from 'package:base':
## 
##     date
```


```r
activity = read.csv("./data/activity.csv")
#Make sure the format of the date var is in fact, in date format:
        activity$date = as.Date(activity$date)
        activity$day = day(activity$date)
        activity$month = month(activity$date)
        str(activity)
```

```
## 'data.frame':	17568 obs. of  5 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Date, format: "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
##  $ day     : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ month   : num  10 10 10 10 10 10 10 10 10 10 ...
```
        


## What is mean total number of steps taken per day?

```r
# Make a histogram of the total number of steps taken each day
sumfun <- function(x, ...){
        c(s=sum(x, na.rm=TRUE, ...))
}
        
sum_steps = summaryBy(steps ~ date, data=activity, FUN=sumfun )        
steps_per_day = aggregate(steps ~ date, activity, sum)
colnames(steps_per_day) = c("date", "steps")
head(steps_per_day)
```

```
##         date steps
## 1 2012-10-02   126
## 2 2012-10-03 11352
## 3 2012-10-04 12116
## 4 2012-10-05 13294
## 5 2012-10-06 15420
## 6 2012-10-07 11015
```

```r
ggplot(steps_per_day, aes(x = steps)) +
geom_histogram(fill = "blue", binwidth = 1000) +
labs(title = "Histogram of daily step count in October & November 2012") + 
labs(x = "Total number of steps per day", y = "Number of times in a day (count)") +
theme_bw()
```

![](PA1_template_files/figure-html/mean_sumsteps-1.png)<!-- -->

```r
# Calculate and report the mean and median of the total number of steps taken per day
        mean_steps = round(mean(steps_per_day$steps, na.rm = TRUE), digits = 0)
        median_steps = round(median(steps_per_day$steps, na.rm = TRUE), digits = 0)
```

The mean total number of steps taken per day is **10766 steps** and the median total number of steps taken per day is **10765 steps**. *Number of steps are rounded to the nearest integer.*

## What is the average daily activity pattern?

```r
# Make a time series plot (i.e. type = "l") of 
# the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
        #Step 1 - Create a data frame that summarizes the mean # of steps taken at each interval, across all days
        meanfun <- function(x, ...){
                c(m=mean(x, na.rm=TRUE, ...))
        }
        mean_steps_by_interval = summaryBy(steps ~ interval, data = activity, FUN=meanfun)  
        by_interval = merge(activity, mean_steps_by_interval, by.x="interval")
        
        #Step 2 - plot it!
        ggplot(data = by_interval, aes(x = interval, y = steps.m)) +
                geom_line(stat = "identity", color = "#00AFBB", size = 1) + 
                labs(title = "Average step count by time interval across full study period") + 
                labs(x = "Time interval", y = "Mean step count")
```

![](PA1_template_files/figure-html/average_daily-1.png)<!-- -->

```r
# Which 5-minute interval, on average across all the days in the dataset, contains the maximum
# number of steps?
        #table(by_interval$interval, by_interval$steps.m)
        max(by_interval$steps.m)
```

```
## [1] 206.1698
```

```r
        # sort by steps.m (descending)
        sorted_by_interval = by_interval[order(-by_interval$steps.m),]
        
        #Display the 1st row, which corresponds to the highest mean step count
        sorted_by_interval[1,]
```

```
##      interval steps       date day month  steps.m
## 6284      835    NA 2012-10-08   8    10 206.1698
```

```r
        #Display only the interval associated with the highest mean step count
        max_interval = round(sorted_by_interval[1, 1], digits = 0)
        max_steps = round(sorted_by_interval[1, 6], digits = 0)
```

The 5-minute interval that contains the maximum number of steps (**206**) on average across all days in the dataset is the **835th** interval. *Number of steps are rounded to the nearest integer.*

## Imputing missing values

```r
# Note that there are a number of days/intervals where there are missing values 
# (coded as NA). The presence of missing days may introduce bias 
# into some calculations or summaries of the data.

# Calculate and report the total number of missing values in the dataset 
# (i.e. the total number of rows with NAs)
        #How many missing values are in the steps column?
        num_miss = sum(is.na(activity$steps))
        
# Devise a strategy for filling in all of the missing values in the dataset. The strategy does not 
# need to be sophisticated. For example, you could use the mean/median for that day, or the mean 
# for that 5-minute interval, etc.
        #I'm going to assume that individuals are more consistent throughout the course of each day
        #Rather than consistent within a given day. 
        #So, I'm going to impute based on the mean # of steps for the given interval
        
# Create a new dataset that is equal to the original dataset but with the missing data filled in.
        #by_interval$steps_nomiss=by_interval$steps
        #by_interval$steps_nomiss[is.na(by_interval$steps)] = by_interval$steps.m
                #This is the same as stata language "replace steps=steps.m if steps=="NA"
        #The above code gives you a warning message, the below code is better:
        by_interval$steps_nomiss = ifelse(is.na(by_interval$steps), by_interval$steps.m, by_interval$steps)
        
# Make a histogram of the total number of steps taken each day and Calculate and report the mean and 
# median total number of steps taken per day. Do these values differ from the estimates from the 
# first part of the assignment? What is the impact of imputing missing data on the estimates of 
# the total daily number of steps?
        #Step 1: Recreate the total # of steps each day using the imputed step variable
                sum_steps_nomiss = summaryBy(steps_nomiss ~ date, data=by_interval, FUN=sumfun )
        #Step 2:Plot it!
                ggplot(sum_steps_nomiss, aes(x = steps_nomiss.s)) +
                                geom_histogram(fill = "blue", binwidth = 1000) +
                                labs(title = "Histogram of daily step count in October & November 2012") + 
                                labs(x = "Total number of steps per day", y = "Number of times in a day (count)")
```

![](PA1_template_files/figure-html/impute-1.png)<!-- -->

```r
        #Step 3: Calculate and report the mean and median of total number of steps taken per day
                summary(sum_steps_nomiss)
```

```
##       date            steps_nomiss.s 
##  Min.   :2012-10-01   Min.   :   41  
##  1st Qu.:2012-10-16   1st Qu.: 9819  
##  Median :2012-10-31   Median :10766  
##  Mean   :2012-10-31   Mean   :10766  
##  3rd Qu.:2012-11-15   3rd Qu.:12811  
##  Max.   :2012-11-30   Max.   :21194
```

```r
        #Step 5: What is the impact of imputing missing data on the estimates of the total daily number of steps?        
                summary(steps_per_day)
```

```
##       date                steps      
##  Min.   :2012-10-02   Min.   :   41  
##  1st Qu.:2012-10-16   1st Qu.: 8841  
##  Median :2012-10-29   Median :10765  
##  Mean   :2012-10-30   Mean   :10766  
##  3rd Qu.:2012-11-16   3rd Qu.:13294  
##  Max.   :2012-11-29   Max.   :21194
```

```r
                summary(sum_steps_nomiss)
```

```
##       date            steps_nomiss.s 
##  Min.   :2012-10-01   Min.   :   41  
##  1st Qu.:2012-10-16   1st Qu.: 9819  
##  Median :2012-10-31   Median :10766  
##  Mean   :2012-10-31   Mean   :10766  
##  3rd Qu.:2012-11-15   3rd Qu.:12811  
##  Max.   :2012-11-30   Max.   :21194
```

```r
        mean_steps_nomiss = round(mean(sum_steps_nomiss$steps_nomiss.s, na.rm = TRUE), digits = 0)
        median_steps_nomiss = round(median(sum_steps_nomiss$steps_nomiss.s, na.rm = TRUE), digits = 0)
```
The total number of missing values in the dataset is **2304**. The strategy I used to impute the missing values is based on the assumption that individuals have more consistent step counts based on the time of day rather than the day itself. Therefore, I'm going to impute based on the average number of steps by interval rather than the average number of steps by day. The median and median total number of steps taken per day differ when evaluating the non-imputed data set and the imputed data set. The mean and median total number of steps taken per day in the imputed data set are higher (**10766** and **10766**) than the mean and median total number of steps taken per day in the non-imputed data set (**10766** and **10765**). *Number of steps are rounded to the nearest integer.*

## Are there differences in activity patterns between weekdays and weekends?

```r
# For this part the weekdays() function may be of 
# some help here. Use the dataset with the filled-in missing values for this part.

# Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating 
# whether a given date is a weekday or weekend day.
        by_interval$dayofweek = wday(by_interval$date, label = TRUE)
        by_interval$daytype = ifelse(by_interval$dayofweek=="Sat" | by_interval$dayofweek=="Sun", "weekend", "weekday")
        
                
# Make a panel plot containing a time series plot (i.e. type = "l") 
# of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday 
# days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what
# this plot should look like using simulated data.
        
#Step 1 - Calculate the mean of total number of steps taken at each interval with IMPUTED DATA
#when the weektype == "weekend" and separately, when the weektype=="weekday"! 
        mean_steps_intervalweekday_nomiss = summaryBy(steps_nomiss ~ interval, data=subset(by_interval, daytype=="weekday"), FUN=meanfun)
        mean_steps_intervalweekend_nomiss = summaryBy(steps_nomiss ~ interval, data=subset(by_interval, daytype=="weekend"), FUN=meanfun)

        mean_steps_intervalweekday_nomiss$dayofweek="weekday"
        mean_steps_intervalweekend_nomiss$dayofweek="weekend"
        mean_steps_intervalweekday_nomiss$steps_nomiss.m=as.integer(mean_steps_intervalweekday_nomiss$steps_nomiss.m)
        mean_steps_intervalweekend_nomiss$steps_nomiss.m=as.integer(mean_steps_intervalweekend_nomiss$steps_nomiss.m)
        
        mean_steps_interval_bydaytype=rbind(mean_steps_intervalweekday_nomiss, mean_steps_intervalweekend_nomiss)
```

The panel plot is below:

```r
#Step 2 - Create a stacked panel plot
ggplot(data = mean_steps_interval_bydaytype, aes(x = interval, y = steps_nomiss.m)) +
        geom_line(stat="identity", color = "#00AFBB", size = 1) +
        facet_wrap(~dayofweek, ncol = 1) +
        labs(title = "Average step count by time interval across full study period") + 
        labs(x = "Time interval", y = "Mean step count")
```

![](PA1_template_files/figure-html/stackedpanel-1.png)<!-- -->

Yes, there are differences in activity patterns between weekdays and weekends. Using the imputed dataset, the figure displays how individuals are more active earlier in the day on weekdays, specifically around the 500-750 interval range. The largest spike between the two graphs is captured on the weekday figure, around the 800-900 interval range. We can also see that after the 1000 interval, individuals report a smaller average number of steps taken on weekdays than on weekends until around the 1800 interval. The two largest spikes in the weekday plot may correspond to individuals commuting on foot/bike to and from work. The multitude of spikes in the weekend plot may represent how individuals are more likely to be active throughout the course of the day when they are not at work. 
