---
categories:
- ""
- ""
date: "2017-10-31T21:28:43-05:00"
description: ""
draft: false
image: pic10.jpg
keywords: ""
slug: blog1
title: Ipsum
---


```{r, setup, include=FALSE}
knitr::opts_chunk$set(
  message = FALSE, 
  warning = FALSE, 
  tidy=FALSE,     # display code as typed
  size="small")   # slightly smaller font for code
options(digits = 3)

# default figure size
knitr::opts_chunk$set(
  fig.width=6.75, 
  fig.height=6.75,
  fig.align = "center"
)
```


```{r load-libraries, include=FALSE}
library(tidyverse)  # Load ggplot2, dplyr, and all the other tidyverse packages
library(mosaic)
library(ggthemes)
library(lubridate)
library(here)
library(skimr)
library(janitor)
library(httr)
library(readxl)
library(vroom)
```



# Climate change and temperature anomalies 


If we wanted to study climate change, we can find data on the *Combined Land-Surface Air and Sea-Surface Water Temperature Anomalies* in the Northern Hemisphere at [NASA's Goddard Institute for Space Studies](https://data.giss.nasa.gov/gistemp). The [tabular data of temperature anomalies can be found here](https://data.giss.nasa.gov/gistemp/tabledata_v3/NH.Ts+dSST.txt)

To define temperature anomalies you need to have a reference, or base, period which NASA clearly states that it is the period between 1951-1980.

Run the code below to load the file:

```{r weather_data, cache=TRUE}

weather <- 
  read_csv("https://data.giss.nasa.gov/gistemp/tabledata_v3/NH.Ts+dSST.csv", 
           skip = 1, 
           na = "***")

```

Notice that, when using this function, we added two options: `skip` and `na`.

1. The `skip=1` option is there as the real data table only starts in Row 2, so we need to skip one row. 
1. `na = "***"` option informs R how missing observations in the spreadsheet are coded. When looking at the spreadsheet, you can see that missing data is coded as "***". It is best to specify this here, as otherwise some of the data is not recognized as numeric data.

Once the data is loaded, notice that there is a object titled `weather` in the `Environment` panel. If you cannot see the panel (usually on the top-right), go to `Tools` > `Global Options` > `Pane Layout` and tick the checkbox next to `Environment`. Click on the `weather` object, and the dataframe will pop up on a seperate tab. Inspect the dataframe.

For each month and year, the dataframe shows the deviation of temperature from the normal (expected). Further the dataframe is in wide format. 

You have two objectives in this section:

1. Select the year and the twelve month variables from the `weather` dataset. We do not need the others (J-D, D-N, DJF, etc.) for this assignment. Hint: use `select()` function.

1. Convert the dataframe from wide to 'long' format. Hint: use `gather()` or `pivot_longer()` function. Name the new dataframe as `tidyweather`, name the variable containing the name of the month as `month`, and the temperature deviation values as `delta`.


```{r tidyweather}
weather_v2 <- weather[,1:13]
weather_v3 <- weather %>%
  select(c(1:13))
tidyweather <-   weather_v2 %>%
  pivot_longer(cols = 2:13, #columns 3 to 5
               names_to = "month",
               values_to = "delta")

```

Inspect your dataframe. It should have three variables now, one each for 

1. year, 
1. month, and 
1. delta, or temperature deviation.

## Plotting Information

Let us plot the data using a time-series scatter plot, and add a trendline. To do that, we first need to create a new variable called `date` in order to ensure that the `delta` values are plot chronologically. 


> In the following chunk of code, I used the `eval=FALSE` argument, which does not run a chunk of code; I did so that you can knit the document before tidying the data and creating a new dataframe `tidyweather`. When you actually want to run this code and knit your document, you must delete `eval=FALSE`, **not just here but in all chunks were `eval=FALSE` appears.**


```{r scatter_plot,warnings= FALSE, message=FALSE}

tidyweather <- tidyweather %>%
  mutate(date = ymd(paste(as.character(Year), month, "1")),
         month = month(date, label=TRUE),
         year = year(date))

ggplot(tidyweather, aes(x= date, y = delta))+
  geom_point()+
  geom_smooth(color="red") +
  theme_bw() +
  labs (
    title = "Weather Anomalies"
  )


```

Is the effect of increasing temperature more pronounced in some months? Use `facet_wrap()` to produce a seperate scatter plot for each month, again with a smoothing line. Your chart should human-readable labels; that is, each month should be labeled "Jan", "Feb", "Mar" (full or abbreviated month names are fine), not `1`, `2`, `3`. 

```{r facet_wrap, warnings= FALSE, message=FALSE}

#Your code goes here...
ggplot(tidyweather, aes(x=date, y = delta))+
  geom_point()+
  geom_smooth(color="red") +
  theme_bw() +
  labs (
    title = "Weather Anomalies"
  ) + facet_wrap(~month) +
  theme_wsj()

```


It is sometimes useful to group data into different time periods to study historical data. For example, we often refer to decades such as 1970s, 1980s, 1990s etc. to refer to a period of time. NASA calcuialtes a temperature anomaly, as difference form the base periof of 1951-1980. The code below creates a new data frame called `comparison` that groups data in five time periods: 1881-1920, 1921-1950, 1951-1980, 1981-2010 and 2011-present. 

We remove data before 1800 and before using `filter`. Then, we use the `mutate` function to create a new variable `interval` which contains information on which period each observation belongs to. We can assign the different periods using `case_when()`.


```{r intervals, warnings= FALSE, message=FALSE}

comparison <- tidyweather %>%
  filter(Year>= 1881) %>%     #remove years prior to 1881
  #create new variable 'interval', and assign values based on criteria below:
  mutate(interval = case_when(
    Year %in% c(1881:1920) ~ "1881-1920",
    Year %in% c(1921:1950) ~ "1921-1950",
    Year %in% c(1951:1980) ~ "1951-1980",
    Year %in% c(1981:2010) ~ "1981-2010",
    TRUE ~ "2011-present"
  ))

```

Inspect the `comparison` dataframe by clicking on it in the `Environment` pane.

Now that we have the `interval` variable, we can create a density plot to study the distribution of monthly deviations (`delta`), grouped by the different time periods we are interested in. Set `fill` to `interval` to group and colour the data by different time periods.

```{r density_plot, warnings= FALSE, message=FALSE}

ggplot(comparison, aes(x=delta, fill=interval))+
  geom_density(alpha=0.2) +   #density plot with tranparency set to 20%
  theme_bw() +                #theme
  labs (
    title = "Density Plot for Monthly Temperature Anomalies",
    y     = "Density"         #changing y-axis label to sentence case
  )

```

So far, we have been working with monthly anomalies. However, we might be interested in average annual anomalies. We can do this by using `group_by()` and `summarise()`, followed by a scatter plot to display the result. 

```{r averaging, warnings= FALSE, message=FALSE}

#creating yearly averages
average_annual_anomaly <- tidyweather %>%
  group_by(Year) %>%   #grouping data by Year
  
  # creating summaries for mean delta
  # use `na.rm=TRUE` to eliminate NA (not available) values
  summarise(annual_average_delta = mean(delta, na.rm=TRUE))

#plotting the data:
ggplot(average_annual_anomaly, aes(x=Year, y= annual_average_delta))+
  geom_point()+
  
  #Fit the best fit line, using LOESS method
  geom_smooth() +
  
  #change to theme_bw() to have white background + black frame around plot
  theme_bw() +
  labs (
    title = "Average Yearly Anomaly",
    y     = "Average Annual Delta"
  )                         


```


## Confidence Interval for `delta`

[NASA points out on their website](https://earthobservatory.nasa.gov/world-of-change/decadaltemp.php) that 

> A one-degree global change is significant because it takes a vast amount of heat to warm all the oceans, atmosphere, and land by that much. In the past, a one- to two-degree drop was all it took to plunge the Earth into the Little Ice Age.

Your task is to construct a confidence interval for the average annual delta since 2011, both using a formula and using a bootstrap simulation with the `infer` package. Recall that the dataframe `comparison` has already grouped temperature anomalies according to time intervals; we are only interested in what is happening  between 2011-present.

```{r calculate_CI_using_formula,warnings= FALSE, message=FALSE}

#install.packages("sjstats")
library(sjstats)
formula_ci <- comparison %>%
  filter(interval == "2011-present",!is.na(delta)) %>%
  summarise(mean_delta = mean(delta),
            sd_delta = sd(delta),
            count=n(),
            se_delta = se(delta),
            rating_low = mean_delta - se_delta*qt(0.975, count-1),
            rating_high = mean_delta + se_delta*qt(0.975, count-1))

#print out formula_CI
formula_ci
```



library(infer)
# use the infer package to construct a 95% CI for delta
boot_ratings <- comparison %>%
  # Select after 2011 withoun NA values
  filter(interval == "2011-present",!is.na(delta)) %>%
  # Specify the variable of interest
  specify(response =delta)%>%
  # Generate a bunch of bootstrap samples
  generate(reps =1000, type ="bootstrap")%>%
  # Find the mean of each sample
  calculate(stat ="mean")

percentali_ci <- boot_ratings %>%
  get_confidence_interval(level = 0.95, type = "percentile")

percentali_ci
formula_ci[,5:6]



> What is the data showing us? Please type your answer after (and outside!) this blockquote. You have to explain what you have done, and the interpretation of the result. One paragraph max, please!

Using the data from the comparison tab, we have selected years 2011 to 2019 using 103 observations to calculate the average annual delta, the standard deviation of the average annual delta, and built a 95% confidence interval. The calculations show that the average annual delta between 2011 to 2019 is 0.966, meaning that the temperature increased by an annual average of 0.966 between 2011 to 2019. We are 95% confident that the average annual delta will be between 0.915 and 1.02.
