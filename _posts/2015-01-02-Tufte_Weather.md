---
layout: post
title: Tufte Visualization of Dayton Weather
---


<style>
div {
    text-align: justify;
    text-justify: inter-word;
}
</style>

<br>

This report provides insight and explanation behind the code used to produce the following graphic which is formatted to resemble the illustration provided in [Edward Tufte](http://www.edwardtufte.com/tufte/)'s classic book [Visual Display of Quantitative Information, 2<sup>nd</sup> Ed.](http://www.amazon.com/The-Visual-Display-Quantitative-Information/dp/0961392142) *(page 30)*.


<img src="/public/images/tufte/unnamed-chunk-1-1.png" alt="Tufte Recreated" align="middle">

### Tufte's Image
The original illustration in Tufte's book comes from *The New York Times*, January 4, 2004, A15.  Although the original graphic had two parts; the temperature component, as shown in the below image, and then a precipitation compenent, I chose to only focus on the temperature component since I could not locate historical precipitation data. I should also point out that the original graphic was based on daily high and low temperatures, whereas my graphic is based on daily average temperatures.  This is likely the reason for the different "thickness" in the range bands and also the reason why the original graphic uses range bars for the current year temps and my graphic uses lines.  It is my assumption that the original graphics *"Normal Range"*, which is the center dark band, represents the mean high and mean low temps for each day.  Since I only have the daily average temps, my *"Normal Range"* represents the 95% confidence interval around the historical mean daily average temps *(thats right, it's an average of an average...just lovely! But since we're reflecting the same measurement for a single group and, for the most part, all days have the same number of observations our interpretation is not being skewed.)*


<img src="/public/images/tufte/tufte_original.JPG" alt="Tufte Original" align="middle">


### <font face="serif">R Packages Utilized</font>
The following packages were used to develop the visualization.

```r
# Preprocessing & summarizing data
library(dplyr)
library(tidyr)

# Visualizatin development
library(ggplot2)
```

### <font face="serif">Getting & Preparing the Data</font>
The data used was obtained from the University of Dayton's [Average Daily Temperature archive](http://academic.udayton.edu/kissock/http/Weather/) website which contains daily average temperatures for 157 U.S. and 167 international cities with data spanning from January 1, 1995 to present.  Individual text files are provided for each city and the raw data represents month, day, year, and average daily temperature (°F) and looks like:


```
##   V1 V2   V3   V4
## 1  1  1 1995 39.0
## 2  1  2 1995 19.6
## 3  1  3 1995 20.6
## 4  1  4 1995 11.3
## 5  1  5 1995  6.8
## 6  1  6 1995 23.0
```

First, I process the raw data to create a "Past" dataframe, which represent 1995-2013 data.  This, ultimately, will be the background data to compare the current year data against.  I create a new variable *("newDay")* that labels observations *(aka days)* from 1 to 365, which will represent the x-axis.  I filter out missing data, represented by the value -99 *(there were only 16 days across all 20 years with missing data - 0.002% observations)*, and remove current year data.  Next, I identify the *min* and *max* value for each day; this will create the wider wheat colored band in the background that represents the record high and low average temps for the past 20 years.  I then calculate the mean temp for each day along with the 95% confidence interval to represent the darker brown color which represents the *Normal Range*.

I also create a "Present" dataframe, which represents current year *(2014)* data.  I create a matching x-axis variable *("newDay")* and filter out missing data *(turns out there were no missing data for 2014)*.  This data will represent the black line in the graphic.


```r
# rename variables
names(DAY) <- c("Month", "Day", "Year", "Temp")

# create dataframe that represents 1995-2013 historical data
Past <- DAY %>%
        group_by(Year, Month) %>%
        arrange(Day) %>%
        ungroup() %>%
        group_by(Year) %>%
        mutate(newDay = seq(1, length(Day))) %>%   # label days as 1:365         
        ungroup() %>%
        filter(Temp != -99 & Year != 2014) %>%     # filter out missing data
        group_by(newDay) %>%
        mutate(upper = max(Temp), # identify max value for each day
               lower = min(Temp), # identify min value for each day
               avg = mean(Temp),  # calculate mean value for each day
               se = sd(Temp)/sqrt(length(Temp))) %>%  # calculate standard error of mean
        mutate(avg_upper = avg+(2.101*se),  # calculate 95% CI for mean
               avg_lower = avg-(2.101*se)) %>%  # calculate 95% CI for mean
        ungroup()

# create dataframe that represents current year data
Present <- DAY %>%
        group_by(Year, Month) %>%
        arrange(Day) %>%
        ungroup() %>%
        group_by(Year) %>%
        mutate(newDay = seq(1, length(Day))) %>%  # create matching x-axis data
        ungroup() %>%
        filter(Temp != -99 & Year == 2014)  # filter out missing data
```

With the primary data in place, which allows me to match Tufte's illustration closely, I decided that adding one more comparable dimension to our data would be interesting.  We had a harsh winter in 2014 so I was curious as to how many days we had in which the average daily temperature was colder than all previous years.  I figured while assessing this, I might as well compare the current year highs versus historical record highs as well *(record high and lows meaning the highest average daily temp and the lowest average daily temp)*.

To add this dimension to the graphic, I create dataframes to represent the days in which the current year had the coldest temperature than all previous 19 years *("PresentLows")* and days in which the current year had the warmest temperature than all previous 19 years *("PresentHighs")*.  This data will be used to represent the blue and red points on the graphic.


```r
# create dataframe that represents the lowest temp for each day for the historical data
PastLows <- Past %>%
        group_by(newDay) %>%
        summarise(Pastlow = min(Temp)) # identify lowest temp for each day from 1995-2013

# create dataframe that identifies the days in 2014 in which the temps were lower than all
# previous 19 years
PresentLows <- Present %>%
        left_join(PastLows) %>%  # merge historical lows to current year low data
        mutate(record = ifelse(Temp < Pastlow, "Y", "N")) %>% # identify if record low
        filter(record == "Y")  # filter for days that represent current year record lows

# create dataframe that represents the highest temp for each day for the historical data
PastHighs <- Past %>%
        group_by(newDay) %>%
        summarise(Pasthigh = max(Temp))  # identify highest temp for each day from '95-'13

# create dataframe that identifies the days in 2014 in which the temps were higher than 
# all previous 19 years
PresentHighs <- Present %>%
        left_join(PastHighs) %>%  # merge historical highs to current year low data
        mutate(record = ifelse(Temp>Pasthigh, "Y", "N")) %>% # identifies if record high
        filter(record == "Y")  # filter for days that represent current year record highs
```

Lastly, to create the appropriate y-axis, I need to generate a function to turn the y-axis labels into the appropriate format with the degree symbol (°) after the numbers.  I then create a variable that represents the y-axis labels I want to display using `seq()` and apply the degree formatting function to these labels.  I save it as variable "a" to be used later in my ggplot code.  

The final data I need to process is to create a small line that will represent the legend symbol for the 2014 Temperature.  In Tufte's illustration, the highs and lows data for each day were available so it made sense to represent the current year bars; however, in my case I only have access to the average daily temperature for each day.  Since I don't have a range of data for the individual days in 2014, I can only represent the data by a single line.  This, ultimately, is why my legend differs slightly compared to the original graphic.


```r
# function to turn y-axis labels into degree formatted values
dgr_fmt <- function(x, ...) {
        parse(text = paste(x, "*degree", sep = ""))
}

# create y-axis variable
a <- dgr_fmt(seq(-20,100, by=10))

# create a small dataframe to represent legend symbol for 2014 Temperature
legend_data <- data.frame(x=seq(175,182),y=rnorm(8,15,2))
```

### <font face="serif">Creating the Visual Graphic</font>
Now that the data is ready, I can begin developing the illustration.  The important thing to remember is that graphics should be built on layers.  This is important because it helps you to organize your visualization steps and forces you to think about smaller details and where, in the graphic building process, they should be developed.  To explain the logic I'll go through chunks of the development process to illustrate how the layers create their given effects.

#### <u><font face="serif">Step 1</font></u>
The first thing I like to do is to create my "canvas".  In this case, the underlying theme is very basic with no borders and custom gridlines.  I say "custom gridlines" for two reasons.  First, the x-axis gridlines intend to separate the months; however, the real x-axis is number of days from 1 to 365.  Since the length of months are irregular *(31 days, 30 days, 28 days, etc)*, we will manually insert x-axis gridlines.  The second reason for custom gridlines is, if you look at Tufte's original illustration, you will notice that the y-axis gridlines lay on top of the data.  This allows the white y-axis gridlines to blend into white space *(reducing ink ratio)* but to show up on top of the data to provide reference points.  Since gridlines in ggplot are always the first layer, we need to remove these gridlines and incorporate our own later in the process.

To begin, I'll remove all basic theme layers.  This includes removing background color; major and minor gridlines; and x & y axis borders, ticks, and titles.  It's important to note that although you may be eager to include `axis.text = element_blank()` to remove the x & y axis labels, this will keep you from displaying any future labels in their place.  So for now, we keep the axis labels in place.

This essentially gives us a fresh canvas to work with. I can then plot our first layer which represents the range of average daily temperatures for 1995-2013.  


```r
p <- ggplot(Past, aes(newDay, Temp)) +
        theme(plot.background = element_blank(),
              panel.grid.minor = element_blank(),
              panel.grid.major = element_blank(),
              panel.border = element_blank(),
              panel.background = element_blank(),
              axis.ticks = element_blank(),
              #axis.text = element_blank(),  
              axis.title = element_blank()) +
        geom_linerange(Past, mapping=aes(x=newDay, ymin=lower, ymax=upper), 
                       colour = "wheat2", alpha=.1)

print(p)
```

<img src="/public/images/tufte/unnamed-chunk-7-1.png" alt="Tufte Recreated" align="middle">

#### <u><font face="serif">Step 2</font></u>
Next, we can add the data that represents the 95% confidence interval around the daily mean temperatures for 1995-2013.


```r
p <- p + 
        geom_linerange(Past, mapping=aes(x=newDay, ymin=avg_lower, ymax=avg_upper), 
                       colour = "wheat4")

print(p)
```

<img src="/public/images/tufte/unnamed-chunk-8-1.png" alt="Tufte Recreated" align="middle">

#### <u><font face="serif">Step 3</font></u>
Here, we can incorporate the current year temperature data. This is also the step in which I incorporate the y-axis border.  As you can see in Tufte's original, the y-axis border appears as dashes; however, in reality it is a solid line that has the y-axis gridlines laying over top of it which creates the dashed effect at the tickmarks of interest


```r
p <- p + 
        geom_line(Present, mapping=aes(x=newDay, y=Temp, group=1)) +
        geom_vline(xintercept = 0, colour = "wheat4", linetype=1, size=1)

print(p)
```

<img src="/public/images/tufte/unnamed-chunk-9-1.png" alt="Tufte Recreated" align="middle">

#### <u><font face="serif">Step 4</font></u>
Now it's time to add the y-axis gridlines.  These gridlines are very discreet and are meant to only provide reference points when necessary.  The only place the viewer needs to reference the degree relationship is within the "band" of data; otherwise, we want the gridlines to blend into the background to keep the ink ratio low.  Another purpose of these gridlines is to create the dashed effect on the custom y-axis gridline.


```r
p <- p + 
        geom_hline(yintercept = -20, colour = "white", linetype=1) +
        geom_hline(yintercept = -10, colour = "white", linetype=1) +
        geom_hline(yintercept = 0, colour = "white", linetype=1) +
        geom_hline(yintercept = 10, colour = "white", linetype=1) +
        geom_hline(yintercept = 20, colour = "white", linetype=1) +
        geom_hline(yintercept = 30, colour = "white", linetype=1) +
        geom_hline(yintercept = 40, colour = "white", linetype=1) +
        geom_hline(yintercept = 50, colour = "white", linetype=1) +
        geom_hline(yintercept = 60, colour = "white", linetype=1) +
        geom_hline(yintercept = 70, colour = "white", linetype=1) +
        geom_hline(yintercept = 80, colour = "white", linetype=1) +
        geom_hline(yintercept = 90, colour = "white", linetype=1) +
        geom_hline(yintercept = 100, colour = "white", linetype=1)

print(p)
```

<img src="/public/images/tufte/unnamed-chunk-10-1.png" alt="Tufte Recreated" align="middle"> 

#### <u><font face="serif">Step 5</font></u>
Now we will start to add the x-axis gridlines.  We add the dotted gridlines to the last day of each month.


```r
p <- p + 
        geom_vline(xintercept = 31, colour = "wheat4", linetype=3, size=.5) +
        geom_vline(xintercept = 59, colour = "wheat4", linetype=3, size=.5) +
        geom_vline(xintercept = 90, colour = "wheat4", linetype=3, size=.5) +
        geom_vline(xintercept = 120, colour = "wheat4", linetype=3, size=.5) +
        geom_vline(xintercept = 151, colour = "wheat4", linetype=3, size=.5) +
        geom_vline(xintercept = 181, colour = "wheat4", linetype=3, size=.5) +
        geom_vline(xintercept = 212, colour = "wheat4", linetype=3, size=.5) +
        geom_vline(xintercept = 243, colour = "wheat4", linetype=3, size=.5) +
        geom_vline(xintercept = 273, colour = "wheat4", linetype=3, size=.5) +
        geom_vline(xintercept = 304, colour = "wheat4", linetype=3, size=.5) +
        geom_vline(xintercept = 334, colour = "wheat4", linetype=3, size=.5) +
        geom_vline(xintercept = 365, colour = "wheat4", linetype=3, size=.5) 

print(p)
```

<img src="/public/images/tufte/unnamed-chunk-11-1.png" alt="Tufte Recreated" align="middle">

#### <u><font face="serif">Step 6</font></u>
Now it's time to dress up the axis labels.  First, I limit the y-axis to a range of [-20°, 100°].  I then force the breaks to line up with the custom y-axis gridlines at even degrees in multiples of 10.  I assign the labels to the degree formatted variable *("a")* that I created earlier to display the degree symbol.  For the x-axis, I removed the spacing *(it's hard to see but there is padded space added to the original x-axis)* at the edges of the x-axis, identified the breaks to place labels, and then provided the month names as the labels.

