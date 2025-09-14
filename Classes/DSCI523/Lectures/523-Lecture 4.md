# Lecture 4 -  Combining data frames (binds and joins) & base R control flow

### Lecture learning objectives:
By the end of this lecture and worksheet 4, students should be able to:

* Compare and contrast mutating joins, filtering joins and set operations
* Choose the appropriate two table `dplyr` function based on the type of join desired between two data frames, and use it in R to obtained the desired data frame from joining two tables
* Write conditional statements in R with `if`, `else if` and `else` to run different code depending on the input
* Write for loops in R to repeatedly run code


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


## Combining two data tables

We will talk about two different ways of combining data tables in R:

- binds

- joins

### Binds

This is basically smashing rocks tibbles together. You can smash things together row-wise (“row binding”) or column-wise (“column binding”). 

Why do I characterize this as rock-smashing? They’re often fairly crude operations, with lots of responsibility falling on the analyst for making sure that the whole enterprise even makes sense.

#### Row binding

When row binding, you need to consider the variables in the two tibbles. Do the same variables exist in each? Are they of the same type? Different approaches for row binding have different combinations of flexibility vs rigidity around these matters.

Let's bind 3 data frames together vertically (row binding):


```R
fship <- tribble(
                         ~Film,    ~Race, ~Female, ~Male,
  "The Fellowship Of The Ring",    "Elf",    1229,   971,
  "The Fellowship Of The Ring", "Hobbit",      14,  3644,
  "The Fellowship Of The Ring",    "Man",       0,  1995
)
rking <- tribble(
                         ~Film,    ~Race, ~Female, ~Male,
      "The Return Of The King",    "Elf",     183,   510,
      "The Return Of The King", "Hobbit",       2,  2673,
      "The Return Of The King",    "Man",     268,  2459
)
ttow <- tribble(
                         ~Film,    ~Race, ~Female, ~Male,
              "The Two Towers",    "Elf",     331,   513,
              "The Two Towers", "Hobbit",       0,  2463,
              "The Two Towers",    "Man",     401,  3589
)
fship
rking
ttow
```


<table class="dataframe">
<caption>A tibble: 3 × 4</caption>
<thead>
	<tr><th scope=col>Film</th><th scope=col>Race</th><th scope=col>Female</th><th scope=col>Male</th></tr>
	<tr><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>The Fellowship Of The Ring</td><td>Elf   </td><td>1229</td><td> 971</td></tr>
	<tr><td>The Fellowship Of The Ring</td><td>Hobbit</td><td>  14</td><td>3644</td></tr>
	<tr><td>The Fellowship Of The Ring</td><td>Man   </td><td>   0</td><td>1995</td></tr>
</tbody>
</table>




<table class="dataframe">
<caption>A tibble: 3 × 4</caption>
<thead>
	<tr><th scope=col>Film</th><th scope=col>Race</th><th scope=col>Female</th><th scope=col>Male</th></tr>
	<tr><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>The Return Of The King</td><td>Elf   </td><td>183</td><td> 510</td></tr>
	<tr><td>The Return Of The King</td><td>Hobbit</td><td>  2</td><td>2673</td></tr>
	<tr><td>The Return Of The King</td><td>Man   </td><td>268</td><td>2459</td></tr>
</tbody>
</table>




<table class="dataframe">
<caption>A tibble: 3 × 4</caption>
<thead>
	<tr><th scope=col>Film</th><th scope=col>Race</th><th scope=col>Female</th><th scope=col>Male</th></tr>
	<tr><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>The Two Towers</td><td>Elf   </td><td>331</td><td> 513</td></tr>
	<tr><td>The Two Towers</td><td>Hobbit</td><td>  0</td><td>2463</td></tr>
	<tr><td>The Two Towers</td><td>Man   </td><td>401</td><td>3589</td></tr>
</tbody>
</table>



> `tribble` is another function that you can use to make tibbles, but let's you write the data in a more human readable form.


