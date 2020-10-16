---
categories:
- ""
- ""
date: "2017-10-31T22:26:13-05:00"
description: Nullam et orci eu lorem consequat tincidunt vivamus et sagittis magna sed nunc rhoncus condimentum sem.
draft: false
image: drinks.png
keywords: ""
slug: blog3
title: Where Do People Drink The Most Beer, Wine And Spirits?
---

# Where Do People Drink The Most Beer, Wine And Spirits?




```{r load_alcohol_data}
library(fivethirtyeight)
data(drinks)
# or download directly
# alcohol_direct <- read_csv("https://raw.githubusercontent.com/fivethirtyeight/data/master/alcohol-consumption/drinks.csv")
```


What are the variable types? Any missing values we should worry about? 

```{r glimpse_skim_data}
# YOUR CODE GOES HERE
glimpse(drinks)
skim(drinks)
```


Make a plot that shows the top 25 beer consuming countries

```{r beer_plot,fig.height=8}
# YOUR CODE GOES HERE
top_25_beer <- drinks %>% 
  arrange(desc(beer_servings)) %>%
  slice(1:25)
ggplot(top_25_beer, aes(x = reorder(country, beer_servings), y = beer_servings))+ 
  geom_col() + 
  coord_flip() +
  theme(axis.text.x = element_text(size = 14, margin=margin(15,0,0,0)),
        axis.text.y = element_text(size = 14, margin = margin(0,15,0,0)),
        plot.title = element_text(size = 18, face = "bold", margin = margin(0,0,30,0)), 
        axis.title.x = element_text(size=15, face="bold", margin=margin(20,0,0,0)),
        axis.title.y = element_text(size=15, face="bold"))+
  labs(x="Country", 
       y="Beer Servings", 
       title="Top 25 beer serving countries")
```

Make a plot that shows the top 25 wine consuming countries

```{r wine_plot,fig.height=8}
# YOUR CODE GOES HERE
top_25_wine <- drinks %>% 
  arrange(desc(wine_servings)) %>%
  slice(1:25)
ggplot(top_25_wine, aes(x=reorder(country, wine_servings), y=wine_servings)) + 
  geom_col() + 
  coord_flip() +
  theme(axis.text.x = element_text(size = 14, margin=margin(15,0,0,0)),
        axis.text.y = element_text(size = 14, margin = margin(0,15,0,0)),
        plot.title = element_text(size = 18, face = "bold", margin = margin(0,0,30,0)), 
        axis.title.x = element_text(size=15, face="bold", margin=margin(20,0,0,0)),
        axis.title.y = element_text(size=15, face="bold"))+
  labs(x="Country", 
       y="Wine Servings", 
       title="Top 25 wine serving countries")
```

Finally, make a plot that shows the top 25 spirit consuming countries
```{r spirit_plot,fig.height=8}
# YOUR CODE GOES HERE
top_25_spirit <- drinks %>% 
  arrange(desc(spirit_servings)) %>%
  slice(1:25)
ggplot(top_25_spirit, aes(x=reorder(country, spirit_servings), y=spirit_servings))+ 
  geom_col() + 
  coord_flip() +
  theme(axis.text.x = element_text(size = 14, margin=margin(15,0,0,0)),
        axis.text.y = element_text(size = 14, margin = margin(0,15,0,0)),
        plot.title = element_text(size = 18, face = "bold", margin = margin(0,0,30,0)), 
        axis.title.x = element_text(size=15, face="bold", margin=margin(20,0,0,0)),
        axis.title.y = element_text(size=15, face="bold"))+
  labs(x="Country", 
       y="Spirit Servings", 
       title="Top 25 spirit serving countries")
```

What can you infer from these plots? Don't just explain what's in the graph, but speculate or tell a short story (1-2 paragraphs max).

> TYPE YOUR ANSWER AFTER (AND OUTSIDE!) THIS BLOCKQUOTE.
Surprisingly, Namibia is the world’s biggest beer-drinking country, with 376 cans of beer consumed per person. As one could expect, Germany is also on top of the list. On average, German citizens drink 346 cans of beer per year. Further, there is a strong presence of Eastern-European countries, with the Czech Republic, Poland and Lithuania occupying the top positions. France and Portugal are on top when it comes to the consumption of wine, averaging 370 and 339 glasses of wine per person per year, respectively. These results come unsurprisingly and confirms the French as being wine lovers. Grenada is the world’s main spirit drinking country, closely followed by Belarus, Haiti and the Russian Federation. Again, we can confirm some cultural preferences as Russians and Belarusians are known for having appetite towards these drinks. What maybe comes with a bit of surprise is the presence of many Caribbean countries in the top positions. 

All in all, it can be said that, with the exception of wine in France, the countries which occupy the top positions in their respective drinks’ categories, have relatively low prices for their servings. This may partially explain the above average consumption of the respective beverages. 
