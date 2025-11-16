

# Lecture 6: Views, CTEs, window functions, indexing

## Lecture outline

- Views
- Common Table Expressions (CTEs)
- Window functions
- indexing and performance

<br><br><br>

## Views

In the last lecture, we tried to write a query for the following example:

---

**Example:** Which countries speak at least one language that is not spoken an any other country in their continent?

---

<br><br><br>

This query involves data both from the `country` table as well and the `countrylanguage` table.

On the other hand, we also had to use an `EXISTS` subquery. This was why we first created an intermediate query to first store the results from joining the two tables, and then add the subquery part.

While that solution worked, it might not always be ideal to store the data in another table (why?).

<br><br><br>

Although the separation of country data from language data makes sense from a logical point of view, it might happen that country + language data are the ones that are accessed the most by our users. Also it might be the case that our users only need names, continents, and population data from the `country` table, and language and percentage from the `countrylanguage` table. In this situation, we can create a **view** of the data that we have in our database.

A view is a **named** result of a `SELECT` query that behaves just like a table; i.e. you can query it just like you query any other table in your database.

<br><br><br>

### How to create and delete a view

```sql
CREATE VIEW view_name AS
    select_statment
;
```

<br>

```sql
DROP VIEW [IF EXISTS] view_name;
```

<br><br><br>

### Properties of a view

- A view acts as if it is a table, while it's actually only a way of looking at the data differently. This is why they're sometimes called **virtual tables**. As opposed to the temporary table approach, there is no duplication of data.

- A view is always **current and dynamic**. Whenever we access the view, the query that generates it is re-run and we will see an up-to-date version of the data. This may not be true with a temporary table.

- **Views persist.** You might argue that a temporary table only occupies space in the duration of a session, but it's not certain if that's truly an advantage. While it's good that the taken space is released after closing the connection, each time a user needs that particular query, the table also needs to be recreated.

- Views can **hide unnecessary details, or add details that are computed on demand**. Especially in complex databases, not every database user needs to, or wants to have access to all details of the dataset. Can you think of an example?

- With views we can also **manage what different users can see**. For example, in the database of a company not every department needs (or should) have access to all financial information of employees. But they may need their account and SIN numbers to be able to pay them for the job the do for that department. This can be easily managed with views.

- Views do not support constraints.

<br><br><br>

### Materialized views: the best of both worlds

Sometimes the query underlying a view might be too slow to compute on the fly in a simple view. In this situation, there is a way to physically store a view. In other words, views can be **materialized**.

A materialized view stores the results of the view query on the physical disk, instead of recomputing the entire query, which is why materialized views are much faster to use.

<br><br><br>

Here is the syntax for creating and dropping materialized views:

```sql
CREATE MATERIALIZED VIEW my_mat_view AS
    select_statement
;

DROP MATERIALIZED VIEW my_mat_view;
```

<br><br><br>

Like almost anything in this world, you won't gain anything unless you lose something else. Same is true with materialized views. The **trade-offs** you need to be aware of when using materialized views:

- Just like a temporary table, a materialized view occupies disk space

- The data in a materialized view needs to be refreshed if the data in the related tables are modified.

<br><br><br>

Materialized views can be refreshed using the following syntax:

```sql
REFRESH MATERIALIZED VIEW [CONCURRENTLY] my_mat_view;
```

**Note:** If the keyword `CONCURRENTLY` is used, the view will be usable during the refresh process, but there's no guarantee that the data will be up to date.

<br><br><br>

**Takeaway:** Using a materialized view is justified if the query takes **a lot of time but little space**. A simple query, on the other hand, is best suited to a query that takes **a considerable amount of space, but relatively little time**.

<br><br><br>

---

**Question:** Can you think of two advantages of a temporary table over a simple view?

---

<br><br><br>


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
%sql postgresql://{username}:{password}@{host}/imdb
```




    'Connected: postgres@imdb'



In this example here, I have intentionally written a slow query (involving a correlated subquery) to find movies whose number of votes is higher than the average number of votes of all movies belonging to the same genre. Running this query will take some time:


```sql
%%sql

SELECT
    *
FROM
    movies m
JOIN
    movie_genres mg
ON
    m.id = mg.movie_id
WHERE        
    rating > 8
    AND
    nvotes > (
        SELECT
            AVG(m2.nvotes)
        FROM
            movies m2
        JOIN
            movie_genres mg2
        ON
            m2.id = mg2.movie_id
        WHERE
            mg.genre = mg2.genre
            AND
            m.id <> m2.id
    )
