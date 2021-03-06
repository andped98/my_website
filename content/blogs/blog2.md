---
categories:
- ""
- ""
date: "2017-10-31T22:26:09-05:00"
description: Lorem Etiam Nullam
draft: false
image: snap-inst.png
keywords: ""
slug: blog2
title: General Social Survey
---
# General Social Survey (GSS)

The [General Social Survey (GSS)](http://www.gss.norc.org/) gathers data on American society in order to monitor and explain trends in attitudes, behaviours, and attributes. Many trends have been tracked for decades, so one can see the evolution of attitudes, etc in American Society.


In this assignment we analyze data from the **2016 GSS sample data**, using it to estimate values of *population parameters* of interest about US adults. The GSS sample data file has 2867 observations of 935 variables, but we are only interested in very few of these variables and you are using a smaller file.


```{r, read_gss_data, cache=TRUE,warnings= FALSE, message=FALSE}
gss <- read_csv(here::here("data", "smallgss2016.csv"), 
                na = c("", "Don't know",
                       "No answer", "Not applicable"))
```

You will also notice that many responses should not be taken into consideration, like "No Answer", "Don't Know", "Not applicable", "Refused to Answer".

We will be creating 95% confidence intervals for population parameters. The variables we have are the following:

- hours and minutes spent on email weekly. The responses to these questions are recorded in the `emailhr` and `emailmin` variables. For example, if the response is 2.50 hours, this would be recorded as emailhr = 2 and emailmin = 30.
- `snapchat`, `instagrm`, `twitter`: whether respondents used these social media in 2016
- `sex`: Female - Male
- `degree`: highest education level attained

## Instagram and Snapchat, by sex

Can we estimate the *population* proportion of Snapchat or Instagram users in 2016?
```{r population proportion of Snapchat & Instagram, warnings= FALSE, message=FALSE}
#Remove NA from dataframe
gss_v3 <- gss %>% 
  filter(gss$instagrm!= "NA") 


#Coding answer to number: if yes for Instagram or snapchat - 1, otherwise 0
gss_v3 <- gss_v3 %>%   
  mutate(instagram_or_snap = ifelse(gss_v3$instagrm == "Yes", 1, ifelse(gss_v3$snapchat == "Yes", 1,0)))

total_user =  sum(gss_v3$instagram_or_snap)
gss_v3 %>% 
  group_by(sex) %>% 
  summarise(number_of_users = sum(instagram_or_snap), n_distinct=length(sex),proportion_within_sex = number_of_users / n_distinct, proportion_inside_users = number_of_users / total_user)

```


1. Create a  new variable, `snap_insta` that is *Yes* if the respondent reported using any of Snapchat (`snapchat`) or Instagram (`instagrm`), and *No* if not. If the recorded value was NA for both of these questions, the value in your new variable should also be NA.
```{r create snap_insta, warnings= FALSE, message=FALSE}
gss_v2 <- gss %>% mutate(snap_insta = ifelse(gss$instagrm == "Yes"| gss$snapchat == "Yes", "Yes", ifelse(gss$instagrm == "No"| gss$snapchat == "No", "No", NA)))
```

1. Calculate the proportion of Yes’s for `snap_insta` among those who answered the question, i.e. excluding NAs.
```{r snap_insta_proportion, warnings= FALSE, message=FALSE}
snap_insta_proportion <- gss_v2 %>% summarise(number_of_users = count(snap_insta == "Yes"), number_respondents = count(!is.na(snap_insta)), proportion = number_of_users/number_respondents)
 
snap_insta_proportion   
  
```

1. Using the CI formula for proportions, please construct 95% CIs for men and women who used either Snapchat or Instagram
```{r 95% CIs for men and women who used either Snapchat or Instagram, warnings= FALSE, message=FALSE}
gss_v2$snap_insta_code <-  if_else(gss_v2$snap_insta == "Yes", 1,0)
formula_ci_user <- gss_v2 %>% 
filter(!is.na(snap_insta)) %>% 
summarise(mean_user = mean(snap_insta_code),
  sd_user = sd(snap_insta_code),
  count=n(),
  se_user = parameters::standard_error(snap_insta_code),
  rating_low = mean_user - se_user*qt(0.975, count-1),
  rating_high = mean_user + se_user*qt(0.975, count-1))

formula_ci_user
```


## Twitter, by education level
Can we estimate the *population* proportion of Twitter users by education level in 2016?. 

There are 5 education levels in variable `degree` which, in ascneding order of years of education, are Lt high school, High School, Junior college, Bachelor, Graduate. 

1. Turn `degree` from a character variable into a factor variable. Make sure the order is the correct one and that levels are not sorted alphabetically which is what R by default does. 
```{r degree, warnings= FALSE, message=FALSE}
twitter_user <- gss 
twitter_user$degree = factor(twitter_user$degree, labels=c('Bachelor', 'Graduate', 'High school', 'Junior college', 'Lt high school', NA))
```
1. Create a  new variable, `bachelor_graduate` that is *Yes* if the respondent has either a `Bachelor` or `Graduate` degree. As before, if the recorded value for either was NA, the value in your new variable should also be NA.
```{r high education level, warnings= FALSE, message=FALSE}
twitter_user <- twitter_user %>% 
  mutate(bachelor_graduate = ifelse(degree == 'Bachelor'|degree == 'Graduate', 'Yes', ifelse( degree == 'Lt high school'|degree == 'High school'|degree ==  'Junior college', 'No' , NA)))
```

1. Calculate the proportion of `bachelor_graduate` who do (Yes) and who don't (No) use twitter. 
```{r proportion of bachelor graduate, warnings= FALSE, message=FALSE}
#With NA in dataset
twitter_user %>% 
  filter(bachelor_graduate == "Yes") %>% 
  summarise(Use_twitter_prop = sum(twitter == "Yes") / length(bachelor_graduate == "Yes"), Not_Use_twitter_proportion = sum(twitter == "No") / length(bachelor_graduate == "Yes"), na_answer_proportion = sum(twitter == "NA") / length(bachelor_graduate == "Yes"))
#Without NA in dataset
twitter_user %>% 
  filter(bachelor_graduate == "Yes" & twitter!="NA") %>% 
  summarise(Use_twitter_prop = sum(twitter == "Yes") / length(bachelor_graduate == "Yes"), Not_Use_twitter_proportion = sum(twitter == "No") / length(bachelor_graduate == "Yes"))
```

1. Using the CI formula for proportions, please construct two 95% CIs for `bachelor_graduate` vs whether they use (Yes) and don't (No) use twitter. 
```{r CI for use and not use, warnings= FALSE, message=FALSE}
#CI for use
formula_ci_use <- twitter_user %>% 
filter(bachelor_graduate == "Yes" & twitter!="NA") %>% 
mutate(yes_code = ifelse(twitter == "Yes", 1,0)) %>% 
summarise(mean_yes = mean(yes_code),
  sd_yes = sd(yes_code),
  count=n(),
  se_yes = parameters::standard_error(yes_code),
  rating_low = mean_yes - se_yes*qt(0.975, count-1),
  rating_high = mean_yes + se_yes*qt(0.975, count-1))
formula_ci_use
#CI for not use
formula_ci_not_use <- twitter_user %>% 
filter(bachelor_graduate == "Yes" & twitter!="NA") %>% 
mutate(no_code = ifelse(twitter == "No", 1,0)) %>% 
summarise(mean_no = mean(no_code),
  sd_no = sd(no_code),
  count=n(),
  se_no = parameters::standard_error(no_code),
  rating_low = mean_no - se_no*qt(0.975, count-1),
  rating_high = mean_no + se_no*qt(0.975, count-1))

formula_ci_not_use
```

1. Do these two Confidence Intervals overlap?

Yes. The 95% confidence internal for yes is [19.55%,27.07%], the second interval shows that proportion of user is between [100% - 80.45%, 100%- 72.93%] or [19.55% , 27.07%].They show almost the same interval.



## Email usage

Can we estimate the *population* parameter on time spent on email weekly?

1. Create a new variable called `email` that combines `emailhr` and `emailmin` to reports the number of minutes the respondents spend on email weekly.

```{r email, warnings= FALSE, message=FALSE}
email_minutes <- gss %>% filter(emailhr !='NA') %>% mutate(email= as.numeric(emailhr)*60  + as.numeric(emailmin))
```


1. Visualise the distribution of this new variable. Find the mean and the median number of minutes respondents spend on email weekly. Is the mean or the median a better measure of the typical amount of time Americans spend on email weekly? Why?

```{r distribution of email, warnings= FALSE, message=FALSE}
ggplot(email_minutes, aes(x=email))+
  theme_bw() +                #theme
  geom_density(alpha=.2, fill="#FF6666") +
  labs(title="Most people spend on email not more than 120 minutes weekly", 
  y     = "Density",        #changing y-axis label to sentence case
    x     = "Minutes the respondents spend on email weekly") +
 geom_vline(aes(xintercept=mean(email)),
            color="blue", linetype="dashed", size=0.5)
            
#histogram
ggplot(email_minutes, aes(x=email)) + 
 geom_histogram(fill="#FF6666") +
  labs(title="Most people spend on email not more than 120 minutes weekly",
  y = "Number of people",
  x =  "Minutes the respondents spend on email weekly" ) +
 geom_vline(aes(xintercept=median(email)),
            color="black", linetype="dashed", size=0.5)

getmode <- function(v) {
   uniqv <- unique(v)
   uniqv[which.max(tabulate(match(v, uniqv)))]
}

print(c('mean' = mean(email_minutes$email), 
 'median' = median(email_minutes$email),
  'mode' = getmode(email_minutes$email)))

```

As we can see in the density plot, there of tons of outliers in the number of minutes the respondents spend on email weekly and the distribution of email minutes is significantly positive skewed which makes the mean significantly larger than median. Thus, in this case, clearly median is less affected by these outliers and can measure the the typical amount of time Americans spend on email weekly.

1. Using the `infer` package, calculate a 95% bootstrap confidence interval for the mean amount of time Americans spend on email weekly. Interpret this interval in context of the data, reporting its endpoints in “humanized” units (e.g. instead of 108 minutes, report 1 hr and 8 minutes). If you get a result that seems a bit odd, discuss why you think this might be the case.

```{r 95% bootstrap CI for email_minutes, warnings= FALSE, message=FALSE}
# use the infer package to construct a 95% CI for delta
email_mean <- email_minutes %>%
  
  # Specify the variable of interest
  specify(response =email)%>%
  # Generate a bunch of bootstrap samples
  generate(reps =1000, type ="bootstrap")%>%
  # Find the mean of each sample 
  calculate(stat ="mean")

email_ci <- email_mean %>% 
  get_confidence_interval(level = 0.95, type = "percentile")

print(email_ci)
```

As we can see, the 95% bootstrap confidence interval for the mean amount of time Americans spend on email weekly is [6 hr and 23.3 minutes, 7 hr and 28.6 minutes]. We can see that the mean number of minutes respondents spend on email weekly (6 hr and 56.8 minutes) falls in the confidence interval.

1. Would you expect a 99% confidence interval to be wider or narrower than the interval you calculated above? Explain your reasoning.

The 99% confidence interval shall be wider than the 95% confidence interval. Because Increasing the confidence to 99% will increase the margin of error and result in a wider interval.

```{r 99% bootstrap CI for email_minutes, warnings= FALSE, message=FALSE}

email_ci99 <- email_mean %>% 
  get_confidence_interval(level = 0.99, type = "percentile")
print(email_ci99)
```