
# Classification {#classification}


```r
library(tidyverse)
```

```
## ── Attaching packages ───────────────────────────────────────── tidyverse 1.2.1 ──
```

```
## ✔ ggplot2 3.0.0     ✔ purrr   0.2.5
## ✔ tibble  1.4.2     ✔ dplyr   0.7.6
## ✔ tidyr   0.8.1     ✔ stringr 1.3.1
## ✔ readr   1.1.1     ✔ forcats 0.3.0
```

```
## Warning: package 'dplyr' was built under R version 3.5.1
```

```
## ── Conflicts ──────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
```

## An overview of classification


## Why not linear regression?


## Logistic regression


## Linear discriminant analysis


## A comparison of classification methods


## Lab: Logistic regression, LDA, QDA, and KNN


### The stock market data

Load the `SMarket` data from `ISLR`.


```r
library(ISLR)

Smarket <- as_tibble(Smarket)
Smarket
```

```
## # A tibble: 1,250 x 9
##    Year   Lag1   Lag2   Lag3   Lag4   Lag5 Volume  Today Direction
## * <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl> <fct>    
## 1  2001  0.381 -0.192 -2.62  -1.06   5.01    1.19  0.959 Up       
## 2  2001  0.959  0.381 -0.192 -2.62  -1.06    1.30  1.03  Up       
## 3  2001  1.03   0.959  0.381 -0.192 -2.62    1.41 -0.623 Down     
## 4  2001 -0.623  1.03   0.959  0.381 -0.192   1.28  0.614 Up       
## 5  2001  0.614 -0.623  1.03   0.959  0.381   1.21  0.213 Up       
## 6  2001  0.213  0.614 -0.623  1.03   0.959   1.35  1.39  Up       
## # ... with 1,244 more rows
```

Brief overview of the data using a scatterplot matrix:


```r
library(GGally)
```

```
## 
## Attaching package: 'GGally'
```

```
## The following object is masked from 'package:dplyr':
## 
##     nasa
```

```r
ggpairs(Smarket)
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

<img src="04-classification_files/figure-html/smarket-ggpairs-1.png" width="70%" style="display: block; margin: auto;" />

Volume over time:


```r
Smarket %>%
  mutate(id = row_number()) %>%
  ggplot(mapping = aes(x = id, y = Volume)) +
  geom_line() +
  geom_smooth()
```

```
## `geom_smooth()` using method = 'gam' and formula 'y ~ s(x, bs = "cs")'
```

<img src="04-classification_files/figure-html/volume-1.png" width="70%" style="display: block; margin: auto;" />

### Logistic regression

Logistic regression is a type of **generalized linearmo del** (GLM), a class of models for fitting regression lines to many types of response variables. `glm()` is the base function in R for estimating these models. The syntax is the same as `lm()` except we also pass the argument `family = binomial` to run the logistic regression form of GLM:


```r
glm_fit <- glm(Direction ~ Lag1 + Lag2 + Lag3 + Lag4 + Lag5 + Volume,
               data = Smarket,
               family = binomial)
```

We can again use `broom` to summarize the output of `glm()`:


```r
library(broom)

tidy(glm_fit)
```

```
##          term estimate std.error statistic p.value
## 1 (Intercept) -0.12600    0.2407    -0.523   0.601
## 2        Lag1 -0.07307    0.0502    -1.457   0.145
## 3        Lag2 -0.04230    0.0501    -0.845   0.398
## 4        Lag3  0.01109    0.0499     0.222   0.824
## 5        Lag4  0.00936    0.0500     0.187   0.851
## 6        Lag5  0.01031    0.0495     0.208   0.835
## 7      Volume  0.13544    0.1584     0.855   0.392
```

```r
glance(glm_fit)
```

```
##   null.deviance df.null logLik  AIC  BIC deviance df.residual
## 1          1731    1249   -864 1742 1778     1728        1243
```

To extract predicted probabilities for each observation (that is, in the form $P(Y = 1|X)$), we use `augment()` with the argument `type.predict = "response"`; if we omit that argument, the predicted values are generated in the log-odds form.


```r
augment(glm_fit, type.predict = "response") %>%
  as_tibble()
