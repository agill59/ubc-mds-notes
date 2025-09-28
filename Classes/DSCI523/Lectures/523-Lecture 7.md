# Lecture 7: Mapping and nested data frames

### Lecture learning objectives:

By then end of this lecture & worksheet 7, students should be able to:

* Write and use anonymous functions in R in isolation and in combination the functional {purrr} `map`
* (OPTIONAL) Understand the general ideas of the `map*`, `map2*` and `pmap*` variant functions in the {purrr} package and the six types of output options (list, double, integer, logical, character and data frame).
* Create and modify nested data frames using {dplyr} `group_by` + {tidyr} `nest` and {purrr} `map*` functions
* Create unnested data frames using {tidyr} `unnest` 
* Describe situations where nested data frames are useful 


```R
library(gapminder)
library(infer)
library(tidyverse)
options(repr.matrix.max.rows = 10)
```

## More functional style programming in R with `purrr`


![](https://ih1.redbubble.net/image.329884292.2339/sticker,375x360-bg,ffffff.u1.png)

https://purrr.tidyverse.org/

### Passing arguments to functions you are mapping

- What if you need to pass arguments to the functions you are mapping, in some cases you can take advantage of `...` (e.g., `na.rm = TRUE`) but if things get more complex than that, anonymous functions are your friends. 

- For example, imagine mapping `str_replace` to all the columns of a very wide and nested data frame to replace all instances of `"Cdn"` with `"Canadian"` to fix a data entry error that occurs in several columns in a data set that occur in a nested list column. Here we need to provide the arguments `pattern` and `replacement` in addition to the data we are operating on.

- This is probably the only time I use anonymous functions regularly.

#### Anonymous functions in R

General format: `function(x) body_of_function`
 
 To use one in the global environment, outside of another function call, you do the following:


```R
(function(x) x + 1)(1)
```


2


Above the function takes in x as an argument and adds one to it. The function definition is surrounded by round brackets, as is the value being passed to the anonymous function.

#### Back to anonymous function calls within {purrr} `map_*`

Long form:

```
map_*(data, function(arg) function_being_called(arg, other_arg))
```


Short form:

```
map_*(data, ~ function_being_called(., other_arg))
```

In the shortcut we replace `function(arg)` with a `~` and replace the `arg` in the function call with a `.`

#### Example:

Map `str_replace` to all the columns of a very wide data frame to replace all instances of `"Cdn"` with `"Canadian"` to fix a data entry error that occurs in several columns in a data set. 


```R
data_entry <- tibble(id = c("25323", "45234", "23471"),
                    birth_citizenship = c("Canadian", "American", "Cdn"),
                    current_citizenship = c("Canadian", "Vietnamese", "Cdn"))
data_entry
```


<table class="dataframe">
<caption>A tibble: 3 × 3</caption>
<thead>
	<tr><th scope=col>id</th><th scope=col>birth_citizenship</th><th scope=col>current_citizenship</th></tr>
	<tr><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th></tr>
</thead>
<tbody>
	<tr><td>25323</td><td>Canadian</td><td>Canadian  </td></tr>
	<tr><td>45234</td><td>American</td><td>Vietnamese</td></tr>
	<tr><td>23471</td><td>Cdn     </td><td>Cdn       </td></tr>
</tbody>
</table>



Using verbose anonymous function syntax:


```R
map_df(data_entry, function(vect) str_replace(vect, pattern = "Cdn", replacement = "Canadian"))
```


<table class="dataframe">
<caption>A tibble: 3 × 3</caption>
<thead>
	<tr><th scope=col>id</th><th scope=col>birth_citizenship</th><th scope=col>current_citizenship</th></tr>
	<tr><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th></tr>
</thead>
<tbody>
	<tr><td>25323</td><td>Canadian</td><td>Canadian  </td></tr>
	<tr><td>45234</td><td>American</td><td>Vietnamese</td></tr>
	<tr><td>23471</td><td>Canadian</td><td>Canadian  </td></tr>
</tbody>
</table>



Using shorthand anonymous function syntax:


```R
map_df(data_entry, ~str_replace(., pattern = "Cdn", replacement = "Canadian"))
```


<table class="dataframe">
<caption>A tibble: 3 × 3</caption>
<thead>
	<tr><th scope=col>id</th><th scope=col>birth_citizenship</th><th scope=col>current_citizenship</th></tr>
	<tr><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th></tr>
</thead>
<tbody>
	<tr><td>25323</td><td>Canadian</td><td>Canadian  </td></tr>
	<tr><td>45234</td><td>American</td><td>Vietnamese</td></tr>
	<tr><td>23471</td><td>Canadian</td><td>Canadian  </td></tr>
</tbody>
</table>



Acknowledgement: here you do not need to use an anonymous function, you can take advantage of `...` but we will see more complicated cases later in the lecture and in the lab. Complicated cases arise when you are working with nested data frames.


```R
map_df(data_entry, str_replace, pattern = "Cdn", replacement = "Canadian")
```


<table class="dataframe">
<caption>A tibble: 3 × 3</caption>
<thead>
	<tr><th scope=col>id</th><th scope=col>birth_citizenship</th><th scope=col>current_citizenship</th></tr>
	<tr><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th></tr>
</thead>
<tbody>
	<tr><td>25323</td><td>Canadian</td><td>Canadian  </td></tr>
	<tr><td>45234</td><td>American</td><td>Vietnamese</td></tr>
	<tr><td>23471</td><td>Canadian</td><td>Canadian  </td></tr>
</tbody>
</table>



## (OPTIONAL) Mapping with > 1 data objects

What if the function you want to map takes in > 1 data objects?

`map2*` and `pmap*` are your friends here!

#### `purrr::map2*`

```
map2*(.x, .y, .f, ...)
```

Above reads as: `for` every element of `.x` and `.y` apply `.f` 

Or picture as...

<img src="img/minis_map2_data.png" width=700>

*Source: [purrr workshop](https://speakerdeck.com/jennybc/purrr-workshop) by Jenny Bryan*

<img src="img/minis_map2.png" width=700>

*Source: [purrr workshop](https://speakerdeck.com/jennybc/purrr-workshop) by Jenny Bryan*

#### `purrr::map2_df` example:


For example, say you want to calculate a weighted means (using `weighted.mean`) for columns of a data frame where you had another data frame containing those weights.

Let's make some data:


```R
data <- tibble(frequency = runif(10),
             loudness = runif(10),
               power = runif(10),
              rating = rpois(10, 5) + 1,
                year = rpois(10, 5) + 1999)
data[1, 1] <- NA
data
```


<table class="dataframe">
<caption>A tibble: 10 × 5</caption>
<thead>
	<tr><th scope=col>frequency</th><th scope=col>loudness</th><th scope=col>power</th><th scope=col>rating</th><th scope=col>year</th></tr>
	<tr><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>        NA</td><td>0.8971602</td><td>0.09913619</td><td>6</td><td>2004</td></tr>
	<tr><td>0.07389762</td><td>0.9885117</td><td>0.13073881</td><td>3</td><td>2003</td></tr>
	<tr><td>0.20319018</td><td>0.4464637</td><td>0.40757000</td><td>5</td><td>2002</td></tr>
	<tr><td>0.40116953</td><td>0.2585920</td><td>0.77210928</td><td>4</td><td>2001</td></tr>
	<tr><td>0.61199899</td><td>0.2772221</td><td>0.50574766</td><td>5</td><td>2004</td></tr>
	<tr><td>0.97175302</td><td>0.4691002</td><td>0.95197273</td><td>7</td><td>2001</td></tr>
	<tr><td>0.21701388</td><td>0.8217019</td><td>0.23764613</td><td>8</td><td>2007</td></tr>
	<tr><td>0.33370896</td><td>0.8287219</td><td>0.20251747</td><td>2</td><td>2009</td></tr>
	<tr><td>0.06301434</td><td>0.5701445</td><td>0.71843132</td><td>8</td><td>2005</td></tr>
	<tr><td>0.40508690</td><td>0.1965594</td><td>0.24950980</td><td>6</td><td>2003</td></tr>
</tbody>
</table>




```R
library(dplyr, quietly = TRUE)
data <- tibble(x1 = runif(10),
               x2 = runif(10),
               x3 = runif(10))
data[1, 1] <- NA
weights <- tibble(x1 = rpois(10, 5) + 1,
                 x2 = rpois(10, 5) + 1,
                 x3 = rpois(10, 5) + 1,)

data
weights
```


<table class="dataframe">
<caption>A tibble: 10 × 3</caption>
<thead>
	<tr><th scope=col>x1</th><th scope=col>x2</th><th scope=col>x3</th></tr>
	<tr><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>        NA</td><td>0.99335250</td><td>0.50502241</td></tr>
	<tr><td>0.70825947</td><td>0.57027714</td><td>0.44712861</td></tr>
	<tr><td>0.56335713</td><td>0.00521319</td><td>0.03118581</td></tr>
	<tr><td>0.87800334</td><td>0.17278128</td><td>0.22822493</td></tr>
	<tr><td>0.45827125</td><td>0.72443217</td><td>0.37442013</td></tr>
	<tr><td>0.39525061</td><td>0.42935012</td><td>0.22785951</td></tr>
	<tr><td>0.21861254</td><td>0.05073587</td><td>0.88371480</td></tr>
	<tr><td>0.07253473</td><td>0.84117550</td><td>0.02669344</td></tr>
	<tr><td>0.08383891</td><td>0.68849135</td><td>0.25799151</td></tr>
	<tr><td>0.93908425</td><td>0.71357396</td><td>0.62029109</td></tr>
</tbody>
</table>




<table class="dataframe">
<caption>A tibble: 10 × 3</caption>
<thead>
	<tr><th scope=col>x1</th><th scope=col>x2</th><th scope=col>x3</th></tr>
	<tr><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td> 6</td><td>8</td><td>6</td></tr>
	<tr><td> 1</td><td>8</td><td>3</td></tr>
	<tr><td> 6</td><td>8</td><td>5</td></tr>
	<tr><td> 7</td><td>8</td><td>5</td></tr>
	<tr><td> 4</td><td>8</td><td>4</td></tr>
	<tr><td> 5</td><td>8</td><td>6</td></tr>
	<tr><td> 7</td><td>5</td><td>5</td></tr>
	<tr><td> 6</td><td>6</td><td>7</td></tr>
	<tr><td> 5</td><td>9</td><td>8</td></tr>
	<tr><td>11</td><td>4</td><td>3</td></tr>
</tbody>
</table>



### `purrr::map2_df` example:

Let's use `map2_df` to calculate the weighted mean using these two data frames.


```R
?weighted.mean
```



<table style="width: 100%;"><tr><td>weighted.mean {stats}</td><td style="text-align: right;">R Documentation</td></tr></table>

<h2>Weighted Arithmetic Mean</h2>

<h3>Description</h3>

<p>Compute a weighted mean.
</p>


<h3>Usage</h3>

<pre><code class='language-R'>weighted.mean(x, w, ...)

## Default S3 method:
weighted.mean(x, w, ..., na.rm = FALSE)
</code></pre>


<h3>Arguments</h3>

<table>
<tr style="vertical-align: top;"><td><code>x</code></td>
<td>
<p>an object containing the values whose weighted mean is to be
computed.</p>
</td></tr>
<tr style="vertical-align: top;"><td><code>w</code></td>
<td>
<p>a numerical vector of weights the same length as <code>x</code> giving
the weights to use for elements of <code>x</code>.</p>
</td></tr>
<tr style="vertical-align: top;"><td><code>...</code></td>
<td>
<p>arguments to be passed to or from methods.</p>
</td></tr>
<tr style="vertical-align: top;"><td><code>na.rm</code></td>
<td>
<p>a logical value indicating whether <code>NA</code>
values in <code>x</code> should be stripped before the computation proceeds.</p>
</td></tr>
</table>


<h3>Details</h3>

<p>This is a generic function and methods can be defined for the first
argument <code>x</code>: apart from the default methods there are methods
for the date-time classes <code>"POSIXct"</code>, <code>"POSIXlt"</code>,
<code>"difftime"</code> and <code>"Date"</code>.  The default method will work for
any numeric-like object for which <code>[</code>, multiplication, division
and <code>sum</code> have suitable methods, including complex vectors.
</p>
<p>If <code>w</code> is missing then all elements of <code>x</code> are given the
same weight, otherwise the weights 
are normalized to sum to one (if possible: if
their sum is zero or infinite the value is likely to be <code>NaN</code>).
</p>
<p>Missing values in <code>w</code> are not handled specially and so give a
missing value as the result.  However, zero weights <em>are</em> handled
specially and the corresponding <code>x</code> values are omitted from the
sum.
</p>


<h3>Value</h3>

<p>For the default method, a length-one numeric vector.
</p>


<h3>See Also</h3>

<p><code>mean</code>
</p>


<h3>Examples</h3>

<pre><code class='language-R'>## GPA from Siegel 1994
wt &lt;- c(5,  5,  4,  1)/15
x &lt;- c(3.7,3.3,3.5,2.8)
xm &lt;- weighted.mean(x, wt)
</code></pre>

<hr /><div style="text-align: center;">[Package <em>stats</em> version 4.2.1 ]</div>
</div>


Now that we know how to use `weighted.mean`, let's map it! (and use `na.rm = TRUE` to deal with NA's)


```R
map2_df(data, weights, weighted.mean, na.rm = TRUE)
```


<table class="dataframe">
<caption>A tibble: 1 × 3</caption>
<thead>
	<tr><th scope=col>x1</th><th scope=col>x2</th><th scope=col>x3</th></tr>
	<tr><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>0.5145843</td><td>0.5210375</td><td>0.328147</td></tr>
</tbody>
</table>



Not too bad eh! 

#### `purrr::map2*`

Also, if `y` has less elements than `x`, it recycles `y`:

<img src="https://d33wubrfki0l68.cloudfront.net/55032525ec77409e381dcd200a47e1787e65b964/dcaef/diagrams/functionals/map2-recycle.png" width=400>

This is most useful when y has only one element.

#### `purrr::pmap*`

```
pmap*(list(.x1, .x2, ... .xn), .f, ...)
```

Above reads as: `for` every element of in the **list** (that contains `.x1, .x2, ... .xn`) apply `.f` 

But what happens when you have > 2 arguments?

## (OPTIONAL) Mapping more than two arguments

Without an anonymous function, works as so:


```R
f1 <- function(x, y, z) {
    x + y + z
}

pmap_dbl(list(c(1, 1), c(1, 2), c(2, 2)), f1)
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>4</li><li>5</li></ol>



If you want to use an anonymous function, then use `..1`, `..2`, `..3`, and so on to specify where the mapped objets go in your function:


```R
f2 <- function(x, y, z, a = 0) {
    x + y + z + a
}

pmap_dbl(list(c(1, 1), c(1, 2), c(2, 2)), ~ f2(..1, ..2, ..3, a = -1))
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>3</li><li>4</li></ol>



We only used two inputs to our function here, but we can use any number with `pmap`, we just need to add them to our list!

### Want to iterate row-wise, instead of column-wise?

Here you can use `purrr::pmap` on a single data frame!

This: ```purrr::pmap(df, .f)```

reads as: `for` every tuple in `.l` (*i.e.*, each row of `df`) apply `.f`

The key point is that `pmap()` iterates over tuples = the collection of `i`-th elements of `k` lists. A data frame row is an interesting special case.

### Here's an example of row-wise iteration 

Here we calculate the sum for each row in the `mtcars` data frame:


```R
pmap(mtcars, sum)
```


<ol>
	<li>328.98</li>
	<li>329.795</li>
	<li>259.58</li>
	<li>426.135</li>
	<li>590.31</li>
	<li>385.54</li>
	<li>656.92</li>
	<li>270.98</li>
	<li>299.57</li>
	<li>350.46</li>
	<li>349.66</li>
	<li>510.74</li>
	<li>511.5</li>
	<li>509.85</li>
	<li>728.56</li>
	<li>726.644</li>
	<li>725.695</li>
	<li>213.85</li>
	<li>195.165</li>
	<li>206.955</li>
	<li>273.775</li>
	<li>519.65</li>
	<li>506.085</li>
	<li>646.28</li>
	<li>631.175</li>
	<li>208.215</li>
	<li>272.57</li>
	<li>273.683</li>
	<li>670.69</li>
	<li>379.59</li>
	<li>694.71</li>
	<li>288.89</li>
</ol>



### Mapping over groups of rows

There are two strategies, we have already learned about the first:
- `dplyr::group_by` + `dplyr::summarize`
- `dplyr::group_by` + `tidyr::nest`

## Nested data frames

In future stastistics classes, we will use nested data frames for working with models and estimates. For example, in DSCI 552 we will use them for calculating confidence intervals. Here is some sample code below to calculcate confidence intervals for the mean life expectency for each each country in the `gapminder` data set:


```R
gap_lifeExp_ci <- function(df, statistic) {
  df %>% 
        specify(response = lifeExp) %>% 
        generate(reps = 1000, type = "bootstrap")  %>% 
        calculate(stat = statistic)  %>% 
        get_ci()
}

by_country <- gapminder %>%
    group_by(continent, country) %>%
    nest() %>% 
    mutate(mean_life_exp = map_dbl(data, ~mean(.$lifeExp)), 
    life_exp_ci = map(data, ~gap_lifeExp_ci(., "mean")))
print(by_country)
```

    [90m# A tibble: 142 × 5[39m
    [90m# Groups:   continent, country [142][39m
       country     continent data              mean_life_exp life_exp_ci     
       [3m[90m<fct>[39m[23m       [3m[90m<fct>[39m[23m     [3m[90m<list>[39m[23m                    [3m[90m<dbl>[39m[23m [3m[90m<list>[39m[23m          
    [90m 1[39m Afghanistan Asia      [90m<tibble [12 × 4]>[39m          37.5 [90m<tibble [1 × 2]>[39m
    [90m 2[39m Albania     Europe    [90m<tibble [12 × 4]>[39m          68.4 [90m<tibble [1 × 2]>[39m
    [90m 3[39m Algeria     Africa    [90m<tibble [12 × 4]>[39m          59.0 [90m<tibble [1 × 2]>[39m
    [90m 4[39m Angola      Africa    [90m<tibble [12 × 4]>[39m          37.9 [90m<tibble [1 × 2]>[39m
    [90m 5[39m Argentina   Americas  [90m<tibble [12 × 4]>[39m          69.1 [90m<tibble [1 × 2]>[39m
    [90m 6[39m Australia   Oceania   [90m<tibble [12 × 4]>[39m          74.7 [90m<tibble [1 × 2]>[39m
    [90m 7[39m Austria     Europe    [90m<tibble [12 × 4]>[39m          73.1 [90m<tibble [1 × 2]>[39m
    [90m 8[39m Bahrain     Asia      [90m<tibble [12 × 4]>[39m          65.6 [90m<tibble [1 × 2]>[39m
    [90m 9[39m Bangladesh  Asia      [90m<tibble [12 × 4]>[39m          49.8 [90m<tibble [1 × 2]>[39m
    [90m10[39m Belgium     Europe    [90m<tibble [12 × 4]>[39m          73.6 [90m<tibble [1 × 2]>[39m
    [90m# … with 132 more rows[39m
    

This is called a nested data frame. A data frame which contains other data frames. This data structure is extremely useful when fitting many models and allows you to keep the data, the model meta data, and the model and its results all associated together as a single row in a data frame.

Let's unpack this data structure a bit more and learn how to create, manipulate them.

### List-columns

To create a nested data frame we start with a **grouped** data frame, and “nest” it:


```R
# create a nested data frame
by_country <- gapminder %>% 
    group_by(continent, country) %>% 
    nest()
print(by_country)
```

    [90m# A tibble: 142 × 3[39m
    [90m# Groups:   continent, country [142][39m
       country     continent data             
       [3m[90m<fct>[39m[23m       [3m[90m<fct>[39m[23m     [3m[90m<list>[39m[23m           
    [90m 1[39m Afghanistan Asia      [90m<tibble [12 × 4]>[39m
    [90m 2[39m Albania     Europe    [90m<tibble [12 × 4]>[39m
    [90m 3[39m Algeria     Africa    [90m<tibble [12 × 4]>[39m
    [90m 4[39m Angola      Africa    [90m<tibble [12 × 4]>[39m
    [90m 5[39m Argentina   Americas  [90m<tibble [12 × 4]>[39m
    [90m 6[39m Australia   Oceania   [90m<tibble [12 × 4]>[39m
    [90m 7[39m Austria     Europe    [90m<tibble [12 × 4]>[39m
    [90m 8[39m Bahrain     Asia      [90m<tibble [12 × 4]>[39m
    [90m 9[39m Bangladesh  Asia      [90m<tibble [12 × 4]>[39m
    [90m10[39m Belgium     Europe    [90m<tibble [12 × 4]>[39m
    [90m# … with 132 more rows[39m
    

(I’m cheating a little by grouping on both continent and country. Given country, continent is fixed, so this doesn’t add any more groups, but it’s an easy way to carry an extra variable along for the ride.)

What is the `data` column here? 


```R
class(by_country$data)
```


'list'


The `data` column is actually a list of data frames (or tibbles, to be precise). This seems like a crazy idea: we have a data frame with a column that is a list of other data frames! 

Let's look at what the first element of the `data` list column looks like:


```R
# look at first element of the data list column
by_country$data[[1]]
```


<table class="dataframe">
<caption>A tibble: 12 × 4</caption>
<thead>
	<tr><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>1952</td><td>28.801</td><td> 8425333</td><td>779.4453</td></tr>
	<tr><td>1957</td><td>30.332</td><td> 9240934</td><td>820.8530</td></tr>
	<tr><td>1962</td><td>31.997</td><td>10267083</td><td>853.1007</td></tr>
	<tr><td>1967</td><td>34.020</td><td>11537966</td><td>836.1971</td></tr>
	<tr><td>1972</td><td>36.088</td><td>13079460</td><td>739.9811</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>1987</td><td>40.822</td><td>13867957</td><td>852.3959</td></tr>
	<tr><td>1992</td><td>41.674</td><td>16317921</td><td>649.3414</td></tr>
	<tr><td>1997</td><td>41.763</td><td>22227415</td><td>635.3414</td></tr>
	<tr><td>2002</td><td>42.129</td><td>25268405</td><td>726.7341</td></tr>
	<tr><td>2007</td><td>43.828</td><td>31889923</td><td>974.5803</td></tr>
</tbody>
</table>



Note the difference between a standard grouped data frame and a nested data frame: in a grouped data frame, each row is an observation; in a nested data frame, each row is a group. Another way to think about a nested dataset is we now have a meta-observation: a row that represents the complete time course for a country, rather than a single point in time.

#### (OPTIONAL) List-columns in data frames versus tibbles

List columns are possible in base R data frames, but making them isn't easy and you have to jump through some hoops.

If we try to make a data frame with a single list column, `data.frame` treats the list as a list of columns...




```R
# try to make a list column in a data frame
data.frame(x = list(c(1, 2, 3), c(3, 4, 5)))
```


<table class="dataframe">
<caption>A data.frame: 3 × 2</caption>
<thead>
	<tr><th scope=col>x.c.1..2..3.</th><th scope=col>x.c.3..4..5.</th></tr>
	<tr><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>1</td><td>3</td></tr>
	<tr><td>2</td><td>4</td></tr>
	<tr><td>3</td><td>5</td></tr>
</tbody>
</table>



To make the work, we need to use the `I` function (which changes the class of an object to indicate that it should be treated ‘as is’):


```R
# try to make a list column in a data frame by using I
data.frame(x = I(list(c(1, 2, 3), c(3, 4, 5))))
```


<table class="dataframe">
<caption>A data.frame: 2 × 1</caption>
<thead>
	<tr><th scope=col>x</th></tr>
	<tr><th scope=col>&lt;I&lt;list&gt;&gt;</th></tr>
</thead>
<tbody>
	<tr><td>1, 2, 3</td></tr>
	<tr><td>3, 4, 5</td></tr>
</tbody>
</table>



Tibble alleviates this problem by being lazier (`tibble` doesn’t modify its inputs) and by providing a better print method:


```R
# try to make a list column in a tibble
tibble(x = list(c(1, 2, 3), c(3, 4, 5)))
```


<table class="dataframe">
<caption>A tibble: 2 × 1</caption>
<thead>
	<tr><th scope=col>x</th></tr>
	<tr><th scope=col>&lt;list&gt;</th></tr>
</thead>
<tbody>
	<tr><td>1, 2, 3</td></tr>
	<tr><td>3, 4, 5</td></tr>
</tbody>
</table>



### List column workflow:

1. Create a list column using a function such as `nest` (other possibilities are `summarise` + `list`, or `mutate` + a `map_*` function, as described in [Creating list-columns](https://r4ds.had.co.nz/many-models.html#creating-list-columns))

2. Create other intermediate list-columns by transforming existing list columns with `map`, `map2` or `pmap`

3. Simplify the list-column back down to a data frame or atomic vector, often by `unnest`, `mutate` + `map_*` functions that return atomic vectors as opposed to lists. 

We've talked about step 1, and what a list column is. Now let's explore how we can create other intermediate list-columns by transforming existing columns with `map`.

### Create other intermediate list-columns with `map`

Remembering our example:


```R
gap_lifeExp_ci <- function(df, statistic) {
  df %>% 
        specify(response = lifeExp) %>% 
        generate(reps = 1000, type = "bootstrap")  %>% 
        calculate(stat = statistic)  %>% 
        get_ci()
}

# create a nested gapminder tibble
by_country <- gapminder %>% 
    group_by(continent, country) %>% 
    nest() 
```


```R
print(by_country)
```

    [90m# A tibble: 142 × 3[39m
    [90m# Groups:   continent, country [142][39m
       country     continent data             
       [3m[90m<fct>[39m[23m       [3m[90m<fct>[39m[23m     [3m[90m<list>[39m[23m           
    [90m 1[39m Afghanistan Asia      [90m<tibble [12 × 4]>[39m
    [90m 2[39m Albania     Europe    [90m<tibble [12 × 4]>[39m
    [90m 3[39m Algeria     Africa    [90m<tibble [12 × 4]>[39m
    [90m 4[39m Angola      Africa    [90m<tibble [12 × 4]>[39m
    [90m 5[39m Argentina   Americas  [90m<tibble [12 × 4]>[39m
    [90m 6[39m Australia   Oceania   [90m<tibble [12 × 4]>[39m
    [90m 7[39m Austria     Europe    [90m<tibble [12 × 4]>[39m
    [90m 8[39m Bahrain     Asia      [90m<tibble [12 × 4]>[39m
    [90m 9[39m Bangladesh  Asia      [90m<tibble [12 × 4]>[39m
    [90m10[39m Belgium     Europe    [90m<tibble [12 × 4]>[39m
    [90m# … with 132 more rows[39m
    

We'd like to apply the mean function to get the mean life expectancy in a column:


```R
by_country <- gapminder %>%
    group_by(continent, country) %>%
    nest() %>% 
    mutate(mean_life_exp = map_dbl(data, ~ mean(.$lifeExp)))  
print(by_country)
```

    [90m# A tibble: 142 × 4[39m
    [90m# Groups:   continent, country [142][39m
       country     continent data              mean_life_exp
       [3m[90m<fct>[39m[23m       [3m[90m<fct>[39m[23m     [3m[90m<list>[39m[23m                    [3m[90m<dbl>[39m[23m
    [90m 1[39m Afghanistan Asia      [90m<tibble [12 × 4]>[39m          37.5
    [90m 2[39m Albania     Europe    [90m<tibble [12 × 4]>[39m          68.4
    [90m 3[39m Algeria     Africa    [90m<tibble [12 × 4]>[39m          59.0
    [90m 4[39m Angola      Africa    [90m<tibble [12 × 4]>[39m          37.9
    [90m 5[39m Argentina   Americas  [90m<tibble [12 × 4]>[39m          69.1
    [90m 6[39m Australia   Oceania   [90m<tibble [12 × 4]>[39m          74.7
    [90m 7[39m Austria     Europe    [90m<tibble [12 × 4]>[39m          73.1
    [90m 8[39m Bahrain     Asia      [90m<tibble [12 × 4]>[39m          65.6
    [90m 9[39m Bangladesh  Asia      [90m<tibble [12 × 4]>[39m          49.8
    [90m10[39m Belgium     Europe    [90m<tibble [12 × 4]>[39m          73.6
    [90m# … with 132 more rows[39m
    

Now we'd like to apply the `gap_lifeExp_ci` function to each tibble in the `data` list column to obtain another list column containing the confidence interval tibbles. We can use `mutate` + `map` to do this:


```R
gap_lifeExp_ci <- function(df, statistic) {
  df %>% 
        specify(response = lifeExp) %>% 
        generate(reps = 1000, type = "bootstrap")  %>% 
        calculate(stat = statistic)  %>% 
        get_ci()
}

by_country <- gapminder %>%
    group_by(continent, country) %>%
    nest() %>% 
    mutate(mean_life_exp = map_dbl(data, ~mean(.$lifeExp)), 
    life_exp_ci = map(data, ~gap_lifeExp_ci(., "mean")))
print(by_country)
```

    [90m# A tibble: 142 × 5[39m
    [90m# Groups:   continent, country [142][39m
       country     continent data              mean_life_exp life_exp_ci     
       [3m[90m<fct>[39m[23m       [3m[90m<fct>[39m[23m     [3m[90m<list>[39m[23m                    [3m[90m<dbl>[39m[23m [3m[90m<list>[39m[23m          
    [90m 1[39m Afghanistan Asia      [90m<tibble [12 × 4]>[39m          37.5 [90m<tibble [1 × 2]>[39m
    [90m 2[39m Albania     Europe    [90m<tibble [12 × 4]>[39m          68.4 [90m<tibble [1 × 2]>[39m
    [90m 3[39m Algeria     Africa    [90m<tibble [12 × 4]>[39m          59.0 [90m<tibble [1 × 2]>[39m
    [90m 4[39m Angola      Africa    [90m<tibble [12 × 4]>[39m          37.9 [90m<tibble [1 × 2]>[39m
    [90m 5[39m Argentina   Americas  [90m<tibble [12 × 4]>[39m          69.1 [90m<tibble [1 × 2]>[39m
    [90m 6[39m Australia   Oceania   [90m<tibble [12 × 4]>[39m          74.7 [90m<tibble [1 × 2]>[39m
    [90m 7[39m Austria     Europe    [90m<tibble [12 × 4]>[39m          73.1 [90m<tibble [1 × 2]>[39m
    [90m 8[39m Bahrain     Asia      [90m<tibble [12 × 4]>[39m          65.6 [90m<tibble [1 × 2]>[39m
    [90m 9[39m Bangladesh  Asia      [90m<tibble [12 × 4]>[39m          49.8 [90m<tibble [1 × 2]>[39m
    [90m10[39m Belgium     Europe    [90m<tibble [12 × 4]>[39m          73.6 [90m<tibble [1 × 2]>[39m
    [90m# … with 132 more rows[39m
    

What does the new tibbles look like in the `life_exp_ci` column?


```R
by_country$life_exp_ci[[1]]
```


<table class="dataframe">
<caption>A tibble: 1 × 2</caption>
<thead>
	<tr><th scope=col>lower_ci</th><th scope=col>upper_ci</th></tr>
	<tr><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>34.52446</td><td>40.12731</td></tr>
</tbody>
</table>



### Simplifying the list-column back down to a data frame or atomic vector

After we create some other intermediate list-columns with `map`, we usually want to get some values back as regular atomic vector columns in our data frame, for visualization, further analysis, or reporting. 

We will first demonstrate how to do this using `unnest` in our example to covert the `life_exp_ci` list column to two columns, one for the lower bound of the confidence interval, and one for the upper bound of the confidence interval:


```R
# unnest the ci column
by_country %>% 
    unnest(life_exp_ci) %>% 
    print()
```

    [90m# A tibble: 142 × 6[39m
    [90m# Groups:   continent, country [142][39m
       country     continent data              mean_life_exp lower_ci upper_ci
       [3m[90m<fct>[39m[23m       [3m[90m<fct>[39m[23m     [3m[90m<list>[39m[23m                    [3m[90m<dbl>[39m[23m    [3m[90m<dbl>[39m[23m    [3m[90m<dbl>[39m[23m
    [90m 1[39m Afghanistan Asia      [90m<tibble [12 × 4]>[39m          37.5     34.5     40.1
    [90m 2[39m Albania     Europe    [90m<tibble [12 × 4]>[39m          68.4     64.8     71.7
    [90m 3[39m Algeria     Africa    [90m<tibble [12 × 4]>[39m          59.0     53.4     64.4
    [90m 4[39m Angola      Africa    [90m<tibble [12 × 4]>[39m          37.9     35.7     39.8
    [90m 5[39m Argentina   Americas  [90m<tibble [12 × 4]>[39m          69.1     66.7     71.4
    [90m 6[39m Australia   Oceania   [90m<tibble [12 × 4]>[39m          74.7     72.5     77.0
    [90m 7[39m Austria     Europe    [90m<tibble [12 × 4]>[39m          73.1     70.8     75.4
    [90m 8[39m Bahrain     Asia      [90m<tibble [12 × 4]>[39m          65.6     60.8     70.2
    [90m 9[39m Bangladesh  Asia      [90m<tibble [12 × 4]>[39m          49.8     44.8     54.6
    [90m10[39m Belgium     Europe    [90m<tibble [12 × 4]>[39m          73.6     71.7     75.7
    [90m# … with 132 more rows[39m
    

### The whole shebang:

1. Create a list column using a function such as `nest`

2. Create other intermediate list-columns by transforming existing list columns with `map`

3. Simplify the list-column back down to a data frame or atomic vector using `unnest` & `mutate` + `map_dbl`:


```R
gap_lifeExp_ci <- function(df, statistic) {
  df %>% 
        specify(response = lifeExp) %>% 
        generate(reps = 1000, type = "bootstrap")  %>% 
        calculate(stat = statistic)  %>% 
        get_ci()
}

by_country <- gapminder %>%
    group_by(continent, country) %>%
    nest() %>% 
    mutate(mean_life_exp = map_dbl(data, ~mean(.$lifeExp)), 
    life_exp_ci = map(data, ~gap_lifeExp_ci(., "mean")))
print(by_country)
```

    [90m# A tibble: 142 × 5[39m
    [90m# Groups:   continent, country [142][39m
       country     continent data              mean_life_exp life_exp_ci     
       [3m[90m<fct>[39m[23m       [3m[90m<fct>[39m[23m     [3m[90m<list>[39m[23m                    [3m[90m<dbl>[39m[23m [3m[90m<list>[39m[23m          
    [90m 1[39m Afghanistan Asia      [90m<tibble [12 × 4]>[39m          37.5 [90m<tibble [1 × 2]>[39m
    [90m 2[39m Albania     Europe    [90m<tibble [12 × 4]>[39m          68.4 [90m<tibble [1 × 2]>[39m
    [90m 3[39m Algeria     Africa    [90m<tibble [12 × 4]>[39m          59.0 [90m<tibble [1 × 2]>[39m
    [90m 4[39m Angola      Africa    [90m<tibble [12 × 4]>[39m          37.9 [90m<tibble [1 × 2]>[39m
    [90m 5[39m Argentina   Americas  [90m<tibble [12 × 4]>[39m          69.1 [90m<tibble [1 × 2]>[39m
    [90m 6[39m Australia   Oceania   [90m<tibble [12 × 4]>[39m          74.7 [90m<tibble [1 × 2]>[39m
    [90m 7[39m Austria     Europe    [90m<tibble [12 × 4]>[39m          73.1 [90m<tibble [1 × 2]>[39m
    [90m 8[39m Bahrain     Asia      [90m<tibble [12 × 4]>[39m          65.6 [90m<tibble [1 × 2]>[39m
    [90m 9[39m Bangladesh  Asia      [90m<tibble [12 × 4]>[39m          49.8 [90m<tibble [1 × 2]>[39m
    [90m10[39m Belgium     Europe    [90m<tibble [12 × 4]>[39m          73.6 [90m<tibble [1 × 2]>[39m
    [90m# … with 132 more rows[39m
    

## What did we learn:

- how to write anonymous functions
- how to use {purrr} `map_*` with anonymous functions to add extra arguments
- what are nested data frames
- how to use {tidyr}'s `nest` & `unnest` and {purrr} `map_*` functions to work with data frames to nest, modify and unnest data frames

### Attribution

- [Advanced R](https://adv-r.hadley.nz/) by Hadley Wickham
- [Jenny Bryan's `purrr` tutorial](https://jennybc.github.io/purrr-tutorial/index.html)
