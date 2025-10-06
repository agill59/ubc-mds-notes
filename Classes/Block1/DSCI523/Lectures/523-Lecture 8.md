# Lecture 8: Tidy evaluation

### Lecture learning objectives:

By then end of this lecture & worksheet 8, students should be able to:
* Describe data masking as it relates to the `dplyr` functions. Explain the problems it solves for interactive programming and the problems it creates for programming in a non-interactive setting
* Explain what the `enquo()` function and the `!!` operator do in R in the context of data masking as it relates to the `dplyr` functions
* Use the `{{` (read: curly curly) operator (abstracts quote-and-unquote into a single interpolation step), the `:=` (read: walrus) operatpr in R, and `...` (read: pass the dots) to write functions which wrap the `dplyr` functions



```R
library(gapminder)
library(tidyverse)
options(repr.matrix.max.rows = 5)
```

### What Metaprogramming lets you do in R

- write `library(purrr)` instead of `library("purrr")`
- enable `plot(x, sin(x))` to automatically label the axes with `x` and `sin(x)`
- create a model object via `lm(y ~ x1 + x2, data = df)`
- and much much more (that you will see in Data Wrangling as we explore the tidyverse)

#### What is metaprogramming?

Code that writes code/code that mutates code.

#### Our narrow focus on metaprogramming for this course:

Tidy evaluation

#### Why focus on tidy evaluation

In the rest of MDS you will be relying on functions from the tidyverse to do a lot of:
- data wrangling
- statistics
- data visualization

## Tidy evaluation

The functions from the tidyverse are beautiful to use interactively.


```R
gapminder
```


<table class="dataframe">
<caption>A tibble: 1704 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>Afghanistan</td><td>Asia</td><td>1952</td><td>28.801</td><td> 8425333</td><td>779.4453</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1957</td><td>30.332</td><td> 9240934</td><td>820.8530</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1962</td><td>31.997</td><td>10267083</td><td>853.1007</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>2002</td><td>39.989</td><td>11926563</td><td>672.0386</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>2007</td><td>43.487</td><td>12311143</td><td>469.7093</td></tr>
</tbody>
</table>



with base r:


```R
gapminder[gapminder$country == "Canada" & gapminder$year == 1952, ]
```


<table class="dataframe">
<caption>A tibble: 1 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>Canada</td><td>Americas</td><td>1952</td><td>68.75</td><td>14785584</td><td>11367.16</td></tr>
</tbody>
</table>



In the tidyverse:


```R
filter(gapminder, country == "Canada", year == 1952)
```


<table class="dataframe">
<caption>A tibble: 1 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>Canada</td><td>Americas</td><td>1952</td><td>68.75</td><td>14785584</td><td>11367.16</td></tr>
</tbody>
</table>



#### How does that even work?

- When functions like `filter` are called, there is a delay in evaluation and the data frame is temporarily promoted as first class objects, we say the data masks the workspace

- This is to allow the promotion of the data frame, such that it masks the workspace (global environment) 

- When this happens, R can then find the relevant columns for the computation

*This is referred to as data masking*

#### Back to our example:

What is going on here?

- code evaluation is delayed 
- the `filter` function quotes columns `country` and `year`
- the `filter` function then creates a data mask (to mingle variables from the environment and the data frame)
- the columns `country` and `year` and unquoted and evaluated within the data mask


```R
filter(gapminder, country == "Canada", year == 1952)
```


<table class="dataframe">
<caption>A tibble: 1 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>Canada</td><td>Americas</td><td>1952</td><td>68.75</td><td>14785584</td><td>11367.16</td></tr>
</tbody>
</table>



#### Trade off of lovely interactivity of tidyverse functions...

#### programming with them can be more challenging.

Let's try writing a function which wraps filter for gapminder:

```
filter_gap <- function(col, val) {
    filter(gapminder, col == val)
}

filter_gap(country, "Canada")
```

```
Error: object 'country' not found
Traceback:

1. filter_gap(country, "Canada")
2. filter(gapminder, col == val)   # at line 4 of file <text>
3. filter.tbl_df(gapminder, col == val)
4. filter_impl(.data, quo)

```

Why does `filter` work with non-quoted variable names, but our function `filter_gap` fail?

### Defining functions using tidy eval's `enquo` and `!!`:

Use `enquo` to quote the column names, and then `!!` to unquote them in context.


