### Exploring MySQL
```sql
SHOW DATABASES;
USE mysql;
SHOW CREATE DATABASE mysql;
SHOW TABLES;
DESCRIBE user;
SHOW CREATE TABLE servers;
SHOW COLUMNS FROM servers;
```

### Create Database
```sql
CREATE DATABASE MovieIndustry;
CREATE DATABASE IF NOT EXISTS MovieIndustry;
SHOW DATABASES;
DROP DATABASE MovieIndustry;
```

### Data Types
- Numeric e.g., INT, BIGINT, TINYINT, DECIMAL, etc.

- Date and Time e.g., DATE, TIME, TIMESTAMP, YEAR, etc.

- String e.g., VARCHAR, CHAR, ENUM, SET, BLOB, etc.

- JSON e.g., JSON

- Spatial Data represents the location, size, and shape of an object on planet Earth such as a building, lake, mountain, or township. MySQL also supports spatial data types, e.g., GEOMETRY, POINT, etc.

### Create Table
Syntax for creating a table:
```sql
CREATE TABLE tableName (
col1 <dataType> <Restrictions>,
col2 <dataType> <Restrictions>,
col3 <dataType> <Restrictions>,
<Primary Key or Index definitions>);
```
Syntax for defining a column:
Note that column names are case-insensitive and portable across all operating systems, unlike database names.
```sql
columnName columnType [NOT NULL | NULL][DEFAULT columnValue]
- `NOT NULL` and `DEFAULT` can be used together
```

Example 1
```sql
CREATE TABLE Actors (
FirstName VARCHAR(20),
SecondName VARCHAR(20),
DoB DATE,
Gender ENUM('Male','Female','Transgender'),
MaritalStatus ENUM('Married', 'Divorced', 'Single'),
NetWorthInMillions DECIMAL);

SHOW TABLES;/*show names of all the tables in the database*/

DESC Actors; /*display all the columns of the table and related metadata(data type and nullability)*/
```

Example 2
```sql
CREATE TABLE IF NOT EXISTS Actors (
Id INT AUTO_INCREMENT,
FirstName VARCHAR(20) NOT NULL,
SecondName VARCHAR(20) NOT NULL,
DoB DATE NOT NULL,
Gender ENUM('Male','Female','Transgender') NOT NULL,
MaritalStatus ENUM('Married', 'Divorced', 'Single', 'Unknown') DEFAULT "Unknown",
NetWorthInMillions DECIMAL NOT NULL,
PRIMARY KEY (Id));
```
- `PRIMARY KEY (**)`: Uniquely identify a row by designating a single column or a set of columns.
- `AUTO_INCREMENT`: begins at 1; only one column can be marked as AUTO_increment; can't have a default value; must be indexed.

### Temporary Table
Temporary tables are persisted for the duration of the MySQL monitor session and removed once the session is terminated.

Can be used to work with intermediate data or results.

- Syntax
```sql
CREATE TEMPORARY TABLE tableName (
col1 <dataType> <Restrictions>,
col2 <dataType> <Restrictions>,
col3 <dataType> <Restrictions>,
<Primary Key or Index definitions>);
```

Example: Say we want to capture only the first names of actors in a table. We can create a temporary table and populate it with only the first name as follows:
```sql
CREATE TEMPORARY TABLE ActorNames (FirstName CHAR(20));
```

### Collations & Character Sets
A character set defines what characters MySQL can store.
A collation set decides how strings are ordered.
By default, MySQL uses the `latin-1` character set that has an associated default `latin1_swedish_ci` collation. The `ci` in the name implies case insensitive and the accented characters are sorted using Swedish conventions.
```sql
SHOW CHARACTER SET; /*available character sets on the server*/
SHOW COLLATION; /*list the collation*/
SHOW VARIABLES LIKE "c%"; /*show defaults for the server*/
```
### Inserting Data
Syntax
```sql
INSERT INTO table (col1, col2 … coln)
VALUES (val1, val2, … valn);
```