```R
# bind the rows
bind_rows(fship, ttow, rking)
```


<table class="dataframe">
<caption>A tibble: 9 × 4</caption>
<thead>
	<tr><th scope=col>Film</th><th scope=col>Race</th><th scope=col>Female</th><th scope=col>Male</th></tr>
	<tr><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>The Fellowship Of The Ring</td><td>Elf   </td><td>1229</td><td> 971</td></tr>
	<tr><td>The Fellowship Of The Ring</td><td>Hobbit</td><td>  14</td><td>3644</td></tr>
	<tr><td>The Fellowship Of The Ring</td><td>Man   </td><td>   0</td><td>1995</td></tr>
	<tr><td>The Two Towers            </td><td>Elf   </td><td> 331</td><td> 513</td></tr>
	<tr><td>The Two Towers            </td><td>Hobbit</td><td>   0</td><td>2463</td></tr>
	<tr><td>The Two Towers            </td><td>Man   </td><td> 401</td><td>3589</td></tr>
	<tr><td>The Return Of The King    </td><td>Elf   </td><td> 183</td><td> 510</td></tr>
	<tr><td>The Return Of The King    </td><td>Hobbit</td><td>   2</td><td>2673</td></tr>
	<tr><td>The Return Of The King    </td><td>Man   </td><td> 268</td><td>2459</td></tr>
</tbody>
</table>



#### Column binding

When column binding, the onus is entirely on the analyst to make sure that the rows are aligned. I would avoid column binding whenever possible. If you can introduce new variables through any other, safer means, do so! By safer, I mean: use a mechanism where the row alignment is correct by definition. A proper join is the gold standard. In addition to joins, functions like dplyr::mutate() and tidyr::separate() can be very useful for forcing yourself to work inside the constraint of a tibble.

Let's bind 3 columns onto a data frame:


```R
fship <- tribble(
                         ~Film,    
  "The Fellowship Of The Ring",
  "The Fellowship Of The Ring", 
  "The Fellowship Of The Ring"
)

fship_data <- tribble(
  ~Race, ~Female, ~Male,
  "Elf",    1229,   971,
  "Hobbit",      14,  3644,
  "Man",       0,  1995
)

fship
fship_data
```


<table class="dataframe">
<caption>A tibble: 3 × 1</caption>
<thead>
	<tr><th scope=col>Film</th></tr>
	<tr><th scope=col>&lt;chr&gt;</th></tr>
</thead>
<tbody>
	<tr><td>The Fellowship Of The Ring</td></tr>
	<tr><td>The Fellowship Of The Ring</td></tr>
	<tr><td>The Fellowship Of The Ring</td></tr>
</tbody>
</table>




<table class="dataframe">
<caption>A tibble: 3 × 3</caption>
<thead>
	<tr><th scope=col>Race</th><th scope=col>Female</th><th scope=col>Male</th></tr>
	<tr><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>Elf   </td><td>1229</td><td> 971</td></tr>
	<tr><td>Hobbit</td><td>  14</td><td>3644</td></tr>
	<tr><td>Man   </td><td>   0</td><td>1995</td></tr>
</tbody>
</table>




```R
# bind the columns
bind_cols(fship, fship_data)
```


<table class="dataframe">
<caption>A tibble: 3 × 4</caption>
<thead>
	<tr><th scope=col>Film</th><th scope=col>Race</th><th scope=col>Female</th><th scope=col>Male</th></tr>
	<tr><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>The Fellowship Of The Ring</td><td>Elf   </td><td>1229</td><td> 971</td></tr>
	<tr><td>The Fellowship Of The Ring</td><td>Hobbit</td><td>  14</td><td>3644</td></tr>
	<tr><td>The Fellowship Of The Ring</td><td>Man   </td><td>   0</td><td>1995</td></tr>
</tbody>
</table>



### Joins

