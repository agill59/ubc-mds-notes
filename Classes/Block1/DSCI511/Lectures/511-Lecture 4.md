# Lecture 4: Plotting, Merging and Grouping

## Lecture Learning Objectives

- Make basic plots in Pandas by accessing the `.plot` attribute or importing functions from `pandas.plotting`.
- Combine dataframes using `df.merge()` and `pd.concat()` and know when to use these different methods.
- Apply functions to a dataframe or Series `DataFrame.apply()` and `DataFrame.map()` and `Series.map()`
- Perform grouping and aggregating operations using `df.groupby()` and `df.agg()`.
- Perform aggregating methods on grouped or ungrouped objects such as finding the minimum, maximum and sum of values in a dataframe using `df.agg()`.

```python
import numpy as np
import pandas as pd
```

## Visualizing your data

You will study data visualization in depth in DSCI 531. However, it is good to know that pandas provides some basic plotting functionality-- you can use this to take a quick look at your data.

This feature of pandas is accessed via the `plot()` method. It uses the `matplotlib` library, which you can install with `conda install matplotlib` as you have done with other packages.

Once we have `matplotlib` installed, let's load in the dataset `'data/cycling_data.csv'` which includes some dates, times and comments about bike rides.

```python
bike_df = pd.read_csv('data/cycling_data.csv', index_col=0, parse_dates=True).dropna()
bike_df.head()
```

Let's go ahead and make a plot of the distances ridden.

```python
bike_df['Distance'].plot()
```

Notice the line graph joined adjacent points together, which may suggest there was a non-zero riding distance on (say) Sep 13 or Sep 21. But there are no biking entries for these days!

Plotting with pandas can be a good way to take a quick look at your data. It can also help you to spot potential problems. For example, let's make a graph of *cumulative* distance covered over time.

```python
bike_df['Distance'].cumsum().plot()
```

That kink in the graph is funny... since we're plotting cumulative distance, the line should always be moving up and to the right!

```python
bike_df.loc['2019-09-18']
```

Looks like some entries were out of chronological order. We should sort the index before plotting.

```python
bike_df = bike_df.sort_index()
bike_df['Distance'].cumsum().plot()
```

That's better! We can also change plot parameters like colour and line thickness.

```python
bike_df['Distance'].cumsum().plot.line(fontsize=8, linewidth=3, color='r', ylabel='km');
```

There are many other kinds of plots you can make:

| Method             | Plot Type           |
| ------------------ | ------------------- |
| `bar` or `barh`    | bar plots           |
| `hist`             | histogram           |
| `box`              | boxplot             |
| `kde` or `density` | density plots       |
| `area`             | area plots          |
| `scatter`          | scatter plots       |
| `hexbin`           | hexagonal bin plots |
| `pie`              | pie plots           |

Plus, there are more advanced plotting functions (scatter matrix, lag plots and others) that can be accessed via the `pandas.plotting` module.

## Combining multiple data frames

Sometimes we have meaningful data spread across different data frames (or read in from different files) that we wish to combine. There are many ways to combine data in pandas, and we will look at some of them today.

To begin let's load in some data frames with weather information for different dates. You might notice that these frames share information across certain rows or columns, i.e. the data sets are related.

```python
w1 = pd.read_csv('data/weather1.csv', index_col=0)
w2 = pd.read_csv('data/weather2.csv', index_col=0)
w3 = pd.read_csv('data/weather3.csv', index_col=0)
w4 = pd.read_csv('data/weather4.csv', index_col=0)
```

```python
w1 # Let's look at these Data Frames
```

```python
w2
```

```python
w3
```

```python
w4
```

### Combining with `pd.concat()`

The simplest way to combine Data Frames in pandas is to use `pd.concat()`, which concatenates them along a shared axis. The default is to concatenate rows:

```python
pd.concat([w1,w2])
```

The above worked well since both Data Frames `w1` and `w2` have the same set of columns. Let's see what happens when we have a few different columns:

```python
pd.concat([w1,w4])
```

We get a new frame containing columns from both `w1` and `w4`. The common columns are concatenated, and missing data shows up as `NaN`. Notice that pandas did not know that the indices were 'common', i.e. indices 2 and 3 are repeated in the output.

Next, let's concatenate two Data Frames with shared rows but different columns:

```python
pd.concat([w1,w3], axis = 1)
```

