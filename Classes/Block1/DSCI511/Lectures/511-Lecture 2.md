
---

[[511-Worksheet 2]]
[[511-Lab 1]]

## Lecture 2: Reshaping data

## Lecture learning objectives

- Filter columns in a DataFrame using logical operators or the `.query()` method
- Remove or fill missing values in a dataframe with `.dropna()` and `.fillna()`.
- Rename columns of a dataframe using the `.rename()` function or by accessing the `.columns` attribute.
- Use `.melt()`, `.pivot()` and `.pivot_table()` to reshape dataframes, specifically to make tidy dataframes.

```{python}
#| slideshow: {slide_type: skip}
import pandas as pd
```

## Data types

- In lecture 1 you learned about the following types: DataFrame (table), Series (column), integer (whole number), and string (text)
- A new type for this lecture is 'boolean'. There are only two boolean values: True or False (note the first letter is capitalized in Python code).
- Booleans are useful when you want to search/filter through a DataFrame, i.e. find all the data where some particular condition is True. Load the file `data/YVR_weather_data.csv` into pandas to experiment with some filters.

Each row is a day, and each column is a weather measurement. Let's find all the days when the mean temperature is greater than 15. We'll start by creating a Series of boolean values like this: 

```{python}
weather = pd.read_csv('data/YVR_weather_data.csv')
weather.head()
```

```{python}
condition = weather['Mean Temp (°C)'] > 15
condition
```

The Series has True if that row has a value of greater than 15 in the mean temperature column, and False otherwise. Take a moment to compare the output of the code with the actual CSV file, to convince yourself this is what's happening.

You can then 'apply' this to your DataFrame using the  `[ ]` notation you learned in Lecture 1.

```{python}
weather[condition]
```

The following comparisons are supported:
- `>` greater than
- `<` less than
- `>=` greater than or equal to
- `<=` less than or equal to
- `==` exactly equal to
- `!=` not equal to

```{python}
condition = weather['Mean Temp (°C)'] == 19.0
exactly_19_degrees = weather[condition]
exactly_19_degrees[['Mean Temp (°C)']]
```

```{python}
condition = weather['Month'] != 1
not_january = weather[condition]
not_january[['Month']]
```

In practice, most people do not write out the condition on its own line. Code is usually combined into one line like this:

```{python}
not_january = weather[weather['Month'] != 1]
```

Python ignores extra blank spaces inside brackets, and some people prefer to spread their code over multiple lines for increased readability. 

```{python}
not_january = weather[
                    weather['Month'] != 1
                    ]
```

You can combine multiple conditions using the symbols `&`, `|`, and `^`. Note that each condition must be enclosed in parentheses.

```{python}
#Use & to match BOTH conditions
#find every day in May when it snowed
condition = (weather['Month'] == 5) & (weather['Total Snow (cm)'] > 0)
weather[condition]
```

```{python}
#Use | to match AT LEAST ONE condition
#find every day when it rained or snowed or both
condition = (weather['Total Snow (cm)'] > 0) | (weather['Total Rain (mm)'] > 0)
weather[condition]
```

```{python}
#Use ^ to match EXACTLY ONE condition
#find every day when it was either very windy or very cold, but not both
condition = (weather['Spd of Max Gust (km/h)'] > 75) ^ (weather['Mean Temp (°C)'] <= -10)
weather[condition]
```

As an alternative, you can use the `.query()` function on your DataFrame. In this case, you write the condition out as a string, enclosed in double-quotes.

```{python}
#Find days with cool temperatures and with trace amounts of snowfall in either January or February
condition = "`Total Snow Flag` == 'T' &  `Mean Temp (°C)` <= 5 & Month == 1 | Month == 2" 
snowy_days = weather.query(condition)
snowy_days.tail(5)
```

Notes on using `.query()`
- You don't have to mention your DataFrame specifically inside the condition, you only need the column names
- If a column name has spaces in it (like Total Snow Flag), you must enclose it in backticks (*not* quotes!)
- If you are trying to match against a string (like 'T') you must enclose it in single-quotes
- Parentheses are not required around each condition, though you can choose to add them if you need to group certain conditions together.

