

#### Setup


```r
library(data.table)
library(dplyr)

library(rbenchmark)
set.seed(123)
```


#### Generate data


```r
n <- 100e6
m <- 1e6

benchmark(
  d <- data_frame(x = sample(m, n, replace=TRUE), y = runif(n)),
  dt <- tbl_dt(d),
  dtk <- tbl_dt(d),
  setkey(dtk,x),
  dm <- data_frame(x = sample(m)),
  dtm <- tbl_dt(dm),
replications=1, columns=c("test", "elapsed"), order=NULL)
```

```
##                                                              test elapsed
## 1 d <- data_frame(x = sample(m, n, replace = TRUE), y = runif(n))   6.011
## 2                                                 dt <- tbl_dt(d)   0.668
## 3                                                dtk <- tbl_dt(d)   0.670
## 4                                                  setkey(dtk, x)   9.194
## 5                                 dm <- data_frame(x = sample(m))   0.021
## 6                                               dtm <- tbl_dt(dm)   0.001
```


#### Filter


```r
benchmark(
  d[d$x>=10 & d$x<20,],
  d %>% filter(x>=10, x<20),
  dt %>% filter(x>=10, x<20),
  dtk %>% filter(x>=10, x<20),
  dt[x>=10 & x<20],
  dtk[x>=10 & x<20],
replications=1, columns=c("test", "elapsed", "relative"), order=NULL)
```

```
##                              test elapsed relative
## 1       d[d$x >= 10 & d$x < 20, ]   9.621    1.956
## 2   d %>% filter(x >= 10, x < 20)   5.264    1.070
## 3  dt %>% filter(x >= 10, x < 20)   4.922    1.001
## 4 dtk %>% filter(x >= 10, x < 20)   4.920    1.000
## 5            dt[x >= 10 & x < 20]   4.919    1.000
## 6           dtk[x >= 10 & x < 20]   4.919    1.000
```

maybe dt has some tricks to use the "key" for ranges?


#### Sort


```r
benchmark(
  d[order(d$x),],
  d %>% arrange(x),
  dt %>% arrange(x),
  dtk %>% arrange(x),
  dt[order(x)],  
  dtk[order(x)],
replications=1, columns=c("test", "elapsed", "relative"), order=NULL)
```

```
##                 test elapsed relative
## 1    d[order(d$x), ] 196.298   70.891
## 2   d %>% arrange(x) 124.377   44.918
## 3  dt %>% arrange(x)  11.216    4.051
## 4 dtk %>% arrange(x)   2.770    1.000
## 5       dt[order(x)]  11.214    4.050
## 6      dtk[order(x)]   2.769    1.000
```


#### New column


```r
benchmark(
  d$y2 <- 2*d$y,
  d %>% mutate(y2 = 2*y),
  dt %>% mutate(y2 = 2*y),
  dtk %>% mutate(y2 = 2*y),
  dt[,y2 := 2*y],
  dtk[,y2 := 2*y],
replications=1, columns=c("test", "elapsed", "relative"), order=NULL)
```

```
##                         test elapsed relative
## 1            d$y2 <- 2 * d$y   0.414    1.000
## 2   d %>% mutate(y2 = 2 * y)   0.414    1.000
## 3  dt %>% mutate(y2 = 2 * y)   2.530    6.111
## 4 dtk %>% mutate(y2 = 2 * y)   2.528    6.106
## 5      dt[, `:=`(y2, 2 * y)]   1.461    3.529
## 6     dtk[, `:=`(y2, 2 * y)]   1.461    3.529
```

```r
invisible(dt[,y2 := NULL])
invisible(dtk[,y2 := NULL])
```


#### Aggregate


```r
benchmark(
  tapply(d$y, d$x, mean),
  d %>% group_by(x) %>% summarize(ym = mean(y)),
  dt %>% group_by(x) %>% summarize(ym = mean(y)),
  dtk %>% group_by(x) %>% summarize(ym = mean(y)),
  dt[, mean(y), by=x],
  dtk[, mean(y), by=x],
replications=1, columns=c("test", "elapsed", "relative"), order=NULL)
```

```
##                                              test elapsed relative
## 1                          tapply(d$y, d$x, mean)  67.843   42.966
## 2   d %>% group_by(x) %>% summarize(ym = mean(y))  45.621   28.892
## 3  dt %>% group_by(x) %>% summarize(ym = mean(y))  11.828    7.491
## 4 dtk %>% group_by(x) %>% summarize(ym = mean(y))   2.663    1.687
## 5                           dt[, mean(y), by = x]   9.645    6.108
## 6                          dtk[, mean(y), by = x]   1.579    1.000
```


#### Joins


```r
benchmark(
  # merge(d,dm, by="x"),
  d %>% inner_join(dm, by="x"),
  dt %>% inner_join(dtm, by="x"),
  dtk %>% inner_join(dtm, by="x"),
  dtk[dtm, nomatch=0],
replications=1, columns=c("test", "elapsed", "relative"), order=NULL)
```

```
##                                test elapsed relative
## 1    d %>% inner_join(dm, by = "x")  47.163   15.560
## 2  dt %>% inner_join(dtm, by = "x")  13.965    4.607
## 3 dtk %>% inner_join(dtm, by = "x")   4.736    1.563
## 4             dtk[dtm, nomatch = 0]   3.031    1.000
```

base is too slow (>100x)




