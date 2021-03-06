### Nested Scalar Queries
- Nested: a query within another query. 
- Nested Scalar Queries: nested queries that returned a single value

With the ability to nest a query, we can combine queries to get the desired result in a single uber query rather than executing each constituent query individually. 

Nested queries are generally **slower** but **more readable and expressive** than equivalent join queries. 

Also, in some situations nested queries are the only way to retrieve desired information from a database.

Example
```sql
-- Query 1
-- find the Instagram page for Brad Pitt
SELECT URL AS "Brad's Insta Page" 
FROM Actors 
INNER JOIN DigitalAssets 
WHERE AssetType = "Instagram" AND FirstName  ="Brad";

-- Query 2
-- The above join query can be rewritten as a nested query as follows:
SELECT URL 
FROM DigitalAssets 
WHERE AssetType = "Instagram" AND 
ActorId = (SELECT Id 
           FROM Actors 
           WHERE FirstName = "Brad");
```
- Nesting allows us to get the result in one shot. 
- In the example above, if we change the sub-query to return multiple values or return the entire row, the overall query will fail.
  - For example: 
  ```sql 
  SELECT URL FROM ActorId = (SELECT * from Actor WHERE FirstName="Brad");
  ```

```sql
-- Query 3
-- return the actor who has most recently updated any of his or her online social accounts
SELECT FirstName 
FROM Actors 
INNER JOIN DigitalAssets 
ON ActorId = Id 
WHERE LastUpdatedOn = (SELECT MAX(LastUpdatedOn) 
                       FROM DigitalAssets);
```
- Note that if there is more than one actor with the most recent activity, the above query will return more than one row. 
- This query demonstrates a scenario where we don't have an alternative other than a nested query to get our desired information in a single query execution.


### Nested Column Queries
Nested queries that return a column of values.

Example
```sql
-- Query 1
-- returns all the enum values for the column AssetType except the value "Website"
SELECT * FROM Actors 
INNER JOIN DigitalAssets ON ActorId=Id 
WHERE AssetType = ANY (SELECT DISTINCT AssetType 
                       FROM DigitalAssets
                       WHERE AssetType != 'Website');
```
- The `ANY` operator allows us to match the column AssetType with **any one of the values** returned for the column AssetType.
- The `ANY` clause has an alias `IN` that can be used interchangeably.

```sql
-- Query 2
-- same as Query 1
SELECT * FROM Actors 
INNER JOIN DigitalAssets ON ActorId=Id 
WHERE AssetType != 'Website';

-- Query 3
-- find the names of all the actors that have a Facebook presence
SELECT FirstName, SecondName
FROM Actors
WHERE Id = ANY (SELECT ActorId
                FROM DigitalAssets
                WHERE AssetType = 'Facebook');

-- Query 4
-- repalce ANY with IN, same
SELECT FirstName, SecondName
FROM Actors
WHERE Id IN (SELECT ActorId
             FROM DigitalAssets
             WHERE AssetType = 'Facebook');
```
The operator `ANY`, and its alias `IN`, match at least one value from a group of values. 

On the contrary, there's another operator, `ALL`, that must match all the values in the group. 
```sql
-- Query 5
-- find out the list of actors that have a net worth greater than all the actors whose first name starts with the letter 'J'
SELECT FirstName, SecondName 
FROM Actors 
WHERE NetworthInMillions > ALL (SELECT NetworthInMillions 
                                FROM Actors
                                WHERE FirstName LIKE "j%");
```

### Nested Row Queries
Nested queries that return rows, allowing the outer query to match on multiple different column values. 

Furthermore, so far, we have used nested queries only with the `WHERE` clause, but now we'll also use them with the `FROM` clause.

