
# Lecture 7 Dynamic Programming I
<br>

## Pre-Reading:
- [Dynamic Programing Intro video](https://www.youtube.com/watch?v=vYquumk4nWw) (14.5m)


## Outline:
- Introduction to Dynamic Programming (5 mins)
- Climbing Stairs Problem (30 mins)
- Connection to Fibonacci sequence (5 mins)
- Memoization and Caching (10 mins)
- Bottom-up approach (15 mins)
- Problem solving template (10 mins)

## Learning objectives

- Identify cases of repeated or redundant computation and address the issues with caching or memoization.
- Describe the basic idea of dynamic programming.
- Identify problems when dynamic programming is applicable.
- Implement simple dynamic programming algorithms.

## Introduction to Dynamic Programming

Dynamic programming (DP) is a powerful technique for solving problems that can be broken down into **simpler subproblems** whose solutions can be **reused**.  If you notice that your recursive solution is doing the same work over and over again, DP provides a way to cache intermediate results so that each subproblem is solved only once.  The two essential properties that make a problem amenable to dynamic programming are:

- **Optimal substructure** — the optimal solution to the problem can be composed of optimal solutions to its subproblems.
- **Overlapping subproblems** — the space of subproblems is small and the same subproblems are solved repeatedly.

Dynamic programming appears in a wide variety of applications, from **global sequence alignment** in genetics (aligning DNA or protein sequences) and **edit distance** computations in text processing, to optimisation problems like the **knapsack problem** in resource allocation, **matrix chain multiplication** in compiler optimisation, and shortest-path algorithms such as **Floyd–Warshall**.

In this lecture we will explore dynamic programming through two classical problems:

1. **Climbing Stairs** — counting the number of ways to climb a staircase when you can take 1 or 2 steps at a time.  This example highlights DP's ability to count possibilities and reuse previously computed counts.
   
2. **Fibonacci Sequence** — computing the nth Fibonacci number efficiently by reusing solutions to smaller subproblems.  This example introduces the basic DP recurrence and shows how memoization and tabulation speed up naive recursion.

We will compare **top‑down** (recursive with memoization) and **bottom‑up** (tabulation) strategies, discuss caching in Python, and learn to recognise when DP applies and implement efficient solutions.

### Visualizing Dynamic Programming

The diagram below provides an abstract view of how dynamic programming works.
Overlapping subproblems are represented as interconnected nodes. The arrows show how each subproblem depends on smaller ones, 
and the highlighted cache node illustrates how memoization stores results so they can be reused instead of recomputed.

<img src="static/dp_concept.png" width="500">


```python
# If you need the below:
#%pip install -U joblib scikit-learn 
```


```python
# Some imports
import functools
import numpy as np
import joblib
import sklearn.metrics
from IPython.display import HTML, display
```

---
# Climbing Stairs (Counting Paths)

**Problem.** There are _n_ stairs. You may climb **1** or **2** steps at a time.  
**Question:** How many **distinct** ways can you reach the top? (Distinct means order matters: `[1,2]` differs from `[2,1]`.)

<img src="static/climbing_stairs-main.jpg" width="400">

### Examples

**n = 1 → 1 way**
- `[1]`

**n = 2 → 2 ways**
- `[1,1]`
- `[2]`

**n = 3 → 3 ways**
- `[1,1,1]`
- `[1,2]`
- `[2,1]`

**n = 4 → 5 ways**
- `[1,1,1,1]`
- `[1,1,2]`
- `[1,2,1]`
- `[2,1,1]`
- `[2,2]`

**Edge case.** Define `ways(0) = 1` (the empty climb).


### Key idea

To reach stair _n_, the last move is either a 1-step from _n−1_ or a 2-step from _n−2_. So

$$
\mathrm{ways}(n)=\mathrm{ways}(n-1)+\mathrm{ways}(n-2),\qquad
\mathrm{ways}(0)=1,\ \mathrm{ways}(1)=1.
$$

These counts are Fibonacci numbers (shifted by one).

### Quick self-check

Without enumerating, how many ways for _n_ = 5?

**Answer:** `ways(5) = ways(4) + ways(3) = 5 + 3 = 8`.

<img src="https://favtutor.com/resources/images/uploads/mceu_48293760511659177124940.png" width="700">

*Source:* [favtutor.com](https://favtutor.com/blogs/climbing-stairs-problem)

---

Before diving into algorithms, one might try to enumerate all ways of combining 1- and 2-step moves to reach the top.

This *brute-force* approach offers intuition about the problem but quickly becomes inefficient.

### Brute Force Solution (using recursion)

- We can solve the problem using a brute force approach.
- We iterate through *all combinations* of both one and two steps until the overall amount of steps equals `n`.
- The number of options that provide a total of `n` steps is the solution.
- When every possible combination of steps is considered, the time complexity will be  $O$(2^n). You will see below that it is $O$(2^n) since *we are making two recursive calls for each stair*.
- This is very slow as `n` becomes larger. Let's introduce recusion to this problem:

#### Recursion

- This problem has a **recursive nature**, because a person can reach nth stair either from **(n-1)** th stair or **(n-2)** th stair.
- For this reason, for each stair `n`, we will need to calculate the number of ways to reach the `(n-1)`th and `(n-2)`th stairs, and add them to get the *total number of ways* to reach the nth stair.
- Let's think about the case of `n=4`. Picture it as a tree:

```
climb(4)
├─ climb(3)
│  ├─ climb(2)
│  │  ├─ climb(1) = 1
│  │  └─ climb(0) = 1
│  └─ climb(1) = 1
└─ climb(2)
   ├─ climb(1) = 1
   └─ climb(0) = 1
```

- Notice how `climb(2)` gets recomputed—this duplication is why the brute-force approach is slow.

- Here's the tree for `n=5`. Here also notice how both `climb(2)` and `climb(3)` get recomputed. Again, this duplication is why the brute-force approach is slow.
  
```
climb(5)
├─ climb(4)
│  ├─ climb(3)
│  │  ├─ climb(2)
│  │  │  ├─ climb(1) = 1
│  │  │  └─ climb(0) = 1
│  │  └─ climb(1) = 1
│  └─ climb(2)
│     ├─ climb(1) = 1
│     └─ climb(0) = 1
└─ climb(3)
   ├─ climb(2)
   │  ├─ climb(1) = 1
   │  └─ climb(0) = 1
   └─ climb(1) = 1
```


- Now let's write the code for the problem, using **recursion**:


```python
def climb_stairs_recursive(n):
    if n == 0 or n == 1:
        return 1
    
    return climb_stairs_recursive(n-1) + climb_stairs_recursive(n-2)
```

- This is nice, but can really get very slow quickly. 
- Let's look at some run times:


```python
%timeit -n1 -r1 climb_stairs_recursive(3)
```

    3.25 μs ± 0 ns per loop (mean ± std. dev. of 1 run, 1 loop each)



```python
%timeit -n1 -r1 climb_stairs_recursive(5)
```

    8.17 μs ± 0 ns per loop (mean ± std. dev. of 1 run, 1 loop each)



```python
%timeit -n1 -r1 climb_stairs_recursive(20)
```

    2.24 ms ± 0 ns per loop (mean ± std. dev. of 1 run, 1 loop each)



```python
%timeit -n1 -r1 climb_stairs_recursive(25)
```

    30.5 ms ± 0 ns per loop (mean ± std. dev. of 1 run, 1 loop each)



```python
# Nicer version:

def climb_stairs_recursive(n: int) -> int:
    """
    Count distinct ways to climb n stairs using steps of size 1 or 2.

    Recurrence:
        ways(n) = ways(n-1) + ways(n-2)
    Base cases:
        ways(0) = 1  (empty climb)
        ways(1) = 1
    Raises:
        ValueError if n < 0
    """
    if n < 0:
        raise ValueError("n must be >= 0")
    if n <= 1:
        return 1
    return climb_stairs_recursive(n - 1) + climb_stairs_recursive(n - 2)

```

### Cost of naive recursion

**Big picture.**  
Each time you call `climb(n)`—unless you hit a base case—you immediately branch into **two** subcalls: `climb(n−1)` and `climb(n−2)`. That means the work spreads out like a **binary tree** as you go down the recursion. The tree’s height is about `n`, and the number of nodes in a binary tree can **double at each level**, so the total number of calls grows exponentially. That is why the naive recursion takes $O(2^n)$ time.

---

### Intuition you can picture

- Level `0`: start with `climb(n)` → at most **1** call.  
- Level `1`: its children → at most **2** calls.  
- Level `2`: each (non-base) call branches again → at most **4** calls.  
- …  
- Level `k`: at most **$2^k$** calls.

Adding up all levels (a geometric series) up to depth $n$ gives:

$
1 + 2 + 4 + \dots + 2^n \;=\; 2^{\,n+1} - 1,
$

So the total number of calls—and thus the running time—is bounded by a constant times $2^n$. Hence $O(2^n)$).

---

### A small numeric feel

Counting **all** calls (including base cases) for the naive version:
- \(n=2:\) 3 calls  
- \(n=3:\) 5 calls  
- \(n=4:\) 9 calls  
- \(n=5:\) 15 calls  
- \(n=6:\) 25 calls  
- \(n=7:\) 41 calls

Notice the totals aren’t adding a fixed amount; they **balloon** because the same subproblems (like `climb(2)`) are recomputed in many branches.

---

### Takeaway

The naive recursion does the **same work repeatedly** (e.g., `climb(2)` appears in many subtrees). This duplication makes the call tree exponential, giving $O(2^n)$ time. Adding **memoization** (cache results) or using a **bottom-up DP** removes the duplication and reduces the time to $O(n)$ (with $O(n)$) or even $O(1)$ extra space).

