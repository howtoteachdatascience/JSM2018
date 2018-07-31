Basic data wrangling
================
Rafael Irizarry
July 30, 2018

Basic data wrangling
====================

We find that students often strugle with R, is its inconsistency in syntax and output of functions. As an example consider that if we define

``` r
x <- data.frame(a = 1:6, b = 7:12)
```

subsetting can produce a data.frame or a vector:

``` r
class(x[,1:2])
```

    ## [1] "data.frame"

``` r
class(x[,1])
```

    ## [1] "integer"

We also have several ways to access the columns

``` r
x$a
```

    ## [1] 1 2 3 4 5 6

``` r
x[["a"]]
```

    ## [1] 1 2 3 4 5 6

``` r
x[[1]]
```

    ## [1] 1 2 3 4 5 6

and differnt types of parenthesis

``` r
class(x[1])
```

    ## [1] "data.frame"

``` r
class(x[[1]])
```

    ## [1] "integer"

Our experience is that introducing data science with the tidyverse provides a easier entry point for students. The tidyverse imposes some strong restrictions, for exampl, it is meant to work with data frames exclusively, it permits the analysis of a suprprinsingly broad set of problems. Here we introduce some basics.

We will start by introducing the `dplyr` package which provides intuitive functionality for working with tables. We later use `dplyr` to perform some more advanced data wrangling operations.

Once you install `dplyr` you can load it using:

``` r
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

This package introduces functions that perform the most common operations in data wrangling and uses names for these functions that are relatively easy to remember. For instance, to change the data table by adding a new column, we use `mutate`. To filter the data table to a subset of rows, we use `filter`. Finally, to subset the data by selecting specific columns, we use `select`. We can also perform a series of operations, for example select and then filter, by sending the results of one function to another using what is called the *pipe operator*: `%>%`.
\#\# Case study

To illustrate how this work we introduce the US gun muders dataset, included in the dslabs package:

``` r
library(dslabs)
data("murders")
```

It includes US gun murders by state for 2010

``` r
head(murders)
```

    ##        state abb region population total
    ## 1    Alabama  AL  South    4779736   135
    ## 2     Alaska  AK   West     710231    19
    ## 3    Arizona  AZ   West    6392017   232
    ## 4   Arkansas  AR  South    2915918    93
    ## 5 California  CA   West   37253956  1257
    ## 6   Colorado  CO   West    5029196    65

Our taks will be to convince a European colleague with a job offer from the US, worried about the high US murder rate, that there is actually quite a bit of variability. We will compute state-level murder rate and then filter states by different qualities.

Adding a column with `mutate`
-----------------------------

We want all the necessary information for our analysis to be included in the data table. So the first task is to add the murder rates to our data frame.

Using regular R sytax we could write this:

``` r
murders$rate = murders$total / murders$population * 100000
```

The function mutate provides a slighty more readeable way of doingthis. It takes the data frame as a first argument and the name and values of the variable in the second using the convention `name = values`. So to add murder rates we use:

``` r
murders <- mutate(murders, rate = total / population * 100000)
```

Notice that here we used unquoted `total` and `population` inside the function, which are objects that are **not** defined in our workspace. So why do we not get an error?

This is one of the main features of dplyr. Function in this package, such `mutate`, know to look for variables in the data frame provided in the first argument. So in the call to mutate above, `total` will have the values in `murders$total`. This approach makes the code much more readable.

We can see that the new column is added:

``` r
head(murders)
```

    ##        state abb region population total     rate
    ## 1    Alabama  AL  South    4779736   135 2.824424
    ## 2     Alaska  AK   West     710231    19 2.675186
    ## 3    Arizona  AZ   West    6392017   232 3.629527
    ## 4   Arkansas  AR  South    2915918    93 3.189390
    ## 5 California  CA   West   37253956  1257 3.374138
    ## 6   Colorado  CO   West    5029196    65 1.292453

The tidyverse version of this operation is not that much readable that then R-base, but now we will see some better examples.

Subsetting with `filter`
------------------------

Now suppose that we want to filter the data table to only show the entries for which the murder rate is lower than 0.71. To do this, we use the `filter` function which takes the data table as an argument and then the conditional statement as the next. Like mutate, we can use the unquoted variable names from `murders` inside the function and it will know we mean the columns and not objects in the workspace.

This is what this opreation looks like in R-base:

``` r
murders$rate[murders$rate <= 0.71]
```

The tidyverse version is much more readable:

``` r
filter(murders, rate <= 0.71)
```

    ##           state abb        region population total      rate
    ## 1        Hawaii  HI          West    1360301     7 0.5145920
    ## 2          Iowa  IA North Central    3046355    21 0.6893484
    ## 3 New Hampshire  NH     Northeast    1316470     5 0.3798036
    ## 4  North Dakota  ND North Central     672591     4 0.5947151
    ## 5       Vermont  VT     Northeast     625741     2 0.3196211

Selecting columns with `select`
-------------------------------

Although our data table only has six columns, some data tables include hundreds. If we want to view just a few, we can use the dplyr `select` function. In the code below we select three columns, assign this to a new object and then filter the new object.

Here is the R-base approach:

``` r
new_table <- murders[, c("state", "region", "rate")]
new_table$rate[new_table$rate <= 0.71]
```

    ## [1] 0.5145920 0.6893484 0.3798036 0.5947151 0.3196211

Here is a frist attempt at the tidyverse approach, alhough soon we see an improvment.

``` r
new_table <- select(murders, state, region, rate)
filter(new_table, rate <= 0.71)
```

    ##           state        region      rate
    ## 1        Hawaii          West 0.5145920
    ## 2          Iowa North Central 0.6893484
    ## 3 New Hampshire     Northeast 0.3798036
    ## 4  North Dakota North Central 0.5947151
    ## 5       Vermont     Northeast 0.3196211

In the call to `select`, the first argument, `murders`, is an object but `state`, `region`, and `rate` are variable names.

The pipe: `%>%`
---------------

We wrote the code above because we wanted to show the three variables for states that have murder rates below 0.71. To do this we defined the intermediate object `new_table`. In `dplyr` we can write code that looks more like a description of what we want to do:

original data  →  select  →  filter 

For such an operation, we can use the pipe `%>%`. The code looks like this:

``` r
murders %>% select(state, region, rate) %>% filter(rate <= 0.71)
```

    ##           state        region      rate
    ## 1        Hawaii          West 0.5145920
    ## 2          Iowa North Central 0.6893484
    ## 3 New Hampshire     Northeast 0.3798036
    ## 4  North Dakota North Central 0.5947151
    ## 5       Vermont     Northeast 0.3196211

This line of code is equivalent to the two lines of code above. What is going on here?

In general, the pipe *sends* the result of the left side of the pipe to be the **first argument** of the function on the right side of the pipe. Here is a very simple example:

``` r
16 %>% sqrt()
```

    ## [1] 4

We can continue to pipe values along:

``` r
16 %>% sqrt() %>% log2()
```

    ## [1] 2

The parenthesis are not needed but we recommend using them to clearly show that a function is being applied.

The above statement is equivalent to

``` r
log2(sqrt(16))
```

    ## [1] 2

We find that most students find the former sytnax more readable

Remember that the pipe sends values to the first argument so we can define arguments as if the first argument is already defined:

``` r
16 %>% sqrt() %>% log(base = 2)
```

    ## [1] 2

Therefore when using the pipe with data frames and `dplyr`, we no longer need to specify the required first argument since the `dplyr` functions we have described all take the data as the first argument. In the code we wrote:

``` r
murders %>% select(state, region, rate) %>% filter(rate <= 0.71)
```

`murders` is the first argument of the `select` function and the new data frame, formerly `new_table`, is the first argument of the `filter` function.

Assessments
-----------

1.  Load the `dplyr` package and the murders dataset.

    ``` r
    library(dplyr)
    library(dslabs)
    data(murders)
    ```

    You can add columns using the `dplyr` function `mutate`. This function is aware of the column names and inside the function you can call them unquoted. Like this:

    ``` r
    murders <- mutate(murders, population_in_millions = population / 10^6)
    ```

    We can write `population` rather than `murders$population`. The function `mutate` knows we are grabing columns from `murders`.

    Use the function `mutate` to add a murders column named `rate` with the per 100,000 murder rate. Make sure you redefine `murders` as done in the example code above and remember the murder rate is defined by the total divided by the population size times 100,000.

2.  If `rank(x)` gives you the ranks of `x` from lowest to highest, `rank(-x)` gives you the ranks from highest to lowest. Use the function `mutate` to add a column `rank` containing the rank, from highest to lowest murder rate. Make sure you redefine murders.

3.  With `dplyr` we can use `select` to show only certain columns. For example, with this code we would only show the states and population sizes:

    ``` r
    select(murders, state, population) %>% head()
    ```

        ##        state population
        ## 1    Alabama    4779736
        ## 2     Alaska     710231
        ## 3    Arizona    6392017
        ## 4   Arkansas    2915918
        ## 5 California   37253956
        ## 6   Colorado    5029196

    Use `select` to show the state names and abbreviations in `murders`. Just show it, do not define a new object.

4.  The `dplyr` function `filter` is used to choose specific rows of the data frame to keep. Unlike `select` which is for columns, `filter` is for rows. For example, you can show just the New York row like this:

    ``` r
    filter(murders, state == "New York")
    ```

        ##      state abb    region population total    rate
        ## 1 New York  NY Northeast   19378102   517 2.66796

    You can use other logical vector to filter rows.

    Use `filter` to show the top 5 states with the highest murder rates. After we add murder rate and rank, do not change the murders dataset, just show the result. Remember that you can filter based on the `rank` column.

5.  We can remove rows using the `!=` operator. For example, to remove Florida we would do this:

    ``` r
    no_florida <- filter(murders, state != "Florida")
    ```

    Create a new data frame called `no_south` that removes states from the South region. How many states are in this category? You can use the function `nrow` for this.

6.  We can also use the `%in%` to filter with `dplyr`. You can thus see the data from New York and Texas like this:

    ``` r
    filter(murders, state %in% c("New York", "Texas"))
    ```

        ##      state abb    region population total    rate
        ## 1 New York  NY Northeast   19378102   517 2.66796
        ## 2    Texas  TX     South   25145561   805 3.20136

    Create a new data frame called `murders_nw` with only the states from the Northeast and the West. How many states are in this category?

7.  Suppose you want to live in the Northeast or West **and** want the murder rate to be less than 1. We want to see the data for the states satisfying these options. Note that you can use logical operators with `filter`:

    ``` r
    filter(murders, population < 5000000 & region == "Northeast")
    ```

        ##           state abb    region population total      rate
        ## 1   Connecticut  CT Northeast    3574097    97 2.7139722
        ## 2         Maine  ME Northeast    1328361    11 0.8280881
        ## 3 New Hampshire  NH Northeast    1316470     5 0.3798036
        ## 4  Rhode Island  RI Northeast    1052567    16 1.5200933
        ## 5       Vermont  VT Northeast     625741     2 0.3196211

    Add a murder rate column and a rank column as done before. Create a table, call it `my_states`, that satisfies both the conditions: it is in the Northeast or West and the murder rate is less than 1. Use select to show only the state name, the rate and the rank.

    ``` r
    library(dplyr)
    library(dslabs)
    data(murders)
    ```

8.  The pipe `%>%` can be used to perform operations sequentially without having to define intermediate objects. After redefining murder to include rate and rank.

    ``` r
    library(dplyr)
    murders <- mutate(murders, rate =  total / population * 100000, rank = rank(-rate))
    ```

    in the solution to the previous exercise we did the following:

    ``` r
    # Created a table 
    my_states <- filter(murders, region %in% c("Northeast", "West") & rate < 1)

    # Used select to show only the state name, the murder rate and the rank
    select(my_states, state, rate, rank)
    ```

        ##           state      rate rank
        ## 1        Hawaii 0.5145920   49
        ## 2         Idaho 0.7655102   46
        ## 3         Maine 0.8280881   44
        ## 4 New Hampshire 0.3798036   50
        ## 5        Oregon 0.9396843   42
        ## 6          Utah 0.7959810   45
        ## 7       Vermont 0.3196211   51
        ## 8       Wyoming 0.8871131   43

    The pipe `%>%` permits us to perform both operation sequentially and without having to define an intermediate variable `my_states`. We therefore could have mutated and selected in the same line like this:

    ``` r
    mutate(murders, rate =  total / population * 100000, rank = rank(-rate)) %>%
      select(state, rate, rank)
    ```

        ##                   state       rate rank
        ## 1               Alabama  2.8244238   23
        ## 2                Alaska  2.6751860   27
        ## 3               Arizona  3.6295273   10
        ## 4              Arkansas  3.1893901   17
        ## 5            California  3.3741383   14
        ## 6              Colorado  1.2924531   38
        ## 7           Connecticut  2.7139722   25
        ## 8              Delaware  4.2319369    6
        ## 9  District of Columbia 16.4527532    1
        ## 10              Florida  3.3980688   13
        ## 11              Georgia  3.7903226    9
        ## 12               Hawaii  0.5145920   49
        ## 13                Idaho  0.7655102   46
        ## 14             Illinois  2.8369608   22
        ## 15              Indiana  2.1900730   31
        ## 16                 Iowa  0.6893484   47
        ## 17               Kansas  2.2081106   30
        ## 18             Kentucky  2.6732010   28
        ## 19            Louisiana  7.7425810    2
        ## 20                Maine  0.8280881   44
        ## 21             Maryland  5.0748655    4
        ## 22        Massachusetts  1.8021791   32
        ## 23             Michigan  4.1786225    7
        ## 24            Minnesota  0.9992600   40
        ## 25          Mississippi  4.0440846    8
        ## 26             Missouri  5.3598917    3
        ## 27              Montana  1.2128379   39
        ## 28             Nebraska  1.7521372   33
        ## 29               Nevada  3.1104763   19
        ## 30        New Hampshire  0.3798036   50
        ## 31           New Jersey  2.7980319   24
        ## 32           New Mexico  3.2537239   15
        ## 33             New York  2.6679599   29
        ## 34       North Carolina  2.9993237   20
        ## 35         North Dakota  0.5947151   48
        ## 36                 Ohio  2.6871225   26
        ## 37             Oklahoma  2.9589340   21
        ## 38               Oregon  0.9396843   42
        ## 39         Pennsylvania  3.5977513   11
        ## 40         Rhode Island  1.5200933   35
        ## 41       South Carolina  4.4753235    5
        ## 42         South Dakota  0.9825837   41
        ## 43            Tennessee  3.4509357   12
        ## 44                Texas  3.2013603   16
        ## 45                 Utah  0.7959810   45
        ## 46              Vermont  0.3196211   51
        ## 47             Virginia  3.1246001   18
        ## 48           Washington  1.3829942   37
        ## 49        West Virginia  1.4571013   36
        ## 50            Wisconsin  1.7056487   34
        ## 51              Wyoming  0.8871131   43

    Notice that `select` no longer has a data frame as the first argument. The first argument is assumed to be the result of the operation conducted right before the `%>%`

    Repeat the previous exercise, but now instead of creating a new object, show the result and only include the state, rate, and rank columns. Use a pipe `%>%` to do this in just one line.

9.  Now we will make murders the original table one gets when loading using `data(murders)`. Use just one line to create a new data frame, called `my_states`, that has a murder rate and a rank column, considers only states in the Northeast or West, which have a murder rate lower than 1, and contains only the state, rate, and rank columns. The line should have four components separated by three `%>%`.

    -   The original dataset `murders`.
    -   A call to `mutate` to add the murder rate and the rank.
    -   A call to `filter` to keep only the states from the Northeast or West and that have a murder rate below 1.
    -   A call to `select` that keeps only the columns with the state name, the murder rate and the rank.

    The line should look something like this `my_states <- murders %>%` mutate something `%>%` filter something `%>%` select something.

Summarizing data with dplyr
===========================

An common operation in data analysis is to stratify data and then summarize. Here, we cover two new dplyr verbs that make these computations easier: `summarize` and `group_by`. We learn to access resulting values using what we call the *dot placeholder*. Finally, we also learn to use `arrange`, which helps us examine data after sorting.

Summarize
---------

The `summarize` function in dplyr provides a way to compute summary statistics with intuitive and readable code.

The function `summarize` permits us to compute as many summaries of the data as we want. For example, if we wanted to compute the average and standard deviation for the murder rate we simply do as follows:

``` r
s <- murders %>% summarize(average = mean(rate), standard_deviation = sd(rate))
s
```

    ##    average standard_deviation
    ## 1 2.779125           2.456118

This takes our original data table as input then averages and the standard deviation of rates. We get to choose the names of the columns of the resulting table. For example, above we decided to use `average` and `standard_deviation`, but we could have used other names just the same.

Note the consistency, we start with a data frame and end with a data frame

``` r
class(murders)
```

    ## [1] "data.frame"

``` r
class(s)
```

    ## [1] "data.frame"

Because the resulting table, stored in `s`, is a data frame, we can access the components with the accessor `$`, which in this case will be a numeric:

``` r
s$average
```

    ## [1] 2.779125

``` r
s$standard_deviation
```

    ## [1] 2.456118

As with most other dplyr functions, `summarize` is aware of the variable names and we can use them directly. So when inside the call to the `summarize` function we write `mean(rate)`, it is accessing the column with the name, and then computing the average of the respective numeric vector. We can compute any other summary that operates on vectors and returns a single value. For example, we can add the median, min and max like this:

``` r
murders %>%
  summarize(median = median(rate), minimum = min(rate), maximum = max(rate))
