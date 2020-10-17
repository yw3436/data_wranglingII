Reading data
================

``` r
library(tidyverse)
```

    ## ── Attaching packages ──────────────────────────────────────────────────────────────────── tidyverse 1.3.0 ──

    ## ✓ ggplot2 3.3.2     ✓ purrr   0.3.4
    ## ✓ tibble  3.0.3     ✓ dplyr   1.0.2
    ## ✓ tidyr   1.1.2     ✓ stringr 1.4.0
    ## ✓ readr   1.3.1     ✓ forcats 0.5.0

    ## ── Conflicts ─────────────────────────────────────────────────────────────────────── tidyverse_conflicts() ──
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(rvest)
```

    ## Loading required package: xml2

    ## 
    ## Attaching package: 'rvest'

    ## The following object is masked from 'package:purrr':
    ## 
    ##     pluck

    ## The following object is masked from 'package:readr':
    ## 
    ##     guess_encoding

``` r
library(httr)

# add all the other plot options
```

## Script a table

First, we want the first table from the web. read in the HTML

``` r
url = "http://samhda.s3-us-gov-west-1.amazonaws.com/s3fs-public/field-uploads/2k15StateFiles/NSDUHsaeShortTermCHG2015.htm" # save the url to the variable called url
drug_use_html = read_html(url)

drug_use_html
```

    ## {html_document}
    ## <html lang="en">
    ## [1] <head>\n<link rel="P3Pv1" href="http://www.samhsa.gov/w3c/p3p.xml">\n<tit ...
    ## [2] <body>\r\n\r\n<noscript>\r\n<p>Your browser's Javascript is off. Hyperlin ...

extract the table(s); focus on the first one

``` r
tabl_marj = 
  drug_use_html %>% 
  html_nodes(css = "table") %>% ## transfer the original data into table; 15 tables in total
  first() %>%  # take the first one; also there is end() or other similar functions
  html_table() %>% 
  slice(-1) %>% 
  as_tibble()
```

## CSS Selector: star wars movie info

get info from the web.

``` r
url = "https://www.imdb.com/list/ls070150896/"

swm_html = 
  read_html("https://www.imdb.com/list/ls070150896/")
```

grab elements that we need:

``` r
title_vector =
  swm_html %>% 
  html_nodes(css = ".lister-item-header a") %>% 
  html_text()

gross_rev_vec =
  swm_html %>% 
  html_nodes(css = ".text-small:nth-child(7) span:nth-child(5)") %>% 
  html_text()

runtime_vec = 
  swm_html %>% 
  html_nodes(css = ".runtime") %>% 
  html_text()

swm_df = 
  tibble(
    title = title_vector,
    rev = gross_rev_vec,
    runtime = runtime_vec)
```

## APIs

Get some water data

``` r
nyc_water = 
  GET("https://data.cityofnewyork.us/resource/ia2d-e54m.csv") %>% 
  content("parsed") # a function in httr package
```

    ## Parsed with column specification:
    ## cols(
    ##   year = col_double(),
    ##   new_york_city_population = col_double(),
    ##   nyc_consumption_million_gallons_per_day = col_double(),
    ##   per_capita_gallons_per_person_per_day = col_double()
    ## )

``` r
nyc_water_json = 
  GET("https://data.cityofnewyork.us/resource/ia2d-e54m.json") %>% 
  content("text") %>% 
  jsonlite::fromJSON() %>% 
  as_tibble()
```

## BREFSS dataset

same process, different data

``` r
brfss_smart2010 = 
  GET("https://chronicdata.cdc.gov/resource/acme-vg9e.csv",
      query = list("$limit" = 5000)) %>% # to request a query that is a list with the query parameter of 5000 $limit
  content("parsed")
```

    ## Parsed with column specification:
    ## cols(
    ##   .default = col_character(),
    ##   year = col_double(),
    ##   sample_size = col_double(),
    ##   data_value = col_double(),
    ##   confidence_limit_low = col_double(),
    ##   confidence_limit_high = col_double(),
    ##   display_order = col_double(),
    ##   locationid = col_logical()
    ## )

    ## See spec(...) for full column specifications.

## some data aren’t so nice

Let’s look at Pokemon

``` r
poke = 
  GET("http://pokeapi.co/api/v2/pokemon/1") %>%
  content()

poke$name
```

    ## [1] "bulbasaur"

``` r
poke$height
```

    ## [1] 7

``` r
poke$abilities
```

    ## [[1]]
    ## [[1]]$ability
    ## [[1]]$ability$name
    ## [1] "overgrow"
    ## 
    ## [[1]]$ability$url
    ## [1] "https://pokeapi.co/api/v2/ability/65/"
    ## 
    ## 
    ## [[1]]$is_hidden
    ## [1] FALSE
    ## 
    ## [[1]]$slot
    ## [1] 1
    ## 
    ## 
    ## [[2]]
    ## [[2]]$ability
    ## [[2]]$ability$name
    ## [1] "chlorophyll"
    ## 
    ## [[2]]$ability$url
    ## [1] "https://pokeapi.co/api/v2/ability/34/"
    ## 
    ## 
    ## [[2]]$is_hidden
    ## [1] TRUE
    ## 
    ## [[2]]$slot
    ## [1] 3

## Closing thoughts
