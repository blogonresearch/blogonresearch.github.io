---
title: Show Options Set by lavaan
author: Shu Fai Cheung
date: '2022-09-26'
slug: []
categories: ["R"]
tags: ["R", "lavaan", "semhelpinghands"]
bibliography: references.bib
csl: apa.csl
---

`lavaan` is a convenient tool for doing
structural equation modelling in R
(Rosseel, 2012). One
of its strength is having “prepackaged”
estimators, which are shortcuts to a set
of options, such as “ML”, “MLR”, “MLMVS”,
and others (Savalei & Rosseel, 2022).
It also tries to set
default values for options based on the
model and data.

However, probably due to my not-so-good
memory, I sometimes forgot what the settings
are for a model. Therefore, in the
package [`semhelpinghands`](https://sfcheung.github.io/semhelpinghands/),
I wrote the function [`show_more_options()`](https://sfcheung.github.io/semhelpinghands/reference/show_more_options.html)
to show some of the settings of the output of
`lavaan()` and its wrappers, such as
`sem()` and `cfa()`.\[1\]

The function `show_more_options()` is
very easy to use … because it accepts
only one argument, the output of `lavaan()`.

This is an example based on the example of
`lavaan::cfa()`:

``` r
library(lavaan)
```

    ## This is lavaan 0.6-12
    ## lavaan is FREE software! Please report any bugs.

``` r
HSmodel <-
"
visual  =~ x1 + x2 + x3
textual =~ x4 + x5 + x6
speed   =~ x7 + x8 + x9
"
fit <- cfa(HSmodel,
           data = HolzingerSwineford1939)
```

To show the major options, just pass the output
to `show_more_options()`:

``` r
library(semhelpinghands)
show_more_options(fit)
```

    ##  Options                             Call    Actual  
    ##  Estimator(s)                        default ML      
    ##  Standard Error (SE)                 default standard
    ##  Model Test Statistic(s)             default standard
    ##  How Missing Data is Handled         default listwise
    ##  Information Matrix (for SE)         default expected
    ##  Information Matrix (for Model Test) default expected
    ##  Mean Structure                      default No      
    ##  'x' Fixed                           default FALSE

The column `Call` shows whether the
default setting is used for each row,
based of the call used when fitting the
model. However, it is not always clear
to me what the default values are.

The column `Actual` shows the values
extracted by `lavaan::lavInspect()` or
from the `Options` slot. These are what
the default values stand for in the
fitted model.

Many of the entries are either
(a) already available in the output of
`summary()`, or (b) can be deduced from
the output. However, I would like to have
a table for quick reference, hence I
wrote this function.

Suppose `"MLR"` is used as the estimator:

``` r
fit_MLR <- cfa(HSmodel,
               data = HolzingerSwineford1939,
               estimator = "MLR")
show_more_options(fit_MLR)
```

    ##  Options                             Call    Actual            
    ##  Estimator(s)                        MLR     ML                
    ##  Standard Error (SE)                 default robust.huber.white
    ##  Model Test Statistic(s)             default yuan.bentler.mplus
    ##  How Missing Data is Handled         default listwise          
    ##  Information Matrix (for SE)         default observed          
    ##  Information Matrix (for Model Test) default observed          
    ##  Mean Structure                      default No                
    ##  'x' Fixed                           default FALSE

The output shows the exact names of the
options (e.g., `"robust.huber.white"`
and `"yuan.bentler.mplus"`). They can
complement the more readable output
of `summary()` if we need to manually
set these options, or want to know
which values these options refer to when
consulting the help page.

For example, `summary()` reports that
`"Sandwich"` is the method used for
standard errors, and `show_more_options()`
shows that the exact name in the option
is `"robust.huber.white"`. This is useful
because the word `"sandwich"` does not
appear in the help page of `lavOptions()`,
while the word `"robust.huber.white"` does.
Some users may not know what `"Sandwich"`
stands for.

This is a dataset for a path model,
with missing data:

``` r
set.seed(8745315)
n <- 100
x <- rnorm(n)
m <- 5 + .2 * x + rnorm(n, 0, .8)
y <- 10 + .3 * m + rnorm(n, 0, .8)
path_data <- data.frame(x, m, y)
path_data[c(1, 3, 5, 7), "m"] <- NA
path_data[c(1, 3, 6, 8), "y"] <- NA
```

Suppose we use only the default options
to fit a path model:

``` r
path_model <-
"
m ~ a * x
y ~ b * m
ab := a * b
"
fit_path <- sem(path_model,
                data = path_data)
show_more_options(fit_path)
```

    ##  Options                             Call    Actual  
    ##  Estimator(s)                        default ML      
    ##  Standard Error (SE)                 default standard
    ##  Model Test Statistic(s)             default standard
    ##  How Missing Data is Handled         default listwise
    ##  Information Matrix (for SE)         default expected
    ##  Information Matrix (for Model Test) default expected
    ##  Mean Structure                      default No      
    ##  'x' Fixed                           default TRUE

The output shows that, by default,
the mean structure is not modelled,
listwise selection is used to handle
missing data, and
`x` variables (exogenous covariates,
`x` in this example)
are treated as fixed. This can be verified
by the parameter estimates, in which
the variance of `x` is a fixed
parameter and hence has no standard
error and no *p*-value:

``` r
parameterEstimates(fit_path)
```

    ##   lhs op rhs label   est    se     z pvalue ci.lower ci.upper
    ## 1   m  ~   x     a 0.075 0.078 0.966  0.334   -0.077    0.228
    ## 2   y  ~   m     b 0.248 0.119 2.081  0.037    0.014    0.482
    ## 3   m ~~   m       0.550 0.080 6.856  0.000    0.393    0.708
    ## 4   y ~~   y       0.745 0.109 6.856  0.000    0.532    0.958
    ## 5   x ~~   x       0.967 0.000    NA     NA    0.967    0.967
    ## 6  ab := a*b    ab 0.019 0.021 0.876  0.381   -0.023    0.060

Suppose we set missing to `"FIML"`:

``` r
fit_path_fiml <- sem(path_model,
                     data = path_data,
                     missing = "FIML")
show_more_options(fit_path_fiml)
```

    ##  Options                             Call    Actual  
    ##  Estimator(s)                        default ML      
    ##  Standard Error (SE)                 default standard
    ##  Model Test Statistic(s)             default standard
    ##  How Missing Data is Handled         FIML    ml      
    ##  Information Matrix (for SE)         default observed
    ##  Information Matrix (for Model Test) default observed
    ##  Mean Structure                      default Yes     
    ##  'x' Fixed                           default TRUE

`x` variables are still treated as fixed,
but now mean structure is modelled
(required for FIML, full information
maximum likelihood), even though I did
not explicitly ask for it.

Only options I think are likely needed
(by me) are included
in the output.\[2\]
More may be added in
the future. In any case, if other options
are needed, they can be
retrieved by `lavaan::lavInspect()` or
from the `Options` slot of the output.
In most cases I
myself encountered, all I want is a simple
function that is easy to remember and no
need to set any arguments other than
the `lavaan` output. If I need something
else, I will just extract the information
myself.

This function was inspired by a script
I wrote to enumerate the options set by
the prepackaged shortcuts. Interested
readers can read [this thread](https://groups.google.com/g/lavaan/c/6oLwoboi-vQ/m/IQLAXChPAwAJ)
at the [Google Group for `lavaan`](https://groups.google.com/g/lavaan)
and [this gist](https://gist.github.com/sfcheung/baa5e43c32d4763b859f5338a1738d79),
to check how options will be set for
different
combinations of estimator, data, and
some other options.

## References

<div id="refs" class="references hanging-indent">

<div id="ref-lavaan_2012">

Rosseel, Y. (2012). lavaan: An R package for structural equation modeling. *Journal of Statistical Software*, *48*(2), 1–36. <https://doi.org/10.18637/jss.v048.i02>

</div>

<div id="ref-savalei_computational_2022">

Savalei, V., & Rosseel, Y. (2022). Computational options for standard errors and test statistics with incomplete normal and nonnormal data in SEM. *Structural Equation Modeling: A Multidisciplinary Journal*, *29*(2), 163–181. <https://doi.org/10.1080/10705511.2021.1877548>

</div>

</div>

1.  To be precise, any object of the class `lavaan`.

2.  The information matrices are technical but I occasionally need them.