```

```
## # A tibble: 1,250 x 14
##   Direction   Lag1   Lag2   Lag3   Lag4   Lag5 Volume .fitted .se.fit
##   <fct>      <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl>   <dbl>   <dbl>
## 1 Up         0.381 -0.192 -2.62  -1.06   5.01    1.19   0.507  0.0732
## 2 Up         0.959  0.381 -0.192 -2.62  -1.06    1.30   0.481  0.0415
## 3 Down       1.03   0.959  0.381 -0.192 -2.62    1.41   0.481  0.0401
## 4 Up        -0.623  1.03   0.959  0.381 -0.192   1.28   0.515  0.0253
## 5 Up         0.614 -0.623  1.03   0.959  0.381   1.21   0.511  0.0275
## 6 Up         0.213  0.614 -0.623  1.03   0.959   1.35   0.507  0.0255
## # ... with 1,244 more rows, and 5 more variables: .resid <dbl>,
## #   .hat <dbl>, .sigma <dbl>, .cooksd <dbl>, .std.resid <dbl>
```

To convert these predicted probabilities to actual predictions using a $.5$ threshold, we create a new column using `mutate()` which checks the `.fitted` value and returns `Up` if the probability is greater than or equal to $.5$ and `Down` if the probability is less than $.5$.


```r
augment(glm_fit, type.predict = "response") %>%
  as_tibble() %>%
  mutate(.predict = ifelse(.fitted >= .5, "Up", "Down"))
```

```
## # A tibble: 1,250 x 15
##   Direction   Lag1   Lag2   Lag3   Lag4   Lag5 Volume .fitted .se.fit
##   <fct>      <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl>   <dbl>   <dbl>
## 1 Up         0.381 -0.192 -2.62  -1.06   5.01    1.19   0.507  0.0732
## 2 Up         0.959  0.381 -0.192 -2.62  -1.06    1.30   0.481  0.0415
## 3 Down       1.03   0.959  0.381 -0.192 -2.62    1.41   0.481  0.0401
## 4 Up        -0.623  1.03   0.959  0.381 -0.192   1.28   0.515  0.0253
## 5 Up         0.614 -0.623  1.03   0.959  0.381   1.21   0.511  0.0275
## 6 Up         0.213  0.614 -0.623  1.03   0.959   1.35   0.507  0.0255
## # ... with 1,244 more rows, and 6 more variables: .resid <dbl>,
## #   .hat <dbl>, .sigma <dbl>, .cooksd <dbl>, .std.resid <dbl>,
## #   .predict <chr>
```

We can create a confusion matrix by first counting the number Up/Down, Up/Up, Down/Up, and Down/Down pairs of actual and predicted outcomes, then using `spread()` from `tidyr` to cast the data frame into a wide format.


```r
augment(glm_fit, type.predict = "response") %>%
  as_tibble() %>%
  mutate(.predict = ifelse(.fitted >= .5, "Up", "Down")) %>%
  count(Direction, .predict) %>%
  spread(Direction, n)
```

```
## # A tibble: 2 x 3
##   .predict  Down    Up
##   <chr>    <int> <int>
## 1 Down       145   141
## 2 Up         457   507
```

Alternatively (and I think a bit more easily), the ISLR solution based on `table()` also works reasonably well:


```r
augment(glm_fit, type.predict = "response") %>%
  as_tibble() %>%
  mutate(.predict = ifelse(.fitted >= .5, "Up", "Down")) %>%
  with(table(.predict, Direction))
```

```
##         Direction
## .predict Down  Up
##     Down  145 141
##     Up    457 507
```

`with()` allows us to directly refer to the column names without any additional notation. To calculate the predictive accuracy of the model, use `mean()`:


```r
augment(glm_fit, type.predict = "response") %>%
  as_tibble() %>%
  mutate(.predict = ifelse(.fitted >= .5, "Up", "Down")) %>%
  with(mean(Direction != .predict))
