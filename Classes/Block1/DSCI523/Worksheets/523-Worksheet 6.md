---

title: "DSCI 523 Worksheet 6"

output:

  html_document: default

  pdf_document:

    latex_engine: xelatex

  word_document: default

---

```{r setup, include=FALSE}

knitr::opts_chunk$set(echo = TRUE)

```

Welcome to worksheet 6! This worksheet was designed to allow you to practice what was covered in the assigned readings & lectures/recordings. The labs problems will be more challenging than those in the worksheet, and the worksheet questions will help prepare you for them. Time will be provided to work on the worksheets in small groups during lecture, however it is not expected that you will complete the worksheet in this time. You are expected to spend some independent time working on the worksheet after lecture.

<mark>To ensure you do not break the autograder remove all code for installing packages (i.e., DO NOT have `install.packages(...)` or `devtools::install_github(...)` in your homework!</mark>

**Worksheets are due on Saturdays at 6 pm, and must be submitted as `.Rmd` file in this worksheet/lab to Gradescope.**

### Worksheet mechanics

-   **There is autograding in this lab, so please do not move or rename this file. Also, do not copy and paste cells, if you need to add new cells, create new cells via the "Insert a cell below" button instead.**

Run the cell below to load the libraries needed for this worksheet, and to check your answers as you go!

```{r}

library(testthat)

library(digest)

```

### Exercise 1

rubric={autograde:1}

Write a function called `mpg_to_kml` that takes a vector of numbers in miles per gallon units and converts it to a vector in kilometeres per litre.

*Hint: 1 mile per gallon is 0.425144 kilometres per litre.*

```{r}

mpg_to_kml <- function(vec) {

  vec * 0.425144

}

mpg_to_kml(1)

```

```{r}

. <- ottr::check("tests/e1.R")

```

### Exercise 2 (not graded)

*Solution posted at end of worksheet.*

Write **at least** two {testthat} `expect_equal` tests inside of an `test_that` statement to ensure that your function works as expected (i.e., do you get back expected values given your input values - is the math correct?).

*Hint: test edge cases that you can compute by hand, and be sure of your answer!*

<img src="img/unit-tests.png" width=350>

*Source: Austin Powers + http://www.quickmeme.com/meme/3ua14n*

```{r}

# YOUR CODE HERE

```

### Exercise 3

rubric={autograde:1}

Add exceptions to your `mpg_to_kml` function so that you can handle inappropriate user input, such as character vectors, lists or data frames.

<img src="img/fail-you-will-fail-fast-you-should.jpg" width=300>

*Source: Star Wars + https://memegenerator.net/*

```{r}

mpg_to_kml <- function(vec) {

  if (is.vector(vec)) {

    vec * 0.425144

  } else {

    stop()

  }

}

# YOUR CODE HERE

```

```{r}

. <- ottr::check("tests/e3.R")

```

### Exercise 4 (not graded)

*Solution posted at end of worksheet.*

Now write **at least** two {testthat} `expect_error` tests inside of an `test_that` statement to ensure that your function works as expected (i.e., does your function throw errors when incorrect inputs are given?).

```{r}

# YOUR CODE HERE

```

### Exercise 5 (not graded)

*Solution posted at end of worksheet.*

Now write {roxygen2} style documentation for your function!

```{r}

# YOUR CODE HERE

```

### Exercise 6

rubric={autograde:1}

Go on an R package scavengeR hunt! Explore one of the [`stringr` R package on GitHub](https://github.com/tidyverse/stringr) to do the following question:

Create a list that has the following four elements in the order listed below:

1. The relative path (from the project root) to the file containing the `str_replace` function (as a character vector).

2. The line number of the start of the `str_replace` function definition (not including the Roxygen2 comments) (as a numeric vector).

3. The relative path (from the project root) to the file containing the tests for the `str_replace` function (as a character vector).

4. The line number of the start of the first test of the `str_replace` function (as a numeric vector).

Bind the name `str_replace_metadata` to this list.

<img src="https://media.giphy.com/media/PjxWpnShj8mAS2qYAK/giphy-downsized.gif">

*Source: The Big Bang Theory + https://giphy.com/*

```{r}

str_replace_metadata <- list("R/replace.R", c(71), "tests/testthat/test-replace.R", c(1))

str_replace_metadata

```

```{r}

. <- ottr::check("tests/e6.R")

```

Congratulations! You are done the worksheet!!! Pat yourself on the back, and submit your worksheet to Gradescope!

> ### Exercise 2 solution

>

>```

>test_that('1 mile per gallon is should be 0.425144 kilometres per litre', {

>    expect_equal(mpg_to_kml(1), 0.425144)

>    expect_equal(mpg_to_kml(2), 0.850288)

>    expect_equal(mpg_to_kml(10), 4.25144)

>})

>```

> ### Exercise 4 solution

>

>```

>test_that('errors should be thrown when non-numeric vectors, lists or data frames are given to the function', {

>    expect_error(mpg_to_kml("A"))

>    expect_error(mpg_to_kml(list(1, 2)))

>    expect_error(mpg_to_kml(data.frame(x = c(1))))

>})

>```

>### Exercise 5 solution

>

> Roxygen2 comments only:

>

>```

>#' Miles per gallon to kilometres per litre

>#'

>#' Convert a fuel efficiency from miles per gallon to kilometres per litre.

>#'

>#' @param x numeric

>#'

>#' @return numeric

>#' @export

>#'

>#' @examples

>#' mpg_to_kml(1)

>```