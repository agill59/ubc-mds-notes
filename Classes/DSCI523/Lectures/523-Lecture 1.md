# Lecture 1 -  Introduction to R via the tidyverse

(or Reading data, single data frame manipulations & tidying data in R)


[[523-Worksheet 1]]
[[523-Lab 1]]
#### Learning objectives

By the end of this lecture and worksheet 1, students should be able to:

* Choose and use the appropriate `readr::read_*` function and function arguments to load a given rectangular, plain text data set into R
* Use the assignment symbol, `<-`, to assign values to objects in R
* Write a dataframe to a .csv file using `readr::write_csv`
* Use `readr::read_csv` to bring data from standard comma separated value (`.csv`) files into R
* Recall and use the following `dplyr` functions and operators for their intended data wrangling tasks:
  - `select`
  - `filter`
  - `mutate`
  - `arrange`
  - `desc`
  - `slice`
  - `pull`
  - `%in%`
* Use the pipe operator, `|>`, to combine two or more functions
* Define the term "tidy data"
* Discuss the advantages and disadvantages of the tidy data format
* Use `tidyr::pivot_wider` & `tidyr::pivot_longer` in R to make untidy data tidy

### First, a bit of history about me

- Ph.D. in Neuroscience (2012)
- Started using R in ~ 2010 because I needed to do “complex” statistics
- Other programming languages I have used:
  - Turing
  - Java
  - Matlab
  - Python (only other language that I still currently use and remember)

Oh, and I like gifs, so you might see some in my lecture notes…

### Now, a bit of history about R

- An implementation of the S programming language (created at Bell labs in 1976)
- written in C, Fortran, and R itself
- R was created by Ross Ihaka and Robert Gentleman (Statisticians from NZ)
- R is named partly after the authors and partly as a play on the name of S
- First stable beta version in 2000

<img src="http://revolution-computing.typepad.com/.a/6a010534b1db25970b01b8d2594d25970c-pi" width="600">

*Source: https://blog.revolutionanalytics.com/2016/03/16-years-of-r-history.html*

R currently has more than 15,000 additional packages (as of September 2018)!

```{admonition}
A) YES, I used R before.

B) NO, I haven't used R before.

C) I haven't even heard of it.

```

```{toggle}

***Answer:  NA ***

```

## Loading/importing data

Taking our first step in data analysis:

![](https://d33wubrfki0l68.cloudfront.net/795c039ba2520455d833b4034befc8cf360a70ba/558a5/diagrams/data-science-explore.png)

*Source: Grolemund & Wickham, [R for Data Science](https://r4ds.had.co.nz/)*

#### The four most common ways to do this in Data Science

1. **read in a text file with data in a spreadsheet format**
2. read from a database (e.g., SQLite, PostgreSQL)
3. scrape data from the web
4. use a web API to read data from a website

#### Reading spreadsheet-like data into R

- We recommend using the `readr` package (part of the tidyverse) functions for plain text files
- We recommed using the `readxl` package (part of the tidyverse but needs to be explicitly loaded in addition to tidyverse) for Microsoft Excel files.

#### Workflow for reading in spreadsheet-like data

**Step 1: LOOK AT THE FILE!** 👀

**Step 2:** Match what you see with an appropriate {readr} or {readxl} package function

**Step 3:** Choose the correct arguments for that file and that function

**Step 1: LOOK AT THE RESULT TO MAKE SURE IT WORKED!** 👀

#### The simplest case: a comma separated value file

- The simplest plain text data file you will encounter is a a comma separated value (`.csv`) file.
- `read_csv`, from the {readr} package, is the function of choice here.
- In its most basic use-case, `read_csv` expects that the data file:

  - has column names (or headers),
  - uses a comma (,) to separate the columns, and
  - does not have row names.

Reading in `data/can_lang.csv` data using `read_csv`:

First, load the {readr} or {tidyverse} library:

> Note: {readr} is part of the {tidyverse} metapackage, so when you load {tidyverse} you get the {readr} package functions, as well as a bunch of other goodies we'll learn about shortly!

```R
library(tidyverse)
```

    ── [1mAttaching packages[22m ───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── tidyverse 1.3.2 ──
    [32m✔[39m [34mggplot2[39m 3.3.6     [32m✔[39m [34mpurrr  [39m 0.3.4
    [32m✔[39m [34mtibble [39m 3.1.8     [32m✔[39m [34mdplyr  [39m 1.0.9
    [32m✔[39m [34mtidyr  [39m 1.2.0     [32m✔[39m [34mstringr[39m 1.4.1
    [32m✔[39m [34mreadr  [39m 2.1.2     [32m✔[39m [34mforcats[39m 0.5.1
    ── [1mConflicts[22m ──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── tidyverse_conflicts() ──
    [31m✖[39m [34mdplyr[39m::[32mfilter()[39m masks [34mstats[39m::filter()
    [31m✖[39m [34mdplyr[39m::[32mlag()[39m    masks [34mstats[39m::lag()

Next, use the `read_csv` function to read the data into R from the `can_lang.csv` file:

```R
can_lang1 <- read_csv("data/can_lang.csv")
head(can_lang1)
```

    [1mRows: [22m[34m214[39m [1mColumns: [22m[34m6[39m
    [36m──[39m [1mColumn specification[22m [36m──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────[39m
    [1mDelimiter:[22m ","
    [31mchr[39m (2): category, language
    [32mdbl[39m (4): mother_tongue, most_at_home, most_at_work, lang_known

    [36mℹ[39m Use`spec()` to retrieve the full column specification for this data.
    [36mℹ[39m Specify the column types or set `show_col_types = FALSE` to quiet this message.

<table class="dataframe">
<caption>A tibble: 6 × 6</caption>
<thead>
	<tr><th scope=col>category</th><th scope=col>language</th><th scope=col>mother_tongue</th><th scope=col>most_at_home</th><th scope=col>most_at_work</th><th scope=col>lang_known</th></tr>
	<tr><th scope=col><chr></th><th scope=col><chr></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Aboriginal languages                   </td><td>Aboriginal languages, n.o.s.  </td><td>  590</td><td>  235</td><td> 30</td><td>  665</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td><span style=white-space:pre-wrap>Afrikaans                     </span></td><td>10260</td><td> 4785</td><td> 85</td><td>23415</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td>Afro-Asiatic languages, n.i.e.</td><td> 1150</td><td><span style=white-space:pre-wrap>  445</span></td><td> 10</td><td> 2775</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td><span style=white-space:pre-wrap>Akan (Twi)                    </span></td><td>13460</td><td> 5985</td><td> 25</td><td>22150</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td><span style=white-space:pre-wrap>Albanian                      </span></td><td>26895</td><td>13135</td><td>345</td><td>31930</td></tr>
	<tr><td>Aboriginal languages                   </td><td>Algonquian languages, n.i.e.  </td><td>   45</td><td>   10</td><td>  0</td><td>  120</td></tr>
</tbody>
</table>

#### Skipping rows when reading in data

- Often times information about how data was collected, or other relevant information, is included at the top of the data file.
- Referred to as "metadata"

Reading in `data/can_lang-meta-data.csv`

```R
can_lang2 <- read_csv("data/can_lang-meta-data.csv", skip = 2)
head(can_lang2)
```

    [1mRows: [22m[34m214[39m [1mColumns: [22m[34m6[39m
    [36m──[39m [1mColumn specification[22m [36m──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────[39m
    [1mDelimiter:[22m ","
    [31mchr[39m (2): category, language
    [32mdbl[39m (4): mother_tongue, most_at_home, most_at_work, lang_known

    [36mℹ[39m Use`spec()` to retrieve the full column specification for this data.
    [36mℹ[39m Specify the column types or set `show_col_types = FALSE` to quiet this message.

<table class="dataframe">
<caption>A tibble: 6 × 6</caption>
<thead>
	<tr><th scope=col>category</th><th scope=col>language</th><th scope=col>mother_tongue</th><th scope=col>most_at_home</th><th scope=col>most_at_work</th><th scope=col>lang_known</th></tr>
	<tr><th scope=col><chr></th><th scope=col><chr></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Aboriginal languages                   </td><td>Aboriginal languages, n.o.s.  </td><td>  590</td><td>  235</td><td> 30</td><td>  665</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td><span style=white-space:pre-wrap>Afrikaans                     </span></td><td>10260</td><td> 4785</td><td> 85</td><td>23415</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td>Afro-Asiatic languages, n.i.e.</td><td> 1150</td><td><span style=white-space:pre-wrap>  445</span></td><td> 10</td><td> 2775</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td><span style=white-space:pre-wrap>Akan (Twi)                    </span></td><td>13460</td><td> 5985</td><td> 25</td><td>22150</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td><span style=white-space:pre-wrap>Albanian                      </span></td><td>26895</td><td>13135</td><td>345</td><td>31930</td></tr>
	<tr><td>Aboriginal languages                   </td><td>Algonquian languages, n.i.e.  </td><td>   45</td><td>   10</td><td>  0</td><td>  120</td></tr>
</tbody>
</table>

If you need to skip rows at the bottom of a file, use the `n_max` argument.

Note - you will need to know how many lines are in the file to succesfully use this. You can use some shell/command line tools to do this.

- `wc -l FILENAME` will tell you how many lines are in the file
- `tail -n 10 FILENAME` will print the last 10 lines of the file

#### `read_delim` as a more flexible method to get data into R

- most flexible `readr` function is `read_delim`
- doesn't assume any delimiter, you have to specify

Reading in `data/can_lang.tsv`

```R
can_lang3 <- read_delim("data/can_lang.tsv",  
                     delim = "\t", 
                     col_names = FALSE)
head(can_lang3)
```

    [1mRows: [22m[34m214[39m [1mColumns: [22m[34m6[39m
    [36m──[39m [1mColumn specification[22m [36m──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────[39m
    [1mDelimiter:[22m "\t"
    [31mchr[39m (2): X1, X2
    [32mdbl[39m (4): X3, X4, X5, X6

    [36mℹ[39m Use`spec()` to retrieve the full column specification for this data.
    [36mℹ[39m Specify the column types or set `show_col_types = FALSE` to quiet this message.

<table class="dataframe">
<caption>A tibble: 6 × 6</caption>
<thead>
	<tr><th scope=col>X1</th><th scope=col>X2</th><th scope=col>X3</th><th scope=col>X4</th><th scope=col>X5</th><th scope=col>X6</th></tr>
	<tr><th scope=col><chr></th><th scope=col><chr></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Aboriginal languages                   </td><td>Aboriginal languages, n.o.s.  </td><td>  590</td><td>  235</td><td> 30</td><td>  665</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td><span style=white-space:pre-wrap>Afrikaans                     </span></td><td>10260</td><td> 4785</td><td> 85</td><td>23415</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td>Afro-Asiatic languages, n.i.e.</td><td> 1150</td><td><span style=white-space:pre-wrap>  445</span></td><td> 10</td><td> 2775</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td><span style=white-space:pre-wrap>Akan (Twi)                    </span></td><td>13460</td><td> 5985</td><td> 25</td><td>22150</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td><span style=white-space:pre-wrap>Albanian                      </span></td><td>26895</td><td>13135</td><td>345</td><td>31930</td></tr>
	<tr><td>Aboriginal languages                   </td><td>Algonquian languages, n.i.e.  </td><td>   45</td><td>   10</td><td>  0</td><td>  120</td></tr>
</tbody>
</table>

> Note: you can see that above we also used another argument: `col_names = FALSE`. This is because this version of the data set had no column names, and if we have such a file and don't specify that, then the first observation will be taken (incorrectly) to be the column names. Another useful extension of this arguement is using it to assign column names, see the example below:

> ```
> can_lang3 <- read_delim("data/can_lang.tsv",  
>                      delim = "\t", 
>                      col_names = c("category", "language", "mother_tongue", 
>                                  "most_at_home", "most_at_work", "lang_known"))
> ```

#### Reading tabular data directly from a URL

- We can also use `read_csv` or `read_delim` (and related functions) to read in tabular data directly from a url that contains tabular data
- In this case, we provide the url to the `read_*` function as the path to the file instead of a path to a local file on our computer

Reading in https://github.com/ttimbers/canlang/blob/master/inst/extdata/can_lang.csv

```R
can_lang4 <- read_csv("https://raw.githubusercontent.com/ttimbers/canlang/master/inst/extdata/can_lang.csv")
head(can_lang4)
```

    [1mRows: [22m[34m214[39m [1mColumns: [22m[34m6[39m
    [36m──[39m [1mColumn specification[22m [36m──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────[39m
    [1mDelimiter:[22m ","
    [31mchr[39m (2): category, language
    [32mdbl[39m (4): mother_tongue, most_at_home, most_at_work, lang_known

    [36mℹ[39m Use`spec()` to retrieve the full column specification for this data.
    [36mℹ[39m Specify the column types or set `show_col_types = FALSE` to quiet this message.

<table class="dataframe">
<caption>A tibble: 6 × 6</caption>
<thead>
	<tr><th scope=col>category</th><th scope=col>language</th><th scope=col>mother_tongue</th><th scope=col>most_at_home</th><th scope=col>most_at_work</th><th scope=col>lang_known</th></tr>
	<tr><th scope=col><chr></th><th scope=col><chr></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Aboriginal languages                   </td><td>Aboriginal languages, n.o.s.  </td><td>  590</td><td>  235</td><td> 30</td><td>  665</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td><span style=white-space:pre-wrap>Afrikaans                     </span></td><td>10260</td><td> 4785</td><td> 85</td><td>23415</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td>Afro-Asiatic languages, n.i.e.</td><td> 1150</td><td><span style=white-space:pre-wrap>  445</span></td><td> 10</td><td> 2775</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td><span style=white-space:pre-wrap>Akan (Twi)                    </span></td><td>13460</td><td> 5985</td><td> 25</td><td>22150</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td><span style=white-space:pre-wrap>Albanian                      </span></td><td>26895</td><td>13135</td><td>345</td><td>31930</td></tr>
	<tr><td>Aboriginal languages                   </td><td>Algonquian languages, n.i.e.  </td><td>   45</td><td>   10</td><td>  0</td><td>  120</td></tr>
</tbody>
</table>

#### Reading data from an Microsoft Excel file

- it is very common to encounter, and need to load into R, data stored as a Microsoft Excel spreadsheet (with the filename extension `.xlsx`)
- To be able to do this, a key thing to know is that even though .csv and `.xlsx` files look almost identical when loaded into Excel, the data themselves are stored completely differently.

Read in `data/can_lang.xlsx`:

```R
library(readxl)
```

```R
can_lang5 <- read_excel("data/can_lang.xlsx")
head(can_lang5)
```

<table class="dataframe">
<caption>A tibble: 6 × 6</caption>
<thead>
	<tr><th scope=col>category</th><th scope=col>language</th><th scope=col>mother_tongue</th><th scope=col>most_at_home</th><th scope=col>most_at_work</th><th scope=col>lang_known</th></tr>
	<tr><th scope=col><chr></th><th scope=col><chr></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Aboriginal languages                   </td><td>Aboriginal languages, n.o.s.  </td><td>  590</td><td>  235</td><td> 30</td><td>  665</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td><span style=white-space:pre-wrap>Afrikaans                     </span></td><td>10260</td><td> 4785</td><td> 85</td><td>23415</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td>Afro-Asiatic languages, n.i.e.</td><td> 1150</td><td><span style=white-space:pre-wrap>  445</span></td><td> 10</td><td> 2775</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td><span style=white-space:pre-wrap>Akan (Twi)                    </span></td><td>13460</td><td> 5985</td><td> 25</td><td>22150</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td><span style=white-space:pre-wrap>Albanian                      </span></td><td>26895</td><td>13135</td><td>345</td><td>31930</td></tr>
	<tr><td>Aboriginal languages                   </td><td>Algonquian languages, n.i.e.  </td><td>   45</td><td>   10</td><td>  0</td><td>  120</td></tr>
</tbody>
</table>

Note - if there are multiple sheets, use the `sheet` argument to specify the sheet number or name.

#### Reading a Microsoft Excel file from the web

When trying to read a Microsoft Excel file from the web into R using a URL you cannot just pass the URL to `read_excel`.

First you must download the file, and then read it locally from there:

```R
url <- "https://github.com/ttimbers/canlang/blob/master/inst/extdata/can_lang.xlsx?raw=true"
download.file(url, "temp.xlsx")
can_lang6 <- read_excel("temp.xlsx")
head(can_lang6)
```

<table class="dataframe">
<caption>A tibble: 6 × 6</caption>
<thead>
	<tr><th scope=col>category</th><th scope=col>language</th><th scope=col>mother_tongue</th><th scope=col>most_at_home</th><th scope=col>most_at_work</th><th scope=col>lang_known</th></tr>
	<tr><th scope=col><chr></th><th scope=col><chr></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Aboriginal languages                   </td><td>Aboriginal languages, n.o.s.  </td><td>  590</td><td>  235</td><td> 30</td><td>  665</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td><span style=white-space:pre-wrap>Afrikaans                     </span></td><td>10260</td><td> 4785</td><td> 85</td><td>23415</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td>Afro-Asiatic languages, n.i.e.</td><td> 1150</td><td><span style=white-space:pre-wrap>  445</span></td><td> 10</td><td> 2775</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td><span style=white-space:pre-wrap>Akan (Twi)                    </span></td><td>13460</td><td> 5985</td><td> 25</td><td>22150</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td><span style=white-space:pre-wrap>Albanian                      </span></td><td>26895</td><td>13135</td><td>345</td><td>31930</td></tr>
	<tr><td>Aboriginal languages                   </td><td>Algonquian languages, n.i.e.  </td><td>   45</td><td>   10</td><td>  0</td><td>  120</td></tr>
</tbody>
</table>

> Note: When using `download.file` on Windows for Excel files (e.g., `.xlsx`) you need to specify an additional argument: `mode = "wb"`. For example:

```
download.file(url, "temp.xlsx", mode = "wb")
```

#### Note about loading data

- It's important to do it carefully + check results after!
  - will help reduce bugs and speed up your analyses down the road
- Think of it as tying your shoes before you run; not exciting, but if done wrong it will trip you up later!

<center><img src="https://media.giphy.com/media/Se449o7yNjziw/giphy.gif"/></center>

#### Writing files from R to a `.csv` file

- Once you have modified or summarized some data, you often want to save that data to a file for use later, or sharing with others.
- One of the most common formats for this is `.csv`
- To write to `.csv` in R we recommend using `readr::write_csv` as it uses sensible choices for a `.csv` file (e.g., includes column, but not row names, does not write quotes around the data, uses a comma as the delimiter, etc)

```R
write_csv(can_lang6, "data/can_lang7.csv")
```

    Error in write_csv(can_lang6, "data/can_lang7.csv"): could not find function "write_csv"
    Traceback:

### Cleaning up column names

Column names from wild data can be wild! For example, column names with symbols or white space require special syntax (surround the column names with back ticks) to program with them in R. One of the first things you want to do when you read in data can be to clean these up!

If things are not too bad, and there are not too many, `rename` can be a friend. Let's use `rename` to change the column names that have white space to using underscores. Let's look at the `can_lang-colnames.csv` file for example:

```R
can_lang8 <- read_csv("data/can_lang-colnames.csv")
head(can_lang8)
```

    [1mRows: [22m[34m214[39m [1mColumns: [22m[34m6[39m
    [36m──[39m [1mColumn specification[22m [36m──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────[39m
    [1mDelimiter:[22m ","
    [31mchr[39m (2): Category, Language
    [32mdbl[39m (4): Mother tongue, Spoken most at home, Spoken most at work, Language k...

    [36mℹ[39m Use`spec()` to retrieve the full column specification for this data.
    [36mℹ[39m Specify the column types or set `show_col_types = FALSE` to quiet this message.

<table class="dataframe">
<caption>A tibble: 6 × 6</caption>
<thead>
	<tr><th scope=col>Category</th><th scope=col>Language</th><th scope=col>Mother tongue</th><th scope=col>Spoken most at home</th><th scope=col>Spoken most at work</th><th scope=col>Language known</th></tr>
	<tr><th scope=col><chr></th><th scope=col><chr></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Aboriginal languages                   </td><td>Aboriginal languages, n.o.s.  </td><td>  590</td><td>  235</td><td> 30</td><td>  665</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td><span style=white-space:pre-wrap>Afrikaans                     </span></td><td>10260</td><td> 4785</td><td> 85</td><td>23415</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td>Afro-Asiatic languages, n.i.e.</td><td> 1150</td><td><span style=white-space:pre-wrap>  445</span></td><td> 10</td><td> 2775</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td><span style=white-space:pre-wrap>Akan (Twi)                    </span></td><td>13460</td><td> 5985</td><td> 25</td><td>22150</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td><span style=white-space:pre-wrap>Albanian                      </span></td><td>26895</td><td>13135</td><td>345</td><td>31930</td></tr>
	<tr><td>Aboriginal languages                   </td><td>Algonquian languages, n.i.e.  </td><td>   45</td><td>   10</td><td>  0</td><td>  120</td></tr>
</tbody>
</table>

```R
can_lang9 <- rename(can_lang8, Mother_tongue = `Mother tongue`,
                   Spoken_most_at_home = `Spoken most at home`,
                   Spoken_most_at_work = `Spoken most at work`,
                   Language_known = `Language known`)
head(can_lang9)
```

<table class="dataframe">
<caption>A tibble: 6 × 6</caption>
<thead>
	<tr><th scope=col>Category</th><th scope=col>Language</th><th scope=col>Mother_tongue</th><th scope=col>Spoken_most_at_home</th><th scope=col>Spoken_most_at_work</th><th scope=col>Language_known</th></tr>
	<tr><th scope=col><chr></th><th scope=col><chr></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Aboriginal languages                   </td><td>Aboriginal languages, n.o.s.  </td><td>  590</td><td>  235</td><td> 30</td><td>  665</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td><span style=white-space:pre-wrap>Afrikaans                     </span></td><td>10260</td><td> 4785</td><td> 85</td><td>23415</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td>Afro-Asiatic languages, n.i.e.</td><td> 1150</td><td><span style=white-space:pre-wrap>  445</span></td><td> 10</td><td> 2775</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td><span style=white-space:pre-wrap>Akan (Twi)                    </span></td><td>13460</td><td> 5985</td><td> 25</td><td>22150</td></tr>
	<tr><td>Non-Official & Non-Aboriginal languages</td><td><span style=white-space:pre-wrap>Albanian                      </span></td><td>26895</td><td>13135</td><td>345</td><td>31930</td></tr>
	<tr><td>Aboriginal languages                   </td><td>Algonquian languages, n.i.e.  </td><td>   45</td><td>   10</td><td>  0</td><td>  120</td></tr>
</tbody>
</table>

#### Cleaning up a lot of column names

Sometimes there are A LOT of column names with non-syntactic names. `rename` is not the best choice here. Instead we'll use the `clean_names` from the {janitor} package to transform all column names to syntactic ones.

```R
library(janitor)
```

    Attaching package: ‘janitor’

    The following objects are masked from ‘package:stats’:

    chisq.test, fisher.test

```R
census <- read_csv("data/census_snippet.csv")
head(census, 3)
```

    [1mRows: [22m[34m12[39m [1mColumns: [22m[34m16[39m
    [36m──[39m [1mColumn specification[22m [36m──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────[39m
    [1mDelimiter:[22m ","
    [31mchr[39m  (4): Geographic name, Geographic type, Geographic name, Province or ter...
    [32mdbl[39m (12): Geographic code, Geographic code, Province or territory, Global no...

    [36mℹ[39m Use`spec()` to retrieve the full column specification for this data.
    [36mℹ[39m Specify the column types or set `show_col_types = FALSE` to quiet this message.

<table class="dataframe">
<caption>A tibble: 3 × 16</caption>
<thead>
	<tr><th scope=col>Geographic code</th><th scope=col>Geographic name</th><th scope=col>Geographic type</th><th scope=col>Geographic name, Province or territory</th><th scope=col>Geographic code, Province or territory</th><th scope=col>Global non-response rate</th><th scope=col>Data quality flag</th><th scope=col>Household type</th><th scope=col>Number of households, 2006</th><th scope=col>Number of households, 2016</th><th scope=col>Median household total income (2015 constant dollars), 2005</th><th scope=col>Median household total income (2015 constant dollars), 2015</th><th scope=col>Median household total income (2015 constant dollars), % change</th><th scope=col>Median household after-tax income (2015 constant dollars), 2005</th><th scope=col>Median household after-tax income (2015 constant dollars), 2015</th><th scope=col>Median household after-tax income (2015 constant dollars), % change</th></tr>
	<tr><th scope=col><dbl></th><th scope=col><chr></th><th scope=col><chr></th><th scope=col><chr></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><chr></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>1001</td><td>Division No.  1</td><td>Census division</td><td>Newfoundland and Labrador</td><td>10</td><td>3.7</td><td>0</td><td>Total – Household type including census family structure                  </td><td>97015</td><td>112620</td><td>57667</td><td>74676</td><td>29.5</td><td>49750</td><td>64504</td><td>29.7</td></tr>
	<tr><td>1001</td><td>Division No.  1</td><td>Census division</td><td>Newfoundland and Labrador</td><td>10</td><td>3.7</td><td>0</td><td>Census-family households                                                  </td><td>71960</td><td> 78825</td><td>69720</td><td>96124</td><td>37.9</td><td>59713</td><td>81486</td><td>36.5</td></tr>
	<tr><td>1001</td><td>Division No.  1</td><td>Census division</td><td>Newfoundland and Labrador</td><td>10</td><td>3.7</td><td>0</td><td>Households consisting of only one census family without additional persons</td><td>66250</td><td> 73020</td><td>68837</td><td>94566</td><td>37.4</td><td>58715</td><td>80012</td><td>36.3</td></tr>
</tbody>
</table>

```R
clean_census <- clean_names(census)
head(clean_census, 3)
```

<table class="dataframe">
<caption>A tibble: 3 × 16</caption>
<thead>
	<tr><th scope=col>geographic_code</th><th scope=col>geographic_name</th><th scope=col>geographic_type</th><th scope=col>geographic_name_province_or_territory</th><th scope=col>geographic_code_province_or_territory</th><th scope=col>global_non_response_rate</th><th scope=col>data_quality_flag</th><th scope=col>household_type</th><th scope=col>number_of_households_2006</th><th scope=col>number_of_households_2016</th><th scope=col>median_household_total_income_2015_constant_dollars_2005</th><th scope=col>median_household_total_income_2015_constant_dollars_2015</th><th scope=col>median_household_total_income_2015_constant_dollars_percent_change</th><th scope=col>median_household_after_tax_income_2015_constant_dollars_2005</th><th scope=col>median_household_after_tax_income_2015_constant_dollars_2015</th><th scope=col>median_household_after_tax_income_2015_constant_dollars_percent_change</th></tr>
	<tr><th scope=col><dbl></th><th scope=col><chr></th><th scope=col><chr></th><th scope=col><chr></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><chr></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>1001</td><td>Division No.  1</td><td>Census division</td><td>Newfoundland and Labrador</td><td>10</td><td>3.7</td><td>0</td><td>Total – Household type including census family structure                  </td><td>97015</td><td>112620</td><td>57667</td><td>74676</td><td>29.5</td><td>49750</td><td>64504</td><td>29.7</td></tr>
	<tr><td>1001</td><td>Division No.  1</td><td>Census division</td><td>Newfoundland and Labrador</td><td>10</td><td>3.7</td><td>0</td><td>Census-family households                                                  </td><td>71960</td><td> 78825</td><td>69720</td><td>96124</td><td>37.9</td><td>59713</td><td>81486</td><td>36.5</td></tr>
	<tr><td>1001</td><td>Division No.  1</td><td>Census division</td><td>Newfoundland and Labrador</td><td>10</td><td>3.7</td><td>0</td><td>Households consisting of only one census family without additional persons</td><td>66250</td><td> 73020</td><td>68837</td><td>94566</td><td>37.4</td><td>58715</td><td>80012</td><td>36.3</td></tr>
</tbody>
</table>

`clean_names` is a thing of great beauty!!!

## Single data frame manipulations

Here we are going to learn how to use some of the most essential single data frame manipulation functions from the {dplyr} tidyverse package. To explore these, we will work with some of the data from the [Gapminder project](https://www.gapminder.org/). Jenny Bryan (MDS Founder and Software Developer at RStudio) released this as an R package called {[gapminder](https://github.com/jennybc/gapminder)}. We can load the data by loading the {gapminder} library:

```R
library(gapminder)
```

Let's take a quick look at the first 6 rows using `head`:

```R
head(gapminder)
```

<table class="dataframe">
<caption>A tibble: 6 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col><fct></th><th scope=col><fct></th><th scope=col><int></th><th scope=col><dbl></th><th scope=col><int></th><th scope=col><dbl></th></tr>
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

### Use `select` to subset columns

`select` from the [dplyr] package is used to subset the data on variables or columns. Here’s a conventional call to select two columns, `year` and `lifeExp`:

```R
select(gapminder, year, lifeExp)
```

<table class="dataframe">
<caption>A tibble: 1704 × 2</caption>
<thead>
	<tr><th scope=col>year</th><th scope=col>lifeExp</th></tr>
	<tr><th scope=col><int></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>1952</td><td>28.801</td></tr>
	<tr><td>1957</td><td>30.332</td></tr>
	<tr><td>1962</td><td>31.997</td></tr>
	<tr><td>1967</td><td>34.020</td></tr>
	<tr><td>1972</td><td>36.088</td></tr>
	<tr><td>1977</td><td>38.438</td></tr>
	<tr><td>1982</td><td>39.854</td></tr>
	<tr><td>1987</td><td>40.822</td></tr>
	<tr><td>1992</td><td>41.674</td></tr>
	<tr><td>1997</td><td>41.763</td></tr>
	<tr><td>2002</td><td>42.129</td></tr>
	<tr><td>2007</td><td>43.828</td></tr>
	<tr><td>1952</td><td>55.230</td></tr>
	<tr><td>1957</td><td>59.280</td></tr>
	<tr><td>1962</td><td>64.820</td></tr>
	<tr><td>1967</td><td>66.220</td></tr>
	<tr><td>1972</td><td>67.690</td></tr>
	<tr><td>1977</td><td>68.930</td></tr>
	<tr><td>1982</td><td>70.420</td></tr>
	<tr><td>1987</td><td>72.000</td></tr>
	<tr><td>1992</td><td>71.581</td></tr>
	<tr><td>1997</td><td>72.950</td></tr>
	<tr><td>2002</td><td>75.651</td></tr>
	<tr><td>2007</td><td>76.423</td></tr>
	<tr><td>1952</td><td>43.077</td></tr>
	<tr><td>1957</td><td>45.685</td></tr>
	<tr><td>1962</td><td>48.303</td></tr>
	<tr><td>1967</td><td>51.407</td></tr>
	<tr><td>1972</td><td>54.518</td></tr>
	<tr><td>1977</td><td>58.014</td></tr>
	<tr><td>⋮</td><td>⋮</td></tr>
	<tr><td>1982</td><td>49.113</td></tr>
	<tr><td>1987</td><td>52.922</td></tr>
	<tr><td>1992</td><td>55.599</td></tr>
	<tr><td>1997</td><td>58.020</td></tr>
	<tr><td>2002</td><td>60.308</td></tr>
	<tr><td>2007</td><td>62.698</td></tr>
	<tr><td>1952</td><td>42.038</td></tr>
	<tr><td>1957</td><td>44.077</td></tr>
	<tr><td>1962</td><td>46.023</td></tr>
	<tr><td>1967</td><td>47.768</td></tr>
	<tr><td>1972</td><td>50.107</td></tr>
	<tr><td>1977</td><td>51.386</td></tr>
	<tr><td>1982</td><td>51.821</td></tr>
	<tr><td>1987</td><td>50.821</td></tr>
	<tr><td>1992</td><td>46.100</td></tr>
	<tr><td>1997</td><td>40.238</td></tr>
	<tr><td>2002</td><td>39.193</td></tr>
	<tr><td>2007</td><td>42.384</td></tr>
	<tr><td>1952</td><td>48.451</td></tr>
	<tr><td>1957</td><td>50.469</td></tr>
	<tr><td>1962</td><td>52.358</td></tr>
	<tr><td>1967</td><td>53.995</td></tr>
	<tr><td>1972</td><td>55.635</td></tr>
	<tr><td>1977</td><td>57.674</td></tr>
	<tr><td>1982</td><td>60.363</td></tr>
	<tr><td>1987</td><td>62.351</td></tr>
	<tr><td>1992</td><td>60.377</td></tr>
	<tr><td>1997</td><td>46.809</td></tr>
	<tr><td>2002</td><td>39.989</td></tr>
	<tr><td>2007</td><td>43.487</td></tr>
</tbody>
</table>

Wow! That's a lot of rows... For teaching purposes I am intentionally not binding a name to the output of these commands, as they are for demonstration only, however, that means we don't have an object to call `head` on... In Jupyter we can use this `options` function call to limit the number of rows output when we call an entire data frame.

```R
# run this command to limit data frame output to 10 rows
options(repr.matrix.max.rows = 10)
```

```R
select(gapminder, year, lifeExp)
```

<table class="dataframe">
<caption>A tibble: 1704 × 2</caption>
<thead>
	<tr><th scope=col>year</th><th scope=col>lifeExp</th></tr>
	<tr><th scope=col><int></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>1952</td><td>28.801</td></tr>
	<tr><td>1957</td><td>30.332</td></tr>
	<tr><td>1962</td><td>31.997</td></tr>
	<tr><td>1967</td><td>34.020</td></tr>
	<tr><td>1972</td><td>36.088</td></tr>
	<tr><td>⋮</td><td>⋮</td></tr>
	<tr><td>1987</td><td>62.351</td></tr>
	<tr><td>1992</td><td>60.377</td></tr>
	<tr><td>1997</td><td>46.809</td></tr>
	<tr><td>2002</td><td>39.989</td></tr>
	<tr><td>2007</td><td>43.487</td></tr>
</tbody>
</table>

You can also use `select` to get a range of columns using names:

```R
select(gapminder, country:lifeExp)
```

<table class="dataframe">
<caption>A tibble: 1704 × 4</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th></tr>
	<tr><th scope=col><fct></th><th scope=col><fct></th><th scope=col><int></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Afghanistan</td><td>Asia</td><td>1952</td><td>28.801</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1957</td><td>30.332</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1962</td><td>31.997</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1967</td><td>34.020</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1972</td><td>36.088</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>1987</td><td>62.351</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>1992</td><td>60.377</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>1997</td><td>46.809</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>2002</td><td>39.989</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>2007</td><td>43.487</td></tr>
</tbody>
</table>

### Use `filter` to subset rows that meet a specific condition

`filter` from the {dplyr package, takes logical expressions and returns the rows for which all are `TRUE`, for example we can subset rows which have a life expectancy > 29:

```R
filter(gapminder, lifeExp < 29)
```

<table class="dataframe">
<caption>A tibble: 2 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col><fct></th><th scope=col><fct></th><th scope=col><int></th><th scope=col><dbl></th><th scope=col><int></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Afghanistan</td><td>Asia  </td><td>1952</td><td>28.801</td><td>8425333</td><td>779.4453</td></tr>
	<tr><td>Rwanda     </td><td>Africa</td><td>1992</td><td>23.599</td><td>7290203</td><td>737.0686</td></tr>
</tbody>
</table>

We can susbet rows that meet two conditions, for example we can subset rows which are from the country Rwanda and whose year is > 1979. Note we use the comma `,` to separate the two conditions:

```R
filter(gapminder, country == "Rwanda", year > 1979)
```

<table class="dataframe">
<caption>A tibble: 6 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col><fct></th><th scope=col><fct></th><th scope=col><int></th><th scope=col><dbl></th><th scope=col><int></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Rwanda</td><td>Africa</td><td>1982</td><td>46.218</td><td>5507565</td><td>881.5706</td></tr>
	<tr><td>Rwanda</td><td>Africa</td><td>1987</td><td>44.020</td><td>6349365</td><td>847.9912</td></tr>
	<tr><td>Rwanda</td><td>Africa</td><td>1992</td><td>23.599</td><td>7290203</td><td>737.0686</td></tr>
	<tr><td>Rwanda</td><td>Africa</td><td>1997</td><td>36.087</td><td>7212583</td><td>589.9445</td></tr>
	<tr><td>Rwanda</td><td>Africa</td><td>2002</td><td>43.413</td><td>7852401</td><td>785.6538</td></tr>
	<tr><td>Rwanda</td><td>Africa</td><td>2007</td><td>46.242</td><td>8860588</td><td>863.0885</td></tr>
</tbody>
</table>

to filter for multiple conditions that **need not** co-occur, use `|`:

```R
filter(gapminder, lifeExp > 80 | year == 2007)
```

<table class="dataframe">
<caption>A tibble: 150 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col><fct></th><th scope=col><fct></th><th scope=col><int></th><th scope=col><dbl></th><th scope=col><int></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Afghanistan</td><td>Asia    </td><td>2007</td><td>43.828</td><td>31889923</td><td>  974.5803</td></tr>
	<tr><td>Albania    </td><td>Europe  </td><td>2007</td><td>76.423</td><td> 3600523</td><td> 5937.0295</td></tr>
	<tr><td>Algeria    </td><td>Africa  </td><td>2007</td><td>72.301</td><td>33333216</td><td> 6223.3675</td></tr>
	<tr><td>Angola     </td><td>Africa  </td><td>2007</td><td>42.731</td><td>12420476</td><td> 4797.2313</td></tr>
	<tr><td>Argentina  </td><td>Americas</td><td>2007</td><td>75.320</td><td>40301927</td><td>12779.3796</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>Vietnam           </td><td>Asia  </td><td>2007</td><td>74.249</td><td>85262356</td><td>2441.5764</td></tr>
	<tr><td>West Bank and Gaza</td><td>Asia  </td><td>2007</td><td>73.422</td><td> 4018332</td><td>3025.3498</td></tr>
	<tr><td>Yemen, Rep.       </td><td>Asia  </td><td>2007</td><td>62.698</td><td>22211743</td><td>2280.7699</td></tr>
	<tr><td>Zambia            </td><td>Africa</td><td>2007</td><td>42.384</td><td>11746035</td><td>1271.2116</td></tr>
	<tr><td>Zimbabwe          </td><td>Africa</td><td>2007</td><td>43.487</td><td>12311143</td><td> 469.7093</td></tr>
</tbody>
</table>

Finally, we can use the `%in%` operator to subset rows which have values that match a value from several possible values:

```R
filter(gapminder, country %in% c("Mexico", "United States", "Canada"))
```

<table class="dataframe">
<caption>A tibble: 36 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col><fct></th><th scope=col><fct></th><th scope=col><int></th><th scope=col><dbl></th><th scope=col><int></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Canada</td><td>Americas</td><td>1952</td><td>68.75</td><td>14785584</td><td>11367.16</td></tr>
	<tr><td>Canada</td><td>Americas</td><td>1957</td><td>69.96</td><td>17010154</td><td>12489.95</td></tr>
	<tr><td>Canada</td><td>Americas</td><td>1962</td><td>71.30</td><td>18985849</td><td>13462.49</td></tr>
	<tr><td>Canada</td><td>Americas</td><td>1967</td><td>72.13</td><td>20819767</td><td>16076.59</td></tr>
	<tr><td>Canada</td><td>Americas</td><td>1972</td><td>72.88</td><td>22284500</td><td>18970.57</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>United States</td><td>Americas</td><td>1987</td><td>75.020</td><td>242803533</td><td>29884.35</td></tr>
	<tr><td>United States</td><td>Americas</td><td>1992</td><td>76.090</td><td>256894189</td><td>32003.93</td></tr>
	<tr><td>United States</td><td>Americas</td><td>1997</td><td>76.810</td><td>272911760</td><td>35767.43</td></tr>
	<tr><td>United States</td><td>Americas</td><td>2002</td><td>77.310</td><td>287675526</td><td>39097.10</td></tr>
	<tr><td>United States</td><td>Americas</td><td>2007</td><td>78.242</td><td>301139947</td><td>42951.65</td></tr>
</tbody>
</table>

### Combining functions with the pipe `|>` operator

In R, we often have to call multiple functions in a sequence to process a data frame. The basic ways of doing this can become quickly unreadable if there are many steps. For example, suppose we need to perform three operations on a data frame `data`:

1. add a new column `new_col` that is double another `old_col`
2. filter for rows where another column, `other_col`, is more than 5, and
3. select only the new column `new_col` for those rows.

One way of doing is to just write multiple lines of code, storing temporary objects as you go:

```
output_1 <- mutate(data, new_col = old_col * 2)
output_2 <- filter(output_1, other_col > 5)
output <- select(output_2, new_col)
```

This is difficult to understand for multiple reasons. The reader may be tricked into thinking the named `output_1` and `output_2` objects are important for some reason, while they are just temporary intermediate computations. Further, the reader has to look through and find where `output_1` and `output_2` are used in each subsequent line.

Another option for doing this would be to compose the functions:

```
output <- select(filter(mutate(data, new_col = old_col * 2), other_col > 5), new_col)
```

Code like this can also be difficult to understand. Functions compose (reading from left to right) in the opposite order in which they are computed by R (above, `mutate` happens first, then `filter`, then `select`). It is also just a really long line of code to read in one go.

The pipe operator `|>` solves this problem, resulting in cleaner and easier-to-follow code. |> in built into R so you don’t need to load any packages to use it. The code below accomplishes the same thing as the previous two code blocks:

```
output <- data |>
mutate(new_col = old_col * 2) |>
filter(other_col > 5) |>
select(new_col)
```

The pipe operator takes the thing on the left-hand-side and pipes it into the function call on the right-hand-side – ***literally, drops it in as the first argument.***

- this year (2021) R created `|>` as a built-in pipe operator!!!
- it was inspired from the pipe from the `magrittr` package `%>%`; which is imported by the `tidyverse` (and `dplyr`) package

![](http://hexb.in/hexagons/magrittr.png)
*Referring to the [1948 painting La Trahison des images](https://www.wikiart.org/en/rene-magritte/the-treachery-of-images-this-is-not-a-pipe-1948) by Rene Magritte*

So now we also have:

Logo by [@LuisDVerde](https://twitter.com/LuisDVerde)

*Source: https://twitter.com/LuisDVerde/status/1430905603405144065*

Which to use? Most of the time there will be no difference. I will be adopting `|>` in my notes, homeworks, etc however, some from past times will use `%>%` and you will still see `|>` in other people's code.

I am choosing `|>` because I can then use piping in R packages without depending upon the `magrittr` package. More on this in DSCI 524: Collaborative Software Developement.

So instead of using intermediate objects to combine two functions (*e.g.*, `select` & `filter`) like this:

```R
gap_under_29 <- filter(gapminder, lifeExp < 29)
select(gap_under_29, country, year)
```

<table class="dataframe">
<caption>A tibble: 2 × 2</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>year</th></tr>
	<tr><th scope=col><fct></th><th scope=col><int></th></tr>
</thead>
<tbody>
	<tr><td>Afghanistan</td><td>1952</td></tr>
	<tr><td>Rwanda     </td><td>1992</td></tr>
</tbody>
</table>

And instead of composing functions in a hard to read manner:

```R
select(filter(gapminder, lifeExp < 29), country, year)
```

<table class="dataframe">
<caption>A tibble: 2 × 2</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>year</th></tr>
	<tr><th scope=col><fct></th><th scope=col><int></th></tr>
</thead>
<tbody>
	<tr><td>Afghanistan</td><td>1952</td></tr>
	<tr><td>Rwanda     </td><td>1992</td></tr>
</tbody>
</table>

We can use the pipe `|>` to create easy to read piplines to connect our function calls:

```R
filter(gapminder, lifeExp < 29) |>
    select(country, year)
```

<table class="dataframe">
<caption>A tibble: 2 × 2</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>year</th></tr>
	<tr><th scope=col><fct></th><th scope=col><int></th></tr>
</thead>
<tbody>
	<tr><td>Afghanistan</td><td>1952</td></tr>
	<tr><td>Rwanda     </td><td>1992</td></tr>
</tbody>
</table>

We can even start out our pipelines with an R data frame!

```R
gapminder |>
    filter(lifeExp < 29) |>
    select(country, year)
```

<table class="dataframe">
<caption>A tibble: 2 × 2</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>year</th></tr>
	<tr><th scope=col><fct></th><th scope=col><int></th></tr>
</thead>
<tbody>
	<tr><td>Afghanistan</td><td>1952</td></tr>
	<tr><td>Rwanda     </td><td>1992</td></tr>
</tbody>
</table>

We will embrace the pipe going forward in MDS!

#### Pro-tip for creating objects using the pipe

Don't start writing your code by assigning to an object - it's hard to debug and find the errors:

```
new_df <- gapminder |> 
    filter(country == "Cabodia") |>
    select(Year, lifeExp) 
```

```
Error in .f(.x[[i]], ...): object 'Year' not found
Traceback:

1. gapminder |> filter(country == "Cabodia") |> select(Year, lifeExp)
2. withVisible(eval(quote(`_fseq`(`_lhs`)), env, env))
3. eval(quote(`_fseq`(`_lhs`)), env, env)
4. eval(quote(`_fseq`(`_lhs`)), env, env)
5. `_fseq`(`_lhs`)
6. freduce(value, `_function_list`)
7. withVisible(function_list[[k]](value))
8. function_list[[k]](value)
9. select(., Year, lifeExp)
10. select.data.frame(., Year, lifeExp)
11. tidyselect::vars_select(tbl_vars(.data), !!!enquos(...))
12. vars_select_eval(.vars, quos)
13. map_if(quos, !is_helper, eval_tidy, mask)
14. map(.x[sel], .f, ...)
15. .f(.x[[i]], ...)
```

Instead, create your pipeline, just dumping it to the screen:

```R
gapminder |> 
    filter(country == "Cambodia") |>
    select(year, lifeExp)
```

<table class="dataframe">
<caption>A tibble: 12 × 2</caption>
<thead>
	<tr><th scope=col>year</th><th scope=col>lifeExp</th></tr>
	<tr><th scope=col><int></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>1952</td><td>39.417</td></tr>
	<tr><td>1957</td><td>41.366</td></tr>
	<tr><td>1962</td><td>43.415</td></tr>
	<tr><td>1967</td><td>45.415</td></tr>
	<tr><td>1972</td><td>40.317</td></tr>
	<tr><td>⋮</td><td>⋮</td></tr>
	<tr><td>1987</td><td>53.914</td></tr>
	<tr><td>1992</td><td>55.803</td></tr>
	<tr><td>1997</td><td>56.534</td></tr>
	<tr><td>2002</td><td>56.752</td></tr>
	<tr><td>2007</td><td>59.723</td></tr>
</tbody>
</table>

Then, once you know it works, assign it to an object:

```R
new_df <- gapminder |>
    filter(country == "Cambodia") |>
    select(year, lifeExp) 
```

### Use `mutate` to add new variables

Imagine we wanted to recover each country’s GDP. After all, the Gapminder data has a variable for population and GDP per capita. Let’s multiply them together.

`mutate` is a function that defines and inserts new variables into a tibble. You can refer to existing variables by name.

```R
gapminder |> 
    mutate(tot_gdp = pop * gdpPercap)
```

<table class="dataframe">
<caption>A tibble: 1704 × 7</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th><th scope=col>tot_gdp</th></tr>
	<tr><th scope=col><fct></th><th scope=col><fct></th><th scope=col><int></th><th scope=col><dbl></th><th scope=col><int></th><th scope=col><dbl></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Afghanistan</td><td>Asia</td><td>1952</td><td>28.801</td><td> 8425333</td><td>779.4453</td><td>6567086330</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1957</td><td>30.332</td><td> 9240934</td><td>820.8530</td><td>7585448670</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1962</td><td>31.997</td><td>10267083</td><td>853.1007</td><td>8758855797</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1967</td><td>34.020</td><td>11537966</td><td>836.1971</td><td>9648014150</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1972</td><td>36.088</td><td>13079460</td><td>739.9811</td><td>9678553274</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>1987</td><td>62.351</td><td> 9216418</td><td>706.1573</td><td>6508240905</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>1992</td><td>60.377</td><td>10704340</td><td>693.4208</td><td>7422611852</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>1997</td><td>46.809</td><td>11404948</td><td>792.4500</td><td>9037850590</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>2002</td><td>39.989</td><td>11926563</td><td>672.0386</td><td>8015110972</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>2007</td><td>43.487</td><td>12311143</td><td>469.7093</td><td>5782658337</td></tr>
</tbody>
</table>

To `mutate` more than one column, use commas to separate the mutation for each column:

```R
gapminder |>
    mutate(tot_gdp = pop * gdpPercap,
          pop_thousands = pop / 1000)
```

<table class="dataframe">
<caption>A tibble: 1704 × 8</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th><th scope=col>tot_gdp</th><th scope=col>pop_thousands</th></tr>
	<tr><th scope=col><fct></th><th scope=col><fct></th><th scope=col><int></th><th scope=col><dbl></th><th scope=col><int></th><th scope=col><dbl></th><th scope=col><dbl></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Afghanistan</td><td>Asia</td><td>1952</td><td>28.801</td><td> 8425333</td><td>779.4453</td><td>6567086330</td><td> 8425.333</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1957</td><td>30.332</td><td> 9240934</td><td>820.8530</td><td>7585448670</td><td> 9240.934</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1962</td><td>31.997</td><td>10267083</td><td>853.1007</td><td>8758855797</td><td>10267.083</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1967</td><td>34.020</td><td>11537966</td><td>836.1971</td><td>9648014150</td><td>11537.966</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1972</td><td>36.088</td><td>13079460</td><td>739.9811</td><td>9678553274</td><td>13079.460</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>1987</td><td>62.351</td><td> 9216418</td><td>706.1573</td><td>6508240905</td><td> 9216.418</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>1992</td><td>60.377</td><td>10704340</td><td>693.4208</td><td>7422611852</td><td>10704.340</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>1997</td><td>46.809</td><td>11404948</td><td>792.4500</td><td>9037850590</td><td>11404.948</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>2002</td><td>39.989</td><td>11926563</td><td>672.0386</td><td>8015110972</td><td>11926.563</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>2007</td><td>43.487</td><td>12311143</td><td>469.7093</td><td>5782658337</td><td>12311.143</td></tr>
</tbody>
</table>

We can even change a column in place:

```R
gapminder |>
    mutate(lifeExp = round(lifeExp, 0))
```

<table class="dataframe">
<caption>A tibble: 1704 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col><fct></th><th scope=col><fct></th><th scope=col><int></th><th scope=col><dbl></th><th scope=col><int></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Afghanistan</td><td>Asia</td><td>1952</td><td>29</td><td> 8425333</td><td>779.4453</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1957</td><td>30</td><td> 9240934</td><td>820.8530</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1962</td><td>32</td><td>10267083</td><td>853.1007</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1967</td><td>34</td><td>11537966</td><td>836.1971</td></tr>
	<tr><td>Afghanistan</td><td>Asia</td><td>1972</td><td>36</td><td>13079460</td><td>739.9811</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>1987</td><td>62</td><td> 9216418</td><td>706.1573</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>1992</td><td>60</td><td>10704340</td><td>693.4208</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>1997</td><td>47</td><td>11404948</td><td>792.4500</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>2002</td><td>40</td><td>11926563</td><td>672.0386</td></tr>
	<tr><td>Zimbabwe</td><td>Africa</td><td>2007</td><td>43</td><td>12311143</td><td>469.7093</td></tr>
</tbody>
</table>

### Use `arrange` to sort a data frame

Imagine we wanted to know the country (and year) with the shortest life expectancy? How can we do this?

`arrange` is a function that sorts data frames based on a column, or columns. Let's use it below to answer the questiom we just posed above!

```R
gapminder |>
    arrange(lifeExp)
```

<table class="dataframe">
<caption>A tibble: 1704 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col><fct></th><th scope=col><fct></th><th scope=col><int></th><th scope=col><dbl></th><th scope=col><int></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Rwanda      </td><td>Africa</td><td>1992</td><td>23.599</td><td>7290203</td><td> 737.0686</td></tr>
	<tr><td>Afghanistan </td><td>Asia  </td><td>1952</td><td>28.801</td><td>8425333</td><td> 779.4453</td></tr>
	<tr><td>Gambia      </td><td>Africa</td><td>1952</td><td>30.000</td><td> 284320</td><td> 485.2307</td></tr>
	<tr><td>Angola      </td><td>Africa</td><td>1952</td><td>30.015</td><td>4232095</td><td>3520.6103</td></tr>
	<tr><td>Sierra Leone</td><td>Africa</td><td>1952</td><td>30.331</td><td>2143249</td><td> 879.7877</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>Switzerland     </td><td>Europe</td><td>2007</td><td>81.701</td><td>  7554661</td><td>37506.42</td></tr>
	<tr><td>Iceland         </td><td>Europe</td><td>2007</td><td>81.757</td><td>   301931</td><td>36180.79</td></tr>
	<tr><td>Japan           </td><td>Asia  </td><td>2002</td><td>82.000</td><td>127065841</td><td>28604.59</td></tr>
	<tr><td>Hong Kong, China</td><td>Asia  </td><td>2007</td><td>82.208</td><td>  6980412</td><td>39724.98</td></tr>
	<tr><td>Japan           </td><td>Asia  </td><td>2007</td><td>82.603</td><td>127467972</td><td>31656.07</td></tr>
</tbody>
</table>

What about the hightest life expectancy? We can pair the `arrange` function, with another function, called `desc` to sort the data frame in descending order:

```R
gapminder |>
    arrange(desc(lifeExp))
```

<table class="dataframe">
<caption>A tibble: 1704 × 6</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>continent</th><th scope=col>year</th><th scope=col>lifeExp</th><th scope=col>pop</th><th scope=col>gdpPercap</th></tr>
	<tr><th scope=col><fct></th><th scope=col><fct></th><th scope=col><int></th><th scope=col><dbl></th><th scope=col><int></th><th scope=col><dbl></th></tr>
</thead>
<tbody>
	<tr><td>Japan           </td><td>Asia  </td><td>2007</td><td>82.603</td><td>127467972</td><td>31656.07</td></tr>
	<tr><td>Hong Kong, China</td><td>Asia  </td><td>2007</td><td>82.208</td><td>  6980412</td><td>39724.98</td></tr>
	<tr><td>Japan           </td><td>Asia  </td><td>2002</td><td>82.000</td><td>127065841</td><td>28604.59</td></tr>
	<tr><td>Iceland         </td><td>Europe</td><td>2007</td><td>81.757</td><td>   301931</td><td>36180.79</td></tr>
	<tr><td>Switzerland     </td><td>Europe</td><td>2007</td><td>81.701</td><td>  7554661</td><td>37506.42</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>Sierra Leone</td><td>Africa</td><td>1952</td><td>30.331</td><td>2143249</td><td> 879.7877</td></tr>
	<tr><td>Angola      </td><td>Africa</td><td>1952</td><td>30.015</td><td>4232095</td><td>3520.6103</td></tr>
	<tr><td>Gambia      </td><td>Africa</td><td>1952</td><td>30.000</td><td> 284320</td><td> 485.2307</td></tr>
	<tr><td>Afghanistan </td><td>Asia  </td><td>1952</td><td>28.801</td><td>8425333</td><td> 779.4453</td></tr>
	<tr><td>Rwanda      </td><td>Africa</td><td>1992</td><td>23.599</td><td>7290203</td><td> 737.0686</td></tr>
</tbody>
</table>

### Getting a single value from a data frame

What if you want the name of the country with the longest life expectancy back as just a single character vector, and not a data frame? How do we do this?

We need two additional {dplyr} functions to get this job done using the tidyverse, `slice` and `pull`. `slice` allows you to numerically index rows in R, returning the row number(s) specified. `pull` removes the data frame structure, returning the data as the next simpler data structure in R, a vector.

Let's use `slice` and `pull` to now extract the name of the country with the longest life expectancy:

```R
gapminder |> 
    arrange(desc(lifeExp)) |>
    slice(1) |>
    pull(lifeExp)
```

82.603

We just learned A LOT of new functions and operators:

- `select`
- `filter`
- `mutate`
- `arrange`
- `desc`
- `slice`
- `pull`
- ` |>`
- `%in%`

Trust that with practice (and you will get a lot of practice by completing the worksheet and lab homeworks) you will learn these. Also, when needed, you can refer to the data transformation cheat sheet: https://github.com/rstudio/cheatsheets/raw/main/data-transformation.pdf

## Tidy data

#### Shameless borrowing of slides from Jenny Bryan

https://www.slideshare.net/Plotly/plotcon-nyc-behind-every-great-plot-theres-a-great-deal-of-wrangling


#### How should you wrangle your data?

#### We make it "tidy"!

### What is tidy data?

A tidy data is one that is satified by these three criteria:

- each row is a single observation,
- each variable is a single column, and
- each value is a single cell (i.e., its row, column position in the data frame is not shared with another value)

![](https://d33wubrfki0l68.cloudfront.net/6f1ddb544fc5c69a2478e444ab8112fb0eea23f8/91adc/images/tidy-1.png)

What is a variable and an observation may depend on your immediate goal.

*Source: [R for Data Science](https://r4ds.had.co.nz/) by Garrett Grolemund & Hadley Wickham*

### A tale of 4 data tables...

...here is the same data represented in 4 different ways, let's vote on which are tidy

*Example source: https://garrettgman.github.io/tidying/*

Statistical question: What variables are associated with the number of TB cases?

| country     | year | cases_per_capita  |
| ----------- | ---- | ----------------- |
| Afghanistan | 1999 | 745/19987071      |
| Afghanistan | 2000 | 2666/20595360     |
| Brazil      | 1999 | 37737/172006362   |
| Brazil      | 2000 | 80488/174504898   |
| China       | 1999 | 212258/1272915272 |
| China       | 2000 | 213766/1280428583 |

```{admonition}

A) TRUE

B) FALSE

```

```{toggle}

***Answer:  B. FALSE ***
2 different values stored in same cell.
```

Statistical question: What variables are associated with the number of TB cases?

| country     | cases (year=1999) | cases (year=2000) |
| ----------- | ----------------- | ----------------- |
| Afghanistan | 745               | 2666              |
| Brazil      | 37737             | 80488             |
| China       | 212258            | 213766            |

| country     | population (year=1999) | population (year=2000) |
| ----------- | ---------------------- | ---------------------- |
| Afghanistan | 19987071               | 20595360               |
| Brazil      | 172006362              | 174504898              |
| China       | 1272915272             | 1280428583             |

```{admonition}

A) TRUE

B) FALSE

```

```{toggle}

***Answer:  B. FALSE ***
- Information spread across 2 different tables.
- We have information on year associated with the column name and it is not easy to capture it programatically during our analysis.
```

Statistical question: What variables are associated with the number of TB cases?

| country     | year | cases  | population |
| ----------- | ---- | ------ | ---------- |
| Afghanistan | 1999 | 745    | 19987071   |
| Afghanistan | 2000 | 2666   | 20595360   |
| Brazil      | 1999 | 37737  | 172006362  |
| Brazil      | 2000 | 80488  | 174504898  |
| China       | 1999 | 212258 | 1272915272 |
| China       | 2000 | 213766 | 1280428583 |

```{admonition}

A) TRUE

B) FALSE

```

```{toggle}

***Answer:  B. TRUE ***
- Each observation is a single row
- Each variable is a single column 
- Each value has it's own cell
```

Statistical question: What variables are associated with the number of TB cases?

| country     | year | key        | value      |
| ----------- | ---- | ---------- | ---------- |
| Afghanistan | 1999 | cases      | 745        |
| Afghanistan | 1999 | population | 19987071   |
| Afghanistan | 2000 | cases      | 2666       |
| Afghanistan | 2000 | population | 20595360   |
| Brazil      | 1999 | cases      | 37737      |
| Brazil      | 1999 | population | 172006362  |
| Brazil      | 2000 | cases      | 80488      |
| Brazil      | 2000 | population | 174504898  |
| China       | 1999 | cases      | 212258     |
| China       | 1999 | population | 1272915272 |
| China       | 2000 | cases      | 213766     |
| China       | 2000 | population | 1280428583 |

```{admonition}

A) TRUE

B) FALSE

```

```{toggle}

***Answer:  B. FALSE ***
- Each variable is not its own column. Instead, we have cases and population in the key column and those values in the value column.
```

### Pivoting longer

`pivot_longer` from the {tidyr} package takes a wide data frame and making it longer! {tidyr} is another package in the {tidyverse} metapackage.

![ ](https://github.com/apreshill/teachthat/blob/master/pivot/pivot_longer_smaller.gif?raw=true)

*Source: April Hill's [teachthat](https://github.com/apreshill/teachthat/) GitHub repository*

Consider the data frame below and that we are interested in finding what variables are associated with the number of TB cases?

```R
table4a
```

<table class="dataframe">
<caption>A tibble: 3 × 3</caption>
<thead>
	<tr><th></th><th scope=col>country</th><th scope=col>1999</th><th scope=col>2000</th></tr>
	<tr><th></th><th scope=col><chr></th><th scope=col><int></th><th scope=col><int></th></tr>
</thead>
<tbody>
	<tr><th scope=row>1</th><td>Afghanistan</td><td>   745</td><td>  2666</td></tr>
	<tr><th scope=row>2</th><td>Brazil     </td><td> 37737</td><td> 80488</td></tr>
	<tr><th scope=row>3</th><td>China      </td><td>212258</td><td>213766</td></tr>
</tbody>
</table>

This is currently difficult as the values for the variable year are stuck in column names. We can use the `pivot_longer` function to tidy this data:

```R
table4a |>
    pivot_longer(`1999`:`2000`, names_to = "year", values_to = "cases")
```

<table class="dataframe">
<caption>A tibble: 6 × 3</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>year</th><th scope=col>cases</th></tr>
	<tr><th scope=col><chr></th><th scope=col><chr></th><th scope=col><int></th></tr>
</thead>
<tbody>
	<tr><td>Afghanistan</td><td>1999</td><td>   745</td></tr>
	<tr><td>Afghanistan</td><td>2000</td><td>  2666</td></tr>
	<tr><td>Brazil     </td><td>1999</td><td> 37737</td></tr>
	<tr><td>Brazil     </td><td>2000</td><td> 80488</td></tr>
	<tr><td>China      </td><td>1999</td><td>212258</td></tr>
	<tr><td>China      </td><td>2000</td><td>213766</td></tr>
</tbody>
</table>

```R
# or a less verbose and efficient way to specify this:
table4a |>
    pivot_longer(-country, names_to = "year", values_to = "cases")
```

<table class="dataframe">
<caption>A tibble: 6 × 3</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>year</th><th scope=col>cases</th></tr>
	<tr><th scope=col><chr></th><th scope=col><chr></th><th scope=col><int></th></tr>
</thead>
<tbody>
	<tr><td>Afghanistan</td><td>1999</td><td>   745</td></tr>
	<tr><td>Afghanistan</td><td>2000</td><td>  2666</td></tr>
	<tr><td>Brazil     </td><td>1999</td><td> 37737</td></tr>
	<tr><td>Brazil     </td><td>2000</td><td> 80488</td></tr>
	<tr><td>China      </td><td>1999</td><td>212258</td></tr>
	<tr><td>China      </td><td>2000</td><td>213766</td></tr>
</tbody>
</table>

To use `pivot_longer` we need to specify:

1. the dataset
2. The columns we want to apply our pivot operation to (we use `select`-like syntax for this, and if you want to operate on all columns in a data frame, you can provide the function `everything()` here.
3. the name of a new column that will be created, whose values will come from the names of the columns that we want to combine (the result argument)
4. the name of a new column that will be created, whose values will come from the values of the columns we want to combine (the value argument)

### Pivoting wider

`pivot_wider` from the {tidyr} package takes a narrow data frame and making it wider! It is the opposite of `pivot_longer`!

Consider the data frame below and that we are interested in finding what variables are associated with the number of TB cases?

```R
table2
```

<table class="dataframe">
<caption>A tibble: 12 × 4</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>year</th><th scope=col>type</th><th scope=col>count</th></tr>
	<tr><th scope=col><chr></th><th scope=col><int></th><th scope=col><chr></th><th scope=col><int></th></tr>
</thead>
<tbody>
	<tr><td>Afghanistan</td><td>1999</td><td>cases     </td><td>     745</td></tr>
	<tr><td>Afghanistan</td><td>1999</td><td>population</td><td>19987071</td></tr>
	<tr><td>Afghanistan</td><td>2000</td><td>cases     </td><td>    2666</td></tr>
	<tr><td>Afghanistan</td><td>2000</td><td>population</td><td>20595360</td></tr>
	<tr><td>Brazil     </td><td>1999</td><td>cases     </td><td>   37737</td></tr>
	<tr><td>⋮</td><td>⋮</td><td>⋮</td><td>⋮</td></tr>
	<tr><td>Brazil</td><td>2000</td><td>population</td><td> 174504898</td></tr>
	<tr><td>China </td><td>1999</td><td>cases     </td><td>    212258</td></tr>
	<tr><td>China </td><td>1999</td><td>population</td><td>1272915272</td></tr>
	<tr><td>China </td><td>2000</td><td>cases     </td><td>    213766</td></tr>
	<tr><td>China </td><td>2000</td><td>population</td><td>1280428583</td></tr>
</tbody>
</table>

This is currently difficult as observation is scattered across multiple rows. We can tidy this data frame with the function `pivot_wider`.

```R
table2 |>
    pivot_wider(names_from = type, values_from = count)
```

<table class="dataframe">
<caption>A tibble: 6 × 4</caption>
<thead>
	<tr><th scope=col>country</th><th scope=col>year</th><th scope=col>cases</th><th scope=col>population</th></tr>
	<tr><th scope=col><chr></th><th scope=col><int></th><th scope=col><int></th><th scope=col><int></th></tr>
</thead>
<tbody>
	<tr><td>Afghanistan</td><td>1999</td><td>   745</td><td>  19987071</td></tr>
	<tr><td>Afghanistan</td><td>2000</td><td>  2666</td><td>  20595360</td></tr>
	<tr><td>Brazil     </td><td>1999</td><td> 37737</td><td> 172006362</td></tr>
	<tr><td>Brazil     </td><td>2000</td><td> 80488</td><td> 174504898</td></tr>
	<tr><td>China      </td><td>1999</td><td>212258</td><td>1272915272</td></tr>
	<tr><td>China      </td><td>2000</td><td>213766</td><td>1280428583</td></tr>
</tbody>
</table>

To use `pivot_wider` we need to specify:

1. the dataset
2. the name of the column whose values we would like to use as column names when we widen the data
3. the name of the column whose values you would like to spread into separate columns based on the value they had in the column specified as the key

https://www.tidyverse.org/

### The tidy tools manifesto

"*The goal of these principles is to provide a uniform interface so that tidyverse packages work together naturally, and once you’ve mastered one, you have a head start on mastering the others.*"

https://cran.r-project.org/web/packages/tidyverse/vignettes/manifesto.html

### The tidy tools manifesto

There are four basic principles to a tidy API:

1. Reuse existing data structures.
2. Compose simple functions with the pipe (`|>`).
3. Embrace functional programming.
4. Design for humans.

### Reuse existing data structures.

- Where possible, re-use existing data structures, rather than creating custom data structures for your own package
- When it is not possible to use an existing data structure, use S3 class (simplest OO system in R) built on top of an atomic vector or list
- When working on data.frames/tibbles, assume the data is tidy

### Compose simple functions with the pipe (|>)

- A powerful strategy for solving complex problems is to combine many simple pieces. Each piece should be easily understood in isolation, and have a standard way to combine with other pieces.
- In R, this strategy plays out by composing single functions with the pipe, `|>`.
- The pipe is a common composition tool that works across all packages.

### Embrace functional programming

R is a functional programming language; embrace it, don’t fight it. If you’re familiar with an object-oriented language like Python or C#, this is going to take some adjustment. But in the long run you will be much better off working with the language rather than fighting it.

Generally, this means you should favour:

- Immutable objects and copy-on-modify semantics. This makes your code easier to reason about.
- The generic functions provided by S3 and S4. These work very naturally inside a pipe. If you do need mutable state, try to make it an internal implementation detail, rather than exposing it to the user.
- Tools that abstract over for-loops, like the `apply` family of functions or the map functions in `purrr`.

### Design for humans

Design your API primarily so that it is easy to use by humans. Computer efficiency is a secondary concern because the bottleneck in most data analysis is thinking time, not computing time.

- Invest time in naming your functions. Evocative function names make your API easier to use and remember.
- Favour explicit, lengthy names, over short, implicit, names. Save the shortest names for the most important operations.
- Think about how autocomplete can also make an API that’s easy to write. Make sure that function families are identified by a common prefix, not a common suffix. This makes autocomplete more helpful, as you can jog your memory with the prompts. For smaller packages, this may mean that every function has a common prefix (e.g. stringr, xml2, rvest).

## What did we learn today?

- How to read and write data using R
- Single table functions from {dplyr} to manipulate data frames
- A new operator, the pipe `|>` for combining function calls
- What tidy data is, and how to use the {tidyr} functions to get our data tidy
- *That R is awesome for manipulating data!*

## Attributions

- MUCH of these notes were derived from [Stat 545](https://stat545.com/) created by Jenny Bryan
- [R for Data Science](https://r4ds.had.co.nz/index.html) by Garrett Grolemund & Hadley Wickham

```R

```