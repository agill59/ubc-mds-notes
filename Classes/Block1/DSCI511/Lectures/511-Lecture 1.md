---
jupyter: python3
---
[[511-Worksheet 1]]
[[511-Lab 1]]
# Lecture 1: Introduction to Pandas

## Lecture learning objectives

- Create a DataFrame from a text file using Pandas `pd.read_csv()` and `pd.read_excel()`
- Examine a DataFrame with `.head()`, `.tail()`, `.describe()` and `.shape`
- Access values from a DataFrame using `[]`, `.loc[]`, and `.iloc[]`
- Apply mathematical functions to columns in a DataFrame
- Create new columns in a DataFrame by performing operations on existing columns
- Remove columns from a DataFrame 
- Sort a DataFrame using `.sort_values()`
- Write the contents of a DataFrame to file using `.to_csv()`

## Introduction to Pandas


- The most popular Python library for tabular data structures
- You can think of Pandas as an extremely powerful version of Excel (but free and with a lot more features!) 
- The only tool you'll need for many (most?) data wrangling tasks


To install Pandas, run this code. You only need to do this once. Ideally this is done in the terminal, not a notebook.

```
$ conda install pandas
```

To use Pandas in your code, you must *import* it. It's common to import and rename Pandas to simply "pd".

```{python}
import pandas as pd
```

When you run this code, Python does a lot of stuff behind the scenes, and we won't discuss details here. What's important is that you can now type `pd.` followed by the name of something from the Pandas library to use it. Pandas is a very large library, with too many functions to discuss in this class. We will focus on just a few parts of Pandas in this course, that we think are going to be most relevant/useful in a data science career. The official documentation for Pandas explains everything that you can do with it, and you should bookmark it now: https://pandas.pydata.org/docs/index.html

# Opening a file

The first thing we'll learn to do with Pandas is open a file. We'll start with data from the Internet Movie Database (IMDB). We use the `.read_csv()` function to open files. Run the next cell to open the file `imdb.csv`, and save it as a variable called `imdb`.

```{python}
#| scrolled: true
imdb = pd.read_csv('data/imdb.csv')
```

CSV stands for 'comma-separated values, and it represents a table in a text file. Each row in the file is a row in a table, and columns in each row are separated by commas. Pandas can read from a variety of different file types, but CSV is one of the most common formats in data science and machine learning. Try opening this file in text editor to see what it looks like.

Notice that the name of the file is written bewteen quotation marks. This is how we tell the difference between code and text in Python (and virtually all other programming languages). In technical terms, text is referred to as a *string*, which is one of the fundamental types of information used in programming.

Pandas transforms the information from the file into a type of object called a `DataFrame`, which you can think of like an Excel spreadsheet. DataFrames are your best friend in this course, and you will use them for practically everything. 

The concepts of *objects* and *types* are fundamental to programming, and each type has different uses. During this course you'll deal with a variety of types representing different formats (numbers, text, sequences, tables, mappings, etc.). We will discuss types as they come up. If you have a background in programming, you can read about Python's built-in types here: https://docs.python.org/3/reference/index.html

The DataFrame type has many useful *methods* or *functions*. You can think of a function as one or more lines of code which perform some computation(s), and then return a result to you. Most functions allow you to supply *arguments*, which are like options you can set, to modify how the function operates. DataFrames have a very large number of methods and we will only cover some of the more commonly used ones in this class. There is a full list available in the [official documentation](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.html).

The next few cells have code for quickly inspecting your data:

```{python}
imdb.head(8) #Show the first 8 rows, 8 is an argument
```

```{python}
imdb.tail(5) #Show the last 5 rows, 5 is an argument
```

```{python}
imdb.describe() #Get basic stats, only works with numeric columns, there are no arguments
```

Notice that the numbers above do not have quotation marks. This indicates that they should be treated as actual numbers, and not as strings of text. In Python, the number `8` and the string `'8'` are not equivalent. Whole numbers are another fundamental type in programming, technically referred to as *integers* or just *ints*. Decimals numbers e.g. (`10.4`) are treated as a different type, called a *float*.

In addition to functions, DataFrames also have *attributes*, which are just values that you can look up, and which don't require any special computation. For example, `.shape` tells you the number of rows and columns. Attributes are not written with parenthesis.

```{python}
imdb.shape #No brackets!
```

In addition to comma separated files, it's also common to store tables as tab separated files (TSV). The extra space makes this format a little easier for people to read. Sometimes it is necessary to use tabs because you are storing data that contains commas, such as sentences of natural language text. In this case, you don't want the commas treated as column separators. 

