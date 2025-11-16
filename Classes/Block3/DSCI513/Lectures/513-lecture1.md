
# Lecture 1: Introduction to relational databases

## Course announcements


- Significance of this course
- What this course is, and what it isn't
- Overview of the course

## Lecture outline

- Why use databases
- The relational model
- Query languages, SQL, Postgres
- How to run SQL
- Basic SQL queries

## Why not spreadsheets?

At this point in MDS, you have a good idea of why spreadsheet software like Excel or its equivalents are not suitable for most data science purposes. 

Pandas:
- reproducible
- range of functionalities
- scalable
- fast
- can be automated

## Why not Pandas dataframes?

But Pandas was pretty nice and powerful, wasn't it? let's see.

---
**Question:**

What kind of problems do you think you might run into using a pandas dataframe like the above?

---

Think about what happens if:

- Your dataframe is 100 GB in size

- Multiple people want to use and make changes to the dataset simultaneously
- You want to be able to manage what each user can do
- You want to be able to let different users see different parts of the dataset
- You don't want to store everything at one place
- You want to restrict the kind of data to be stored
- The dataset file is corrupted
- The system crashes half way through making a change
- You want to optimize access to your data
- ...

## Databases and database management systems

A **database management system (DBMS)** addresses all of the above problems.

**What is a database?**
A database is an organized collection of related data

**What is a database management system?**
A DBMS is a collection of programs that enables users to create, query, modify and manage a database in an optimized and efficient manner. A DBMS relieves us from worrying about storing a manage

Using a DBMS ensures:

- Efficient data access
- Data integrity
- Data security
- Concurrent access
- Crash recovery
- Data independence

---

**Remember:**
    
database $\ne$ database management system

---

### Data model

A data model is the way we choose to represent data. We usually try to model the data in a way that is closer to how we think about the data.

You probably remember from when we talked about tidy data, that we like to see
- each observation or measurement as a **row**
- each variable or attribute as **column**

Is that the only way to represent data? No! But that's the one the makes sense for a variety of applications.

That particular way of representing the data is called a **data model**.

There are different types of DBMS for different kinds of data models:

- Relational (most widely used)
- Document
- Hierarchical
- Network
- Object-oriented
- Graph

---

**Example:**

In a graph database for a social media application, people may be represented as the nodes of a graph, whereas graph edges may define the relationship of each person to another person.

---

<br><br><br>

---

**Example:**

In a document database, we may choose to store information about students of a university as individual documents. Inside each document, attributes are stored as key-value pairs.

---

In this course, we'll talk mostly about **relational** DBMSs (RDBMS) and briefly about **non-relational** DBMSs.

## The relational model

### Why the relational model?

Take a moment and think about the kind of problems that you may run into if you choose to store data in a single table.

<img src="img/lecture1/table.png" width="800">

The most famous data model today is the relational model, while other models have also gained traction in the past few years.

