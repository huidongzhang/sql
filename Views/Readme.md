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

### Updateable Views
Views are not only used to query data; they can also be used to update data in the underlying tables. 

It is possible to insert or update rows in the base table, and in the same vein, delete rows from the table using an updatable view. 

In order for a view to become updatable, it must abide by certain conditions:
- If the `SELECT` query that creates the view has aggregate functions (`MAX`, `MIN`, `COUNT`, `SUM`, etc.), `DISTINCT` keyword, `LEFT JOIN` or `GROUP BY`, `HAVING`, and `UNION` clauses, the resulting view will not be updatable. 
- Similarly, a subquery that refers to the same table that appears in the `FROM` clause prohibits updates to the base table.

Syntax
```sql
UPDATE view
SET col1 = value1, col2 = value2,…coln = valuen
WHERE <condition>
```

Example
```sql
-- Query 1
-- creating a simple view to show the Actor names and their net worth
CREATE VIEW ActorView AS
SELECT Id, FirstName, SecondName, NetWorthInMillions 
FROM Actors;

-- Query 2
-- update the net worth of Brad Pitt to 250 million dollars
-- The change is visible in the view as well as the underlying Actors table
UPDATE ActorView 
SET 
NetWorthInMillions = 250 
WHERE 
Id =1;
```
To find out which views in the database are updatable we can query the views table in the **information_schema** database. 

This table has a column **is_updatable** that indicates the type of view. Execute the following query to find out the updatable views in the MovieIndustry database:
```sql
-- Query 3
-- It shows the name and is_updatable columns of all the tables and views in the database
SELECT Table_name, is_updatable
FROM information_schema.views
WHERE table_schema = 'MovieIndustry';
```
- The database contains five views we created in the last lesson as well as the one created above. 
- We can see that the DigitalAssetCount view is not updatable because the aggregate `COUNT` function was used in its creation. 
- Rest of the views created so far are updatable views.
```sql
-- Query 4
-- delete the actor details corresponding to Id number 11.
-- it also delete the row in underlying table
DELETE FROM ActorView
WHERE Id = 11;
```

### With Check Option
Forbids the user to insert or update rows that are not visible through the view.
- Views usually contain a subset of rows from a base table. 
- It is possible to insert or update rows which are not visible through the view. 
- The `WITH CHECK OPTION` clause is used at the time of creation of the view and is used to maintain consistency when updating a table through an updatable view. 

Syntax
```sql
CREATE [OR REPLACE] VIEW view_name AS
select_statement
WITH CHECK OPTION;
```

Example
```sql
-- Query 1
-- a view to show details of only those actors who are single
CREATE VIEW SingleActors AS 
SELECT FirstName, SecondName, DoB, Gender, MaritalStatus, NetWorthInMillions 
FROM Actors 
WHERE MaritalStatus = 'Single';
```
Since the SingleActors view is updatable, we can insert a row in the Actors table through it. 

To show the inconsistency that can arise, we will insert a married actor in the table. 
```sql
-- Query 2
-- The insert operation is successful as the row appears in the table.
-- But the newly inserted row doesn’t appear in the SingleActors view.
INSERT INTO SingleActors (FirstName, SecondName, DoB, Gender, MaritalStatus,  NetWorthInMillions) 
VALUES ('Tom', 'Hanks', '1956-07-09', 'Male', 'Married', 350);
```
The `WITH CHECK OPTION` clause restricts users to update or insert data which is visible through the view. 
```sql
-- Query 3
--If we omit the OR REPLACE clause from the above query, 
-- we will get an error because a view with this name already exists.
CREATE OR REPLACE VIEW SingleActors AS 
SELECT FirstName, SecondName, DoB, Gender, MaritalStatus, NetWorthInMillions 
FROM Actors 
WHERE MaritalStatus = 'Single' 
WITH CHECK OPTION;

-- Query 4
-- try to insert a row in the Actors table through the SingleActors view
-- this encounters the CHECK OPTION failed error message. 
-- The WITH CHECK OPTION clause ensures that only actors who are single can be inserted into the Actors table through this view. 
INSERT INTO SingleActors (FirstName, SecondName, DoB, Gender, MaritalStatus, NetWorthInMillions) 
VALUES ('Matt', 'Damon', '1970-10-08', 'Male', 'Married', 160);

-- Query 5
-- The row is inserted in the table and is visible through the view as well.
INSERT INTO SingleActors (FirstName, SecondName, DoB, Gender, MaritalStatus, NetWorthInMillions) 
VALUES ('Charlize', 'Theron', '1975-08-07', 'Female', 'Single', 130);
```

### Local & Cascaded Check
Determine the scope of rule testing when a view is created based on another view. 
- **Local** check option restricts the rule checking to only the view being defined 
- whereas the **Cascaded** check option checks the rules of all underlying views. 
In the absence of these keywords, cascaded check is used as default.

Syntax
```sql
CREATE [OR REPLACE] VIEW view_name AS
select_statement
WITH [LOCAL | CASCADED] CHECK OPTION;
```