And again when only some rows are shared:

```python
pd.concat([w1,w4], axis = 1)
```

This time pandas knew to use a shared index for rows, but the column indices were treated independently. As a result, we have "duplicate" columns in the output.

We can concatenate as many Data Frames as we want, but each call to `pd.concat` creates a new copy of the data frame. So instead of copying 'one-at-a-time', the best practice is to assemble all required Data Frames into a single list, and only call `pd.concat` once.

```python
pd.concat([w1,w2,w3,w4])
```

We were certainly able to "stick" our data frames together here, but we may not have gained the information we wanted. For example, the data frame above contains entries for both `Mean Temp` and `Total Rain` for November 12, 2012, but these are split across different rows.

If we know beforehand that shared dates should correspond to the same example in our data, it would be good to use this fact when combining the data frames. The next section discuss how to implement this in pandas.

### Combining with `pd.Merge`

Another pandas function that is useful for joining Data Frames together is `pd.merge`. This function implements SQL-style joins in pandas. You will learn more about SQL when you study databases in DSCI 513. `pd.merge()`.

When using `pd.concat()`, you have to do all the work beforehand to make sure the rows and/or columns line up just right for the concatenation to give the correct, meaningful result. This can be tricky when working with large Data Frames. With `pd.merge`, we can pass a little more meaningful information to pandas which it uses combine our Data Frames to get the result we intended. Essentially, the matching is done based on columns shared between the Data Frames.

```python
pd.merge(w1, w4, how='outer', on = ['Date/Time', 'Month', 'Year'])
```

This time, pandas knew that rows with matching Date/Time, Month and Year should count as the same observation for both frames. But notice that the index was ignored! The resulting Data Frame has an entirely new index, since we were matching on columns. Matching on the index is also possible.

```python
pd.merge(w1, w4, how='outer', left_index = True, right_index = True)
```

The `how='outer'` parameter tells pandas to produce a Data Frame containing the full list of keys from both Data Frames. We could restrict this to keys in only one Data Frame:

```python
pd.merge(left = w1, right = w4, how='right', on = ['Date/Time', 'Month', 'Year'])
```

Notice again that the index was not retained when matching on columns.

Let's create a Data Frame of 'seasons' to categorize the different months of the year.

```python
seasons = pd.DataFrame({
                    'month':[1,2,3,4,5,6,7,8,9,10,11,12], 
                    'season': ['Winter', 'Winter', 'Spring', 'Spring', 'Spring', 'Summer', 'Summer', 'Summer', 'Fall', 'Fall', 'Fall', 'Winter']
                       }
                      )

seasons
```

We made this Data Frame "by hand", using a dictionary object. Dictionaries are another object type in Python, similar to lists but indexed based on "keys" that do not need to be integers. The keys in this case serve as column names when passed to `pd.DataFrame()`.

Now if we have some dataset that includes months as a column, we can merge with `seasons`. For example, we can use the following data set that we saw in Lecture 1 (we are dropping some columns):

```python
weather = pd.read_csv('data/YVR_weather_data.csv').dropna(axis=1)
weather.head()
```

Let's attempt a left join for `weather` with `seasons`. We will have to tell pandas that `Month` in the left frame matches `month` in the second (note that the strings are not identical).

```python
pd.merge(left = weather, right = seasons, how = 'left', left_on = 'Month', right_on = 'month')
```

With `pd.merge()`, we can combine data frames in a one-to-one, one-to-many or a many-to-many fashion, leading to a large range of possibilities. You will see many examples of joins in DSCI 513.

### Applying Custom Functions

There will be times when you want to apply a function that is not built-in to Pandas. For this, we also have methods:

* `df.apply()`, applies a function **column-wise or row-wise** across a dataframe (the function must be able to accept/return an array)
* `df.map()`, applies a function element-wise (so individually) across **every element** in the data frame (meant for functions that accept/return single values at a time)
* `series.apply()`, same as above for Pandas `Series`.
* `series.map()`, also for Series, but optionally accepts a dictionary as input.

It is important to understand how `.apply()` and `.map()` differ. Sometimes it appears as though these functions work the same. This is the case for functions which accept and return single values at a time, like the `round` function:

<img src="img/apply-round.png" width=600>

<img src="img/map-round.png" width=600>