The `.read_csv()` method, despite its name, can also be used to read TSV files by adding a `sep` argument. The next cells opens a file with information about villagers in the video game Stardew Valley. Each villager has likes and dislikes, written out as a comma-separated list, making the CSV format impractical. 

```{python}
stardew = pd.read_csv('data/likes_dislikes.tsv', sep='\t') #Note the comma between arguments!
stardew
```

The sequence `\t` is a special symbol representing a tab. In fact, the `sep` argument can take any value, but it's rare to find files that use other separators.

What happens if you forget to add this argument, and ask Pandas to open a TSV file? Let's find out...

```{python}
oops = pd.read_csv('data/likes_dislikes.tsv')
oops
```

This raises an error, sometimes called an exception. The error message contains a 'traceback', which describes where the code stopped working, and sometimes why. The traceback can look overwhelming at first, especially when it is very long. Normally, you start by scrolling to the very bottom of the traceback, where the specific type of error will be mentioned and (sometimes) there is also a message explaining the error. There are many types of errors, just like there are many types of data structures, and you will learn to recognize them through practice.

Don't worry if your code raises errors. It's completely normal part of learning, and there's no harm. Try your best to interpret the error message, and re-examine your code for potential mistakes. If you don't know what the error means, ask your instructor.

### Practice
Open the file `data/villains.txt` in a text editor. Determine what the delimiter is, then write code below to open the file with pandas.

```{python}
#PRACTICE CELL
```

The first line in a csv/tsv file is typically a "header", which is a list of column names/labels. It doesn't contain data. Pandas knows this, and by default assumes your file contains a header, as seen in all previous examples. When the DataFrame is created, header names are stored in an atrribute called `columns`:

```{python}
stardew.columns
```

For a slightly more readable output, you can add `.to_list()`

```{python}
stardew.columns.to_list()
```

Sometimes files lack headers, and every line consists of data. This can occurs when the data is intended to be fed directly into a computer program, where the code is written to locate information by its ordinal position (first column, second column, etc.) and it doesn't need to know anything about the names of the columns. 

In such a situation, you don't want Pandas to treat the first line as column names. To ensure these files are read correctly, use the argument `headers=None` Note that `None` doesn't take quotes, and starts with a capital letter. This is a special type in Python that indicates an absence of any value. You don't need the details of how this works, but you should be aware of `None` as it comes up from time to time.

The `measurements.tsv` file contains some (randomly genreated) measurement data without headers, as an example.

```{python}
measurements = pd.read_csv('data/measurements.tsv', sep='\t', header=None)
measurements.head()
```

Notice that Pandas added column headers for you in this case, although they are simply numbers. 

**Important**: *Python starts counting from 0!* The "first" column in the table has the label 0. This is true of most, but not all, programming languages. The R language is a noteable exception, which starts from 1, and if you're also taking the R class this may trip you up at first. 

If you want to add column names, you can do this by adding an argument called `names` like this:

```{python}
measurements = pd.read_csv('data/measurements.tsv', sep='\t', header=None, names=['Experiment ID', 'Temperature', 'Speed', 'Direction'])
measurements.head()
```

Note how the value for `names` is written inside square brackets. This is another basic type in Python called a *list*. In this example, the list contains a set of four strings because that's most appropriate for column labels, but you can store information of any type inside of a list. 

Headers are generally contained in the first line of a file, but in some rare cases they might appear on a different line. In this case, instead of using `header=None` you can supply a number that corresponds to the header line. For example, in the file `data/sales.csv` the first two lines includes some general statistics followed by a blank line, then the headers appear on the 4th line. Since Python starts counting at zero, that should be line 3:

```{python}
sales = pd.read_csv('data/sales.csv', header=3)
sales
```

Except that doesn't look right! Those aren't normal column headers. It turns out that Pandas ignores blank lines in the file and so they don't count. We actually have to set the header to 2

```{python}
sales = pd.read_csv('data/sales.csv', header=2)
sales
```

### Opening files on the internet

The `read_csv()` function can be used for reading csv/tsv files stored on the internet. This works exactly the same way as loading a file from your computer, except you specify the web address. A very famous machine-learning dataset called the "Iris Dataset" is available online as a CSV file for example. Note this file lacks headers.

```{python}
iris = pd.read_csv('https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data', header=None)
iris
```

In addition to plain text csv/tsv files, Pandas also supports opening spreadsheets in the Microsoft Excel. Spreadsheets are stored in a different format beacuse they also have to record information about font types and sizes, cell colours and borders, formulas, etc. In addition, a single spreadsheet file can save multiple 'sheets' of data at once.