Because the naive recursion repeatedly solves the same subproblems, its time complexity explodes.

A natural remedy is to **remember** (memoize) subproblem solutions or to build them bottom-up. This leads us to the dynamic programming techniques presented next.

### Using Dynamic Programming 
- We can solve recursive problems with dynamic programming.
- Dynamic programming will help us reduce the time complexity from exponential to linear.
- #### Remember: Why DP?
- - **Optimal substructure**: Ways to reach stair `n` = ways to reach `n-1` **plus** ways to reach `n-2`.
- - **Overlapping subproblems**: Computing `ways(4)` calls `ways(3)` and `ways(2)`, and `ways(3)` also calls `ways(2)` again.
 
Let's now come up with a plan.

### Our Plan:
- Let `n` be the number of stairs.
- If `n <=0`, the number of ways to climb the staircase should be `zero`.
- If `n == 1`, then there is only *one* way to climb the stair.
- For the numbers greater than 1, we can simply find the solution by:
- - adding previous steps i.e., `dp[n-1]` + `dp[n-2]`.
- - create a new array and store the output at each step.
- - then finally return the last value in the array as the output.

### Memo

- Let's create the array to store the output at each step.
- This will be our memo.
- The memo will be a list with slots for all subproblems from 0 to n.
- It is safest to put "-1" values in each slt to know we have not computed that slot yet. Let's start and we will explain the details again below:


```python
memo = [-1] * (n + 1)
memo
```




    [-1, -1, -1, -1, -1, -1]



#### Explanation of `memo = [-1] * (n + 1)`

- **What it creates:** a list of length **`n + 1`** (valid indices `0…n`).  
  Example for `n = 4` → `[-1, -1, -1, -1, -1]`.

- **Purpose:** this list is the **memo table** for dynamic programming.  
  Index `k` will store the computed number of ways `W(k)` so we don’t recompute it.

- **Why `-1`:** a **sentinel** meaning “not computed yet.”  
  Counts are non-negative, so `-1` safely marks “empty.” (`None` would also work.)

- **Why `n + 1`:** we need slots for **all subproblems** `0, 1, …, n` (inclusive).


```python
def climb_stairs_memoized(n: int) -> int:
    """
    Count ways to climb n stairs taking 1 or 2 steps at a time,
    using top-down recursion with memoization.
    """
    if n < 0:
        return 0 # or we can: raise ValueError("n must be >= 0")

    memo = [-1] * (n + 1)  # -1 marks "not computed yet"

    def _rec(k: int) -> int:
        if k <= 1:                # W(0)=1, W(1)=1
            return 1
        if memo[k] != -1:
            return memo[k]
        memo[k] = _rec(k - 1) + _rec(k - 2)
        return memo[k]

    return _rec(n)
```


