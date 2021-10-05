Vignette
================
Kaylee Frazier
9/27/2021

-   [Required Packages](#required-packages)
-   [API Interaction Functions](#api-interaction-functions)
    -   [`changeToDF`](#changetodf)
    -   [`CME.Analysis`](#cmeanalysis)
-   [’Speed.Analysis\`](#speedanalysis)
-   [`solarFlare`](#solarflare)
-   [`hazardousAsteroid`](#hazardousasteroid)
-   [`findName`](#findname)

This is a vignette to demonstrate how to collect data from an
[API](https://en.wikipedia.org/wiki/API). I’ll be using the [NASA
API](https://api.nasa.gov/index.html). With this API, I’m going to
create functions to interact with endpoints and explore the data in this
API.

# Required Packages

To use the functions for interacting with the NASA API, I used the
following packages:

-   [`tidyverse`](https://www.tidyverse.org/): useful features for data
    science
-   [`jsonlite`](https://cran.r-project.org/web/packages/jsonlite/): API
    interaction
-   [`httr`](https://cran.r-project.org/web/packages/httr/index.html):
    tools for dealing with URLs and HTTP

``` r
# Load in the packages.
library(tidyverse)
library(jsonlite)
library(httr)
```

# API Interaction Functions

Below are functions I have created to help interact with the [NASA
APIs](https://api.nasa.gov/index.html). These APIs provide insights,
images, and meaningful information collected by NASA’s various
resources.

## `changeToDF`

This function changes a list element to a data frame.

``` r
changeToDF <- function(x) {
  do.call(rbind.data.frame, x)
}
```

## `CME.Analysis`

This function takes data from the Coronal Mass Ejection (CME) Analysis
endpoint. This function selects the type, longitude, latitude, and speed
variables. Then, it filters by the type variable. A user can choose what
type they are searching for and the function will return that type’s
longitude, latitude, and speed.

``` r
#start the function
CME.Analysis <- function(x) {
#get data from CME Analysis API
dataCME <- GET("https://api.nasa.gov/DONKI/CME?startDate=yyyy-MM-dd&endDate=yyyy-MM-dd&api_key=b7Y6xwkXiwoOc5RRa38biqLuuhcwzC2MoGZ70ecM&feedtype=json&ver=1.0")
#parse the data
parsedCME <- dataCME$content %>% rawToChar() %>% fromJSON()
  #change the list to a data frame
dfCME <- changeToDF(parsedCME$cmeAnalyses)
#select and filter the values you want for the data frame
  dfCME %>% filter(type == x) %>% select(type, longitude, latitude, speed)
}

#example
CME.Analysis("C")
```

    ##    type longitude latitude speed
    ## 1     C         0        7   537
    ## 2     C       117      -12   557
    ## 3     C        NA      -54   572
    ## 4     C      -117      -29   652
    ## 5     C      -148      -26   577
    ## 6     C       -61      -30   511
    ## 7     C      -100      -41   994
    ## 8     C       -80      -37   991
    ## 9     C       158       30   681
    ## 10    C       -24      -29   766
    ## 11    C       -24      -31   710
    ## 12    C       116      -10   675
    ## 13    C        15      -21   503
    ## 14    C        43      -17   649
    ## 15    C       107      -28   769

# ’Speed.Analysis\`

This function takes data from the CME API and groups the data by the
speed of the coronal mass ejection. I split the categories of speed into
low medium and high, and the values for the different categories of
speed can be customized to whichever speeds the user wants.

``` r
#start the function
Speed.Analysis <- function(x, y) {
#get data from CME Analysis API
dataCME <- GET("https://api.nasa.gov/DONKI/CME?startDate=yyyy-MM-dd&endDate=yyyy-MM-dd&api_key=b7Y6xwkXiwoOc5RRa38biqLuuhcwzC2MoGZ70ecM&feedtype=json&ver=1.0")
#parse the data
parsedCME <- dataCME$content %>% rawToChar() %>% fromJSON()
#change the list to a data frame
speed.df <- changeToDF(parsedCME$cmeAnalyses)
#select the variables you want and add a new variable "speedPace"
speed.df %>% mutate(speedPace = if_else(speed >= x, "High", 
    if_else(speed >= y, "Medium", "Low"))) %>% group_by(speedPace) %>% select(speedPace, type, latitude, longitude, halfAngle, speed)
}

#example
Speed.Analysis(500, 400)
```

    ## # A tibble: 61 x 6
    ## # Groups:   speedPace [3]
    ##    speedPace type  latitude longitude halfAngle speed
    ##    <chr>     <chr>    <dbl>     <dbl>     <dbl> <dbl>
    ##  1 Medium    S          -34       131        11   438
    ##  2 High      C            7         0        45   537
    ##  3 Low       S            7       114        29   338
    ##  4 High      C          -12       117        22   557
    ##  5 Low       S          -13        56        40   287
    ##  6 Low       S            1        59        23   388
    ##  7 Low       S          -43       154        18   360
    ##  8 Low       S          -35      -160        26   383
    ##  9 Medium    S          -38      -174        22   451
    ## 10 Medium    S          -40      -160        18   481
    ## # ... with 51 more rows

# `solarFlare`

This function grabs data from the solar flare (FLR) endpoint and returns
information about the flareID, the source location, the class type, and
the region number. Even though the original data went by the variable
names above, this function lets you use whichever names you want for the
columns with a being the first column, b being the second column, etc.

``` r
#start the function
solarFlare <- function(a, b, c, d) {
#grab data from the solar flare API
dataFLR <- GET("https://api.nasa.gov/DONKI/FLR?startDate=2000-01-01&endDate=2020-01-01&api_key=b7Y6xwkXiwoOc5RRa38biqLuuhcwzC2MoGZ70ecM")
#parse the data
parsedFLR <- dataFLR$content %>% rawToChar() %>% fromJSON()
#combine variables in the data
FLR <- cbind(parsedFLR$flrID, parsedFLR$sourceLocation, parsedFLR$activeRegionNum, parsedFLR$classType)
#make the data into a data frame
FLRdf <- as_tibble(FLR)
#change the column names
colnames(FLRdf) <- c(a, b, c, d)
FLRdf %>% select(everything())
}

#example
solarFlare("flrID","sourceLocation", "RegionNum", "classType")
```

    ## Warning: The `x` argument of `as_tibble.matrix()` must have unique column names if `.name_repair` is omitted as of tibble 2.0.0.
    ## Using compatibility `.name_repair`.

    ## # A tibble: 575 x 4
    ##    flrID                       sourceLocation RegionNum classType
    ##    <chr>                       <chr>          <chr>     <chr>    
    ##  1 2010-04-03T09:04:00-FLR-001 S25W03         11059     B7.4     
    ##  2 2010-06-12T00:30:00-FLR-001 N22W43         11081     M2.0     
    ##  3 2010-08-07T17:55:00-FLR-001 N14E37         11093     M1.0     
    ##  4 2010-08-14T09:38:00-FLR-001 N11W65         11093     C4.4     
    ##  5 2010-08-18T04:45:00-FLR-001 N18W88         11099     C4.5     
    ##  6 2010-10-16T19:07:00-FLR-001 S20W26         11112     M2.9     
    ##  7 2011-01-28T00:44:00-FLR-001 N17W88         11149     M1.3     
    ##  8 2011-02-09T01:23:00-FLR-001 N16W70         11153     M1.9     
    ##  9 2011-02-13T17:28:00-FLR-001 S20E04         11158     M6.6     
    ## 10 2011-02-14T17:20:00-FLR-001 S20W04         11158     M2.2     
    ## # ... with 565 more rows

# `hazardousAsteroid`

This function takes data from the Asteroids - NeoWs API and returns data
about charted asteroids. This function allows you to find whichever
summary statistic you want, whether it’s the mean, median, standard
deiation, etc., for the “absolute\_magnitude\_h” variable when
“is\_potentially\_hazardous” equals true or false.

``` r
#start the function
hazardousAsteroid <- function(stat) {
#grab data from the Asteroids API
dataAsteroid <- GET("https://api.nasa.gov/neo/rest/v1/neo/browse?api_key=b7Y6xwkXiwoOc5RRa38biqLuuhcwzC2MoGZ70ecM") 
#parse the data
parsedAsteroid <- dataAsteroid$content %>% rawToChar() %>% fromJSON()
#grab information from the near_earth_objects data frame
Asteroid <- parsedAsteroid$near_earth_objects
#calculate mean magnitude when the asteroid is and is not hazardous
Magnitude <- aggregate(Asteroid$absolute_magnitude_h, list(Asteroid$is_potentially_hazardous_asteroid), FUN=stat)
Magnitude <- as_tibble(Magnitude)
#create column names
colnames(Magnitude) <- c("is_potentially_hazardous_asteriod", "absolute_magnitude_h")
return(Magnitude)
}

#example
hazardousAsteroid(mean)
```

    ## # A tibble: 2 x 2
    ##   is_potentially_hazardous_asteriod absolute_magnitude_h
    ##   <lgl>                                            <dbl>
    ## 1 FALSE                                             14.4
    ## 2 TRUE                                              15.9

# `findName`

This function takes information from the Asteroids API. In this function
you put in any name of the asteroid you want and it returns the
asteriod’s name, id, designation, and if it was hazardous.

``` r
findName <- function(x) {
  #grab data from the Asteroids API
dataAsteroid <- GET("https://api.nasa.gov/neo/rest/v1/neo/browse?api_key=b7Y6xwkXiwoOc5RRa38biqLuuhcwzC2MoGZ70ecM") 
#parse the data
parsedAsteroid <- dataAsteroid$content %>% rawToChar() %>% fromJSON()
#grab information from the near_earth_objects data frame
Asteroid <- parsedAsteroid$near_earth_objects
#make the output a tibble
Asteroid <- as_tibble(Asteroid)
#select the variables you want outputted and filter by the name of the asteroid
Asteroid %>% filter(name_limited == x) %>% select(name_limited, id, designation, absolute_magnitude_h, is_potentially_hazardous_asteroid)
}

#example
findName("Eros")
```

    ## # A tibble: 1 x 5
    ##   name_limited id      designation absolute_magnitude_h is_potentially_hazardou~
    ##   <chr>        <chr>   <chr>                      <dbl> <lgl>                   
    ## 1 Eros         2000433 433                         10.4 FALSE