To open spreadsheets, use the `.read_excel()` function. This method has the same `header` and `names` arguments you learned for `.read_csv()`, but it doesn't need a `sep` argument (that informatoin is stored as part of the spreadsheet, so Pandas can look it up for you). 

Some spreadsheet file contain multiple sheet, but you can only load one at a time. andas assumes you want the first one. If you need to specify another one, you can add a `sheet_name` argument. You can pass an integer to this argument (e.g. `sheet_name=4` loads the 3rd sheet) or you can pass a string representing the name of the sheet (e.g. `sheet_name='Quarterly Earnings'`)

#### Practice
The file `data/World_Development_Indicators.xlsx` contains some randomly selected data from the World Bank's World Development Indicators (https://databank.worldbank.org/source/world-development-indicators#). It is saved as a spreadsheet with two sheets. The first sheet is metadata, the second sheet contains the actual Indicator data. Try writing some code below to open this second sheet with Pandas.

Note: You may get an `ImportError` when you use `pd.read_excel()`, depending on how your Python is set up. If this happens, uncomment the following code and run it to install another package (just like you installed Pandas earlier). Delete the cell, since you don't need to install the package twice, then re-run your `.open_excel()` code.

```{python}
#Run this if necessary
#!pip install openpyxl
```

```{python}
#PRACTICE CELL

```


Lastly, it's worth mentioning that Pandas has over a dozen different `.read_something()` methods, for various files types, such as `pd.read_html()`, `pd.read_json()`, and `pd.read_sql()`. You can consult the official documentation for more details on these: https://pandas.pydata.org/docs/search.html?q=read_

# Locating data inside DataFrames

- There are several ways to select data from a DataFrame:
    1. `[ ]` and `[[ ]]`
    2. `.iloc[]`
    3. `.loc[]`

#### Indexing with [ ] and [[ ]]

You can access a single column by placing the name in quotes between square brackets.

```{python}
imdb['Series_Title']
```

To get data from multiple columns, list all column names inside double square brackets.

```{python}
imdb[['Series_Title', 'Genre']]
```

**Note:** The double-bracket method creates a new DataFrame, containing only the columns you specified. The single-bracket method actually creates a different type of object called a *Series*, which represents a single column, not a table. A DataFrame is actually made up of multiple Series objects. You can think of their relationship like this:


Once you have a column (Series), `.value_counts()` is a useful way to quickly inspect the data.

```{python}
directors = imdb['Director']
directors.value_counts()
```

A related function is `.nunique()` which counts the number of unique values in your data:

```{python}
directors.nunique()
```

This function actually returns 

### Practice
- Create a DataFrame with columns for the 4 starring actors
- Create a Series with just the names of the directors
- Create a DataFrame that only contains the run time of the movies
- Find all the different genres with `.value_counts()`

```{python}
#PRACTICE CELL
```

## Indexing with `.iloc`

First we'll try out `.iloc[]` which accepts *integers* as references to rows/columns. Integers are another type of Python object, which represent whole numbers.

The code in the following block returns the first row of the IMDB table:

(**Remember:** *Python starts counting from 0, not 1!*)

```{python}
imdb.iloc[0]
```

Just like before, using a single bracket will get you a Series (a column of data) and using double brackets gets you a DataFrame (a table of data).

```{python}
imdb.iloc[[0]]
```

To count 'backwards' you can use negative numbers, starting from -1

```{python}
imdb.iloc[[-1]]  # Returns a DataFrame with only the last row of the table
```

If you want multiple rows, you can list them all in double brackets

```{python}
imdb.iloc[[0, 5, 99]] 
```

You can select both a row and column like this:

```{python}
imdb.iloc[6, 0] #7th row, 1st column
```

To select multiple rows and columns, you can list them all individually in square brackets:

```{python}
imdb.iloc[ [3,5,10], [1,2] ] #4th, 6th, and 10th rows, 2nd and 3rd columns
```

Or you can use "slice" notation to select range of rows or columns. Slices are written as `x:y` which reads as "starting from position x, going up to *but not including* position y". You can omit the first number, and Python will assume you want to start on the first position (position 0). You can omit the last number and Python will assume you want up to and including the last row.  

```{python}
imdb.iloc[0:5] #get the first five rows, i.e. from row 0 up to but not including row 5
```

```{python}
imdb.iloc[:10, 4:] #get the first 10 row, and from the 5th column onward
```

```{python}
imdb.iloc[:-200, :] #What does this do?
```

