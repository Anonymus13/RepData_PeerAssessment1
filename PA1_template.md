---
title: "Reproducible Research - Project 1"
output: 
  html_document: 
    keep_md: yes
---

###Loading and preprocessing the data
Data are first loaded into __R__ by especifying the location path where it is saved. Then the dates in the original data are processed with [as.Date](https://cran.r-project.org/web/packages/date/date.pdf) function, in order to have them in the appropiate POSIX date format.


```r
##dir <- readline("Please refer to the location path of the downloaded data")
dir <- "N:/Reproducible analysis/Project1"
setwd(dir)
data <- read.csv(unz("repdata%2Fdata%2Factivity.zip", "activity.csv"))
data$date <- as.Date(data$date, format='%Y-%m-%d')
```
***
###Mean total number of steps taken per day
To compute the total number of steps taken in each day, steps are aggregated using [aggregate](https://www.rdocumentation.org/packages/stats/versions/3.4.3/topics/aggregate) function with argument _FUN_ = __"sum"__ for each date using. Then a histogram is created from this array using [hist](https://www.rdocumentation.org/packages/graphics/versions/3.4.3/topics/hist) function.  

```r
steps_day <- aggregate(data$steps, list(data$date), FUN = sum)
colnames(steps_day) <- c("date", "tot_steps")
hist(steps_day$tot_steps, breaks = 10 , main = "Distribution of total steps per day", xlab="Steps per day")
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

from the obtained array the mean and median total number of steps taken each day are computed.

```r
mean(steps_day$tot_steps, na.rm = T)
```

```
## [1] 10766.19
```

```r
median(steps_day$tot_steps, na.rm = T)
```

```
## [1] 10765
```
***
###Average daily activity pattern
In a similar code the mean number of steps taken for every interval accross each day is computed. However, this time instead of __"sum"__, __"mean"__ is used with the argument _na.rm_ as __T__ (True) to omit all "NA"" values. Then the time series is ploted using this average number of steps by interval, with the _type_ argument as __"l"__(line). 

```r
steps_interval <- aggregate(data$steps, list(data$interval), FUN = mean, na.rm = T)
colnames(steps_interval) <- c("interval", "avg_steps")
plot(steps_interval$avg_steps, type="l", main = "average steps by interval time series", xlab = "5-minute interval", ylab = "average number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

From the array obtained the interval value linked to the maximum number of steps on average is obtained.

```r
steps_interval$interval[which.max(steps_interval$avg_steps)]
```

```
## [1] 835
```
***
###Imputing missing values
First the total number of NA values is computed.

```r
sum(is.na(data))
```

```
## [1] 2304
```

Then the average number of steps obtained by interval accross all days was merged to the original data usign [merge](https://www.rdocumentation.org/packages/data.table/versions/1.10.4-2/topics/merge) funtion. In other to create a new data without "NA" values, each "NA"" value is replaced by each interval-averaged number of steps.

```r
nonNA <- merge(data, steps_interval, by = "interval")
nonNA$steps[is.na(nonNA$steps)] <- nonNA$avg_steps[is.na(nonNA$steps)]
steps_day2 <- aggregate(nonNA$steps, list(nonNA$date), FUN = sum)
colnames(steps_day2) <- colnames(steps_day)
hist(steps_day2$tot_steps, breaks = 10 , main = "Distribution of total steps per day", xlab="Steps per day")
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

Next the mean and medium steps taken each day of the original and the non-NA datasets are calculated and compared. 

```r
mean(steps_day$tot_steps, na.rm = T)
```

```
## [1] 10766.19
```

```r
mean(steps_day2$tot_steps)
```

```
## [1] 10766.19
```

```r
median(steps_day$tot_steps, na.rm = T)
```

```
## [1] 10765
```

```r
median(steps_day2$tot_steps)
```

```
## [1] 10766.19
```
***
###Differences in activity patterns between weekdays and weekends
For this final analysis the weekday is determined from the date columns and added to a new column. Then all __"Sat"__ and __"Sun"__ are labeled as __"Weekend"__ days and others as __"Weekdays"__ , updating the same column. Next, the average number of steps taken for every interval accross all weekends and weekdays is calculated and aggregated into a new dataset, _weekdays_. 


```r
data$wday <- weekdays(data$date,abbreviate = T)
data$wday <- ifelse(data$wday %in% c("Sat", "Sun"), "Weekend", "Weekday")
week_days <- aggregate(data$steps, list(data$interval, data$wday), FUN = mean, na.rm = T)
colnames(week_days) <- c("interval", "wday", "avg_steps")
```

Finally two plot panels, one for weekdays and the other for weekends are ploted using [ggplot2](https://cran.r-project.org/web/packages/ggplot2/index.html) package, to show the differences in the activity patterns.

```r
library("ggplot2")
g <- ggplot(week_days, aes(x=interval, y=avg_steps))
g + geom_line() + facet_grid(wday~.) + labs(title="average steps in weekdays and weekends") + ylab("Average steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

