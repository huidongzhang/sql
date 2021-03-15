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

### Primary Key and Indexes

Example
```sql
SHOW INDEX FROM Actors;
```
The *cardinality* shows the number of unique values for the primary key. It's also described as an estimate of the number of unique values in the index and may not be exact for smaller tables.

However, if you execute the above query in the console, the cardinality may not come out to be 11. Cardinality is counted based on statistics stored as integers, so the value is not necessarily exact even for small tables.

However, if you execute the following command and then check for cardinality, it should come out to be exact.

All columns that make up the primary key must be non-null.
```sql
ANALYZE TABLE Actors;
SHOW INDEX FROM Actors;
```
There are two kinds of indexes: Clustered Index and Non-clustered Index

#### Clustered Index

In the case of a clustered index, the table rows are sorted and kept in a *B-tree structure* (or an R-tree in the case of a spatial index). 
 - A **page** is the smallest unit of data that a database can write to or read from a disk. A page contains rows and forms the leaf node of the B+ tree. In MySQL the default size of a page is fixed at 16KB though it is configurable.
 - A collection of pages forms an **extent**.
 - A collection of extents forms a **segment**.
 - Segments in turn form a **tablespace**. A tablespace consists of tables and their associated indexes. There is a tablespace called the **system tablespace**, and in older versions of MySQL, all user tables were also part of the system tablespace. With later MySQL versions a configuration can be specified to have a separate tablespace for each user table. 

```sql
INSERT INTO Actors (Id, FirstName, SecondName,DoB, Gender, MaritalStatus, NetWorthInMillions) VALUES (15, "First","Row", "1999-01-01", "Male", "Single",0.00);
INSERT INTO Actors (Id, FirstName, SecondName,DoB, Gender, MaritalStatus, NetWorthInMillions) VALUES (13, "Second","Row", "1999-01-01", "Male", "Single",0.00);
INSERT INTO Actors (Id, FirstName, SecondName,DoB, Gender, MaritalStatus, NetWorthInMillions) VALUES (12, "Third","Row", "1999-01-01", "Male", "Single",0.00);
```
The first row we add appears last and the last row we add appears earlier than the first and the second rows. This is because the rows are retrieved in the order of the index.

#### How rows are stored within each page
The pages of the B-tree split and merge as required. 
- If a page is full and a new key is inserted, the existing page splits. 
- Similarly, if enough rows are deleted from a page it may get merged with another. 
Within a leaf-node page, the records or rows exist as a singly linked-list.
- The linked list enforces the index order on the rows.
- The new row is placed in the free space available within the page and the linked list pointers are manipulated to fix the order.

Every table is stored as a clustered index with the primary key as the sort key in MySQL when the database engine is selected as InnoDB.

A **database engine** is the software module that a database management system uses to create, read, update, and delete data from a database.In case of MySQL, we have **InnoDB** and **MyISAM** as examples of two popular storage engines.
```sql
SHOW ENGINES;
```

#### Primary key
MySQL creates a clustered index on the primary key. 

- If no primary key is defined it looks for the first UNIQUE index with all the columns that form the key set as NOT NULL. 
- If no UNIQUE index is available, MySQL generates a hidden clustered index named `GEN_CLUST_INDEX` on a synthetic column containing row ID values. The rows are ordered by the ID that gets assigned to each row. 
- This also implies that there can only be one clustered index per table as the table rows can only be arranged in one order on the disk. All other indexes are secondary indexes.

#### Non-clustered Index

In the case of the clustered index, we saw that leaf-nodes consisted of pages that contained the actual data. 

On the contrary, in a non-clustered index, the leaf-nodes don't hold the actual data, rather, a pointer to data stored elsewhere on the disk. 

- If the selected database engine is MyISAM then the rows aren't stored in sorted order; rather they appear as a heap without any ordering. 
- Such a table is called a **heap-table** since it contains an unordered pile of data. The **MyISAM** database engine is based on ISAM (Indexed Sequential Access Method), an indexing algorithm developed by IBM that allows retrieving information from large sets of data in a fast way.

#### Cost of Indexing
An index doesn't come for free. 
- It takes up additional disk space.
- It needs to be modified whenever an insert or an update is made to the table.

### Alterations
`ALTER TABLE table`, then:`CHANGE`,`MODIFY`,`ADD`,`DROP`
We can rename tables, add, remove, or rename columns, change type of an existing column, etc.

Syntax
```sql
ALTER TABLE table
CHANGE oldColumnName newColumnName <datatype> <restrictions>;
```

Example
```sql
-- Query 1
-- rename the column FirstName to First_Name
ALTER TABLE Actors CHANGE FirstName First_Name varchar(120);
```