;
```

     * postgresql://postgres:***@localhost/imdb
    795 rows affected.
    




<table>
    <thead>
        <tr>
            <th>id</th>
            <th>title</th>
            <th>orig_title</th>
            <th>start_year</th>
            <th>end_year</th>
            <th>runtime</th>
            <th>rating</th>
            <th>nvotes</th>
            <th>movie_id</th>
            <th>genre</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>10042876</td>
            <td>Rashomon</td>
            <td>Rashômon</td>
            <td>1950</td>
            <td>None</td>
            <td>88</td>
            <td>8.2</td>
            <td>138304</td>
            <td>10042876</td>
            <td>crime</td>
        </tr>
        <tr>
            <td>10042876</td>
            <td>Rashomon</td>
            <td>Rashômon</td>
            <td>1950</td>
            <td>None</td>
            <td>88</td>
            <td>8.2</td>
            <td>138304</td>
            <td>10042876</td>
            <td>drama</td>
        </tr>
        <tr>
            <td>10042876</td>
            <td>Rashomon</td>
            <td>Rashômon</td>
            <td>1950</td>
            <td>None</td>
            <td>88</td>
            <td>8.2</td>
            <td>138304</td>
            <td>10042876</td>
            <td>mystery</td>
        </tr>
        <tr>
            <td>10043014</td>
            <td>Sunset Blvd.</td>
            <td>None</td>
            <td>1950</td>
            <td>None</td>
            <td>110</td>
            <td>8.4</td>
            <td>183282</td>
            <td>10043014</td>
            <td>drama</td>
        </tr>
        <tr>
            <td>10043014</td>
            <td>Sunset Blvd.</td>
            <td>None</td>
            <td>1950</td>
            <td>None</td>
            <td>110</td>
            <td>8.4</td>
            <td>183282</td>
            <td>10043014</td>
            <td>film-noir</td>
        </tr>
        <tr>
            <td>10044741</td>
            <td>Ikiru</td>
            <td>None</td>
            <td>1952</td>
            <td>None</td>
            <td>143</td>
            <td>8.3</td>
            <td>60061</td>
            <td>10044741</td>
            <td>drama</td>
        </tr>
        <tr>
            <td>10044837</td>
            <td>Limelight</td>
            <td>None</td>
            <td>1952</td>
            <td>None</td>
            <td>137</td>
            <td>8.1</td>
            <td>16445</td>
            <td>10044837</td>
            <td>music</td>
        </tr>
        <tr>
            <td>10045152</td>
            <td>Singin&#x27; in the Rain</td>
            <td>None</td>
            <td>1952</td>
            <td>None</td>
            <td>103</td>
            <td>8.3</td>
            <td>201077</td>
            <td>10045152</td>
            <td>comedy</td>
        </tr>
        <tr>
            <td>10045152</td>
            <td>Singin&#x27; in the Rain</td>
            <td>None</td>
            <td>1952</td>
            <td>None</td>
            <td>103</td>
            <td>8.3</td>
            <td>201077</td>
            <td>10045152</td>
            <td>musical</td>
        </tr>
        <tr>
            <td>10045152</td>
            <td>Singin&#x27; in the Rain</td>
            <td>None</td>
            <td>1952</td>
            <td>None</td>
            <td>103</td>
            <td>8.3</td>
            <td>201077</td>
            <td>10045152</td>
            <td>romance</td>
        </tr>
        <tr>
            <td>10046268</td>
            <td>The Wages of Fear</td>
            <td>Le salaire de la peur</td>
            <td>1953</td>
            <td>None</td>
            <td>131</td>
            <td>8.1</td>
            <td>49244</td>
            <td>10046268</td>
            <td>drama</td>
        </tr>
        <tr>
            <td>10046268</td>
            <td>The Wages of Fear</td>
            <td>Le salaire de la peur</td>
            <td>1953</td>
            <td>None</td>
            <td>131</td>
            <td>8.1</td>
            <td>49244</td>
            <td>10046268</td>
            <td>thriller</td>
        </tr>
        <tr>
            <td>10046478</td>
            <td>Ugetsu</td>
            <td>Ugetsu monogatari</td>
            <td>1953</td>
            <td>None</td>
            <td>96</td>
            <td>8.3</td>
            <td>18356</td>
            <td>10046478</td>
            <td>war</td>
        </tr>
        <tr>
            <td>10046912</td>
            <td>Dial M for Murder</td>
            <td>None</td>
            <td>1954</td>
            <td>None</td>
            <td>105</td>
            <td>8.2</td>
            <td>143268</td>
            <td>10046912</td>
            <td>crime</td>
        </tr>
        <tr>
            <td>10046912</td>
            <td>Dial M for Murder</td>
            <td>None</td>
            <td>1954</td>
            <td>None</td>
            <td>105</td>
            <td>8.2</td>
            <td>143268</td>
            <td>10046912</td>
            <td>thriller</td>
        </tr>
        <tr>
            <td>10047296</td>
            <td>On the Waterfront</td>
            <td>None</td>
            <td>1954</td>
            <td>None</td>
            <td>108</td>
            <td>8.1</td>
            <td>130916</td>
            <td>10047296</td>
            <td>crime</td>
        </tr>
        <tr>
            <td>10047296</td>
            <td>On the Waterfront</td>
            <td>None</td>
            <td>1954</td>
            <td>None</td>
            <td>108</td>
            <td>8.1</td>
            <td>130916</td>
            <td>10047296</td>
            <td>drama</td>
        </tr>
        <tr>
            <td>10047296</td>
            <td>On the Waterfront</td>
            <td>None</td>
            <td>1954</td>
            <td>None</td>
            <td>108</td>
            <td>8.1</td>
            <td>130916</td>
            <td>10047296</td>
            <td>thriller</td>
        </tr>
        <tr>
            <td>10047396</td>
            <td>Rear Window</td>
            <td>None</td>
            <td>1954</td>
            <td>None</td>
            <td>112</td>
            <td>8.5</td>
            <td>407290</td>
            <td>10047396</td>
            <td>mystery</td>
        </tr>
        <tr>
            <td>10047396</td>
            <td>Rear Window</td>
            <td>None</td>
            <td>1954</td>
            <td>None</td>
            <td>112</td>
            <td>8.5</td>
            <td>407290</td>
            <td>10047396</td>
            <td>thriller</td>
        </tr>
        <tr>
            <td>10050083</td>
            <td>12 Angry Men</td>
            <td>None</td>
            <td>1957</td>
            <td>None</td>
            <td>96</td>
            <td>8.9</td>
            <td>610724</td>
            <td>10050083</td>
            <td>drama</td>
        </tr>
        <tr>
            <td>10050613</td>
            <td>Throne of Blood</td>
            <td>Kumonosu-jô</td>
            <td>1957</td>
            <td>None</td>
            <td>110</td>
            <td>8.1</td>
            <td>40639</td>
            <td>10050613</td>
            <td>drama</td>
        </tr>
        <tr>
            <td>10050613</td>
            <td>Throne of Blood</td>
            <td>Kumonosu-jô</td>
            <td>1957</td>
            <td>None</td>
            <td>110</td>
            <td>8.1</td>
            <td>40639</td>
            <td>10050613</td>
            <td>history</td>
        </tr>
        <tr>
            <td>10050783</td>
            <td>The Nights of Cabiria</td>
            <td>Le notti di Cabiria</td>
            <td>1957</td>
            <td>None</td>
            <td>110</td>
            <td>8.1</td>
            <td>39350</td>
            <td>10050783</td>
            <td>drama</td>
        </tr>
        <tr>
            <td>10050825</td>
            <td>Paths of Glory</td>
            <td>None</td>
            <td>1957</td>
            <td>None</td>
            <td>88</td>
            <td>8.4</td>
            <td>160459</td>
            <td>10050825</td>
            <td>drama</td>
        </tr>
        <tr>
            <td>10050825</td>
            <td>Paths of Glory</td>
            <td>None</td>
            <td>1957</td>
            <td>None</td>
            <td>88</td>
            <td>8.4</td>
            <td>160459</td>
            <td>10050825</td>
            <td>war</td>
        </tr>
        <tr>
            <td>10050986</td>
            <td>Wild Strawberries</td>
            <td>Smultronstället</td>
            <td>1957</td>
            <td>None</td>
            <td>91</td>
            <td>8.2</td>
            <td>85988</td>
            <td>10050986</td>
            <td>drama</td>
        </tr>
        <tr>
            <td>10050986</td>
            <td>Wild Strawberries</td>
            <td>Smultronstället</td>
            <td>1957</td>
            <td>None</td>
            <td>91</td>
            <td>8.2</td>
            <td>85988</td>
            <td>10050986</td>
            <td>romance</td>
        </tr>
        <tr>
            <td>10051036</td>
            <td>Sweet Smell of Success</td>
            <td>None</td>
            <td>1957</td>
            <td>None</td>
            <td>96</td>
            <td>8.1</td>
            <td>24897</td>
            <td>10051036</td>
            <td>drama</td>
        </tr>
        <tr>
            <td>10051036</td>
            <td>Sweet Smell of Success</td>
            <td>None</td>
            <td>1957</td>
            <td>None</td>
            <td>96</td>
            <td>8.1</td>
            <td>24897</td>
            <td>10051036</td>
            <td>film-noir</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">795 rows, truncated to displaylimit of 30</span>



Let's create a simple view for this query:


```sql
%%sql

CREATE VIEW
    simple_view
AS
    SELECT
        *
    FROM
        movies m
    JOIN
        movie_genres mg
    ON
        m.id = mg.movie_id
    WHERE        
        rating > 8
        AND
        nvotes > (
            SELECT
                AVG(m2.nvotes)
            FROM
                movies m2
            JOIN
                movie_genres mg2
            ON
                m2.id = mg2.movie_id
            WHERE
                mg.genre = mg2.genre
                AND
                m.id <> m2.id
        )
    ;
```

     * postgresql://postgres:***@localhost/imdb
    Done.
    




    []



Retrieving data from this view will be as slow as running the original query:


```python
%%timeit -r 1 -n 1
%%sql

SELECT * FROM simple_view;
```

     * postgresql://postgres:***@localhost/imdb
    795 rows affected.
    13.6 s ± 0 ns per loop (mean ± std. dev. of 1 run, 1 loop each)
    

<br><br><br>

The result of this query takes little space (only 795 rows), but it takes a long time to compute, which makes it a good candidate for a materialized view. Note that creating the materialized view will be slow the first time:


```python
%%timeit -r 1 -n 1
%%sql

CREATE MATERIALIZED VIEW
    mat_view
AS
    SELECT
        *
    FROM
        movies m
    JOIN
        movie_genres mg
    ON
        m.id = mg.movie_id
    WHERE        
        rating > 8
        AND
        nvotes > (
            SELECT
                AVG(m2.nvotes)
            FROM
                movies m2
            JOIN
                movie_genres mg2
            ON
                m2.id = mg2.movie_id
            WHERE
                mg.genre = mg2.genre
                AND
                m.id <> m2.id
        )
;
```

     * postgresql://postgres:***@localhost/imdb
    795 rows affected.
    13.4 s ± 0 ns per loop (mean ± std. dev. of 1 run, 1 loop each)
    

But retrieving data from the materialized view should be almost instantaneous:


```python
%%timeit -r 1 -n 1
%%sql

SELECT * FROM mat_view;
```

     * postgresql://postgres:***@localhost/imdb
    795 rows affected.
    3.55 ms ± 0 ns per loop (mean ± std. dev. of 1 run, 1 loop each)
    

Let's not forget to clean up:


```sql
%%sql

