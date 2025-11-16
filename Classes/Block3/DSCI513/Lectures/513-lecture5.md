

# Lecture 5: Transactions, ACID, subqueries

## Lecture outline

- Transactions
- ACID properties
- Subqueries

## Transactions: all or nothing

Let's consider the classical example of a bank database. Suppose that I bought a vintage guitar from my friend, Joel, for $\$1000$, and now I want to transfer $\$1000$ from my bank account to Joel's account using the banking app on my phone. This operation involves:

1. Deducting (i.e. debiting) $\$1000$ from my account,
2. Adding (i.e. crediting) $\$1000$ to Joel's account.

We've already learned how to do this in SQL:

```sql
-- debiting source account
UPDATE
    accounts
SET
    balance = balance - 1000.0
WHERE
    account_number = 1132;
```

<br>

```sql
-- crediting target account
UPDATE
    accounts
SET
    balance = balance + 1000.0
WHERE
    account_number = 1279;
```

Now, what if a power outage happens right between the two `UPDATE` statements? In that case, my account would be left debited $\$1000$, while Joel has also received no money. We'd both be furious!

This example nicely illustrates a common situation where we would want either **all** or **none** of the changes to be made to the database. In other words, the changes need to be considered and treated as **a single logical unit of work**: either all changes are successfully made, or all of them should fail.

In database (and in banking) terms, each unit of work is called a **transaction**.

<br><br><br>

### Transactions in Postgres

In order to bundle the above update statements into one single transaction, we can use the following syntax:

```sql
BEGIN TRANSACTION;

-- debiting source account
UPDATE
    accounts
SET
    balance = balance - 1000.0
WHERE
    account_number = 1132;

-- crediting target account
UPDATE
    accounts
SET
    balance = balance + 1000.0
WHERE
    account_number = 1279;
    
COMMIT;
```

With the above syntax, now the DBMS knows how to treat the SQL commands: both update commands are a part of a larger unit of work. Either both changes will be made, or none.

<br><br><br>

The overall structure of a transaction is as follows:

```sql
[{BEGIN|START} [TRANSACTION]];

<SQL statements>

{COMMIT|ROLLBACK};
```

The `BEGIN` and `START` can be used interchangeably, although `BEGIN` is much more common. Moreover, the `BEGIN TRANSACTION` statement is optional. You can also start a transaction just using the command `BEGIN;` without the `TRANSACTION` keyword.

<br><br><br>

**Commit:** All data modifications during the transaction should be made permanent in the database.

**Rollback:** All changes made during the transaction should be discarded, and the database is restored to its original state before the transaction.

<br><br><br>


```python
import pandas as pd
import json
import psycopg2
import urllib.parse

with open('data/credentials.json') as f:
    login = json.load(f)
    
username = login['user']
password = urllib.parse.quote(login['password'])
host = login['host']
port = login['port']
```


```python
%load_ext sql
%config SqlMagic.displaylimit = 30
```


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

     * postgresql://postgres:***@localhost:5432/mds
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

     * postgresql://postgres:***@localhost:5432/mds
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
conn = psycopg2.connect(database='mds', **login)
```

<br><br><br>

Let's take a look at the following example. Here I begin a transaction and try to set all phone numbers in the `instructor` table to `NULL`:


```python
cur = conn.cursor()
cur.execute("""
    BEGIN TRANSACTION;
    
    UPDATE
        instructor
    SET
        phone = NULL;
""")
```

Within the current database session/connection, the changes are visible:


```python
cur.execute("SELECT * FROM instructor")
cur.fetchall()
```




    [(1, 'Mike', 'mike@mds.ubc.ca', None, 'Computer Science'),
     (2, 'Tiffany', 'tiff@mds.ubc.ca', None, 'Neuroscience'),
     (3, 'Arman', 'arman@mds.ubc.ca', None, 'Physics'),
     (4, 'Varada', 'varada@mds.ubc.ca', None, 'Computer Science'),
     (5, 'Quan', 'quan@mds.ubc.ca', None, 'Economics'),
     (6, 'Joel', 'joel@mds.ubc.ca', None, 'Biomedical Engineering'),
     (7, 'Florencia', 'flor@mds.ubc.ca', None, 'Biology'),
     (8, 'Alexi', 'alexiu@mds.ubc.ca', None, 'Statistics'),
     (15, 'Vincenzo', 'vincenzo@mds.ubc.ca', None, 'Statistics'),
     (19, 'Gittu', 'gittu@mds.ubc.ca', None, 'Biomedical Engineering'),
     (16, 'Jessica', 'jessica@mds.ubc.ca', None, 'Computer Science')]



We can see that the phone number column in all of the above tuples returned by psycopg2 is set to `NULL` (shown as `None` in Python).

However, if we look at the database from the viewpoint of any other connection/session/user, no changes seem to have been made:


```python
%sql SELECT * FROM instructor;
```

     * postgresql://postgres:***@localhost:5432/mds
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



Well, that's because the changes we've made in our original connection are not made permanent until we issue a `COMMIT` command:


```python
cur.execute("COMMIT;")
```


```python
%sql SELECT * FROM instructor;
```

     * postgresql://postgres:***@localhost:5432/mds
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
            <td>None</td>
            <td>Computer Science</td>
        </tr>
        <tr>
            <td>2</td>
            <td>Tiffany</td>
            <td>tiff@mds.ubc.ca</td>
            <td>None</td>
            <td>Neuroscience</td>
        </tr>
        <tr>
            <td>3</td>
            <td>Arman</td>
            <td>arman@mds.ubc.ca</td>
            <td>None</td>
            <td>Physics</td>
        </tr>
        <tr>
            <td>4</td>
            <td>Varada</td>
            <td>varada@mds.ubc.ca</td>
            <td>None</td>
            <td>Computer Science</td>
        </tr>
        <tr>
            <td>5</td>
            <td>Quan</td>
            <td>quan@mds.ubc.ca</td>
            <td>None</td>
            <td>Economics</td>
        </tr>
        <tr>
            <td>6</td>
            <td>Joel</td>
            <td>joel@mds.ubc.ca</td>
            <td>None</td>
            <td>Biomedical Engineering</td>
        </tr>
        <tr>
            <td>7</td>
            <td>Florencia</td>
            <td>flor@mds.ubc.ca</td>
            <td>None</td>
            <td>Biology</td>
        </tr>
        <tr>
            <td>8</td>
            <td>Alexi</td>
            <td>alexiu@mds.ubc.ca</td>
            <td>None</td>
            <td>Statistics</td>
        </tr>
        <tr>
            <td>15</td>
            <td>Vincenzo</td>
            <td>vincenzo@mds.ubc.ca</td>
            <td>None</td>
            <td>Statistics</td>
        </tr>
        <tr>
            <td>19</td>
            <td>Gittu</td>
            <td>gittu@mds.ubc.ca</td>
            <td>None</td>
            <td>Biomedical Engineering</td>
        </tr>
        <tr>
            <td>16</td>
            <td>Jessica</td>
            <td>jessica@mds.ubc.ca</td>
            <td>None</td>
            <td>Computer Science</td>
        </tr>
    </tbody>
</table>



Voila, the changes are now permanently recorded in the database, and visible by any user/connection. Note that instead of running `COMMIT;` as a SQL statement, we could have run `conn.commit()`.

<br><br><br>

Here's an example of `ROLLBACK`. I'm going to delete all rows from the `instructor` table in a transaction:


```python
cur.execute("""
    BEGIN TRANSACTION;
    
    DELETE FROM instructor;
    """)