```

    ##     median   minimum  maximum
    ## 1 2.687123 0.3196211 16.45275

We can obtain these three values with just one line using the `quantiles` function; e.g. `quantile(x, c(0,0.5,1))` returns the min, median, and max of the vector `x`. However, if we attempt to use a function that returns two or more values:

``` r
murders %>%
  summarize(range = quantile(height, c(0, 0.5, 1)))
```

we will receive an error: `Error: expecting result of length one, got : 2`. With the function `summarize`, we can only call functions that return a single value. To perform this type of operation we need more advanced tidyverse tool whihc we may learn later.

For another example of how we can use the `summarize` function, let's compute the average murder rate for the United States. Remember our data table includes total murders and population size for each state and we have already used dplyr to add a murder rate column

Remember that the US murder is **not** the average of the state murder rates:

``` r
murders %>% summarize(mean(rate))
```

    ##   mean(rate)
    ## 1   2.779125

This is because in the computation above the small states are given the same weight as the large ones. The US murder rate is the total US murders divided by the total US population. So the correct computation is:

``` r
us_murder_rate <- murders %>% 
  summarize(rate = sum(total) / sum(population) * 100000)
us_murder_rate
```

    ##       rate
    ## 1 3.034555

This computation counts larger states proportionally to their size which results in a larger value.

The dot operator
----------------

The `us_murder_rate` object defined above represents just one number. Yet we are storing it in a data frame:

``` r
class(us_murder_rate)
```

    ## [1] "data.frame"

since, as most dplyr functions, `summarize` always returns a data frame.

This might be problematic if we want to use the result with functions that require a numeric value. Here we show a useful trick for accessing values stored in data piped via `%>%`: when a data object is piped it can be accessed using the dot `.`. To understand what we mean take a look at this line of code:

``` r
us_murder_rate %>% .$rate 
```

    ## [1] 3.034555

This returns the value in the `rate` column of `us_murder_rate` making it equivalent to `us_murder_rate$rate`. To understand this line, you just need to think of `.` as a placeholder for the data that is being passed through the pipe. Because this data object is a data frame, we can access its columns with the `$`.

To get a number from the original data table with one line of code we can type:

``` r
us_murder_rate <- murders %>% 
  summarize(rate = sum(total) / sum(population) * 100000) %>%
  .$rate

