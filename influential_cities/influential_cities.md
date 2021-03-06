---
title: "Extra, extra, read all about it"
author: "A. Abdias Baldiviezo"
date: "June 6, 2020"
output:
  html_document:  
    keep_md: true
    toc: true
    toc_float: true
    code_folding: hide
    fig_height: 6
    fig_width: 12
    fig_align: 'center'
---






```r
# Use this R-Chunk to import all your datasets!
ny <- read_csv("https://storybench.org/reinventingtv/abc7ny.csv")
cal <- read_csv("https://storybench.org/reinventingtv/kcra.csv")
cities <- us.cities
```

## Background

We believe if there is anything "of good report or praiseworthy we seek after these things." You are working for management consulting agency A.T. Kearney which produces the Global Cities report (https://www.kearney.com/global-cities/2019). They have put you on a team in charge of developing a new report: the USA Cities report - which identifies the most influential cities in the United States. Your manager would like to explore using the frequency with which a city appears in news headlines as a contributing factor to their city rankings. You find data from two major news outlets: one in California (KCRA) and one in New York (ABC7NY). The data spans July 18, 2017 - Jan 16, 2018. You will use the headlines to find which cities are mentioned most in the news. After creating reproducible R code (i.e. an R markdown file) on this test dataset, your goal is to apply the code to a larger, more up-to-date dataset.

Specifically, you are interested in identifying the 15 cities with the highest headline count overall. You are curious if the city has sustained headlines or if there was a singular event driving the headlines over time. Lastly, they are especially interested in fast growing cities, such as Houston, TX and Charlotte, NC. 

You may want to consider how to deal with the burroughs of New York City. Think about the data you are getting and whether it makes sense.

## Data Wrangling


```r
# # You can run this to calculate the data, but I'll just load it since I already run
# # the calcualtions once
# # consolidate datasets
# general <- bind_rows(cal, ny)
# #narrow down and parse datetime column into only date
# general <- general %>% select(datetime, headline) %>% mutate(datetime = parse_date(str_remove(datetime, "([ ]at[ ][0-9][0-9]:[0-9][0-9][A-Z][A-Z])"), "%B %d %. %Y")) %>% mutate(datetime = months(datetime))
# #build cities
# cities <- cities %>% select(name,country.etc, pop) %>%
#   mutate(name = str_remove(name, "[ ]([A-Z][A-Z])"), headline_mentions = 0, July = 0, August = 0, September = 0, October = 0, November = 0, December = 0, January = 0)
# names(cities)[names(cities) == "country.etc"] <- "state"
# names(cities)[names(cities) == "name"] <- "city"
# 
# #calculate frequency (took like 12 minutes in my i3 laptop)
# for (i in 1:nrow(cities)) {
#   for (j in 1:nrow(general)) {
#     if (str_detect(general$headline[j], str_c("(?i)(", cities$city[i], ")", sep = ""))) {
#       #increase count for mentions
#       cities$headline_mentions[i] <- cities$headline_mentions[i] + 1
#       #increase count on months
#       cities[i, as.character(general$datetime[j])] <- cities[i, as.character(general$datetime[j])] + 1
# 
#     }
#   }
# }
# saveRDS(cities, file = "city_mentions_in_journals.rds")
cities <- read_rds("./city_mentions_in_journals.rds")
sorted_cities <- cities[order(-cities$headline_mentions),]
topfifteen <- sorted_cities[c(1:17),c(1:11)]
```

## Data Visualization


```r
# top 15
ggplot(topfifteen, aes(city, headline_mentions, fill = state)) + geom_bar(stat = "identity") + scale_y_continuous(breaks = seq(0,800,50)) + theme(axis.text.x = element_text(angle = 90)) + labs(title = "15 cities with the most headline mentions", subtitle = "From July 2017 to January 2018")
```

![](Case_Study_07_files/figure-html/plot_data-1.png)<!-- -->

## Graph #1

From thee results the most evident thing is that California cities are amongst the most quoted in the articles, followed by Texas. There is an interesting phenomenon which is Newark, the results of this city are errors, caused by the same city name in 3 different states.


```r
# top 15 (minus Newark)
topfifteen <- filter(topfifteen, city != "Newark")
every_month <- topfifteen %>% pivot_longer(cols = c(July, August, September, October, November, December, January), names_to = "Month", values_to= "new_headline_data") 
#factor
months <- c("July", "August", "September", "October", "November", "December", "January")
every_month$Month <- factor(every_month$Month, levels = months)
#plot
ggplot(every_month, aes(Month, new_headline_data, fill = state)) + geom_bar(stat = "identity") + scale_y_continuous(breaks = seq(0,800,50)) + theme(axis.text.x = element_text(angle = 90)) + facet_wrap(~city, nrow = 2) + labs(title = "14 cities with most headline mentions by month", y = "# headline mentions", subtitle = "From July 2017 to January 2018")
```

![](Case_Study_07_files/figure-html/plot_data2-1.png)<!-- -->

## Graph #2

The most evident thing at first glance is the high amount of mentions for Las Vegas in October, it leads us to think that something very interesting might have happened in that month. We can also see how Sacramento has always a good amount of mentions.


```r
# top 15 (minus Newark)
two_cities <- filter(cities, city == "Houston" | city == "Charlotte")
every_month2 <- two_cities %>% pivot_longer(cols = c(July, August, September, October, November, December, January), names_to = "Month", values_to= "new_headline_data") 
#factor
months <- c("July", "August", "September", "October", "November", "December", "January")
every_month2$Month <- factor(every_month2$Month, levels = months)
#plot
ggplot(every_month2, aes(Month, new_headline_data, fill = state)) + geom_bar(stat = "identity") + scale_y_continuous(breaks = seq(0,800,50)) + theme(axis.text.x = element_text(angle = 90)) + facet_wrap(~city, nrow = 2) + labs(title = "14 cities with most headline mentions by month", y = "# headline mentions", subtitle = "From July 2017 to January 2018")
```

![](Case_Study_07_files/figure-html/plot_data3-1.png)<!-- -->

## Graph #3

We can observate that these two cities peaked during the month of august, suggesting that something happened in this month and that these 2 cities might have been connected in what happened.
