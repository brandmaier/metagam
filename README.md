
<!-- README.md is generated from README.Rmd. Please edit that file -->

# metagam

<!-- badges: start -->

[![Travis build
status](https://travis-ci.org/lifebrain/metagam.svg?branch=master)](https://travis-ci.org/lifebrain/metagam)
<!-- badges: end -->

`metagam` is an R-package for flexible meta-analysis of generalized
additive (mixed) models (GAMs/GAMMs).

The package is under development, so changes to the interface can be
expected. Suggestions for improvements and bug reports are warmly
welcome, either by filing an
[Issue](https://github.com/lifebrain/metagam/issues) or opening a [Pull
Request](https://github.com/lifebrain/metagam/pulls).

## Installation

Install the current development version of `metagam` from
[GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("lifebrain/metagam")
```

## Example

`metagam` has two main functions: `prepare_meta` and `metagam`. To
illustrate them, we start by simulating three datasets. We need the to
load the `mgcv` package in order to fit GAMs.

``` r
library(metagam)
library(mgcv)
#> Loading required package: nlme
#> This is mgcv 1.8-31. For overview type 'help("mgcv-package")'.
## simulate three datasets
set.seed(123)
datasets <- lapply(1:3, function(x) gamSim(scale = 3, verbose = FALSE))
```

First we fit a separate GAM to each dataset. In the typical application
of `metagam`, this will be done separately in the location of each
dataset. The function `prepare_meta` removes all subject-specific data
from the model object, so that it can be shared to a common location
while protecting privacy.

``` r
## fit a generalized additive model to each dataset separately
fits <- lapply(datasets, function(dat){
  ## Full fit using mgcv
  gamfit <- gam(y ~ s(x0) + s(x1) + s(x2), data = dat)
  ## Extract the necessary components for performing a meta-analysis
  ## This removes all subject-specific data
  prepare_meta(gamfit)
})
```

Finally we meta-analyze the fits. To this end, we need to define a grid
over which to make predictions. In this case, we are interested in
inference for the term `s(x2)`.

``` r
grid <- data.frame(x0 = 0, x1 = 0, x2 = seq(from = 0, to = 1, by = .01))
meta_analysis <- metagam(fits, grid = grid, terms = "s(x2)", intercept = FALSE)
```

Methods for visualizing and summarizing `metagam` objects will be added
during further package development. At the moment, scripts for plotting
have to be supplied by the user. The code chunk below plots the
meta-analytic fit as a solid line and the separate fits as dashed lines.

``` r
plot_data <- meta_analysis$prediction

plot(plot_data$x2, plot_data$fit, type = "l", xlab = "x2", ylab = "fit", ylim = c(-6, 6))
for(fit in fits){
  lines(plot_data$x2, predict(fit, newdata = grid, type = "iterms", terms = "s(x2)"), lty = 2)  
}
```

<img src="man/figures/README-unnamed-chunk-5-1.png" width="100%" />
