Databases using SQL
===================

Setup
-----

_Note: this should have been done by participants before the start of the workshop._

1. Install Firefox
2. Install the SQLite Manager add on: **Menu (the three horizontal lines near the
top right corner of Firefox) -> Add-ons -> Search -> SQLite
Manager -> Install -> Restart now**
4. Add SQLite Manager to the menu: **Menu -> Customize, then drag the SQLite
   Manager icon to one of the empty menu squares on the right, Exit Customize**
5. Open SQLite Manager: **Menu -> SQLite Manager**


Relational databases
--------------------

* Relational databases store data in tables with fields (columns) and records
  (rows)
* Data in tables has types, just like in Python, and all values in a field have
  the same type ([list of data types](#datatypes))
* Queries let us look up data or make calculations based on columns


Why use relational databases
----------------------------

* Data separate from analysis.
  * No risk of accidentally changing data when analyzing it
  * If we change the data we can just rerun the query
* Fast for large amounts of data
* Improve quality control of data entry (type constraints and use of forms in
  Access, Filemaker, etc.)
* The concepts of relational database querying are core to understanding how to
  do similar things in R and Python


Database Management Systems
---------------------------

There are a number of different database management systems for working with
relational data. We're going to use SQLite today, but basically everything we
teach you will apply to the other database systems as well (e.g., MySQL,
PostgreSQL, MS Access, Filemaker Pro). The only things that will differ are the
details of exactly how to import and export data and the
[details of data types](#datatypediffs).


Database Design
---------------

1. Every row-column combination contains a single *atomic* value, i.e., not
   containing parts we might want to work with separately.
2. One field per type of information
3. No redundant information
     * Split into separate tables with one table per class of information
	   * Needs an identifier in common between tables – shared column - to
       reconnect (foreign key).


Introduction to SQLite Manager
------------------------------

Let's all open the database we downloaded in SQLite Manager by clicking on the
open file icon.

You can see the tables in the database by looking at the left hand side of the
screen under Tables.

To see the contents of a table, click on that table and then click on the Browse
and search tab in the right hand section of the screen.

If we want to write a query, we click on the Execute SQL tab.


Dataset Description
-------------------

The dataset for this lesson is cleaned Gapminder data from Jenny Bryant
https://github.com/jennybc/gapminder



Import
------

1. Download the two files in a zip file from https://www.dropbox.com/s/vh03r4ayzzc865c/gapminder-sql-data.zip?dl=0
1. Start a New Database **Database -> New Database**
2. Start the import **Database -> Import**
3. Select the file to import - gap-every-five-years.csv
4. Give the table a name that matches the file name. We'll call it **surveys**
5. Here the first row has column headings, so check the 'First row contains column names' box
6. Use the defaults of Fields separated by: comma and Fields enclosed by: double quotes if necessary.
We don't have a column that has unique values, so we don't have a 'Primary key' and SQLite will
create one for us.
7. Press **OK**
8. When asked if you want to modify the table, click **OK**
9. Set the data types for each field: choose TEXT for fields with text
   (`country`) and INTEGER for fields with non-decimal numbers (`year`,
   `pop`) and FLOAT for fields with decimal numbers ('lifeExp', 'gdpPercap')
10. When it says "Are you sure you want to perform the following operation(s):
Import Data: Rows = 1704" click **OK**
11. Then it will say "Import: 1704 rows imported"
12. Click on the 'Browse and Search' tab
13. Click on the 'gpdata' in the Tables section on the left and you can see the
data just imported.

> ### Challenge
>
> Import the country table - countries.csv

In this lesson, we have two tables rather than the one combined table. Rather
than repeating the continent information along with the country in the surveys
table, we have a separate table that contains that information. Then we
can put that information back together whenever we need to by matching
country to continent. We'll go over how to join that information at the end.
This is a very valuable feature of SQL.

You can also use this same approach to append new data to an existing table.


Basic queries
-------------

Let's start by using the **surveys** table.
Here we have data on all the countries and years

Let’s write an SQL query that selects only the year column from the surveys
table.

Go to the 'Execute SQL' tab. In the box type:

    SELECT year FROM surveys;

We have capitalized the words SELECT and FROM because they are SQL keywords.
SQL is case insensitive, but it helps for readability – good style.

If we want more information, we can just add a new column to the list of fields,
right after SELECT:

    SELECT year, pop FROM surveys;

Or we can select all of the columns in a table using the wildcard *

    SELECT * FROM surveys;

### Unique values

If we want only the unique values so that we can quickly see what countries have
been surveyed we use ``DISTINCT``

    SELECT DISTINCT country FROM surveys;

If we select more than one column, then the distinct pairs of values are
returned

    SELECT DISTINCT country, year FROM surveys;

### Calculated values

We can also do calculations with the values in a query.
For example, if we wanted to look at the population in thousands of people, to
make it easier to look at, we could divide population by 1000

    SELECT country, year, pop/1000.0 from surveys;

When we run the query, the expression `pop / 1000.0` is evaluated for each row
and appended to that row, in a new column.  Expressions can use any fields, any
arithmetic operators (+ - * /) and a variety of built-in functions (). For
example, we could round the values of gpdPercap to two decimal values to make them easier to read.

    SELECT country, year, ROUND(gpdPercap, 2) FROM surveys;

We can also rename the column using AS

    SELECT country, year, ROUND(gdpPercap, 2) AS gdp FROM surveys;

> ## Challenge
>
> Write a query that returns the country, year, population in millions of people,
> rounded to two decimal places and life expectancy rounded to one decimal place.
> As a bonus rename the columns.

Filtering
---------

Databases can also filter data – selecting only the data meeting certain
criteria.  For example, let’s say we only want data for the species _Dipodomys
merriami_, which has a species code of DM.  We need to add a WHERE clause to our
query:

    SELECT * FROM surveys WHERE country="United States";

We can do the same thing with numbers.
Here, we only want the data since 2000:

    SELECT * FROM surveys WHERE year >= 2000;

We can use more sophisticated conditions by combining tests with AND and OR.
For example, suppose we want the data on _United States_ starting in the year
2000:

    SELECT * FROM surveys WHERE (year >= 2000) AND (country = "United States");

Note that the parentheses aren’t needed, but again, they help with readability.
They also ensure that the computer combines AND and OR in the way that we
intend.

If we wanted to get data for three countries,
United States, Canada and Mexico we could combine the tests using OR:

    SELECT * FROM surveys WHERE (country = "United States") OR (country = "Canada") OR (country = "Mexico");

> ### Challenge
>
> Write a query that returns the country, year, life expectancy and population in thousands of people
> for any field with a life expectancy greater than 70.
> How many records are there?
> How many records are there if you change lifeExp to 75?
>
> Write a query that returns the country, year, life expectancy and population in thousands of people
> for any field with a life expectancy greater than 70 and before 1990.
> How many records are there?
> How many records are there if you just look at the year 1952?
> How many records are there if you just look at the year 2007?


Saving & Exporting queries
--------------------------

* Exporting:  **Actions** button and choosing **Save Result to File**.
* Save: **View** drop down and **Create View**


Building more complex queries
-----------------------------

Now, lets combine the above queries to get data for the 3 countries from
the year 2000 on.  This time, let’s use IN as one way to make the query easier
to understand.  It is equivalent to saying `WHERE (species_id = "DM") OR (species_id
= "DO") OR (species_id = "DS")`, but reads more neatly:

    SELECT * FROM surveys WHERE (year >= 2000) AND (country IN ("United States", "Canada", "Mexico"));

    SELECT *
    FROM surveys
    WHERE (year >= 2000) AND (country IN ("United States", "Canada", "Mexico"));

We started with something simple, then added more clauses one by one, testing
their effects as we went along.  For complex queries, this is a good strategy,
to make sure you are getting what you want.  Sometimes it might help to take a
subset of the data that you can easily see in a temporary database to practice
your queries on before working on a larger or more complicated database.


Sorting
-------

We can also sort the results of our queries by using ORDER BY.
Let's now use our countries table and put them in order. Here instead
of saying 'FROM surveys' we say 'FROM countries' because we're referring to
that table instead.

    SELECT * FROM countries ORDER BY country ASC;

The keyword ASC tells us to order it in Ascending order.
We could alternately use DESC to get descending order.

    SELECT * FROM countries ORDER BY country DESC;

ASC is the default.

We can also sort on several fields at once.
To truly be alphabetical within each continent, we can order by
continent and then country.

    SELECT * FROM countries ORDER BY continent ASC, country ASC;

> ### Challenge
>
> Write a query that returns country, year, and population in millions of people from
> the surveys table, sorted with the largest populations at the top.


Order of execution
------------------

Another note for ordering. We don’t actually have to display a column to sort by
it.  For example, let’s say we want to look at countries with population higher than
a million and order the countries by their population, but
we only want to see gdpPercap.

    SELECT country, gdpPercap FROM surveys WHERE (pop > 1000000) ORDER BY pop ASC;

We can do this because sorting occurs earlier in the computational pipeline than
field selection.

The computer is basically doing this:

1. Filtering rows according to WHERE
2. Sorting results according to ORDER BY
3. Displaying requested columns or expressions.


Order of clauses
----------------

The order of the clauses when we write a query is dictated by SQL: SELECT, FROM, WHERE, ORDER BY
and we often write each of them on their own line for readability.


> ### Challenge
>
> Let's try to combine what we've learned so far in a single
> query.  Using the surveys table write a query to display country, year
> lifeExp, and population in millions (rounded to two decimal places), for
> the year 2007, ordered by life expectancy with highest life expectancy at
> the top.


**BREAK**


Aggregation
-----------

Aggregation allows us to combine results by grouping records based on value and
calculating combined values in groups.

Let’s go to the surveys table and find out how many individuals there are.
Using the wildcard simply counts the number of records (rows)

    SELECT COUNT(*) FROM surveys

We can also find out the overall population (maybe the number of people
  who have been on earth since 1952?).

    SELECT COUNT(*), SUM(pop) FROM surveys;

***Do you think you could output this value in kilograms, rounded to 3 decimal
   places?***

    SELECT ROUND(SUM(weight)/1000.0, 3) FROM surveys

There are many other aggregate functions included in SQL including
`MAX`, `MIN`, and `AVG`.

***From the surveys table, can we use one query to output the total population,
   average populaton, and the min and max population? How about the range of population?***

Now, let's see how many countries were surveys in each continent We do this
using a GROUP BY clause

    SELECT continent, COUNT(*)
    FROM countries
    GROUP BY continent

GROUP BY tells SQL what field or fields we want to use to aggregate the data.
If we want to group by multiple fields, we give GROUP BY a comma separated list.
Here we just have 12 countries each, but there are

We can order the results of our aggregation by a specific column, including the
aggregated column.  Let’s count the number of countries in each continent,
ordered by the count

    SELECT continent, COUNT(*)
    FROM countries
    GROUP BY continent
    ORDER BY COUNT(continent)


Joins
-----

To combine data from two tables we use the SQL `JOIN` command, which comes after
the `FROM` command.

We also need to tell the computer which columns provide the link between the two
tables using the word `ON`.  What we want is to join the data with the same
species codes.

    SELECT *
    FROM surveys
    JOIN countries ON surveys.country = countries.country

ON is like `WHERE`, it filters things out according to a test condition.  We use
the `table.colname` format to tell the manager what column in which table we are
referring to.

We often won't want all of the fields from both tables, so anywhere we would
have used a field name in a non-join query, we can use `table.colname`.

For example, what if we wanted information on the population of each country
with the continent information.

    SELECT surveys.country, countries.continent, surveys.year, surveys.pop
    FROM surveys
    JOIN countries ON surveys.country = countries.country

> ### Challenge:
>
> Write a query that returns the country, continent, and the life expectancy
> for each record

Joins can be combined with sorting, filtering, and aggregation.  So, let's
combine things as a final challenge.

> #### Challenge
> Write a query that returns the continent, the average life expectancy
> and the maximum population for each continent.  
> Bonus: Do this just for the year 1952 then for the year 2007



Aliases
-------

As queries get more complex names can get long and unwieldy. To help make things
clearer we can use aliases to assign new names to things in the query.

We can alias both table names:

  SELECT surv.year, surv.country
  FROM surveys AS surv
  JOIN countries AS co ON surv.country = co.country

And column names:

  SELECT surv.year AS yr, surv.country AS con
  FROM surveys AS surv
  JOIN countries AS co ON surv.country = co.country

The `AS` isn't technically required, so you could do

  SELECT surv.year yr
  FROM surveys surv

but using `AS` is much clearer so it's good style to include it.

Adding data to existing tables
------------------------------

* Browse & Search -> Add
* Enter data into a csv file and append


Other database management systems
---------------------------------

* Access or Filemaker Pro
    * GUI
    * Forms w/QAQC
	* But not cross-platform
* MySQL/PostgreSQL
    * Multiple simultaneous users
	* More difficult to setup and maintain


Q & A on Database Design (review if time)
-----------------------------------------

1. Order doesn't matter
2. Every row-column combination contains a single *atomic* value, i.e., not
   containing parts we might want to work with separately.
3. One field per type of information
4. No redundant information
     * Split into separate tables with one table per class of information
	 * Needs an identifier in common between tables – shared column - to
       reconnect (foreign key).


<a name="datatypes"></a> Data types
-----------------------------------

| Data type  | Description |
| :------------- | :------------- |
| CHARACTER(n)  | Character string. Fixed-length n  |
| VARCHAR(n) or CHARACTER VARYING(n) |	Character string. Variable length. Maximum length n |
| BINARY(n) |	Binary string. Fixed-length n |
| BOOLEAN	| Stores TRUE or FALSE values |
| VARBINARY(n) or BINARY VARYING(n) |	Binary string. Variable length. Maximum length n |
| INTEGER(p) |	Integer numerical (no decimal). |
| SMALLINT | 	Integer numerical (no decimal). |
| INTEGER |	Integer numerical (no decimal). |
| BIGINT |	Integer numerical (no decimal). |
| DECIMAL(p,s) |	Exact numerical, precision p, scale s. |
| NUMERIC(p,s) |	Exact numerical, precision p, scale s. (Same as DECIMAL) |
| FLOAT(p) |	Approximate numerical, mantissa precision p. A floating number in base 10 exponential notation. |
| REAL |	Approximate numerical |
| FLOAT |	Approximate numerical |
| DOUBLE PRECISION |	Approximate numerical |
| DATE |	Stores year, month, and day values |
| TIME |	Stores hour, minute, and second values |
| TIMESTAMP |	Stores year, month, day, hour, minute, and second values |
| INTERVAL |	Composed of a number of integer fields, representing a period of time, depending on the type of interval |
| ARRAY |	A set-length and ordered collection of elements |
| MULTISET | 	A variable-length and unordered collection of elements |
| XML |	Stores XML data |


<a name="datatypediffs"></a> SQL Data Type Quick Reference
----------------------------------------------------------

Different databases offer different choices for the data type definition.

The following table shows some of the common names of data types between the various database platforms:

| Data type |	Access |	SQLServer |	Oracle | MySQL | PostgreSQL |
| :------------- | :------------- | :---------------- | :----------------| :----------------| :---------------|
| boolean	| Yes/No |	Bit |	Byte |	N/A	| Boolean |
| integer	| Number (integer) | Int |	Number | Int / Integer	| Int / Integer |
| float	| Number (single)	|Float / Real |	Number |	Float |	Numeric
| currency | Currency |	Money |	N/A |	N/A	| Money |
| string (fixed) | N/A | Char |	Char | Char |	Char |
| string (variable)	| Text (<256) / Memo (65k+)	| Varchar |	Varchar / Varchar2 |	Varchar |	Varchar |
| binary object	OLE Object Memo	Binary (fixed up to 8K) | Varbinary (<8K) | Image (<2GB)	Long | Raw	Blob | Text	Binary | Varbinary |