```

```
## [1] 0.478
```

This is the **training error rate** (portion of observations where the actual outcome does not match the predicted outcome). To calculate the **test error rate**, we hold back a portion of the data to evaluate the model's effectiveness. Let's split the data into years 2001-04 and 2005:


```r
Smarket_0104 <- filter(Smarket, Year < 2005)
Smarket_05 <- filter(Smarket, Year == 2005)
Smarket_0104
```

```
## # A tibble: 998 x 9
##    Year   Lag1   Lag2   Lag3   Lag4   Lag5 Volume  Today Direction
##   <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl> <fct>    
## 1  2001  0.381 -0.192 -2.62  -1.06   5.01    1.19  0.959 Up       
## 2  2001  0.959  0.381 -0.192 -2.62  -1.06    1.30  1.03  Up       
## 3  2001  1.03   0.959  0.381 -0.192 -2.62    1.41 -0.623 Down     
## 4  2001 -0.623  1.03   0.959  0.381 -0.192   1.28  0.614 Up       
## 5  2001  0.614 -0.623  1.03   0.959  0.381   1.21  0.213 Up       
## 6  2001  0.213  0.614 -0.623  1.03   0.959   1.35  1.39  Up       
## # ... with 992 more rows
```

```r
Smarket_05
```

```
## # A tibble: 252 x 9
##    Year   Lag1   Lag2   Lag3   Lag4   Lag5 Volume  Today Direction
##   <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl> <fct>    
## 1  2005 -0.134  0.008 -0.007  0.715 -0.431  0.787 -0.812 Down     
## 2  2005 -0.812 -0.134  0.008 -0.007  0.715  1.51  -1.17  Down     
## 3  2005 -1.17  -0.812 -0.134  0.008 -0.007  1.72  -0.363 Down     
## 4  2005 -0.363 -1.17  -0.812 -0.134  0.008  1.74   0.351 Up       
## 5  2005  0.351 -0.363 -1.17  -0.812 -0.134  1.57  -0.143 Down     
## 6  2005 -0.143  0.351 -0.363 -1.17  -0.812  1.48   0.342 Up       
## # ... with 246 more rows
```

Let's now train the model using the 2001-04 data:


```r
glm_fit <- glm(Direction ~ Lag1 + Lag2 + Lag3 + Lag4 + Lag5 + Volume,
               data = Smarket_0104,
               family = binomial)
```

And evaluate it using the 2005 data. The difference from before is we specify `newdata = Smarket_05` to tell `augment()` to generate predicted values for the held-out 2005:


```r
augment(glm_fit, newdata = Smarket_05, type.predict = "response") %>%
  as_tibble() %>%
  mutate(.predict = ifelse(.fitted >= .5, "Up", "Down")) %>%
  with(mean(Direction != .predict))
```

```
## [1] 0.52
```

### Linear discriminant analysis

No `broom` implementation. Need to figure out how to proceed.

### Quadratic discriminant analysis

No `broom` implementation. Need to figure out how to proceed.

### $K$-nearest neighbors

Perform KNN using the `knn()` function in the `class` package. Unlike the past functions, we need to explicitly separate the predictors from the response variables. `knn()` requires four arguments:

1. `train` - a data frame containing the predictors for the training data
1. `test` - a data frame containing the predictors for the test data
1. `cl` - a vector containing the class labels (i.e. outcomes) for the training observations
1. `k` - the number of nearest neighbors to be used by the classifier

We use `select()` to create the appropriate data frames for 1 and 2.


```r
Smarket_0104_x <- select(Smarket_0104, -Direction)
Smarket_05_x <- select(Smarket_05, -Direction)
```

Then we pass these data frames to `knn()`. To ensure reproducibility, we set the random seed before applying this function.


```r
set.seed(1234)

library(class)
knn_pred <- knn(train = Smarket_0104_x,
                test = Smarket_05_x,
                cl = Smarket_0104$Direction,
                k = 1)
