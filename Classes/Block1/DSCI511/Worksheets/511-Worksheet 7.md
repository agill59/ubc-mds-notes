```python
# Initialize Otter
import otter
grader = otter.Notebook("worksheet7.ipynb")
```

# Worksheet 7 - Functions & Testing

These exercises complement [DSCI 511 lecture 7](../lecture7-functions-testing.ipynb).


```python
import pytest
import math
import unittest
import io
from contextlib import redirect_stdout
```

## 1. Functions

### 1.1 Website domain

Create a function `website()` that grabs the website domain from a url string. For example, if your function is passed `"www.google.com"`, it should return `"google"`.

Hint: You can use the `split()` built-in function to parse a url


```python
def website(url):
    domain = str(str.split(url, sep = ".")[1])
    return domain

```


```python
# TEST CASE
website('www.apple.com') # EXPECT OUTPUT: 'apple'
```




    'apple'




```python
grader.check("q1.1")
```




<p><strong><pre style='display: inline;'>q1.1</pre></strong> passed! 💯</p>



### 1.2 Divisible?

Create a function `divisible(a, b)` that accepts two integers (`a` and `b`) and returns `True` if `a` is divisble by `b` without a remainder. For example, `divisible(10, 3)` should return `False`, while `divisible(6, 3)` should return `True`.


```python
def divisible(a, b):
    if a % b == 0:
        return True
    else: 
        return False

print(divisible(10, 3)) # output should be FALSE
print(divisible(6, 3)) # output should be TRUE
```

    False
    True



```python
grader.check("q1.2")
```




<p><strong><pre style='display: inline;'>q1.2</pre></strong> passed! 🎉</p>



### 1.3 Generators

This question is a little harder. Create a generator function called `listgen(n)` that yields numbers from 0 to n, in batches of lists of maximum 10 numbers at a time. For example, your function should behave as follows:

```python
for batch in listgen(25):
    print(batch)
    
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
[10, 11, 12, 13, 14, 15, 16, 17, 18, 19]
[20, 21, 22, 23, 24]
```


```python

def listgen(n):
    temp_list = []
    full_list = []
    loops = math.floor(n / 10)
    for i in range(0, loops+1):
        for l in range(i*10,i*10+10):
            if (l < n):
                temp_list.append(l)
        full_list.append(temp_list)
        if (len(temp_list) == 10):
            temp_list = []
    full_list = iter(full_list)
    return full_list


for batch in listgen(25):
    print(batch)
```

    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
    [10, 11, 12, 13, 14, 15, 16, 17, 18, 19]
    [20, 21, 22, 23, 24]



```python
grader.check("q1.3")
```




<p><strong><pre style='display: inline;'>q1.3</pre></strong> passed! 🙌</p>



### 1.4 *args and **kwargs

Create a function `lucky_sum()` that takes all the integers a user enters and returns their sum. *However*, if one of the values is 13 then it does not count towards the sum, nor do any values to its right.

For example, your function should behave as follows:

```python
lucky_sum(1, 2, 3, 4)
10

lucky_sum(1, 13, 3, 4)
1

lucky_sum(13)
0
```

*This example is inspired by the related [codingbat challenge](https://codingbat.com/prob/p130788).*


```python
def lucky_sum(*args):
    sum = 0
    for num in args:
        if (num == 13):
            return sum
        else:
            sum+=num
    return sum
```


```python
grader.check("q1.4")
```




<p><strong><pre style='display: inline;'>q1.4</pre></strong> passed! 🌈</p>



### 2.1 Unit tests

rubric = {autograde: 1}

The function `area_2_1()` accepts the argument `radius` and calculates the area of a circle. In the function `unit_test_area_2_1()` below, write three tests using `assert` statements for the following conditions:

1. Check if the function `area_2_1()` returns the correct data type
2. Check if the function `area_2_1()` returns the correct calculation up to 2 decimals with a input value of your choice
3. Check if the function `area_2_1()` returns the correct calculation up to 2 decimals with a different input value

(hint: `math.isclose(..., abs_tol=0.01)`)


```python
def area_2_1(radius):
    """Calculate the area of a circle based on the given radius."""
    return math.pi * radius ** 2
    
def unit_test_area_2_1():
    try:
        assert type(area_2_1) == float
    except TypeError:
        print("Wrong type")
    assert math.isclose(area_2_1(math.pi), math.pi**3, abs_tol=0.01), "incorrect"
    assert math.isclose(area_2_1(1), math.pi, abs_tol=0.01), "yes"


```


```python
grader.check("q2.1")
```




<p><strong><pre style='display: inline;'>q2.1</pre></strong> passed! 🎉</p>



### 2.2 EAFP unit test

rubric = {autograde: 1}

In the spirit of the EAFP (easier to ask for forgiveness than permission) philosophy. Modify the code of `area_2_1()` to create a new function `area_2_2()` by adding a `try`/`except` statement to catch the `TypeError` if users pass a non-numeric input. Print out a custom message "radius should be a number but you entered a X" where X is the data type of the input object. For example:

```python
area_2_2('10')  # EXPECTED OUTPUT: radius should be a number but you entered a <class 'str'>
```

Note: You should print out an error message instead of raising an error


```python
def area_2_2(radius):
    try:
        radius + 1
        return math.pi * radius ** 2
    except TypeError:
        print(f"EXPECTED OUTPUT: radius should be a number but you entered a {type(radius)}")
```




    50.26548245743669




```python
grader.check("q2.2")
```




<p><strong><pre style='display: inline;'>q2.2</pre></strong> passed! 🚀</p>



### 2.3 LBYL unit test

rubric = {autograde: 1}

In the spirit of the LBYL (look before you leap) philosophy. This time, modify the code of `area_2_1()` and create a new function `area_2_3()` by adding a conditional `if`/`else` statement to make sure that a user has passed a number (`int` or `float`). If they pass something else, raise a `TypeError` with a message of your choice. For example:

```python
area_2_3('10') # EXPECTED OUTPUT: TypeError: Your input is not a number!
```

Note: You should raise an error instead of printing out an error message.

**Hint:** Use the built-in Python function `isinstance(obj, (class1, class2, ...)` to inspect input object types.


```python
def area_2_3(radius):
        if (isinstance(radius, (int, float))):
            return math.pi * radius ** 2
        else:
            raise TypeError
```


```python
grader.check("q2.3")
```




<p><strong><pre style='display: inline;'>q2.3</pre></strong> passed! 🌈</p>


