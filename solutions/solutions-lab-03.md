Part 3: Data wrangling
================

We will be using the `tidyverse` in this lab. So let's start by loading it.

``` r
library(tidyverse)
```

Life expectancy and fertility
-----------------------------

Read in the `life-expectancy-and-fertility-two-countries-example.csv` following file included in the **dslabs** package. You find the full path to the file like this:

``` r
path <- system.file("extdata", package = "dslabs")
filename <- file.path(path, "life-expectancy-and-fertility-two-countries-example.csv")
```

Now read in the data and save it in an object called `raw_data`.

``` r
raw_dat <- read_csv(filename)
```

Examine the first 10 columns of `raw_dat`.

``` r
raw_dat %>% select(1:5) %>% View()
```

Notice that five separate variables are represented here: country, year, fertility and life expectancy. However, the data is not tidy. A tidy dataset should have five columns, one for each variable.

Because two different variable are represented in the column names it will be impossible to wrangle this dataset with just on call to `gather`. So instead, we will form a temporary table with country, a column with column names, and a column with the corresponding values. Use `gather` to create this table, call it `dat` and call the two new columns `key` and `value`.

``` r
dat <- raw_dat %>% gather(key, value, -country)
```

Now we see that two of our values are stored in the key column: year and the variable name.

``` r
dat$key[1:5]
```

    ## [1] "1960_fertility"       "1960_fertility"       "1960_life_expectancy"
    ## [4] "1960_life_expectancy" "1961_fertility"

We need to separate the year from the variable name. We can do this with the `separate` function like this:

``` r
dat %>% separate(key, c("year", "variable"), sep = "_")
```

Because `_` is the default separator we can can write

``` r
dat %>% separate(key, c("year", "variable"))
```

    ## # A tibble: 224 x 4
    ##    country     year  variable  value
    ##    <chr>       <chr> <chr>     <dbl>
    ##  1 Germany     1960  fertility  2.41
    ##  2 South Korea 1960  fertility  6.16
    ##  3 Germany     1960  life      69.3 
    ##  4 South Korea 1960  life      53.0 
    ##  5 Germany     1961  fertility  2.44
    ##  6 South Korea 1961  fertility  5.99
    ##  7 Germany     1961  life      69.8 
    ##  8 South Korea 1961  life      53.8 
    ##  9 Germany     1962  fertility  2.47
    ## 10 South Korea 1962  fertility  5.79
    ## # ... with 214 more rows

However, there is a problem with this call because `life_expectancy` is divided into two. We can get around this by using the `extra` argument:

``` r
dat %>% separate(key, c("year", "variable"), extra = "merge")
```

    ## # A tibble: 224 x 4
    ##    country     year  variable        value
    ##    <chr>       <chr> <chr>           <dbl>
    ##  1 Germany     1960  fertility        2.41
    ##  2 South Korea 1960  fertility        6.16
    ##  3 Germany     1960  life_expectancy 69.3 
    ##  4 South Korea 1960  life_expectancy 53.0 
    ##  5 Germany     1961  fertility        2.44
    ##  6 South Korea 1961  fertility        5.99
    ##  7 Germany     1961  life_expectancy 69.8 
    ##  8 South Korea 1961  life_expectancy 53.8 
    ##  9 Germany     1962  fertility        2.47
    ## 10 South Korea 1962  fertility        5.79
    ## # ... with 214 more rows

However, we are not done because we want to have life expectancy and fertility in two different columns. Use the `spread` function to achieve this.

``` r
dat %>% 
  separate(key, c("year", "variable"), extra = "merge") %>%
  spread(variable, value) 
```

    ## # A tibble: 112 x 4
    ##    country year  fertility life_expectancy
    ##    <chr>   <chr>     <dbl>           <dbl>
    ##  1 Germany 1960       2.41            69.3
    ##  2 Germany 1961       2.44            69.8
    ##  3 Germany 1962       2.47            70.0
    ##  4 Germany 1963       2.49            70.1
    ##  5 Germany 1964       2.49            70.7
    ##  6 Germany 1965       2.48            70.6
    ##  7 Germany 1966       2.44            70.8
    ##  8 Germany 1967       2.37            71.0
    ##  9 Germany 1968       2.28            70.6
    ## 10 Germany 1969       2.17            70.5
    ## # ... with 102 more rows

There is one remaining problem. The year variable is a character and should be numeric. Change the year variable to a numeric and then order the table by year. Call the final table `dat`.

``` r
dat %>% 
  separate(key, c("year", "variable"), extra = "merge") %>%
  spread(variable, value) %>%
  mutate(year = as.numeric(year)) %>%
  arrange(year)
```

    ## # A tibble: 112 x 4
    ##    country      year fertility life_expectancy
    ##    <chr>       <dbl>     <dbl>           <dbl>
    ##  1 Germany      1960      2.41            69.3
    ##  2 South Korea  1960      6.16            53.0
    ##  3 Germany      1961      2.44            69.8
    ##  4 South Korea  1961      5.99            53.8
    ##  5 Germany      1962      2.47            70.0
    ##  6 South Korea  1962      5.79            54.5
    ##  7 Germany      1963      2.49            70.1
    ##  8 South Korea  1963      5.57            55.3
    ##  9 Germany      1964      2.49            70.7
    ## 10 South Korea  1964      5.36            56.0
    ## # ... with 102 more rows

The mouse data (advanced)
-------------------------