knn_pred
```

```
##   [1] Down Down Down Up   Up   Down Down Up   Down Up   Up   Down Down Down
##  [15] Up   Down Up   Up   Up   Up   Up   Up   Down Up   Down Up   Down Up  
##  [29] Up   Down Up   Up   Down Up   Down Up   Up   Up   Down Up   Down Up  
##  [43] Up   Up   Down Down Down Down Up   Up   Down Up   Down Down Down Up  
##  [57] Up   Up   Down Up   Down Up   Up   Up   Up   Up   Down Down Up   Down
##  [71] Down Down Up   Up   Down Up   Down Up   Down Up   Down Up   Up   Down
##  [85] Up   Up   Down Up   Down Up   Down Down Up   Up   Up   Up   Down Up  
##  [99] Up   Up   Up   Up   Down Up   Down Down Up   Down Down Up   Down Up  
## [113] Up   Up   Up   Up   Down Down Down Down Down Down Up   Up   Up   Down
## [127] Up   Down Up   Up   Up   Up   Up   Up   Up   Down Up   Up   Down Up  
## [141] Down Up   Up   Up   Down Down Up   Down Down Down Down Up   Up   Up  
## [155] Down Down Down Down Up   Up   Up   Down Down Down Down Up   Up   Up  
## [169] Down Down Up   Up   Down Up   Down Down Up   Down Up   Down Down Down
## [183] Up   Up   Down Up   Down Up   Down Down Down Down Down Up   Down Up  
## [197] Down Down Up   Up   Down Up   Down Down Up   Down Down Down Up   Up  
## [211] Up   Up   Up   Up   Down Down Up   Up   Up   Up   Down Up   Up   Up  
## [225] Up   Up   Up   Down Down Up   Down Up   Up   Down Up   Down Up   Up  
## [239] Up   Up   Up   Down Down Down Up   Down Up   Up   Down Up   Down Down
## Levels: Down Up
```

The output is a vector containing the predicted outcomes for the test data. We can generate the confusion matrix and the test error rate:


```r
table(knn_pred, Smarket_05$Direction)
```

```
##         
## knn_pred Down  Up
##     Down   92  22
##     Up     19 119
```

```r
data_frame(
  actual = Smarket_05$Direction,
  predict = knn_pred
)  %>%
  with(mean(actual != predict))
```

```
## [1] 0.163
```

Repeat with $K=3$ and compare performance:


```r
knn_pred <- knn(train = Smarket_0104_x,
                test = Smarket_05_x,
                cl = Smarket_0104$Direction,
                k = 3)

table(knn_pred, Smarket_05$Direction)
```

```
##         
## knn_pred Down  Up
##     Down   92  21
##     Up     19 120
```

```r
data_frame(
  actual = Smarket_05$Direction,
  predict = knn_pred
)  %>%
  with(mean(actual != predict))
