## Worksheet 4 - Grouping

Run the cell below to import the necessary packages for this worksheet.

```python
import pandas as pd
import numpy as np
```

### Q1

rubric={autograde:1}

We return to the Narrabeen beach survey dataset that we encountered in Lab 1. Read in the data from `data/beach_data.csv` and save it to a data frame named `beach_df`, making sure to set the dates as index in a pandas datetime format.

```python
beach_df = pd.read_csv("data/beach_data.csv")
beach_df.index = pd.DatetimeIndex(beach_df["Unnamed: 0"])

```

```python
grader.check("q1")
```

<p><strong><pre style='display: inline;'>q1</pre></strong> passed! 🌈</p>

## Q2

rubric={autograde:1}

The data is quite irregularly spaced, with a frequency ranging between a few weeks to a few months. Your first tasks are to

* Resample `beach_df` from the previous step to monthly intervals with the mean as the aggregation function (you can use both month start and month end frequency for resampling-- please use month start to pass the autograder).
* After resampling, subtract the mean value of each resulting "location" column from the same column. This will help to see if a location on the beach is narrower (negative numbers) or wider (positive numbers) at a certain time compared to the average.
* Finally, reshape the data set so it has two columnns `location` and `width`. The `location` column should indicate which of the five locations each measurement was taken at( Hint: make sure to pass `ignore_index=False` parameter while using melt method).

The final data frame should look something like the following:

| &nbsp;             | location   | width      |
| ------------------ | ---------- | ---------- |
| **datetime** | &nbsp;     | &nbsp;     |
| 1980-01-01         | location_1 | 4.796227   |
| 1980-02-01         | location_1 | 4.296227   |
| 1980-03-01         | location_1 | 0.596227   |
| 1980-04-01         | location_1 | 5.296227   |
| 1980-05-01         | location_1 | -13.903773 |
| ...                | ...        | ...        |
| 2019-07-01         | location_5 | -8.395284  |
| 2019-08-01         | location_5 | -7.578618  |
| 2019-09-01         | location_5 | -15.778618 |
| 2019-10-01         | location_5 | -16.478618 |
| 2019-11-01         | location_5 | -16.895284 |

```python
beach_df_fresh = beach_df.drop("Unnamed: 0", axis = 1)
beach_df_ms = beach_df_fresh.resample("MS").mean()
cols = [f"location_{i}" for i in range(1, 6)]
beach_df_ms[cols] = beach_df_ms[cols] - beach_df_ms[cols].mean()

beach_df = beach_df_ms.melt(var_name = 'location', value_name='width' ,ignore_index = False)
```

```python
beach_df
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
      <th>location</th>
      <th>width</th>
    </tr>
    <tr>
      <th>Unnamed: 0</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1980-01-01</th>
      <td>location_1</td>
      <td>4.796227</td>
    </tr>
    <tr>
      <th>1980-02-01</th>
      <td>location_1</td>
      <td>4.296227</td>
    </tr>
    <tr>
      <th>1980-03-01</th>
      <td>location_1</td>
      <td>0.596227</td>
    </tr>
    <tr>
      <th>1980-04-01</th>
      <td>location_1</td>
      <td>5.296227</td>
    </tr>
    <tr>
      <th>1980-05-01</th>
      <td>location_1</td>
      <td>-13.903773</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2019-07-01</th>
      <td>location_5</td>
      <td>-8.395284</td>
    </tr>
    <tr>
      <th>2019-08-01</th>
      <td>location_5</td>
      <td>-7.578618</td>
    </tr>
    <tr>
      <th>2019-09-01</th>
      <td>location_5</td>
      <td>-15.778618</td>
    </tr>
    <tr>
      <th>2019-10-01</th>
      <td>location_5</td>
      <td>-16.478618</td>
    </tr>
    <tr>
      <th>2019-11-01</th>
      <td>location_5</td>
      <td>-16.895284</td>
    </tr>
  </tbody>
</table>
<p>2395 rows × 2 columns</p>
</div>

```python
grader.check("q2")
```

<p><strong><pre style='display: inline;'>q2</pre></strong> passed! 🌟</p>

## Q3

rubric={autograde: 1}

* Extract the month from the `DatetimeIndex`, and use it to create a new `month` column in the `beach_df` dataframe that you wrangled in question 2.
* Group the data by month and aggregate it based on the mean. Use this to find the three months where the beach was narrowest on average. Store your answer in a dataframe called `beach_top_3` with 3 rows, 'month' as index and 1 column for 'width'.

```python
beach_df['month'] = beach_df.index.month
beach_top_3 = beach_df.groupby(by = 'month').mean('width').sort_values('width').head(3)
```

```python
beach_top_3
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
      <th>width</th>
    </tr>
    <tr>
      <th>month</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>9</th>
      <td>-1.935821</td>
    </tr>
    <tr>
      <th>8</th>
      <td>-1.766926</td>
    </tr>
    <tr>
      <th>7</th>
      <td>-1.678812</td>
    </tr>
  </tbody>