```python
climb_stairs_memoized(5)
```




    8



You can now compare to the brute force version:


```python
%timeit -n1 -r1 climb_stairs_memoized(50)
```

    19.3 μs ± 0 ns per loop (mean ± std. dev. of 1 run, 1 loop each)



```python
%timeit -n1 -r1 climb_stairs_recursive(50)
```

    35min 30s ± 0 ns per loop (mean ± std. dev. of 1 run, 1 loop each)


#### Time and Space Complexity
- The *time complexity* of this solution is $O$(n), where `n` is the number of stairs.
- This is because we are iterating through a single loop to find a number of ways to climb the stairs.

- The *space complexity* is also $O$(n).
- We are storing the values using the array.


You can also rewrite the code above with two functions if you like:


```python
# Two functions:
def climb_stairs_recursive(k, memo):
    if k <= 1:
        return 1
    if memo[k] != -1:
        return memo[k]
    memo[k] = climb_stairs_recursive(k - 1, memo) + climb_stairs_recursive(k - 2, memo)
    return memo[k]

def climb_stairs_recursive_memoized(n):
    if n < 0:
        return 0
    memo = [-1] * (n + 1)
    return climb_stairs_recursive(n, memo)
```


```python
climb_stairs_recursive_memoized(5)
```




    8



---

## Fibonacci Sequence 
- But wait! The same problem of climbing stairs is equivalent to that of a Fibonacci sequence.
- A Fibonacci sequence is defined as follows:
- - $F_0=0$
- - $F_1=1$.
- For $n\ge2$, the sequence is generated by the relation:
 $F_{n}=F_{n-1}+F_{n-2}$

This means each number in the sequence is the sum of the two preceding numbers.

We can compute the Fibonacci sequence iteratively and recursively:


```python
def fib(n):
    f = np.zeros(n+1, dtype=int)
    f[1] = 1
    
    for i in range(2,n+1):
        f[i] = f[i-1] + f[i-2]
    
    return f[-1]

fib(7)
```


```python
def fib_recursive(n):
    if n == 0 or n == 1:
        return n
    
    return fib_recursive(n-1) + fib_recursive(n-2)

fib_recursive(7)
```

- Similar to climbing stairs, the recursive function looks much cleaner. However, computation using the recurrence relation can lead to exponential time complexity due to repeated calculations.


```python
%timeit -n1 -r1 fib(35)
```


```python
%timeit -n1 -r1 fib_recursive(35)
```

The problem is that there is a significant amount of repeated computation in our calculations.
For example, `fib(33)` is called by `fib(34)` and `fib(35)`, and it gets much worse deeper down the tree. In fact, `fib(1)` may be called millions of times. This problem is not unique to Fibonacci numbers and you will meet many similar problems. 

Let's now introduce **memoization** a little more formally:

--- 

### Caching & Memoization

To solve the problem of **redundant computation**, we have already used a **memo** with the climbing stairs problem. Let's now introduce the two key concepts **caching** and **memoization** a little more formally. 

- **Caching** is a general term that refers to the practice of storing data for later use.
  
- **Memoization** is a specific form of caching that involves storing the results of function calls so that when the same inputs are encountered again, the previously computed result can be reused. This avoids the need to recompute values that have already been calculated.