```

```
## [1] 0.159
```

### An application to Caravan insurance data

Let's apply KNN to the `Caravan` data set from `ISLR`. The response variable is `Purchase` which indicates whether or not a given individual purchases a caravan insurance policy.


```r
Caravan <- as_tibble(Caravan)
Caravan
```

```
## # A tibble: 5,822 x 86
##   MOSTYPE MAANTHUI MGEMOMV MGEMLEEF MOSHOOFD MGODRK MGODPR MGODOV MGODGE
## *   <dbl>    <dbl>   <dbl>    <dbl>    <dbl>  <dbl>  <dbl>  <dbl>  <dbl>
## 1      33        1       3        2        8      0      5      1      3
## 2      37        1       2        2        8      1      4      1      4
## 3      37        1       2        2        8      0      4      2      4
## 4       9        1       3        3        3      2      3      2      4
## 5      40        1       4        2       10      1      4      1      4
## 6      23        1       2        1        5      0      5      0      5
## # ... with 5,816 more rows, and 77 more variables: MRELGE <dbl>,
## #   MRELSA <dbl>, MRELOV <dbl>, MFALLEEN <dbl>, MFGEKIND <dbl>,
## #   MFWEKIND <dbl>, MOPLHOOG <dbl>, MOPLMIDD <dbl>, MOPLLAAG <dbl>,
## #   MBERHOOG <dbl>, MBERZELF <dbl>, MBERBOER <dbl>, MBERMIDD <dbl>,
## #   MBERARBG <dbl>, MBERARBO <dbl>, MSKA <dbl>, MSKB1 <dbl>, MSKB2 <dbl>,
## #   MSKC <dbl>, MSKD <dbl>, MHHUUR <dbl>, MHKOOP <dbl>, MAUT1 <dbl>,
## #   MAUT2 <dbl>, MAUT0 <dbl>, MZFONDS <dbl>, MZPART <dbl>, MINKM30 <dbl>,
## #   MINK3045 <dbl>, MINK4575 <dbl>, MINK7512 <dbl>, MINK123M <dbl>,
## #   MINKGEM <dbl>, MKOOPKLA <dbl>, PWAPART <dbl>, PWABEDR <dbl>,
## #   PWALAND <dbl>, PPERSAUT <dbl>, PBESAUT <dbl>, PMOTSCO <dbl>,
## #   PVRAAUT <dbl>, PAANHANG <dbl>, PTRACTOR <dbl>, PWERKT <dbl>,
## #   PBROM <dbl>, PLEVEN <dbl>, PPERSONG <dbl>, PGEZONG <dbl>,
## #   PWAOREG <dbl>, PBRAND <dbl>, PZEILPL <dbl>, PPLEZIER <dbl>,
## #   PFIETS <dbl>, PINBOED <dbl>, PBYSTAND <dbl>, AWAPART <dbl>,
## #   AWABEDR <dbl>, AWALAND <dbl>, APERSAUT <dbl>, ABESAUT <dbl>,
## #   AMOTSCO <dbl>, AVRAAUT <dbl>, AAANHANG <dbl>, ATRACTOR <dbl>,
## #   AWERKT <dbl>, ABROM <dbl>, ALEVEN <dbl>, APERSONG <dbl>,
## #   AGEZONG <dbl>, AWAOREG <dbl>, ABRAND <dbl>, AZEILPL <dbl>,
## #   APLEZIER <dbl>, AFIETS <dbl>, AINBOED <dbl>, ABYSTAND <dbl>,
## #   Purchase <fct>
```

```r
Caravan %>%
  count(Purchase) %>%
  mutate(pct = n / sum(n))
```

```
## # A tibble: 2 x 3
##   Purchase     n    pct
##   <fct>    <int>  <dbl>
## 1 No        5474 0.940 
## 2 Yes        348 0.0598
```

Only approximately 6% of individuals in the dataset purchased a caravan insurance policy.

To perform KNN, first we standardize the data set using the `scale()` function. `scale()` normalizes any vector/variable to mean 0 and standard deviation 1. To apply this standardization to each column in `Caravan` (except for the `Purchase` column), we use `mutate_at()` to apply the same mutation function to multiple columns.


```r
Caravan_scale <- Caravan %>%
  mutate_at(.vars = vars(-Purchase), .funs = funs(scale(.) %>% as.vector))

# confirm the transformation worked
Caravan_scale %>%
  summarize_at(.vars = vars(-Purchase), .funs = funs(mean, sd)) %>%
  glimpse()