Example
```sql
-- Query 1
INSERT INTO Actors ( 
FirstName, SecondName, DoB, Gender, MaritalStatus, NetworthInMillions) 
VALUES ("Brad", "Pitt", "1963-12-18", "Male", "Single", 240.00);

-- Query 2
INSERT INTO Actors ( 
FirstName, SecondName, DoB, Gender, MaritalStatus, NetworthInMillions) 
VALUES 
("Jennifer", "Aniston", "1969-11-02", "Female", "Single", 240.00),
("Angelina", "Jolie", "1975-06-04", "Female", "Single", 100.00),
("Johnny", "Depp", "1963-06-09", "Male", "Single", 200.00);

-- Query 3
-- The order of the values should be the same as the order of the columns in the table 
-- or that listed by the describe table query.
INSERT INTO Actors 
VALUES (DEFAULT, "Dream", "Actress", "9999-01-01", "Female", "Single", 000.00);

-- Query 4
INSERT INTO Actors VALUES (NULL, "Reclusive", "Actor", "1980-01-01", "Male", "Single", DEFAULT);

-- Query 5: insert a row with all default values 
-- If a column doesn't have a default value defined, it is assigned NULL as default.
-- The above query will fail if any one of the table columns is specified as not-null. 
-- DEFAULT keyword also comes in handy when working with the TIMESTAMP column. 
-- The default value for a TIMESTAMP column is the current timestamp, 
-- which may be what we want when inserting a new row.
INSERT INTO Actors () VALUES ();

-- Query 6
INSERT INTO Actors SET DoB="1950-12-12", FirstName="Rajnikanth", SecondName="",  Gender="Male", NetWorthInMillions=50,  MaritalStatus="Married";
```
### Querying Data
Syntax
```sql
SELECT col1, col2, … coln
FROM table
WHERE <condition>
```
Example
```sql
-- Query 1
SELECT * from Actors;
-- Query 2
SELECT <columns> FROM <TableName>
-- Query 3
SELECT FirstName, SecondName from Actors;
-- Query 4
SELECT FirstName, SecondName from Actors WHERE FirstName="Travolta";
-- Query 5
SELECT FirstName, SecondName from Actors WHERE FirstName="Brad";
-- Query 6
SELECT FirstName, SecondName from Actors WHERE NetWorthInMillions > 500;
-- Query 7
SELECT FirstName, SecondName from Actors WHERE NetWorthInMillions > 0;
```
|Operator |Purpose|
|:---|:---|
|>	|Greater than operator|
|>=	|Greater than or equal to operator|
|<	|Less than operator|
|<=	|Less than or equal to operator|
|!=	|Not equal operator|
|<>	|Not equal operator|
|<=>	|NULL-safe equal to operator|
|=	|Equal to operator|
|LIKE	|Simple pattern matching|
|NOT LIKE	|Negation of simple pattern matching|
|BETWEEN ... AND ...	|Whether a value is within a range of values|
|NOT BETWEEN ... AND ...	|Whether a value is not within a range of values|
|IN	|Whether a value is within a set of values|
|NOT IN()	|Whether a value is not within a set of values|
|IS	|Test a value against a boolean|
|IS NOT	|Test a value against a boolean|
|IS NULL	|NULL value test|
|IS NOT NULL	|NOT NULL value test|
|ISNULL()	|Test whether the argument is NULL|
|COALESCE()	|Return the first non-NULL argument|
|GREATEST()	|Return the largest argument|
|LEAST()	|Return the smallest argument|
|STRCMP()	|Compare two strings|
|INTERVAL	|Return the index of the argument that is less than the first argument|

### LIKE Operator
Syntax
```sql
SELECT col1, col2, … coln
FROM table
WHERE col3 LIKE "%some-string%"
```
Example
```sql
-- Query 1
SELECT * from Actors WHERE FirstName LIKE "Jen%";
-- Query 2
SELECT * from Actors where FirstName LIKE "Jennifer%";
-- Query 3: this matches all the rows in the table
SELECT * from Actors where FirstName LIKE "%";
-- Query 4: use the underscore _ character to match exactly one character
SELECT * from Actors WHERE FirstName LIKE "_enn%";
-- Query 5: show database whose name starts with M
SHOW DATABASES LIKE "M%";
-- Query 6: show table whose name starts with A
SHOW TABLES LIKE "A%";
```

### Combining Conditions
Syntax
```sql
SELECT col1, col2, … coln
FROM table
WHERE col3 LIKE "%some-string%"
AND
col4 = 55;
```
Example
```sql
-- Query 1
SELECT * FROM Actors WHERE FirstName > "B" AND NetWorthInMillions > 200;
-- Query 2
SELECT * FROM Actors WHERE FirstName > "B" OR NetWorthInMillions > 200;
-- Query 3
SELECT * FROM Actors WHERE (FirstName > 'B' AND FirstName < 'J') OR (SecondName >'I' AND SecondName < 'K');
-- Query 4
-- The rows matching the two conditions in the parentheses are excluded by the NOT 
-- and everything else is included. 
SELECT * FROM Actors WHERE NOT(FirstName > "B" OR NetWorthInMillions > 200);
-- Query 5
SELECT * FROM Actors WHERE NOT NetWorthInMillions = 200;
-- Query 6
-- Applying NOT on a non-zero column value makes it a zero, 
-- and since zero isn't equal to 200, no rows are displayed.
SELECT * FROM Actors WHERE (NOT NetWorthInMillions) = 200;
-- Query 7
-- Exclusive OR returns true when one of the two conditions is true. 
-- If both conditions are true, or both are false, the result of the XOR operations is false.
SELECT * FROM Actors WHERE FirstName > "B" XOR NetWorthInMillions > 200;
```

