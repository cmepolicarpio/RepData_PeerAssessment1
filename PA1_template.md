---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data
Load packages

```r
library(dplyr)
library(ggplot2)
library(tidyverse)
library(lattice)
```

Read Data

```r
activity <- read.csv(".\\data\\activity.csv")
activity$date <- as.POSIXct(activity$date, format = "%Y-%m-%d") #convert date
head(activity)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

```r
#get total per day
perday <- tapply(activity$steps, activity$date, sum, na.rm = TRUE)
perday <- data.frame(date=names(perday), total_steps=perday)
perday$date <- as.POSIXct(perday$date, format = "%Y-%m-%d") #convert date
head(perday)
```

```
##                  date total_steps
## 2012-10-01 2012-10-01           0
## 2012-10-02 2012-10-02         126
## 2012-10-03 2012-10-03       11352
## 2012-10-04 2012-10-04       12116
## 2012-10-05 2012-10-05       13294
## 2012-10-06 2012-10-06       15420
```

```r
#get average per interval
perinterval <- tapply(activity$steps, activity$interval, mean, na.rm = TRUE)
perinterval <- data.frame(interval=unique(activity$interval), average=perinterval)
head(perinterval)
```

```
##    interval   average
## 0         0 1.7169811
## 5         5 0.3396226
## 10       10 0.1320755
## 15       15 0.1509434
## 20       20 0.0754717
## 25       25 2.0943396
```


## What is mean total number of steps taken per day?
For this part of the assignment, you can ignore the missing values in the dataset.

1. Calculate the total number of steps taken per day

2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day

3. Calculate and report the mean and median of the total number of steps taken per day


```r
#create plot
hist(perday$total_steps, breaks = 10, 
     main = "Total Number of Steps per Day",
     xlab = "Total Steps") 
abline(v = mean(perday$total_steps), col="red", lwd = 2) #mean
abline(v = median(perday$total_steps), col = "blue", lwd = 2) #median

legend("topleft", legend=c("Mean", "Median"),
       col=c("red", "blue"), lty=1:1, cex=0.8)
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

## What is the average daily activity pattern?
1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
plot(perinterval$average ~ perinterval$interval, 
     pch = 18,
     col = "red",
     type="l",
     xaxt = "n",
     xlab = "Interval",
     ylab = "Total Steps")
axis(1, perinterval$interval, perinterval$interval, cex.axis = .7) #label x axis
max(perinterval$average)
```

```
## [1] 206.1698
```

```r
mxdate <- perinterval[perinterval$average == max(perinterval$average), 1] #get max steps
abline(v = mxdate, col = "blue", lwd = 2) #identity max steps
legend("topleft", legend=c("Max # of steps per interval"),
       col=c("blue"), lty=1, lwd = 2, cex=0.5)
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->


## Imputing missing values
Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NA|NAs)

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
#total number of missing values
nas <- !complete.cases(activity$steps)
sum(nas)
```

```
## [1] 2304
```

```r
#get mean per day
withmean <- filter(perday, !is.nan(total_steps))

#if there are dates that has no entry or all are NAs, mean=0 on that day
withoutmean <- filter(perday, is.nan(total_steps))
withoutmean <- withoutmean %>%
    mutate(total_steps = 0)
lmeans <- rbind(withmean, withoutmean)
names(lmeans) <- c("date","meanperday")

#merge activity with its mean per day
imputed <- merge(activity, lmeans, by.x = "date", by.y = "date")
complete <- complete.cases(imputed$steps)
imputed[!complete,] <- imputed[!complete, ] %>%
    mutate(steps  = meanperday)
imputed$date <- as.POSIXct(imputed$date, format = "%Y-%m-%d")
head(imputed)
```

```
##         date steps interval meanperday
## 1 2012-10-01     0        0          0
## 2 2012-10-01     0        5          0
## 3 2012-10-01     0       10          0
## 4 2012-10-01     0       15          0
## 5 2012-10-01     0       20          0
## 6 2012-10-01     0       25          0
```

```r
#get sum per day
imputedperday <- tapply(imputed$steps, imputed$date, sum)
imputedperday <- data.frame(date=names(imputedperday), total_steps=imputedperday)

#create plot
hist(imputedperday$total_steps, breaks = 10, 
     main = "Total Number of Steps per Day",
     xlab = "Total Steps") 
abline(v = mean(imputedperday$total_steps), col="red", lwd = 2) #mean
abline(v = median(imputedperday$total_steps), col = "blue", lwd = 2) #median

legend("topleft", legend=c("Mean", "Median"),
       col=c("red", "blue"), lty=1:1, cex=0.8)
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

## Are there differences in activity patterns between weekdays and weekends?
For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

```r
#tag weekday and weekend
wday <- c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")

imputed <- imputed %>%
    mutate(weekday = factor((weekdays(date) %in% wday),
                            levels = c(FALSE, TRUE), labels = c("weekend","weekday")),
           weekday.interval = paste(weekday, interval, sep="."))

#get average per weekday & interval
imputedperinterval <- tapply(imputed$steps, imputed$weekday.interval, mean)
imputedperinterval <- data.frame(weekday.interval=names(imputedperinterval), total_steps=imputedperinterval)

imputedperinterval <- imputedperinterval %>%
    separate(weekday.interval, c("weekday","interval"), sep="[.]")

#convert data type
imputedperinterval$weekday <- as.factor(imputedperinterval$weekday)
imputedperinterval$interval <- as.numeric(imputedperinterval$interval)
imputedperinterval$total_steps<- as.numeric(imputedperinterval$total_steps)

#arrange
imputedperinterval <- imputedperinterval %>%
    arrange(weekday, as.numeric(interval))

#create plot
xyplot(total_steps ~ interval | weekday,
       data = imputedperinterval,
       type = "l") 
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->
