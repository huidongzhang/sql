### Nested Scalar Queries
A query within another query. 

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
