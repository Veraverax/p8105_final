
``` r
library(tidyverse)
```

    ## ── Attaching packages ────────────────────────────────────────────────── tidyverse 1.3.0 ──

    ## ✓ ggplot2 3.3.2     ✓ purrr   0.3.4
    ## ✓ tibble  3.0.3     ✓ dplyr   1.0.2
    ## ✓ tidyr   1.1.2     ✓ stringr 1.4.0
    ## ✓ readr   1.3.1     ✓ forcats 0.5.0

    ## ── Conflicts ───────────────────────────────────────────────────── tidyverse_conflicts() ──
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

Read in wine data.

``` r
year_extract <- function(string) {
  t <- regmatches(string, regexec("[1-2][9|0][0-9][0-9]", string))
  sapply(t, function(x) {
    if(length(x) > 0){
      return(as.numeric(x))
    } else {
      return(NA)    
    }
  })
}
```

``` r
wine_df = 
  read_csv(
  "./wine_data/winemag-data-130k-v2.csv") %>% 
  select(-region_2, -taster_twitter_handle) %>% 
  filter(variety == "Pinot Noir",
         !is.na(region_1)
         ) %>%
  mutate(year = year_extract(title))
```

    ## Warning: Missing column names filled in: 'X1' [1]

    ## Parsed with column specification:
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

``` r
### remove region 2, taster twitter and missing values in region 1.

wine_type <- read_csv("./wine_data/winemag-data-130k-v2.csv") %>% 
            group_by(variety) %>% 
            count() %>% 
            arrange(desc(n)) %>% 
            as.tibble()
```

    ## Warning: Missing column names filled in: 'X1' [1]

    ## Parsed with column specification:
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

    ## Warning: `as.tibble()` is deprecated as of tibble 2.0.0.
    ## Please use `as_tibble()` instead.
    ## The signature and semantics have changed, see `?as_tibble`.
    ## This warning is displayed once every 8 hours.
    ## Call `lifecycle::last_warnings()` to see where this warning was generated.

Separate wine types by four major types: white, red, sparkling

<https://media.winefolly.com/Different-Types-of-Wine-v2.jpg>

Explore the data

``` r
#wine_df %>% 
#  count(taster_name) 

# 17 tasters, could be one of the x variables

#wine_df %>% 
#  count(region_1) %>% 
#  summarise(mean = mean(n))
### more than 200 distinct regions

#wine_df %>% 
#  count(is.na(region_1))

### 1154 NA regions
```

Questions to address for data cleaning and following analysis

  - how to deal w/ NA values in region

  - possible to search for coordinates for the winery

  - how to focus on major wine types

  - find out frequency of wine notes from description and make
    visualization - to each type \*\* red \*\* white \*\* rose \*\*
    sparkling \*\* dessert
    
    \*\* filter by higher ratings

Work breakdown: 1. categorize by major types and subtypes, generate
subsets - Vera and Jady 2. figure out the word cloud functions - Yolanda
3. figure out the geo\_map searching functions - Helen
