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

-- storing the average net worth of all actors in our Actors table
-- The values 6 and 2 in parentheses mean that the number can be 6 digits long 
-- and the decimal part will have 2 digits.
DECLARE AvgNetWorth DEC(6,2) DEFAULT 0.0;

-- create more than one variable in a single DECLARE statement provided they have the same data type
-- store the total number of males and females from our Actors table in two variables TotalM and TotalF
DECLARE TotalM, TotalF INT DEFAULT 0;

-- assign values to them using SET or SELECT INTO
SET TotalM = 6;
SET TotalF = 4;

-- use the AVG aggregate function to find the average value of 
-- the NetWorthInMillions column of the Actors table 
-- and then assign that value to the AvgNetWorth variable.
SELECT AVG(NetWorthInMillions)
INTO AvgNetWorth
FROM Actors;
```
Define a stored procedure that declares, assigns and displays value to variables:
```sql
DELIMITER **

CREATE PROCEDURE Summary()
BEGIN
    DECLARE TotalM, TotalF INT DEFAULT 0;
    DECLARE AvgNetWorth DEC(6,2) DEFAULT 0.0;
    
    SELECT COUNT(*) INTO TotalM
    FROM Actors
    WHERE Gender = 'Male';
    
    SELECT COUNT(*) INTO TotalF
    FROM Actors
    WHERE Gender = 'Female';
    
    SELECT AVG(NetWorthInMillions)
    INTO AvgNetWorth
    FROM Actors;
    
    SELECT TotalM, TotalF, AvgNetWorth;
END**

DELIMITER ;
```
This stored procedure will calculate the total number of males and females in the Actors table as well as the average net worth of all actors and display the result when called. 

The variables are declared in the `BEGIN` `END` block and will be out of scope after the `END` statement is executed. 

To execute the Summary stored procedure run the following statement:
```sql
CALL Summary();
```

### Parameters
A parameter is a placeholder for a variable that is used to pass data to and from a stored procedure. 

- An **input parameter** is used to pass data value to a stored procedure A
- An **output parameter** lets the stored procedure pass a data value back to the caller. 
 
A parameter can be defined by specifying the **mode** as well as the **data type** and an **optional maximum length**.

Parameters are used to make a stored procedure flexible. 

Another reason of using parameters is avoiding direct user inputs in a query string. 
- A user input can result in a runtime error and in worst case a malicious input can potentially harm the system.

In MySQL a parameter can have three **modes**: `IN`, `OUT` and `INOUT`. 
- If the mode of a parameter is defined as `IN`, it indicates that the application calling the stored procedure has to pass an argument. The stored procedure can not alter the value of the argument, rather it only works on a copy of the `IN` parameter and the original value of the parameter is retained after the stored procedure ends. 
- A parameter defined as having `OUT` mode indicates that the stored procedure will pass an argument back to the caller. The value of an `OUT` parameter can change in the stored procedure and the new value is passed back in the end. The stored procedure, however, cannot access the initial value of the `OUT` parameter and its value is `NULL` when the procedure begins execution. 
- The third mode, `INOUT` has properties of both the `IN` and `OUT` mode. The caller may pass an argument to the stored procedure and the stored procedure can work on it and pass the altered value back to the caller.

Syntax
```sql
[IN | OUT | INOUT] parameter_name datatype [(length)]
```

Example:

A stored procedure that displays the list of actors whose net worth in millions is more than an input parameter NetWorth.
```sql
-- Query 1
DELIMITER **
CREATE PROCEDURE GetActorsByNetWorth(IN NetWorth INT )
BEGIN
	SELECT * FROM Actors
	WHERE NetWorthInMillions >= NetWorth;
END **
DELIMITER ;

-- Query 2
CALL GetActorsByNetWorth(500);
CALL GetActorsByNetWorth(750);