### Practice
- Filter for every day in 1937.
- Filter for months from August to December (inclusive)
- Find days when it snowed less than 10cm
- Find all the days when it was at least 20 degrees and it didn't rain
- Filter for days when the mean temperature is between 5 and 15 degrees (exclusive)
- Find days when it was either windy (max gust of at least 75) or rainy (more than 50mm) but not both
- Filter for anything that's not in 2012

```{python}
#PRACTICE CELL
```

## Changing values based on conditions

You can combine boolean conditionals with `.loc[]` from Lecture 1 to update your DataFrame. Let's open up the language database from the previous lecture and make a change to it. Some languages have an unknown genetic grouping ('family'), which is indicated by a '?' in the data. Let's find all instances of '?' and replace it with the string 'Isolate', which is the more common technical term for such languages.

```{python}
languages = pd.read_csv('data/WACL.csv')
languages.tail(5) #Check the 'family' column of the Zuni language (row 3336)
```

```{python}
column = 'family'
condition = languages[column] == '?'
languages.loc[condition, column] = 'Isolate'
languages.tail(5) #Now check the 'family' column for Zuni
```

This can be condensed to a single line:

```{python}
languages.loc[languages['family'] == '?', 'family'] = 'Isolate'
```

## Empty cells/missing values

Sometimes a cell in a spreadsheet will be missing values. In the language data, some rows have an entry for source material, but others do not. Pandas represents empty cells with the special value `NaN`, meaning "not a number". You can check if your dataset has missing values like this:

```{python}
#Check the entire DataFrame for Nan
languages.isna() 
```

```{python}
#Check specific columns
languages['source'].isna()
```

```{python}
#Use Python's any() function to find out if there is at least one NaN value in the DataFrame
any(languages.isna())
```

```{python}
#Or check just a specific column
any(languages['family'].isna())
```

If you want to get rid of  any rows with `NaN` use `.dropna()`

```{python}
languages.dropna() #This returns a copy, it doesn't modify your original!
```

You can replace all occurences of `NaN` with a different value using `.fillna()`

```{python}
languages.fillna('No source') #This returns a copy, it doesn't modify the original!
```

### Renaming columns

To rename all the columns at once, you can directly set the `columns` attribute of your DataFrame. You'll need to use a data type called a *list* which contains all of your column names between square brackets. 

```{python}
scores = pd.read_csv('data/student_scores.csv', index_col='Student_ID')
scores.columns.to_list() #check the original columns
```

```{python}
#Create a list of new names
scores.columns = ['BIOL-101', 'CHEM-200', 'PHYS-132', 'ENGL-100', 'DRAM-301', 'ART-216'] #this is a list type variable
scores
```

You can choose to rename only some columns with the `rename()` function. To illustrate this, let's rename some columns from the languges dataset.

```{python}
languages = pd.read_csv('data/WACL.csv')
languages.columns.to_list() #remind yourself what the current headers are
```

For this function, you need to create a *dictionary* which is another basic type of object in Python. This type is used when you want to map, associate, or pair together two peices of information. Dictionaries are surrounded by curly braces, and each pair in the dictionary is separated by a colon. For example:

```{python}
old_to_new = {"language_name": "name", 
               "area": "region", 
               "source": "bibliography"}
```

The first item in each pair is called a 'key', and second item is called a 'value'. Here the keys are `language_name`, `area`, and `source`, and the corresponding values are `name`, `region`, and `bibliography`.

Once you have the dictionary, you pass it to the `columns` argument of the `.rename()` function on your DataFrame.

```{python}
languages = pd.read_csv('data/WACL.csv')
languages = languages.rename(columns = old_to_new) #this returns a copy!
languages.head()
```

We will discuss how to use dictionaries in more detail in a future lecture. For now, you'll just need to memorize this syntax for renaming columns.

### Data type cheatsheet

