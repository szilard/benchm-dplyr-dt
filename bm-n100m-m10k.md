

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
m <- 1e4

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
## 1 d <- data_frame(x = sample(m, n, replace = TRUE), y = runif(n))   5.920
## 2                                                 dt <- tbl_dt(d)   0.665
## 3                                                dtk <- tbl_dt(d)   0.668
## 4                                                  setkey(dtk, x)   6.807
## 5                                 dm <- data_frame(x = sample(m))   0.000
## 6                                               dtm <- tbl_dt(dm)   0.000
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
## 1       d[d$x >= 10 & d$x < 20, ]   9.664    1.965
## 2   d %>% filter(x >= 10, x < 20)   5.277    1.073
## 3  dt %>% filter(x >= 10, x < 20)   4.925    1.001
## 4 dtk %>% filter(x >= 10, x < 20)   4.918    1.000
## 5            dt[x >= 10 & x < 20]   4.926    1.002
## 6           dtk[x >= 10 & x < 20]   4.918    1.000
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
## 1    d[order(d$x), ] 165.217   60.831
## 2   d %>% arrange(x)  88.982   32.762
## 3  dt %>% arrange(x)   7.496    2.760
## 4 dtk %>% arrange(x)   2.716    1.000
## 5       dt[order(x)]   7.505    2.763
## 6      dtk[order(x)]   2.717    1.000
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
## 1            d$y2 <- 2 * d$y   0.386    1.000
## 2   d %>% mutate(y2 = 2 * y)   0.387    1.003
## 3  dt %>% mutate(y2 = 2 * y)   2.474    6.409
## 4 dtk %>% mutate(y2 = 2 * y)   2.474    6.409
## 5      dt[, `:=`(y2, 2 * y)]   1.432    3.710
## 6     dtk[, `:=`(y2, 2 * y)]   1.429    3.702
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
## 1                          tapply(d$y, d$x, mean)  23.467   17.552
## 2   d %>% group_by(x) %>% summarize(ym = mean(y))  12.150    9.088
## 3  dt %>% group_by(x) %>% summarize(ym = mean(y))   8.534    6.383
## 4 dtk %>% group_by(x) %>% summarize(ym = mean(y))   2.172    1.625
## 5                           dt[, mean(y), by = x]   4.766    3.565
## 6                          dtk[, mean(y), by = x]   1.337    1.000
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
## 1    d %>% inner_join(dm, by = "x")  10.743    8.029
## 2  dt %>% inner_join(dtm, by = "x")  10.285    7.687
## 3 dtk %>% inner_join(dtm, by = "x")   3.869    2.892
## 4             dtk[dtm, nomatch = 0]   1.338    1.000
```

base is too slow (>100x)




