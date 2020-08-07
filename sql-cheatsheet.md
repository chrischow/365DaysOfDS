# SQL Cheatsheet

## Creating Tables

```sql
-- Generic Example
CREATE TABLE [Table Name] (
    -- [Column Name] [Data Type] [[Options: PRIMARY KEY | NOT NULL | UNIQUE]],
    director_id SERIAL PRIMARY KEY,
    first_name VARCHAR(30),
    last_name VARCHAR(30) NOT NULL,
    date_of_birth DATE,
    nationality VARCHAR(20)
);
```

## Creating Foreign Keys

```sql
CREATE TABLE [Table Name] (
    -- [Column Name] [Data Type] REFERENCES [Other Table] ([Foreign Key])
    revenue_id SERIAL PRIMARY KEY,
    movie_id INT REFERENCES movies (movie_id),
    domestic_takings NUMERIC(6,2),
    international_takings NUMERIC(6,2)
);
```

## Creating Composite Keys

```sql
CREATE TABLE [Table Name] (
    -- PRIMARY KEY ([Column 1, Column 2])
    movie_id INT REFERENCES movies (movie_id),
    actor_id INT REFERENCES actors (actor_id),
    PRIMARY KEY (movie_id, actor_id)
);
```

## Adding Columns

```sql
-- ALTER TABLE [Table Name]
-- ADD COLUMN [Column Name] [Data Type] [[Options: Contraints]]
ALTER TABLE examples
ADD COLUMN nationality VARCHAR(30),
ADD COLUMN age INT NOT NULL;
```

## Altering Column Data Types

```sql
ALTER TABLE examples

-- ALTER COLUMN [Column Name] [New Data Type]
ALTER COLUMN nationality TYPE CHAR(3),
ALTER COLUMN last_name TYPE VARCHAR(50),
ALTER COLUMN email TYPE VARCHAR(80);
```

## Deleting Tables

```sql
-- DROP TABLE [Table Name]
```

## Inserting Data into Tables

```sql
-- INSERT INTO [Table Name] ([Column 1], [Column 2], [Column 3])
-- VALUES ('Value 1', 'Value 2', 'Value 3')
INSERT INTO examples (first_name, last_name, email, nationality, age)
VALUES
	('Frank', 'Lampard', 'franklampard@chelsea.com', 'GBR', 35),
	('Jurgen', 'Klopp', 'jurgenklopp@liverpool.com', 'GER', 45);
```

## Updating Data in Tables
Use primary key as the `WHERE` condition where possible.

```sql
-- UPDATE [Table Name]
-- SET [Column 1] = 'newvalue1', [Column 2] = 'newvalue2
-- WHERE [Column X] = 'value';

UPDATE examples
SET email = 'chrischow@gmail.com', first_name = 'Chrissy'
WHERE example_id = 1;
```

## Deleting Data
```sql
-- DELETE FROM [Table Name]
-- WHERE [Column X] = 'value';
DELETE FROM examples
WHERE example_id = 3;

-- DELETE FROM [Table Name] to delete everything
DELETE FROM examples;
```

## Querying Data

### Basic Queries

```sql
-- All columns, all rows
SELECT * from directors;

-- Specific columns, all rows
SELECT first_name, last_name FROM directors;
```

### Queries with Basic Conditions
Logical operators can work with strings as well: based on alphabetic order.

```sql
/*
SELECT [Column Names] FROM [Table Name]
WHERE
    [Column 1] [[= | > | >= | < | <=]] 'value1' [[AND | OR]]
    [Column 2] [[= | > | >= | < | <=]] 'value2';
*/

SELECT * FROM movies
WHERE
    age_certificate = '15' AND
    movie_lang = 'Chinese';
```

### Queries with Value Lists
```sql
/* 
SELECT [Column 1], [Column 2] FROM [Table Name]
WHERE [Column] [[IN | NOT IN]] ('value1', 'value2');
*/

SELECT * FROM actors
WHERE first_name in ('Bruce', 'John', 'Peter');
```

### Queries on Strings `LIKE` a Pattern
Operators:
* `%`: Any number of characters
* `_`: Exactly one character

```sql
/*
SELECT [Columns] from [Table Name]
WHERE [Column 1] LIKE 'pattern'
*/

SELECT * FROM actors
WHERE first_name LIKE '%rl%';
```

### Queries for `BETWEEN`
This selects records between a range **inclusive of the boundary values**. Works on dates, numerics, and strings. For strings, 'P_' comes after 'P'.

```sql
/*
SELECT [Columns] FROM [Table Name]
WHERE [Column 1] BETWEEN value1 AND value2
*/

SELECT * FROM movies
WHERE release_date BETWEEN '1995-01-01' AND '1999-07-16';

SELECT * FROM movies
WHERE movie_length BETWEEN 90 AND 120;

SELECT * FROM movies
WHERE movie_lang BETWEEN 'E' AND 'P';
```

