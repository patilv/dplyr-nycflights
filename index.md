---
title       : dplyr
subtitle    : with nycflights13 
author      : Most of the tutorial from Kevin Markham
job         : And Hadley Wikham's dplyr resources without which nothing exists
framework   : io2012        # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight.js  # {highlight.js, prettify, highlight}
hitheme     : tomorrow      # 
widgets     : [mathjax]            # {mathjax, quiz, bootstrap}
mode        : selfcontained # {standalone, draft}
---

## What is dplyr?

* Tools for data exploration and transformation
* Intuitive to write and easy to read
* Super-fast on data frames

---
## Installing


```r
if (packageVersion("devtools") < 1.6) {
  install.packages("devtools")
}
devtools::install_github("hadley/lazyeval")
devtools::install_github("hadley/dplyr")
install.packages("nycflights13")
```

---

##  Load our data package


```r
library(nycflights13)
suppressMessages(library(dplyr))
```

Package `nycflights13` has data about all flights that departed NYC in 2013 -  5 datasets

1. `flights`:  Flights data
2. `airlines`: Airline names
3. `airports`: Airport metadata
4. `planes` : Plane metadata
5. `weather`: Hourly weather data

---
## Look at `flights` data


```r
# explore data
data(flights)
dim(flights)
```

```
## [1] 336776     16
```

```r
head(flights,3)
```

```
## Source: local data frame [3 x 16]
## 
##   year month day dep_time dep_delay arr_time arr_delay carrier tailnum
## 1 2013     1   1      517         2      830        11      UA  N14228
## 2 2013     1   1      533         4      850        20      UA  N24211
## 3 2013     1   1      542         2      923        33      AA  N619AA
## Variables not shown: flight (int), origin (chr), dest (chr), air_time
##   (dbl), distance (dbl), hour (dbl), minute (dbl)
```