- - **Note:** In Python's context, the term "caching" is frequently used interchangeably with "memoization", which can be confusing. Python has built-in decorators like `functools.lru_cache`, which implement memoization but use the term "cache." 


#### Using Python decorators to implement caching

- In the code below, we use the `@functools.lru_cache(maxsize=None)` decorator to implement caching in the Fibonacci function. 


```python
@functools.lru_cache(maxsize=None)
def fib_recursive_cache(n):
    if n == 0 or n == 1:
        return n
    
    return fib_recursive_cache(n-1) + fib_recursive_cache(n-2)
```


```python
%timeit -n1 -r1 fib_recursive_cache(35)
```

This decorator allows the function to "remember" the results of previous function calls. If `fib_recursive_cache(30)` has already been computed, the result ($832040$) is saved and used if `fib_recursive_cache(30`) is ever called again. This means that each Fibonacci number is computed only once, significantly reducing the number of recursive calls and speeding up the process. 

While this approach speeds up computation by avoiding redundant calculations, it comes at the cost of increased memory usage, since we need to store results for all function calls. This trade-off—using more memory to achieve faster running time—is a common and useful concept in optimization.

Another approach to caching involves storing the cache outside of memory. 

`functools.lru_cache` stores the cached results in memory. We can use the `joblib` library to store the cache in a file. This file-backed caching mechanism is referred to as `Memory` in `joblib`, which can be a bit confusing, as the name might suggest it is still stored in memory. Despite the terminology, this method is useful when you want to cache results but avoid using too much RAM, especially for larger datasets or longer-running computations.


```python
memory = joblib.Memory("/tmp", verbose=0)
```


```python
@memory.cache
def fib_recursive_cache2(n):
    if n == 0 or n == 1:
        return n
    
    return fib_recursive_cache2(n-1) + fib_recursive_cache2(n-2)
```


```python
%timeit -n1 -r1 fib_recursive_cache2(35)
```

---
### Bottom-Up/Tabulation

Up to this point we've used memoization in a **top-down** manner: starting from \(n\) and recursively computing smaller subproblems while caching results.

There is another flavour of dynamic programming called **bottom-up** or **tabulation**. Instead of recursion, we start from the simplest subproblems, such as $ways(0)$) and $ways(1)$, and iteratively build up to $ways(n)$.

This approach eliminates recursion overhead and can sometimes reduce memory usage. In the next section we'll implement a bottom-up DP for the climbing stairs problem.


#### Climbing Stairs — Bottom-Up DP (Tabulation)

- We can solve the same problem bottom-up (tabulation).
- This approach is similar to the top-down (memoization) approach except that instead of breaking down the problem recursively, we iteratively build up the solution by calculating in a bottom-up manner.
- We will maintain a dp[] table such that dp[i] stores the number of ways to reach ith stair.
- This runs in **O(n)** time.
- The classic tabulation uses **O(n)** space for the table; we also show a space-optimized variant that keeps only the last two values (**O(1)** space).


```python
from typing import List

def climb_stairs_tab(n: int) -> int:
    """
    Bottom-up DP (tabulation) for the 'climb stairs' problem.

    dp[i] = number of ways to reach step i using 1 or 2 steps.
    Recurrence: dp[i] = dp[i-1] + dp[i-2]
    Base cases: dp[0] = 1 (empty climb), dp[1] = 1

    Time:  O(n)
    Space: O(n)
    """
    if n < 0:
        raise ValueError("n must be >= 0")
    if n <= 1:
        return 1

    dp: List[int] = [0] * (n + 1)
    dp[0] = 1
    dp[1] = 1
    for i in range(2, n + 1):
        dp[i] = dp[i - 1] + dp[i - 2]
    return dp[n]


def climb_stairs_tab_optimized(n: int) -> int:
    """
    Space-optimized bottom-up DP.
    Keeps only the last two dp values (Fibonacci-like update).

    Time:  O(n)
    Space: O(1)
    """
    if n < 0:
        raise ValueError("n must be >= 0")
    if n <= 1:
        return 1

    a, b = 1, 1   # dp[0], dp[1]
    for _ in range(2, n + 1):
        a, b = b, a + b
    return b


# --- quick sanity checks ---
_expected = [1, 1, 2, 3, 5, 8, 13, 21]
for i, want in enumerate(_expected):
    assert climb_stairs_tab(i) == want
    assert climb_stairs_tab_optimized(i) == want
print("Passed basic tests for n = 0..7")
```

    Passed basic tests for n = 0..7