### Ordering Data
```sql
/*
SELECT [Columns] FROM [Table Name]
ORDER BY [Column 1] [[ASC | DESC]]
*/

SELECT * FROM actors
ORDER BY date_of_birth DESC;
```

### Limiting Results
#### Option 1: `LIMIT` and `OFFSET`
Offsetting shifts the window for the results down.

```sql
/*
SELECT [Columns] from [Table Name]
LIMIT [N] OFFSET [K]
*/

SELECT * FROM movie_revenues
ORDER BY domestic_takings DESC
LIMIT 5 OFFSET 5;
```

#### Option 2: `OFFSET` and `FETCH`
```sql
/*
SELECT [Columns] from [Table Name]
OFFSET [K]
FETCH FIRST [N] ROW ONLY;
*/

SELECT * FROM movie_revenues
ORDER BY domestic_takings DESC
OFFSET 5 ROWS
FETCH FIRST 10 ROW ONLY;
```

### Distinct Results
```sql
/*
SELECT DISTINCT [Column Names] FROM [Table Name];
*/

SELECT DISTINCT movie_lang, age_certificate FROM movies
ORDER BY movie_lang;
```

### Null Values
```sql
/*
SELECT * FROM [Table Name]
WHERE [Column 1] IS [[NOT]] NULL;
*/

SELECT * FROM actors
WHERE date_of_birth IS NOT NULL;
```

### Setting Column Aliases
Aliases:

* Cannot be used in `WHERE` clauses, because these are run before the aliasing
* Can be used in `ORDER BY` clauses, because these are run after the data is queried and after aliases are applied

```sql
/*
SELECT [Column 1] AS [Column 1 Alias] FROM [Table Name];
*/

SELECT last_name, gender AS sex FROM actors
WHERE gender = 'F'
ORDER BY last_name;
```

### Concatenating Strings
```sql
/*
OPTION 1:
SELECT 'String 1' || 'String 2' AS new_string;

OPTION 2:
SELECT CONCAT([Column 1], [Column 2]) AS [New Column Name] FROM [Table Name];

OPTION 3: Concat with separator
SELECT CONCAT_WS(' ', [Column 1], [Column 2]) AS [New Column Name] FROM [Table Name];
*/

-- Option 1
SELECT 'Concatted' || 'String' AS new_string;

-- Option 2
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM actors;

-- Option 3
SELECT CONCAT_WS(' ', first_name, last_name) AS full_name FROM actors;
```

## Aggregating and Grouping Data

### Counts
`NULL` entries are not counted.

```sql
/*
SELECT COUNT([Columns]) FROM [Table Name];
*/

SELECT COUNT(*) FROM movie_revenues;
SELECT COUNT(international_takings) FROM movie_revenues;
```

### Sum

```sql
SELECT SUM(domestic_takings) FROM movie_revenues;
```

### Max/Min
```sql
SELECT MAX(movie_length) FROM movies;
SELECT MIN(date_of_birth) FROM actors;
```

### Group By
```sql
-- Select 
SELECT COUNT(movie_lang) FROM movies
GROUP BY movie_lang;

-- Cannot combine a column with an aggregate
SELECT movie_lang, AVG(movie_length) FROM movies
GROUP BY movie_lang;

-- Can group by multiple columns
SELECT movie_lang, age_certificate, AVG(movie_length) FROM movies
GROUPBY movie_lang, age_certificate;
```

### `HAVING` Clause
* Filters the aggregate number
* Aggregate functions are not allowed in a `WHERE` clause, hence the need for `HAVING`
* Must be used after the `GROUP BY` clause

```sql
/*
SELECT [Column 1], [Column 2] FROM [Table Name]
GROUP BY [Column 1]
HAVING AGGFUN([Column 3]) = value;
*/

SELECT movie_lang, COUNT(movie_lang) FROM movies
GROUP BY movie_lang
HAVING COUNT(movie_lang) > 1;
```

## Mathematical Operators
* Division: Will return an integer if only using integers
* When performing operations on columns, `NULL` values in any column will result in a math operation returning a `NULL` value for that row

```sql
-- Addition
SELECT 1 + 1 AS addition;

-- Subtraction
SELECT 1 - 1 AS subtraction;

-- Multiplication
SELECT 1 * 2 AS multiplication_int;

-- Division (int) - returns 11
SELECT 35 / 3 AS division_int;

-- Division (float) - returns 11.67
SELECT 35.0 / 3 AS division_float

-- Operation on real columns
SELECT movie_id, (domestic_takings + international_takings) AS total_takings FROM movie_revenues;
```

