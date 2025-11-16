

# Lecture 2: Data types, filtering, functions

## Lecture outline

- Various data types in SQL
- `WHERE` conditionals, pattern matching
- Derived columns, aliases with `AS`
- Conditionals with `CASE`
- Functions and operators in SQL


```python
%load_ext sql
%config SqlMagic.displaylimit = 20
```


```python
import json
import urllib.parse

with open('data/credentials.json') as f:
    login = json.load(f)

user = login['user']
password = urllib.parse.quote(login['password'])
host = login['host']
port = login['port']
```


```python
%sql postgresql://{user}:{password}@{host}:{port}/imdb
```

## Data types

You might remember from previous lecture that in relational databases, each column is characterized with its name and its **domain**. A domain is the set of permissible or valid values that a column is allowed to store. This highlights one of the advantages of using a DBMS, which enforces particular data types for the columns of a table.

Postgres supports

- boolean
- character
- number
- datetime
- binary

and some extension types specific to Postgres.

<br><br><br>

### Type conversion

To demonstrate how different data types work in SQL, I first need to show you how we convert values from one type to another. In standard SQL, type conversion can be done using the `CAST` function:

```sql
CAST(<column> AS <data_type>)
```
In Postgres, we can also use the double-colon syntax as a shorthand for the above `CAST` function:

```sql
<column>::<data_type>
```

<br><br><br>

### Boolean

We can specify this data type using the keyword `BOOLEAN` or `BOOL`.

Valid values are
- `'TRUE'`, `1` (or any other positive integer), `'YES'`, `'Y'`, `'T'`,
- `'FALSE'`, `'0'`, `'NO'`, `'N'`, `'F'`

Note that all of these values will be interpreted as `'TRUE'`, `'FALSE'`  by Postgres:


```sql
%%sql

SELECT
    'TRUE'::BOOLEAN,
    'T'::BOOLEAN,
    '0'::BOOLEAN,
    'NO'::BOOLEAN
;
```

     * postgresql://postgres:***@localhost:5432/imdb
    1 rows affected.
    




<table>
    <tr>
        <th>bool</th>
        <th>bool_1</th>
        <th>bool_2</th>
        <th>bool_3</th>
    </tr>
    <tr>
        <td>True</td>
        <td>True</td>
        <td>False</td>
        <td>False</td>
    </tr>
</table>



<br><br><br>

### Characters

The character data type is used to represent fixed-length and variable length character strings. This type can be defined using the following keywords:

- `CHAR(n)`: a string of exactly `n` characters padded with spaces
- `VARCHAR(n)`: a variable set of `n` characters
- `TEXT` which is a Postgres specific type for which there is practically no limit on the number of characters.


```sql
%%sql

SELECT
    'Arman'::CHAR(50),
    'Arman'::VARCHAR(2),
    'Arman'::TEXT
;
```

     * postgresql://postgres:***@localhost:5432/imdb
    1 rows affected.
    




<table>
    <tr>
        <th>bpchar</th>
        <th>varchar</th>
        <th>text</th>
    </tr>
    <tr>
        <td>Arman                                             </td>
        <td>Ar</td>
        <td>Arman</td>
    </tr>
</table>



> Note that you can't see the space-padding for `CHAR(50) 'Arman'` in the Jupyter notebook, but if you run the same statement in `psql`, you will see `'Arman'` + 45 spaces in the output.

<br><br><br>

### Numbers

Numerical values in Postgres belong to the following general categories:
- Integers
- Floating-point numbers
- Arbitrary precision numbers

**Integers:**

| Name     | Storage Size | Description                | Range                                        |
|----------|--------------|----------------------------|----------------------------------------------|
| `smallint` | 2 bytes      | small-range integer        | -32768 to +32767                             |
| `integer`  | 4 bytes      | typical choice for integer | -2147483648 to +2147483647                   |
| `bigint`   | 8 bytes      | large-range integer        | -9223372036854775808 to +9223372036854775807 |
| `serial`      | 4 bytes | auto-incrementing integer       | 1 to 2147483647          |
| `bigserial`   | 8 bytes | large auto-incrementing integer | 1 to 9223372036854775807 |

We'll learn later that the `serial` type (which is not an actual data type) is a shortcut to tell Postgres create unique "auto-incrementing" often used for the primary key column of table.

**Floating-point numbers:**

| Name     | Storage Size | Description                | Range                                        |
|----------|--------------|----------------------------|----------------------------------------------|
| `real`             | 4 bytes  | variable-precision, inexact     | at least 6 decimal digits (implementation dependent) |
| `double precision` | 8 bytes  | variable-precision, inexact     | at least 15 decimal digits (implementation dependent) |

**Arbitrary precision numbers**

| Name     | Storage Size | Description                | Range                                        |
|----------|--------------|----------------------------|----------------------------------------------|
| `numeric`          | variable | user-specified precision, exact | 131072 digits before and 16383 digits after the decimal point |
| `decimal`          | variable | user-specified precision, exact | 131072 digits before and 16383 digits after the decimal point |

<br>

> `DECIMAL` and `NUMERIC` data types are exactly the same thing in Postgres.


```sql
%%sql

SELECT CAST(44.7874 AS SMALLINT);
```

     * postgresql://postgres:***@localhost:5432/imdb
    1 rows affected.
    




<table>
    <tr>
        <th>int2</th>
    </tr>
    <tr>
        <td>45</td>
    </tr>
</table>



<br><br><br>

Note the `::` notation can also be used instead of `CAST()` (and maybe preferred, but specific to Postgres):


```sql
%%sql

SELECT 44.7874::SMALLINT;
```

     * postgresql://postgres:***@localhost:5432/imdb
    1 rows affected.
    




<table>
    <tr>
        <th>int2</th>
    </tr>
    <tr>
        <td>45</td>
    </tr>
</table>




```sql
%%sql

SELECT CAST(4.54021223948E-8 AS REAL);
```

     * postgresql://postgres:***@localhost:5432/imdb
    1 rows affected.
    




<table>
    <tr>
        <th>float4</th>
    </tr>
    <tr>
        <td>4.540212e-08</td>
    </tr>
</table>



With the `numeric` data type, we can specify the total number of significant digits to store (known as precision) as well as the number of digits in the fractional part (known as scale) by specifying `NUMERIC(precision, scale)`:


```sql
%%sql

SELECT CAST('183.123456789' AS NUMERIC(5, 2));
```

     * postgresql://postgres:***@localhost:5432/imdb
    1 rows affected.
    




<table>
    <tr>
        <th>numeric</th>
    </tr>
    <tr>
        <td>183.12</td>
    </tr>
</table>



The `numeric` type is exact (as opposed to other types of floats) and immune to the round-off error, but it is **slow to work with for the DBMS**. It is often used for monetary and financial data, where either numbers with a many digits may be stored or exactness is important.

For example, the following number cannot be represented as `BIGINT` and would throw an error, but it works with `NUMERIC`:


```sql
%%sql

SELECT CAST(9223372036854775808983277853434530982 AS NUMERIC);
```

     * postgresql://postgres:***@localhost:5432/imdb
    1 rows affected.
    




<table>
    <tr>
        <th>numeric</th>
    </tr>
    <tr>
        <td>9223372036854775808983277853434530982</td>
    </tr>
</table>



<br><br><br>

### Date/time

Postgres provides **datetime** and **interval** data types similar to those we've seen in DSCI 511 in Python and Pandas.

#### Datetimes

- `DATE` for dates
- `TIME` for the time of day

Postgres also provides two ways to store the **timestamp** datatype;
- `TIMESTAMP` for date + time
- `TIMESTAMPTZ` for date + time + timezone (Postgres specific)

When a timestamp value is queried:
- For `TIMESTAMP`, Postgres returns the timestamp as originally stored in the database server
- For `TIMESTAMPTZ`, Postgres converts the timestamp into the local timezone of the database server

Note that Postgres does not store timezone information. It always internally stores `TIMESTAMPTZ` in [UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time) value, and does the back-conversion using the local time zone of the database server.

<br><br><br>

**Entering datetime data**

Postgres does a pretty good job of getting the datetimes right even if we don't enter them in the standard ISO way. Let's take a look at a few examples:


```sql
%%sql

SELECT
    'January 23, 2021'::DATE,
    '23 January 2021'::DATE,
    '2021 1 23'::DATE,
    '1/23/2021'::DATE,
    'today'::DATE,
    'tomorrow'::DATE
;
```

     * postgresql://postgres:***@localhost:5432/
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>date</th>
            <th>date_1</th>
            <th>date_2</th>
            <th>date_3</th>
            <th>date_4</th>
            <th>date_5</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2021-01-23</td>
            <td>2021-01-23</td>
            <td>2021-01-23</td>
            <td>2021-01-23</td>
            <td>2022-11-17</td>
            <td>2022-11-18</td>
        </tr>
    </tbody>
</table>




```sql
%%sql

SELECT
    '14:24:00'::TIME,
    '2:24pm'::TIME,
    '2:24 PM PST'::TIME WITH TIME ZONE,
    'now'::TIME,
    'now'::TIME WITH TIME ZONE
;
```

     * postgresql://postgres:***@localhost:5432/
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>time</th>
            <th>time_1</th>
            <th>timetz</th>
            <th>time_2</th>
            <th>timetz_1</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>14:24:00</td>
            <td>14:24:00</td>
            <td>14:24:00-08:00</td>
            <td>02:11:08.571179</td>
            <td>02:11:08.571179-08:00</td>
        </tr>
    </tbody>
</table>



<br><br><br>

When datetime is stored without timezone, it is oblivious to the local server timezone:


```python
%sql SELECT '2021-11-18 8:30:00'::TIMESTAMP;
```

     * postgresql://postgres:***@localhost:5432/
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>timestamp</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2021-11-18 08:30:00</td>
        </tr>
    </tbody>
</table>



<br><br><br>


```python
%sql SELECT '2021-11-18 8:30:00'::TIMESTAMPTZ;
```

     * postgresql://postgres:***@localhost:5432/
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>timestamptz</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2021-11-18 08:30:00-08:00</td>
        </tr>
    </tbody>
</table>




```python
%sql SHOW TIMEZONE;
```

     * postgresql://postgres:***@localhost:5432/
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>TimeZone</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>America/Vancouver</td>
        </tr>
    </tbody>
</table>



<br><br><br>


```python
%sql SET timezone = 'America/New_York';
```

     * postgresql://postgres:***@localhost:5432/
    Done.
    




    []




```python
%sql SELECT '2021-11-18 8:30:00 -08:00'::TIMESTAMPTZ;
```

     * postgresql://postgres:***@localhost:5432/
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>timestamptz</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2021-11-18 11:30:00-05:00</td>
        </tr>
    </tbody>
</table>



<br><br><br>


```python
%sql SET timezone = 'America/Vancouver';
```

     * postgresql://postgres:***@localhost:5432/
    Done.
    




    []




```python
%sql SELECT '2021-11-18 8:30:00'::TIMESTAMPTZ;
```

     * postgresql://postgres:***@localhost:5432/
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>timestamptz</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2021-11-18 08:30:00-08:00</td>
        </tr>
    </tbody>
</table>



<br><br><br>

#### Intervals (OPTIONAL)

There is also another datatype for storing intervals of time. Intervals are useful for doing date and time arithmetic, such as adding a duration of time to a timestamp.

