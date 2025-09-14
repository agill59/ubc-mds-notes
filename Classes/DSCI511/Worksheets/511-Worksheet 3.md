# Worksheet 3 - Advanced Wrangling With Pandas

These exercises complement DSCI 511 lecture 3.

## Exercises

```python
import pandas as pd
import numpy as np
import datetime as dt
```

### Q1

rubric={autograde:1}

Given the following variables:

```
thing = "light"
speed = 299792458 
```

Use f-strings to print:

```
The speed of light is 299792458 m/s.
```

```python
thing = "light"
speed = 299792458  # m/s
q5_answer = f"The speed of {thing} is {speed} m/s."

print(q5_answer)
```

    The speed of light is 299792458 m/s.

```python
grader.check("q1")
```

<p><strong><pre style='display: inline;'>q1</pre></strong> passed! ✨</p>

### Q2

rubric={autograde:1}

In this set of practice exercises we'll be looking at a cool dataset of real passwords (made available from actual data breaches) sourced and compiled from [Information is Beautiful](https://informationisbeautiful.net/visualizations/top-500-passwords-visualized/?utm_content=buffer994fa&utm_medium=social&utm_source=twitter.com&utm_campaign=buffer) and contributed to [R&#39;s Tidy Tuesday project](https://github.com/rfordatascience/tidytuesday). These passwords are common ("bad") passwords that you should avoid using!

The dataset has the following columns:

| variable          | class | description                                                                                                                           |
| :---------------- | :---- | :------------------------------------------------------------------------------------------------------------------------------------ |
| rank              | int   | popularity in their database of released passwords                                                                                    |
| password          | str   | Actual text of the password                                                                                                           |
| category          | str   | What category does the password fall in to?                                                                                           |
| value             | float | Time to crack by online guessing                                                                                                      |
| time_unit         | str   | Time unit to match with value                                                                                                         |
| offline_crack_sec | float | Time to crack offline in seconds                                                                                                      |
| rank_alt          | int   | Rank 2                                                                                                                                |
| strength          | int   | Strength = quality of password where 10 is highest, 1 is lowest, please note that these are relative to these generally bad passwords |
| font_size         | int   | Used to create the graphic for KIB                                                                                                    |

In these exercises, we're only interested in the `password`, `value` and `time_unit` columns so import only these columns as a dataframe named `df` from this url: [https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2020/2020-01-14/passwords.csv](https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2020/2020-01-14/passwords.csv)

Note: There are several 'NA' rows at the end. Pass the optional parameters `skipfooter=7` and `engine='python'` in your call to `pd.read_csv` in order to skip these.

```python
data_Q2 = pd.read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2020/2020-01-14/passwords.csv", skipfooter = 7, engine = "python")
df = data_Q2[["password", "value", "time_unit"]]
```

```python
grader.check("q2")
```

<p><strong><pre style='display: inline;'>q2</pre></strong> passed! 🌟</p>

### Q 3.

rubric = {autograde:1}

How many password contain the sequence `123`? Store your output as a single value in `q3` object

```python
q3 = sum(df['password'].str.contains("123"))
q3
```

    12

```python
grader.check("q3")
```

<p><strong><pre style='display: inline;'>q3</pre></strong> passed! 🚀</p>

### Q 4.

rubric = {autograde:1}

The `value` column contains times in many different units. The code in the cell below will convert them all to hours so that they can be easily compared. Run the cell below before proceeding.

```python
# RUN THIS CELL

units = {
    "seconds": 1 / 3600,
    "minutes": 1 / 60,
    "days": 24,
    "weeks": 168,
    "months": 720, # assuming there are 30days in each month
    "years": 8760,
}

for key, val in units.items():
    df.loc[df['time_unit'] == key, 'value'] *= val 

df['time_unit'] = 'hours'
df.head()
```

    /tmp/ipykernel_24142/2774482533.py:15: SettingWithCopyWarning:
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead

    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      df['time_unit'] = 'hours'

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }`</style>`

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>password</th>
      <th>value</th>
      <th>time_unit</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>password</td>
      <td>60531.600000</td>
      <td>hours</td>
    </tr>
    <tr>
      <th>1</th>
      <td>123456</td>
      <td>0.308667</td>
      <td>hours</td>
    </tr>
    <tr>
      <th>2</th>
      <td>12345678</td>
      <td>30.960000</td>
      <td>hours</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1234</td>
      <td>0.003086</td>
      <td>hours</td>
    </tr>
    <tr>
      <th>4</th>
      <td>qwerty</td>
      <td>89.280000</td>
      <td>hours</td>
    </tr>
  </tbody>
</table>
</div>

#### Question:

What is the average time in hours needed to crack these passwords that include the sequence `123`?

Store your output as a single value in `avg_crack_123`

```python
avg_crack_123 = df[(df['password'].str.contains("123"))]['value'].mean()


```

```python
grader.check("q4")
```

<p><strong><pre style='display: inline;'>q4</pre></strong> passed! 🌟</p>

### Q5.

Create a `DatetimeIndex` called `date_range` consisting of dates starting 2010-01-01 00:00:00 until today, with daily frequency.

Hint: To get today's datetime, you can use `pd.Timestamp.now()`

```python
date_range = pd.date_range(dt.date(year = 2010, month = 1, day = 1), pd.Timestamp.now(), freq = "D")
```

```python
grader.check("q5")
```

<p><strong><pre style='display: inline;'>q5</pre></strong> passed! ✨</p>

### Q 6

The code cell below creates a new data frame called `df2` using the `date_range` you defined in the previous question. Run this cell.

```python
# create a DataFrame with the datetime index
df2 = pd.DataFrame(index=date_range)

df2
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }`</style>`

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2010-01-01</th>
    </tr>
    <tr>
      <th>2010-01-02</th>
    </tr>
    <tr>
      <th>2010-01-03</th>
    </tr>
    <tr>
      <th>2010-01-04</th>
    </tr>
    <tr>
      <th>2010-01-05</th>
    </tr>
    <tr>
      <th>...</th>
    </tr>
    <tr>
      <th>2025-09-07</th>
    </tr>
    <tr>
      <th>2025-09-08</th>
    </tr>
    <tr>
      <th>2025-09-09</th>
    </tr>
    <tr>
      <th>2025-09-10</th>
    </tr>
    <tr>
      <th>2025-09-11</th>
    </tr>
  </tbody>
</table>
<p>5733 rows × 0 columns</p>
</div>

For this question, create the following 4 additional columns for the dataframe `df2`:

- year
- month_name
- day
- weekday_name.

Fill them in with the appropriate values based on the index.

```python
df2['year'] = df2.index.year
df2['month_name'] = df2.index.strftime("%B")
df2['day'] = df2.index.day
df2['weekday_name'] = df2.index.strftime("%A")
```

```python
grader.check("q6")
```

<p><strong><pre style='display: inline;'>q6</pre></strong> passed! 🌟</p>