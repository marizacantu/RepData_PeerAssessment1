---
title: "PA1_template"
author: "Mariza Cantu"
date: "11 de junio de 2016"
output: html_document
---

This is an R Markdown document. Markdown is a simple formatting syntax for authoring HTML, PDF, and MS Word documents. For more details on using R Markdown see <http://rmarkdown.rstudio.com>.

When you click the **Knit** button a document will be generated that includes both content as well as the output of any embedded R code chunks within the document. You can embed an R code chunk like this:

#Removes all variables from environment
rm(list=ls(all=TRUE)) 

echo=TRUE

#Setting the new directory
setwd("C:/Users/Mariza/Documents/Cursos/Especializaci�n estad�stica/Reporting Data")

library(ggplot2)
library(knitr)

#Reading the data
rdata <- read.csv("activity.csv", header = T, sep ="," , colClasses = c("integer","Date","factor"))

 
#Removing rows with NA
clean.data <- na.omit(rdata)


#Transforming date en data date type
clean.data$date <- as.Date(clean.data$date, format = "%Y%m%d")

#Interval field
clean.data$interval <- as.factor(clean.data$interval)

#Calculate steps ignoring missing values
Total.steps <- aggregate(clean.data$steps,list(date = clean.data$date),FUN=sum,na.rm=TRUE)

#Plotting histogram
plot1 <- ggplot(clean.data, aes(date, steps)) + geom_bar(stat = "identity",binwidth = .5) +
  labs(title = "Histogram of Total Number of Steps Taken Each Day",x = "Date", y = "Total Number of Steps")
print(plot1)
ECHO=TRUE

#Calculating mean
mean(Total.steps$x)
###[1] 10766.19

#Calculating median
median(Total.steps$x)
###[1] 10765

#Making a time series plot
steps.interval <- aggregate(steps ~ interval, data = rdata, FUN = mean)
plot(steps.interval, type = "l")

#Calculating the 5-minute interval with more steps
steps.interval$interval[which.max(steps.interval$steps)]
###[1] 835

#Reporting amount of missing values
sum(is.na(rdata))
###[1] 2304

#Creating a new data set with the missing values
newdata <- merge(rdata, steps.interval, by = "interval", suffixes = c("", 
                                                                          ".y"))
nas <- is.na(rdata$steps)
newdata$steps[nas] <- newdata$steps.y[nas]
newdata <- newdata[, c(1:3)]

#Histogram with missing values
date.steps <- aggregate(steps ~ date, data = newdata, FUN = sum)
barplot(date.steps$steps, names.arg = date.steps$date, xlab = "date", ylab = "steps")
ECHO=TRUE

##Mean with imputed data
total.steps <- tapply(date.steps$steps, date.steps$date, FUN = sum)
mean(Total.steps$x)
###[1] 10766.19

#Median with imputed data
median(Total.steps$x)
###[1] 10765


##Comparission of activity on weekdays and weekends, creating another variable


daytype <- function(date) {
  if (weekdays(as.Date(date)) %in% c("Saturday", "Sunday")) {
    "weekend"
  } else {
    "weekday"
  }
}
newdata$daytype <- as.factor(sapply(newdata$date, daytype))

#Creating a variable indicating if it is a weekday or not
newdata$weekdays <- factor(format(newdata$date, "%A"))
levels(newdata$weekdays)

## [1] "domingo"   "jueves"    "lunes"     "martes"    "mi�rcoles" "s�bado"   
##[7] "viernes"

levels(newdata$weekdays) <- list(weekday = c("lunes", "martes",
                                             "miercoles", 
                                             "jueves", "viernes"),
                                 weekend = c("sabado", "domingo"))
levels(newdata$weekdays)
## [1] "weekday" "weekend"

table(newdata$weekdays)
##weekday weekend 
##9792    2592


#Plotting a time series plot
new.averages <- aggregate(newdata$steps, 
                          list(interval = as.numeric(as.character(newdata$interval)), 
                               weekdays = newdata$weekdays),
                          FUN = "mean")
names(new.averages)[3] <- "meanOfSteps"
library(lattice)
plot2 <- xyplot(new.averages$meanOfSteps ~ new.averages$interval | new.averages$weekdays, 
                layout = c(1, 2), type = "1", 
                xlab = "Interval", ylab = "Number of steps")
print(plot2)
ECHO=TRUE