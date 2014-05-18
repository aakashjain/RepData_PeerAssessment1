# Reproducible Research: Peer Assessment 1

## Loading the data

The **plyr** and **lattice** packages are used in this code.

```r
library(plyr)
library(lattice)
unzip("activity.zip")
data <- read.csv("activity.csv")
head(data)
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


## What is mean total number of steps taken per day?

The sum of steps for each day is obtained, ignoring days where steps are missing:

```r
stepsPerDay <- ddply(data[!is.na(data$steps), ], .(date), summarize, steps = sum(steps))
head(stepsPerDay)
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

The corresponding histogram:

```r
histogram(stepsPerDay$steps, xlab = "Steps", main = "Total steps per day", breaks = 10)
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 

The mean and median number of steps per day:

```r
mean(stepsPerDay$steps)
```

```
## [1] 10766
```

```r
median(stepsPerDay$steps)
```

```
## [1] 10765
```

They are very close but not equal.

## What is the average daily activity pattern?

The mean number of steps for each interval across all days is obtained, ignoring days where steps are missing:

```r
intervalAvg <- ddply(data[!is.na(data$steps), ], .(interval), summarize, steps = mean(steps))
head(intervalAvg)
```

```
##   interval   steps
## 1        0 1.71698
## 2        5 0.33962
## 3       10 0.13208
## 4       15 0.15094
## 5       20 0.07547
## 6       25 2.09434
```

The corresponding time series plot:

```r
xyplot(steps ~ interval, data = intervalAvg, type = "l", xlab = "5-minute Interval", 
    ylab = "Steps", main = "Average steps")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6.png) 

The maximum number of steps, on average, are in the following interval:

```r
intervalAvg[intervalAvg$steps == max(intervalAvg$steps), ]$interval
```

```
## [1] 835
```


## Imputing missing values

The number of rows in the dataset with number of steps missing is:

```r
sum(is.na(data$steps))
```

```
## [1] 2304
```

The average steps for each interval, calculated earlier, are substituted for days with number of steps missing.  
Notice that for each day, either none of the intervals have missing steps or all of the intervals have missing steps.  
As a result, the interval averages can be susbtituted as a whole for days with missing steps.

```r
imputed <- data
imputed[is.na(imputed$steps), ]$steps <- intervalAvg$steps
head(imputed)
```

```
##     steps       date interval
## 1 1.71698 2012-10-01        0
## 2 0.33962 2012-10-01        5
## 3 0.13208 2012-10-01       10
## 4 0.15094 2012-10-01       15
## 5 0.07547 2012-10-01       20
## 6 2.09434 2012-10-01       25
```

The histogram for the total number of steps per day using the imputed data:

```r
stepsPerDay <- ddply(imputed, .(date), summarize, steps = sum(steps))
histogram(stepsPerDay$steps, xlab = "Steps", main = "Total steps (No missing values)", 
    breaks = 10)
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10.png) 

The new mean and median:

```r
mean(stepsPerDay$steps)
```

```
## [1] 10766
```

```r
median(stepsPerDay$steps)
```

```
## [1] 10766
```

The mean remains the same as earlier, but the median increases slightly.  
Replacing several values in the dataset with interval means results in the median becoming equal to the mean.  
Likewise, the histogram also shows a noticable increase in frequency of days with total number of steps around the median.

## Are there differences in activity patterns between weekdays and weekends?

A new column is added to the imputed dataset, factoring it into weekdays and weekends. Saturday and Sunday are taken to be weekends.

```r
imputed$day <- factor(weekdays(as.Date(imputed$date)))
levels(imputed$day)
```

```
## [1] "Friday"    "Monday"    "Saturday"  "Sunday"    "Thursday"  "Tuesday"  
## [7] "Wednesday"
```

```r
levels(imputed$day) <- c("weekday", "weekday", "weekend", "weekend", "weekday", 
    "weekday", "weekday")
```

The mean number of steps for each interval interval across all weekdays and all weekends separately is obtained:

```r
weekSteps <- ddply(imputed, .(interval, day), summarize, steps = mean(steps))
head(weekSteps)
```

```
##   interval     day   steps
## 1        0 weekday 2.25115
## 2        0 weekend 0.21462
## 3        5 weekday 0.44528
## 4        5 weekend 0.04245
## 5       10 weekday 0.17317
## 6       10 weekend 0.01651
```

The corresponding time series plot:

```r
xyplot(steps ~ interval | day, data = weekSteps, type = "l", layout = c(1, 2), 
    xlab = "Interval", ylab = "Number of steps", main = "Average steps by day")
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-14.png) 

The plot shows comparitively higher activity in mornings during weekdays, and higher activity at midday and evenings during weekends.  

**END**
