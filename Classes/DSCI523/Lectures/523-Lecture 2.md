# Lecture 2 - Key datatypes & operators in R

  
[[523-Lab 1]]
[[523-Worksheet 2]]
### Lecture learning objectives:

  

By the end of this lecture and worksheet 2, students should be able to:

  

* Explain how the assignment symbol, `<-` differs from `=` in R

* Create in R, and define and differentiate in English, the below listed key datatypes in R:

- logical, numeric, character and factor vectors

- lists

- data frames and tibbles

* Use R to determine the type and structure of an object

* Explain the distinction between names and values, and when R will copy an object.

* Use the three subsetting operators, `[[`, `[`, and `$`, to subset single and multiple elements from vectors and data frames, lists and matrices

* Compute numeric and boolean values using their respective types and operations

  

### Getting help in R

  

No one, even experienced, professional programmers remember what every function does, nor do they remember every possible function argument/option. So both experienced and new programmers ( like you! ) need to look things up, A LOT!

  

One of the most efficient places to look for help on how a function works is the R help files. Let’s say we wanted to pull up the help file for the `max()` function. We can do this by typing a question mark in front of the function we want to know more about.

  
  

```R

?max

```

  
  
  

<table style="width: 100%;"><tr><td>Extremes {base}</td><td style="text-align: right;">R Documentation</td></tr></table>

  

<h2>Maxima and Minima</h2>

  

<h3>Description</h3>

  

<p>Returns the (regular or <b>p</b>arallel) maxima and minima of the

input values.

</p>

<p><code>pmax*()</code> and <code>pmin*()</code> take one or more vectors as

arguments, recycle them to common length and return a single vector

giving the <em>&lsquo;parallel&rsquo;</em> maxima (or minima) of the argument

vectors.

</p>

  
  

<h3>Usage</h3>

  

<pre><code class='language-R'>max(..., na.rm = FALSE)

min(..., na.rm = FALSE)

  

pmax(..., na.rm = FALSE)

pmin(..., na.rm = FALSE)

  

pmax.int(..., na.rm = FALSE)

pmin.int(..., na.rm = FALSE)

</code></pre>

  
  

<h3>Arguments</h3>

  

<table>

<tr style="vertical-align: top;"><td><code>...</code></td>

<td>

<p>numeric or character arguments (see Note).</p>

</td></tr>

<tr style="vertical-align: top;"><td><code>na.rm</code></td>

<td>

<p>a logical indicating whether missing values should be

removed.</p>

</td></tr>

</table>

  
  

<h3>Details</h3>

  

<p><code>max</code> and <code>min</code> return the maximum or minimum of <em>all</em>

the values present in their arguments, as <code>integer</code> if

all are <code>logical</code> or <code>integer</code>, as <code>double</code> if

all are numeric, and character otherwise.

</p>

<p>If <code>na.rm</code> is <code>FALSE</code> an <code>NA</code> value in any of the

arguments will cause a value of <code>NA</code> to be returned, otherwise

<code>NA</code> values are ignored.

</p>

<p>The minimum and maximum of a numeric empty set are <code>+Inf</code> and

<code>-Inf</code> (in this order!) which ensures <em>transitivity</em>, e.g.,

<code>min(x1, min(x2)) == min(x1, x2)</code>. For numeric <code>x</code>

<code>max(x) == -Inf</code> and <code>min(x) == +Inf</code>

whenever <code>length(x) == 0</code> (after removing missing values if

requested). However, <code>pmax</code> and <code>pmin</code> return

<code>NA</code> if all the parallel elements are <code>NA</code> even for

<code>na.rm = TRUE</code>.

</p>

<p><code>pmax</code> and <code>pmin</code> take one or more vectors (or matrices) as

arguments and return a single vector giving the &lsquo;parallel&rsquo;

maxima (or minima) of the vectors. The first element of the result is

the maximum (minimum) of the first elements of all the arguments, the

second element of the result is the maximum (minimum) of the second

elements of all the arguments and so on. Shorter inputs (of non-zero

length) are recycled if necessary. Attributes (see

<code>attributes</code>: such as <code>names</code> or

<code>dim</code>) are copied from the first argument (if applicable,

e.g., <em>not</em> for an <code>S4</code> object).

</p>

<p><code>pmax.int</code> and <code>pmin.int</code> are faster internal versions only

used when all arguments are atomic vectors and there are no classes:

they drop all attributes. (Note that all versions fail for raw and

complex vectors since these have no ordering.)

