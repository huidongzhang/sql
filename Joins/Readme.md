Joins allow us to combine rows from multiple tables using columns common between them. 

In fact, the relations defined amongst tables is what makes relational databases, relational.

### Self Inner Join
```sql
SELECT *
FROM table1
INNER JOIN table1
ON <join condition>;
```
Example
```sql
-- Query 1
-- cartesian product of two tables (11*11=121 rows)
SELECT * FROM Actors a INNER JOIN Actors b;

-- Query 2
-- 11 rows now because each row in the first table matches exactly one row in the second table.
SELECT * FROM Actors a INNER JOIN Actors b USING(FirstName);

-- Query 3
-- 13 rows, because the two rows with the same value for the NetWorthInMillions column 
-- match twice for a total of four rows.
SELECT * FROM Actors a INNER JOIN Actors b USING(NetWorthInMillions);
```
- `USING` clause defines one or more columns that are in both tables or results and used to join or match rows. 
- If any rows from the two tables don't match, they aren't included in the output. 
- This obviously, will not happen in the case of a self-join.

### Inner Join
Syntax
```sql
SELECT *
FROM table1
INNER JOIN table2
ON <join condition>;
```
A New Table called ** DigitalAssets**
|Column Name	|Column Type|
|:---:|:---|
|URL |VARCHAR(200)|
|AssetType	|Enum('Facebook','Twitter', 'Instagram','Pinterest','Website')|
|LastUpdatedOn	|TIMESTAMP|
|ActorId |INT|

Example
```sql
-- Query 1
-- listing the Facebook pages for each celebrity
-- Each table in isolation can't answer this query as the 
-- Actors table doesn't hold the digital assets information for each actor and the 
-- DigitalAssets table doesn't hold the names for each actor.
SELECT FirstName, SecondName, AssetType, URL
FROM Actors 
INNER JOIN DigitalAssets  
ON Actors.Id = DigitalAssets.ActorID;

-- Query 2
-- If the two tables had the same column name for the actor's ID 
-- then we could have used the alternative syntax with USING clause
SELECT FirstName, SecondName, AssetType, URL 
FROM Actors 
INNER JOIN DigitalAssets 
USING(Id);
```
- Note that the columns listed in the `SELECT` clause are unique across the two tables. 
- However, if the two tables had columns with the same names then we would need to disambiguate the two by fully qualifying the column with the table name.
- Think of it asan intersection of the two tables based on the IDs of the celebrities.

```sql
-- Query 3
SELECT FirstName, SecondName, AssetType, URL 
FROM Actors, DigitalAssets 
WHERE ActorId=Id;
```
- There's no difference in using the WHERE clause or the INNER JOIN clause in query performance
```sql
-- Query 4
-- create a cartesian product between the two tables as we did in the self join section
SELECT FirstName, SecondName, AssetType, URL 
FROM Actors, DigitalAssets;

-- Query 5
-- or 
SELECT FirstName, SecondName, AssetType, URL 
FROM Actors 
INNER JOIN DigitalAssets;
```
Both the following queries result in empty sets.
```sql
-- Query 6
-- Makes no sense to join tables on FirstName and URL columns as they aren't related. 
SELECT * 
FROM Actors 
INNER JOIN DigitalAssets ON URL = FirstName;

-- Query 7
-- Again no sense in combining net worth and actor id. Additionally, one is an int and the other a decimal but still comparable.
SELECT * 
FROM Actors 
INNER JOIN DigitalAssets 
ON NetWorthInMillions = ActorId;
```

### Union
Combine the results from several queries together. 

The clause doesn't join the table but merely clubs the two results together.

Syntax
```sql
<Query1>
UNION
<Query2>
```

Example
```sql
-- Query 1
-- prints all the first names from the Actors table and all the URLs from the DigitalAssets table
-- in the output, the culumn name URL is ignored
SELECT FirstName FROM Actors 
UNION 
SELECT URL FROM DigitalAssets;

-- Query 2
-- print the top two richest actors and the least two richest
-- The alias from the second query is ignored in the output
-- Wrap the two queries in parentheses which is a requirement when using the order by or limit clause in subqueries of a union query
(SELECT CONCAT(FirstName, ' ', SecondName) AS "Actor Name" 
FROM Actors 
ORDER BY NetworthInMillions DESC 
LIMIT 2)
UNION
(SELECT CONCAT(FirstName, ' ', SecondName) AS "ThisAliasIsIgnored" 
FROM Actors 
ORDER BY NetworthInMillions ASC 
LIMIT 2);
```
When using the `UNION` clause, the two result sets being combined should have the **same number and order of columns**. The columns from the result sets should be of the **same type or types that are compatible**. 

