Reproducible Research: Week 2 Assignment
========================================================
  
  ```{r}
library(ggplot2)
library(dplyr)
```


##  Loading and preprocessing the data

#### 1.Load the data(i.e read.csv())

I used read.csv() to load the data.
```{r,echo=TRUE}
data <- read.csv("activity.csv",header = TRUE)
```

To see few observations, I used head() command.
```{r top 6 rows,echo=TRUE}
head(data)
```

#### 2.Process the data into a format suitable for your analysis

I used na.omit() to remove the missing values from the table

```{r}
data.complete <- na.omit(data)
```


## What is mean total number of steps taken per day?

#### 1.Calculate the total number of steps taken per day

I used the dplyr library to calculate the sum of steps per day.

```{r}
data.day <- group_by(data.complete,date)
data.day <- summarize(data.day,steps=sum(steps))
```

#### 2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day

I then plot a histogram of the total number of steps taken each day.
```{r}
qplot(steps,data=data.day)
```

#### 3. Calculate and report the mean and median of the total number of steps taken per day

I used the mean() and median() functions.

```{r}
mean(data.day$steps)
```

```{r}
median(data.day$steps)
```

## What is the average daily activity pattern?

#### 1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

First, we create a data frame in which steps are aggregated into averages within each 5 minute interval
```{r}
data.int <- group_by(data.complete, interval)
data.int <- summarize(data.int, steps=mean(steps))
```

Now, we plot the average daily steps against the intervals

```{r}
ggplot(data.int, aes(interval, steps)) + geom_line()+ labs(title="Average daily activity pattern")
```

#### 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

We find the row in the interval data frame for which steps is equal to the maximum number of steps, then we look at the interval of that row
```{r}
data.int[data.int$steps==max(data.int$steps),]
```

## Imputing missing values

#### 1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

Here, we will find the total number of NA's in the data frame.

```{r}
count(data)-count(data.complete)
```

#### 2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

I replaced missing values with the mean number of steps for each interval across all of the days. The data.int data frame contains these means. I start by merging the data.int data with the raw data

```{r}
names(data.int)[2] <- "mean.steps"
data.impute <- merge(data, data.int)
```

#### 3. Create a new dataset that is equal to the original dataset but with the missing data filled in

If steps is NA, I replaced the value with the mean number of steps for the interval

```{r}
data.impute$steps[is.na(data.impute$steps)] <- data.impute$mean.steps[is.na(data.impute$steps)]
```

#### 4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

I first create a dataset with the total number of steps per day using the imputed data

```{r}
data.day.imp <- group_by(data.impute, date)
data.day.imp <- summarize(data.day.imp, steps=sum(steps))
```

Then I generate the histogram and summary statistics

```{r}
qplot(steps, data=data.day.imp)
```

```{r}
mean(data.day.imp$steps)
```

```{r}
median(data.day.imp$steps)
```

## Are there differences in activity patterns between weekdays and weekends?

#### For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

#### 1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

I convert the date variable to the date class, then use the weekdays() function to generate the day of the week of each date. I create a binary factor to indicate the two weekend days

```{r}
data.impute$dayofweek <- weekdays(as.Date(data.impute$date))
data.impute$weekend <-as.factor(data.impute$dayofweek=="Saturday"|data.impute$dayofweek=="Sunday")
levels(data.impute$weekend) <- c("Weekday", "Weekend")
```

#### 2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

First I create separate data frames for weekends and weekdays

```{r}
data.weekday <- data.impute[data.impute$weekend=="Weekday",]
data.weekend <- data.impute[data.impute$weekend=="Weekend",]
```

Then for each one, I find the mean number of steps across days for each 5 minute interval
```{r}
data.int.weekday <- group_by(data.weekday, interval)
data.int.weekday <- summarize(data.int.weekday, steps=mean(steps))
data.int.weekday$weekend <- "Weekday"
data.int.weekend <- group_by(data.weekend, interval)
data.int.weekend <- summarize(data.int.weekend, steps=mean(steps))
data.int.weekend$weekend <- "Weekend"
```

I append the two data frames together, and I make the two time series plots

```{r}
data.int <- rbind(data.int.weekday, data.int.weekend)
data.int$weekend <- as.factor(data.int$weekend)
ggplot(data.int, aes(interval, steps)) + geom_line() + facet_grid(weekend ~ .)
```





