For more detailed information, refer to the Postgres documentation [here](https://www.postgresql.org/docs/8.4/datatype-datetime.html).


```sql
%%sql

SELECT
    '1 day 23 hours 8 minutes'::INTERVAL,
    '2m 18s'::INTERVAL,
    '3 years 2 months'::INTERVAL
;
```

     * postgresql://postgres:***@localhost:5432/
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>interval</th>
            <th>interval_1</th>
            <th>interval_2</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>1 day, 23:08:00</td>
            <td>0:02:18</td>
            <td>1155 days, 0:00:00</td>
        </tr>
    </tbody>
</table>



### Binary data (OPTIONAL)

It is also possible to have binary data in a table (e.g. documents, images, videos). We don't use binary data in this course.

<br><br><br>

### Nulls

A null is marker to indicate that the value for a column is unknown, or not entered yet. A null is not equal to 0, or an empty string. In fact, a null is not even equal to another null!

How different environments show nulls:
- `ipython-sql` -> `None`
- psql -> blank space
- pgAdmin -> `[null]`

<br><br><br>

## Filtering rows with `WHERE`

We've seen the `WHERE` keyword in passing in the last lecture. `WHERE` is an intuitive keyword that is used to filter rows based on a particular condition. The syntax is as follows:

```sql
SELECT
    column1, column2
FROM
    table1
WHERE
    condition
;
```

| Condition        | Operator                        |
|------------------|---------------------------------|
| Comparison       | `=`, `<>`, `<`, `<=`, `>`, `>=` |
| Pattern matching | `LIKE`                          |
| Range            | `BETWEEN`                       |
| List             | `IN`                            |
| Null testing     | `IS NULL`                       |

<br><br><br>


```sql
%%sql

SELECT
    *
FROM
    movies;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb_dsci513
    26058 rows affected.
    




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
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>10035423</td>
            <td>Kate &amp; Leopold</td>
            <td>None</td>
            <td>2001</td>
            <td>None</td>
            <td>118</td>
            <td>6.4</td>
            <td>74982</td>
        </tr>
        <tr>
            <td>10042742</td>
            <td>Mister 880</td>
            <td>None</td>
            <td>1950</td>
            <td>None</td>
            <td>90</td>
            <td>7.1</td>
            <td>1171</td>
        </tr>
        <tr>
            <td>10041181</td>
            <td>Black Hand</td>
            <td>None</td>
            <td>1950</td>
            <td>None</td>
            <td>92</td>
            <td>6.4</td>
            <td>666</td>
        </tr>
        <tr>
            <td>10041387</td>
            <td>Francis</td>
            <td>None</td>
            <td>1950</td>
            <td>None</td>
            <td>91</td>
            <td>6.4</td>
            <td>979</td>
        </tr>
        <tr>
            <td>10041719</td>
            <td>Orpheus</td>
            <td>Orphée</td>
            <td>1950</td>
            <td>None</td>
            <td>95</td>
            <td>8.0</td>
            <td>9346</td>
        </tr>
        <tr>
            <td>10041931</td>
            <td>Stromboli</td>
            <td>Stromboli, terra di Dio</td>
            <td>1950</td>
            <td>None</td>
            <td>107</td>
            <td>7.3</td>
            <td>5239</td>
        </tr>
        <tr>
            <td>10042052</td>
            <td>Woman in Hiding</td>
            <td>None</td>
            <td>1950</td>
            <td>None</td>
            <td>92</td>
            <td>6.9</td>
            <td>553</td>
        </tr>
        <tr>
            <td>10042179</td>
            <td>Abbott and Costello in the Foreign Legion</td>
            <td>None</td>
            <td>1950</td>
            <td>None</td>
            <td>80</td>
            <td>6.6</td>
            <td>2573</td>
        </tr>
        <tr>
            <td>10042200</td>
            <td>Annie Get Your Gun</td>
            <td>None</td>
            <td>1950</td>
            <td>None</td>
            <td>107</td>
            <td>6.9</td>
            <td>4050</td>
        </tr>
        <tr>
            <td>10042206</td>
            <td>Armored Car Robbery</td>
            <td>None</td>
            <td>1950</td>
            <td>None</td>
            <td>67</td>
            <td>7.0</td>
            <td>2077</td>
        </tr>
        <tr>
            <td>10042208</td>
            <td>The Asphalt Jungle</td>
            <td>None</td>
            <td>1950</td>
            <td>None</td>
            <td>112</td>
            <td>7.9</td>
            <td>22106</td>
        </tr>
        <tr>
            <td>10042211</td>
            <td>Atom Man vs. Superman</td>
            <td>None</td>
            <td>1950</td>
            <td>None</td>
            <td>252</td>
            <td>7.0</td>
            <td>579</td>
        </tr>
        <tr>
            <td>10042219</td>
            <td>Backfire</td>
            <td>None</td>
            <td>1950</td>
            <td>None</td>
            <td>91</td>
            <td>6.6</td>
            <td>955</td>
        </tr>
        <tr>
            <td>10042229</td>
            <td>The Baron of Arizona</td>
            <td>None</td>
            <td>1950</td>
            <td>None</td>
            <td>97</td>
            <td>7.0</td>
            <td>1748</td>
        </tr>
        <tr>
            <td>10042249</td>
            <td>The Big Lift</td>
            <td>None</td>
            <td>1950</td>
            <td>None</td>
            <td>120</td>
            <td>6.5</td>
            <td>1266</td>
        </tr>
        <tr>
            <td>10042256</td>
            <td>The Black Rose</td>
            <td>None</td>
            <td>1950</td>
            <td>None</td>
            <td>120</td>
            <td>6.3</td>
            <td>1657</td>
        </tr>
        <tr>
            <td>10042265</td>
            <td>The Blue Lamp</td>
            <td>None</td>
            <td>1950</td>
            <td>None</td>
            <td>84</td>
            <td>6.9</td>
            <td>1426</td>
        </tr>
        <tr>
            <td>10042274</td>
            <td>Borderline</td>
            <td>None</td>
            <td>1950</td>
            <td>None</td>
            <td>88</td>
            <td>6.1</td>
            <td>996</td>
        </tr>
        <tr>
            <td>10042275</td>
            <td>Born to Be Bad</td>
            <td>None</td>
            <td>1950</td>
            <td>None</td>
            <td>94</td>
            <td>6.7</td>
            <td>1797</td>
        </tr>
        <tr>
            <td>10042279</td>
            <td>Branded</td>
            <td>None</td>
            <td>1950</td>
            <td>None</td>
            <td>104</td>
            <td>6.7</td>
            <td>702</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">26058 rows, truncated to displaylimit of 20</span>



<br><br><br>

---

**Example:** Retrieve rows for movies produced in or after 2010.

---


```sql
%%sql

SELECT
    *
FROM
    movies
WHERE
    start_year >= 2010
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb_dsci513
    8804 rows affected.
    




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
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>10069049</td>
            <td>The Other Side of the Wind</td>
            <td>None</td>
            <td>2018</td>
            <td>None</td>
            <td>122</td>
            <td>6.9</td>
            <td>4904</td>
        </tr>
        <tr>
            <td>10176694</td>
            <td>The Tragedy of Man</td>
            <td>Az ember tragédiája</td>
            <td>2011</td>
            <td>None</td>
            <td>160</td>
            <td>7.8</td>
            <td>610</td>
        </tr>
        <tr>
            <td>10293069</td>
            <td>Dark Blood</td>
            <td>None</td>
            <td>2012</td>
            <td>None</td>
            <td>86</td>
            <td>6.5</td>
            <td>1073</td>
        </tr>
        <tr>
            <td>10315642</td>
            <td>Wazir</td>
            <td>None</td>
            <td>2016</td>
            <td>None</td>
            <td>103</td>
            <td>7.1</td>
            <td>15796</td>
        </tr>
        <tr>
            <td>10337692</td>
            <td>On the Road</td>
            <td>None</td>
            <td>2012</td>
            <td>None</td>
            <td>124</td>
            <td>6.1</td>
            <td>38216</td>
        </tr>
        <tr>
            <td>10359950</td>
            <td>The Secret Life of Walter Mitty</td>
            <td>None</td>
            <td>2013</td>
            <td>None</td>
            <td>114</td>
            <td>7.3</td>
            <td>278645</td>
        </tr>
        <tr>
            <td>10365907</td>
            <td>A Walk Among the Tombstones</td>
            <td>None</td>
            <td>2014</td>
            <td>None</td>
            <td>114</td>
            <td>6.5</td>
            <td>106413</td>
        </tr>
        <tr>
            <td>10369610</td>
            <td>Jurassic World</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>124</td>
            <td>7.0</td>
            <td>547391</td>
        </tr>
        <tr>
            <td>10376136</td>
            <td>The Rum Diary</td>
            <td>None</td>
            <td>2011</td>
            <td>None</td>
            <td>119</td>
            <td>6.2</td>
            <td>95417</td>
        </tr>
        <tr>
            <td>10376479</td>
            <td>American Pastoral</td>
            <td>None</td>
            <td>2016</td>
            <td>None</td>
            <td>108</td>
            <td>6.1</td>
            <td>13376</td>
        </tr>
        <tr>
            <td>10398286</td>
            <td>Tangled</td>
            <td>None</td>
            <td>2010</td>
            <td>None</td>
            <td>100</td>
            <td>7.7</td>
            <td>373355</td>
        </tr>
        <tr>
            <td>10401729</td>
            <td>John Carter</td>
            <td>None</td>
            <td>2012</td>
            <td>None</td>
            <td>132</td>
            <td>6.6</td>
            <td>244429</td>
        </tr>
        <tr>
            <td>10409379</td>
            <td>In Secret</td>
            <td>None</td>
            <td>2013</td>
            <td>None</td>
            <td>107</td>
            <td>6.1</td>
            <td>7146</td>
        </tr>
        <tr>
            <td>10409847</td>
            <td>Cowboys &amp; Aliens</td>
            <td>None</td>
            <td>2011</td>
            <td>None</td>
            <td>119</td>
            <td>6.0</td>
            <td>197867</td>
        </tr>
        <tr>
            <td>10420293</td>
            <td>The Stanford Prison Experiment</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>122</td>
            <td>6.9</td>
            <td>33319</td>
        </tr>
        <tr>
            <td>10429493</td>
            <td>The A-Team</td>
            <td>None</td>
            <td>2010</td>
            <td>None</td>
            <td>117</td>
            <td>6.7</td>
            <td>237537</td>
        </tr>
        <tr>
            <td>10433035</td>
            <td>Real Steel</td>
            <td>None</td>
            <td>2011</td>
            <td>None</td>
            <td>127</td>
            <td>7.1</td>
            <td>285715</td>
        </tr>
        <tr>
            <td>10435761</td>
            <td>Toy Story 3</td>
            <td>None</td>
            <td>2010</td>
            <td>None</td>
            <td>103</td>
            <td>8.3</td>
            <td>701340</td>
        </tr>
        <tr>
            <td>10435651</td>
            <td>The Giver</td>
            <td>None</td>
            <td>2014</td>
            <td>None</td>
            <td>97</td>
            <td>6.5</td>
            <td>106439</td>
        </tr>
        <tr>
            <td>10437086</td>
            <td>Alita: Battle Angel</td>
            <td>None</td>
            <td>2019</td>
            <td>None</td>
            <td>122</td>
            <td>7.4</td>
            <td>162513</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">8804 rows, truncated to displaylimit of 20</span>



<br><br><br>

---

**Example:** Retrieve the row for the movie called "Lost Highway".

---


```sql
%%sql

SELECT
    *
FROM
    movies
WHERE
    title = 'Lost Highway'
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb
    1 rows affected.
    




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
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>10116922</td>
            <td>Lost Highway</td>
            <td>None</td>
            <td>1997</td>
            <td>None</td>
            <td>134</td>
            <td>7.6</td>
            <td>120549</td>
        </tr>
    </tbody>
</table>



> Note that in SQL, strings are enclosed in single quotes, i.e. `'string'`.

> While SQL syntax is case-insensitive, SQL is **case-sensitive** when it comes to **comparing strings**. In the above example, `'Lost highway'` will not return any rows.

<br><br><br>

### Logical operators `AND`, `OR`, and `NOT`

Just like in Python, we can combine multiple conditions logical/boolean operators `AND`, `OR`, and `NOT`.

When there are multiple logical operators, `NOT` is evaluated first, then `AND` and finally `OR`.

We can enclose each condition in parentheses if we want. This can be done either for readability, or to override the default precedence rules.

---

**Example:** Retrieve the rows for movies that are produced in 2015 and are rated higher than 8.

---


```sql
%%sql

SELECT
    *
FROM
    movies
WHERE
    start_year = 2015
    AND
    rating > 8
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb_dsci513
    0 rows affected.
    




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
        </tr>
    </thead>
    <tbody>
    </tbody>
</table>



<br><br><br>

---

**Example:** Retrieve the rows for movies that are produced either in 2015 or 2018, and are rated higher than 8.

---


```sql
%%sql

SELECT
    *
FROM
    movies
WHERE
    start_year = 2015
    OR 
    start_year = 2018
    AND
    rating > 8
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb_dsci513
    1048 rows affected.
    




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
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>10369610</td>
            <td>Jurassic World</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>124</td>
            <td>7.0</td>
            <td>547391</td>
        </tr>
        <tr>
            <td>10420293</td>
            <td>The Stanford Prison Experiment</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>122</td>
            <td>6.9</td>
            <td>33319</td>
        </tr>
        <tr>
            <td>10478970</td>
            <td>Ant-Man</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>117</td>
            <td>7.3</td>
            <td>517941</td>
        </tr>
        <tr>
            <td>10790770</td>
            <td>Miles Ahead</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>100</td>
            <td>6.4</td>
            <td>8650</td>
        </tr>
        <tr>
            <td>10884732</td>
            <td>The Wedding Ringer</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>101</td>
            <td>6.6</td>
            <td>67575</td>
        </tr>
        <tr>
            <td>11533089</td>
            <td>Tab Hunter Confidential</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>90</td>
            <td>7.8</td>
            <td>2852</td>
        </tr>
        <tr>
            <td>11596363</td>
            <td>The Big Short</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>130</td>
            <td>7.8</td>
            <td>318033</td>
        </tr>
        <tr>
            <td>11598642</td>
            <td>Z for Zachariah</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>98</td>
            <td>6.0</td>
            <td>25985</td>
        </tr>
        <tr>
            <td>11618448</td>
            <td>Racing Extinction</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>90</td>
            <td>8.3</td>
            <td>7042</td>
        </tr>
        <tr>
            <td>11638355</td>
            <td>The Man from U.N.C.L.E.</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>116</td>
            <td>7.3</td>
            <td>245184</td>
        </tr>
        <tr>
            <td>11655441</td>
            <td>The Age of Adaline</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>112</td>
            <td>7.2</td>
            <td>141949</td>
        </tr>
        <tr>
            <td>11658801</td>
            <td>Freeheld</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>103</td>
            <td>6.6</td>
            <td>10772</td>
        </tr>
        <tr>
            <td>11663202</td>
            <td>The Revenant</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>156</td>
            <td>8.0</td>
            <td>636721</td>
        </tr>
        <tr>
            <td>11666801</td>
            <td>The Duff</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>101</td>
            <td>6.5</td>
            <td>76639</td>
        </tr>
        <tr>
            <td>11674771</td>
            <td>Entourage</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>104</td>
            <td>6.6</td>
            <td>72367</td>
        </tr>
        <tr>
            <td>11683048</td>
            <td>De Palma</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>110</td>
            <td>7.4</td>
            <td>4202</td>
        </tr>
        <tr>
            <td>11698654</td>
            <td>Catching the Sun</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>75</td>
            <td>6.8</td>
            <td>517</td>
        </tr>
        <tr>
            <td>11702429</td>
            <td>Eisenstein in Guanajuato</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>105</td>
            <td>6.2</td>
            <td>2317</td>
        </tr>
        <tr>
            <td>11708135</td>
            <td>Elser</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>114</td>
            <td>7.0</td>
            <td>7455</td>
        </tr>
        <tr>
            <td>11727770</td>
            <td>Absolutely Anything</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>85</td>
            <td>6.0</td>
            <td>35659</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">1048 rows, truncated to displaylimit of 20</span>



What? This isn't the right result! We have multiple returned movies that are rated below 8.

The reason is that the `AND` operator takes precedence over `OR`. Therefore, `start_year = 2018 AND rating > 8` gets evaluated first, and then the result is passed to the `OR` part of the condition. In order to override this behaviour, we can rewrite our query in the following way:


```sql
%%sql

SELECT
    *
FROM
    movies
WHERE
    (start_year = 2015
    OR
    start_year = 2018)
    AND
    rating > 8
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb_dsci513
    119 rows affected.
    




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
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>11618448</td>
            <td>Racing Extinction</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>90</td>
            <td>8.3</td>
            <td>7042</td>
        </tr>
        <tr>
            <td>12096673</td>
            <td>Inside Out</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>95</td>
            <td>8.2</td>
            <td>550606</td>
        </tr>
        <tr>
            <td>12473476</td>
            <td>Be Here Now</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>100</td>
            <td>8.7</td>
            <td>2863</td>
        </tr>
        <tr>
            <td>12631186</td>
            <td>Baahubali: The Beginning</td>
            <td>Bahubali: The Beginning</td>
            <td>2015</td>
            <td>None</td>
            <td>159</td>
            <td>8.1</td>
            <td>94989</td>
        </tr>
        <tr>
            <td>12865822</td>
            <td>All the World in a Design School</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>59</td>
            <td>8.4</td>
            <td>1270</td>
        </tr>
        <tr>
            <td>13170832</td>
            <td>Room</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>118</td>
            <td>8.2</td>
            <td>326042</td>
        </tr>
        <tr>
            <td>13270538</td>
            <td>Requiem for the American Dream</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>73</td>
            <td>8.1</td>
            <td>8061</td>
        </tr>
        <tr>
            <td>13717510</td>
            <td>The Drop Box</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>79</td>
            <td>8.1</td>
            <td>604</td>
        </tr>
        <tr>
            <td>13865286</td>
            <td>My Lonely Me</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>95</td>
            <td>8.2</td>
            <td>671</td>
        </tr>
        <tr>
            <td>14112208</td>
            <td>Kuttram Kadithal</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>120</td>
            <td>8.1</td>
            <td>638</td>
        </tr>
        <tr>
            <td>14145178</td>
            <td>Listen to Me Marlon</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>103</td>
            <td>8.2</td>
            <td>6447</td>
        </tr>
        <tr>
            <td>14154756</td>
            <td>Avengers: Infinity War</td>
            <td>None</td>
            <td>2018</td>
            <td>None</td>
            <td>149</td>
            <td>8.5</td>
            <td>711500</td>
        </tr>
        <tr>
            <td>14393514</td>
            <td>Bitter Lake</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>136</td>
            <td>8.2</td>
            <td>2356</td>
        </tr>
        <tr>
            <td>14430212</td>
            <td>Drishyam</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>163</td>
            <td>8.3</td>
            <td>57527</td>
        </tr>
        <tr>
            <td>14429128</td>
            <td>Papanasam</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>179</td>
            <td>8.4</td>
            <td>4802</td>
        </tr>
        <tr>
            <td>14449576</td>
            <td>Tomorrow</td>
            <td>Demain</td>
            <td>2015</td>
            <td>None</td>
            <td>118</td>
            <td>8.1</td>
            <td>2887</td>
        </tr>
        <tr>
            <td>14476736</td>
            <td>Hamlet</td>
            <td>National Theatre Live: Hamlet</td>
            <td>2015</td>
            <td>None</td>
            <td>217</td>
            <td>8.6</td>
            <td>1670</td>
        </tr>
        <tr>
            <td>14519488</td>
            <td>Mudras Calling</td>
            <td>None</td>
            <td>2018</td>
            <td>None</td>
            <td>95</td>
            <td>8.7</td>
            <td>1196</td>
        </tr>
        <tr>
            <td>14635372</td>
            <td>Masaan</td>
            <td>None</td>
            <td>2015</td>
            <td>None</td>
            <td>109</td>
            <td>8.1</td>
            <td>19638</td>
        </tr>
        <tr>
            <td>14633694</td>
            <td>Spider-Man: Into the Spider-Verse</td>
            <td>None</td>
            <td>2018</td>
            <td>None</td>
            <td>117</td>
            <td>8.4</td>
            <td>262505</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">119 rows, truncated to displaylimit of 20</span>



<br><br><br>

---

**Example:** Count the number of movies that have no less than 1 million votes.

---

We need to use the `COUNT()` function to count the number of returned rows (more on `COUNT()` in a later lecture):


```sql
%%sql

SELECT
    COUNT(*)
FROM
    movies
WHERE
    NOT nvotes < 1000000
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb_dsci513
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>count</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>33</td>
        </tr>
    </tbody>
</table>



<br><br><br>

---

**Example:** Find the genres listed for the movie "The Godfather".

---

We will learn in later lectures how to answer this question in a single SQL query in various ways, but for now, we have to take a two step process. Here, we are trying to find information related to each are that are stored in two tables. This is the first time we actually encounter the notion of a **relational** database in practice!

It turns out that are the `id` column in the `movie` table and `movie_id` in `movies_genre` table reference the same movies. These columns actually relate to two tables together. For our query, we have to find out the `id` of the movie `'The Godfather'` first, and then use it to retrieve the genres associated with that movie:


```sql
%%sql

SELECT
    id, title
FROM
    movies
WHERE
    title = 'The Godfather'
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb_dsci513
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>id</th>
            <th>title</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>10068646</td>
            <td>The Godfather</td>
        </tr>
    </tbody>
</table>



Alright, the id for `'The Godfather'` is `10068646`. Now let's retrieve the genres for this id from the `movie_genres` table:


```sql
%%sql

SELECT
    *
FROM
    movie_genres
WHERE
    movie_id = 10068646
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb_dsci513
    2 rows affected.
    




<table>
    <thead>
        <tr>
            <th>movie_id</th>
            <th>genre</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>10068646</td>
            <td>crime</td>
        </tr>
        <tr>
            <td>10068646</td>
            <td>drama</td>
        </tr>
    </tbody>
</table>



<br><br><br>

---

**Question:**
Do you think using the following query, you can find the number of movies in the `movie_genres` table that are NOT listed as `'drama'`?
    
```sql
SELECT
    COUNT(DISTINCT movie_id)
FROM
    movie_genres
WHERE
    genre <> 'drama'
;
```

---

<br><br><br>

### Pattern matching

It is a quite common situation that we want to find rows for which the values of one or more columns match a particular pattern. In SQL, this can be done either using `LIKE` or by using regular expressions. The syntax is as follows:

```sql
SELECT
    column1, column2
FROM
    table1
WHERE
    column1 [NOT] LIKE '<pattern>'
;
```

Postgres provides us with two wild-cards that we can use with `LIKE`:
- `%` matches any string of characters
- `_` matches a single character.

Pattern matching with `LIKE` is case sensitive; however, Postgres also provides the `ILIKE` keyword that has the same functionality as `LIKE` but is case-insensitive.

> **Note:** With `LIKE` or `ILIKE`, the entire string should match the pattern.


```sql
%%sql

SELECT
    'Arman' LIKE '%a_',
    'UBC' LIKE '_B_',
    'MDS is awesome!' LIKE '%!_'
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb_dsci513
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>?column?</th>
            <th>?column?_1</th>
            <th>?column?_2</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>True</td>
            <td>True</td>
            <td>False</td>
        </tr>
    </tbody>
</table>



<br><br><br>

---

**Example:** Retrieve those movies from the `movie` table whose title contains the word `'violin'` (note that `LIKE` is picky about letter cases in strings!)

---


```sql
%%sql

SELECT
    *
FROM
    movies
WHERE
    title LIKE '%Violin%'
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb_dsci513
    5 rows affected.
    




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
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>10120802</td>
            <td>The Red Violin</td>
            <td>Le violon rouge</td>
            <td>1998</td>
            <td>None</td>
            <td>130</td>
            <td>7.6</td>
            <td>30285</td>
        </tr>
        <tr>
            <td>10451966</td>
            <td>The Violin</td>
            <td>El violín</td>
            <td>2005</td>
            <td>None</td>
            <td>98</td>
            <td>7.7</td>
            <td>2212</td>
        </tr>
        <tr>
            <td>12401715</td>
            <td>The Devil&#x27;s Violinist</td>
            <td>None</td>
            <td>2013</td>
            <td>None</td>
            <td>122</td>
            <td>6.1</td>
            <td>3033</td>
        </tr>
        <tr>
            <td>14972904</td>
            <td>The Violin Teacher</td>
            <td>Tudo Que Aprendemos Juntos</td>
            <td>2015</td>
            <td>None</td>
            <td>102</td>
            <td>6.8</td>
            <td>645</td>
        </tr>
        <tr>
            <td>10053987</td>
            <td>The Steamroller and the Violin</td>
            <td>Katok i skripka</td>
            <td>1961</td>
            <td>None</td>
            <td>46</td>
            <td>7.5</td>
            <td>4867</td>
        </tr>
    </tbody>
</table>



<br><br><br>

---

**Example:** Retrieve those movies from the `movie` table whose title starts with the word `'Zero'`.

---


```sql
%%sql

SELECT
    *
FROM
    movies
WHERE
    title LIKE 'Zero%'
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb_dsci513
    18 rows affected.
    




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
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>10095244</td>
            <td>Zerograd</td>
            <td>Gorod Zero</td>
            <td>1988</td>
            <td>None</td>
            <td>103</td>
            <td>7.5</td>
            <td>1463</td>
        </tr>
        <tr>
            <td>10113557</td>
            <td>Zero Kelvin</td>
            <td>Kjærlighetens kjøtere</td>
            <td>1995</td>
            <td>None</td>
            <td>118</td>
            <td>7.3</td>
            <td>1711</td>
        </tr>
        <tr>
            <td>10120906</td>
            <td>Zero Effect</td>
            <td>None</td>
            <td>1998</td>
            <td>None</td>
            <td>116</td>
            <td>6.9</td>
            <td>13383</td>
        </tr>
        <tr>
            <td>10198837</td>
            <td>Zero Tolerance</td>
            <td>Noll tolerans</td>
            <td>1999</td>
            <td>None</td>
            <td>108</td>
            <td>6.4</td>
            <td>3288</td>
        </tr>
        <tr>
            <td>10283693</td>
            <td>Zero Woman: Red Handcuffs</td>
            <td>Zeroka no onna: Akai wappa</td>
            <td>1974</td>
            <td>None</td>
            <td>88</td>
            <td>6.6</td>
            <td>783</td>
        </tr>
        <tr>
            <td>10365960</td>
            <td>Zero Day</td>
            <td>None</td>
            <td>2002</td>
            <td>None</td>
            <td>92</td>
            <td>7.2</td>
            <td>3840</td>
        </tr>
        <tr>
            <td>10421090</td>
            <td>Zerophilia</td>
            <td>None</td>
            <td>2005</td>
            <td>None</td>
            <td>90</td>
            <td>6.2</td>
            <td>2177</td>
        </tr>
        <tr>
            <td>11592292</td>
            <td>Zero 2</td>
            <td>None</td>
            <td>2010</td>
            <td>None</td>
            <td>90</td>
            <td>7.6</td>
            <td>5360</td>
        </tr>
        <tr>
            <td>11790885</td>
            <td>Zero Dark Thirty</td>
            <td>None</td>
            <td>2012</td>
            <td>None</td>
            <td>157</td>
            <td>7.4</td>
            <td>254644</td>
        </tr>
        <tr>
            <td>12294965</td>
            <td>Zero Charisma</td>
            <td>None</td>
            <td>2013</td>
            <td>None</td>
            <td>86</td>
            <td>6.2</td>
            <td>2384</td>
        </tr>
        <tr>
            <td>13576084</td>
            <td>Zero Motivation</td>
            <td>Efes beyahasei enosh</td>
            <td>2014</td>
            <td>None</td>
            <td>97</td>
            <td>7.3</td>
            <td>3688</td>
        </tr>
        <tr>
            <td>15446858</td>
            <td>Zero Days</td>
            <td>None</td>
            <td>2016</td>
            <td>None</td>
            <td>116</td>
            <td>7.8</td>
            <td>8081</td>
        </tr>
        <tr>
            <td>10051221</td>
            <td>Zero Hour!</td>
            <td>None</td>
            <td>1957</td>
            <td>None</td>
            <td>81</td>
            <td>6.6</td>
            <td>1295</td>
        </tr>
        <tr>
            <td>10968712</td>
            <td>Zero. Lilac Lithuania</td>
            <td>Zero. Alyvine Lietuva</td>
            <td>2006</td>
            <td>None</td>
            <td>82</td>
            <td>7.5</td>
            <td>2147</td>
        </tr>
        <tr>
            <td>11297858</td>
            <td>Zero: An Investigation Into 9/11</td>
            <td>None</td>
            <td>2008</td>
            <td>None</td>
            <td>104</td>
            <td>7.8</td>
            <td>650</td>
        </tr>
        <tr>
            <td>11519661</td>
            <td>Zero</td>
            <td>None</td>
            <td>2009</td>
            <td>None</td>
            <td>110</td>
            <td>6.8</td>
            <td>555</td>
        </tr>
        <tr>
            <td>12323372</td>
            <td>Zero Point</td>
            <td>Nullpunkt</td>
            <td>2014</td>
            <td>None</td>
            <td>115</td>
            <td>7.4</td>
            <td>877</td>
        </tr>
        <tr>
            <td>16194850</td>
            <td>Zero 3</td>
            <td>None</td>
            <td>2017</td>
            <td>None</td>
            <td>88</td>
            <td>6.6</td>
            <td>1094</td>
        </tr>
    </tbody>
</table>



<br><br><br>

---

**Example:** Retrieve those movies from the `movie` table whose title is 4 letters long and ends with the letter `'e'`.

---


```sql
%%sql

SELECT
    *
FROM
    movies
WHERE
    title LIKE '___e'
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb_dsci513
    71 rows affected.
    




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
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>10043539</td>
            <td>Five</td>
            <td>None</td>
            <td>1951</td>
            <td>None</td>
            <td>93</td>
            <td>6.3</td>
            <td>1068</td>
        </tr>
        <tr>
            <td>10064694</td>
            <td>More</td>
            <td>None</td>
            <td>1969</td>
            <td>None</td>
            <td>112</td>
            <td>6.5</td>
            <td>2155</td>
        </tr>
        <tr>
            <td>10066500</td>
            <td>Hope</td>
            <td>Umut</td>
            <td>1970</td>
            <td>None</td>
            <td>100</td>
            <td>8.2</td>
            <td>2770</td>
        </tr>
        <tr>
            <td>10067814</td>
            <td>Love</td>
            <td>Szerelem</td>
            <td>1971</td>
            <td>None</td>
            <td>88</td>
            <td>7.9</td>
            <td>1582</td>
        </tr>
        <tr>
            <td>10068306</td>
            <td>Bone</td>
            <td>None</td>
            <td>1972</td>
            <td>None</td>
            <td>95</td>
            <td>6.8</td>
            <td>905</td>
        </tr>
        <tr>
            <td>10069158</td>
            <td>Rage</td>
            <td>None</td>
            <td>1972</td>
            <td>None</td>
            <td>100</td>
            <td>6.3</td>
            <td>765</td>
        </tr>
        <tr>
            <td>10071803</td>
            <td>Mame</td>
            <td>None</td>
            <td>1974</td>
            <td>None</td>
            <td>132</td>
            <td>6.1</td>
            <td>2490</td>
        </tr>
        <tr>
            <td>10080716</td>
            <td>Fame</td>
            <td>None</td>
            <td>1980</td>
            <td>None</td>
            <td>134</td>
            <td>6.6</td>
            <td>18864</td>
        </tr>
        <tr>
            <td>10087182</td>
            <td>Dune</td>
            <td>None</td>
            <td>1984</td>
            <td>None</td>
            <td>137</td>
            <td>6.5</td>
            <td>113255</td>
        </tr>
        <tr>
            <td>10088930</td>
            <td>Clue</td>
            <td>None</td>
            <td>1985</td>
            <td>None</td>
            <td>94</td>
            <td>7.3</td>
            <td>71433</td>
        </tr>
        <tr>
            <td>10106438</td>
            <td>Blue</td>
            <td>None</td>
            <td>1993</td>
            <td>None</td>
            <td>79</td>
            <td>7.3</td>
            <td>1778</td>
        </tr>
        <tr>
            <td>10112431</td>
            <td>Babe</td>
            <td>None</td>
            <td>1995</td>
            <td>None</td>
            <td>91</td>
            <td>6.7</td>
            <td>110464</td>
        </tr>
        <tr>
            <td>10114323</td>
            <td>Safe</td>
            <td>None</td>
            <td>1995</td>
            <td>None</td>
            <td>119</td>
            <td>7.2</td>
            <td>11475</td>
        </tr>
        <tr>
            <td>10116308</td>
            <td>Fire</td>
            <td>None</td>
            <td>1996</td>
            <td>None</td>
            <td>108</td>
            <td>7.2</td>
            <td>5386</td>
        </tr>
        <tr>
            <td>10116722</td>
            <td>Jude</td>
            <td>None</td>
            <td>1996</td>
            <td>None</td>
            <td>123</td>
            <td>6.9</td>
            <td>9547</td>
        </tr>
        <tr>
            <td>10123948</td>
            <td>Cure</td>
            <td>None</td>
            <td>1997</td>
            <td>None</td>
            <td>111</td>
            <td>7.4</td>
            <td>8963</td>
        </tr>
        <tr>
            <td>10123755</td>
            <td>Cube</td>
            <td>None</td>
            <td>1997</td>
            <td>None</td>
            <td>90</td>
            <td>7.2</td>
            <td>195849</td>
        </tr>
        <tr>
            <td>10123964</td>
            <td>Life</td>
            <td>None</td>
            <td>1999</td>
            <td>None</td>
            <td>108</td>
            <td>6.8</td>
            <td>41588</td>
        </tr>
        <tr>
            <td>10188568</td>
            <td>Rage</td>
            <td>Do koske</td>
            <td>1997</td>
            <td>None</td>
            <td>97</td>
            <td>6.9</td>
            <td>1623</td>
        </tr>
        <tr>
            <td>10217019</td>
            <td>Sade</td>
            <td>None</td>
            <td>2000</td>
            <td>None</td>
            <td>100</td>
            <td>6.2</td>
            <td>1609</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">71 rows, truncated to displaylimit of 20</span>



<br><br><br>

---

**Example:** Retrieve those movies from the `movie` table whose title contains the character `'%'`.

---

We can specify an escape character using the keyword `ESCAPE` that tells SQL to not interpret a `%` or `_` that immediately follows it:


```sql
%%sql

SELECT
    *
FROM
    movies
WHERE
    title LIKE '%$%%' ESCAPE '$'
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb_dsci513
    3 rows affected.
    




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
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>10487092</td>
            <td>Who the #$&amp;% Is Jackson Pollock?</td>
            <td>None</td>
            <td>2006</td>
            <td>None</td>
            <td>74</td>
            <td>7.0</td>
            <td>1134</td>
        </tr>
        <tr>
            <td>12662228</td>
            <td>10%: What Makes a Hero?</td>
            <td>None</td>
            <td>2013</td>
            <td>None</td>
            <td>88</td>
            <td>6.8</td>
            <td>543</td>
        </tr>
        <tr>
            <td>11869226</td>
            <td>100% Love</td>
            <td>None</td>
            <td>2011</td>
            <td>None</td>
            <td>141</td>
            <td>7.0</td>
            <td>2369</td>
        </tr>
    </tbody>
</table>



<br><br><br>

### `IN`

Sometimes we want to check whether a column value matches any one of the items in a list. We can express this with an `OR` operator:

```sql
SELECT
    column1, column2
FROM
    table1
WHERE
    column1 = value1
    OR
    column1 = value2
    OR
    column1 = value3
;
```

This can be rewritten more succinctly using the `IN` operator:

```sql
SELECT
    column1, column2
FROM
    table1
WHERE
    column1 [NOT] IN (value1, value2, value3)
;
```

<br><br><br>

---

**Example:** Retrieve rows from the `movie` table that correspond to the movies `'Donnie Brasco'`, `'The Usual Suspects'`, `'Schindler''s List'`, `'Shutter Island'`, `'A Beautiful Mind'`.

---


```sql
%%sql

SELECT
    *
FROM
    movies
WHERE
    title IN ('Donnie Brasco',
              'The Usual Suspects',
              'Schindler''s List',
              'Shutter Island',
              'A Beautiful Mind'
               )
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb_dsci513
    5 rows affected.
    




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
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>10108052</td>
            <td>Schindler&#x27;s List</td>
            <td>None</td>
            <td>1993</td>
            <td>None</td>
            <td>195</td>
            <td>8.9</td>
            <td>1110590</td>
        </tr>
        <tr>
            <td>10114814</td>
            <td>The Usual Suspects</td>
            <td>None</td>
            <td>1995</td>
            <td>None</td>
            <td>106</td>
            <td>8.5</td>
            <td>922333</td>
        </tr>
        <tr>
            <td>10119008</td>
            <td>Donnie Brasco</td>
            <td>None</td>
            <td>1997</td>
            <td>None</td>
            <td>127</td>
            <td>7.7</td>
            <td>258120</td>
        </tr>
        <tr>
            <td>10268978</td>
            <td>A Beautiful Mind</td>
            <td>None</td>
            <td>2001</td>
            <td>None</td>
            <td>135</td>
            <td>8.2</td>
            <td>784095</td>
        </tr>
        <tr>
            <td>11130884</td>
            <td>Shutter Island</td>
            <td>None</td>
            <td>2010</td>
            <td>None</td>
            <td>138</td>
            <td>8.1</td>
            <td>1027318</td>
        </tr>
    </tbody>
</table>



<br><br><br>

### `BETWEEN`

The `BETWEEN` keyword is helpful for when we want to select a range of values, and it can be used for number, character and datetime ranges:

```sql
SELECT
    column1, column2
FROM
    table1
WHERE
    column1 [NOT] BETWEEN value1 AND value2
;
```

> **Note:** `BETWEEN` is **inclusive** of both ends of the interval.

We can try it out using a `SELECT` statement without any tables:


```sql
%%sql

SELECT 
    5 BETWEEN 1 AND 10,
    DATE '2021-11-01' BETWEEN DATE '2021-01-01' AND '2021-11-10',
    'w' BETWEEN 'e' AND 'm';
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb_dsci513
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>?column?</th>
            <th>?column?_1</th>
            <th>?column?_2</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>True</td>
            <td>True</td>
            <td>False</td>
        </tr>
    </tbody>
</table>



<br><br><br>

---

**Example:** Retrieve the name, production year and rating of the top 5 movies from the `movie` table that are produced between 2018 and 2020, and have a rating of at least 8.5 with at least 100000 votes. Sort the results in descending order based on ratings.

---


```sql
%%sql

SELECT
    title, start_year, rating
FROM
    movies
WHERE
    start_year BETWEEN 2018 AND 2020
    AND
    rating >= 8.5
    AND
    nvotes >= 100000
ORDER BY
    rating DESC
LIMIT
    5
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb_dsci513
    5 rows affected.
    




<table>
    <thead>
        <tr>
            <th>title</th>
            <th>start_year</th>
            <th>rating</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Once Upon a Time... in Hollywood</td>
            <td>2019</td>
            <td>8.0</td>
        </tr>
        <tr>
            <td>Toy Story 4</td>
            <td>2019</td>
            <td>8.0</td>
        </tr>
        <tr>
            <td>Bohemian Rhapsody</td>
            <td>2018</td>
            <td>8.0</td>
        </tr>
        <tr>
            <td>Green Book</td>
            <td>2018</td>
            <td>8.2</td>
        </tr>
        <tr>
            <td>Spider-Man: Into the Spider-Verse</td>
            <td>2018</td>
            <td>8.4</td>
        </tr>
    </tbody>
</table>



<br><br><br>

### `IS NULL`

Trying to find `NULL` values using `WHERE column = NULL` fails. This is because a `NULL` value is by definition not known and _could be anything_, so it's not necessarily equal to another `NULL`. To find `NULL` values in a column, we can use `IS NULL`:

```sql
SELECT
    column1, column2
FROM
    table1
WHERE
    column1 IS [NOT] NULL
;
```

<br><br><br>

---

**Example:** Find movies the `movie` whose `orig_title` is different from that listed in the `title` column.

---


```sql
%%sql

SELECT
    *
FROM
    movies
WHERE
    orig_title IS NOT NULL
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb_dsci513
    8270 rows affected.
    




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
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>10041719</td>
            <td>Orpheus</td>
            <td>Orphée</td>
            <td>1950</td>
            <td>None</td>
            <td>95</td>
            <td>8.0</td>
            <td>9346</td>
        </tr>
        <tr>
            <td>10041931</td>
            <td>Stromboli</td>
            <td>Stromboli, terra di Dio</td>
            <td>1950</td>
            <td>None</td>
            <td>107</td>
            <td>7.3</td>
            <td>5239</td>
        </tr>
        <tr>
            <td>10042355</td>
            <td>Story of a Love Affair</td>
            <td>Cronaca di un amore</td>
            <td>1950</td>
            <td>None</td>
            <td>98</td>
            <td>7.1</td>
            <td>2209</td>
        </tr>
        <tr>
            <td>10042619</td>
            <td>Diary of a Country Priest</td>
            <td>Journal d&#x27;un curé de campagne</td>
            <td>1951</td>
            <td>None</td>
            <td>115</td>
            <td>8.0</td>
            <td>8621</td>
        </tr>
        <tr>
            <td>10042692</td>
            <td>Variety Lights</td>
            <td>Luci del varietà</td>
            <td>1950</td>
            <td>None</td>
            <td>97</td>
            <td>7.1</td>
            <td>2416</td>
        </tr>
        <tr>
            <td>10042804</td>
            <td>The Young and the Damned</td>
            <td>Los olvidados</td>
            <td>1950</td>
            <td>None</td>
            <td>85</td>
            <td>8.3</td>
            <td>16453</td>
        </tr>
        <tr>
            <td>10042810</td>
            <td>Operation Disaster</td>
            <td>Morning Departure</td>
            <td>1950</td>
            <td>None</td>
            <td>102</td>
            <td>7.0</td>
            <td>668</td>
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
        </tr>
        <tr>
            <td>10042906</td>
            <td>La Ronde</td>
            <td>La ronde</td>
            <td>1950</td>
            <td>None</td>
            <td>93</td>
            <td>7.6</td>
            <td>4456</td>
        </tr>
        <tr>
            <td>10043048</td>
            <td>To Joy</td>
            <td>Till glädje</td>
            <td>1950</td>
            <td>None</td>
            <td>98</td>
            <td>7.2</td>
            <td>2109</td>
        </tr>
        <tr>
            <td>10043303</td>
            <td>The Red Inn</td>
            <td>L&#x27;auberge rouge</td>
            <td>1951</td>
            <td>None</td>
            <td>98</td>
            <td>7.4</td>
            <td>1201</td>
        </tr>
        <tr>
            <td>10043313</td>
            <td>Early Summer</td>
            <td>Bakushû</td>
            <td>1951</td>
            <td>None</td>
            <td>125</td>
            <td>8.2</td>
            <td>6349</td>
        </tr>
        <tr>
            <td>10043386</td>
            <td>Casque d&#x27;Or</td>
            <td>Casque d&#x27;or</td>
            <td>1952</td>
            <td>None</td>
            <td>96</td>
            <td>7.7</td>
            <td>4414</td>
        </tr>
        <tr>
            <td>10043411</td>
            <td>The Emperor and the Golem</td>
            <td>Císaruv pekar - Pekaruv císar</td>
            <td>1952</td>
            <td>None</td>
            <td>144</td>
            <td>8.1</td>
            <td>1069</td>
        </tr>
        <tr>
            <td>10043511</td>
            <td>Europe &#x27;51</td>
            <td>Europa &#x27;51</td>
            <td>1952</td>
            <td>None</td>
            <td>118</td>
            <td>7.5</td>
            <td>2874</td>
        </tr>
        <tr>
            <td>10043567</td>
            <td>Miss Julie</td>
            <td>Fröken Julie</td>
            <td>1951</td>
            <td>None</td>
            <td>90</td>
            <td>7.3</td>
            <td>1499</td>
        </tr>
        <tr>
            <td>10043614</td>
            <td>The Idiot</td>
            <td>Hakuchi</td>
            <td>1951</td>
            <td>None</td>
            <td>166</td>
            <td>7.3</td>
            <td>4110</td>
        </tr>
        <tr>
            <td>10043652</td>
            <td>One Summer of Happiness</td>
            <td>Hon dansade en sommar</td>
            <td>1951</td>
            <td>None</td>
            <td>103</td>
            <td>6.9</td>
            <td>623</td>
        </tr>
        <tr>
            <td>10043668</td>
            <td>I&#x27;ll Never Forget You</td>
            <td>The House in the Square</td>
            <td>1951</td>
            <td>None</td>
            <td>90</td>
            <td>7.1</td>
            <td>716</td>
        </tr>
        <tr>
            <td>10043686</td>
            <td>Forbidden Games</td>
            <td>Jeux interdits</td>
            <td>1952</td>
            <td>None</td>
            <td>86</td>
            <td>8.1</td>
            <td>10142</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">8270 rows, truncated to displaylimit of 20</span>



<br><br><br>

## Column Aliases with `AS`

In SQL, we are not required to use the same column and table names in the schema. We can create **aliases** for a column or a table with the following syntax:

```sql
SELECT
    column1 [AS] c1,
    column2 [AS] c2
FROM
    table1 [AS] t1
;
```

Note that the keyword `AS` is optional. I usually choose to use it because it makes the query more readable.

We will use table aliases a lot when we work on SQL joins in the upcoming lectures!


```sql
%%sql

SELECT
    title AS movieTitle,
    orig_title AS "oringinal Title",
    runtime AS Duration
FROM
    movies;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb
    26058 rows affected.
    




<table>
    <thead>
        <tr>
            <th>movietitle</th>
            <th>oringinal Title</th>
            <th>duration</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Kate &amp; Leopold</td>
            <td>None</td>
            <td>118</td>
        </tr>
        <tr>
            <td>Mister 880</td>
            <td>None</td>
            <td>90</td>
        </tr>
        <tr>
            <td>Black Hand</td>
            <td>None</td>
            <td>92</td>
        </tr>
        <tr>
            <td>Francis</td>
            <td>None</td>
            <td>91</td>
        </tr>
        <tr>
            <td>Orpheus</td>
            <td>Orphée</td>
            <td>95</td>
        </tr>
        <tr>
            <td>Stromboli</td>
            <td>Stromboli, terra di Dio</td>
            <td>107</td>
        </tr>
        <tr>
            <td>Woman in Hiding</td>
            <td>None</td>
            <td>92</td>
        </tr>
        <tr>
            <td>Abbott and Costello in the Foreign Legion</td>
            <td>None</td>
            <td>80</td>
        </tr>
        <tr>
            <td>Annie Get Your Gun</td>
            <td>None</td>
            <td>107</td>
        </tr>
        <tr>
            <td>Armored Car Robbery</td>
            <td>None</td>
            <td>67</td>
        </tr>
        <tr>
            <td>The Asphalt Jungle</td>
            <td>None</td>
            <td>112</td>
        </tr>
        <tr>
            <td>Atom Man vs. Superman</td>
            <td>None</td>
            <td>252</td>
        </tr>
        <tr>
            <td>Backfire</td>
            <td>None</td>
            <td>91</td>
        </tr>
        <tr>
            <td>The Baron of Arizona</td>
            <td>None</td>
            <td>97</td>
        </tr>
        <tr>
            <td>The Big Lift</td>
            <td>None</td>
            <td>120</td>
        </tr>
        <tr>
            <td>The Black Rose</td>
            <td>None</td>
            <td>120</td>
        </tr>
        <tr>
            <td>The Blue Lamp</td>
            <td>None</td>
            <td>84</td>
        </tr>
        <tr>
            <td>Borderline</td>
            <td>None</td>
            <td>88</td>
        </tr>
        <tr>
            <td>Born to Be Bad</td>
            <td>None</td>
            <td>94</td>
        </tr>
        <tr>
            <td>Branded</td>
            <td>None</td>
            <td>104</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">26058 rows, truncated to displaylimit of 20</span>



Note that we've used a column alias with a space in its name. This is generally not a good practice, but if you absolutely need to do it, in Postgres you should enclose the alias in double quotes, e.g. `"alias name"`. A situation where double quotes are necessary is when you want to name a column with a word that is reserved keyword in Postgres, e.g. `"COUNT"`.

<br><br><br>

### Order of execution/processing in SQL


If you try running the following cell, you'll notice that you **cannot** use column aliases in the `WHERE` clause, since it is evaluated by SQL before setting aliases:


```sql
%%sql

SELECT
    title AS movieTitle,
    orig_title AS "oringinal Title",
    runtime AS Duration
FROM
    movies
WHERE
    Duration > 100
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb
    (psycopg2.errors.UndefinedColumn) column "duration" does not exist
    LINE 8:     Duration > 100
                ^
    
    [SQL: SELECT
        title AS movieTitle,
        orig_title AS "oringinal Title",
        runtime AS Duration
    FROM
        movies
    WHERE
        Duration > 100
    ;]
    (Background on this error at: https://sqlalche.me/e/14/f405)
    

SQL-based database engines have a particular order by which they execute different clauses in your SQL query:

<br>

**Order of execution/processing in SQL:**

```sql
FROM and JOIN
      |
    WHERE
      |
  GROUP BY
      |
    HAVING
      |
    SELECT
      |
   DISTINCT
      |
   ORDER BY
      |
    LIMIT
```

<br><br>

Contrast this with the **order of SQL clauses in a statement:**

```sql
    SELECT
      |
     FROM
      |
     JOIN
      |
    WHERE
      |
  GROUP BY
      |
    HAVING
      |
   ORDER BY
      |
    LIMIT
```

<br><br><br>

## Derived columns

Derived columns in SQL are columns that are the result of doing operations on existing columns of a table.

For example, suppose that we want to convert the `runtime` column of our table `movies` from minutes to hours. We can do that by manipulating the `runtime` column right in the `SELECT` statement:


```sql
%%sql

SELECT
    title,
    runtime / 60.
FROM
    movies;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb
    26058 rows affected.
    




<table>
    <thead>
        <tr>
            <th>title</th>
            <th>?column?</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Kate &amp; Leopold</td>
            <td>1.9666666666666667</td>
        </tr>
        <tr>
            <td>Mister 880</td>
            <td>1.5000000000000000</td>
        </tr>
        <tr>
            <td>Black Hand</td>
            <td>1.5333333333333333</td>
        </tr>
        <tr>
            <td>Francis</td>
            <td>1.5166666666666667</td>
        </tr>
        <tr>
            <td>Orpheus</td>
            <td>1.5833333333333333</td>
        </tr>
        <tr>
            <td>Stromboli</td>
            <td>1.7833333333333333</td>
        </tr>
        <tr>
            <td>Woman in Hiding</td>
            <td>1.5333333333333333</td>
        </tr>
        <tr>
            <td>Abbott and Costello in the Foreign Legion</td>
            <td>1.3333333333333333</td>
        </tr>
        <tr>
            <td>Annie Get Your Gun</td>
            <td>1.7833333333333333</td>
        </tr>
        <tr>
            <td>Armored Car Robbery</td>
            <td>1.1166666666666667</td>
        </tr>
        <tr>
            <td>The Asphalt Jungle</td>
            <td>1.8666666666666667</td>
        </tr>
        <tr>
            <td>Atom Man vs. Superman</td>
            <td>4.2000000000000000</td>
        </tr>
        <tr>
            <td>Backfire</td>
            <td>1.5166666666666667</td>
        </tr>
        <tr>
            <td>The Baron of Arizona</td>
            <td>1.6166666666666667</td>
        </tr>
        <tr>
            <td>The Big Lift</td>
            <td>2.0000000000000000</td>
        </tr>
        <tr>
            <td>The Black Rose</td>
            <td>2.0000000000000000</td>
        </tr>
        <tr>
            <td>The Blue Lamp</td>
            <td>1.4000000000000000</td>
        </tr>
        <tr>
            <td>Borderline</td>
            <td>1.4666666666666667</td>
        </tr>
        <tr>
            <td>Born to Be Bad</td>
            <td>1.5666666666666667</td>
        </tr>
        <tr>
            <td>Branded</td>
            <td>1.7333333333333333</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">26058 rows, truncated to displaylimit of 20</span>



> Note that I've written `60.` with the decimal point on purpose. If you divide by `60` instead, SQL assumes that the result of this operation should also be an integer (given that the column `runtime` is also of type integer), and will return truncated integer values instead of floats.

<br><br><br>

SQL doesn't know what to call the derived column, and by default you will see `?column?` as the column name. We can use an alias to name the new derived column:


```sql
%%sql

SELECT
    title,
    runtime / 60. AS runtime_hours
FROM
    movies;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb
    26058 rows affected.
    




<table>
    <thead>
        <tr>
            <th>title</th>
            <th>runtime_hours</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Kate &amp; Leopold</td>
            <td>1.9666666666666667</td>
        </tr>
        <tr>
            <td>Mister 880</td>
            <td>1.5000000000000000</td>
        </tr>
        <tr>
            <td>Black Hand</td>
            <td>1.5333333333333333</td>
        </tr>
        <tr>
            <td>Francis</td>
            <td>1.5166666666666667</td>
        </tr>
        <tr>
            <td>Orpheus</td>
            <td>1.5833333333333333</td>
        </tr>
        <tr>
            <td>Stromboli</td>
            <td>1.7833333333333333</td>
        </tr>
        <tr>
            <td>Woman in Hiding</td>
            <td>1.5333333333333333</td>
        </tr>
        <tr>
            <td>Abbott and Costello in the Foreign Legion</td>
            <td>1.3333333333333333</td>
        </tr>
        <tr>
            <td>Annie Get Your Gun</td>
            <td>1.7833333333333333</td>
        </tr>
        <tr>
            <td>Armored Car Robbery</td>
            <td>1.1166666666666667</td>
        </tr>
        <tr>
            <td>The Asphalt Jungle</td>
            <td>1.8666666666666667</td>
        </tr>
        <tr>
            <td>Atom Man vs. Superman</td>
            <td>4.2000000000000000</td>
        </tr>
        <tr>
            <td>Backfire</td>
            <td>1.5166666666666667</td>
        </tr>
        <tr>
            <td>The Baron of Arizona</td>
            <td>1.6166666666666667</td>
        </tr>
        <tr>
            <td>The Big Lift</td>
            <td>2.0000000000000000</td>
        </tr>
        <tr>
            <td>The Black Rose</td>
            <td>2.0000000000000000</td>
        </tr>
        <tr>
            <td>The Blue Lamp</td>
            <td>1.4000000000000000</td>
        </tr>
        <tr>
            <td>Borderline</td>
            <td>1.4666666666666667</td>
        </tr>
        <tr>
            <td>Born to Be Bad</td>
            <td>1.5666666666666667</td>
        </tr>
        <tr>
            <td>Branded</td>
            <td>1.7333333333333333</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">26058 rows, truncated to displaylimit of 20</span>



Remember I mentioned that the `SELECT` statement is powerful, but not dangerous? Derived columns returned by Postgres are not saved anywhere, nor do they change existing columns.

<br><br><br>

---

**Example:** Using table `names` from the `imdb` database, find the age of all actors/actresses who are still alive. Who is the youngest person alive listed in the table?

---


```sql
%%sql

SELECT
    name,
    2022 - birth_year AS age
FROM
    names
WHERE
    birth_year IS NOT NULL
    AND
    death_year IS NULL
ORDER BY
    age
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb
    35413 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>age</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Blue Ivy Carter</td>
            <td>10</td>
        </tr>
        <tr>
            <td>Julia Butters</td>
            <td>13</td>
        </tr>
        <tr>
            <td>Run-yin Bai</td>
            <td>13</td>
        </tr>
        <tr>
            <td>Mckenna Grace</td>
            <td>16</td>
        </tr>
        <tr>
            <td>Timothy Radford</td>
            <td>16</td>
        </tr>
        <tr>
            <td>Momone Shinokawa</td>
            <td>16</td>
        </tr>
        <tr>
            <td>Jacob Tremblay</td>
            <td>16</td>
        </tr>
        <tr>
            <td>Rina Endô</td>
            <td>17</td>
        </tr>
        <tr>
            <td>Eun-hyung Jo</td>
            <td>17</td>
        </tr>
        <tr>
            <td>Sunny Suljic</td>
            <td>17</td>
        </tr>
        <tr>
            <td>Teo Briones</td>
            <td>17</td>
        </tr>
        <tr>
            <td>Merve Ates</td>
            <td>17</td>
        </tr>
        <tr>
            <td>Kristina Pimenova</td>
            <td>17</td>
        </tr>
        <tr>
            <td>Alexis Neblett</td>
            <td>17</td>
        </tr>
        <tr>
            <td>Ella Anderson</td>
            <td>17</td>
        </tr>
        <tr>
            <td>Noah Jupe</td>
            <td>17</td>
        </tr>
        <tr>
            <td>Towa Araki</td>
            <td>17</td>
        </tr>
        <tr>
            <td>Lulu Wilson</td>
            <td>17</td>
        </tr>
        <tr>
            <td>Farrah Mackenzie</td>
            <td>17</td>
        </tr>
        <tr>
            <td>Fantine Harduin</td>
            <td>17</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">35413 rows, truncated to displaylimit of 20</span>



<br><br><br>

## Conditionals with `CASE`

The `CASE` structure is very useful in SQL: it enables us to treat a column differently based on the values in each row. Here is the syntax:

```sql
SELECT
    column1,
    CASE
        WHEN condition THEN expression
        WHEN condition THEN expression
        .
        .
        .
        ELSE expression
    END,
    column3,
    column4
FROM
    table1
;
```

For example, let's say we want to retrieve the name of movies and also want to have a column in the results that in each row has the value "long" if a movie is over 90 minutes long, "normal" if it's between 30 to 90 minutes, and "short" if it's under 30 minutes:


```sql
%%sql

SELECT
    title,
    runtime,
    CASE
        WHEN runtime > 90 THEN 'long'
        WHEN runtime BETWEEN 30 AND 90 THEN 'normal'
        ELSE 'short'
    END AS duration
FROM
    movies
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb_dsci513
    26058 rows affected.
    




<table>
    <thead>
        <tr>
            <th>title</th>
            <th>runtime</th>
            <th>duration</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Kate &amp; Leopold</td>
            <td>118</td>
            <td>long</td>
        </tr>
        <tr>
            <td>Mister 880</td>
            <td>90</td>
            <td>normal</td>
        </tr>
        <tr>
            <td>Black Hand</td>
            <td>92</td>
            <td>long</td>
        </tr>
        <tr>
            <td>Francis</td>
            <td>91</td>
            <td>long</td>
        </tr>
        <tr>
            <td>Orpheus</td>
            <td>95</td>
            <td>long</td>
        </tr>
        <tr>
            <td>Stromboli</td>
            <td>107</td>
            <td>long</td>
        </tr>
        <tr>
            <td>Woman in Hiding</td>
            <td>92</td>
            <td>long</td>
        </tr>
        <tr>
            <td>Abbott and Costello in the Foreign Legion</td>
            <td>80</td>
            <td>normal</td>
        </tr>
        <tr>
            <td>Annie Get Your Gun</td>
            <td>107</td>
            <td>long</td>
        </tr>
        <tr>
            <td>Armored Car Robbery</td>
            <td>67</td>
            <td>normal</td>
        </tr>
        <tr>
            <td>The Asphalt Jungle</td>
            <td>112</td>
            <td>long</td>
        </tr>
        <tr>
            <td>Atom Man vs. Superman</td>
            <td>252</td>
            <td>long</td>
        </tr>
        <tr>
            <td>Backfire</td>
            <td>91</td>
            <td>long</td>
        </tr>
        <tr>
            <td>The Baron of Arizona</td>
            <td>97</td>
            <td>long</td>
        </tr>
        <tr>
            <td>The Big Lift</td>
            <td>120</td>
            <td>long</td>
        </tr>
        <tr>
            <td>The Black Rose</td>
            <td>120</td>
            <td>long</td>
        </tr>
        <tr>
            <td>The Blue Lamp</td>
            <td>84</td>
            <td>normal</td>
        </tr>
        <tr>
            <td>Borderline</td>
            <td>88</td>
            <td>normal</td>
        </tr>
        <tr>
            <td>Born to Be Bad</td>
            <td>94</td>
            <td>long</td>
        </tr>
        <tr>
            <td>Branded</td>
            <td>104</td>
            <td>long</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">26058 rows, truncated to displaylimit of 20</span>



<br><br><br>

## Functions & operators

### Math

We've just seen how arithmetic operators (i.e. `+`, `-`, `*`, `/`) can be used to make derived columns. Like other programming languages, PostgreSQL comes built-in with the most common mathematical operators (for a full list of operators see the documentation of Postgres [here](https://www.postgresql.org/docs/9.0/functions-math.html)):

**Operators**:

| Operator   | Description       
|------------|--------------------
| `+`        | addition           
| `-`        | subtraction        
| `*`        | multiplication     
| `/`        | division           
| `%`        | modulo (remainder) 
| `^`        | exponentiation     
| `@`        | absolute value     

**Functions**

| Description                               | Example             | 
|---------------------------|-------------------------------------------
| absolute value                            | `abs(-17.4)`        |
| smallest integer not less than argument   | `ceil(-42.8)`       | 
| exponential                               | `exp(1.0)`          |
| largest integer not greater than argument | `floor(-42.8)`      | 
| natural logarithm                         | `ln(2.0)`           |
| logarithm to base b                       | `log(2.0, 64.0)`    | 
| "π" constant                              | `pi()`              | 
| a raised to the power of b                | `power(9.0, 3.0)`   | 
| round to s decimal places                 | `round(42.4382, 2)` |
| square root                               | `sqrt(2.0)`         |


> Note the order of evaluation for different operators:
> 1. arithmetic operators (e.g. `+`, `*`)
> 2. comparison operators (e.g. `>`, `<=`)
> 3. logical operators (e.g. `AND`, `OR`)


```sql
%%sql

SELECT
    25 * 2,
    ABS(-2^10),
    ROUND(23.24545, 2),
    SQRT(25),
    PI()
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb_dsci513
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>?column?</th>
            <th>abs</th>
            <th>round</th>
            <th>sqrt</th>
            <th>pi</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>50</td>
            <td>1024.0</td>
            <td>23.25</td>
            <td>5.0</td>
            <td>3.141592653589793</td>
        </tr>
    </tbody>
</table>



> **Note:** The `ROUND()` function only works with the `NUMERIC` (or equivalently `DECIMAL`) data type.

<br><br><br>

### Strings

[documentation](https://www.postgresql.org/docs/9.0/functions-string.html)

| Function                                           | Description                                          |
|---------------------------------------------------|------------------------------------------------------|
| `string \|\| string`                                | String concatenation                                | 
| `string \|\| non-string or non-string \|\| string`  | String concatenation with one non-string input      |
| `char_length(string) or character_length(string)`   | Number of characters in string                      | 
| `lower(string)`                                     | Convert string to lower case                        | 
| `position(substring in string)`                     | Location of specified substring                     |
| `substring(string)`                                 | Extract substring                                   | 
| `upper(string)`                                     | Convert string to upper case                        |
| `length(string)`             | Number of characters in string                      |


One of the useful string operators is `||`, or the concatenation operators. It can be used with multiple strings, or non-string values.

---

**Example:** Using table `movies` from the `imdb` database, print: `"<title>" is <hours> hours long, and rated <rating> / 10.`. Round the hour to 1 decimal point.

---


```sql
%%sql

SELECT
    '"' || title || '"'
    || ' is ' || ROUND(runtime / 60., 1)
    || ' hours long, and rated '
    || rating || ' / 10.'
FROM
    movies
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb_dsci513
    26058 rows affected.
    




<table>
    <thead>
        <tr>
            <th>?column?</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>&quot;Kate &amp; Leopold&quot; is 2.0 hours long, and rated 6.4 / 10.</td>
        </tr>
        <tr>
            <td>&quot;Mister 880&quot; is 1.5 hours long, and rated 7.1 / 10.</td>
        </tr>
        <tr>
            <td>&quot;Black Hand&quot; is 1.5 hours long, and rated 6.4 / 10.</td>
        </tr>
        <tr>
            <td>&quot;Francis&quot; is 1.5 hours long, and rated 6.4 / 10.</td>
        </tr>
        <tr>
            <td>&quot;Orpheus&quot; is 1.6 hours long, and rated 8 / 10.</td>
        </tr>
        <tr>
            <td>&quot;Stromboli&quot; is 1.8 hours long, and rated 7.3 / 10.</td>
        </tr>
        <tr>
            <td>&quot;Woman in Hiding&quot; is 1.5 hours long, and rated 6.9 / 10.</td>
        </tr>
        <tr>
            <td>&quot;Abbott and Costello in the Foreign Legion&quot; is 1.3 hours long, and rated 6.6 / 10.</td>
        </tr>
        <tr>
            <td>&quot;Annie Get Your Gun&quot; is 1.8 hours long, and rated 6.9 / 10.</td>
        </tr>
        <tr>
            <td>&quot;Armored Car Robbery&quot; is 1.1 hours long, and rated 7 / 10.</td>
        </tr>
        <tr>
            <td>&quot;The Asphalt Jungle&quot; is 1.9 hours long, and rated 7.9 / 10.</td>
        </tr>
        <tr>
            <td>&quot;Atom Man vs. Superman&quot; is 4.2 hours long, and rated 7 / 10.</td>
        </tr>
        <tr>
            <td>&quot;Backfire&quot; is 1.5 hours long, and rated 6.6 / 10.</td>
        </tr>
        <tr>
            <td>&quot;The Baron of Arizona&quot; is 1.6 hours long, and rated 7 / 10.</td>
        </tr>
        <tr>
            <td>&quot;The Big Lift&quot; is 2.0 hours long, and rated 6.5 / 10.</td>
        </tr>
        <tr>
            <td>&quot;The Black Rose&quot; is 2.0 hours long, and rated 6.3 / 10.</td>
        </tr>
        <tr>
            <td>&quot;The Blue Lamp&quot; is 1.4 hours long, and rated 6.9 / 10.</td>
        </tr>
        <tr>
            <td>&quot;Borderline&quot; is 1.5 hours long, and rated 6.1 / 10.</td>
        </tr>
        <tr>
            <td>&quot;Born to Be Bad&quot; is 1.6 hours long, and rated 6.7 / 10.</td>
        </tr>
        <tr>
            <td>&quot;Branded&quot; is 1.7 hours long, and rated 6.7 / 10.</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">26058 rows, truncated to displaylimit of 20</span>



We can use the `SUBSTRING(string FROM pos FOR num)` function to extract parts of a string value, starting with a particular position and continuing for a specified number of characters.

The function `SUBSTR(string, pos, num)` also exists in Postgres which is pretty similar to `SUBSTRING`, but with a different syntax.

<br><br><br>

---

**Example:** Using table `movies` from the `imdb` database, print the `title` column in upper-case letters. Also, create two more columns with the first and last three characters of the title. Name these columns "First 3 characters" and "Last 3 characters".

---


```sql
%%sql

SELECT
    UPPER(title),
    SUBSTRING(title FROM 1 FOR 3) AS "First 3 characters",
    SUBSTR(title, LENGTH(title) - 3, 3) AS "Last 3 characters"
FROM
    movies
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb_dsci513
    26058 rows affected.
    




<table>
    <thead>
        <tr>
            <th>upper</th>
            <th>First 3 characters</th>
            <th>Last 3 characters</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>KATE &amp; LEOPOLD</td>
            <td>Kat</td>
            <td>pol</td>
        </tr>
        <tr>
            <td>MISTER 880</td>
            <td>Mis</td>
            <td> 88</td>
        </tr>
        <tr>
            <td>BLACK HAND</td>
            <td>Bla</td>
            <td>Han</td>
        </tr>
        <tr>
            <td>FRANCIS</td>
            <td>Fra</td>
            <td>nci</td>
        </tr>
        <tr>
            <td>ORPHEUS</td>
            <td>Orp</td>
            <td>heu</td>
        </tr>
        <tr>
            <td>STROMBOLI</td>
            <td>Str</td>
            <td>bol</td>
        </tr>
        <tr>
            <td>WOMAN IN HIDING</td>
            <td>Wom</td>
            <td>din</td>
        </tr>
        <tr>
            <td>ABBOTT AND COSTELLO IN THE FOREIGN LEGION</td>
            <td>Abb</td>
            <td>gio</td>
        </tr>
        <tr>
            <td>ANNIE GET YOUR GUN</td>
            <td>Ann</td>
            <td> Gu</td>
        </tr>
        <tr>
            <td>ARMORED CAR ROBBERY</td>
            <td>Arm</td>
            <td>ber</td>
        </tr>
        <tr>
            <td>THE ASPHALT JUNGLE</td>
            <td>The</td>
            <td>ngl</td>
        </tr>
        <tr>
            <td>ATOM MAN VS. SUPERMAN</td>
            <td>Ato</td>
            <td>rma</td>
        </tr>
        <tr>
            <td>BACKFIRE</td>
            <td>Bac</td>
            <td>fir</td>
        </tr>
        <tr>
            <td>THE BARON OF ARIZONA</td>
            <td>The</td>
            <td>zon</td>
        </tr>
        <tr>
            <td>THE BIG LIFT</td>
            <td>The</td>
            <td>Lif</td>
        </tr>
        <tr>
            <td>THE BLACK ROSE</td>
            <td>The</td>
            <td>Ros</td>
        </tr>
        <tr>
            <td>THE BLUE LAMP</td>
            <td>The</td>
            <td>Lam</td>
        </tr>
        <tr>
            <td>BORDERLINE</td>
            <td>Bor</td>
            <td>lin</td>
        </tr>
        <tr>
            <td>BORN TO BE BAD</td>
            <td>Bor</td>
            <td> Ba</td>
        </tr>
        <tr>
            <td>BRANDED</td>
            <td>Bra</td>
            <td>nde</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">26058 rows, truncated to displaylimit of 20</span>



<br><br><br>

### Datetimes

In addition to `+`, `-`, `*`, and `/` operators, Postgres also provides useful functions for working with datetimes and intervals. Let's look at a few of those here:


```sql
%%sql

SELECT
    CURRENT_DATE,
    NOW(),
    CURRENT_TIMESTAMP(0),
    CURRENT_TIME(0),
    LOCALTIMESTAMP(0),
    LOCALTIME(2)
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb_dsci513
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>current_date</th>
            <th>now</th>
            <th>current_timestamp</th>
            <th>current_time</th>
            <th>localtimestamp</th>
            <th>localtime</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2021-11-18</td>
            <td>2021-11-18 07:39:44.472676-08:00</td>
            <td>2021-11-18 07:39:44-08:00</td>
            <td>07:39:44-08:00</td>
            <td>2021-11-18 07:39:44</td>
            <td>07:39:44.470000</td>
        </tr>
    </tbody>
</table>



<br><br><br>

The argument to `CURRENT_TIMESTAMP()` and other functions above specifies the desired precision for the seconds field.


```sql
%%sql

SELECT
    CURRENT_TIME,
    LOCALTIME,
    LOCALTIMESTAMP
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb_dsci513
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>current_time</th>
            <th>localtime</th>
            <th>localtimestamp</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>07:39:44.756441-08:00</td>
            <td>07:39:44.756441</td>
            <td>2021-11-18 07:39:44.756441</td>
        </tr>
    </tbody>
</table>



<br><br><br>

`NOW()` and `CURRENT_TIMESTAMP` are equivalent, with the latter being SQL-standard.

> Note that both of these functions/variables are timezone-aware.


```sql
%%sql

SELECT
    EXTRACT(hour FROM NOW()),
    EXTRACT(year FROM '2021-11-15 8:00:00'::TIMESTAMP)
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb_dsci513
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>extract</th>
            <th>extract_1</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>7</td>
            <td>2021</td>
        </tr>
    </tbody>
</table>



<br><br><br>


```sql
%%sql

SELECT
    age(NOW(), '1979-01-05'::TIMESTAMP)
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>age</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>16007 days, 2:04:37.906168</td>
        </tr>
    </tbody>
</table>



The `age()` function formats the output nicely in `psql` and pgAdmin, but it unfortunately shows up as the number of days in `ipython-sql`.

<br><br><br>

There are a couple of functions for conversions between datetimes and intervals to and from strings:


```sql
%%sql

SELECT
    to_date('2021', 'YYYY'),
    to_date('05 Dec 2000', 'DD Mon YYYY'),
    to_timestamp('05 Dec 2021', 'DD Mon YYYY')
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>to_date</th>
            <th>to_date_1</th>
            <th>to_timestamp</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2021-01-01</td>
            <td>2000-12-05</td>
            <td>2021-12-05 00:00:00-08:00</td>
        </tr>
    </tbody>
</table>




```sql
%%sql

SELECT
    to_char(current_timestamp, 'Day, DD  HH:MI'),
    to_char(interval '15h 2m 12s', 'HH12:MI:SS')
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>to_char</th>
            <th>to_char_1</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Thursday , 17  02:04</td>
            <td>03:02:12</td>
        </tr>
    </tbody>
</table>



For a full description of various datetime formatting functions and detailed string formatting patterns, see the documentation [here](https://www.postgresql.org/docs/current/functions-formatting.html).

<br><br><br>

---

**Example:** Using table `names` from the `imdb` database, retrieve the name and age of each alive person (according to the table!) in years using the `AGE()` function. You need to convert data types for this.

---


```sql
%%sql

SELECT
    name,
    EXTRACT(
        year FROM AGE(NOW(), to_date(birth_year::varchar, 'YYYY')))
        AS "Age"
FROM
    names
WHERE
    death_year IS NULL
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb
    95083 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>Age</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Brigitte Bardot</td>
            <td>88</td>
        </tr>
        <tr>
            <td>Olivia de Havilland</td>
            <td>106</td>
        </tr>
        <tr>
            <td>Kirk Douglas</td>
            <td>106</td>
        </tr>
        <tr>
            <td>Sophia Loren</td>
            <td>88</td>
        </tr>
        <tr>
            <td>Raquel Welch</td>
            <td>82</td>
        </tr>
        <tr>
            <td>Li Gong</td>
            <td>57</td>
        </tr>
        <tr>
            <td>Armin Mueller-Stahl</td>
            <td>92</td>
        </tr>
        <tr>
            <td>Gérard Pirès</td>
            <td>80</td>
        </tr>
        <tr>
            <td>John Cleese</td>
            <td>83</td>
        </tr>
        <tr>
            <td>Brad Pitt</td>
            <td>59</td>
        </tr>
        <tr>
            <td>Woody Allen</td>
            <td>87</td>
        </tr>
        <tr>
            <td>Gillian Anderson</td>
            <td>54</td>
        </tr>
        <tr>
            <td>Jennifer Aniston</td>
            <td>53</td>
        </tr>
        <tr>
            <td>Patricia Arquette</td>
            <td>54</td>
        </tr>
        <tr>
            <td>Rowan Atkinson</td>
            <td>67</td>
        </tr>
        <tr>
            <td>Dan Aykroyd</td>
            <td>70</td>
        </tr>
        <tr>
            <td>Kevin Bacon</td>
            <td>64</td>
        </tr>
        <tr>
            <td>Fairuza Balk</td>
            <td>48</td>
        </tr>
        <tr>
            <td>Antonio Banderas</td>
            <td>62</td>
        </tr>
        <tr>
            <td>Adrienne Barbeau</td>
            <td>77</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">95083 rows, truncated to displaylimit of 20</span>



<br><br><br>

#### Use interval datatype (OPTIONAL)

We can easily do time arithmetic:

%%sql

SELECT
    '2h 50m'::INTERVAL,
    ' 5.5h'::INTERVAL + 3 * '14:00'::TIME,
    '2021-11-1'::DATE + '3 months 12 days'::INTERVAL,
    '2:00'::TIME + '18 hours 9 seconds'::INTERVAL,
    14 * '1 day'::INTERVAL
;

### Nulls



`NULLIF(value1, value2)` returns null if `value1` and `value2` are equal. This is helpful for replacing known values with nulls, or prevent, for example, division by zero.


```sql
%%sql

SELECT
    *,
    NULLIF(genre, 'drama')
FROM
    movie_genres
;
```

       postgresql://postgres:***@localhost:5432/
     * postgresql://postgres:***@localhost:5432/imdb_dsci513
    57633 rows affected.
    




<table>
    <thead>
        <tr>
            <th>movie_id</th>
            <th>genre</th>
            <th>nullif</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>10035423</td>
            <td>comedy</td>
            <td>comedy</td>
        </tr>
        <tr>
            <td>10035423</td>
            <td>fantasy</td>
            <td>fantasy</td>
        </tr>
        <tr>
            <td>10035423</td>
            <td>romance</td>
            <td>romance</td>
        </tr>
        <tr>
            <td>10042742</td>
            <td>comedy</td>
            <td>comedy</td>
        </tr>
        <tr>
            <td>10042742</td>
            <td>crime</td>
            <td>crime</td>
        </tr>
        <tr>
            <td>10042742</td>
            <td>romance</td>
            <td>romance</td>
        </tr>
        <tr>
            <td>10041181</td>
            <td>crime</td>
            <td>crime</td>
        </tr>
        <tr>
            <td>10041181</td>
            <td>film-noir</td>
            <td>film-noir</td>
        </tr>
        <tr>
            <td>10041181</td>
            <td>thriller</td>
            <td>thriller</td>
        </tr>
        <tr>
            <td>10041387</td>
            <td>comedy</td>
            <td>comedy</td>
        </tr>
        <tr>
            <td>10041387</td>
            <td>drama</td>
            <td>None</td>
        </tr>
        <tr>
            <td>10041387</td>
            <td>family</td>
            <td>family</td>
        </tr>
        <tr>
            <td>10041719</td>
            <td>drama</td>
            <td>None</td>
        </tr>
        <tr>
            <td>10041719</td>
            <td>fantasy</td>
            <td>fantasy</td>
        </tr>
        <tr>
            <td>10041719</td>
            <td>romance</td>
            <td>romance</td>
        </tr>
        <tr>
            <td>10041931</td>
            <td>drama</td>
            <td>None</td>
        </tr>
        <tr>
            <td>10042052</td>
            <td>crime</td>
            <td>crime</td>
        </tr>
        <tr>
            <td>10042052</td>
            <td>drama</td>
            <td>None</td>
        </tr>
        <tr>
            <td>10042052</td>
            <td>film-noir</td>
            <td>film-noir</td>
        </tr>
        <tr>
            <td>10042179</td>
            <td>adventure</td>
            <td>adventure</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">57633 rows, truncated to displaylimit of 20</span>



<br><br><br>

### (optional) Postgres-specific functions

There are also a number of informative functions that are specific to Postgres. You can find a list of all of them here: [link](https://www.postgresql.org/docs/current/functions-info.html).

The one that I particularly find useful is `pg_typeof()` for when I want to make sure about the type of values after doing computations:


```sql
%%sql

SELECT
    pg_typeof(54 / 3.),
    pg_typeof(100 > 1)
;
```

     * postgresql://postgres:***@localhost:5432/imdb_dsci513
       postgresql://postgres:***@localhost:5432/world_dsci513
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>pg_typeof</th>
            <th>pg_typeof_1</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>numeric</td>
            <td>boolean</td>
        </tr>
    </tbody>
</table>


