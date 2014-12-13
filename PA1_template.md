# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

#### Reading in the dataset


```r
activityData <- read.csv("./activity.csv")
```
#### Processing the data
This changes the date to a POSIX format.

```r
activityData$date <- as.POSIXct(activityData$date)
```


## What is mean total number of steps taken per day?

### Histogram 

This histogram shows the total number of steps taken each day


```r
dailySteps <- with(activityData, by(steps, date, function(x) sum(x, na.rm = TRUE)))
hist(dailySteps, xlab = "Steps per Day", main = NULL, breaks = 20)
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png) 

### Mean and median number of steps taken each day
#### Mean

```r
mean(dailySteps, na.rm = TRUE)
```

```
## [1] 9354.23
```
#### Median

```r
median(dailySteps, na.rm = TRUE)
```

```
## [1] 10395
```


## What is the average daily activity pattern?

### Time series plot 

This time series plot shows the average number of steps taken (averaged across all days) versus the 5-minute intervals

```r
intervalMeanSteps <- with(activityData, by(steps, interval, function(x) mean(x, na.rm = TRUE)))
plot(unique(activityData$interval), intervalMeanSteps, type = "l", ylab = "5-minute intervals", xlab = "Average number of steps taken")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png) 

### Interval with maximum steps

This shows   the 5-minute interval that, on average, contains the maximum number of steps

```r
with(activityData, unique(interval)[intervalMeanSteps == max(intervalMeanSteps)])
```

```
## [1] 835
```


## Imputing missing values

#### Number of missing values in the dataset


```r
sum(is.na(activityData$steps))
```

```
## [1] 2304
```

#### Strategy for filling in missing values in the dataset. 

I used the mean steps for the 5-minute interval associated with the missing interval.

#### New dataset with missing data filled in.



```r
estimatedData <- activityData
sum(is.na(estimatedData$steps))
```

```
## [1] 2304
```

```r
meanStepVector <- rep(as.vector(intervalMeanSteps), length = length(estimatedData$steps))
estimatedData$steps <- replace(estimatedData$steps, is.na(estimatedData$steps), meanStepVector[is.na(estimatedData$steps)])
```

### Histogram etc. after missing values filled in
#### Histogram

```r
estDailySteps <- with(estimatedData, by(steps, date, function(x) sum(x, na.rm = TRUE)))
hist(estDailySteps, xlab = "Steps per Day", main = NULL, breaks = 20)
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png) 

#### Mean total steps 


```r
mean(estDailySteps, na.rm = TRUE)
```

```
## [1] 10766.19
```
#### Median total steps

```r
median(estDailySteps, na.rm = TRUE)
```

```
## [1] 10766.19
```
#### Impact of imputing missing data
The imputed data appears to increase the mean and median number of steps. 


## Are there differences in activity patterns between weekdays and weekends?


### Panel plot comparing weekday and weekend steps

The following panel plot compares the average number of steps taken per 5-minute interval across weekdays and weekends

```r
dayFactorData <- data.frame(activityData, weekend = weekdays(activityData$date) %in% c("Saturday", "Sunday"))
par(mfrow=c(1,2))
weekendMeanSteps <- with(subset(dayFactorData, weekend == TRUE), by(steps, interval, function(x) mean(x, na.rm = TRUE)))
plot(unique(dayFactorData$interval), weekendMeanSteps, type = "l", ylab = "weekend", ylim = c(0, 250), xlab = "Average steps per 5-minute interval")
weekdayMeanSteps <- with(subset(dayFactorData, weekend == FALSE), by(steps, interval, function(x) mean(x, na.rm = TRUE)))
plot(unique(dayFactorData$interval), weekdayMeanSteps, type = "l", ylab = "weekday", xlab = "Average steps per 5-minute interval", ylim = c(0, 250))
```

![](PA1_template_files/figure-html/unnamed-chunk-13-1.png) 