```R
filter_gap <- function(col, val) {
    col <- enquo(col)
    filter(gapminder, !!col == val)
}

filter_gap(country, "Canada")
```


<table class="dataframe">
<caption>A tibble: 12 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>Canada</td><td>Americas</td><td>1952</td><td>68.75</td><td>14785584</td><td>11367.16</td></tr>
	<tr><td>Canada</td><td>Americas</td><td>1957</td><td>69.96</td><td>17010154</td><td>12489.95</td></tr>
	<tr><td>Canada</td><td>Americas</td><td>1962</td><td>71.30</td><td>18985849</td><td>13462.49</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>Canada</td><td>Americas</td><td>2002</td><td>79.770</td><td>31902268</td><td>33328.97</td></tr>
	<tr><td>Canada</td><td>Americas</td><td>2007</td><td>80.653</td><td>33390141</td><td>36319.24</td></tr>
</tbody>
</table>



### Defining functions by embracing column names: `{{ }}`

- In the newest release of `rlang`, there has been the introduction of the `{{` (pronounced "curly curly") operator.

- Does the same thing as `enquo` and `!!` but (hopefully) easier to use.


```R
filter_gap <- function(col, val) {
    filter(gapminder, {{col}} == val)
}

filter_gap(country, "Canada")
```


<table class="dataframe">
<caption>A tibble: 12 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>Canada</td><td>Americas</td><td>1952</td><td>68.75</td><td>14785584</td><td>11367.16</td></tr>
	<tr><td>Canada</td><td>Americas</td><td>1957</td><td>69.96</td><td>17010154</td><td>12489.95</td></tr>
	<tr><td>Canada</td><td>Americas</td><td>1962</td><td>71.30</td><td>18985849</td><td>13462.49</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>Canada</td><td>Americas</td><td>2002</td><td>79.770</td><td>31902268</td><td>33328.97</td></tr>
	<tr><td>Canada</td><td>Americas</td><td>2007</td><td>80.653</td><td>33390141</td><td>36319.24</td></tr>
</tbody>
</table>



### (OPTIONAL) Creating functions that handle column names as strings:

Sometimes you want to pass a column name into a function as a string (often useful when you are programming and have the column names as a character vector).

You can do this by using symbols + unquoting with `sym` + `!!` :


```R
# example of what we want to wrap: filter(gapminder, country == "Canada")
filter_gap <- function(col, val) {
    col <- sym(col)
    filter(gapminder, !!col == val)
}

filter_gap("country", "Canada")
```


<table class="dataframe">
<caption>A tibble: 12 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>Canada</td><td>Americas</td><td>1952</td><td>68.75</td><td>14785584</td><td>11367.16</td></tr>
	<tr><td>Canada</td><td>Americas</td><td>1957</td><td>69.96</td><td>17010154</td><td>12489.95</td></tr>
	<tr><td>Canada</td><td>Americas</td><td>1962</td><td>71.30</td><td>18985849</td><td>13462.49</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>Canada</td><td>Americas</td><td>2002</td><td>79.770</td><td>31902268</td><td>33328.97</td></tr>
	<tr><td>Canada</td><td>Americas</td><td>2007</td><td>80.653</td><td>33390141</td><td>36319.24</td></tr>
</tbody>
</table>



### The walrus operator `:=` is needed when assigning values
- `:=` is needed when addinging values with tidyevaluation


```R
group_summary <- function(data, group, col, fun) {
    data %>% 
        group_by({{ group }}) %>% 
        summarise( {{ col }} := fun({{ col }}))
}

group_summary(gapminder, continent, gdpPercap, mean)
```


<table class="dataframe">
<caption>A tibble: 5 × 2</caption>
<thead>
	<tr><th scope=col>continent</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>Africa  </td><td> 2193.755</td></tr>
	<tr><td>Americas</td><td> 7136.110</td></tr>
	<tr><td>Asia    </td><td> 7902.150</td></tr>
	<tr><td>Europe  </td><td>14469.476</td></tr>
	<tr><td>Oceania </td><td>18621.609</td></tr>
</tbody>
</table>



### Pass the dots when you can

If you are only passing on variable to a tidyverse function, and that variable is not used in logical comparisons, or in variable assignment, you can get away with passing the dots:


```R
sort_gap <- function(...) {
    arrange(gapminder, ...)
}

sort_gap(year)
```


