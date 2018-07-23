
--- 
title: "An Introduction to Statistical Learning"
subtitle: "with (Tidy) Applications in R"
author: "Benjamin Soltoff"
date: "2018-07-23"
site: bookdown::bookdown_site
documentclass: book
description: "This is a minimally documented implementation of *An Introduction to Statistical Learning* using a `tidyverse` philosophy for all the applications in R."
---

# What this is about {-}

This book is intended to serve as a companion to [*An Introduction to Statistical Learning with Applications in R*](http://www-bcf.usc.edu/~gareth/ISL/) (ISLR) and reimplement the R programs using primarily [`tidyverse`](https://www.tidyverse.org/) packages and a tidy design philosophy. It is not intended to stand on its own or review the statistical learning concepts from ISLR. It is not intended to teach R programming or computational problem solving in general. For that, I recommend any number of online resources including the materials from my graduate course on [Computing for the Social Sciences](https://cfss.uchicago.edu/).

The original *ISLR* book is copyrighted by Springer. The original contribution in work is licensed under the [CC BY-NC 4.0 Creative Commons License](http://creativecommons.org/licenses/by-nc/4.0/).

## Session Info


```r
devtools::session_info()
```

```
## Session info -------------------------------------------------------------
```

```
##  setting  value                       
##  version  R version 3.5.0 (2018-04-23)
##  system   x86_64, darwin15.6.0        
##  ui       X11                         
##  language (EN)                        
##  collate  en_US.UTF-8                 
##  tz       America/Chicago             
##  date     2018-07-23
```

```
## Packages -----------------------------------------------------------------
```

```
##  package   * version date       source        
##  backports   1.1.2   2017-12-13 CRAN (R 3.5.0)
##  base      * 3.5.0   2018-04-24 local         
##  bookdown    0.7     2018-02-18 CRAN (R 3.5.0)
##  compiler    3.5.0   2018-04-24 local         
##  datasets  * 3.5.0   2018-04-24 local         
##  devtools    1.13.5  2018-02-18 CRAN (R 3.5.0)
##  digest      0.6.15  2018-01-28 CRAN (R 3.5.0)
##  evaluate    0.10.1  2017-06-24 CRAN (R 3.5.0)
##  graphics  * 3.5.0   2018-04-24 local         
##  grDevices * 3.5.0   2018-04-24 local         
##  htmltools   0.3.6   2017-04-28 CRAN (R 3.5.0)
##  knitr       1.20    2018-02-20 CRAN (R 3.5.0)
##  magrittr    1.5     2014-11-22 CRAN (R 3.5.0)
##  memoise     1.1.0   2017-04-21 CRAN (R 3.5.0)
##  methods   * 3.5.0   2018-04-24 local         
##  Rcpp        0.12.17 2018-05-18 CRAN (R 3.5.0)
##  rmarkdown   1.9     2018-03-01 CRAN (R 3.5.0)
##  rprojroot   1.3-2   2018-01-03 CRAN (R 3.5.0)
##  stats     * 3.5.0   2018-04-24 local         
##  stringi     1.2.2   2018-05-02 CRAN (R 3.5.0)
##  stringr     1.3.1   2018-05-10 CRAN (R 3.5.0)
##  tools       3.5.0   2018-04-24 local         
##  utils     * 3.5.0   2018-04-24 local         
##  withr       2.1.2   2018-03-15 CRAN (R 3.5.0)
##  xfun        0.1     2018-01-22 CRAN (R 3.5.0)
##  yaml        2.1.19  2018-05-01 CRAN (R 3.5.0)
```
