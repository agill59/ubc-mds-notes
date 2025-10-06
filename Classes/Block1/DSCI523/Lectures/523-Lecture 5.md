# Lecture 5 -  Tidy control flow in R

### Lecture learning objectives:

By the end of this lecture and worksheet 5, students should be able to:

* Explain what a grouped data frame is, and how it can be used
* Use {dplyr}'s `group_by` + `summarize` to perform the split-apply-combine approach in R to iterate over and summarize data by groups
* Identify missing and erroneous values and manage them by removing (via {dplyr}'s `filter`) or replacing (using {dplyr}'s `mutate` + `case_when`)
* Identify where in R code a commonly used functional, a {purrr}'s `map*` function, could be used in place of for loops and write code to do this

```R
library(gapminder)
library(tidyverse)
options(repr.matrix.max.rows = 10)
```

## Change or remove specific values

- Sometimes we'd like to selectively change values, we can do this using control flow within `mutate`
- A related task is removing rows where there are NAs, either in a specified column or across the whole data frame or tibble

### Selectively change a values

What if we want to programatically change certain values? For example, update the country name "Cambodia" to its official English name, "Kingdom of Cambodia"?

If we want to change one or more (but not all things) in a column, one function we can use is `case_when` inside a `mutate` call:

```
gapminder %>% 
    mutate(country = case_when(country == "Cambodia" ~ "Kingdom of Cambodia",
                            TRUE ~ country))
```

```
Error: Problem with `mutate()` input `country`.
✖ must be a character vector, not a `factor` object.
ℹ Input `country` is `case_when(...)`.
Traceback:

1. gapminder %>% mutate(country = case_when(country == "Cambodia" ~ 
 .     "Kingdom of Cambodia", TRUE ~ country))
2. withVisible(eval(quote(`_fseq`(`_lhs`)), env, env))
3. eval(quote(`_fseq`(`_lhs`)), env, env)
4. eval(quote(`_fseq`(`_lhs`)), env
...
```

Ah! This doesn't work? What is going on here? Let's look at the structure of `gapminder`:

```R
str(gapminder)
```

    tibble [1,704 × 6] (S3: tbl_df/tbl/data.frame)
     $ country  : Factor w/ 142 levels "Afghanistan",..: 1 1 1 1 1 1 1 1 1 1 ...
     $ continent: Factor w/ 5 levels "Africa","Americas",..: 3 3 3 3 3 3 3 3 3 3 ...
     $ year     : int [1:1704] 1952 1957 1962 1967 1972 1977 1982 1987 1992 1997 ...
     $ lifeExp  : num [1:1704] 28.8 30.3 32 34 36.1 ...
     $ pop      : int [1:1704] 8425333 9240934 10267083 11537966 13079460 14880372 12881816 13867957 16317921 22227415 ...
     $ gdpPercap: num [1:1704] 779 821 853 836 740 ...

`country` (and `continent`) are factors! Let's change these to character vectors so we can manipulate the character values they hold. We'll make a new tibble named `gap` to do this:

```R
gap <- gapminder %>% 
    mutate(country = as.character(country),
          continent = as.character(continent))
```

OK, now let's try that conditional `mutate` again!

```R
gap <- gap %>% 
    mutate(country = case_when(country == "Cambodia" ~ "Kingdom of Cambodia",
                            TRUE ~ country))
gap
```

<table class="dataframe">
<caption>A tibble: 1704 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col><chr></th><th scope=col><chr></th><th scope=col><int></th><th scope=col><dbl></th><th scope=col><int></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Afghanistan</td><td>Asia</td><td>1952</td><td>28.801</td><td> 8425333</td><td>779.4453</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1957</td><td>30.332</td><td> 9240934</td><td>820.8530</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1962</td><td>31.997</td><td>10267083</td><td>853.1007</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1967</td><td>34.020</td><td>11537966</td><td>836.1971</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1972</td><td>36.088</td><td>13079460</td><td>739.9811</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>1987</td><td>62.351</td><td> 9216418</td><td>706.1573</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>1992</td><td>60.377</td><td>10704340</td><td>693.4208</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>1997</td><td>46.809</td><td>11404948</td><td>792.4500</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>2002</td><td>39.989</td><td>11926563</td><td>672.0386</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>2007</td><td>43.487</td><td>12311143</td><td>469.7093</td></tr>
</tbody>
</table>