-- Query 3
-- the results of our stored procedure are now dependent on the input parameter. 
-- A call to this stored procedure without providing any input parameter will result in an error
CALL GetActorsByNetWorth();
```

A stored procedure that passes an argument back to the caller:
```sql
-- Query 4
-- pass the number of actors whose NetWorthInMillions value is greater than the input parameter.
DELIMITER **
CREATE PROCEDURE GetActorCountByNetWorth (
	IN  NetWorth INT,
	OUT ActorCount INT)
BEGIN
	SELECT COUNT(*) 
    INTO ActorCount
	FROM Actors
	WHERE NetWorthInMillions >= NetWorth;
END**
DELIMITER ;

-- Query 5
-- We have specified two parameters for our stored procedure, one is input and the other output. 
-- To receive the return value, we will pass a session variable. 
-- A session variable is preceded by @ sign. 
-- The value returned by the stored procedure can then be displayed using a SELECT statement as follows:
CALL GetActorCountByNetWorth(500, @ActorCount);
SELECT @ActorCount;
```

When the mode of a parameter is `INOUT`, the stored procedure can access the original value of the parameter when it is passed to it and that becomes the parameter's initial value within the procedure. We will show this with the example of a stored procedure that takes a value to be added to the NetWorthInMillions value as follows:
```sql
-- Query 6
DELIMITER **
CREATE PROCEDURE IncreaseInNetWorth(
	INOUT Increase INT,
	IN ActorId INT )
BEGIN
	DECLARE OriginalNetWorth INT;

	SELECT NetWorthInMillions Into OriginalNetWorth
    FROM Actors 
	WHERE Id = ActorId;

	SET Increase = OriginalNetWorth + Increase;	
END**
DELIMITER ;

-- Query 7
-- To run the stored procedure execute the following:
SET @IncreasedWorth = 50;
CALL IncreaseInNetWorth(@IncreasedWorth, 11);
SELECT @IncreasedWorth;
```
- This is a contrived example to show how the value of a parameter with `INOUT` mode changes. 
- The value of Increase is 50 before the stored procedure is called. 
- Within the procedure, the value is changed and the new value is displayed afterwards. 
- When the mode is `INOUT` the stored procedure can take the initial value, modify it and return the new value. 
- If we had passed Increase in `IN` mode then the stored procedure would not be allowed to modify its value.

A stored procedure can **return multiple values back to the caller**. This is in contrast to a stored function which can only return one value. We will discuss stored functions later. For now let’s consider an example where a stored procedure returns the number of male and female actors whose net worth is more than an input amount.

```sql
-- Query 8
DELIMITER **
CREATE PROCEDURE GenderCountByNetWroth(
	IN NetWorth INT,
	OUT MaleCount INT,
	OUT FemaleCount INT)
BEGIN
        SELECT COUNT(*) INTO MaleCount
        FROM Actors
        WHERE NetWorthInMillions >= NetWorth
              AND Gender = 'Male';

	SELECT COUNT(*) INTO FemaleCount
        FROM Actors
        WHERE NetWorthInMillions >= NetWorth
              AND Gender = 'Female';
END**
DELIMITER ;

-- Query 9
-- the two OUT parameters get their value based on the IN parameter NetWorth.
CALL GenderCountByNetWroth(500, @Male, @Female);
SELECT @Male, @Female;
```

### Conditional Statements
MySQL supports two conditional control statements `IF` and `CASE`. Both provide similar functionality and the choice between the two is a matter of personal preference, However, we will discuss situations in which choosing one type of statement may be better than the other.

#### IF Statement
A single `IF-THEN` block is used if some statements are to be executed based on a specific condition. 

In the case of MySQL, a condition can evaluate to `TRUE`, `FALSE`, or `NULL` (neither true nor false). 
- So unlike other programming languages if a condition is not `TRUE` it does not automatically mean that it is `FALSE`. 
- The condition to execute the code is given between the `IF` and `THEN` words. 
- If the condition holds true then statements written between the `IF-THEN` and `END IF` are executed otherwise the control moves to the next statement after the `IF` block. 
- Multiple statements can be written in a block including calls to stored procedures, `SET` statements, loops and nested IFs.

Syntax
```sql
IF Condition THEN
If_statements;
END IF;