</p>

<p><code>max</code> and <code>min</code> are generic functions: methods can be

defined for them individually or via the

<code>Summary</code> group generic. For this to

work properly, the arguments <code>...</code> should be unnamed, and

dispatch is on the first argument.

</p>

<p>By definition the min/max of a numeric vector containing an <code>NaN</code>

is <code>NaN</code>, except that the min/max of any vector containing an

<code>NA</code> is <code>NA</code> even if it also contains an <code>NaN</code>.

Note that <code>max(NA, Inf) == NA</code> even though the maximum would be

<code>Inf</code> whatever the missing value actually is.

</p>

<p>Character versions are sorted lexicographically, and this depends on

the collating sequence of the locale in use: the help for

&lsquo;Comparison&rsquo; gives details. The max/min of an empty

character vector is defined to be character <code>NA</code>. (One could

argue that as <code>""</code> is the smallest character element, the maximum

should be <code>""</code>, but there is no obvious candidate for the

minimum.)

</p>

  
  

<h3>Value</h3>

  

<p>For <code>min</code> or <code>max</code>, a length-one vector. For <code>pmin</code> or

<code>pmax</code>, a vector of length the longest of the input vectors, or

length zero if one of the inputs had zero length.

</p>

<p>The type of the result will be that of the highest of the inputs in

the hierarchy integer &lt; double &lt; character.

</p>

<p>For <code>min</code> and <code>max</code> if there are only numeric inputs and all

are empty (after possible removal of <code>NA</code>s), the result is double

(<code>Inf</code> or <code>-Inf</code>).

</p>

  
  

<h3>S4 methods</h3>

  

<p><code>max</code> and <code>min</code> are part of the S4

<code>Summary</code> group generic. Methods

for them must use the signature <code>x, ..., na.rm</code>.

</p>

  
  

<h3>Note</h3>

  

<p>&lsquo;Numeric&rsquo; arguments are vectors of type integer and numeric,

and logical (coerced to integer). For historical reasons, <code>NULL</code>

is accepted as equivalent to <code>integer(0)</code>.

</p>

<p><code>pmax</code> and <code>pmin</code> will also work on classed S3 or S4 objects

with appropriate methods for comparison, <code>is.na</code> and <code>rep</code>

(if recycling of arguments is needed).

</p>

  
  

<h3>References</h3>

  

<p>Becker, R. A., Chambers, J. M. and Wilks, A. R. (1988)

<em>The New S Language</em>.

Wadsworth &amp; Brooks/Cole.

</p>

  
  

<h3>See Also</h3>

  

<p><code>range</code> (<em>both</em> min and max) and

<code>which.min</code> (<code>which.max</code>) for the <em>arg min</em>,

i.e., the location where an extreme value occurs.

</p>

<p>&lsquo;plotmath&rsquo; for the use of <code>min</code> in plot annotation.

</p>

  
  

<h3>Examples</h3>

  

<pre><code class='language-R'>require(stats); require(graphics)

min(5:1, pi) #-&gt; one number

pmin(5:1, pi) #-&gt; 5 numbers

  

x &lt;- sort(rnorm(100)); cH &lt;- 1.35

pmin(cH, quantile(x)) # no names

pmin(quantile(x), cH) # has names

plot(x, pmin(cH, pmax(-cH, x)), type = "b", main = "Huber's function")

  

cut01 &lt;- function(x) pmax(pmin(x, 1), 0)

curve( x^2 - 1/4, -1.4, 1.5, col = 2)

curve(cut01(x^2 - 1/4), col = "blue", add = TRUE, n = 500)

## pmax(), pmin() preserve attributes of *first* argument

D &lt;- diag(x = (3:1)/4) ; n0 &lt;- numeric()

stopifnot(identical(D, cut01(D) ),

identical(n0, cut01(n0)),

identical(n0, cut01(NULL)),

identical(n0, pmax(3:1, n0, 2)),

identical(n0, pmax(n0, 4)))

</code></pre>

  

<hr /><div style="text-align: center;">[Package <em>base</em> version 4.2.1 ]</div>

</div>

  
  

At the very top of the file, you will see the function itself and the package it is in (in this case, it is base). Next is a description of what the function does. You’ll find that the most helpful sections on this page are “Usage”, “Arguments” and "Examples".

  

- **Usage** gives you an idea of how you would use the function when coding--what the syntax would be and how the function itself is structured.