However, when using `.apply()` and `.map()` with methods/functions that act on a series or array and return a single value, like `np.sum`, the objects they return are quite different:

<img src="img/apply-sum.png" width=600>

<img src="img/map-sum.png" width=600>

It is also important to know how `pandas.DataFrame.apply()` works with regards to what `axis=0` and `axis=1` means. This clarification is important because in many other pandas functions/methods,  `axis=0` means row-wise, and  `axis=1` means column-wise. But that is not the case here. From the [`pandas.DataFrame.apply()` docs](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.apply.html):

*"Objects passed to the function are Series objects whose index is either the DataFrame’s index (axis=0) or the DataFrame’s columns (axis=1)."*

So for `pandas.DataFrame.apply()`:

- `axis=0` means the **columns** (saying you want the index to be the DataFrame’s index)
- `axis=1` means the **rows** (saying you want the index to be the DataFrame’s columns)

**For `pandas.DataFrame.apply()` `axis=0` is the default, meaning it works column-wise unless you specify otherwise (as shown in the examples above).**

Here's some examples of using `.apply()` and `.map()`  with our weather data:

```python
weather[['Month', 'Mean Temp (°C)']].apply(np.log) # we will see more from numpy in future lectures
```

```python
weather['Mean Temp (°C)'].apply(int) # apply is a method for both Data Frames and Series
```

We provide a dictionary as input to `Series.map`.

```python
weather[['Month', 'Mean Temp (°C)']].apply(np.sum)
```

```python
seasons['season'].map(
                        {'Winter':'Cold', 'Spring':'Not Cold', 'Summer': 'Warm'}
                     )
```

Note that `'Fall'` did not appear as a key in the dictionary supplied, and the output shows `NaN` for the corresponding entries.

### Grouping

Often we are interested in examining specific groups in our data. In pandas, `df.groupby()` allows us to group our data based on one or more variables. It is analgous to the `group_by` function in R

```python
imdb = pd.read_csv('data/imdb.csv')
imdb.head()
```

Let's group this Data Frame on `Released_Year`:

```python
imdb.groupby(by = 'Released_Year')
```

What is a `DataFrameGroupBy` object? It's really just a dictionary of index-mappings! The keys represent the different groups, and each key is mapped to a list of indices corresponding to that group. We can look at them directly if we really wanted to:

```python
imdb.groupby(by = 'Released_Year').groups
```

We can access a specific group using the `get_group()` method.

```python
imdb.groupby(by='Released_Year').get_group(1995)
```

The usual thing to do, however, is to apply aggregation functions to the `groupby` object. For example, we could take the mean of the numeric columns. Use `numeric_only=True` to tell pandas to ignore non-numeric columns.

```python
imdb.groupby(by='Released_Year').mean(numeric_only=True)
```

Even better would be to select the numeric columns ahead of time, and then call `grouby()` and `mean()`. We can apply multiple functions using `aggregate()`.

```python
(
    imdb
    .loc[:, ['Released_Year', 'IMDB_Rating', 'Meta_score', 'No_of_Votes']]
    .groupby(by='Released_Year')
    .aggregate(['mean', 'sum', 'count'])
)
```

We could also apply different functions to different columns.

```python
imdb.groupby(by="Released_Year").agg(
    {
        "Meta_score": ["max", "min", "mean"],
        "Gross": ["sum"]
    }
)
```

Later on, we will learn how to write our own Python functions. Such 'custom' functions can also be passed to `aggregate()`.

Note that we can pass multiple columns to `groupby()`. The groups will be indexed based on all combinations of entries that appear in the listed columns

```python
imdb.groupby(by=["Released_Year", "Certificate"]).agg(
    {
        "Meta_score": ["mean"],
        "Gross": ["sum"]
    }
)
```

Finally, you can `aggregate()` on a non-grouped data frame as well. This is essentially what `DataFrame.describe()` does under the hood.

```python
(
    imdb
    .loc[:, ['IMDB_Rating', 'Meta_score', 'No_of_Votes']]
    .agg(['mean', 'min', 'count'])
)
```

We could also use aggregate function to apply on a single column, and assign the aggreated values to a new column in the original data frame.

```python
imdb.groupby('Released_Year').aggregate(imdb_avg = ('IMDB_Rating','mean'), Meta_Score_avg = ('Meta_score', 'mean'))
```

```python

```