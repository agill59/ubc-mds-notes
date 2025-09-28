```python
# Initialize Otter
import otter
grader = otter.Notebook("worksheet5.ipynb")
```

# Worksheet 5 - NumPy


```python
import numpy as np
```

### Exercise 1
rubric={autograde:1}

Create the following array and save it as a variable named `ex1`:

```python
array([[1, 2],
       [3, 4],
       [5, 6]])
```


```python
ex1 = np.array([[1,2],[3,4],[5,6]])
ex1
```




    array([[1, 2],
           [3, 4],
           [5, 6]])




```python
grader.check("ex1")
```




<p><strong><pre style='display: inline;'>ex1</pre></strong> passed! 🌈</p>



### Exercise 2

Create a 1D array of 5 zeros and save it as a variable named `ex2`:


```python
ex2 = np.zeros(5)
```


```python
grader.check("ex2")
```




<p><strong><pre style='display: inline;'>ex2</pre></strong> passed! 🎉</p>



### Exercise 3

Create a 1D array of ten 1's and save it as a variable named `ex3`:


```python
ex3 = np.ones(10)
```


```python
grader.check("ex3")
```




<p><strong><pre style='display: inline;'>ex3</pre></strong> passed! 🎉</p>



### Exercise 4

Create a 1D array of five numbers, each having the value of 3.141, and save it as a variable named `ex4`:


```python
ex4 = np.repeat(3.141,5)
```


```python
grader.check("ex4")
```




<p><strong><pre style='display: inline;'>ex4</pre></strong> passed! 🍀</p>



### Exercise 5

Create a 1D array of the integers 1 to 20 and save it as a variable named `ex5`:


```python
ex5 = np.arange(1,21,1)
```


```python
grader.check("ex5")
```




<p><strong><pre style='display: inline;'>ex5</pre></strong> passed! 🍀</p>



### Exercise 6

Create a 5 x 5 matrix of ones with a dtype `int` and save it as a variable named `ex6`:


```python
ex6 = np.ones([5,5], int)
```


```python
grader.check("ex6")
```




<p><strong><pre style='display: inline;'>ex6</pre></strong> passed! ✨</p>



### Exercise 7

Create an 3D matrix of 3 x 3 x 3 full of random numbers drawn from a standard normal distribution (hint: `np.random.randn()`) and save it as a variable named `ex7`:

> Hint: To ensure the code reproducibility, set the random seed = 42 using the `np.random.seed()` function.


```python
# set your seed to 42 for reproducibility
np.random.seed(42)
ex7 = np.random.randn(3,3,3)
```


```python
grader.check("ex7")
```




<p><strong><pre style='display: inline;'>ex7</pre></strong> passed! 🚀</p>



### Exercise 8

Reshape the above array (`ex7`) into shape `(27,)` and save it as a variable named `ex8`:


```python
ex8 = ex7.reshape(27,)
```


```python
grader.check("ex8")
```




<p><strong><pre style='display: inline;'>ex8</pre></strong> passed! 💯</p>



### Exercise 9

Reshape the following `arr1` into shape `(6, 1)` and save it as a variable named `ex9`:


```python
arr1 = np.array([[1, 2], [3, 4], [5, 6]])
arr1
```




    array([[1, 2],
           [3, 4],
           [5, 6]])




```python
ex9 = arr1.reshape(6,1)
```


```python
grader.check("ex9")
```




<p><strong><pre style='display: inline;'>ex9</pre></strong> passed! 🙌</p>



### Exercise 10

Create a `(6, 6)` array by multiplying the following `arr2` by itself.

> Hint: transpose the array to leverage broadcasting here.

The first element (top left corner) should be 1, and the last element (bottom right corner) should be 36. Save it as a variable named `ex10`.


```python
arr2 = np.arange(1, 7)[np.newaxis, :] # np.newaxis is an alias for None
arr2
```




    array([[1, 2, 3, 4, 5, 6]])




```python
ex10 = arr2 * arr2.reshape(6,1)
ex10
```




    array([[ 1,  2,  3,  4,  5,  6],
           [ 2,  4,  6,  8, 10, 12],
           [ 3,  6,  9, 12, 15, 18],
           [ 4,  8, 12, 16, 20, 24],
           [ 5, 10, 15, 20, 25, 30],
           [ 6, 12, 18, 24, 30, 36]])




```python
grader.check("ex10")
```




<p><strong><pre style='display: inline;'>ex10</pre></strong> passed! 🎉</p>



### Exercise 11

"Center" the following `arr3` by 
1. subtracting the mean from every element, 
2. then flatten it to shape `(20,)`, and
3. use `numpy.sort` to sort it in descending order. 

Save it as a variable named `ex11`.

> Hint: [`numpy.sort`](https://numpy.org/doc/stable/reference/generated/numpy.sort.html) by default sorts in the ascending order.


```python
arr3 = np.arange(1, 21).reshape(4, 5)
arr3
```




    array([[ 1,  2,  3,  4,  5],
           [ 6,  7,  8,  9, 10],
           [11, 12, 13, 14, 15],
           [16, 17, 18, 19, 20]])




```python
ex11 = np.sort((arr3 - np.mean(arr3)).reshape(20,))[::-1]
ex11
```




    array([ 9.5,  8.5,  7.5,  6.5,  5.5,  4.5,  3.5,  2.5,  1.5,  0.5, -0.5,
           -1.5, -2.5, -3.5, -4.5, -5.5, -6.5, -7.5, -8.5, -9.5])




```python
grader.check("ex11")
```




<p><strong><pre style='display: inline;'>ex11</pre></strong> passed! 🙌</p>




```python
(np.ones([2,3]) + np.ones([2,1,3])).shape
```




    (2, 2, 3)



Congratulations! You are done the worksheet!!! Pat yourself on the back and submit your worksheet to Gradescope!