```


```python
cur.execute("SELECT * FROM instructor")
cur.fetchall()
```




    []



As expected, there are no rows left in the instructor table.

Since the transaction is still ongoing and changes have not been made permanent, I can **abort the transaction** and rollback to bring the database back to its original state before the transaction:


```python
cur.execute("ROLLBACK;")
```


```python
cur.execute("SELECT * FROM instructor")
cur.fetchall()
```




    [(1, 'Mike', 'mike@mds.ubc.ca', None, 'Computer Science'),
     (2, 'Tiffany', 'tiff@mds.ubc.ca', None, 'Neuroscience'),
     (3, 'Arman', 'arman@mds.ubc.ca', None, 'Physics'),
     (4, 'Varada', 'varada@mds.ubc.ca', None, 'Computer Science'),
     (5, 'Quan', 'quan@mds.ubc.ca', None, 'Economics'),
     (6, 'Joel', 'joel@mds.ubc.ca', None, 'Biomedical Engineering'),
     (7, 'Florencia', 'flor@mds.ubc.ca', None, 'Biology'),
     (8, 'Alexi', 'alexiu@mds.ubc.ca', None, 'Statistics'),
     (15, 'Vincenzo', 'vincenzo@mds.ubc.ca', None, 'Statistics'),
     (19, 'Gittu', 'gittu@mds.ubc.ca', None, 'Biomedical Engineering'),
     (16, 'Jessica', 'jessica@mds.ubc.ca', None, 'Computer Science')]



For us Postgres users through psycopg2 in Python, it is important not to leave transactions open unintentionally. By default, even a `SELECT` statement begins a transaction (see [here](https://www.psycopg.org/docs/usage.html#transactions-control)), and unless the transaction is committed or rolled back explicitly, the session remains in the undesirable idle transaction state.

In order to avoid an unintentionally long-running transaction, we can do one of the following things:

- Use auto-commit mode by running `conn.autocommit = True`. In this mode, every executed statement is automatically committed if successful, and rolled back if an error occurs.

- Use `with` context manager for the connection and the cursor:
  ```python
  with conn, conn.cursor() as cur:
      cur.execute("SELECT * FROM instructor;")
  ```

  The transaction is committed when a connection exits the `with` block with no raised exceptions, otherwise the transaction is rolled back.


```python
with conn, conn.cursor() as cur:
    cur.execute("DELETE FROM instructor;")
```


```python
%sql SELECT * FROM instructor;
```

     * postgresql://postgres:***@localhost:5432/mds
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



> **Note:** Exiting the `with` block does not close a connection (as opposed to reading files using `with`). To close a connection, you need to explicitly run `conn.close()`.

> **Note:** `psql` and `ipython-sql` run in auto-commit mode by default.


```python
%config SqlMagic.autocommit
```




    True



<br><br><br>

## ACID

In the database world, a transaction must have a number of properties to guarantee data integrity and validity, which ensure that in the event of any sort of database failure, the data will not be corrupt or lost. These properties are **atomicity, consistency, isolation, and durability (ACID)**.

<br><br><br>

### Atomicity

This is the all-or-nothing property of a transaction that we've seen earlier. Transactions should be atomic, meaning that the whole transaction is supposed to be treated as a single unit of work. Either all or none of the operations inside a transaction are executed.

**Example**: We have a database with one table (_Values_) and two columns (**a** and **b**).  We have an operation where, on a particular row, a value is subtracted from **a** and added to **b**.  This might be similar to a banking transaction, where I withdraw money from my account to pay for a new pair of slippers.  The operation takes place in two steps:

1.  Remove value from **a**
2.  Place value in **b**

Atomicity ensures that if a system failure occurs between steps one and two, or if either step one or step two is invalid for any other reason, the entire transaction will not be processed.

<br><br><br>

### Consistency

You've seen in the previous section that some constraints were temporarily violated inside a transaction, but by the end of the transaction all constraints were respected. It is the responsibility of a transaction to maintain data integrity. If a transaction makes the database inconsistent or invalid (i.e. in violation of the constraints), the transaction is aborted and the database is brought back to the consistent state before the transaction.

**Example**:  We have a table *marks* that gives the breakdown of marking schemes for various courses.  It is divided into columns for **quiz**, **participation**, **midterm** and **final**.  All fields must be integers, and, because these values make up the course marking scheme, each row must sum to 100.  We can add a constraint to the table that ensures this using the `CHECK` operator.

A database that ensures consistency would reject any transaction where the final result of the transaction results in a *marks* sum <> 100.

<br><br><br>

### Isolation

Each database modification operation is either implicitly or explicitly run within a transaction. The DBMS guarantees that each transaction is kept separate and isolated from any other transaction. No other transaction can interfere with the current transaction that is in progress. Although it would be ideal to have the highest levels of isolation, this would cause serious performance issues in real-world applications. So depending on the situation, there are various levels of isolation that can be used as a trade-off between full isolation of transactions and concurrency.

**Example**:  In that same *marks* table as before, let's assume that a department has made a decision that all **final** exam scores will be reduced by 5% and that that value will be shifted to **participation**.  At the same time, an individual instructor in the program has decided to increase the weighting of the **final** and reduce the weighting of the **midterm**.

If these operations are not isolated, it is possible that the program-level transaction reduces the weighting of the final exam.

<br><br><br>

### Durability

The changes that are made by a successful transaction are permanent, even if there is a failure and the database shuts down. In Postgres and other relational databases, this is achieved by using transaction logs.

**Example**: Data that goes in following a successfully completed transaction doesn't come out unless another transaction operates to remove the data.

<br><br><br>

## Subqueries

There is a particular type of question that we have avoided so far for our SQL queries, and that is one for which we need the result of a second query to be able to run the first query. Take the following question as an example:

---

**Example:** Using the `world` database, find the countries with surface area above the average value of all countries in the world.

---

This query looks simple. You might be tempted to try

```sql
-- This will NOT work
SELECT
    name
FROM
    country
WHERE
    surfacearea > AVG(surfacearea)
;
```

but we've learned before that aggregate functions **cannot** be used within a `WHERE` clause.

<br><br><br>

To answer this question, we need to query the database twice: once to obtain the average surface area, and once to actually retrieve the rows that satisfy the condition. However, we don't need to do that in two separate queries and manually take the data coming from the first query and use it in the second. We can use a **subquery** to do that for us.

<br><br><br>

A subquery is a `SELECT` statement that is incorporated into another SQL statement. For example, the query that computes the average surface area is:

```sql
SELECT
    AVG(surfacearea)