Here you designate a variable (or a combination of variables) as a key. A row in one data frame gets matched with a row in another data frame because they have the same key. You can then bring information from variables in a secondary data frame into a primary data frame based on this key-based lookup. That description is incredibly oversimplified, but that’s the basic idea.

A variety of row- and column-wise operations fit into this framework, which implies there are many different flavors of join. The concepts and vocabulary around joins come from the database world. The relevant functions in `dplyr` follow this convention and all mention `join`. 

Your best cheatsheet: https://stat545.com/join-cheatsheet.html

Compliments of Jenny Bryan!

#### Left join

`left_join(x, y)`: Return all rows from `x`, and all columns from `x` and `y`. If there are multiple matches between `x` and `y`, all combination of the matches are returned. This is a mutating join.

```{figure} img/left_join.png
    ---
    height: 400px
    name: left_join
    align: left
    ---
```

*Source: https://stat545.com/join-cheatsheet.html*

Let's work through an example. We will create a data frame called `am_af_countries` from American and African countries from the gapminder data set, that contains the country and continent rows:


```R
am_af_countries <- gapminder %>% 
    filter(continent == "Americas" | continent == "Africa") %>% 
    select(country, continent) %>% 
    group_by(country) %>% 
    slice(1)
am_af_countries
```


<table class="dataframe">
<caption>A grouped_df: 77 × 2</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th></tr>
</thead>
<tbody>
	<tr><td>Algeria  </td><td>Africa  </td></tr>
	<tr><td>Angola   </td><td>Africa  </td></tr>
	<tr><td>Argentina</td><td>Americas</td></tr>
	<tr><td>Benin    </td><td>Africa  </td></tr>
	<tr><td>Bolivia  </td><td>Americas</td></tr>
	<tr><td>⋮</td><td>⋮</td></tr>
	<tr><td>United States</td><td>Americas</td></tr>
	<tr><td>Uruguay      </td><td>Americas</td></tr>
	<tr><td>Venezuela    </td><td>Americas</td></tr>
	<tr><td>Zambia       </td><td>Africa  </td></tr>
	<tr><td>Zimbabwe     </td><td>Africa  </td></tr>
</tbody>
</table>




```R
country_codes
```


<table class="dataframe">
<caption>A tibble: 187 × 3</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>iso_alpha</th><th scope=col>iso_num</th></tr>
	<tr><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;int&gt;</th></tr>
</thead>
<tbody>
	<tr><td>Afghanistan</td><td>AFG</td><td> 4</td></tr>
	<tr><td>Albania    </td><td>ALB</td><td> 8</td></tr>
	<tr><td>Algeria    </td><td>DZA</td><td>12</td></tr>
	<tr><td>Angola     </td><td>AGO</td><td>24</td></tr>
	<tr><td>Argentina  </td><td>ARG</td><td>32</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>Vietnam           </td><td>VNM</td><td>704</td></tr>
	<tr><td>West Bank and Gaza</td><td>PSE</td><td>275</td></tr>
	<tr><td>Yemen, Rep.       </td><td>YEM</td><td>887</td></tr>
	<tr><td>Zambia            </td><td>ZMB</td><td>894</td></tr>
	<tr><td>Zimbabwe          </td><td>ZWE</td><td>716</td></tr>
</tbody>
</table>