Example
```sql
-- Query 1
-- find the list of all the actors whose latest update to any of their online accounts was on the day of their birthday.
SELECT FirstName
FROM Actors
INNER JOIN DigitalAssets
ON Id=ActorId 
AND MONTH(DoB) = MONTH(LastUpdatedOn) 
AND DAY(DoB) = DAY(LastUpdatedOn);

-- Query 2
-- Instead of the inner join we can also use a nested query.
SELECT FirstName
FROM Actors 
WHERE (Id, MONTH(DoB), DAY(DoB))
IN ( SELECT ActorId, MONTH(LastUpdatedOn), DAY(LastUpdatedOn)
     FROM DigitalAssets);

--Query 3
-- list all the online accounts belonging to that actor along with their latest update times
SELECT ActorId, AssetType, LastUpdatedOn FROM DigitalAssets;

-- Query 4
-- to know which rows from the DigitalAssets table belong to Kardashian
SELECT FirstName, AssetType, LastUpdatedOn 
FROM Actors  
INNER JOIN (SELECT ActorId, AssetType, LastUpdatedOn 
            FROM DigitalAssets) AS tbl 
ON ActorId = Id;

-- Query 5
-- add a WHERE clause with the condition FirstName="Kim"
SELECT FirstName, AssetType, LastUpdatedOn 
FROM Actors 
INNER JOIN (SELECT ActorId, AssetType, LastUpdatedOn 
            FROM DigitalAssets) AS tbl 
ON ActorId = Id
WHERE FirstName = "Kim";

-- Query 6
- order the rows by LastUpdatedOn to get the latest updated online account for Kardashian.
SELECT FirstName, AssetType, LastUpdatedOn 
FROM Actors 
INNER JOIN (SELECT ActorId, AssetType, LastUpdatedOn 
            FROM DigitalAssets) AS tbl 
ON ActorId = Id
WHERE FirstName = "Kim"
ORDER BY LastUpdatedOn DESC LIMIT 1;
```
- When the result of an inner query is used as a derived table, MySQL requires us to provide an alias for the table.
- Note, in the `FROM` clause we could have just as well used the DigitalAssets table instead of plugging in a nested query.

### EXISTS Operator
Test if a subquery returns any rows or none at all.

Example
```sql
-- Query 1
-- check if our table DigitalAssets has any account owned by the actor George Clooney. 
-- If yes, we print the list of all the actors from our Actors table.
-- otherwise, return an empty set
SELECT *
FROM Actors
WHERE EXISTS ( SELECT * 
               FROM DigitalAssets
               WHERE BINARY URL LIKE "%clooney%"); 
```
Note we have used the `BINARY` operator to make the comparison **case-insensitive** which is not required if the collation is already set to be case-insensitive for the database.

```sql

-- Query 2
-- We add the NOT operator to the EXISTS clause and see the entire Actors table print out.
SELECT *
FROM Actors
WHERE NOT EXISTS ( SELECT * 
               FROM DigitalAssets
               WHERE BINARY URL LIKE "%clooney%"); 
```

### Correlated Queries
- A type of nested queries. 
- The distinguishing feature about them is that the inner query **references a table or a column** from the outer query.

Example
```sql
-- Query 1
-- want to know which actors have their first name as part of their Twitter handle
SELECT FirstName
FROM Actors 
INNER JOIN DigitalAssets
ON Id = ActorId
WHERE URL LIKE CONCAT("%",FirstName,"%") 
AND AssetType="Twitter";

-- Query 2
-- An alternative is to write a correlated query to glean the same information from the database.
SELECT FirstName
FROM Actors
WHERE EXISTS (SELECT URL 
              FROM DigitalAssets
              WHERE URL LIKE CONCAT("%",FirstName,"%") 
              AND AssetType="Twitter");
```
The inner query references the column `FirstName` in its `WHERE` clause even though `FirstName` is part of the Actors table which is referenced only in the outer query. 

It is legal to access a table or any of its columns referenced in the outer query inside a sub-query. The value of `FirstName` for each row in the Actors table is provided as a scalar value to the inner query.

Pay attention to how we have used the `EXISTS` operator. We want to check for a condition that is true about each actor. We aren't interested in what the inner query returns for each actor, rather only if it returns anything or not. If no result is returned for an actor, then the outer query for that actor evaluates to false and the name isn't printed on the console.



