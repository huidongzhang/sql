### Multi-Table Delete
How to delete data from multiple tables.

If you are confronted with a situation where you want to delete data from one table and also any related data from other tables, you can employ the multi-table delete queries. 

Older Syntax
```sql
DELETE T1, T2
FROM T1, T2, T3
WHERE <condition>
```
Newer Syntax
```sql
DELETE FROM T1, T2
USING T1, T2, T3
WHERE <condition>
```

Example
```sql
-- Query 1
-- delete actors (rows) who have a Twitter account
DELETE Actors, DigitalAssets   -- Mention tables to delete rows from
FROM Actors   -- The inner join creates a derived table with matching rows from both tables    
INNER JOIN DigitalAssets
ON Actors.Id = DigitalAssets.ActorId
WHERE AssetType = "Twitter";

-- Query 2
-- The alternative and newer syntax
DELETE FROM Actors, DigitalAssets
USING Actors        
INNER JOIN DigitalAssets
ON Actors.Id = DigitalAssets.ActorId
WHERE AssetType = "Twitter";
```
- Note, all the rows that are returned by the inner join of the two tables based on the joining criteria and the where condition are deleted from both the tables

```sql
-- Query 3
-- this does not work
DELETE Actors 
FROM Actors 
WHERE EXISTS ( SELECT * 
               FROM Actors 
               INNER JOIN DigitalAssets
               ON  Id = ActorId 
               WHERE AssetType="Twitter");
```
- The above query fails because MySQL disallows rows to be deleted from a table if the same table also appears in the SELECT clause.
- i.e., we can’t delete from a table that’s read from in a nested subquery. 
- In this case, the Actors table also appears in the inner query's SELECT clause.

The same query is rewritten as a correlated query works:
```sql
-- Query 4
DELETE Actors 
FROM Actors 
WHERE EXISTS (SELECT * 
              FROM DigitalAssets 
              WHERE ActorId = Id AND AssetType = "Twitter");
```

```sql
-- Query 5
-- remove Johnny Depp from the Actors table and all his accounts except for his Pinterest from the DigitalAssets table at the same time.
DELETE Actors, DigitalAssets   -- specify the tables to delete from
FROM Actors, DigitalAssets   -- reference tables
WHERE ActorId = Id   -- conditions to narrow down rows         
AND FirstName = "Johnny"
AND AssetType != "Pinterest";
```

### Multi-Table Update
How we can update several tables at the same time.

Syntax
```sql
UPDATE T1, T2
SET col1 = newVal1, col2 = newVal2
WHERE <condition1>
```

Example
```sql
-- Query 1
-- converts the FirstName and SecondName strings stored in the Actors table to upper case for those actors who are on Facebook, 
-- and at the same time we also want to convert the associated Facebook URL to uppercase.
UPDATE 
Actors INNER JOIN DigitalAssets 
ON Id = ActorId 
SET FirstName = UPPER(FirstName), SecondName = UPPER(SecondName), URL = UPPER(URL) 
WHERE AssetType = "Facebook";

-- Query 2
-- an alternative 
UPDATE  Actors, DigitalAssets
SET FirstName = UPPER(FirstName), SecondName = UPPER(SecondName), URL = UPPER(URL) 
WHERE AssetType = "Facebook"
AND ActorId = Id;
```
- Similarly to multi delete, we can't update a table that is also being read from in a subquery.

- `ORDER BY` and `LIMIT` clauses can't be used with multi table deletes.

