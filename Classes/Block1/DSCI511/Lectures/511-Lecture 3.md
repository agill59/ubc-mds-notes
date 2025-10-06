# Lecture 3: Advanced data wrangling with Pandas

## Lecture learning objectives

- Manipulate strings in Python with methods like `find`, `replace` and `join`.
- Access string methods in Pandas via `Series.str`.
- Understand how to use regular expressions in Pandas for wrangling strings.
- Differentiate between datetime object in Pandas such as `Timestamp`, `Timedelta`, `Period`, `DateOffset`.
- Create these datetime objects with functions like `pd.Timestamp()`, `pd.Period()`, `pd.date_range()`, `pd.period_range()`.
- Index a datetime index with partial string indexing.
- Perform basic datetime operations like splitting a datetime into constituent parts (e.g., `year`, `weekday`, `second`, etc), apply offsets, change timezones, and resample with `.resample()`.
- Create a categorical column in a DataFrame using the `Categorical` data type in Pandas.

```python
# !pip install altair
```

```python
import pandas as pd
import numpy as np
import altair as alt
pd.set_option('display.max_rows', 10)
```

## Strings

Strings are a core data type in Python used for text-based data. They are enclosed in single (`'`) or double (`"`) quotes.

```python
single = 'I love Python!'
double = "I love Data Science!"
single
```

Sometimes our strings may themselves contain quotation marks or apostrophes.

```python
single = 'That's weird' 
# raises error 
```

We can use a backlash `\` (called an escape character) to prevent Python from interpreting `'` as a string delimiter

```python
single = 'That\'s weird'
single
```

You may also have strings that contain backslashes, so you need to tell Python not to treat them as escape characters.

```python
path1 = 'Documents\\Lecture1'
path2 = r'Documents\Lecture1\solutions.ipynb'
```

#### Manipulating Strings

- Strings can be concatenated using the `+` operator.
- The `len()` function returns the length of a string, i.e. the number of characters it contains.
- Use the `lower()`, `upper()` and `capitalize()` methods for case conversion.
- You can remove leading and trailing whitespace from strings using `strip()`, `lstrip()`, and `rstrip()`.

```python
first = 'florence'
last = 'nightingale'
first + last
```

```python
len(first)
```

```python
first.upper()
```

```python
fullname = '  Florence Nightingale      '
fullname.strip()
```

I can also check if a string is empty or not by using simple conditional statements. This can be useful for filtering out empty strings.

```python
# Not empty as we assigned a non empty string to fullname. 
fullname == ""
```

```python
empty_string = ""
empty_string == ""
# True as it is empty
```

**Note:** A whitespace character is any character that is displayed as "blank space" when text is rendered for display. Usual spaces and tabs (`'t'`)  are both whitespace characters. You might also frequently encounter new line characters (`'\n'`), which force line breaks. Less commonly used whitespace characters are carriage returns, vertical tabs, and form feeds.

#### Substrings, Splitting and Joining

- You can slice a string using integer indexing.
- The `find()` and `index()` methods help you locate substrings.
- The `replace()` method allows you to replace occurences of one substring with another.
- The `split()` method splits one string into a list of substrings based on a delimiter
- The `join()` method joins a list of strings into one, using a specified delimiter

```python
text = 'The-University-of-British-Columbia'
text.find('v')
```

```python
text[1:].find('v')
```

```python
text.find('z')
```

```python
text.replace('-',' ') # Note this returns a new string, and does not modify the original in place.
```

```python
words = text.split('-')
words
```

```python
' '.join(words)
```

#### Formatted Strings

Python can "fill in the blanks" using specified input to create strings. One way of doing this is using f-strings. These are useful when you want to print statements including variables.

```python
num = 12
print(f"There are {num} months in a year")
```

The `print()` function above has some useful options that you can find in the Python documentation.

### Using String Methods in Pandas

In previous lectures, we saw that Pandas can apply arithmetic operations on an entire Series object "all at once". We can do the same for text data using `Series.str` which gives us access to the string methods we have seen.

```python
imdb = pd.read_csv('data/imdb.csv')
imdb['Genre']
```

```python
genres = imdb['Genre'].str.split(',', expand = True)
genres
```

