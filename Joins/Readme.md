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
SELECT FirstName, SecondName, AssetType, URL
FROM Actors 
INNER JOIN DigitalAssets  
ON Actors.Id = DigitalAssets.ActorID;

-- Query 2
SELECT FirstName, SecondName, AssetType, URL 
FROM Actors 
INNER JOIN DigitalAssets 
USING(Id);

-- Query 3
SELECT FirstName, SecondName, AssetType, URL 
FROM Actors, DigitalAssets 
WHERE ActorId=Id;

-- Query 4
SELECT FirstName, SecondName, AssetType, URL 
FROM Actors, DigitalAssets;

-- Query 5
SELECT FirstName, SecondName, AssetType, URL 
FROM Actors 
INNER JOIN DigitalAssets;

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

