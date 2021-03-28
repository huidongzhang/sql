### What are Stored Procedures?
**Procedures** are a simple way to save queries for later access. 

A **stored procedure** can be defined as a set of SQL statements stored in the MySQL server. 
- For example a SELECT statement that returns all the male actors can be turned into a stored procedure. 

A stored procedure can be invoked using `CALL` statement which executes the query and returns the result.
- When the stored procedures are **called**, MySQL **compiles** the code of the procedure and stores it in the cache. 
- Then the code is executed. 
- Any subsequent calls to the same stored procedure result in execution of the **previously compiled code**.

Just like **functions**, we can pass parameters to stored procedures. Stored procedures have conditional statements and loops, and can call other stored procedures.

There are several **advantages** of using stored procedures. 
1. They reduce the traffic between applications and server because instead of sending queries, only the **name** of the stored procedure is sent to the server. 
2. Since a stored procedure can be called by other procedures, it **reduces the duplication of code**. 
3. **Performance gains** are also achieved by using store procedures because the code can be pre-compiled instead of parsing the query every time. 
4. Stored procedures can be used for **security** by giving users access to certain procedures without giving access to the tables.

Stored procedures have certain **limitations**. 
1. Stored procedures are **difficult to debug** as they get automatically executed and it is not possible to apply checkpoints in the code. 
2. Stored procedures incur a **resource overhead** and may result in **overuse of memory and CPU**. There is no way to rollback a change to a stored procedure, once it is made.

### Create, Delte, and Alter a Stored Procedure
To create a stored procedure we use the `CREATE PROCEDURE` statement. 
- `CREATE PROCEDURE` keywords are followed by the procedure name. 
- We can also specify parameters to our procedure as a comma separated list within parenthesis after the procedure name. 
- The body of the stored procedure is enclosed with in the `BEGIN` and `END` keywords.

In order to alter a stored procedure, the `ALTER ROUTINE` privilege is a must. 

The `DROP PROCEDURE` statement deletes a stored procedure from the database. It can be used with the optional `IF EXISTS` clause.

Syntax
```sql
-- create a stored procedure ShowActors that displays all the actors in our database
DELIMITER **
CREATE PROCEDURE procedure_name( parameter_list )
BEGIN
procedure_body
END**
DELIMITER ;

DROP PROCEDURE [IF EXISTS] procedure_name
```

Example
```sql
-- Query 1
DELIMITER **
CREATE PROCEDURE ShowActors()
BEGIN
    SELECT *  FROM Actors;
END **
DELIMITER ;
```
We have used the `DELIMITER` command in line 1. 
- By default semicolon (;) is used to separate two statements. 
- Since a stored procedure can have multiple statements that end with semicolon character, it will not be considered as a single statement. 
- The `DELIMITER` keyword is used to __redefine the delimiter to **__ so that we can pass the whole stored procedure to the server as a single statement. 
- We use our redefined delimiter in line 5 to signal the end of the stored procedure. Then the delimiter is set back to semicolon in the last line.

```sql
-- Query 2
-- invoke a stored procedure, same as executing a query
CALL ShowActors();

-- Query 3
-- view the stored procedures in a database
-- it shows: Db, Name, Typer, Definer, Modified time, Created time, Security_type, Comment and 
-- Description, Parameters, Example...
SHOW PROCEDURE STATUS;

-- Query 4
SHOW PROCEDURE STATUS WHERE db = 'MovieIndustry';

-- Query 5
SELECT routine_name
FROM information_schema.routines
WHERE routine_type = 'PROCEDURE'
    AND routine_schema = 'sys';

-- Query 6        
DROP PROCEDURE IF EXISTS ShowActors;
```

### Variables
A variable is nothing but a data object with a name associated to it. Variables help store a user-defined, temporary value which can be referred to in subsequent statements.

A variable must be declared before it can be used in the code. The `DECLARE` keyword is used to declare variables by providing the data type and an optional default value. If `DEFAULT` is not used at the time of declaration, then the variable will have `NULL` value.

Assignment of a value to a variable is done using the `SET` keyword. Another way to assign values to a variable is to use it in a query with a `SELECT INTO` statement.

The **scope of a variable** defines its lifetime after which it becomes unavailable. The scope of a variable created in a stored procedure is local meaning that it will not be accessible after the `END` statement of the stored procedure. Inside the stored procedure, the scope of a variable depends on where it is declared. This means we can have multiple variables of the same name in a stored procedure as long as they have different scopes.

Syntax
```sql
DECLARE VarName DataType (VarLength) [DEFAULT DefaultValue];

SET VarName = value;

SELECT ColName
INTO VarName
FROM TableName;
```

Example
```sql
-- Query 1
DELIMITER **
CREATE PROCEDURE Summary()
BEGIN
    ``` storing the average net worth of all actors in our Actors table
    ``` The values 6 and 2 in parentheses mean that the number can be 6 digits long 
    ``` and the decimal part will have 2 digits.
    DECLARE AvgNetWorth DEC(6,2) DEFAULT 0.0;
    DECLARE TotalM, TotalF INT DEFAULT 0;
    
    SELECT COUNT(*) INTO TotalM
    FROM Actors
    WHERE Gender = 'Male';
    
    SELECT COUNT(*) INTO TotalF
    FROM Actors
    WHERE Gender = 'Female';
    
    SELECT AVG(NetWorthInMillions) INTO AvgNetWorth
    FROM Actors;
    
    SELECT TotalM, TotalF, AvgNetWorth;
END**
DELIMITER ;

-- Query 2
CALL Summary();

```
