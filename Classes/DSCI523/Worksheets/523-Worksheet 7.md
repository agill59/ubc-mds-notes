---

title: "DSCI 523 Worksheet 7"

output: 

    pdf_document:

        latex_engine: xelatex

---

```{r setup, include=FALSE}

knitr::opts_chunk$set(echo = TRUE)

```

Welcome to worksheet 7! This worksheet was designed to allow you to practice what was covered in the assigned readings & lectures/recordings. The labs problems will be more challenging than those in the worksheet, and the worksheet questions will help prepare you for them. Time will be provided to work on the worksheets in small groups during lecture, however it is not expected that you will complete the worksheet in this time. You are expected to spend some independent time working on the worksheet after lecture.

<mark>To ensure you do not break the autograder remove all code for installing packages (i.e., DO NOT have `install.packages(...)` or `devtools::install_github(...)` in your homework!</mark>

**Worksheets are due on Saturdays at 6 pm, and must be submitted as `.Rmd` file and the rendered `html` in this worksheet/lab to Gradescope.**

### Worksheet mechanics

-   **There is autograding in this lab, so please do not move or rename this file. Also, do not copy and paste cells, if you need to add new cells, create new cells via the "Insert a cell below" button instead.**

Run the cell below to load the libraries needed for this worksheet, and to check your answers as you go!

```{r}

library(canlang)

library(palmerpenguins)

library(readr)

library(dplyr)

library(purrr)

library(tidyr)

library(ggplot2)

library(testthat)

library(digest)

```

### Exercise 1

rubric={autograde:1}

Write an anonymous function in R that takes a single vector, `x`, and squares each element of a `x`. Pass it a vector containing the numbers from 1 to 10, inclusive. Save the output of the anonymous function to a variable name `squares`.

```{r}

squares <- (function(x) x^2)(seq(1, 10))

squares

```

```{r}

. <- ottr::check("tests/e1.R")

```

### Exercise 2

rubric={autograde:1}

The `paste0` function in R is analagous to the `+` operator in Python for concatenating strings. Use `map_chr`, an anonymous function, and the `paste0` function to concatenate the list `month` with the value 2020 so that there is a comma and a single whitespace between the month and the year (e.g., "September, 2020"). Bind the name `months_2020` to the character vector you create.

```{r}

month <- list(

    "January", "February", "March", "April", "May", "June",

    "July", "August", "September", "October", "November", "December"

)

months_2020 <- (function(x) paste0(x, ", 2020"))(month)

months_2020

```

```{r}

. <- ottr::check("tests/e2.R")

```

### Exercise 3

rubric={autograde:1}

Use the `tibble` function to create a data frame from the `species` and `data` objects provided below. Your data frame should have 3 rows and two columns (`species` and `data`). The first column should be a character vector, the second column should be a list column, where each element is a tibble itself. Bind the name `nested_penguins` to the final tibble you create.

```{r}

species <- c("Gentoo", "Chinstrap", "Adelie")

data <- list(

    tibble(island = c("Biscoe", "Biscoe"), sex = c("male", "female")),

    tibble(island = c("Dream", "Dream"), sex = c("male", "male")),

    tibble(island = c("Torgersen", "Dream"), sex = c("female", "female"))

)

nested_penguins <- tibble(species = species, data = data)

print(nested_penguins)

```

```{r}

. <- ottr::check("tests/e3.R")

```

### Exercise 4

rubric={autograde:1}

Use {tidyr} `unnest` on the `nested_penguins` data set you created above to see what the unnested data frame would look like. Bind the name `unnested_penguins` to this new tibble.

*Hint: provide the argument `cols = c(data)` inside `unnest` to specify which column you would like to unnest.*

```{r}

unnested_penguins <- unnest(nested_penguins, cols = c(data))

unnested_penguins

```

```{r}

. <- ottr::check("tests/e4.R")

```

### Exercise 5

rubric={autograde:1}

Let's explore the {canlang} `region_lang` data set again! 

Use {dpylr}'s `group_by` and {tidyr}'s `nest` to create a nested data frame where the data for each language category (the column `category` in `region_lang` data set) is nested in a list column named `data`. Your final data frame should have two columns (named `category` and `data`); the column `data` should contain the nested data corresponding to each category from the `region_lang` data set. The three rows of the data frame should be 3 distinct language categories. Bind the name `nested_reg_lang` to it.

```{r}

nested_reg_lang <- region_lang %>%

  group_by(category) %>%

  nest(colnames(region_lang)[-2])

nested_reg_lang

```

```{r}

. <- ottr::check("tests/e5.R")

```

### Exercise 6

rubric={autograde:1}

Now let's do something insteresting with the `nested_reg_lang` data frame you created above!

Let's calculate the correlation between the number of people who speak a language at home with the number who speak that same language at work, for each language category. Add this summary statistic in the `nested_reg_lang` data set, and give the name `cor_home_work` to the column containing the correlations. Use {dplyr}'s `mutate`, {purrr}'s `map_dbl` an anonymouse function, and the base `cor` function to accomplish this. The new data frame should have 3 rows and 3 columns (named `category`, `data`, and `cor_home_work`). Bind the name `nested_reg_lang_hw_cor` to this new data frame.

```{r}

nested_reg_lang_hw_cor <- nested_reg_lang %>%

  mutate(cor_home_work = map_dbl(data, ~cor(.$most_at_home, .$most_at_work)))

print(nested_reg_lang_hw_cor)

```

```{r}

. <- ottr::check("tests/e6.R")

```

Congratulations! You are done the worksheet!!! Pat yourself on the back, and submit your worksheet to Gradescope!