IF Condition THEN
If_statements;
ELSE
else_statements;
END IF;

IF Condition THEN
If_statements;
ELSEIF else-if_condition
else-if_statements;
…
ELSE
else_statements;
END IF;
```

#### CASE Statement
There are two forms of CASE statement.

The **simple** `CASE` statement is when we compare the output to multiple distinct values. 
- The values after the `WHEN` keyword are sequentially compared and if there is a match then the statements in the corresponding `THEN` block are executed. 
- If no match is found then the optional `ELSE` block is executed. 
- In case the `ELSE` block does not exist and `CASE` cannot find a match, then an **error** message is issued.

Syntax
```sql
CASE case_value
WHEN case_value1 THEN statements
WHEN case_value2 THEN statements
…
[ELSE else-statements]
END CASE;
```

The other type of `CASE` statement is the searched `CASE` statement which can handle complex conditions with multiple expressions. 
- It is used to test conditions involving ranges. 
- The condition after the `WHEN` keyword is evaluated and if it evaluates to `TRUE` then the statements in the corresponding `THEN` block are executed. 
- Similar to the simple case statement, if no condition evaluates to `TRUE` then the optional `ELSE` block is executed. 
- An **error** message is issued if the `ELSE` block does not exist. 
- To ignore errors an empty `BEGIN END` block can be written in the `ELSE` block.

Syntax
```sql
CASE
WHEN case_1 condition THEN statements
WHEN case_2 condition THEN statements
…
[ELSE else-statements]
END CASE;
```

Example:

1. The `GetMaritalStatus` stored procedure has two parameters; an input parameter ActorID and an output parameter which returns the marital status of the actor. 

A variable `Status` is created in which we fetch the marital status of the actor based on the input actor id. 

Then we test if the status is married and return the string ‘Actor is married’ if the condition in the `IF` block is true. If the condition is not true then `NULL` is returned.

To test our stored procedure we will call it with two different actor ids. In the first call, `NULL` is returned because the marital status is single and `IF` block is not executed.

```sql
-- Query 1
DELIMITER **
CREATE PROCEDURE GetMaritalStatus(
    IN  ActorId INT, 
    OUT ActorStatus  VARCHAR(30))
BEGIN
    DECLARE Status VARCHAR (15);

    SELECT MaritalStatus INTO Status
    FROM Actors
    WHERE Id = ActorId;

    IF Status LIKE 'Married' THEN
        SET ActorStatus = 'Actor is married';
    END IF;
END**
DELIMITER ;

-- Query 2
CALL GetMaritalStatus(1, @status);
SELECT @status;
CALL GetMaritalStatus(5, @status);
SELECT @status;
```
2. Modify the above stored procedure to return 'Actor is not married' if the condition in the `IF` branch is not true. To include an `ELSE` branch, we will drop the procedure first and then recreate it.

```sql
-- Query 3
DROP PROCEDURE GetMaritalStatus;
DELIMITER **
CREATE PROCEDURE GetMaritalStatus(
    IN  ActorId INT, 
    OUT ActorStatus  VARCHAR(30))
BEGIN
    DECLARE Status VARCHAR (15);

    SELECT MaritalStatus INTO Status
    FROM Actors
    WHERE Id = ActorId;

    IF Status LIKE 'Married' THEN
        SET ActorStatus = 'Actor is married';
    ELSE
        SET ActorStatus = 'Actor is not married';
    END IF;
END **
DELIMITER ;

-- Query 4
CALL GetMaritalStatus(1, @status);
SELECT @status;
CALL GetMaritalStatus(5, @status);
SELECT @status;
```

3. Test for more conditions based on the three values of the MaritalStatus column. The `IF THEN ELSEIF ELSE` statement is used to test for multiple conditions.

In this example, the `IF` and `ELSEIF` blocks test the different values of the MaritalStatus column. The `ELSE` block is used to handle errors in case there is a `NULL` value in the MaritalStatus column since this block is executed if no match is found.

```sql
-- Query 5
DROP PROCEDURE GetMaritalStatus;
DELIMITER **
CREATE PROCEDURE GetMaritalStatus(
    IN  ActorId INT, 
    OUT ActorStatus  VARCHAR(30))
