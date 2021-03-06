Reproducible Research: Peer Assessment 1
========================================


```r
# ensure that packages are available
library("knitr")
library("lubridate")
library("ggplot2")
library("plyr")
# set global options
opts_chunk$set(echo = TRUE)
```

- Used MacBook Pro, MacOS 10.8.5
- Intel Core i7 Processor
- 8 GB RAM
- R 3.1.0 (2014-4-10) -- "Spring Dance"
- RStudio 0.98.976

### Data are obtained from repository 'rdpeng/RepData_PeerAssessment1'  
- No original data source is referenced in README file there

### Load and preprocess data

```r
setwd("~/Documents/R/repR/PA1")
activity <- read.csv("activity.csv",header=TRUE)
# interpret date and extract year, month, day
activity$date <- ymd(activity$date)
activity$year <- year(activity$date)
activity$month <- month(activity$date)
activity$day <- day(activity$date)
```

### Summarize total number of steps taken per day

```r
# sum steps by date
activByDay <- aggregate(steps ~ date, activity, sum)
# show histogram and summary statistics
hist(activByDay$steps, breaks = 20,
    main = "Activity Level for October/November",
    xlab = "Total Steps per Day")
```

![plot of chunk daySteps](figure/daySteps.png) 

```r
meanSteps <- mean(activByDay$steps) 
medianSteps <- median(activByDay$steps)
```

- The mean of steps per day is 10766.188679  
- The median of steps per day is 10765  

### Find average daily activity pattern


```r
# create function to compute and display daily activity pattern
dailyAct <- function(df, plotTitle) {
  # convert to interval (time of day) 4-characters, and average steps by these
  df$time <- sprintf("%04d",df$interval)
  activByInterv <- aggregate(steps ~ time, df, mean)
  # create POSIT version of time
  activByInterv$hrMin <- strptime((activByInterv$time), "%H%M")
  # draw line plot, using decimal hours on x-scale for continuity
  activByInterv$decHr <- hour(activByInterv$hrMin) + 
                          minute(activByInterv$hrMin)/60
  p <- ggplot(activByInterv, aes(decHr, steps))
  p <- p + ggtitle(plotTitle) +
    geom_line() + theme_bw() +
    xlab("Hours of Day, with 5-Minute Interval Granularity") +
    scale_x_continuous(breaks = c(0,4,8,12,16,20,24)) +
    ylab("Average Number of Steps")
  print(p)
  # determine which interval had most activity, and return data
  # (because this is combined with plot in a function, need to return data)
  bestAct <- numeric(length = 3)
  names(bestAct) <- c("steps", "timeHr", "timeMin")
  bestAct["steps"] <- activByInterv[activByInterv$steps ==
                                  max(activByInterv$steps), 2]
  bestAct["timeHr"] <- hour(activByInterv[activByInterv$steps ==
                                  max(activByInterv$steps), 3])         
  bestAct["timeMin"] <- minute(activByInterv[activByInterv$steps ==
                                  max(activByInterv$steps), 3])
  return(bestAct)
}
```


```r
# call function to process daily activity data
bestAct <- dailyAct(activity, "Steps by Time of Day")
```

![plot of chunk dailyActiv](figure/dailyActiv.png) 

- The most active 5-minute time interval during the day starts at 
8:35, with average steps = 206.17  

### Impute missing values

```r
missing <- colSums(is.na(activity[,1:3]))
# use averages by interval for imputing missing values
activImp <- activity
activImp$steps[is.na(activImp$steps)] <- 
  ave(activImp$steps, as.factor(activImp$interval),
      FUN = function(x)
        mean(x, na.rm = TRUE))[c(which(is.na(activImp$steps)))]
# call function to reproduce daily activity, but with imputed data
bestActImp <- dailyAct(activImp, "With Imputed Data, Steps by Time of Day")
```

![plot of chunk impute](figure/impute.png) 

```r
# compute aggregates by day with imputed data
activByDayImp <- aggregate(steps ~ date, activImp, sum, na.action = na.omit)
meanStepsImp <- mean(activByDayImp$steps)
medianStepsImp <- median(activByDayImp$steps)
```

- The number of missing values for  
   'date' is 0  
   'interval' is 0  
   'steps' is 2304  
- With imputation:  
   The mean of steps per day is 10766.188679   
   The median of steps per day is 10765  

- This data imputation method yields identical numerical results for daily summaries as the data without imputation.
- This is because the imputation method, by time of day, is aligned with the aggregation; if we fill in missing values with averages from existing data, those averages are maintained.
- The data imputation was tested by temporary insertion of many NA values across different portions of different days (code and data not shown here).

### Determine differences in activity patterns between weekdays and weekends

```r
#activity$interval <- activity$interval
activity$dayOfWeek <- wday(activity$date)
activity$dayType <- as.factor(mapvalues(activity$dayOfWeek, from = c(1:7),
  to = c("weekend","weekday","weekday","weekday","weekday","weekday","weekend")
  ))
activity$time <- as.character(sprintf("%04d",activity$interval))
actByType <- aggregate(steps ~ time+dayType, activity, mean, na.rm = TRUE)
actByType$hrMin <- strptime((actByType$time), "%H%M")

# draw facet line plots, using decimal hours on x-scale for continuity
actByType$decHr <- hour(actByType$hrMin) + 
                          minute(actByType$hrMin)/60
p <- ggplot(actByType, aes(x = decHr, y = steps)) +
  geom_line() + theme_bw() + ggtitle("Daily Activity Patterns by Weekday & Weekend") +
  xlab("Hours of Day, with 5-Minute Interval Granularity") +
  scale_x_continuous(breaks = c(0,4,8,12,16,20,24)) +
  ylab("Average Number of Steps") +
  facet_grid(dayType ~ .)
print(p)
```

![plot of chunk dayOfWeek](figure/dayOfWeek.png) 

- There are noticeable differences between weekday and weekend activity patterns  
- The weekend times are generally shifted later in the day