### Practice
Use `.iloc` to do the following:
- Get the 7th, 8th, and 27th rows of the table. Does the order of the integers have to match the order of the table?
- Get the first row and second-to-last column
- Get the IMDB Rating column for row 95
- Get rows 20 through 25 and the second, third, and seventh columns

```{python}
#PRACTICE CELL
```

## Indexing with `.loc`

- Now let's look at `.loc` which accepts *labels* as references to rows/columns. Column labels are also called "headers", and row labels are also called "indexes".

- Labels are often represented as text (especially column headers), which is known as a *string* type in Python. Strings are always written between quotation marks.

- If your CSV file doesn't have any labels, then Pandas will assign it integers as labels by default. For example, the IMDB table doesn't have row labels included, so the rows are labelled as `0`, `1`, `2`, `3`, etc. This can be very confusing at first, since `.iloc[]` and `.loc[]` seem to return the same values:

```{python}
imdb.loc[[0]] #Get first row using label
```

```{python}
imdb.iloc[[0]] #Get first row using integer - looks the same as above!
```

```{python}
imdb.loc[0, 'Genre'] # Get the row labelled '0' and the column labelled 'Genre'
```

```{python}
imdb.iloc[0, 'Genre'] #this will raise an error, because iloc only accepts integers
```

In the IMBD data, numbers work perfectly well as row labels, but this isn't always the case. Let's open up another table with data about languages around the world, and set up more appropriate row labels.

### Practice
- Use the `.read_csv()` method from the beginning of the lecture to load the dataset called "WACL.csv" from the "data" folder
- Assign it to a variable called 'languages'
- Use the `head()` function to inspect the first 8 rows.

```{python}
#PRACTICE CELL

languages = pd.read_csv('data/WACL.csv')
languages.head()
```

Note on the left there is a series of integers, representing the default row index. 
All of the languages have a unique 3 letter code in the `iso_column`, and that would make a better index label. We can convert a column to an index with the `set_index()` function like this:

```{python}
languages.set_index('iso_code')
```

Note the new set of codes on the left, which replace the integer indexes from before. The `iso_code` column is also gone from the table. Now try using `.loc` to access the first language like this:

```{python}
languages.loc['aiw']
```

Ooops! That raises a KeyError, which means that the label 'aiw' still doesn't exist! Why not? 

This is because `set_index()`, like many functions in Pandas, returns a *copy* of your DataFrame. It does not modify the original. After setting the index, you need re-assign the new DataFrame to the old variable. Get comfortable with this pattern of coding.

```{python}
languages = languages.set_index('iso_code')
```

```{python}
languages.loc['aiw'] #This returns a Series (a single column), use double brackets [['aiw']] to get a DataFrame
```

You can also tell Pandas about the index column immediately when you create the DataFrame, instead of setting it later:

```{python}
languages = pd.read_csv('data/WACL.csv', index_col='iso_code')
languages
```

Using `.loc`, you can specify both a row and column label:

```{python}
languages.loc['mnk', 'status']
```

Lastly, you can select many rows and many columns by putting the labels in square brackets:

```{python}
languages.loc[ ['dmg','klv','ute'], ['latitude', 'longitude'] ]
```

In some cases, you might want to combine an integer with a label. 
If you have a row label and a column number you can combine them using `.loc` like this:

```{python}
languages.loc['aiw', languages.columns[0]] # Get the row labelled 'aiw', and the first column
```

And if you have a row number and a column label, then use this pattern:

```{python}
languages.loc[languages.index[-1], 'continent'] #Get the last row, and the column labelled the 'Continent'
```

### Practice
- Use .loc to return a DataFrame for the language with the code 'aiw'
- Use .loc to find the continent for the Zuni language ('zun')
- Use .loc to find the language family and endangerment status of Hausa ('hau'), Japanese ('jpn'), and Warlpiri ('wbp')
- Use .loc to get the longitude column for the 7th row
- Use .loc to get the language family for the 3rd row from the bottom

```{python}
#PRACTICE CELL
```

#### Indexing cheatsheet

|Method|Syntax|Output|
|---|---|---|
|Select column|`df[col_label]`|Series|
|Select row/column by label|`df.loc[row_label, col_label]`|Object for single selection, Series for one row/column, otherwise DataFrame|
|Select row/column by integer|`df.iloc[row_int, col_int]`|Object for single selection, Series for one row/column, otherwise DataFrame|
|Select by row integer & column label|`df.loc[df.index[row_int], col_label]`|Object for single selection, Series for one row/column, otherwise DataFrame|
|Select by row label & column integer|`df.loc[row_label, df.columns[col_int]]`|Object for single selection, Series for one row/column, otherwise DataFrame|

