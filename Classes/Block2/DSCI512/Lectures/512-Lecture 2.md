
# Lecture 2: Space Complexity and Basic Data Structures

<br>


## Pre-Reading

- Hashing and Hash Tables: [short video (~9 mins)](https://www.youtube.com/watch?v=pMM9cIAFAug)
- Stacks: [short video (3 mins)](https://www.youtube.com/watch?v=KcT3aVgrrpU) 
- Queues: [short video (3 mins)](https://www.youtube.com/watch?v=D6gu-_tmEpQ)

## Outline
- Intro and space complexity (10 mins)
- Memory profiling (25 mins)
- Intro to basic data structures (5 mins)
- Hash tables and hash functions (15 mins)
- Stacks and queues (15 mins)

## Learning Objectives

- Given simple code, determine the space complexity using Big-O notation.
- Measure the amount of memory usage in Python using `psutil`, `getsizeof`, and `nbytes`.
- Explain how data types can affect memory, and why it is important to choose the right data types. 
- Explain what a data structure is and why it is useful in computing at a high level.
- List Python's built-in data structures and describe their properties (review from 511).
- Explain basic (user-defined) data structures: stacks, queues, and hash tables.
- Explain the pros and cons of hash tables, and how they work at a high level.





```python
import numpy as np
import pandas as pd
from collections import defaultdict
import psutil
import matplotlib.pyplot as plt
import altair as alt
import os, time, threading
```

## 1. Lecture Intro (5mins)

Having a sense of how much memory your code requires to run successfully is especially important. In some cases, a program may run for a long time before eventually running out of memory. Without estimating memory usage in advance, you risk wasting significant time and energy without ever obtaining useful output.

When working with large datasets and complex code, it can be frustrating not knowing whether a run will finish within a reasonable time or crash due to insufficient memory. One strategy to avoid this is:

- **Analyze time and space complexity** to understand how your code scales. As you may recall, complexity analysis answers the question: "If $n$ gets bigger, how much more time (or space) will my code require to run?"

- **Test on a fraction of your data** and use time and memory profiling to estimate resource needs. Then, apply your complexity knowledge to extrapolate. For example, if your code grows linearly in space, doubling your dataset means you will roughly need about twice as much memory.

```{note}
Memory growth is not always exactly linear with dataset size; it depends on the algorithm and data structures. For example, some operations might create temporary copies or have overhead that makes growth slightly non-linear.
``` 

We’ve already discussed time complexity in the previous lecture. In this lecture, we will focus on space complexity, which is conceptually very similar. After that, we’ll explore memory profiling and provide practical strategies for applying it. Finally, we will cover data structures, which are powerful tools to help you save both time and memory when your code runs.

## 2. Space Complexity (5mins)

- If code takes too long to run, that might be a problem and one possible problem is running out of memory.
  - Note: this is NOT the same as "disk space".


```python
psutil.virtual_memory()
```




    svmem(total=19327352832, available=11059838976, percent=42.8, used=6495764480, free=7652360192, active=3430236160, inactive=3276505088, wired=3065528320)



- Apparently I have about 19327352832 / (1024**3) ≈ 18 GB of RAM.

- A number typically takes up 8 bytes, so I can store around 2 billion numbers.
  - Actually less, because I have other stuff going on, not just Python.
  - Plus, there's overhead from within Python.
- If my code needs to store 2 billion numbers _at the same time_, I can't run it on my laptop. 


- Similar to time complexity, we can analyze space complexity using Big O notation.

```{admonition} Example 1:
:class: tip
```python
x = np.zeros(n)
```

```{dropdown} Space complexity
Space complexity: $O(n)$
```



```{admonition} Example 2:
:class: tip
```python
x = np.zeros((n,n))
```

```{dropdown} Space complexity
Space complexity: $O(n^2)$
```




```{admonition} Example 3:
:class: tip
```python
x = np.zeros((n,n,5))
```


```{dropdown} Space complexity
Space complexity: $O(n^2)$
```



```{admonition} Example 4:
:class: tip
```python
x = np.zeros(5)
```

```{dropdown} Space complexity
Space complexity: $O(1)$
```





Similar to time complexity, which we discussed in the previous lecture, we can also use profiling in practice—along with Big O notation—to detect potential memory leaks or inefficiencies in our code. 

## 3. Memory profiling (25mins)

Memory profiling can be done using a variety of tools. For instance, line-by-line memory usage can be tracked with `memory_profiler`. However, monitoring memory is not as straightforward as measuring execution time, as it is much more dynamic (a program may allocate and free memory at different stages, libraries can make hidden allocations in the background, etc.). Also, profiling gives estimates, not exact values, and can vary depending on the environment (Python interpreter, OS, etc.). In this course, we will introduce only the basic concepts and simple memory profiling methods using tools such as `getsizeof`, `nbytes`, and `psutil`. If you are interested in exploring this topic further, please refer to this [document](https://jakevdp.github.io/PythonDataScienceHandbook/01.07-timing-and-profiling.html).


```python
x = np.random.rand(1_000_000_000)
```


```python
x.shape
```




    (1000000000,)




```python
x.nbytes
```




    8000000000




```python
from sys import getsizeof
```


```python
getsizeof(x)
```




    8000000112



### 3.1 Data types and memory usage (20mins)

- In the following, we will use memory profiling to see **how data types affect memory** when loading the *same* CSV in pandas.  
- We will see that a “naïve” read (letting pandas guess types) can use a lot of RAM, while an “optimized” read (explicit dtypes, categories, parsed dates) can cut memory dramatically.

#### Reminder: Common built-in Python data types

| English name          | Type name  | Type Category  | Description                                   | Example                                    |
| :-------------------- | :--------- | :------------- | :-------------------------------------------- | :----------------------------------------- |
| integer               | `int`      | Numeric Type   | positive/negative whole numbers               | `42`                                       |
| floating point number | `float`    | Numeric Type   | real number in decimal form                   | `3.14159`                                  |
| boolean               | `bool`     | Boolean Values | true or false                                 | `True`                                     |
| string                | `str`      | Sequence Type  | text                                          | `"Can I have a cheezburger?"`              |
| list                  | `list`     | Sequence Type  | a collection of objects - mutable & ordered   | `['Ali', 'Xinyi', 'Miriam']`               |
| tuple                 | `tuple`    | Sequence Type  | a collection of objects - immutable & ordered | `('Thursday', 6, 9, 2018)`                 |
| dictionary            | `dict`     | Mapping Type   | mapping of key-value pairs - mutable & ordered                   | `{'name':'DSCI', 'code':512, 'credits':2}` |
| none                  | `None` | Null Object    | represents no value                           | `None`                                     |

- Imagine that we are given the following `.csv` file:


```python
from pathlib import Path

csv_path = Path("data/big_demo_mem.csv")
df = pd.read_csv(csv_path)
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
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>age</th>
      <th>income</th>
      <th>country</th>
      <th>active</th>
      <th>signup</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>8</td>
      <td>85443.588314</td>
      <td>IN</td>
      <td>no</td>
      <td>2022-12-02</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>77</td>
      <td>61958.201592</td>
      <td>BR</td>
      <td>no</td>
      <td>2018-04-10</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>65</td>
      <td>59609.498028</td>
      <td>BR</td>
      <td>yes</td>
      <td>2021-09-30</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>43</td>
      <td>80425.140423</td>
      <td>IN</td>
      <td>no</td>
      <td>2022-04-05</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>43</td>
      <td>95244.827084</td>
      <td>MX</td>
      <td>no</td>
      <td>2022-06-14</td>
    </tr>
  </tbody>
</table>
</div>



In the following, we:
- load the same `.csv` using two different ways, prints peak memory/time and the steady-state DataFrame size.
- show side-by-side dtypes and prints **% memory saved**.



```python
# Helpers

def df_mem_mb(df):
    """Deep DataFrame memory in MB."""
    return df.memory_usage(deep=True).sum() / 1024**2

def profile_peak_mem(label, func, *args, **kwargs):
    """
    Run a function and print true peak process memory (MB) + elapsed time (s).
    Uses psutil with background sampling.
    """
    process = psutil.Process(os.getpid())
    peak_mem = process.memory_info().rss  # start value
    running = True

    def monitor():
        nonlocal peak_mem, running
        while running:
            mem = process.memory_info().rss
            if mem > peak_mem:
                peak_mem = mem
            time.sleep(0.01)  # 10ms sampling interval

    # start background monitor
    t0 = time.perf_counter()
    thread = threading.Thread(target=monitor)
    thread.start()

    out = func(*args, **kwargs)

    # stop monitor
    running = False
    thread.join()
    elapsed = time.perf_counter() - t0

    print(f"{label:<6} | peak={peak_mem/1024**2:8.2f} MB | elapsed={elapsed:6.2f}s "
          f"| df_mem={df_mem_mb(out):8.2f} MB")
    return out

```


```python
# defining the two different read functions:

def load_naive(path):
    # Let pandas guess types (often object/64-bit) → larger memory
    '''
    Load a CSV file into a pandas DataFrame.
    
    This function is a simple wrapper around `pandas.read_csv` that lets pandas
    automatically infer data types. Note that this can use more memory, as string
    columns are loaded as `object` and numeric columns default to 64-bit types.

    Parameters
    ----------
    path : str
        Path to the CSV file to load.

    Returns
    -------
    pd.DataFrame
        A DataFrame containing the data from the CSV file.

    Notes
    -----
    For large datasets, consider specifying `dtype` for columns to reduce memory usage.
    '''
    return pd.read_csv(path)

def load_tight(path):
    '''
    Load a CSV file into a pandas DataFrame with optimized memory usage.

    This function explicitly sets column data types, parses date columns, and
    converts string columns with repeated values to categorical type, which
    significantly reduces memory usage compared to letting pandas infer types.

    Parameters
    ----------
    path : str
        Path to the CSV file to load.

    Returns
    -------
    pd.DataFrame
        A DataFrame containing the data from the CSV file with optimized dtypes.

    Notes
    -----
    The following dtypes are applied by default:
    - 'id': uint32
    - 'age': uint8
    - 'income': float32
    - 'country': category
    - 'active': category
    The column 'signup' is parsed as a datetime.

    Example
    -------
    >>> df = load_tight("data/big_demo_mem.csv")
    >>> df.info(memory_usage="deep")
    >>> df.head()
    '''
    # Explicit dtypes + parsed dates + categories → smaller memory
    dtype_opt = {
        "id": "uint32",
        "age": "uint8",
        "income": "float32",
        "country": "category",
        "active": "category",
    }
    return pd.read_csv(path, dtype=dtype_opt, parse_dates=["signup"])
```


```python
# Run & compare


naive = profile_peak_mem("naive", load_naive, csv_path)
tight = profile_peak_mem("tight", load_tight, csv_path)

saved_pct = (1 - df_mem_mb(tight)/df_mem_mb(naive)) * 100
print(f"\nMemory saved (DataFrame): {saved_pct:.1f}%")

# Show dtypes side-by-side
pd.DataFrame({
    "naive_dtypes": naive.dtypes.astype(str),
    "tight_dtypes": tight.dtypes.astype(str)
})
```

    naive  | peak= 5932.88 MB | elapsed=  1.02s | df_mem=   59.94 MB
    tight  | peak=  323.39 MB | elapsed=  0.47s | df_mem=    5.44 MB
    
    Memory saved (DataFrame): 90.9%





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
      <th>naive_dtypes</th>
      <th>tight_dtypes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>id</th>
      <td>int64</td>
      <td>uint32</td>
    </tr>
    <tr>
      <th>age</th>
      <td>int64</td>
      <td>uint8</td>
    </tr>
    <tr>
      <th>income</th>
      <td>float64</td>
      <td>float32</td>
    </tr>
    <tr>
      <th>country</th>
      <td>object</td>
      <td>category</td>
    </tr>
    <tr>
      <th>active</th>
      <td>object</td>
      <td>category</td>
    </tr>
    <tr>
      <th>signup</th>
      <td>object</td>
      <td>datetime64[ns]</td>
    </tr>
  </tbody>
</table>
</div>



- Per-column memory table to see **where savings come from** (e.g., converting `object` to `category`, downcasting 64-bit to 8/32-bit):


```python
# Per-column memory table
naive_mem = (naive.memory_usage(deep=True) / 1024**2).rename("naive_MB")
tight_mem = (tight.memory_usage(deep=True) / 1024**2).rename("tight_MB")

mem_table = pd.concat([naive_mem, tight_mem], axis=1)
mem_table["saving_MB"]  = mem_table["naive_MB"] - mem_table["tight_MB"]
mem_table["saving_%"]   = 100 * mem_table["saving_MB"] / mem_table["naive_MB"]
mem_table.sort_values("saving_MB", ascending=False)
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
      <th>naive_MB</th>
      <th>tight_MB</th>
      <th>saving_MB</th>
      <th>saving_%</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>signup</th>
      <td>19.168854</td>
      <td>2.288818</td>
      <td>16.880035</td>
      <td>88.059701</td>
    </tr>
    <tr>
      <th>active</th>
      <td>17.023112</td>
      <td>0.286216</td>
      <td>16.736897</td>
      <td>98.318664</td>
    </tr>
    <tr>
      <th>country</th>
      <td>16.880035</td>
      <td>0.286951</td>
      <td>16.593084</td>
      <td>98.300056</td>
    </tr>
    <tr>
      <th>age</th>
      <td>2.288818</td>
      <td>0.286102</td>
      <td>2.002716</td>
      <td>87.500000</td>
    </tr>
    <tr>
      <th>id</th>
      <td>2.288818</td>
      <td>1.144409</td>
      <td>1.144409</td>
      <td>50.000000</td>
    </tr>
    <tr>
      <th>income</th>
      <td>2.288818</td>
      <td>1.144409</td>
      <td>1.144409</td>
      <td>50.000000</td>
    </tr>
    <tr>
      <th>Index</th>
      <td>0.000126</td>
      <td>0.000126</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
  </tbody>
</table>
</div>



### 3.2 Key take-aways:

```{admonition} Key take-aways
:class: tip
- **Dtypes matter:** 64-bit numbers and `object` strings are expensive; downcasting and `category` can save **40–80%** memory.
- **Parsed dates:** store dates as `datetime64[ns]` instead of plain strings.
- **Measure, don’t guess:** use `memory_usage(deep=True)` for size in RAM and `psutil` for **peak** usage while loading.

## 4. Intro to data structures (5mins)


- With _time complexity_, we tend to think more about the _algorithms_.
- With _space complexity_, we tend to think more about the _data structures_.



_Reminder from the last lecture_: a _data structure_ is a way to organize and manage data, allowing us to write more efficient code in terms of both time and space. 

When working with data structures, the main operations we usually consider are:
- inserting a new element
- deleting an element
- checking if an element is present (contain)

### 4.1 Python Built-in Data Structures:
 - **Lists**
    - inserting a new element: `list.append()`, `list.insert()`, etc. 
    - deleting an element: `list.remove()`, `list.pop()`, etc.
    - checking if an element is present: `in`
 - **Tuples**
    - inserting a new element: immutable (but we can still "add" two tuples)
    - deleting an element: immutable
    - checking if an element is present: `in` 
 - **Sets**
    - inserting a new element: `set.add()`, `^` , `|`, etc.
    - deleting an element: `set.remove()`, `-`, etc.
    - checking if an element is present: `in` 
 - **Dictionaries**
    - inserting a new element: `dict.update()`, etc. 
    - deleting an element: `dict.pop()`, etc.
    - checking if an element is present: `in` (for `dict.keys(`), `dict.values()`)  


In the previous lecture, we saw that checking if an element is present in a Python set is very efficient. Let's look into this more.

## 5. Hash Tables & Hash Functions (25mins)


Hash tables and hash functions are fundamental in computer science and data science. They are the backbone of dictionaries/maps in most programming languages, and many coding interviews and real-world problems require understanding how to use hash maps effectively. 

#### Example: 
Python's `set` type supports the following operations in $O(1)$ time:


  - inserting a new element
  - deleting an element
  - checking if an element is present

How could we implement this using the tools we already have?

- Well, what about using linear search to find elements, e.g. a `list`?
  - This is too slow
- What about using binary search?
  - Now searching is fast, but insertion/deletion is slow, because we need to maintain an ordered list
- Enter the [hash table](https://en.wikipedia.org/wiki/Hash_table) - to save the day!
  - Trees could also work, but hash tables are the most popular.
  
### 5.1 Hash Functions (10mins)

From [_Wikipedia_](https://en.wikipedia.org/wiki/Hash_function): 
- "A hash function is any function that can be used to map data of arbitrary size to fixed-size values, though there are some hash functions that support variable-length output. The values returned by a hash function are called hash values, hash codes, (hash/message) digests, or simply hashes."

Python objects have a _hash_:


```python
hash("mds")
```




    1693511145997534212




```python
hash("")
```




    0



It looks like the hash function returns an integer.


```python
hash(5.5)
```




    1152921504606846981




```python
hash(5)
```




    5




```python
hash(-9999)
```




    -9999



It looks like the hash function of a Python integer is itself. Or at least small enough integers.


```python
hash(999999999999999999999999)
```




    2003764205207330319



Sometimes it fails?


```python
hash([1, 2, 3])

```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    Cell In[19], line 1
    ----> 1 hash([1, 2, 3])


    TypeError: unhashable type: 'list'



```python
hash((1, 2, 3))
```




    529344067295497451




```python
hash(None)
```




    -9223372036585674007



If a Python `set` is a hash table, that means items in it must be hashable (`dict` has the same requirement, for keys):


```python
s = set()
```


```python
s.add(5.5)
```


```python
s.add("mds")
```


```python
s
```




    {5.5, 'mds'}




```python
s.add([1, 2, 3])
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    Cell In[26], line 1
    ----> 1 s.add([1, 2, 3])


    TypeError: unhashable type: 'list'



```python
s.add((1, 2, 3))
```


```python
s
```




    {(1, 2, 3), 5.5, 'mds'}



##### Hash Function Overview
- Immutable objects (numbers, strings, tuples) are hashable, mutable objects (lists, dicts) are not
- Hash functions are **deterministic**
- Hash functions have a **fixed-size output**
- Hash function are **collision-resistant**
- Hash functions are extremely broadly useful, beyond what we talk about in this course 🤓

### 5.2 Hash Tables (5mins)

- So, it looks like the hash function maps from an object to an integer.
- And that Python `set`s use these hash functions.
- How do they work?
- The hash table is basically a list of lists, and the hash function (mod the array size) maps an object to its location in the outer list.
  - But it's a bit more complicated than that.
  - The list typically expands and contracts automatically as needed.
  - These operations may be slow, but averaged or "amortized" over many operations, the runtime is $O(1)$
  - The hash function depends on this array size.
  - There's also an issue of collisions: when two different objects hash to the same place.
- Roughly speaking, we can insert, retrieve, and delete things in $O(1)$ time so long as we have a "good" hash function.
  - The hash function will be "good" for default Python objects, and if you end up needing to implement your own one day you should read a bit more about it.

For more details about Hashing, Hash function, and Hash tables, please refer to this [document](https://runestone.academy/ns/books/published/pythonds/SortSearch/Hashing.html).

### 5.3 A Simple Hash Table (5mins)

Below is a (very low-quality) hash table implementation, with only 4 buckets by default:

<!--
class HashTable:
    
    def __init__(self, num_buckets=4):
        self.stuff = list() # A list of lists
        self.n = num_buckets
        
        for i in range(num_buckets):
            self.stuff.append([]) # Create the inner lists, one per bucket
        
    def add(self, item):
        if not self.contains(item):
            self.stuff[hash(item) % self.n].append(item)
        
    def contains(self, item):
        return item in self.stuff[hash(item) % self.n]
    
    def __str__(self):
        return str(self.stuff)
-->


```python
class HashTable:
    
    def __init__(self, num_buckets=4):
        self.stuff = list() # A list of lists
        self.n = num_buckets
        
        for i in range(num_buckets):
            self.stuff.append([]) # Create the inner lists, one per bucket
        
    def add(self, item):
        if not self.contains(item):
            self.stuff[hash(item) % self.n].append(item)
        
    def contains(self, item):
        return item in self.stuff[hash(item) % self.n]
    
    def __str__(self):
        return str(self.stuff)
```

(Note: The `hash` function has a random seed that is set at the start of every Python session, so your actual results my vary from mine.)


```python
ht = HashTable()
print(ht)
```

    [[], [], [], []]


- So far, all 4 buckets are empty. 
- Now let's add something:


```python
ht.add("hello")
print(ht)
```

    [[], ['hello'], [], []]


"hello" went into this bucket because 


```python
hash("hello")
```




    -2379814820969177039




```python
hash("hello") % 4
```




    1



Now let's add more things:


```python
ht.add("goodbye")
print(ht)
```

    [['goodbye'], ['hello'], [], []]



```python
ht.add("test")
print(ht)
```

    [['goodbye', 'test'], ['hello'], [], []]



```python
ht.add("item")
print(ht)
```

    [['goodbye', 'test', 'item'], ['hello'], [], []]



```python
ht.add("what")
print(ht)
```

    [['goodbye', 'test', 'item', 'what'], ['hello'], [], []]


If we want to look for something:


```python
ht.contains("blah")
```




    False



False because 


```python
hash("blah") % 4
```




    3



And "blah" is not found in bucket. 


```python
ht.contains("item")
```




    True



- Same thing here.
- The key idea is that you **only need to look in one bucket** - either it's in that bucket, or it's not in the _entire_ hash table.


```python
print(ht)
```

    [['goodbye', 'test', 'item', 'what'], ['hello'], [], []]


- Above we have a _collision_: that is, 2 items in the same bucket.

```{admonition}

**Question:** If my hash table has 4 buckets, what is the time complexity of `contains`?

A: $O(1)$ \
B: $O(n)$ \
C: $O(log(n))$ \
D: $O(nlog(n))$
```


```{dropdown} Answer

$O(n)$ -> we narrow our search down to one of the 4 buckets, but we still need to search through all the elements in that bucket. Since the size of the bucket is $\frac{n}{4}$, our time complexity is $O(n)$.
```




- If the main list is able to dynamically grow as the number of items grows, we can keep the number of collisions low.
- This preserves the $O(1)$ operations.

### 5.4 Lookup Tables, Python `dict` (5mins)

- Python's `dict` type is a dictionary (aka symbol table)
- A dictionary should support the following operations:
  - inserting a new element
  - deleting an element
  - finding an element
- It is much like a `set` except the entries, called "keys", now have some data payload associated with them, which we call "values".
- It is also implemented as a hash table, meaning you can expect $O(1)$ operations.
- Only the keys are hashed, so only the keys have to be hashable.
  - A list can be a value, but not a key.


```python
d = dict()
d[5] = "a"
d["b"] = 9
d
```




    {5: 'a', 'b': 9}




```python
5 in d
```




    True




```python
9 in d  # it only searches the keys
```




    False




```python
d[5]
```




    'a'




```python
d[6]
```


    ---------------------------------------------------------------------------

    KeyError                                  Traceback (most recent call last)

    Cell In[46], line 1
    ----> 1 d[6]


    KeyError: 6


Hashable types:


```python
d[[1,2,3]] = 10
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    Cell In[47], line 1
    ----> 1 d[[1,2,3]] = 10


    TypeError: unhashable type: 'list'



```python
d[10] = [1,2,3] # OK
```

A reminder of some dictionary syntax:


```python
# Comprehensions for dictionaries
f = {i: i*2 for i in range(10)}
f
```




    {0: 0, 1: 2, 2: 4, 3: 6, 4: 8, 5: 10, 6: 12, 7: 14, 8: 16, 9: 18}




```python
# Iteration syntax
for key, val in f.items():
    print("key =", key, " val =", val)
```

    key = 0  val = 0
    key = 1  val = 2
    key = 2  val = 4
    key = 3  val = 6
    key = 4  val = 8
    key = 5  val = 10
    key = 6  val = 12
    key = 7  val = 14
    key = 8  val = 16
    key = 9  val = 18



```python
# A dictionary can be a value inside another dictionary
g = dict()
g[5] = f
g
```




    {5: {0: 0, 1: 2, 2: 4, 3: 6, 4: 8, 5: 10, 6: 12, 7: 14, 8: 16, 9: 18}}



Next, we are going to learn about stacks and queues, two other fundamental data structures. These structures form the foundation for more complex data structures, such as [deques](https://www.geeksforgeeks.org/dsa/deque-set-1-introduction-applications/), [priority queues](https://www.geeksforgeeks.org/dsa/priority-queue-set-1-introduction/), [heaps](https://www.geeksforgeeks.org/dsa/heap-data-structure/), and graphs (which we will discuss in Week 3), that often build on the concepts of stacks and queues.

## 6. Stacks and queues (15mins)


- We want a data structure that we can put things into, and then retrieve them later.
- A [stack](https://en.wikipedia.org/wiki/Stack_(abstract_data_type)) does with with the "last in, first out" (LIFO) mentality - like a stack of books or plates.

### 6.1 Stacks


<img src="static/stack.jpg" alt="drawing" width="250"/>


```python
class Stack:
    """A stack data structure."""

    def __init__(self):
        self.data = list()

    def push(self, item):
        """
        Adds a new item to the top of the stack.
        
        Parameters
        ----------
        item : object
           An item added to the stack
        """
        self.data.append(item)

    def pop(self):
        """
        Removes the item that is at the top of the stack and returns the item.

        Returns
        -------
        object 
             The item that was last added to the stack.

        Examples
        --------
        >>> stack = Stack()
        >>> stack.push(1)
        >>> stack.push([1, 2, "dog"])
        >>> stack.push("popcorn")
        >>> stack.pop()
        'popcorn'
        """
        return self.data.pop()

    def isEmpty(self):
        """ 
        Checks to see if the stack is empty.

        Returns
        -------
        bool 
             True if the stack contains no items, False otherwise.    

        Example
        --------
        >>> stack = Stack()
        >>> stack.isEmpty()
        True
        """
        return len(self.data) == 0
    
    def __str__(self):
        return self.data.__str__()
```


```python
s = Stack()

s.push("first")
s.push("second")

print(s)
```

    ['first', 'second']



```python
s.pop()
```




    'second'




```python
print(s)
```

    ['first']



```python
s.push("third")
s.push("fourth")
```


```python
print(s)
```

    ['first', 'third', 'fourth']



```python
s.pop()
```




    'fourth'




```python
s.pop()
```




    'third'




```python
s.pop()
```




    'first'




```python
s.pop()
```


    ---------------------------------------------------------------------------

    IndexError                                Traceback (most recent call last)

    Cell In[61], line 1
    ----> 1 s.pop()


    Cell In[52], line 36, in Stack.pop(self)
         18 def pop(self):
         19     """
         20     Removes the item that is at the top of the stack and returns the item.
         21 
       (...)
         34     'popcorn'
         35     """
    ---> 36     return self.data.pop()


    IndexError: pop from empty list


```{admonition} **Question:**
If I have a stack containing `n` items, what is the time complexity to retrieve the "bottom" item?

A: $O(1)$ \
B: $O(n)$ \
C: $O(log(n))$ \
D: $O(n^2)$ \
E: Not Sure
```

```{dropdown} Answer
$O(n)$ -> we need to pop off all `n` items to get to the bottom item.
```

### 6.2 Queues
- Along with stacks we also have [queues](https://en.wikipedia.org/wiki/Queue_(abstract_data_type)), which use "first in, first out" (FIFO) ordering.
  - e.g. an actual queue/lineup

<img src="static/queue.jpg" alt="drawing" width="250"/>


```python
class Queue:
    """A Queue data structure."""
    
    def __init__(self):
        self.data = list()

    def push(self, item):
        """
        Adds a new item to the end of the queue.
        
        Parameters
        ----------
        item : object
           An item added to the queue
        """
        
        self.data.append(item)

    def pop(self):
        """
        Removes the item that is at the front of the queue and returns the item.

        Returns
        -------
        object 
             The least recent item added to the queue.     

        Example
        --------
        >>> queue = Queue()
        >>> queue.push(1)
        >>> queue.push([1, 2, "dog"])
        >>> queue.push("popcorn")
        >>> queue.pop()
        '1'
        """
        return self.data.pop(0)

    def isEmpty(self):
        """ 
        Checks to see if the queue is empty. 

        Returns
        -------
        bool 
            True if the stack contains no items, False otherwise.    

        Example
        --------
        >>> queue = Queue()
        >>> queue.push(1)
        >>> Queue.isEmpty()
        False
        
        """
        return len(self.data) == 0
    
    def __str__(self):
        return self.data.__str__()
```


```python
q = Queue()

q.push("first")  # often called "enqueue"
q.push("second")

print(q)
```

    ['first', 'second']



```python
q.pop() # often called "dequeue"
```




    'first'




```python
print(q)
```

    ['second']



```python
q.push("third")
q.push("fourth")
```


```python
print(q)
```

    ['second', 'third', 'fourth']



```python
while not q.isEmpty():
    print(q.pop())
```

    second
    third
    fourth


```{admonition} **Question:**
If we have a queue `[A, B, C, D]` with A at the front/head of the queue, what will the queue look like after the following operations:
- push `E` onto the queue
- push `F` onto the queue
- remove one item from the queue
- push `A` onto the queue

A: `[A, B, C, E, F]` \
B: `[A, F, E, B, C, D]` \
C: `[B, C, D, E, F, A]` \
D: Not sure
```

```{dropdown} Answer
`C: [B, C, D, E, F, A]` -> we add `E` and `F` to the back of the queue, remove `A` from the front of the queue, and then add `A` to the back of the queue.
```