Don't be fooled, there was no magical approach to identifying the best breaks for the x-axis.  I started with the day that represented the middle of each month and then moved them around as required to get the month names to appear centered.


```r
p <- p +
        coord_cartesian(ylim = c(-20,100)) +
        scale_y_continuous(breaks = seq(-20,100, by=10), labels = a) +
        scale_x_continuous(expand = c(0, 0), 
                           breaks = c(15,45,75,105,135,165,195,228,258,288,320,350),
                           labels = c("January", "February", "March", "April",
                                      "May", "June", "July", "August", "September",
                                      "October", "November", "December"))

print(p)
```

<img src="/public/images/tufte/unnamed-chunk-12-1.png" alt="Tufte Recreated" align="middle">

#### <u><font face="serif">Step 7</font></u>
At this point we have the basic underlying graphic that is similar to Tufte's temperature plot.  Now it's time to add in the extra comparisons that I wanted to look at; this includes adding in the points that identify the days in which the current year *(2014)* had the record high and low temperature.


```r
p <- p +
        geom_point(data=PresentLows, aes(x=newDay, y=Temp), colour="blue3") +
        geom_point(data=PresentHighs, aes(x=newDay, y=Temp), colour="firebrick3")

print(p)
```

<img src="/public/images/tufte/unnamed-chunk-13-1.png" alt="Tufte Recreated" align="middle">

