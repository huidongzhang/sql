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
DELETE Actors, DigitalAssets   -- Mention tables to delete rows from
FROM Actors   -- The inner join creates a derived table with matching rows from both tables    
INNER JOIN DigitalAssets
ON Actors.Id = DigitalAssets.ActorId
WHERE AssetType = "Twitter";

-- Query 2
DELETE FROM Actors, DigitalAssets
USING Actors        
INNER JOIN DigitalAssets
ON Actors.Id = DigitalAssets.ActorId
WHERE AssetType = "Twitter";

-- Query 3
DELETE Actors 
FROM Actors 
WHERE EXISTS ( SELECT * 
               FROM Actors 
               INNER JOIN DigitalAssets
               ON  Id = ActorId 
               WHERE AssetType="Twitter");

-- Query 4
DELETE Actors 
FROM Actors 
WHERE EXISTS (SELECT * 
              FROM DigitalAssets 
              WHERE ActorId = Id AND AssetType = "Twitter");

-- Query 5
DELETE Actors, DigitalAssets   -- specify the tables to delete from
FROM Actors, DigitalAssets   -- reference tables
WHERE ActorId = Id   -- conditions to narrow down rows         
AND FirstName = "Johnny"
AND AssetType != "Pinterest";
```