<table class="dataframe">
<caption>A tibble: 1704 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>Afghanistan</td><td>Asia  </td><td>1952</td><td>28.801</td><td>8425333</td><td> 779.4453</td></tr>
	<tr><td>Albania    </td><td>Europe</td><td>1952</td><td>55.230</td><td>1282697</td><td>1601.0561</td></tr>
	<tr><td>Algeria    </td><td>Africa</td><td>1952</td><td>43.077</td><td>9279525</td><td>2449.0082</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>Zambia  </td><td>Africa</td><td>2007</td><td>42.384</td><td>11746035</td><td>1271.2116</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>2007</td><td>43.487</td><td>12311143</td><td> 469.7093</td></tr>
</tbody>
</table>



#### Notes on passing the dots

- the dots should be the last function argument (or you will not be able to use positional arguments)
- they are useful because you can add multiple arguments

For example:

```
sort_gap <- function(..., x) {
    print(x + 1)
    arrange(gapminder, ...)
}

sort_gap(year, continent, country, 2)
```

```
Error in print(x + 1): argument "x" is missing, with no default
Traceback:

1. sort_gap(year, continent, country, 2)
2. print(x + 1)   # at line 2 of file <text>
```


```R
sort_gap <- function(x, ...) {
    print(x + 1)
    arrange(gapminder, ...)
}

sort_gap(1, year, continent, country)
```

    [1] 2
    


<table class="dataframe">
<caption>A tibble: 1704 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>Algeria</td><td>Africa</td><td>1952</td><td>43.077</td><td>9279525</td><td>2449.008</td></tr>
	<tr><td>Angola </td><td>Africa</td><td>1952</td><td>30.015</td><td>4232095</td><td>3520.610</td></tr>
	<tr><td>Benin  </td><td>Africa</td><td>1952</td><td>38.223</td><td>1738315</td><td>1062.752</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>Australia  </td><td>Oceania</td><td>2007</td><td>81.235</td><td>20434176</td><td>34435.37</td></tr>
	<tr><td>New Zealand</td><td>Oceania</td><td>2007</td><td>80.204</td><td> 4115771</td><td>25185.01</td></tr>
</tbody>
</table>



#### Pass the dots is not always the solution...

```
square_diff_n_select <- function(data, ...) {
    data %>% 
        mutate(... := (... - mean(...))^2) %>% 
        select(...)
}

square_diff_n_select(mtcars, mpg, mpg:hp)
```

```
Error: Problem with `mutate()` input `...`.
✖ object 'hp' not found
ℹ Input `...` is `(... - mean(...))^2`.
Traceback:

1. square_diff_n_select(mtcars, mpg, mpg:hp)
2. data %>% mutate(`:=`(..., (... - mean(...))^2)) %>% select(...)   # at line 2-4 of file <text>
3. withVisible(eval(quote(`_fseq`(`_lhs`)), env, env))
4. eval(quote(`_fseq`(`_lhs`)), env, env)
5. eval(quote(`_fseq`(`_lhs`)), env, env)
6. `_fseq`(`_lhs`)
7. freduce(value, `_function_list`)
8. function_list[[i]](value)
9. mutate(., `:=`(..., (... - mean(...))^2))
10. mutate.data.frame(., `:=`(..., (... - mean(...))^2))
11. mutate_cols(.data, ...)
12. withCallingHandlers({
  .     for (i in seq_along(dots)) {
  .         not_named <- (is.null(dots_names) || dots_names[i] == 
  .             "")
  ...

 ```

### When passing in different column names to different functions, embrace multiple column names


```R
square_diff_n_select <- function(data, col_to_change, col_range) {
    data %>% 
        mutate({{ col_to_change }} := ({{ col_to_change }} - mean({{ col_to_change }}))^2) %>% 
        select({{col_range}})
}

square_diff_n_select(mtcars, mpg, mpg:hp)
```


<table class="dataframe">
<caption>A data.frame: 32 × 4</caption>
<thead>
	<tr><th></th><th scope=col>mpg</th><th scope=col>cyl</th><th scope=col>disp</th><th scope=col>hp</th></tr>
	<tr><th></th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>Mazda RX4</th><td>0.8269629</td><td>6</td><td>160</td><td>110</td></tr>
	<tr><th scope=row>Mazda RX4 Wag</th><td>0.8269629</td><td>6</td><td>160</td><td>110</td></tr>
	<tr><th scope=row>Datsun 710</th><td>7.3407129</td><td>4</td><td>108</td><td> 93</td></tr>
	<tr><th scope=row>⋮</th><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><th scope=row>Maserati Bora</th><td>25.914463</td><td>8</td><td>301</td><td>335</td></tr>
	<tr><th scope=row>Volvo 142E</th><td> 1.714463</td><td>4</td><td>121</td><td>109</td></tr>