For instance, the following query will error out:
```sql
-- Query 3
SELECT FirstName, Id FROM Actors 
UNION 
SELECT FirstName FROM Actors;
```
To make the above query work, we can insert a **fake column or null** as follows:
```sql
-- Query 4
SELECT FirstName, Id FROM Actors 
UNION 
SELECT FirstName, null FROM Actors;
```
Note that the union clause **doesn't output duplicate values** and works similarly to the distinct clause. 

If we want duplicate values to be included in the query result, we need to use the `UNION ALL` clause as follows:
```sql
-- Query 5
-- this only outputs 5 distinct values: single, married, divorced, male, female
SELECT MaritalStatus FROM Actors 
UNION 
SELECT Gender FROM Actors;

-- Query 6
--  this outputs all the rows in the two columns
SELECT MaritalStatus FROM Actors 
UNION ALL
SELECT Gender FROM Actors;
```
It may ignore the `ORDER BY` clause when used without the `LIMIT` clause in a subquery.
```sql
-- Query 7
-- this does not order the networth, even though we asked ASC
(SELECT CONCAT(FirstName, ' ', SecondName) AS "Actor Name"  
FROM Actors  
ORDER BY NetworthInMillions DESC  LIMIT 2)  
UNION  
(SELECT NetworthInMillions 
FROM Actors 
ORDER BY NetworthInMillions ASC);

-- Query 8
-- this orders the networth, as we set a LIMIT
(SELECT CONCAT(FirstName, ' ', SecondName) AS "Actor Name"  
FROM Actors  
ORDER BY NetworthInMillions DESC  LIMIT 2)  
UNION  
(SELECT NetworthInMillions 
FROM Actors 
ORDER BY NetworthInMillions ASC LIMIT 3);
```
- Note, the types of the two columns in the result set aren't the same.
- The query works because MySQL converts the `int` to `varchar`.

### Left and Right Joins
Syntax
```sql
-- Left Join
SELECT *
FROM table1
LEFT [OUTER] JOIN table2
ON <join condition>

-- Right Join
SELECT *
FROM table1
RIGHT [OUTER] JOIN table2
ON <join condition>
```
The `LEFT JOIN` includes those rows from the table on its left that don't match with rows in the table to its right.

Example
```sql
-- Query 1
-- the inner join query only outputs celebrities who have a digital presence. 
-- If we use the LEFT JOIN instead, we'll get a list of all the actors with or without digital presence.
-- Note that the output now includes those actors who don't have a digital presence. (NULL)
SELECT FirstName, SecondName, AssetType, URL
FROM Actors 
LEFT JOIN DigitalAssets  
ON Actors.Id = DigitalAssets.ActorID;

-- Query 2
-- If we flip the order of the two tables in the query we get a different result
-- Note that actors without digital presence are left out.
SELECT FirstName, SecondName, AssetType, URL
FROM DigitalAssets 
LEFT JOIN Actors
ON Actors.Id = DigitalAssets.ActorID;

-- Query 3
-- this is the same as Query 2
SELECT FirstName, SecondName, AssetType, URL
FROM Actors 
RIGHT JOIN DigitalAssets  
ON Actors.Id = DigitalAssets.ActorID;
```

- Note that an alternative syntax for left and right joins is `LEFT OUTER JOIN` and `RIGHT OUTER JOIN` respectively, though there's no difference in functionality if you skip the `OUTER` keyword.

### Natural Join
Find the natural join between participating tables by **matching on columns with same name**.

Performs an inner join of the participating tables essentially without the user having to specify the matching columns. 

Syntax
```sql
SELECT *
FROM table1
NATURAL JOIN table2
```
Example
```sql
-- Query 1
-- since none of the columns in the two tables share the same name, the result is a cartesian product.
SELECT FirstName, SecondName, AssetType, URL
FROM Actors 
NATURAL JOIN DigitalAssets;

-- Query 2
-- the same as Query 1
SELECT FirstName, SecondName, AssetType, URL
FROM Actors 
INNER JOIN DigitalAssets;

-- Query 3
-- now the server matched the columns with the same name in both the tables
-- Alter the column name
ALTER TABLE DigitalAssets CHANGE ActorId Id INT;
-- rerun the previous query
SELECT FirstName, SecondName, AssetType, URL
FROM Actors 
NATURAL JOIN DigitalAssets;

-- Query 4
-- same as Query 3 after alter ActorId in Digital Assets to Id
SELECT FirstName, SecondName, AssetType, URL
FROM Actors 
INNER JOIN DigitalAssets USING (Id);

-- Query 5
SELECT FirstName, SecondName, AssetType, URL
FROM Actors 
NATURAL LEFT OUTER JOIN DigitalAssets;
```
- Ideally, we should **write expressive queries and avoid using the natural join** as it hides the columns that'll be used for the join and can subtly introduce bugs. 
- Imagine a situation where a table is altered to have an additional column that has the same name as a column in another table which is naturally joined with the first table in an existing query. Suddenly, the results from the natural join query will stop to make sense.