us_murder_rate
```

    ## [1] 3.034555

which is now a numeric:

``` r
class(us_murder_rate)
```

    ## [1] "numeric"

We eventually see other instances in which using the `.` is useful. For now, we will only use it to produce numeric vectors from pipelines constructed with dplyr.

Group then summarize
--------------------

A common operation in data exploration is to first split data into groups and then compute summaries for each group. For example, we may want to compute the median and IQR for each region of the country. The `group_by` function helps us do this.

If we type this:

``` r
murders %>% group_by(region)
```

    ## # A tibble: 51 x 7
    ## # Groups:   region [4]
    ##    state                abb   region    population total  rate  rank
    ##    <chr>                <chr> <fct>          <dbl> <dbl> <dbl> <dbl>
    ##  1 Alabama              AL    South        4779736   135  2.82    23
    ##  2 Alaska               AK    West          710231    19  2.68    27
    ##  3 Arizona              AZ    West         6392017   232  3.63    10
    ##  4 Arkansas             AR    South        2915918    93  3.19    17
    ##  5 California           CA    West        37253956  1257  3.37    14
    ##  6 Colorado             CO    West         5029196    65  1.29    38
    ##  7 Connecticut          CT    Northeast    3574097    97  2.71    25
    ##  8 Delaware             DE    South         897934    38  4.23     6
    ##  9 District of Columbia DC    South         601723    99 16.5      1
    ## 10 Florida              FL    South       19687653   669  3.40    13
    ## # ... with 41 more rows

the result does not look very different from `murders`, except we see this `Groups: region [4]` when we print the object. Although not immediately obvious from its appearance, this is now a special data frame called a *grouped data frame* and dplyr functions, in particular `summarize`, will behave differently when acting on this object. Conceptually, you can think of this table as many tables, with the same columns but not necessarily the same number of rows, stacked together in one object. When we summarize the data after grouping, this is what happens:

``` r
murders %>% 
  group_by(region) %>%
  summarize(median = median(rate), iqr = IQR(rate))
