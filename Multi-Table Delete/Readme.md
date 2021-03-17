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
UPDATE 
Actors INNER JOIN DigitalAssets 
ON Id = ActorId 
SET FirstName = UPPER(FirstName), SecondName = UPPER(SecondName), URL = UPPER(URL) 
WHERE AssetType = "Facebook";

-- Query 2
UPDATE  Actors, DigitalAssets
SET FirstName = UPPER(FirstName), SecondName = UPPER(SecondName), URL = UPPER(URL) 
WHERE AssetType = "Facebook"
AND ActorId = Id;
```



