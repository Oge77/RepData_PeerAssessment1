---
title: "REPRODUCTIBLE RESEARCH: Peer Assigment 1"
author: "Gerard NIGNON"
date: "8 mai 2015"
output: 
    html_document:
        toc: true
---


##Introduction

It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These types of devices are part of the “quantified self” movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data is hard to obtain and there is a lack of the lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5-minute intervals throughout the day. The data consists of two months of data from an anonymous individual collected during the months of October and November 2012 and include the number of steps taken in 5-minute intervals each day.

##Data

The data for this assignment can be downloaded from the course web site:

Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) 

The variables included in this dataset are:

* __steps__: Number of steps taking in a 5-minute interval (missing values are coded as NA)

* __date__: The date on which the measurement was taken in YYYY-MM-DD format

* __interval__: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

##Loading and preprocessing the data

Loading need library:

```r
packages <- c("dplyr", "knitr", "ggplot2")
sapply(packages, require, character.only = TRUE, quietly = TRUE)
```

```
## Warning: package 'ggplot2' was built under R version 3.1.3
```

```r
library(dplyr)
library(knitr)
library(ggplot2)
```


Loading the data:


```r
fileUrl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
download.file(fileUrl, destfile = "./class/activity.zip", method = "curl")
if(!file.exists("activity.csv")){
    unzip("activity.zip")
}
dataset <- read.csv("activity.csv")
```

1. Calculating the total number of steps taken per day:


```r
dataset1 <- (dataset%>%
                 group_by(date)%>%
                 summarize(StepsPerDay = sum(steps)))
```

2. Calculating the number of steps taken per interval:


```r
dataset2 <- (dataset%>%
                 group_by(interval)%>%
                 summarize(MeanStepsPerInterval = mean(steps, na.rm = TRUE)))
```

3. Inspecting the resulting raw values of variables:


```r
glimpse(dataset)
```

```
## Observations: 17568
## Variables:
## $ steps    (int) NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N...
## $ date     (fctr) 2012-10-01, 2012-10-01, 2012-10-01, 2012-10-01, 2012...
## $ interval (int) 0, 5, 10, 15, 20, 25, 30, 35, 40, 45, 50, 55, 100, 10...
```

```r
glimpse(dataset1)
```

```
## Observations: 61
## Variables:
## $ date        (fctr) 2012-10-01, 2012-10-02, 2012-10-03, 2012-10-04, 2...
## $ StepsPerDay (int) NA, 126, 11352, 12116, 13294, 15420, 11015, NA, 12...
```

```r
glimpse(dataset2)
```

```
## Observations: 288
## Variables:
## $ interval             (int) 0, 5, 10, 15, 20, 25, 30, 35, 40, 45, 50,...
## $ MeanStepsPerInterval (dbl) 1.7169811, 0.3396226, 0.1320755, 0.150943...
```

4. Processing/transforming the data (if necessary) into a format suitable for your analysis:

Converting the dates into a POSIXlt class.


```r
date <- as.Date(dataset$date, "%Y-%m-%d")
dataset$date <- date
```

## What is mean total number of steps taken per day?

1. Plotting the number of steps taken per day:


```r
g <- ggplot(dataset1, aes(StepsPerDay)) 
g + geom_histogram(fill = "blue", colour = "white", breaks = c(0, 5000, 10000, 15000, 20000, 25000)) + labs(y = expression("frequency")) + labs(x = expression("number of steps per day")) + labs(title = expression("Number of steps per day"))
```

![plot of chunk unnamed-chunk-7](figure/unnamed-chunk-7-1.png) 

We have 28 days with over 10000 steps.

2. Calculating and reporting the mean and median of the total number of steps taken per day:
* __Mean__:

```r
Mean <- as.integer(mean(dataset1$StepsPerDay, na.rm = T))
```

* __Median__:

```r
Median <- as.integer(median(dataset1$StepsPerDay, na.rm = T))
```

Measures      | Value
------------- | ---------------------
Mean          | 10766
Median        | 10765