```

    ## # A tibble: 4 x 3
    ##   region        median   iqr
    ##   <fct>          <dbl> <dbl>
    ## 1 Northeast       1.80  1.89
    ## 2 South           3.40  1.23
    ## 3 North Central   1.97  1.73
    ## 4 West            1.29  2.22

The `summarize` function applies the summarization to each group separately.

Sorting data frames
-------------------

When examining a dataset, it is often convenient to sort the table by the different columns. We know about the `order` and `sort` function, but for ordering entire tables, the dplyr function `arrange` is useful. For example, here we order the states by population size when we type:

``` r
murders %>% arrange(population) %>% head()
```

    ##                  state abb        region population total       rate rank
    ## 1              Wyoming  WY          West     563626     5  0.8871131   43
    ## 2 District of Columbia  DC         South     601723    99 16.4527532    1
    ## 3              Vermont  VT     Northeast     625741     2  0.3196211   51
    ## 4         North Dakota  ND North Central     672591     4  0.5947151   48
    ## 5               Alaska  AK          West     710231    19  2.6751860   27
    ## 6         South Dakota  SD North Central     814180     8  0.9825837   41

We get to decide which column to sort by. To see the states by population, from smallest to largest, we arrange by `rate` instead:

``` r
murders %>% 
  arrange(rate) %>% 
  head()