DROP VIEW simple_view;
DROP MATERIALIZED VIEW mat_view;
```

     * postgresql://postgres:***@localhost/imdb
    Done.
    Done.
    




    []



<br><br><br>

## Common Table Expressions (CTEs)

It's not uncommon for our queries to become overly complex. For example, consider the following question:

---

**Example:** Which countries speak at least one language that is not spoken an any other country in their continent?

---

A query to answer the above question involves joining tables first to have both country and language data in the same place, and then use a subquery.

To that end, we can create a temporary table first and then use the subquery on that table:

```sql
DROP TABLE IF EXISTS ccl;

CREATE TEMPORARY TABLE ccl AS (
    SELECT
        co.name, co.continent, cl.language
    FROM
        country co
    JOIN
        countrylanguage cl
    ON
        co.code = cl.countrycode
)
;

SELECT
    t1.*
FROM
    ccl t1
WHERE
    NOT EXISTS (
        SELECT
            *
        FROM
            ccl t2
        WHERE
            t1.name <> t2.name
            AND
            t1.continent = t2.continent
            AND
            t1.language = t2.language
    )
;
```

<br><br><br>

For this situation, we may not want to create a **temporary table** because:

- copying the data of two entire tables might create a disk space problem during a session

- if the data is modified in anyways before another subsequent query, it won't be reflected in the temporary table

- the temporary table has to be created each time a connection to the database is made

<br><br><br>

Creating a **view** looks like a nice alternative because it doesn't pose any of the above problems, but:

- The purpose of a view is to provide a more or less permanent way of accessing the data or hiding details. It doesn't make sense to create a view that is intended just for a very specific query and useless otherwise. For example, do we really want a view in our database that shows countries and languages just in Asia and Europe, or is it just for the purpose of a special query?

- Creating views requires special access privileges, which as a user we might not have!

<br><br><br>

Standard SQL has an elegant way of addressing this issue: **Common Table Expressions**

A common table expression is a temporary intermediate result set that we can use within a `SELECT`, `INSERT`, `UPDATE`, or `DELETE` statement. The syntax for a CTE is:

```sql
WITH
    expression_name [(column_names, ...)]
AS (
    query
)
query
;
```

<br><br><br>

Let's take a look at a few examples:


```python
%sql postgresql://{username}:{password}@{host}:{port}/world
```




    'Connected: postgres@world'



The following CTE creates an intermediate result set from the `country` and `countrylanguage` tables:


```sql
%%sql

WITH my_cte AS (
    SELECT
        name, population, language
    FROM
        country co
    JOIN
        countrylanguage cl
    ON
        cl.countrycode = co.code
)
SELECT * FROM my_cte
;
```

       postgresql://postgres:***@localhost/imdb
     * postgresql://postgres:***@localhost:5432/world
    984 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>population</th>
            <th>language</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Afghanistan</td>
            <td>22720000</td>
            <td>Pashto</td>
        </tr>
        <tr>
            <td>Netherlands</td>
            <td>15864000</td>
            <td>Dutch</td>
        </tr>
        <tr>
            <td>Netherlands Antilles</td>
            <td>217000</td>
            <td>Papiamento</td>
        </tr>
        <tr>
            <td>Albania</td>
            <td>3401200</td>
            <td>Albaniana</td>
        </tr>
        <tr>
            <td>Algeria</td>
            <td>31471000</td>
            <td>Arabic</td>
        </tr>
        <tr>
            <td>American Samoa</td>
            <td>68000</td>
            <td>Samoan</td>
        </tr>
        <tr>
            <td>Andorra</td>
            <td>78000</td>
            <td>Spanish</td>
        </tr>
        <tr>
            <td>Angola</td>
            <td>12878000</td>
            <td>Ovimbundu</td>
        </tr>
        <tr>
            <td>Anguilla</td>
            <td>8000</td>
            <td>English</td>
        </tr>
        <tr>
            <td>Antigua and Barbuda</td>
            <td>68000</td>
            <td>Creole English</td>
        </tr>
        <tr>
            <td>United Arab Emirates</td>
            <td>2441000</td>
            <td>Arabic</td>
        </tr>
        <tr>
            <td>Argentina</td>
            <td>37032000</td>
            <td>Spanish</td>
        </tr>
        <tr>
            <td>Armenia</td>
            <td>3520000</td>
            <td>Armenian</td>
        </tr>
        <tr>
            <td>Aruba</td>
            <td>103000</td>
            <td>Papiamento</td>
        </tr>
        <tr>
            <td>Australia</td>
            <td>18886000</td>
            <td>English</td>
        </tr>
        <tr>
            <td>Azerbaijan</td>
            <td>7734000</td>
            <td>Azerbaijani</td>
        </tr>
        <tr>
            <td>Bahamas</td>
            <td>307000</td>
            <td>Creole English</td>
        </tr>
        <tr>
            <td>Bahrain</td>
            <td>617000</td>
            <td>Arabic</td>
        </tr>
        <tr>
            <td>Bangladesh</td>
            <td>129155000</td>
            <td>Bengali</td>
        </tr>
        <tr>
            <td>Barbados</td>
            <td>270000</td>
            <td>Bajan</td>
        </tr>
        <tr>
            <td>Belgium</td>
            <td>10239000</td>
            <td>Dutch</td>
        </tr>
        <tr>
            <td>Belize</td>
            <td>241000</td>
            <td>English</td>
        </tr>
        <tr>
            <td>Benin</td>
            <td>6097000</td>
            <td>Fon</td>
        </tr>
        <tr>
            <td>Bermuda</td>
            <td>65000</td>
            <td>English</td>
        </tr>
        <tr>
            <td>Bhutan</td>
            <td>2124000</td>
            <td>Dzongkha</td>
        </tr>
        <tr>
            <td>Bolivia</td>
            <td>8329000</td>
            <td>Spanish</td>
        </tr>
        <tr>
            <td>Bosnia and Herzegovina</td>
            <td>3972000</td>
            <td>Serbo-Croatian</td>
        </tr>
        <tr>
            <td>Botswana</td>
            <td>1622000</td>
            <td>Tswana</td>
        </tr>
        <tr>
            <td>Brazil</td>
            <td>170115000</td>
            <td>Portuguese</td>
        </tr>
        <tr>
            <td>United Kingdom</td>
            <td>59623400</td>
            <td>English</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">984 rows, truncated to displaylimit of 30</span>



Let's say we want to count the number of official languages in each country. We can write this query as:


```sql
%%sql

WITH my_cte AS (
    SELECT
        co.name, cl.language
    FROM
        country co
    JOIN
        countrylanguage cl
    ON
        cl.countrycode = co.code
    WHERE
        cl.isofficial = TRUE
)
SELECT
    name, COUNT(language)
FROM
    my_cte
GROUP BY
    name
ORDER BY
    COUNT(language) DESC
;
```

       postgresql://postgres:***@localhost/imdb
     * postgresql://postgres:***@localhost:5432/world
    190 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>count</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Switzerland</td>
            <td>4</td>
        </tr>
        <tr>
            <td>South Africa</td>
            <td>4</td>
        </tr>
        <tr>
            <td>Singapore</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Peru</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Bolivia</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Vanuatu</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Belgium</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Luxembourg</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Seychelles</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Cyprus</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Netherlands Antilles</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Romania</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Kyrgyzstan</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Burundi</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Afghanistan</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Guam</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Togo</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Israel</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Faroe Islands</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Nauru</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Canada</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Palau</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Tonga</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Madagascar</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Greenland</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Tuvalu</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Samoa</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Malta</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Rwanda</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Sri Lanka</td>
            <td>2</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">190 rows, truncated to displaylimit of 30</span>



We can also rewrite this query from the last lecture using a CTE:

**Question:** Which countries speak at least one language that is not spoken in any other country on their continent?


```sql
%%sql

WITH ccl AS (
    SELECT
        co.name, co.continent, cl.language
    FROM
        country co
    JOIN
        countrylanguage cl
    ON
        co.code = cl.countrycode
)
SELECT
    t1.*
FROM
    ccl t1