```

```
## Observations: 1
## Variables: 170
## $ MOSTYPE_mean  <dbl> -7.03e-17
## $ MAANTHUI_mean <dbl> -1.47e-16
## $ MGEMOMV_mean  <dbl> -1.78e-16
## $ MGEMLEEF_mean <dbl> 2.04e-16
## $ MOSHOOFD_mean <dbl> -1.46e-17
## $ MGODRK_mean   <dbl> -4.68e-17
## $ MGODPR_mean   <dbl> 7.08e-17
## $ MGODOV_mean   <dbl> 1.08e-17
## $ MGODGE_mean   <dbl> 9.04e-17
## $ MRELGE_mean   <dbl> -1.35e-16
## $ MRELSA_mean   <dbl> -1.44e-17
## $ MRELOV_mean   <dbl> -1.2e-16
## $ MFALLEEN_mean <dbl> -1.13e-18
## $ MFGEKIND_mean <dbl> 4.74e-17
## $ MFWEKIND_mean <dbl> -6.68e-17
## $ MOPLHOOG_mean <dbl> -4.63e-17
## $ MOPLMIDD_mean <dbl> 1.07e-16
## $ MOPLLAAG_mean <dbl> 4.04e-17
## $ MBERHOOG_mean <dbl> -2.21e-17
## $ MBERZELF_mean <dbl> 1.14e-17
## $ MBERBOER_mean <dbl> -3.95e-17
## $ MBERMIDD_mean <dbl> -1.71e-17
## $ MBERARBG_mean <dbl> 1.16e-16
## $ MBERARBO_mean <dbl> -1.03e-16
## $ MSKA_mean     <dbl> -6.42e-17
## $ MSKB1_mean    <dbl> 5.22e-17
## $ MSKB2_mean    <dbl> -3.22e-18
## $ MSKC_mean     <dbl> 3.38e-17
## $ MSKD_mean     <dbl> 1.03e-16
## $ MHHUUR_mean   <dbl> 1.08e-17
## $ MHKOOP_mean   <dbl> -3.33e-17
## $ MAUT1_mean    <dbl> -2.71e-16
## $ MAUT2_mean    <dbl> 1.02e-16
## $ MAUT0_mean    <dbl> 1.76e-17
## $ MZFONDS_mean  <dbl> 4.33e-17
## $ MZPART_mean   <dbl> -3.26e-17
## $ MINKM30_mean  <dbl> -8.52e-17
## $ MINK3045_mean <dbl> 4.2e-17
## $ MINK4575_mean <dbl> -3.71e-17
## $ MINK7512_mean <dbl> -2.19e-17
## $ MINK123M_mean <dbl> -2.4e-17
## $ MINKGEM_mean  <dbl> 1.6e-16
## $ MKOOPKLA_mean <dbl> -1.85e-16
## $ PWAPART_mean  <dbl> -3.31e-17
## $ PWABEDR_mean  <dbl> -2.36e-18
## $ PWALAND_mean  <dbl> -2.25e-17
## $ PPERSAUT_mean <dbl> -1.57e-17
## $ PBESAUT_mean  <dbl> 2.32e-18
## $ PMOTSCO_mean  <dbl> -1.64e-18
## $ PVRAAUT_mean  <dbl> 5.54e-18
## $ PAANHANG_mean <dbl> 5.31e-18
## $ PTRACTOR_mean <dbl> 2.94e-17
## $ PWERKT_mean   <dbl> -4.79e-18
## $ PBROM_mean    <dbl> 2.24e-17
## $ PLEVEN_mean   <dbl> -1.39e-18
## $ PPERSONG_mean <dbl> -7.14e-19
## $ PGEZONG_mean  <dbl> -4.44e-18
## $ PWAOREG_mean  <dbl> -4.16e-18
## $ PBRAND_mean   <dbl> 3.03e-17
## $ PZEILPL_mean  <dbl> 3.52e-19
## $ PPLEZIER_mean <dbl> 1.48e-18
## $ PFIETS_mean   <dbl> 2.25e-17
## $ PINBOED_mean  <dbl> -1.32e-18
## $ PBYSTAND_mean <dbl> -1.42e-17
## $ AWAPART_mean  <dbl> -3.76e-17
## $ AWABEDR_mean  <dbl> -8.27e-18
## $ AWALAND_mean  <dbl> 1.18e-17
## $ APERSAUT_mean <dbl> 2.61e-17
## $ ABESAUT_mean  <dbl> -4.78e-18
## $ AMOTSCO_mean  <dbl> 2.48e-17
## $ AVRAAUT_mean  <dbl> 5.18e-18
## $ AAANHANG_mean <dbl> 4.88e-18
## $ ATRACTOR_mean <dbl> -9.36e-18
## $ AWERKT_mean   <dbl> -2.05e-18
## $ ABROM_mean    <dbl> 9.16e-18
## $ ALEVEN_mean   <dbl> -9.64e-18
## $ APERSONG_mean <dbl> 7.86e-18
## $ AGEZONG_mean  <dbl> 1.1e-18
## $ AWAOREG_mean  <dbl> 2.2e-18
## $ ABRAND_mean   <dbl> -4.89e-17
## $ AZEILPL_mean  <dbl> 2.85e-18
## $ APLEZIER_mean <dbl> -1.06e-18
## $ AFIETS_mean   <dbl> 7.78e-18
## $ AINBOED_mean  <dbl> 6.96e-18
## $ ABYSTAND_mean <dbl> -9.91e-18
## $ MOSTYPE_sd    <dbl> 1
## $ MAANTHUI_sd   <dbl> 1
## $ MGEMOMV_sd    <dbl> 1
## $ MGEMLEEF_sd   <dbl> 1
## $ MOSHOOFD_sd   <dbl> 1
## $ MGODRK_sd     <dbl> 1
## $ MGODPR_sd     <dbl> 1
## $ MGODOV_sd     <dbl> 1
## $ MGODGE_sd     <dbl> 1
## $ MRELGE_sd     <dbl> 1
## $ MRELSA_sd     <dbl> 1
## $ MRELOV_sd     <dbl> 1
## $ MFALLEEN_sd   <dbl> 1
## $ MFGEKIND_sd   <dbl> 1
## $ MFWEKIND_sd   <dbl> 1
## $ MOPLHOOG_sd   <dbl> 1
## $ MOPLMIDD_sd   <dbl> 1
## $ MOPLLAAG_sd   <dbl> 1
## $ MBERHOOG_sd   <dbl> 1
## $ MBERZELF_sd   <dbl> 1
## $ MBERBOER_sd   <dbl> 1
## $ MBERMIDD_sd   <dbl> 1
## $ MBERARBG_sd   <dbl> 1
## $ MBERARBO_sd   <dbl> 1
## $ MSKA_sd       <dbl> 1
## $ MSKB1_sd      <dbl> 1
## $ MSKB2_sd      <dbl> 1
## $ MSKC_sd       <dbl> 1
## $ MSKD_sd       <dbl> 1
## $ MHHUUR_sd     <dbl> 1
## $ MHKOOP_sd     <dbl> 1
## $ MAUT1_sd      <dbl> 1
## $ MAUT2_sd      <dbl> 1
## $ MAUT0_sd      <dbl> 1
## $ MZFONDS_sd    <dbl> 1
## $ MZPART_sd     <dbl> 1
## $ MINKM30_sd    <dbl> 1
## $ MINK3045_sd   <dbl> 1
## $ MINK4575_sd   <dbl> 1
## $ MINK7512_sd   <dbl> 1
## $ MINK123M_sd   <dbl> 1
## $ MINKGEM_sd    <dbl> 1
## $ MKOOPKLA_sd   <dbl> 1
## $ PWAPART_sd    <dbl> 1
## $ PWABEDR_sd    <dbl> 1
## $ PWALAND_sd    <dbl> 1
## $ PPERSAUT_sd   <dbl> 1
## $ PBESAUT_sd    <dbl> 1
## $ PMOTSCO_sd    <dbl> 1
## $ PVRAAUT_sd    <dbl> 1
## $ PAANHANG_sd   <dbl> 1
## $ PTRACTOR_sd   <dbl> 1
## $ PWERKT_sd     <dbl> 1
## $ PBROM_sd      <dbl> 1
## $ PLEVEN_sd     <dbl> 1
## $ PPERSONG_sd   <dbl> 1
## $ PGEZONG_sd    <dbl> 1
## $ PWAOREG_sd    <dbl> 1
## $ PBRAND_sd     <dbl> 1
## $ PZEILPL_sd    <dbl> 1
## $ PPLEZIER_sd   <dbl> 1
## $ PFIETS_sd     <dbl> 1
## $ PINBOED_sd    <dbl> 1
## $ PBYSTAND_sd   <dbl> 1
## $ AWAPART_sd    <dbl> 1
## $ AWABEDR_sd    <dbl> 1
## $ AWALAND_sd    <dbl> 1
## $ APERSAUT_sd   <dbl> 1
## $ ABESAUT_sd    <dbl> 1
## $ AMOTSCO_sd    <dbl> 1
## $ AVRAAUT_sd    <dbl> 1
## $ AAANHANG_sd   <dbl> 1
## $ ATRACTOR_sd   <dbl> 1
## $ AWERKT_sd     <dbl> 1
## $ ABROM_sd      <dbl> 1
## $ ALEVEN_sd     <dbl> 1
## $ APERSONG_sd   <dbl> 1
## $ AGEZONG_sd    <dbl> 1
## $ AWAOREG_sd    <dbl> 1
## $ ABRAND_sd     <dbl> 1
## $ AZEILPL_sd    <dbl> 1
## $ APLEZIER_sd   <dbl> 1
## $ AFIETS_sd     <dbl> 1
## $ AINBOED_sd    <dbl> 1
## $ ABYSTAND_sd   <dbl> 1
```

We can now fit the KNN model. First we split the observations into a test set containing the first 1,000 observations, and a training set containing the remaining observations. Then we fit a KNN model using the training data and $K=1$ and evaluate its performance on the test data.


```r
Caravan_test <- slice(Caravan_scale, 1:1000)
Caravan_train <- slice(Caravan_scale, 1001:n())