Did it work?

```R
gap %>%
    filter(country == "Kingdom of Cambodia")
```

<table class="dataframe">
<caption>A tibble: 12 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col><chr></th><th scope=col><chr></th><th scope=col><int></th><th scope=col><dbl></th><th scope=col><int></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Kingdom of Cambodia</td><td>Asia</td><td>1952</td><td>39.417</td><td>4693836</td><td>368.4693</td></tr>
	<tr><td>Kingdom of Cambodia</td><td>Asia</td><td>1957</td><td>41.366</td><td>5322536</td><td>434.0383</td></tr>
	<tr><td>Kingdom of Cambodia</td><td>Asia</td><td>1962</td><td>43.415</td><td>6083619</td><td>496.9136</td></tr>
	<tr><td>Kingdom of Cambodia</td><td>Asia</td><td>1967</td><td>45.415</td><td>6960067</td><td>523.4323</td></tr>
	<tr><td>Kingdom of Cambodia</td><td>Asia</td><td>1972</td><td>40.317</td><td>7450606</td><td>421.6240</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>Kingdom of Cambodia</td><td>Asia</td><td>1987</td><td>53.914</td><td> 8371791</td><td> 683.8956</td></tr>
	<tr><td>Kingdom of Cambodia</td><td>Asia</td><td>1992</td><td>55.803</td><td>10150094</td><td> 682.3032</td></tr>
	<tr><td>Kingdom of Cambodia</td><td>Asia</td><td>1997</td><td>56.534</td><td>11782962</td><td> 734.2852</td></tr>
	<tr><td>Kingdom of Cambodia</td><td>Asia</td><td>2002</td><td>56.752</td><td>12926707</td><td> 896.2260</td></tr>
	<tr><td>Kingdom of Cambodia</td><td>Asia</td><td>2007</td><td>59.723</td><td>14131858</td><td>1713.7787</td></tr>
</tbody>
</table>

#### Selectively change two or more values with `case_when`

`case_when` will also let us easily modify > 2 cases. What if, for example, we wanted to change the continents from their English name to their French name?

```R
french_contients <- gap %>% 
  mutate(continent = case_when(continent == "Asia" ~ "Asie", 
                               continent == "Europe" ~ "L'Europe",
                               continent == "Africa" ~ "Afrique", 
                               continent == "Americas" ~ "les amériques", 
                               continent == "Oceania" ~ "Océanie"))
head(french_contients)
```

<table class="dataframe">
<caption>A tibble: 6 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col><chr></th><th scope=col><chr></th><th scope=col><int></th><th scope=col><dbl></th><th scope=col><int></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Afghanistan</td><td>Asie</td><td>1952</td><td>28.801</td><td> 8425333</td><td>779.4453</td></tr>
	<tr><td>Afghanistan</td><td>Asie</td><td>1957</td><td>30.332</td><td> 9240934</td><td>820.8530</td></tr>
	<tr><td>Afghanistan</td><td>Asie</td><td>1962</td><td>31.997</td><td>10267083</td><td>853.1007</td></tr>
	<tr><td>Afghanistan</td><td>Asie</td><td>1967</td><td>34.020</td><td>11537966</td><td>836.1971</td></tr>
	<tr><td>Afghanistan</td><td>Asie</td><td>1972</td><td>36.088</td><td>13079460</td><td>739.9811</td></tr>
	<tr><td>Afghanistan</td><td>Asie</td><td>1977</td><td>38.438</td><td>14880372</td><td>786.1134</td></tr>
</tbody>
</table>

What if we don't want to change Asia?

```R
french_contients <- gap %>% 
  mutate(continent = case_when(#continent == "Asia" ~ "Asie", 
                               continent == "Europe" ~ "L'Europe",
                               continent == "Africa" ~ "Afrique", 
                               continent == "Americas" ~ "les amériques", 
                               continent == "Oceania" ~ "Océanie"))
head(french_contients)
```

