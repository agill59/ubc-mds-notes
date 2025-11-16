

# Lecture 3: Summarizing & joining tables

## Lecture outline

- Aggregations
- Grouping
- Joins

Well, things have finally gotten serious with SQL and relational databases! In this lecture, we will learn how to do more advanced operations to gain more insight into the data in the tables of a database. Furthermore, we will explore how to connect the related data in multiple tables together through SQL joins. **It is through using joins in relational databases that we can benefit from the true power of these databases**.

First things first, let's connect to our database:


```python
%load_ext sql
%config SqlMagic.displaylimit = 30
```


```python
import json
import urllib.parse

with open('data/credentials.json') as f:
    login = json.load(f)
    
username = login['user']
password = urllib.parse.quote(login['password'])
host = login['host']
port = login['port']
```


```python
%sql postgresql://{username}:{password}@{host}:{port}/world
```

## Aggregations

So far in the course, we have seen many functions for various purposes. For example, `ROUND()` and `SQRT()` for math operations, `CHAR_LENGTH` and `SUBSTR()` for manipulating strings, or `EXTRACT()` and `to_date()` for working with datetimes. As you may have noticed, these functions produce an individual output for each and every row of a column in an element-wise manner (remember vectorized operations in NumPy?).

There is also a small class of useful functions in SQL called **aggregation** functions, which operate on groups of rows and summarize the data stored in those rows in the form of a single value. Here is a list of standard aggregation functions in SQL:

| Function   | What it computes                    |
|------------|-------------------------------------|
| `COUNT(*)` | Count of all rows in a table        |
| `COUNT()`  | Count of non-null rows of a column  |
| `MIN()`    | Minimum value in a column           |
| `MAX()`    | Maximum value in a column           |
| `AVG()`    | Average of values in a column       |
| `SUM()`    | Total sum of values in a column     |

A couple of points to remember:

- Except for `COUNT(*)`, all aggregation functions ignore `NULL`s
- In addition to numbers, `MIN()` and `MAX()` also work with strings.

<br><br><br>

---

**Example:** Find the population of the world according to the `country` table in the `world` database.

---


```sql
%%sql

SELECT
    SUM(population)
FROM
    country
;
```

     * postgresql://postgres:***@localhost:5432/world
    1 rows affected.
    




<table>
    <tr>
        <th>sum</th>
    </tr>
    <tr>
        <td>6078749450</td>
    </tr>
</table>



<br><br><br>

There are a few things that we need to remember when using aggregation functions:

- It is valid to have multiple aggregations in a SQL query, but it is NOT possible have both aggregations and regular columns in a single query:

```sql
-- This is CORRECT:
SELECT
    AVG(lifeexpectancy), SUM(population)
FROM
    country
WHERE
    continent = 'North America'
;

-- This is WRONG:
SELECT
    AVG(lifeexpectancy), name
FROM
    country
WHERE
    continent = 'North America'
;
```

There is only one exception to the latter rule, and that's when we have a `GROUP BY` clause (we'll learn about that in a bit).

<br><br><br>

- An aggregation function CANNOT be used in the `WHERE` clause.

For example, we can't find the name of countries with above-average populations using the following query:

```sql
-- This is WRONG:
SELECT
    name
FROM
    country
WHERE
    population > AVG(population)
;
```

It is, of course, possible to write a query to answer the above question, but we have to wait until we learn about subqueries in a later lecture.

### Postgres-specific aggregations (OPTIONAL)

