---

title: "DSCI 523 Worksheet 2"

output:

pdf_document:

latex_engine: xelatex

html_document: default

---

  

```{r setup, include=FALSE}

knitr::opts_chunk$set(echo = TRUE)

```

  

Welcome to worksheet 1! This worksheet was designed to allow you to practice what was covered in the assigned readings & lectures/recordings. The labs problems will be more challenging than those in the worksheet, and the worksheet questions will help prepare you for them. Time will be provided to work on the worksheets in small groups during lecture, however it is not expected that you will complete the worksheet in this time. You are expected to spend some independent time working on the worksheet after lecture.

  

<mark>To ensure you do not break the autograder remove all code for installing packages (i.e., DO NOT have `install.packages(...)` or `devtools::install_github(...)` in your homework!</mark>

  

**Worksheets are due on Saturdays at 6 pm, and must be submitted as `.Rmd` file and the rendered `pdf` in this worksheet/lab to Gradescope.**

  

### Worksheet mechanics

  

- **There is autograding in this lab, so please do not move or rename this file. Also, do not copy and paste cells, if you need to add new cells, create new cells via the "Insert a cell below" button instead.**

  

Run the cell below to load the libraries needed for this worksheet, and to check your answers as you go!

  

```{r}

library(digest)

library(testthat)

```

  

# 1.0 The operators and key datatypes for Data Science in R

  

### Exercise 1

  

rubric={autograde:1}

  

In R, all elements of vectors and lists must have the same type. True or false?

  

Create an object with a valid logical value corresponding to whether you believe the answer to the question above is `TRUE` or `FALSE`. Bind the name `answer1` to this object.

  

```{r}

answer1 <- FALSE

answer1

```

  

```{r}

. = ottr::check("tests/e1.R")

```

  

### Exercise 2

  

rubric={autograde:1}

  

Create a vector containing the names of the types of the four most common types of vectors. Bind the name `answer2` to this object.

  

```{r}

answer2 <- c("logical", "double", "character", "integer")

  

answer2

```

  

```{r}

. = ottr::check("tests/e2.R")

```

  

### Exercise 3

  

rubric={autograde:1}

  

Create vector containing the following values: 2.2, 15.0, 3.5, 4.0, 30.0 (in the order we list them) and bind the name `values` to it. ***Hint**: The columns function, `c()`, is helpful for constructing vectors.*

  

```{r}

values <- c(2.2, 15.0, 3.5, 4.0, 30.0)

  

values

```

  

```{r}

. = ottr::check("tests/e4.R")

```

  

### Exercise 4: writing files from R

  

rubric={autograde:1}

  

Use the `[` syntax to get the first element from the vector `values` and bind the name `values_first` to it.

  

```{r}

values_first <- values[1]

  

values_first

```

  

```{r}

. = ottr::check("tests/e5.R")

```

  

### Exercise 5

  

rubric={autograde:1}

  

Create a list (not a vector) with the **original** values contained in the `values` vector you just explored ( 2.2, 15.0, 3.5, 4.0, and 30.0). Bind the name `value_list` to the list you create.

  

*Hint: use the `list` function to create a list in R.*

  

```{r}

value_list <- list(2.2, 15.0, 3.5, 4.0, 30.0)

  

value_list

```

  

```{r}

. = ottr::check("tests/e8.R")

```

  

### Exercise 6

  

rubric={autograde:1}

  

Get the first element from the list and bind the name `value_list_first` to it. Ensure that what you return is a number (**not a list itself**). Remember: When indexing, using `[` gives you the same type of object as the object being indexed (e.g., returns a list when applied to a list), whereas `[[` removes one level of structure (e.g., if the object is a list of numbers, you get the number back itself).

  

```{r}

value_list_first <- value_list[[1]]

  

value_list_first

```

  

```{r}

. = ottr::check("tests/e9.R")

```

  

### Exercise 7

rubric={autograde:1}

  

Consider the built-in/base R `CO2` data frame shown below, which contains data from an experiment on carbon dioxide uptake in the grass plant *Echinochloa crus-galli*. Use the `[` syntax to extract rows 1 - 5, and only the columns `conc` and `uptake`. Bind the name `first_five_measurements` to this smaller data frame.

  

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/1/19/Echinochloa_crus-galli01.jpg/440px-Echinochloa_crus-galli01.jpg" width=200>

  

*Source: https://en.wikipedia.org/wiki/Echinochloa_crus-galli*

  

```{r}

# view the first 10 rows

CO2[1:10, ]

```

  

```{r}

first_five_measurements <- CO2[1:5, c("conc", "uptake")]

  

first_five_measurements

```

  

```{r}

. = ottr::check("tests/e11.R")

```

  

Congratulations! You are done the worksheet!!! Pat yourself on the back, and submit your worksheet to Gradescope!