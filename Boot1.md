Bootstrap 1
================
2022-02-16

<!-- notedown --nomagic Boot1.Rmd > Boot1.ipyn -->

# Bootstrap 1

## A few examples

This generates a sample, and print 1 over the mean

``` r
n <- 20
set.seed(123)
x <- rexp(n, rate=2)
1/mean(x)
```

    ## [1] 2.465566