```

    ##           state abb        region population total      rate rank
    ## 1       Vermont  VT     Northeast     625741     2 0.3196211   51
    ## 2 New Hampshire  NH     Northeast    1316470     5 0.3798036   50
    ## 3        Hawaii  HI          West    1360301     7 0.5145920   49
    ## 4  North Dakota  ND North Central     672591     4 0.5947151   48
    ## 5          Iowa  IA North Central    3046355    21 0.6893484   47
    ## 6         Idaho  ID          West    1567582    12 0.7655102   46

Note that the default behavior is to order in ascending order. In dplyr, the function `desc` transforms a vector so that it is in descending order. To sort the table in descending order we can type:

``` r
murders %>% 
  arrange(desc(rate)) %>% 
  head()
```

    ##                  state abb        region population total      rate rank
    ## 1 District of Columbia  DC         South     601723    99 16.452753    1
    ## 2            Louisiana  LA         South    4533372   351  7.742581    2
    ## 3             Missouri  MO North Central    5988927   321  5.359892    3
    ## 4             Maryland  MD         South    5773552   293  5.074866    4
    ## 5       South Carolina  SC         South    4625364   207  4.475323    5
    ## 6             Delaware  DE         South     897934    38  4.231937    6

### Nested sorting

If we are ordering by a column with ties, we can use a second column to break the tie. Similarly, a third column can be used to break ties between first and second and so on. Here we order by `region` then, within region, we order by murder rate:

``` r
murders %>% 
  arrange(region, rate) %>% 
  head()
