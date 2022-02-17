Bootstrap 1
================
2022-02-16

<!-- notedown --nomagic Boot1.Rmd > Boot1.ipynb -->

### Setting up / running Jupyter notebooks

You have essentially two choices to use this notebook “live”:

1.  Follow the instructions [here](https://jupyter.org/install.html) to
    install Jupyter on your laptop. You will also need to follow [these
    instructions](https://www.datacamp.com/community/blog/jupyter-notebook-r)
    to install the `R kernel` for Jupyter.

2.  Or, run the notebooks on the [syzygy](https://ubc.syzygy.ca/)
    server. There are Julia, Python 2, Python 3, and R kernels available
    (although we will only use the R one). Sign in with your UBC CWL.
    Once you are logged in, use [this
    link](https://ubc.syzygy.ca/jupyter/user-redirect/git-pull?repo=https://github.com/msalibian/STAT461)
    to clone the GitHub repository (STAT461) where these notebooks are
    stored directly onto your [syzygy](https://ubc.syzygy.ca/) home
    directory.

## A few examples

Here we will use a few simple examples to illustrate the use of the
bootstrap in practice to construct confidence intervals for a
“parameter” of an underlying and uknown distribution. In these synthetic
examples, we actually do know the true distribution (and thus, the value
of the parameter of interest), so we will be able to assess the level of
success of this inferential approach.

These examples use the simple bootstrap percentile method to construct
confidence intervals.

### Example 1 - Estimate the square root of the expected value

Consider a sample *X*<sub>1</sub>, , *X*<sub>*n*</sub> from an unknown
distribution *F* with mean
*μ* = *E*<sub>*F*</sub>(*X*) = ∫*t**d**F*(*t*). We are interested in
building an approximate 95% confidence interval for
$$
\\theta = \\sqrt{\\mu}
$$
First, lets generate a sample

``` r
n <- 30
x <- rexp(n, rate=4)
```

A natural estimator is

``` r
( t0 <- sqrt(mean(x)) )
```

    ## [1] 0.507411

We now use simulation to approximate the bootstrap estimator of the
distribution of $\\sqrt{ \\bar{X}\_n}$.

``` r
M <- 5000
rho <- vector('numeric', M)
for(j in 1:M) {
  bb <- sample.int(n, repl=TRUE)
  # bb <- sample(1:n, repl=TRUE)
  rho[j] <- sqrt(mean(x[bb]))
}
lo0 <- t0 - quantile(rho - t0, .975)
up0 <- t0 - quantile(rho - t0, .025)
as.numeric(c(lo=lo0, up=up0))
```

    ## [1] 0.4273840 0.5927378

``` r
# 
# lo0 <- t0 - (quantile(rho - t0, .975) - t0)
# up0 <- t0 - (quantile(rho - t0, .025) - t0)
```

A CLT-type approximation yields the CI

``` r
lo1 <- t0 - qnorm(.975)*sd(x)/2/mean(x)/sqrt(n)
up1 <- t0 + qnorm(.975)*sd(x)/2/mean(x)/sqrt(n)
```

We can compare it with the previous one

``` r
( rbind(c(lo=lo0, up=up0),
      c(lo=lo1, up=up1)) )
```

    ##       lo.97.5%   up.2.5%
    ## [1,] 0.4273840 0.5927378
    ## [2,] 0.3417635 0.6730584

### Example 2 - Estimate the expected value of the square root

In this example, we have a sample *X*<sub>1</sub>, …, *X*<sub>*n*</sub>
and are interested in $\\theta = E\_F\[ \\sqrt{X} \]$.

Because we simulate these observations, we will have
*X*<sub>*i*</sub> ∼ *χ*<sub>1</sub><sup>2</sup>, and thus
$\\sqrt{X} \\sim \|Z\|$ where $Z \\sim {\\cal N}(0,1)$. We have
$\\theta = E\[ \|Z\| \] = \\sqrt{2/pi} = 0.7978846$.

First generate a sample

``` r
set.seed(123)
n <- 30
x <- rchisq(n, df=1)
```

A natural estimator is $(1/n) \\sum\_{i=1}^n \\sqrt{X}\_i$:

``` r
(t0 <- mean(sqrt(x)))
```

    ## [1] 0.6507962

We now use 5000 bootstrap samples to estimate the distribution of
$(1/n) \\sum\_{i=1}^n \\sqrt{X}\_i$:

``` r
M <- 5000
rho <- vector('numeric', M)
for(j in 1:M) {
  bb <- sample.int(n, repl=TRUE)
  rho[j] <- mean(sqrt(x[bb]))
}
```

We can now use the estimated quantiles to build an approximate 95%
confidence interval:

``` r
lo0 <- t0 - quantile(rho - t0, .975)
up0 <- t0 - quantile(rho - t0, .025)
as.numeric(c(lo=lo0, up=up0))
```

    ## [1] 0.4653701 0.8193270

Can you think of another method to build an approximate CI for
$E\[ \\sqrt{X} \]$?

### Example 3 - Mean of a ratio of dependent variables

We generate a small sample of correlated variables *X* and *Y*:

``` r
set.seed(123)
n <- 30
la <- rchisq(n, df=1)
x <- rexp(n, rate=1) + la
y <- runif(n, min=2, max=3) + la
```

We are interested in *E*\[*X*/*Y*\]. Note that *P*(*Y* &gt; 0) = 1. A
natural point estimator is

``` r
(t0 <- mean(x/y))
```

    ## [1] 0.4903691

Use bootstrap to estimate quantiles of the statistic of interest

``` r
M <- 5000
rho <- vector('numeric', M)
for(j in 1:M) {
  bb <- sample.int(n, repl=TRUE)
  rho[j] <- mean(x[bb]/y[bb])
}
```

We can use *E*<sup>\*</sup>\[*X*/*Y*\] as an estimator for
*E*\[*X*/*Y*\] and thus an estimator of the bias is

``` r
mean(rho) - t0 
```

    ## [1] -0.0002578016

and an estimator of the variance of $(1/n) \\sum\_{i=1}^n X\_i/Y\_i$

``` r
var(rho)
```

    ## [1] 0.004222528

We can now use the estimated quantiles to build an approximate 95%
confidence interval:

``` r
lo0 <- t0 - quantile(rho - t0, .975)
up0 <- t0 - quantile(rho - t0, .025)
as.numeric(c(lo=lo0, up=up0))
```

    ## [1] 0.3548364 0.6090059

<!-- These are simple quantile CI's.  -->
<!-- Better (bias corrected) quantile estimators: -->
<!-- ### Example 4 - CI for a correlation -->
<!-- Generate correlated $X$ and $Y$: -->
<!-- ```{r ex4.0} -->
<!-- set.seed(123) -->
<!-- n <- 100 -->
<!-- la <- rchisq(n, df=1) - 1 -->
<!-- x <- rexp(n, rate=1) + la -->
<!-- y <- runif(n, min=-1, max=1) + la -->
<!-- ``` -->
<!-- Now compute the bootstrapped estimators -->
<!-- ```{r ex4.1} -->
<!-- M <- 5000 -->
<!-- rho <- vector('numeric', M) -->
<!-- for(j in 1:M) { -->
<!--   bb <- sample.int(n, repl=TRUE) -->
<!--   rho[j] <- cor(x[bb], y[bb]) -->
<!-- } -->
<!-- ``` -->
<!-- Compute the quantiles -->
<!-- ```{r ex4.2} -->
<!-- t0 <- cor(x, y) -->
<!-- lo0 <- t0 - (quantile(rho, .975) - t0) -->
<!-- up0 <- t0 - (quantile(rho, .025) - t0) -->
<!-- ``` -->
<!-- The normal (transformed) approach: -->
<!-- ```{r ex4.3} -->
<!-- zz <- log((1+t0)/(1-t0))/2 -->
<!-- lo <- zz - qnorm(.975)/sqrt(n-3) -->
<!-- up <- zz + qnorm(.975)/sqrt(n-3) -->
<!-- lo2 <- (exp(2*lo)-1)/(exp(2*lo)+1) -->
<!-- up2 <- (exp(2*up)-1)/(exp(2*up)+1) -->
<!-- ``` -->
<!-- The bootstrap one isn't really better -->
<!-- ```{r ex4.4} -->
<!-- rbind(c(lo0, up0),  -->
<!--       c(lo2, up2)) -->
<!-- ``` -->
<!-- We can do better: -->
<!-- ```{r ex4.5} -->
<!-- (tmp <- mean(rho <= t0)) -->
<!-- (alph <- pnorm(qnorm(.025) + 2*qnorm(tmp))) -->
<!-- lo3 <- t0 - (quantile(rho, 1-alph) - t0) -->
<!-- up3 <- t0 - (quantile(rho, alph) - t0) -->
<!-- rbind(c(lo0, up0),  -->
<!--       c(lo2, up2), -->
<!--       c(lo3, up3)) -->
<!-- ``` -->
<!-- M <- 1000 -->
<!-- lev5 <- lev4 <- lev3 <- lev2 <- lev <- 0 -->
<!-- len5 <- len4 <- len3 <- len2 <- len <- 0 -->
<!-- ff <- function(a, x) 1/mean(x[a]) -->
<!-- n <- 10 -->
<!-- lambda0 <- 5 -->
<!-- a <- qnorm(.025) -->
<!-- b <- qnorm(.975) -->
<!-- for(j in 1:M) { -->
<!--   set.seed(123 + 17*j) -->
<!--   x <- rexp(n, rate=lambda0) -->
<!--   # Estimate quantiles -->
<!--   B <- 5000 -->
<!--   bootsam <- matrix(sample.int(n, size=B*n, replace=TRUE), B, n) -->
<!--   tstar <- apply(bootsam, 1, ff, x=x) -->
<!--   t0 <- 1/mean(x) -->
<!--   lo <- t0 - quantile(tstar, .975) + t0 -->
<!--   up <- t0 - quantile(tstar, .025) + t0 -->
<!--   if((lambda0 > lo ) & (lambda0 < up) ) lev <- lev + 1 -->
<!--   len <- len + (up - lo) -->
<!--   bootsam <- matrix(rexp(n*B, rate=1/mean(x)), B, n) -->
<!--   tstar <- apply(bootsam, 1, function(a) 1/mean(a)) -->
<!--   lo <- t0 - quantile(tstar, .975) + t0 -->
<!--   up <- t0 - quantile(tstar, .025) + t0 -->
<!--   if((lambda0 > lo ) & (lambda0 < up) ) lev2 <- lev2 + 1 -->
<!--   len2 <- len2 + (up - lo) -->
<!--   up <- t0*(1-a/sqrt(n)) -->
<!--   lo <- t0*(1-b/sqrt(n)) -->
<!--   if( (lambda0 < up ) & (lambda0 > lo ) ) lev3 <- lev3 + 1 -->
<!--   len3 <- len3 + (up - lo) -->
<!--   up <-t0*(1+b/sqrt(n)) -->
<!--   lo <- t0*(1+a/sqrt(n)) -->
<!--   if( (lambda0 < up ) & (lambda0 > lo ) ) lev4 <- lev4 + 1 -->
<!--   len4 <- len4 + (up - lo) -->
<!--   up <- (1/t0) - a*sd(x)/sqrt(n) -->
<!--   lo <- (1/t0) - b*sd(x)/sqrt(n) -->
<!--   if( (lambda0 < (1/lo) ) & (lambda0 > (1/up) ) ) lev5 <- lev5 + 1 -->
<!--   len5 <- len5 + (1/lo - 1/up) -->
<!--   # print(c(it=j, bnp=lev/j, bp=lev2/j, clt=lev3/j, clt2=lev4/j))   -->
<!--   print(c(it=j, bnp=len/j, bp=len2/j, clt=len3/j, clt2=len4/j, clt3=len5/j))   -->
<!-- } -->
<!-- print(c(bnp=lev/M, bp=lev2/M, clt=lev3/M, clt2=lev4/M, clt3=lev5/M))   -->
<!-- print(c(bnp=len/M, bp=len2/M, clt=len3/M, clt2=len4/M, clt3=len5/M))   -->
<!-- ### CI for a correlation -->
<!-- set.seed(123) -->
<!-- n <- 30 -->
<!-- la <- rchisq(n, df=1) - 1 -->
<!-- x <- rexp(n, rate=1) + la -->
<!-- y <- runif(n, min=-1, max=1) + la -->
<!-- M <- 5000 -->
<!-- rho <- vector('numeric', M) -->
<!-- for(j in 1:M) { -->
<!--   bb <- sample.int(n, repl=TRUE) -->
<!--   rho[j] <- cor(x[bb], y[bb]) -->
<!-- } -->
<!-- t0 <- cor(x, y) -->
<!-- lo0 <- t0 - (quantile(rho, .975) - t0) -->
<!-- up0 <- t0 - (quantile(rho, .025) - t0) -->
<!-- zz <- log((1+t0)/(1-t0))/2 -->
<!-- lo <- zz - qnorm(.975)/sqrt(n-3) -->
<!-- up <- zz + qnorm(.975)/sqrt(n-3) -->
<!-- lo2 <- (exp(2*lo)-1)/(exp(2*lo)+1) -->
<!-- up2 <- (exp(2*up)-1)/(exp(2*up)+1) -->
<!-- print(c(lo0, up0)) -->
<!-- print(c(lo2, up2)) -->
<!-- ## Mean of a ratio of dependent variables -->
<!-- set.seed(123) -->
<!-- n <- 30 -->
<!-- la <- rchisq(n, df=1) - 1 -->
<!-- x <- rexp(n, rate=1) + la -->
<!-- y <- runif(n, min=2, max=3) + la -->
<!-- t0 <- mean(x/y) -->
<!-- M <- 5000 -->
<!-- rho <- vector('numeric', M) -->
<!-- for(j in 1:M) { -->
<!--   bb <- sample.int(n, repl=TRUE) -->
<!--   rho[j] <- mean(x[bb]/y[bb]) -->
<!-- } -->
<!-- # bias? -->
<!-- mean(rho) - t0 -->
<!-- # variance -->
<!-- var(rho) -->
<!-- ## mean of sqrt(X^2_1) = |N(0,1)| -->
<!-- # sigma*sqrt(2/pi)*exp(-mu^2/(2 sigma^2)) + mu(1-2*pnorm(-mu/sigma)) -->
<!-- # mu=0, sigma=1 -->
<!-- # sqrt(2/pi) = 0.7978846 -->
<!-- set.seed(123) -->
<!-- n <- 30 -->
<!-- x <- rchisq(n, df=1) -->
<!-- t0 <- mean(sqrt(x)) -->
<!-- M <- 5000 -->
<!-- rho <- vector('numeric', M) -->
<!-- for(j in 1:M) { -->
<!--   bb <- sample.int(n, repl=TRUE) -->
<!--   rho[j] <- mean(sqrt(x[bb])) -->
<!-- } -->
<!-- lo0 <- t0 - (quantile(rho, .975) - t0) -->
<!-- up0 <- t0 - (quantile(rho, .025) - t0) -->
<!-- c(lo=lo0, up=up0) -->
<!-- # an alternative method? -->
<!-- ## sqrt(E(X)) -->
<!-- set.seed(123) -->
<!-- n <- 30 -->
<!-- x <- rexp(n, rate=4) -->
<!-- t0 <- sqrt(mean(x)) -->
<!-- M <- 5000 -->
<!-- rho <- vector('numeric', M) -->
<!-- for(j in 1:M) { -->
<!--   bb <- sample.int(n, repl=TRUE) -->
<!--   rho[j] <- sqrt(mean(x[bb])) -->
<!-- } -->
<!-- lo0 <- t0 - (quantile(rho, .975) - t0) -->
<!-- up0 <- t0 - (quantile(rho, .025) - t0) -->
<!-- lo1 <- t0 - qnorm(.975)*sd(x)/2/mean(x)/sqrt(n) -->
<!-- up1 <- t0 + qnorm(.975)*sd(x)/2/mean(x)/sqrt(n) -->
<!-- rbind(c(lo=lo0, up=up0), -->
<!--       c(lo=lo1, up=up1)) -->
