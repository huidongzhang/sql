### Creating a View
Views are virtual tables that are created as a result of a SELECT query. 
- Showing only a subset of data that is meaningful to users 
- or restricting the number of rows and columns shown for security reasons. 
- A view containing columns from multiple tables can simplify queries by changing a multi-table query to a single-table query against a view. 
- Views are stored in the database along with tables.
- A view can be created from a single table, by joining two tables, or from another view.

Syntax
```sql
CREATE [OR REPLACE] VIEW view_name AS
SELECT col1, col2, …coln
FROM table
WHERE < condition>
```

Example
```sql
-- Query 1
CREATE VIEW DigitalAssetCount AS 
SELECT ActorId, COUNT(AssetType) AS NumberOfAssets 
FROM DigitalAssets
GROUP BY ActorId;

-- Query 2
SELECT * FROM DigitalAssetCount;

-- Query 3
CREATE VIEW ActorsTwitterAccounts AS
SELECT FirstName, SecondName, URL
FROM Actors
INNER JOIN DigitalAssets  
ON Actors.Id = DigitalAssets.ActorID 
WHERE AssetType = 'Twitter';

-- Query 4
CREATE OR REPLACE VIEW ActorsTwitterAccounts AS
SELECT CONCAT(FirstName, ' ', SecondName) AS ActorName, URL
FROM Actors
INNER JOIN DigitalAssets  
ON Actors.Id = DigitalAssets.ActorID 
WHERE AssetType = 'Twitter';

-- Query 5
CREATE VIEW RichActors AS
SELECT FirstName, SecondName, Gender, NetWorthInMillions  
FROM Actors
WHERE NetWorthInMillions > (
SELECT AVG(NetWorthInMillions)
FROM Actors)
ORDER BY NetWorthInMillions DESC;

-- Query 6
CREATE VIEW RichFemaleActors AS
SELECT * FROM RichActors
WHERE Gender = 'Female';

-- Query 7
CREATE VIEW ActorDetails (ActorName, Age, MaritalStatus, NetWorthInMillions) AS
SELECT CONCAT(FirstName,' ',SecondName) AS ActorName, 
       TIMESTAMPDIFF(YEAR, DoB, CURDATE()) AS Age, 
       MaritalStatus, NetWorthInMillions 
FROM Actors;

-- Query 8
SELECT ActorName, Age, NetWorthInMillions
    FROM ActorDetails
    ORDER BY Age DESC;
```