BEGIN
    DECLARE Status VARCHAR (15);

    SELECT MaritalStatus INTO Status
    FROM Actors
    WHERE Id = ActorId;

    IF Status LIKE 'Married' THEN
        SET ActorStatus = 'Actor is married';
    ELSEIF Status LIKE 'Single' THEN
        SET ActorStatus = 'Actor is single';
    ELSEIF Status LIKE 'Divorced' THEN
        SET ActorStatus = 'Actor is divorced';
    ELSE
        SET ActorStatus = 'Status not found';
    END IF;
END **
DELIMITER ;

--Query 6
CALL GetMaritalStatus(1, @status);
SELECT @status;
CALL GetMaritalStatus(5, @status);
SELECT @status;
CALL GetMaritalStatus(6, @status);
SELECT @status;
```

4. Another way to add if else logic to the code is by using the Simple `CASE` statement as follows. The Simple `CASE` statement is used to determine the marital status of an actor based on the value of the Status variable.
```sql
-- Query 7
DROP PROCEDURE GetMaritalStatus;
DELIMITER **
CREATE PROCEDURE GetMaritalStatus(
	IN  ActorId INT, 
	OUT ActorStatus VARCHAR(30))
BEGIN
    DECLARE Status VARCHAR (15);

    SELECT MaritalStatus INTO Status
    FROM Actors 
    WHERE Id = ActorId;

    CASE Status
        WHEN 'Married' THEN
            SET ActorStatus = 'Actor is married';
        WHEN 'Single' THEN
	        SET ActorStatus = 'Actor is single';
        WHEN 'Divorced' THEN
            SET ActorStatus = 'Actor is divorced';
        ELSE
            SET ActorStatus = 'Status not found';
    END CASE;
END**
DELIMITER ;

-- Query 8
CALL GetMaritalStatus(1, @status);
SELECT @status;
CALL GetMaritalStatus(5, @status);
SELECT @status;
CALL GetMaritalStatus(6, @status);
SELECT @status;
```

5. The Searched `CASE` is used to test for complex conditions. Let’s say we want to find the age bracket of an actor. We will create a stored procedure `GetAgeBracket()` and use the `DATEDIFF()` function to find the age of the actor as follows.

The `ELSE` clause is executed in case the age comes out to be `NULL`. We can also use an empty `BEGIN END` block in `ELSE` to handle errors arising because of NULL values.
```sql
-- Query 9
DELIMITER **
CREATE PROCEDURE GetAgeBracket(
       IN ActorId INT,
       OUT AgeRange VARCHAR(30))
BEGIN
    DECLARE age INT DEFAULT 0;

    SELECT TIMESTAMPDIFF(YEAR, DoB, CURDATE())
	INTO age
	FROM Actors
    WHERE Id = ActorId;
    
    CASE 
        WHEN age < 20 THEN 
            SET AgeRange = 'Less than 20 years';
        WHEN age >= 20 AND age < 30 THEN
            SET AgeRange = '20+';
        WHEN age >= 30 AND age < 40 THEN
            SET AgeRange = '30+';
        WHEN age >= 40 AND age < 50 THEN
            SET AgeRange = '40+';
        WHEN age >= 50 AND age < 60 THEN
            SET AgeRange = '50+';
        WHEN age >= 60 THEN
            SET AgeRange = '60+';
        ELSE
            SET AgeRange = 'Age not found';
	END CASE;	
END**
DELIMITER ;

-- Query 10
CALL GetAgeBracket(1, @status);
SELECT @status;
CALL GetAgeBracket(5, @status);
SELECT @status;
```
6. Both `IF` and `CASE` can be used interchangeably. `CASE` statement is more readable while `IF` is familiar to programmers and hence more easily understood. When using `CASE`, explicit error handling is needed for NULL values because failure to match a condition will result in an error.