The relational model works with **entities** and **relationships**. It is based on the set theory in mathematics was introduced by by Edgar Codd (IBM) in 1970 ([more details here](https://en.wikipedia.org/wiki/Relational_model)). It's foundations in **set theory** is the reason you will here words like "tuples", "domain", "union", "cross product", etc.

<br><br><br>

---

**Example:**

Entities:
- students in a school
- employees of an organization
- cars of a rental company
- houses in a city

Relations:
- students to a department
- purchases to customers
- movies to actors
- customers to a bank
    
---

In a relational model, entities and relationships are both **sets of tuples** called **relations**. These relations are represented as **tables** with rows and columns.

**What is a relational database?** A collection of relations

A **Relation** is an instance of a schema (just like an object was an instance of a class!)

The **Schema** specifies
1. Name of a relation
2. Name and domain of each attribute

**Domain**: A set of constraints that determines the type, length, format, range, uniqueness and nullability of values stored for an attribute.

<br><br><br>

---

**Example:**

Student (**sid**: _string_, **name**: _string_, **login**: _string_, **age**: _integer_, **gpa**: _real_)

---

<br><br><br>

**Instance**: a particular relation that follows a certain schema

**Relational Database Schema**: collection of schemas in the database

**Database Instance**: a collection of instances of its relations

### Anatomy of a table

<img src="img/lecture1/table_anatomy.png" width="700">

---

**Question:**
Can you name a few differences between tables in a database and spreadsheets? How about between tables and Pandas dataframes?

---

<br><br><br>

### Properties of a table (or relation)

- Contains data about a particular entity or relationship between entities.

- Has a unique name in a database.
- Each row-column intersection stores a single value, that is to say values should be atomic.
- Has at least one column and zero or more rows.

<br><br><br>

### Properties of rows (or tuples, records)

- The order of rows are not important. Therefore, there is no index-based method to retrieve rows like in Pandas.

- Contains information about an instance of the entity/relationship which the table represents
- Each row is identified by its _primary_ key (we'll learn more about this in future lectures)

<br><br><br>

### Properties of columns (or attributes)

- Has a unique name in a table.

- Represents a particular property of the entity/relationship which the table represents
- The order of columns are not important. Therefore, there is no index-based method to retrieve columns like in Pandas.
- Is valued according to a domain (rules for values)

<br><br><br>

## Query language in a DBMS

**What is a query?** A question that we ask about the data. The result of a query is a new relation.

In order to talk to the database and ask questions, we need to speak its language. A DBMS
- provides a specialized language for us to write our queries
- optimizes how our queries are executed

The data query language (DQL) is part of a bigger set of languages for working with data in a relational database, which consists of
- data definition language (DDL) for creating, altering and deleting tables
- data manipulation language (DML) for inserting new data, updating values, etc.
- data query language (DQL) for querying and retrieving data
- data control language (DCL) for management and controlling user access, rights and privileges

<br><br><br>

## What is SQL?

Well, it's finally time to learn about SQL!

- SQL stands for Structured Query Language ([or... does it?](https://en.wikipedia.org/wiki/SQL#History)).

- It is a programming language that we use to talk to a relational DBMS.
- Originally developed by IBM in 1970s to manipulate and retrieve data stored in their DBMS, System R.
- SQL ≠ relational model ≠ database ≠ DBMS

### A peak at SQL queries

Suppose that we have the following table (relation) in our database, and 

> we want to retrieve the names and GPAs of students older than 25.

|  sid  | name      | login      | age | gpa |
|-------|-----------|------------|-----|-----|
| 23792 | Arman     | arman@mds  | 28  | 2.5 |
| 82347 | Varada    | varada@mds | 29  | 2.9 |
| 11238 | Tiffany   | tiff@mds   | 23  | 2.8 |
| 87263 | Mike      | mike@mds   | 19  | 3.8 |
| 13298 | Joel      | joel@mds   | 25  | 3.2 |
| 91287 | Florencia | flor@mds   | 20  | 3.3 |

<br><br>

We can write this as the following SQL query:

```sql
SELECT
    name, age, gpa
FROM
    Students
WHERE
    age > 25;
```

<br><br>

Running the above query should return this relation:

| name   | age | gpa |
|--------|-----|-----|
| Arman  | 28  | 2.5 |
| Varada | 29  | 2.9 |

<br><br><br>

### SQL syntax

Let's dissect the different parts of the our SQL query here:

```sql
SELECT
    name, age, gpa
FROM
    Students
WHERE
    age > 25;
```

A SQL statement consists of keywords, clauses, identifiers, terminating semi-colon and sometimes comments which together form a complete executable and independent piece of code.

<br><br>

```sql
SELECT
```
- The keyword `SELECT` is the **keyword** that exists in every SQL query. It is used to select and return data from columns, given the conditions that follow it.

```sql
name, age, gpa
Students
```

- `SELECT` is very powerful, but not dangerous: A `SELECT` statement never changes any values or tables in the database.

- The fact that we select only a few columns (instead of all of them) is called **projection** in database terms.
  
- These are called **identifiers**, and refer to the labels of columns and tables that exist in the database.

<br><br>

```sql
FROM
```

- This is another keyword that tells SQL which relation (i.e. table) to retrieve the columns from.

<br><br>

```sql
WHERE
```
- Yet another SQL keyword that is used to place a condition (also known as a "predicate") on the returned values.

<br><br>

- We can also have comments in a SQL query by preceding text with `--`:

```sql
-- Hey, I'm a comment!
-- ===========================
SELECT
    name, age, gpa  -- column names
FROM
    Students        -- table name
WHERE
    age > 25;       -- condition
```

<br><br>

- Block comments are also possible by enclosing comment lines in `/*` and `*/`:

```sql
/*
This is our first SQL query, and we
are learning about the following keywords:
SELECT
FROM
WHERE
*/

SELECT
    name, age, gpa
FROM
    Students
WHERE
    age > 25;
```

<br><br>

- Don't forget that every SQL statement needs to be terminated with a `;`.

- SQL keywords are traditionally written in upper case letters, but that is not a requirement. I prefer to follow this tradition because it makes the query more readable.

<br><br>

- A Keyword, together with its following identifiers, expressions, etc. is collectively called a **clause**. For example:

```sql
SELECT
    name, age, gpa  -- columns are chosen here
FROM
    Students        -- table is specified here
WHERE
    age > 25;       -- filter is applied here
```

- It is common to put each clause or each keyword on a different line, but there is no generally agreed-upon style.

- In general, it doesn't matter whether the entire SQL statement is on one line or broken over several lines. Anything that comes before a `;` belongs to the same statement.

<br><br><br>

There are many other keywords that we will use throughout DSCI 513. The ones that you just saw are a few that are usually used when querying data.

> Note that SQL **is not imperative** (like Python or C++); it is a **declarative** language: We don't tell SQL **how** to retrieve data, but **what** to retrieve.
>
> For instance, we didn't write a for loop to retrieve the data from each row according to a certain condition. We told SQL what we wanted, and SQL did it for us.

<br><br><br>

### Flavours of SQL

- SQL is not owned by a particular company or organization

- It became a database language standard by the American National Standards Institute (ANSI) in 1986, and the International Organization for Standardization (ISO) in 1987.
- However, there are various SQL flavors and implementations, such as Oracle SQL, MySQL (open source), PostgreSQL (open source), IBM DB2, Microsoft SQL Server, Microsoft Access, SQLite (open source)
- These implementations have slightly different syntax and various additional features.
- In DSCI 513, we use **PostgreSQL**

<img src="img/lecture1/flavours_sql.png" width="700">

<br><br><br>

## What is PostgreSQL?

[PostgreSQL](https://www.postgresql.org/about/) (also known simply by its nickname _Postgres_) is an open-source, cross-platform DBMS that implements the relational model. PostgreSQL is very reliable with great performance characteristics, and is equipped with almost all features of the commercial and proprietary DBMSs.

PostgreSQL appeared in 1980s as a research project in University of California, Berkeley. It was meant to improve an earlier prototype relational DBMS called INGRES, which explains the name Postgres, which is short for PostINGRES. [Here](https://medium.com/launch-school/a-brief-history-of-postgresql-36d8d392c611) is an informative blog post about PostgreSQL's history if you're interested!

<br><br><br>

## How to run SQL in Postgres?

Well, we have a variety of options to run our SQL statements in PostgreSQL, the most common of which are the following:

- pgAdmin is the official web-based GUI for interacting with PostgreSQL databases

- `psql` is PostgreSQL's interactive command-line interface
- `%sql` and `%%sql` magic commands in Jupyter notebooks, which are provided by the `ipython-sql` package
- `psycopg2` is the official Python adapter for PostgreSQL databases
- Using `.read_sql_query()` method in Pandas

I will demonstrate the usage of all these interfaces here.

### pgAdmin

I will demo this in the lecture.

> I like to use a `Shift + Enter` keyboard shortcut to run my queries. You can configure this too by going Preferences -> Query Tool -> Keyboard shortcuts -> Change "Execute query" to `Shift + Enter`.

### `psql`

This is PostgreSQL's command-line tool that allows us to interactively run
- SQL statements, as well as
- `psql`'s special "meta" commands.

I introduce a couple of useful `psql` meta commands here, but you can find all the other ones in Postgres documentations [here](https://www.postgresql.org/docs/current/app-psql.html) or a shorter version in this [cheatsheet](http://www.postgresonline.com/downloads/special_feature/postgresql83_psql_cheatsheet.pdf).

| Command | Usage                                         |
|---------|-----------------------------------------------|
| `\l`    | list all databases                            |
| `\c`    | connect to a database                         |
| `\d`    | describe tables and views                     |
| `\dt`   | list tables                                   |
| `\dt+`  | list tables with additional info              |
| `\d+`   | list tables and views with additional info    |
| `\!`    | execute shell commands                        |
| `\cd`   | change directory                              |
| `\i`    | execute commands from file                    |
| `\h`    | view help on SQL commands                     |
| `\?`    | view help on psql meta commands               |
| `\q`    | quit interactive shell                        |

> Note that you don't need to terminate meta commands with `;`.

### `ipython-sql` (`%sql` and `%%sql` magics)

`ipython-sql` is a package that enables us to run SQL statements right from a Jupyter notebook. This package is included in the [`dsci513env.yaml`](../dsci513env.yml) environment file, so you should have it installed in your conda environment. In order to use it, we should load it first:


```python
%load_ext sql
```

Now we need the host address of where the database is stored, along with a username and a password.

It is always a bad idea to store login information directly in a notebook or code file because of security reasons. For example, you don't want to commit your sensitive login information to a Git repo.

In order to avoid that, we store that kind of information in a separate file, like `credentials.json` here, and read the username and password into our IPython session:


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

And also make sure to add your file name (e.g. `credentials.json`) to your `.gitignore` file, so you don't accidentally commit it.

Now we can establish the connection to the `world` database using the following code:


```python
%sql postgresql://{username}:{password}@{host}:{port}/world
```

    MetaData.__init__() got an unexpected keyword argument 'bind'
    Connection info needed in SQLAlchemy format, example:
                   postgresql://username:password@hostname/dbname
                   or an existing connection: dict_keys([])
    

Note that we have used the `%sql` line magic to interpret the line in front of it as a magic command. This is similar to the `%timit` magic that we used in DSCI 511 and 512.

We can also use `%%sql` cell magic to apply the magic to an entire notebook cell.

<br><br><br>

Let's run some SQL statements now. Let's retrieve the `name` and `population` columns from the `country` table:


```python
%sql SELECT name, population FROM country;
```

<br><br><br>

#### Limiting returned and displayed rows

As you can see, all rows are returned and displayed by default. This behaviour can be problematic if our table is very large for two reasons:
1. Retrieving large tables can be slow, and maybe not necessary
2. Displaying a lot of rows clutters our Jupyter notebook

We can modifying `ipython-sql` configuration to limit the number of returned and displayed rows. For example, here we change the display limit:


```python
%config SqlMagic.displaylimit = 20
```


```python
%sql SELECT name, population FROM country;
```

     * postgresql://postgres:***@localhost:5432/world
    239 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>population</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Afghanistan</td>
            <td>22720000</td>
        </tr>
        <tr>
            <td>Netherlands</td>
            <td>15864000</td>
        </tr>
        <tr>
            <td>Netherlands Antilles</td>
            <td>217000</td>
        </tr>
        <tr>
            <td>Albania</td>
            <td>3401200</td>
        </tr>
        <tr>
            <td>Algeria</td>
            <td>31471000</td>
        </tr>
        <tr>
            <td>American Samoa</td>
            <td>68000</td>
        </tr>
        <tr>
            <td>Andorra</td>
            <td>78000</td>
        </tr>
        <tr>
            <td>Angola</td>
            <td>12878000</td>
        </tr>
        <tr>
            <td>Anguilla</td>
            <td>8000</td>
        </tr>
        <tr>
            <td>Antigua and Barbuda</td>
            <td>68000</td>
        </tr>
        <tr>
            <td>United Arab Emirates</td>
            <td>2441000</td>
        </tr>
        <tr>
            <td>Argentina</td>
            <td>37032000</td>
        </tr>
        <tr>
            <td>Armenia</td>
            <td>3520000</td>
        </tr>
        <tr>
            <td>Aruba</td>
            <td>103000</td>
        </tr>
        <tr>
            <td>Australia</td>
            <td>18886000</td>
        </tr>
        <tr>
            <td>Azerbaijan</td>
            <td>7734000</td>
        </tr>
        <tr>
            <td>Bahamas</td>
            <td>307000</td>
        </tr>
        <tr>
            <td>Bahrain</td>
            <td>617000</td>
        </tr>
        <tr>
            <td>Bangladesh</td>
            <td>129155000</td>
        </tr>
        <tr>
            <td>Barbados</td>
            <td>270000</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">239 rows, truncated to displaylimit of 20</span>



Looks good. Let's apply the magic to an entire cell so that we can break the lines:


```sql
%%sql

SELECT
    name, population
FROM
    country
;
```

     * postgresql://postgres:***@localhost:5432/world
    239 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>population</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Afghanistan</td>
            <td>22720000</td>
        </tr>
        <tr>
            <td>Netherlands</td>
            <td>15864000</td>
        </tr>
        <tr>
            <td>Netherlands Antilles</td>
            <td>217000</td>
        </tr>
        <tr>
            <td>Albania</td>
            <td>3401200</td>
        </tr>
        <tr>
            <td>Algeria</td>
            <td>31471000</td>
        </tr>
        <tr>
            <td>American Samoa</td>
            <td>68000</td>
        </tr>
        <tr>
            <td>Andorra</td>
            <td>78000</td>
        </tr>
        <tr>
            <td>Angola</td>
            <td>12878000</td>
        </tr>
        <tr>
            <td>Anguilla</td>
            <td>8000</td>
        </tr>
        <tr>
            <td>Antigua and Barbuda</td>
            <td>68000</td>
        </tr>
        <tr>
            <td>United Arab Emirates</td>
            <td>2441000</td>
        </tr>
        <tr>
            <td>Argentina</td>
            <td>37032000</td>
        </tr>
        <tr>
            <td>Armenia</td>
            <td>3520000</td>
        </tr>
        <tr>
            <td>Aruba</td>
            <td>103000</td>
        </tr>
        <tr>
            <td>Australia</td>
            <td>18886000</td>
        </tr>
        <tr>
            <td>Azerbaijan</td>
            <td>7734000</td>
        </tr>
        <tr>
            <td>Bahamas</td>
            <td>307000</td>
        </tr>
        <tr>
            <td>Bahrain</td>
            <td>617000</td>
        </tr>
        <tr>
            <td>Bangladesh</td>
            <td>129155000</td>
        </tr>
        <tr>
            <td>Barbados</td>
            <td>270000</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">239 rows, truncated to displaylimit of 20</span>



<br><br><br>

We can use `*` to retrieve all columns:


```sql
%%sql

SELECT
    *
FROM
    country
;
```

     * postgresql://postgres:***@localhost:5432/world
    239 rows affected.
    




<table>
    <thead>
        <tr>
            <th>code</th>
            <th>name</th>
            <th>continent</th>
            <th>region</th>
            <th>surfacearea</th>
            <th>indepyear</th>
            <th>population</th>
            <th>lifeexpectancy</th>
            <th>gnp</th>
            <th>gnpold</th>
            <th>localname</th>
            <th>governmentform</th>
            <th>headofstate</th>
            <th>capital</th>
            <th>code2</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>AFG</td>
            <td>Afghanistan</td>
            <td>Asia</td>
            <td>Southern and Central Asia</td>
            <td>652090.0</td>
            <td>1919</td>
            <td>22720000</td>
            <td>45.9</td>
            <td>5976.00</td>
            <td>None</td>
            <td>Afganistan/Afqanestan</td>
            <td>Islamic Emirate</td>
            <td>Mohammad Omar</td>
            <td>1</td>
            <td>AF</td>
        </tr>
        <tr>
            <td>NLD</td>
            <td>Netherlands</td>
            <td>Europe</td>
            <td>Western Europe</td>
            <td>41526.0</td>
            <td>1581</td>
            <td>15864000</td>
            <td>78.3</td>
            <td>371362.00</td>
            <td>360478.00</td>
            <td>Nederland</td>
            <td>Constitutional Monarchy</td>
            <td>Beatrix</td>
            <td>5</td>
            <td>NL</td>
        </tr>
        <tr>
            <td>ANT</td>
            <td>Netherlands Antilles</td>
            <td>North America</td>
            <td>Caribbean</td>
            <td>800.0</td>
            <td>None</td>
            <td>217000</td>
            <td>74.7</td>
            <td>1941.00</td>
            <td>None</td>
            <td>Nederlandse Antillen</td>
            <td>Nonmetropolitan Territory of The Netherlands</td>
            <td>Beatrix</td>
            <td>33</td>
            <td>AN</td>
        </tr>
        <tr>
            <td>ALB</td>
            <td>Albania</td>
            <td>Europe</td>
            <td>Southern Europe</td>
            <td>28748.0</td>
            <td>1912</td>
            <td>3401200</td>
            <td>71.6</td>
            <td>3205.00</td>
            <td>2500.00</td>
            <td>Shqipëria</td>
            <td>Republic</td>
            <td>Rexhep Mejdani</td>
            <td>34</td>
            <td>AL</td>
        </tr>
        <tr>
            <td>DZA</td>
            <td>Algeria</td>
            <td>Africa</td>
            <td>Northern Africa</td>
            <td>2381741.0</td>
            <td>1962</td>
            <td>31471000</td>
            <td>69.7</td>
            <td>49982.00</td>
            <td>46966.00</td>
            <td>Al-Jazair/Algérie</td>
            <td>Republic</td>
            <td>Abdelaziz Bouteflika</td>
            <td>35</td>
            <td>DZ</td>
        </tr>
        <tr>
            <td>ASM</td>
            <td>American Samoa</td>
            <td>Oceania</td>
            <td>Polynesia</td>
            <td>199.0</td>
            <td>None</td>
            <td>68000</td>
            <td>75.1</td>
            <td>334.00</td>
            <td>None</td>
            <td>Amerika Samoa</td>
            <td>US Territory</td>
            <td>George W. Bush</td>
            <td>54</td>
            <td>AS</td>
        </tr>
        <tr>
            <td>AND</td>
            <td>Andorra</td>
            <td>Europe</td>
            <td>Southern Europe</td>
            <td>468.0</td>
            <td>1278</td>
            <td>78000</td>
            <td>83.5</td>
            <td>1630.00</td>
            <td>None</td>
            <td>Andorra</td>
            <td>Parliamentary Coprincipality</td>
            <td></td>
            <td>55</td>
            <td>AD</td>
        </tr>
        <tr>
            <td>AGO</td>
            <td>Angola</td>
            <td>Africa</td>
            <td>Central Africa</td>
            <td>1246700.0</td>
            <td>1975</td>
            <td>12878000</td>
            <td>38.3</td>
            <td>6648.00</td>
            <td>7984.00</td>
            <td>Angola</td>
            <td>Republic</td>
            <td>José Eduardo dos Santos</td>
            <td>56</td>
            <td>AO</td>
        </tr>
        <tr>
            <td>AIA</td>
            <td>Anguilla</td>
            <td>North America</td>
            <td>Caribbean</td>
            <td>96.0</td>
            <td>None</td>
            <td>8000</td>
            <td>76.1</td>
            <td>63.20</td>
            <td>None</td>
            <td>Anguilla</td>
            <td>Dependent Territory of the UK</td>
            <td>Elisabeth II</td>
            <td>62</td>
            <td>AI</td>
        </tr>
        <tr>
            <td>ATG</td>
            <td>Antigua and Barbuda</td>
            <td>North America</td>
            <td>Caribbean</td>
            <td>442.0</td>
            <td>1981</td>
            <td>68000</td>
            <td>70.5</td>
            <td>612.00</td>
            <td>584.00</td>
            <td>Antigua and Barbuda</td>
            <td>Constitutional Monarchy</td>
            <td>Elisabeth II</td>
            <td>63</td>
            <td>AG</td>
        </tr>
        <tr>
            <td>ARE</td>
            <td>United Arab Emirates</td>
            <td>Asia</td>
            <td>Middle East</td>
            <td>83600.0</td>
            <td>1971</td>
            <td>2441000</td>
            <td>74.1</td>
            <td>37966.00</td>
            <td>36846.00</td>
            <td>Al-Imarat al-´Arabiya al-Muttahida</td>
            <td>Emirate Federation</td>
            <td>Zayid bin Sultan al-Nahayan</td>
            <td>65</td>
            <td>AE</td>
        </tr>
        <tr>
            <td>ARG</td>
            <td>Argentina</td>
            <td>South America</td>
            <td>South America</td>
            <td>2780400.0</td>
            <td>1816</td>
            <td>37032000</td>
            <td>75.1</td>
            <td>340238.00</td>
            <td>323310.00</td>
            <td>Argentina</td>
            <td>Federal Republic</td>
            <td>Fernando de la Rúa</td>
            <td>69</td>
            <td>AR</td>
        </tr>
        <tr>
            <td>ARM</td>
            <td>Armenia</td>
            <td>Asia</td>
            <td>Middle East</td>
            <td>29800.0</td>
            <td>1991</td>
            <td>3520000</td>
            <td>66.4</td>
            <td>1813.00</td>
            <td>1627.00</td>
            <td>Hajastan</td>
            <td>Republic</td>
            <td>Robert Kotarjan</td>
            <td>126</td>
            <td>AM</td>
        </tr>
        <tr>
            <td>ABW</td>
            <td>Aruba</td>
            <td>North America</td>
            <td>Caribbean</td>
            <td>193.0</td>
            <td>None</td>
            <td>103000</td>
            <td>78.4</td>
            <td>828.00</td>
            <td>793.00</td>
            <td>Aruba</td>
            <td>Nonmetropolitan Territory of The Netherlands</td>
            <td>Beatrix</td>
            <td>129</td>
            <td>AW</td>
        </tr>
        <tr>
            <td>AUS</td>
            <td>Australia</td>
            <td>Oceania</td>
            <td>Australia and New Zealand</td>
            <td>7741220.0</td>
            <td>1901</td>
            <td>18886000</td>
            <td>79.8</td>
            <td>351182.00</td>
            <td>392911.00</td>
            <td>Australia</td>
            <td>Constitutional Monarchy, Federation</td>
            <td>Elisabeth II</td>
            <td>135</td>
            <td>AU</td>
        </tr>
        <tr>
            <td>AZE</td>
            <td>Azerbaijan</td>
            <td>Asia</td>
            <td>Middle East</td>
            <td>86600.0</td>
            <td>1991</td>
            <td>7734000</td>
            <td>62.9</td>
            <td>4127.00</td>
            <td>4100.00</td>
            <td>Azärbaycan</td>
            <td>Federal Republic</td>
            <td>Heydär Äliyev</td>
            <td>144</td>
            <td>AZ</td>
        </tr>
        <tr>
            <td>BHS</td>
            <td>Bahamas</td>
            <td>North America</td>
            <td>Caribbean</td>
            <td>13878.0</td>
            <td>1973</td>
            <td>307000</td>
            <td>71.1</td>
            <td>3527.00</td>
            <td>3347.00</td>
            <td>The Bahamas</td>
            <td>Constitutional Monarchy</td>
            <td>Elisabeth II</td>
            <td>148</td>
            <td>BS</td>
        </tr>
        <tr>
            <td>BHR</td>
            <td>Bahrain</td>
            <td>Asia</td>
            <td>Middle East</td>
            <td>694.0</td>
            <td>1971</td>
            <td>617000</td>
            <td>73.0</td>
            <td>6366.00</td>
            <td>6097.00</td>
            <td>Al-Bahrayn</td>
            <td>Monarchy (Emirate)</td>
            <td>Hamad ibn Isa al-Khalifa</td>
            <td>149</td>
            <td>BH</td>
        </tr>
        <tr>
            <td>BGD</td>
            <td>Bangladesh</td>
            <td>Asia</td>
            <td>Southern and Central Asia</td>
            <td>143998.0</td>
            <td>1971</td>
            <td>129155000</td>
            <td>60.2</td>
            <td>32852.00</td>
            <td>31966.00</td>
            <td>Bangladesh</td>
            <td>Republic</td>
            <td>Shahabuddin Ahmad</td>
            <td>150</td>
            <td>BD</td>
        </tr>
        <tr>
            <td>BRB</td>
            <td>Barbados</td>
            <td>North America</td>
            <td>Caribbean</td>
            <td>430.0</td>
            <td>1966</td>
            <td>270000</td>
            <td>73.0</td>
            <td>2223.00</td>
            <td>2186.00</td>
            <td>Barbados</td>
            <td>Constitutional Monarchy</td>
            <td>Elisabeth II</td>
            <td>174</td>
            <td>BB</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">239 rows, truncated to displaylimit of 20</span>



<br><br><br>

#### Assigning returned rows to Python variables (time permitting)

Single line queries:


```python
query_output = %sql SELECT name, population FROM country;
```

     * postgresql://postgres:***@localhost:5432/world
    239 rows affected.
    

Multi-line queries:


```sql
%%sql query_output <<

SELECT
    name, population
FROM
    country
;
```

     * postgresql://postgres:***@localhost:5432/world
    239 rows affected.
    Returning data to local variable query_output
    


```python
query_output
```




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>population</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Afghanistan</td>
            <td>22720000</td>
        </tr>
        <tr>
            <td>Netherlands</td>
            <td>15864000</td>
        </tr>
        <tr>
            <td>Netherlands Antilles</td>
            <td>217000</td>
        </tr>
        <tr>
            <td>Albania</td>
            <td>3401200</td>
        </tr>
        <tr>
            <td>Algeria</td>
            <td>31471000</td>
        </tr>
        <tr>
            <td>American Samoa</td>
            <td>68000</td>
        </tr>
        <tr>
            <td>Andorra</td>
            <td>78000</td>
        </tr>
        <tr>
            <td>Angola</td>
            <td>12878000</td>
        </tr>
        <tr>
            <td>Anguilla</td>
            <td>8000</td>
        </tr>
        <tr>
            <td>Antigua and Barbuda</td>
            <td>68000</td>
        </tr>
        <tr>
            <td>United Arab Emirates</td>
            <td>2441000</td>
        </tr>
        <tr>
            <td>Argentina</td>
            <td>37032000</td>
        </tr>
        <tr>
            <td>Armenia</td>
            <td>3520000</td>
        </tr>
        <tr>
            <td>Aruba</td>
            <td>103000</td>
        </tr>
        <tr>
            <td>Australia</td>
            <td>18886000</td>
        </tr>
        <tr>
            <td>Azerbaijan</td>
            <td>7734000</td>
        </tr>
        <tr>
            <td>Bahamas</td>
            <td>307000</td>
        </tr>
        <tr>
            <td>Bahrain</td>
            <td>617000</td>
        </tr>
        <tr>
            <td>Bangladesh</td>
            <td>129155000</td>
        </tr>
        <tr>
            <td>Barbados</td>
            <td>270000</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">239 rows, truncated to displaylimit of 20</span>



Although this looks like a dataframe, `query_output` is not a dataframe:


```python
type(query_output)
```




    sql.run.ResultSet



But we can easily convert it to a Pandas dataframe:


```python
df = query_output.DataFrame()
```


```python
type(df)
```




    pandas.core.frame.DataFrame




```python
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>population</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Afghanistan</td>
      <td>22720000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Netherlands</td>
      <td>15864000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Netherlands Antilles</td>
      <td>217000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Albania</td>
      <td>3401200</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Algeria</td>
      <td>31471000</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>234</th>
      <td>British Indian Ocean Territory</td>
      <td>0</td>
    </tr>
    <tr>
      <th>235</th>
      <td>South Georgia and the South Sandwich Islands</td>
      <td>0</td>
    </tr>
    <tr>
      <th>236</th>
      <td>Heard Island and McDonald Islands</td>
      <td>0</td>
    </tr>
    <tr>
      <th>237</th>
      <td>French Southern territories</td>
      <td>0</td>
    </tr>
    <tr>
      <th>238</th>
      <td>United States Minor Outlying Islands</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>239 rows × 2 columns</p>
</div>




```python
df.loc[df['name'] == 'Canada', 'population']
```




    88    31147000
    Name: population, dtype: int64



If you want the result of every query to be automatically converted to a Pandas dataframe, there is an option for that in `ipython-sql`:


```python
%config SqlMagic.autopandas = True
```


```python
new_query = %sql SELECT name, population FROM country;
new_query
```

     * postgresql://postgres:***@localhost:5432/world
    239 rows affected.
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>population</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Afghanistan</td>
      <td>22720000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Netherlands</td>
      <td>15864000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Netherlands Antilles</td>
      <td>217000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Albania</td>
      <td>3401200</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Algeria</td>
      <td>31471000</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>234</th>
      <td>British Indian Ocean Territory</td>
      <td>0</td>
    </tr>
    <tr>
      <th>235</th>
      <td>South Georgia and the South Sandwich Islands</td>
      <td>0</td>
    </tr>
    <tr>
      <th>236</th>
      <td>Heard Island and McDonald Islands</td>
      <td>0</td>
    </tr>
    <tr>
      <th>237</th>
      <td>French Southern territories</td>
      <td>0</td>
    </tr>
    <tr>
      <th>238</th>
      <td>United States Minor Outlying Islands</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>239 rows × 2 columns</p>
</div>




```python
type(new_query)
```




    pandas.core.frame.DataFrame



Now going back to the default:


```python
%config SqlMagic.autopandas = False
```

<br><br><br>

#### Embedding variables

Much like when we embed variables in strings in Python using f-strings, we can do the same in `ipython-sql` by preceding the variable name with a `:`, i.e. a colon:


```python
loc1 = 'Canada'

%sql SELECT name, population FROM country WHERE name = :loc1
```

     * postgresql://postgres:***@localhost:5432/world
    1 rows affected.
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>population</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Canada</td>
      <td>31147000</td>
    </tr>
  </tbody>
</table>
</div>



<br><br><br>

## More SQL commands

### `DISTINCT`

The `DISTINCT` keyword is used to return only distinct rows from a table, and ignore duplicates:

```sql
SELECT
    DISTINCT column1, column2, ...
FROM
    table1;
```

<br><br><br>

Note that `DISTINCT` is applied to **all columns that we list in front of `SELECT`**, and returns all distinct combinations of values stored in those columns. In the above code snippet, columns other than `column1` and `column2` can still have duplicate values.


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




```sql
%%sql

SELECT
    DISTINCT continent, region
FROM
    country
;
```

     * postgresql://postgres:***@localhost:5432/world
    25 rows affected.
    




<table>
    <thead>
        <tr>
            <th>continent</th>
            <th>region</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Oceania</td>
            <td>Melanesia</td>
        </tr>
        <tr>
            <td>Oceania</td>
            <td>Australia and New Zealand</td>
        </tr>
        <tr>
            <td>North America</td>
            <td>Central America</td>
        </tr>
        <tr>
            <td>Africa</td>
            <td>Northern Africa</td>
        </tr>
        <tr>
            <td>Asia</td>
            <td>Eastern Asia</td>
        </tr>
        <tr>
            <td>Oceania</td>
            <td>Polynesia</td>
        </tr>
        <tr>
            <td>Europe</td>
            <td>Nordic Countries</td>
        </tr>
        <tr>
            <td>Asia</td>
            <td>Middle East</td>
        </tr>
        <tr>
            <td>Oceania</td>
            <td>Micronesia/Caribbean</td>
        </tr>
        <tr>
            <td>Europe</td>
            <td>Baltic Countries</td>
        </tr>
        <tr>
            <td>Asia</td>
            <td>Southern and Central Asia</td>
        </tr>
        <tr>
            <td>Africa</td>
            <td>Southern Africa</td>
        </tr>
        <tr>
            <td>Africa</td>
            <td>Western Africa</td>
        </tr>
        <tr>
            <td>Oceania</td>
            <td>Micronesia</td>
        </tr>
        <tr>
            <td>Europe</td>
            <td>British Islands</td>
        </tr>
        <tr>
            <td>North America</td>
            <td>Caribbean</td>
        </tr>
        <tr>
            <td>Asia</td>
            <td>Southeast Asia</td>
        </tr>
        <tr>
            <td>Africa</td>
            <td>Central Africa</td>
        </tr>
        <tr>
            <td>Europe</td>
            <td>Western Europe</td>
        </tr>
        <tr>
            <td>Europe</td>
            <td>Southern Europe</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">25 rows, truncated to displaylimit of 20</span>



<br><br><br>

### `ORDER BY`

The `ORDER BY` keyword sorts the results according to one or particular set of columns:

```sql
SELECT
    column1, column2, ...
FROM
    table1
ORDER BY
    column1, column2, ...;
```

The rows are sorted in **ascending** order by default.


```sql
%%sql

SELECT
    name, population
FROM
    country
ORDER
    BY population
;
```

     * postgresql://postgres:***@localhost:5432/world
    239 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>population</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Heard Island and McDonald Islands</td>
            <td>0</td>
        </tr>
        <tr>
            <td>United States Minor Outlying Islands</td>
            <td>0</td>
        </tr>
        <tr>
            <td>South Georgia and the South Sandwich Islands</td>
            <td>0</td>
        </tr>
        <tr>
            <td>Antarctica</td>
            <td>0</td>
        </tr>
        <tr>
            <td>Bouvet Island</td>
            <td>0</td>
        </tr>
        <tr>
            <td>British Indian Ocean Territory</td>
            <td>0</td>
        </tr>
        <tr>
            <td>French Southern territories</td>
            <td>0</td>
        </tr>
        <tr>
            <td>Pitcairn</td>
            <td>50</td>
        </tr>
        <tr>
            <td>Cocos (Keeling) Islands</td>
            <td>600</td>
        </tr>
        <tr>
            <td>Holy See (Vatican City State)</td>
            <td>1000</td>
        </tr>
        <tr>
            <td>Niue</td>
            <td>2000</td>
        </tr>
        <tr>
            <td>Tokelau</td>
            <td>2000</td>
        </tr>
        <tr>
            <td>Falkland Islands</td>
            <td>2000</td>
        </tr>
        <tr>
            <td>Norfolk Island</td>
            <td>2000</td>
        </tr>
        <tr>
            <td>Christmas Island</td>
            <td>2500</td>
        </tr>
        <tr>
            <td>Svalbard and Jan Mayen</td>
            <td>3200</td>
        </tr>
        <tr>
            <td>Saint Helena</td>
            <td>6000</td>
        </tr>
        <tr>
            <td>Saint Pierre and Miquelon</td>
            <td>7000</td>
        </tr>
        <tr>
            <td>Anguilla</td>
            <td>8000</td>
        </tr>
        <tr>
            <td>Montserrat</td>
            <td>11000</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">239 rows, truncated to displaylimit of 20</span>



We can also sort the returned rows in descending order by adding the keyword `DESC` keyword after the column names. In fact, there is a `ASC` keyword as well for ascending sorting, which is optional:

```sql
SELECT
    column1, column2, ...
FROM
    table1
ORDER BY
    column1 [ASC|DESC], column2 [ASC|DESC], ...;
```


```sql
%%sql

SELECT
    name, population
FROM
    country
ORDER BY
    population DESC
;
```

     * postgresql://postgres:***@localhost:5432/world
    239 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>population</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>China</td>
            <td>1277558000</td>
        </tr>
        <tr>
            <td>India</td>
            <td>1013662000</td>
        </tr>
        <tr>
            <td>United States</td>
            <td>278357000</td>
        </tr>
        <tr>
            <td>Indonesia</td>
            <td>212107000</td>
        </tr>
        <tr>
            <td>Brazil</td>
            <td>170115000</td>
        </tr>
        <tr>
            <td>Pakistan</td>
            <td>156483000</td>
        </tr>
        <tr>
            <td>Russian Federation</td>
            <td>146934000</td>
        </tr>
        <tr>
            <td>Bangladesh</td>
            <td>129155000</td>
        </tr>
        <tr>
            <td>Japan</td>
            <td>126714000</td>
        </tr>
        <tr>
            <td>Nigeria</td>
            <td>111506000</td>
        </tr>
        <tr>
            <td>Mexico</td>
            <td>98881000</td>
        </tr>
        <tr>
            <td>Germany</td>
            <td>82164700</td>
        </tr>
        <tr>
            <td>Vietnam</td>
            <td>79832000</td>
        </tr>
        <tr>
            <td>Philippines</td>
            <td>75967000</td>
        </tr>
        <tr>
            <td>Egypt</td>
            <td>68470000</td>
        </tr>
        <tr>
            <td>Iran</td>
            <td>67702000</td>
        </tr>
        <tr>
            <td>Turkey</td>
            <td>66591000</td>
        </tr>
        <tr>
            <td>Ethiopia</td>
            <td>62565000</td>
        </tr>
        <tr>
            <td>Thailand</td>
            <td>61399000</td>
        </tr>
        <tr>
            <td>United Kingdom</td>
            <td>59623400</td>
        </tr>
    </tbody>
</table>
<span style="font-style:italic;text-align:center;">239 rows, truncated to displaylimit of 20</span>



<br><br><br>

### `LIMIT`

We've already talked about how we can limit the number of returned rows from the database using `ipython-sql`'s configuration, but that is specific to `ipython-sql` extension. With SQL in general, we can use the `LIMIT` keyword to limit the number of returned rows:

```sql
SELECT
    column1, column2, ...
FROM
    table1
LIMIT
    N_ROWS;
```


```sql
%%sql

SELECT
    name, continent
FROM
    country
LIMIT
    5
;
```

     * postgresql://postgres:***@localhost:5432/world_dsci513
    5 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>continent</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Afghanistan</td>
            <td>Asia</td>
        </tr>
        <tr>
            <td>Netherlands</td>
            <td>Europe</td>
        </tr>
        <tr>
            <td>Netherlands Antilles</td>
            <td>North America</td>
        </tr>
        <tr>
            <td>Albania</td>
            <td>Europe</td>
        </tr>
        <tr>
            <td>Algeria</td>
            <td>Africa</td>
        </tr>
    </tbody>
</table>



It is also possible to skip the first `n` rows by supplying the optional `OFFSET` keyword:

```sql
SELECT
    column1, column2, ...
FROM
    table1
LIMIT
    N_ROWS OFFSET N_OFFSET;
```


```sql
%%sql

SELECT
    name, continent
FROM
    country
LIMIT
    5 OFFSET 10;
```

     * postgresql://postgres:***@localhost:5432/world_dsci513
    5 rows affected.
    




<table>
    <thead>
        <tr>
            <th>name</th>
            <th>continent</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>United Arab Emirates</td>
            <td>Asia</td>
        </tr>
        <tr>
            <td>Argentina</td>
            <td>South America</td>
        </tr>
        <tr>
            <td>Armenia</td>
            <td>Asia</td>
        </tr>
        <tr>
            <td>Aruba</td>
            <td>North America</td>
        </tr>
        <tr>
            <td>Australia</td>
            <td>Oceania</td>
        </tr>
    </tbody>
</table>



<br><br><br>

**The order of SQL clauses does matter**. The correct order for the ones that we've learned so far is:

- `SELECT`
- `FROM`
- `WHERE`
- `ORDER BY`
- `LIMIT`

More on this in the next lectures.

<br><br><br>

## (optional) SQL-Pandas similarity

|SQL Command|Functionality|Example|Pandas Equivalent|
|---|---|---|---|
|`SELECT`|Extracts data from a database|`SELECT surname, email FROM info;`|`df[["surname", "email"]]`|
|`LIMIT`|Limits the number of rows returned|`SELECT * FROM info`<br>`LIMIT 5;`|`df.head()`|
|`COUNT()`|Counts how many rows returned, note the parentheses because it's a function|`SELECT COUNT(*) FROM info;`|`df.shape[0]`|
|`SELECT DISTINCT`|Returns only unique values|`SELECT DISTINCT city FROM info;`|`df.drop_duplicates()`|
|`WHERE`|Filters data based on a condition(s) like `>`, `<`, `=`, `!=`, etc.)|`SELECT * FROM info` <br>`WHERE city='Vancouver';`|`df.query("city == 'Vancouver'")`|
|`ORDER BY`|Sorts returned data in ascending (default) or descending order|`SELECT * FROM info` <br> `ORDER BY stu_id` <br>`SELECT * FROM info`<br>`ORDER BY stu_id DESC`|`df.sort_values(by="stu_id")`|
|`MIN()`, `MAX()`, `AVG()`|Performs specified operation on selected data|`SELECT MIN(dsci_511) FROM grades`|`df["dsci_511"].min()`|

More resources on comparison of SQL and Pandas for data retrieval
- [Pandas documentation](https://pandas.pydata.org/docs/getting_started/comparison/comparison_with_sql.html)
- [Pandas cheatsheet for SQL people from Kaggle](https://www.kaggle.com/adilaliyev/pandas-cheatsheet-for-sql-people)