```R
# join data frames
left_join(am_af_countries, country_codes) 
```

    [1m[22mJoining, by = "country"



<table class="dataframe">
<caption>A grouped_df: 77 × 4</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>iso_alpha</th><th scope=col>iso_num</th></tr>
	<tr><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;int&gt;</th></tr>
</thead>
<tbody>
	<tr><td>Algeria  </td><td>Africa  </td><td>DZA</td><td> 12</td></tr>
	<tr><td>Angola   </td><td>Africa  </td><td>AGO</td><td> 24</td></tr>
	<tr><td>Argentina</td><td>Americas</td><td>ARG</td><td> 32</td></tr>
	<tr><td>Benin    </td><td>Africa  </td><td>BEN</td><td>204</td></tr>
	<tr><td>Bolivia  </td><td>Americas</td><td>BOL</td><td> 68</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>United States</td><td>Americas</td><td>USA</td><td>840</td></tr>
	<tr><td>Uruguay      </td><td>Americas</td><td>URY</td><td>858</td></tr>
	<tr><td>Venezuela    </td><td>Americas</td><td>VEN</td><td>862</td></tr>
	<tr><td>Zambia       </td><td>Africa  </td><td>ZMB</td><td>894</td></tr>
	<tr><td>Zimbabwe     </td><td>Africa  </td><td>ZWE</td><td>716</td></tr>
</tbody>
</table>



#### What if your column names don't match?

You can specify which columns to join by! 


```R
#rename country column _renamed <- am_af_countries %>% 
am_af_countries <- am_af_countries %>% 
    rename(countries = country)
am_af_countries
```


<table class="dataframe">
<caption>A grouped_df: 77 × 2</caption>
<thead>
	<tr><th scope=col>countries</th><th scope=col>continent</th></tr>
	<tr><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;fct&gt;</th></tr>
</thead>
<tbody>
	<tr><td>Algeria  </td><td>Africa  </td></tr>
	<tr><td>Angola   </td><td>Africa  </td></tr>
	<tr><td>Argentina</td><td>Americas</td></tr>
	<tr><td>Benin    </td><td>Africa  </td></tr>
	<tr><td>Bolivia  </td><td>Americas</td></tr>
	<tr><td>⋮</td><td>⋮</td></tr>
	<tr><td>United States</td><td>Americas</td></tr>
	<tr><td>Uruguay      </td><td>Americas</td></tr>
	<tr><td>Venezuela    </td><td>Americas</td></tr>
	<tr><td>Zambia       </td><td>Africa  </td></tr>
	<tr><td>Zimbabwe     </td><td>Africa  </td></tr>
</tbody>
</table>




```R
left_join(am_af_countries, country_codes, 
          by = c("countries" = "country"))
```


<table class="dataframe">
<caption>A grouped_df: 77 × 4</caption>
<thead>
	<tr><th scope=col>countries</th><th scope=col>continent</th><th scope=col>iso_alpha</th><th scope=col>iso_num</th></tr>
	<tr><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;fct&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;int&gt;</th></tr>
</thead>
<tbody>
	<tr><td>Algeria  </td><td>Africa  </td><td>DZA</td><td> 12</td></tr>
	<tr><td>Angola   </td><td>Africa  </td><td>AGO</td><td> 24</td></tr>
	<tr><td>Argentina</td><td>Americas</td><td>ARG</td><td> 32</td></tr>
	<tr><td>Benin    </td><td>Africa  </td><td>BEN</td><td>204</td></tr>
	<tr><td>Bolivia  </td><td>Americas</td><td>BOL</td><td> 68</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>United States</td><td>Americas</td><td>USA</td><td>840</td></tr>
	<tr><td>Uruguay      </td><td>Americas</td><td>URY</td><td>858</td></tr>
	<tr><td>Venezuela    </td><td>Americas</td><td>VEN</td><td>862</td></tr>
	<tr><td>Zambia       </td><td>Africa  </td><td>ZMB</td><td>894</td></tr>
	<tr><td>Zimbabwe     </td><td>Africa  </td><td>ZWE</td><td>716</td></tr>
</tbody>
</table>



#### Other joins:

There are several other joins available to you in the {dplyr} package. I give a high level overview next, and then you will get a chance to interact and explore these functions in more detail in your worksheet and lab.

#### Inner join

`inner_join(x, y)`: Return all rows from `x` where there are matching values in `y`, and all columns from `x` and `y`. If there are multiple matches between `x` and `y`, all combination of the matches are returned. This is a mutating join.

```{figure} img/inner_join.png
    ---
    height: 400px
    name: inner_join
    align: left
    ---
```

*Source: https://stat545.com/join-cheatsheet.html*

#### Semi join

`semi_join(x, y)`: Return all rows from `x` where there are matching values in `y`, keeping just columns from `x`. A semi join differs from an inner join because an inner join will return one row of x for each matching row of y, where a semi join will never duplicate rows of `x`. This is a filtering join.

```{figure} img/semi_join.png
    ---
    height: 400px
    name: semi_join
    align: left
    ---
```

*Source: https://stat545.com/join-cheatsheet.html*

#### Anti join

`anti_join(x, y)`: Return all rows from `x` where there are not matching values in `y`, keeping just columns from `x`. This is a filtering join.

```{figure} img/anti_join.png
    ---
    height: 400px
    name: anti_join
    align: left
    ---
```

*Source: https://stat545.com/join-cheatsheet.html*

#### Full join

`full_join(x, y)`: Return all rows and all columns from both `x` and `y`. Where there are not matching values, returns NA for the one missing. This is a mutating join.

```{figure} img/full_join.png
    ---
    height: 400px
    name: full_join
    align: left
    ---
```

*Source: https://stat545.com/join-cheatsheet.html*

## Control flow in base R
Similar to other programming languages, R has the now standard control flow capabilities. We will talk about two in this course:

- `for` loops

- `if` / `else if` / `else` statements (conditionals)

### Control Flow: `for` loops

- For loops in R, work like this: `for (item in vector) perform_action`
- When code needs to be split across lines in R, we use the `{` operator to surround it

#### Iterating over an object

Let's write a for loop that iterates over a vector of the numbers 1, 2, 3 and prints out the square of each. 


```R
sequence <- c(2, 3, 4)
sequence
typeof(sequence)
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>2</li><li>3</li><li>4</li></ol>




'double'



```R
for (number in sequence) {
    print(number ^ 2)
}
```

    [1] 4
    [1] 9
    [1] 16


#### Using indices in a for loop


```R
for (i in seq_along(sequence)) {
    print(sequence[i] ^ 2)
}
```

    [1] 4
    [1] 9
    [1] 16


#### Why use `seq_along`? 

It gives you a sequence along the item you want to iterate over.


```R
seq_along(sequence)
```


<style>
.list-inline {list-style: none; margin:0; padding: 0}
.list-inline>li {display: inline-block}
.list-inline>li:not(:last-child)::after {content: "\00b7"; padding: 0 .5ex}
</style>
<ol class=list-inline><li>1</li><li>2</li><li>3</li></ol>



Beware of using `length` instead:


```R
sequence <- c()
for (i in seq_along(sequence)) {
    print(sequence[i]^2)
    print(i)
}
```

There is nothing in the vector! Nothing should be printed!

*Source: [Advanced R](https://adv-r.hadley.nz/) by Hadley Wickham*

### Control Flow: `if`, `if else` and `else` statements 

The basic form of an if statement in R is as follows:

`if (condition) true_action`

`if (condition) true_action else false_action`

- Again, when code needs to be split across lines in R, we use the `{` operator to surround it to create code blocks

#### Example 

Below we write a conditional which compares a threshold with a measure. It prints "Over the limit" if the measure is over the threshold, "Under the limit" if the measure is under the threshold and "Exactly at threshold" otherwise. 


```R
threshold <- 95.0
measure  <- 93.5

if (measure > threshold) {
    print("Over the limit")
} else if (measure < threshold) {
    print("Under the limit")
} else {
    print("Exactly at threshold")
}
```

    [1] "Under the limit"


> Notes - you do not have to use `else if`, or even `else` if it doesn't make sense. I just wanted to demonstrate the full control flow syntax. Additionally, you can have multiple `else if` statements if you have > 3 choices to make.

### Attributions
- [Stat 545](https://stat545.com/) created by Jenny Bryan
- [R for Data Science](https://r4ds.had.co.nz/index.html) by Garrett Grolemund & Hadley Wickham