FROM
    country
;
```

We can use this intermediate information in our original query by embedding the above query in the `WHERE` clause of the original query:

```sql
SELECT
    name
FROM
    country
WHERE
    surfacearea > (
        SELECT AVG(surfacearea) FROM country
    )
;
```


```python
%sql postgresql://{username}:{password}@{host}:{port}/world
```




    'Connected: postgres@world'




```sql
%%sql

SELECT
    name
FROM
    country
WHERE
    surfacearea > (
        SELECT
            AVG(surfacearea)
        FROM
            country
    )
;
```

       postgresql://postgres:***@localhost:5432/mds
     * postgresql://postgres:***@localhost:5432/world
    43 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Afghanistan</td>
        </tr>
        <tr>
            <td>Algeria</td>
        </tr>
        <tr>
            <td>Angola</td>
        </tr>
        <tr>
            <td>Argentina</td>
        </tr>
        <tr>
            <td>Australia</td>
        </tr>
        <tr>
            <td>Bolivia</td>
        </tr>
        <tr>
            <td>Brazil</td>
        </tr>
        <tr>
            <td>Chile</td>
        </tr>
        <tr>
            <td>Egypt</td>
        </tr>
        <tr>
            <td>South Africa</td>
        </tr>
        <tr>
            <td>Ethiopia</td>
        </tr>
        <tr>
            <td>Greenland</td>
        </tr>
        <tr>
            <td>Indonesia</td>
        </tr>
        <tr>
            <td>India</td>
        </tr>
        <tr>
            <td>Iran</td>
        </tr>
        <tr>
            <td>Canada</td>
        </tr>
        <tr>
            <td>Kazakstan</td>
        </tr>
        <tr>
            <td>China</td>
        </tr>
        <tr>
            <td>Colombia</td>
        </tr>
        <tr>
            <td>Congo, The Democratic Republic of the</td>
        </tr>
        <tr>
            <td>Libyan Arab Jamahiriya</td>
        </tr>
        <tr>
            <td>Mali</td>
        </tr>
        <tr>
            <td>Mauritania</td>
        </tr>
        <tr>
            <td>Mexico</td>
        </tr>
        <tr>
            <td>Mongolia</td>
        </tr>
        <tr>
            <td>Mozambique</td>
        </tr>
        <tr>
            <td>Myanmar</td>
        </tr>
        <tr>
            <td>Namibia</td>
        </tr>
        <tr>
            <td>Niger</td>
        </tr>
        <tr>
            <td>Nigeria</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">43 rows, truncated to displaylimit of 30</span>



<br><br><br>

Note that:

- A subquery should always be enclosed in parentheses, e.g. `(SELECT ...)`

- Subqueries should **not** be terminated by a semi-colon, as opposed to regular queries

- Sometimes the main SQL statement is called the **outer query** and the subquery is called the **inner query**

A subquery can be used in the `SELECT`, `FROM`, `WHERE`, and `HAVING` clauses. Subqueries are most commonly used in the `WHERE` clause.

<br><br><br>

In our last query, we could have alternatively used a subquery in the `SELECT` clause to check if the returned rows do satisfy the `surfacearea > AVG(surfacearea)` condition:


```sql
%%sql

SELECT
    name,
    ROUND(surfacearea::NUMERIC / (SELECT AVG(surfacearea) FROM country)::NUMERIC, 2)
        AS ratio
FROM
    country
WHERE
    surfacearea > (
        SELECT AVG(surfacearea) FROM country
    )
ORDER BY
    ratio
;
```

       postgresql://postgres:***@localhost:5432/mds
     * postgresql://postgres:***@localhost:5432/world
    43 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>ratio</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Somalia</td>
            <td>1.02</td>
        </tr>
        <tr>
            <td>Afghanistan</td>
            <td>1.05</td>
        </tr>
        <tr>
            <td>Myanmar</td>
            <td>1.09</td>
        </tr>
        <tr>
            <td>Zambia</td>
            <td>1.21</td>
        </tr>
        <tr>
            <td>Chile</td>
            <td>1.21</td>
        </tr>
        <tr>
            <td>Turkey</td>
            <td>1.24</td>
        </tr>
        <tr>
            <td>Pakistan</td>
            <td>1.28</td>
        </tr>
        <tr>
            <td>Mozambique</td>
            <td>1.29</td>
        </tr>
        <tr>
            <td>Namibia</td>
            <td>1.32</td>
        </tr>
        <tr>
            <td>Tanzania</td>
            <td>1.42</td>
        </tr>
        <tr>
            <td>Venezuela</td>
            <td>1.46</td>
        </tr>
        <tr>
            <td>Nigeria</td>
            <td>1.48</td>
        </tr>
        <tr>
            <td>Egypt</td>
            <td>1.61</td>
        </tr>
        <tr>
            <td>Mauritania</td>
            <td>1.65</td>
        </tr>
        <tr>
            <td>Bolivia</td>
            <td>1.76</td>
        </tr>
        <tr>
            <td>Ethiopia</td>
            <td>1.77</td>
        </tr>
        <tr>
            <td>Colombia</td>
            <td>1.83</td>
        </tr>
        <tr>
            <td>South Africa</td>
            <td>1.96</td>
        </tr>
        <tr>
            <td>Mali</td>
            <td>1.99</td>
        </tr>
        <tr>
            <td>Angola</td>
            <td>2.00</td>
        </tr>
        <tr>
            <td>Niger</td>
            <td>2.03</td>
        </tr>
        <tr>
            <td>Chad</td>
            <td>2.06</td>
        </tr>
        <tr>
            <td>Peru</td>
            <td>2.06</td>
        </tr>
        <tr>
            <td>Mongolia</td>
            <td>2.51</td>
        </tr>
        <tr>
            <td>Iran</td>
            <td>2.64</td>
        </tr>
        <tr>
            <td>Libyan Arab Jamahiriya</td>
            <td>2.82</td>
        </tr>
        <tr>
            <td>Indonesia</td>
            <td>3.06</td>
        </tr>
        <tr>
            <td>Mexico</td>
            <td>3.14</td>
        </tr>
        <tr>
            <td>Saudi Arabia</td>
            <td>3.45</td>
        </tr>
        <tr>
            <td>Greenland</td>
            <td>3.48</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">43 rows, truncated to displaylimit of 30</span>



> **Note:** A subquery in the `SELECT` clause should always return a single value, not a column or rows of values.

<br><br><br>

---

**Example:** Retrieve the name of countries whose capital cities have a population larger than 5 million.

---


```sql
%%sql

SELECT
    name
FROM
    country
WHERE
    capital IN (
        SELECT id
        FROM city
        WHERE population > 5000000
    )