In addition to the standard SQL aggregation functions, Postgres also provides a number of functions of the same kind which can be useful for some statistical calculations (find a comprehensive list in the documentations [here](https://www.postgresql.org/docs/current/functions-aggregate.html#FUNCTIONS-ORDEREDSET-TABLE)). Here is a few examples of Postgres-specific aggregations:

| Function               | What it computes                                                       |
|------------------------|------------------------------------------------------------------------|
| `stddev_pop()`         | Population standard deviation                                          |
| `stddev_samp()`        | Sample standard deviation                                              |
| `regr_r2(X, Y)`        | Coefficient of determination for linear regression between `X` and `Y` |
| `regr_slope(X, Y)`     | Slope of the regression line of `X` and `Y`                            |
| `regr_intercept(X, Y)` | Intercept of the regression line of `X` and `Y`                        |

<br><br><br>

---

**Example:** Compute the average ± sample standard deviation of the population of cities located in the US using `city` table. Write your query such that its output looks like this: `Average ± STDEV population of US cities = <average_population> ± <stdev_population>`

---


```sql
%%sql

SELECT
    'Average ± STDEV population of cities in the US = ' ||
    AVG(population)::INT || ' ± ' || stddev_samp(population)::INT 
FROM
    city
WHERE
    countrycode = 'USA'
;
```

     * postgresql://postgres:***@localhost:5432/world
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>?column?</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Average ± STDEV population of cities in the US = 286955 ± 586583</td>
        </tr>
    </tbody>
</table>



<br><br><br>

## Grouping

If we divide a table into groups of rows based on values of one or more columns, that is called grouping.

For example, in the `country` table of the `world` database, we find several countries located in the same continent. In this situation, we can group the rows in our `country` table based on the values in the `continent` column. In this way, we would end up with bunch of "sub-tables": A sub-table for all rows where `continent = 'Asia'`, another sub-table for all rows where `continent = 'Europe`, and so on.

The formal syntax of the grouping operation in SQL looks like this:

```sql
SELECT
    grouping_columns, aggregated_columns
FROM
    table1
WHERE
    condition
GROUP BY
    grouping_columns
ORDER BY
    grouping_columns
```

Typically, it is not the sub-tables themselves that we're interested in, but some sort of summary statistics: For example, we might want to know the average population for each continent, i.e. for each sub-table or group. In order to do this, we can use aggregation functions that learned about them in the previous section. The question of **"what is the average population of countries in each continent"** can be asked in SQL terms as follows:


```sql
%%sql

SELECT
    continent, AVG(population)
FROM
    country
GROUP BY
    continent
;
```

     * postgresql://postgres:***@localhost:5432/world
    7 rows affected.
    




<table>
    <thead>
        <tr>
            <th>continent</th>
            <th>avg</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Asia</td>
            <td>72647562.745098039216</td>
        </tr>
        <tr>
            <td>South America</td>
            <td>24698571.428571428571</td>
        </tr>
        <tr>
            <td>North America</td>
            <td>13053864.864864864865</td>
        </tr>
        <tr>
            <td>Oceania</td>
            <td>1085755.357142857143</td>
        </tr>
        <tr>
            <td>Antarctica</td>
            <td>0E-20</td>
        </tr>
        <tr>
            <td>Africa</td>
            <td>13525431.034482758621</td>
        </tr>
        <tr>
            <td>Europe</td>
            <td>15871186.956521739130</td>
        </tr>
    </tbody>
</table>



Important points:

- If your SQL query involves grouping as well as filtering with `WHERE` and sorting with `ORDER BY`, the `GROUP BY` clause MUST appear between `WHERE` and `ORDER BY`.

- There can't be any non-aggregated column in a grouping query, except for the columns which are used for grouping (remember the exception I talked about with aggregation functions?). In other words, a non-aggregated column in the `SELECT` clause MUST appear in the `GROUP BY` clause as well.
- If there are null values in the grouping column, there will be a separate group for null values in the results.

<br><br><br>

---

**Example:** Write a query to return the average and maximum population of cities in the `city` table for China, India, Canada, US, Australia, and Russia.

Show the results for each country using the corresponding country code, and order groups alphabetically in ascending order.

---


```sql
%%sql

SELECT
    countrycode, AVG(population), MAX(population)
FROM
    city
WHERE
    countrycode IN ('CHN', 'IND', 'CAN', 'USA', 'AUS', 'RUS')
GROUP BY
    countrycode
ORDER BY
    countrycode
;
```

     * postgresql://postgres:***@localhost:5432/world
    6 rows affected.
    




<table>
    <thead>
        <tr>
            <th>countrycode</th>
            <th>avg</th>
            <th>max</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>AUS</td>
            <td>808119.000000000000</td>
            <td>3276207</td>
        </tr>
        <tr>
            <td>CAN</td>
            <td>258649.795918367347</td>
            <td>1016376</td>
        </tr>
        <tr>
            <td>CHN</td>
            <td>484720.699724517906</td>
            <td>9696300</td>
        </tr>
        <tr>
            <td>IND</td>
            <td>361579.255131964809</td>
            <td>10500000</td>
        </tr>
        <tr>
            <td>RUS</td>
            <td>365876.719576719577</td>
            <td>8389200</td>
        </tr>
        <tr>
            <td>USA</td>
            <td>286955.379562043796</td>
            <td>8008278</td>
        </tr>
    </tbody>
</table>



<br><br><br>

### Filtering revisited: the `HAVING` clause

So far, we have used the `WHERE` clause to filter rows. However, I mentioned before that aggregation functions cannot be used inside a `WHERE` clause to enforce a condition on the returned grouped rows.

There is another reserved keyword, `HAVING`, for when we need to do filtering using aggregated values **after grouping rows**. The syntax is as follows (order is important!):

```sql
SELECT
    grouping_columns, aggregated_columns
FROM
    table1
[WHERE
    condition]
GROUP BY
    grouping_columns
HAVING
    group_condition
[ORDER BY
    grouping_columns]
```

<br><br><br>


Important note:

- `WHERE` filters rows **before** grouping

- `HAVING` filters rows **after** (or resulting from) grouping

<br><br><br>

---

**Example:**

Write a query to return the average and maximum population of cities for countries that have at least 60 cities listed in the `city` table.

Show the results for each country using the corresponding country code, and order groups by the number of cities of each country in descending order. Also, convert the returned values to integer type.

---


```sql
%%sql

SELECT
    countrycode,
    AVG(population)::int,
    MAX(population)::int,
    COUNT(population) AS city_count
FROM
    city
GROUP BY
    countrycode
HAVING
    COUNT(*) > 60
ORDER BY
    city_count DESC
;
```

     * postgresql://postgres:***@localhost:5432/world
    15 rows affected.
    




<table>
    <thead>
        <tr>
            <th>countrycode</th>
            <th>avg</th>
            <th>max</th>
            <th>city_count</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>CHN</td>
            <td>484721</td>
            <td>9696300</td>
            <td>363</td>
        </tr>
        <tr>
            <td>IND</td>
            <td>361579</td>
            <td>10500000</td>
            <td>341</td>
        </tr>
        <tr>
            <td>USA</td>
            <td>286955</td>
            <td>8008278</td>
            <td>274</td>
        </tr>
        <tr>
            <td>BRA</td>
            <td>343507</td>
            <td>9968485</td>
            <td>250</td>
        </tr>
        <tr>
            <td>JPN</td>
            <td>314375</td>
            <td>7980230</td>
            <td>248</td>
        </tr>
        <tr>
            <td>RUS</td>
            <td>365877</td>
            <td>8389200</td>
            <td>189</td>
        </tr>
        <tr>
            <td>MEX</td>
            <td>345390</td>
            <td>8591309</td>
            <td>173</td>
        </tr>
        <tr>
            <td>PHL</td>
            <td>227462</td>
            <td>2173831</td>
            <td>136</td>
        </tr>
        <tr>
            <td>DEU</td>
            <td>282209</td>
            <td>3386667</td>
            <td>93</td>
        </tr>
        <tr>
            <td>IDN</td>
            <td>441008</td>
            <td>9604900</td>
            <td>85</td>
        </tr>
        <tr>
            <td>GBR</td>
            <td>276996</td>
            <td>7285000</td>
            <td>81</td>
        </tr>
        <tr>
            <td>KOR</td>
            <td>557141</td>
            <td>9981619</td>
            <td>70</td>
        </tr>
        <tr>
            <td>IRN</td>
            <td>388552</td>
            <td>6758845</td>
            <td>67</td>
        </tr>
        <tr>
            <td>NGA</td>
            <td>271358</td>
            <td>1518000</td>
            <td>64</td>
        </tr>
        <tr>
            <td>TUR</td>
            <td>456888</td>
            <td>8787958</td>
            <td>62</td>
        </tr>
    </tbody>
</table>



Note that just like with the `WHERE` clause, the expression used for filtering with `HAVING` does not necessarily need to appear in the `SELECT` clause (why?).

For instance, the `HAVING` clause will still do its job even if `COUNT(population)` is omitted from the `SELECT` clause:


```sql
%%sql

SELECT
    countrycode,
    AVG(population)::INT,
    MAX(population)::INT
FROM
    city
GROUP BY
    countrycode
HAVING
    COUNT(*) > 60
ORDER BY
    COUNT(*) DESC
;
```

     * postgresql://postgres:***@localhost:5432/world
    15 rows affected.
    




<table>
    <thead>
        <tr>
            <th>countrycode</th>
            <th>avg</th>
            <th>max</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>CHN</td>
            <td>484721</td>
            <td>9696300</td>
        </tr>
        <tr>
            <td>IND</td>
            <td>361579</td>
            <td>10500000</td>
        </tr>
        <tr>
            <td>USA</td>
            <td>286955</td>
            <td>8008278</td>
        </tr>
        <tr>
            <td>BRA</td>
            <td>343507</td>
            <td>9968485</td>
        </tr>
        <tr>
            <td>JPN</td>
            <td>314375</td>
            <td>7980230</td>
        </tr>
        <tr>
            <td>RUS</td>
            <td>365877</td>
            <td>8389200</td>
        </tr>
        <tr>
            <td>MEX</td>
            <td>345390</td>
            <td>8591309</td>
        </tr>
        <tr>
            <td>PHL</td>
            <td>227462</td>
            <td>2173831</td>
        </tr>
        <tr>
            <td>DEU</td>
            <td>282209</td>
            <td>3386667</td>
        </tr>
        <tr>
            <td>IDN</td>
            <td>441008</td>
            <td>9604900</td>
        </tr>
        <tr>
            <td>GBR</td>
            <td>276996</td>
            <td>7285000</td>
        </tr>
        <tr>
            <td>KOR</td>
            <td>557141</td>
            <td>9981619</td>
        </tr>
        <tr>
            <td>IRN</td>
            <td>388552</td>
            <td>6758845</td>
        </tr>
        <tr>
            <td>NGA</td>
            <td>271358</td>
            <td>1518000</td>
        </tr>
        <tr>
            <td>TUR</td>
            <td>456888</td>
            <td>8787958</td>
        </tr>
    </tbody>
</table>



<br><br><br>

A `GROUP BY` clause can be considered as equivalent to using `DISTINCT` if no aggregate functions are used:


```sql
%%sql

SELECT
    continent
FROM
    country
GROUP BY
    continent
;
```

     * postgresql://postgres:***@localhost:5432/world
    7 rows affected.
    




<table>
    <thead>
        <tr>
            <th>continent</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Asia</td>
        </tr>
        <tr>
            <td>South America</td>
        </tr>
        <tr>
            <td>North America</td>
        </tr>
        <tr>
            <td>Oceania</td>
        </tr>
        <tr>
            <td>Antarctica</td>
        </tr>
        <tr>
            <td>Africa</td>
        </tr>
        <tr>
            <td>Europe</td>
        </tr>
    </tbody>
</table>




```sql
%%sql

SELECT
    DISTINCT continent
FROM
    country
;
```

     * postgresql://postgres:***@localhost:5432/world
    7 rows affected.
    




<table>
    <thead>
        <tr>
            <th>continent</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Asia</td>
        </tr>
        <tr>
            <td>South America</td>
        </tr>
        <tr>
            <td>North America</td>
        </tr>
        <tr>
            <td>Oceania</td>
        </tr>
        <tr>
            <td>Antarctica</td>
        </tr>
        <tr>
            <td>Africa</td>
        </tr>
        <tr>
            <td>Europe</td>
        </tr>
    </tbody>
</table>



<br><br><br>

**Note:** Neither `GROUP BY` nor `DISTINCT` ignore null values.

For example, here you'll see `None` (Jupyter Notebook's way of showing `NULL`s) appearing at the top when we group rows by `headofstate`, which contain null values:


```sql
%%sql

SELECT
    headofstate
FROM
    country
GROUP BY
    headofstate
ORDER BY
    headofstate DESC
LIMIT
    5
;
```

     * postgresql://postgres:***@localhost:5432/world
    5 rows affected.
    




<table>
    <thead>
        <tr>
            <th>headofstate</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>None</td>
        </tr>
        <tr>
            <td>Ólafur Ragnar Grímsson</td>
        </tr>
        <tr>
            <td>Émile Lahoud</td>
        </tr>
        <tr>
            <td>tipe Mesic</td>
        </tr>
        <tr>
            <td>kenraali Than Shwe</td>
        </tr>
    </tbody>
</table>



Once again remember that:

As long as aggregated, columns appearing in the `HAVING` clause don't necessarily need to be present in the `SELECT` clause. Here, we're retrieving continents having at least 40 countries:


```sql
%%sql

SELECT
    continent
FROM
    country
GROUP BY
    continent
HAVING
    COUNT(name) >= 40
;
```

Note that we didn't use the column `name` in `SELECT`, but we still had access to it.

**Question:** Why are we able to access columns in the `HAVING` clause that are not listed in the `SELECT` clause?

<br><br><br>

### Multi-level grouping

The `GROUP BY` clause can accommodate more than one column to construct multi-level groups. For example, we can group the rows in the `country` table of the `world` database first based on `continent` and then based on `region`, all in one go:


```sql
%%sql

SELECT
    continent, region, AVG(population)::INT
FROM
    country
GROUP BY
    continent, region
ORDER BY
    continent, region
;
```

     * postgresql://postgres:***@localhost:5432/world
    25 rows affected.
    




<table>
    <thead>
        <tr>
            <th>continent</th>
            <th>region</th>
            <th>avg</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Africa</td>
            <td>Central Africa</td>
            <td>10628000</td>
        </tr>
        <tr>
            <td>Africa</td>
            <td>Eastern Africa</td>
            <td>12349950</td>
        </tr>
        <tr>
            <td>Africa</td>
            <td>Northern Africa</td>
            <td>24752286</td>
        </tr>
        <tr>
            <td>Africa</td>
            <td>Southern Africa</td>
            <td>9377200</td>
        </tr>
        <tr>
            <td>Africa</td>
            <td>Western Africa</td>
            <td>13039529</td>
        </tr>
        <tr>
            <td>Antarctica</td>
            <td>Antarctica</td>
            <td>0</td>
        </tr>
        <tr>
            <td>Asia</td>
            <td>Eastern Asia</td>
            <td>188416000</td>
        </tr>
        <tr>
            <td>Asia</td>
            <td>Middle East</td>
            <td>10465594</td>
        </tr>
        <tr>
            <td>Asia</td>
            <td>Southeast Asia</td>
            <td>47140091</td>
        </tr>
        <tr>
            <td>Asia</td>
            <td>Southern and Central Asia</td>
            <td>106484000</td>
        </tr>
        <tr>
            <td>Europe</td>
            <td>Baltic Countries</td>
            <td>2520633</td>
        </tr>
        <tr>
            <td>Europe</td>
            <td>British Islands</td>
            <td>31699250</td>
        </tr>
        <tr>
            <td>Europe</td>
            <td>Eastern Europe</td>
            <td>30702600</td>
        </tr>
        <tr>
            <td>Europe</td>
            <td>Nordic Countries</td>
            <td>3452343</td>
        </tr>
        <tr>
            <td>Europe</td>
            <td>Southern Europe</td>
            <td>9644947</td>
        </tr>
        <tr>
            <td>Europe</td>
            <td>Western Europe</td>
            <td>20360844</td>
        </tr>
        <tr>
            <td>North America</td>
            <td>Caribbean</td>
            <td>1589167</td>
        </tr>
        <tr>
            <td>North America</td>
            <td>Central America</td>
            <td>16902625</td>
        </tr>
        <tr>
            <td>North America</td>
            <td>North America</td>
            <td>61926400</td>
        </tr>
        <tr>
            <td>Oceania</td>
            <td>Australia and New Zealand</td>
            <td>4550620</td>
        </tr>
        <tr>
            <td>Oceania</td>
            <td>Melanesia</td>
            <td>1294400</td>
        </tr>
        <tr>
            <td>Oceania</td>
            <td>Micronesia</td>
            <td>77571</td>
        </tr>
        <tr>
            <td>Oceania</td>
            <td>Micronesia/Caribbean</td>
            <td>0</td>
        </tr>
        <tr>
            <td>Oceania</td>
            <td>Polynesia</td>
            <td>63305</td>
        </tr>
        <tr>
            <td>South America</td>
            <td>South America</td>
            <td>24698571</td>
        </tr>
    </tbody>
</table>



<br><br><br>

## Joins

Joins are probably the most fundamentally important operation in relational databases. The reason is that the whole idea of such databases is that data can be broken down into various tables that are related to each other, and can be joined together whenever related information from multiple tables is required. Consider the following query as an example:

---

**Example:** Write a query that returns the name of all countries along with their corresponding continents and their cities.

---

As we've been working with the `world` database, we immediately notice that information about countries and cities are stored in two different tables, so we should somehow combine or "join" the data from the two tables. 

The syntax for a joining tables in SQL is as follows:

```sql
SELECT
    columns
FROM
    left_table
join_type
    right_table
ON
    join_condition
WHERE
    row_filter
GROUP BY
    columns
HAVING
    group_filter
ORDER BY
    columns
;
```

In this section, we'll learn how to do a join to answer the question we posed for the `world`, but I prefer to use a smaller database to demonstrate various joining methods first, and then use our larger databases.

First, let's create a new database called `mds` on the local host (i.e. our own computer) and connect to it:


```python
%sql CREATE DATABASE mds;
```

     * postgresql://postgres:***@localhost:5432/world
    Done.
    




    []




```python
%sql postgresql://{username}:{password}@{host}:{port}/mds
```




    'Connected: postgres@mds'



The following cell creates two tables with the names `instructor` and `instructor_course` with the information about MDS instructors and courses they teach. **Don't worry about the content of this cell!** We will learn how to create tables in the next lecture. For now, just run the cell to create and populate the tables:


```sql
%%sql

DROP TABLE IF EXISTS
    instructor,
    instructor_course,
    course_cohort
;

CREATE TABLE instructor (
    id INTEGER PRIMARY KEY,
    name TEXT,
    email TEXT,
    phone VARCHAR(12),
    department VARCHAR(50)
    )
;

INSERT INTO
    instructor (id, name, email, phone, department)
VALUES
    (1, 'Mike', 'mike@mds.ubc.ca', '605-332-2343', 'Computer Science'),
    (2, 'Tiffany', 'tiff@mds.ubc.ca', '445-794-2233', 'Neuroscience'),
    (3, 'Arman', 'arman@mds.ubc.ca', '935-738-5796', 'Physics'),
    (4, 'Varada', 'varada@mds.ubc.ca', '243-924-4446', 'Computer Science'),
    (5, 'Quan', 'quan@mds.ubc.ca', '644-818-0254', 'Economics'),
    (6, 'Joel', 'joel@mds.ubc.ca', '773-432-7669', 'Biomedical Engineering'),
    (7, 'Florencia', 'flor@mds.ubc.ca', '773-926-2837', 'Biology'),
    (8, 'Alexi', 'alexiu@mds.ubc.ca', '421-888-4550', 'Statistics'),
    (15, 'Vincenzo', 'vincenzo@mds.ubc.ca', '776-543-1212', 'Statistics'),
    (19, 'Gittu', 'gittu@mds.ubc.ca', '776-334-1132', 'Biomedical Engineering'),
    (16, 'Jessica', 'jessica@mds.ubc.ca', '211-990-1762', 'Computer Science')
;

    
CREATE TABLE instructor_course (
    id SERIAL PRIMARY KEY,
    instructor_id INTEGER,
    course TEXT,
    enrollment INTEGER,
    begins DATE
    )
;

INSERT INTO
    instructor_course (instructor_id, course, enrollment, begins)
VALUES
    (8, 'Statistical Inference and Computation I', 125, '2021-10-01'),
    (8, 'Regression II', 102, '2022-02-05'),
    (1, 'Descriptive Statistics and Probability', 79, '2021-09-10'),
    (1, 'Algorithms and Data Structures', 25, '2021-10-01'),
    (3, 'Algorithms and Data Structures', 25, '2021-10-01'),
    (3, 'Python Programming', 133, '2021-09-07'),
    (3, 'Databases & Data Retrieval', 118, '2021-11-16'),
    (6, 'Visualization I', 155, '2021-10-01'),
    (6, 'Privacy, Ethics & Security', 148, '2022-03-01'),
    (2, 'Programming for Data Manipulation', 160, '2021-09-08'),
    (7, 'Data Science Workflows', 98, '2021-09-15'),
    (2, 'Data Science Workflows', 98, '2021-09-15'),
    (12, 'Web & Cloud Computing', 78, '2022-02-10'),
    (10, 'Introduction to Optimization', NULL, '2022-09-01'),
    (9, 'Parallel Computing', NULL, '2023-01-10'),
    (13, 'Natural Language Processing', NULL, '2023-09-10')
;

CREATE TABLE course_cohort (
    id INTEGER,
    cohort VARCHAR(7)
    )
;

INSERT INTO
    course_cohort (id, cohort)
VALUES
    (13, 'MDS-CL'),
    (8, 'MDS-CL'),
    (1, 'MDS-CL'),
    (3, 'MDS-CL'),
    (1, 'MDS-V'),
    (9, 'MDS-V'),
    (3, 'MDS-V')
;
```

     * postgresql://postgres:***@localhost:5432/mds
       postgresql://postgres:***@localhost:5432/world
    Done.
    Done.
    11 rows affected.
    Done.
    16 rows affected.
    Done.
    7 rows affected.
    




    []



Awesome! Let's take a look at the first two tables:


```python
%sql SELECT * FROM instructor;
```

     * postgresql://postgres:***@localhost:5432/mds
       postgresql://postgres:***@localhost:5432/world
    11 rows affected.
    




<table>
    <thead>
        <tr>
            <th>id</th>
            <th>name</th>
            <th>email</th>
            <th>phone</th>
            <th>department</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>1</td>
            <td>Mike</td>
            <td>mike@mds.ubc.ca</td>
            <td>605-332-2343</td>
            <td>Computer Science</td>
        </tr>
        <tr>
            <td>2</td>
            <td>Tiffany</td>
            <td>tiff@mds.ubc.ca</td>
            <td>445-794-2233</td>
            <td>Neuroscience</td>
        </tr>
        <tr>
            <td>3</td>
            <td>Arman</td>
            <td>arman@mds.ubc.ca</td>
            <td>935-738-5796</td>
            <td>Physics</td>
        </tr>
        <tr>
            <td>4</td>
            <td>Varada</td>
            <td>varada@mds.ubc.ca</td>
            <td>243-924-4446</td>
            <td>Computer Science</td>
        </tr>
        <tr>
            <td>5</td>
            <td>Quan</td>
            <td>quan@mds.ubc.ca</td>
            <td>644-818-0254</td>
            <td>Economics</td>
        </tr>
        <tr>
            <td>6</td>
            <td>Joel</td>
            <td>joel@mds.ubc.ca</td>
            <td>773-432-7669</td>
            <td>Biomedical Engineering</td>
        </tr>
        <tr>
            <td>7</td>
            <td>Florencia</td>
            <td>flor@mds.ubc.ca</td>
            <td>773-926-2837</td>
            <td>Biology</td>
        </tr>
        <tr>
            <td>8</td>
            <td>Alexi</td>
            <td>alexiu@mds.ubc.ca</td>
            <td>421-888-4550</td>
            <td>Statistics</td>
        </tr>
        <tr>
            <td>15</td>
            <td>Vincenzo</td>
            <td>vincenzo@mds.ubc.ca</td>
            <td>776-543-1212</td>
            <td>Statistics</td>
        </tr>
        <tr>
            <td>19</td>
            <td>Gittu</td>
            <td>gittu@mds.ubc.ca</td>
            <td>776-334-1132</td>
            <td>Biomedical Engineering</td>
        </tr>
        <tr>
            <td>16</td>
            <td>Jessica</td>
            <td>jessica@mds.ubc.ca</td>
            <td>211-990-1762</td>
            <td>Computer Science</td>
        </tr>
    </tbody>
</table>




```python
%sql SELECT * FROM instructor_course;
```

     * postgresql://postgres:***@localhost:5432/mds
       postgresql://postgres:***@localhost:5432/world
    16 rows affected.
    




<table>
    <thead>
        <tr>
            <th>id</th>
            <th>instructor_id</th>
            <th>course</th>
            <th>enrollment</th>
            <th>begins</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>1</td>
            <td>8</td>
            <td>Statistical Inference and Computation I</td>
            <td>125</td>
            <td>2021-10-01</td>
        </tr>
        <tr>
            <td>2</td>
            <td>8</td>
            <td>Regression II</td>
            <td>102</td>
            <td>2022-02-05</td>
        </tr>
        <tr>
            <td>3</td>
            <td>1</td>
            <td>Descriptive Statistics and Probability</td>
            <td>79</td>
            <td>2021-09-10</td>
        </tr>
        <tr>
            <td>4</td>
            <td>1</td>
            <td>Algorithms and Data Structures</td>
            <td>25</td>
            <td>2021-10-01</td>
        </tr>
        <tr>
            <td>5</td>
            <td>3</td>
            <td>Algorithms and Data Structures</td>
            <td>25</td>
            <td>2021-10-01</td>
        </tr>
        <tr>
            <td>6</td>
            <td>3</td>
            <td>Python Programming</td>
            <td>133</td>
            <td>2021-09-07</td>
        </tr>
        <tr>
            <td>7</td>
            <td>3</td>
            <td>Databases &amp; Data Retrieval</td>
            <td>118</td>
            <td>2021-11-16</td>
        </tr>
        <tr>
            <td>8</td>
            <td>6</td>
            <td>Visualization I</td>
            <td>155</td>
            <td>2021-10-01</td>
        </tr>
        <tr>
            <td>9</td>
            <td>6</td>
            <td>Privacy, Ethics &amp; Security</td>
            <td>148</td>
            <td>2022-03-01</td>
        </tr>
        <tr>
            <td>10</td>
            <td>2</td>
            <td>Programming for Data Manipulation</td>
            <td>160</td>
            <td>2021-09-08</td>
        </tr>
        <tr>
            <td>11</td>
            <td>7</td>
            <td>Data Science Workflows</td>
            <td>98</td>
            <td>2021-09-15</td>
        </tr>
        <tr>
            <td>12</td>
            <td>2</td>
            <td>Data Science Workflows</td>
            <td>98</td>
            <td>2021-09-15</td>
        </tr>
        <tr>
            <td>13</td>
            <td>12</td>
            <td>Web &amp; Cloud Computing</td>
            <td>78</td>
            <td>2022-02-10</td>
        </tr>
        <tr>
            <td>14</td>
            <td>10</td>
            <td>Introduction to Optimization</td>
            <td>None</td>
            <td>2022-09-01</td>
        </tr>
        <tr>
            <td>15</td>
            <td>9</td>
            <td>Parallel Computing</td>
            <td>None</td>
            <td>2023-01-10</td>
        </tr>
        <tr>
            <td>16</td>
            <td>13</td>
            <td>Natural Language Processing</td>
            <td>None</td>
            <td>2023-09-10</td>
        </tr>
    </tbody>
</table>



<br><br><br>

### Cross join

A cross join is the simplest way to join two tables:

by cross-joining tables A and B, we match each every row from table A with every row from table B!

In other words, it returns all combinations of rows from table A and table B. This type of join is also sometimes called _the Cartesian product_ of two relations or tables:


```python
%config SqlMagic.displaylimit = 200
```


```sql
%%sql

SELECT
    *
FROM
    instructor
CROSS JOIN
    instructor_course
;
```

<br><br><br>

**How to deal with ambiguous column names**

Now suppose that we want to return only the names of instructors and their IDs from the `instructor` table, and names of courses and their IDs from the `course` table. Since there is a column named `id` in both tables, we cannot use `id` in the `SELECT` clause, because it would be ambiguous.

In this situation, we should either prepend the column name by the full name of its parent table (e.g. `instructor.id`), or we can create table aliases using the keyword `AS` (just like we did before with columns) and prepend the column name with the parent table alias. A table name followed by a dot and the name of a column is called a **_qualified name_**.

Here is an example of using qualified names for ambiguous column names:


```sql
%%sql

SELECT
    name, i.id, course, ic.id
FROM
    instructor AS i
CROSS JOIN
    instructor_course AS ic
LIMIT 10
;
```

     * postgresql://postgres:***@localhost:5432/mds
       postgresql://postgres:***@localhost:5432/world
    10 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>id</th>
            <th>course</th>
            <th>id_1</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Mike</td>
            <td>1</td>
            <td>Statistical Inference and Computation I</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Tiffany</td>
            <td>2</td>
            <td>Statistical Inference and Computation I</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Arman</td>
            <td>3</td>
            <td>Statistical Inference and Computation I</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Varada</td>
            <td>4</td>
            <td>Statistical Inference and Computation I</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Quan</td>
            <td>5</td>
            <td>Statistical Inference and Computation I</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Joel</td>
            <td>6</td>
            <td>Statistical Inference and Computation I</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Florencia</td>
            <td>7</td>
            <td>Statistical Inference and Computation I</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Alexi</td>
            <td>8</td>
            <td>Statistical Inference and Computation I</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Vincenzo</td>
            <td>15</td>
            <td>Statistical Inference and Computation I</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Gittu</td>
            <td>19</td>
            <td>Statistical Inference and Computation I</td>
            <td>1</td>
        </tr>
    </tbody>
</table>



- The keyword `AS` can be dropped

- Table aliases only exist during the execution of a statement
- Using table aliases is a great way to reduce clutter in SQL join statements
- Once you create an alias for a table, you should only use the alias to refer to that table in the statement. For example, the following query would throw an error:

```sql
-- This is WRONG
SELECT
    instructor.name, instructor.id, course, ic.id
FROM
    instructor AS i
CROSS JOIN
    instructor_course AS ic
;
```

- When you retrieve data from multiple tables, you can still use `*` to return all columns of a particular table. The only difference is that you should prepend it with the corresponding tables name. For example, we can return all columns of the table `instructor` and just the `course` column of the table `instructor_course` in a cross join with the following query:

```sql
SELECT
    i.*, ic.course
FROM
    instructor AS i
CROSS JOIN
    instructor_course AS ic
;
```

<br><br><br>

### Inner join

Except for a cross join, all other types of joins use a condition using the `ON` keyword to figure out which rows from the two tables to pair up. An inner join is a type of join that only returns the matching rows from the left and right tables. The image below ([source](https://www.postgresqltutorial.com/postgresql-joins/)) shows Venn diagram of an inner join:

<img src="img/lecture3/inner_join.png" width="250">

For example, in our `instructor` table there are some instructors who are assigned one or more courses in the `instructor_course` table, some who are not. Similarly, there are courses in the `instructor_course` table that have an instructor, and some that don't have an instructor yet. With an inner join based on `instructor.id` and `instructor_course.id` columns, we would retrieve matching rows, meaning that only instructors are retrieved that have one or more assigned courses, and vice versa:


```sql
%%sql

SELECT
    name, i.id, ic.instructor_id, course
FROM
    instructor AS i
INNER JOIN
    instructor_course AS ic
ON
    i.id = ic.instructor_id
;
```

     * postgresql://postgres:***@localhost:5432/mds
       postgresql://postgres:***@localhost:5432/world
    12 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>id</th>
            <th>instructor_id</th>
            <th>course</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Alexi</td>
            <td>8</td>
            <td>8</td>
            <td>Statistical Inference and Computation I</td>
        </tr>
        <tr>
            <td>Alexi</td>
            <td>8</td>
            <td>8</td>
            <td>Regression II</td>
        </tr>
        <tr>
            <td>Mike</td>
            <td>1</td>
            <td>1</td>
            <td>Descriptive Statistics and Probability</td>
        </tr>
        <tr>
            <td>Mike</td>
            <td>1</td>
            <td>1</td>
            <td>Algorithms and Data Structures</td>
        </tr>
        <tr>
            <td>Arman</td>
            <td>3</td>
            <td>3</td>
            <td>Algorithms and Data Structures</td>
        </tr>
        <tr>
            <td>Arman</td>
            <td>3</td>
            <td>3</td>
            <td>Python Programming</td>
        </tr>
        <tr>
            <td>Arman</td>
            <td>3</td>
            <td>3</td>
            <td>Databases &amp; Data Retrieval</td>
        </tr>
        <tr>
            <td>Joel</td>
            <td>6</td>
            <td>6</td>
            <td>Visualization I</td>
        </tr>
        <tr>
            <td>Joel</td>
            <td>6</td>
            <td>6</td>
            <td>Privacy, Ethics &amp; Security</td>
        </tr>
        <tr>
            <td>Tiffany</td>
            <td>2</td>
            <td>2</td>
            <td>Programming for Data Manipulation</td>
        </tr>
        <tr>
            <td>Florencia</td>
            <td>7</td>
            <td>7</td>
            <td>Data Science Workflows</td>
        </tr>
        <tr>
            <td>Tiffany</td>
            <td>2</td>
            <td>2</td>
            <td>Data Science Workflows</td>
        </tr>
    </tbody>
</table>



In the above returned table, "Quan" and "Varada" are missing as instructors since they are not yet assigned any courses. Also, the courses "Web & Cloud Computing", "Parallel Computing", and "Introduction to Optimization" are missing, since there not yet any instructors assigned for these courses.

> **Note:** The `INNER` keyword is optional.

<br><br><br>

### Self join

Sometimes we want to compare a table to itself. For example, we may want to know which paris of instructors in the `instructors` table are from the same department. In order to find out, we need to compare the values in the `department` column of each row to all other rows to find matches:


```sql
%%sql

SELECT
    i1.name, i1.department, i2.department, i2.name
FROM
    instructor i1
JOIN
    instructor i2
ON
    i1.department = i2.department
    AND
    i1.id <> i2.id
;
```

     * postgresql://postgres:***@localhost:5432/mds
       postgresql://postgres:***@localhost:5432/world
    10 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>department</th>
            <th>department_1</th>
            <th>name_1</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Mike</td>
            <td>Computer Science</td>
            <td>Computer Science</td>
            <td>Jessica</td>
        </tr>
        <tr>
            <td>Mike</td>
            <td>Computer Science</td>
            <td>Computer Science</td>
            <td>Varada</td>
        </tr>
        <tr>
            <td>Varada</td>
            <td>Computer Science</td>
            <td>Computer Science</td>
            <td>Jessica</td>
        </tr>
        <tr>
            <td>Varada</td>
            <td>Computer Science</td>
            <td>Computer Science</td>
            <td>Mike</td>
        </tr>
        <tr>
            <td>Joel</td>
            <td>Biomedical Engineering</td>
            <td>Biomedical Engineering</td>
            <td>Gittu</td>
        </tr>
        <tr>
            <td>Alexi</td>
            <td>Statistics</td>
            <td>Statistics</td>
            <td>Vincenzo</td>
        </tr>
        <tr>
            <td>Vincenzo</td>
            <td>Statistics</td>
            <td>Statistics</td>
            <td>Alexi</td>
        </tr>
        <tr>
            <td>Gittu</td>
            <td>Biomedical Engineering</td>
            <td>Biomedical Engineering</td>
            <td>Joel</td>
        </tr>
        <tr>
            <td>Jessica</td>
            <td>Computer Science</td>
            <td>Computer Science</td>
            <td>Varada</td>
        </tr>
        <tr>
            <td>Jessica</td>
            <td>Computer Science</td>
            <td>Computer Science</td>
            <td>Mike</td>
        </tr>
    </tbody>
</table>



The `i1.id <> i2.id` join condition ensures that a row does not match itself.

<br><br><br>

### Natural join

For joins involving a join condition, e.g. inner or self joins, we have so far explicitly specified the matching condition. In a situation that columns in different tables have the same name and we want to simply match rows with similar values **in all similarly named columns**, we can do a **natural join** using the keywords `NATURAL JOIN`. For example, the `id` column in the `course_cohort` refers to the `id` column let's find which courses are offered for which cohorts using a natural join:


```python
%sql SELECT * FROM course_cohort;
```

     * postgresql://postgres:***@localhost:5432/mds
       postgresql://postgres:***@localhost:5432/world
    7 rows affected.
    




<table>
    <thead>
        <tr>
            <th>id</th>
            <th>cohort</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>13</td>
            <td>MDS-CL</td>
        </tr>
        <tr>
            <td>8</td>
            <td>MDS-CL</td>
        </tr>
        <tr>
            <td>1</td>
            <td>MDS-CL</td>
        </tr>
        <tr>
            <td>3</td>
            <td>MDS-CL</td>
        </tr>
        <tr>
            <td>1</td>
            <td>MDS-V</td>
        </tr>
        <tr>
            <td>9</td>
            <td>MDS-V</td>
        </tr>
        <tr>
            <td>3</td>
            <td>MDS-V</td>
        </tr>
    </tbody>
</table>




```sql
%%sql

SELECT
    ic.course, cc.cohort
FROM
    instructor_course ic
NATURAL JOIN
    course_cohort cc
;
```

     * postgresql://postgres:***@localhost:5432/mds
       postgresql://postgres:***@localhost:5432/world
    7 rows affected.
    




<table>
    <thead>
        <tr>
            <th>course</th>
            <th>cohort</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Web &amp; Cloud Computing</td>
            <td>MDS-CL</td>
        </tr>
        <tr>
            <td>Visualization I</td>
            <td>MDS-CL</td>
        </tr>
        <tr>
            <td>Statistical Inference and Computation I</td>
            <td>MDS-CL</td>
        </tr>
        <tr>
            <td>Descriptive Statistics and Probability</td>
            <td>MDS-CL</td>
        </tr>
        <tr>
            <td>Statistical Inference and Computation I</td>
            <td>MDS-V</td>
        </tr>
        <tr>
            <td>Privacy, Ethics &amp; Security</td>
            <td>MDS-V</td>
        </tr>
        <tr>
            <td>Descriptive Statistics and Probability</td>
            <td>MDS-V</td>
        </tr>
    </tbody>
</table>



If there are no matching columns in the two tables, `NATURAL JOIN` acts like `JOIN ... ON TRUE` and results in a cross-product join between the participating tables.

<br><br><br>

### Outer joins

An outer join is a type of join that returns all the rows from one or both of the tables that takes part in the joining. Outer joins are useful in questions that involve missing values.

#### Left outer join

In the joining process, the first table from which data is retrieved using `SELECT` is called the **left** table, and the table that is joined onto that is called the **right** table. In other words, the first table that appears in the query is the left table (table on the left of the query), and the one appearing later is the right table (table on the right of the query).

A left outer join is a type of join that returns all rows from the left table (matching or not), in addition to the matching rows from both tables. The non-matching rows from the left table are assigned null values in the columns that belong to the 
right table. This is schematically shown in the diagram below ([source](https://www.postgresqltutorial.com/postgresql-joins/)):

<img src="img/lecture3/left_join.png" width="250">

For example, in the [inner join](#Inner-join) example, instructors who don't teach any course are not returned by the join operation. Let's say we want to retrieve a list of all instructors and the courses they teach, as well as those who don't teach any courses:


```sql
%%sql

SELECT
    name, i.id, ic.instructor_id, course
FROM
    instructor AS i
LEFT OUTER JOIN
    instructor_course AS ic
ON
    i.id = ic.instructor_id
;
```

     * postgresql://postgres:***@localhost:5432/mds
       postgresql://postgres:***@localhost:5432/world
    17 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>id</th>
            <th>instructor_id</th>
            <th>course</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Alexi</td>
            <td>8</td>
            <td>8</td>
            <td>Statistical Inference and Computation I</td>
        </tr>
        <tr>
            <td>Alexi</td>
            <td>8</td>
            <td>8</td>
            <td>Regression II</td>
        </tr>
        <tr>
            <td>Mike</td>
            <td>1</td>
            <td>1</td>
            <td>Descriptive Statistics and Probability</td>
        </tr>
        <tr>
            <td>Mike</td>
            <td>1</td>
            <td>1</td>
            <td>Algorithms and Data Structures</td>
        </tr>
        <tr>
            <td>Arman</td>
            <td>3</td>
            <td>3</td>
            <td>Algorithms and Data Structures</td>
        </tr>
        <tr>
            <td>Arman</td>
            <td>3</td>
            <td>3</td>
            <td>Python Programming</td>
        </tr>
        <tr>
            <td>Arman</td>
            <td>3</td>
            <td>3</td>
            <td>Databases &amp; Data Retrieval</td>
        </tr>
        <tr>
            <td>Joel</td>
            <td>6</td>
            <td>6</td>
            <td>Visualization I</td>
        </tr>
        <tr>
            <td>Joel</td>
            <td>6</td>
            <td>6</td>
            <td>Privacy, Ethics &amp; Security</td>
        </tr>
        <tr>
            <td>Tiffany</td>
            <td>2</td>
            <td>2</td>
            <td>Programming for Data Manipulation</td>
        </tr>
        <tr>
            <td>Florencia</td>
            <td>7</td>
            <td>7</td>
            <td>Data Science Workflows</td>
        </tr>
        <tr>
            <td>Tiffany</td>
            <td>2</td>
            <td>2</td>
            <td>Data Science Workflows</td>
        </tr>
        <tr>
            <td>Vincenzo</td>
            <td>15</td>
            <td>None</td>
            <td>None</td>
        </tr>
        <tr>
            <td>Quan</td>
            <td>5</td>
            <td>None</td>
            <td>None</td>
        </tr>
        <tr>
            <td>Gittu</td>
            <td>19</td>
            <td>None</td>
            <td>None</td>
        </tr>
        <tr>
            <td>Jessica</td>
            <td>16</td>
            <td>None</td>
            <td>None</td>
        </tr>
        <tr>
            <td>Varada</td>
            <td>4</td>
            <td>None</td>
            <td>None</td>
        </tr>
    </tbody>
</table>



> **Note:** The keyword `OUTER` is optional.

How can this be helpful? As an example, we can return the name of instructors who don't teach any courses with the following query:


```sql
%%sql

SELECT
    name
FROM
    instructor AS i
LEFT JOIN
    instructor_course AS ic
ON
    i.id = ic.instructor_id
WHERE
    ic.course IS NULL
;
```

     * postgresql://postgres:***@localhost:5432/mds
       postgresql://postgres:***@localhost:5432/world
    5 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Vincenzo</td>
        </tr>
        <tr>
            <td>Quan</td>
        </tr>
        <tr>
            <td>Gittu</td>
        </tr>
        <tr>
            <td>Jessica</td>
        </tr>
        <tr>
            <td>Varada</td>
        </tr>
    </tbody>
</table>



#### Right outer join

A right join acts exactly in the same way as a left join, except that it keeps all rows from the right table and only the matching ones from the left table. The diagram below demonstrates a right join schematically ([source](https://www.postgresqltutorial.com/postgresql-joins/)):

<img src="img/lecture3/right_join.png" width="250">

Let's retrieve a list of all courses and their appointed instructors, as well as those courses without an instructor:


```sql
%%sql

SELECT
    name, i.id, ic.instructor_id, course
FROM
    instructor AS i
RIGHT OUTER JOIN
    instructor_course AS ic
ON
    i.id = ic.instructor_id
;
```

     * postgresql://postgres:***@localhost:5432/mds
       postgresql://postgres:***@localhost:5432/world
    16 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>id</th>
            <th>instructor_id</th>
            <th>course</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Alexi</td>
            <td>8</td>
            <td>8</td>
            <td>Statistical Inference and Computation I</td>
        </tr>
        <tr>
            <td>Alexi</td>
            <td>8</td>
            <td>8</td>
            <td>Regression II</td>
        </tr>
        <tr>
            <td>Mike</td>
            <td>1</td>
            <td>1</td>
            <td>Descriptive Statistics and Probability</td>
        </tr>
        <tr>
            <td>Mike</td>
            <td>1</td>
            <td>1</td>
            <td>Algorithms and Data Structures</td>
        </tr>
        <tr>
            <td>Arman</td>
            <td>3</td>
            <td>3</td>
            <td>Algorithms and Data Structures</td>
        </tr>
        <tr>
            <td>Arman</td>
            <td>3</td>
            <td>3</td>
            <td>Python Programming</td>
        </tr>
        <tr>
            <td>Arman</td>
            <td>3</td>
            <td>3</td>
            <td>Databases &amp; Data Retrieval</td>
        </tr>
        <tr>
            <td>Joel</td>
            <td>6</td>
            <td>6</td>
            <td>Visualization I</td>
        </tr>
        <tr>
            <td>Joel</td>
            <td>6</td>
            <td>6</td>
            <td>Privacy, Ethics &amp; Security</td>
        </tr>
        <tr>
            <td>Tiffany</td>
            <td>2</td>
            <td>2</td>
            <td>Programming for Data Manipulation</td>
        </tr>
        <tr>
            <td>Florencia</td>
            <td>7</td>
            <td>7</td>
            <td>Data Science Workflows</td>
        </tr>
        <tr>
            <td>Tiffany</td>
            <td>2</td>
            <td>2</td>
            <td>Data Science Workflows</td>
        </tr>
        <tr>
            <td>None</td>
            <td>None</td>
            <td>12</td>
            <td>Web &amp; Cloud Computing</td>
        </tr>
        <tr>
            <td>None</td>
            <td>None</td>
            <td>10</td>
            <td>Introduction to Optimization</td>
        </tr>
        <tr>
            <td>None</td>
            <td>None</td>
            <td>9</td>
            <td>Parallel Computing</td>
        </tr>
        <tr>
            <td>None</td>
            <td>None</td>
            <td>13</td>
            <td>Natural Language Processing</td>
        </tr>
    </tbody>
</table>



Now, let's find out which courses do not have an appointed instructor yet:


```sql
%%sql

SELECT
    course
FROM
    instructor AS i
RIGHT OUTER JOIN
    instructor_course AS ic
ON
    i.id = ic.instructor_id
WHERE
    i.id IS NULL
;
```

     * postgresql://postgres:***@localhost:5432/mds
       postgresql://postgres:***@localhost:5432/world
    4 rows affected.
    




<table>
    <thead>
        <tr>
            <th>course</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Web &amp; Cloud Computing</td>
        </tr>
        <tr>
            <td>Introduction to Optimization</td>
        </tr>
        <tr>
            <td>Parallel Computing</td>
        </tr>
        <tr>
            <td>Natural Language Processing</td>
        </tr>
    </tbody>
</table>



#### Full outer join

A full outer join is the combination of a left and right join: it retrieves **matching and non-matching** rows from **both** tables. Take a look at the schematic diagram of a full outer join ([source](https://www.postgresqltutorial.com/postgresql-joins/)):

<img src="img/lecture3/full_outer_join.png" width="250">

Let's do a full outer join between the `instructor` and `instructor_course` tables to retrieve all instructors and courses:


```sql
%%sql

SELECT
    name, i.id, ic.instructor_id, course
FROM
    instructor AS i
FULL OUTER JOIN
    instructor_course AS ic
ON
    i.id = ic.instructor_id
;
```

     * postgresql://postgres:***@localhost:5432/mds
       postgresql://postgres:***@localhost:5432/world
    21 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>id</th>
            <th>instructor_id</th>
            <th>course</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Alexi</td>
            <td>8</td>
            <td>8</td>
            <td>Statistical Inference and Computation I</td>
        </tr>
        <tr>
            <td>Alexi</td>
            <td>8</td>
            <td>8</td>
            <td>Regression II</td>
        </tr>
        <tr>
            <td>Mike</td>
            <td>1</td>
            <td>1</td>
            <td>Descriptive Statistics and Probability</td>
        </tr>
        <tr>
            <td>Mike</td>
            <td>1</td>
            <td>1</td>
            <td>Algorithms and Data Structures</td>
        </tr>
        <tr>
            <td>Arman</td>
            <td>3</td>
            <td>3</td>
            <td>Algorithms and Data Structures</td>
        </tr>
        <tr>
            <td>Arman</td>
            <td>3</td>
            <td>3</td>
            <td>Python Programming</td>
        </tr>
        <tr>
            <td>Arman</td>
            <td>3</td>
            <td>3</td>
            <td>Databases &amp; Data Retrieval</td>
        </tr>
        <tr>
            <td>Joel</td>
            <td>6</td>
            <td>6</td>
            <td>Visualization I</td>
        </tr>
        <tr>
            <td>Joel</td>
            <td>6</td>
            <td>6</td>
            <td>Privacy, Ethics &amp; Security</td>
        </tr>
        <tr>
            <td>Tiffany</td>
            <td>2</td>
            <td>2</td>
            <td>Programming for Data Manipulation</td>
        </tr>
        <tr>
            <td>Florencia</td>
            <td>7</td>
            <td>7</td>
            <td>Data Science Workflows</td>
        </tr>
        <tr>
            <td>Tiffany</td>
            <td>2</td>
            <td>2</td>
            <td>Data Science Workflows</td>
        </tr>
        <tr>
            <td>None</td>
            <td>None</td>
            <td>12</td>
            <td>Web &amp; Cloud Computing</td>
        </tr>
        <tr>
            <td>None</td>
            <td>None</td>
            <td>10</td>
            <td>Introduction to Optimization</td>
        </tr>
        <tr>
            <td>None</td>
            <td>None</td>
            <td>9</td>
            <td>Parallel Computing</td>
        </tr>
        <tr>
            <td>None</td>
            <td>None</td>
            <td>13</td>
            <td>Natural Language Processing</td>
        </tr>
        <tr>
            <td>Vincenzo</td>
            <td>15</td>
            <td>None</td>
            <td>None</td>
        </tr>
        <tr>
            <td>Quan</td>
            <td>5</td>
            <td>None</td>
            <td>None</td>
        </tr>
        <tr>
            <td>Gittu</td>
            <td>19</td>
            <td>None</td>
            <td>None</td>
        </tr>
        <tr>
            <td>Jessica</td>
            <td>16</td>
            <td>None</td>
            <td>None</td>
        </tr>
        <tr>
            <td>Varada</td>
            <td>4</td>
            <td>None</td>
            <td>None</td>
        </tr>
    </tbody>
</table>



We can now write a query to find instructors who are free to teach a course, and courses that need an instructor:


```sql
%%sql

SELECT
    name, course
FROM
    instructor AS i
FULL OUTER JOIN
    instructor_course AS ic
ON
    i.id = ic.instructor_id
WHERE
    i.name IS NULL
    OR
    ic.course IS NULL
;
```

<br><br><br>

---

**Question:** What's the difference between a cross join and a full outer join?

---

<br><br><br>

### `WHERE` joins (OPTIONAL)

Prior to the SQL-92 standard, there were no reserved keywords (e.g `JOIN`, `CROSS JOIN`, `LEFT JOIN`) for joining tables in SQL. Instead, joins used to be done by pulling data from two (or more) tables directly in the `FROM` clause, and filtering for the desired rows using the `WHERE` clause, which is why this type of join is known as a `WHERE` join.


The following syntax returns the result of a cross join of `table1` and `table2`:

```sql
SELECT
    table1_columns, table2_columns
FROM
    table1, table2
```

<br><br>

In order to create an **inner** join, for example, we can express the join condition in the `WHERE` clause:

```sql
SELECT
    table1_columns, table2_columns
FROM
    table1, table2
WHERE
    table1.column1 = table2.column1
```

Let's give this a try using the tables we created in this section. First, I'll do a simple cross join of `instructor` and `instructor_course`:


```sql
%%sql

SELECT
    i.*, ic.*
FROM
    instructor i,
    instructor_course ic
;
```

To turn it into an inner join on two related columns, I can use the `WHERE` clause to check for equality of values in those columns:


```sql
%%sql

SELECT
    i.*, ic.*
FROM
    instructor i,
    instructor_course ic
WHERE
    i.id = ic.instructor_id
;
```

Note that there is not a predefined way in SQL to perform an outer join using this older syntax. Each RDBMS provides its own specific syntax for doing outer joins. This is why using the newer `OUTER JOIN` keywords is almost always preferred, as it is much cleaner and more explicit.

<br><br><br>

### `USING` keyword (OPTIONAL)

If there are multiple columns with the same names in both participating tables of a join, we can take advantage of the keyword `USING` and write:

```sql
SELECT
    ...
FROM
    left_table t1
JOIN
    right_table t2
USING
    (column1, column2, ...)
```

instead of

```sql
SELECT
    ...
FROM
    left_table t1
JOIN
    right_table t2
ON
    t1.column1 = t2.column1
    AND
    t1.column2 = t2.column2
    AND
    .
    .
    .
```
