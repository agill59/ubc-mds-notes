

# Lecture 3: Graphs and Sparse Matrices


```python
# import packages needed in this lecture
import numpy as np
import networkx as nx
from collections import Counter
import matplotlib.pyplot as plt
import queue
import scipy.sparse

# The defaults are hard to see on a projector in class
plt.rcParams["figure.figsize"] = (4, 3)
%config InlineBackend.figure_formats = ['svg']

draw_params = {
    "node_color": "lightblue",
    "font_size": 14,
    "with_labels": True,
    "arrowsize": 20,
    "width": 1,
    "node_size": 1200
}

# Ignore deprecation warnings, in particular for getnnz.
# It is deprecated but the replacement doesn't seem to work
# for the examples we want to show here...
import warnings
warnings.filterwarnings("ignore", category=DeprecationWarning)
warnings.filterwarnings("ignore", category=UserWarning)
```

## Pre-Reading:
- [Graphs Introduction](https://runestone.academy/ns/books/published/pythonds/Graphs/toctree.html) - read sections [8.1](https://runestone.academy/ns/books/published/pythonds/Graphs/Objectives.html) and [8.2](https://runestone.academy/ns/books/published/pythonds/Graphs/VocabularyandDefinitions.html)

## Outline:

- Recap: `dict` and `defaultdict` (5 min)
- Graphs intro (5 min)
- Graph terminology (5 min)
- Graph types (10 min)
- Graph representations (15 min)
- Break (5 min)
- Sparse Matrices (30 min)

## Learning Objectives
By the end of this lecture, you will be able to:

- Map real-world problems to various kinds of graphs (directed/undirected, weighted/unweighted).
- Convert between adjacency matrices and their corresponding graphs.
- Manipulate scipy sparse matrix objects in Python.
- Analyze the space complexity of sparse and dense matrix representations.
- Appropriately use row-wise vs. column-wise sparse matrices.

## Recap: `dict` and `defaultdict`

- `dict` is a built-in Python data structure that implements a hash table.
- It maps keys to values.
- Python dictionary (`dict` library) maps key to values.
- $O(1)$ average time complexity for search, insert, and delete operations.

Example:



```python
grades = {'Alice': 85, 'Bob': 90}
print(grades['Alice'])   # 85

```

    85
    

But access to a non-existent key raises a `KeyError`:


```python
print(grades['Charlie']) # KeyError

```


    ---------------------------------------------------------------------------

    KeyError                                  Traceback (most recent call last)

    Cell In[6], line 1
    ----> 1 print(grades['Charlie']) # KeyError
    

    KeyError: 'Charlie'


To handle this safely, we often use the `get` method or check for key existence. `dict` requires you to check or handle missing keys manually, as shown below.


```python
grades.get('Charlie', 0)     # returns 0 if 'Charlie' not in grades
if 'Charlie' not in grades:
    grades['Charlie'] = 0
```

### Introducing `defaultdict`

[`defaultdict`](https://docs.python.org/3/library/collections.html#defaultdict-objects) is a subclass of `dict` from the `collections` module that automatically handles missing keys.

- In DSCI 511 you saw classes and inheritance.
- Just for fun, we can look at the implementation of `defaultdict` in PyPy, which is an implementation of Python in Python (the OG Python is written in C): [https://github.com/reingart/pypy/blob/master/lib_pypy/_collections.py#L387](https://github.com/reingart/pypy/blob/master/lib_pypy/_collections.py#L387)

```python
from collections import defaultdict
dd = defaultdict(default_factory)
```
- `default_factory` is a function that provides the default value for a missing key. Usually, it's a type like `list`, `int`, `set`, etc.
- `default_factory = None` by default, check [documentation](https://docs.python.org/3/library/collections.html#defaultdict-objects) for more details
- When a key is missing, `defaultdict` calls `default_factory` to create a default value and inserts it into the dictionary

#### Counting example with `defaultdict`:




```python
from collections import defaultdict

counts = defaultdict(int)
for fruit in ['apple', 'banana', 'apple']:
    counts[fruit] += 1

print(counts)
```

    defaultdict(<class 'int'>, {'apple': 2, 'banana': 1})
    

`defaultdict(list)` is perfect when you want to collect multiple values under the same key. In this case, `list()` creates `[]` automatically, so we can append safely.


```python
groups = defaultdict(list)

groups['fruits'].append('apple')
groups['fruits'].append('banana')
groups['colors'].append('red')

print(groups)
```

    defaultdict(<class 'list'>, {'fruits': ['apple', 'banana'], 'colors': ['red']})
    

Here is what happened in the above example:
- `'fruits'` didn’t exist $\rightarrow$ `defaultdict` created `groups['fruits'] = []`.

- Then `'apple'` and `'banana'` were appended to that list.

- Same for `'colors'`.

We can check the types:



```python
print(type(grades))
print(type(counts))
```

    <class 'dict'>
    <class 'collections.defaultdict'>
    

```{admonition} Key take-aways
:class: tip
- `dict` is simple and you control all key creation.
- `defaultdict` automatically initializes missing keys.
- Use `defaultdict(int)` for counting, `defaultdict(list)` for grouping, and `defaultdict(set)` for collecting unique items.

```

## 1. Why should we study Graphs?

A **graph** is a versatile and powerful data structure used to model relationships and interactions between entities. Graphs can represent any set of relationships, including cyclic and non-hierarchical ones.

Graphs are powerful tools to represent and analyze relationships between data points. Consider the following examples:
- Social networks: Graphs are widely used to model relationships on platforms like Facebook or Twitter.
- Internet: machines connected across the Internet can be modeled as a graph.
- Transportation systems: airlines use graphs to map flight routes between cities.
- Course prerequisites:  graphs help to visualize the sequence of courses required for a degree in data science. 

These examples illustrate how graphs provide a flexible framework for modeling diverse systems. Throughout this lecture, we'll study different types of graphs and algorithms, allowing us to solve a wide range of problems in data science.

### 1.1. What are Graphs?

A graph is a data structure consisting of vertices (also known as nodes) connected by edges. A vertex represents an entity, and an edge shows a relationship between two entities. 

#### Formal Definition

A graph $G=(V,E)$ is a set of vertices $V$ and edges $E$ where each edge $(u, v)$ connects a pair of vertices. $u, v \in V$.
- $V$ is a set of vertices or nodes.
- $E$ is a set of edges.

For example, let's consider a graph where:
- $V$ represents a set of airports, and
$E$ represents direct flight connections between two airports.  


```python
# Create a graph
G = nx.Graph()

# Add nodes
G.add_node("VAN") # Vancouver
G.add_node("TOR") # Toronto
G.add_node("VIC") # Victoria
G.add_node("EDM") # Edmonton
G.add_node("SFO") # San Francisco  

# Add edges 
G.add_edge("VAN", "TOR")
G.add_edge("VAN", "VIC")
G.add_edge("VAN", "EDM")
G.add_edge("VAN", "SFO")
G.add_edge("TOR", "EDM")
G.add_edge("TOR", "VIC")
```

- We are using `networkx` graph package. It randomly visualizes the graph each time.
    - By re-running the next cell, we can see a few equivalent representations.
    - They are all the same graph!
    - See [`networkx.draw` documentation](https://networkx.github.io/documentation/stable/reference/generated/networkx.drawing.nx_pylab.draw_networkx.html).
    - Note: if you are interested, there's an Altair interface to `networkx`: [nx_altair](https://github.com/Zsailer/nx_altair).


```python
nx.draw(G, **draw_params)
```


    
![svg](03_graphs-sparse-matrices_23_0.svg)
    


## 2. Graph terminologies:
- **Neighbors:** vertices $u$ and $v$ are neighbors if there is an edge $(u, v)$ between them.
  - In the above example, Vancouver (YVR) and Calgary (YYC) are neighbors because there is a direct flight between them.
- The **degree** of a vertex is the number of edges connected to it.
  -  The number of its neighbors is the degree of vertex
  - In the above example, the Vancouver airport (VAN) has a degree of 4, meaning it is directly connected to four other airports. 
  - In contrast, San Francisco (SFO) has a degree of 1, as it is connected to only one other airport.
- A **path** is defined as a sequence of vertices that are connected by edges.
  - Let's consider a path from Edmonton (EDM) to San Francisco (SFO) in the graph below.

The path is shown in red, and it goes through the following sequence of airports: Edmonton $\rightarrow$ Vancouver $\rightarrow$ San Francisco.
- **Path length** is the number of edges in the path.
  - In this case, the path length is 2, as there are two edges connecting the three airports in the sequence.

- A **cycle** is a closed path that starts and ends at the same vertex, with all other vertices visited only once. In the example above, VAN $\rightarrow$ TOR $\rightarrow$ VIC $\rightarrow$ VAN forms a cycle.



```python
# Create graph
G = nx.Graph()
G.add_nodes_from(["VAN", "TOR", "VIC", "EDM", "SFO"])
G.add_edges_from([
    ("VAN", "TOR"),
    ("VAN", "VIC"),
    ("VAN", "EDM"),
    ("VAN", "SFO"),
    ("TOR", "EDM"),
    ("TOR", "VIC")
])

# Define a path from EDM to SFO
path = ["EDM", "VAN", "SFO"]

# Get edges along this path
path_edges = list(zip(path, path[1:]))

# Draw base graph
pos = nx.spring_layout(G, seed=42)
nx.draw(G, pos, with_labels=True, node_color="lightblue", node_size=1000, font_size=14)

# Highlight path
nx.draw_networkx_nodes(G, pos, nodelist=path, node_color="orange")
nx.draw_networkx_edges(G, pos, edgelist=path_edges, edge_color="red", width=3)

plt.show()
```


    
![svg](03_graphs-sparse-matrices_25_0.svg)
    


- A graph is **connected** if there exists a path between every pair of nodes. In other words, in a connected graph, you can start at any vertex and reach any other vertex through a sequence of edges.

- If there is at least one pair of nodes between which no path exists, the graph is considered **disconnected**. 

Below is an example of a disconnected graph, where some nodes are isolated from others, meaning no path connects them.


```python
disconnected = nx.Graph()

disconnected.add_node("A")
disconnected.add_node("B")
disconnected.add_node("C")
disconnected.add_node("D")
disconnected.add_node("E")
disconnected.add_node("F")

disconnected.add_edge("A", "B")
disconnected.add_edge("A", "C")
disconnected.add_edge("D", "C")
disconnected.add_edge("B", "C")

disconnected.add_edge("E", "F")

pos = nx.spring_layout(disconnected, k=10)
nx.draw(disconnected, pos, **draw_params)
```


    
![svg](03_graphs-sparse-matrices_27_0.svg)
    


- A graph is **complete** if every pair of vertices is joined by an edge.
  - The graph above is not complete because not every airport has a direct flight to every other airport. 
  - For instance, there is no direct flight between Edmonton and SFO, so the graph is incomplete.

Below is an example of a complete graph.


```python
complete_graph = nx.complete_graph(G.nodes)
nx.draw(complete_graph, **draw_params)
```


    
![svg](03_graphs-sparse-matrices_29_0.svg)
    



What is the degree of vertex "VAN" above?

**A.** 0 

**B.** 1 

**C.** 4 

**A.** 5 


## 3. Graph types: Directed vs Undirected

#### Undirected graphs

- This is what we have been looking at so far.
- Saying "_there is an edge from A to B_" is the same as saying "_there is an edge from B to A_"
- **Example:** Think of Facebook friendships or a two-way road system. If Alice is friends with Bob, Bob is also friends with Alice. Similarly, if there is a road between two cities, traffic can flow in both directions.


#### Directed graphs

- Sometimes the relationship between entities is **directional** or **asymmetric**.
    - For example, on Twitter, you may follow someone without them following you back. This creates an asymmetric relationship that we can model using a directed graph, where each edge has a specific direction.
- Such relations are represented as a directed graph. We typically draw the direction with an arrow.
- In `networkx`, directed graphs are created using the `DiGraph()` function. Here's an example of adding a directed edge:



```python
G = nx.DiGraph()

G.add_node("Alice")
G.add_node("Bob")
G.add_node("Cal")

G.add_edge("Alice", "Bob")
G.add_edge("Cal", "Alice")
G.add_edge("Cal", "Bob")

nx.draw(G, **draw_params)
```


    
![svg](03_graphs-sparse-matrices_34_0.svg)
    


Now let's reverse the direction of the edge between Cal and Bob:


```python
G = nx.DiGraph()

G.add_node("Alice")
G.add_node("Bob")
G.add_node("Cal")

G.add_edge("Alice", "Bob")
G.add_edge("Cal", "Alice")
G.add_edge("Bob", "Cal")

nx.draw(G, **draw_params)
```


    
![svg](03_graphs-sparse-matrices_36_0.svg)
    


Directed graphs can also have bidirectional connections:


```python
G = nx.DiGraph()

G.add_node("Alice")
G.add_node("Bob")
G.add_node("Cal")

G.add_edge("Bob", "Cal")

# add a 2-way edge
G.add_edge("Cal", "Alice")
G.add_edge("Alice", "Cal")

nx.draw(G, **draw_params)
```


    
![svg](03_graphs-sparse-matrices_38_0.svg)
    


Here, we have a double-headed arrow between Alice and Cal. 

Note that in an undirected graph, it wouldn't have made sense to have both 

```python
G.add_edge("Cal", "Alice")
G.add_edge("Alice", "Cal")
```
because one would be sufficient.

Directed graphs also can have **self-connections**. However, `networkx` supports self-connections but it doesn't draw them properly. You will see these when getting to Markov chains in DSCI 575.


```{admonition} Exercise
:class: tip
Suppose you have a list of film actors, and you want to keep track of which actors have been in a film together. Would you model that with a directed or undirected graph?

**A.** Directed 

**B.** Undirected 
```

## 4. Graph types: Weighted vs Unweighted

In addition to directionality, graphs can also differ based on whether or not the edges have weights. The weight of an edge can represent various quantities such as distance, cost, or capacity. Depending on the problem you're trying to solve, it may be essential to take these weights into account when analyzing the graph. This leads us to the distinction between unweighted and weighted graphs, both of which can be either directed or undirected.

#### Unweighted graphs

All the graphs above are "unweighted" because all edges are treated equally. There is either **a relationship** or there is **no relationship** between two nodes.

- In an unweighted graph, an edge simply indicates whether or not a relationship exists between two nodes—there is no additional information associated with that edge. 
- For example, in an unweighted graph representing an airline network, the presence of an edge between two airports indicates that there is a direct flight between them, but no further details (such as the distance or cost) are considered.

#### Weighted graphs

In contrast, both undirected and directed graphs can have weighted edges, where each edge is assigned a numerical value or weight. This weight conveys extra information about the relationship between two nodes. 

- The weight tells us some **numerical information about the relationship**:
    - In an airline network, the weight might represent the flight time between two airports or the cost of a flight.
    - In a social network, the weight could reflect how long two people have been friends or how frequently they interact.

- Weighted graphs allow for more detailed modeling of relationships, making them particularly useful for problems involving optimization, such as [finding the shortest path](https://en.wikipedia.org/wiki/Shortest_path_problem).

Here's an example of an undirected and weighted graph representing the cost to fly between two airports:



```python
G = nx.Graph()

G.add_node("VAN")
G.add_node("EDM")
G.add_node("TOR")

G.add_edge("VAN", "EDM", weight=110)
G.add_edge("VAN", "TOR", weight=250)
G.add_edge("TOR", "EDM", weight=150)

pos = nx.spring_layout(G)
nx.draw(G, pos, **draw_params)
labels = nx.get_edge_attributes(G, 'weight')
nx.draw_networkx_edge_labels(G, pos, edge_labels=labels, font_size=20)
```




    {('VAN', 'EDM'): Text(0.4290472774934839, -0.09150716657311464, '110'),
     ('VAN', 'TOR'): Text(-0.4083622469103184, -0.4084933155264585, '250'),
     ('EDM', 'TOR'): Text(-0.020680438890595454, 0.500003931057659, '150')}




    
![svg](03_graphs-sparse-matrices_43_1.svg)
    


Here's an example of a directed and weighted graph representing the time to fly a particular direction between two airports:


```python
G = nx.DiGraph()

G.add_node("VAN")
G.add_node("EDM")
G.add_node("TOR")

G.add_edge("VAN", "EDM", weight=1.2)
G.add_edge("VAN", "TOR", weight=5)
G.add_edge("TOR", "EDM", weight=3.3)

pos = nx.spring_layout(G)
nx.draw(G, pos, **draw_params)
labels = nx.get_edge_attributes(G, 'weight')
nx.draw_networkx_edge_labels(G, pos, edge_labels=labels, font_size=20)
```




    {('VAN', 'EDM'): Text(-0.3549461609988697, 0.2677719894357439, '1.2'),
     ('VAN', 'TOR'): Text(0.32817809094463, -0.5000002991243959, '5'),
     ('TOR', 'EDM'): Text(0.02676835120314658, 0.23222977556378577, '3.3')}




    
![svg](03_graphs-sparse-matrices_45_1.svg)
    


## 5. Graph representations

There are two standard ways to represent a graph $G=(V,E)$ as a collection of adjacency lists or as an adjacency matrix. These representations can be applied to both directed and undirected graphs.

#### Adjacency List Representation

We can represent the graph as an array of **adjacency list**: for each vertex $u$, we have an adjacency list containing all the vertices $v$ such that there is an edge $(u,v)\in E$. That is, it lists all pairs of vertices that are connected to $u$ by an edge. 

The graph below can be represented as the following adjecency list.


```python
G4 = nx.bull_graph()
nx.draw(G4, **draw_params)
```


    
![svg](03_graphs-sparse-matrices_47_0.svg)
    



```python
# Get the adjacency list as a dictionary
adj_list_dict = {node: list(neighbors) for node, neighbors in G4.adjacency()}
print(adj_list_dict)
```

    {0: [1, 2], 1: [0, 2, 3], 2: [0, 1, 4], 3: [1], 4: [2]}
    

```{admonition} Exercise
:class: tip
What is the space complexity of an adjacency list, in terms of $V$, the number of vertices, and $E$, the number of edges?

**A.** $O(V)$

**B.** $O(E)$

**C.** $O(VE)$

**D.** $O(V+E)$ 
```

#### Adjacency Matrix Representation

The adjacency-matrix representation of a graph $G=(V,E)$ assumes that the vertices are numbered $1,2,\dots,|V|$ in some arbitrary manner. Then the adjacency matrix representation of a graph $G$ consists of a $|V| \times |V|$ matrix $A$ such that

$$a_{ij} = 
\begin{cases}
1,\ \text{if } (i,j) \in E \\
0,\ \text{otherwise}.
\end{cases}$$

In this matrix, a value of $1$ indicates the presence of an edge between vertex $i$ and vertex $j$, while a value of $0$ indicates that no edge exists between these vertices.



#### Matrix Notation

An $m \times n$ matrix $A$ is a 2D rectangular array of numbers with m rows and n columns
and written as
$$
A = 
\begin{bmatrix}
a_{11} & a_{12} & \dots & a_{1n} \\
a_{21} & a_{22} & \dots & a_{2n} \\
\vdots & \vdots & \ddots & \vdots \\
a_{m1} & a_{m2} & \dots & a_{mn}
\end{bmatrix}_{m \times n}
= [a_{ij}], (1 \leq i \leq m, 1 \leq j \leq n)
$$

$a_{ij}$ is the element in the $i^{th}$ row and $j^{th}$ column of matrix $A$.

- A matrix is usually represented by a capital letter in printed text and handwriting, such as $A$.
- The element in the $i^{th}$ row and $j^{th}$ column of matrix $A$ is denoted by a lowercase letter with two subscripts, such as $a_{ij}$.


Using the same graph G4, the adjacency matrix representation of G4 is:


```python
G4 = nx.bull_graph()
adj = nx.adjacency_matrix(G4).toarray()
adj
```




    array([[0, 1, 1, 0, 0],
           [1, 0, 1, 1, 0],
           [1, 1, 0, 0, 1],
           [0, 1, 0, 0, 0],
           [0, 0, 1, 0, 0]])



Note that this is a **symmetric matrix**.


```python
# adjacent matrix for node 0 
adj[0, :]
```




    array([0, 1, 1, 0, 0])




```python
# adjacent matrix between node 1 and 3
adj[1, 3]
```




    np.int64(1)



```{admonition} Exercise
:class: tip
What is the space complexity of an adjacency matrix, in terms of $V$, the number of vertices, and $E$, the number of edges?

**A** $O(VE)$

**B** $O(E^2)$

**C** $O(V^2)$

**D** $O(V+E)$ 
```

#### Comparing two representations:

When choosing between the adjacency matrix and the adjacency list, it's important to consider their respective advantages and disadvantages.

The adjacency matrix is straightforward and intuitive, making it easy to visualize the connections between nodes, especially in small graphs. However, a significant drawback is that most cells in the adjacency matrix will contain zeros, indicating no connections. This leads to several implications:

- Space complexity: $O(V^2)$. 
- Lookup Time: Checking if any two nodes $i$ and $j$ are connected can be done in $O(1)$ time.

On the other hand, the adjacency list implementation provides a more compact representation, particularly suited for sparse graphs. The advantage of the adjacency list implementation is that it allows us to compactly represent a sparse graph. The adjacency list also allows us to easily find all the edges that are directly connected to a particular vertex.

- Space complexity: $O(V+E)$. In the worst case of a complete graph, where every vertex is connected to every other vertex, $E=O(V^2)$, making the space requirements similar to that of an adjacency matrix.
- Lookup Time: However, checking if any two nodes are connected takes $O(V)$ time in the worst case.

Wouldn't it be great if we could combine the lookup speed of the adjacency matrix with the storage efficiency of the adjacency list? While we can't achieve both perfectly, we can come close by using structured adjacency lists, which lead us to the sparse matrices we talked about last week. Sparse matrices allow us to efficiently store and manage connections in a way that balances lookup speed and memory usage.

In fact, `nx` uses sparse matrix to store the adjacency matrix:


```python
am_matrix = nx.adjacency_matrix(G4)
type(am_matrix)
```




    scipy.sparse._csr.csr_array



```{admonition} Exercise
:class: tip
If $|E|$ is much less than $|V^2|$, which representation is better?

**A.** Adjacency list 

**B.** Adjacency matrix 
```

## 6. Sparse Matrices

**🚨 Terminology Alert**: 

- In math, a sparse matrix is **a matrix with a lot of zeros in it**.
- In coding, a sparse matrix is **a data structure** to efficiently store and manipulate large datasets with mostly zero values.
- _This is extremely confusing!_
- It is a good idea to use a sparse matrix data structure when you have a sparse matrix.
  - But it is _possible_ to use a sparse matrix data structure when your matrix is not sparse.
  - And it is _possible_ to use a dense matrix data structure (e.g. numpy array) when your matrix is sparse.


```python
G = nx.ladder_graph(5)
nx.draw(G, **draw_params)
```


    
![svg](03_graphs-sparse-matrices_62_0.svg)
    



```python
am_ladder = nx.adjacency_matrix(G)
am_ladder
```




    <Compressed Sparse Row sparse array of dtype 'int64'
    	with 26 stored elements and shape (10, 10)>




```python
type(am_ladder)
```




    scipy.sparse._csr.csr_array



- Sparse matrices are conceptual data structure like a list, dictionary, set, etc.
- [`scipy.sparse`](https://docs.scipy.org/doc/scipy/reference/sparse.html) matrices are the Python implementation of this conceptual data structure, like `list`, `dict`, `set`, etc.
- Going to that link, we can see there are many types of scipy sparse matrix.
  - This one is a [`csr_matrix`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.csr_matrix.html#scipy.sparse.csr_matrix)
  - More later on these types.
- You can convert them to numpy arrays with `toarray()`, but this is usually a bad idea.


```python
print(am_ladder.toarray())
```

    [[0 1 0 0 0 1 0 0 0 0]
     [1 0 1 0 0 0 1 0 0 0]
     [0 1 0 1 0 0 0 1 0 0]
     [0 0 1 0 1 0 0 0 1 0]
     [0 0 0 1 0 0 0 0 0 1]
     [1 0 0 0 0 0 1 0 0 0]
     [0 1 0 0 0 1 0 1 0 0]
     [0 0 1 0 0 0 1 0 1 0]
     [0 0 0 1 0 0 0 1 0 1]
     [0 0 0 0 1 0 0 0 1 0]]
    


```python
type(am_ladder.toarray())
```




    numpy.ndarray




```python
type(am_ladder)
```




    scipy.sparse._csr.csr_array




```python
A = am_ladder.toarray() # undirect graph = symmetric matrix;

print(A)
print("---")
print(A.T)
```

    [[0 1 0 0 0 1 0 0 0 0]
     [1 0 1 0 0 0 1 0 0 0]
     [0 1 0 1 0 0 0 1 0 0]
     [0 0 1 0 1 0 0 0 1 0]
     [0 0 0 1 0 0 0 0 0 1]
     [1 0 0 0 0 0 1 0 0 0]
     [0 1 0 0 0 1 0 1 0 0]
     [0 0 1 0 0 0 1 0 1 0]
     [0 0 0 1 0 0 0 1 0 1]
     [0 0 0 0 1 0 0 0 1 0]]
    ---
    [[0 1 0 0 0 1 0 0 0 0]
     [1 0 1 0 0 0 1 0 0 0]
     [0 1 0 1 0 0 0 1 0 0]
     [0 0 1 0 1 0 0 0 1 0]
     [0 0 0 1 0 0 0 0 0 1]
     [1 0 0 0 0 0 1 0 0 0]
     [0 1 0 0 0 1 0 1 0 0]
     [0 0 1 0 0 0 1 0 1 0]
     [0 0 0 1 0 0 0 1 0 1]
     [0 0 0 0 1 0 0 0 1 0]]
    


```python
# make a bigger sparse matrix
G = nx.fast_gnp_random_graph(100_000,1e-4) 
am = nx.adjacency_matrix(G)
type(am)
```




    scipy.sparse._csr.csr_array




```python
print("Total number of entries: ", am.shape) # 10 billion
```

    Total number of entries:  (100000, 100000)
    


```python
# non-zero elements of the matrix 
print("Total number of Non-zero entries: ", am.nnz)
```

    Total number of Non-zero entries:  1001220
    

- Okay, so we have a $100,000 \times 100,000$ matrix, which would have $10^{10}$ elements.
- But only around $10^6$ of them are non-zero. Or 0.01% nonzero.
  - We say "this matrix is very sparse".
  
**How bad would it be to store this as a numpy array?**

Stored in full form, the matrix would take up:


```python
full_size = np.prod(am.shape)*8/1e9
print("The full matrix would take up %d GB" % full_size)
```

    The full matrix would take up 80 GB
    

That's a lot! How big is the sparse matrix?


```python
sparse_size = (am.data.nbytes + am.indptr.nbytes + am.indices.nbytes)/1e6
print("The sparse matrix takes up %d MB" % sparse_size)
```

    The sparse matrix takes up 16 MB
    

So, the fraction of space saved is:


```python
frac_nz = am.nnz / np.prod(am.shape)
print("The sparse matrix is %dx smaller" % (1/frac_nz))
```

    The sparse matrix is 10008x smaller
    

- Right, so we definitely don't want to store the full matrix.
- Let's do some operations on the sparse matrix.

#### Let's see how to scipy stores sparse matrices


```python
G = nx.fast_gnp_random_graph(100_000,1e-4) 
am_g = nx.adjacency_matrix(G)
i = 0
print(am_g[i])
print(am_g.shape)
```

    <COOrdinate sparse array of dtype 'int64'
    	with 12 stored elements and shape (100000,)>
      Coords	Values
      (7826,)	1
      (32274,)	1
      (34458,)	1
      (34855,)	1
      (43366,)	1
      (44054,)	1
      (51881,)	1
      (56288,)	1
      (65661,)	1
      (68651,)	1
      (73190,)	1
      (78493,)	1
    (100000, 100000)
    

#### Notational nuisances: Indexing

- There are some weird things to know about scipy sparse matrices.
- For example, with a regular numpy array, `x[i,j]` and `x[i][j]` are equivalent:


```python
x = np.random.rand(10, 10)
print("x[1, 2] = ", x[1, 2])
print("x[1][2] = ", x[1][2])
print("(x[1])[2] = ", (x[1])[2])
```

    x[1, 2] =  0.223830727703069
    x[1][2] =  0.223830727703069
    (x[1])[2] =  0.223830727703069
    

This is because `x[1]` returns the first row, and then the `[2]` indexes into that row.

However, with `scipy.sparse` matrices, things are a bit different:


```python
x_sparse = scipy.sparse.csr_matrix(x)
print("x_sparse[1, 2] = ", x_sparse[1, 2])
print("x_sparse[1][2] = ", x_sparse[1][2])
```

    x_sparse[1, 2] =  0.223830727703069
    


    ---------------------------------------------------------------------------

    IndexError                                Traceback (most recent call last)

    Cell In[11], line 3
          1 x_sparse = scipy.sparse.csr_matrix(x)
          2 print("x_sparse[1, 2] = ", x_sparse[1, 2])
    ----> 3 print("x_sparse[1][2] = ", x_sparse[1][2])
    

    File /opt/anaconda3/envs/ubc-mds-2025/lib/python3.12/site-packages/scipy/sparse/_index.py:30, in IndexMixin.__getitem__(self, key)
         29 def __getitem__(self, key):
    ---> 30     index, new_shape = self._validate_indices(key)
         32     # 1D array
         33     if len(index) == 1:
    

    File /opt/anaconda3/envs/ubc-mds-2025/lib/python3.12/site-packages/scipy/sparse/_index.py:270, in IndexMixin._validate_indices(self, key)
        268 N = self._shape[index_ndim]
        269 if not (-N <= idx < N):
    --> 270     raise IndexError(f'index ({idx}) out of range')
        271 idx = int(idx + N if idx < 0 else idx)
        272 index.append(idx)
    

    IndexError: index (2) out of range


Let's see what does `x_sparse[1]` return.


```python
x_sparse[1]
```




    <Compressed Sparse Row sparse matrix of dtype 'float64'
    	with 10 stored elements and shape (1, 10)>




```python
row_1_sparse = x_sparse[1]
print("row_1_sparse.shape = ", row_1_sparse.shape)
```

    row_1_sparse.shape =  (1, 10)
    


```python
print(row_1_sparse)
```

    <Compressed Sparse Row sparse matrix of dtype 'float64'
    	with 10 stored elements and shape (1, 10)>
      Coords	Values
      (0, 0)	0.7140087489015666
      (0, 1)	0.7041973516681698
      (0, 2)	0.223830727703069
      (0, 3)	0.5271019262120182
      (0, 4)	0.29273703273031837
      (0, 5)	0.08492690122377611
      (0, 6)	0.013281993797003011
      (0, 7)	0.9267671450490138
      (0, 8)	0.8368976159797188
      (0, 9)	0.37075663905827505
    

- The sparse matrix returns a different shape, leaving in the first dimension.
- This can be annoying and is something to watch out for.
- In general, I suggest using the `x[1,2]` notation when possible because chaining the `[]` can be problematic in several places (e.g., also pandas).
- However, **this is only for numpy**, not, say, a list of lists:


```python
lst = [[1, 2, 3], [4, 5, 6], [7, 9]]

print(lst[0][1])
```

    2
    


```python
print(lst[0, 1])
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    Cell In[17], line 1
    ----> 1 print(lst[0, 1])
    

    TypeError: list indices must be integers or slices, not tuple


#### Sparse Matrix format (Scipy)

- This is "Compressed Sparse Row" format.
- It means we store an adjacency list _per row_. 
- So if we want Row 4, we just grab it right away.
- But if we want column 4, we have to iterate through every row and check if there's a 4!


```python
x = scipy.sparse.random(5, 5, density=0.2, format="csr", random_state=321)

print(x.toarray())
```

    [[0.86182738 0.         0.         0.         0.        ]
     [0.         0.         0.         0.36493942 0.        ]
     [0.         0.         0.52952801 0.         0.        ]
     [0.         0.         0.         0.         0.        ]
     [0.84758728 0.         0.98840654 0.         0.        ]]
    


```python
print(x)
```

    <Compressed Sparse Row sparse matrix of dtype 'float64'
    	with 5 stored elements and shape (5, 5)>
      Coords	Values
      (0, 0)	0.8618273768819481
      (1, 3)	0.3649394158657733
      (2, 2)	0.529528005602308
      (4, 0)	0.8475872823428623
      (4, 2)	0.9884065398125097
    

Here's how it is actually stored:


```python
x.data # the non-zero entries in the matrix
```




    array([0.86182738, 0.36493942, 0.52952801, 0.84758728, 0.98840654])




```python
x.indices # column indices - "which column is the corresponding entry in"
```




    array([0, 3, 2, 0, 2], dtype=int32)




```python
x.indptr # where does each row start?
```




    array([0, 1, 2, 3, 3, 5], dtype=int32)



Above: 

- Row 0 starts at `x.indices[0]`
- Row 1 starts at `x.indices[1]`
- Row 2 starts at `x.indices[2]`
- Row 3 starts at `x.indices[3]`
- Row 4 also starts at `x.indices[3]`, because row 3 is empty
- Row 5 starts at `x.indices[5]`


Here is a diagram that makes these numbers much more understandable:

<img src="https://i.stack.imgur.com/12bPL.png" width="800">

([Image source](https://stackoverflow.com/a/52299730/8823277))

#### Sparse matrix operations: `sum` example
- First, some numpy syntax:


```python
x = np.random.randint(10, size=(4,5))
x
```




    array([[1, 7, 7, 2, 5],
           [1, 8, 5, 0, 4],
           [6, 6, 4, 7, 2],
           [8, 7, 1, 6, 2]])




```python
np.sum(x)
```




    np.int64(89)




```python
x.sum() # <- do it this way for sparse matrices (for numpy either is fine)
```




    np.int64(89)



#### Sparse Data Sets

Sparse matrices come up _a lot_ in practice, beyond just adjacency matrices in graphs. For example:

- **Word counts:** We might represent a document by the words in it, but only a small fraction of all words would appear in a given document.
- **Ratings:** We might represent an Amazon item by the user ratings, but only a small fraction of all users have rated a given item.
- **Physical processes:** Many experimental datasets could be sparsed. During the Capstone project in 2019, some students examined images from a particle physics dataset, in which most of the sensors got zero signal.



I am storing a dataset on the annual trade between  countries in Europe. The data is modelled as a graph, where countries are vertices and the total value of goods traded between two countries are edges. What representation should I use?

A: Adjacency Matrix

B: Adjacency List

C: Sparce Matrix


#### Sparse Matrix timing

- The amount of time things take with sparse matrices can be quite surprising. 
- First off, let's look at regular numpy arrays.


```python
numpy_array = np.random.rand(10000, 10000)
```

Numpy arrays are actually stored row-by-row, so summing a row is faster that summing a column:


```python
%timeit numpy_array[0,:].sum() # sum the first row
```

    3.37 μs ± 11.4 ns per loop (mean ± std. dev. of 7 runs, 100,000 loops each)
    


```python
%timeit numpy_array[:,0].sum() # sum the first column
```

    31 μs ± 192 ns per loop (mean ± std. dev. of 7 runs, 10,000 loops each)
    

Overall, both of these are _much_ faster than summing the whole thing (which makes sense!):


```python
%timeit numpy_array.sum()
```

    24.8 ms ± 206 μs per loop (mean ± std. dev. of 7 runs, 10 loops each)
    


```python
G = nx.fast_gnp_random_graph(10_000, 0.0025)
sparse_matrix = nx.adjacency_matrix(G)
num_rows, num_cols = sparse_matrix.shape
```


```python
%timeit sparse_matrix[0,:].sum()
```

    91.9 μs ± 851 ns per loop (mean ± std. dev. of 7 runs, 10,000 loops each)
    


```python
%timeit sparse_matrix[:,0].sum()
```

    517 μs ± 22.7 μs per loop (mean ± std. dev. of 7 runs, 1,000 loops each)
    


```python
%%timeit -r1 -n1 

# iterate through the rows - this is much slower than numpy functions
for i in range(num_rows):
    sparse_matrix[i,:]
```

    765 ms ± 0 ns per loop (mean ± std. dev. of 1 run, 1 loop each)
    


```python
# %%timeit -r1 -n1

# # iterate through the columns - this is much MUCH slower
# for i in range(num_cols):
#     sparse_matrix[:,i]
```

#### Note- iterate through the rows - this is much slower than numpy functions

- Looping is slower, as per usual.
- But at least looping through the rows of a `csr_matrix` isn't that bad.
- However, looping through the columns of a `csr_matrix` is a disaster - it took several min on my laptop!!
  - Because it is stored _row by row_. 
  - To grab a single column, it needs to loop through each row and look for items in that column.
- If you want to loop through columns you should first convert the matrix type.

## 7. Other graph properties (Optional)

There are a lot of standard problems in graph theory, not covered in this course, such as:

- Find the largest [clique](https://en.wikipedia.org/wiki/Clique_(graph_theory)) in a graph.
- Find the largest [independent set](https://en.wikipedia.org/wiki/Independent_set_(graph_theory)) in a graph.
- Find the [vertex cover](https://en.wikipedia.org/wiki/Vertex_cover) of a graph with the smallest number of vertices.
- Find the smallest [dominating set](https://en.wikipedia.org/wiki/Dominating_set) in a graph.
- [Colour](https://en.wikipedia.org/wiki/Graph_coloring) a graph with the smallest number of colours.

It turns out a lot of real-world problems can be mapped to these problems, and thus solving these problems is important. **However**, some of the problems are [NP-Complete](https://www.geeksforgeeks.org/np-completeness-set-1/). That is, we don't have fast algorithms to solve these problems.

## 8. Additional reading on `defaultdict`, `list` and `list()` (Optional) 

- Question: why do we use `defaultdict(list)` instead of `defaultdict(list())`?
- What happens when I run this code:


```python
my_list = list()
my_list
```




    []




```python
bad = defaultdict([])
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    Cell In[19], line 1
    ----> 1 bad = defaultdict([])
    

    TypeError: first argument must be callable or None



```python
bad = defaultdict(list())
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    Cell In[20], line 1
    ----> 1 bad = defaultdict(list())
    

    TypeError: first argument must be callable or None


- But why?
- The code that executes first is `list()`.
- So we pass in **a particular list**.
- But we don't want one particular list, we want to create new lists all the time.
- So we need to pass in **the ability to create lists**
  - in other words, a **function that creates lists**
- That function is `list`


```python
list # This is a function
```




    list




```python
list() # This is one list
```




    []




```python
x = defaultdict(lambda : "hello I am your friendly neighbourhood default value")
```


```python
x[5]
```




    'hello I am your friendly neighbourhood default value'



In a future lab you might need to count occurrences.
So, in that case, what is my default value?


```python
text = "Blah blah is he still talking about dictionaries??"
```

This can be done with a Python function:


```python
text.count('a')
```




    5



If you are doing something more sophisticated, you might want to ignore this for now, and implement the counting ourselves:


```python
number_of_a = 0
for t in text:
    if t == "a":
        number_of_a += 1
        
number_of_a
```




    5



- Ok, but now we want to count "a" and "b".
- Same as before, let's use a dict.
- We already know this won't work - it's the same problem as before:


```python
number_of_times = dict()

for char in ('a', 'b'):
    for t in text:
        if t == char:
            number_of_times[char] = number_of_times[char] + 1
        
number_of_times
```


    ---------------------------------------------------------------------------

    KeyError                                  Traceback (most recent call last)

    Cell In[32], line 6
          4     for t in text:
          5         if t == char:
    ----> 6             number_of_times[char] = number_of_times[char] + 1
          8 number_of_times
    

    KeyError: 'a'


So, we use a `defaultdict`.

```{admonition} Exercise
:class: tip
What do we want the default value to be?

```{dropdown} Answer
0
```



```python
defaultdict(0)
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    Cell In[33], line 1
    ----> 1 defaultdict(0)
    

    TypeError: first argument must be callable or None


Oops! Right, it needs to be a **function**, not a specific value.


```python
defaultdict(lambda: 0)
```




    defaultdict(<function __main__.<lambda>()>, {})




```python
# The `int` function will return zero, so we can use that too
defaultdict(int)
```




    defaultdict(int, {})




```python
# int() function creates the an integer with 0 value
int()
```




    0




```python
number_of_times = defaultdict(int)

for char in ('a', 'b', 'c'):
    for t in text:
        if t == char:
            number_of_times[char] += 1
        
number_of_times
```




    defaultdict(int, {'a': 5, 'b': 2, 'c': 1})



#### Counter
And finally, for the supremely ~~lazy~~ awesome, we can use `Counter`. We need to import it from `collections`:
```python
from collections import Counter
```

 This is basically a `defaultdict(int)` but with some fancy methods added: 


```python
from collections import Counter

number_of_times = Counter()

for char in ('a', 'b'):
    for t in text:
        if t == char:
            number_of_times[char] += 1
        
number_of_times
```




    Counter({'a': 5, 'b': 2})




```python
number_of_times.most_common(1)
```




    [('a', 5)]


