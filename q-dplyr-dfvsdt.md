

```r
library(dplyr)

library(rbenchmark)
set.seed(123)
```


```r
n <- 10e6
m <- 1e6

d <- data_frame(x = sample(m, n, replace=TRUE), y = runif(n))
dt <- tbl_dt(d)
```

```
## Loading required namespace: data.table
```

```r
benchmark(
  d %>% group_by(x) %>% summarize(ym = mean(y)),
  dt %>% group_by(x) %>% summarize(ym = mean(y)),
replications=1, columns=c("test", "elapsed", "relative"), order=NULL)
```

```
##                                             test elapsed relative
## 1  d %>% group_by(x) %>% summarize(ym = mean(y))   7.377    7.275
## 2 dt %>% group_by(x) %>% summarize(ym = mean(y))   1.014    1.000
```
