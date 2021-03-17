### Nrested Scalar Queries
A query within another query. 

With the ability to nest a query, we can combine queries to get the desired result in a single uber query rather than executing each constituent query individually. 

Nested queries are generally **slower** but **more readable and expressive** than equivalent join queries. 

Also, in some situations nested queries are the only way to retrieve desired information from a database.

Example
```sql
-- Query 1
SELECT URL AS "Brad's Insta Page" 
FROM Actors 
INNER JOIN DigitalAssets 
WHERE AssetType = "Instagram" AND FirstName  ="Brad";

-- Query 2
SELECT URL 
FROM DigitalAssets 
WHERE AssetType = "Instagram" AND 
ActorId = (SELECT Id 
           FROM Actors 
           WHERE FirstName = "Brad");

-- Query 3
SELECT FirstName 
FROM Actors 
INNER JOIN DigitalAssets 
ON ActorId = Id 
WHERE LastUpdatedOn = (SELECT MAX(LastUpdatedOn) 
                       FROM DigitalAssets);
```