## Joins

### Inner Joins
```sql
/*
SELECT [Table 1].[Column 1], [Table 1].[Column 2], [Table 2].[Column 1] FROM [Table 1]
INNER JOIN [Table 2] ON [Table 1].[Column X] = [Table 2].[Column Y];
*/

SELECT directors.director_id, directors.first_name, directors.last_name, movies.movie_name FROM directors
INNER JOIN movies ON directors.director_id = movies.director_id;
```

### Inner Joins with Aliases
Add aliases by assigning a name immediately after mentioning Table 1 and Table 2.

```sql
/*
SELECT [Alias 1].[Column 1], [Alias 1].[Column 2], [Alias 2].[Column 1] 
FROM [Table 1] [Alias 1] JOIN [Table 2] [Alias 2] ON [Alias 1].[Column X] = [Alias 2].[Column Y];
*/

SELECT d.director_id, d.first_name, d.last_name, mo.movie_name FROM directors d
JOIN movies mo ON d.director_id = mo.director_id;
```

### Inner Joins with `USING`
* Only applicable if Primary Key in Table 1 has the same name as the Foreign Key in Table 2

```sql
/*
SELECT [Alias 1].[Column 1], [Alias 2].[Column 1] 
FROM [Table 1] [Alias 1] JOIN [Table 2] [Alias 2] USING ([Key Column]);
*/

SELECT mo.movie_name, mr.domestic_takings FROM movies mo
JOIN movie_revenues mr USING (movie_id);
```

### Left Joins

```sql
/*
SELECT [Alias 1].[Column 1], [Alias 2].[Column 1] FROM [Table 1] [Alias 1]
LEFT JOIN [Table 2] [Alias 2] ON [Alias 1].[Column X] = [Alias 2].[Column Y];
*/

SELECT d.director_id, d.first_name, d.last_name, mo.movie_name FROM directors d
LEFT JOIN movies mo ON d.director_id = mo.director_id;
```

### Right Joins
```sql
/*
SELECT [Alias 1].[Column 1], [Alias 2].[Column 1] FROM [Table 1] [Alias 1]
RIGHT JOIN [Table 2] [Alias 2] ON [Alias 1].[Column X] = [Alias 2].[Column Y];
*/

SELECT d.director_id, d.first_name, d.last_name, mo.movie_name FROM directors d
RIGHT JOIN movies mo ON d.director_id = mo.director_id;
```

### Full Joins
```sql
/*
SELECT [Alias 1].[Column 1], [Alias 2].[Column 1] FROM [Table 1] [Alias 1]
FULL JOIN [Table 2] [Alias 2] ON [Alias 1].[Column X] = [Alias 2].[Column Y];
*/

SELECT d.director_id, d.first_name, d.last_name, mo.movie_name FROM directors d
FULL JOIN movies mo ON d.director_id = mo.director_id;
```

> 27 Jul 2020

### Joining More Than Two Tables
```sql
/*
SELECT t1.Column1, t2.Column1, t3.Column1 FROM Table1 t1
JOIN Table2 t2 ON t1.ColumnK = t2.ColumnK
JOIN Table3 t3 ON t3.ColumnK = t2.ColumnK
*/

SELECT d.first_name, d.last_name, mo.movie_name, mr.international_takings, mr.domestic_takings
FROM directors d
JOIN movies mo ON d.director_id = mo.director_id
JOIN movie_revenues mr ON mr.movie_id = mo.movie_id;

SELECT ac.first_name, ac.last_name, mo.movie_name FROM actors ac
JOIN movies_actors ma ON ac.actor_id = ma.actor_id
JOIN movies mo ON ma.movie_id = mo.movie_id;

SELECT
	d.first_name as dir_fname, d.last_name as dir_lname,
	mo.movie_name,
	ac.first_name as act_fname, ac.last_name as act_lname,
	mr.international_takings, mr.domestic_takings
FROM directors d
JOIN movies mo ON mo.director_id = d.director_id
JOIN movies_actors ma ON ma.movie_id = mo.movie_id
JOIN actors ac ON ac.actor_id = ma.actor_id
JOIN movie_revenues mr ON mr.movie_id = mo.movie_id;
```

### Union
Rules:
* Must select the same number of columns from each table
* The corresponding columns from each table must have a compatible data type

Removes duplicated values.

```sql
/*
SELECT Column1, Column2 FROM Table1
UNION
SELECT Column1, Column2 FROM Table2
*/

SELECT first_name, last_name, 'director' as category FROM directors
UNION
SELECT first_name, last_name, 'actor' as category FROM actors
ORDER BY last_name;
```

### Union All
The only difference from `UNION` is that this does not remove duplicates.

