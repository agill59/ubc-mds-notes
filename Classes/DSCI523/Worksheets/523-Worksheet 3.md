---

title: "DSCI 523 Worksheet 3"

output:

html_document: default

pdf_document:

latex_engine: xelatex

---

  

```{r setup, include=FALSE}

knitr::opts_chunk$set(echo = TRUE)

```

  

Welcome to worksheet 3! This worksheet was designed to allow you to practice what was covered in the assigned readings & lectures/recordings. The labs problems will be more challenging than those in the worksheet, and the worksheet questions will help prepare you for them. Time will be provided to work on the worksheets in small groups during lecture, however it is not expected that you will complete the worksheet in this time. You are expected to spend some independent time working on the worksheet after lecture.

  

<mark>To ensure you do not break the autograder remove all code for installing packages (i.e., DO NOT have `install.packages(...)` or `devtools::install_github(...)` in your homework!</mark>

  

**Worksheets are due on Saturdays at 6 pm, and must be submitted as `.Rmd` file to Gradescope.**

  

### Worksheet mechanics

  

- **There is autograding in this lab, so please do not move or rename this file. Also, do not copy and paste cells, if you need to add new cells, create new cells via the "Insert a cell below" button instead.**

  

Run the cell below to load the libraries needed for this worksheet, and to check your answers as you go!

  

```{r}

library(canlang)

library(digest)

library(dplyr)

library(forcats)

library(ggplot2)

library(lubridate)

library(nycflights13)

library(readr)

library(stringr)

library(testthat)

library(tidyr)

```

  

# Dates and times!

  

The next couple questions will be focused on getting you practice working with dates and times using the {[lubridate](https://lubridate.tidyverse.org/)} R package.

  

Don't forget to reference the [Dates and Times Cheatsheet](https://github.com/rstudio/cheatsheets/raw/main/lubridate.pdf) if you need help remembering things from lecture!

  

<img src="https://github.com/allisonhorst/stats-illustrations/blob/main/rstats-artwork/lubridate.png?raw=true" width=400>

  

*Illustrator: Allison Horst*

  

### Exercise 1

rubric={autograde:1}

  

Use the appropriate {lubridate} function to parse each of the dates given below. Bind the names `answer_1`, `answer_2`, `answer_3`, `answer_4`, `answer_5`, respectively, to the parsed dates.

  

```{r}

d1 <- "January 1, 2010"

d2 <- "2015-Mar-07"

d3 <- "06-Jun-2017"

d4 <- c("August 19 (2015)", "July 1 (2015)")

d5 <- "12/30/14" # Dec 30, 2014

  

answer_1 <- mdy(d1)

answer_2 <- ymd(d2)

answer_3 <- dmy(d3)

answer_4 <- mdy(d4)

answer_5 <- mdy(d5)

  

answer_1

answer_2

answer_3

answer_4

answer_5

```

  

```{r}

. = ottr::check("tests/e1.R")

```

  

### Exercise 2

rubric={autograde:1}

  

Let's go the other way now with dates - let's use the {lubridate} function `wday` get the day of week from the date vector given below. Make sure the day returned is the full name of the weekday (e.g., "Monday" and not "Mon"). To ensure this, look a the `label` and `abbr` function arguments.

  

Bind the name `days_of_week` to the new vector of weekdays you create.

  

```{r}

dates <- c("1982-01-25 01:50:12",

"1967-09-19 19:09:59",

"1999-04-07 22:02:44",

"2050-12-15 03:24:38",

"2020-01-31 11:59:59",

"2009-01-31 01:32:05",

"2010-01-31 23:03:17")

  

days_of_week <- wday(dates, label = TRUE, abbr = FALSE)

  

days_of_week

```

  

```{r}

. = ottr::check("tests/e2.R")

```

  

### Exercise 3

rubric={autograde:1}

  

Let's create a date interval, so that in the following question we can ask whether a date (or dates) falls within that interval! Use the `interval` function to create an interval named `mds_2024_2025` for the time period of when MDS is in session for this upcoming year (so an interval between September 1st, 2024 and June 30th, 2025). *Note: you can ignore time zones for this.*

  

```{r}

#September 1st, 2024 and June 30th, 2025

mds_2024_2025 <- interval(mdy("09/01/2024"), mdy("06/30/2025"))

mds_2024_2025

```

  

```{r}

. = ottr::check("tests/e3.R")

```

  

### Exercise 4

rubric={autograde:1}

  

Using the interval from exercise 3, `filter` the data set below to return just the rows from the `canada_2024-25.csv` data set (which can be found in the `data` directory) which have the Canadian statutory holidays that fall during the 2024-25 MDS academic session. Name the data frame containing those rows `mds_stat_holidays_2024_25`.

  

A couple hints:

- the `holiday_date` column should be converted to the class "Date"

- `%within%` is useful for pairing with `filter` to get rows in an interval

  

```{r}

  

data_e4 <- read_csv("data/canada_2024-25.csv", skip = 2)

data_e4$holiday_date <- ymd(data_e4$holiday_date)

  

mds_stat_holidays_2024_25 <- data_e4 %>% filter(holiday_date %within% mds_2024_2025)

mds_stat_holidays_2024_25

```

  

```{r}

. = ottr::check("tests/e4.R")

```

  

# Strings!

  

The next couple questions will be focused on getting you practice working with strings using the {[stringr](https://stringr.tidyverse.org/)} package. To do this, we will work with the `region_lang` data set again from the {can_lang} package.

  

Don't forget to reference the [Work with Strings Cheatsheet](https://github.com/rstudio/cheatsheets/blob/main/strings.pdf) if you need help remembering things from lecture!

  

<img src="https://github.com/allisonhorst/stats-illustrations/blob/main/rstats-artwork/str_squish.png?raw=true" width=500>

  

*Illustrator: Allison Horst*

  

### Exercise 5

rubric={autograde:1}

  

Sometimes we want to `filter` for rows that contain part of string, either because we do not know the exact string to test for in our `filter` function, or because we want to do partial string matching. The {stringr} `str_detect` function can be really useful for this. We can nest it inside our call to `filter` as demonstrated below:

  

```

data_frame %>%

filter(str_detect(column_name, "pattern_to_match"))

```

  

Your task here is to use `filter` + `str_detect` to obtain rows for the region containing Canada's capital city: "Ottawa"(the `region` column of `region_lang` data frame). Bind the name `ottawa_region` to the data frame.

  

```{r}

ottawa_region <- region_lang %>% filter(str_detect(region, ".*Ottawa*."))

  

ottawa_region

```

  

```{r}

. = ottr::check("tests/e5.R")

```

  

### Exercise 6

rubric={autograde:1}

  

Often times we need to change a string in a data frame column. We can use `mutate` and the {stringr} `str_replace` function to do this. We can nest it inside our call to `mutate` as demonstrated below:

  

```

data_frame %>%

mutate(col_name = str_replace(column_name, "pattern_to_match", "replacement"))

```

  

Your task here is to use `mutate` + `str_replace` to replace all the `-`'s with `&`'s in the `region` column of `region_lang` data frame. Bind the name `replaced_dash` to the data frame.

  

```{r}

replaced_dash <- region_lang %>% mutate(region = str_replace(region, "-", "&"))

  

replaced_dash

```

  

```{r}

. = ottr::check("tests/e6.R")

```

  

# Factors!

  

The next couple questions will be focused on getting you practice working with factors using some base R functions and the {[forcats](https://forcats.tidyverse.org/)} package. Don't forget to reference the [Factors with forcats Cheatsheet](https://github.com/rstudio/cheatsheets/raw/main/factors.pdf) if you need help remembering things from lecture!

  

<img src="https://d33wubrfki0l68.cloudfront.net/d933c65c70add32da585abadaf14fbf3002c6ffc/d42f0/blog/2019/05/forcats-0-4-0/forcats-0-4-0-wd.jpg">

  

We will be working with the `airports` data set which comes from the {[nycflights13](https://github.com/hadley/nycflights13)} R data package. The `airports` data set contains information about airports in the United States, Puerto Rico, and the American Virgin Islands.

  

### Exercise 7

rubric={autograde:1}

  

Convert the `tzone` column in the `airports` data set into a factor. Bind the name `airports_factor` to this data frame. Use the `levels` function to get the levels of the `tzone` column. Store these in a vector named `tzone_levels`.

  

```{r}

  

airports_factor <- airports

airports_factor$tzone <- as.factor(airports_factor$tzone)

  

tzone_levels <- levels(airports_factor$tzone)

  

head(airports_factor)

tzone_levels

```

  

```{r}

. = ottr::check("tests/e7.R")

```

  

Congratulations! You are done the worksheet!!! Pat yourself on the back, and submit your worksheet to Gradescope!