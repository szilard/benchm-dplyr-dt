

```r
library(data.table)

library(rbenchmark)
set.seed(123)
```


```r
n <- 100e6

d <- data.frame(y = runif(n))
dt <- as.data.table(d)

benchmark(
  d$y2 <- 2*d$y,
  dt[,y2 := 2*y],
replications=1, columns=c("test", "elapsed", "relative"), order=NULL)
```

```
##                    test elapsed relative
## 1       d$y2 <- 2 * d$y   0.387    1.000
## 2 dt[, `:=`(y2, 2 * y)]   1.553    4.013
```