336,776 rows... because of its size, this particular dataset has been wrapped, by default, using `tbl_df` (what's that?)

---
## tbl_df

Prints nicely and prevents an accidental display of the whole dataset


```r
tblflights <- tbl_df(flights)
head(tblflights,3) # Can also use print(flights,3) instead
```

```
## Source: local data frame [3 x 16]
## 
##   year month day dep_time dep_delay arr_time arr_delay carrier tailnum
## 1 2013     1   1      517         2      830        11      UA  N14228
## 2 2013     1   1      533         4      850        20      UA  N24211
## 3 2013     1   1      542         2      923        33      AA  N619AA
## Variables not shown: flight (int), origin (chr), dest (chr), air_time
##   (dbl), distance (dbl), hour (dbl), minute (dbl)
```

---
## But if one insists, we can always revert back to a regular dataframe

```r
# convert to a normal data frame to see all of the columns
head(data.frame(tblflights),3)
```

```
##   year month day dep_time dep_delay arr_time arr_delay carrier tailnum
## 1 2013     1   1      517         2      830        11      UA  N14228
## 2 2013     1   1      533         4      850        20      UA  N24211
## 3 2013     1   1      542         2      923        33      AA  N619AA
##   flight origin dest air_time distance hour minute
## 1   1545    EWR  IAH      227     1400    5     17
## 2   1714    LGA  IAH      227     1416    5     33
## 3   1141    JFK  MIA      160     1089    5     42
```

---
## Basic single table (df) verbs

1. `filter`: for subsetting variables
2. `select`: for subsetting rows
3. `arrange`: for re-ordering rows
4. `mutate`: for adding new columns
5. `summarise` or `summarize`: for reducing each group to a smaller number of summary statistics 

---

## filter: Keep rows matching criteria


```r
# base R: tblflights[tblflights$carrier=="AA" & tblflights$origin=="LGA", ]
filter(tblflights, carrier=="AA" & origin=="LGA")
```

```
## Source: local data frame [15,459 x 16]
## 
##    year month day dep_time dep_delay arr_time arr_delay carrier tailnum
## 1  2013     1   1      558        -2      753         8      AA  N3ALAA
## 2  2013     1   1      559        -1      941        31      AA  N3DUAA
## 3  2013     1   1      623        13      920         5      AA  N3EMAA
## 4  2013     1   1      629        -1      824        14      AA  N3CYAA
## 5  2013     1   1      635         0     1028        48      AA  N3GKAA
## 6  2013     1   1      656        -4      854         4      AA  N4WNAA
## 7  2013     1   1      659        -1     1008        -7      AA  N3EKAA
## 8  2013     1   1      724        -6     1111        31      AA  N541AA
## 9  2013     1   1      739        -6      918       -12      AA  N4WPAA
## 10 2013     1   1      753        -2     1056       -14      AA  N3HMAA
## ..  ...   ... ...      ...       ...      ...       ...     ...     ...
## Variables not shown: flight (int), origin (chr), dest (chr), air_time
##   (dbl), distance (dbl), hour (dbl), minute (dbl)
```

```r
# same as filter(tblflights, carrier=="AA", origin=="LGA")
```

---
## filter again


```r
filter(tblflights, carrier=="AA" | carrier=="UA") 
```

```
## Source: local data frame [91,394 x 16]
## 
##    year month day dep_time dep_delay arr_time arr_delay carrier tailnum
## 1  2013     1   1      517         2      830        11      UA  N14228
## 2  2013     1   1      533         4      850        20      UA  N24211
## 3  2013     1   1      542         2      923        33      AA  N619AA
## 4  2013     1   1      554        -4      740        12      UA  N39463
## 5  2013     1   1      558        -2      753         8      AA  N3ALAA
## 6  2013     1   1      558        -2      924         7      UA  N29129
## 7  2013     1   1      558        -2      923       -14      UA  N53441
## 8  2013     1   1      559        -1      941        31      AA  N3DUAA
## 9  2013     1   1      559        -1      854        -8      UA  N76515
## 10 2013     1   1      606        -4      858       -12      AA  N633AA
## ..  ...   ... ...      ...       ...      ...       ...     ...     ...
## Variables not shown: flight (int), origin (chr), dest (chr), air_time
##   (dbl), distance (dbl), hour (dbl), minute (dbl)
```

---
## filter again


```r
filter(tblflights, carrier %in% c("AA", "UA"))
```

```
## Source: local data frame [91,394 x 16]
## 
##    year month day dep_time dep_delay arr_time arr_delay carrier tailnum
## 1  2013     1   1      517         2      830        11      UA  N14228
## 2  2013     1   1      533         4      850        20      UA  N24211
## 3  2013     1   1      542         2      923        33      AA  N619AA
## 4  2013     1   1      554        -4      740        12      UA  N39463
## 5  2013     1   1      558        -2      753         8      AA  N3ALAA
## 6  2013     1   1      558        -2      924         7      UA  N29129
## 7  2013     1   1      558        -2      923       -14      UA  N53441
## 8  2013     1   1      559        -1      941        31      AA  N3DUAA
## 9  2013     1   1      559        -1      854        -8      UA  N76515
## 10 2013     1   1      606        -4      858       -12      AA  N633AA
## ..  ...   ... ...      ...       ...      ...       ...     ...     ...
## Variables not shown: flight (int), origin (chr), dest (chr), air_time
##   (dbl), distance (dbl), hour (dbl), minute (dbl)
```

---
## select: Pick columns by name


```r
# base R approach to select DepTime, ArrTime, and FlightNum columns
head(tblflights[, c("dep_time", "arr_time", "flight")])
```

```
## Source: local data frame [6 x 3]
## 
##   dep_time arr_time flight
## 1      517      830   1545
## 2      533      850   1714
## 3      542      923   1141
## 4      544     1004    725
## 5      554      812    461
## 6      554      740   1696
```

---
## select again


```r
# dplyr approach
print(select(tblflights, dep_time, arr_time, flight),n=6)
```

```
## Source: local data frame [336,776 x 3]
## 
##    dep_time arr_time flight
## 1       517      830   1545
## 2       533      850   1714
## 3       542      923   1141
## 4       544     1004    725
## 5       554      812    461
## 6       554      740   1696
## ..      ...      ...    ...
```

---
## select again


```r
# use colon to select multiple contiguous columns, and use `contains` to match columns by name
# note: `starts_with`, `ends_with`, and `matches` (for regular expressions) can also be used to match columns by name
head(select(tblflights, flight:dest, contains("arr"), contains("dep")))
```

```
## Source: local data frame [6 x 8]
## 
##   flight origin dest arr_time arr_delay carrier dep_time dep_delay
## 1   1545    EWR  IAH      830        11      UA      517         2
## 2   1714    LGA  IAH      850        20      UA      533         4
## 3   1141    JFK  MIA      923        33      AA      542         2
## 4    725    JFK  BQN     1004       -18      B6      544        -1
## 5    461    LGA  ATL      812       -25      DL      554        -6
## 6   1696    EWR  ORD      740        12      UA      554        -4
```

---
## Chaining over Nesting?


```r
# nesting method to select carrier and dep_delay columns and filter for delays over 60 minutes
head(filter(select(tblflights, carrier, dep_delay), dep_delay > 60))
```

```
## Source: local data frame [6 x 2]
## 
##   carrier dep_delay
## 1      MQ       101
## 2      AA        71
## 3      MQ       853
## 4      UA       144
## 5      UA       134
## 6      EV        96
```

---
## Chaining over Nesting


```r
tblflights %>%
    select(carrier, dep_delay) %>%
    filter(dep_delay > 60) %>%
    head()
```

```
## Source: local data frame [6 x 2]
## 
##   carrier dep_delay
## 1      MQ       101
## 2      AA        71
## 3      MQ       853
## 4      UA       144
## 5      UA       134
## 6      EV        96
```

---
## Chaining

* Chaining increases readability significantly when there are many commands
* Operator is automatically imported from the [magrittr](https://github.com/smbache/magrittr) package
* Can be used to replace nesting in R commands outside of dplyr

---
## Chaining


```r
# create two vectors and calculate Euclidian distance between them
x1 <- 1:5; x2 <- 2:6

# Usual 
sqrt(sum((x1-x2)^2))
```

```
## [1] 2.236
```

```r
# chaining method
(x1-x2)^2 %>% sum() %>% sqrt()
```

```
## [1] 2.236
```

---
## arrange: Reorder rows


```r
# base R approach to select carrier and dep_delay columns and sort by dep_delay
head(tblflights[order(tblflights$dep_delay), c("carrier", "dep_delay")])
```

```
## Source: local data frame [6 x 2]
## 
##   carrier dep_delay
## 1      B6       -43
## 2      DL       -33
## 3      EV       -32
## 4      DL       -30
## 5      F9       -27
## 6      MQ       -26
```

---
## arrange


```r
# dplyr approach
tblflights %>%
    select(carrier, dep_delay) %>%
    arrange(dep_delay) %>% # arrange(desc(dep_delay)) for descending order
    head()
```

```
## Source: local data frame [6 x 2]
## 
##   carrier dep_delay
## 1      B6       -43
## 2      DL       -33
## 3      EV       -32
## 4      DL       -30
## 5      F9       -27
## 6      MQ       -26
```

---
## mutate: create new variables that are functions of existing variables


```r
# base R approach to create a new variable - sum of squares of delays (arr and dep)
tblflights$delaysquare <- tblflights$dep_delay^2 + tblflights$arr_delay^2
head(tblflights[, c("dep_delay", "arr_delay", "delaysquare")])
```

```
## Source: local data frame [6 x 3]
## 
##   dep_delay arr_delay delaysquare
## 1         2        11         125
## 2         4        20         416
## 3         2        33        1093
## 4        -1       -18         325
## 5        -6       -25         661
## 6        -4        12         160
```

---
## mutate


```r
# dplyr approach (prints the new variable but does not store it)
tblflights %>%
    select(dep_delay, arr_delay) %>%
    mutate(delaysquare = dep_delay^2 + arr_delay^2) %>%
    head()
```

```
## Source: local data frame [6 x 3]
## 
##   dep_delay arr_delay delaysquare
## 1         2        11         125
## 2         4        20         416
## 3         2        33        1093
## 4        -1       -18         325
## 5        -6       -25         661
## 6        -4        12         160
```

```r
# store the new variable: tblflights <- tblflights %>% 
#         mutate(delaysquare = dep_delay^2 + arr_delay^2) 
```

---
## summarise/summarize: Reduce multiple variables to values

* Primarily useful with data that has been grouped by one or more variables
* `group_by` creates the groups that will be operated on
* `summarise` uses the provided aggregation function to summarise each group


```r
# base R approaches to calculate the mean arrival delays at different airports
aggregate(arr_delay ~ origin, tblflights, mean)
```

```
##   origin arr_delay
## 1    EWR     9.107
## 2    JFK     5.551
## 3    LGA     5.783
```

```r
# or with(tblflights, tapply(arr_delay, origin, mean, na.rm=TRUE))
```

---
## summarise


```r
# dplyr approach: create a table grouped by origin, and then summarise each group by taking the mean of arr_delay
tblflights %>%
    group_by(origin) %>%
    summarise(avg_delay = mean(arr_delay, na.rm=TRUE))
```

```
## Source: local data frame [3 x 2]
## 
##   origin avg_delay
## 1    EWR     9.107
## 2    JFK     5.551
## 3    LGA     5.783
```

---
## summarise_each/mutate_each: apply the same summary/mutate function(s) to multiple columns at once


```r
# for each carrier, calculate the mean arrival and departure delays at the different origin airports
tblflights %>%
    group_by(origin) %>%
    summarise_each(funs(mean(.,na.rm=TRUE)), arr_delay, dep_delay) %>%
    head()
```

```
## Source: local data frame [3 x 3]
## 
##   origin arr_delay dep_delay
## 1    EWR     9.107     15.11
## 2    JFK     5.551     12.11
## 3    LGA     5.783     10.35
```

---
## summarise_each


```r
# for each carrier, calculate the minimum and maximum of arrival and departure delays
tblflights %>%
    group_by(carrier) %>%
    summarise_each(funs(min(., na.rm=TRUE), max(., na.rm=TRUE)), matches("_delay")) %>%
    head()
```

```
## Source: local data frame [6 x 5]
## 
##   carrier dep_delay_min arr_delay_min dep_delay_max arr_delay_max
## 1      9E           -24           -68           747           744
## 2      AA           -24           -75          1014          1007
## 3      AS           -21           -74           225           198
## 4      B6           -43           -71           502           497
## 5      DL           -33           -71           960           931
## 6      EV           -32           -62           548           577
```

---
## mutate_each 

```r
tblflights %>%
    select(matches("_delay")) %>%
    head(3)
```

```
## Source: local data frame [3 x 2]
## 
##   dep_delay arr_delay
## 1         2        11
## 2         4        20
## 3         2        33
```

```r
tblflights %>%
    select(matches("_delay")) %>%
    mutate_each(funs(half=./2)) %>%
    head(3)
```

```
## Source: local data frame [3 x 2]
## 
##   dep_delay arr_delay
## 1         1       5.5
## 2         2      10.0
## 3         1      16.5
```

---
## n():  counts the number of rows in a group


```r
# for each month, count the total number of flights each day and sort in order of busiest days
tblflights %>%
    group_by(month,day) %>%
    summarise(flight_count = n()) %>%
    arrange(desc(flight_count)) %>%
    head()
```

```
## Source: local data frame [6 x 3]
## Groups: month
## 
##   month day flight_count
## 1     1   2          943
## 2     1   7          933
## 3     1  10          932
## 4     1  11          930
## 5     1  14          928
## 6     1  31          928
```


```r
# rewrite more simply with the `tally` function
tblflights %>%
    group_by(month, day) %>%
    tally(sort = TRUE)
```

---
## n_distinct(vector): counts the number of unique items in that vector


```r
# for each destination, count the total number of flights and the number of distinct planes that flew there
tblflights %>%
    group_by(dest) %>%
    summarise(flight_count = n(), plane_count = n_distinct(tailnum)) %>%
    head()
```

```
## Source: local data frame [6 x 3]
## 
##   dest flight_count plane_count
## 1  ABQ          254         108
## 2  ACK          265          58
## 3  ALB          439         172
## 4  ANC            8           6
## 5  ATL        17215        1180
## 6  AUS         2439         993
```

---
## Grouping without summarising


```r
# for each destination, show the number of flights from the 3 origin airports 
tblflights %>%
    group_by(dest) %>%
    select(origin) %>%
    table() %>%
    head()
```

```
##      origin
## dest   EWR  JFK   LGA
##   ABQ    0  254     0
##   ACK    0  265     0
##   ALB  439    0     0
##   ANC    8    0     0
##   ATL 5022 1930 10263
##   AUS  968 1471     0
```

---
## Window Functions

* Aggregation function (like `mean`) takes n inputs and returns 1 value
* [Window function](http://cran.r-project.org/web/packages/dplyr/vignettes/window-functions.html) takes n inputs and returns n values
* Includes ranking and ordering functions (like `min_rank`), offset functions (`lead` and `lag`), and cumulative aggregates (like `cummean`).

---
## Try this


```r
# for each carrier, calculate which two days of the year they had their longest departure delays
# note: smallest (not largest) value is ranked as 1, so you have to use `desc` to rank by largest value
tblflights %>%
    group_by(carrier) %>%
    select(month, day, dep_delay) %>%
    filter(min_rank(desc(dep_delay)) <= 2 & dep_delay!="NA") %>%
    arrange(carrier, desc(dep_delay)) %>%
    head()
```

```
## Source: local data frame [6 x 4]
## Groups: carrier
## 
##   carrier month day dep_delay
## 1      9E     2  16       747
## 2      9E     7  24       430
## 3      AA     9  20      1014
## 4      AA    12   5       896
## 5      AS     5  23       225
## 6      AS     1  20       222
```

---
## Other things to play with


```r
# for each carrier, calculate which two days of the year they had their longest departure delays--- rewrite previous with the `top_n` function
tblflights %>%
    group_by(carrier) %>%
    select(month, day, dep_delay) %>%
    filter(dep_delay!="NA") %>%
    top_n(2) %>%
    arrange(carrier, desc(dep_delay)) %>% head()

# for each month, calculate the number of flights and the change from the previous month
tblflights %>%
    group_by(month) %>%
    summarise(flight_count = n()) %>%
    mutate(change = flight_count - lag(flight_count))

# rewrite previous with the `tally` function
tblflights %>%
    group_by(month) %>%
    tally() %>%
    mutate(change = n - lag(n))
```

---
## Other Useful Convenience Functions


```r
# randomly sample a fixed number of rows, without replacement
tblflights %>% sample_n(5)

# randomly sample a fraction of rows, with replacement
tblflights %>% sample_frac(0.25, replace=TRUE)

# base R approach to view the structure of an object
str(tblflights)

# dplyr approach: better formatting, and adapts to your screen width
glimpse(tblflights)
```

---
## do : for doing arbitrary operations


```r
model=tblflights %>% group_by(origin) %>% do(lm=lm(dep_delay~arr_delay+carrier,data=.)) 
model %>% summarise(rsq=summary(lm)$r.squared)
```

```
## Source: local data frame [3 x 1]
## 
##      rsq
## 1 0.8636
## 2 0.8182
## 3 0.8417
```

---
## Binary verbs

4 joins from SQL

* `inner_join(x, y)`: matching x + y
* `left_join(x, y)` : all x + matching y
* `semi_join(x, y)` : all x with match in y
* `anti_join(x, y)` : all x without match in y

---
## Quick look at 3 of 5 nycflights13 datasets

```r
tblflights %>% head(1) 
```

```
## Source: local data frame [1 x 17]
## 
##   year month day dep_time dep_delay arr_time arr_delay carrier tailnum
## 1 2013     1   1      517         2      830        11      UA  N14228
## Variables not shown: flight (int), origin (chr), dest (chr), air_time
##   (dbl), distance (dbl), hour (dbl), minute (dbl), delaysquare (dbl)
```

```r
airlines %>% head(1)
```

```
## Source: local data frame [1 x 2]
## 
##   carrier              name
## 1      9E Endeavor Air Inc.
```

```r
airports %>% head(1)
```

```
## Source: local data frame [1 x 7]
## 
##   faa              name   lat    lon  alt tz dst
## 1 04G Lansdowne Airport 41.13 -80.62 1044 -5   A
```

---
## inner_join(x,y): matching x + y

```r
tblflights %>% inner_join(airlines) %>% head(1)
```

```
## Joining by: "carrier"
```

```
## Source: local data frame [1 x 18]
## 
##   carrier year month day dep_time dep_delay arr_time arr_delay tailnum
## 1      UA 2013     1   1      517         2      830        11  N14228
## Variables not shown: flight (int), origin (chr), dest (chr), air_time
##   (dbl), distance (dbl), hour (dbl), minute (dbl), delaysquare (dbl), name
##   (fctr)
```

```r
tblflights %>% inner_join(airlines) %>% select(distance:name) %>% head(1)
```

```
## Joining by: "carrier"
```

```
## Source: local data frame [1 x 5]
## 
##   distance hour minute delaysquare                  name
## 1     1400    5     17         125 United Air Lines Inc.
```

---
## left_join(x,y): all x + matching y


```r
faatblflights=tblflights %>% select(origin) %>% mutate(faa=origin)
faatblflights %>% left_join(airports) %>% head(2) 
```

```
## Joining by: "faa"
```

```
## Source: local data frame [2 x 8]
## 
##   faa origin                name   lat    lon alt tz dst
## 1 EWR    EWR Newark Liberty Intl 40.69 -74.17  18 -5   A
## 2 LGA    LGA          La Guardia 40.78 -73.87  22 -5   A
```

```r
airports %>% left_join(faatblflights) %>% head(2)
```

```
## Joining by: "faa"
```

```
## Source: local data frame [2 x 8]
## 
##   faa                          name   lat    lon  alt tz dst origin
## 1 04G             Lansdowne Airport 41.13 -80.62 1044 -5   A     NA
## 2 06A Moton Field Municipal Airport 32.46 -85.68  264 -5   A     NA
```

---
## semi_join(x, y) : all x with match in y


```r
airports %>% left_join(faatblflights) %>% head(2)
```

```
## Joining by: "faa"
```

```
## Source: local data frame [2 x 8]
## 
##   faa                          name   lat    lon  alt tz dst origin
## 1 04G             Lansdowne Airport 41.13 -80.62 1044 -5   A     NA
## 2 06A Moton Field Municipal Airport 32.46 -85.68  264 -5   A     NA
```

```r
airports %>% semi_join(faatblflights) %>% head(2)
```

```
## Joining by: "faa"
```

```
## Source: local data frame [2 x 7]
## 
##   faa                name   lat    lon alt tz dst
## 1 EWR Newark Liberty Intl 40.69 -74.17  18 -5   A
## 2 LGA          La Guardia 40.78 -73.87  22 -5   A
```

---

## anti_join(x, y) : all x without match in y

```r
faatblflights %>% anti_join(airports)
```

```
## Joining by: "faa"
```

```
## Source: local data frame [0 x 2]
```

```r
airports %>% anti_join(faatblflights)
```

```
## Joining by: "faa"
```

```
## Source: local data frame [1,394 x 7]
## 
##    faa                       name   lat     lon  alt tz dst
## 1  RDD               Redding Muni 40.51 -122.29  504 -8   A
## 2  HXD        Hilton Head Airport 32.22  -80.70   19 -4   A
## 3  HDO    Hondo Municipal Airport 29.36  -99.18  930 -5   A
## 4  PIH Pocatello Regional Airport 42.91 -112.60 4452 -7   A
## 5  PNM             Princeton Muni 45.56  -93.61  979 -6   A
## 6  IAD     Washington Dulles Intl 38.94  -77.46  313 -5   A
## 7  SCK      Stockton Metropolitan 37.89 -121.24   33 -8   A
## 8  GEG               Spokane Intl 47.62 -117.53 2376 -8   A
## 9  ONH  Oneonta Municipal Airport 42.52  -75.06 1763 -5   A
## 10 YIP                 Willow Run 42.24  -83.53  716 -5   A
## .. ...                        ...   ...     ...  ... .. ...
```

---
## The fun has to stop here...

* [Source Code of this slidify presentation](https://github.com/patilv/dplyr-nycflights)
* [Kevin Markham's tutorial](http://www.dataschool.io/dplyr-tutorial-for-faster-data-manipulation-in-r/)
* [Official dplyr reference manual and vignettes on CRAN](http://cran.r-project.org/web/packages/dplyr/index.html): vignettes are well-written and cover many aspects of dplyr
* [July 2014 webinar about dplyr (and ggvis) by Hadley Wickham](http://pages.rstudio.net/Webinar-Series-Recording-Essential-Tools-for-R.html) and related [slides/code](https://github.com/rstudio/webinars/tree/master/2014-01): mostly conceptual, with a bit of code
* [dplyr tutorial by Hadley Wickham](https://www.dropbox.com/sh/i8qnluwmuieicxc/AAAgt9tIKoIm7WZKIyK25lh6a) at the [useR! 2014 conference](http://user2014.stat.ucla.edu/): excellent, in-depth tutorial with lots of example code (Dropbox link includes slides, code files, and data files)
* [dplyr GitHub repo](https://github.com/hadley/dplyr) and [list of releases](https://github.com/hadley/dplyr/releases)
