Reproducible Research - Peer Assessment 1
=========================
***
## Summary

This is the first assignment for the Reproducible Research course in the Data Science 
Specialization, offered by Johns Hopkins Bloomberg School of Public Health. The following is an excerpt from the assignment introduction:

>This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

The data is provided from this [link](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) and was downloaded and extracted to the following directory
* ```./data/temp/repdata/``` 

relative to the working directory, with the file named as
* ```activity.csv```  

The file **PA1_template.Rmd** was created and is located in that directory path. 

The working environment used is 
* Windows 7, Service Pack 1 (64 bit)
* RStudio, version 0.98.978 (64 bit)
* R version 3.1.1 (64 bit).

## Libraries

Only one library was added for this assignment, which is the **ggplot2** graphics library:


```r
library(ggplot2)
```

All plots created in this assignment are **ggplot2** plots, consisting of histograms and time series plots.

## Loading the Data

Since the data file is a CSV file, the function ```read.csv()``` was used to read the data into R.


```r
theData <- read.csv("activity.csv", sep=",")
theData$date <- as.Date(theData$date)
```

Once read, you can see the simple structure of the data frame:


```r
str(theData)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Date, format: "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

```r
head(theData)
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

Note that the **date** variable was converted to a ```date``` class using ```as.Date```

## Answering the Questions

The assignment consists of 4 sections, each with question(s) to answer with exploratory data analysys, using the downloaded CSV file.

### I. What is mean total number of steps taken per day?


```r
histData <- tapply(theData$steps, theData$date, FUN=mean)
ggplot(as.data.frame(histData), aes(histData)) + 
    geom_histogram(binwidth=10, color="black", aes(fill= ..count..)) + 
    theme_bw() + xlab("Number of Steps") + 
    ggtitle("Total Number of Steps Each Day") + 
    theme(plot.title = element_text(face="bold"))
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 

Next, looking at the summary of the histogram data,


```r
summary(histData)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
##    0.14   30.70   37.40   37.40   46.20   73.60       8
```

we can see that the Mean and Median values are both 37.38

### II. What is the average daily activity pattern?


```r
justInt <- theData[,c(1,3)]
intAgg <- aggregate(.~interval, data=justInt, FUN=mean)

ggplot(intAgg, aes(interval, steps)) + geom_line() + 
    theme_bw() + xlab("Time - 5 min Intervals") + ylab("Average Steps Across All Days") + 
    ggtitle("Average Daily Activity Pattern") + 
    theme(plot.title = element_text(face="bold"))
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6.png) 

#### II.1. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
intAgg[intAgg$steps==max(intAgg$steps),]
```

```
##     interval steps
## 104      835 206.2
```

### III. Missing Values

In this section, we need to analyze the NA values in the data.  We can quickly see the total NA's in each variable:


```r
summary(theData)
```

```
##      steps            date               interval   
##  Min.   :  0.0   Min.   :2012-10-01   Min.   :   0  
##  1st Qu.:  0.0   1st Qu.:2012-10-16   1st Qu.: 589  
##  Median :  0.0   Median :2012-10-31   Median :1178  
##  Mean   : 37.4   Mean   :2012-10-31   Mean   :1178  
##  3rd Qu.: 12.0   3rd Qu.:2012-11-15   3rd Qu.:1766  
##  Max.   :806.0   Max.   :2012-11-30   Max.   :2355  
##  NA's   :2304
```

The summary shows that all NA's are in the **steps** variable, and the total is 2304.

In addition to NA's, we need to analyze the data to see if there are any missing days:


```r
## using the date and steps columns to aggregate the mean for each day. the mean for 
## that day will be substituted for any NA's within the same day.
justDate <- theData[,1:2]
dateAgg <- aggregate(.~ date, data=justDate, FUN=mean)
  
## check and see what days are missing using dateAgg. dateAgg results only days with 
## data that can be calculated.
noDate <- theData[!(theData$date %in% dateAgg$date),]
uniqueDate <- unique(noDate$date)
uniqueDate
```

```
## [1] "2012-10-01" "2012-10-08" "2012-11-01" "2012-11-04" "2012-11-09"
## [6] "2012-11-10" "2012-11-14" "2012-11-30"
```

```r
## make sure that the missing dates do not have data mixed with NA.  If they do not have 
## any data then they are all NA, which is the case.
nrow(theData[theData$date %in% uniqueDate & !is.na(theData$steps),])
```

```
## [1] 0
```

Since the result of this analysis shows that the missing dates do not have values mixed with NA's, we will substitute all NA values with zero (0):


```r
theData2 <- theData
theData2$steps <- sapply(theData2$steps, function(i) ifelse(is.na(i), 0, i))
```

Now that we have a new data set with all the NA values substituted with the value zero (0), we can re-plot the histogram, and get the new values for the Mean and Median:


```r
histData2 <- tapply(theData2$steps, theData2$date, FUN=mean)
  
ggplot(as.data.frame(histData2), aes(histData2)) + 
    geom_histogram(binwidth=10, color="black", aes(fill= ..count..)) + 
    theme_bw() + xlab("Number of Steps") + 
    ggtitle("Total Number of Steps Each Day") + 
    theme(plot.title = element_text(face="bold"))
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11.png) 

```r
summary(histData2)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##     0.0    23.5    36.1    32.5    44.5    73.6
```

And we can see that the median value is now 36.09, and the mean value is 32.48. Now that we have analyzed the NA values and missing dates, we can answer some questions.


```r
## plot side-by-side both histograms for comparison in answering the next 2 questions
```
#### III.1. Do these values differ from the estimates from the first part of the assignment? 


#### III.2. What is the impact of imputing missing data on the estimates of the total daily number of steps?




### IV. Are there differences in activity patterns between weekdays and weekends?


```r
##   1. Create a new factor variable in the dataset with two levels - "weekday" and 
##      "weekend" indicating whether a given date is a weekday or weekend day.
days <- weekdays(theData2$date)
theData2.1 <- cbind(theData2, days)
  
theData2.1$days <- sapply(theData2.1$days, function(i) 
    ifelse(i=="Saturday" | i=="Sunday","Weekend", "Weekday"))
  
theData2.1$days <- as.factor(theData2.1$days)
str(theData2.1)
```

```
## 'data.frame':	17568 obs. of  4 variables:
##  $ steps   : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ date    : Date, format: "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
##  $ days    : Factor w/ 2 levels "Weekday","Weekend": 1 1 1 1 1 1 1 1 1 1 ...
```

```r
summary(theData2.1)
```

```
##      steps            date               interval         days      
##  Min.   :  0.0   Min.   :2012-10-01   Min.   :   0   Weekday:12960  
##  1st Qu.:  0.0   1st Qu.:2012-10-16   1st Qu.: 589   Weekend: 4608  
##  Median :  0.0   Median :2012-10-31   Median :1178                  
##  Mean   : 32.5   Mean   :2012-10-31   Mean   :1178                  
##  3rd Qu.:  0.0   3rd Qu.:2012-11-15   3rd Qu.:1766                  
##  Max.   :806.0   Max.   :2012-11-30   Max.   :2355
```


```r
y <- aggregate(.~ interval + days, data=theData2.1, FUN=mean)

ggplot(y, aes(interval, steps)) + geom_line() + facet_grid(days~.) +
    theme_bw() + xlab("Time - 5 min Intervals") + ylab("Average Steps Across All Days")
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-14.png) 