</table>
</div>

```python
grader.check("q3")
```

<p><strong><pre style='display: inline;'>q3</pre></strong> passed! 💯</p>

## Q4

rubric={autograde:1}

Perform a double `groupby()` to determine the combination of **month** *and* **location** for which the beach is the widest. This time, aggregate the data based on the median value per group.

Store your output in a dataframe called `beach_df_widest` with a single row. It should have the `month` and `location` as the index, and 1 column called `width`.

> Note: Continue to use the `beach_df` you output in Q2 in order to solve this question.

```python
beach_df_widest = (
    beach_df.groupby(['month', 'location'])['width']
    .median()
    .reset_index()
)

max_width = beach_df_widest['width'].max()

beach_df_widest = beach_df_widest.loc[beach_df_widest['width'] == max_width].set_index(['month','location'])
beach_df_widest
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
      <th></th>
      <th>width</th>
    </tr>
    <tr>
      <th>month</th>
      <th>location</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5</th>
      <th>location_5</th>
      <td>10.471382</td>
    </tr>
  </tbody>
</table>
</div>

```python
grader.check("q4")
```

<p><strong><pre style='display: inline;'>q4</pre></strong> passed! 🌟</p>

## Q5

rubric={autograde: 1}

Run the code cell below to create two dataframes, `dates` and `rooms`.

```python
# RUN THIS CELL

dates = pd.DataFrame(
                    { 'name': ['Kate', 'Kaiyun', 'Prajeet', 'Tiffany', 'Mohit', 'Eric'],
                      'day': ['Monday', 'Tuesday', 'Tuesday', 'Wednesday', 'Thursday', 'Wednesday'],
                      'time': ['5pm', '4:30pm', '1pm', '1pm', '4pm', '4pm']
                    }
                    )

rooms = pd.DataFrame(
                    {
                      'day': ['Wednesday', 'Wednesday', 'Thursday', 'Tuesday', 'Tuesday', 'Monday'],
                      'time': ['4pm', '1pm', '4pm', '4:30pm', '1pm', '5pm'],
                      'room': ['MCML 160', 'ESB 3174', 'ICCS X153', 'ICCS X153', 'ESB 1046', 'ICCS X153']
                    }
                    )
```

The data frame `dates` contains the dates and times for some instructor and TA office hours for DSCI 511 this term. In `rooms` you will find room booking information for these dates and times. Your task is to:

* Combine the two data frames meaningfully to make a 'time table' of office hours. You will have to decide whether `concat()` or `merge()` is more suitable for this.
* Save this time table in a data frame titled `oh_info` with index `name`
* Make sure `oh_info` has the three columns `day`, `time` and `room`.

```python
oh_info = pd.merge(rooms, dates, how =  "inner").set_index(['name'])
...
```

```python
oh_info
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
      <th>day</th>
      <th>time</th>
      <th>room</th>
    </tr>
    <tr>
      <th>name</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Eric</th>
      <td>Wednesday</td>
      <td>4pm</td>
      <td>MCML 160</td>
    </tr>
    <tr>
      <th>Tiffany</th>
      <td>Wednesday</td>
      <td>1pm</td>
      <td>ESB 3174</td>
    </tr>
    <tr>
      <th>Mohit</th>
      <td>Thursday</td>
      <td>4pm</td>
      <td>ICCS X153</td>
    </tr>
    <tr>
      <th>Kaiyun</th>
      <td>Tuesday</td>
      <td>4:30pm</td>
      <td>ICCS X153</td>
    </tr>
    <tr>
      <th>Prajeet</th>
      <td>Tuesday</td>
      <td>1pm</td>
      <td>ESB 1046</td>
    </tr>
    <tr>
      <th>Kate</th>
      <td>Monday</td>
      <td>5pm</td>
      <td>ICCS X153</td>
    </tr>
  </tbody>
</table>
</div>

```python
grader.check("q5")
```

<p><strong><pre style='display: inline;'>q5</pre></strong> passed! 🚀</p>