;
```

       postgresql://postgres:***@localhost:5432/mds
     * postgresql://postgres:***@localhost:5432/world
    13 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>United Kingdom</td>
        </tr>
        <tr>
            <td>Egypt</td>
        </tr>
        <tr>
            <td>Indonesia</td>
        </tr>
        <tr>
            <td>Iran</td>
        </tr>
        <tr>
            <td>Japan</td>
        </tr>
        <tr>
            <td>China</td>
        </tr>
        <tr>
            <td>Colombia</td>
        </tr>
        <tr>
            <td>Congo, The Democratic Republic of the</td>
        </tr>
        <tr>
            <td>South Korea</td>
        </tr>
        <tr>
            <td>Mexico</td>
        </tr>
        <tr>
            <td>Peru</td>
        </tr>
        <tr>
            <td>Thailand</td>
        </tr>
        <tr>
            <td>Russian Federation</td>
        </tr>
    </tbody>
</table>



<br><br><br>

Well, as you might have guessed, we can rewrite this query using a **join**:


```sql
%%sql

SELECT
    co.name
FROM
    country co
JOIN
    city ci
ON
    co.capital = ci.id
WHERE
    ci.population > 5000000
;
```

       postgresql://postgres:***@localhost:5432/mds
     * postgresql://postgres:***@localhost:5432/world
    13 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>United Kingdom</td>
        </tr>
        <tr>
            <td>Egypt</td>
        </tr>
        <tr>
            <td>Indonesia</td>
        </tr>
        <tr>
            <td>Iran</td>
        </tr>
        <tr>
            <td>Japan</td>
        </tr>
        <tr>
            <td>China</td>
        </tr>
        <tr>
            <td>Colombia</td>
        </tr>
        <tr>
            <td>Congo, The Democratic Republic of the</td>
        </tr>
        <tr>
            <td>South Korea</td>
        </tr>
        <tr>
            <td>Mexico</td>
        </tr>
        <tr>
            <td>Peru</td>
        </tr>
        <tr>
            <td>Thailand</td>
        </tr>
        <tr>
            <td>Russian Federation</td>
        </tr>
    </tbody>
</table>



<br><br><br>

Using a subquery is actually another way to gain access to data stored in other tables.

Typically, joins can be rewritten as a subquery and vice-versa, so what's the difference?

- Subqueries tend to be more readable and more intuitive
- Subqueries cannot be used if you need to include columns from the inner query in your results

<br><br><br>

---

**Example:** Retrieve the name of countries where English is an official language, and have a population of over 1 million.

---


```sql
%%sql

SELECT
    name
FROM
    country
WHERE
    population > 1000000
    AND
    code IN (
        SELECT
            countrycode
        FROM
            countrylanguage
        WHERE
            language = 'English'
            AND
            isofficial = True
    )
;
```

       postgresql://postgres:***@localhost:5432/mds
     * postgresql://postgres:***@localhost:5432/world
    10 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Australia</td>
        </tr>
        <tr>
            <td>United Kingdom</td>
        </tr>
        <tr>
            <td>South Africa</td>
        </tr>
        <tr>
            <td>Hong Kong</td>
        </tr>
        <tr>
            <td>Ireland</td>
        </tr>
        <tr>
            <td>Canada</td>
        </tr>
        <tr>
            <td>Lesotho</td>
        </tr>
        <tr>
            <td>New Zealand</td>
        </tr>
        <tr>
            <td>United States</td>
        </tr>
        <tr>
            <td>Zimbabwe</td>
        </tr>
    </tbody>
</table>



<br><br><br>

### Correlated subqueries

Subqueries that we have seen so far are called **simple or uncorrelated subqueries**, as they are executed **once** and **independently** of the outer query.

It sometimes happens that we need data from each row of the outer query in the subquery. This is an instance of what's called a **correlated subquery**. In a correlated subquery, the subquery takes the the current row of the outer query, and executes over all rows of the inner query. When finished, the next row from the outer query is selected, and the subquery is executed entirely for that outer row again, and so on.

---

**Example:** Which countries have the largest population in the continent where they are located?

---

The query for the above example requires the population of each country to be compared with all other countries that are in the same continent. The comparison of each row, with every other row that meet a particular condition can be achieved with a correlated subquery:


```sql
%%sql

SELECT
    c1.name, c1.continent
FROM
    country c1
WHERE
    c1.population = (
        SELECT
            MAX(c2.population)
        FROM
            country c2
        WHERE
            c2.continent = c1.continent
    )
;
```

       postgresql://postgres:***@localhost:5432/mds
     * postgresql://postgres:***@localhost:5432/world
    11 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>continent</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Australia</td>
            <td>Oceania</td>
        </tr>
        <tr>
            <td>Brazil</td>
            <td>South America</td>
        </tr>
        <tr>
            <td>China</td>
            <td>Asia</td>
        </tr>
        <tr>
            <td>Nigeria</td>
            <td>Africa</td>
        </tr>
        <tr>
            <td>Russian Federation</td>
            <td>Europe</td>
        </tr>
        <tr>
            <td>United States</td>
            <td>North America</td>
        </tr>
        <tr>
            <td>Antarctica</td>
            <td>Antarctica</td>
        </tr>
        <tr>
            <td>Bouvet Island</td>
            <td>Antarctica</td>
        </tr>
        <tr>
            <td>South Georgia and the South Sandwich Islands</td>
            <td>Antarctica</td>
        </tr>
        <tr>
            <td>Heard Island and McDonald Islands</td>
            <td>Antarctica</td>
        </tr>
        <tr>
            <td>French Southern territories</td>
            <td>Antarctica</td>
        </tr>
    </tbody>
</table>



<br><br><br>

---

**Example:** Write a query that returns the most populated city listed for each country code in the `city` table.

---


```sql
%%sql

SELECT
    ci1.name, ci1.countrycode
FROM
    city ci1
WHERE
    ci1.population = (
        SELECT
            MAX(ci2.population)
        FROM
            city ci2
        WHERE
            ci2.countrycode = ci1.countrycode
    )
