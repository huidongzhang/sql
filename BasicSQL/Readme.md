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
- Syntax for creating a table:
```sql
CREATE TABLE tableName (

col1 <dataType> <Restrictions>,

col2 <dataType> <Restrictions>,

col3 <dataType> <Restrictions>,

<Primary Key or Index definitions>);
```
- Syntax for defining a column:
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
- By default, MySQL uses the `latin-1` character set that has an associated default `latin1_swedish_ci` collation. The `ci` in the name implies case insensitive and the accented characters are sorted using Swedish conventions.
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