- **Arguments** tells you the different parts that can be added to the function to make it more simple or more complicated. Often the “Usage” and “Arguments” sections don’t provide you with step by step instructions, because there are so many different ways that a person can incorporate a function into their code. Instead, they provide users with a general understanding as to what the function could do and parts that could be added. At the end of the day, the user must interpret the help file and figure out how best to use the functions and which parts are most important to include for their particular task.

- The **Examples** section is often the most useful part of the help file as it shows how a function could be used with real data. It provides a skeleton code that the users can work off of.

  

Below is a useful graphical summary of the help docs that might be useful to start getting you oriented to them:

  

*Source: https://socviz.co/appendix.html#a-little-more-about-r*

  

### The assignment symbol, `<-`

  

- R came from S, S used `<-`

- S was inspired from APL, which also used `<-`

- APL was designed on a specific keyboard, which had a key for `<-`

- At that time there was no `==` for testing equality, it was tested with `=`, so something else need to be used for assignment.

  



source: https://colinfay.me/r-assignment/

  

- Nowadays, `=` can also be used for assignment, however there are some things to be aware of...

  

- stylistically, `<-` is preferred over `=` for readability

  

- `<-` and `->` are valid in R, the latter can be useful in pipelines (more on this in data wrangling)

  

- `<-` and `=` have different emphasis in regards to environments

  

- **we expect you to use `<-` in MDS for object assignment in R**

  

#### Assignment readability

  

Consider this code:

  

```

c <- 12

d <- 13

```

  

Which equality is easier to read?

  

```

e = c == d

```

  

or

  

```

e <- c == d

```

  

#### Assignment environment

  

What value does x hold at the end of each of these code chunks?

  
  
  

```median(x = 1:10)```

  

vs

  

```median(x <- 1:10)```

  

Here, in the first example where `=` is used to set `x`, `x` only exists in the `median` function call, so we are returned the result from that function call, however, when we call `x` later, it does not exist and so R returns an error.

  

```

median(x = 1:10)

x

```

  

```

5.5

Error in eval(expr, envir, enclos): object 'x' not found

Traceback:

```

  

Here, in the second example where `<-` is used to set `x`, `x` exists in the median function call, **and** in the global environment (outside the `median` function call). So when we call `x` later, it **does** exist and so R returns the value that the name `x` is bound to.

  

```

median(x <- 1:10)

x

```

  
  

```

5.5

1 2 3 4 5 6 7 8 9 10

```

  

#### What does assignment do in R?

  

When you type this into R: `x <- c(1, 2, 3)`

  

This is what R does:

  

  