#### <u><font face="serif">Step 8</font></u>
Since all data have been plotted, it's now time to dress up the graphic with the appropriate text.  All the steps that follow involve a lot of trial and error to find the right location and spacing for the text.  So what you are seeing are the final function parameters resulting from several iterations.  We'll start with the title and subtitle.


```r
p <- p +
        ggtitle("Dayton's Weather in 2014") +
        theme(plot.title=element_text(face="bold",hjust=.012,vjust=.8,colour="#3C3C3C", size=20)) +
        annotate("text", x = 19, y = 98, label = "Temperature", size=4, fontface="bold")

print(p)
```

<img src="/public/images/tufte/unnamed-chunk-14-1.png" alt="Tufte Recreated" align="middle"> 

#### <u><font face="serif">Step 9</font></u>
We can now add the paragraph under the subtitle that provides a little explanation about the data.  I broke this paragraph up into four separate annotations because when you apply `\n` within `annotate()` to create line breaks it will center the text rather than left justify.  There may be a way to left justify...I just couldn't figure it out.  


```r
p <- p +
        annotate("text", x = 66, y = 93, 
                 label = "Data represents average daily temperatures. Accessible data dates back to", 
                 size=3, colour="gray30") +
        annotate("text", x = 62, y = 89, 
                 label = "January 1, 1995. Data for 2014 is only available through December 16.", 
                 size=3, colour="gray30") +
        annotate("text", x = 64, y = 85, 
                 label = "Average temperature for the year was 51.9° making 2014 the 9th coldest", 
                 size=3, colour="gray30") +
        annotate("text", x = 18, y = 81, label = "year since 1995", size=3, 
                 colour="gray30")

print(p)
```