;
```

       postgresql://postgres:***@localhost:5432/mds
     * postgresql://postgres:***@localhost:5432/world
    232 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>countrycode</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Kabul</td>
            <td>AFG</td>
        </tr>
        <tr>
            <td>Amsterdam</td>
            <td>NLD</td>
        </tr>
        <tr>
            <td>Willemstad</td>
            <td>ANT</td>
        </tr>
        <tr>
            <td>Tirana</td>
            <td>ALB</td>
        </tr>
        <tr>
            <td>Alger</td>
            <td>DZA</td>
        </tr>
        <tr>
            <td>Tafuna</td>
            <td>ASM</td>
        </tr>
        <tr>
            <td>Andorra la Vella</td>
            <td>AND</td>
        </tr>
        <tr>
            <td>Luanda</td>
            <td>AGO</td>
        </tr>
        <tr>
            <td>South Hill</td>
            <td>AIA</td>
        </tr>
        <tr>
            <td>Saint John´s</td>
            <td>ATG</td>
        </tr>
        <tr>
            <td>Dubai</td>
            <td>ARE</td>
        </tr>
        <tr>
            <td>Buenos Aires</td>
            <td>ARG</td>
        </tr>
        <tr>
            <td>Yerevan</td>
            <td>ARM</td>
        </tr>
        <tr>
            <td>Oranjestad</td>
            <td>ABW</td>
        </tr>
        <tr>
            <td>Sydney</td>
            <td>AUS</td>
        </tr>
        <tr>
            <td>Baku</td>
            <td>AZE</td>
        </tr>
        <tr>
            <td>Nassau</td>
            <td>BHS</td>
        </tr>
        <tr>
            <td>al-Manama</td>
            <td>BHR</td>
        </tr>
        <tr>
            <td>Dhaka</td>
            <td>BGD</td>
        </tr>
        <tr>
            <td>Bridgetown</td>
            <td>BRB</td>
        </tr>
        <tr>
            <td>Antwerpen</td>
            <td>BEL</td>
        </tr>
        <tr>
            <td>Belize City</td>
            <td>BLZ</td>
        </tr>
        <tr>
            <td>Cotonou</td>
            <td>BEN</td>
        </tr>
        <tr>
            <td>Saint George</td>
            <td>BMU</td>
        </tr>
        <tr>
            <td>Thimphu</td>
            <td>BTN</td>
        </tr>
        <tr>
            <td>Santa Cruz de la Sierra</td>
            <td>BOL</td>
        </tr>
        <tr>
            <td>Sarajevo</td>
            <td>BIH</td>
        </tr>
        <tr>
            <td>Gaborone</td>
            <td>BWA</td>
        </tr>
        <tr>
            <td>São Paulo</td>
            <td>BRA</td>
        </tr>
        <tr>
            <td>London</td>
            <td>GBR</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">232 rows, truncated to displaylimit of 30</span>



<br><br><br>

---

**Example:** Use the above query to return the name of countries whose capital city is not their most populated city.

---

The outer query in a subquery can be the result of joining other tables. In this example, we first need to join `country` and `city` to list capital cities of each country:


```sql
%%sql

SELECT
    co.name, ci.name
FROM
    country co
JOIN
    city ci
ON
    co.capital = ci.id
;
```

       postgresql://postgres:***@localhost:5432/mds
     * postgresql://postgres:***@localhost:5432/world
    232 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>name_1</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Afghanistan</td>
            <td>Kabul</td>
        </tr>
        <tr>
            <td>Netherlands</td>
            <td>Amsterdam</td>
        </tr>
        <tr>
            <td>Netherlands Antilles</td>
            <td>Willemstad</td>
        </tr>
        <tr>
            <td>Albania</td>
            <td>Tirana</td>
        </tr>
        <tr>
            <td>Algeria</td>
            <td>Alger</td>
        </tr>
        <tr>
            <td>American Samoa</td>
            <td>Fagatogo</td>
        </tr>
        <tr>
            <td>Andorra</td>
            <td>Andorra la Vella</td>
        </tr>
        <tr>
            <td>Angola</td>
            <td>Luanda</td>
        </tr>
        <tr>
            <td>Anguilla</td>
            <td>The Valley</td>
        </tr>
        <tr>
            <td>Antigua and Barbuda</td>
            <td>Saint John´s</td>
        </tr>
        <tr>
            <td>United Arab Emirates</td>
            <td>Abu Dhabi</td>
        </tr>
        <tr>
            <td>Argentina</td>
            <td>Buenos Aires</td>
        </tr>
        <tr>
            <td>Armenia</td>
            <td>Yerevan</td>
        </tr>
        <tr>
            <td>Aruba</td>
            <td>Oranjestad</td>
        </tr>
        <tr>
            <td>Australia</td>
            <td>Canberra</td>
        </tr>
        <tr>
            <td>Azerbaijan</td>
            <td>Baku</td>
        </tr>
        <tr>
            <td>Bahamas</td>
            <td>Nassau</td>
        </tr>
        <tr>
            <td>Bahrain</td>
            <td>al-Manama</td>
        </tr>
        <tr>
            <td>Bangladesh</td>
            <td>Dhaka</td>
        </tr>
        <tr>
            <td>Barbados</td>
            <td>Bridgetown</td>
        </tr>
        <tr>
            <td>Belgium</td>
            <td>Bruxelles [Brussel]</td>
        </tr>
        <tr>
            <td>Belize</td>
            <td>Belmopan</td>
        </tr>
        <tr>
            <td>Benin</td>
            <td>Porto-Novo</td>
        </tr>
        <tr>
            <td>Bermuda</td>
            <td>Hamilton</td>
        </tr>
        <tr>
            <td>Bhutan</td>
            <td>Thimphu</td>
        </tr>
        <tr>
            <td>Bolivia</td>
            <td>La Paz</td>
        </tr>
        <tr>
            <td>Bosnia and Herzegovina</td>
            <td>Sarajevo</td>
        </tr>
        <tr>
            <td>Botswana</td>
            <td>Gaborone</td>
        </tr>
        <tr>
            <td>Brazil</td>
            <td>Brasília</td>
        </tr>
        <tr>
            <td>United Kingdom</td>
            <td>London</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">232 rows, truncated to displaylimit of 30</span>



In the next step, we need to check whether the population of that capital city is the maximum population of cities of that particular country or not. This can be achieved via a correlated subquery:


```sql
%%sql

SELECT
    co.name
FROM
    country co
JOIN
    city ci
ON
    co.capital = ci.id
WHERE
    ci.population <> (
        SELECT
            MAX(ci2.population)
        FROM
            city ci2
        WHERE
            ci.countrycode = ci2.countrycode
    )
ORDER BY
    co.population DESC
;
```

       postgresql://postgres:***@localhost:5432/mds
     * postgresql://postgres:***@localhost:5432/world
    48 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>China</td>
        </tr>
        <tr>
            <td>India</td>
        </tr>
        <tr>
            <td>United States</td>
        </tr>
        <tr>
            <td>Brazil</td>
        </tr>
        <tr>
            <td>Pakistan</td>
        </tr>
        <tr>
            <td>Nigeria</td>
        </tr>
        <tr>
            <td>Vietnam</td>
        </tr>
        <tr>
            <td>Philippines</td>
        </tr>
        <tr>
            <td>Turkey</td>
        </tr>
        <tr>
            <td>South Africa</td>
        </tr>
        <tr>
            <td>Tanzania</td>
        </tr>
        <tr>
            <td>Canada</td>
        </tr>
        <tr>
            <td>Sudan</td>
        </tr>
        <tr>
            <td>Morocco</td>
        </tr>
        <tr>
            <td>Australia</td>
        </tr>
        <tr>
            <td>Kazakstan</td>
        </tr>
        <tr>
            <td>Cameroon</td>
        </tr>
        <tr>
            <td>Côte dIvoire</td>
        </tr>
        <tr>
            <td>Ecuador</td>
        </tr>
        <tr>
            <td>Malawi</td>
        </tr>
        <tr>
            <td>Belgium</td>
        </tr>
        <tr>
            <td>Senegal</td>
        </tr>
        <tr>
            <td>Bolivia</td>
        </tr>
        <tr>
            <td>Switzerland</td>
        </tr>
        <tr>
            <td>Hong Kong</td>
        </tr>
        <tr>
            <td>Benin</td>
        </tr>
        <tr>
            <td>New Zealand</td>
        </tr>
        <tr>
            <td>Jamaica</td>
        </tr>
        <tr>
            <td>Oman</td>
        </tr>
        <tr>
            <td>United Arab Emirates</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">48 rows, truncated to displaylimit of 30</span>



Correlated subqueries are usually quite inefficient (remember time complexity of nested loops from DSCI 512?), but they can answer some interesting and complex questions which might otherwise be impossible to answer using SQL.

<br><br><br>

### `ANY` and `ALL`

Syntax:

```sql
SELECT
    column_name(s)
