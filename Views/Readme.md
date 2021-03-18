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
#### 1. A view can be created from a single table.
```sql
-- Query 1
-- shows the number of digital assets owned by an actor. 
CREATE VIEW DigitalAssetCount AS 
SELECT ActorId, COUNT(AssetType) AS NumberOfAssets 
FROM DigitalAssets
GROUP BY ActorId;

-- Query 2
-- A view can be queried in the same manner as a table.
SELECT * FROM DigitalAssetCount;
```
- Views are stored as virtual tables and also appear in the list of tables when `SHOW TABLES` is executed.
- To find out which entities in the above image are **tables** and which are **views**, the `SHOW FULL TABLES` command is used. The `Table_Type` column in the result specifies whether the object is a view or a table

#### 2. A view can be created from multiple tables using JOIN.
```sql
-- Query 3
-- create a view of Actors who have Twitter accounts
CREATE VIEW ActorsTwitterAccounts AS
SELECT FirstName, SecondName, URL
FROM Actors
INNER JOIN DigitalAssets  
ON Actors.Id = DigitalAssets.ActorID 
WHERE AssetType = 'Twitter';

-- Query 4
-- use the [OR REPLACE] clause to make changes to a view that we just created. 
-- If the view does not exist, the OR REPLACE clause has no effect.
-- show the first and last names in one column instead of two separate columns
CREATE OR REPLACE VIEW ActorsTwitterAccounts AS
SELECT CONCAT(FirstName, ' ', SecondName) AS ActorName, URL
FROM Actors
INNER JOIN DigitalAssets  
ON Actors.Id = DigitalAssets.ActorID 
WHERE AssetType = 'Twitter';
```
#### 3. The SELECT statement that creates a view can also have a nested subquery.
```sql
-- Query 5
-- The view shows five rich actors whose net worth is more than the average of all actors.
CREATE VIEW RichActors AS
SELECT FirstName, SecondName, Gender, NetWorthInMillions  
FROM Actors
WHERE NetWorthInMillions > (
SELECT AVG(NetWorthInMillions)
FROM Actors)
ORDER BY NetWorthInMillions DESC;
```
#### 4. A view can also be created from another view.
```sql
-- Query 6
CREATE VIEW RichFemaleActors AS
SELECT * FROM RichActors
WHERE Gender = 'Female';
```
#### 5.Define columns in a view by listing them in parentheses after the view name.
In the previous examples, the `SELECT` statement specifies the columns of the view.
```sql
-- Query 7
-- create the ActorDetails view where we define a column, Age, that is based on the DoB column in the Actors table
CREATE VIEW ActorDetails (ActorName, Age, MaritalStatus, NetWorthInMillions) AS
SELECT CONCAT(FirstName,' ',SecondName) AS ActorName, 
       TIMESTAMPDIFF(YEAR, DoB, CURDATE()) AS Age, 
       MaritalStatus, NetWorthInMillions 
FROM Actors;

-- Query 8
-- lists the actors from the view created above according to age
SELECT ActorName, Age, NetWorthInMillions
    FROM ActorDetails
    ORDER BY Age DESC;
```