*Source: [Advanced R](https://adv-r.hadley.nz/) by Hadley Wickham*

  

What does this mean? It means that even if you don't bind a name to an object in R using ` <- `, it still exists somewhere in memory during the R session it was created in. This is typically not a problem unless your data sets are very large.

  

### A note on names

  

#### Rules for syntactic names:

- May use: letters, digits, `.` and `_`

- Cannot begin with `_` or a digit

- Cannot use reserved words (e.g., `for`, `if`, `return`)

  

#### How to manage non-syntactic names

- Usually come across these when reading in someone else's data

- Backticks, \`, can be used manage these cases (e.g., ``` `_abc` <- 1 ```)

- If your data contains these, use R to rename things to make them syntactic (for your future sanity)

  

### Key datatypes in R

  

  

*note - There are no scalars in R, they are represented by vectors of length 1.*

  

*Source: [Advanced R](https://adv-r.hadley.nz/) by Hadley Wickham*

  

- `NULL` is not a vector, but related and frequently functions in the role of a generic zero length vector.

  

#### What is a data frame?

  

From a data perspective, it is a rectangle where the rows are the observations:



#### What is a data frame?

  

and the columns are the variables:

  

#### What is a data frame?

  

From a computer programming perspective, in R, a data frame is a special subtype of a list object whose elements (columns) are vectors.

  

**Question:** What do you notice about the elements of each of the vectors in this data frame?

  

#### What is a vector?

  

- objects that can contain 1 or more elements

- elements are ordered

- must all be of the same type (e.g., double, integer, character, logical)

  
  

#### How are vectors different from a list?

  

#### Reminder: what do lists have to do with data frames?

  

### A bit more about Vectors

  

Your closest and most important friend in R

  


#### Creating vectors and vector types

  
  

```R

char_vec <- c("joy", "peace", "help", "fun", "sharing")

char_vec

typeof(char_vec)

```

  

  
  

'character'

  
  
  

```R

log_vec <- c(TRUE, TRUE, FALSE, FALSE, TRUE)

log_vec

typeof(log_vec)

```

  

  
  
  
  

'logical'

  
  
  

```R

double_vec <- c(1, 2, 3, 4, 5)

double_vec

typeof(double_vec)

```

  
'double'


```R

int_vec <- c(1L, 2L, 3L, 4L, 5L)

int_vec

typeof(int_vec)

```

  
  

'integer'

  
  

`str` is a useful command to get even more information about an object:

  
  

```R

str(int_vec)

```

  

int [1:5] 1 2 3 4 5

  
  

#### What happens to vectors of mixed type?

  
  
  
  

```R

mixed_vec <- c("joy", 5.6, TRUE, 1L, "sharing")

typeof(mixed_vec)

```

  
  

'character'

  
  

Hierarchy for coercion:

  

character → double → integer → logical

  

#### Useful functions for testing type and forcing coercion:

  

- `is.logical()`, `is.integer()`, `is.double()`, and `is.character()` returns `TRUE` or `FALSE`, depending on type of object and function used.

- `as.logical()`, `as.integer()`,` as.double()`, or `as.character()` coerce vector to type specified by function name.

  

#### How to subset and modify vectors

  

#### Subsetting

  

- R counts from 1!!!

  
  

```R

name <- c("T", "i", "f", "f", "a", "n", "y")

```

  

What letter will I get in R? What would I get in Python?

  
  

```R

name[2]

```

  
  

'i'

  
  

What letters will I get in R? What would I get in Python?

  

```

name <- c("T", "i", "f", "f", "a", "n", "y")

```

  
  

```R

name[2:4]

```

 
  

What letter will I get in R? What would I get in Python?

  

```

name <- c("T", "i", "f", "f", "a", "n", "y")

```

  
  

```R

name[-1]

```

  

  

How do I get the last element in a vector in R?

  

```

name <- c("T", "i", "f", "f", "a", "n", "y")

```

  
  

```R

name[length(name)]

```

  
  

'y'

  
  

#### Modifing vectors

  

We can combine the assignment symbol and subsetting to modify vectors:

  

```

name <- c("T", "i", "f", "f", "a", "n", "y")

```

  
  

```R

name[1] <- "t"

name

```


  
  

This can be done for more than one element:

  
  

```R

name[1:3] <- c("T", "I", "F")

name

```

  
  
  

What if you ask for elements that are not there?

  
  

```R

name[8:12]

```

  


This syntax also lets you add additional elements:

  
  

```R

name[8:12] <- c("-", "A", "n", "n", "e")

name

```

  
 
  
#### What happens when you modify a vector in R?

  

Consider:

  

```

x <- c(1, 2, 3)

y <- x

  

y[3] <- 4

y

#> [1] 1 2 4

```

  

What is happening in R's memory for each line of code?

  
  

|Code | R's memory representation |


  

This is called "copy-on-modify".

  

*Source: [Advanced R](https://adv-r.hadley.nz/) by Hadley Wickham*

  

#### Why copy-on-modify

- Since there are no scalars in R, vectors are essentially immutable

- If you change one element of the vector, you have to copy the whole thing to update it

  

#### Why do we care about knowing this?

- Given that data frames are built on-top of vectors, this has implications for speed when working with large data frames

  

#### Why vectors?

  

Vectorized operations!

  
  

```R

c(1, 2, 3, 4) + c(1, 1, 1, 1)

```

  
 

But watch out for vector recycling in R!

  

This makes sense:

  
  

```R

c(1, 2, 3, 4) + c(1)

```

  
  

but this does not!

  
  

```R

c(1, 2, 3, 4) + c(1, 2)

```

  
A list of vector operators here: [R Operators cheat sheet](https://cran.r-project.org/doc/contrib/Baggott-refcard-v2.pdf)

  

One to watch out for, logical and (`&`) and or (`|`) operators come in both an elementwise and first element comparison form, for example:

  
  

```R

# compares each elements of each vector by position

c(TRUE, TRUE, TRUE) & c(FALSE, TRUE, TRUE)

```

  
  
  
  

```R

# compares only the first elements of each vector

c(TRUE, TRUE, TRUE) && c(FALSE, TRUE, TRUE)

```

  

Warning message in c(TRUE, TRUE, TRUE) && c(FALSE, TRUE, TRUE):

“'length(x) = 3 > 1' in coercion to 'logical(1)'”

Warning message in c(TRUE, TRUE, TRUE) && c(FALSE, TRUE, TRUE):

“'length(x) = 3 > 1' in coercion to 'logical(1)'”

  
  
  

FALSE

  
  

### Extending our knowledge to data frames

  
  

#### Getting to know a data frame

  
  

```R

head(mtcars)

```

  
  

<table class="dataframe">

<caption>A data.frame: 6 × 11</caption>

<thead>

<tr><th></th><th scope=col>mpg</th><th scope=col>cyl</th><th scope=col>disp</th><th scope=col>hp</th><th scope=col>drat</th><th scope=col>wt</th><th scope=col>qsec</th><th scope=col>vs</th><th scope=col>am</th><th scope=col>gear</th><th scope=col>carb</th></tr>

<tr><th></th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>

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

str(mtcars)

```

  

'data.frame': 32 obs. of 11 variables:

$ mpg : num 21 21 22.8 21.4 18.7 18.1 14.3 24.4 22.8 19.2 ...

$ cyl : num 6 6 4 6 8 6 8 4 4 6 ...

$ disp: num 160 160 108 258 360 ...

$ hp : num 110 110 93 110 175 105 245 62 95 123 ...

$ drat: num 3.9 3.9 3.85 3.08 3.15 2.76 3.21 3.69 3.92 3.92 ...

$ wt : num 2.62 2.88 2.32 3.21 3.44 ...

$ qsec: num 16.5 17 18.6 19.4 17 ...

$ vs : num 0 0 1 1 0 1 0 1 1 1 ...

$ am : num 1 1 1 0 0 0 0 0 0 0 ...

$ gear: num 4 4 4 3 3 3 3 4 4 4 ...

$ carb: num 4 4 1 1 2 1 4 2 2 4 ...

  
  

#### Subsetting and modifying data frames

  

There are 3 operators that can be used when subsetting data frames: `[`, `$` and `[[`

  

| Operator | Example use | What it returns |

|----------|-------------|-----------------|

| `[` | `mtcars[1:10, 2:4]` | rows 1-10 for columns 2-4 of the data frame, as a data frame |

| `[` | `mtcars[1:10, ]` | rows 1-10 for all columns of the data frame, as a data frame |

| `[` | `mtcars[1]` | the first column of the data frame, as a data frame |

| `[[` | `mtcars[[1]]` | the first column of the data frame, as a vector |

| `$` | `mtcars$cyl` | the column the corresponds to the name that follows the `$`, as a vector |

  

Note that `$` and `[[` remove a level of structure from the data frame object (this happens with lists too).

  

### Other R objects

  

We are focusing on vectors and data frames in this lecture because these are the objects you will encounter most frequently in R for data science. These subsetting (and modification) syntax also work on other objects in R, in the same way.

  

Examples that you will encounter in the worksheet and lab are matrices and lists.

  

#### Logical indexing of data frames

  

We can also use logical statements to filter for rows containing certain values, or values above or below a threshold. For example, if we want to filter for rows where the cylinder value in the `cyl` column is 6 in the mtcars data frame shown below:

  
  

```R

options(repr.matrix.max.rows = 10) # limit number of rows that are output

mtcars

```

  
  

<table class="dataframe">

<caption>A data.frame: 32 × 11</caption>

<thead>

<tr><th></th><th scope=col>mpg</th><th scope=col>cyl</th><th scope=col>disp</th><th scope=col>hp</th><th scope=col>drat</th><th scope=col>wt</th><th scope=col>qsec</th><th scope=col>vs</th><th scope=col>am</th><th scope=col>gear</th><th scope=col>carb</th></tr>

<tr><th></th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>

</thead>

<tbody>

<tr><th scope=row>Mazda RX4</th><td>21.0</td><td>6</td><td>160</td><td>110</td><td>3.90</td><td>2.620</td><td>16.46</td><td>0</td><td>1</td><td>4</td><td>4</td></tr>

<tr><th scope=row>Mazda RX4 Wag</th><td>21.0</td><td>6</td><td>160</td><td>110</td><td>3.90</td><td>2.875</td><td>17.02</td><td>0</td><td>1</td><td>4</td><td>4</td></tr>

<tr><th scope=row>Datsun 710</th><td>22.8</td><td>4</td><td>108</td><td> 93</td><td>3.85</td><td>2.320</td><td>18.61</td><td>1</td><td>1</td><td>4</td><td>1</td></tr>

<tr><th scope=row>Hornet 4 Drive</th><td>21.4</td><td>6</td><td>258</td><td>110</td><td>3.08</td><td>3.215</td><td>19.44</td><td>1</td><td>0</td><td>3</td><td>1</td></tr>

<tr><th scope=row>Hornet Sportabout</th><td>18.7</td><td>8</td><td>360</td><td>175</td><td>3.15</td><td>3.440</td><td>17.02</td><td>0</td><td>0</td><td>3</td><td>2</td></tr>

<tr><th scope=row>⋮</th><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>

<tr><th scope=row>Lotus Europa</th><td>30.4</td><td>4</td><td> 95.1</td><td>113</td><td>3.77</td><td>1.513</td><td>16.9</td><td>1</td><td>1</td><td>5</td><td>2</td></tr>

<tr><th scope=row>Ford Pantera L</th><td>15.8</td><td>8</td><td>351.0</td><td>264</td><td>4.22</td><td>3.170</td><td>14.5</td><td>0</td><td>1</td><td>5</td><td>4</td></tr>

<tr><th scope=row>Ferrari Dino</th><td>19.7</td><td>6</td><td>145.0</td><td>175</td><td>3.62</td><td>2.770</td><td>15.5</td><td>0</td><td>1</td><td>5</td><td>6</td></tr>

<tr><th scope=row>Maserati Bora</th><td>15.0</td><td>8</td><td>301.0</td><td>335</td><td>3.54</td><td>3.570</td><td>14.6</td><td>0</td><td>1</td><td>5</td><td>8</td></tr>

<tr><th scope=row>Volvo 142E</th><td>21.4</td><td>4</td><td>121.0</td><td>109</td><td>4.11</td><td>2.780</td><td>18.6</td><td>1</td><td>1</td><td>4</td><td>2</td></tr>

</tbody>

</table>

  
  
  
  

```R

mtcars[mtcars$cyl == 6, ]

```

  
  

<table class="dataframe">

<caption>A data.frame: 7 × 11</caption>

<thead>

<tr><th></th><th scope=col>mpg</th><th scope=col>cyl</th><th scope=col>disp</th><th scope=col>hp</th><th scope=col>drat</th><th scope=col>wt</th><th scope=col>qsec</th><th scope=col>vs</th><th scope=col>am</th><th scope=col>gear</th><th scope=col>carb</th></tr>

<tr><th></th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>

</thead>

<tbody>

<tr><th scope=row>Mazda RX4</th><td>21.0</td><td>6</td><td>160.0</td><td>110</td><td>3.90</td><td>2.620</td><td>16.46</td><td>0</td><td>1</td><td>4</td><td>4</td></tr>

<tr><th scope=row>Mazda RX4 Wag</th><td>21.0</td><td>6</td><td>160.0</td><td>110</td><td>3.90</td><td>2.875</td><td>17.02</td><td>0</td><td>1</td><td>4</td><td>4</td></tr>

<tr><th scope=row>Hornet 4 Drive</th><td>21.4</td><td>6</td><td>258.0</td><td>110</td><td>3.08</td><td>3.215</td><td>19.44</td><td>1</td><td>0</td><td>3</td><td>1</td></tr>

<tr><th scope=row>Valiant</th><td>18.1</td><td>6</td><td>225.0</td><td>105</td><td>2.76</td><td>3.460</td><td>20.22</td><td>1</td><td>0</td><td>3</td><td>1</td></tr>

<tr><th scope=row>Merc 280</th><td>19.2</td><td>6</td><td>167.6</td><td>123</td><td>3.92</td><td>3.440</td><td>18.30</td><td>1</td><td>0</td><td>4</td><td>4</td></tr>

<tr><th scope=row>Merc 280C</th><td>17.8</td><td>6</td><td>167.6</td><td>123</td><td>3.92</td><td>3.440</td><td>18.90</td><td>1</td><td>0</td><td>4</td><td>4</td></tr>

<tr><th scope=row>Ferrari Dino</th><td>19.7</td><td>6</td><td>145.0</td><td>175</td><td>3.62</td><td>2.770</td><td>15.50</td><td>0</td><td>1</td><td>5</td><td>6</td></tr>

</tbody>

</table>

  
  
  

Another example:

  
  

```R

mtcars[mtcars$hp > 200, ]

```

  
  

<table class="dataframe">

<caption>A data.frame: 7 × 11</caption>

<thead>

<tr><th></th><th scope=col>mpg</th><th scope=col>cyl</th><th scope=col>disp</th><th scope=col>hp</th><th scope=col>drat</th><th scope=col>wt</th><th scope=col>qsec</th><th scope=col>vs</th><th scope=col>am</th><th scope=col>gear</th><th scope=col>carb</th></tr>

<tr><th></th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>

</thead>

<tbody>

<tr><th scope=row>Duster 360</th><td>14.3</td><td>8</td><td>360</td><td>245</td><td>3.21</td><td>3.570</td><td>15.84</td><td>0</td><td>0</td><td>3</td><td>4</td></tr>

<tr><th scope=row>Cadillac Fleetwood</th><td>10.4</td><td>8</td><td>472</td><td>205</td><td>2.93</td><td>5.250</td><td>17.98</td><td>0</td><td>0</td><td>3</td><td>4</td></tr>

<tr><th scope=row>Lincoln Continental</th><td>10.4</td><td>8</td><td>460</td><td>215</td><td>3.00</td><td>5.424</td><td>17.82</td><td>0</td><td>0</td><td>3</td><td>4</td></tr>

<tr><th scope=row>Chrysler Imperial</th><td>14.7</td><td>8</td><td>440</td><td>230</td><td>3.23</td><td>5.345</td><td>17.42</td><td>0</td><td>0</td><td>3</td><td>4</td></tr>

<tr><th scope=row>Camaro Z28</th><td>13.3</td><td>8</td><td>350</td><td>245</td><td>3.73</td><td>3.840</td><td>15.41</td><td>0</td><td>0</td><td>3</td><td>4</td></tr>

<tr><th scope=row>Ford Pantera L</th><td>15.8</td><td>8</td><td>351</td><td>264</td><td>4.22</td><td>3.170</td><td>14.50</td><td>0</td><td>1</td><td>5</td><td>4</td></tr>

<tr><th scope=row>Maserati Bora</th><td>15.0</td><td>8</td><td>301</td><td>335</td><td>3.54</td><td>3.570</td><td>14.60</td><td>0</td><td>1</td><td>5</td><td>8</td></tr>

</tbody>

</table>

  
  
  

#### Modifing data frames

  

Similar to vectors, we can combine the assignment symbol and subsetting to modify data frames.

  

For example, here we create a new column called `kml`:

  
  

```R

mtcars$kml <- mtcars$mpg / 2.3521458

head(mtcars)

```

  
  

<table class="dataframe">

<caption>A data.frame: 6 × 12</caption>

<thead>

<tr><th></th><th scope=col>mpg</th><th scope=col>cyl</th><th scope=col>disp</th><th scope=col>hp</th><th scope=col>drat</th><th scope=col>wt</th><th scope=col>qsec</th><th scope=col>vs</th><th scope=col>am</th><th scope=col>gear</th><th scope=col>carb</th><th scope=col>kml</th></tr>

<tr><th></th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>

</thead>

<tbody>

<tr><th scope=row>Mazda RX4</th><td>21.0</td><td>6</td><td>160</td><td>110</td><td>3.90</td><td>2.620</td><td>16.46</td><td>0</td><td>1</td><td>4</td><td>4</td><td>8.928018</td></tr>

<tr><th scope=row>Mazda RX4 Wag</th><td>21.0</td><td>6</td><td>160</td><td>110</td><td>3.90</td><td>2.875</td><td>17.02</td><td>0</td><td>1</td><td>4</td><td>4</td><td>8.928018</td></tr>

<tr><th scope=row>Datsun 710</th><td>22.8</td><td>4</td><td>108</td><td> 93</td><td>3.85</td><td>2.320</td><td>18.61</td><td>1</td><td>1</td><td>4</td><td>1</td><td>9.693277</td></tr>

<tr><th scope=row>Hornet 4 Drive</th><td>21.4</td><td>6</td><td>258</td><td>110</td><td>3.08</td><td>3.215</td><td>19.44</td><td>1</td><td>0</td><td>3</td><td>1</td><td>9.098075</td></tr>

<tr><th scope=row>Hornet Sportabout</th><td>18.7</td><td>8</td><td>360</td><td>175</td><td>3.15</td><td>3.440</td><td>17.02</td><td>0</td><td>0</td><td>3</td><td>2</td><td>7.950187</td></tr>

<tr><th scope=row>Valiant</th><td>18.1</td><td>6</td><td>225</td><td>105</td><td>2.76</td><td>3.460</td><td>20.22</td><td>1</td><td>0</td><td>3</td><td>1</td><td>7.695101</td></tr>

</tbody>

</table>

  
  
  

The same syntax works to overwrite an existing column.

  

#### What happens when we modify an entire column? or a row?

  

To answer this we need to look at how data frames are represented in R's memory.

  

#### How R represents data frames:

  

- Remember that data frames are lists of vectors

- As such, they don't store the values themselves, they store references to them:

  

```d1 <- data.frame(x = c(1, 5, 6), y = c(2, 4, 3))```

  

*Source: [Advanced R](https://adv-r.hadley.nz/) by Hadley Wickham*

  

#### How R represents data frames:

  

If you modify a column, only that column needs to be modified; the others will still point to their original references:

  

```

d2 <- d1

d2[, 2] <- d2[, 2] * 2

```

  


*Source: [Advanced R](https://adv-r.hadley.nz/) by Hadley Wickham*

  

#### How R represents data frames:

  

However, if you modify a row, every column is modified, which means every column must be copied:

  

```

d3 <- d1

d3[1, ] <- d3[1, ] * 3

```

  


*Source: [Advanced R](https://adv-r.hadley.nz/) by Hadley Wickham*

  

#### An exception to copy-on-modify

  

If an object has a single name bound to it, R will modify it in place:

  

```

v <- c(1, 2, 3)

```

  


```

v[[3]] <- 4

```

  
  
  

- Hence, modify in place can be a useful optimization for speeding up code.

- However, there are some complications that make predicting exactly when R applies this optimisation challenging (see [here](https://adv-r.hadley.nz/names-values.html#modify-in-place) for details)

- There is one other time R will do this, we will cover this when we get to environments.

  

*Source: [Advanced R](https://adv-r.hadley.nz/) by Hadley Wickham*

  

### Writing readable R code

  

- WriTing AND reading (code) TaKes cognitive RESOURCES, & We only hAvE so MUCh!

  

- To help free up cognitive capacity, we will follow the [tidyverse style guide](https://style.tidyverse.org/index.html)

  
  

#### Sample code **not** in tidyverse style

  

Can we spot what's wrong?

  

```

library(tidyverse)

us.2015.econ=read_csv( "data/state_property_data.csv")

us.2016.vote=read_csv( "data/2016_presidential_election_state_vote.csv")

stateData=left_join (us.2015.econ,us.2016.vote) %>%

filter(party!="Not Applicable") %>%

mutate(meanCommuteHours=mean_commute_minutes/60)

ggplot(stateData, aes (x=mean_commute_minutes, y=med_prop_val, color=party)) +

geom_point()+

xlab( "Income (USD)" )+

ylab("Median property value (USD)")+

scale_colour_manual (values = c("blue","red"))+

scale_x_continuous (labels = scales::dollar_format())+

scale_y_continuous (labels = scales::dollar_format())

```

  

#### Sample code in tidyverse style

  

```

library(tidyverse, quietly = TRUE)

us_2015_econ <- read_csv("data/state_property_data.csv")

us_2016_vote <- read_csv("data/2016_presidential_election_state_vote.csv")

state_data <- left_join(us_2015_econ, us_2016_vote) %>%

filter(party != "Not Applicable") %>%

mutate(mean_commute_hours = mean_commute_minutes / 60)

ggplot(state_data, aes(x = med_income, y = med_prop_val, color = party)) +

geom_point() +

xlab("Income (USD)") +

ylab("Median property value (USD)") +

scale_colour_manual(values = c("blue", "red")) +

scale_x_continuous(labels = scales::dollar_format()) +

scale_y_continuous(labels = scales::dollar_format())

```

  

### What did we learn today?

  

- How to get help in R

- How the ` <- ` differs from `=` in R

  

- Base R syntax for subsetting and modifying R objects

  

- Some aspects of tidyverse code style

  
  

## Additional resources:

- [RStudio base R cheat sheet](https://github.com/rstudio/cheatsheets/blob/main/base-r.pdf)

- [R Operators cheat sheet](https://cran.r-project.org/doc/contrib/Baggott-refcard-v2.pdf)

  

## Attribution:

- [Advanced R](https://adv-r.hadley.nz/) by Hadley Wickham

- [Why do we use arrow as an assignment operator?](https://colinfay.me/r-assignment/) by Colin Fay