<img src="/public/images/tufte/unnamed-chunk-15-1.png" alt="Tufte Recreated" align="middle">

#### <u><font face="serif">Step 10</font></u>
Next, we'll create annotations to explain the points representing the days in which record highs or lows were experienced.


```r
p <- p +
        annotate("segment", x = 30, xend = 40, y = -5, yend = -10, colour = "blue3") +
        annotate("text", x = 65, y = -10, label = "We had 35 days that were the", size=3, colour="blue3") +
        annotate("text", x = 56, y = -14, label = "coldest since 1995", size=3, colour="blue3") +
        annotate("segment", x = 302, xend = 307, y = 74, yend = 82, colour = "firebrick3") +
        annotate("text", x = 333, y = 82, label = "We had 19 days that were the", size=3, colour="firebrick3") +
        annotate("text", x = 324, y = 78, label = "hottest since 1995", size=3, colour="firebrick3")

print(p)
```

<img src="/public/images/tufte/unnamed-chunk-16-1.png" alt="Tufte Recreated" align="middle"> 

#### <u><font face="serif">Step 11</font></u>        
The final step is to add a legend to explain to the reader the difference between the different data point layers.


```r
p <- p +
        annotate("segment", x = 181, xend = 181, y = 5, yend = 25, colour = "wheat2", size=3) +
        annotate("segment", x = 181, xend = 181, y = 12, yend = 18, colour = "wheat4", size=3) +
        geom_line(data=legend_data, aes(x=x,y=y)) +
        annotate("segment", x = 183, xend = 185, y = 17.7, yend = 17.7, colour = "wheat4", size=.5) +
        annotate("segment", x = 183, xend = 185, y = 12.2, yend = 12.2, colour = "wheat4", size=.5) +
        annotate("segment", x = 185, xend = 185, y = 12.2, yend = 17.7, colour = "wheat4", size=.5) +
        annotate("text", x = 196, y = 14.75, label = "NORMAL RANGE", size=2, colour = "gray30") +
        annotate("text", x = 162, y = 14.75, label = "2014 TEMPERATURE", size=2, colour = "gray30") +
        annotate("text", x = 193, y = 25, label = "RECORD HIGH", size=2, colour = "gray30") +
        annotate("text", x = 193, y = 5, label = "RECORD LOW", size=2, colour = "gray30")

print(p)
```

<img src="/public/images/tufte/unnamed-chunk-17-1.png" alt="Tufte Recreated" align="middle">

### <font face="serif">Conclusion</font>
The end result is, what I believe, a wonderful looking graphic that compares quite well with the original and, most importantly, tells a good story about last year's weather.  If satisfaction could be measured by "retweets", then Edward Tufte's retweet of this graphic provides me with enough gratification to label it a success.


<small><a href="#">Go to top</a></small>



