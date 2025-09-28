```python
# Initialize Otter
import otter
grader = otter.Notebook("worksheet6.ipynb")
```

## Worksheet 6 - Review of data types, as well as how to do conditionals and iteration in Python


```python
import pandas as pd
import numpy as np
from hashlib import sha1
```

### Exercise 1

rubric={autograde:1}

Given this nested list, use indexing to grab the string "MDS". Store this in a variable named `ex1`.


```python
my_nested_list = [10, [3, 4], [5, [100, 200, ["MDS"]], 23, 11], 1, 7]
my_nested_list
```




    [10, [3, 4], [5, [100, 200, ['MDS']], 23, 11], 1, 7]




```python
ex1 = my_nested_list[2][1][2][0]
ex1
```




    'MDS'




```python
grader.check("ex1")
```




<p><strong><pre style='display: inline;'>ex1</pre></strong> passed! 🌈</p>



### Exercise 2

rubric={autograde:1}

Given this nested dictionary grab the word "MDS". Store this in a variable named `ex2`.


```python
my_nested_dictionary = {
    "outer": [
        1,
        2,
        3,
        {"inner": ["this", "is", "inception", {"inner_inner": [1, 2, 3, "MDS"]}]},
    ]
}
my_nested_dictionary
```




    {'outer': [1,
      2,
      3,
      {'inner': ['this', 'is', 'inception', {'inner_inner': [1, 2, 3, 'MDS']}]}]}




```python
ex2 = my_nested_dictionary["outer"][3]["inner"][3]["inner_inner"][3]
ex2
```




    'MDS'




```python
grader.check("ex2")
```




<p><strong><pre style='display: inline;'>ex2</pre></strong> passed! 🍀</p>



### Exercise 3

rubric={autograde:1}

The code below is trying to append the number 6 to the variable `my_sequence` below. It is not working and will result in an error when run. 

Fix the code so that it will work correctly, and as such  `my_sequence` will contain the numbers from 1 to 6, by ones.


```python
my_sequence = (1, 2, 3, 4, 5)
```


```python
my_sequence = list(my_sequence)
my_sequence
```




    [1, 2, 3, 4, 5]




```python
my_sequence.append(6)
my_sequence
```




    [1, 2, 3, 4, 5, 6]




```python
grader.check("ex3")
```




<p><strong><pre style='display: inline;'>ex3</pre></strong> passed! 💯</p>



### Exercise 4

rubric={autograde:1}

Write a `for` loop that will take the list of integers given below and return a list of squared integers. Name the variable you create `squared_integers`.


```python
integers = [1, 3, 5, 7, 9]
integers
```




    [1, 3, 5, 7, 9]




```python
squared_integers = []
for i in integers:
    squared_integers.append(int(np.square(i)))
squared_integers
```




    [1, 9, 25, 49, 81]




```python
grader.check("ex4")
```




<p><strong><pre style='display: inline;'>ex4</pre></strong> passed! 🎉</p>



### Exercise 5

rubric={autograde:1}

Write a code to count the number of vowels in the word "supercalifragilisticexpialidocious". Name the variable you store your answer in `vowels`. 


```python
word = "supercalifragilisticexpialidocious"
word
```




    'supercalifragilisticexpialidocious'




```python
vowels = 0
for letter in word:
    if letter in ['a','e','i','o','u']:
        vowels += 1
```


```python
print(f"There are {vowels} vowels in the word {word}")
```

    There are 16 vowels in the word supercalifragilisticexpialidocious



```python
grader.check("ex5")
```




<p><strong><pre style='display: inline;'>ex5</pre></strong> passed! 🎉</p>



### Exercise 6

rubric={autograde:1}

Given a list of numbers from 1 to 100, use a list comprehension to create a new list named `even_numbers` that contains only the even numbers from this list [2, 4, 6, 8, 10, ..., 100].


```python
numbers = list(range(1, 101)) # generate a list of numbers from 1 to 100
numbers[:10] # look at the first 10 numbers
```




    [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]




```python
even_numbers = [num for num in numbers if num % 2 == 0] 
even_numbers[:10] # look at the first 10 numbers
```




    [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]




```python
grader.check("ex6")
```




<p><strong><pre style='display: inline;'>ex6</pre></strong> passed! 🚀</p>



### Exercise 7  (Additional practice question with **NO** points)

rubric={manual:0} (See the end of the worksheet for a possible solution.)

Given the variable `language` which contains a string, use `if/elif/else` to write code that will:
- print "I love snakes!" if `language` is `"python"` or `"Python"` or `"PYTHON"` (or any variation of capitalization of this word).
- print "Are you a pirate?" if `language` is `"R"` or `"r"`
- else print "What is `language`?" if `language` is anything else.

<!-- BEGIN QUESTION -->




```python
language = 'Python'
...
```

<!-- END QUESTION -->

### Exercise 8  (Additional practice question with **NO** points)

rubric={manual:0} (See the end of the worksheet for a possible solution.)

Write code that counts down from any positive interger `x` to 1 and print each number, and then finally prints "Happy New Year!" 

For example, if x = 5, it should print 
```
5
4
3
2
1
Happy New year!
```

<!-- BEGIN QUESTION -->




```python
x = 10
```


```python
...
```

<!-- END QUESTION -->

### Exercise 9 (Additional practice question with **NO** points)

rubric={manual:0} (See the end of the worksheet for a possible solution.)

If you used a `while` loop to answer Exercise 7, try answering it with a `for` loop. Conversely, if you used a `for` loop to answer Exercise 7, try answering it with a `while` loop.

<!-- BEGIN QUESTION -->




```python
x = 10
```


```python
...
```

<!-- END QUESTION -->

Congratulations! You are done the worksheet!!! Pat yourself on the back and submit your worksheet to Gradescope!

> ### Possible solutions
> 
> #### Exercise 7
> 
> ```python
> language = 'Python'
> if language.lower() == "python":
>     # language.lower() converts the string to lowercase so capitalization doesn’t matter
>     print("I love snakes!")
> elif language.lower() == "r":
>     print("Are you a pirate?")
> else:
>     print(f"What is {language}?")
> ```
> 
> #### Exercises 8 & 9
> 
> **`while` loop**
> ```python
> while x > 0:
>     print(x)
>     x -= 1
> print("Happy New Year!")
> ```
>
>**`for` loop**
>
> ```python
> for i in range(x):
>     print(x - i)
> print("Happy New Year!")
> ```


