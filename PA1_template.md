# Coursera Peer Assesment 1 Project


### Introduction

It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the "quantified self" movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

### Data

The data for this assignment can be downloaded from the course web site:

Dataset: Activity monitoring data [52K]
The variables included in this dataset are:

- steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)

- date: The date on which the measurement was taken in YYYY-MM-DD format

- interval: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

### Assignment

This assignment will be described in multiple parts. You will need to write a report that answers the questions detailed below. Ultimately, you will need to complete the entire assignment in a single R markdown document that can be processed by knitr and be transformed into an HTML file.

Throughout your report make sure you always include the code that you used to generate the output you present. When writing code chunks in the R markdown document, always use echo = TRUE so that someone else will be able to read the code. This assignment will be evaluated via peer assessment so it is essential that your peer evaluators be able to review the code for your analysis.

For the plotting aspects of this assignment, feel free to use any plotting system in R (i.e., base, lattice, ggplot2)

Fork/clone the GitHub repository created for this assignment. You will submit this assignment by pushing your completed files into your forked repository on GitHub. The assignment submission will consist of the URL to your GitHub repository and the SHA-1 commit ID for your repository state.

NOTE: The GitHub repository also contains the dataset for the assignment so you do not have to download the data separately.

#### Loading and preprocessing the data

Show any code that is needed to

1. Load the data (i.e. read.csv())

    
    ```r
    activity <- read.csv(file = "./data/activity.csv", header = TRUE)
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



2. Process/transform the data (if necessary) into a format suitable for your analysis
    
    ```r
    # Convert character column date into date format
    activity$date <- as.Date(activity$date)
    ```



#### What is mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in the dataset.

1. Make a histogram of the total number of steps taken each day 

    
    ```r
    steps_per_day <- aggregate(activity$steps, by = list(activity$date), sum, na.rm = TRUE)
    
    colnames(steps_per_day) <- c("date", "no_of_steps")
    
    # steps_per_day$date <- as.Date(steps_per_day$date)
    
    hist(steps_per_day$no_of_steps, col = "blue", main = "Histogram of sum of steps per day", 
        ylab = "percentage of days", xlab = "no of steps", ylim = c(0, 30))
    ```
    
    ![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 



2. Calculate and report the mean and median total number of steps taken per day
   
   
   ```r
   mean_steps <- mean(steps_per_day$no_of_steps, na.rm = TRUE)
   # aggregate(activity$steps, by = list(activity$date), mean, na.rm = TRUE)
   median_steps <- median(steps_per_day$no_of_steps, na.rm = TRUE)
   # aggregate(activity$steps, by = list(activity$date), median, na.rm = TRUE)
   cat("Mean steps = ", mean_steps, "; Median steps = ", median_steps)
   ```
   
   ```
   ## Mean steps =  9354 ; Median steps =  10395
   ```



#### What is the average daily activity pattern?

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)  

    
    ```r
    mean_steps_per_interval <- aggregate(activity$steps, by = list(activity$interval), 
        mean, na.rm = TRUE)
    
    colnames(mean_steps_per_interval) <- c("interval", "mean_steps")
    
    plot(x = mean_steps_per_interval$interval, y = mean_steps_per_interval$mean_steps, 
        type = "l", lwd = 2, col = "blue", main = "Mean steps per interval", ylab = "mean steps", 
        xlab = "interval")
    ```
    
    ![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 



2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?  
    
    
    ```r
    head(mean_steps_per_interval[order(mean_steps_per_interval$mean_steps, decreasing = TRUE), 
        ], 1)
    ```
    
    ```
    ##     interval mean_steps
    ## 104      835      206.2
    ```


#### Imputing missing values

Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)
    
    
    ```r
    length(which(is.na(activity$steps)))
    ```
    
    ```
    ## [1] 2304
    ```


2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

    
    ```r
    activity_is_na <- activity[which(is.na(activity$steps)), ]
    # Strategy used is mean of each interval of all days is used to replace NA's
    a <- merge(activity_is_na, mean_steps_per_interval, by = "interval")
    ```



3. Create a new dataset that is equal to the original dataset but with the missing data filled in.
    
    ```r
    clean_activity <- activity  # make a copy of activity dataset
    library(dplyr)
    ```
    
    ```
    ## 
    ## Attaching package: 'dplyr'
    ## 
    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag
    ## 
    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union
    ```
    
    ```r
    clean_activity_df = tbl_df(clean_activity)
    a_df <- tbl_df(a)
    
    b_df <- left_join(clean_activity_df, a_df, by = c("date", "interval"))
    clean_df <- select(mutate(b_df, clean_steps = ifelse(is.na(mean_steps), steps.x, 
        mean_steps)), date, interval, clean_steps)
    ```



4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

    
    ```r
    clean_steps_per_day <- aggregate(clean_df$clean_steps, by = list(clean_df$date), 
        sum, na.rm = TRUE)
    
    colnames(clean_steps_per_day) <- c("date", "no_of_steps")
    hist(clean_steps_per_day$no_of_steps, col = "blue", main = "Histogram of sum of steps per day (cleaned)", 
        ylab = "percentage of days", xlab = "no of steps", ylim = c(0, 40))
    ```
    
    ![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9.png) 
    
    ```r
    
    clean_mean_steps <- mean(clean_steps_per_day$no_of_steps, na.rm = TRUE)
    # aggregate(activity$steps, by = list(activity$date), mean, na.rm = TRUE)
    clean_median_steps <- median(clean_steps_per_day$no_of_steps, na.rm = TRUE)
    cat("original mean = ", mean_steps, "; cleaned data mean = ", clean_mean_steps)
    ```
    
    ```
    ## original mean =  9354 ; cleaned data mean =  10766
    ```
    
    ```r
    cat("original median = ", median_steps, "; cleaned data median = ", clean_median_steps)
    ```
    
    ```
    ## original median =  10395 ; cleaned data median =  10766
    ```


#### Are there differences in activity patterns between weekdays and weekends?

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

    
    ```r
    clean_week_df <- mutate(clean_df, weekday = weekdays(date), week_indicator = ifelse(weekdays(date) %in% 
        "Sunday", "weekend", "weekday"))
    clean_week_steps_per_day <- aggregate(clean_week_df$clean_steps, by = list(clean_week_df$week_indicator, 
        clean_week_df$interval), mean, na.rm = TRUE)
    colnames(clean_week_steps_per_day) <- c("week_indicator", "interval", "mean_steps")
    ```


2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). The plot should look something like the following, which was creating using simulated data:

    
    ```r
    library(ggplot2)
    ggplot(clean_week_steps_per_day, aes(x = interval, y = mean_steps)) + geom_line() + 
        facet_wrap(~week_indicator, scales = "free_y", ncol = 1) + ylab("Number of steps")
    ```
    
    ![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11.png) 