WHERE
    NOT EXISTS (
        SELECT
            *
        FROM
            ccl t2
        WHERE
            t1.name <> t2.name
            AND
            t1.continent = t2.continent
            AND
            t1.language = t2.language
    )
```

       postgresql://postgres:***@localhost/imdb
     * postgresql://postgres:***@localhost:5432/world
    362 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>continent</th>
            <th>language</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Angola</td>
            <td>Africa</td>
            <td>Ovimbundu</td>
        </tr>
        <tr>
            <td>Barbados</td>
            <td>North America</td>
            <td>Bajan</td>
        </tr>
        <tr>
            <td>Benin</td>
            <td>Africa</td>
            <td>Fon</td>
        </tr>
        <tr>
            <td>Bhutan</td>
            <td>Asia</td>
            <td>Dzongkha</td>
        </tr>
        <tr>
            <td>Burundi</td>
            <td>Africa</td>
            <td>Kirundi</td>
        </tr>
        <tr>
            <td>Ethiopia</td>
            <td>Africa</td>
            <td>Oromo</td>
        </tr>
        <tr>
            <td>Falkland Islands</td>
            <td>South America</td>
            <td>English</td>
        </tr>
        <tr>
            <td>Fiji Islands</td>
            <td>Oceania</td>
            <td>Fijian</td>
        </tr>
        <tr>
            <td>Philippines</td>
            <td>Asia</td>
            <td>Pilipino</td>
        </tr>
        <tr>
            <td>Faroe Islands</td>
            <td>Europe</td>
            <td>Faroese</td>
        </tr>
        <tr>
            <td>Georgia</td>
            <td>Asia</td>
            <td>Georgiana</td>
        </tr>
        <tr>
            <td>Greenland</td>
            <td>North America</td>
            <td>Greenlandic</td>
        </tr>
        <tr>
            <td>Haiti</td>
            <td>North America</td>
            <td>Haiti Creole</td>
        </tr>
        <tr>
            <td>Indonesia</td>
            <td>Asia</td>
            <td>Javanese</td>
        </tr>
        <tr>
            <td>Iceland</td>
            <td>Europe</td>
            <td>Icelandic</td>
        </tr>
        <tr>
            <td>Japan</td>
            <td>Asia</td>
            <td>Japanese</td>
        </tr>
        <tr>
            <td>Kenya</td>
            <td>Africa</td>
            <td>Kikuyu</td>
        </tr>
        <tr>
            <td>Central African Republic</td>
            <td>Africa</td>
            <td>Gbaya</td>
        </tr>
        <tr>
            <td>Kyrgyzstan</td>
            <td>Asia</td>
            <td>Kirgiz</td>
        </tr>
        <tr>
            <td>Congo, The Democratic Republic of the</td>
            <td>Africa</td>
            <td>Luba</td>
        </tr>
        <tr>
            <td>Cocos (Keeling) Islands</td>
            <td>Oceania</td>
            <td>Malay</td>
        </tr>
        <tr>
            <td>Cyprus</td>
            <td>Asia</td>
            <td>Greek</td>
        </tr>
        <tr>
            <td>Latvia</td>
            <td>Europe</td>
            <td>Latvian</td>
        </tr>
        <tr>
            <td>Lesotho</td>
            <td>Africa</td>
            <td>Sotho</td>
        </tr>
        <tr>
            <td>Luxembourg</td>
            <td>Europe</td>
            <td>Luxembourgish</td>
        </tr>
        <tr>
            <td>Malawi</td>
            <td>Africa</td>
            <td>Chichewa</td>
        </tr>
        <tr>
            <td>Maldives</td>
            <td>Asia</td>
            <td>Dhivehi</td>
        </tr>
        <tr>
            <td>Mali</td>
            <td>Africa</td>
            <td>Bambara</td>
        </tr>
        <tr>
            <td>Malta</td>
            <td>Europe</td>
            <td>Maltese</td>
        </tr>
        <tr>
            <td>Marshall Islands</td>
            <td>Oceania</td>
            <td>Marshallese</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">362 rows, truncated to displaylimit of 30</span>



<br><br><br>

---

**Question:** Can you think of two advantages of a temporary table over CTEs?

---

<br><br><br>

Besides simplifying long and complex queries and increase readability, we can do things with CTEs that are impossible otherwise. One example is with grouping and aggregation.

Remember that when aggregate functions are used, only columns can be in the `SELECT` clause that are either aggregated (used inside an aggregate function) or appear also in the `GROUP BY` clause. Now, consider the following example:

<br><br><br>

---

**Example:** List city names, their country and region where they are located. Also in each row (i.e. for each city)

- list the number of official languages and
- number of cities with population over 1,000,000

in their respective countries.

---

We can obviously join the three tables `country`, `city`, and `countrylanguage` and do grouping, but we won't be able to list city names for the reason just discussed. But we can actually do this using a CTE:


```sql
%%sql

WITH
    ccl (code, country, region, "num_cities_over_1_mil", "num_off_lang")
AS (
    SELECT
        co.code, co.name, co.region,
        COUNT(DISTINCT ci.name),
        COUNT(DISTINCT cl.language)
    FROM
        country co
    JOIN
        city ci
    ON
        co.code = ci.countrycode
    JOIN
        countrylanguage cl
    ON
        co.code = cl.countrycode
    WHERE
        ci.population > 1000000
    GROUP BY
        co.code, co.name, co.region
)
SELECT
    ci.name, ccl.*
FROM
    ccl
JOIN
    city ci
ON
    ccl.code = ci.countrycode
