

#### Setup


```r
library(data.table)
library(dplyr)

library(rbenchmark)
set.seed(123)
```


#### Generate data


```r
n <- 10e6
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
## 1 d <- data_frame(x = sample(m, n, replace = TRUE), y = runif(n))   0.632
## 2                                                 dt <- tbl_dt(d)   0.103
## 3                                                dtk <- tbl_dt(d)   0.103
## 4                                                  setkey(dtk, x)   0.733
## 5                                 dm <- data_frame(x = sample(m))   0.022
## 6                                               dtm <- tbl_dt(dm)   0.002
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
## 1       d[d$x >= 10 & d$x < 20, ]   0.968    1.956
## 2   d %>% filter(x >= 10, x < 20)   0.529    1.069
## 3  dt %>% filter(x >= 10, x < 20)   0.495    1.000
## 4 dtk %>% filter(x >= 10, x < 20)   0.496    1.002
## 5            dt[x >= 10 & x < 20]   0.495    1.000
## 6           dtk[x >= 10 & x < 20]   0.495    1.000
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
## 1    d[order(d$x), ]   9.991   34.812
## 2   d %>% arrange(x)   6.707   23.369
## 3  dt %>% arrange(x)   0.806    2.808
## 4 dtk %>% arrange(x)   0.287    1.000
## 5       dt[order(x)]   0.806    2.808
## 6      dtk[order(x)]   0.287    1.000
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
## 1            d$y2 <- 2 * d$y   0.039    1.000
## 2   d %>% mutate(y2 = 2 * y)   0.040    1.026
## 3  dt %>% mutate(y2 = 2 * y)   0.249    6.385
## 4 dtk %>% mutate(y2 = 2 * y)   0.249    6.385
## 5      dt[, `:=`(y2, 2 * y)]   0.144    3.692
## 6     dtk[, `:=`(y2, 2 * y)]   0.144    3.692
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
## 1                          tapply(d$y, d$x, mean)  18.222   97.968
## 2   d %>% group_by(x) %>% summarize(ym = mean(y))   6.660   35.806
## 3  dt %>% group_by(x) %>% summarize(ym = mean(y))   0.901    4.844
## 4 dtk %>% group_by(x) %>% summarize(ym = mean(y))   0.260    1.398
## 5                           dt[, mean(y), by = x]   1.036    5.570
## 6                          dtk[, mean(y), by = x]   0.186    1.000
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
## 1    d %>% inner_join(dm, by = "x")   7.462   14.350
## 2  dt %>% inner_join(dtm, by = "x")   1.418    2.727
## 3 dtk %>% inner_join(dtm, by = "x")   0.591    1.137
## 4             dtk[dtm, nomatch = 0]   0.520    1.000
```

base is too slow (>100x)