Example
```sql
-- CASCADED CHECK
-- Query 1
-- creating a view ActorsView1 which shows all actors who are older than 40
CREATE VIEW ActorsView1 AS
SELECT * 
FROM Actors 
WHERE TIMESTAMPDIFF(YEAR, DoB, CURDATE()) > 40;

-- Query 2
-- In the absence of the WITH CHECK OPTION clause there is no restriction on updates through ActorsView1. 
-- We can insert a 20-year-old actor to the Actors table
-- This is a disparity that can be handled using the WITH CHECK OPTION clause
INSERT INTO ActorsView1 
VALUES (DEFAULT, 'Young', 'Actress', '2000-01-01', 'Female', 'Single', 000.00);
```
#### the CASCADED CHECK OPTION clause
```sql
-- Query 3
-- create a view ActorsView2 based on ActorsView1
CREATE OR REPLACE VIEW ActorsView2 AS
SELECT * 
FROM ActorsView1
WITH CASCADED CHECK OPTION; 

-- Query 4
-- Since ActorsView2 is based on ActorsView1, it also has seven rows.
-- The view has a CASCADED check option which means that 
-- insert or update through ActorsView2 should not only be compatible with this view 
-- but also the underlying view, which is, ActorsView1.
-- Insert an actor to the Actors table using ActorsView2 whose age is 20 years
INSERT INTO ActorsView2 
VALUES (DEFAULT, 'Young', 'Actor', '2000-01-01', 'Male', 'Single', DEFAULT);
```
- We encounter an error message, `CHECK OPTION` failed 'MovieIndustry.ActorsView2'. 
  - The record is not inserted into the Actors table even though ActorsView2 did not impose any age restrictions. 
  - This is because the `CASCADED CHECK OPTION` clause also tests the rules of the underlying view, ActorsView1. 
  - Since ActorsView1 only allowed actors who are older than 40 years 
  - and ActorsView2 is based on ActorsView1 
  - so we were unable to add a 20 year old actor.

#### Demonstrate the scope of rule testing of the CASCADED check option
```sql
-- Query 5
-- create a view ActorsView3 based on ActorsView2 which should only display actors who are younger than 50
CREATE OR REPLACE VIEW ActorsView3 AS
SELECT * 
FROM ActorsView2
WHERE TIMESTAMPDIFF(YEAR, DoB, CURDATE()) < 50;  

-- Query 6
-- There is only one row in this view. 
-- We should be able to insert a 20-year-old actor through this view as it satisfies the age < 50 rule.
INSERT INTO ActorsView3 
VALUES (DEFAULT, 'Young', 'Actor', '2000-01-01', 'Male', 'Single', DEFAULT);
-- We encounter the error, CHECK OPTION failed 'MovieIndustry.ActorsView3'. 
-- This is because the CASCADED check option checks the rules of all underlying views before an update is allowed. 
-- ActorsView3 is based on ActorsView2 and ActorsView2 is based on ActorsView1 
-- which only allows actors older than 40 so we were unable to add a 20-year-old actor through ActorsView3.

-- Query 7
-- insert a 60-year-old actor with this view
INSERT INTO ActorsView3 
VALUES (DEFAULT, 'Old', 'Actor', '1960-01-01', 'Male', 'Single', DEFAULT);
```
- Insert operation is successful even though an actor whose age is more than 50 years does not comply with the age restriction of ActorsView3. 
- Since we did not mention the `WITH CHECK OPTION` clause when creating ActorsView3, the above insert operation was successful. 
- Here the rules for ActorsView2 and ActorsView1 were checked because of the `CASCADED check option` and the insert was made as the row conforms with the rules of the underlying view (age should be more than 40 years).
- Note that if ActorsView3 was created using the `WITH CHECK OPTION` clause then the above insert operation would fail.

#### Lock Check
To limit the scope of rule checking let’s redefine ActorsView2 with a LOCAL check option.
```sql
-- LOCAL CHECK 
-- Query 8
-- this view is based on ActorsView1 which shows actors who are older than 40
ALTER VIEW ActorsView2 AS
SELECT * 
FROM ActorsView1
WITH LOCAL CHECK OPTION; 

-- Query 9
-- Since ActorsView2 does not specify any age criterion, we can try inserting a 20-year-old actor
INSERT INTO ActorsView2 
VALUES (DEFAULT, 'Young', 'Actor', '2000-01-01', 'Male', 'Single', DEFAULT);
```
- The row is successfully inserted. 
- `LOCAL check` option in ActorsView2 means that insert operation should only conform to the age restriction of ActorsView2 (none in our case). 
- When we used the CASCADED check, we got an error message when the above query was executed, because the rule of the underlying ActorsView1 was also checked.
- However, the row just inserted into the Actors table is not visible through ActorsView2 because this view only shows actors who are older than 40.

Now that we have changed the scope of the check option for ActorsView2, we can see the effects on ActorsView3 as well (ActorsView3 is based on ActorsView2 and specifies a rule, age < 50). 

```sql
-- Query 10
-- insert a 20-year-old actor using ActorsView3 
INSERT INTO ActorsView3 
VALUES (DEFAULT, 'Young', 'Actor', '2000-01-01', 'Male', 'Single', DEFAULT);
```
- Because of the LOCAL check, the insert operation is successful. 
  - Previously we had encountered an error with the same query. 
  - At that time ActorsView2 had a `CASCADED` check option and MYSQL checked the age restriction of underlying ActorsView1. 
  - Now ActorsView2 has a `LOCAL` check option. Hence MYSQL inserts the record without checking the rule of ActorsView1 (age>40).


