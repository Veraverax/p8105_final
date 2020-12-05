Data Cleaning
================

``` r
library(tidyverse)
library(leaflet)
library(ggplot2)
library(tigris)
library(readxl)
```

Read in wine data.

``` r
year_extract <- function(string) {
  t <- regmatches(string, regexec("[1][9][4-9][0-9]|[2][0-9][0-9][0-9]", string))
  sapply(t, function(x) {
    if (length(x) > 0) {
      return(as.numeric(x))
    } else {
      return(NA)    
    }
  })
}
```

Read in the wine data and generate year variable and save as csv file.

``` r
wine_df = 
  read_csv(
  "./wine_data/winemag-data-130k-v2.csv") %>% 
  select(-region_2, -taster_twitter_handle, -X1) %>% 
  mutate(year = year_extract(title))
```

    ## Warning: Missing column names filled in: 'X1' [1]

    ## 
    ## ── Column specification ────────────────────────────────────────────────────────
    ## cols(
    ##   X1 = col_double(),
    ##   country = col_character(),
    ##   description = col_character(),
    ##   designation = col_character(),
    ##   points = col_double(),
    ##   price = col_double(),
    ##   province = col_character(),
    ##   region_1 = col_character(),
    ##   region_2 = col_character(),
    ##   taster_name = col_character(),
    ##   taster_twitter_handle = col_character(),
    ##   title = col_character(),
    ##   variety = col_character(),
    ##   winery = col_character()
    ## )

##### Code new dummy variables for old/new world wine:

The new-world and old-world wines are categorized mainly by the country
of origin. The detailed introduction can be found
at：<https://winefolly.com/deep-dive/new-world-vs-old-world-wine/>.

``` r
old_world_country = c("France", "Italy", "Portugal", "Spain", "Germany", "Hungary", "Croatia", "England")

new_world_country = c("US", "Canada", "Argentina", "Australia", "New Zealand", "South Africa", "China")

wine_df = 
  wine_df %>% 
  mutate(new_world = country %in% new_world_country,
         old_world = country %in% old_world_country)
```

Separate wine types by four major types: white, red, sparkling

<https://media.winefolly.com/Different-Types-of-Wine-v2.jpg>

# Subset data to four types of wine: red, white, sparkling, and rose.

``` r
wine_type = 
    read_xlsx(
  "./wine_data/wine type.xlsx")  

wine_df = 
  wine_df %>% 
  mutate(type = ifelse(variety %in% wine_type$red, "red", 
                       ifelse(variety %in% wine_type$white, "white", 
                              ifelse(variety %in% wine_type$sparkling, "sparkling", 
                                     ifelse(variety %in% wine_type$rose, "rose", NA )))))

red_df = 
  wine_df %>% 
    filter(!is.na(type),
           type == "red")
  
white_df = 
  wine_df %>% 
    filter(!is.na(type),
           type == "white")

sparkling_df = 
  wine_df %>% 
    filter(!is.na(type),
           type == "sparkling")

rose_df = 
  wine_df %>% 
    filter(!is.na(type),
           type == "rose")
```

``` r
write.csv(wine_df, file = "./wine_data/tidy/wine_all.csv")
write.csv(red_df, file = "./wine_data/tidy/wine_red.csv")
write.csv(white_df, file = "./wine_data/tidy/wine_white.csv")
write.csv(sparkling_df, file = "./wine_data/tidy/wine_sparkling.csv")
write.csv(rose_df, file = "./wine_data/tidy/wine_rose.csv")
```

Questions to address for data cleaning and following analysis

  - how to deal w/ NA values in region

  - possible to search for coordinates for the winery

  - how to focus on major wine types

  - find out frequency of wine notes from description and make
    visualization - to each type (filter by higher ratings) \*\* red
    \*\* white \*\* rose \*\* sparkling \*\* dessert

Work breakdown: 1. categorize by major types and subtypes, generate
subsets - Vera and Jady 2. figure out the word cloud functions - Yolanda
3. figure out the geo\_map searching functions - Helen