# Column operations
One powerful feature of Pandas is the ability to perform operations on entire columns of data at once (technically called 'vectorization'). This is often useful when dealing with numbers, so let's load another dataset with student scores across a six different high school subjects. Open the file `data/student_scores.csv` using the `pd.read_csv()` function and save it in a variable called `scores`. 

### Practice
- Set the row index to the student ID
- What did student #41169 score in History?
- Get only the Geography and Art scores.
- How did student #52230 score on the 10th question?
- How did the first three students score in English?

```{python}
#PRACTICE CELL
```

It's common to want the highest, lowest, and average scores for a student, or for a particular subject. This which can be done using the functions `.max()`, `.idxmax()`, `.min()`, `.idxmin()`, or `.mean()`. 

```{python}
scores = pd.read_csv('data/student_scores.csv')
scores = scores.set_index('Student_ID')
```

```{python}
#Find the highest score for a particular student
scores.loc[46410].max()
```

```{python}
#Find the row index (=student IDs) of the students with the highest score in each subject
scores.idxmax()
```

```{python}
#For each student, find their highest scoring subject
scores.idxmax(axis=1)
```

```{python}
#Find the lowest score in a particular subject
scores['English'].min()
```

```{python}
#Find the row index (=student ID) of the students with the lowest scores in each subject
scores.idxmin()
```

```{python}
#For each row (student), find their lowest scoring subject
scores.idxmin(axis=1)
```

```{python}
#Find the average for every column (subject) at once
scores.mean()
```

```{python}
#Find the average for every row (student) at once
scores.mean(axis=1)
```

The results of these operations can be assigned to new columns in your DataFrame.

```{python}
scores['Student Mean'] = scores.mean(axis=1) 
scores
```

A highly useful feature of Pandas is the ability to perform math on entire columns at once. For example, suppose the Chemistry exam was too difficult, and we need to scale everyone's grade up by a small amount. 

```{python}
scores['Scaled Chemistry'] = scores['Chemistry'] * 1.03
scores
```

You can also perform the operations between columns. For example, to find the mean of just the sciences:

```{python}
scores['Science Mean'] = (scores['Biology'] + scores['Chemistry'] + scores['Physics']) / 3
```

### Practice
- Add a new column called "Lowest grade" that contains the lowest grade for each student
- Add a new column called "Best subject" that contains the subject where the student got the highest score
- Suppose a TA made a grading error. Increase everyone's English score by 1%

```{python}
#PRACTICE CELL
```

# Deleting rows and columns

You can remove rows with the `.drop()` function by specifying a row label

```{python}
#drop() returns a copy, so don't forget to save it back to a variable!
scores = scores.drop(79496) #drops the student with id 79496
scores = scores.drop([14379, 54578]) #drops multiple students, note the square brackets
```

You can remove columns with `.drop()` by adding `axis=1`

```{python}
scores = scores.drop('Biology', axis=1) #drops the Biology column
scores = scores.drop(['Art', 'Drama'], axis=1) #drops multiple columns, note the square brackets
```

You can also created new DataFrames by selecting only certain rows from an old one, which effectively 'drops' them

```{python}
scores = pd.read_csv('data/student_scores.csv', index_col='Student_ID') #deleted too many things earlier, have to reload
science_scores = scores[['Biology', 'Chemistry', 'Physics']] #note the double-brackets!
science_scores
```

# Sorting

DataFrames can be sorted according to column values with the `sort_values` function. Let's return to the Internet Movie Database.

```{python}
imdb = pd.read_csv('data/imdb.csv')
```

```{python}
imdb.head(5) #remind yourself what this looks like
```

```{python}
#By default sorts from lowest to highest
imdb.sort_values(by='Gross')
```

```{python}
#If you want to sort highest-to-lowest, set the ascending argument to False
imdb.sort_values(by='Released_Year', ascending=False)
```

# Writing data back to a file
After you've created a dataset in Pandas and made some changes, you may want to save those changes back to a file. This can be done easily with `.to_csv()`:

```{python}
imdb.to_csv('my_imdb.csv')
```

### Practice
- Load the IMBD data
- Make a new DataFrame that contains only these columns: Series_Title, Runtime, IMDB_Rating, Director
- Set the Series_Title as the row index
- Sort the data in reverse alphabetical order, by director name
- Convert the IMDB rating into a score out of 100
- Assume the Gross value is in American dollars. Convert it to Canadian dollars (1 USD = 1.37 CAD) then remove the original Gross column.
- Write the DataFrame to a csv file and open it in Excel. Note what happens to your row labels in this file!

