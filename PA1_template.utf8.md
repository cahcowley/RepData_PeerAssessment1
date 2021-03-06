---
title: "Coursera - Reproducible Research Course Project 1"
author: "Charl Arthur Henry Cowley"
date: "May 18, 2017"
output:
  html_document: default
  pdf_document: default
  word_document: default
---
## Loading and preprocessing the data

1.Load the data using read.csv

```r
activity <- read.csv("C:activity.csv",header = TRUE)
activity$date <- as.POSIXct(activity$date)
class(activity$date)
```

```
## [1] "POSIXct" "POSIXt"
```

library(dplyr)
## What is mean total number of steps taken per day?
1. Calculate the total number of steps taken per day

```r
daymean <- aggregate(steps ~ date, data = activity, sum)
```
If you do not understand the difference between a histogram and a barplot, research the difference between them. 
2. Make a histogram of the total number of steps taken each day

```r
par(mfrow=c(1,1))
hist(daymean$steps,col="red",xlab="Date",ylab="Steps",main="Total number of steps per day")
```

<img src="PA1_template_files/figure-html/unnamed-chunk-3-1.png" width="672" />
3. Calculate and report the mean and median of the total number of steps taken per day

```r
h <- summary(daymean$steps)
h[c("Median","Mean")]
```

```
## Median   Mean 
##  10760  10770
```

## What is the average daily activity pattern?
2. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken,
   averaged across all days (y-axis)

```r
intmean <- aggregate(steps~interval,data=activity,mean)
plot(intmean$interval,intmean$steps,type="l",xlab="Interval",col="purple",ylab="Steps",main="Ave number of steps taken over intervals")
```

<img src="PA1_template_files/figure-html/unnamed-chunk-5-1.png" width="672" />
3. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
intmean[which(intmean$steps == max(intmean$steps)),]
```

```
##     interval    steps
## 104      835 206.1698
```
## Imputing missing values
1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
summary(is.na(activity$steps))
```

```
##    Mode   FALSE    TRUE    NA's 
## logical   15264    2304       0
```

```r
summary(activity$steps)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
##    0.00    0.00    0.00   37.38   12.00  806.00    2304
```
2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. 

I will impute the NAs with the mean of the interval it falls in.

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
library(dplyr)
df1 <- activity %>%
        group_by(interval) %>%
        summarise(avg=mean(steps,na.rm=TRUE)) %>%
        merge(activity, ., all.x=TRUE) %>%
        mutate(steps=ifelse(is.na(steps)==TRUE,avg, steps))%>%
        select(-avg)
```
4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number
   of steps taken per day. 
   - Do these values differ from the estimates from the first part of the assignment? *Yes*
   - What is the impact of imputing missing data on the estimates of the total daily number of steps? *It increases the estimates*

```r
daymean2 <- aggregate(steps ~ date, data = df1, sum)
par(mfrow=c(1,1))
hist(daymean2$steps,col="green",xlab="# Steps",ylab="Frequency",main="Total number of steps")
hist(daymean$steps,col="red",xlab="# Steps",ylab="Frequency",add=TRUE)
legend("topright",c("Imputed","NAs"),col=c("green","red"),lwd=3)
```

<img src="PA1_template_files/figure-html/unnamed-chunk-9-1.png" width="672" />


## Are there differences in activity patterns between weekdays and weekends?

1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" 
   indicating whether a given date is a weekday or weekend day.

```r
weekend <- c("Friday","Saturday","Sunday")
activity$typeday <- factor((weekdays(activity$date) %in% weekend),levels=c(FALSE,TRUE),labels=c("Weekday","Weekend"))
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number
   of steps taken, averaged across all weekday days or weekend days (y-axis).

```r
library(lattice)
step_int <- aggregate(steps ~ interval + typeday,data = activity, mean)

xyplot(step_int$steps ~ step_int$interval | step_int$typeday, main = "Mean Steps (daily by interval)",xlab="Interval",ylab="Steps",layout=c(1,2),type="l")
```

<img src="PA1_template_files/figure-html/unnamed-chunk-11-1.png" width="672" />

**THE END**