```sql
/*
SELECT Column1, Column2 FROM Table1
UNION ALL
SELECT Column1, Column2 FROM Table2
*/

SELECT first_name FROM directors
UNION ALL
SELECT first_name FROM actors
ORDER BY first_name;
```

## Subqueries

### Uncorrelated Subqueries
* Inner query can run independently
```sql
/*
SELECT Column1, Column2 FROM Table1
WHERE Column2 > 
    (SELECT AGGFUN(ColumnX) FROM TableX);
*/

-- Use filters containing values from another table without joining them
SELECT movie_name FROM movies
WHERE movie_id IN
(SELECT movie_id FROM movie_revenues
WHERE international_takings > domestic_takings);

SELECT mo.movie_id, mo.movie_name, d.first_name, d.last_name FROM movies mo
JOIN directors d ON d.director_id = mo.director_id
WHERE mo.movie_id IN
(SELECT movie_id FROM movie_revenues
WHERE international_takings > domestic_takings);
```

### Correlated Subqueries
```sql
SELECT mo1.movie_name, mo1.movie_lang, mo1.movie_length FROM movies mo1
WHERE mo1.movie_length = 
(
	SELECT MAX(movie_length) FROM movies mo2
    -- Inner query uses a WHERE clause that links table from inner query to the table in the outer query
	WHERE mo2.movie_lang = mo1.movie_lang
);
```

> 28 Jul 2020

## String Functions

### Upper and Lower Case
```sql
/*
SELECT UPPER('string');
SELECT LOWER('STRING');
SELECT UPPER(Column1) FROM Table1;
SELECT LOWER(Column1) FROM Table1;
*/

-- Need to rename processed text or it will appear as the function name UPPER/LOWER/etc.
SELECT first_name, UPPER(last_name) as last_name FROM actors;
SELECT LOWER(movie_name) as movie_name FROM movies;
```

### Initcap
Capitalises the first letter of each word.

```sql
/*
SELECT INITCAP('string');
SELECT INITCAP(Column1) FROM Table1;
*/

SELECT INITCAP('example string');
SELECT INITCAP(movie_name) as movie_name FROM movies;
```

### Left and Right
```sql
/*
-- Select 4 characters from the left
SELECT LEFT('string', Int)

-- Drop 4 characters from the right
SELECT LEFT('string', -Int)

-- Select 4 characters from the right
SELECT RIGHT('string', Int)

-- Drop 4 characters from the left
SELECT RIGHT('string', -Int)
*/

SELECT LEFT(movie_name, 5) FROM movies;
SELECT RIGHT(first_name, 3) FROM actors;
```

### Reverse
```sql
/*
SELECT REVERSE('string');
SELECT REVERSE(Column1) FROM Table1;
*/

SELECT movie_name, REVERSE(movie_name) FROM movies;
```

### Substring
```sql
/*
SELECT SUBSTRING('string', From, Count);
SELECT SUBSTRING(Column1, From, Count);
*/

SELECT first_name, SUBSTRING(first_name, 3, 4) FROM actors;
```

### Replace
```sql
/*
SELECT REPLACE('source_string', 'old_string', 'new_string');
SELECT REPLACE(Column1, 'old_string', 'new_string') FROM Table1;
*/

SELECT first_name, last_name, REPLACE(gender, 'M', 'Male') FROM actors;

-- Update with REPLACE
UPDATE directors
SET nationality = REPLACE(nationality, 'Brit', 'Engl')
WHERE nationality = 'British';
```

### Split Part (`SPLIT_PART`)
* The delimiter separates the string into multiple fields
* The `FieldNumber` parameter selects the relevant field (see example below)

```sql
/*
SELECT SPLIT_PART('string', 'delimiter', FieldNumber);
SELECT SPLIT_PART(Column1, 'delimiter', FieldNumber) FROM Table1;
*/

-- Returns 'first_name.last_name'
SELECT SPLIT_PART('first_name.last_name@gmail.com', '@', 1)

-- Returns 'gmail.com'
SELECT SPLIT_PART('first_name.last_name@gmail.com', '@', 2)

SELECT
	movie_name,
	SPLIT_PART(movie_name, ' ', 1) AS first_word,
	SPLIT_PART(movie_name, ' ', 2) AS first_word
FROM movies
```

### Casting Non-String Data Types to Strings
```sql
/*
SELECT Column1::DATATYPE FROM Table1;
*/

SELECT
	date_of_birth,
	SPLIT_PART(date_of_birth::TEXT, '-', 1)::INT as year,
	SPLIT_PART(date_of_birth::TEXT, '-', 2)::INT as month,
	SPLIT_PART(date_of_birth::TEXT, '-', 3)::INT as day
FROM directors;
```