;
```

       postgresql://postgres:***@localhost/imdb
     * postgresql://postgres:***@localhost:5432/world
    3671 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>code</th>
            <th>country</th>
            <th>region</th>
            <th>num_cities_over_1_mil</th>
            <th>num_off_lang</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Kabul</td>
            <td>AFG</td>
            <td>Afghanistan</td>
            <td>Southern and Central Asia</td>
            <td>1</td>
            <td>5</td>
        </tr>
        <tr>
            <td>Qandahar</td>
            <td>AFG</td>
            <td>Afghanistan</td>
            <td>Southern and Central Asia</td>
            <td>1</td>
            <td>5</td>
        </tr>
        <tr>
            <td>Herat</td>
            <td>AFG</td>
            <td>Afghanistan</td>
            <td>Southern and Central Asia</td>
            <td>1</td>
            <td>5</td>
        </tr>
        <tr>
            <td>Mazar-e-Sharif</td>
            <td>AFG</td>
            <td>Afghanistan</td>
            <td>Southern and Central Asia</td>
            <td>1</td>
            <td>5</td>
        </tr>
        <tr>
            <td>Alger</td>
            <td>DZA</td>
            <td>Algeria</td>
            <td>Northern Africa</td>
            <td>1</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Oran</td>
            <td>DZA</td>
            <td>Algeria</td>
            <td>Northern Africa</td>
            <td>1</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Constantine</td>
            <td>DZA</td>
            <td>Algeria</td>
            <td>Northern Africa</td>
            <td>1</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Annaba</td>
            <td>DZA</td>
            <td>Algeria</td>
            <td>Northern Africa</td>
            <td>1</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Batna</td>
            <td>DZA</td>
            <td>Algeria</td>
            <td>Northern Africa</td>
            <td>1</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Sétif</td>
            <td>DZA</td>
            <td>Algeria</td>
            <td>Northern Africa</td>
            <td>1</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Sidi Bel Abbès</td>
            <td>DZA</td>
            <td>Algeria</td>
            <td>Northern Africa</td>
            <td>1</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Skikda</td>
            <td>DZA</td>
            <td>Algeria</td>
            <td>Northern Africa</td>
            <td>1</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Biskra</td>
            <td>DZA</td>
            <td>Algeria</td>
            <td>Northern Africa</td>
            <td>1</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Blida (el-Boulaida)</td>
            <td>DZA</td>
            <td>Algeria</td>
            <td>Northern Africa</td>
            <td>1</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Béjaïa</td>
            <td>DZA</td>
            <td>Algeria</td>
            <td>Northern Africa</td>
            <td>1</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Mostaganem</td>
            <td>DZA</td>
            <td>Algeria</td>
            <td>Northern Africa</td>
            <td>1</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Tébessa</td>
            <td>DZA</td>
            <td>Algeria</td>
            <td>Northern Africa</td>
            <td>1</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Tlemcen (Tilimsen)</td>
            <td>DZA</td>
            <td>Algeria</td>
            <td>Northern Africa</td>
            <td>1</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Béchar</td>
            <td>DZA</td>
            <td>Algeria</td>
            <td>Northern Africa</td>
            <td>1</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Tiaret</td>
            <td>DZA</td>
            <td>Algeria</td>
            <td>Northern Africa</td>
            <td>1</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Ech-Chleff (el-Asnam)</td>
            <td>DZA</td>
            <td>Algeria</td>
            <td>Northern Africa</td>
            <td>1</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Ghardaïa</td>
            <td>DZA</td>
            <td>Algeria</td>
            <td>Northern Africa</td>
            <td>1</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Luanda</td>
            <td>AGO</td>
            <td>Angola</td>
            <td>Central Africa</td>
            <td>1</td>
            <td>9</td>
        </tr>
        <tr>
            <td>Huambo</td>
            <td>AGO</td>
            <td>Angola</td>
            <td>Central Africa</td>
            <td>1</td>
            <td>9</td>
        </tr>
        <tr>
            <td>Lobito</td>
            <td>AGO</td>
            <td>Angola</td>
            <td>Central Africa</td>
            <td>1</td>
            <td>9</td>
        </tr>
        <tr>
            <td>Benguela</td>
            <td>AGO</td>
            <td>Angola</td>
            <td>Central Africa</td>
            <td>1</td>
            <td>9</td>
        </tr>
        <tr>
            <td>Namibe</td>
            <td>AGO</td>
            <td>Angola</td>
            <td>Central Africa</td>
            <td>1</td>
            <td>9</td>
        </tr>
        <tr>
            <td>Buenos Aires</td>
            <td>ARG</td>
            <td>Argentina</td>
            <td>South America</td>
            <td>3</td>
            <td>3</td>
        </tr>
        <tr>
            <td>La Matanza</td>
            <td>ARG</td>
            <td>Argentina</td>
            <td>South America</td>
            <td>3</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Córdoba</td>
            <td>ARG</td>
            <td>Argentina</td>
            <td>South America</td>
            <td>3</td>
            <td>3</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">3671 rows, truncated to displaylimit of 30</span>



<br><br><br>

### (OPTIONAL) Data-modifying CTEs

CTEs can also be used with `INSERT`, `DELETE`, and `UPDATE` to catch the rows modified by these statements via the `RETURNING` statement and use them for another purpose.

Let's connect to the `mds` database we created earlier and reload it with the original data:


```python
%sql postgresql://{username}:{password}@{host}:{port}/mds
```




    'Connected: postgres@mds'




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
    (9, 'MDS-V'),
    (3, 'MDS-V')
;
```

       postgresql://postgres:***@localhost/imdb
     * postgresql://postgres:***@localhost:5432/mds
       postgresql://postgres:***@localhost:5432/world
    Done.
    Done.
    11 rows affected.
    Done.
    16 rows affected.
    Done.
    8 rows affected.
    




    []




```python
%sql SELECT * FROM instructor;
```

       postgresql://postgres:***@localhost/imdb
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




```sql
%%sql

DROP TABLE IF EXISTS former_instructor;

CREATE TABLE
    former_instructor AS (
    SELECT
        *
    FROM
        instructor
    WHERE
        1 = 2
)
;
```

       postgresql://postgres:***@localhost/imdb
     * postgresql://postgres:***@localhost:5432/mds
       postgresql://postgres:***@localhost:5432/world
    Done.
    0 rows affected.
    




    []




```python
%sql SELECT * FROM former_instructor;
```

       postgresql://postgres:***@localhost/imdb
     * postgresql://postgres:***@localhost:5432/mds
       postgresql://postgres:***@localhost:5432/world
    0 rows affected.
    




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
    </tbody>
</table>



Here I use the `RETURNING` statement to retrieve the results of deleting two rows using a CTE, and insert the deleted rows into another table, namely, `former_instructor`:


```sql
%%sql

WITH deleted_rows AS (
    DELETE FROM
        instructor
    WHERE
        name IN ('Arman', 'Mike')
    RETURNING
        *
)
INSERT INTO
    former_instructor
SELECT
    *
FROM
    deleted_rows
;
```

       postgresql://postgres:***@localhost/imdb
     * postgresql://postgres:***@localhost:5432/mds
       postgresql://postgres:***@localhost:5432/world
    2 rows affected.
    




    []



Let's check out the `former_instructor` table now:


```python
%sql SELECT * FROM former_instructor;
```

       postgresql://postgres:***@localhost/imdb
     * postgresql://postgres:***@localhost:5432/mds
       postgresql://postgres:***@localhost:5432/world
    2 rows affected.
    




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
            <td>3</td>
            <td>Arman</td>
            <td>arman@mds.ubc.ca</td>
            <td>935-738-5796</td>
            <td>Physics</td>
        </tr>
    </tbody>
</table>



The two deleted rows have been successfully moved to the `former_instructor` table!

<br><br><br>

## Window functions

We've been using aggregation and grouping quite a lot so far. But as you may recall, there are situations that aggregation and grouping alone cannot address, such as having aggregated and non-aggregated columns together.

We know that aggregation functions perform calculations on groups of rows, and return a single value for all rows that take part in that calculation. They also collapse the participant rows into one single row with the the common column(s) values between all the collapsed rows and the aggregated value. For example, in the `instructor` table of the `mds` database, we can group rows by `department`, and apply `COUNT()` to find the number of instructors in each department, but we no longer see individual instructor rows. Instead, we will see a `COUNT()` number for each `department`.

Window functions act in a similar fashion: they perform operations on groups of rows that are related in some way to the current row (like `GROUP BY` that bases the relatedness on common values in grouped columns), but these functions **do not collapse rows**! This is the most important difference between aggregation and window functions

Let's take a look at an example first. The following query finds the maximum population of countries in each continent:


```python
%sql postgresql://{username}:{password}@{host}:{port}/world
```




    'Connected: postgres@world'




```sql
%%sql

SELECT
    continent,
    MAX(population)
FROM
    country
GROUP BY
    continent
;
```

       postgresql://postgres:***@localhost/imdb
       postgresql://postgres:***@localhost:5432/mds
     * postgresql://postgres:***@localhost:5432/world
    7 rows affected.
    




<table>
    <thead>
        <tr>
            <th>continent</th>
            <th>max</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Asia</td>
            <td>1277558000</td>
        </tr>
        <tr>
            <td>South America</td>
            <td>170115000</td>
        </tr>
        <tr>
            <td>North America</td>
            <td>278357000</td>
        </tr>
        <tr>
            <td>Oceania</td>
            <td>18886000</td>
        </tr>
        <tr>
            <td>Antarctica</td>
            <td>0</td>
        </tr>
        <tr>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Europe</td>
            <td>146934000</td>
        </tr>
    </tbody>
</table>



<br><br><br>

However, what if we want each country to be listed along with its own population, as well as the maximum population in their respective continent? This is not possible with aggregation functions alone.

In order to list country names, their population, and the maximum population in their continent we can use `MAX()` as a window function:


```sql
%%sql

SELECT
    name,
    population,
    continent,
    MAX(population)
        OVER (PARTITION BY continent)
FROM
    country
;
```

       postgresql://postgres:***@localhost/imdb
       postgresql://postgres:***@localhost:5432/mds
     * postgresql://postgres:***@localhost:5432/world
    239 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>population</th>
            <th>continent</th>
            <th>max</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Algeria</td>
            <td>31471000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Western Sahara</td>
            <td>293000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Madagascar</td>
            <td>15942000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Uganda</td>
            <td>21778000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Malawi</td>
            <td>10925000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Mali</td>
            <td>11234000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Morocco</td>
            <td>28351000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Côte dIvoire</td>
            <td>14786000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Mauritania</td>
            <td>2670000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Mauritius</td>
            <td>1158000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Mayotte</td>
            <td>149000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Nigeria</td>
            <td>111506000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Niger</td>
            <td>10730000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Mozambique</td>
            <td>19680000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Namibia</td>
            <td>1726000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Botswana</td>
            <td>1622000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Tunisia</td>
            <td>9586000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Angola</td>
            <td>12878000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Burkina Faso</td>
            <td>11937000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Burundi</td>
            <td>6695000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Chad</td>
            <td>7651000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Togo</td>
            <td>4629000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Tanzania</td>
            <td>33517000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Djibouti</td>
            <td>638000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Egypt</td>
            <td>68470000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Swaziland</td>
            <td>1008000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Eritrea</td>
            <td>3850000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>South Africa</td>
            <td>40377000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Ethiopia</td>
            <td>62565000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Sudan</td>
            <td>29490000</td>
            <td>Africa</td>
            <td>111506000</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">239 rows, truncated to displaylimit of 30</span>



<br><br><br>

### Syntax

A lot of new keywords appeared all at once here, so let me explain them all in detail. The syntax of a window function is:
‍‍‍
```sql
SELECT
    column,
    .
    .
    .
    window_function_name(expression) OVER (
        PARTITION BY column
        ORDER BY column
        frame_clause
    .
    .
    .
FROM
    table
;
```

- `window_function_name()` specifies which window function we want to use. I'll list the different types of window functions in a bit.

- `OVER` is what tells the SQL engine that we are going to use a window function

- `PARTITION BY` determines how or based on which column we want the window function to subdivide the rows into groups

- `ORDER BY` specifies how we want the rows in each subgroup to be ordered

- `frame_clause` gives us further control over which rows in each subgroup can participate in the window function calculations

<br><br><br>

> **Note:** Window functions in SQL are processed **after** `GROUP BY` and `HAVING`, and **before** `SELECT`. This is a crucial point to understand when using window functions in SQL.

> **Note:** Window functions are only allowed in the `SELECT` and `ORDER BY` clauses.

<br><br><br>

### Types of window functions

<br><br><br>

#### Aggregate functions

These are the regular aggregate functions that we've learned about before, i.e. `AVG()`, `COUNT()`, `MAX()`, `MIN()`, and `SUM()`, which can also be used as window functions.

<br><br><br>

#### Ranking functions

- `CUME_DIST()`: returns the cumulative distribution, i.e. the percentage of values less than or equal to the current value.
- `NTILE()`: given a specified number of buckets, it tells us in which bucket each row goes among other rows in the partition.
- `PERCENT_RANK()`: similar to `CUME_DIST()`, but considers only the percentage of values less than (and not equal to) the current value.

- `DENSE_RANK()`: returns the rank of a row within a partition without jumps after duplicate ranks (e.g. 1, 2, 2, 3, ...)
- `RANK()`: returns the rank of row within a partition but with jumps after duplicates ranks (e.g. 1, 2, 2, 4, ...)
- `ROW_NUMBER()`: returns simply the number of a row in a partition, regardless of duplicate values

<br><br><br>

#### Value functions

- `FIRST_VALUE()`: returns the first value of the ordered partition

- `LAST_VALUE()`: returns the last value of the ordered partition
- `LAG()`: returns value from the preceding row with a specified offset
- `LEAD()`: returns value from the following row with a specified offset

<br><br><br>

**Differences with subqueries**

- Subqueries can operate on multiple tables, window functions are limited to the current table

- Subqueries are far more general (e.g. `WHERE` conditions including `EXITS`, etc.)
- Window functions are more readable
- Window functions offer functionalities such as `RANK()` or `LAG()` and `LEAD()` that are either very hard, if not impossible, to implement using subqueries
- Window functions are generally faster than subqueries when used for a same purpose

<br><br><br>

---

**Example:** Using window functions, count the number of countries in which each language is officially spoken. Remove duplicates and sort the results in descending order by the count value.

---


```sql
%%sql

SELECT
    DISTINCT language,
    COUNT(*) OVER (PARTITION BY language) AS count
FROM
    countrylanguage
WHERE
    isofficial = TRUE
ORDER BY
    count DESC
```

       postgresql://postgres:***@localhost/imdb
       postgresql://postgres:***@localhost:5432/mds
     * postgresql://postgres:***@localhost:5432/world
    102 rows affected.
    




<table>
    <thead>
        <tr>
            <th>language</th>
            <th>count</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>English</td>
            <td>44</td>
        </tr>
        <tr>
            <td>Arabic</td>
            <td>22</td>
        </tr>
        <tr>
            <td>Spanish</td>
            <td>20</td>
        </tr>
        <tr>
            <td>French</td>
            <td>18</td>
        </tr>
        <tr>
            <td>German</td>
            <td>6</td>
        </tr>
        <tr>
            <td>Portuguese</td>
            <td>6</td>
        </tr>
        <tr>
            <td>Dutch</td>
            <td>4</td>
        </tr>
        <tr>
            <td>Italian</td>
            <td>4</td>
        </tr>
        <tr>
            <td>Malay</td>
            <td>4</td>
        </tr>
        <tr>
            <td>Danish</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Russian</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Serbo-Croatian</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Aimará</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Chinese</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Greek</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Ketua</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Korean</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Norwegian</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Romanian</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Samoan</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Swedish</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Tamil</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Turkish</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Afrikaans</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Albaniana</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Armenian</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Azerbaijani</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Belorussian</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Bengali</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Bislama</td>
            <td>1</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">102 rows, truncated to displaylimit of 30</span>



<br><br><br>

**Question:** In the above query, is it possible to use a `WHERE` clause to limit the results to languages which are spoken in more than one country?

<br><br><br>

In the following example, we rank countries by life expectancy in each continent, but we also create a column to retrieve the name of the country whose ranking follows the one in the current row:


```sql
%%sql

SELECT
    name,
    continent,
    DENSE_RANK()
        OVER (PARTITION BY continent ORDER BY lifeexpectancy DESC),
    LEAD(name, 1)
        OVER (PARTITION BY continent ORDER BY lifeexpectancy DESC)
FROM
    country
WHERE
    population > 10000000
```

       postgresql://postgres:***@localhost/imdb
       postgresql://postgres:***@localhost:5432/mds
     * postgresql://postgres:***@localhost:5432/world
    78 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>continent</th>
            <th>dense_rank</th>
            <th>lead</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Algeria</td>
            <td>Africa</td>
            <td>1</td>
            <td>Morocco</td>
        </tr>
        <tr>
            <td>Morocco</td>
            <td>Africa</td>
            <td>2</td>
            <td>Egypt</td>
        </tr>
        <tr>
            <td>Egypt</td>
            <td>Africa</td>
            <td>3</td>
            <td>Ghana</td>
        </tr>
        <tr>
            <td>Ghana</td>
            <td>Africa</td>
            <td>4</td>
            <td>Sudan</td>
        </tr>
        <tr>
            <td>Sudan</td>
            <td>Africa</td>
            <td>5</td>
            <td>Madagascar</td>
        </tr>
        <tr>
            <td>Madagascar</td>
            <td>Africa</td>
            <td>6</td>
            <td>Cameroon</td>
        </tr>
        <tr>
            <td>Cameroon</td>
            <td>Africa</td>
            <td>7</td>
            <td>Tanzania</td>
        </tr>
        <tr>
            <td>Tanzania</td>
            <td>Africa</td>
            <td>8</td>
            <td>Nigeria</td>
        </tr>
        <tr>
            <td>Nigeria</td>
            <td>Africa</td>
            <td>9</td>
            <td>South Africa</td>
        </tr>
        <tr>
            <td>South Africa</td>
            <td>Africa</td>
            <td>10</td>
            <td>Congo, The Democratic Republic of the</td>
        </tr>
        <tr>
            <td>Congo, The Democratic Republic of the</td>
            <td>Africa</td>
            <td>11</td>
            <td>Kenya</td>
        </tr>
        <tr>
            <td>Kenya</td>
            <td>Africa</td>
            <td>12</td>
            <td>Burkina Faso</td>
        </tr>
        <tr>
            <td>Burkina Faso</td>
            <td>Africa</td>
            <td>13</td>
            <td>Mali</td>
        </tr>
        <tr>
            <td>Mali</td>
            <td>Africa</td>
            <td>13</td>
            <td>Somalia</td>
        </tr>
        <tr>
            <td>Somalia</td>
            <td>Africa</td>
            <td>14</td>
            <td>Ethiopia</td>
        </tr>
        <tr>
            <td>Ethiopia</td>
            <td>Africa</td>
            <td>15</td>
            <td>Côte dIvoire</td>
        </tr>
        <tr>
            <td>Côte dIvoire</td>
            <td>Africa</td>
            <td>15</td>
            <td>Uganda</td>
        </tr>
        <tr>
            <td>Uganda</td>
            <td>Africa</td>
            <td>16</td>
            <td>Niger</td>
        </tr>
        <tr>
            <td>Niger</td>
            <td>Africa</td>
            <td>17</td>
            <td>Angola</td>
        </tr>
        <tr>
            <td>Angola</td>
            <td>Africa</td>
            <td>18</td>
            <td>Zimbabwe</td>
        </tr>
        <tr>
            <td>Zimbabwe</td>
            <td>Africa</td>
            <td>19</td>
            <td>Malawi</td>
        </tr>
        <tr>
            <td>Malawi</td>
            <td>Africa</td>
            <td>20</td>
            <td>Mozambique</td>
        </tr>
        <tr>
            <td>Mozambique</td>
            <td>Africa</td>
            <td>21</td>
            <td>None</td>
        </tr>
        <tr>
            <td>Japan</td>
            <td>Asia</td>
            <td>1</td>
            <td>Taiwan</td>
        </tr>
        <tr>
            <td>Taiwan</td>
            <td>Asia</td>
            <td>2</td>
            <td>South Korea</td>
        </tr>
        <tr>
            <td>South Korea</td>
            <td>Asia</td>
            <td>3</td>
            <td>Sri Lanka</td>
        </tr>
        <tr>
            <td>Sri Lanka</td>
            <td>Asia</td>
            <td>4</td>
            <td>China</td>
        </tr>
        <tr>
            <td>China</td>
            <td>Asia</td>
            <td>5</td>
            <td>Turkey</td>
        </tr>
        <tr>
            <td>Turkey</td>
            <td>Asia</td>
            <td>6</td>
            <td>Malaysia</td>
        </tr>
        <tr>
            <td>Malaysia</td>
            <td>Asia</td>
            <td>7</td>
            <td>North Korea</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">78 rows, truncated to displaylimit of 30</span>



<br><br><br>


```sql
%%sql

WITH cte(name, continent, rank, lead_rank) AS
(
SELECT
    name,
    continent,
    DENSE_RANK()
        OVER (PARTITION BY continent ORDER BY lifeexpectancy DESC),
    LEAD(name, 1)
        OVER (PARTITION BY continent ORDER BY lifeexpectancy DESC)
FROM
    country
WHERE
    population > 10000000
)
SELECT
    *
FROM
    cte
WHERE
    rank = 1
```

       postgresql://postgres:***@localhost/imdb
       postgresql://postgres:***@localhost:5432/mds
     * postgresql://postgres:***@localhost:5432/world
    6 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>continent</th>
            <th>rank</th>
            <th>lead_rank</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Algeria</td>
            <td>Africa</td>
            <td>1</td>
            <td>Morocco</td>
        </tr>
        <tr>
            <td>Japan</td>
            <td>Asia</td>
            <td>1</td>
            <td>Taiwan</td>
        </tr>
        <tr>
            <td>Italy</td>
            <td>Europe</td>
            <td>1</td>
            <td>France</td>
        </tr>
        <tr>
            <td>Canada</td>
            <td>North America</td>
            <td>1</td>
            <td>United States</td>
        </tr>
        <tr>
            <td>Australia</td>
            <td>Oceania</td>
            <td>1</td>
            <td>None</td>
        </tr>
        <tr>
            <td>Chile</td>
            <td>South America</td>
            <td>1</td>
            <td>Argentina</td>
        </tr>
    </tbody>
</table>



<br><br><br>

---

**Example:** Write a query that in each row returns each movie's name, production year (`start_year`), and its number of votes, for all movies produced in or after 2018. Your query should also return a column that ranks each movie in descending order, according to its number of votes in the year it was produced.

- Sort your results according the rank value in ascending order

- Retrieve only 20 rows

---


```python
%sql postgresql://{username}:{password}@{host}:{port}/imdb
```




    'Connected: postgres@imdb'




```sql
%%sql

SELECT
    title,
    start_year,
    nvotes,
    RANK()
        OVER (PARTITION BY start_year ORDER BY nvotes DESC)
            AS rank
FROM
    movies
WHERE
    start_year >= 2018
ORDER BY
    rank
LIMIT
    20
;
```

       postgresql://postgres:***@localhost/imdb
     * postgresql://postgres:***@localhost:5432/imdb
       postgresql://postgres:***@localhost:5432/mds
       postgresql://postgres:***@localhost:5432/world
    20 rows affected.
    




<table>
    <thead>
        <tr>
            <th>title</th>
            <th>start_year</th>
            <th>nvotes</th>
            <th>rank</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Avengers: Infinity War</td>
            <td>2018</td>
            <td>711500</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Avengers: Endgame</td>
            <td>2019</td>
            <td>567968</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Black Panther</td>
            <td>2018</td>
            <td>543002</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Captain Marvel</td>
            <td>2019</td>
            <td>362609</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Once Upon a Time... in Hollywood</td>
            <td>2019</td>
            <td>204266</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Deadpool 2</td>
            <td>2018</td>
            <td>416811</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Bohemian Rhapsody</td>
            <td>2018</td>
            <td>382622</td>
            <td>4</td>
        </tr>
        <tr>
            <td>Spider-Man: Far from Home</td>
            <td>2019</td>
            <td>198519</td>
            <td>4</td>
        </tr>
        <tr>
            <td>A Quiet Place</td>
            <td>2018</td>
            <td>325428</td>
            <td>5</td>
        </tr>
        <tr>
            <td>Shazam!</td>
            <td>2019</td>
            <td>182200</td>
            <td>5</td>
        </tr>
        <tr>
            <td>John Wick: Chapter 3 - Parabellum</td>
            <td>2019</td>
            <td>172900</td>
            <td>6</td>
        </tr>
        <tr>
            <td>Ready Player One</td>
            <td>2018</td>
            <td>316326</td>
            <td>6</td>
        </tr>
        <tr>
            <td>Alita: Battle Angel</td>
            <td>2019</td>
            <td>162513</td>
            <td>7</td>
        </tr>
        <tr>
            <td>Venom</td>
            <td>2018</td>
            <td>305111</td>
            <td>7</td>
        </tr>
        <tr>
            <td>Glass</td>
            <td>2019</td>
            <td>159064</td>
            <td>8</td>
        </tr>
        <tr>
            <td>Aquaman</td>
            <td>2018</td>
            <td>300064</td>
            <td>8</td>
        </tr>
        <tr>
            <td>Us</td>
            <td>2019</td>
            <td>150951</td>
            <td>9</td>
        </tr>
        <tr>
            <td>A Star Is Born</td>
            <td>2018</td>
            <td>276484</td>
            <td>9</td>
        </tr>
        <tr>
            <td>Spider-Man: Into the Spider-Verse</td>
            <td>2018</td>
            <td>262505</td>
            <td>10</td>
        </tr>
        <tr>
            <td>Aladdin</td>
            <td>2019</td>
            <td>139370</td>
            <td>10</td>
        </tr>
    </tbody>
</table>



<br><br><br>

---

**Example:** We want to find out how movies are ranked in their respective genres. Write a query that in each row returns movie name, genre, and production year (`start_year`) for movies with more than 1,000,000 votes. Your query should also have a column that shows the ranking of each movie according to its rating in descending order.

- Sort your results by genre and rank in ascending order
- Limit your results to 50 rows

---