<table class="dataframe">
<caption>A tibble: 6 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col><chr></th><th scope=col><chr></th><th scope=col><int></th><th scope=col><dbl></th><th scope=col><int></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Afghanistan</td><td>NA</td><td>1952</td><td>28.801</td><td> 8425333</td><td>779.4453</td></tr>
	<tr><td>Afghanistan</td><td>NA</td><td>1957</td><td>30.332</td><td> 9240934</td><td>820.8530</td></tr>
	<tr><td>Afghanistan</td><td>NA</td><td>1962</td><td>31.997</td><td>10267083</td><td>853.1007</td></tr>
	<tr><td>Afghanistan</td><td>NA</td><td>1967</td><td>34.020</td><td>11537966</td><td>836.1971</td></tr>
	<tr><td>Afghanistan</td><td>NA</td><td>1972</td><td>36.088</td><td>13079460</td><td>739.9811</td></tr>
	<tr><td>Afghanistan</td><td>NA</td><td>1977</td><td>38.438</td><td>14880372</td><td>786.1134</td></tr>
</tbody>
</table>

Uh oh, now Asia is NA?? We need to say `TRUE ~ column_name` to tell R to not put in NA's in the non-specified class, but instead leave the values that were already there:

```R
french_contients <- gap %>% 
  mutate(continent = case_when(#continent == "Asia" ~ "Asie", 
                               continent == "Europe" ~ "L'Europe",
                               continent == "Africa" ~ "Afrique", 
                               continent == "Americas" ~ "les amériques", 
                               continent == "Oceania" ~ "Océanie", 
                               TRUE ~ continent))
head(french_contients)
```

<table class="dataframe">
<caption>A tibble: 6 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col><chr></th><th scope=col><chr></th><th scope=col><int></th><th scope=col><dbl></th><th scope=col><int></th><th scope=col><dbl></th></tr>
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

### Removing rows where a specific column holds NAs

Here, we'd like to remove the rows where the column `x` is `NA`:

```R
df <- tibble(x = c(3, 1, 2, NA), y = c("z", "a", NA, "b"))
df
```

<table class="dataframe">
<caption>A tibble: 4 × 2</caption>
<thead>
	<tr><th scope=col>x</th><th scope=col>y</th></tr>
	<tr><th scope=col><dbl></th><th scope=col><chr></th></tr>
</thead>
<tbody>
	<tr><td> 3</td><td>z </td></tr>
	<tr><td> 1</td><td>a </td></tr>
	<tr><td> 2</td><td>NA</td></tr>
	<tr><td>NA</td><td>b </td></tr>
</tbody>
</table>

`drop_na` is the tidyverse function we can use to do this.

In this case we pass the column for which we would like to use to drive the removal of rows (based on the presence of NAs in that row):

```R
df %>% drop_na(x:y)
```

<table class="dataframe">
<caption>A tibble: 2 × 2</caption>
<thead>
	<tr><th scope=col>x</th><th scope=col>y</th></tr>
	<tr><th scope=col><dbl></th><th scope=col><chr></th></tr>
</thead>
<tbody>
	<tr><td>3</td><td>z</td></tr>
	<tr><td>1</td><td>a</td></tr>
</tbody>
</table>

#### Removing all rows where there is a NA in any column

If no arguments are given to `drop_na`, then all rows with NAs will be dropped:

```R
df %>% drop_na()
```

<table class="dataframe">
<caption>A tibble: 2 × 2</caption>
<thead>
	<tr><th scope=col>x</th><th scope=col>y</th></tr>
	<tr><th scope=col><dbl></th><th scope=col><chr></th></tr>
</thead>
<tbody>
	<tr><td>3</td><td>z</td></tr>
	<tr><td>1</td><td>a</td></tr>
</tbody>
</table>

## Iterate over groups of rows

### `summarise` calculates summaries over rows

Examples might be calculating the mean horsepower and mean miles per gallon of cars in the `mtcars` data set:

```R
# calculate mean hp and mpg for all cars
mtcars %>% 
    summarise(mean_hp = mean(hp),
             mean_mpg = mean(mpg),
             sum_hp = sum(hp))
```

<table class="dataframe">
<caption>A data.frame: 1 × 3</caption>
<thead>
	<tr><th scope=col>mean_hp</th><th scope=col>mean_mpg</th><th scope=col>sum_hp</th></tr>
	<tr><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>146.6875</td><td>20.09062</td><td>4694</td></tr>
