
### Simple/basic/limited/incomplete benchmark for dplyr and data.table 

For parameters `n = 10M, 100M` and `m = 100, 10K, 1M`, create data.frames
```{r eval=FALSE}
d <- data.frame(x = sample(m, n, replace=TRUE), y = runif(n))
dm <- data.frame(x = sample(m))
```
and corresponding data.tables with and without key on `x` (`d`'s size in RAM is
around 100MB and 1GB, respectively).

The basic tabular operations (filter, aggregate, join etc.) are applied using base, dplyr (with data.frame and data.table backends, with and without key for data.table) and standard data.table (with and without key).

This is just a simple/basic/limited/incomplete benchmark, could do more with various data types (e.g. character), several grouping variables (x1,x2,...), more values for size parameters (n,m), different distributions of values in the data.frames etc. (or with real-world datasets). 


##### Filter 

```{r eval=FALSE}
d[d$x>=10 & d$x<20,]
d %>% filter(x>=10, x<20)
dt[x>=10 & x<20]
```

##### Sort

```{r eval=FALSE}
d[order(d$x),]
d %>% arrange(x)
dt[order(x)]
```

##### New column

```{r eval=FALSE}
d$y2 <- 2*d$y
d %>% mutate(y2 = 2*y)
dt[,y2 := 2*y]
```

##### Aggregation

```{r eval=FALSE}
tapply(d$y, d$x, mean)
d %>% group_by(x) %>% summarize(ym = mean(y))
dt[, mean(y), by=x]
```

##### Join

```{r eval=FALSE}
merge(d, dm, by="x")
d %>% inner_join(dm, by="x")
dt[dtm, nomatch=0]
```


#### Results

Full code in `bm.Rmd` and results for each n,m in `bm-nxx-mxx.md` files in the repo. Latest CRAN 
versions of R, dplyr and data.table have been used (R 3.1.1, dplyr 0.3.0.2 and data.table 1.9.4). 
A summary of results is here:

|                 |    base     |   dplyr-df  |  dplyr-dt  |  dplyr-dt-k  |     dt     |     dt-k    |
| --------------- | ----------- | ----------- | ---------- | ------------ | ---------- |  ---------- |
| Filter          |     2       |     1       |     1      |      1       |      1     |       1     |
| Sort            |    30-60    |   20-30     |   1.5-3    |      1       |   1.5-3    |       1     |
| New column      |     1       |     1       |     6      |      6       |      4     |       4     |
| Aggregation     |    8-100    |    4-30     |    4-6     |     1.5      |   1.5-5    |       1     |
| Join            |    >100     |    4-15     |    4-6     |   1.5-2.5    |    cannot  |       1     |



##### Obvious findings:

- Having a key (which for data.table it means having the data pre-sorted in place) helps with
sorting, aggregation and joins.


##### Surprize (for me):

- dplyr with data.table backend almost as fast as data.table - 
so, it looks like you can have both: dplyr API (my personal preference) and speed

- Defining a new column in data.table (or dplyr with the data.table backend) is slow

These are by no means a critique of the dplyr and data.table developers. They are great people
who did a lot of good by working on these excellent open source tools. I'm just trying to 
understand things.


##### To understand (for me/maybe developers):

- Why is dplyr with data.frame backend slow (vs. dplyr with data.table backend) - esp. for
sorting, aggregation with a large number of groups and joins with a large table

- Why is defining a new column in data.table slow (vs. base/dplyr) - while data.table used to be
\>100x faster than base, R 3.1 modifies data.frames in place and now data.table is slower


##### To do:

- Benchmark a chain of operations













