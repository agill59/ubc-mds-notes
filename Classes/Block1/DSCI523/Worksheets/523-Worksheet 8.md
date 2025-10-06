---

title: "DSCI 523 Worksheet 8"

output: 

    pdf_document:

        latex_engine: xelatex

---

```{r setup, include=FALSE}

knitr::opts_chunk$set(echo = TRUE)

```

Welcome to worksheet 8! This worksheet was designed to allow you to practice what was covered in the assigned readings & lectures/recordings. The labs problems will be more challenging than those in the worksheet, and the worksheet questions will help prepare you for them. Time will be provided to work on the worksheets in small groups during lecture, however it is not expected that you will complete the worksheet in this time. You are expected to spend some independent time working on the worksheet after lecture.

<mark>To ensure you do not break the autograder remove all code for installing packages (i.e., DO NOT have `install.packages(...)` or `devtools::install_github(...)` in your homework!</mark>

**Worksheets are due on Saturdays at 6 pm, and must be submitted as `.Rmd` file and the rendered `html` in this worksheet/lab to Gradescope.**

### Worksheet mechanics

-   **There is autograding in this lab, so please do not move or rename this file. Also, do not copy and paste cells, if you need to add new cells, create new cells via the "Insert a cell below" button instead.**

Run the cell below to load the libraries needed for this worksheet, and to check your answers as you go!

```{r}

library(palmerpenguins)

library(readr)

library(dplyr)

library(stringr)

library(tidyr)

library(purrr)

library(rlang)

library(testthat)

library(digest)

```

### Exercise 1

rubric={autograde:1}

The `get_rectangle` function shown below does not work for unquoted column names. Use the `{{` operator to embrace the column names/user inputs given by the user so that the function works as described:

```

#' Get smaller rectangle from a bigger data frame

#' 

#' `get_rectangle` subsets a data frame using filter and select syntax.

#'

#' @param data A data frame to subset

#' @param row_filter filter logical syntax used to filter rows, column name should be unquoted.

#' @param column_range select syntax (single column or range via `col1:coln`) to 

#' select columns, default is everything

#'

#' @return A smaller/subsetted data frame

#' @export

#'

#' @examples

#' library(palmerpenguins)

#' get_rectangle(penguins, body_mass_g > 3000, species:island)

get_rectangle <- function(data, row_filter, column_range = everything()) {

    data |> 

        filter(row_filter) |> 

        select(column_range)

}

```

*Hint: embracing works for more than just a column name - you can even embrace a variable that will hold a filter logical statement (e.g., `body_mass_g > 3000`).*

```{r}

#' Get smaller rectangle from a bigger data frame

#'

#' `get_rectangle` subsets a data frame using filter and select syntax.

#'

#' @param data A data frame to subset

#' @param row_filter filter logical syntax used to filter rows, column name should be unquoted.

#' @param column_range select syntax (single column or range via `col1:coln`) to

#' select columns, default is everything

#'

#' @return A smaller/subsetted data frame

#' @export

#'

#' @examples

#' library(palmerpenguins)

#' get_rectangle(penguins, body_mass_g > 3000, species:island)

get_rectangle <- function(data, row_filter, column_range = everything()) {

    data |>

        filter({{ row_filter }}) |>

        select({{ column_range }})

}

get_rectangle(penguins, body_mass_g > 3000, species:island)

```

```{r}

. <- ottr::check("tests/e1.R")

```

### Exercise 2

rubric={autograde:1}

Why do we need to embrace the columns with `{{` (or do something else that is equivalent) when writing functions %>% that wrap {tidyverse} functions when we want to pass unquoted column names in as function arguments? Choose all answers listed below that you think are correct:

A. R performs lazy, not eager, evaluation of function arguments

B. A data frame column name is not something that is known/accessible in the global environment

C. Column names cannot be referred to as strings in R

D. `{{` quotes the column names, and then unquotes and evaluates them inside the data mask - where they can be successfully evaluated

Create a character vector of the answers above you think are correct, for example, `c("E", "F")`. Bind the name `answer_2` to this vector.

```{r}

answer_2 <- c("B", "D")

answer_2

```

```{r}

. <- ottr::check("tests/e2.R")

```

### Exercise 3

rubric={autograde:1}

The `nest_and_count function` shown below does not work. Your task is to identify why and fix it so that the function works as described:

```

#' Nest data and count observations in each group

#'

#' @param data A data frame

#' @param nest_by A column to group by. Column name should be unquoted.

#' @param col_name A new column name for the count column. Column name should be unquoted.

#'

#' @return A data frame where the data, other than the group specified in 

#' `nest_by`, is nested. The number of rows for each data frame in the list

#' column is given in a column whose name is specified by the user.

#' @export

#'

#' @examples

#' library(palmerpenguins)

#' nest_and_count(penguins, species, n)

nest_and_count <- function(data, nest_by, col_name) {

    data |> 

        group_by({{ nest_by }}) |> 

        nest() |> 

        mutate({{ col_name }} = map_dbl(data, nrow))

}

```

```{r}

#' Nest data and count observations in each group

#'

#' @param data A data frame

#' @param nest_by A column to group by. Column name should be unquoted.

#' @param col_name A new column name for the count column. Column name should be unquoted.

#'

#' @return A data frame where the data, other than the group specified in

#' `nest_by`, is nested. The number of rows for each data frame in the list

#' column is given in a column whose name is specified by the user.

#' @export

#'

#' @examples

#' library(palmerpenguins)

#' nest_and_count(penguins, species, n)

nest_and_count <- function(data, nest_by, col_name) {

    data |>

        group_by({{ nest_by }}) |>

        nest() |>

        mutate({{ col_name }} := map_dbl(data, nrow))

}

nest_and_count(penguins, species, n) |> print()

```

```{r}

. <- ottr::check("tests/e3.R")

```

### Exercise 4

rubric={autograde:1}

Take the function you fixed in exercise 1 and **rename it** `get_rectangle2`. Now make it work better with select syntax by added pass the dots, `...`, in place of `column_range`. When you do this, play with inputs to this function, what can you do with `...` as opposed to using a named argument? *Hint: try selecting columns that are not beside each other.* What can't you do? *Hint: think about function argument defaults.*

```{r}

#' Get smaller rectangle from a bigger data frame

#'

#' `get_rectangle` subsets a data frame using filter and select syntax.

#'

#' @param data A data frame to subset

#' @param row_filter filter logical syntax used to filter rows, column name should be unquoted.

#' @param column_range select syntax (single column or range via `col1:coln`) to

#' select columns, default is everything

#'

#' @return A smaller/subsetted data frame

#' @export

#'

#' @examples

#' library(palmerpenguins)

#' get_rectangle(penguins, body_mass_g > 3000, species:island)

get_rectangle2 <- function(data, row_filter, ...) {

    data |>

        filter({{ row_filter }}) |>

        select(...)

}

get_rectangle2(penguins, body_mass_g > 3000, species:island)

```

```{r}

. <- ottr::check("tests/e4.R")

```

### Exercise 5

rubric={autograde:1}

Write a function called `select_and_arrange` whose documentation we provide below. This function should use embracing with `{{` and pass the dots `...` syntax to accomplish it's job.

```

#' Select and arrange

#'

#' @param data A data frame

#' @param sort_by A column that you would like to order the data frame by in 

#' ascending order. Column name should be unquoted.

#' @param ... columns you would like to select using select syntax

#'

#' @return A data frame ordered by ascending order of the `sort_by` column and 

#' containing the columns as indicated by the user using select syntax in `...`

#' @export

#'

#' @examples

#' library(palmerpenguins)

#' select_and_arrange(penguins, body_mass_g, species:island, sex)

```

```{r}

#' Select and arrange

#'

#' @param data A data frame

#' @param sort_by A column that you would like to order the data frame by in

#' ascending order. Column name should be unquoted.

#' @param ... columns you would like to select using select syntax

#'

#' @return A data frame ordered by ascending order of the `sort_by` column and

#' containing the columns as indicated by the user using select syntax in `...`

#' @export

#'

#' @examples

#' library(palmerpenguins)

#' select_and_arrange(penguins, body_mass_g, species:island, sex)

select_and_arrange <- function(data, sort_by, ...) {

  data %>%

    arrange({{sort_by}}) %>%

    select(...)

}

select_and_arrange(penguins, body_mass_g, species:island, sex)

```

```{r}

. <- ottr::check("tests/e5.R")

```

### Exercise 6

rubric={autograde:1}

This is your last worksheet question for this course! Congrats! This one is a choose your own adventure. Write any function that takes a data frame, and an quoted column name. Use either `{{` or `..` (or both!) to deal with the unquoted column name(s). All you need to do is make sure it works. Be creative! 

We provide a function name so we can test that you used `{{` or `..`. Feel free to add whatever else you want to it, just don't change the `have_fun` function name.

```{r}

have_fun <- function(data, ...) { 

  data %>%

    select(...)

}

```

```{r}

. <- ottr::check("tests/e6.R")

```

Congratulations! You are done the **LAST** worksheet!!! Pat yourself on the back, and submit your worksheet to Gradescope!