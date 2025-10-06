---
title: Organization of Data Science projects and some useful tools
---

## Learning outcomes

1. Understand the basic syntax and functionality of regular expressions (regex) for pattern matching.
2. Explore the use of special characters, ranges, and anchors in regex to match specific patterns within text.
3. Apply regex to search, extract, and manipulate data in various formats using practical examples.
4. Use regular expressions to navigate and organize files within the filesystem.

:::{.activity}
## Activity 1

Imagine you are starting a new data science project. Based on what you know about project organization, how would you structure your project directory? Consider where you would place important files like raw data, processed data, code, reports, and environment settings.

Take a moment to think through these questions:
```
- What folders would you create, and why?
- Where would you place key files like your `.csv` data, analysis notebooks, and README?
- How would you structure the project to ensure it is easy to maintain and reproducible?
```
Jot down your ideas.

After reflecting on your answers, compare your project structure with your partner's. Identify similarities and differences between your approaches. What organizational choices did you both make? Were there any unique folder structures or file placements?
:::


## Project organization

Broadly speaking you will find two different ways of organizing a project: as a data analysis project and code project. Neither of this options is rigid enough to can give you here a exhaustive list of folder types and files you would expect on this projects. You will spend some time exploring different examples.
However, we can work with a general structure:


```bash
project/
├── data/              *.csv
│   ├── processed/
│   └── raw/
├── reports/           *.ipynb *.Rmd
├── src/               *.py *.R
├── doc/               *.md
├── README.md
└── environment.yaml (or renv.lock)
```

**data**
If your original data file is stored as part of the repository and should never be overridden.
That is why the original files are stored in a sub-folder called `raw`.

**reports**
The documentation is the biggest difference between data analysis projects and software projects.
Would you expect a report on a software project?
In general notebooks and other
[literate programming documents](./5-rstudio-projects-notebooks.html)
(that prioritize narrative and storytelling)
are the favorite options to share data analysis results [1].
You will create your first data science project in DSCI 522: Statistical Inference and Computation.
Software projects are mostly focused in the development of tools and in this cases other types of documentation
(docstrings, vignettes, readtheDocs, JupyterBooks)
are the favorite source of documentation for this kind of projects.
We will learn more about them in DSCI 524: Collaborative Software Development.

**src**
is where the source code for your project will be.
If you are not creating a package,
you may use the **analysis** directory.
In package development,
you will have a `src/project_name` directory in Python,
and just the `R` directory (instead of `src`) in R.
You will learn more about project hierarchy when making packages in DSCI 524.

**doc**
Documentation.
You can also use this directory for a GitHub pages enabled website.

**README.md**
The README file is probably the first file a potential user/contributor will read.

**environment**
Specifying the computational environment will help you to ensure the reproducibility of your analysis.

<!--
A place for everything, everything in its place.

Benjamin Franklin



---

![beer_messy_tidy](img/beer_messy_tidy.png)

---

![files_messy_tidy](img/files_messy_tidy.png)

## Face it...

- There are going to be files

- LOTS of files

- The files will change over time

- The files will have relationships to each other

- It'll probably get complicated

## Mighty weapon

- File organization and naming is a mighty weapon against chaos

- Make a file's name and location VERY INFORMATIVE about what it is, why it exists, how it relates to other things

- The more things are self-explanatory, the better

- READMEs are great, but don't document something if you could just make that thing self-documenting by definition


## Attribution:

- Slides borrowed from the [Data Carpentry](https://datacarpentry.org/) [Reproducible Science Workshop](https://datacarpentry.org/rr-organization1/)

-->

:::{.activity}
## Activity 2
Which set of file(name)s do you want at 3 a.m. before a deadline?
```
A. `Joe’s Filenames Use Spaces and Punctuation.xlsx`
B. `figure 1.png`
C. `JW7d^(2sl@deletethisandyourcareerisoverWx2*.txt`
D. `2014-06-08_abstract-for-sla.docx`
E. `myabstract.docx`
```
:::

## Filenames

In our first lecture, we learned about bash and talked about the downsides of making your
files and folder names "too pretty" with capitalizations and spaces.
Here we will discuss better ways to name our files and folders.

The below section comes from Jenny Bryan's
[Naming things](https://www2.stat.duke.edu/~rcs46/lectures_2015/01-markdown-git/slides/naming-slides/naming-slides.pdf)

- slides: <https://www2.stat.duke.edu/~rcs46/lectures_2015/01-markdown-git/slides/naming-slides/naming-slides.pdf>
- Source code: <https://github.com/jennybc/organization-and-naming>


### Names matter

![cheers](img/cheers.png)

### What works, what doesn't?

**NO**

~~~
myabstract.docx
Joe’s Filenames Use Spaces and Punctuation.xlsx
figure 1.png
fig 2.png
JW7d^(2sl@deletethisandyourcareerisoverWx2*.txt
~~~

**YES**

~~~
2014-06-08_abstract-for-sla.docx
joes-filenames-are-getting-better.xlsx
fig01_talk-scatterplot-length-vs-interest.png
fig02_talk-histogram-attendance.png
1986-01-28_raw-data-from-challenger-o-rings.txt
~~~

## Three principles for (file) names

1. Machine readable
2. Human readable
3. Plays well with default ordering

### Machine readable

![awesome_names](img/awesome_names.png)

- Regular expression and globbing friendly
    + Avoid spaces, punctuation, accented characters, case sensitivity

- Easy to compute on
    + Deliberate use of delimiters

#### Globbing

**Excerpt of complete file listing:**

![plasmid_names](img/plasmid_names.png)

<hr>

**Example of globbing to narrow file listing:**

![plasmid_names](img/plasmid_glob.png)


#### Same using Mac OS Finder search facilities

<center>
![plasmid_mac_os_search](img/plasmid_mac_os_search.png)
</center>

#### Same using regex in R

<center>
![plasmid_regex](img/plasmid_regex.png)
</center>

#### Punctuation {.smaller}

Deliberate use of `"-"` and `"_"` allows recovery of meta-data from the filenames:

- `"_"` underscore used to delimit units of meta-data I want later
- `"-"` hyphen used to delimit words so my eyes don't bleed

<center>
![plasmid_delimiters](img/plasmid_delimiters.png)
</center>

<hr>

<center>
![plasmid_delimiters_code](img/plasmid_delimiters_code.png)
</center>

This happens to be R but also possible in the shell, Python, etc.

#### Recap: machine readable

- Easy to search for files later
- Easy to narrow file lists based on names
- Easy to extract info from file names, e.g. by splitting

- New to regular expressions and globbing? be kind to yourself and avoid
    + Spaces in file names
    + Punctuation
    + Accented characters
    + Different files named `foo` and `Foo`

### Human readable

- Name contains info on content

- Connects to concept of a *slug* from semantic URLs

**Which set of file(name)s do you want at 3 a.m. before a deadline?**

![human_readable_not_options](img/human_readable_not_options.png)

#### Embrace the slug

<div class="columns-2">
slug filenames
![](img/slug_filenames.png)

slug
![](img/slug.jpg)
</div>

#### Recap: Human readable

Easy to figure out what the heck something is, based on its name

### Plays well with default ordering

- Put something numeric first
- Use the ISO 8601 standard for dates
- Left pad other numbers with zeros

**Chronological order:**

![chronological_order](img/chronological_order.png)

<hr>

**Logical order:** Put something numeric first

![logical_order](img/logical_order.png)

#### Dates

Use the ISO 8601 standard for dates: YYYY-MM-DD

![chronological_order](img//chronological_order.png)

<center>
![iso_psa](img/iso_8601.tiff)
</center>

#### Comprehensive map of all countries in the world that use the MM-DD-YYYY format

<center>
![map_mmddyyy](img/map_mmddyyy.tiff)
</center>

<br>

From https://twitter.com/donohoe/status/597876118688026624.

#### Left pad other numbers with zeros

<center>
![logical_order](img/logical_order.png)
</center>

<br>

If you don’t left pad, you get this:

~~~
10_final-figs-for-publication.R
1_data-cleaning.R
2_fit-model.R
~~~

which is just sad :(

#### Recap: Plays well with default ordering

- Put something numeric first
- Use the ISO 8601 standard for dates
- Left pad other numbers with zeros

### Recap

Three principles for (file) names

1. Machine readable
2. Human readable
3. Plays well with default ordering


Pros:
- Easy to implement NOW
- Payoffs accumulate as your skills evolve and projects get more complex

#### Go forth and use awesome file names :)

![chronological_order](img/chronological_order.png)

<br>

![logical_order](img/logical_order.png)

---

:::{.activity}
## Activity 3
Which of the following items describe three important rules for naming files on a computer?
Select all correct answers.

```
A. File name parts should be easy to extract programmatically
B. Humans would understand something about file contents by looking at their names
C. Files with appropriate names would be nicely organized by default
D. File names should always use lower-case letters
E. Human understanding of file names should be prioritized over machine understanding
```
<!--comment because iClicker only allows 5 choices
E. Underscores are the only allowed separator in file names
-->
:::

:::{.activity}
## Activity 4
Which of the following are GOOD date formats to use in a filename?
```
A. 2020-01-26T0233
B. 20200126T0233
C. 2020-01-26
```
:::

<!--

:::{.activity}
## Activity 4
Match each explanation with its corresponding file naming principle:

G.  By following this principle, we are able to use tools such as regular expressions and file-name globbing to search, select and manipulate files using tools such as the unix shell, R and Python.

H. By following this principle, we can have a good idea of what is inside files without having to open them (saves us time) and when we come back to work on a project we haven't worked on in a while it is much easier for us to remember what we were working on and get started again.


I. By following this principle, we can make our files more organized and understandable, and files more findable. Ordered lists will be sorted into a logical order and so it is easy to orient yourself in the directory and find the files you want to find.

J. By following this principle, we will be sure that our files have the appropriate formatting in terms of their content. Such files will be ready for subsequent analysis in a data science pipeline.
```
:::
-->

## References

[1] Kery, M. B., Radensky, M., Arya, M., John, B. E., & Myers, B. A. (2018, April). The story in the notebook: Exploratory data science using a literate programming tool. In Proceedings of the 2018 CHI Conference on Human Factors in Computing Systems (pp. 1-11).



# Introduction to Regular Expressions (RegEx)



:::{.activity}
## Activity 1

Imagine you're working as a data scientist for an e-commerce company,
and you have been given a dataset of user reviews for a product.
The reviews contain noisy data:
some reviews include email addresses, phone numbers, and URLs,
which you want to remove before conducting sentiment analysis.
Additionally,
you need to extract specific pieces of information,
such as any mentions of product model numbers that follow a specific pattern.

Would you use
<kbd>⌘</kbd> + <kbd>f</kbd> (Mac) or <kbd>Ctrl</kbd> + <kbd>f</kbd> (Windows / Linux)
to find and replace what you needed?
:::

## Introduction

Like with most things,
the best way for you to learn Regex is to get practice using it.
There are a few exercises included in the notebook,
and at the end I have also included links interactive online exercises
with are great to practice your regular expressions (RegEx)!

To see what a particular regex is matching and how,
you can use one of these two webpages,
which both do a great job visualizing and explaining the different parts of a regex match:

- <https://regexr.com/>
    - `regexr` interprets text input as one big string by default,
      so you need to check "multiline" under "flags" (top right)
      for it to behave as expected with beginning and end of line matches
      (it hints at this in the output for both ^ and $).
- <https://regex101.com/>
    - regex101 has the "multiline" flag set by default.

## Basic matching

Basic matching: if you look for a regular string, like `banana`,
regex will match the exact string (including its upper/lower case).
Both JupyterLab and VS Code have built in regex functionality
(bring up the search box and click the `.*` symbol to use regex
rather than the default search).
When learning regex it is helpful to use one of the two web tools
mentioned in the previous cell
in order to visualize how your regex is matching the text.
We will use a list of fruits to learn about regex.

```markdown
applesas
apple
apricot
banana
bilberry
blackberry
blackcurrant
blood orange
blueberry
canary melon
cantaloupe
cherry
clementine
cloudberry
coconut
cranberry
cucumber
currant
dragonfruit
durian
elderberry
gooseberry
grape
grapefruit
papaya
passionfruit
peach
orange
oranges unripe
persimmon
pineapple
pomegranate
pomelo
purple mangosteen
rock melon
salal berry
satsuma
star fruit
strawberry
watermelon
```

## The square brackets: `[]`

If you want to specify the set of possible characters
you can use square brackets `[]`;
For example, `[Aa]pple` would match `Apple` and `apple`.



:::{.exercise}
::::{.exercise-header}
### Exercise 1
::::
::::{.exercise-container}
Find all the pairs of vowels in the fruit list.

Highlight the black box below to see than correct answer
(the black box will not show up on GitHub,
so download the notebook unless you want the answer displayed)
Remember to use one of the websites linked above to help you understand
what your regex is matching
(<https://regexr.com/> or <https://regex101.com/>).

```
[aeiou][aeiou]
```
::::
:::

### Ranges within `[]`

You can also define ranges when using brackets. For example:

    - `[A-Z]`: will match any upper case letter
    - `[a-z]`: will match any lower case letter
    - `[0-9]`: will match any digit
    - `[0-5]`: will match any digit between 0 and 5

The order cannot be reversed, `[z-A]` does not work.
You can combine ranges: `[A-Za-z]`.

You can use square brackets starting with a caret. For example:

    - `[^A-Z]`: will match anything that is not an upper case letter
    - `[^0-9]`: will match anything that is not a digit

The caret needs to be inside the bracket,
if it is outside it will match the beginning of a line as described under the "Anchors" section below.

These ranges are ordered based on [ASCI codes](https://en.wikipedia.org/wiki/ASCII#Printable_characters)
where every character is represented by a number.
The first character in the list is ` ` (space) and the last is `~` (tilde).
The full list is shown below:

```
 !"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghijklmnopqrstuvwxyz{|}~
```

## Special matching characters

A common operation is to match any character (e.g. between two important characters).
Instead of writing out the full range `[ -~]` (space to tilde),
the special character `.` can be used to match any character in the list above.
`.` does not match the newline character,
So if you have an expression that continues on the next line it will not be matched.

To match a literal `.` (the period character),
you can "escape" its special meaning by prefacing it with a backslash `\.` (most common)
or surrounding it with square brackets `[.]`.

Another useful special character is `\w`,
which matches any character that normally occurs inside a word
(so it does not match spaces, underlines, etc)


:::{.exercise}
::::{.exercise-header}
### Exercise 2
::::
::::{.exercise-container}

What is the difference between writing `[A-Za-z]` and [`A-z`]?

```
[A-z] will also match the characters `[/]^_`, as you can see in the list above.
```
::::
:::


:::{.exercise}
::::{.exercise-header}
### Exercise 3
::::
::::{.exercise-container}

Match any characters between two `_`.

```
_.*_
```

::::
:::

## Anchors

The caret outside the brackets means beginning of line.
For example, `^apple` will match all lines that start with `apple`,
including `apple sauce` and `apples`.
The dollar sign `$` means end of line,
e.g., `fruit$` will match lines that end with `fruit`.
To remember this, you can use the mnemonic "Start with power (`^`) and end with money (`$`)" (originally from Jenny Bryan).

Another useful anchor is `\b`, which matches end of word.


:::{.exercise}
::::{.exercise-header}
### Exercise 4
::::
::::{.exercise-container}
Write a regex that will match a line that contains only pineapple.
(Hint: you cannot just write `pineapple` - it will not work - why?)

```
^pineapple$
```

note that if you use just `pineapple`,
lines that also contain other words would match too.
::::
:::

## Repetitions

To match multiple of the same character,
you can either repeat it or use the following syntax:

- `{n}`: exactly `n` occurrences
- `{n,}`: at least `n` occurrences
- `{0,m}`: at most `m` occurrences
- `{n,m}`: between `n` and `m` (inclusive) occurrences

### Special repetition characters

There are some shortcuts for the most common repetitions:

- `?`: means 0 or 1 time (`{0,1}`)
- `*`: means 0 or more time (`{0,}`)
- `+`: means 1 or more time (`{1,}`)

 For example, `apples?` will match `apple` and `apples`.
But `apples+` will not match `apple` or `appplesq`,
but it will match `apples`, `appless`, `applesss`, etc.

:::{.exercise}
::::{.exercise-header}
### Exercise 5
::::
::::{.exercise-container}
Find the fruits with names between 10 and 12 characters.

```
.{10,12}
```
::::
:::

:::{.exercise}
::::{.exercise-header}
### Exercise 6
::::
::::{.exercise-container}
Find the lines with no more than 4 letters.

```
^.{0,4}$
```
::::
:::

:::{.exercise}
::::{.exercise-header}
### Exercise 7
::::
::::{.exercise-container}
Find all the words that contain at least two consecutive vowels.


```
[aeiou]{2,}
```

```
[aeiou][aeiou]+
```

::::
:::

:::{.exercise}
::::{.exercise-header}
### Exercise 8
::::
::::{.exercise-container}

This is a bit harder and derives from all previous sections: Match entire words that end in `_`.

```
\\w*_\\b
```

::::
:::

:::{.exercise}
::::{.exercise-header}
### Additional exercises
::::
::::{.exercise-container}


Go through the interactive tutorials and practice sessions at <https://regexone.com/>
that correspond to the topics we have covered during class.

The Library Carpentry organization has many regex exercises
in all sections of their regex course here:
<https://librarycarpentry.org/lc-data-intro/>
(you can just to do the exercises).

::::
:::


## References

[1] Kery, M. B., Radensky, M., Arya, M., John, B. E., & Myers, B. A. (2018, April).
The story in the notebook: Exploratory data science using a literate programming tool.
In Proceedings of the 2018 CHI Conference on Human Factors in Computing Systems (pp. 1-11).

---
title: Asking Effective Questions
---

## Why even bother?

Asking questions effectively means that the person helping you
will be able to answer your question better and quicker.
Being able to answer a question quicker means more time to help others,
including your future self.
When questions require clarification,
fewer people will be helped overall.
Sometimes this in unavoidable because the question is complex,
but all too often it if because the person trying to help
is not able to reproduce problem,
or the question is unclear.

When you are asking for help online,
e.g. on StackOverflow or on GitHub,
remember that you are often receiving help from people
who are volunteering their time.
So please make it as easy as possible for them to help you.

You might be frustrated by a problem
to the point where you just want to ask something like.

> WHY IS THIS \*\*\*\* CODE NOT WORKING??????

Don't do this.
No one will help you.
You will get more frustrated.

When I feel like this,
I find it really helpful and calming to sit down
and type out a proper question.
You can start banging out words in the beginning,
but as you slowly adhere to the format of asking properly,
it will become like a meditative practice
which also calms you down.

In addition to your mental wellbeing,
writing down questions properly has another superb quality:
they help you solve your own problems.
The act of formulating a question in either speech or text
helps you uncover what you missed while the problem was a mere thought.
This is so common that it has a name:
"Rubber duck debugging"
allegedly from a software developer who put a rubber duck on their desk
and whenever they had a problem they couldn't solve,
they starting talking to the toy duck,
and often came upon the resolution during while describing the problem.

## How to ask effectively

In essence,
you want to make your question as easy to understand as possible
and your specific problem as easy to reproduce as possible.
If you just include a screenshot and title your question "Help",
the person helping you has to spend time trying to figure out
what you actually want help with instead of helping,
Instead include a succinct, clear
description of your problem
and the minimal code needed to reproduce it.
If you are asking your question on stack overflow
you can use tags to categorize it,
and these can then be used to search for an answer via the syntax `[tag-name]`.
This can be especially useful for R,
since the name is just one letter
and it can be hard to find relevant matches otherwise.

## Minimal reproducible example

There is a Swedish expression:
"beloved child has many names".
No,
it does not translate very well to English,
but the message is that someone or something that many people like,
will be referred to differently by different people.
This is true for minimal reproducible examples,
which you might find referred to by any of the following:

- MRE Minimal Reproducible Example
- MCVE Minimal Complete Verifiable Example
- MWE Minimal Working Example
- reprex REPRoducible EXample

There have been great articles written on what goes into an MRE,
and here are some of them that I recommend that you check out:

- <https://stackoverflow.com/help/how-to-ask>
- <https://stackoverflow.com/help/minimal-reproducible-example>
- <https://community.rstudio.com/t/faq-whats-a-reproducible-example-reprex-and-how-do-i-do-one/5219>
- <https://reprex.tidyverse.org/> (an R package to help creating MREs from code)

In summary,
asking effectively
and creating an MRE includes the following tasks:

0. Search for other questions similar to yours.
1. Describe the issue clearly in the title and elaborate briefly in the text body.
2. Reduce the code to the minimum required to recreate your error, and paste it as text.
    - If your code includes functions or classes, include their definitions.
    - Create small toy dataset instead of using real data.
    - Use markdown code blocks for proper indentation and syntax highlighting.
3. Describe what you have tried so far,
   what you don't understand or what went wrong,
   including any error messages and their full traceback.

The points are elaborated on below:

0. Search for other questions similar to yours.
   Many questions already have an answer,
   and finding it is faster both for you and for others.
   If the answer to an existing question is not good enough,
   improve it by adding the missing info!
1. Write the tile as a summary of your issue.
   Think about what you would want the title to say
   if you were searching the issue list for help.
   Just "Error" or "Question" is not helpful,
   but "How to list content in a folder?" is.
2. Introduce the problem by briefly describing what you want to do.
3. Show what you have tried,
   explain what you expected to happen,
   and what went wrong.
   It is often critical that the person helping you can reproduce the problem,
   so include both the code or command you tried to run and the error message.
      - For coding questions,
        text is preferred over a screenshot since it is easy to copy and paste,
        which facilitates reproducing your problem.
      - Inline code should be surrounded by single backticks for clarity.
        Longer blocks of code with multiple lines should be surrounded by triple backticks.
4. Include versions of any packages you are using,
   and the operating system if relevant,
   e.g. Win10, Python 3.8, pandas 1.0.2.
   On R you can use `devtools::session_info()` to see this information
   (after `install.packages("devtools")`).
   and on Python you can use `sinfo()`
   (after importing: `from sinfo import sinfo`,
   needs to be installed via `pip install sinfo`).
5. When your problem is solved,
   acknowledge the solution,
   close the issue/ticket/question.
   If you found the solution yourself,
   post it in a comment before closing,
   so that others can find it.