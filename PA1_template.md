---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: TRUE
editor_options: 
  chunk_output_type: console
---



## Loading and preprocessing the data


```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(ggplot2)
library(lubridate)
```

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
library(lattice)
library(grid)

act <- read.csv("activity.csv")

for(i in seq(act$interval)){
  if (nchar(act$interval[i]) == 1){
    x <- act$interval[i]
    act$interval[i] <- paste("000",sep="", x)
  } else if(nchar(act$interval[i]) == 2){
    x <- act$interval[i]
    act$interval[i] <- paste("00", sep="", x)
  } else if(nchar(act$interval[i]) == 3){
    x <- act$interval[i]
    act$interval[i] <- paste("0", sep="", x)
  }
}

act$interval <- paste(format(as.POSIXct(strptime(act$interval,"%H%M", tz="")), format="%H:%M"))
```

## What is mean total number of steps taken per day?

1. Calculate the total number of steps taken per day

2. If you do not understand the difference between a histogram and a barplot, research the difference between       them. Make a histogram of the total number of steps taken each day


```r
act_mspd <- act %>% group_by(date) %>% summarise(sum(steps))
names(act_mspd) <- c("date", "steps")
qplot(steps, data=act_mspd)
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

```
## Warning: Removed 8 rows containing non-finite values (stat_bin).
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

3. Calculate and report the mean and median of the total number of steps taken per day

```r
mean(act_mspd$steps, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(act_mspd$steps, na.rm = TRUE)
```

```
## [1] 10765
```

## What is the average daily activity pattern?
1. Make a time series plot (i.e.type="l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
act_avs <- act %>% group_by(interval) %>% summarise(mean(steps, na.rm=TRUE))
names(act_avs) <- c("interval", "steps")
act_avs$interval <- gsub("*:", "", act_avs$interval)
act_avs$interval <- as.numeric(act_avs$interval)
ggplot(act_avs, aes(interval, steps, group=1)) + geom_line()
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->
## Imputing missing values
Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
sum(is.na(act))
```

```
## [1] 2304
```

```r
sum(is.na(act$steps))
```

```
## [1] 2304
```
2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
sum(is.na(act))
```

```
## [1] 2304
```

```r
sum(is.na(act$steps))
```

```
## [1] 2304
```

```r
act_avs <- act %>% group_by(interval) %>% summarise(mean(steps, na.rm=TRUE))
act_filled <- act
names(act_avs) <- c("interval", "steps")

for(i in seq(act_filled$steps)){
  if(is.na(act_filled$steps[i])){
    for(j in seq(act_avs$interval)){
      if(act_filled$interval[i] == act_avs$interval[j]){
        act_filled$steps[i] <- act_avs$steps[j]
      }
    }
  }
}

act_mspd <- act_filled %>% group_by(date) %>% summarise(sum(steps))
names(act_mspd) <- c("date", "steps")
qplot(steps, data=act_mspd)
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

```r
mean(act_mspd$steps, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(act_mspd$steps, na.rm = TRUE)
```

```
## [1] 10766.19
```
## Are there differences in activity patterns between weekdays and weekends?

1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.



```r
act$date <- as.Date(act$date, format="%Y-%m-%d")
act$day <- weekdays(act$date, abbreviate = TRUE)
act$day <- gsub("*Sa|*So", "Weekend", act$day)
act$day <- sub("[^(Weekend)]{1,2}", "Weekday", act$day)
```
2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
act_avs <- act %>% group_by(interval, day) %>% summarise(mean(steps, na.rm=TRUE))
names(act_avs) <- c("interval", "day", "steps")
act_avs$interval <- gsub("*:", "", act_avs$interval)
act_avs$interval <- as.numeric(act_avs$interval)
ggplot(act_avs, aes(interval, log(steps), color= day, group=1)) + geom_line() + facet_grid(day~.) + scale_x_continuous(breaks=c(500,1000,1500,2000,2500))
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