FROM
    table_name
WHERE
    column_name operator {ALL|ANY} (
        SELECT
            column_name
        FROM
            table_name
        WHERE
            condition
    );
```

<br><br><br>

---

**Example:** Find all non-European countries whose population is larger than every European country.

---


```sql
%%sql

SELECT
    name
FROM
    country
WHERE
    continent <> 'Europe'
    AND
    population > ALL (
        SELECT
            population
        FROM
            country
        WHERE
            continent = 'Europe'
    )
;
```

       postgresql://postgres:***@localhost:5432/mds
     * postgresql://postgres:***@localhost:5432/world
    6 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Brazil</td>
        </tr>
        <tr>
            <td>Indonesia</td>
        </tr>
        <tr>
            <td>India</td>
        </tr>
        <tr>
            <td>China</td>
        </tr>
        <tr>
            <td>Pakistan</td>
        </tr>
        <tr>
            <td>United States</td>
        </tr>
    </tbody>
</table>



<br><br><br>

---

**Example:** Find all European countries whose population is smaller than at least one city in the US.

---


```sql
%%sql

SELECT
    name
FROM
    country
WHERE
    continent = 'Europe'
    AND
    population < ANY (
        SELECT
            population
        FROM
            city
        WHERE
            countrycode = (
                SELECT
                    code
                FROM
                    country
                WHERE
                    name ILIKE '%United%States'
            )
    )
;
```

       postgresql://postgres:***@localhost:5432/mds
     * postgresql://postgres:***@localhost:5432/world
    26 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Albania</td>
        </tr>
        <tr>
            <td>Andorra</td>
        </tr>
        <tr>
            <td>Bosnia and Herzegovina</td>
        </tr>
        <tr>
            <td>Faroe Islands</td>
        </tr>
        <tr>
            <td>Gibraltar</td>
        </tr>
        <tr>
            <td>Svalbard and Jan Mayen</td>
        </tr>
        <tr>
            <td>Ireland</td>
        </tr>
        <tr>
            <td>Iceland</td>
        </tr>
        <tr>
            <td>Croatia</td>
        </tr>
        <tr>
            <td>Latvia</td>
        </tr>
        <tr>
            <td>Liechtenstein</td>
        </tr>
        <tr>
            <td>Lithuania</td>
        </tr>
        <tr>
            <td>Luxembourg</td>
        </tr>
        <tr>
            <td>Macedonia</td>
        </tr>
        <tr>
            <td>Malta</td>
        </tr>
        <tr>
            <td>Moldova</td>
        </tr>
        <tr>
            <td>Monaco</td>
        </tr>
        <tr>
            <td>Norway</td>
        </tr>
        <tr>
            <td>San Marino</td>
        </tr>
        <tr>
            <td>Slovakia</td>
        </tr>
        <tr>
            <td>Slovenia</td>
        </tr>
        <tr>
            <td>Finland</td>
        </tr>
        <tr>
            <td>Switzerland</td>
        </tr>
        <tr>
            <td>Denmark</td>
        </tr>
        <tr>
            <td>Holy See (Vatican City State)</td>
        </tr>
        <tr>
            <td>Estonia</td>
        </tr>
    </tbody>
</table>



<br><br><br>

Subqueries written with `ALL` or `ANY` and `<`, `<=`, `>`, and `>=` be rewritten using aggregations. For example, we can rewrite the above query as:


```sql
%%sql

SELECT
    name
FROM
    country
WHERE
    continent = 'Europe'
    AND
    population < (
        SELECT
            MAX(population)
        FROM
            city
        WHERE
            countrycode = (
                SELECT
                    code
                FROM
                    country
                WHERE
                    name ILIKE '%United%States'
            )
    )
;
```

       postgresql://postgres:***@localhost:5432/mds
     * postgresql://postgres:***@localhost:5432/world
    26 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Albania</td>
        </tr>
        <tr>
            <td>Andorra</td>
        </tr>
        <tr>
            <td>Bosnia and Herzegovina</td>
        </tr>
        <tr>
            <td>Faroe Islands</td>
        </tr>
        <tr>
            <td>Gibraltar</td>
        </tr>
        <tr>
            <td>Svalbard and Jan Mayen</td>
        </tr>
        <tr>
            <td>Ireland</td>
        </tr>
        <tr>
            <td>Iceland</td>
        </tr>
        <tr>
            <td>Croatia</td>
        </tr>
        <tr>
            <td>Latvia</td>
        </tr>
        <tr>
            <td>Liechtenstein</td>
        </tr>
        <tr>
            <td>Lithuania</td>
        </tr>
        <tr>
            <td>Luxembourg</td>
        </tr>
        <tr>
            <td>Macedonia</td>
        </tr>
        <tr>
            <td>Malta</td>
        </tr>
        <tr>
            <td>Moldova</td>
        </tr>
        <tr>
            <td>Monaco</td>
        </tr>
        <tr>
            <td>Norway</td>
        </tr>
        <tr>
            <td>San Marino</td>
        </tr>
        <tr>
            <td>Slovakia</td>
        </tr>
        <tr>
            <td>Slovenia</td>
        </tr>
        <tr>
            <td>Finland</td>
        </tr>
        <tr>
            <td>Switzerland</td>
        </tr>
        <tr>
            <td>Denmark</td>
        </tr>
        <tr>
            <td>Holy See (Vatican City State)</td>
        </tr>
        <tr>
            <td>Estonia</td>
        </tr>
    </tbody>
</table>



<br><br><br>

Here are equivalencies between using `ALL` and `ANY` with `MAX` and `MIN` in subqueries:

| Using `ALL` and `ANY` | Using `MIN` and `MAX` |
|-----------------------|-----------------------|
| `< ALL (subquery)`    | `< MIN(values)`       |
| `> ALL (subquery)`    | `> MAX(values)`       |
| `< ANY (subquery)`    | `< MAX(values)`       |
| `> ANY (subquery)`    | `> MIN(values)`       |

> **Remember:** The fact that two queries achieve the same result does not mean that they are also the same in terms of performance. For example, using `> MAX()` is usually faster than `> ALL()`.

<br><br><br>

### `EXISTS`

With subqueries, sometimes we don't care about the rows that are returned, but if a row is returned at all or not. The `EXISTS` and `NOT EXISTS` keyword in SQL provide this type of functionality for us, i.e., the check for **existence** of rows, not their values. Here is the syntax for an `EXISTS` subquery:

```sql
SELECT
    column_name(s)