```sql
%%sql

SELECT
    title,
    genre,
    start_year,
    RANK()
        OVER (PARTITION BY genre ORDER BY rating)
            AS rank
FROM
    movies m
JOIN
    movie_genres mg
ON
    m.id = mg.movie_id
WHERE
    nvotes > 1000000
ORDER BY
    genre, rank
LIMIT
    50
;
```

       postgresql://postgres:***@localhost/imdb
     * postgresql://postgres:***@localhost:5432/imdb
       postgresql://postgres:***@localhost:5432/mds
       postgresql://postgres:***@localhost:5432/world
    50 rows affected.
    




<table>
    <thead>
        <tr>
            <th>title</th>
            <th>genre</th>
            <th>start_year</th>
            <th>rank</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Avatar</td>
            <td>action</td>
            <td>2009</td>
            <td>1</td>
        </tr>
        <tr>
            <td>The Avengers</td>
            <td>action</td>
            <td>2012</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Batman Begins</td>
            <td>action</td>
            <td>2005</td>
            <td>3</td>
        </tr>
        <tr>
            <td>The Dark Knight Rises</td>
            <td>action</td>
            <td>2012</td>
            <td>4</td>
        </tr>
        <tr>
            <td>Gladiator</td>
            <td>action</td>
            <td>2000</td>
            <td>5</td>
        </tr>
        <tr>
            <td>Star Wars: Episode IV - A New Hope</td>
            <td>action</td>
            <td>1977</td>
            <td>6</td>
        </tr>
        <tr>
            <td>The Matrix</td>
            <td>action</td>
            <td>1999</td>
            <td>7</td>
        </tr>
        <tr>
            <td>Star Wars: Episode V - The Empire Strikes Back</td>
            <td>action</td>
            <td>1980</td>
            <td>7</td>
        </tr>
        <tr>
            <td>Inception</td>
            <td>action</td>
            <td>2010</td>
            <td>9</td>
        </tr>
        <tr>
            <td>The Dark Knight</td>
            <td>action</td>
            <td>2008</td>
            <td>10</td>
        </tr>
        <tr>
            <td>Avatar</td>
            <td>adventure</td>
            <td>2009</td>
            <td>1</td>
        </tr>
        <tr>
            <td>The Avengers</td>
            <td>adventure</td>
            <td>2012</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Batman Begins</td>
            <td>adventure</td>
            <td>2005</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Inglourious Basterds</td>
            <td>adventure</td>
            <td>2009</td>
            <td>4</td>
        </tr>
        <tr>
            <td>Gladiator</td>
            <td>adventure</td>
            <td>2000</td>
            <td>5</td>
        </tr>
        <tr>
            <td>Interstellar</td>
            <td>adventure</td>
            <td>2014</td>
            <td>6</td>
        </tr>
        <tr>
            <td>Star Wars: Episode IV - A New Hope</td>
            <td>adventure</td>
            <td>1977</td>
            <td>6</td>
        </tr>
        <tr>
            <td>Star Wars: Episode V - The Empire Strikes Back</td>
            <td>adventure</td>
            <td>1980</td>
            <td>8</td>
        </tr>
        <tr>
            <td>The Lord of the Rings: The Two Towers</td>
            <td>adventure</td>
            <td>2002</td>
            <td>8</td>
        </tr>
        <tr>
            <td>The Lord of the Rings: The Fellowship of the Ring</td>
            <td>adventure</td>
            <td>2001</td>
            <td>10</td>
        </tr>
        <tr>
            <td>Inception</td>
            <td>adventure</td>
            <td>2010</td>
            <td>10</td>
        </tr>
        <tr>
            <td>The Lord of the Rings: The Return of the King</td>
            <td>adventure</td>
            <td>2003</td>
            <td>12</td>
        </tr>
        <tr>
            <td>The Wolf of Wall Street</td>
            <td>biography</td>
            <td>2013</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Schindler&#x27;s List</td>
            <td>biography</td>
            <td>1993</td>
            <td>2</td>
        </tr>
        <tr>
            <td>The Wolf of Wall Street</td>
            <td>crime</td>
            <td>2013</td>
            <td>1</td>
        </tr>
        <tr>
            <td>The Departed</td>
            <td>crime</td>
            <td>2006</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Se7en</td>
            <td>crime</td>
            <td>1995</td>
            <td>3</td>
        </tr>
        <tr>
            <td>The Green Mile</td>
            <td>crime</td>
            <td>1999</td>
            <td>3</td>
        </tr>
        <tr>
            <td>The Silence of the Lambs</td>
            <td>crime</td>
            <td>1991</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Pulp Fiction</td>
            <td>crime</td>
            <td>1994</td>
            <td>6</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">50 rows, truncated to displaylimit of 30</span>



<br><br><br>

It is also possible to define multiple windows over which multiple window functions may operate:


```python
%sql postgresql://{username}:{password}@{host}:{port}/world
```




    'Connected: postgres@world'




```sql
%%sql

SELECT
    name,
    RANK() OVER w1 AS region_rank,
    RANK() OVER w2 AS continent_rank
FROM
    country
WHERE
    population > 1000000
WINDOW
    w1 AS (PARTITION BY region ORDER BY lifeexpectancy DESC),
    w2 AS (PARTITION BY continent ORDER BY lifeexpectancy DESC)
ORDER BY
    region_rank
```

       postgresql://postgres:***@localhost/imdb
       postgresql://postgres:***@localhost:5432/imdb
       postgresql://postgres:***@localhost:5432/mds
     * postgresql://postgres:***@localhost:5432/world
    154 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>region_rank</th>
            <th>continent_rank</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Cameroon</td>
            <td>1</td>
            <td>12</td>
        </tr>
        <tr>
            <td>Czech Republic</td>
            <td>1</td>
            <td>18</td>
        </tr>
        <tr>
            <td>Israel</td>
            <td>1</td>
            <td>4</td>
        </tr>
        <tr>
            <td>Cuba</td>
            <td>1</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Senegal</td>
            <td>1</td>
            <td>7</td>
        </tr>
        <tr>
            <td>Canada</td>
            <td>1</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Mauritius</td>
            <td>1</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Chile</td>
            <td>1</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Sri Lanka</td>
            <td>1</td>
            <td>10</td>
        </tr>
        <tr>
            <td>South Africa</td>
            <td>1</td>
            <td>17</td>
        </tr>
        <tr>
            <td>Libyan Arab Jamahiriya</td>
            <td>1</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Italy</td>
            <td>1</td>
            <td>3</td>
        </tr>
        <tr>
            <td>United Kingdom</td>
            <td>1</td>
            <td>10</td>
        </tr>
        <tr>
            <td>Papua New Guinea</td>
            <td>1</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Sweden</td>
            <td>1</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Estonia</td>
            <td>1</td>
            <td>29</td>
        </tr>
        <tr>
            <td>Switzerland</td>
            <td>1</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Australia</td>
            <td>1</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Singapore</td>
            <td>1</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Japan</td>
            <td>1</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Costa Rica</td>
            <td>1</td>
            <td>4</td>
        </tr>
        <tr>
            <td>Jordan</td>
            <td>2</td>
            <td>5</td>
        </tr>
        <tr>
            <td>Hong Kong</td>
            <td>2</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Spain</td>
            <td>2</td>
            <td>4</td>
        </tr>
        <tr>
            <td>Malaysia</td>
            <td>2</td>
            <td>16</td>
        </tr>
        <tr>
            <td>Ireland</td>
            <td>2</td>
            <td>14</td>
        </tr>
        <tr>
            <td>Ghana</td>
            <td>2</td>
            <td>8</td>
        </tr>
        <tr>
            <td>Slovakia</td>
            <td>2</td>
            <td>20</td>
        </tr>
        <tr>
            <td>Chad</td>
            <td>2</td>
            <td>21</td>
        </tr>
        <tr>
            <td>Norway</td>
            <td>2</td>
            <td>6</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">154 rows, truncated to displaylimit of 30</span>



<br><br><br>

## Indexing and performance (optional)

Check the optional notebook if you want to learn more about indexing and performance.
