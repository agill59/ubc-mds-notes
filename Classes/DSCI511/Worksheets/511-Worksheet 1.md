
# Worksheet 1 - Introduction to Pandas

## Exercises

In this set of practice exercises we'll be investigating the carbon footprint of different foods. We'll be leveraging a dataset compiled by [Kasia Kulma](https://r-tastic.co.uk/post/from-messy-to-tidy/) and contributed to [R&#39;s Tidy Tuesday project](https://github.com/rfordatascience/tidytuesday).

```python
import pandas as pd
import numpy as np
from hashlib import sha1
```

### 1.1

rubric={autograde:1}

The dataset we'll be working with has the following columns:

| column        | description                       |
| :------------ | :-------------------------------- |
| country       | Country Name                      |
| food_category | Food Category                     |
| consumption   | Consumption (kg/person/year)      |
| co2_emmission | Co2 Emission (Kg CO2/person/year) |

Import the dataset as a dataframe named `df` from this url: [https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2020/2020-02-18/food_consumption.csv](https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2020/2020-02-18/food_consumption.csv)

```python
url = "https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2020/2020-02-18/food_consumption.csv"
df = pd.read_csv(url)
df.head()
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
      <th>country</th>
      <th>food_category</th>
      <th>consumption</th>
      <th>co2_emmission</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Argentina</td>
      <td>Pork</td>
      <td>10.51</td>
      <td>37.20</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Argentina</td>
      <td>Poultry</td>
      <td>38.66</td>
      <td>41.53</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Argentina</td>
      <td>Beef</td>
      <td>55.48</td>
      <td>1712.00</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Argentina</td>
      <td>Lamb & Goat</td>
      <td>1.56</td>
      <td>54.63</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Argentina</td>
      <td>Fish</td>
      <td>4.36</td>
      <td>6.96</td>
    </tr>
  </tbody>
</table>
</div>

```python
grader.check("q1_1")
```

<p><strong><pre style='display: inline;'>q1_1</pre></strong> passed! 🎉</p>

### 1.2

rubric={autograde:1}

Create a DataFrame that contains a single column, the one containing food categories. Save it in a variable called `food`.
Note this must be a DataFrame, not a Series, so be careful with your bracketing!

```python
food = pd.DataFrame(df["food_category"])
food
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
      <th>food_category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Pork</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Poultry</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Beef</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Lamb & Goat</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Fish</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>1425</th>
      <td>Milk - inc. cheese</td>
    </tr>
    <tr>
      <th>1426</th>
      <td>Wheat and Wheat Products</td>
    </tr>
    <tr>
      <th>1427</th>
      <td>Rice</td>
    </tr>
    <tr>
      <th>1428</th>
      <td>Soybeans</td>
    </tr>
    <tr>
      <th>1429</th>
      <td>Nuts inc. Peanut Butter</td>
    </tr>
  </tbody>
</table>
<p>1430 rows × 1 columns</p>
</div>

```python
grader.check("q1_2")
```

<p><strong><pre style='display: inline;'>q1_2</pre></strong> passed! 🌈</p>

### 1.3

rubric={autograde:1}

Find the country in row #1234. Save this in a variable called `country_1234`

```python
country_1234 = df.iloc[1234].country
country_1234
```

    'Madagascar'

```python
grader.check("q1_3")
```

<p><strong><pre style='display: inline;'>q1_3</pre></strong> passed! 🚀</p>

### 1.4

rubric={autograde:1}

What is the mean `co2_emission` of the whole dataset? Save the answer to a variable named `mean_co2_emmission`. Your answer should be a `np.float64`.

```python
mean_co2_emmission = df['co2_emmission'].mean()
mean_co2_emmission
```

    np.float64(74.383993006993)

```python
grader.check("q1_4")
```

<p><strong><pre style='display: inline;'>q1_4</pre></strong> passed! 🌈</p>

### 1.5

rubric={autograde:1}

How many different kinds of foods are there in the dataset? How many countries are in the dataset? Save your answers as two variables whose type should be `int`.

```python
n_food = df['food_category'].unique().size
n_country = df['country'].unique().size

n_food
```

    11

```python
n_country
```

    130

```python
grader.check("q1_5")
```

<p><strong><pre style='display: inline;'>q1_5</pre></strong> passed! 🎉</p>

### 1.6

rubric={autograde:1}

Sort the dataframe by CO2 emmissions, from biggest to smallest, and get the first 11 rows and the last column as a series. Save this series to a variable called `sorted_co2_emmission`.

```python
sorted_co2_emmission = df.sort_values(by = ['co2_emmission'], ascending=False).iloc[0:11]['co2_emmission']

sorted_co2_emmission
```

    2      1712.00
    90     1211.17
    57     1118.29
    13     1044.85
    123    1022.94
    222     953.51
    189     933.45
    79      922.03
    68      897.96
    233     888.09
    134     878.22
    Name: co2_emmission, dtype: float64

```python
grader.check("q1_6")
```

<p><strong><pre style='display: inline;'>q1_6</pre></strong> passed! 🚀</p>

### 1.7

rubric={autograde:1}

Assume that the `consumption` column represents kilograms. Create a new dataframe named `modified_df` with a new column called `consumption_lbs` which contains the equivalent in pounds.
Note that 1kg = 2.2 pounds.

```python
# make a copy of df to modify
modified_df = df.copy()
modified_df['consumption_lbs'] = modified_df['consumption']*2.2
modified_df.head()
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
      <th>country</th>
      <th>food_category</th>
      <th>consumption</th>
      <th>co2_emmission</th>
      <th>consumption_lbs</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Argentina</td>
      <td>Pork</td>
      <td>10.51</td>
      <td>37.20</td>
      <td>23.122</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Argentina</td>
      <td>Poultry</td>
      <td>38.66</td>
      <td>41.53</td>
      <td>85.052</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Argentina</td>
      <td>Beef</td>
      <td>55.48</td>
      <td>1712.00</td>
      <td>122.056</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Argentina</td>
      <td>Lamb & Goat</td>
      <td>1.56</td>
      <td>54.63</td>
      <td>3.432</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Argentina</td>
      <td>Fish</td>
      <td>4.36</td>
      <td>6.96</td>
      <td>9.592</td>
    </tr>
  </tbody>
</table>
</div>

```python
grader.check("q1_7")
```

<p><strong><pre style='display: inline;'>q1_7</pre></strong> passed! 🍀</p>

### 1.8

rubric={autograde:1}

Find out total consumption in pounds (lbs) for Canada, across all food products. Your answer should be of type `np.float64`. Save the result in a variable called `canada`. *Hint* You might want to set the index to the `country` column.

```python
canada = modified_df.set_index('country').filter(like = 'Canada', axis = 0)['consumption_lbs'].sum()
canada
```

    np.float64(907.6100000000001)

```python
grader.check("q1_8")
```

<p><strong><pre style='display: inline;'>q1_8</pre></strong> passed! 🌈</p>

Congratulations! You are done the worksheet!!! Pat yourself on the back and submit your worksheet to Gradescope!