FROM
    table_name
WHERE
    [NOT] EXISTS (
        SELECT column_name FROM table_name WHERE condition
    )
;
```

If the subquery returns one or more rows, then the `WHERE` condition becomes `TRUE`.

<br><br><br>

Remember that:

- With `EXISTS`, the columns returned by the subquery do not matter at all, which is why we usually use `SELECT *` in the subquery.

- The subquery following `EXISTS` can be either simple or correlated, but typically they are correlated.

<br><br><br>

---

**Example:** Find all countries that have at least a city with a population greater that 5 million.

---


```sql
%%sql

SELECT
    co.name
FROM
    country co
WHERE
    EXISTS (
        SELECT
            *
        FROM
            city ci
        WHERE
            co.code = ci.countrycode
            AND
            ci.population > 5000000
    )
;
```

       postgresql://postgres:***@localhost:5432/mds
     * postgresql://postgres:***@localhost:5432/world
    18 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Brazil</td>
        </tr>
        <tr>
            <td>United Kingdom</td>
        </tr>
        <tr>
            <td>Egypt</td>
        </tr>
        <tr>
            <td>Indonesia</td>
        </tr>
        <tr>
            <td>India</td>
        </tr>
        <tr>
            <td>Iran</td>
        </tr>
        <tr>
            <td>Japan</td>
        </tr>
        <tr>
            <td>China</td>
        </tr>
        <tr>
            <td>Colombia</td>
        </tr>
        <tr>
            <td>Congo, The Democratic Republic of the</td>
        </tr>
        <tr>
            <td>South Korea</td>
        </tr>
        <tr>
            <td>Mexico</td>
        </tr>
        <tr>
            <td>Pakistan</td>
        </tr>
        <tr>
            <td>Peru</td>
        </tr>
        <tr>
            <td>Thailand</td>
        </tr>
        <tr>
            <td>Turkey</td>
        </tr>
        <tr>
            <td>Russian Federation</td>
        </tr>
        <tr>
            <td>United States</td>
        </tr>
    </tbody>
</table>



<br><br><br>

---

**Example:** Which countries speak at least one language that is not spoken in any other country in their continent?

---

It's best to first create a temporary table that stores the names of countries, continents and their languages:


```sql
%%sql

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
    WHERE
        co.continent IN ('Asia', 'Europe')
)
;
```

       postgresql://postgres:***@localhost:5432/mds
     * postgresql://postgres:***@localhost:5432/world
    Done.
    441 rows affected.
    




    []



Now we can use `NOT EXISTS` to detect countries that have languages not spoken elsewhere in their continent. Note that because a subquery runs its query over all rows, we need to exclude the current row of the outer query, otherwise there would always be a country in the same continent speaking all languages of the country in the outer query, i.e. itself!


```sql
%%sql

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

       postgresql://postgres:***@localhost:5432/mds
     * postgresql://postgres:***@localhost:5432/world
    129 rows affected.
    




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
            <td>Bhutan</td>
            <td>Asia</td>
            <td>Dzongkha</td>
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
            <td>Kyrgyzstan</td>
            <td>Asia</td>
            <td>Kirgiz</td>
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
            <td>Luxembourg</td>
            <td>Europe</td>
            <td>Luxembourgish</td>
        </tr>
        <tr>
            <td>Maldives</td>
            <td>Asia</td>
            <td>Dhivehi</td>
        </tr>
        <tr>
            <td>Malta</td>
            <td>Europe</td>
            <td>Maltese</td>
        </tr>
        <tr>
            <td>Myanmar</td>
            <td>Asia</td>
            <td>Burmese</td>
        </tr>
        <tr>
            <td>Sri Lanka</td>
            <td>Asia</td>
            <td>Singali</td>
        </tr>
        <tr>
            <td>Taiwan</td>
            <td>Asia</td>
            <td>Min</td>
        </tr>
        <tr>
            <td>Afghanistan</td>
            <td>Asia</td>
            <td>Dari</td>
        </tr>
        <tr>
            <td>Netherlands</td>
            <td>Europe</td>
            <td>Fries</td>
        </tr>
        <tr>
            <td>Bangladesh</td>
            <td>Asia</td>
            <td>Chakma</td>
        </tr>
        <tr>
            <td>United Kingdom</td>
            <td>Europe</td>
            <td>Kymri</td>
        </tr>
        <tr>
            <td>Brunei</td>
            <td>Asia</td>
            <td>Malay-English</td>
        </tr>
        <tr>
            <td>Philippines</td>
            <td>Asia</td>
            <td>Cebuano</td>
        </tr>
        <tr>
            <td>Ireland</td>
            <td>Europe</td>
            <td>Irish</td>
        </tr>
        <tr>
            <td>Italy</td>
            <td>Europe</td>
            <td>Sardinian</td>
        </tr>
        <tr>
            <td>Yemen</td>
            <td>Asia</td>
            <td>Soqutri</td>
        </tr>
        <tr>
            <td>Jordan</td>
            <td>Asia</td>
            <td>Circassian</td>
        </tr>
        <tr>
            <td>China</td>
            <td>Asia</td>
            <td>Zhuang</td>
        </tr>
        <tr>
            <td>Laos</td>
            <td>Asia</td>
            <td>Mon-khmer</td>
        </tr>
        <tr>
            <td>Monaco</td>
            <td>Europe</td>
            <td>Monegasque</td>
        </tr>
        <tr>
            <td>Myanmar</td>
            <td>Asia</td>
            <td>Shan</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">129 rows, truncated to displaylimit of 30</span>



We can also replace `t1.*` in the `SELECT` clause with `COUNT(DISTINCT name)` to find how many countries with such a property exist in the world.

To summarize, we use `EXISTS` when:

- We don't need the data from a related table. With joins, we always have access to the columns of the target tables as well.
- We just need to check existence, which can also be achieved by using outer joins and checking for nulls.

<br><br><br>

---

**Example:** Find city names that happen to be used in more than one country!

---


