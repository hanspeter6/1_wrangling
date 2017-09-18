Lesson 1 // Data wrangling
================
My practice

In this lesson we'll:

1.  see how to save .csv data in the form of an .RData object
2.  introduce data transformations using the **dplyr** package, using the five key dplyr verbs:

-   filter
-   arrange
-   mutate
-   summarise
-   grouped\_by

1.  introduce the pipe operator `%>%`
2.  see a few nice things you can do by combining dplyr verbs (grouped filters and mutates, for example)
3.  use the dplyr verbs to build a small movie rating dataset that we'll use in the next lesson on recommender systems.
4.  introduce various *join* operations that can be used to combine information across multiple tables (relational data)

### Sources and references

-   <http://r4ds.had.co.nz/transform.html>
-   <http://r4ds.had.co.nz/relational-data.html>

Get the MovieLens data and save as .RData
-----------------------------------------

[MovieLens](https://grouplens.org/datasets/movielens/) is a great resource for data on movie ratings. The full dataset has ratings on 40 000 movies by 260 000 users, some 24 million ratings in all. We'll use a smaller dataset with ratings of 9 000 movies by 700 users (100 000 ratings in all).

Download the file "ml-latest-small.zip" from <https://grouplens.org/datasets/movielens/> and unzip it to the *data* directory in your main project folder (make a folder called *data* if you haven't already). You should see four csv files: `links.csv`, `movies.csv`, `ratings.csv`, and `tags.csv`.

First let's save the data we downloaded as an .RData object. .RData objects are smaller than csv, plus we can save all four csvs in a single .RData object that we can call with a single call to `load` the dataset later on.

``` r
# read in the csv files
links <- read.csv("data/links.csv")
movies <- read.csv("data/movies.csv")
ratings <- read.csv("data/ratings.csv")
tags <- read.csv("data/tags.csv")

# save as .RData
save(links,movies,ratings,tags,file="data/movielens-small.RData")

# check that its worked
rm(list=ls())
load("data/movielens-small.RData")
```

You'll only need to do the above part once so, once you've got the data saved as .RData, start running the notebook from here.

Loading the tidyverse
---------------------

Load the **tidyverse** collection of packages, which loads the following packages: **ggplot2**, **tibble**, **tidyr**, **readr**, **purrr**, and **dplyr**.

``` r
library(tidyverse)
```

    ## Loading tidyverse: ggplot2
    ## Loading tidyverse: tibble
    ## Loading tidyverse: tidyr
    ## Loading tidyverse: readr
    ## Loading tidyverse: purrr
    ## Loading tidyverse: dplyr

    ## Conflicts with tidy packages ----------------------------------------------

    ## filter(): dplyr, stats
    ## lag():    dplyr, stats

Load the MovieLens data

``` r
load("data/movielens-small.RData")
```

Tibbles are a special kind of dataframe that work well with tidyverse packages ("in the tidyverse" in tidyversese).

``` r
# convert ratings to a "tibble"
ratings <- as.tibble(ratings)
```

A nice feature of tibbles is that if you display them in the console (by typing `ratings`, for example) only first few rows and columns are shown. Unfortunately this doesn't carry over to jupyter notebook, so I need to explicitly say `print(ratings)` or all the rows are shown.

``` r
print(ratings)
```

    ## # A tibble: 100,004 x 4
    ##    userId movieId rating  timestamp
    ##     <int>   <int>  <dbl>      <int>
    ##  1      1      31    2.5 1260759144
    ##  2      1    1029    3.0 1260759179
    ##  3      1    1061    3.0 1260759182
    ##  4      1    1129    2.0 1260759185
    ##  5      1    1172    4.0 1260759205
    ##  6      1    1263    2.0 1260759151
    ##  7      1    1287    2.0 1260759187
    ##  8      1    1293    2.0 1260759148
    ##  9      1    1339    3.5 1260759125
    ## 10      1    1343    2.0 1260759131
    ## # ... with 99,994 more rows

Explore some of the variables

``` r
str(ratings)
```

    ## Classes 'tbl_df', 'tbl' and 'data.frame':    100004 obs. of  4 variables:
    ##  $ userId   : int  1 1 1 1 1 1 1 1 1 1 ...
    ##  $ movieId  : int  31 1029 1061 1129 1172 1263 1287 1293 1339 1343 ...
    ##  $ rating   : num  2.5 3 3 2 4 2 2 2 3.5 2 ...
    ##  $ timestamp: int  1260759144 1260759179 1260759182 1260759185 1260759205 1260759151 1260759187 1260759148 1260759125 1260759131 ...

``` r
glimpse(ratings)
```

    ## Observations: 100,004
    ## Variables: 4
    ## $ userId    <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1...
    ## $ movieId   <int> 31, 1029, 1061, 1129, 1172, 1263, 1287, 1293, 1339, ...
    ## $ rating    <dbl> 2.5, 3.0, 3.0, 2.0, 4.0, 2.0, 2.0, 2.0, 3.5, 2.0, 2....
    ## $ timestamp <int> 1260759144, 1260759179, 1260759182, 1260759185, 1260...

``` r
glimpse(movies)
```

    ## Observations: 9,125
    ## Variables: 3
    ## $ movieId <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16,...
    ## $ title   <fctr> Toy Story (1995), Jumanji (1995), Grumpier Old Men (1...
    ## $ genres  <fctr> Adventure|Animation|Children|Comedy|Fantasy, Adventur...

We'll look at database joins in more detail, but for now, this just adds movie title to the `ratings` data by pulling that information from `movies`.

``` r
ratings <- left_join(ratings, movies)
```

    ## Joining, by = "movieId"

checking it out

``` r
ratings
```

    ## # A tibble: 100,004 x 6
    ##    userId movieId rating  timestamp
    ##     <int>   <int>  <dbl>      <int>
    ##  1      1      31    2.5 1260759144
    ##  2      1    1029    3.0 1260759179
    ##  3      1    1061    3.0 1260759182
    ##  4      1    1129    2.0 1260759185
    ##  5      1    1172    4.0 1260759205
    ##  6      1    1263    2.0 1260759151
    ##  7      1    1287    2.0 1260759187
    ##  8      1    1293    2.0 1260759148
    ##  9      1    1339    3.5 1260759125
    ## 10      1    1343    2.0 1260759131
    ## # ... with 99,994 more rows, and 2 more variables: title <fctr>,
    ## #   genres <fctr>

``` r
glimpse(ratings)
```

    ## Observations: 100,004
    ## Variables: 6
    ## $ userId    <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1...
    ## $ movieId   <int> 31, 1029, 1061, 1129, 1172, 1263, 1287, 1293, 1339, ...
    ## $ rating    <dbl> 2.5, 3.0, 3.0, 2.0, 4.0, 2.0, 2.0, 2.0, 3.5, 2.0, 2....
    ## $ timestamp <int> 1260759144, 1260759179, 1260759182, 1260759185, 1260...
    ## $ title     <fctr> Dangerous Minds (1995), Dumbo (1941), Sleepers (199...
    ## $ genres    <fctr> Drama, Animation|Children|Drama|Musical, Thriller, ...

Filtering rows with `filter()`
------------------------------

Here we illustrate the use of `filter()` by extracting user 1's observations from the *ratings* data frame.

``` r
u1 <- filter(ratings, userId == 1)
u1
```

    ## # A tibble: 20 x 6
    ##    userId movieId rating  timestamp
    ##     <int>   <int>  <dbl>      <int>
    ##  1      1      31    2.5 1260759144
    ##  2      1    1029    3.0 1260759179
    ##  3      1    1061    3.0 1260759182
    ##  4      1    1129    2.0 1260759185
    ##  5      1    1172    4.0 1260759205
    ##  6      1    1263    2.0 1260759151
    ##  7      1    1287    2.0 1260759187
    ##  8      1    1293    2.0 1260759148
    ##  9      1    1339    3.5 1260759125
    ## 10      1    1343    2.0 1260759131
    ## 11      1    1371    2.5 1260759135
    ## 12      1    1405    1.0 1260759203
    ## 13      1    1953    4.0 1260759191
    ## 14      1    2105    4.0 1260759139
    ## 15      1    2150    3.0 1260759194
    ## 16      1    2193    2.0 1260759198
    ## 17      1    2294    2.0 1260759108
    ## 18      1    2455    2.5 1260759113
    ## 19      1    2968    1.0 1260759200
    ## 20      1    3671    3.0 1260759117
    ## # ... with 2 more variables: title <fctr>, genres <fctr>

Next we extract the observations for user 1 that received a rating greater than 3. Multiple filter conditions are created with `&` (and) and `|` (or).

``` r
filter(ratings, userId == 1 & rating > 3)
```

    ## # A tibble: 4 x 6
    ##   userId movieId rating  timestamp
    ##    <int>   <int>  <dbl>      <int>
    ## 1      1    1172    4.0 1260759205
    ## 2      1    1339    3.5 1260759125
    ## 3      1    1953    4.0 1260759191
    ## 4      1    2105    4.0 1260759139
    ## # ... with 2 more variables: title <fctr>, genres <fctr>

Here's another way of writing the same condition as above:

``` r
filter(ratings, userId == 1, rating > 3)
```

    ## # A tibble: 4 x 6
    ##   userId movieId rating  timestamp
    ##    <int>   <int>  <dbl>      <int>
    ## 1      1    1172    4.0 1260759205
    ## 2      1    1339    3.5 1260759125
    ## 3      1    1953    4.0 1260759191
    ## 4      1    2105    4.0 1260759139
    ## # ... with 2 more variables: title <fctr>, genres <fctr>

The `%in%` command is often useful when using dplyr verbs:

``` r
filter(ratings, userId == 1, rating %in% c(1,4))
```

    ## # A tibble: 5 x 6
    ##   userId movieId rating  timestamp
    ##    <int>   <int>  <dbl>      <int>
    ## 1      1    1172      4 1260759205
    ## 2      1    1405      1 1260759203
    ## 3      1    1953      4 1260759191
    ## 4      1    2105      4 1260759139
    ## 5      1    2968      1 1260759200
    ## # ... with 2 more variables: title <fctr>, genres <fctr>

Introducing the pipe
--------------------

The pipe operator `%>%` is a very useful way of chaining together multiple operations. A typical format is something like:

*data* `%>%` *operation 1* `%>%` *operation 2*

You read the code from left to right: Start with *data*, apply some operation (operation 1) to it, get a result, and then apply another operation (operation 2) to that result, to generate another result (the final result, in this example). A useful way to think of the pipe is as similar to "then".

The main goal of the pipe is to make code easier, by focusing on the transformations rather than on what is being transformed. Usually this is the case, but its also possible to get carried away and end up with a huge whack of piped statements. Deciding when to break a block up is an art best learned by experience.

``` r
# filtering with the pipe
ratings %>% filter(userId == 1)
```

    ## # A tibble: 20 x 6
    ##    userId movieId rating  timestamp
    ##     <int>   <int>  <dbl>      <int>
    ##  1      1      31    2.5 1260759144
    ##  2      1    1029    3.0 1260759179
    ##  3      1    1061    3.0 1260759182
    ##  4      1    1129    2.0 1260759185
    ##  5      1    1172    4.0 1260759205
    ##  6      1    1263    2.0 1260759151
    ##  7      1    1287    2.0 1260759187
    ##  8      1    1293    2.0 1260759148
    ##  9      1    1339    3.5 1260759125
    ## 10      1    1343    2.0 1260759131
    ## 11      1    1371    2.5 1260759135
    ## 12      1    1405    1.0 1260759203
    ## 13      1    1953    4.0 1260759191
    ## 14      1    2105    4.0 1260759139
    ## 15      1    2150    3.0 1260759194
    ## 16      1    2193    2.0 1260759198
    ## 17      1    2294    2.0 1260759108
    ## 18      1    2455    2.5 1260759113
    ## 19      1    2968    1.0 1260759200
    ## 20      1    3671    3.0 1260759117
    ## # ... with 2 more variables: title <fctr>, genres <fctr>

The main usefulness of the pipe is when combining multiple operations

``` r
# first filter on userId then on rating
u1_likes <- ratings %>% filter(userId == 1) %>% filter(rating > 3)
u1_likes
```

    ## # A tibble: 4 x 6
    ##   userId movieId rating  timestamp
    ##    <int>   <int>  <dbl>      <int>
    ## 1      1    1172    4.0 1260759205
    ## 2      1    1339    3.5 1260759125
    ## 3      1    1953    4.0 1260759191
    ## 4      1    2105    4.0 1260759139
    ## # ... with 2 more variables: title <fctr>, genres <fctr>

``` r
# another way of doing the same thing
ratings %>% filter(userId == 1 & rating > 3)
```

    ## # A tibble: 4 x 6
    ##   userId movieId rating  timestamp
    ##    <int>   <int>  <dbl>      <int>
    ## 1      1    1172    4.0 1260759205
    ## 2      1    1339    3.5 1260759125
    ## 3      1    1953    4.0 1260759191
    ## 4      1    2105    4.0 1260759139
    ## # ... with 2 more variables: title <fctr>, genres <fctr>

Arranging rows with `arrange()`
-------------------------------

Ordering user 1's "liked" movies in descending order of rating (note the use of `desc`)

``` r
arrange(u1_likes, desc(rating))
```

    ## # A tibble: 4 x 6
    ##   userId movieId rating  timestamp
    ##    <int>   <int>  <dbl>      <int>
    ## 1      1    1172    4.0 1260759205
    ## 2      1    1953    4.0 1260759191
    ## 3      1    2105    4.0 1260759139
    ## 4      1    1339    3.5 1260759125
    ## # ... with 2 more variables: title <fctr>, genres <fctr>

Subsequent arguments to `arrange()` can be used to arrange by multiple columns. Here we first order user 1's liked movies by rating (in descending order) and then by timestamp (in ascending order)

``` r
arrange(u1_likes, desc(rating),timestamp)
```

    ## # A tibble: 4 x 6
    ##   userId movieId rating  timestamp
    ##    <int>   <int>  <dbl>      <int>
    ## 1      1    2105    4.0 1260759139
    ## 2      1    1953    4.0 1260759191
    ## 3      1    1172    4.0 1260759205
    ## 4      1    1339    3.5 1260759125
    ## # ... with 2 more variables: title <fctr>, genres <fctr>

We can also use the pipe to do the same thing

``` r
u1_likes %>% arrange(desc(rating))
```

    ## # A tibble: 4 x 6
    ##   userId movieId rating  timestamp
    ##    <int>   <int>  <dbl>      <int>
    ## 1      1    1172    4.0 1260759205
    ## 2      1    1953    4.0 1260759191
    ## 3      1    2105    4.0 1260759139
    ## 4      1    1339    3.5 1260759125
    ## # ... with 2 more variables: title <fctr>, genres <fctr>

Finally, here's an example of combining filter and arrange operations with the pipe

``` r
ratings %>% filter(userId == 1 & rating > 3) %>% arrange(desc(rating))
```

    ## # A tibble: 4 x 6
    ##   userId movieId rating  timestamp
    ##    <int>   <int>  <dbl>      <int>
    ## 1      1    1172    4.0 1260759205
    ## 2      1    1953    4.0 1260759191
    ## 3      1    2105    4.0 1260759139
    ## 4      1    1339    3.5 1260759125
    ## # ... with 2 more variables: title <fctr>, genres <fctr>

Selecting columns with `select()`
---------------------------------

Select is a bit like `filter()` for columns. The syntax is straightforward, the first argument gives the dataframe, and then you list the variables you want to select!

``` r
select(u1_likes,title,rating)
```

    ## # A tibble: 4 x 2
    ##                                            title rating
    ##                                           <fctr>  <dbl>
    ## 1 Cinema Paradiso (Nuovo cinema Paradiso) (1989)    4.0
    ## 2         Dracula (Bram Stoker's Dracula) (1992)    3.5
    ## 3                  French Connection, The (1971)    4.0
    ## 4                                    Tron (1982)    4.0

To exclude variables just put a minus sign in front of them

``` r
select(u1_likes,-userId,-timestamp)
```

    ## # A tibble: 4 x 4
    ##   movieId rating                                          title
    ##     <int>  <dbl>                                         <fctr>
    ## 1    1172    4.0 Cinema Paradiso (Nuovo cinema Paradiso) (1989)
    ## 2    1339    3.5         Dracula (Bram Stoker's Dracula) (1992)
    ## 3    1953    4.0                  French Connection, The (1971)
    ## 4    2105    4.0                                    Tron (1982)
    ## # ... with 1 more variables: genres <fctr>

You can also use `select()` to reorder variables. A useful helpful function here is `everything()`.

``` r
# original order
u1_likes
```

    ## # A tibble: 4 x 6
    ##   userId movieId rating  timestamp
    ##    <int>   <int>  <dbl>      <int>
    ## 1      1    1172    4.0 1260759205
    ## 2      1    1339    3.5 1260759125
    ## 3      1    1953    4.0 1260759191
    ## 4      1    2105    4.0 1260759139
    ## # ... with 2 more variables: title <fctr>, genres <fctr>

``` r
# reorder so title is first
select(u1_likes, title, everything())
```

    ## # A tibble: 4 x 6
    ##                                            title userId movieId rating
    ##                                           <fctr>  <int>   <int>  <dbl>
    ## 1 Cinema Paradiso (Nuovo cinema Paradiso) (1989)      1    1172    4.0
    ## 2         Dracula (Bram Stoker's Dracula) (1992)      1    1339    3.5
    ## 3                  French Connection, The (1971)      1    1953    4.0
    ## 4                                    Tron (1982)      1    2105    4.0
    ## # ... with 2 more variables: timestamp <int>, genres <fctr>

Adding new variables with `mutate()`
------------------------------------

Mutating operations add a new column to a dataframe. Here's a trivial example to get started:

``` r
mutate(u1_likes, this_is = "stupid")  
```

    ## # A tibble: 4 x 7
    ##   userId movieId rating  timestamp
    ##    <int>   <int>  <dbl>      <int>
    ## 1      1    1172    4.0 1260759205
    ## 2      1    1339    3.5 1260759125
    ## 3      1    1953    4.0 1260759191
    ## 4      1    2105    4.0 1260759139
    ## # ... with 3 more variables: title <fctr>, genres <fctr>, this_is <chr>

A more useful use of mutate is to construct new variable based on existing variables. This is the way that `mutate` is almost always used.

``` r
mutate(u1, like = ifelse(rating > 3, 1, 0))  
```

    ## # A tibble: 20 x 7
    ##    userId movieId rating  timestamp
    ##     <int>   <int>  <dbl>      <int>
    ##  1      1      31    2.5 1260759144
    ##  2      1    1029    3.0 1260759179
    ##  3      1    1061    3.0 1260759182
    ##  4      1    1129    2.0 1260759185
    ##  5      1    1172    4.0 1260759205
    ##  6      1    1263    2.0 1260759151
    ##  7      1    1287    2.0 1260759187
    ##  8      1    1293    2.0 1260759148
    ##  9      1    1339    3.5 1260759125
    ## 10      1    1343    2.0 1260759131
    ## 11      1    1371    2.5 1260759135
    ## 12      1    1405    1.0 1260759203
    ## 13      1    1953    4.0 1260759191
    ## 14      1    2105    4.0 1260759139
    ## 15      1    2150    3.0 1260759194
    ## 16      1    2193    2.0 1260759198
    ## 17      1    2294    2.0 1260759108
    ## 18      1    2455    2.5 1260759113
    ## 19      1    2968    1.0 1260759200
    ## 20      1    3671    3.0 1260759117
    ## # ... with 3 more variables: title <fctr>, genres <fctr>, like <dbl>

We can also use the pipe for mutating operations. Hopefully you're getting used to the pipe by now, so let's embed a mutating operation within a larger pipe than we've used before.

``` r
ratings %>% 
mutate(like = ifelse(rating > 3, 1, 0)) %>% 
filter(userId == 1) %>% 
select(like, everything()) 
```

    ## # A tibble: 20 x 7
    ##     like userId movieId rating  timestamp
    ##    <dbl>  <int>   <int>  <dbl>      <int>
    ##  1     0      1      31    2.5 1260759144
    ##  2     0      1    1029    3.0 1260759179
    ##  3     0      1    1061    3.0 1260759182
    ##  4     0      1    1129    2.0 1260759185
    ##  5     1      1    1172    4.0 1260759205
    ##  6     0      1    1263    2.0 1260759151
    ##  7     0      1    1287    2.0 1260759187
    ##  8     0      1    1293    2.0 1260759148
    ##  9     1      1    1339    3.5 1260759125
    ## 10     0      1    1343    2.0 1260759131
    ## 11     0      1    1371    2.5 1260759135
    ## 12     0      1    1405    1.0 1260759203
    ## 13     1      1    1953    4.0 1260759191
    ## 14     1      1    2105    4.0 1260759139
    ## 15     0      1    2150    3.0 1260759194
    ## 16     0      1    2193    2.0 1260759198
    ## 17     0      1    2294    2.0 1260759108
    ## 18     0      1    2455    2.5 1260759113
    ## 19     0      1    2968    1.0 1260759200
    ## 20     0      1    3671    3.0 1260759117
    ## # ... with 2 more variables: title <fctr>, genres <fctr>

Aggregating over rows with `summarise()`
----------------------------------------

The `summarise()` verb (or `summarize()` will also work) summarises the rows in a data frame in some way. When applied to the whole data frame, it will collapse it to a single row. For example, here we take user 1's data, and calculate their average rating and the number of movies they have given a rating higher than 3 to:

``` r
summarise(u1, mean = mean(rating), likes = sum(rating > 3))
```

    ## # A tibble: 1 x 2
    ##    mean likes
    ##   <dbl> <int>
    ## 1  2.55     4

You need to watch out for NAs when using `summarise()`. If one exists, operations like `mean()` will return NA. You can exclude NAs from calculations using `na.rm = TRUE`:

``` r
# introduce an NA
u1$rating[1] <- NA

# see what happens
summarise(u1, mean = mean(rating), likes = sum(rating > 3))
```

    ## # A tibble: 1 x 2
    ##    mean likes
    ##   <dbl> <int>
    ## 1    NA    NA

``` r
# with na.rm = T
summarise(u1, mean = mean(rating, na.rm = T), likes = sum(rating > 3, na.rm = T))
```

    ## # A tibble: 1 x 2
    ##       mean likes
    ##      <dbl> <int>
    ## 1 2.552632     4

`summarise()` is most useful when combined with `group_by()`, which imposes a grouping structure on a data frame. After applying `group_by()`, subsequent dplyr verbs will be applied to individual groups, basically repeating the code for each group. That means that `summarise()` will calculate a summary for each group:

``` r
# tell dplyr to group ratings by userId
ratings_by_user <- group_by(ratings, userId)

# apply summarize() to see how many movies each user has rated
ratings_by_user %>% summarize(count = n()) %>% head()
```

    ## # A tibble: 6 x 2
    ##   userId count
    ##    <int> <int>
    ## 1      1    20
    ## 2      2    76
    ## 3      3    51
    ## 4      4   204
    ## 5      5   100
    ## 6      6    44

``` r
# get sorted counts (plus some presentation stuff)
ratings %>% 
group_by(userId) %>% 
summarize(count = n()) %>% 
arrange(desc(count)) %>% 
head(20) %>%     # take first two rows
t()  # transpose 
```

    ##        [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8] [,9] [,10] [,11] [,12]
    ## userId  547  564  624   15   73  452  468  380  311    30   294   509
    ## count  2391 1868 1735 1700 1610 1340 1291 1063 1019  1011   947   923
    ##        [,13] [,14] [,15] [,16] [,17] [,18] [,19] [,20]
    ## userId   580   213   212   472   388    23   457   518
    ## count    922   910   876   830   792   726   713   707

``` r
# or with the pipe (last time)
ratings %>% group_by(userId) %>% summarize(count = n()) %>% head(10)
```

    ## # A tibble: 10 x 2
    ##    userId count
    ##     <int> <int>
    ##  1      1    20
    ##  2      2    76
    ##  3      3    51
    ##  4      4   204
    ##  5      5   100
    ##  6      6    44
    ##  7      7    88
    ##  8      8   116
    ##  9      9    45
    ## 10     10    46

Other uses of `grouped_by()`: grouped filters and grouped mutates
-----------------------------------------------------------------

While you'll probably use `group_by()` most often with `summarise()`, it can also be useful when used in conjunction with `filter()` and `mutate()`. Grouped filters perform the filtering within each group. Below we use it to extract each user's favourite movie (or movies, if there's a tie).

``` r
# example of a grouped filter
ratings %>% group_by(userId) %>% filter(rank(desc(rating)) < 2)
```

    ## # A tibble: 102 x 6
    ## # Groups:   userId [69]
    ##    userId movieId rating  timestamp
    ##     <int>   <int>  <dbl>      <int>
    ##  1     13       1      5 1331380058
    ##  2     13     356      5 1331380018
    ##  3     14    3175      5  976244313
    ##  4     18      25      5  856006886
    ##  5     18      32      5  856006885
    ##  6     24       6      5  849321588
    ##  7     24     296      5  849282414
    ##  8     35    3072      5 1174450056
    ##  9     50     589      5  847412628
    ## 10     72   55820      5 1464722872
    ## # ... with 92 more rows, and 2 more variables: title <fctr>, genres <fctr>

Here we use a grouped mutate to standardise each user's ratings so that they have a mean of zero (for each user, which guarantees the overall mean rating is also zero).

``` r
# example of a grouped mutate
ratings %>% 
group_by(userId) %>%
mutate(centered_rating = rating - mean(rating)) %>% 
select(-movieId,-timestamp,-genres)
```

    ## # A tibble: 100,004 x 4
    ## # Groups:   userId [671]
    ##    userId rating                                          title
    ##     <int>  <dbl>                                         <fctr>
    ##  1      1    2.5                         Dangerous Minds (1995)
    ##  2      1    3.0                                   Dumbo (1941)
    ##  3      1    3.0                                Sleepers (1996)
    ##  4      1    2.0                    Escape from New York (1981)
    ##  5      1    4.0 Cinema Paradiso (Nuovo cinema Paradiso) (1989)
    ##  6      1    2.0                        Deer Hunter, The (1978)
    ##  7      1    2.0                                 Ben-Hur (1959)
    ##  8      1    2.0                                  Gandhi (1982)
    ##  9      1    3.5         Dracula (Bram Stoker's Dracula) (1992)
    ## 10      1    2.0                               Cape Fear (1991)
    ## # ... with 99,994 more rows, and 1 more variables: centered_rating <dbl>

Putting it all together: extracting a sample set of reviews for Lesson 2
------------------------------------------------------------------------

In this section we'll take what we've learned and do something useful: build a 15x20 matrix containing the reviews made on 20 movies by 15 users. We'll use this matrix in the next lesson to build a recommendation system.

First, we select the 15 users we want to use. I've chosen to use 15 users with moderately frequent viewing habits (remember there are 700 users and 9000 movies), mainly to make sure there are some (but not too many) empty ratings.

``` r
users_frq <- ratings %>% group_by(userId) %>% summarize(count = n()) %>% arrange(desc(count))
my_users <- users_frq$userId[101:115]
```

Next, we select the 20 movies we want to use:

``` r
movies_frq <- ratings %>% group_by(movieId) %>% summarize(count = n()) %>% arrange(desc(count))
my_movies <- movies_frq$movieId[101:120]
```

Now we make a dataset with only those 15 users and 20 movies:

``` r
ratings_red <- ratings %>% filter(userId %in% my_users, movieId %in% my_movies) 
# and check there are 15 users and 20 movies in the reduced dataset
n_users <- length(unique(ratings_red$userId))
n_movies <- length(unique(ratings_red$movieId))
print(paste("number of users is",n_users))
```

    ## [1] "number of users is 15"

``` r
print(paste("number of movies is",n_movies))
```

    ## [1] "number of movies is 20"

Let's see what the 20 movies are:

``` r
movies %>% filter(movieId %in% my_movies) %>% select(title)
```

    ##                                                      title
    ## 1                                       Taxi Driver (1976)
    ## 2                                        Waterworld (1995)
    ## 3                                          Outbreak (1995)
    ## 4                            Star Trek: Generations (1994)
    ## 5                          Clear and Present Danger (1994)
    ## 6                                        Casablanca (1942)
    ## 7                                 Wizard of Oz, The (1939)
    ## 8                                    Apocalypse Now (1979)
    ## 9                                       Stand by Me (1986)
    ## 10                               Fifth Element, The (1997)
    ## 11                                       Armageddon (1998)
    ## 12                                         Rain Man (1988)
    ## 13                              Breakfast Club, The (1985)
    ## 14            Austin Powers: The Spy Who Shagged Me (1999)
    ## 15                                     American Pie (1999)
    ## 16 Crouching Tiger, Hidden Dragon (Wo hu cang long) (2000)
    ## 17                                Beautiful Mind, A (2001)
    ## 18                                  Minority Report (2002)
    ## 19                                Kill Bill: Vol. 1 (2003)
    ## 20                                        Inception (2010)

However, note all the movie titles are still being kept:

``` r
head(levels(ratings_red$title))
```

    ## [1] "¡Three Amigos! (1986)"                  
    ## [2] "...And God Spoke (1993)"                
    ## [3] "...And Justice for All (1979)"          
    ## [4] "'burbs, The (1989)"                     
    ## [5] "'Hellboy': The Seeds of Creation (2004)"
    ## [6] "'Neath the Arizona Skies (1934)"

This actually isn't what we want, so let's drop the ones we won't use.

``` r
ratings_red <- droplevels(ratings_red)
levels(ratings_red$title)
```

    ##  [1] "American Pie (1999)"                                    
    ##  [2] "Apocalypse Now (1979)"                                  
    ##  [3] "Armageddon (1998)"                                      
    ##  [4] "Austin Powers: The Spy Who Shagged Me (1999)"           
    ##  [5] "Beautiful Mind, A (2001)"                               
    ##  [6] "Breakfast Club, The (1985)"                             
    ##  [7] "Casablanca (1942)"                                      
    ##  [8] "Clear and Present Danger (1994)"                        
    ##  [9] "Crouching Tiger, Hidden Dragon (Wo hu cang long) (2000)"
    ## [10] "Fifth Element, The (1997)"                              
    ## [11] "Inception (2010)"                                       
    ## [12] "Kill Bill: Vol. 1 (2003)"                               
    ## [13] "Minority Report (2002)"                                 
    ## [14] "Outbreak (1995)"                                        
    ## [15] "Rain Man (1988)"                                        
    ## [16] "Stand by Me (1986)"                                     
    ## [17] "Star Trek: Generations (1994)"                          
    ## [18] "Taxi Driver (1976)"                                     
    ## [19] "Waterworld (1995)"                                      
    ## [20] "Wizard of Oz, The (1939)"

We now want to reshape the data frame into a 15x20 matrix i.e.from "long" format to "wide" format. We can do this using the `spread()` verb.

``` r
ratings_red %>% spread(key = movieId, value = rating)
```

    ## # A tibble: 106 x 24
    ##    userId  timestamp                           title
    ##  *  <int>      <int>                          <fctr>
    ##  1    149 1436919794                Inception (2010)
    ##  2    149 1436921723      Breakfast Club, The (1985)
    ##  3    149 1436923259       Fifth Element, The (1997)
    ##  4    149 1436923311             American Pie (1999)
    ##  5    149 1436923357              Stand by Me (1986)
    ##  6    177  907379242               Armageddon (1998)
    ##  7    177  907380001       Fifth Element, The (1997)
    ##  8    177  907380055   Star Trek: Generations (1994)
    ##  9    177  907380710 Clear and Present Danger (1994)
    ## 10    177  907380710              Taxi Driver (1976)
    ## # ... with 96 more rows, and 21 more variables: genres <fctr>,
    ## #   `111` <dbl>, `208` <dbl>, `292` <dbl>, `329` <dbl>, `349` <dbl>,
    ## #   `912` <dbl>, `919` <dbl>, `1208` <dbl>, `1259` <dbl>, `1527` <dbl>,
    ## #   `1917` <dbl>, `1961` <dbl>, `1968` <dbl>, `2683` <dbl>, `2706` <dbl>,
    ## #   `3996` <dbl>, `4995` <dbl>, `5445` <dbl>, `6874` <dbl>, `79132` <dbl>

The preceding line *doesn't* work: as you can see we land up with more than one row per user. But it is useful as an illustration of `spread()`. Question: why doesn't it work?

Here's the corrected version:

``` r
ratings_red %>% select(userId,title,rating) %>% spread(key = title, value = rating)
```

    ## # A tibble: 15 x 21
    ##    userId `American Pie (1999)` `Apocalypse Now (1979)`
    ##  *  <int>                 <dbl>                   <dbl>
    ##  1    149                   3.0                      NA
    ##  2    177                    NA                      NA
    ##  3    200                   1.5                      NA
    ##  4    236                    NA                     5.0
    ##  5    240                    NA                      NA
    ##  6    270                    NA                     4.0
    ##  7    287                    NA                      NA
    ##  8    295                    NA                      NA
    ##  9    303                   2.5                      NA
    ## 10    408                    NA                     5.0
    ## 11    426                   1.0                      NA
    ## 12    442                   4.5                     4.5
    ## 13    500                   2.0                      NA
    ## 14    522                   3.0                     3.5
    ## 15    562                   4.5                     4.0
    ## # ... with 18 more variables: `Armageddon (1998)` <dbl>, `Austin Powers:
    ## #   The Spy Who Shagged Me (1999)` <dbl>, `Beautiful Mind, A
    ## #   (2001)` <dbl>, `Breakfast Club, The (1985)` <dbl>, `Casablanca
    ## #   (1942)` <dbl>, `Clear and Present Danger (1994)` <dbl>, `Crouching
    ## #   Tiger, Hidden Dragon (Wo hu cang long) (2000)` <dbl>, `Fifth Element,
    ## #   The (1997)` <dbl>, `Inception (2010)` <dbl>, `Kill Bill: Vol. 1
    ## #   (2003)` <dbl>, `Minority Report (2002)` <dbl>, `Outbreak
    ## #   (1995)` <dbl>, `Rain Man (1988)` <dbl>, `Stand by Me (1986)` <dbl>,
    ## #   `Star Trek: Generations (1994)` <dbl>, `Taxi Driver (1976)` <dbl>,
    ## #   `Waterworld (1995)` <dbl>, `Wizard of Oz, The (1939)` <dbl>

Finally, since we just want to know who has seen what, we replace all NAs with 0 and all other ratings with 1:

``` r
viewed_movies <- ratings_red %>% 
  select(userId,title,rating) %>% 
  complete(userId, title) %>% 
  mutate(seen = ifelse(is.na(rating),0,1)) %>% 
  select(userId,title,seen) %>% 
  spread(key = title, value = seen)
```

We could have got this more simply with a call to `table()`, which creates a two-way frequency table.

``` r
table(ratings_red$userId,ratings_red$title)
```

    ##      
    ##       American Pie (1999) Apocalypse Now (1979) Armageddon (1998)
    ##   149                   1                     0                 0
    ##   177                   0                     0                 1
    ##   200                   1                     0                 1
    ##   236                   0                     1                 0
    ##   240                   0                     0                 0
    ##   270                   0                     1                 0
    ##   287                   0                     0                 1
    ##   295                   0                     0                 0
    ##   303                   1                     0                 0
    ##   408                   0                     1                 1
    ##   426                   1                     0                 1
    ##   442                   1                     1                 1
    ##   500                   1                     0                 0
    ##   522                   1                     1                 1
    ##   562                   1                     1                 1
    ##      
    ##       Austin Powers: The Spy Who Shagged Me (1999)
    ##   149                                            0
    ##   177                                            0
    ##   200                                            0
    ##   236                                            0
    ##   240                                            0
    ##   270                                            0
    ##   287                                            0
    ##   295                                            0
    ##   303                                            0
    ##   408                                            1
    ##   426                                            0
    ##   442                                            1
    ##   500                                            1
    ##   522                                            0
    ##   562                                            1
    ##      
    ##       Beautiful Mind, A (2001) Breakfast Club, The (1985)
    ##   149                        0                          1
    ##   177                        0                          1
    ##   200                        1                          0
    ##   236                        0                          0
    ##   240                        1                          0
    ##   270                        1                          0
    ##   287                        0                          0
    ##   295                        1                          0
    ##   303                        1                          1
    ##   408                        0                          1
    ##   426                        1                          0
    ##   442                        1                          1
    ##   500                        1                          1
    ##   522                        1                          0
    ##   562                        0                          0
    ##      
    ##       Casablanca (1942) Clear and Present Danger (1994)
    ##   149                 0                               0
    ##   177                 0                               1
    ##   200                 0                               0
    ##   236                 1                               0
    ##   240                 0                               0
    ##   270                 0                               0
    ##   287                 0                               0
    ##   295                 1                               0
    ##   303                 0                               0
    ##   408                 0                               0
    ##   426                 0                               0
    ##   442                 0                               1
    ##   500                 0                               0
    ##   522                 0                               0
    ##   562                 0                               0
    ##      
    ##       Crouching Tiger, Hidden Dragon (Wo hu cang long) (2000)
    ##   149                                                       0
    ##   177                                                       0
    ##   200                                                       0
    ##   236                                                       1
    ##   240                                                       1
    ##   270                                                       0
    ##   287                                                       1
    ##   295                                                       1
    ##   303                                                       0
    ##   408                                                       0
    ##   426                                                       0
    ##   442                                                       0
    ##   500                                                       0
    ##   522                                                       0
    ##   562                                                       1
    ##      
    ##       Fifth Element, The (1997) Inception (2010) Kill Bill: Vol. 1 (2003)
    ##   149                         1                1                        0
    ##   177                         1                0                        0
    ##   200                         1                1                        0
    ##   236                         0                0                        0
    ##   240                         0                0                        1
    ##   270                         0                1                        1
    ##   287                         0                1                        0
    ##   295                         1                0                        1
    ##   303                         1                1                        1
    ##   408                         1                0                        0
    ##   426                         0                1                        1
    ##   442                         0                0                        1
    ##   500                         0                0                        0
    ##   522                         0                1                        1
    ##   562                         0                0                        1
    ##      
    ##       Minority Report (2002) Outbreak (1995) Rain Man (1988)
    ##   149                      0               0               0
    ##   177                      0               1               0
    ##   200                      1               0               0
    ##   236                      0               0               0
    ##   240                      1               0               0
    ##   270                      1               0               0
    ##   287                      1               0               0
    ##   295                      1               1               1
    ##   303                      0               0               1
    ##   408                      0               0               1
    ##   426                      0               0               1
    ##   442                      0               0               1
    ##   500                      0               0               0
    ##   522                      0               0               1
    ##   562                      1               1               0
    ##      
    ##       Stand by Me (1986) Star Trek: Generations (1994) Taxi Driver (1976)
    ##   149                  1                             0                  0
    ##   177                  1                             1                  1
    ##   200                  0                             0                  0
    ##   236                  0                             0                  1
    ##   240                  0                             0                  1
    ##   270                  0                             0                  0
    ##   287                  0                             1                  0
    ##   295                  0                             0                  0
    ##   303                  1                             0                  0
    ##   408                  1                             0                  0
    ##   426                  0                             0                  0
    ##   442                  0                             0                  0
    ##   500                  0                             1                  0
    ##   522                  0                             0                  0
    ##   562                  0                             0                  0
    ##      
    ##       Waterworld (1995) Wizard of Oz, The (1939)
    ##   149                 0                        0
    ##   177                 0                        0
    ##   200                 0                        0
    ##   236                 0                        0
    ##   240                 0                        1
    ##   270                 0                        0
    ##   287                 0                        1
    ##   295                 1                        1
    ##   303                 0                        1
    ##   408                 0                        1
    ##   426                 0                        0
    ##   442                 1                        1
    ##   500                 0                        1
    ##   522                 0                        0
    ##   562                 1                        0

Finally, we save our output for use in the next lesson!

``` r
save(ratings_red, viewed_movies, file = "output/recommender.RData")
```

Combining data frames with *joins*
----------------------------------

We'll often need to combine the information contained in two or more tables. To do this, we need various kinds of database *joins*. This section describes the basic join operations that we need to combine data frames. The examples are taken from [Chapter 13](http://r4ds.had.co.nz/relational-data.html) of R4DS, which also contains a lot more general information on relational data.

First, we make some very simple data tables to show how the joins work:

``` r
# make some example data
x <- tribble(
  ~key, ~xvalue,
  1, "x1",
  2, "x2",
  3, "x3"
)

y <- tribble(
  ~key, ~yvalue,
  1, "y1",
  2, "y2",
  4, "y3"
)
```

### Mutating joins: `inner_join`, `left_join`, `right_join`, `full_join`

The first set of joins we look at are called mutating joins. These first match observations in two tables in some way, and then combine variables from the two tables.

There are four types of mutating joins: inner joins, left joins, right joins, and full joins.

An **inner join** keeps observations that appear in *both* tables.

``` r
inner_join(x,y)
```

    ## Joining, by = "key"

    ## # A tibble: 2 x 3
    ##     key xvalue yvalue
    ##   <dbl>  <chr>  <chr>
    ## 1     1     x1     y1
    ## 2     2     x2     y2

``` r
inner_join(y,x)
```

    ## Joining, by = "key"

    ## # A tibble: 2 x 3
    ##     key yvalue xvalue
    ##   <dbl>  <chr>  <chr>
    ## 1     1     y1     x1
    ## 2     2     y2     x2

The other three joints are all **outer joins**: they keep observations that appear in *at least one* of the tables.

A **left join** keeps all observations in x.

``` r
left_join(x,y)
```

    ## Joining, by = "key"

    ## # A tibble: 3 x 3
    ##     key xvalue yvalue
    ##   <dbl>  <chr>  <chr>
    ## 1     1     x1     y1
    ## 2     2     x2     y2
    ## 3     3     x3   <NA>

``` r
left_join(y,x)
```

    ## Joining, by = "key"

    ## # A tibble: 3 x 3
    ##     key yvalue xvalue
    ##   <dbl>  <chr>  <chr>
    ## 1     1     y1     x1
    ## 2     2     y2     x2
    ## 3     4     y3   <NA>

A **right join** keeps all observations in y.

``` r
# note this is the same as left_join(y,x)
right_join(x,y)
```

    ## Joining, by = "key"

    ## # A tibble: 3 x 3
    ##     key xvalue yvalue
    ##   <dbl>  <chr>  <chr>
    ## 1     1     x1     y1
    ## 2     2     x2     y2
    ## 3     4   <NA>     y3

A **full join** keeps observations in x or y.

``` r
full_join(x,y)
```

    ## Joining, by = "key"

    ## # A tibble: 4 x 3
    ##     key xvalue yvalue
    ##   <dbl>  <chr>  <chr>
    ## 1     1     x1     y1
    ## 2     2     x2     y2
    ## 3     3     x3   <NA>
    ## 4     4   <NA>     y3

We can now re-examine the join we used to add movie titles to the ratings data frame earlier:

``` r
# reload the MovieLens data
load("data/movielens-small.RData")
ratings <- as.tibble(ratings)
movies <- as.tibble(movies)
```

Note that the same *movieId* can appear multiple times in the *ratings* data frame:

``` r
print(ratings %>% arrange(movieId)) # note duplicate movieIds
```

    ## # A tibble: 100,004 x 4
    ##    userId movieId rating  timestamp
    ##     <int>   <int>  <dbl>      <int>
    ##  1      7       1    3.0  851866703
    ##  2      9       1    4.0  938629179
    ##  3     13       1    5.0 1331380058
    ##  4     15       1    2.0  997938310
    ##  5     19       1    3.0  855190091
    ##  6     20       1    3.5 1238729767
    ##  7     23       1    3.0 1148729853
    ##  8     26       1    5.0 1360087980
    ##  9     30       1    4.0  944943070
    ## 10     37       1    4.0  981308121
    ## # ... with 99,994 more rows

But each *movieId* only appears once in the *movies* data frame:

``` r
print(movies %>% arrange(movieId)) # note unique movieIds
```

    ## # A tibble: 9,125 x 3
    ##    movieId                              title
    ##      <int>                             <fctr>
    ##  1       1                   Toy Story (1995)
    ##  2       2                     Jumanji (1995)
    ##  3       3            Grumpier Old Men (1995)
    ##  4       4           Waiting to Exhale (1995)
    ##  5       5 Father of the Bride Part II (1995)
    ##  6       6                        Heat (1995)
    ##  7       7                     Sabrina (1995)
    ##  8       8                Tom and Huck (1995)
    ##  9       9                Sudden Death (1995)
    ## 10      10                   GoldenEye (1995)
    ## # ... with 9,115 more rows, and 1 more variables: genres <fctr>

In this case a left join by the *movieId* key copies across the movie title information (as well as any other information in the *movies* data frame):

``` r
print(left_join(ratings, movies, by = "movieId") %>% select(title,everything()))
```

    ## # A tibble: 100,004 x 6
    ##                                             title userId movieId rating
    ##                                            <fctr>  <int>   <int>  <dbl>
    ##  1                         Dangerous Minds (1995)      1      31    2.5
    ##  2                                   Dumbo (1941)      1    1029    3.0
    ##  3                                Sleepers (1996)      1    1061    3.0
    ##  4                    Escape from New York (1981)      1    1129    2.0
    ##  5 Cinema Paradiso (Nuovo cinema Paradiso) (1989)      1    1172    4.0
    ##  6                        Deer Hunter, The (1978)      1    1263    2.0
    ##  7                                 Ben-Hur (1959)      1    1287    2.0
    ##  8                                  Gandhi (1982)      1    1293    2.0
    ##  9         Dracula (Bram Stoker's Dracula) (1992)      1    1339    3.5
    ## 10                               Cape Fear (1991)      1    1343    2.0
    ## # ... with 99,994 more rows, and 2 more variables: timestamp <int>,
    ## #   genres <fctr>

### Filtering joins: `semi_join`, `anti_join`

The last two joins we look at are **filtering joins**. These match observations in two tables, but do not add variables. There are two types of filtering joins: semi-joins and anti-joins.

A **semi join** keeps all observations in x that appear in y (note variables are from x only),

``` r
semi_join(x,y)
```

    ## Joining, by = "key"

    ## # A tibble: 2 x 2
    ##     key xvalue
    ##   <dbl>  <chr>
    ## 1     1     x1
    ## 2     2     x2

``` r
semi_join(y,x)
```

    ## Joining, by = "key"

    ## # A tibble: 2 x 2
    ##     key yvalue
    ##   <dbl>  <chr>
    ## 1     1     y1
    ## 2     2     y2

while an **anti join** *drops* all observations in x that appear in y (note variables are from x only).

``` r
anti_join(x,y)
```

    ## Joining, by = "key"

    ## # A tibble: 1 x 2
    ##     key xvalue
    ##   <dbl>  <chr>
    ## 1     3     x3

``` r
anti_join(y,x)
```

    ## Joining, by = "key"

    ## # A tibble: 1 x 2
    ##     key yvalue
    ##   <dbl>  <chr>
    ## 1     4     y3

Exercises
---------

Do the exercises in [Chapter 5](http://r4ds.had.co.nz/transform.html) (data transformation using the **dplyr** verbs) and [Chapter 13](http://r4ds.had.co.nz/relational-data.html) (on database joins) of R4DS. There are exercises at the end of each major subsection. Do as many of these exercises as you need to feel comfortable with the material - I suggest doing at least the first two of each set of exercises.