---

## Comparing Approaches

| Problem | Approach | Time Complexity | Space Complexity | Comments |
|-------|---------|---------------|-----------------|---------|
| **Fibonacci** | Naive recursion | $O(2^n)$ | $O(n)$* | Recomputes subproblems exponentially; simple but inefficient. |
|  | Memoization (top‑down) | $O(n)$ | $O(n)$ | Caches results of each $F_i$; avoids redundant work. |
|  | Tabulation (bottom‑up) | $O(n)$ | $O(n)$ | Builds the sequence iteratively; no recursion overhead. |
|  | Space‑optimised tabulation | $O(n)$ | $O(1)$ | Keeps only last two values since each $F_n$ depends on $F_{n-1}$ and $F_{n-2}$. |
| **Climbing Stairs** | Naive recursion | $O(2^n)$ | $O(n)$* | Similar to Fibonacci; exponential growth of call tree. |
|  | Memoization (top‑down) | $O(n)$ | $O(n)$ | Caches the number of ways to reach each step. |
|  | Tabulation (bottom‑up) | $O(n)$ | $O(n)$ | Fills a DP array from small steps up to $n$. |
|  | Space‑optimised tabulation | $O(n)$ | $O(1)$ | Maintains only two variables representing the last two DP values. |

\*Time complexity counts the number of calls, but the recursion stack depth is $O(n)$ for naive recursion. Memoised and tabulated solutions reduce the number of calls to linear.

---

## DP Problem‑Solving Template

When approaching a new problem with dynamic programming, follow these steps:

1. **Identify the subproblem.** Determine how you can break the original problem into smaller instances. For sequences, this often involves prefixes (e.g., the first \(i\) elements) or suffixes.  For counting paths, consider the last move or state.
2. **Establish base cases.** Specify the simplest inputs and their outputs.  These anchor the recursion and initialise the DP table.
3. **Derive the recurrence relation.** Express the solution to a subproblem in terms of solutions to smaller subproblems.  Ensure that the relation only refers to earlier (smaller) states to avoid cycles.
4. **Choose a DP strategy.** Decide between a **top‑down** approach (recursion with memoization) and a **bottom‑up** approach (iterative tabulation).  Top‑down mirrors the mathematical recurrence but may incur recursion overhead; bottom‑up can be more memory‑efficient and easier to test for iterative order.
5. **Implement and test.** Translate your recurrence into code, initialise your base cases, and verify correctness with small examples.  Measure performance to ensure the expected complexity is achieved.
6. **Optimise if necessary.** For some problems, you can reduce space by recognising that only a fixed number of previous states are needed.  Always consider whether the full DP table must be stored.

---

## Conclusion

Dynamic programming turns seemingly exponential problems into polynomial‑time solutions by identifying and reusing overlapping subproblems.


- In **Climbing Stairs**, we counted the number of ways to reach the top using 1‑ or 2‑step moves.  We compared the brute‑force approach to memoized and tabulated solutions and observed the dramatic improvement in performance.

- In **Fibonacci**, we defined a simple recurrence $F_n = F_{n-1} + F_{n-2}$ and saw how naive recursion leads to repeated work.  Memoization and bottom‑up tabulation allow us to compute $F_n$ efficiently in $O(n)$ time.

These examples illustrate two common uses of dynamic programming: **computing sequence values** and **counting possibilities**.  The same principles extend to many other problems across computer science and engineering.  By defining clear subproblems, establishing recurrences, and choosing an appropriate memoization or tabulation strategy, you can transform complex recursive definitions into efficient algorithms.