```sql
%%sql

SELECT
    DISTINCT ci1.name
FROM
    city ci1
WHERE
    EXISTS (
        SELECT
            ci2.name
        FROM
            city ci2
        WHERE
            ci2.name = ci1.name
            AND
            ci1.countrycode <> ci2.countrycode
    )
ORDER BY
    ci1.name DESC
;
```

       postgresql://postgres:***@localhost:5432/mds
     * postgresql://postgres:***@localhost:5432/world
    47 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>York</td>
        </tr>
        <tr>
            <td>Worcester</td>
        </tr>
        <tr>
            <td>Victoria</td>
        </tr>
        <tr>
            <td>Vancouver</td>
        </tr>
        <tr>
            <td>Valencia</td>
        </tr>
        <tr>
            <td>Tripoli</td>
        </tr>
        <tr>
            <td>Toledo</td>
        </tr>
        <tr>
            <td>Taiping</td>
        </tr>
        <tr>
            <td>Santa Rosa</td>
        </tr>
        <tr>
            <td>Santa Maria</td>
        </tr>
        <tr>
            <td>Santa Clara</td>
        </tr>
        <tr>
            <td>Santa Ana</td>
        </tr>
        <tr>
            <td>San Miguel</td>
        </tr>
        <tr>
            <td>San Mateo</td>
        </tr>
        <tr>
            <td>San Juan</td>
        </tr>
        <tr>
            <td>San Jose</td>
        </tr>
        <tr>
            <td>San Fernando</td>
        </tr>
        <tr>
            <td>San Felipe</td>
        </tr>
        <tr>
            <td>Salem</td>
        </tr>
        <tr>
            <td>Salamanca</td>
        </tr>
        <tr>
            <td>Saint John´s</td>
        </tr>
        <tr>
            <td>Richmond</td>
        </tr>
        <tr>
            <td>Portsmouth</td>
        </tr>
        <tr>
            <td>Plymouth</td>
        </tr>
        <tr>
            <td>Newcastle</td>
        </tr>
        <tr>
            <td>Mérida</td>
        </tr>
        <tr>
            <td>Manzanillo</td>
        </tr>
        <tr>
            <td>Manchester</td>
        </tr>
        <tr>
            <td>Los Angeles</td>
        </tr>
        <tr>
            <td>London</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">47 rows, truncated to displaylimit of 30</span>



Alternatively, we can use a self-join to find city names used in more than one country:


```sql
%%sql

SELECT
    DISTINCT ON (ci1.name) ci1.name,
    ci1.countrycode,
    ci2.name,
    ci2.countrycode
FROM
    city ci1
JOIN
    city ci2
ON
    ci2.name = ci1.name
    AND
    ci1.countrycode <> ci2.countrycode
ORDER BY
    ci1.name DESC
;
```

       postgresql://postgres:***@localhost:5432/mds
     * postgresql://postgres:***@localhost:5432/world
    47 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>countrycode</th>
            <th>name_1</th>
            <th>countrycode_1</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>York</td>
            <td>CAN</td>
            <td>York</td>
            <td>GBR</td>
        </tr>
        <tr>
            <td>Worcester</td>
            <td>USA</td>
            <td>Worcester</td>
            <td>GBR</td>
        </tr>
        <tr>
            <td>Victoria</td>
            <td>MEX</td>
            <td>Victoria</td>
            <td>SYC</td>
        </tr>
        <tr>
            <td>Vancouver</td>
            <td>CAN</td>
            <td>Vancouver</td>
            <td>USA</td>
        </tr>
        <tr>
            <td>Valencia</td>
            <td>ESP</td>
            <td>Valencia</td>
            <td>VEN</td>
        </tr>
        <tr>
            <td>Tripoli</td>
            <td>LBY</td>
            <td>Tripoli</td>
            <td>LBN</td>
        </tr>
        <tr>
            <td>Toledo</td>
            <td>BRA</td>
            <td>Toledo</td>
            <td>PHL</td>
        </tr>
        <tr>
            <td>Taiping</td>
            <td>MYS</td>
            <td>Taiping</td>
            <td>TWN</td>
        </tr>
        <tr>
            <td>Santa Rosa</td>
            <td>PHL</td>
            <td>Santa Rosa</td>
            <td>USA</td>
        </tr>
        <tr>
            <td>Santa Maria</td>
            <td>BRA</td>
            <td>Santa Maria</td>
            <td>PHL</td>
        </tr>
        <tr>
            <td>Santa Clara</td>
            <td>USA</td>
            <td>Santa Clara</td>
            <td>CUB</td>
        </tr>
        <tr>
            <td>Santa Ana</td>
            <td>USA</td>
            <td>Santa Ana</td>
            <td>SLV</td>
        </tr>
        <tr>
            <td>San Miguel</td>
            <td>ARG</td>
            <td>San Miguel</td>
            <td>PHL</td>
        </tr>
        <tr>
            <td>San Mateo</td>
            <td>USA</td>
            <td>San Mateo</td>
            <td>PHL</td>
        </tr>
        <tr>
            <td>San Juan</td>
            <td>PRI</td>
            <td>San Juan</td>
            <td>ARG</td>
        </tr>
        <tr>
            <td>San Jose</td>
            <td>USA</td>
            <td>San Jose</td>
            <td>PHL</td>
        </tr>
        <tr>
            <td>San Fernando</td>
            <td>PHL</td>
            <td>San Fernando</td>
            <td>ARG</td>
        </tr>
        <tr>
            <td>San Felipe</td>
            <td>MEX</td>
            <td>San Felipe</td>
            <td>VEN</td>
        </tr>
        <tr>
            <td>Salem</td>
            <td>IND</td>
            <td>Salem</td>
            <td>USA</td>
        </tr>
        <tr>
            <td>Salamanca</td>
            <td>ESP</td>
            <td>Salamanca</td>
            <td>MEX</td>
        </tr>
        <tr>
            <td>Saint John´s</td>
            <td>CAN</td>
            <td>Saint John´s</td>
            <td>ATG</td>
        </tr>
        <tr>
            <td>Richmond</td>
            <td>CAN</td>
            <td>Richmond</td>
            <td>USA</td>
        </tr>
        <tr>
            <td>Portsmouth</td>
            <td>GBR</td>
            <td>Portsmouth</td>
            <td>USA</td>
        </tr>
        <tr>
            <td>Plymouth</td>
            <td>GBR</td>
            <td>Plymouth</td>
            <td>MSR</td>
        </tr>
        <tr>
            <td>Newcastle</td>
            <td>ZAF</td>
            <td>Newcastle</td>
            <td>AUS</td>
        </tr>
        <tr>
            <td>Mérida</td>
            <td>MEX</td>
            <td>Mérida</td>
            <td>VEN</td>
        </tr>
        <tr>
            <td>Manzanillo</td>
            <td>CUB</td>
            <td>Manzanillo</td>
            <td>MEX</td>
        </tr>
        <tr>
            <td>Manchester</td>
            <td>USA</td>
            <td>Manchester</td>
            <td>GBR</td>
        </tr>
        <tr>
            <td>Los Angeles</td>
            <td>USA</td>
            <td>Los Angeles</td>
            <td>CHL</td>
        </tr>
        <tr>
            <td>London</td>
            <td>GBR</td>
            <td>London</td>
            <td>CAN</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">47 rows, truncated to displaylimit of 30</span>



<br><br><br>

## Set operators (OPTIONAL)

 The set operations are used to combine different `SELECT` statements. All the three set operators require that the `SELECT` clause lists the same number of columns.

- `UNION` combines all rows from both selects into the same table. It removes duplicates. (use UNION ALL to keep duplicates).

- `INTERSECT` combines only rows that are present in both tables. It removes duplicates. (use INTERSECT ALL to keep duplicates).

- `EXCEPT` only keeps the rows that are in the first table but are not in the second table.