### SELECT and INSERT
Syntax to Insert in an Existing Table
```sql
INSERT INTO table1 (col1, col2)
SELECT col3, col4
FROM table2;
```
Syntax to Insert in a New Table
```sql
CREATE TABLE newTable (col1 <datatype>, <col2>)
SELECT col3, col4
FROM table2;
```
Example
```sql
-- populate data into a table from another table using INSERT and SELECT in a single query.
-- Query 1
-- create a table of all the second names of actors
CREATE TABLE Names (name VARCHAR(20),
                    PRIMARY KEY(name));

-- Query 2
INSERT INTO Names(name) 
SELECT SecondName FROM Actors;
```
- Note that in creating the table Names we have set the only column as the primary key.
- Trying to add a duplicate rows result in an error. 
- MySQL provides a way to bypass this error and continue execution using the `IGNORE` clause. 
- It doesn't mean a duplicate row is added to the table, rather it means that MySQL issues a **warning** instead of issuing an error and aborting.

```sql
-- Query 3
-- This query finishes successfully but informs the user about the duplicate row. 
-- Both the duplicate and warning counts read one.
INSERT IGNORE INTO Names(name) 
SELECT SecondName 
FROM Actors WHERE Id = 1;
```
We can do both tasks in one shot.
```sql
-- Query 4
CREATE TABLE MyTempTable SELECT * FROM Actors;
```
- We get a copy of the data with the above query but table we create doesn't inherit the primary key constraints. 
- In fact, creating and copying data as above will not create foreign or primary key constraints on the copy table.

All the modifiers that can be used in a stand-alone create table statement can also be used in a combined create and populate table statement.
```sql
-- Query 5
CREATE TABLE NamesWithDoBs ( 
Id INT AUTO_INCREMENT,
Name VARCHAR(20) NOT NULL DEFAULT "unknown",  
DoB DATE,  
PRIMARY KEY(Id), KEY(Name), KEY(DoB))  SELECT FirstName, DoB FROM Actors;
```

We can also create a copy of an existing table without the data using the `LIKE` operator. 
```sql
-- Query 6
CREATE TABLE CopyOfActors LIKE Actors;
```
- The copy table doesn't contain any data, but its structure is exactly the same as the source table. 
- The primary keys and any indexes defined on the source table are also defined on the copy table.

### REPLACE 
`REPLACE` is much like the `INSERT` statement with one key difference: we can't insert a row if a table already contains a row with the same primary key. 

However, `REPLACE` allows us the convenience of adding a row with the same primary key as an existing row in the table. 

Under the hood, `REPLACE` deletes the row and then adds the new row thereby maintaining the primary key constraint at all times. Sure, we can also use the `UPDATE` clause to achieve the same effect. 

However, `REPLACE` can be useful in automated scripts where **it is not known ahead of time if a particular table already contains a particular primary key**. If it doesn't, the replacement behaves like an insertion, otherwise, it deletes and writes in the new row with the same primary key.

Syntax
```sql
REPLACE INTO table (col1, col2, … coln)
VALUES (val1, val2, … valn)
WHERE <condition>
```

Example
```sql
-- Query 1
-- the output of the replace query says 2 rows affected, which implies one row was deleted and a second was inserted.
REPLACE INTO
Actors (Id, FirstName, SecondName,DoB, Gender, MaritalStatus, NetworthInMillions)
VALUES (3, "George", "Clooney", "1961-05-06","Male", "Married", 500.00);

-- Query 2
-- repeat the previous query but only provide the value for the primary key column and observe the outcome.
-- the rest of the columns not specified in the query end up taking the default value which is NULL.
REPLACE INTO 
Actors (Id)
VALUES (3);
```
- If a table doesn't have a **primary key** defined, `REPLACE` behaves exactly like an `INSERT`. Without a primary key `REPLACE` can't uniquely identify a row to replace.

- Remember that when inserting the duplicate row using the `INSERT IGNORE` clause, the duplicate row is ignored and not added to the table 
- Whereas when using REPLACE the existing row is deleted and the duplicate row is added to the table.

```sql
-- Query 3
-- this returns an error
SET id = (SELECT Id 
          FROM Actors 
          WHERE FirstName="Brad");
```
- Similar to multi-delete and update, we can't replace into a table that is also being read from a subquery.
REPLACE INTO  Actors

