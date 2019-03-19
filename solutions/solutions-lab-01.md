Identifying DNA fragment that increases part 1: the tidyverse
================

Manipulating data tables
------------------------

We will be using the tidyverse in this lab. So let's start by loading it.

``` r
library(tidyverse)
```

Your lab conducted an experiment in which four different fragments of chromosome 21 were integrated into mice. The four parts are denoted with *141G6*, *152F7*, *230E8* and *285E6*. The mice were bred resulting in dozens of transgenic mice. The DNA fragment is not always inherited so some mice have the extra copy and others don't. We are interested in determining if any of these fragments result in an increase in weight, a characteristic associated with trisomic mice.

The data can be loaded like this.

``` r
load("../rdas/mouse.rda")
```

Which loads the `dat` object into your environment:

``` r
dat
```

    ## # A tibble: 537 x 8
    ##      DNA line         tg   sex   age weight    bp  cage
    ##    <dbl> <chr>     <dbl> <dbl> <dbl>  <dbl> <dbl> <dbl>
    ##  1     3 #50-69-1      1     1   113   31.6  123.     1
    ##  2     3 #50-69-2      1     1   113   31.2  125.     1
    ##  3     3 #50-69-3      1     1   113   28.6  122      1
    ##  4     3 #50-69-4      0     1   113   30.1  126      1
    ##  5     3 #50-69-11     0     1   121   31.3  129.     2
    ##  6     3 #50-69-12     0     1   121   36.4  126.     2
    ##  7     3 #50-69-13     1     1   121   36.5  126.     2
    ##  8     3 #50-69-15     1     1   121   29.8  124.     2
    ##  9     3 #50-69-16     1     1   121   35.6  127      2
    ## 10     3 #50-69-17     1     1   121   33.5  123.     2
    ## # ... with 527 more rows

The columns included in this table are the following:

-   *DNA*: Fragment of chromosome 21 integrated in parent mouse (1=141G6; 2=152F7; 3=230E8; 4=285E6).
-   *line*: Family line.
-   *tg* - Whether the mouse contains the extra DNA (1) or not (0).
-   *sex*: Sex of mouse (1=male; 0=female).
-   *age*: Age of mouse (in days) at time of weighing.
-   *weight*: Weight of mouse in grams, to the nearest tenth of a gram.
-   *bp*: Blood pressure of the mouse.
-   *cage*: Number of the cage in which the mouse lived

Let's start by comparing the weights of the no trisomic mice to the weights of mice with the other four fragments. Determine which columns tells us the fragment of the mouse and determine the number of mouse in each group? Hint: use the *count* function.

``` r
dat %>% count(DNA)
```

    ## # A tibble: 4 x 2
    ##     DNA     n
    ##   <dbl> <int>
    ## 1     1   182
    ## 2     2   158
    ## 3     3    37
    ## 4     4   160

Note that the names are 1, 2, 3, 4. Let's change these to the actual names 1=141G6; 2=152F7; 3=230E8; 4=285E6. Create a new column called `fragment` with the actual fragment names. Hint: Use the `recode` function.

``` r
dat <- mutate(dat, fragment = recode(DNA, 
                                "1"="141G6", 
                                "2"="152F7", 
                                "3"="230E8", 
                                "4"="285E6"))
```

Note that all the mice in our table have one of these names. However, we know that not all mice have the fragments. Remember that not all inherited the extra copy. Use `filter` and `count` to see how many mice in the `141G6` group have the extra copy.

``` r
filter(dat, fragment == "141G6") %>% count(tg)
```

    ## # A tibble: 2 x 2
    ##      tg     n
    ##   <dbl> <int>
    ## 1     0    78
    ## 2     1   104

Now change the `fragment` column so that the mice that do not have the extra copy, have are called `No trisomy`. Hint: use the `ifelse` function.

``` r
dat <- dat %>% mutate(fragment = ifelse(tg == 0, "No trisomy", fragment)) 
```

Before we continue let's learn about the `n()` function. Note that we can perform the same as the `count()` function using `group_by()` and `n()`

``` r
dat %>% group_by(DNA) %>% summarize(freq = n())
```

    ## # A tibble: 4 x 2
    ##     DNA  freq
    ##   <dbl> <int>
    ## 1     1   182
    ## 2     2   158
    ## 3     3    37
    ## 4     4   160

Now compute the average and standard error in each of the four groups and the control. Hint: Use `group_by` and `summarize`.

``` r
dat %>% group_by(fragment) %>% 
  summarize(average = mean(weight), se = sd(weight)/sqrt(n()))
```

    ## # A tibble: 5 x 3
    ##   fragment   average    se
    ##   <chr>        <dbl> <dbl>
    ## 1 141G6         29.4 0.423
    ## 2 152F7         31.0 0.401
    ## 3 230E8         30.4 0.833
    ## 4 285E6         28.1 0.342
    ## 5 No trisomy    47.0 8.26

Bonus: Is the above difference statistically significant at the 0.05 level?

``` r
lm(weight ~ tg, data = dat) %>% summary() %>% .$coef
```

    ##              Estimate Std. Error   t value     Pr(>|t|)
    ## (Intercept)  46.96169   5.757313  8.156875 2.464957e-15
    ## tg          -17.47074   8.030682 -2.175499 3.002977e-02
