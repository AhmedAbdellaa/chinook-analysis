### Question 1.1: Which countries have the most Invoices?
```sql
SELECT "BillingCountry",count("InvoiceId") Invoices
FROM  "Invoice"
GROUP BY BillingCountry
ORDER BY 2 DESC
```
### Question 1.2: Which city has the best customers?
```sql
SELECT "BillingCity", sum("Total") amount
FROM  "Invoice"
GROUP BY BillingCity
ORDER BY 2 DESC
LIMIT 1
```
### Question 1.3: Who is the best customer?
```sql
with cusid as (
        SELECT "CustomerId", sum("Total") amount
        FROM  "Invoice"
        GROUP BY "CustomerId"
        ORDER BY 2 DESC
        LIMIT 1)
select FirstName ,cusid.amount
from Customer
JOIN cusid on cusid.CustomerId = Customer.CustomerId
where Customer.CustomerId =  cusid.CustomerId
```
----------------------------------------------------

### Question 2.1
#### Use your query to return the email, first name, last name, and Genre of all Rock Music listeners (Rock & Roll would be considered a different category for this exercise). Return your list ordered alphabetically by email address starting with A.
```sql
SELECT "Email" , "FirstName"  , "LastName" ,Genre."Name"
FROM "Customer"
JOIN invoice on Customer.CustomerId = Invoice.CustomerId
join InvoiceLine on invoice.InvoiceId = InvoiceLine.InvoiceId
join Track on InvoiceLine.TrackId = Track.TrackId
JOIN Genre on Genre.GenreId = Track.GenreId
where Genre.Name = "Rock"
ORDER by Email
```
### Question 2.2: Who is writing the rock music?
```sql
    SELECT Artist.ArtistId,Artist.Name , Track.AlbumId ,count(Artist.ArtistId) count
FROM Track
JOIN Genre on Genre.GenreId = Track.GenreId
JOIN Album on Album.AlbumId = Track.AlbumId
JOIN Artist on Artist.ArtistId = Album.ArtistId
where Genre.Name = "Rock"
GROUP by Artist.ArtistId
ORDER by count DESC
limit 10
```
### Question 2.3
#### First, find which artist has earned the most according to the InvoiceLines?
sales amount for each artist
```sql
SELECT Artist.ArtistId,Artist.Name ,sum(InvoiceLine.UnitPrice * InvoiceLine.Quantity) total_sales
FROM Track
JOIN Album on Album.AlbumId = Track.AlbumId
JOIN Artist on Artist.ArtistId = Album.ArtistId
JOIN InvoiceLine on InvoiceLine.TrackId = Track.TrackId
-- join Invoice on Invoice.InvoiceId = InvoiceLine.InvoiceId
GROUP by Artist.ArtistId 
order by total_sales DESC
LIMIT 1
```
#### Now use this artist to find which customer spent the most on this artist.
sales amount by each customer for top paid artist
```sql
with top_sales_arists as (
        SELECT Artist.ArtistId,Artist.Name ,sum(InvoiceLine.UnitPrice * InvoiceLine.Quantity) total_sales
        FROM Track
        JOIN Album on Album.AlbumId = Track.AlbumId
        JOIN Artist on Artist.ArtistId = Album.ArtistId
        JOIN InvoiceLine on InvoiceLine.TrackId = Track.TrackId
        GROUP by Artist.ArtistId 
        order by total_sales DESC
        LIMIT 1
    ), invoices as (
        SELECT InvoiceLineId, top_sales_arists.Name 
        From InvoiceLine 
        JOIN Track on InvoiceLine.TrackId = Track.TrackId
        JOIN Album on Album.AlbumId = Track.AlbumId
        JOIN top_sales_arists on top_sales_arists.ArtistId = Album.ArtistId
        )
select Customer.CustomerId,Customer.FirstName,customer.LastName ,sum (InvoiceLine.UnitPrice * InvoiceLine.Quantity) AmountSpent, Invoices.Name
FROM Customer 
JOIN Invoice on Invoice.CustomerId = Customer.CustomerId
join InvoiceLine on InvoiceLine.InvoiceId = Invoice.InvoiceId
JOIN invoices on invoices.InvoiceLineId = InvoiceLine.InvoiceLineId
GROUP by Customer.CustomerId
ORDER by AmountSpent DESC
```
-----------------------------------------------------

### Question 3.1
#### We want to find out the most popular music Genre for each country. We determine the most popular genre as the genre with the highest amount of purchases. Write a query that returns each country along with the top Genre. For countries where the maximum number of purchases is shared, return all Genres.
most popular music genre for each country
```sql
with count_genre_purchase as (
        SELECT BillingCountry , Genre.Name , count (InvoiceLine.InvoiceLineId) Purchases
        FROM Invoice
        join InvoiceLine on InvoiceLine.InvoiceId = Invoice.InvoiceId
        JOIN Track on InvoiceLine.TrackId = Track.TrackId
        JOIN Genre on Genre.GenreId = Track.GenreId
        GROUP by BillingCountry, Genre.Name

    ),max_p as(
        SELECT * , max( Purchases) from count_genre_purchase as c_g_p
        GROUP by BillingCountry
)

SELECT cgp.BillingCountry , cgp.Name ,cgp.Purchases from count_genre_purchase as cgp
join max_p on cgp.BillingCountry =max_p.BillingCountry and 
	 cgp.Purchases = max_p.Purchases
ORDER by cgp.BillingCountry ASC , cgp.Purchases DESC
```

### Question 3.2
#### Return all the track names that have a song length longer than the average song length. Though you could perform this with two queries. Imagine you wanted your query to update based on when new data is put in the database. Therefore, you do not want to hard code the average into your query. You only need the Track table to complete this query.
song length where length greater than avg 
```sql
SELECT Track.Name , Track.Milliseconds
from Track 
WHERE Track.Milliseconds > (SELECT avg(Milliseconds) FROM Track)
ORDER by Milliseconds DESC
```

### Question 3
#### Write a query that determines the customer that has spent the most on music for each country. Write a query that returns the country along with the top customer and how much they spent. For countries where the top amount spent is shared, provide all customers who spent this amount.
top spent customer for each country 
```sql
with ts as (SELECT BillingCountry,Customer.CustomerId, FirstName,LastName,sum(Invoice.Total) TotalSpent
			FROM Customer 
			JOIN Invoice on Invoice.CustomerId = Customer.CustomerId
			GROUP by BillingCountry,Customer.CustomerId)

SELECT ts.* 
FROM ts
JOIN (SELECT * ,max(ts.TotalSpent)maxTs
		from ts 
		GROUP by ts.BillingCountry )as max_ts
	on ts.BillingCountry = max_ts.BillingCountry
	and ts.TotalSpent = max_ts.maxTs

```
