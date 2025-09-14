# Lecture 3 -  dates & times, strings,  as well as factors

### Lecture learning objectives:
By the end of this lecture and worksheet 3, students should be able to:

* Manipulate dates and times using the {lubridate} package
* Be able to modify strings in a data frame using regular expressions and the {stringr} package
* Cast categorical columns in a data frame as factors when appropriate, and manipulate factor levels as needed in preparation for data visualisation and statistical analysis (using base R and {forcats} package functions)



```R
library(tidyverse)
library(gapminder)
options(repr.matrix.max.rows = 10)
```

    ── [1mAttaching packages[22m ───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── tidyverse 1.3.2 ──
    [32m✔[39m [34mggplot2[39m 3.3.6     [32m✔[39m [34mpurrr  [39m 0.3.4
    [32m✔[39m [34mtibble [39m 3.1.8     [32m✔[39m [34mdplyr  [39m 1.0.9
    [32m✔[39m [34mtidyr  [39m 1.2.0     [32m✔[39m [34mstringr[39m 1.4.1
    [32m✔[39m [34mreadr  [39m 2.1.2     [32m✔[39m [34mforcats[39m 0.5.1
    ── [1mConflicts[22m ──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── tidyverse_conflicts() ──
    [31m✖[39m [34mdplyr[39m::[32mfilter()[39m masks [34mstats[39m::filter()
    [31m✖[39m [34mdplyr[39m::[32mlag()[39m    masks [34mstats[39m::lag()


## Tibbles versus data frames

You have seen and heard me talk about data frames and tibbles in R, and sometimes I carelessly interchange the terms. Let's take moment to discuss the difference between the two!

Tibbles are special data frames, with special/extra properties. The two important ones you will care about are:

- In RStudio, tibbles only output the first 10 rows

- When you numerically subset a data frame to 1 column, you get a vector. However, when you numerically subset a tibble you still get a tibble back.

When you create a data frame using base R functions, either via `data.frame` or one of the base R `read.*` functions, you get an objects whose class is data frame:


```R
example <- data.frame(a = c(1, 5, 9), b = "z", "a", "t")
example

class(example)
```


<table class="dataframe">
<caption>A data.frame: 3 × 4</caption>
<thead>
	<tr><th scope=col>a</th><th scope=col>b</th><th scope=col>X.a.</th><th scope=col>X.t.</th></tr>
	<tr><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th></tr>
</thead>
<tbody>
	<tr><td>1</td><td>z</td><td>a</td><td>t</td></tr>
	<tr><td>5</td><td>z</td><td>a</td><td>t</td></tr>
	<tr><td>9</td><td>z</td><td>a</td><td>t</td></tr>
</tbody>
</table>




'data.frame'


Tibbles inherit from the data frame class (meaning that have many of the same properties as data frames), but they also have the extra properties I just discussed:


```R
example2 <- tibble(a = c(1, 5, 9), b = "z", "a", "t")
example

class(example2)
```


<table class="dataframe">
<caption>A data.frame: 3 × 4</caption>
<thead>
	<tr><th scope=col>a</th><th scope=col>b</th><th scope=col>X.a.</th><th scope=col>X.t.</th></tr>
	<tr><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th></tr>
</thead>
<tbody>
	<tr><td>1</td><td>z</td><td>a</td><td>t</td></tr>
	<tr><td>5</td><td>z</td><td>a</td><td>t</td></tr>
	<tr><td>9</td><td>z</td><td>a</td><td>t</td></tr>
</tbody>
</table>




<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'tbl_df'</li><li>'tbl'</li><li>'data.frame'</li></ol>



Note: there are **some** tidyverse functions that will coerce a data frame to a tibble, because what the user is asking for is not possible with a data frame. One such example is `group_by` (which we will learn about next week):


```R
group_by(example, a) %>% 
    class()
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'grouped_df'</li><li>'tbl_df'</li><li>'tbl'</li><li>'data.frame'</li></ol>



**Rule of thumb:**  if you want a tibble, it’s on you to know that and express that explicitly with `as_tibble()` (if it’s a data frame to start out with).

## Dates and times

<img src="https://d33wubrfki0l68.cloudfront.net/baa19d0ebf9b97949a7ad259b29a1c4ae031c8e2/8e9b8/diagrams/vectors/summary-tree-s3-1.png" width=300>

*Source: [Advanced R](https://adv-r.hadley.nz/) by Hadley Wickham*

### Working with dates and times

Your weapon: The lubridate package (CRAN; [GitHub](https://github.com/tidyverse/lubridate); [main vignette](https://cran.r-project.org/web/packages/lubridate/vignettes/lubridate.html)).


```R
library(lubridate)
```

    
    Attaching package: ‘lubridate’
    
    
    The following objects are masked from ‘package:base’:
    
        date, intersect, setdiff, union
    
    


### Get your hands on some dates or date-times

Use `lubridate`’s `today` to get today’s date, without any time:


```R
today()
```


<time datetime="2022-09-10">2022-09-10</time>



```R
class(today())
```


'Date'


Use `lubridate`'s `now` to get RIGHT NOW, meaning the date and the time:


```R
now()
```


    [1] "2022-09-10 22:36:51 PDT"



```R
class(now())
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'POSIXct'</li><li>'POSIXt'</li></ol>



### Get date or date-time from character

Use the `lubridate` helpers to convert character or unquoted numbers into dates or date-times:


```R
ymd("2017-01-31")
```


<time datetime="2017-01-31">2017-01-31</time>



```R
mdy("January 31st, 2017")
```


<time datetime="2017-01-31">2017-01-31</time>



```R
dmy("31-Jan-2017")
```


<time datetime="2017-01-31">2017-01-31</time>



```R
ymd(20170131)
```


<time datetime="2017-01-31">2017-01-31</time>



```R
ymd_hms("2017-01-31 20:11:59")
```


    [1] "2017-01-31 20:11:59 UTC"



```R
mdy_hm("01/31/2017 08:01")
```


    [1] "2017-01-31 08:01:00 UTC"


You can also force the creation of a date-time from a date by supplying a timezone:


```R
class(ymd(20170131, tz = "UTC"))
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'POSIXct'</li><li>'POSIXt'</li></ol>



### Build date or date-time from parts

Instead of a single string, sometimes you’ll have the individual components of the date-time spread across multiple columns. 


```R
dates <- tibble(year = c(2015, 2016, 2017, 2018, 2019),
               month = c(9, 9, 9, 9, 9),
               day = c(3, 4, 2, 6, 3))

dates
```


<table class="dataframe">
<caption>A tibble: 5 × 3</caption>
<thead>
	<tr><th scope=col>year</th><th scope=col>month</th><th scope=col>day</th></tr>
	<tr><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>2015</td><td>9</td><td>3</td></tr>
	<tr><td>2016</td><td>9</td><td>4</td></tr>
	<tr><td>2017</td><td>9</td><td>2</td></tr>
	<tr><td>2018</td><td>9</td><td>6</td></tr>
	<tr><td>2019</td><td>9</td><td>3</td></tr>
</tbody>
</table>



To create a date/time from this sort of input, use `make_date` for dates, or `make_datetime` for date-times:


```R
# make a single date from year, month and day
dates %>% 
    mutate(date = make_date(year, month, day))
```


<table class="dataframe">
<caption>A tibble: 5 × 4</caption>
<thead>
	<tr><th scope=col>year</th><th scope=col>month</th><th scope=col>day</th><th scope=col>date</th></tr>
	<tr><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;date&gt;</th></tr>
</thead>
<tbody>
	<tr><td>2015</td><td>9</td><td>3</td><td>2015-09-03</td></tr>
	<tr><td>2016</td><td>9</td><td>4</td><td>2016-09-04</td></tr>
	<tr><td>2017</td><td>9</td><td>2</td><td>2017-09-02</td></tr>
	<tr><td>2018</td><td>9</td><td>6</td><td>2018-09-06</td></tr>
	<tr><td>2019</td><td>9</td><td>3</td><td>2019-09-03</td></tr>
</tbody>
</table>



### Getting components from a date or date-time

Sometimes you have the date or date-time and you want to extract a component, such as year or day.


```R
datetime <- ymd_hms("2016-07-08 12:34:56")
datetime
```


    [1] "2016-07-08 12:34:56 UTC"



```R
year(datetime)
```


2016



```R
month(datetime)
```


7



```R
mday(datetime)
```


8



```R
yday(datetime)
```


190



```R
wday(datetime, label = TRUE, abbr = FALSE)
```


Friday
<details>
	<summary style=display:list-item;cursor:pointer>
		<strong>Levels</strong>:
	</summary>
	<style>
	.list-inline {list-style: none; margin:0; padding: 0}
	.list-inline>li {display: inline-block}
	.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
	</style>
	<ol class=list-inline><li>'Sunday'</li><li>'Monday'</li><li>'Tuesday'</li><li>'Wednesday'</li><li>'Thursday'</li><li>'Friday'</li><li>'Saturday'</li></ol>
</details>


For `month` and `wday` you can set `label = TRUE` to return the abbreviated name of the month or day of the week. Set `abbr = FALSE` to return the full name.

### More date and time wrangling possibilities:

- [Tools for working with time spans](https://r4ds.had.co.nz/dates-and-times.html#time-spans), to calculate durations, periods and intervals
- [Tools for dealing with time zones](https://r4ds.had.co.nz/dates-and-times.html#time-zones)

## String manipulations

#### The recommended tools:

- **[stringr](https://stringr.tidyverse.org/) package:** A core package in the tidyverse. Main functions start with str_. Auto-complete is your friend. [`stringr` cheatsheet](https://github.com/rstudio/cheatsheets/blob/master/strings.pdf)

- **[tidyr](https://tidyr.tidyverse.org/) package:** Useful for functions that split one character vector into many and vice versa: `separate`, `unite`, `extract`. [`tidyr` cheatsheet](https://github.com/rstudio/cheatsheets/blob/master/data-import.pdf)

- **Base functions:** `nchar`, `strsplit`, `substr`, `paste`, and `paste0`.

#### Regex-free string manipulation with stringr and tidyr

Basic string manipulation tasks:

- Study a single character vector
    - How long are the strings?
    - Presence/absence of a literal string
- Operate on a single character vector
    - Keep/discard elements that contain a literal string
    - Split into two or more character vectors using a fixed delimiter
    - Snip out pieces of the strings based on character position
    - Collapse into a single string
- Operate on two or more character vectors
    - Glue them together element-wise to get a new character vector.

`fruit`, `words`, and `sentences` are character vectors that ship with `stringr` for practicing.

NOTE - we will be working with vectors today. If you want to operate on data frames, you will need to use these functions inside of data frame/tibble functions, like `filter` and `mutate`.

#### Detect or filter on a target string

Determine presence/absence of a literal string with `str_detect`. Spoiler: later we see `str_detect` also detects regular expressions.

Which fruits actually use the word “fruit”?


```R
# detect "fruit"
#typeof(fruit)
#fruit
str_detect(fruit, "fruit")
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>TRUE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>TRUE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>TRUE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>TRUE</li><li>FALSE</li><li>FALSE</li><li>TRUE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>TRUE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>TRUE</li><li>FALSE</li><li>FALSE</li><li>FALSE</li><li>TRUE</li><li>FALSE</li></ol>



What’s the easiest way to get the actual fruits that match? Use `str_subset` to keep only the matching elements. Note we are storing this new vector `my_fruit` to use in later examples!


```R
# subset "fruit"
my_fruit <- str_subset(fruit, "fruit")
my_fruit
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'breadfruit'</li><li>'dragonfruit'</li><li>'grapefruit'</li><li>'jackfruit'</li><li>'kiwi fruit'</li><li>'passionfruit'</li><li>'star fruit'</li><li>'ugli fruit'</li></ol>



#### String splitting by delimiter

Use `stringr::str_split` to split strings on a delimiter. 

Some of our fruits are compound words, like “grapefruit”, but some have two words, like “ugli fruit”. Here we split on a single space " ", but show use of a regular expression later.


```R
# split on " "
str_split(my_fruit, " ")
```


<ol>
	<li>'breadfruit'</li>
	<li>'dragonfruit'</li>
	<li>'grapefruit'</li>
	<li>'jackfruit'</li>
	<li><style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'kiwi'</li><li>'fruit'</li></ol>
</li>
	<li>'passionfruit'</li>
	<li><style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'star'</li><li>'fruit'</li></ol>
</li>
	<li><style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'ugli'</li><li>'fruit'</li></ol>
</li>
</ol>



It’s bummer that we get a list back. But it must be so! In full generality, split strings must return list, because who knows how many pieces there will be?

If you are willing to commit to the number of pieces, you can use `str_split_fixed` and get a character matrix. You’re welcome!


```R
str_split_fixed(my_fruit, pattern = " ", n = 2)
```


<table class="dataframe">
<caption>A matrix: 8 × 2 of type chr</caption>
<tbody>
	<tr><td>breadfruit  </td><td>     </td></tr>
	<tr><td>dragonfruit </td><td>     </td></tr>
	<tr><td>grapefruit  </td><td>     </td></tr>
	<tr><td>jackfruit   </td><td>     </td></tr>
	<tr><td>kiwi        </td><td>fruit</td></tr>
	<tr><td>passionfruit</td><td>     </td></tr>
	<tr><td>star        </td><td>fruit</td></tr>
	<tr><td>ugli        </td><td>fruit</td></tr>
</tbody>
</table>



If the to-be-split variable lives in a data frame, `tidyr::separate` will split it into 2 or more variables:


```R
# separate on " "
my_fruit[5] <- "yellow kiwi fruit"
my_fruit
tibble(unsplit = my_fruit) %>% 
    separate(unsplit, into = c("pre", "post"), sep = " ")
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'breadfruit'</li><li>'dragonfruit'</li><li>'grapefruit'</li><li>'jackfruit'</li><li>'yellow kiwi fruit'</li><li>'passionfruit'</li><li>'star fruit'</li><li>'ugli fruit'</li></ol>



    Warning message:
    “Expected 2 pieces. Additional pieces discarded in 1 rows [5].”
    Warning message:
    “Expected 2 pieces. Missing pieces filled with `NA` in 5 rows [1, 2, 3, 4, 6].”



<table class="dataframe">
<caption>A tibble: 8 × 2</caption>
<thead>
	<tr><th scope=col>pre</th><th scope=col>post</th></tr>
	<tr><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th></tr>
</thead>
<tbody>
	<tr><td>breadfruit  </td><td>NA   </td></tr>
	<tr><td>dragonfruit </td><td>NA   </td></tr>
	<tr><td>grapefruit  </td><td>NA   </td></tr>
	<tr><td>jackfruit   </td><td>NA   </td></tr>
	<tr><td>yellow      </td><td>kiwi </td></tr>
	<tr><td>passionfruit</td><td>NA   </td></tr>
	<tr><td>star        </td><td>fruit</td></tr>
	<tr><td>ugli        </td><td>fruit</td></tr>
</tbody>
</table>



#### Substring extraction (and replacement) by position

Count characters in your strings with `str_length`. *Note this is different from the length of the character vector itself.*


```R
# get length of each string
str_length(my_fruit)
my_fruit
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>10</li><li>11</li><li>10</li><li>9</li><li>17</li><li>12</li><li>10</li><li>10</li></ol>




<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'breadfruit'</li><li>'dragonfruit'</li><li>'grapefruit'</li><li>'jackfruit'</li><li>'yellow kiwi fruit'</li><li>'passionfruit'</li><li>'star fruit'</li><li>'ugli fruit'</li></ol>



You can snip out substrings based on character position with `str_sub`.


```R
# remove first three strings
str_sub(my_fruit, 1, 3)
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'bre'</li><li>'dra'</li><li>'gra'</li><li>'jac'</li><li>'yel'</li><li>'pas'</li><li>'sta'</li><li>'ugl'</li></ol>



Finally, `str_sub` also works for assignment, i.e. on the left hand side of `<-`


```R
# replace three characters with AAA
str_sub(my_fruit, 1, 3)  <- "AAA"
my_fruit
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'AAAadfruit'</li><li>'AAAgonfruit'</li><li>'AAApefruit'</li><li>'AAAkfruit'</li><li>'AAAlow kiwi fruit'</li><li>'AAAsionfruit'</li><li>'AAAr fruit'</li><li>'AAAi fruit'</li></ol>



#### Collapse a vector

You can collapse a character vector of length n > 1 to a single string with `str_c`, which also has other uses (see the next section).


```R
# collapse a character vector into one 
head(fruit) %>% 
    str_c(collapse = "-")
```


'apple-apricot-avocado-banana-bell pepper-bilberry'


#### Create a character vector by catenating multiple vectors

If you have two or more character vectors of the same length, you can glue them together element-wise, to get a new vector of that length. Here are some … awful smoothie flavors?


```R
# concatenate character vectors
fruit[1:4]
fruit[5:8]
str_c(fruit[1:4], fruit[5:8], sep = "")
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'apple'</li><li>'apricot'</li><li>'avocado'</li><li>'banana'</li></ol>




<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'bell pepper'</li><li>'bilberry'</li><li>'blackberry'</li><li>'blackcurrant'</li></ol>




<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'applebell pepper'</li><li>'apricotbilberry'</li><li>'avocadoblackberry'</li><li>'bananablackcurrant'</li></ol>



If the to-be-combined vectors are variables in a data frame, you can use `tidyr::unite` to make a single new variable from them.


```R
# concatenate character vectors when they are in a data frame
tibble(fruit1 = fruit[1:4],
      fruit2 = fruit[5:8]) %>% 
    unite("flavour_combo", fruit1, fruit2, sep = " & ")
```


<table class="dataframe">
<caption>A tibble: 4 × 1</caption>
<thead>
	<tr><th scope=col>flavour_combo</th></tr>
	<tr><th scope=col>&lt;chr&gt;</th></tr>
</thead>
<tbody>
	<tr><td><span style=white-space:pre-wrap>apple &amp; bell pepper  </span></td></tr>
	<tr><td><span style=white-space:pre-wrap>apricot &amp; bilberry   </span></td></tr>
	<tr><td>avocado &amp; blackberry </td></tr>
	<tr><td>banana &amp; blackcurrant</td></tr>
</tbody>
</table>



####  Substring replacement

You can replace a pattern with `str_replace`. Here we use an explicit string-to-replace, but later we revisit with a regular expression.


```R
# replace fruit with vegetable
my_fruit <- str_subset(fruit, "fruit")
my_fruit
str_replace(my_fruit, "fruit", "vegetable")
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'breadfruit'</li><li>'dragonfruit'</li><li>'grapefruit'</li><li>'jackfruit'</li><li>'kiwi fruit'</li><li>'passionfruit'</li><li>'star fruit'</li><li>'ugli fruit'</li></ol>




<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'breadvegetable'</li><li>'dragonvegetable'</li><li>'grapevegetable'</li><li>'jackvegetable'</li><li>'kiwi vegetable'</li><li>'passionvegetable'</li><li>'star vegetable'</li><li>'ugli vegetable'</li></ol>



- A special case that comes up a lot is replacing `NA`, for which there is `str_replace_na`.

- If the NA-afflicted variable lives in a data frame, you can use `tidyr::replace_na`.

#### Other `str_*` functions?

There are many many other useful `str_*` functions from the `stringr` package. Too many to go through them all here. If these shown in lecture aren't what you need, then you should try `?str` + tab to see the possibilities:


```R
?str_
```


No documentation for ‘str_’ in specified packages and libraries:
you could try ‘??str_’


#### Regular expressions with stringr

or...

![](https://stat545.com/img/regexbytrialanderror-big-smaller.png)

#### Examples with gapminder


```R
library(gapminder)
head(gapminder)
```


<table class="dataframe">
<caption>A tibble: 6 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>Afghanistan</td><td>Asia</td><td>1952</td><td>28.801</td><td> 8425333</td><td>779.4453</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1957</td><td>30.332</td><td> 9240934</td><td>820.8530</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1962</td><td>31.997</td><td>10267083</td><td>853.1007</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1967</td><td>34.020</td><td>11537966</td><td>836.1971</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1972</td><td>36.088</td><td>13079460</td><td>739.9811</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1977</td><td>38.438</td><td>14880372</td><td>786.1134</td></tr>
</tbody>
</table>



#### Filtering rows with `str_detect`

Let's filter for rows where the country name starts with "AL":


```R
library(tidyverse)
library(gapminder)
```


```R
# detect countries that start with "AL"
gapminder %>% 
    filter(str_detect(country, "^Al")) %>% 
    pull(country) %>% 
    unique() %>% 
    length()
```


2


And now rows where the country ends in `tan`:


```R
# detect countries that end with "tan"
gapminder %>% 
    filter(str_detect(country, "tan$"))
```


<table class="dataframe">
<caption>A tibble: 24 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>Afghanistan</td><td>Asia</td><td>1952</td><td>28.801</td><td> 8425333</td><td>779.4453</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1957</td><td>30.332</td><td> 9240934</td><td>820.8530</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1962</td><td>31.997</td><td>10267083</td><td>853.1007</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1967</td><td>34.020</td><td>11537966</td><td>836.1971</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1972</td><td>36.088</td><td>13079460</td><td>739.9811</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>Pakistan</td><td>Asia</td><td>1987</td><td>58.245</td><td>105186881</td><td>1704.687</td></tr>
	<tr><td>Pakistan</td><td>Asia</td><td>1992</td><td>60.838</td><td>120065004</td><td>1971.829</td></tr>
	<tr><td>Pakistan</td><td>Asia</td><td>1997</td><td>61.818</td><td>135564834</td><td>2049.351</td></tr>
	<tr><td>Pakistan</td><td>Asia</td><td>2002</td><td>63.610</td><td>153403524</td><td>2092.712</td></tr>
	<tr><td>Pakistan</td><td>Asia</td><td>2007</td><td>65.483</td><td>169270617</td><td>2605.948</td></tr>
</tbody>
</table>



Or countries containing ", Dem. Rep." :


```R
# detect countries that contain ", Dem. Rep." 
gapminder %>% 
    filter(str_detect(country, "\\, Dem. Rep."))
```


<table class="dataframe">
<caption>A tibble: 24 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>Congo, Dem. Rep.</td><td>Africa</td><td>1952</td><td>39.143</td><td>14100005</td><td>780.5423</td></tr>
	<tr><td>Congo, Dem. Rep.</td><td>Africa</td><td>1957</td><td>40.652</td><td>15577932</td><td>905.8602</td></tr>
	<tr><td>Congo, Dem. Rep.</td><td>Africa</td><td>1962</td><td>42.122</td><td>17486434</td><td>896.3146</td></tr>
	<tr><td>Congo, Dem. Rep.</td><td>Africa</td><td>1967</td><td>44.056</td><td>19941073</td><td>861.5932</td></tr>
	<tr><td>Congo, Dem. Rep.</td><td>Africa</td><td>1972</td><td>45.989</td><td>23007669</td><td>904.8961</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>Korea, Dem. Rep.</td><td>Asia</td><td>1987</td><td>70.647</td><td>19067554</td><td>4106.492</td></tr>
	<tr><td>Korea, Dem. Rep.</td><td>Asia</td><td>1992</td><td>69.978</td><td>20711375</td><td>3726.064</td></tr>
	<tr><td>Korea, Dem. Rep.</td><td>Asia</td><td>1997</td><td>67.727</td><td>21585105</td><td>1690.757</td></tr>
	<tr><td>Korea, Dem. Rep.</td><td>Asia</td><td>2002</td><td>66.662</td><td>22215365</td><td>1646.758</td></tr>
	<tr><td>Korea, Dem. Rep.</td><td>Asia</td><td>2007</td><td>67.297</td><td>23301725</td><td>1593.065</td></tr>
</tbody>
</table>



Replace ", Dem. Rep." with " Democratic Republic":


```R
# replace ", Dem. Rep." with " Democratic Republic"
gapminder %>% 
    mutate(country = str_replace(country,
                                "\\, Dem. Rep.",
                                " Democratic Republic")) %>%
    filter(country == "Korea Democratic Republic")
```


<table class="dataframe">
<caption>A tibble: 12 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>Korea Democratic Republic</td><td>Asia</td><td>1952</td><td>50.056</td><td> 8865488</td><td>1088.278</td></tr>
	<tr><td>Korea Democratic Republic</td><td>Asia</td><td>1957</td><td>54.081</td><td> 9411381</td><td>1571.135</td></tr>
	<tr><td>Korea Democratic Republic</td><td>Asia</td><td>1962</td><td>56.656</td><td>10917494</td><td>1621.694</td></tr>
	<tr><td>Korea Democratic Republic</td><td>Asia</td><td>1967</td><td>59.942</td><td>12617009</td><td>2143.541</td></tr>
	<tr><td>Korea Democratic Republic</td><td>Asia</td><td>1972</td><td>63.983</td><td>14781241</td><td>3701.622</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>Korea Democratic Republic</td><td>Asia</td><td>1987</td><td>70.647</td><td>19067554</td><td>4106.492</td></tr>
	<tr><td>Korea Democratic Republic</td><td>Asia</td><td>1992</td><td>69.978</td><td>20711375</td><td>3726.064</td></tr>
	<tr><td>Korea Democratic Republic</td><td>Asia</td><td>1997</td><td>67.727</td><td>21585105</td><td>1690.757</td></tr>
	<tr><td>Korea Democratic Republic</td><td>Asia</td><td>2002</td><td>66.662</td><td>22215365</td><td>1646.758</td></tr>
	<tr><td>Korea Democratic Republic</td><td>Asia</td><td>2007</td><td>67.297</td><td>23301725</td><td>1593.065</td></tr>
</tbody>
</table>



### Extract matches (optional)

To extract the actual text of a match, use `str_extract`. 

We’re going to need a more complicated example, the Harvard `sentences` from the `stringr` package: 


```R
typeof(sentences)
head(sentences)
```


'character'



<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'The birch canoe slid on the smooth planks.'</li><li>'Glue the sheet to the dark blue background.'</li><li>'It\'s easy to tell the depth of a well.'</li><li>'These days a chicken leg is a rare dish.'</li><li>'Rice is often served in round bowls.'</li><li>'The juice of lemons makes fine punch.'</li></ol>



Say we want to extract all of the colours used in the sentences. We can do this by creating a pattern which would match them, and passing that and our vector of sentences to `str_extract`:


```R
colours <- "red|orange|yellow|green|blue|purple"
```


```R
# extract colours used in sentences
str_extract(sentences, colours)
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>NA</li><li>'blue'</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>'blue'</li><li>NA</li><li>'red'</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>'red'</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>'red'</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>'blue'</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>'yellow'</li><li>NA</li><li>NA</li><li>NA</li><li>'red'</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>'red'</li><li>NA</li><li>'green'</li><li>'red'</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>'red'</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>'blue'</li><li>'red'</li><li>NA</li><li>'red'</li><li>'red'</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>'red'</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>⋯</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>'red'</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>'red'</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>'green'</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>'green'</li><li>'green'</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>'red'</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>'red'</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>'yellow'</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>'red'</li><li>NA</li><li>'orange'</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>'red'</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>'red'</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>'red'</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li><li>NA</li></ol>



`str_extract` only returns the first match for each element. To return all matches from an element we need to use `str_extract_all`:


```R
# extract all colours used in sentences
str_extract_all(sentences, colours)
```


<ol>
	<li></li>
	<li>'blue'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'blue'</li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'blue'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'yellow'</li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li>'green'</li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'blue'</li>
	<li>'red'</li>
	<li></li>
	<li>'red'</li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'blue'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li><style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'blue'</li><li>'red'</li></ol>
</li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li>'green'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'green'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'green'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'purple'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'green'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'blue'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'blue'</li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'green'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'green'</li>
	<li><style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'green'</li><li>'red'</li></ol>
</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'yellow'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li><style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'orange'</li><li>'red'</li></ol>
</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li>'red'</li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
	<li></li>
</ol>



Note: `str_extract` returns a character vector, whereas `str_extract_all` returns a litst. This is because when asking for multiple matches back, you do not know how many you will get, and thus we cannot expect a rectangular shape.

### Capture groups (optional)

You can also use parentheses to extract parts of a complex match. 

For example, imagine we want to extract nouns from the sentences. As a heuristic, we’ll look for any word that comes after “a” or “the”. Defining a “word” in a regular expression is a little tricky, so here I use a simple approximation: a sequence of at least one character that isn’t a space.


```R
# extract nouns from sentences
noun <- "(a|the) ([^ ]+)"

str_match(sentences, noun) %>% 
    head()
```


<table class="dataframe">
<caption>A matrix: 6 × 3 of type chr</caption>
<tbody>
	<tr><td>the smooth</td><td>the</td><td>smooth </td></tr>
	<tr><td>the sheet </td><td>the</td><td>sheet  </td></tr>
	<tr><td>the depth </td><td>the</td><td>depth  </td></tr>
	<tr><td>a chicken </td><td>a  </td><td>chicken</td></tr>
	<tr><td>NA        </td><td>NA </td><td>NA     </td></tr>
	<tr><td>NA        </td><td>NA </td><td>NA     </td></tr>
</tbody>
</table>



Like `str_extract`, if you want all matches for each string, you’ll need `str_match_all`

### Summary of string manipulation functions covered so far:

| function | description |
|----------|-------------|
| `str_detect` | Detects elements in a vector that match a pattern, returns a vector of logicals |
| `srt_subset` | Detects and returns elements in a vector that match a pattern |
| `str_split` | Split strings in a vector on a delimiter. Returns a list (used `str_split_fixed` to get a matrix) |
| `separate` | Split character vectors from a data frame on a delimiter which get returned as additional columns in the data frame |
| `str_length` | Counts the number of characters for each element of a character vector, and returns a numeric vector of the counts |
| `str_sub` | Remove substrings based on character position |
| `str_c` | Collapse and/or concatenate elements from a character vector(s) |
| `unite` | Concatenate elements from character vectors from a data frame to create a single column |
| `str_replace` | Replace a pattern in a vector of character vectors with a given string |
| `str_extract` | Extract the actual text of a match from a character vector |
| `str_match` | Use capture groups to extract parts of a complex match from a character vector, returns the match and the capture groups as columns of a matrix |

## Factors

<img src="https://d33wubrfki0l68.cloudfront.net/baa19d0ebf9b97949a7ad259b29a1c4ae031c8e2/8e9b8/diagrams/vectors/summary-tree-s3-1.png" width=300>

*Source: [Advanced R](https://adv-r.hadley.nz/) by Hadley Wickham*

### Be the boss of your factors

- I love and hate factors

- I love them for data visualization and statistics because I do not need to make dummy variables 

- I hate them because if you are not careful, they fool you because they look like character vectors. And when you treat them like character vectors you get cryptic error messages, like we saw when we tried to do a conditional mutate on the gapminder data set

#### Tidyverse philosophy for factors

- Humans, not computers should decide which columns are factors

- Factors are not that useful until you are at the end of your data wrangling, before that you want character vectors so you can do string manipulations

- Tidyverse functions, like `tibble`, and `read_csv` give you columns with strings as character vectors, Base R functions like `data.frame` and `read.csv`

### Factor inspection

Get to know your factor before you start touching it! It’s polite. Let’s use `gapminder$continent` as our example.


```R
str(gapminder$continent)
```

     Factor w/ 5 levels "Africa","Americas",..: 3 3 3 3 3 3 3 3 3 3 ...



```R
levels(gapminder$continent)
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'Africa'</li><li>'Americas'</li><li>'Asia'</li><li>'Europe'</li><li>'Oceania'</li></ol>




```R
nlevels(gapminder$continent)
```


5



```R
class(gapminder$continent)
```


'factor'


### Dropping unused levels

Just because you drop all the rows corresponding to a specific factor level, the levels of the factor itself do not change. Sometimes all these unused levels can come back to haunt you later, e.g., in figure legends.

Watch what happens to the levels of `country` when we filter Gapminder to a handful of countries:


```R
nlevels(gapminder$country)
```


142



```R
h_countries <- gapminder %>% 
    filter(country %in% c("Egypt", "Haiti", "Romania", "Thailand", "Venezuela"))

nlevels(h_countries$country)
```


142


huh? Even though `h_gap` only has data for a handful of countries, we are still schlepping around all 142 levels from the original `gapminder` tibble.

How to get rid of them? We'll use the `forcats::fct_drop` function to do this:


```R
h_countries$country %>% nlevels()
```


142



```R
h_countries$country %>% 
    fct_drop() %>% 
    nlevels
```


5


### Change order of the levels, principled

By default, factor levels are ordered alphabetically. Which might as well be random, when you think about it! It is preferable to order the levels according to some principle:

- Frequency. Make the most common level the first and so on.
- Another variable. Order factor levels according to a summary statistic for another variable. Example: order Gapminder countries by life expectancy.

First, let’s order continent by frequency, forwards and backwards. This is often a great idea for tables and figures, esp. frequency barplots.


```R
## default order is alphabetical
gapminder$continent %>% 
    levels()
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'Africa'</li><li>'Americas'</li><li>'Asia'</li><li>'Europe'</li><li>'Oceania'</li></ol>



Let's use `forcats::fct_infreq` to order by frequency:


```R
gapminder$continent %>% 
    fct_infreq() %>% 
    levels()

gap2 <- gapminder %>% 
    mutate(continent = fct_infreq(continent))

gap2$continent %>% 
    levels()
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'Africa'</li><li>'Asia'</li><li>'Europe'</li><li>'Americas'</li><li>'Oceania'</li></ol>




<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'Africa'</li><li>'Asia'</li><li>'Europe'</li><li>'Americas'</li><li>'Oceania'</li></ol>



Or reverse frequency:


```R
gapminder$continent %>% 
  fct_infreq() %>%
  fct_rev() %>% 
  levels()
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'Oceania'</li><li>'Americas'</li><li>'Europe'</li><li>'Asia'</li><li>'Africa'</li></ol>



### Order one variable by another

You can use `forcats::fct_reorder` to order one variable by another.

The factor is the grouping variable and the default summarizing function is `median` but you can specify something else.


```R
## order countries by median life expectancy
fct_reorder(gapminder$country, gapminder$lifeExp) %>% 
    levels() %>% 
    head()
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'Sierra Leone'</li><li>'Guinea-Bissau'</li><li>'Afghanistan'</li><li>'Angola'</li><li>'Somalia'</li><li>'Guinea'</li></ol>



Using `min` instead to reorder the factors:


```R
## order accoring to minimum life exp instead of median
fct_reorder(gapminder$country, gapminder$lifeExp, min) %>% 
    levels() %>% 
    head()
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'Rwanda'</li><li>'Afghanistan'</li><li>'Gambia'</li><li>'Angola'</li><li>'Sierra Leone'</li><li>'Cambodia'</li></ol>



### Change order of the levels, “because I said so”

Sometimes you just want to hoist one or more levels to the front. Why? Because I said so (sometimes really useful when creating visualizations).

Reminding ourselves of the level order for `gapminder$continent`:


```R
gapminder$continent %>% levels()
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'Africa'</li><li>'Americas'</li><li>'Asia'</li><li>'Europe'</li><li>'Oceania'</li></ol>



Reorder and put Asia and Africa first:


```R
gapminder$continent %>% 
    fct_relevel("Asia", "Africa")  %>% 
    levels()
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>'Asia'</li><li>'Africa'</li><li>'Americas'</li><li>'Europe'</li><li>'Oceania'</li></ol>



#### Why do we need to know how to do this?

- Factor levels impact statistical analysis & data visualization!

- For example, these two barcharts of frequency by continent differ only in the order of the continents. Which do you prefer? Discuss with your neighbour.

![](img/factor_viz.png)

## What did we learn today?

- The differences between data frames and tibbles

- The beautiful {lubridate} package for working with dates and times

- Tools for manipulating and working with character data in the {stringr} and {tidyr} packages

- How to take control of our factors using {forcats} and how to investigate factors using base R functions

## Attributions
- [Stat 545](https://stat545.com/) created by Jenny Bryan
- [R for Data Science](https://r4ds.had.co.nz/index.html) by Garrett Grolemund & Hadley Wickham
