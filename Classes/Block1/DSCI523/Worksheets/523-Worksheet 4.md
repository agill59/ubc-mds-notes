---

title: "DSCI 523 Worksheet 4"

output:

pdf_document:

latex_engine: xelatex

html_document: default

---

  

```{r setup, include=FALSE}

knitr::opts_chunk$set(echo = TRUE)

```

  

Welcome to worksheet 4! This worksheet was designed to allow you to practice what was covered in the assigned readings & lectures/recordings. The labs problems will be more challenging than those in the worksheet, and the worksheet questions will help prepare you for them. Time will be provided to work on the worksheets in small groups during lecture, however it is not expected that you will complete the worksheet in this time. You are expected to spend some independent time working on the worksheet after lecture.

  

<mark>To ensure you do not break the autograder remove all code for installing packages (i.e., DO NOT have `install.packages(...)` or `devtools::install_github(...)` in your homework!</mark>

  

**Worksheets are due on Saturdays at 6 pm, and must be submitted as `.Rmd` file to Gradescope.**

  

### Worksheet mechanics

  

- **There is autograding in this lab, so please do not move or rename this file. Also, do not copy and paste cells, if you need to add new cells, create new cells via the "Insert a cell below" button instead.**

  

Run the cell below to load the libraries needed for this worksheet, and to check your answers as you go!

  

```{r}

library(nycflights13)

library(repurrrsive)

library(dplyr)

library(testthat)

library(digest)

```

  

### Exercise 1

rubric={autograde:1}

  

We have given you two data sets below of highly important data, and we want to join these two data sets so that we can make a data visualization at some point in the future. Use `left_join` to do this. Bind the name `flavours_and_vessls` to the data frame.

  

```{r}

flavour <- tibble(name = c("Joel", "Joel",

"Mike", "Mike", "Mike",

"Tiff", "Tiff",

"Tom",

"Vincenzo", "Vincenzo", "Vincenzo",

"Arman", "Arman",

"Florencia", "Florencia"),

flavour =c("Dark chocolate/pistachio/vanilla combo", "Chocolate",

"Green tea", "Coconut", "Mint Chocolate Chip",

"Salted caramel", "Mint Chocolate Chip",

"Oreo",

"Strawberry", "Salted Caramel", "Lemon",

"Vanilla", "Pistachio",

"Strawberry", "Lemon"))

vessel <- tibble(name = c("Joel", "Mike", "Tiff", "Tom", "Vincenzo", "Arman", "Florencia"),

vessel = c("Vegan cone", "Cone", "Cone", "Cone", "Cone", "Cone", "Bowl"))

  

flavour

vessel

```

  

```{r}

flavours_and_vessls <- left_join(flavour, vessel)

  

flavours_and_vessls

```

  

```{r}

. = ottr::check("tests/e1.R")

```

  

### Exercise 2

rubric={autograde:1}

  

Use {dplyr}'s `left_join` to add the the location of the destination (i.e. the `lat` and `lon`) from the {nycflights13} `airports` data frame to the `flights` data frame. The column in the `flights` data frame you should be joining on is `dest`. Bind the name `flight_dests` to your final data frame.

  

*Hint: if you the column names you want to join on don't match, try using the `by` argument.*

  

```{r}

flight_dests <- left_join(flights, airports %>%

select(faa, lat, lon), by = c("dest" = "faa"))

  

head(flight_dests)

```

  

```{r}

. = ottr::check("tests/e2.R")

```

  

### Exercise 3

rubric={autograde:1}

  

Going back now to our ice cream flavour and vessel data sets, there are some people in that data set who are (sadly!) no longer MDS core teaching team members. Use `inner_join` and the third data set provided below, `core_teaching_team_members`, to get back just the data from `flavours_and_vessls` (the data frame you made exercise 1) for the people listed in there, but only if we have data on their preferred ice cream flavours and vessels. Bind the name `core_flavours_and_vessls` to this data frame.

  
  

```{r}

core_teaching_team_members <- tibble(name = c("Alexi", "Katie", "Joel", "Tiff",

"Varada", "Gittu", "Quan", "Daniel"))

core_teaching_team_members

```

  

```{r}

core_flavours_and_vessls <- inner_join(flavours_and_vessls,

core_teaching_team_members)

  

core_flavours_and_vessls

```

  

```{r}

. = ottr::check("tests/e3.R")

```

  

### Exercise 4

rubric={autograde:1}

  

Now use `full_join` on the same two data sets you joined in exercise 3. Bind the name `full_core_flavours_and_vessls` to the data frame. Note the difference between the date frame you get in this exercise and the one you got in exercise 3.

  

```{r}

full_core_flavours_and_vessls <- full_join(flavours_and_vessls,

core_teaching_team_members)

  

full_core_flavours_and_vessls

```

  

```{r}

. = ottr::check("tests/e4.R")

```

  

### Exercise 5

{not graded - solution at the end of the notebook}

  

Use a `for` loop to print out the song lyrics, one per line in the way they would be sung (one song line corresponds to an element in the vector), in the provided vector.

  

```{r}

lyrics <- c("If there was a problem yo I'll solve it", "Check out the hook while my DJ revolves it", "Ice ice baby", "vanilla, Ice ice baby", "vanilla, Ice ice baby", "vanilla Ice ice baby", "vanilla")

```

  

```{r}

  
  

for (i in seq_along(lyrics)){

print(lyrics[i])

}

  

```

  

### Exercise 6

rubric={autograde:1}

  

Consider the prototype code for an exercise app you are writing in R. Rearrange the jumbled code below so that 20 repetitions are prescibed if the exercise is lunges, 25 repetitions are prescribed if the exercise is squats, and 15 repetitions are prescibed the exercise is burpees, and 10 repetitions for any other type of exercise. `exercise` shoud point to the value `'burpees'`.

  

```{r}

  

# REARRANGED CODE HERE

exercise <- "burpees"

if (exercise == "lunges") {

reps <- 20

} else if (exercise == "squats") {

reps <- 25

} else if (exercise == "burpees") {

reps <- 15

} else {

reps <- 10

}

print(reps)

  

```

  

Now, perform the number of burpee reps that R printed out for you. Joking of course! ;p

  

```{r}

. = ottr::check("tests/e8.R")

```

  

Congratulations! You are done the worksheet!!! Pat yourself on the back, and submit your worksheet to Gradescope!

  

> ### Exercise 5 solution

>

>```

>for (line in lyrics) {

> print(line)

>}

>```