```

    ##           state abb    region population total      rate rank
    ## 1       Vermont  VT Northeast     625741     2 0.3196211   51
    ## 2 New Hampshire  NH Northeast    1316470     5 0.3798036   50
    ## 3         Maine  ME Northeast    1328361    11 0.8280881   44
    ## 4  Rhode Island  RI Northeast    1052567    16 1.5200933   35
    ## 5 Massachusetts  MA Northeast    6547629   118 1.8021791   32
    ## 6      New York  NY Northeast   19378102   517 2.6679599   29

### The top *n*

In the code above, we have used the function `head` to avoid having the page fill up with the entire dataset. If we want to see a larger proportion, we can use the `top_n` function. Here are the first 10 rows:

``` r
murders %>% top_n(10, rate)
```

    ##                   state abb        region population total      rate rank
    ## 1               Arizona  AZ          West    6392017   232  3.629527   10
    ## 2              Delaware  DE         South     897934    38  4.231937    6
    ## 3  District of Columbia  DC         South     601723    99 16.452753    1
    ## 4               Georgia  GA         South    9920000   376  3.790323    9
    ## 5             Louisiana  LA         South    4533372   351  7.742581    2
    ## 6              Maryland  MD         South    5773552   293  5.074866    4
    ## 7              Michigan  MI North Central    9883640   413  4.178622    7
    ## 8           Mississippi  MS         South    2967297   120  4.044085    8
    ## 9              Missouri  MO North Central    5988927   321  5.359892    3
    ## 10       South Carolina  SC         South    4625364   207  4.475323    5

`top_n` picks the highest `n` based on the column given as a second argument. However, the rows are not sorted.

If the second argument is left blank, then it returns the first `n` columns. This means that to see the 10 states with the highest murder rates we can type:

``` r
murders %>% 
  arrange(desc(rate)) %>%
  top_n(10)
```

    ## Selecting by rank

    ##            state abb        region population total      rate rank
    ## 1         Oregon  OR          West    3831074    36 0.9396843   42
    ## 2        Wyoming  WY          West     563626     5 0.8871131   43
    ## 3          Maine  ME     Northeast    1328361    11 0.8280881   44
    ## 4           Utah  UT          West    2763885    22 0.7959810   45
    ## 5          Idaho  ID          West    1567582    12 0.7655102   46
    ## 6           Iowa  IA North Central    3046355    21 0.6893484   47
    ## 7   North Dakota  ND North Central     672591     4 0.5947151   48
    ## 8         Hawaii  HI          West    1360301     7 0.5145920   49
    ## 9  New Hampshire  NH     Northeast    1316470     5 0.3798036   50
    ## 10       Vermont  VT     Northeast     625741     2 0.3196211   51

Assessment
----------

For these exercises we will be using the data from the survey collected by the United States National Center for Health Statistics (NCHS). This center has conducted a series of health and nutrition surveys since the 1960’s. Starting in 1999 about 5,000 individuals of all ages have been interviewed every year and they complete the health examination component of the survey. Part of the data is made available via the NHANES package which can install using:

``` r
install.packages("NHANES")
```

Once you install it you can load the data this way:

``` r
library(NHANES)
data(NHANES)
```

The NHANES data has many missing values. Remember that the main summarization function in R will return `NA` if any of the entries of the input vector is an `NA`. Here is an example:

``` r
library(dslabs)
data(na_example)
mean(na_example)
```

    ## [1] NA

``` r
sd(na_example)
```

    ## [1] NA

To ignore the `NA`s we can use the `na.rm` argument:

``` r
mean(na_example, na.rm=TRUE)
```

    ## [1] 2.301754

``` r
sd(na_example, na.rm=TRUE)
```

    ## [1] 1.22338

Let's now explore the NHANES data.

1.  We will provide some basic facts about blood pressure. First let's select a group to set the standard. We will use 20-29 year old females. Note that the category is coded with `20-29`, with a space in front! The `AgeDecade` is a categorical variable with these ages. What is the average and standard deviation of systolic blood pressure, as saved in the `BPSysAve` variable? Save it to a variable called `ref`. Hint: Use `filter` and `summarize` and use the `na.rm=TRUE` argument when computing the average and standard deviation. You can also filter the NA values using `filter`.

2.  Using only one line of code, assign the average to a numeric variable `ref_avg`. Hint: Use the code similar to above and then the dot.

3.  Now report the min and max values for the same group.

4.  Compute the average and standard deviation for females, but for each age group separately. Note that the age groups are defined by `AgeDecade`. Hint: rather than filtering by age, filter by `Gender` and then use `group_by`.

5.  Now do the same for males.

6.  We can actually combine both these summaries into one line of code. This is because `group_by` permits us to group by more than one variable. Obtain one big summary table using `group_by(AgeDecade, Gender)`.

7.  For males between the ages of 40-49, compare systolic blood pressure across race as reported in the `Race1` variable. Order the resulting table from lowest to highest average systolic blood pressure.
