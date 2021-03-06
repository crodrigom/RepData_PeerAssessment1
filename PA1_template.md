# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

Reading the data:

```r
unzip('activity.zip', exdir = "./data")
activities <- read.csv('./data/activity.csv', header=TRUE, sep = ',')
```

Process/transform the data (if necessary) into a format suitable for your analysis

Changing the data format from factor to data

```r
activities$date <- as.Date( activities$date, format = '%Y-%m-%d' )
```

We decide to sumarize the data by the number of steps by day

```r
library(plyr)
```

```
## Warning: package 'plyr' was built under R version 3.2.1
```

```r
steps_by_day = ddply(activities, .(date), summarize, steps_by_day = sum(steps))
```

## What is mean total number of steps taken per day?

For this part of the assignment, we ignore the missing values in the dataset.

Making a histogram of the total number of steps taken each day:

```r
library(ggplot2)

to.plot <- ggplot(steps_by_day,aes(x = steps_by_day)) + 
        geom_histogram() + 
        ggtitle("Histogram of total steps per day") +
        xlab("Number of steps made per day") +
        ylab("Frequency")
print(to.plot)
```

```
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust this.
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png) 

Calculate and report the mean and median total number of steps taken per day

```r
mean(steps_by_day$steps_by_day, na.rm=TRUE)
```

```
## [1] 10766.19
```

```r
median(steps_by_day$steps_by_day, na.rm=TRUE)
```

```
## [1] 10765
```

## What is the average daily activity pattern?

Making a time series plot (i.e. type = "l") of the 5-minute interval (x-axis)
and the average number of steps taken, averaged across all days (y-axis)


```r
steps_mean_by_day <- aggregate(x=list(avg.steps=activities$steps), by=list(interval=activities$interval), 
                               FUN=mean, na.rm=TRUE)

plot(steps_mean_by_day$avg.steps, type="l", main="Average across all days based on intervals", 
     xlab="Intervals", ylab="Steps Averaged")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png) 

Which 5-minute interval, on average across all the days in the dataset,
contains the maximum number of steps?

```r
my_max = steps_mean_by_day[which.max(steps_mean_by_day$avg.steps),]
my_max
```

```
##     interval avg.steps
## 104      835  206.1698
```

```r
paste("The interval with maximum number of steps is:", my_max$interval)
```

```
## [1] "The interval with maximum number of steps is: 835"
```

## Imputing missing values

Calculating and reporting the total number of missing values in the dataset
First we report the numbers of NAs in all data frame
Second we show the number of completa cases, the rows that contains NAs and the rows that do not.

```r
table(is.na(activities))
```

```
## 
## FALSE  TRUE 
## 50400  2304
```

```r
table(complete.cases(activities))
```

```
## 
## FALSE  TRUE 
##  2304 15264
```

Filling in all of the missing values in the dataset. The strategy 
is to exchange the missing values with the mean values for intervals.
We created a new dataset with the missing data filled in

```r
new.activities = activities
filling_values = steps_mean_by_day$avg.steps[match(new.activities$interval, steps_mean_by_day$interval)]
new.activities$steps[is.na(new.activities$steps)] = filling_values[is.na(activities$steps)]
```

Making a histogram of the total number of steps taken each day and Calculating
and reporting the mean and median total number of steps taken per day. 

```r
new.steps_by_day <- ddply(new.activities, .(date), summarize, steps_by_day = sum(steps))

to.plot.new <- ggplot(new.steps_by_day,aes(x = steps_by_day)) + 
        geom_histogram() + 
        ggtitle("Histogram of total steps per day") +
        xlab("Number of steps made per day") +
        ylab("Frequency")
print(to.plot.new)
```

```
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust this.
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png) 

Now we are able to awnser the following questions:
- Do these values differ from the estimates from the first part of the assignment?
- What is the impact of imputing missing data on the estimates of the total
daily number of steps?

```r
comparing = matrix(c(
        mean(steps_by_day$steps_by_day, na.rm=TRUE),
        median(steps_by_day$steps_by_day, na.rm=TRUE),
        mean(new.steps_by_day$steps_by_day, na.rm=TRUE),
        median(new.steps_by_day$steps_by_day, na.rm=TRUE)),
        ncol = 2)
colnames(comparing) <- c("With NAs", "Without NAs")
rownames(comparing) <- c("Mean", "Median")
```
The mean are the same and the median changed a little bit, but nothing significant.

## Are there differences in activity patterns between weekdays and weekends?

Creating a new factor variable in the dataset with two levels - "weekday"
and "weekend" indicating whether a given date is a weekday or weekend
day.

First we create a new variable with the day of the week

```r
activities$day = as.factor(weekdays(activities$date))
summary(activities$day)
```

```
##    Friday    Monday  Saturday    Sunday  Thursday   Tuesday Wednesday 
##      2592      2592      2304      2304      2592      2592      2592
```

Then we create another variable with the weekday and weekend

```r
days.of.week <- weekdays(x=as.Date(seq(7), origin="1950-01-01"))
weekday = days.of.week[!days.of.week %in% c("Saturday", "Sunday")]

activities$days.of.week =  ifelse(activities$day %in% weekday, "weekday", "weekend")
```

Making a panel plot containing a time series plot (i.e. type = "l") of the
5-minute interval (x-axis) and the average number of steps taken, averaged
across all weekday days or weekend days (y-axis)

```r
steps_by_weekday <- ddply(activities, .(interval, days.of.week), 
                          summarize, avg_steps_by_day = mean(steps, na.rm = TRUE))

to.plot.avg <- ggplot(steps_by_weekday,aes(interval, avg_steps_by_day)) + 
        geom_line() + 
        facet_grid(days.of.week ~.) +
        ggtitle("Average number of steps taken across weekday and weekend") +
        xlab("Number of steps made per day") +
        ylab("Frequency")
print(to.plot.avg)
```

![](PA1_template_files/figure-html/unnamed-chunk-14-1.png) 