Caravan_test_x <- select(Caravan_test, -Purchase)
Caravan_train_x <- select(Caravan_train, -Purchase)

set.seed(1)
knn_pred <- knn(train = Caravan_train_x,
                test = Caravan_test_x,
                cl = Caravan_train$Purchase,
                k = 1)

mean(Caravan_test$Purchase != knn_pred)   # test error rate
```

```
## [1] 0.118
```

```r
mean(Caravan_test$Purchase != "No")   # null baseline
```

```
## [1] 0.059
```

Compared to predicting "No" for each individual, this model performs poorly. If we only look at those predicted to buy insurance, the model actually performs better:


```r
table(knn_pred, Caravan_test$Purchase)
```

```
##         
## knn_pred  No Yes
##      No  873  50
##      Yes  68   9
```

```r
mean(Caravan_test$Purchase[knn_pred == "Yes"] == knn_pred[knn_pred == "Yes"])
```

```
## [1] 0.117
```

Among those predicted to purchase insurance, $11.69%$ actually do purchase insurance. This rate improves using $K=3$ and $K=5$


```r
knn_pred <- knn(train = Caravan_train_x,
                test = Caravan_test_x,
                cl = Caravan_train$Purchase,
                k = 3)
mean(Caravan_test$Purchase[knn_pred == "Yes"] == knn_pred[knn_pred == "Yes"])
```

```
## [1] 0.192
```

```r
knn_pred <- knn(train = Caravan_train_x,
                test = Caravan_test_x,
                cl = Caravan_train$Purchase,
                k = 5)
mean(Caravan_test$Purchase[knn_pred == "Yes"] == knn_pred[knn_pred == "Yes"])
```

```
## [1] 0.267
```

We can compare the performance of KNN to a logistic regression model. By relaxing the threshold for predicting purchase of insurance from $0.5$ to $0.25$, our model's test error rate improves even more than for the KNN model.


```r
glm_fit <- glm(Purchase ~ .,
               data = Caravan_train,
               family = binomial)
```

```
## Warning: glm.fit: fitted probabilities numerically 0 or 1 occurred
```

```r
augment(glm_fit, newdata = Caravan_test, type.predict = "response") %>%
  as_tibble() %>%
  # generate prediction
  mutate(.predict = ifelse(.fitted >= .25, "Yes", "No")) %>%
  # only evaluate individuals predicted to purchase insurance
  filter(.predict == "Yes") %>%
  # calculate accuracy rate for this subset
  with(mean(Purchase == .predict))
```

```
## [1] 0.333
```



## Session information {.toc-ignore}


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
##  date     2018-07-24
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