Notice there's some whitespace in the resulting columns!

```python
genres.loc[1,1]
```

We know how to fix this

```python
genres[1] = genres[1].str.strip()
genres.loc[1,1]
```

There are many string methods available to use in pandas. A full list is available in the [documentation](https://pandas.pydata.org/docs/user_guide/text.html).

### Regular Expressions or REGEX

- A regular expression (regex) is a sequence of characters that defines a search pattern.
- Regex can do some truly magical things, so keep it in mind for complicated text wrangling!
- Regex can also be difficult to get right, but there are many online tools (e.g. [RegExr](https://regexr.com)) that can help you find the correct patterns.

You will learn more about regex in DSCI 521, so you don't need to learn it now. Here, just as an example, we use regex can find all movie titles in our IMDB dataset that start with a vowel and end with a consonant.

```python
findpattern = imdb['Series_Title'].str.findall(r'^[AEIOU].*[^aeiou]$')
findpattern.loc[:10]
```

Let's break down that regex:

- The leading `^` specifies the start of a string.
- Square brackets match a single character, so `[AEIOU]` is saying we want the first character to be a vowel.
- `.` matches any character and `*` means '0 or more times', so `.*` will match any number of any characters in the middle of our string
- Having a `^` inside and at the start of square brackets indicates a "not" and `$` matches the end of a string. So `[^aeiou]$` means we don't want the *last* character to be a vowel.

We could have used regex to split the genres as well, which would allow us to specify a greater number of possible delimiters.

```python
imdb['Genre'].str.split('[,:.!]', expand=True)
```

Many `DataFrame.str()` methods accept regular expression as default, but you don't have to use regex if you don't need it.

```python
imdb['Genre'].str.contains('ri', regex=False) # This tests for exact matches
```

## Time Series Data

Data Scientists often work with "time series" data, i.e. data that is indexed by time periods. We will breifly introduce time series in pandas here; you will encounter them much more in DSCI 574.

Pandas provides several datetime objects, including:

* Timestamp
* Timedelta
* Period
* DateOffset

We explore a few of them below.

#### Creating Datetimes

Most commonly you will want to

- Create a single point in time with `pd.Timestamp()`.
- Create a span of time with `pd.Period()`.
- Create an array of times with `pd.date_range()` or `pd.period_range()`.

```python
pd.Timestamp('July 29, 2005') # parsed from string
```

```python
pd.Timestamp(year = 2005, month = 7, day = 9) # pass data directly
```

Above we have a specific point in time. We can use `pd.Period()` to specify a span of time (like a day)

```python
span = pd.Period('2005-07-09')
print(span)
print(span.start_time)
print(span.end_time)
```

```python
span + pd.Timedelta('1 day')
```

Pandas understands when a `Timestamp` lies within a `Period`.

```python
point = pd.Timestamp('2005-07-09 12:00')
span = pd.Period('2005-07-09')
print(f'Point: {point}')
print(f' Span: {span}')
print(f'Point in span? {span.start_time < point < span.end_time}')
```

Often, you will want to create **arrays** of datetimes, not just single values. In pandas, these come in the form of a `DatetimeIndex`, `PeriodIndex` or `TimedeltaIndex`.

```python
pd.date_range('2020-09-01 12:00',
              '2020-09-11 12:00',
              freq='2d')
```

```python
pd.period_range('2020-09-01',
                '2020-09-11',
                freq='d')
```

We can use `Timedelta` to add or subtract time from a `DatetimeIndex` or `PeriodIndex`.

```python
pd.date_range('2020-09-01 12:00', '2020-09-11 12:00', freq='D') + pd.Timedelta('1.5 hours')
```

Finally, pandas represents missing datetimes with `NaT`.

#### Converting Existing Data

It's fairly common to have an array of dates as strings. We can use `pd.to_datetime()` to convert these to a datetime format.

```python
cycling = pd.read_csv('data/cycling_data.csv')
cycling.head() # Date column will be read in as strings
```

```python
cycling['Date'] = pd.to_datetime(cycling['Date'])
cycling['Date']
```

> ***Note:*** When working with messy or inconsistent date formats in a column, you can use the `format='mixed'` argument in `pd.to_datetime()`. This tells Pandas to automatically infer different formats row by row.
> Useful when your date column has a mix of formats like: "2020-03-15", "15/03/2020", "March 15, 2020"

We can actually tell pandas to do this within the `read_csv` statement. Let's also set the datetime as the Index.

```python
cycling = pd.read_csv('data/cycling_data.csv',
                      parse_dates=True,
                      index_col = 0)
cycling
```

#### Indexing Datetimes

Datetime index objects are just like regular index objects and can be selected, sliced, filtered etc. We can do **partial string indexing** to select datetimes within a certain range.

```python
cycling.loc['2019-10'] # All logs for Oct 2019
```

**Exact** matching:

```python
cycling.loc['2019-10-10 00:10:31']
```

And **slicing**:

```python
cycling.loc['2019-10-01':'2019-10-13'] # raises error
```

oops, that failed. The error suggests the index might not be sorted (i.e. it may not be monotonic). Let's fix that and try again.

```python
cycling.sort_index().loc['2019-10-01':'2019-10-13']
```

For getting all results between a certain time of day (potentially on different days) we can use `df.between_time()`.

```python
cycling.between_time('00:00', '01:00')
```

### Manipulating Datetimes

For more complex filtering, we may have to **decompose** our timeseries. There are many methods and attributes that we can access.

```python
cycling.index.weekday
```

```python
cycling.index.second
```

```python
cycling.index.day_name()
```

```python
cycling.index.month_name()
```

If you're acting on a Series instead of a DatetimeIndex, you can access these methods using `.dt`.

```python
s = pd.Series(pd.date_range('2011-12-29', '2011-12-31'))
s.year  # raises error
```

```python
s.dt.year
```

#### Resampling and Aggregating

One of the most common operations you will want do when working with time series is resampling to a coarser/finer/regular resolution. For example, you may want to resample daily data to weekly data.

We can do that with the `.resample()` method. For example, let's resample the irregular cycling timeseries to a regular 12-hourly series

```python
cycling.head()
```

```python
cycling.resample('1D')
```

The parameter `'1D'` is telling pandas to sample once daily. The letter `D` is the alias for the Daily offset, some other commonly used ones include

* `'h'`, `'min'` and `'s'` for hours, minutes,  and seconds (resp.)
* `'B'` for business day
* `'MS'` and `'ME'` for month start and month end (resp.)

A more complete list is available [here](https://pandas.pydata.org/pandas-docs/stable/user_guide/timeseries.html#offset-aliases).

Let's examine this `Resampler` object a little more.

```python
cycling.resample('2D').groups
```

The output isn't so decipherable, but our data has been aggregated into *groups*. Since some groups have more than one entry, we need to tell pandas what to do with these. One option is to apply `mean()` to get an aggregated summary for each group.

```python
dfr = cycling[["Time","Distance"]].resample('1D').mean()
dfr
```

There's quite a few `NaN`s in there. Some days do not have any data points-- we can choose how to handle these using `fillna` (for example, we could fill 0 distance and 0 time).

The resampled index still has much of the same functionality we have seen before:

```python
dfr['Weekday'] = dfr.index.day_name()
dfr.head(10)
```

We will study aggregate objects in pandas more thoroughly in Lecture 4, when we study the `groupby()` method.

### Categorical Data

What is categorical data? A categorical variable takes on a limited, and usually fixed, number of possible values. Examples are gender, social class, blood type, country affiliation, observation time or rating via Likert scales.

What is a Pandas categorical data type? Internally, the data structure consists of a `categories` array and an integer array of `codes` which point to the real value in the `categories` array.

Why use Pandas categoricals?

- They allow categories to be ordered, which is useful for mapping attributes like color or size to category levels in plots.
- They reduce memory usage and improve performance by storing repeated values more efficiently.
- They improve the performance of certain operations, such as groupby and sorting, by leveraging the categorical nature of the data.

*Source: [https://pandas.pydata.org/docs/user_guide/categorical.html](https://pandas.pydata.org/docs/user_guide/categorical.html)*

#### Examples of categorical data in Pandas

Let's load in `bean.csv` from the `data` directory. This file contains a small sample of a dataset from the [UC Irvine Machine Learning Repository](https://archive.ics.uci.edu/dataset/602/dry+bean+dataset).

```python
bean = pd.read_csv('data/bean.csv')
bean.head()
```

There 18 columns in the dataset. Most are numerical, but the last column contains text data

```python
bean.columns
```

```python
bean['Class']
```

A column of text data could have any number of distinct entries-- potentially each one could be different. Let's check how many unique entries this column contains.

```python
bean['Class'].nunique()
```

There are only 7 possibilities! As it turns out, they correspond to different classes of beans.

Pandas has a `category` data type for columns where the variable takes one of a small (usually fixed) number of categories. We can convert existing columns from `object` to `category` dtype as follows:

```python
bean['Class'] = bean['Class'].astype('category')
bean['Class']
```

```python
bean['Class'].dtype
```

We can look up the categories of a column by accessing the `Series.cat.categories` attribute. So to see the categories of the new 'Class' column, write:

```python
bean['Class'].cat.categories
```

Having categorical data is really useful for data vizualization. For example, we can plot bean length and width on the x and y axes (resp.) of a plot, and colour each data point based on the class it belongs to.

```python
alt.Chart(bean).mark_point().encode(
    x='MajorAxisLength',
    y='MinorAxisLength',
    color=alt.Color('Class', scale=alt.Scale(scheme='category10')),
).interactive()
```

We can clearly see that the beans form clusters based on length and width!

What if we wanted to change the order the colours were assigned to our categories? The default is alphabetical. We could do this to change to reverse alphabetical:

```python
bean['Class'].unique()
```

```python
category_order = sorted(bean['Class'].unique(), reverse=True)
category_order
# Convert 'species' column to a categorical type with the specified order
bean['Class'] = pd.Categorical(bean['Class'], categories=category_order, ordered=True)
bean['Class'].cat.categories
```

What will our plot look like now?

```python
alt.Chart(bean).mark_point().encode(
    x='MajorAxisLength',
    y='MinorAxisLength',
    color=alt.Color('Class', scale=alt.Scale(scheme='category10'))
)
```

Let's load another dataset that contains data about book sales on Amazon by reading in `data/publishers.csv`. This data was sourced from [CORGIS](https://corgis-edu.github.io/corgis/csv/).

```python
pub = pd.read_csv('data/publishers.csv', index_col = 0)
```

Each row contains information on a particular book (although for some reason the dataset omits the book's title and author!) Let's see how many unique entries are contained in the `'publisher.type'` column.

```python
pub['publisher.type'].value_counts()
```

There are only four categories, and they have a natural ordering among them. We may want to sort our dataset based on the 'size' of the publisher-- but our column is an `object` dtype and contains strings. Strings are sorted alphabetically!

```python
pub = pub.sort_values(by = 'publisher.type', ascending = False)
print(pub.iloc[0]['publisher.type']) # Check the first row
print(pub.iloc[240]['publisher.type']) # Somewhere in the middle
print(pub.iloc[440]['publisher.type']) # Somewhere else in the middle
print(pub.iloc[-1]['publisher.type']) # Check the last row
```

That sort wasn't useful here. Pandas categories, in contrast to strings, can be given any order we choose.

```python
pub['publisher.type'] = pub['publisher.type'].astype('category')
pub['publisher.type'] = (pub['publisher.type'].cat.reorder_categories(
                                ['single author', 'indie', 'small/medium', 'big five'], ordered=True)
                        )
```

```python
pub = pub.sort_values('publisher.type', ascending = False)
```

```python
print(pub.iloc[0]['publisher.type']) # Check the first row
print(pub.iloc[240]['publisher.type']) # Somewhere in the middle
print(pub.iloc[440]['publisher.type']) # Somewhere in the middle
print(pub.iloc[-1]['publisher.type']) # Check the last row
```

The Data Frame also contains a column called `'genre'`. That's another candidate for categorical data, so let's see what unique values it contains.

```python
pub['genre'].value_counts()
```

Oops, looks like two spellings (`'comics'` and `'Comics'`) are listed for the same genre! This is a common occurrence. In general, it is best to do your data wrangling first and only convert to the `category` dtype once your data is cleaned up.