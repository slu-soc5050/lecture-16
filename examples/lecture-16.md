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
library(ggplot2)      # for mpg data

# other packages
library(janitor)      # tables
```

## Load Data

This notebook requires the `mpg` data from `ggplot2`:

``` r
autoData <- mpg
```

## Contingency Tables

### Basic Table

A basic contingency table is called using the `tabyl` function from
`janitor`:

``` r
autoData %>%
  tabyl(cyl, drv)
```

    ##  cyl  4  f  r
    ##    4 23 58  0
    ##    5  0  4  0
    ##    6 32 43  4
    ##    8 48  1 21

### Adding Totals

We can add row and column totals with an “adornment” - what `janitor`
calls add-ons called after the initial `tabyl` function is called:

``` r
autoData %>%
  tabyl(cyl, drv) %>%
  adorn_totals(where = c("row", "col"))
```

    ##    cyl   4   f  r Total
    ##      4  23  58  0    81
    ##      5   0   4  0     4
    ##      6  32  43  4    79
    ##      8  48   1 21    70
    ##  Total 103 106 25   234

### Adding Percentages

Percentages can be used in place of frequencies by adding a second
adornment:

``` r
autoData %>%
  tabyl(cyl, drv) %>%
  adorn_totals(where = c("row", "col")) %>%
  adorn_percentages(denominator = "row")
```

    ##    cyl         4          f          r Total
    ##      4 0.2839506 0.71604938 0.00000000     1
    ##      5 0.0000000 1.00000000 0.00000000     1
    ##      6 0.4050633 0.54430380 0.05063291     1
    ##      8 0.6857143 0.01428571 0.30000000     1
    ##  Total 0.4401709 0.45299145 0.10683761     1

### Formatting Percentages

Percentages can be formatting by adding another adornment:

``` r
autoData %>%
  tabyl(cyl, drv) %>%
  adorn_totals(where = c("row", "col")) %>%
  adorn_percentages(denominator = "row") %>%
  adorn_pct_formatting(digits = 3)
```

    ##    cyl       4        f       r    Total
    ##      4 28.395%  71.605%  0.000% 100.000%
    ##      5  0.000% 100.000%  0.000% 100.000%
    ##      6 40.506%  54.430%  5.063% 100.000%
    ##      8 68.571%   1.429% 30.000% 100.000%
    ##  Total 44.017%  45.299% 10.684% 100.000%

### Adding Frequencies Back In

What if we want both frequencies and percentages? We can add them back
in using a final adornment:

``` r
autoData %>%
  tabyl(cyl, drv) %>%
  adorn_totals(where = c("row", "col")) %>%
  adorn_percentages(denominator = "row") %>%
  adorn_pct_formatting(digits = 3) %>%
  adorn_ns(position = "front")
```

    ##    cyl             4              f            r          Total
    ##      4  23 (28.395%)  58  (71.605%)  0  (0.000%)  81 (100.000%)
    ##      5   0  (0.000%)   4 (100.000%)  0  (0.000%)   4 (100.000%)
    ##      6  32 (40.506%)  43  (54.430%)  4  (5.063%)  79 (100.000%)
    ##      8  48 (68.571%)   1   (1.429%) 21 (30.000%)  70 (100.000%)
    ##  Total 103 (44.017%) 106  (45.299%) 25 (10.684%) 234 (100.000%)

## Calculating Chi-Squared

If we want to know if the differences observed between `cyl` and `drv`
are substantive - would we see variation this extreme if there were no
patterning - we can use the chi-square test. It is helpful to fit the
model by wrapping the entire call in `()` so that we can get the model
output *and* store the model in an object
    simultaneously.

``` r
(model <- chisq.test(mpg$cyl, mpg$drv))
```

    ## Warning in chisq.test(mpg$cyl, mpg$drv): Chi-squared approximation may be
    ## incorrect

    ## 
    ##  Pearson's Chi-squared test
    ## 
    ## data:  mpg$cyl and mpg$drv
    ## X-squared = 98.136, df = 6, p-value < 2.2e-16

The chi-square test (\(\chi^{2} = 98.136, df = 6, p < .001\)) indicates
that there is substantial variation in cylinders by drive train type.

## Cochran conditions

Notice that a warning prints with our model output:

> Chi-squared approximation may be incorrect

This is an indication that we have some issues with the model. *We
should explore this assumption anyway even if there is no warning,
however.* Our model object contains a matrix with expected values. We
want to evaluate them:

``` r
model$expected
```

    ##        mpg$drv
    ## mpg$cyl         4         f         r
    ##       4 35.653846 36.692308 8.6538462
    ##       5  1.760684  1.811966 0.4273504
    ##       6 34.773504 35.786325 8.4401709
    ##       8 30.811966 31.709402 7.4786325

There are three values here that are less than 5 and 3/12 = .25 - this
is a sign we’ve ciolated the Cochran conditions. Furthermore, there is
one value below 1, which is a significant concern.

We can neatly summarize this by creating a logical test:

``` r
model$expected < 5
```

    ##        mpg$drv
    ## mpg$cyl     4     f     r
    ##       4 FALSE FALSE FALSE
    ##       5  TRUE  TRUE  TRUE
    ##       6 FALSE FALSE FALSE
    ##       8 FALSE FALSE FALSE

We can do the same for expected counts less than 1:

``` r
model$expected < 1
```

    ##        mpg$drv
    ## mpg$cyl     4     f     r
    ##       4 FALSE FALSE FALSE
    ##       5 FALSE FALSE  TRUE
    ##       6 FALSE FALSE FALSE
    ##       8 FALSE FALSE FALSE

## Fisher’s Exact Test

In the case where we violate Cochran conditions, we can use Fisher’s
Exact Test to similaute a p-value:

``` r
fisher.test(mpg$cyl, mpg$drv, simulate.p.value = TRUE)
```

    ## 
    ##  Fisher's Exact Test for Count Data with simulated p-value (based
    ##  on 2000 replicates)
    ## 
    ## data:  mpg$cyl and mpg$drv
    ## p-value = 0.0004998
    ## alternative hypothesis: two.sided

The Fisher’s Exact test (p = .0005) indicates that there is substantial
variation in cylinders by drive train type.