`varchar(n)`
- SQL Server allocates 1 character space as the default value to the varchar column that is defined without any length. 
- In practical scenarios, varchar(n) is used to store variable length value as a string.
- ‘n’ denotes the string length in bytes and it can go up to 8000 characters.

```sql
-- Query 2
-- specify the default value for the column First_Name to be the string "Anonymous"
ALTER TABLE Actors MODIFY First_Name varchar(20) DEFAULT "Anonymous";
-- Query 3
ALTER TABLE Actors CHANGE First_Name First_Name varchar(20) DEFAULT "Anonymous";
```
- Use the `MODIFY` keyword if we wish to alter the type or the clauses for a column.
- We can also use the `CHANGE` statement but that will require us to specify the same column name twice as we aren't renaming the column.

```sql
-- Query 4
-- if we try to change the first name column from type varchar to int, we'll run into an error
ALTER TABLE Actors MODIFY First_Name INT;

-- Query 5
-- we can change the column first name to have a varchar length of 300
ALTER TABLE Actors MODIFY First_Name varchar(300);

-- Query 6
-- add a new column MiddleName
ALTER TABLE Actors ADD MiddleName varchar(100);

-- Query 7
-- remove the newly added column
ALTER TABLE Actors DROP MiddleName;
```
```sql
-- Query 8
-- adds the middle name as the first column
ALTER TABLE Actors ADD MiddleName varchar(100) FIRST;

-- Query 9
-- add it after the date of birth (DoB) column
ALTER TABLE Actors ADD MiddleName varchar(100) AFTER DoB;
```
- Control the position of the new column within the table using the `FIRST` or `AFTER` keyword
- If an index is defined on a column, dropping the column also removes the index, if the index consists of only that one column.

```sql
--Query 10
-- drop the middle name column and recreate it using a slightly different column name
ALTER TABLE Actors DROP MiddleName, ADD Middle_Name varchar(100);
```
- Combine several alterations in a single MySQL statement separated by comma. 
- Combining alterations is much more efficient as it avoids the cost of 
  - creating a new table
  - copying data from the old table to the new
  - dropping the old table
  - renaming the old table to the new table for each alteration.
 -  An alter operation can be expensive if the table needs to be rebuilt.

### Alter Index
We can add, remove, or modify indexes after the application is deployed. Note that modifying indexes doesn't change the data in the table.

Syntax
```sql
ALTER TABLE table
ADD INDEX indexName (col1, col2, … coln);
```

Example
```sql
-- Query 1
ALTER TABLE Actors ADD INDEX nameIndex (FirstName);

-- Query 2
ALTER TABLE Actors ADD INDEX nameIndexWithOnlyTenChars (FirstName(10));

-- Query 3
ALTER TABLE Actors DROP INDEX nameIndex;

-- Query 4
ALTER TABLE Actors DROP PRIMARY KEY;
```
- We can't add a second primary key to a table that already has a primary key. 
- However, we can drop the existing primary key and declare a new one on the table. 
- In the case of the Actors table, we can't drop the primary key ID as it is an auto_increment column and an auto_increment column must also be the primary key.
- Attempting to drop the ID column as the primary key results in an error.
- 
```sql
-- Query 5
CREATE TABLE Movies (Name VARCHAR(100), Released DATE, PRIMARY KEY (Name));
DESC Movies;
ALTER TABLE Movies DROP PRIMARY KEY;
ALTER TABLE Movies ADD PRIMARY KEY (Released);
```
Syntax
```sql
ALTER TABLE oldTableName
RENAME newTableName;
```
### ALTER or DROP
Example
```sql
-- Query 1
ALTER TABLE Actors RENAME ActorsTable;

-- Query 2
DROP TABLE IF EXISTS ActorsTable;

-- Query 3
DROP TABLE IF EXISTS Table1, Table2, Table3;

-- Query 4
DROP DATABASE IF EXISTS MovieIndustry;
```

### Alias
Aliases are like nicknames, a temporary name given to a table or a column to write expressive and readable queries. We can use aliases with columns, tables, and MySQL functions.

Syntax
```sql
SELECT col1
AS aliasCol1
FROM table;
```

