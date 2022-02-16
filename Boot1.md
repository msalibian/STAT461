Bootstrap 1
================
2022-02-16

<!-- notedown --nomagic Boot1.Rmd > Boot1.ipynb -->

# Bootstrap 1

### Setup

1.  Follow the instructions [here](https://jupyter.org/install.html) to
    install Jupyter on your laptop. You will also need to follow [these
    instructions](https://www.datacamp.com/community/blog/jupyter-notebook-r)
    to install the `R kernel` for Jupyter.

2.  To run the notebooks on the [syzygy](https://ubc.syzygy.ca/) server.
    There are Julia, Python 2, Python 3, and R kernels available
    (although we will only use the R one). Sign in with your UBC CWL.
    Once you are logged in, use [this
    link](https://ubc.syzygy.ca/jupyter/user-redirect/git-pull?repo=https://github.com/msalibian/STAT461)
    to clone this repository (STAT461) (including all notebooks)
    directly onto your [syzygy](https://ubc.syzygy.ca/) home directory.
    You **will** need to do this regularly as the notebooks may (will?)
    change during the Term.

## A few examples

This generates a sample, and print 1 over the mean

``` r
n <- 20
set.seed(123)
x <- rexp(n, rate=2)
1/mean(x)
```

    ## [1] 2.465566
