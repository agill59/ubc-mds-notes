```python
# Initialize Otter
import otter
grader = otter.Notebook("worksheet8.ipynb")
```

# Worksheet 8 - Classes and Style

These exercises complement DSCI 511 lecture 8.

## Exercises


```python
import pandas as pd
import numpy as np
import math
```

### Q1: Creating a Class

rubric={autograde:1}

Create a class called `Circle`. It should have the following characteristics:
1. It should be initiated with the argument `radius` and store this as an instance attribute.
2. Have a method `area()` which calculates the area of the circle. $A = \pi * r^2$
3. Have a method `circumference()` which calculates the circumference of the circle. $C = 2*\pi*r$
4. Have the method `__str__()` which is a magic method that controls what is output to the screen when you `print()` an instance of your class (learn more [here](https://realpython.com/lessons/how-and-when-use-__str__/)). This method should return the string `"A circle with radius r"` where `r` is the radius value. For example:
```python
c1 = Circle(1)
print(c1) # EXPECTED OUTPUT: A circle with radius 1
```


```python
class Circle:
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return math.pi * self.radius**2

    def circumference(self):
        return math.pi * 2 * self.radius

    def __str__(self):
        return f"A circle with radius {self.radius}"
```


```python
grader.check("q1")
```




<p><strong><pre style='display: inline;'>q1</pre></strong> passed! ✨</p>



### Q2: Class Inheritance

rubric={autograde:1}

**Instructions**

1. Define a class called `Animal` with an `__init__()` method that takes a `name` parameter and sets it as an instance attribute.
2. Define a class called `Dog` that inherits from the `Animal` class.
3. In the `Dog` class, define an `__init__()` method that takes a `name` and `breed` parameter, and calls the `__init__()` method of the `Animal` class using `super()`.
4. In the `Dog` class, define a `bark()` method that returns the string "Woof!" (Don't print "Woof!")


```python
class Animal:
    def __init__(self, name):
        self.name = name


class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__(name)
        self.breed = breed

    def bark(self):
        return "Woof!"
```


```python
grader.check("q2")
```




<p><strong><pre style='display: inline;'>q2</pre></strong> passed! 🍀</p>



### Q 3.

rubric = {autograde:1}

This exercise is built upon your answer in Q2.

- Add a `count` **class variable** to the `Dog` class. The `__init__()` method increments the `count` variable each time a new instance of the class is created.

- Add a `get_dog_count()` **class method** to the `Dog` class that returns the number of `Dog` instances that have been created.

- Add a `reset_dog_count()` **class method** to the `Dog` class that sets the count to zero


```python
class Dog(Animal):
    count = 0

    def __init__(self, name, breed):
        super().__init__(name)
        self.breed = breed
        Dog.count = Dog.count + 1

    def bark(self):
        return "Woof!"

    @classmethod
    def get_dog_count(cls):
        return cls.count

    @classmethod
    def reset_dog_count(cls):
        cls.count = 0
```


```python
# Some tests to check your code
Dog.reset_dog_count()
print(f"The number of dogs after resetting is {Dog.get_dog_count()}")

dog1 = Dog("Fido", "Golden Retriever")
print(
    f"After create a dog1 instance, the number of dogs is {Dog.get_dog_count()}")

dog2 = Dog("Hunky", "German Shepherd")
print(
    f"After create a dog2 instance, the number of dogs is {Dog.get_dog_count()}")

# Should return 2
print(f"Get the count of dogs: There are {Dog.get_dog_count()} dogs")
```

    The number of dogs after resetting is 0
    After create a dog1 instance, the number of dogs is 1
    After create a dog2 instance, the number of dogs is 2
    Get the count of dogs: There are 2 dogs



```python
grader.check("q3")
```




<p><strong><pre style='display: inline;'>q3</pre></strong> passed! 🍀</p>



### Q 4. (not for points)

rubric = {manual:0}

Fix the following code to make sure it aligns with PEP8 style guide.


```python
def q1_incorrect():
    CarModel = 'Toyota'
    Make = "RAV4"
    return CarModel + Make
```

<!-- BEGIN QUESTION -->




```python
def q1():
    CarModel = 'Toyota'
    Make = "RAV4"
    return CarModel + Make
```

<!-- END QUESTION -->

### Q5 (not for points)

rubric = {manual:0}

Fix the following code to make sure it aligns with PEP8 style guide

```
def q2_incorrect():
    gpa = (course1 + course2 + course3 + course4 + course5 + course6 + course7 + course8 + course9 + course10) / 10
```

<!-- BEGIN QUESTION -->




```python
def q2():
    gpa = (course1 + course2 + course3 + course4 + course5 +
           course6 + course7 + course8 + course9 + course10) / 10
```

<!-- END QUESTION -->

### Q 6 (not for points)

rubric = {manual:0}

Fix the following code to make sure it aligns with PEP8 style guide.

```
class car_model:
    def __init__(self, car_model, make):
        self.car_model = car_model
        self.make = make   
    def __str__(self):
        return f"{self.car_model} {self.make}"
class car_sale(car_model):
    def __init__(self, car_model, make, price):
        super().__init__(car_model, make)
        self.price = price   
    def sell(self):
        return f"{self.car_model} {self.make} sold for {self.price}"
```

<!-- BEGIN QUESTION -->




```python
class car_model:
    def __init__(self, car_model, make):
        self.car_model = car_model
        self.make = make

    def __str__(self):
        return f"{self.car_model} {self.make}"


class car_sale(car_model):
    def __init__(self, car_model, make, price):
        super().__init__(car_model, make)
        self.price = price

    def sell(self):
        return f"{self.car_model} {self.make} sold for {self.price}"
```

<!-- END QUESTION -->

Congratulations! You are done the **LAST** worksheet!!! Pat yourself on the back, and submit your worksheet to Gradescope!

>### Q4 Solutions
>
>```
>def q1():
>    car_model = "Toyota"
>    make = "RAV4"
>    return car_model + make
>```
>
>
>### Q5 Solutions
>
>```
>def q2():
>    # BEGIN SOLUTION
>    gpa = (
>        course1
>        + course2
>        + course3
>        + course4
>        + course5
>        + course6 
>        + course7
>        + course8
>        + course9
>        + course10
>    ) / 10
>```
>
>
>### Q6 Solutions
>
>```
>class CarModel:
>    def __init__(self, car_model, make):
>        self.car_model = car_model
>        self.make = make
>
>    def __str__(self):
>        return f"{self.car_model} {self.make}"
>
>
>class CarSale(CarModel):
>    def __init__(self, car_model, make, price):
>        super().__init__(car_model, make)
>        self.price = price
>
>    def sell(self):
>        return f"{self.car_model} {self.make} sold for {self.price}"
>```