</tbody>
</table>



#### Combining embracing with pass the dots:


```R
square_diff_n_select <- function(data, col_to_change, ...) {
    data %>% 
        mutate({{ col_to_change }} := ({{ col_to_change }} - mean({{ col_to_change }}))^2) %>% 
        select(..., {{ col_to_change }})
}

square_diff_n_select(mtcars, mpg, drat, carb)
```


<table class="dataframe">
<caption>A data.frame: 32 × 3</caption>
<thead>
	<tr><th></th><th scope=col>drat</th><th scope=col>carb</th><th scope=col>mpg</th></tr>
	<tr><th></th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>Mazda RX4</th><td>3.90</td><td>4</td><td>0.8269629</td></tr>
	<tr><th scope=row>Mazda RX4 Wag</th><td>3.90</td><td>4</td><td>0.8269629</td></tr>
	<tr><th scope=row>Datsun 710</th><td>3.85</td><td>1</td><td>7.3407129</td></tr>
	<tr><th scope=row>⋮</th><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><th scope=row>Maserati Bora</th><td>3.54</td><td>8</td><td>25.914463</td></tr>
	<tr><th scope=row>Volvo 142E</th><td>4.11</td><td>2</td><td> 1.714463</td></tr>
</tbody>
</table>



## Programming defensively with tidy evaluation

You can embrace `{{` the column names in an `if` + `stop` statement to check user input when unquoted column names are used as function arguments.

First, we demonstrate how to check if a column is numeric using `DATA_FRAME  %>% pull({{ COLUMN_NAME }})` to access the column:


```R
check_if_numeric <- function(data, col) {
    is.numeric(data %>% pull({{ col }}))
}

check_if_numeric(gapminder, pop)
```


TRUE


Next, we add a `if` + `stop` to this, to throw an error in our `square_diff_n_select` function when the column type is not what our function is designed to handle. Here our function works, as the `lifeExp` column in the `gapminder` data set is numeric.


```R
square_diff_n_select <- function(data, col_to_change, ...) {
    if (!is.numeric(data %>% pull({{ col_to_change }}))) {
        stop('col_to_change must be numeric')
    }
    
    data %>% 
        mutate({{ col_to_change }} := ({{ col_to_change }} - mean({{ col_to_change }}))^2) %>% 
        select(..., {{ col_to_change }})
}

square_diff_n_select(gapminder, lifeExp, country, year)
```


<table class="dataframe">
<caption>A tibble: 1704 × 3</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>year</th><th scope=col>lifeExp</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>Afghanistan</td><td>1952</td><td>940.8599</td></tr>
	<tr><td>Afghanistan</td><td>1957</td><td>849.2818</td></tr>
	<tr><td>Afghanistan</td><td>1962</td><td>755.0097</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>Zimbabwe</td><td>2002</td><td>379.6823</td></tr>
	<tr><td>Zimbabwe</td><td>2007</td><td>255.5982</td></tr>
</tbody>
</table>



 Here our function throws an error, as the `continent` column in the `gapminder` data set is **not** numeric.

```
square_diff_n_select(gapminder, continent, country, year)
```

```
Error in square_diff_n_select(gapminder, continent, country, year): col_to_change must be numeric
Traceback:

1. square_diff_n_select(gapminder, continent, country, year)
2. stop("col_to_change must be numeric")   # at line 3 of file <text>
```

## What did we learn?
- data masking and its role in tidy evaluation
- programming with tidy-evaluated functions by embracing column names `{{ }}`
- the walrus `:=` operator for assignment when programming with tidy-evaluated functions
- more useful examples of pass the dots `...`

## Attribution:

- [Tidy evaluation](https://tidyeval.tidyverse.org/) by Lionel Henry & Hadley Wickham
- [Tidy eval in context](https://speakerdeck.com/jennybc/tidy-eval-in-context)  talk by Jenny Bryan
- [Programming in the tidyverse](https://dplyr.tidyverse.org/articles/programming.html) 
- [Advanced R](https://adv-r.hadley.nz/) by Hadley Wickham