### ORDER BY
Syntax
```sql
SELECT col1, col2, … coln
FROM table
WHERE col3 LIKE "%some-string%"
ORDER BY col3
```
Example
```sql
-- Query 1
SELECT * FROM Actors ORDER BY FirstName;
-- Query 2
SELECT * FROM Actors ORDER BY FirstName DESC;
-- Query 3
-- If a tie occurs based on the first sort key, it is broken using the second sort key.
SELECT * FROM Actors ORDER BY NetWorthInMillions, FirstName;
-- Query 4
SELECT * FROM Actors ORDER BY NetWorthInMillions, SecondName;
-- Query 5
SELECT * FROM Actors ORDER BY NetWorthInMillions DESC, FirstName ASC;
-- Query 6
SELECT * FROM Actors ORDER BY NetWorthInMillions DESC, FirstName DESC;
-- Query 7
-- MySQL ignores case when comparing strings in the ORDER BY clause, 
-- which implies strings "Kim", "kIm" and "kim" are treated equally. 
-- If we want ASCII comparison we need to specify the BINARY keyword before the sort key.
SELECT * FROM Actors ORDER BY BINARY FirstName;
-- Query 8
SELECT * FROM Actors ORDER BY NetWorthInMillions;
-- Query 9
-- The CAST function allows us to treat a column as a different type.
SELECT * FROM Actors ORDER BY CAST(NetWorthInMillions AS CHAR);
```

### LIMIT
The `LIMIT` clause allows us to restrict the number of rows returned from the result of a select query.

Syntax
```sql
SELECT col1, col2, … coln
FROM table
WHERE col3 LIKE "%some-string%"
ORDER BY col3
LIMIT 10;
```
Example
```sql
-- Query 1
SELECT FirstName, SecondName from Actors ORDER BY NetWorthInMillions DESC LIMIT 3;
-- Query 2
-- retrieve the next 4 richest actors after the top three.
-- Syntax
LIMIT <offset>, <number_of_row_to_print>;
SELECT FirstName, SecondName from Actors ORDER BY NetWorthInMillions DESC LIMIT 4 OFFSET 3;
-- Query 3
-- We can also use the alternative syntax as follows:
SELECT FirstName, SecondName from Actors ORDER BY NetWorthInMillions DESC LIMIT 3,4;
-- Query 4
SELECT FirstName, SecondName from Actors ORDER BY NetWorthInMillions DESC LIMIT 1000 OFFSET 3;
-- Query 5
-- It is the maximum value that can be stored in MySQL’s unsigned BIGINT variable type.
 -- Any value higher than that and MySQL will complain: ERROR
SELECT FirstName, SecondName from Actors ORDER BY NetWorthInMillions DESC LIMIT 18446744073709551616;
```

### Deleting Data
A delete statement deletes an entire row and not individual columns.
Deleting all the rows of a table doesn't delete the table itself.
```sql
DELETE FROM table
WHERE col3 > 5
ORDER BY col1
LIMIT 5;
```
Example
```sql
-- Query 1
-- The string comparison for the FirstName performed by MySQL doesn't take case into account
-- If we specified the WHERE clause with an uppercase as FirstName="PRIYANKA" the row would still be deleted.
DELETE FROM Actors WHERE FirstName="priyanka";
-- Query 2
DELETE FROM Actors WHERE Gender="Male";
-- Query 3
-- Delete the top three actresses by net worth
DELETE FROM Actors ORDER BY NetWorthInMillions DESC LIMIT 3;
-- Query 4
-- Remove all the rows
-- The table can still be queried even after we have removed all the rows from it
DELETE FROM Actors;
```

### TRUNCATE
If we intend to delete all the rows from a table then a faster route is to use the `TRUNCATE` statement. Generally, we don't want to delete all the table rows except in the case of temporary tables. The TRUNCATE statement drops a table and recreates it for faster processing. MySQL doesn't count the number of rows affected and may show the count to be zero or non-zero, but the number doesn't reflect the actual number of rows affected.

Syntax
```sql
TRUNCATE table
```
TRUNCATE doesn't work with locking or transactions and is the equivalent of DELETE when used with InnoDB tables. InnoDB refers to a particular type of database engine

### UPDATE
Change the value of a column for a row or multiple rows.
Two parts: the *matching* phase and then the *modification* phase. 
Syntax
```sql
UPDATE table
SET col1 = val1, col2 = val2, … coln = valn
WHERE <condition>
ORDER BY col5
LIMIT 5
```
Example
```sql
-- Query 1
UPDATE Actors SET NetWorthInMillions=1;
-- Query 2
-- only changes the first 3
UPDATE Actors SET NetWorthInMillions=5 ORDER BY FirstName LIMIT 3;
--Query 3
UPDATE Actors SET NetWorthInMillions=50, MaritalStatus="Single";


```



