
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
A summary of results (relative running times, lower is better) is here:

|                 |    base     |   dplyr-df  |  dplyr-dt  |  dplyr-dt-k  |     dt     |     dt-k    |
| --------------- | ----------- | ----------- | ---------- | ------------ | ---------- |  ---------- |
| Filter          |     2       |     1       |     1      |      1       |      1     |       1     |
| Sort            |    30-60    |   20-30     |   1.5-3    |      1       |   1.5-3    |       1     |
| New column      |     1       |     1       |     6      |      6       |      4     |       4     |
| Aggregation     |    8-100    |    4-30     |    4-6     |     1.5      |   1.5-5    |       1     |
| Join            |    >100     |    4-15     |    4-6     |   1.5-2.5    |    cannot  |       1     |

(the larger numbers are usually for larger `m`, i.e. lots of small groups)


##### Discussion:

- Having a key (which for data.table it means having the data pre-sorted in place) obviously helps with
sorting, aggregation and joins (depending on the use case though, the time to generate the key 
should be added to the timing)

- dplyr with data.table backend/source is almost as fast as plain data.table (because in this case dplyr acts as a wrapper and calls data.table functions behind the scenes) - 
so, you can kindda have both: dplyr API (my personal preference) and speed

- dplyr with data.frame source is slower than data.table for sort, aggregation and joins. Some of
this has apparently to do with radix sort and binary search joining (data.table) being faster
than hash-table based joins (dplyr) [as described here](https://gist.github.com/arunsrinivasan/db6e1ce05227f120a2c9), but some of it is likely to be improved as [Hadley said here](https://twitter.com/hadleywickham/status/527162872200065025).

- Defining a new column in data.table (or dplyr with the data.table backend) is slower. I pointed out this to data.table developers Matt and Arun and [this can be fixed](https://github.com/Rdatatable/data.table/issues/921). The extra slowdown in creating a new column with dplyr with data.table source (vs plain data.table) [can also be fixed](https://github.com/hadley/dplyr/issues/614).

##### More info:

I'm going to give a short 15-min talk at the [next LA R meetup](http://datascience.la/la-r-meetup-november-11-highlights-from-the-user-2014-conference-part-2/) about dplyr, and I'll talk about
these results as well, [slides here](https://speakerdeck.com/datasciencela/szilard-pafka-dplyr-plus-basic-benchmark-la-r-meetup-nov-2014).

There are several other benchmarks, for example Matt's [benchmark of group-by](https://github.com/Rdatatable/data.table/wiki/Benchmarks-%3A-Grouping), or Brodie Gaslam's 
[benchmark of group-by and mutate](http://www.brodieg.com/?p=7). My goal was to look at a wider
range of operations (but keep the work minimal, so I had to concentrate on a few samples) - 
and I also wanted to understand the reasons for such performance, and in this respect I'd like
to thank the developers for the useful pointers.

