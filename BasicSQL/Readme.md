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
```

Example
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