| Type name | Data format | Example |
| --- | --- | --- |
| string | text | `words = 'this is a string'` |
| integer | whole number | `fifteen = 15` |
| float | decimal number | `two_and_a_half = 2.5` |
| list | sequence | `groceries = ['apples', 'pears', 'oranges']` |
| dictionary | mapping | `english_to_french = {'house': 'maison', 'chair': 'chaise'}` |
| boolean | true/false | `not_false = True` | 
| DataFrame | table | `results = pd.read_csv('experiment.csv')` |
| Series | column | `scores = results['scores']` |

## DataFrame reshaping

### Tidy data

- [Tidy data](https://vita.had.co.nz/papers/tidy-data.pdf) is about "linking the structure of a dataset with its semantics (its meaning)"

- It is defined by:

    1. Each variable forms a column

    2. Each observation forms a row

    3. Each type of observational unit forms a table
    
- Often you'll need to reshape a dataframe to make it tidy (or for some other purpose)

- We'll look at four different ways to do this: `.transpose()`, `.pivot()`, `.pivot_table()`, and `.melt()`
    


Source: [r4ds](https://r4ds.had.co.nz/tidy-data.html#fig:tidy-structure)

### Wide vs. long tables


[Image source](https://www.statology.org/wp-content/uploads/2021/12/wideLong1-1.png)

### Transpose
Transposing a table means swapping the rows and columns. DataFrames have a `.transpose()` function for this. Run the next two cells and compare the output.

```{python}
scores = pd.read_csv('data/student_scores.csv')
scores.head(5)
```

```{python}
scores_T = scores.transpose() #This returns a copy!
scores_T.head(5)
```

### Pivot

Pivoting a table means changing it from long to wide format. To illustrate, we will explore a dataset of the results from the 2024 Olympics. 

```{python}
olympics = pd.read_csv('data/Olympics_2024.csv')
olympics
```

Let's reshape the data so each row is a country ("NOC"), each column is a sport ("Competitions"), and the cell values show the total medal count for that country in that sport.

To do this, we need to use `.pivot()`, which take three arguments:
- `index` is the name of an existing column that should serve as your new row index (NOC)
- `columns` is the name of an existing column whose values will be converted to new columns headers (Competitions)
- `values` is the name of an existing column whose values will be used to fill the new columns (Total)

The table has a lot of `NaN` values (because not all countries win medals in all sports). We'll replace those with zeroes for readability.

```{python}
olympics.pivot(index='NOC', columns='Competitions', values='Total').fillna(0).head() 
```

Pandas has to do some calculations when reshaping the table, which results in decimal numbers ("floats"). This is a little strange, since you can't win a partial medal. We can ask Pandas to change the type of the entire DataFrame with the `.astype()` function. 

```{python}
olympics.pivot(index='NOC', columns='Competitions', values='Total').fillna(0).astype(int)
```

```{python}
#This format makes it easy to extract information about individual countries
o = olympics.pivot(index='NOC', columns='Competitions', values='Total').fillna(0).astype(int)
o.loc['Canada']
```

Now let's pivot the other direction, so that each row is a sport, and each column is a country

```{python}
olympics.pivot(index='Competitions', columns='NOC', values='Total').fillna(0).astype(int).head()
```

```{python}
#This format makes it easy to extract data about individual sports
o = olympics.pivot(index='Competitions', columns='NOC', values='Total').fillna(0).astype(int)
o.loc['Fencing']
```

### Practice
Take a few moment to play with `.pivot()` and change the parameters around

```{python}
#PRACTICE CELL
```

### `.pivot_table()`
You can only pivot a table if it has unique combinations of row and column values. If you have duplicates, then you'll need to use the `pivot_table()` function instead. To illustrate this, load the accent study dataset:

```{python}
results = pd.read_csv('data/accentStudyDataset.csv')
results.head()
```

These are partial results from an experiment studying the effects of accent on listening comprehension. Participants with different langauge backgrounds listened to passages recorded in English, spoken by people with different accents (British, Chinese, or Spanish). Participants then completed a short test, answering questions about the passage they had listened to.

It would be useful to reshape this so that rows are organized by first language, columns by accent condition, with cell values showing the quiz scores. That would help us see interactions, e.g. how did first-language Turkish speakers do when listening to British accents?

However, if we try to do this with `.pivot()`, we get an error. 

```{python}
results.pivot(index='FirstLanguage', columns='Accent', values='TestPct')
```

The error message says there are duplicate entries, so the table can't be reshaped. This means that there are some row/column combinations that would appear twice in the new pivoted table. For example, there's more than one person with a first language of Turkish in the Chinese accent condition. 

In this case, we need to use `pivot_table()`, which is similar to `.pivot()` but it can handle the duplicates.

```{python}
results.pivot_table(index='FirstLanguage', columns='Accent', values='TestPct') #index, columns, and values are the same as with .pivot()
```

To find the results for a particular first language, use `.loc[]`

```{python}
r = results.pivot_table(index='FirstLanguage', columns='Accent', values='TestPct')
r.loc['Turkish']
```

By default, `.pivot_table()` combines ("aggregates") all the duplicate values and calculates the average. The cell above shows that the average test scores for first-language Turkish speakers for passages read with British accent is 56.25% You can do other types of calculations as well, but we'll leave that for a later lesson on grouping and aggregation.

Each accent condition in this experiment contained 4 different test passages, and the code above aggregates over all of them. To see the finer details of each test within in each accent, simply provide both as column names:

```{python}
results.pivot_table(index='FirstLanguage', columns=['Accent', 'Test'], values='TestPct').head()
```

Note that this creates a large number of `NaN` values because there are some combinations of language/accent/test that did not occur in the original data (e.g. no Burmese speakers heard the CompSci passage with a Britsh accent)

### Melt
`.melt()` works as the opposite of `.pivot()`, and converts data from a wide format to a long format. To illustrate this, we'll return to the Internet Movie Database.

```{python}
imdb = pd.read_csv('data/imdb.csv')
imdb.head(2) #remind yourself what this looks like
```

Let's reshape the table so that rows are organized by director, and there are only two columns: 'Actor' and 'Billing_Order'.

To do this we'll use `.melt()`, which requires up to four arguments:

- `id_vars` is a list of column names that will act as the index for creating new rows (Director)
- `value_vars` is a list of columns from your original data that you want melted (Star1, Star2, Star3, Star4)
- `var_name` is the header for a new column, which will contain the column names listed in `value_vars` (Billing_Order)
- `value_name` is the header for a new column which will contain the row values from the columns listed in `value_vars` (Actor)

`value_vars` is optional. If you omit it, Pandas will assume you want to melt everything that's not listed in `id_vars`

```{python}
imdb.melt(id_vars=['Director'], value_vars=['Star1', 'Star2', 'Star3', 'Star4'], var_name='Billing_Order', value_name='Actor')
```

```{python}
#We can use this new format to quickly look up which actors have appears with which directors
i = imdb.melt(id_vars=['Director'], value_vars=['Star1', 'Star2', 'Star3', 'Star4'], var_name='Billing_Order', value_name='Actor').set_index('Director')
i = i[['Actor', 'Billing_Order']] #I prefer this order
i.loc['Francis Ford Coppola']
```

### Reseting the index
Reshaping a table can change the way rows are indexed. Sometimes, you may want to reset the index to use the default ordinal numbers. For example, let's load a dataset about the Titanic, and use `.pivot_table()` to compare survival rates for men and women places in different passenger classes.

```{python}
titanic = pd.read_csv('data/titanic.csv')
titanic.head() #get quick look
```

```{python}
survival_rates = titanic.pivot_table(index='Gender', values='Survived', columns='Class')
survival_rates
```

This sets the `Gender` column as the row index, so it is no longer part of the table data. To restore it as a column, use the `.reset_index()` function:

```{python}
survival_rates.reset_index() #This returns a copy!
```


