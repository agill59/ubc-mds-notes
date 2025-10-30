

# Lecture 3: $k$-Nearest Neighbours and SVM RBFs  

UBC Master of Data Science program, 2025-26

> If two things are similar, the thought of one will tend to trigger the thought of the other <br>
-- Aristotle

## Imports and LOs

### Imports


```python
import sys
import os

import IPython
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
from IPython.display import HTML

sys.path.append(os.path.join(os.path.abspath(".."), "code"))

import ipywidgets as widgets
import mglearn
from IPython.display import display
from ipywidgets import interact, interactive
from plotting_functions import *
from sklearn.dummy import DummyClassifier
from sklearn.model_selection import cross_validate, train_test_split
from utils import *

%matplotlib inline

pd.set_option("display.max_colwidth", 200)
import warnings

warnings.filterwarnings("ignore")
DATA_DIR = DATA_DIR = os.path.join(os.path.abspath(".."), "data/")
```

<br><br>

### Learning outcomes

From this lecture, you will be able to 

- explain the notion of similarity-based algorithms; 
- broadly describe how $k$-NNs use distances; 
- discuss the effect of using a small/large value of the hyperparameter $k$ when using the $k$-NN algorithm; 
- describe the problem of curse of dimensionality; 
- explain the general idea of SVMs with RBF kernel;
- broadly describe the relation of `gamma` and `C` hyperparameters of SVMs with the fundamental tradeoff.

<br><br>

### Quick recap

- Why do we split the data? 
- What are the 4 types of data splits we discussed in the last lecture?
- What are the benefits of cross-validation?
- What is overfitting? 
- What's the fundamental trade-off in supervised machine learning? 
- What is the golden rule of machine learning?

```{important} 
If you want to run this notebook you will have to install `ipywidgets`. 
Follow the installation instructions [here](https://ipywidgets.readthedocs.io/en/latest/user_install.html).
```

## Motivation and distances [[video](https://youtu.be/hCa3EXEUmQk)]

### Analogy-based models

- Suppose you are given the following training examples with corresponding labels and are asked to label a given test example.

- An intuitive way to classify the test example is by finding the most "similar" example(s) from the training set and using that label for the test example.  

### Analogy-based algorithms in practice