## What is the average daily activity pattern?


1. Plotting the number of steps taken for a five-minutes interval:


```r
g <- ggplot(dataset2, aes(x=interval, y=MeanStepsPerInterval))
g + geom_line(colour = "blue", size = 1.5) + labs(y = expression("Average number of steps")) + labs(x = expression("Interval")) + labs(title = expression("Average daily activity pattern"))
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png) 

2. Maximum number of steps for a five-minutes interval:


```r
Max <- round(max(dataset2$MeanStepsPerInterval), digits = 0)
```


```r
IntervalMax <- dataset2$interval[which.max(dataset2$MeanStepsPerInterval)]
```

Based on this data with the missing values removed, the five-minute interval "835" contains the maximum number of average steps per day: around 206.

## Imputing missing values

1. Calculating and reporting the total number of missing values in the dataset (i.e. the total number of rows with NAs):


```r
NANumber <- sum(is.na(dataset))
NApct <- mean(is.na(dataset))*100
```
There are 2304 missing values and it represents 4.3715847% of the total values of the dataset.

2. Filling in all of the missing values in the dataset:
 


```r
dataset3 <- full_join(dataset, dataset2)
```

```
## Joining by: "interval"
```

* Replacing the missing values by the mean of the five - minute interval and creating a new dataset:


```r
dataset4 <- (dataset3%>%
        mutate(stepsN = ifelse(is.na(steps), MeanStepsPerInterval, steps))%>%
        select(stepsN, date, interval))
```

## Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
dataset5 <- (dataset4%>%
                 group_by(date)%>%
                 summarize(StepsPerDayN = sum(stepsN)))
```

1. Plotting histogramme of the total number of steps taken per day:


```r
g <- ggplot(dataset5, aes(StepsPerDayN)) 
g + geom_histogram(fill = "blue", colour = "white", breaks = c(0, 5000, 10000, 15000, 20000, 25000)) + labs(y = expression("frequency")) + labs(x = expression("number of steps per day")) + labs(title = expression("Number of steps per day"))
```

![plot of chunk unnamed-chunk-17](figure/unnamed-chunk-17-1.png) 

2. Calculating and reporting the mean and median of the total number of steps taken per day:
* __Mean__:

```r
MeanN <- as.integer(mean(dataset5$StepsPerDayN))
```

* __Median__:

```r
MedianN <- as.integer(median(dataset5$StepsPerDayN))
```

Measures      | Value
------------- | ---------------------
Mean          | 10766
Median        | 10766

Mean value is the same and the median is a negligibly lower.

3. Looking for differences in activity patterns between weekdays and weekends

I have to change the setting because I live outside of the United States


```r
Sys.setlocale("LC_TIME", "en_US.UTF-8")
```

```
## [1] "en_US.UTF-8"
```

4. Creating a dataset with the filled-in missing values for this part and a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.


```r
dataset6 <- (dataset5%>%
                 group_by(date)%>%
                 summarize(DateWeekday = weekdays(date))) 
```


```r
dataset7 <- full_join(dataset5, dataset6)
```

```
## Joining by: "date"
```


```r
dataset8 <- (dataset4%>%
                mutate(DateWeekdayN = ifelse(weekdays(date) == "Saturday" | weekdays(date) == "Sunday", "weekend", "weekday"))%>%
                group_by(interval, DateWeekdayN)%>%
                summarize(MeanStepsPerIntervalN = mean(stepsN)))
```

5. Plotting the average number of steps in a five-minute interval across all weekday days or weekend days.


```r
g <- ggplot(dataset8, aes(x = interval, y = MeanStepsPerIntervalN)) 
g + geom_line(aes(color = DateWeekdayN), size = 1.5) + facet_wrap(~ DateWeekdayN, nrow=2) + labs(y = expression("Average number of steps")) + labs(x = expression("Interval")) + labs(title = expression("Average daily activity pattern")) + scale_color_discrete(name = "Day")
```

![plot of chunk unnamed-chunk-24](figure/unnamed-chunk-24-1.png) 
