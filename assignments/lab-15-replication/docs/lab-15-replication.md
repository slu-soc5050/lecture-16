Lecture-16 Example Code
================
Christopher Prener, Ph.D.
(December 10, 2018)

## Introduction

This notebook illustrates the code for lecture 16.

## Dependencies

This notebook requires only three additional package beyond base `R`
packages:

``` r
# tidyverse packages
library(dplyr)        # for pipe operator
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
# other packages
library(janitor)      # tables
library(testDriveR)   # data
```

## Load Data

This notebook requires the `auto17` data from `testDriveR`:

``` r
autoData <- auto17
```

## Part 1

### Questions 1 and 2.

First, we need to create an indicator for each vehicle that is `TRUE` if
it is made by a German manufacturer. Then we want to subset our data so
that we have only `id`, `german`, and `driveStr`.

``` r
autoData %>% 
  mutate(german = ifelse(mfr == "BMW" | mfr == "Mercedes" | mfr == "Porsche" | 
                           mfr == "Volkswagen", TRUE, FALSE)) %>%
  select(id, german, driveStr) -> autoData
```

## Part 2

### Question 3

Next, we need to create a two-way contingency table of our data:

``` r
autoData %>%
  tabyl(german, driveStr) %>%
  adorn_totals(where = c("row", "col")) %>%
  adorn_percentages(denominator = "row") %>%
  adorn_pct_formatting(digits = 3) %>%
  adorn_ns(position = "front")
```

    ##  german             4             A             F           P
    ##   FALSE 116 (11.197%) 256 (24.710%) 356 (34.363%) 34 (3.282%)
    ##    TRUE  18 (10.000%)  72 (40.000%)  24 (13.333%)  0 (0.000%)
    ##   Total 134 (11.020%) 328 (26.974%) 380 (31.250%) 34 (2.796%)
    ##              R           Total
    ##  274 (26.448%) 1036 (100.000%)
    ##   66 (36.667%)  180 (100.000%)
    ##  340 (27.961%) 1216 (100.000%)

We can see that a higher proportion of German vehicles are all wheel
drive and rear wheel drive (owing to the Porsche sports cars).

## Part 3

### Question 4

We want to know if this variance that we observe visually is
substantive, so we’ll use a chi-square test:

``` r
(model <- chisq.test(autoData$german, autoData$driveStr))
```

    ## 
    ##  Pearson's Chi-squared test
    ## 
    ## data:  autoData$german and autoData$driveStr
    ## X-squared = 46.827, df = 4, p-value = 1.657e-09

The chi-square test (\(\chi^{2} = 46.827, df = 4, p < .001\)) indicates
that there is substantial variation in drive train type based on whether
or not the car is manufactured in Germany.

### Question 5

Even though we did not get a warning, we’ll check to see whether we’ve
violated the Cochran conditions:

``` r
model$expected
```

    ##                autoData$driveStr
    ## autoData$german         4         A      F         P         R
    ##           FALSE 114.16447 279.44737 323.75 28.967105 289.67105
    ##           TRUE   19.83553  48.55263  56.25  5.032895  50.32895

We can see no values less than 5 (though we come very close with one
expected count), meaning we have not violated this assumption.

### Question 6

Had we violated Cochran conditions, we would have used Fisher’s Exact
Test to similaute a p-value:

``` r
fisher.test(autoData$german, autoData$driveStr, simulate.p.value = TRUE)
```

    ## 
    ##  Fisher's Exact Test for Count Data with simulated p-value (based
    ##  on 2000 replicates)
    ## 
    ## data:  autoData$german and autoData$driveStr
    ## p-value = 0.0004998
    ## alternative hypothesis: two.sided

The Fisher’s Exact test (p = .0005) indicates that there is substantial
variation in drive train type based on whether or not the car is
manufactured in Germany.
