# Digital-Music-Store-Investigation
## Introduction
> ### Dataset Description
> This project is about analyzing a digital music store database (The Chinook Database) that holds information about the store's media, employees and customers. By querying the database and exploring the data we can gain an understanding of the type of music purchased, where customers live and how the company can optimize their business practices. The data set contains information about the Albums, Purchase history, Customer details and Employee data. The data is about the purchases made for five years from 2009 to 2013 and the corresponding customer details.
> ### Chinook Database
> <img width="652" alt="Digital-Music-Store-Investigation" src="https://user-images.githubusercontent.com/93550505/156906143-96b709fd-9e47-4e14-9bab-64e377f2f806.png">
>
> For this project, we will be assisting the Chinook team with understanding the media in their store, their customers interests, and their invoice information.
## Data Exploratory
> ### Most popular music Genre for each Country
```sql
WITH t1 as (
  SELECT i.BillingCountry Country,g.Name Genre,
  count(i.BillingCountry) Purchases
  FROM Track t
  JOIN Genre g
  ON t.GenreId = g.GenreId
  JOIN InvoiceLine il
  ON t.TrackId = il.TrackId
  JOIN Invoice i
  ON il.InvoiceId = i.InvoiceId
  GROUP BY 1,2),
t2 as (
  SELECT t1.Country Country,max(t1.Purchases) Purchases
  FROM t1
  GROUP BY 1)
SELECT t1.Country,t1.Genre Genre,t1.Purchases
FROM t1
JOIN t2
ON t1.Country = t2.Country AND t1.Purchases = t2.Purchases
```
> From this query we find that 
> + Rock is the most popular music genre in the World.
> + USA has the largest Rock purchasing. So, It’s a good idea if we want to make a marketing campaign in USA in those years to be related to Rock genre music.

> ### The music Genre purchases at USA in 2013
```sql
SELECT g.Name Genre,count(i.BillingCountry) Purchases
FROM Track t
JOIN Genre g
ON t.GenreId = g.GenreId
JOIN InvoiceLine il
ON t.TrackId = il.TrackId
JOIN Invoice i
ON il.InvoiceId = i.InvoiceId
WHERE i.BillingCountry = 'USA'
AND i.InvoiceDate BETWEEN '2013-01-01' AND '2014-01-01'
GROUP BY 1
ORDER BY 2 DESC 
```
> From this query we find that 
> + Around 70% of the USA purchases in 2013 (last year of the purchases invoces) is Rock and Metal.

> ### Top 10 Rock bands in USA
```sql
SELECT ar.Name,count(t.Name) songs,
sum(il.UnitPrice*il.Quantity) Sales
FROM Track t
JOIN Genre g
ON t.GenreId = g.GenreId
JOIN Album a
ON t.AlbumId = a.AlbumId
JOIN Artist ar
ON a.ArtistId = ar.ArtistId
JOIN InvoiceLine il
ON t.TrackId = il.TrackId
JOIN Invoice i
ON il.InvoiceId = i.InvoiceId
WHERE i.BillingCountry = 'USA'
AND g.Name = 'Rock'
GROUP BY 1
ORDER BY 3 DESC
LIMIT 10
```
> From this query we find that
> + We can see that U2 is the most Productive and populer Rock band in USA.
> + Deep purple ,Iron Maiden and Led zeppelin are also a good chooses in addition to U2 for us if we want to make a marketing campaign for any product in USA in those years 

> ### States spend more than average sales in Rock music
```sql
WITH t1 as (
  SELECT i.BillingState State,c.CustomerId CustomerId,
    CASE WHEN i.BillingState = 'MA' THEN 'Massachusetts'
         WHEN i.BillingState = 'CA' THEN 'California'
	 WHEN i.BillingState = 'WA' THEN 'Washington'
	 WHEN i.BillingState = 'NV' THEN 'Nevada'
	 WHEN i.BillingState = 'WI' THEN 'Wisconsin'
	 WHEN i.BillingState = 'AZ' THEN 'Arizona'
	 WHEN i.BillingState = 'TX' THEN 'Texas'
	 WHEN i.BillingState = 'UT' THEN 'Utah'
	 WHEN i.BillingState = 'FL' THEN 'Florida'
	 WHEN i.BillingState = 'IL' THEN 'Illinois'
	 WHEN i.BillingState = 'NY' THEN 'New York'
	 ELSE 'Not in the data' END AS USA_State,
    sum(il.UnitPrice*il.Quantity) sales
  FROM Track t
  JOIN Genre g
  ON t.GenreId = g.GenreId
  JOIN InvoiceLine il
  ON t.TrackId = il.TrackId
  JOIN Invoice i
  ON il.InvoiceId = i.InvoiceId
  JOIN Customer c
  ON i.CustomerId = c.CustomerId
  WHERE i.BillingCountry = 'USA'
    AND g.Name = 'Rock'
  GROUP BY 1,2,3)
SELECT t1.State State,t1.USA_State USA_State,t1.sales Sales
FROM t1
WHERE t1.sales > ( 
  SELECT avg(t1.sales) sales
  FROM t1)
```
> From this query we find that
> + We can see that New York  is the most Rock music spend state in the USA.
> + Even if New York has the top spend value, it’s better to choose Utah ,Arizona or  California if we want to make Rock concert or marketing campaign as we will get benefit of there  geographical Neighborhood.