</tbody>
</table>

### Iteration with `group_by` + `summarise`

- Useful when you want to do something repeatedly to a group of rows
- An example, say we want to calculate the average life expectancy (`lifeExp`) for each continent from the gapminder data set

```R
# calculate the average life expectancy for each continent
gapminder %>% 
    group_by(continent) %>% 
    summarise(mean_life_exp = mean(lifeExp))
```

<table class="dataframe">
<caption>A tibble: 5 × 2</caption>
<thead>
	<tr><th scope=col>continent</th><th scope=col>mean_life_exp</th></tr>
	<tr><th scope=col><fct></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Africa  </td><td>48.86533</td></tr>
	<tr><td>Americas</td><td>64.65874</td></tr>
	<tr><td>Asia    </td><td>60.06490</td></tr>
	<tr><td>Europe  </td><td>71.90369</td></tr>
	<tr><td>Oceania </td><td>74.32621</td></tr>
</tbody>
</table>

We can also `group_by` multiple columns, for example what if we want to know the mean life expectancy for each continent and each year:

```R
# calculate the mean life expectancy for each continent and each year
gapminder %>% 
    group_by(continent, year) %>% 
    summarise(mean_life_exp = mean(lifeExp))
```

    [1m[22m`summarise()` has grouped output by 'continent'. You can override using the `.groups` argument.

<table class="dataframe">
<caption>A grouped_df: 60 × 3</caption>
<thead>
	<tr><th scope=col>continent</th><th scope=col>year</th><th scope=col>mean_life_exp</th></tr>
	<tr><th scope=col><fct></th><th scope=col><int></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Africa</td><td>1952</td><td>39.13550</td></tr>
	<tr><td>Africa</td><td>1957</td><td>41.26635</td></tr>
	<tr><td>Africa</td><td>1962</td><td>43.31944</td></tr>
	<tr><td>Africa</td><td>1967</td><td>45.33454</td></tr>
	<tr><td>Africa</td><td>1972</td><td>47.45094</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>Oceania</td><td>1987</td><td>75.3200</td></tr>
	<tr><td>Oceania</td><td>1992</td><td>76.9450</td></tr>
	<tr><td>Oceania</td><td>1997</td><td>78.1900</td></tr>
	<tr><td>Oceania</td><td>2002</td><td>79.7400</td></tr>
	<tr><td>Oceania</td><td>2007</td><td>80.7195</td></tr>
</tbody>
</table>

### Watch those NA's!

Calculate the maximum beak length for each of the Penguin species in the {palmerpenguins} `penguins` dataset.

```R
library(palmerpenguins)
penguins
```

<table class="dataframe">
<caption>A tibble: 344 × 8</caption>
<thead>
	<tr><th scope=col>species</th><th scope=col>island</th><th scope=col>bill_length_mm</th><th scope=col>bill_depth_mm</th><th scope=col>flipper_length_mm</th><th scope=col>body_mass_g</th><th scope=col>sex</th><th scope=col>year</th></tr>
	<tr><th scope=col><fct></th><th scope=col><fct></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><int></th><th scope=col><int></th><th scope=col><fct></th><th scope=col><int></th></tr>
</thead>
<tbody>
	<tr><td>Adelie</td><td>Torgersen</td><td>39.1</td><td>18.7</td><td>181</td><td>3750</td><td>male  </td><td>2007</td></tr>
	<tr><td>Adelie</td><td>Torgersen</td><td>39.5</td><td>17.4</td><td>186</td><td>3800</td><td>female</td><td>2007</td></tr>
	<tr><td>Adelie</td><td>Torgersen</td><td>40.3</td><td>18.0</td><td>195</td><td>3250</td><td>female</td><td>2007</td></tr>
	<tr><td>Adelie</td><td>Torgersen</td><td>  NA</td><td>  NA</td><td> NA</td><td>  NA</td><td>NA    </td><td>2007</td></tr>
	<tr><td>Adelie</td><td>Torgersen</td><td>36.7</td><td>19.3</td><td>193</td><td>3450</td><td>female</td><td>2007</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>Chinstrap</td><td>Dream</td><td>55.8</td><td>19.8</td><td>207</td><td>4000</td><td>male  </td><td>2009</td></tr>
	<tr><td>Chinstrap</td><td>Dream</td><td>43.5</td><td>18.1</td><td>202</td><td>3400</td><td>female</td><td>2009</td></tr>
	<tr><td>Chinstrap</td><td>Dream</td><td>49.6</td><td>18.2</td><td>193</td><td>3775</td><td>male  </td><td>2009</td></tr>
	<tr><td>Chinstrap</td><td>Dream</td><td>50.8</td><td>19.0</td><td>210</td><td>4100</td><td>male  </td><td>2009</td></tr>
	<tr><td>Chinstrap</td><td>Dream</td><td>50.2</td><td>18.7</td><td>198</td><td>3775</td><td>female</td><td>2009</td></tr>
</tbody>
</table>

```R
penguins %>%
    group_by(species) %>%
    summarise(max_bill_length = max(bill_length_mm))
```

<table class="dataframe">
<caption>A tibble: 3 × 2</caption>
<thead>
	<tr><th scope=col>species</th><th scope=col>max_bill_length</th></tr>
	<tr><th scope=col><fct></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Adelie   </td><td>NA</td></tr>
	<tr><td>Chinstrap</td><td>58</td></tr>
	<tr><td>Gentoo   </td><td>NA</td></tr>
</tbody>
</table>

Huh???

By default, summary statistics functions in R return `NA` if there are any `NA` observations in the data... This is often not ideal, what we'd like would be for R to just ignore the `NA`'s and kindly return us the statistic we asked for. We can specify this by setting the `na.rm` argument to `TRUE`.

```R
penguins %>%
    group_by(species) %>%
    summarise(max_bill_length = max(bill_length_mm, na.rm = TRUE))
```

<table class="dataframe">
<caption>A tibble: 3 × 2</caption>
<thead>
	<tr><th scope=col>species</th><th scope=col>max_bill_length</th></tr>
	<tr><th scope=col><fct></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Adelie   </td><td>46.0</td></tr>
	<tr><td>Chinstrap</td><td>58.0</td></tr>
	<tr><td>Gentoo   </td><td>59.6</td></tr>
</tbody>
</table>

### Grouped mutate

Sometimes you don’t want to collapse the n rows for each group into one row. You want to keep your groups, but compute within them.

Let’s make a new variable that is the years of life expectancy gained (lost) relative to 1952, for each individual country. We group by `country` and use `mutate` to make a new variable. The `first` function extracts the first value from a vector. Notice that `first` is operating on the vector of life expectancies within each country group.

```R
# calculate life expectancy gained (or lost) relative to 1952
gapminder %>% 
    group_by(country) %>% 
    mutate(life_exp_gain = lifeExp - first(lifeExp)) %>% 
    head()
```

<table class="dataframe">
<caption>A grouped_df: 6 × 7</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th><th scope=col>life_exp_gain</th></tr>
	<tr><th scope=col><fct></th><th scope=col><fct></th><th scope=col><int></th><th scope=col><dbl></th><th scope=col><int></th><th scope=col><dbl></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Afghanistan</td><td>Asia</td><td>1952</td><td>28.801</td><td> 8425333</td><td>779.4453</td><td>0.000</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1957</td><td>30.332</td><td> 9240934</td><td>820.8530</td><td>1.531</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1962</td><td>31.997</td><td>10267083</td><td>853.1007</td><td>3.196</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1967</td><td>34.020</td><td>11537966</td><td>836.1971</td><td>5.219</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1972</td><td>36.088</td><td>13079460</td><td>739.9811</td><td>7.287</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1977</td><td>38.438</td><td>14880372</td><td>786.1134</td><td>9.637</td></tr>
</tbody>
</table>

## Purrring instead of for loops

![](https://ih1.redbubble.net/image.329884292.2339/sticker,375x360-bg,ffffff.u1.png)

https://purrr.tidyverse.org/

####  `map_*` functions

- alternative to `for` loops that help you make less syntax errors

#### If you have programmed in R before

`purrr` is an alternative to "apply" functions

`purrr::map()` ≈ `base::lapply()`

### Iterating over columns of a data frame

Say, for example we wanted to calculate the median for each column in the `mtcars` data frame:

```R
head(mtcars)
```

<table class="dataframe">
<caption>A data.frame: 6 × 11</caption>
<thead>
	<tr><th></th><th scope=col>mpg</th><th scope=col>cyl</th><th scope=col>disp</th><th scope=col>hp</th><th scope=col>drat</th><th scope=col>wt</th><th scope=col>qsec</th><th scope=col>vs</th><th scope=col>am</th><th scope=col>gear</th><th scope=col>carb</th></tr>
	<tr><th></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><th scope=row>Mazda RX4</th><td>21.0</td><td>6</td><td>160</td><td>110</td><td>3.90</td><td>2.620</td><td>16.46</td><td>0</td><td>1</td><td>4</td><td>4</td></tr>
	<tr><th scope=row>Mazda RX4 Wag</th><td>21.0</td><td>6</td><td>160</td><td>110</td><td>3.90</td><td>2.875</td><td>17.02</td><td>0</td><td>1</td><td>4</td><td>4</td></tr>
	<tr><th scope=row>Datsun 710</th><td>22.8</td><td>4</td><td>108</td><td> 93</td><td>3.85</td><td>2.320</td><td>18.61</td><td>1</td><td>1</td><td>4</td><td>1</td></tr>
	<tr><th scope=row>Hornet 4 Drive</th><td>21.4</td><td>6</td><td>258</td><td>110</td><td>3.08</td><td>3.215</td><td>19.44</td><td>1</td><td>0</td><td>3</td><td>1</td></tr>
	<tr><th scope=row>Hornet Sportabout</th><td>18.7</td><td>8</td><td>360</td><td>175</td><td>3.15</td><td>3.440</td><td>17.02</td><td>0</td><td>0</td><td>3</td><td>2</td></tr>
	<tr><th scope=row>Valiant</th><td>18.1</td><td>6</td><td>225</td><td>105</td><td>2.76</td><td>3.460</td><td>20.22</td><td>1</td><td>0</td><td>3</td><td>1</td></tr>
</tbody>
</table>

```R
medians <- vector("double", ncol(mtcars))
for (i in seq_along(mtcars)) {
    medians[i] <- median(mtcars[[i]], na.rm = TRUE)
}
```

OK, then next we want to calculate the mean for all of the columns:

```R
means <- vector("double", ncol(mtcars))
for (i in seq_along(mtcars)) {
    means[i] <- mean(mtcars[[i]], na.rm = TRUE)
}
```

OK, and then the variance...

```R
variances <- vector("double", ncol(mtcars))
for (i in seq_along(mtcars)) {
    variances[i] <- var(mtcars[[i]], na.rm = TRUE)
}
```

This is getting a little repetitive... What are we repeating?

#### Can we write this as a function?

Given that functions are objects in R, this seems reasonable!

```R
medians <- vector("double", ncol(mtcars))
for (i in seq_along(mtcars)) {
    medians[i] <- median(mtcars[[i]], na.rm = TRUE)
}
```

This is essentially the guts of `purrr::map_dbl`. The only difference is that is coded in C and the use of `...` for additional arguments.

```R
mds_map <- function(x, fun)  {
    out <- vector("double", ncol(x))
    for (i in seq_along(x)) {
        out[i] <- fun(x[[i]], na.rm = TRUE)
    }
    out
}
mds_map(mtcars, min)
```



<ol class=list-inline><li>10.4</li><li>4</li><li>71.1</li><li>52</li><li>2.76</li><li>1.513</li><li>14.5</li><li>0</li><li>0</li><li>3</li><li>1</li></ol>

### Functionals

We have just written what is called a functional.

A functional is a function that takes a function (and other things) as an input and returns a vector as output.

R has several other functionals outside of `purrr` that you might have already encountered: `lapply`, `apply`, `tapply`, `integrate` or `optim`.

### What can you do with functionals?

- Common use is as an alternative to for loops
- For loops are actually quite effective for iteration, and efficient when used, however it is easy to make mistakes when setting them up as you have to:

  - pre-allocate space for the output
  - iterate over the thing the right amount of times
  - properly use the iteration index

#### *Of course someone has to write for loops*

#### *It doesn't have to be you*

-- *Jenny Bryan, Software Engineer at RStudio and UBC MDS Founder*

### The `purrr::map*` family of functions


*Source: [Advanced R](https://adv-r.hadley.nz/) by Hadley Wickham*

### Let's start at the beginning with the most general `purrr` function: `map`

```
map(.x, .f, ...)
```

Above reads as: `for` every element of `.x` apply `.f`

and can be pictured as:

<img src="https://d33wubrfki0l68.cloudfront.net/12f6af8404d9723dff9cc665028a35f07759299d/d0d9a/diagrams/functionals/map-list.png" width=500>

Or picture as...

<img src="img/minis_as_data.png" width=800>

*Source: [Row-oriented workflows in R with the tidyverse](https://speakerdeck.com/jennybc/row-oriented-workflows-in-r-with-the-tidyverse) by Jenny Bryan*

<img src="img/minis_map.png" width=800>

*Source: [Row-oriented workflows in R with the tidyverse](https://speakerdeck.com/jennybc/row-oriented-workflows-in-r-with-the-tidyverse) by Jenny Bryan*

### `purrr::map` test drive

Let's calculate the median of all the columns of the `mtcars` data frame using `purrr::map`:

```R
library(purrr)
map(mtcars, median)
```

<dl>
	<dt>$mpg</dt>
		<dd>19.2</dd>
	<dt>$cyl</dt>
		<dd>6</dd>
	<dt>$disp</dt>
		<dd>196.3</dd>
	<dt>$hp</dt>
		<dd>123</dd>
	<dt>$drat</dt>
		<dd>3.695</dd>
	<dt>$wt</dt>
		<dd>3.325</dd>
	<dt>$qsec</dt>
		<dd>17.71</dd>
	<dt>$vs</dt>
		<dd>0</dd>
	<dt>$am</dt>
		<dd>0</dd>
	<dt>$gear</dt>
		<dd>4</dd>
	<dt>$carb</dt>
		<dd>2</dd>
</dl>

That looks different from our `mds_map` function! The output is of type list. We can use `map_df` to get a vector of type double:

```R
map_dbl(mtcars, median)
```


<dl class=dl-inline><dt>mpg</dt><dd>19.2</dd><dt>cyl</dt><dd>6</dd><dt>disp</dt><dd>196.3</dd><dt>hp</dt><dd>123</dd><dt>drat</dt><dd>3.695</dd><dt>wt</dt><dd>3.325</dd><dt>qsec</dt><dd>17.71</dd><dt>vs</dt><dd>0</dd><dt>am</dt><dd>0</dd><dt>gear</dt><dd>4</dd><dt>carb</dt><dd>2</dd></dl>

And `map_df` will give us a tibble:

```R
map_df(mtcars, median)
```

<table class="dataframe">
<caption>A tibble: 1 × 11</caption>
<thead>
	<tr><th scope=col>mpg</th><th scope=col>cyl</th><th scope=col>disp</th><th scope=col>hp</th><th scope=col>drat</th><th scope=col>wt</th><th scope=col>qsec</th><th scope=col>vs</th><th scope=col>am</th><th scope=col>gear</th><th scope=col>carb</th></tr>
	<tr><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>19.2</td><td>6</td><td>196.3</td><td>123</td><td>3.695</td><td>3.325</td><td>17.71</td><td>0</td><td>0</td><td>4</td><td>2</td></tr>
</tbody>
</table>

So now it's super efficient and easy to get several summary stats across columns!

```R
map_df(mtcars, median)
map_df(mtcars, mean)
map_df(mtcars, max)
map_df(mtcars, min)
```

<table class="dataframe">
<caption>A tibble: 1 × 11</caption>
<thead>
	<tr><th scope=col>mpg</th><th scope=col>cyl</th><th scope=col>disp</th><th scope=col>hp</th><th scope=col>drat</th><th scope=col>wt</th><th scope=col>qsec</th><th scope=col>vs</th><th scope=col>am</th><th scope=col>gear</th><th scope=col>carb</th></tr>
	<tr><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>19.2</td><td>6</td><td>196.3</td><td>123</td><td>3.695</td><td>3.325</td><td>17.71</td><td>0</td><td>0</td><td>4</td><td>2</td></tr>
</tbody>
</table>

<table class="dataframe">
<caption>A tibble: 1 × 11</caption>
<thead>
	<tr><th scope=col>mpg</th><th scope=col>cyl</th><th scope=col>disp</th><th scope=col>hp</th><th scope=col>drat</th><th scope=col>wt</th><th scope=col>qsec</th><th scope=col>vs</th><th scope=col>am</th><th scope=col>gear</th><th scope=col>carb</th></tr>
	<tr><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>20.09062</td><td>6.1875</td><td>230.7219</td><td>146.6875</td><td>3.596563</td><td>3.21725</td><td>17.84875</td><td>0.4375</td><td>0.40625</td><td>3.6875</td><td>2.8125</td></tr>
</tbody>
</table>

<table class="dataframe">
<caption>A tibble: 1 × 11</caption>
<thead>
	<tr><th scope=col>mpg</th><th scope=col>cyl</th><th scope=col>disp</th><th scope=col>hp</th><th scope=col>drat</th><th scope=col>wt</th><th scope=col>qsec</th><th scope=col>vs</th><th scope=col>am</th><th scope=col>gear</th><th scope=col>carb</th></tr>
	<tr><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>33.9</td><td>8</td><td>472</td><td>335</td><td>4.93</td><td>5.424</td><td>22.9</td><td>1</td><td>1</td><td>5</td><td>8</td></tr>
</tbody>
</table>

<table class="dataframe">
<caption>A tibble: 1 × 11</caption>
<thead>
	<tr><th scope=col>mpg</th><th scope=col>cyl</th><th scope=col>disp</th><th scope=col>hp</th><th scope=col>drat</th><th scope=col>wt</th><th scope=col>qsec</th><th scope=col>vs</th><th scope=col>am</th><th scope=col>gear</th><th scope=col>carb</th></tr>
	<tr><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>10.4</td><td>4</td><td>71.1</td><td>52</td><td>2.76</td><td>1.513</td><td>14.5</td><td>0</td><td>0</td><td>3</td><td>1</td></tr>
</tbody>
</table>

#### What if our data frame had missing values?

Let's make some to see the consequences...

```R
mtcars_NA <- mtcars
mtcars_NA[1, 1] <- NA

map_dbl(mtcars_NA, median)
```

<dl class=dl-inline><dt>mpg</dt><dd><NA></dd><dt>cyl</dt><dd>6</dd><dt>disp</dt><dd>196.3</dd><dt>hp</dt><dd>123</dd><dt>drat</dt><dd>3.695</dd><dt>wt</dt><dd>3.325</dd><dt>qsec</dt><dd>17.71</dd><dt>vs</dt><dd>0</dd><dt>am</dt><dd>0</dd><dt>gear</dt><dd>4</dd><dt>carb</dt><dd>2</dd></dl>

How do we tell `median` to ignore NA's? Using `na.rm = TRUE`! But how do we add this to our `map_dbl` call?

```R
map_dbl(mtcars_NA, median, na.rm = TRUE)
```

<dl class=dl-inline><dt>mpg</dt><dd>19.2</dd><dt>cyl</dt><dd>6</dd><dt>disp</dt><dd>196.3</dd><dt>hp</dt><dd>123</dd><dt>drat</dt><dd>3.695</dd><dt>wt</dt><dd>3.325</dd><dt>qsec</dt><dd>17.71</dd><dt>vs</dt><dd>0</dd><dt>am</dt><dd>0</dd><dt>gear</dt><dd>4</dd><dt>carb</dt><dd>2</dd></dl>

That's all we will learn of the {purrr} functions for now - however we will meet them again (and learn more about them) next week when we start working with nested data frames.

### What did we learn today?

- How to use `case_when` to selectively change values in a data frame (similar to base R `if` statements)
- How to use `group_by` to iterate over groups of rows (similar to `for` loops in base R)
- How to use {purrr} `map_*` functions to iterate over columns (similar to `apply` in base R)

### Attributions

- [Stat 545](https://stat545.com/) created by Jenny Bryan
- [R for Data Science](https://r4ds.had.co.nz/index.html) by Garrett Grolemund & Hadley Wickham

```R

```