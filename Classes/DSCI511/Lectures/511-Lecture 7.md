<img src="img/dsci511_header.png" width="600">

  

# Lecture 7: Functions & Testing

  

## Lecture learning objectives

  

- Define a function and an anonymous function in Python

- Describe the difference between positional and keyword arguments

- Describe the difference between local and global arguments

- Apply the DRY principle to write modular code

- Assess whether a function has side effects

- Write a docstring for a function that describes parameters, return values, behaviour and usage

- Write a try/except statement

- Explain the notion of scoping in Python

- Predict whether a function modifies a global variable with scope outside of the function definition.

- Formulate a test case to prove a function design specification

- Use an assert statement to validate a test case

  

## Functions

  

- Define a [**function**](https://docs.python.org/3/tutorial/controlflow.html#defining-functions) to re-use a block of code with different inputs.

- Function definition syntax:

  

```python

def function(param1=arg1, param2=arg2, ...):

    # do something

    output = ...

    return output

```

  

* Functions begin with the `def` keyword, then the function name, parameters/arguments in parentheses, and then a colon (`:`)

* The function block defined by indentation

* Output or "return" value of the function is given by the `return` keyword

  

For example, suppose that we want to compute the probability density function of the [normal distribution](https://en.wikipedia.org/wiki/Normal_distribution), which is given by:

  

$$

f(x) = \frac{1}{\sigma\sqrt{2\pi}} e^{-\frac{1}{2}\left(\frac{x - \mu}{\sigma}\right)^2}

$$

  

Let's assume that we want to compute $ f(2) $ for a mean of $ \mu = 2.5 $ and variance of $ \sigma = 0.3 $:

  
  

```python

import math

  

(1 / (0.3 * (2 * math.pi)**0.5)) * math.exp(-0.5 * ((2 - 2.5) / 0.3)**2)

```

  
  
  
  

    0.3315904626424956

  
  
  

- With a function, we can abstract things and avoid repetition:

  
  

```python

def pdf_normal(x, μ, σ):

    prefactor = (1 / (σ * (2 * math.pi)**0.5))

    exp_value = math.exp(-0.5 * ((x - μ) / σ)**2)

    pdf = prefactor * exp_value

    return pdf

```

  
  

```python

pdf_normal(2, 2.5, 0.3)

```

  
  
  
  

    0.3315904626424956

  
  
  
  

```python

pdf_normal(1, 0, 1)

```

  
  
  
  

    0.24197072451914337

  
  
  

### Functions and Data Science

  

#### Reminder from Worksheet 1

  
  

```python

import pandas as pd

url = "https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2020/2020-02-18/food_consumption.csv"

df = pd.read_csv(url)

```

  
  

```python

df.columns

```

  
  
  
  

    Index(['country', 'food_category', 'consumption', 'co2_emmission'], dtype='object')

  
  
  

How many different kinds of foods are there in the dataset? How many countries are in the dataset?

  
  

```python

n_food = df["food_category"].nunique() # Week 1 SOLUTION

n_country = df["country"].nunique() # Week 1 SOLUTION

```

  

What if we want to do this for an arbitrary DataFrame and column?

  
  

```python

def count_kind(df, column_name):

    count = df[column_name].nunique()

    return count

```

  
  

```python

count_kind(df, "food_category")

```

  
  
  
  

    11

  
  
  

Can we read modify this so that it takes the `url` as an input?

  
  

```python

def count_kind(url, column_name):

    df = pd.read_csv(url)

    count = df[column_name].nunique()

    return count

```

  
  

```python

count_kind(url, "food_category")

```

  
  
  
  

    11

  
  
  

### Side effects & local variables

  

- When you create a variable inside a function, it is **local**, which means that it only exists inside the function. For example:

  
  

```python

import numpy as np

  

def sum_arrays(array_1, array_2):

    final_array = array_1 + array_2

    return final_array

```

  
  

```python

array_1 = np.array([1,1,1])

array_2 = np.array([2,2,2])

  

sum_arrays(array_1, array_2)

```

  
  
  
  

    array([3, 3, 3])

  
  
  
  

```python

final_array

```

  
  

    ---------------------------------------------------------------------------

  

    NameError                                 Traceback (most recent call last)

  

    Cell In[4], line 1

    ----> 1 final_array

  

    NameError: name 'final_array' is not defined

  
  

- If a function does anything other than returning a value, it is said to have **side effects**. An example of this is when a function changes the variables passed into it, or when a function prints something to the screen.

  

- Example:

  
  

```python

def silly_sum(my_list):

    my_list.append(8.5)

    return sum(my_list)

```

  
  

```python

nums = [1, 2, 3, 4]

out = silly_sum(nums)

out

```

  
  
  
  

    18.5

  
  
  

- Looks like what we wanted.

- But wait... it changed our `nums` object...

  
  

```python

nums

```

  
  
  
  

    [1, 2, 3, 4, 8.5]

  
  
  

What if we do this with a numpy array?

  
  

```python

def silly_sum_np(my_array):

    my_array = np.append(my_array, 8.5)

    return my_array.sum()

```

  
  

```python

test_array = np.array([1, 2, 3, 4])

np_out = silly_sum_np(test_array)

np_out

```

  
  
  
  

    np.float64(18.5)

  
  
  
  

```python

test_array

```

  
  
  
  

    array([1, 2, 3, 4])

  
  
  

- If your function has side effects, you must mention it in the documentation (later today).

  

### `None` return type

  

- If you do not specify a return value, the function returns `None` when it terminates:

  
  

```python

def f(x):

    x + 1  # no return!

    if x == 999:

        print('x = 999!')

  
  

print(f(999))

```

  

    x = 999!

    None

  

### Optional & required arguments

  

- Sometimes it is convenient to have _default values_ for some arguments in a function.

- Because they have default values, these arguments are optional, hence "optional arguments"

- Example:

  
  

```python

def magnify_numbers(s, n=2):

    return s*n

```

  
  

```python

magnify_numbers(np.array([1,2]), 2)

```

  
  
  
  

    array([2, 4])

  
  
  
  

```python

magnify_numbers(np.array([1,2]), 5)

```

  
  
  
  

    array([ 5, 10])

  
  
  
  

```python

magnify_numbers(np.array([1,2]))  # do not specify `n`; it is optional

```

  
  
  
  

    array([2, 4])

  
  
  

- You can have any number of required arguments and any number of default arguments

- All the default arguments must come after the required arguments

- The required arguments are mapped by the order they appear

- The default arguments can be specified out of order

  
  

```python

def example(a, b, c="DEFAULT", d="DEFAULT"):

    print(a, b, c, d)

  
  

example(1, 2, 3, 4)

```

  

    1 2 3 4

  
  

```python

example(1, b = 2, c = 3, d = 4)

```

  

    1 2 3 4

  

- Using the defaults for `c` and `d`:

  
  

```python

example(1, 2)

```

  

    1 2 DEFAULT DEFAULT

  

- Specifying `c` and `d` as **keyword arguments** (i.e. by name):

  
  

```python

example(1, 2, c=3, d=4)

```

  

    1 2 3 4

  

- Specifying only one of the optional arguments, by keyword:

  
  

```python

example(1, 2, c=3)

```

  

    1 2 3 DEFAULT

  

- Specifying all the arguments as keyword arguments, even though only `c` and `d` are optional:

  
  

```python

example(a=1, b=2, c=3, d=4)

```

  

    1 2 3 4

  

- Specifying `c` by the fact that it comes 3rd (I do not recommend this because I find it is confusing):

  
  

```python

example(1, 2, 3)

```

  

    1 2 3 DEFAULT

  

- Specifying the optional arguments by keyword, but out of order:

  
  

```python

example(1, 2, d=4, c=3)

```

  

    1 2 3 4

  

- Specifying the non-optional arguments by keyword (I am fine with this):

  
  

```python

example(a=1, b=2)

```

  

    1 2 DEFAULT DEFAULT

  

- Specifying the non-optional arguments by keyword, but in the wrong order (not recommended, I find it confusing):

  
  

```python

example(b=2, a=1)

```

  

    1 2 DEFAULT DEFAULT

  

- Specifying keyword arguments before non-keyword arguments (this throws an error):

  
  

```python

example(a=2, 1)

```

  
  

      Cell In[44], line 1

        example(a=2, 1)

                      ^

    SyntaxError: positional argument follows keyword argument

  
  

- In general, I am used to calling non-optional arguments by order, and optional arguments by keyword.

- The language allows us to deviate from this, but it can be unnecessarily confusing sometimes.

  

- Ideally, the default should be carefully chosen.

- Here, the idea of "repeating" something makes me think of having 2 copies, so `n=2` feels like a sane default.

  

### Multiple return values

  

- In many programming languages, functions can only return one object

- That is technically true in Python, but there is a "workaround", which is to return a tuple.

  
  

```python

def mean_and_median(x):

    return (np.mean(x), np.median(x))

```

  
  

```python

mean_and_median([1,2,3,4,5])

```

  
  
  
  

    (np.float64(3.0), np.float64(3.0))

  
  
  

- The parentheses can be omitted in this case, and a `tuple` is implicitly returned as defined by the use of the comma:

  
  

```python

def mean_and_median(x):

    return np.mean(x), np.median(x)

```

  
  

```python

mean_and_median([1,2,3,4,5])

```

  
  
  
  

    (np.float64(3.0), np.float64(3.0))

  
  
  

- It is common to immediately unpack a returned tuple into separate variables, so it really feels like the function is returning multiple values:

  
  

```python

s, p = mean_and_median([1,2,3.2,4,5])

```

  
  

```python

s

```

  
  
  
  

    np.float64(3.04)

  
  
  
  

```python

p

```

  
  
  
  

    np.float64(3.2)

  
  
  

- As an aside, it is conventional in Python to use `_` for values you don't want:

  
  

```python

s, _ = mean_and_median([1,2,3.2,4,5])

```

  
  

```python

s

```

  
  
  
  

    np.float64(3.04)

  
  
  
  

```python

_

```

  
  
  
  

    np.float64(3.2)

  
  
  

### Passing Multiple Arguments to a Function

  

```python

def my_sum(a, b):

    return a + b

```

  

- This function works fine, but it’s limited to only two arguments.

- What if you need to sum a varying number of arguments, where the specific number of arguments passed is only determined at runtime?

- Wouldn’t it be great to create a function that could sum all the integers passed to it, no matter how many there are?

  

- You can also call/define functions that accept an arbitrary number of positional or keyword arguments using `*args` and `**kwargs`. See, e.g. [here](https://realpython.com/python-kwargs-and-args/)

  
  

```python

def add(*args):

    print(args)

    return sum(args)

```

  
  

```python

add(1, 2, 3, 4, 5 , 6)

```

  

    (1, 2, 3, 4, 5, 6)

  
  
  
  

    21

  
  
  

`**kwargs` works just like `*args`, but instead of accepting positional arguments it accepts keyword (or named) arguments

  
  

```python

def add(**kwargs):

    print(kwargs)

    return sum(kwargs.values())

```

  
  

```python

add(a=3, b=4, c=5)

```

  

    {'a': 3, 'b': 4, 'c': 5}

  
  
  
  

    12

  
  
  

- Do not instantiate objects (like empty lists) in the function definition - see [here](https://docs.python-guide.org/writing/gotchas/) under "Mutable Default Arguments"

  
  

```python

def example(a, b=[]):  # don't do this!

    b.append(a)

    return b

```

  
  

```python

example(1)

```

  
  
  
  

    [1]

  
  
  
  

```python

example(2)  # the list inside the function persists and got appended to!

```

  
  
  
  

    [1, 2]

  
  
  
  

```python

example(2)

```

  
  
  
  

    [1, 2, 2]

  
  
  
  

```python

def example(a, b=None):  # instead, do this

    if b is None:

        b = []

    b.append(a)

    return b

```

  
  

```python

example(1)

```

  
  
  
  

    [1]

  
  
  
  

```python

example(2)

```

  
  
  
  

    [2]

  
  
  

## Functions as a data type

  

- In Python, functions are a data type just like anything else.

  
  

```python

def do_nothing(x):

    return x

```

  
  

```python

type(do_nothing)

```

  
  
  
  

    function

  
  
  
  

```python

print(do_nothing)

```

  

    <function do_nothing at 0x107734d60>

  

- This means you can pass functions as arguments into other functions.

  
  

```python

def square(y):

    return y**2

  
  

def evaluate_function_on_x_plus_1(fun, x):

    return fun(x+1)

```

  
  

```python

evaluate_function_on_x_plus_1(square, 5)

```

  
  
  
  

    36

  
  
  

- Above: what happened here?

  - `fun(x+1)` becomes `square(5+1)`

  - `square(6)` becomes `36`

  

- You can also write functions that return functions, or define functions inside of other functions.

- We'll see examples of this when we get to classes & decorators

  

## Anonymous functions

  

- There are two ways to define functions in Python:

  
  

```python

def add_one(x):

    return x+1

```

  
  

```python

add_one(7.2)

```

  
  
  
  

    8.2

  
  
  
  

```python

lambda x: x+1

```

  
  
  
  

    <function __main__.<lambda>(x)>

  
  
  
  

```python

type(lambda x: x+1)

```

  
  
  
  

    function

  
  
  
  

```python

(lambda x: x+1)(7.2)

```

  
  
  
  

    8.2

  
  
  

- The two approaches above are identical. The one with `lambda` is called an **anonymous function**.

- Anonymous functions can only take up one line of code, so they aren't appropriate in most cases, but can be useful for smaller things

  
  

```python

evaluate_function_on_x_plus_1(lambda x: x ** 2, 5)

```

  
  
  
  

    36

  
  
  

Above:

  

- First, `lambda x: x**2` evaluates to a value of type `function`

  - Notice that this function is never given a name - hence "anonymous functions" !

- Then, the function and the integer `5` are passed into `evaluate_function_on_x_plus_1`

- At which point the anonymous function is evaluated on `5+1`, and we get `36`.

  

- Anonymous functions can have multiple arguments, as well as multiple outputs:

  
  

```python

(lambda x, y: (x+y, x-y, x**y))(5, 2)

```

  
  
  
  

    (7, 3, 25)

  
  
  

## DRY principle: designing good functions

  

- DRY: **Don't Repeat Yourself**

- See [Wikipedia article](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)

- Consider the task of, for each element of a list, turning it into a palindrome

  - e.g. "mike" => "mikeekim"

  
  

```python

names = ["hedayat", "scott", "prajeet"]

```

  
  

```python

name = "hedayat"

name[::-1]  # creates a slice that starts at the end and moves backwards, syntax is [begin:end:step]

```

  
  
  
  

    'tayadeh'

  
  
  
  

```python

names_backwards = list()

  

names_backwards.append(names[0] + names[0][::-1])

names_backwards.append(names[1] + names[1][::-1])

names_backwards.append(names[2] + names[2][::-1])

names_backwards

```

  
  
  
  

    ['hedayattayadeh', 'scottttocs', 'prajeetteejarp']

  
  
  

- Above: this is gross and terrible coding:

  1. It only works for a list with 3 elements

  2. It only works for a list named `names`

  3. If we want to change its functionality, we need to change 3 similar lines of code (Don't Repeat Yourself!!)

  4. It is hard to understand what it does just by looking at it

  
  

```python

names_backwards = list()

  

for name in names:

    names_backwards.append(name + name[::-1])

  

names_backwards

```

  
  
  
  

    ['hedayattayadeh', 'scottttocs', 'prajeetteejarp']

  
  
  

- Above: this is slightly better. We have solved problems (1) and (3).

- But let's create a function to make our life easier

  
  

```python

def make_palindromes(names):

    names_backwards = []

  

    for name in names:

        names_backwards.append(name + name[::-1])

  

    return names_backwards

```

  
  

```python

make_palindromes(names)

```

  
  
  
  

    ['hedayattayadeh', 'scottttocs', 'prajeetteejarp']

  
  
  

- Above: this is even better. We have now also solved problem (2), because you can call the function with any list, not just `names`.

- For example, what if we had multiple _lists_:

  
  

```python

names1 = ["hedayat", "scott", "prajeet"]

names2 = ["apple", "orange", "banana", "strawberry"]

```

  
  

```python

make_palindromes(names1)

```

  
  
  
  

    ['hedayattayadeh', 'scottttocs', 'prajeetteejarp']

  
  
  
  

```python

make_palindromes(names2)

```

  
  
  
  

    ['appleelppa', 'orangeegnaro', 'bananaananab', 'strawberryyrrebwarts']

  
  
  

### Designing good functions

  

- How far you go and how you choose to apply the DRY principle is up to you and the programming context

- These decisions are often ambiguous. For example:

  - Should `make_palindromes` be a function if I'm only ever doing it once? Twice?

  - Should the loop be inside the function, or outside?

  - Or should there be TWO functions, one that loops over the other??

  

- In my personal opinion, `make_palindromes` does a bit too much to be understandable.

- I prefer this:

  
  

```python

def make_palindrome(name):

    return name + name[::-1]

```

  
  

```python

make_palindrome("hedayat")

```

  
  
  
  

    'hedayattayadeh'

  
  
  

- From here, we want to "apply `make_palindrome` to every element of a list"

- We could do this with list comprehension

  
  

```python

[make_palindrome(name) for name in names]

```

  
  
  
  

    ['hedayattayadeh', 'scottttocs', 'prajeetteejarp']

  
  
  

- Or there is also the in-built `map()` function which does exactly this, applies a function to every element of a sequence

  
  

```python

list(map(make_palindrome, names))

```

  
  
  
  

    ['hedayattayadeh', 'scottttocs', 'prajeetteejarp']

  
  
  

## (Optional) Generators

  

- Recall list comprehension from the previous lecture

  
  

```python

[n for n in range(10)]

```

  
  
  
  

    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

  
  
  

- Comprehensions evaluate the entire expression at once, and then return the full data product

- Sometimes, we want to work with just one part of our data at a time, for example, when we can't fit all of our data in memory (I'll show an example of this a little later)

- For this, we can use generators (you'll see more of these when we get to DSCI 572!)

  
  

```python

(n for n in range(10))

```

  
  
  
  

    <generator object <genexpr> at 0x10c279be0>

  
  
  

- Notice that we just created a `generator object`

- Generator objects are like a "recipe" for generating values

- They don't actually do any computation until they are asked to

- We can get values from a generator in three main ways:

  - Using `next()`

  - Using `list()`

  - Looping

  
  

```python

gen = (n for n in range(10))

```

  
  

```python

next(gen)

```

  
  
  
  

    0

  
  
  
  

```python

next(gen)

```

  
  
  
  

    1

  
  
  
  

```python

list(gen)

```

  
  
  
  

    [2, 3, 4, 5, 6, 7, 8, 9]

  
  
  

- But once the generator is exhausted, it will no longer return values:

  
  

```python

gen = (n for n in range(10))

for i in range(11):

    print(next(gen))

```

  

    0

    1

    2

    3

    4

    5

    6

    7

    8

    9

  
  

    ---------------------------------------------------------------------------

  

    StopIteration                             Traceback (most recent call last)

  

    Cell In[131], line 3

          1 gen = (n for n in range(10))

          2 for i in range(11):

    ----> 3     print(next(gen))

  

    StopIteration:

  
  

- We can see all the values of a generator using `list()` but this defeats the purpose of using a generator in the first place

  
  

```python

gen = (n for n in range(10))

list(gen)

```

  
  
  
  

    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

  
  
  

- Finally, we can loop over generator objects too

  
  

```python

gen = (n for n in range(10))

for i in gen:

    print(i)

```

  

    0

    1

    2

    3

    4

    5

    6

    7

    8

    9

  

- Above, we saw how to create a generator object using comprehension syntax but with parentheses

- We can also create a generator using functions and the `yield` keyword (instead of the `return` keyword)

  
  

```python

def gen():

    for n in range(10):

        yield (n, n ** 2)

```

  
  

```python

g = gen()

print(next(g))

print(next(g))

print(next(g))

list(g)

```

  

    (0, 0)

    (1, 1)

    (2, 4)

  
  
  
  

    [(3, 9), (4, 16), (5, 25), (6, 36), (7, 49), (8, 64), (9, 81)]

  
  
  

Generators can thought of as **state-preserving** functions.

  - This means that a generator keeps track of what items have been already generated, and what the state of the generator is at any point.

  

- We'll work with generators more when we get to 572 and other ML courses where we are often working with large datasets (images are especially memory-consuming!)

- But so you keep them in the back of your mind, below is some real-world motivation of a case where a generator might be useful

- Say we want to create a list of dictionaries containing information about houses in Canada

  
  

```python

# !conda install -y memory_profiler

```

  
  

```python

import random  

import time

import memory_profiler

city = ['Vancouver', 'Toronto', 'Ottawa',

        'Montreal', 'Edmonton', 'Calgary']

```

  
  

```python

def house_list(n):

    houses = []

    for i in range(n):

        house = {

            'id': i,

            'city': random.choice(city),

            'bedrooms': random.randint(1, 5),

            'bathrooms': random.randint(1, 3),

            'price ($1000s)': random.randint(300, 1000)

        }

        houses.append(house)

    return houses

```

  
  

```python

house_list(2)

```

  
  
  
  

    [{'id': 0,

      'city': 'Ottawa',

      'bedrooms': 1,

      'bathrooms': 1,

      'price ($1000s)': 726},

     {'id': 1,

      'city': 'Edmonton',

      'bedrooms': 1,

      'bathrooms': 2,

      'price ($1000s)': 357}]

  
  
  

- What happens if we want to create a list of 1,000,000 houses?

- How much time/memory will it take?

  
  

```python

start = time.time()

print(f"Memory usage before: {memory_profiler.memory_usage()[0]:.0f} MB")

  

result_list = house_list(1_000_000)

  

print(f"Memory usage after: {memory_profiler.memory_usage()[0]:.0f} MB")

print(f"Time taken: {time.time() - start:.2f}s")

```

  

    Memory usage before: 79 MB

    Memory usage after: 332 MB

    Time taken: 1.04s

  
  

```python

def house_generator(n):

    for i in range(n):

        house = {

            'id': i,

            'city': random.choice(city),

            'bedrooms': random.randint(1, 5),

            'bathrooms': random.randint(1, 3),

            'price ($1000s)': random.randint(300, 1000)

        }

        yield house

```

  
  

```python

start = time.time()

print(f"Memory usage before: {memory_profiler.memory_usage()[0]:.0f} MB")

  

result_gen = house_generator(1_000_000)

  

print(f"Memory usage after: {memory_profiler.memory_usage()[0]:.0f} MB")

print(f"Time taken: {time.time() - start:.2f}s")

```

  

    Memory usage before: 332 MB

    Memory usage after: 332 MB

    Time taken: 0.21s

  
  

```python

next(result_gen)

```

  
  
  
  

    {'id': 0,

     'city': 'Montreal',

     'bedrooms': 2,

     'bathrooms': 2,

     'price ($1000s)': 935}

  
  
  

- Although, if we used `list()` to extract all of the genertator values, we'd lose our memory savings

  
  

```python

print(f"Memory usage before: {memory_profiler.memory_usage()[0]:.0f} MB")

  

result_gen = list(house_generator(1_000_000))

  

print(f"Memory usage after: {memory_profiler.memory_usage()[0]:.0f} MB")

```

  

    Memory usage before: 332 MB

    Memory usage after: 585 MB

  

Other function design considerations:

  

- Should we print output or produce plots inside or outside functions?

  - I would usually say outside, because this is a "side effect" of sorts

  - Although there are certainly cases where I do plot or print within a function

  - In these cases I usually add a function argument such as `plot=False` or `verbose=0` that allows users to control this behaviour.

- Should the function do one thing or many things?

  - This is a tough one, hard to answer in general, depends on the situation and programming style

  

## Docstrings

  

- One problem we never really solved when talking about writing good functions was: **"4. It is hard to understand what it does just by looking at it"**

- Enter the idea of function documentation, called "docstrings"

- The [docstring](https://www.python.org/dev/peps/pep-0257/) goes right after the `def` line and is wrapped in **triple quotes** `"""`

  
  

```python

def make_palindrome(string):

    """Turns the string into a palindrome by concatenating itself with a reversed version of itself."""

    return string + string[::-1]

```

  

- In IPython/Jupyter, we can use `?` to view the documentation string of any function in our environment.

  
  

```python

make_palindrome?

```

  
  

    [0;31mSignature:[0m [0mmake_palindrome[0m[0;34m([0m[0mstring[0m[0;34m)[0m[0;34m[0m[0;34m[0m[0m

    [0;31mDocstring:[0m Turns the string into a palindrome by concatenating itself with a reversed version of itself.

    [0;31mFile:[0m      /var/folders/mp/13tr36k17_n99rc7qsh4dgxc0000gp/T/ipykernel_70234/2916775673.py

    [0;31mType:[0m      function

  
  

- But, even easier than that, if your cursor is in the function parentheses, you can use the shortcut `shift + tab` to open the docstring at will

  
  

```python

make_palindrome('uncomment and try pressing shift+tab here.')

```

  
  
  
  

    'uncomment and try pressing shift+tab here..ereh bat+tfihs gnisserp yrt dna tnemmocnu'

  
  
  

### Docstring structure

  

- General docstring convention in Python is described in [PEP 257 - Docstring Conventions](https://www.python.org/dev/peps/pep-0257/).

- There are many different docstring style conventions used in Python.

- The exact style you use can be important for helping you to render your documentation (more on that in a later course), or for helping your IDE parse your documentation.

- Common styles include:

  

1. **Single-line**: If it's short, then just a single line describing the function will do (as above).

2. **reST style**: see [here](https://www.python.org/dev/peps/pep-0287/).

3. **NumPy/SciPy style**: see [here](https://sphinxcontrib-napoleon.readthedocs.io/en/latest/example_numpy.html). (RECOMMENDED! and MDS-preferred)

4. **Google style**: see [here](https://sphinxcontrib-napoleon.readthedocs.io/en/latest/example_google.html#example-google).

  

The NumPy/Scipy style:

  
  

```python

def function_name(param1, param2, param3):

    """First line is a short description of the function.

  

    A paragraph describing in a bit more detail what the

    function does and what algorithms it uses and common

    use cases.

  

    Parameters

    ----------

    param1 : datatype

        A description of param1.

    param2 : datatype

        A description of param2.

    param3 : datatype

        A longer description because maybe this requires

        more explanation and we can use several lines.

  

    Returns

    -------

    datatype

        A description of the output, datatypes and behaviours.

        Describe special cases and anything the user needs to

        know to use the function.

  

    Examples

    --------

    >>> function_name(3,8,-5)

    2.0

    """

```

  
  

```python

def make_palindrome(string):

    """Turns the string into a palindrome by concatenating

    itself with a reversed version of itself.

  

    Parameters

    ----------

    string : str

        The string to turn into a palindrome.

  

    Returns

    -------

    str

        string concatenated with a reversed version of string

  

    Examples

    --------

    >>> make_palindrome('blah')

    'blahhalb'

    """

    return string + string[::-1]

```

  
  

```python

# make_palindrome?

print(make_palindrome.__doc__)

```

  

    Turns the string into a palindrome by concatenating

        itself with a reversed version of itself.

        Parameters

        ----------

        string : str

            The string to turn into a palindrome.

        Returns

        -------

        str

            string concatenated with a reversed version of string

        Examples

        --------

        >>> make_palindrome('blah')

        'blahhalb'

  

### Docstrings in your labs

  

In MDS we will accept:

  

- One-line docstrings for very simple functions.

- Either the PEP-8 or NumPy/SciPy style for bigger functions.

  - But we think the NumPy/SciPy style is more common in the wild so you may want to get into the habit of using it.

  

### Docstrings with optional arguments

  

- When specifying the parameters, we specify the defaults for optional arguments:

  
  

```python

# NumPy/SciPy style

def repeat_string(s, n=2):

    """

    Repeat the string s, n times.

  

    Parameters

    ----------

    s : str

        the string

    n : int, optional

        the number of times, by default = 2

  

    Returns

    -------

    str

        the repeated string

  

    Examples

    --------

    >>> repeat_string("Blah", 3)

    "BlahBlahBlah"

    """

    return s * n

```

  

### Type hinting

  

- [Type hinting](https://docs.python.org/3/library/typing.html) is exactly what it sounds like, it hints at the data type of function arguments

- You can indicate the type of an argument in a function using the syntax `argument : dtype`, and the type of the return value using `def func() -> dtype`

- Let's see an example:

  
  

```python

# NumPy/SciPy style

def repeat_string(s: str, n: int = 2) -> str:  # <- note the type hinting here

    """

    Repeat the string s, n times.

  

    Parameters

    ----------

    s : str

        the string

    n : int, optional (default = 2)

        the number of times

  

    Returns

    -------

    str

        the repeated string

  

    Examples

    --------

    >>> repeat_string("Blah", 3)

    "BlahBlahBlah"

    """

    return s * n

```

  
  

```python

repeat_string?

```

  
  

    [0;31mSignature:[0m [0mrepeat_string[0m[0;34m([0m[0ms[0m[0;34m:[0m [0mstr[0m[0;34m,[0m [0mn[0m[0;34m:[0m [0mint[0m [0;34m=[0m [0;36m2[0m[0;34m)[0m [0;34m->[0m [0mstr[0m[0;34m[0m[0;34m[0m[0m

    [0;31mDocstring:[0m

    Repeat the string s, n times.

    Parameters

    ----------

    s : str

        the string

    n : int, optional (default = 2)

        the number of times

    Returns

    -------

    str

        the repeated string

    Examples

    --------

    >>> repeat_string("Blah", 3)

    "BlahBlahBlah"

    [0;31mFile:[0m      /var/folders/mp/13tr36k17_n99rc7qsh4dgxc0000gp/T/ipykernel_70234/2733892460.py

    [0;31mType:[0m      function

  
  

- Type hinting just helps your users and IDE identify dtypes and identify bugs

- It's just another level of documentation

- They do not force users to use that dtype, for example, I can still pass an `dict` to `repeat_string` if I want to:

  
  

```python

repeat_string({'key_1': 1, 'key_2': 2})

```

  
  

    ---------------------------------------------------------------------------

  

    TypeError                                 Traceback (most recent call last)

  

    Cell In[154], line 1

    ----> 1 repeat_string({'key_1': 1, 'key_2': 2})

  

    Cell In[152], line 23, in repeat_string(s, n)

          2 def repeat_string(s: str, n: int = 2) -> str:  # <- note the type hinting here

          3     """

          4     Repeat the string s, n times.

          5

       (...)

         21     "BlahBlahBlah"

         22     """

    ---> 23     return s * n

  

    TypeError: unsupported operand type(s) for *: 'dict' and 'int'

  
  

Can we do so with numpy and pandas as well?

Of course!

  
  

```python

import numpy as np

import pandas as pd

def go_wild(s: pd.DataFrame, n:np.ndarray) -> str:

    pass

```

  

- Further, IDE's (e.g VS Code) are clever enough to even read your type hinting and warn you if you're using a different dtype in the function.

- You don't **have** to use type hinting in MDS, but it is **highly recommended** to get into the practice of doing so

  

### Automatically generated documentation

  

- As mentioned before, docstring formatting is important if you want to use standard tools for rendering your documentation into readable, accessible documents using libraries like [sphinx](http://www.sphinx-doc.org/en/master/), [pydoc](https://docs.python.org/3.7/library/pydoc.html) or [Doxygen](http://www.doxygen.nl/).

- For example: compare this [documentation](https://scikit-learn.org/stable/modules/generated/sklearn.neighbors.KNeighborsClassifier.html) with this [code](https://github.com/scikit-learn/scikit-learn/blob/1495f6924/sklearn/neighbors/classification.py#L23).

- Notice the similarities? The webpage was automatically generated because the authors used standard conventions for docstrings!

- You'll have to use some string methods to extract information from a docstring in lab 1.

- The [website for this course](https://pages.github.ubc.ca/mds-2021-22/DSCI_511_py-prog_instructors/) is built with **Jupyter Book** which leverages some of the above libraries.

  

## `try` / `except`

  

- If something goes wrong, we don't want our code to crash - we want it to **fail gracefully**.

- In Python, this can be accomplished using `try`/`except` statements

  

```python

try:

    # code that might raise an exception

except ExceptionType:

    # code to handle the exception

```

  

Here is a basic example:

  
  

```python

this_variable_does_not_exist

print("Another line")  # code fails before getting to this line

```

  
  

    ---------------------------------------------------------------------------

  

    NameError                                 Traceback (most recent call last)

  

    Cell In[156], line 1

    ----> 1 this_variable_does_not_exist

          2 print("Another line")  # code fails before getting to this line

  

    NameError: name 'this_variable_does_not_exist' is not defined

  
  
  

```python

try:

    this_variable_does_not_exist

except:

    pass  # do nothing

    print("You did something bad! But I won't raise an error.")

```

  

    You did something bad! But I won't raise an error.

  

- Python tries to execute the code in the `try` block.

- If an error is encountered, we "catch" this in the `except` block (also called `try`/`catch` in other languages).

  

- There are many different error types, or **exceptions** - we saw `NameError` above.

  
  

```python

5 / 0  # ZeroDivisionError

```

  
  

    ---------------------------------------------------------------------------

  

    ZeroDivisionError                         Traceback (most recent call last)

  

    Cell In[158], line 1

    ----> 1 5 / 0  # ZeroDivisionError

  

    ZeroDivisionError: division by zero

  
  
  

```python

my_list = [1, 2, 3]

my_list[5]  # IndexError

```

  
  

    ---------------------------------------------------------------------------

  

    IndexError                                Traceback (most recent call last)

  

    Cell In[159], line 2

          1 my_list = [1, 2, 3]

    ----> 2 my_list[5]  # IndexError

  

    IndexError: list index out of range

  
  
  

```python

my_tuple = (1, 2, 3)

my_tuple[0] = 0  # TypeError

```

  
  

    ---------------------------------------------------------------------------

  

    TypeError                                 Traceback (most recent call last)

  

    Cell In[160], line 2

          1 my_tuple = (1, 2, 3)

    ----> 2 my_tuple[0] = 0  # TypeError

  

    TypeError: 'tuple' object does not support item assignment

  
  

- Ok, so there are apparently a bunch of different errors one could run into.

- With `try`/`except` you can also catch the exception itself:

  
  

```python

try:

    this_variable_does_not_exist

except Exception as ex:

    print("You did something bad!")

    print(ex)

    print(type(ex))

```

  

    You did something bad!

    name 'this_variable_does_not_exist' is not defined

    <class 'NameError'>

  

- In the above, we caught the exception and assigned it to the variable `ex` so that we could print it out.

- This is useful because you can see what the error message would have been, without crashing your program.

  

- You can also catch specific exceptions types

- This is typically the recommended way to catch errors, you want to be specific in catching your error so you know exactly where and why your code failed.

  
  

```python

try:

#     this_variable_does_not_exist  # name error

#     (1, 2, 3)[0] = 1  # type error

    5/0  # ZeroDivisionError

except TypeError:

    print("You made a type error!")

except NameError:

    print("You made a name error!")

except:

    print("You made some other sort of error")

```

  

    You made some other sort of error

  

- The last `except` would trigger if the error is none of the above types.

  

- There is also an optional `else` and `finally` keyword, read more [here](https://docs.python.org/3/tutorial/errors.html)

  
  

```python

try:

    x = int(input("Enter x:"))

    y = int(input("Enter y:"))

    z = x / y

except ValueError:

    print("You didn't enter a number!")

except ZeroDivisionError:

    print("Why would you divide by zero?")

else:

    print("z**2 = ", z**2)

finally:

    print("Here I am anyway 😈")

```

  

    Enter x: 2

    Enter y: 3

  

    z**2 =  0.4444444444444444

    Here I am anyway 😈

  

- The `finally` clause will always get executed.

  

- We can also write code that raises an exception on purpose, using `raise`

  
  

```python

def add_one(x):  # we'll get to functions in the next section

    return x + 1

```

  
  

```python

add_one("blah")

```

  
  

    ---------------------------------------------------------------------------

  

    TypeError                                 Traceback (most recent call last)

  

    Cell In[170], line 1

    ----> 1 add_one("blah")

  

    Cell In[169], line 2, in add_one(x)

          1 def add_one(x):  # we'll get to functions in the next section

    ----> 2     return x + 1

  

    TypeError: can only concatenate str (not "int") to str

  
  
  

```python

def add_one(x):

    if not isinstance(x, float) and not isinstance(x, int):

        raise TypeError(f"Sorry, x must be numeric, you entered a {type(x)}.")

  

    return x + 1

```

  
  

```python

add_one("blah")

```

  
  

    ---------------------------------------------------------------------------

  

    TypeError                                 Traceback (most recent call last)

  

    Cell In[172], line 1

    ----> 1 add_one("blah")

  

    Cell In[171], line 3, in add_one(x)

          1 def add_one(x):

          2     if not isinstance(x, float) and not isinstance(x, int):

    ----> 3         raise TypeError(f"Sorry, x must be numeric, you entered a {type(x)}.")

          5     return x + 1

  

    TypeError: Sorry, x must be numeric, you entered a <class 'str'>.

  
  

- Finally, we can even define our own exception types.

- We do this by inheriting from the `Exception` class (more on classes and inheritance next lecture)

  
  

```python

class CustomAdditionError(Exception):

    pass

```

  
  

```python

def add_one(x):

    if not isinstance(x, float) and not isinstance(x, int):

        raise CustomAdditionError("Sorry, x must be numeric")

  

    return x + 1

```

  
  

```python

add_one("blah")

```

  
  

    ---------------------------------------------------------------------------

  

    CustomAdditionError                       Traceback (most recent call last)

  

    Cell In[178], line 1

    ----> 1 add_one("blah")

  

    Cell In[177], line 3, in add_one(x)

          1 def add_one(x):

          2     if not isinstance(x, float) and not isinstance(x, int):

    ----> 3         raise CustomAdditionError("Sorry, x must be numeric")

          5     return x + 1

  

    CustomAdditionError: Sorry, x must be numeric

  
  

- This is useful when your function is complicated and would fail in a complicated way, with a weird error message.

- You can make the cause of the error much clearer to the _caller_ of the function.

- Thus, your function is more usable this way.

- If you do this, you should ideally describe these exceptions in the function documentation, so a user knows what to expect if they call your function.  

  

## Intriguing behaviour in Python

  

### References

  

- What do you think the code below will print?

  
  

```python

x = 100

y = x

x = 2

# y

```

  
  

```python

y

```

  
  
  
  

    100

  
  
  

- And how about the next one?

  
  

```python

x = [100]

y = x

x[0] = 2

# y

```

  
  

```python

y

```

  
  
  
  

    [2]

  
  
  

- In Python, the list `x` is a **reference** to an object in the computer's memory.

- When you set `y = x` these two variables now refer to the same object in the memory - the one that `x` referred to.

- Setting `x[0] = 2` modifies the object in memory. So `x` and `y` are both modified.

  - It makes no different if you set `x[0] = 2` or `y[0] = 2`, both modify the same place in the memory.

  
  

```python

x = [100]

y = x

x[0] = 2

y

```

  
  
  
  

    [2]

  
  
  

- However, some basic built-in types `int`, `float`, `bool` etc are _exceptions_ to this logic:

  - When you set `y = x` it actually copies the value `1`, so `x` and `y` are decoupled.

  - Thus, the list example is actually the typical case, the integer example is the "special" case.

  
  

```python

x = 100

y = x

x = 2

y

```

  
  
  
  

    100

  
  
  

- Analogy:

  - I share a Dropbox folder (or git repo) with you, and you modify it -- I sent you _the location of the stuff_ (this is like the list case)

  - I send you an email with a file attached, you download it and modify the file -- I sent you _the stuff itself_ (this is like the integer case)

- [This article](https://realpython.com/pointers-in-python/#names-in-python) does a great job of explaining all this in detail if you'd like to know more

  

- What do you think will happen here:

  
  

```python

x = [100]

y = x

x = [2]  # before we had x[0] = 2

y

```

  
  
  
  

    [100]

  
  
  

- Here we are not modifying the contents of `x`, we are setting `x` to refer to a new list `[2]`. In fact, we are re-creating `x`.

  

### Additional weirdness

  

- We can use `id()` to return the unique id of an object in memory

  
  

```python

x = np.array([1, 2, 3, 4, 5])  # this is a numpy array which we'll see next lecture

y = x

x = x + 5

  

print(f"x has the value: {x}, id: {id(x)}")

print(f"y has the value: {y}, id: {id(y)}")

```

  

    x has the value: [ 6  7  8  9 10], id: 13392695664

    y has the value: [1 2 3 4 5], id: 13392698640

  
  

```python

x = np.array([1, 2, 3, 4, 5])

y = x

x += 5

  

print(f"x has the value: {x}, id: {id(x)}")

print(f"y has the value: {y}, id: {id(y)}")

```

  

    x has the value: [ 6  7  8  9 10], id: 13392698448

    y has the value: [ 6  7  8  9 10], id: 13392698448

  

- So, it turns out `x += 5` is not identical `x = x + 5`.

- `x += 5` modifies the contents of `x`.

- `x = x + 5` first assigns `x + 5` to a new array of the same size, and then overwrites the name `x` with a reference to this new array.

  

- **But there's good news**: we don't need to memorize special rules for calling functions.

- Copying happens with `int`, `float`, `bool`, (maybe some other ones I'm forgetting?),  the rest is "by reference"

- Now you see why we care if objects are mutable or immutable... passing around a reference can be dangerous!

- **General rule**: if you do `x = ...` then you're not modifying the original, but if you do `x.SOMETHING = y` or `x[SOMETHING] = y` or `x += y` then you probably are.

  

### `copy` and `deepcopy`

  

- We can force the certain copying behaviour using the `.copy()` method of lists if we want to

  
  

```python

x = [100]

y = x

x[0] = 2

y

```

  
  
  
  

    [2]

  
  
  
  

```python

x = [100]

y = x.copy()  # We "copied" x and saved that new object as y

x[0] = 2

y

```

  
  
  
  

    [100]

  
  
  

- Ok, so what do you think will happen here?

  
  

```python

x = [[1], [2, 99], [3, "hi"]]  # a list of lists

  

y = x.copy()

print("After .copy():")

print(x)

print(y)

```

  
```
    After .copy():

    [[1], [2, 99], [3, 'hi']]
    
    [[1], [2, 99], [3, 'hi']]
```






```python

x[0][0] = "pikachu"

print("")

print("After modifying x:")

print(x)

print(y)

```

  
```
    After modifying x:

    [['pikachu'], [2, 99], [3, 'hi']]

    [['pikachu'], [2, 99], [3, 'hi']]
```


  

- But wait.. we used `.copy()`, why are `x` and `y` both changed in the latter example?

- `.copy()` makes the _containers_ different, i.e., only the outer list.

- But the outer lists contain references to objects which were not copied!

- This is what happens after `y = x.copy()`:

  

![](img/copy.png)

  

- We can use `is` to tell apart these scenarios (as opposed to `==`)

- `is` tells us if two objects are referring to the same object in memory, while `==` tells us if their contents are the same

  
  

```python

x == y # they are both lists containing the same lists

```

  
  
  
  

    True

  
  
  
  

```python

x is y # but they are not the *same* lists of lists

```

  
  
  
  

    False

  
  
  

- So, by that logic we should be able to append to `y` without affecting `x`

  
  

```python

y.append(5)

  

print(x)

print(y)

```

  

    [['pikachu'], [2, 99], [3, 'hi']]

    [['pikachu'], [2, 99], [3, 'hi'], 5]

  
  

```python

x == y

```

  
  
  
  

    False

  
  
  

- That makes sense, as weird as it seems:

  

![](img/copy-append.png)

  

- In short, `copy` only copies **one level down**.

- What if we want to copy everything? i.e., even the inner lists in our outer list...

- Enter our friend `deepcopy` from the `copy` package (which is part of the standard library):

  
  

```python

import copy

  

x = [[1], [2, 99], [3, "hi"]]

y = copy.deepcopy(x)

  

x[0][0] = "pikachu"

print(x)

print(y)

```

  

    [['pikachu'], [2, 99], [3, 'hi']]

    [[1], [2, 99], [3, 'hi']]

  

![](img/deep-copy.png)

  

- Find a whole compilation of more intriguing behaviour in Python [here](https://github.com/satwikkansal/wtfpython/blob/master/README.md)!

  

## Unit tests

  

- We just talked about Python functions

- But how can we be sure that our function is doing exactly what we expect it to do?

- **Unit testing** is the process of testing our function to ensure it's giving us the results we expect

- You'll explore testing in more detail in DSCI 524, including automating testing and designing robust testing regimes

- Let's briefly introduce the concept here

  

### `assert` statements

  

- `assert` statements are the most common way to test your functions

- They cause your program to fail if the tested condition is `False`

- The syntax is:

  

```python

assert expression, "Error message if expression is False or raises an error."

```

  
  

```python

assert 1 == 2, "1 is not equal to 2."

```

  
  

    ---------------------------------------------------------------------------

  

    AssertionError                            Traceback (most recent call last)

  

    Cell In[203], line 1

    ----> 1 assert 1 == 2, "1 is not equal to 2."

  

    AssertionError: 1 is not equal to 2.

  
  

- Asserting that two numbers are approximately equal can also be helpful

- Due to the limitations of floating-point arithmetic in computers, numbers we expect to be equal are sometimes not (more on that in DSCI 572)

  
  

```python

assert 0.1 + 0.2 == 0.3, "Not equal!"

```

  
  

    ---------------------------------------------------------------------------

  

    AssertionError                            Traceback (most recent call last)

  

    Cell In[80], line 1

    ----> 1 assert 0.1 + 0.2 == 0.3, "Not equal!"

  

    AssertionError: Not equal!

  
  
  

```python

import math

assert math.isclose(0.1 + 0.2, 0.3, abs_tol = 0.001), "Not equal!"

```

  

- You can test any statement that evaluates to a boolean

  
  

```python

assert 'hedayat' in ['scot', 'prajeet', 'hedayat'], "Instructor not present!"

```

  

### Best practices when writing unit test

  

The AAA (Arrange-Act-Assert) principle

  

- Arrange your objects, create and set them up as necessary.

- Act on an object.

- Assert that something is as expected.

  

**Why?**

- Clearly separates what is being tested from the arrange and assert steps.

- Less chance to intermix assertions with "Act" code.

  
  

```python

def sum(a,b):

    return a + b

  

# Not recommended

assert sum(1,2) == 3

  

# Arrange

a = 1

b = 2

  

# Act

result = sum(a,b)

  

# Assert

assert result == 3

```

  

### Best practices when writing unit test

  

Precise assertion is a double-edged sword

  
  

```python

solution = 'Something here'

answer = 'Something Here '

  

# assert solution == answer

# assert solution in answer

assert solution.lower() in answer.lower()

```

  
  

```python

solution = 5/3

answer = 1.666666667

  

# assert solution == answer

import math

assert math.isclose(solution, answer, abs_tol = 0.01)

```

  
  

```python

solution = ['orange', 'apple', 'banana']

answer = ['apple', 'orange', 'banana']

  

# Assuming we just want to test membership and don't care about order of items

  

# assert solution == answer

# assert answer[0] == 'orange'

  

assert 'orange' in answer

assert 'banana' in answer

assert 'apple' in answer

```

  

### Best practices when writing unit test

  

Avoid logic in tests

  

When writing your unit tests, avoid manual string concatenation, logical conditions, such as if, while, for, and switch, and other conditions.

  

**Why?**

  

- Less chance to introduce a bug inside of your tests.

- Focus on the end result, rather than implementation details.

  
  

```python

def odd_or_even(number):

    if number % 2 == 0:

        return 'even'

    else:

        return 'odd'

import random

number = random.randint(0,10)

  

# not recommended

if number % 2 == 1:

    assert odd_or_even(number) == 'odd'

else:

    assert odd_or_even(number) == 'even'

```

  
  

```python

# Arrange

odd_number = random.randrange(1, 11, 2)

  

# Act

result = odd_or_even(odd_number)

  

# Assert

assert result == 'odd'

  

# Arrange

odd_number = random.randrange(2, 12, 2)

  

# Act

result = odd_or_even(odd_number)

  

# Assert

assert result == 'even'

```

  

### Test driven development

  

- Test Driven Development (TDD) is where you write your tests before your actual function

- This may seem a little counter-intuitive, but you're creating the expectations of your function before the actual function

- This can be helpful for several reasons:

    - you will better understand exactly what code you need to write;

    - you are forced to write tests upfront;

    - you will not encounter large time-consuming bugs down the line; and,

    - it helps to keep your workflow manageable by focusing on small, incremental code improvements and additions.

  

- In general, the approach is as follows:

    1. Write a stub: a function that does nothing but accept all input parameters and return the correct datatype.

    2. Write tests to satisfy your design specifications.

    3. Outline the program with pseudo-code.

    4. Write code and test frequently.

    5. Write documentation.

  

- **You do not have to do TDD in MDS**, but you may find it helpful, especially when it comes to designing more complex programs/packages.

  

### Testing woes - false positives

  

- **Just because all your tests pass, this does not mean your program is correct!!**

- This happens all the time. How to deal with it?

  - Write a lot of tests!

  - Write documentation.

  - Don't be overconfident, even after writing a lot of tests!

  
  

```python

def sample_median(x):

    """Finds the median of a list of numbers."""

    x_sorted = sorted(x)

    return x_sorted[len(x_sorted) // 2]

  
  

assert sample_median([1, 3, 2]) == 2, "test failed!"

assert sample_median([0, 0, 0, 0]) == 0, "test failed!"

assert sample_median([1, 2, 3, 4, 5]) == 3, "test failed!"

```

  

- Looks like our tests passed! We must be good to go...

- But wait...

  
  

```python

assert sample_median([1, 2, 3, 4]) == 2.5, "test failed!"

```

  
  

    ---------------------------------------------------------------------------

  

    AssertionError                            Traceback (most recent call last)

  

    Cell In[99], line 1

    ----> 1 assert sample_median([1, 2, 3, 4]) == 2.5, "test failed!"

  

    AssertionError: test failed!

  
  

### Corner cases

  

- A **corner case** is an input that is reasonable but a bit unusual, and may trip up your code.

- For example, taking the median of an empty list, or a list with only one element.

- Often it is desirable to add test cases to address corner cases.

  
  

```python

assert sample_median([1]) == 1

```

  

- In this case the code worked with no extra effort, but sometimes we need `if` statements to handle the weird cases.

- For example, sometimes we want the code to throw a particular error

- You'll learn about writing tests for code that raises a specified error in DSCI 524

  

### EAFP versus LBYL

  

- Somewhat related to testing and function design are the philosophies EAFP and LBYL

- EAFP = "Easier to ask for forgiveness than permission"

    - In coding lingo: try doing something, and if it doesn't work, catch the error

- LBYL = "Look before you leap"

    - In coding lingo: check that you can do something before trying to do it

- These two acronyms refer to coding philosophies about how to write your code

- Let's see an example

  
  

```python

d = {'name': 'Doctor Python',

     'superpower': 'programming',

     'weakness': 'mountain dew',

     'enemies': 10}

```

  
  

```python

# EAFP

try:

    d['address']

except KeyError:

    print('Please forgive me!')

```

  

    Please forgive me!

  
  

```python

# LBYL

if 'address' in d.keys():

    d['address']

else:

    print('Saved you before you leapt!')

```

  

    Saved you before you leapt!

  

- While EAFP is often vouched for in Python, there's no right and wrong way to code and it's often context-specific

  

## Debugging

  

- My Python code doesn't work: what do I do?

- At the moment, most of you probably do "manual testing" or "exploratory testing"

- You keep changing your code until it works, maybe add some `print()` statements around the place to isolate any problems

  

For example, look at the following `random_walker` code, which is adopted with permission from COS 126, [Conditionals and Loops](http://www.cs.princeton.edu/courses/archive/fall10/cos126/assignments/loops.html):

  
  

```python

from random import random

```

  
  

```python

def random_walker(T):

  

    x = 0

    y = 0

  

    for i in range(T):

        rand = random()

        if rand < 0.25:

            x += 1

        if rand < 0.5:

            x -= 1

        if rand < 0.75:

            y += 1

        else:

            y -= 1

        print((x, y))

  

    return round((x ** 2 + y ** 2) ** 0.5, 2)

  
  

random_walker(5)

```

  

    (0, 1)

    (0, 2)

    (0, 3)

    (0, 4)

    (0, 5)

  
  
  
  

    5.0

  
  
  

- If we re-run the code above, our random walker never goes right (the x-coordinate is never positive)...

- We might try to add some print statement here to see what's going on

  
  

```python

from random import random

```

  
  

```python

def random_walker(T):

    """

    Simulates T steps of a 2D random walk, and prints the result of each step.

    Returns the squared distance from the origin.

  

    Parameters

    ----------

    T : int

        Number of steps to take

  

    Returns

    -------

    out : float

        Euclidean distance from the origin rounded to 2 decimal places

  

    Examples

    --------

    >>> random_walker(1)

    1.0

  

    >>> random_walker(1)

    1.41  # this randomly gives 1.41, 2.0, or 0.0

    """

  

    x = 0

    y = 0

  

    for i in range(T):

        rand = random()

        print(rand)

        if rand < 0.25:

            print("I'm going right!")

            x += 1

        if rand < 0.5:

            print("I'm going left!")

            x -= 1

        if rand < 0.75:

            y += 1

            print("I'm going up!")

        else:

            print("I'm going down!")

            y -= 1

        print((x, y), '\n')

  

    return round((x ** 2 + y ** 2) ** 0.5, 2)

  
  

random_walker(5)

```

  

    0.38842962923836344

    I'm going left!

    I'm going up!

    (-1, 1)

    0.37795761762894553

    I'm going left!

    I'm going up!

    (-2, 2)

    0.28895266518989027

    I'm going left!

    I'm going up!

    (-3, 3)

    0.08617485947063797

    I'm going right!

    I'm going left!

    I'm going up!

    (-3, 4)

    0.7686981222633457

    I'm going down!

    (-3, 3)

  
  
  
  

    4.24

  
  
  

- Ah! We see that even every time after a `"I'm going right!"` we immediately get a `"I'm going left!"` and a `"I'm going up!"`

- Note that a left or right move is always followed by an up move as well!

- The problem is in our `if` statements, we should be using `elif` for each statement after the initial `if`, otherwise multiple conditions may be met each time...

  

- This was a pretty simple debugging case, adding print statements is not always helpful or efficient

- Alternative: Use debugger feature in VScode (https://code.visualstudio.com/docs/editor/debugging)

  

<img src="https://code.visualstudio.com/assets/docs/editor/debugging/debugging_hero.png" width="700">

  
  

```python

def random_walker(T):

  

    x = 0

    y = 0

  

    for i in range(T):

        rand = random()

        if rand < 0.25:

            x += 1

        if rand < 0.5:

            x -= 1

        if rand < 0.75:

            y += 1

        else:

            y -= 1

        print((x, y), '\n')

    return round((x ** 2 + y ** 2) ** 0.5, 2)

  
  

random_walker(5)

```

  

    (0, 1)

    (0, 2)

    (0, 3)

    (0, 4)

    (0, 3)

  
  
  
  

    3.0

  
  
  

- So the correct code should be:

  
  

```python

from random import random

  

def random_walker(T):

    """

    Simulates T steps of a 2D random walk, and prints the result of each step.

    Returns the squared distance from the origin.

  

    Parameters

    ----------

    T : int

        Number of steps to take

  

    Returns

    -------

    out : float

        Euclidean distance from the origin rounded to 2 decimal places

  

    Examples

    --------

    >>> random_walker(1)

    1.0

  

    >>> random_walker(1)

    1.41  # this randomly gives 1.41, 2.0, or 0.0

    """

  

    x = 0

    y = 0

  

    for i in range(T):

        rand = random()

        # print(rand)

        if rand < 0.25:

            print("I'm going right!")

            x += 1

        elif rand < 0.5:

            print("I'm going left!")

            x -= 1

        elif rand < 0.75:

            print("I'm going up!")

            y += 1

        else:

            print("I'm going down!")

            y -= 1

        print((x, y), '\n')

    return round((x ** 2 + y ** 2) ** 0.5, 2)

  
  

random_walker(5)

```

  

    I'm going right!

    (1, 0)

    I'm going down!

    (1, -1)

    I'm going right!

    (2, -1)

    I'm going right!

    (3, -1)

    I'm going down!

    (3, -2)

  
  
  
  

    3.61

  
  
  

- Most Python IDE's also have their own debugging workflow, including the visual debugger of VSCode and JupyterLab.