- [Herta's High-tech Facial Recognition](https://www.hertasecurity.com/en)
    - Feature vectors for human faces 
    - $k$-NN to identify which face is on their watch list
- Recommendation systems     

### General idea of $k$-nearest neighbours algorithm

- Consider the following toy dataset with two classes.
    - blue circles $\rightarrow$ class 0
    - red triangles $\rightarrow$ class 1 
    - green stars $\rightarrow$ test examples


```python
X, y = mglearn.datasets.make_forge()
X_test = np.array([[8.2, 3.66214339], [9.9, 3.2], [11.2, 0.5]])
```


```python
plot_train_test_points(X, y, X_test)
```


    
![png](03_kNNs-SVM-RBF_files/03_kNNs-SVM-RBF_15_0.png)
    


- Given a new data point, predict the class of the data point by finding the "closest" data point in the training set, i.e., by finding its "nearest neighbour" or majority vote of nearest neighbours. 


```python
import matplotlib
import panel as pn
from panel import widgets
from panel.interact import interact

pn.extension()
```


```python
def f(n_neighbors):
    plt.clf()
    fig = plt.figure(figsize=(6, 4))
    plot_knn_clf(X, y, X_test, n_neighbors=n_neighbors)
    plt.close()
    return pn.pane.Matplotlib(fig, tight=True)


n_neighbors_selector = pn.widgets.IntSlider(
    name="n_neighbors", start=1, end=10, value=1
)
# interact(f, n_neighbors=n_neighbors_selector)
interactive_plot = interact(f, n_neighbors=n_neighbors_selector)#.embed(max_opts=10)
interactive_plot
```

    n_neighbors 1
    


    <Figure size 640x480 with 0 Axes>



    
![png](03_kNNs-SVM-RBF_files/03_kNNs-SVM-RBF_18_2.png)

### Geometric view of tabular data and dimensions 

- To understand analogy-based algorithms it's useful to think of data as points in a high dimensional space. 
- Our `X` represents the problem in terms of relevant **features** ($d$) with one dimension for each **feature** (column).
- Examples are **points in a $d$-dimensional space**. 

How many dimensions (features) are there in the cities data?


```python
cities_df = pd.read_csv(DATA_DIR + "canada_usa_cities.csv")
X_cities = cities_df[["longitude", "latitude"]]
y_cities = cities_df["country"]
```


```python
mglearn.discrete_scatter(X_cities.iloc[:, 0], X_cities.iloc[:, 1], y_cities)
plt.xlabel("longitude")
plt.ylabel("latitude");
```


    
![png](03_kNNs-SVM-RBF_files/03_kNNs-SVM-RBF_22_0.png)
    


- Recall the [Spotify Song Attributes](https://www.kaggle.com/geomack/spotifyclassification/home) dataset from homework 2. 
- How many dimensions (features) we used in the homework? 


```python
spotify_df = pd.read_csv(DATA_DIR + "spotify.csv", index_col=0)
X_spotify = spotify_df.drop(columns=["target", "song_title", "artist"])
print("The number of features in the Spotify dataset: %d" % X_spotify.shape[1])
X_spotify.head()
```

    The number of features in the Spotify dataset: 13
    




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
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>acousticness</th>
      <th>danceability</th>
      <th>duration_ms</th>
      <th>energy</th>
      <th>instrumentalness</th>
      <th>key</th>
      <th>liveness</th>
      <th>loudness</th>
      <th>mode</th>
      <th>speechiness</th>
      <th>tempo</th>
      <th>time_signature</th>
      <th>valence</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.0102</td>
      <td>0.833</td>
      <td>204600</td>
      <td>0.434</td>
      <td>0.021900</td>
      <td>2</td>
      <td>0.1650</td>
      <td>-8.795</td>
      <td>1</td>
      <td>0.4310</td>
      <td>150.062</td>
      <td>4.0</td>
      <td>0.286</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.1990</td>
      <td>0.743</td>
      <td>326933</td>
      <td>0.359</td>
      <td>0.006110</td>
      <td>1</td>
      <td>0.1370</td>
      <td>-10.401</td>
      <td>1</td>
      <td>0.0794</td>
      <td>160.083</td>
      <td>4.0</td>
      <td>0.588</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.0344</td>
      <td>0.838</td>
      <td>185707</td>
      <td>0.412</td>
      <td>0.000234</td>
      <td>2</td>
      <td>0.1590</td>
      <td>-7.148</td>
      <td>1</td>
      <td>0.2890</td>
      <td>75.044</td>
      <td>4.0</td>
      <td>0.173</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.6040</td>
      <td>0.494</td>
      <td>199413</td>
      <td>0.338</td>
      <td>0.510000</td>
      <td>5</td>
      <td>0.0922</td>
      <td>-15.236</td>
      <td>1</td>
      <td>0.0261</td>
      <td>86.468</td>
      <td>4.0</td>
      <td>0.230</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.1800</td>
      <td>0.678</td>
      <td>392893</td>
      <td>0.561</td>
      <td>0.512000</td>
      <td>5</td>
      <td>0.4390</td>
      <td>-11.648</td>
      <td>0</td>
      <td>0.0694</td>
      <td>174.004</td>
      <td>4.0</td>
      <td>0.904</td>
    </tr>
  </tbody>
</table>
</div>



### Dimensions in ML problems 

In ML, usually we deal with high dimensional problems where examples are hard to visualize.  

- $d \approx 20$ is considered low dimensional
- $d \approx 1000$ is considered medium dimensional 
- $d \approx 100,000$ is considered high dimensional 

### Feature vectors 

**Feature vector**
: is composed of feature values associated with an example.

Some example feature vectors are shown below. 


```python
print(
    "An example feature vector from the cities dataset: %s"
    % (X_cities.iloc[0].to_numpy())
)
print(
    "An example feature vector from the Spotify dataset: \n%s"
    % (X_spotify.iloc[0].to_numpy())
)
```

    An example feature vector from the cities dataset: [-130.0437   55.9773]
    An example feature vector from the Spotify dataset: 
    [ 1.02000e-02  8.33000e-01  2.04600e+05  4.34000e-01  2.19000e-02
      2.00000e+00  1.65000e-01 -8.79500e+00  1.00000e+00  4.31000e-01
      1.50062e+02  4.00000e+00  2.86000e-01]
    

### Similarity between examples

Let's take 2 points (two feature vectors) from the cities dataset.


```python
two_cities = X_cities.sample(2, random_state=120)
two_cities
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
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>longitude</th>
      <th>latitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>69</th>
      <td>-104.8253</td>
      <td>38.8340</td>
    </tr>
    <tr>
      <th>35</th>
      <td>-112.0741</td>
      <td>33.4484</td>
    </tr>
  </tbody>
</table>
</div>



The two sampled points are shown as big black circles.


```python
mglearn.discrete_scatter(
    X_cities.iloc[:, 0], X_cities.iloc[:, 1], y_cities, s=8, alpha=0.3
)
mglearn.discrete_scatter(
    two_cities.iloc[:, 0], two_cities.iloc[:, 1], markers="o", c="k", s=18
);
```


    
![png](03_kNNs-SVM-RBF_files/03_kNNs-SVM-RBF_31_0.png)
    


### Distance between feature vectors 

- For the cities at the two big circles, what is the _distance_ between them?
- A common way to calculate the distance between vectors is calculating the **Euclidean distance**. 
- The euclidean distance between vectors $u = <u_1, u_2, \dots, u_n>$ and $v = <v_1, v_2, \dots, v_n>$ is defined as: 

$$distance(u, v) = \sqrt{\sum_{i =1}^{n} (u_i - v_i)^2}$$ 


### Euclidean distance 


```python
two_cities
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
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>longitude</th>
      <th>latitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>69</th>
      <td>-104.8253</td>
      <td>38.8340</td>
    </tr>
    <tr>
      <th>35</th>
      <td>-112.0741</td>
      <td>33.4484</td>
    </tr>
  </tbody>
</table>
</div>



- Subtract the two cities
- Square the difference
- Sum them up 
- Take the square root 


```python
# Subtract the two cities
print("Subtract the cities: \n%s\n" % (two_cities.iloc[1] - two_cities.iloc[0]))

# Squared sum of the difference
print(
    "Sum of squares: %0.4f" % (np.sum((two_cities.iloc[1] - two_cities.iloc[0]) ** 2))
)

# Take the square root
print(
    "Euclidean distance between cities: %0.4f"
    % (np.sqrt(np.sum((two_cities.iloc[1] - two_cities.iloc[0]) ** 2)))
)
```

    Subtract the cities: 
    longitude   -7.2488
    latitude    -5.3856
    dtype: float64
    
    Sum of squares: 81.5498
    Euclidean distance between cities: 9.0305
    


```python
two_cities
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
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>longitude</th>
      <th>latitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>69</th>
      <td>-104.8253</td>
      <td>38.8340</td>
    </tr>
    <tr>
      <th>35</th>
      <td>-112.0741</td>
      <td>33.4484</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Euclidean distance using sklearn
from sklearn.metrics.pairwise import euclidean_distances

euclidean_distances(two_cities)
```




    array([[0.        , 9.03049217],
           [9.03049217, 0.        ]])



Note: `scikit-learn` supports a number of other [distance metrics](https://scikit-learn.org/stable/modules/generated/sklearn.neighbors.DistanceMetric.html).


### Finding the nearest neighbour

- Let's look at distances from all cities to all other cities


```python
dists = euclidean_distances(X_cities)
np.fill_diagonal(dists, np.inf)
dists.shape
```




    (209, 209)




```python
pd.DataFrame(dists)
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
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>...</th>
      <th>199</th>
      <th>200</th>
      <th>201</th>
      <th>202</th>
      <th>203</th>
      <th>204</th>
      <th>205</th>
      <th>206</th>
      <th>207</th>
      <th>208</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>inf</td>
      <td>4.955113</td>
      <td>9.869531</td>
      <td>10.106452</td>
      <td>10.449666</td>
      <td>19.381676</td>
      <td>28.366626</td>
      <td>33.283857</td>
      <td>33.572105</td>
      <td>36.180388</td>
      <td>...</td>
      <td>9.834455</td>
      <td>58.807684</td>
      <td>16.925593</td>
      <td>56.951696</td>
      <td>59.384127</td>
      <td>58.289799</td>
      <td>64.183423</td>
      <td>52.426410</td>
      <td>58.033459</td>
      <td>51.498562</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4.955113</td>
      <td>inf</td>
      <td>14.677579</td>
      <td>14.935802</td>
      <td>15.305346</td>
      <td>24.308448</td>
      <td>33.200978</td>
      <td>38.082949</td>
      <td>38.359992</td>
      <td>40.957919</td>
      <td>...</td>
      <td>14.668787</td>
      <td>63.533498</td>
      <td>21.656349</td>
      <td>61.691640</td>
      <td>64.045304</td>
      <td>63.032656</td>
      <td>68.887343</td>
      <td>57.253724</td>
      <td>62.771969</td>
      <td>56.252160</td>
    </tr>
    <tr>
      <th>2</th>
      <td>9.869531</td>
      <td>14.677579</td>
      <td>inf</td>
      <td>0.334411</td>
      <td>0.808958</td>
      <td>11.115406</td>
      <td>20.528403</td>
      <td>25.525757</td>
      <td>25.873103</td>
      <td>28.479109</td>
      <td>...</td>
      <td>0.277381</td>
      <td>51.076798</td>
      <td>10.783789</td>
      <td>49.169693</td>
      <td>51.934205</td>
      <td>50.483751</td>
      <td>56.512897</td>
      <td>44.235152</td>
      <td>50.249720</td>
      <td>43.699224</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10.106452</td>
      <td>14.935802</td>
      <td>0.334411</td>
      <td>inf</td>
      <td>0.474552</td>
      <td>10.781004</td>
      <td>20.194002</td>
      <td>25.191396</td>
      <td>25.538702</td>
      <td>28.144750</td>
      <td>...</td>
      <td>0.275352</td>
      <td>50.743133</td>
      <td>10.480249</td>
      <td>48.836189</td>
      <td>51.599860</td>
      <td>50.150395</td>
      <td>56.179123</td>
      <td>43.904226</td>
      <td>49.916254</td>
      <td>43.365623</td>
    </tr>
    <tr>
      <th>4</th>
      <td>10.449666</td>
      <td>15.305346</td>
      <td>0.808958</td>
      <td>0.474552</td>
      <td>inf</td>
      <td>10.306500</td>
      <td>19.719500</td>
      <td>24.716985</td>
      <td>25.064200</td>
      <td>27.670344</td>
      <td>...</td>
      <td>0.675814</td>
      <td>50.269880</td>
      <td>10.051472</td>
      <td>48.363192</td>
      <td>51.125476</td>
      <td>49.677629</td>
      <td>55.705696</td>
      <td>43.435186</td>
      <td>49.443317</td>
      <td>42.892477</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>204</th>
      <td>58.289799</td>
      <td>63.032656</td>
      <td>50.483751</td>
      <td>50.150395</td>
      <td>49.677629</td>
      <td>39.405415</td>
      <td>30.043890</td>
      <td>25.057003</td>
      <td>24.746328</td>
      <td>22.127878</td>
      <td>...</td>
      <td>50.333340</td>
      <td>0.873356</td>
      <td>41.380643</td>
      <td>1.345136</td>
      <td>3.373031</td>
      <td>inf</td>
      <td>6.102435</td>
      <td>6.957987</td>
      <td>0.316363</td>
      <td>6.800190</td>
    </tr>
    <tr>
      <th>205</th>
      <td>64.183423</td>
      <td>68.887343</td>
      <td>56.512897</td>
      <td>56.179123</td>
      <td>55.705696</td>
      <td>45.418031</td>
      <td>36.031385</td>
      <td>31.032874</td>
      <td>30.709185</td>
      <td>28.088948</td>
      <td>...</td>
      <td>56.358333</td>
      <td>5.442806</td>
      <td>47.259286</td>
      <td>7.369875</td>
      <td>5.108681</td>
      <td>6.102435</td>
      <td>inf</td>
      <td>12.950733</td>
      <td>6.303916</td>
      <td>12.819584</td>
    </tr>
    <tr>
      <th>206</th>
      <td>52.426410</td>
      <td>57.253724</td>
      <td>44.235152</td>
      <td>43.904226</td>
      <td>43.435186</td>
      <td>33.258427</td>
      <td>24.059863</td>
      <td>19.187663</td>
      <td>18.932124</td>
      <td>16.380495</td>
      <td>...</td>
      <td>44.100248</td>
      <td>7.767852</td>
      <td>35.637982</td>
      <td>5.930561</td>
      <td>9.731583</td>
      <td>6.957987</td>
      <td>12.950733</td>
      <td>inf</td>
      <td>6.837848</td>
      <td>3.322755</td>
    </tr>
    <tr>
      <th>207</th>
      <td>58.033459</td>
      <td>62.771969</td>
      <td>50.249720</td>
      <td>49.916254</td>
      <td>49.443317</td>
      <td>39.167214</td>
      <td>29.799983</td>
      <td>24.810368</td>
      <td>24.497386</td>
      <td>21.878183</td>
      <td>...</td>
      <td>50.098326</td>
      <td>0.930123</td>
      <td>41.121628</td>
      <td>1.082749</td>
      <td>3.286821</td>
      <td>0.316363</td>
      <td>6.303916</td>
      <td>6.837848</td>
      <td>inf</td>
      <td>6.555740</td>
    </tr>
    <tr>
      <th>208</th>
      <td>51.498562</td>
      <td>56.252160</td>
      <td>43.699224</td>
      <td>43.365623</td>
      <td>42.892477</td>
      <td>32.612755</td>
      <td>23.244592</td>
      <td>18.256813</td>
      <td>17.946783</td>
      <td>15.328953</td>
      <td>...</td>
      <td>43.546610</td>
      <td>7.378764</td>
      <td>34.596810</td>
      <td>5.473691</td>
      <td>8.568009</td>
      <td>6.800190</td>
      <td>12.819584</td>
      <td>3.322755</td>
      <td>6.555740</td>
      <td>inf</td>
    </tr>
  </tbody>
</table>
<p>209 rows × 209 columns</p>
</div>



Let's look at the distances between City 0 and some other cities. 


```python
print("Feature vector for city 0: \n%s\n" % (X_cities.iloc[0]))
print("Distances from city 0 to the first 5 cities: %s" % (dists[0][:5]))
# We can find the closest city with `np.argmin`:
print(
    "The closest city from city 0 is: %d \n\nwith feature vector: \n%s"
    % (np.argmin(dists[0]), X_cities.iloc[np.argmin(dists[0])])
)
```

    Feature vector for city 0: 
    longitude   -130.0437
    latitude      55.9773
    Name: 0, dtype: float64
    
    Distances from city 0 to the first 5 cities: [        inf  4.95511263  9.869531   10.10645223 10.44966612]
    The closest city from city 0 is: 81 
    
    with feature vector: 
    longitude   -129.9912
    latitude      55.9383
    Name: 81, dtype: float64
    

Ok, so the closest city to City 0 is City 81. 

### Question

- Why did we set the diagonal entries to infinity before finding the closest city?

### Finding the distances to a query point

We can also find the distances to a new "test" or "query" city:


```python
# Let's find a city that's closest to the a query city
query_point = [[-80, 25]]

dists = euclidean_distances(X_cities, query_point)
dists[0:10]
```




    array([[58.85545875],
           [63.80062924],
           [49.30530902],
           [49.01473536],
           [48.60495488],
           [39.96834506],
           [32.92852376],
           [29.53520104],
           [29.52881619],
           [27.84679073]])




```python
# The query point is closest to
print(
    "The query point %s is closest to the city with index %d and the distance between them is: %0.4f"
    % (query_point, np.argmin(dists), dists[np.argmin(dists)])
)
```

    The query point [[-80, 25]] is closest to the city with index 72 and the distance between them is: 0.7982
    

<br><br>

## $k$-Nearest Neighbours ($k$-NNs) [[video](https://youtu.be/bENDqXKJLmg)]


```python
small_cities = cities_df.sample(30, random_state=90)
one_city = small_cities.sample(1, random_state=44)
small_train_df = pd.concat([small_cities, one_city]).drop_duplicates(keep=False)
```


```python
X_small_cities = small_train_df.drop(columns=["country"]).to_numpy()
y_small_cities = small_train_df["country"].to_numpy()
test_point = one_city[["longitude", "latitude"]].to_numpy()
```


```python
plot_train_test_points(
    X_small_cities,
    y_small_cities,
    test_point,
    class_names=["Canada", "USA"],
    test_format="circle",
)
```


    
![png](03_kNNs-SVM-RBF_files/03_kNNs-SVM-RBF_54_0.png)
    


- Given a new data point, predict the class of the data point by finding the "closest" data point in the training set, i.e., by finding its "nearest neighbour" or majority vote of nearest neighbours. 

Suppose we want to predict the class of the black point.  
- An intuitive way to do this is predict the same label as the "closest" point ($k = 1$) (1-nearest neighbour)
- We would predict a target of **USA** in this case.


```python
plot_knn_clf(
    X_small_cities,
    y_small_cities,
    test_point,
    n_neighbors=1,
    class_names=["Canada", "USA"],
    test_format="circle",
)
```

    n_neighbors 1
    


    
![png](03_kNNs-SVM-RBF_files/03_kNNs-SVM-RBF_57_1.png)
    


How about using $k > 1$ to get a more robust estimate? 
- For example, we could also use the 3 closest points (*k* = 3) and let them **vote** on the correct class.  
- The **Canada** class would win in this case. 


```python
plot_knn_clf(
    X_small_cities,
    y_small_cities,
    test_point,
    n_neighbors=3,
    class_names=["Canada", "USA"],
    test_format="circle",
)
```

    n_neighbors 3
    


    
![png](03_kNNs-SVM-RBF_files/03_kNNs-SVM-RBF_59_1.png)
    



```python
from sklearn.neighbors import KNeighborsClassifier

k_values = [1, 3]

for k in k_values:
    neigh = KNeighborsClassifier(n_neighbors=k)
    neigh.fit(X_small_cities, y_small_cities)
    print(
        "Prediction of the black dot with %d neighbours: %s"
        % (k, neigh.predict(test_point))
    )
```

    Prediction of the black dot with 1 neighbours: ['USA']
    Prediction of the black dot with 3 neighbours: ['Canada']
    



### Choosing `n_neighbors`

- The primary hyperparameter of the model is `n_neighbors` ($k$) which decides how many neighbours should vote during prediction? 
- What happens when we play around with `n_neighbors`?
- Are we more likely to overfit with a low `n_neighbors` or a high `n_neighbors`?
- Let's examine the effect of the hyperparameter on our cities data. 


```python
X = cities_df.drop(columns=["country"])
y = cities_df["country"]

# split into train and test sets
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.1, random_state=123
)
```


```python
k = 1
knn1 = KNeighborsClassifier(n_neighbors=k)
scores = cross_validate(knn1, X_train, y_train, return_train_score=True)
pd.DataFrame(scores)
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
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>fit_time</th>
      <th>score_time</th>
      <th>test_score</th>
      <th>train_score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.001182</td>
      <td>0.001227</td>
      <td>0.710526</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.000611</td>
      <td>0.000877</td>
      <td>0.684211</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.000725</td>
      <td>0.001134</td>
      <td>0.842105</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.000623</td>
      <td>0.001080</td>
      <td>0.702703</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.000598</td>
      <td>0.000923</td>
      <td>0.837838</td>
      <td>1.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
k = 100
knn100 = KNeighborsClassifier(n_neighbors=k)
scores = cross_validate(knn100, X_train, y_train, return_train_score=True)
pd.DataFrame(scores)
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
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>fit_time</th>
      <th>score_time</th>
      <th>test_score</th>
      <th>train_score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.000868</td>
      <td>0.044757</td>
      <td>0.605263</td>
      <td>0.600000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.000676</td>
      <td>0.001388</td>
      <td>0.605263</td>
      <td>0.600000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.000692</td>
      <td>0.001583</td>
      <td>0.605263</td>
      <td>0.600000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.000609</td>
      <td>0.001874</td>
      <td>0.594595</td>
      <td>0.602649</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.000510</td>
      <td>0.001290</td>
      <td>0.594595</td>
      <td>0.602649</td>
    </tr>
  </tbody>
</table>
</div>




```python
plot_knn_decision_boundaries(X_train, y_train, k_values=[1, 11, 100])
```


    
![png](03_kNNs-SVM-RBF_files/03_kNNs-SVM-RBF_66_0.png)
    


### How to choose `n_neighbors`?

- `n_neighbors` is a hyperparameter
- We can use hyperparameter optimization to choose `n_neighbors`.


```python
results_dict = {
    "n_neighbors": [],
    "mean_train_score": [],
    "mean_cv_score": [],
    "std_cv_score": [],
    "std_train_score": [],
}
param_grid = {"n_neighbors": np.arange(1, 50, 5)}

for k in param_grid["n_neighbors"]:
    knn = KNeighborsClassifier(n_neighbors=k)
    scores = cross_validate(knn, X_train, y_train, return_train_score=True)
    results_dict["n_neighbors"].append(k)

    results_dict["mean_cv_score"].append(np.mean(scores["test_score"]))
    results_dict["mean_train_score"].append(np.mean(scores["train_score"]))
    results_dict["std_cv_score"].append(scores["test_score"].std())
    results_dict["std_train_score"].append(scores["train_score"].std())

results_df = pd.DataFrame(results_dict)
```


```python
results_df = results_df.set_index("n_neighbors")
results_df
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
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>mean_train_score</th>
      <th>mean_cv_score</th>
      <th>std_cv_score</th>
      <th>std_train_score</th>
    </tr>
    <tr>
      <th>n_neighbors</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>1.000000</td>
      <td>0.755477</td>
      <td>0.069530</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0.831135</td>
      <td>0.792603</td>
      <td>0.046020</td>
      <td>0.013433</td>
    </tr>
    <tr>
      <th>11</th>
      <td>0.819152</td>
      <td>0.802987</td>
      <td>0.041129</td>
      <td>0.011336</td>
    </tr>
    <tr>
      <th>16</th>
      <td>0.801863</td>
      <td>0.782219</td>
      <td>0.074141</td>
      <td>0.008735</td>
    </tr>
    <tr>
      <th>21</th>
      <td>0.777934</td>
      <td>0.766430</td>
      <td>0.062792</td>
      <td>0.016944</td>
    </tr>
    <tr>
      <th>26</th>
      <td>0.755364</td>
      <td>0.723613</td>
      <td>0.061937</td>
      <td>0.025910</td>
    </tr>
    <tr>
      <th>31</th>
      <td>0.743391</td>
      <td>0.707681</td>
      <td>0.057646</td>
      <td>0.030408</td>
    </tr>
    <tr>
      <th>36</th>
      <td>0.728777</td>
      <td>0.707681</td>
      <td>0.064452</td>
      <td>0.021305</td>
    </tr>
    <tr>
      <th>41</th>
      <td>0.706128</td>
      <td>0.681223</td>
      <td>0.061241</td>
      <td>0.018310</td>
    </tr>
    <tr>
      <th>46</th>
      <td>0.694155</td>
      <td>0.660171</td>
      <td>0.093390</td>
      <td>0.018178</td>
    </tr>
  </tbody>
</table>
</div>




```python
results_df[["mean_train_score", "mean_cv_score"]].plot();
```


    
![png](03_kNNs-SVM-RBF_files/03_kNNs-SVM-RBF_70_0.png)
    



```python
best_n_neighbours = results_df.idxmax()["mean_cv_score"]
best_n_neighbours
```




    np.int64(11)



Let's try our best model on test data. 


```python
knn = KNeighborsClassifier(n_neighbors=best_n_neighbours)
knn.fit(X_train, y_train)
print("Test accuracy: %0.3f" % (knn.score(X_test, y_test)))
```

    Test accuracy: 0.905
    

Seems like we got lucky with the test set here.

<br><br>

## ❓❓ Questions for you 

### (iClicker) Exercise 3.1 

**Select all of the following statements which are TRUE.**

- (A) Analogy-based models find examples from the test set that are most similar to the query example we are predicting.
- (B) Euclidean distance will always have a non-negative value.
- (C) With $k$-NN, setting the hyperparameter $k$ to larger values typically reduces training error. 
- (D) Similar to decision trees, $k$-NNs finds a small set of good features.
- (E) In $k$-NN, with $k > 1$, the classification of the closest neighbour to the test example always contributes the most to the prediction.

```{admonition} V's Solutions!
:class: tip, dropdown
```

## Break (5 min)

![](../img/eva-coffee.png)


<br><br>

## More on $k$-NNs [[video](https://youtu.be/IRGbqi5S9gQ)]

### Other useful arguments of `KNeighborsClassifier`

- `weights` $\rightarrow$ When predicting label, you can assign higher weight to the examples which are closer to the query example.  
- Exercise for you: Play around with this argument. Do you get a better validation score? 

### Regression with $k$-nearest neighbours ($k$-NNs)

- Can we solve regression problems with $k$-nearest neighbours algorithm? 
- In $k$-NN regression we take the average of the $k$-nearest neighbours. 
- We can also have weighted regression. 

See an example of regression in the lecture notes. 


```python
mglearn.plots.plot_knn_regression(n_neighbors=1)
```


    
![png](03_kNNs-SVM-RBF_files/03_kNNs-SVM-RBF_84_0.png)
    



```python
mglearn.plots.plot_knn_regression(n_neighbors=3)
```


    
![png](03_kNNs-SVM-RBF_files/03_kNNs-SVM-RBF_85_0.png)
    


### Pros of $k$-NNs for supervised learning

- Easy to understand, interpret.
- Simple hyperparameter $k$ (`n_neighbors`) controlling the fundamental tradeoff.
- Can learn very complex functions given enough data.
- Lazy learning: Takes no time to `fit`

### Cons of $k$-NNs for supervised learning

- Can be potentially be VERY slow during prediction time, especially when the training set is very large. 
- Often not that great test accuracy compared to the modern approaches.
- It does not work well on datasets with many features or where most feature values are 0 most of the time (sparse datasets).    


For regular $k$-NN for supervised learning (not with sparse matrices), you should scale your features. We'll be looking into it soon. 

```
		
### (Optional) Parametric vs non parametric 

- You might see a lot of definitions of these terms.
- A simple way to think about this is: 
    - do you need to store at least $O(n)$ worth of stuff to make predictions? If so, it's non-parametric.
- Non-parametric example: $k$-NN is a classic example of non-parametric models.     
- Parametric example: decision stump
- If you want to know more about this, find some reading material [here](https://www.cs.ubc.ca/~schmidtm/Courses/340-F16/L6.pdf), [here](http://mlss.tuebingen.mpg.de/2015/slides/ghahramani/gp-neural-nets15.pdf), and [here](https://machinelearningmastery.com/parametric-and-nonparametric-machine-learning-algorithms/). 
- By the way, the terms "parametric" and "non-paramteric" are often used differently by statisticians, see [here](https://help.xlstat.com/s/article/what-is-the-difference-between-a-parametric-and-a-nonparametric-test?language=en_US) for more...

```{note}
$\mathcal{O}(n)$ is referred to as big $\mathcal{O}$ notation. It tells you how fast an algorithm is or how much storage space it requires. For example, in simple terms, if you have $n$ examples and you need to store them all you can say that the algorithm requires $\mathcal{O}(n)$ worth of stuff. 
```

### Curse of dimensionality

- Affects all learners but especially bad for nearest-neighbour. 
- $k$-NN usually works well when the number of dimensions $d$ is small but things fall apart quickly as $d$ goes up.
- If there are many irrelevant attributes, $k$-NN is hopelessly confused because all of them contribute to finding similarity between examples. 
- With enough irrelevant attributes the accidental similarity swamps out meaningful similarity and $k$-NN is no better than random guessing.  


```python
from sklearn.datasets import make_classification

nfeats_accuracy = {"nfeats": [], "dummy_valid_accuracy": [], "KNN_valid_accuracy": []}
for n_feats in range(4, 2000, 100):
    X, y = make_classification(n_samples=2000, n_features=n_feats, n_classes=2)
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=123
    )
    dummy = DummyClassifier(strategy="most_frequent")
    dummy_scores = cross_validate(dummy, X_train, y_train, return_train_score=True)

    knn = KNeighborsClassifier()
    scores = cross_validate(knn, X_train, y_train, return_train_score=True)
    nfeats_accuracy["nfeats"].append(n_feats)
    nfeats_accuracy["KNN_valid_accuracy"].append(np.mean(scores["test_score"]))
    nfeats_accuracy["dummy_valid_accuracy"].append(np.mean(dummy_scores["test_score"]))
```


```python
pd.DataFrame(nfeats_accuracy)
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
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>nfeats</th>
      <th>dummy_valid_accuracy</th>
      <th>KNN_valid_accuracy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>4</td>
      <td>0.503750</td>
      <td>0.975625</td>
    </tr>
    <tr>
      <th>1</th>
      <td>104</td>
      <td>0.506875</td>
      <td>0.761250</td>
    </tr>
    <tr>
      <th>2</th>
      <td>204</td>
      <td>0.504375</td>
      <td>0.681250</td>
    </tr>
    <tr>
      <th>3</th>
      <td>304</td>
      <td>0.504375</td>
      <td>0.678750</td>
    </tr>
    <tr>
      <th>4</th>
      <td>404</td>
      <td>0.503750</td>
      <td>0.605625</td>
    </tr>
    <tr>
      <th>5</th>
      <td>504</td>
      <td>0.500000</td>
      <td>0.702500</td>
    </tr>
    <tr>
      <th>6</th>
      <td>604</td>
      <td>0.500000</td>
      <td>0.631875</td>
    </tr>
    <tr>
      <th>7</th>
      <td>704</td>
      <td>0.502500</td>
      <td>0.680625</td>
    </tr>
    <tr>
      <th>8</th>
      <td>804</td>
      <td>0.505000</td>
      <td>0.568750</td>
    </tr>
    <tr>
      <th>9</th>
      <td>904</td>
      <td>0.505625</td>
      <td>0.605000</td>
    </tr>
    <tr>
      <th>10</th>
      <td>1004</td>
      <td>0.503125</td>
      <td>0.565000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>1104</td>
      <td>0.500000</td>
      <td>0.588750</td>
    </tr>
    <tr>
      <th>12</th>
      <td>1204</td>
      <td>0.506250</td>
      <td>0.567500</td>
    </tr>
    <tr>
      <th>13</th>
      <td>1304</td>
      <td>0.507500</td>
      <td>0.597500</td>
    </tr>
    <tr>
      <th>14</th>
      <td>1404</td>
      <td>0.506250</td>
      <td>0.616875</td>
    </tr>
    <tr>
      <th>15</th>
      <td>1504</td>
      <td>0.504375</td>
      <td>0.542500</td>
    </tr>
    <tr>
      <th>16</th>
      <td>1604</td>
      <td>0.504375</td>
      <td>0.622500</td>
    </tr>
    <tr>
      <th>17</th>
      <td>1704</td>
      <td>0.510625</td>
      <td>0.564375</td>
    </tr>
    <tr>
      <th>18</th>
      <td>1804</td>
      <td>0.502500</td>
      <td>0.565625</td>
    </tr>
    <tr>
      <th>19</th>
      <td>1904</td>
      <td>0.502500</td>
      <td>0.586250</td>
    </tr>
  </tbody>
</table>
</div>



<br><br>

## Support Vector Machines (SVMs) with RBF kernel [[video](https://youtu.be/ic_zqOhi020)]

- Very high-level overview
- Our goals here are
    - Use `scikit-learn`'s SVM model. 
    - Broadly explain the notion of support vectors.  
    - Broadly explain the similarities and differences between $k$-NNs and SVM RBFs.
    - Explain how `C` and `gamma` hyperparameters control the fundamental tradeoff.
    
> (Optional) RBF stands for radial basis functions. We won't go into what it means in this video. Refer to [this video](https://www.youtube.com/watch?v=Qc5IyLW_hns) if you want to know more. 

### Overview

- Another popular similarity-based algorithm is Support Vector Machines with RBF Kernel (SVM RBFs)
- Superficially, SVM RBFs are more like weighted $k$-NNs.
    - The decision boundary is defined by **a set of positive and negative examples** and **their weights** together with **their similarity measure**. 
    - A test example is labeled positive if on average it looks more like positive examples than the negative examples. 

- The primary difference between $k$-NNs and SVM RBFs is that 
    - Unlike $k$-NNs, SVM RBFs only remember the key examples (support vectors). 
    - SVMs use a different similarity metric which is called a "kernel". A popular kernel is Radial Basis Functions (RBFs)
    - They usually perform better than $k$-NNs! 

### Let's explore SVM RBFs

Let's try SVMs on the cities dataset. 


```python
mglearn.discrete_scatter(X_cities.iloc[:, 0], X_cities.iloc[:, 1], y_cities)
plt.xlabel("longitude")
plt.ylabel("latitude")
plt.legend(loc=1);
```


    
![png](03_kNNs-SVM-RBF_files/03_kNNs-SVM-RBF_99_0.png)
    



```python
X_train, X_test, y_train, y_test = train_test_split(
    X_cities, y_cities, test_size=0.2, random_state=123
)
```


```python
knn = KNeighborsClassifier(n_neighbors=best_n_neighbours)
scores = cross_validate(knn, X_train, y_train, return_train_score=True)
print("Mean validation score %0.3f" % (np.mean(scores["test_score"])))
pd.DataFrame(scores)
```

    Mean validation score 0.803
    




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
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>fit_time</th>
      <th>score_time</th>
      <th>test_score</th>
      <th>train_score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.000922</td>
      <td>0.001480</td>
      <td>0.794118</td>
      <td>0.819549</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.000678</td>
      <td>0.000971</td>
      <td>0.764706</td>
      <td>0.819549</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.000598</td>
      <td>0.000932</td>
      <td>0.727273</td>
      <td>0.850746</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.000578</td>
      <td>0.000859</td>
      <td>0.787879</td>
      <td>0.828358</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.000528</td>
      <td>0.001072</td>
      <td>0.939394</td>
      <td>0.783582</td>
    </tr>
  </tbody>
</table>
</div>




```python
from sklearn.svm import SVC

svm = SVC(gamma=0.01)  # Ignore gamma for now
scores = cross_validate(svm, X_train, y_train, return_train_score=True)
print("Mean validation score %0.3f" % (np.mean(scores["test_score"])))
pd.DataFrame(scores)
```

    Mean validation score 0.820
    




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
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>fit_time</th>
      <th>score_time</th>
      <th>test_score</th>
      <th>train_score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.002194</td>
      <td>0.001628</td>
      <td>0.823529</td>
      <td>0.842105</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.000974</td>
      <td>0.000634</td>
      <td>0.823529</td>
      <td>0.842105</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.000873</td>
      <td>0.000593</td>
      <td>0.727273</td>
      <td>0.858209</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.000932</td>
      <td>0.000614</td>
      <td>0.787879</td>
      <td>0.843284</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.000914</td>
      <td>0.000648</td>
      <td>0.939394</td>
      <td>0.805970</td>
    </tr>
  </tbody>
</table>
</div>



### Decision boundary of SVMs 
- We can think of SVM with RBF kernel as "smooth KNN". 


```python
fig, axes = plt.subplots(1, 2, figsize=(16, 5))

for clf, ax in zip([knn, svm], axes):
    clf.fit(X_train, y_train)
    mglearn.plots.plot_2d_separator(
        clf, X_train.to_numpy(), fill=True, eps=0.5, ax=ax, alpha=0.4
    )
    mglearn.discrete_scatter(X_train.iloc[:, 0], X_train.iloc[:, 1], y_train, ax=ax)
    ax.set_title(clf)
    ax.set_xlabel("longitude")
    ax.set_ylabel("latitude")
```


    
![png](03_kNNs-SVM-RBF_files/03_kNNs-SVM-RBF_104_0.png)
    


### Support vectors 

- Each training example either is or isn't a "support vector".
  - This gets decided during `fit`.

- **Main insight: the decision boundary only depends on the support vectors.**

- Let's look at the support vectors. 


```python
from sklearn.datasets import make_blobs

n = 20
n_classes = 2
X_toy, y_toy = make_blobs(
    n_samples=n, centers=n_classes, random_state=300
)  # Let's generate some fake data
```


```python
mglearn.discrete_scatter(X_toy[:, 0], X_toy[:, 1], y_toy)
plt.xlabel("Feature 0")
plt.ylabel("Feature 1")
svm = SVC(kernel="rbf", C=10, gamma=0.1).fit(X_toy, y_toy)
mglearn.plots.plot_2d_separator(svm, X_toy, fill=True, eps=0.5, alpha=0.4)
```


    
![png](03_kNNs-SVM-RBF_files/03_kNNs-SVM-RBF_107_0.png)
    



```python
svm.support_
```




    array([ 3,  8,  9, 14, 19,  1,  4,  6, 17], dtype=int32)




```python
plot_support_vectors(svm, X_toy, y_toy)
```


    
![png](03_kNNs-SVM-RBF_files/03_kNNs-SVM-RBF_109_0.png)
    


The support vectors are the bigger points in the plot above. 

### Hyperparameters of SVM 

- Key hyperparameters of `rbf` SVM are
    - `gamma`
    - `C`
    
- We are not equipped to understand the meaning of these parameters at this point but you are expected to describe their relation to the fundamental tradeoff. 

See [`scikit-learn`'s explanation of RBF SVM parameters](https://scikit-learn.org/stable/auto_examples/svm/plot_rbf_parameters.html). 

### Relation of `gamma` and the fundamental trade-off

- `gamma` controls the complexity (fundamental trade-off), just like other hyperparameters we've seen.
  - larger `gamma` $\rightarrow$ more complex
  - smaller `gamma` $\rightarrow$ less complex


```python
gamma = [0.001, 0.01, 0.1, 1.0, 10.0]
plot_svc_gamma(
    gamma,
    X_train.to_numpy(),
    y_train.to_numpy(),
    x_label="longitude",
    y_label="latitude",
)
```


    
![png](03_kNNs-SVM-RBF_files/03_kNNs-SVM-RBF_113_0.png)
    


### Relation of `C` and the fundamental trade-off

- `C` _also_ affects the fundamental tradeoff
    - larger `C` $\rightarrow$ more complex 
    - smaller `C` $\rightarrow$ less complex 


```python
C = [0.1, 1.0, 100.0, 1000.0, 100000.0]
plot_svc_C(
    C, X_train.to_numpy(), y_train.to_numpy(), x_label="longitude", y_label="latitude"
)
```


    
![png](03_kNNs-SVM-RBF_files/03_kNNs-SVM-RBF_115_0.png)
    


### Search over multiple hyperparameters

- So far you have seen how to carry out search over a hyperparameter
- In the above case the best training error is achieved by the most complex model (large `gamma`, large `C`).
- Best validation error requires a hyperparameter search to balance the fundamental tradeoff.
  - In general we can't search them one at a time.
  - More on this next week. But if you cannot wait till then, you may look up the following:
    - [sklearn.model_selection.GridSearchCV](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.GridSearchCV.html)
    - [sklearn.model_selection.RandomizedSearchCV](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.RandomizedSearchCV.html)

### SVM Regressor

- Similar to KNNs, you can use SVMs for regression problems as well.
- See [`sklearn.svm.SVR`](https://scikit-learn.org/stable/modules/generated/sklearn.svm.SVR.html) for more details. 

## ❓❓ Questions for you 

## (iClicker) Exercise 3.2 

**Select all of the following statements which are TRUE.**

- (A) $k$-NN may perform poorly in high-dimensional space (say, *d* > 1000). 
- (B) In sklearn’s SVC classifier, large values of gamma tend to result in higher training score but probably lower validation score. 
- (C) If we increase both gamma and C, we can't be certain if the model becomes more complex or less complex.

```{admonition} V's Solutions!
:class: tip, dropdown

```

<br><br>

## Summary

- We have KNNs and SVMs as new supervised learning techniques in our toolbox.
- These are analogy-based learners and the idea is to assign nearby points the same label.
- Unlike decision trees, all features are equally important. 
- Both can be used for classification or regression (much like the other methods we've seen).

### Coming up:

Lingering questions: 
- Are we ready to do machine learning on real-world datasets?
- What would happen if we use $k$-NNs or SVM RBFs on the spotify dataset from hw2?  
- What happens if we have missing values in our data? 
- What do we do if we have features with categories or string values?