In the two previous labs we started with an rda file containing a tidy table. But we are rarely this fortunate. Data often comes formats that are quite far from tidy. The data in that table can be obtained from an excel file which you can find in the data directory:

``` r
list.files("../data")
```

    ## [1] "mouse-raw-data.xlsx"

If you have a copy of Microsoft Excel installed on your computer you can inspect the file directly. You will see that the file has two sheets. The first sheet contains the blood pressure measurements and the second sheet contains the weight measurements and other data. In R you can read them in with the `read_excel` function in the **readxl** package:

``` r
library(readxl)
raw_bp     <- read_excel("../data/mouse-raw-data.xlsx", sheet = 1)
raw_weight <- read_excel("../data/mouse-raw-data.xlsx", sheet = 2) 
```

Examine each of these tables so that you are aware what is in the rows and what is in the columns. Hint: Use the `View` function.

``` r
raw_weight %>% View()
raw_bp %>% View()
```

We have received a jagged array rather than a table.

Let's wrangle this dataset into a tidy form.

The general strategy will be to extract four separate tables and then join them together using `rbind` or the equivalent **dplyr** function `bind_rows`.

You can get the DNA names that define the four tables like this:

``` r
DNA <- names(raw_weight) %>% str_remove("\\.\\D+") %>% unique()
```

The `\\.\\D+` is a regex pattern representing a period followed by anything that is not number.

Now let's extract the part of the table that contains the first of these tables, defined by `152F7`. You can use the `contains` helper function in call to `select` like this:

``` r
s <- DNA[2]
raw_weight %>% select(contains(s))
```

    ## # A tibble: 177 x 6
    ##    `152F7.line` `152F7.tg` `152F7.sex` `152F7.age` `152F7.weight`
    ##    <chr>             <dbl>       <dbl>       <dbl>          <dbl>
    ##  1 #12-14-1              1           1         170           31.1
    ##  2 #12-14-2              1           1         170           33.4
    ##  3 #12-14-3              1           1         170           31.4
    ##  4 #12-14-4              0           1         170           33.2
    ##  5 #12-14-5              1           1         170           30.3
    ##  6 #12-14-6              1           1         170           32.4
    ##  7 #12-14-17             0           1         182           41.3
    ##  8 #12-14-18             0           1         182           33  
    ##  9 #12-14-19             0           1         182           31  
    ## 10 #12-14-20             0           1         182           29.1
    ## # ... with 167 more rows, and 1 more variable: `152F7.cage` <dbl>

To remove the rows with NAs with can use the `drop_na` function from the **tidyr** package.

``` r
weight <- raw_weight %>% 
  select(contains("152F7")) %>% 
  drop_na()
```

Now add a `DNA` column with the DNA fragment name.

``` r
weight <- raw_weight %>% select(contains(s)) %>% 
  drop_na() %>% 
  mutate(DNA = s)
```

We are close to having one of the tables that we want to bind. A remaining problem is that the variable names all contain `152F7.` and we want to remove that. We can achieve this using

``` r
weight <- raw_weight %>% select(contains(s)) %>% 
  setNames(str_remove(names(.), ".*\\."))
```

The regex `.*\\.` means anything following by a dot.

Now you should have line of code that for any given DNA fragment name `s` can create the appropriate table. Use the `lapply` function to do this for each of the fragment names and store in a list called `tmp`

``` r
tmp <- lapply(DNA, function(s){
  select(raw_weight, contains(s)) %>%
    drop_na() %>%
    setNames(str_remove(names(.), ".*\\.")) %>%
    mutate(DNA = s)
})
```

Finally, use the function `do.call` to apply the `bind_rows` function to the elements of the list to form a final table. Call the table `weight`

``` r
weight <- do.call(bind_rows, tmp)
```

So now we have the weight data. The last step is to add the blood pressure data. We can use `inner_join` to do this. However the mouse IDs are spread across 3 different columns:

``` r
raw_bp %>% head()
```

    ## # A tibble: 6 x 4
    ##   line.1 line.2 line.3    bp
    ##    <dbl>  <dbl> <chr>  <dbl>
    ## 1     12     14 1       118.
    ## 2     12     14 2       123.
    ## 3     12     14 3       116.
    ## 4     12     14 4       125.
    ## 5     12     14 5       119.
    ## 6     12     14 6       122.

You can use the `unite` function to do this. Like this:

``` r
bp <- raw_bp %>% unite(line, contains("line"), sep="-")
```

Note that we are not finished. We need to add a `#` to the IDs. Do this using `mutate` and the `paste0` function:

``` r
bp <- raw_bp %>% unite(line, contains("line"), sep="-") %>%
  mutate(line = paste0("#", line))
```

Now use `inner_join` to create the final table. Call it `new_dat`

``` r
new_dat <- inner_join(weight, bp, by = "line")
```

Now let's compare it to our tidy data

``` r
load("../rdas/mouse.rda")
dat <- mutate(dat, DNA = recode(DNA, 
                                "1"="141G6", 
                                "2"="152F7", 
                                "3"="230E8", 
                                "4"="285E6"))
```

Use the `set_diff` function to see if the tables match.

``` r
setdiff(new_dat, dat)
```

    ## # A tibble: 0 x 8
    ## # ... with 8 variables: DNA <chr>, line <chr>, tg <dbl>, sex <dbl>,
    ## #   age <dbl>, weight <dbl>, bp <dbl>, cage <dbl>
