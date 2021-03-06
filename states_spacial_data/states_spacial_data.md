---
title: "Spatial Data and Measure Data"
author: "A. Abdias Baldiviezo"
date: "June 30, 2020"
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
# excluded Alaska, American Samoa, Guam, Puerto Rico, Hawaii
states <-us_states(map_date = NULL, states = c("Alabama", "Arizona", "Arkansas", "California", "Colorado", "Connecticut", "Delaware", "District of Columbia", "Florida", "Georgia", "Idaho", "Illinois", "Indiana", "Iowa", "Kansas", "Kentucky", "Louisiana", "Maine", "Maryland", "Massachusetts", "Michigan", "Minnesota", "Minor Outlying Islands", "Mississippi", "Missouri", "Montana", "Nebraska", "Nevada", "New Hampshire", "New Jersey", "New Mexico", "New York", "North Carolina", "North Dakota", "Northern Mariana Islands", "Ohio", "Oklahoma", "Oregon", "Pennsylvania", "Rhode Island", "South Carolina", "South Dakota", "Tennessee", "Texas", "U.S. Virgin Islands", "Utah", "Vermont", "Virginia", "Washington", "West Virginia", "Wisconsin", "Wyoming"))
idaho_county <- us_counties(states = "Idaho")
cc <- census_cities
```

## Background

Up to this point, we have dealt with data that fits into the tidy format without much effort. Spatial data has many complicating factors that have made handling spatial data in R complicated. Big strides are being made to make spatial data tidy in R. However; we are in the middle of the transition.

We will use library(USAboundaries) and library(sf) to make a map of the US and show the top 3 largest cities in each state. Specifically, you will use library(ggplot2) and the function geom_sf() to recreate the provided image.

## Notes

-To read spatial data in R, use read_sf()
-Here we’re loading from a shapefile which is the way spatial data is most commonly stored. Despite the name a shapefile isn’t just one file, but is a collection of files that have the same name, but different extensions. Typically you’ll have four files:

    .shp contains the geometry, and .shx contains an index into that geometry.

    .dbf contains metadata about each geometry (the other columns in the data frame).

    .prf contains the coordinate system and projection information. You’ll learn more about that shortly.

read_sf() can read in the majority of spatial file formats, so don’t worry if your data isn’t in a shapefile; the chances are read_sf() will still be able to read it.
-If you get a spatial object created by another package, us st_as_sf() to convert it to sf.
-Use plot() to show the geometry

## Data Wrangling


```r
# filter out distant states
cc <- cc %>% filter(state_name != "Alaska",
              state_name != "American Samoa",
              state_name != "Guam",
              state_name != "Puerto Rico",
              state_name != "Hawaii") #%>% filter(population > 100000)
#summarize population years
grouped <- cc %>% group_by(city,state_name) %>% summarise(mean(population))
grouped <- ungroup(grouped)
#get top 3 populations in each state
top <- grouped %>% group_by(state_name) %>% top_n(n = 3, wt = `mean(population)`)
g <- select(grouped, city, `mean(population)`)
final <- st_join(top, g, by = "city")
final <- final[1:4]
final <- as.data.frame(final)
#make a table for only the labels which are the biggest city in each state
finalLabels <- final %>% group_by(state_name) %>% top_n(n = 1, wt = `mean(population).x`)
# get the separate coordinates from the sfc_POINT
coordinates <- unlist(finalLabels$geometry) %>% matrix(ncol = 2, byrow = TRUE) %>% as_tibble() %>% setNames(c("x","y"))
#add coordinates to the labels
finalLabels <- bind_cols(finalLabels, coordinates)
#add Idaho by county to the states
states <- bind_rows(states, idaho_county)
```

## Data Visualization


```r
# Use this R-Chunk to plot & visualize your data!
ggplot(data = states) + geom_sf(fill = NA) + geom_sf(data = final, aes(geometry = geometry, size = `mean(population).x`/1000), colour = "#283e66") + geom_label_repel(data = finalLabels, aes(x, y, label = city.x), label.size = 0.25, label.r = 0.25, box.padding = 0.1, size = 2 ) +  theme_light() + labs(size = "Population\n(1,000)", x = "", y = "")
```

![](Task_21_files/figure-html/plot_data-1.png)<!-- -->

## Conclusions

The population mean from all the years that had population collected was taken to be the population aesthetic
