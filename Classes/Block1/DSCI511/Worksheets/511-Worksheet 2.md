## Worksheet 2 - Renaming and reshaping

```python
import pandas as pd
import numpy as np
from hashlib import sha1
```

## Exercises

In this set of practice exercises we'll be investigating the carbon footprint of different foods. We'll be leveraging a dataset compiled by [Kasia Kulma](https://r-tastic.co.uk/post/from-messy-to-tidy/) and contributed to [R&#39;s Tidy Tuesday project](https://github.com/rfordatascience/tidytuesday). This is the same set we used for Lecture 1.

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

### Q1

rubric={autograde:1}

Create a dataframe to store the countries that produce more than 1000 Kg CO2/person/year for at least one food type. Name the dataframe `df_high_co2`.

```python
df_high_co2 = df[df['co2_emmission'] > 1000]
df_high_co2
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
      <th>2</th>
      <td>Argentina</td>
      <td>Beef</td>
      <td>55.48</td>
      <td>1712.00</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Australia</td>
      <td>Beef</td>
      <td>33.86</td>
      <td>1044.85</td>
    </tr>
    <tr>
      <th>57</th>
      <td>USA</td>
      <td>Beef</td>
      <td>36.24</td>
      <td>1118.29</td>
    </tr>
    <tr>
      <th>90</th>
      <td>Brazil</td>
      <td>Beef</td>
      <td>39.25</td>
      <td>1211.17</td>
    </tr>
    <tr>
      <th>123</th>
      <td>Bermuda</td>
      <td>Beef</td>
      <td>33.15</td>
      <td>1022.94</td>
    </tr>
  </tbody>
</table>
</div>

```python
grader.check("q1")
```

<p><strong><pre style='display: inline;'>q1</pre></strong> passed! 🎉</p>

### Q2

rubric={autograde:1}

Which country consumes the least amount of beef per person per year? Save the answer as a string in a variable called `least_beef`.
*Hint*: This will require multiple steps of filtering, sorting, and locating data.

```python
least_beef = df[df['food_category'] == 'Beef'].sort_values(by = 'co2_emmission').iloc[0].country
least_beef
```

    'Liberia'

```python
grader.check("q2")
```

<p><strong><pre style='display: inline;'>q2</pre></strong> passed! 🍀</p>

### Q3

rubric={autograde:1}

Rename the columns of `df` such that:

- `food_category` becomes `category`
- `co2_emmission` becomes `co2`
- `country` becomes `nation`

When answering this, be sure to modify the original `df`, do not create a new dataframe with a different name.

```python
df = df.rename(columns={'food_category':'category','co2_emmission':'co2', 'country':'nation'})
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
      <th>nation</th>
      <th>category</th>
      <th>consumption</th>
      <th>co2</th>
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
grader.check("q3")
```

<p><strong><pre style='display: inline;'>q3</pre></strong> passed! 🙌</p>

### Q4

rubric={autograde:1}

Make a new DataFrame consisting of all the countries that consume less than 10 kg of rice per year. Save this in a variable named `rice` and sort it from lowest to highest consumption. Don't forget that the name of the food consumption variable was changed in question 3 above.

```python
rice = df[df['category'] == 'Rice'].sort_values(by='consumption')[df['consumption'] < 10]
rice.head()
```

    /tmp/ipykernel_207643/1679354658.py:1: UserWarning: Boolean Series key will be reindexed to match DataFrame index.
      rice = df[df['category'] == 'Rice'].sort_values(by='consumption')[df['consumption'] < 10]

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
      <th>nation</th>
      <th>category</th>
      <th>consumption</th>
      <th>co2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>767</th>
      <td>Poland</td>
      <td>Rice</td>
      <td>0.95</td>
      <td>1.22</td>
    </tr>
    <tr>
      <th>833</th>
      <td>Tunisia</td>
      <td>Rice</td>
      <td>1.07</td>
      <td>1.37</td>
    </tr>
    <tr>
      <th>877</th>
      <td>Morocco</td>
      <td>Rice</td>
      <td>1.10</td>
      <td>1.41</td>
    </tr>
    <tr>
      <th>657</th>
      <td>Serbia</td>
      <td>Rice</td>
      <td>1.22</td>
      <td>1.56</td>
    </tr>
    <tr>
      <th>734</th>
      <td>Bosnia and Herzegovina</td>
      <td>Rice</td>
      <td>1.88</td>
      <td>2.41</td>
    </tr>
  </tbody>
</table>
</div>

```python
grader.check("q4")
```

<p><strong><pre style='display: inline;'>q4</pre></strong> passed! ✨</p>

### Q5

rubric={autograde:1}

Create a Series (one column) of countries that eat over 10kg of either beef or poultry. Save this to a variable called `beef_or_chicken`.

*Hint*: This is a complex condition, so it might be easier to write out with `.query()`.

```python
beef_or_chicken = df[df['category'].isin(['Beef', 'Poultry'])][df['consumption'] > 10]
beef_or_chicken.head()
```

    /tmp/ipykernel_207643/1591264364.py:1: UserWarning: Boolean Series key will be reindexed to match DataFrame index.
      beef_or_chicken = df[df['category'].isin(['Beef', 'Poultry'])][df['consumption'] > 10]

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
      <th>nation</th>
      <th>category</th>
      <th>consumption</th>
      <th>co2</th>
    </tr>
  </thead>
  <tbody>
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
      <th>12</th>
      <td>Australia</td>
      <td>Poultry</td>
      <td>46.12</td>
      <td>49.54</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Australia</td>
      <td>Beef</td>
      <td>33.86</td>
      <td>1044.85</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Albania</td>
      <td>Poultry</td>
      <td>13.23</td>
      <td>14.21</td>
    </tr>
  </tbody>
</table>
</div>

```python
grader.check("q5")
```

<p><strong><pre style='display: inline;'>q5</pre></strong> passed! 🍀</p>

### Q6

rubric={autograde:1}

We're now going to practice tidying data. Remember, to tidy data we need to think about the data and the statistical question we would like to ask about it. Consider this question for the  carbon footprint of different foods data set we have been working on:

*Is there a relationship between the amount of a food type consumed and the $CO_2$ emmission from that food type? And does this differ depending on the country where the food is grown and consumed?*

Considering this question, is version the data below tidy? If not, use the appropriate pandas function to tidy the data so that it is. Save this as a data frame called `df2_tidy`.

> Note: Do not reset the index or modify the column names from your output from `.pivot()` or `.melt()` in this question.

```python
df2 = pd.read_csv('data/food_consumption2.csv')
df2.head()
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
      <th>metrics</th>
      <th>measurements</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Argentina</td>
      <td>Pork</td>
      <td>consumption</td>
      <td>10.51</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Argentina</td>
      <td>Poultry</td>
      <td>consumption</td>
      <td>38.66</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Argentina</td>
      <td>Beef</td>
      <td>consumption</td>
      <td>55.48</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Argentina</td>
      <td>Lamb & Goat</td>
      <td>consumption</td>
      <td>1.56</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Argentina</td>
      <td>Fish</td>
      <td>consumption</td>
      <td>4.36</td>
    </tr>
  </tbody>
</table>
</div>

```python
df2.tail()
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
      <th>metrics</th>
      <th>measurements</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2855</th>
      <td>Bangladesh</td>
      <td>Milk - inc. cheese</td>
      <td>co2_emmission</td>
      <td>31.21</td>
    </tr>
    <tr>
      <th>2856</th>
      <td>Bangladesh</td>
      <td>Wheat and Wheat Products</td>
      <td>co2_emmission</td>
      <td>3.33</td>
    </tr>
    <tr>
      <th>2857</th>
      <td>Bangladesh</td>
      <td>Rice</td>
      <td>co2_emmission</td>
      <td>219.76</td>
    </tr>
    <tr>
      <th>2858</th>
      <td>Bangladesh</td>
      <td>Soybeans</td>
      <td>co2_emmission</td>
      <td>0.27</td>
    </tr>
    <tr>
      <th>2859</th>
      <td>Bangladesh</td>
      <td>Nuts inc. Peanut Butter</td>
      <td>co2_emmission</td>
      <td>1.27</td>
    </tr>
  </tbody>
</table>
</div>

```python
df2_tidy = df2.pivot(index=['country','food_category'], columns=['metrics'], values='measurements')
df2_tidy.head()
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
      <th>metrics</th>
      <th>co2_emmission</th>
      <th>consumption</th>
    </tr>
    <tr>
      <th>country</th>
      <th>food_category</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="5" valign="top">Albania</th>
      <th>Beef</th>
      <td>694.30</td>
      <td>22.50</td>
    </tr>
    <tr>
      <th>Eggs</th>
      <td>11.44</td>
      <td>12.45</td>
    </tr>
    <tr>
      <th>Fish</th>
      <td>6.15</td>
      <td>3.85</td>
    </tr>
    <tr>
      <th>Lamb & Goat</th>
      <td>536.50</td>
      <td>15.32</td>
    </tr>
    <tr>
      <th>Milk - inc. cheese</th>
      <td>432.62</td>
      <td>303.72</td>
    </tr>
  </tbody>
</table>
</div>

```python
grader.check("q6")
```

<p><strong><pre style='display: inline;'>q6</pre></strong> passed! 💯</p>

### Q7

rubric={autograde:1}

When `.pivot()` is called with multiple column names passed to the `index`, those entries become the “name” of each row that would be used when you filter rows with `[]` or `loc` rather than just simple numbers. This can be confusing… Use `.reset_index()` to set `df2_tidy` to have the usual, expected behaviour, where each row is “named” with an integer. This is a subtle point, but the main take-away is that when you call `.pivot()`, it is a good idea to call `.reset_index()` afterwards.

Name your new data frame `df2_tidy_index`.

```python
#df2_tidy_index = df2_tidy
df2_tidy_index = df2_tidy.reset_index()
df2_tidy_index.head()
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
      <th>metrics</th>
      <th>country</th>
      <th>food_category</th>
      <th>co2_emmission</th>
      <th>consumption</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Albania</td>
      <td>Beef</td>
      <td>694.30</td>
      <td>22.50</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Albania</td>
      <td>Eggs</td>
      <td>11.44</td>
      <td>12.45</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Albania</td>
      <td>Fish</td>
      <td>6.15</td>
      <td>3.85</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Albania</td>
      <td>Lamb & Goat</td>
      <td>536.50</td>
      <td>15.32</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Albania</td>
      <td>Milk - inc. cheese</td>
      <td>432.62</td>
      <td>303.72</td>
    </tr>
  </tbody>
</table>
</div>

```python
grader.check("q7")
```

<p><strong><pre style='display: inline;'>q7</pre></strong> passed! 🍀</p>

### Q8

rubric={autograde:1}

When we perform the `.pivot()` operation, it also keeps the original column names and adds the new column name as a second column name. Having two names for a column can be confusing! So we should rename the columns in the data frame so that they only have one name.

Name your new data frame `df2_tidy_index_renamed`.

```python
df2_tidy_index_renamed = df2_tidy_index.copy() # make a copy so as not to modify the object in Q7
df2_tidy_index_renamed = df2_tidy_index
df2_tidy_index_renamed.head()
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
      <th>metrics</th>
      <th>country</th>
      <th>food_category</th>
      <th>co2_emmission</th>
      <th>consumption</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Albania</td>
      <td>Beef</td>
      <td>694.30</td>
      <td>22.50</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Albania</td>
      <td>Eggs</td>
      <td>11.44</td>
      <td>12.45</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Albania</td>
      <td>Fish</td>
      <td>6.15</td>
      <td>3.85</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Albania</td>
      <td>Lamb & Goat</td>
      <td>536.50</td>
      <td>15.32</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Albania</td>
      <td>Milk - inc. cheese</td>
      <td>432.62</td>
      <td>303.72</td>
    </tr>
  </tbody>
</table>
</div>

```python
grader.check("q8")
```

<p><strong><pre style='display: inline;'>q8</pre></strong> passed! 💯</p>

Not sure why but my columns did not need renaming for q8 :|

Congratulations! You are done the worksheet!!! Pat yourself on the back, and submit your worksheet to Gradescope!