Example
```sql
-- Query 1
-- the column heading in the output shows the alias PopularName instead of FirstName column.
SELECT FirstName AS PopularName from Actors;

-- Query 2
-- use the concat function to print the full name
SELECT CONCAT(FirstName,' ', SecondName) AS FullName FROM Actors;

-- Query 3
SELECT CONCAT(FirstName,' ', SecondName) AS FullName FROM Actors ORDER BY FullName;

-- Query 4
-- Without using the alias feature the same query would be written as follows:
SELECT CONCAT(FirstName,' ', SecondName) FROM Actors ORDER BY CONCAT(FirstName,' ', SecondName);
```
- Aliases can be used in `GROUP BY`, `HAVING`, and `ORDER BY` clauses. 
- Notably, aliases for columns can't be used in the `WHERE` clause but aliases for table can, as shown next.
- Table aliases come in handy when working with complex queries that involve joins
```sql
-- Query 5
SELECT FirstName FROM Actors AS tbl WHERE tbl.FirstName='Brad' AND tbl.NetWorthInMillions > 200;
```
We can also use the table alias in the `SELECT` clause before we actually define the alias. We rewrite the previous query and use the alias in the `SELECT` clause as follows:
```sql
-- Query 6
SELECT tbl.FirstName FROM Actors AS tbl WHERE tbl.FirstName='Brad' AND tbl.NetWorthInMillions > 200;
```

#### For some queries table aliases are inevitable
we can alias the Actors table twice to find out all the actors with the same net worth in a single query. 
- Think of picking each row and comparing it with the rest of the rows in the table to find two rows with the same NetWorthInMillions column. 
- - However, the caveat is that we want to skip the row when it tries to match with itself.
```sql
-- Query 7
SELECT t1.FirstName, t1.NetworthInMillions
FROM Actors AS t1,
Actors AS t2
WHERE t1.NetworthInMillions = t2.NetworthInMillions
AND t1.Id != t2.Id;
```

### DISTINCT
- Output unique rows in a result set. 
- Remember DISTINCT is a post processing filter, meaning it is applied to the resulting rows of a query.
Syntax
```sql
SELECT DISTINCT col1
FROM table;
```
Example
```sql
-- Query 1
SELECT DISTINCT MaritalStatus from Actors;
-- Query 2
SELECT DISTINCT MaritalStatus, FirstName from Actors;
```

### Aggregate Methods
Syntax
```sql
SELECT AggregateFunction(col1)
FROM table;
```

Example
```sql
-- Query 1
-- the output of the query is a single value rather than rows.
SELECT COUNT(*) FROM Actors;
-- Query 2
-- add up the numeric values of a column
SELECT SUM(NetworthInMillions) FROM Actors;
-- Query 3
SELECT AVG(NetWorthInMillions) FROM Actors;
-- Query 4
SELECT MIN(NetWorthInMillions) FROM Actors;
-- Query 5
SELECT MAX(NetWorthInMillions) FROM Actors;
-- Query 6
SELECT STDDEV(NetWorthInMillions) FROM Actors;
```
We can also apply the MIN and MAX functions to non-numerical columns such as FirstName. MySQL would return the actor with the first name that occurs first or last.

### GROUP BY
- Sorts rows together into groups. Returns one row for each group. 
- Data is organized using a comma separated list of columns as the criteria specified after the `GROUP BY` clause. 
- Often used with aggregate functions such as `COUNT`, `MAX`, `MIN`, `SUM`, and `AVG` to calculate an aggregated stat for each group.
- Must appear after the `FROM` and `WHERE` clauses and is also evaluated after them. 
- However, `GROUP BY` is evaluated before the ORDER BY, LIMIT, and HAVING clauses.
Syntax
```sql
SELECT col1, AggregateFunction(col3)
FROM table;
GROUP BY col1, col2, … coln
ORDER BY col2;
```
Example
```sql
-- Query 1
SELECT FirstName FROM Actors GROUP BY FirstName;

-- Query 2
-- This returns an error.
SELECT FirstName, SecondName FROM Actors GROUP BY FirstName;
```
- We can't have non-aggregated columns in the `SELECT`, `ORDER BY`, and `HAVING` clauses when these columns don't appear in the `GROUP BY` clause or are functionally dependent on columns that do appear. 
- Though there are exceptions, when the non-aggregated column has a single value it can appear in the `SELECT`, `ORDER BY`, and `HAVING` clauses. 
  - This restriction is ensured by setting `sql_mode` to `only_full_group_by`. 
  - We can unset this setting and retry our query, but the value chosen for SecondName would be arbitrary if multiple actors share the same first name.
```sql
-- Query 3
SELECT Gender, COUNT(*) FROM Actors GROUP BY Gender;

-- Query 4
SELECT Gender FROM Actors GROUP BY Gender;

-- Query 5
SELECT MaritalStatus, AVG(NetworthInMillions) FROM Actors GROUP BY MaritalStatus ORDER BY MaritalStatus ASC;
```








