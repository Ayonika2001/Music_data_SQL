
# SQL Project: Music Store Data Analysis

This project involves analyzing the data of a music store using SQL. The analysis is divided into three question sets: **Easy**, **Moderate**, and **Advanced**, with each set addressing specific business questions. The dataset includes tables such as `employee`, `invoice`, `customer`, `track`, `album`, `artist`, `genre`, and `invoice_line`. The queries aim to extract actionable insights such as identifying top customers, best-performing artists, and popular genres.

---

## Database Setup

```sql
CREATE DATABASE music;
USE music;
```

---

## Question Set 1 - Easy

### 1. Who is the senior most employee based on job title?

```sql
SELECT first_name, last_name, title, MAX(levels) AS senior_most_employee
FROM employee
GROUP BY first_name, last_name, title
ORDER BY senior_most_employee DESC LIMIT 1;
```

### 2. Which countries have the most invoices?

```sql
SELECT billing_country, COUNT(invoice_id) AS most_invoice
FROM invoice
GROUP BY billing_country;
```

### 3. What are the top 3 values of total invoice?

```sql
SELECT total AS total_invoice
FROM invoice
ORDER BY total_invoice DESC LIMIT 3;
```

### 4. Which city has the best customers? 

Return the city with the highest sum of invoice totals.

```sql
SELECT billing_city, SUM(total) AS invoice_totals
FROM invoice
GROUP BY billing_city
ORDER BY invoice_totals DESC LIMIT 1;
```

### 5. Who is the best customer?

The customer who has spent the most money will be declared the best customer.

```sql
SELECT customer.customer_id, customer.first_name, customer.last_name, SUM(invoice.total) AS invoice_totals
FROM customer
JOIN invoice ON customer.customer_id = invoice.customer_id
GROUP BY customer.customer_id, customer.first_name, customer.last_name
ORDER BY invoice_totals DESC LIMIT 1;
```

---

## Question Set 2 - Moderate

### 1. Rock Music Listeners

Return the email, first name, last name, and genre of all rock music listeners, ordered alphabetically by email.

```sql
SELECT c.first_name, c.last_name, c.email, g.name
FROM customer AS c
JOIN invoice AS i ON c.customer_id = i.customer_id
JOIN invoice_line AS il ON i.invoice_id = il.invoice_id
JOIN track AS t ON il.track_id = t.track_id
JOIN genre AS g ON t.genre_id = g.genre_id
WHERE g.name = "Rock"
ORDER BY c.email;
```

### 2. Top 10 Rock Bands

Return the artist name and total track count for the top 10 rock bands.

```sql
SELECT a.name, g.name, COUNT(t.track_id) AS total_track
FROM artist AS a
JOIN album AS al ON a.artist_id = al.artist_id
JOIN track AS t ON al.album_id = t.album_id
JOIN genre AS g ON t.genre_id = g.genre_id
WHERE g.name = "Rock"
GROUP BY a.name, g.name
ORDER BY total_track DESC LIMIT 10;
```

### 3. Tracks Longer than Average Length

Return all track names with a song length longer than the average, sorted by the longest song first.

```sql
SELECT name, milliseconds
FROM track
WHERE milliseconds > (SELECT AVG(milliseconds) FROM track)
ORDER BY milliseconds DESC;
```

---

## Question Set 3 - Advanced

### 1. Amount Spent by Each Customer on Artists

Return customer name, artist name, and the total amount spent.

```sql
WITH best_selling_artist AS (
  SELECT a.artist_id, a.name, SUM(il.unit_price * il.quantity) AS total_spent
  FROM artist AS a
  JOIN album AS al ON a.artist_id = al.artist_id
  JOIN track AS t ON al.album_id = t.album_id
  JOIN invoice_line AS il ON t.track_id = il.track_id
  GROUP BY a.artist_id
  ORDER BY total_spent DESC LIMIT 1
)
SELECT c.customer_id, c.first_name, c.last_name, bsa.name AS artist_name, SUM(il.unit_price * il.quantity) AS amount_spent
FROM best_selling_artist AS bsa
JOIN album AS al ON bsa.artist_id = al.artist_id
JOIN track AS t ON al.album_id = t.album_id
JOIN invoice_line AS il ON t.track_id = il.track_id
JOIN invoice AS i ON il.invoice_id = i.invoice_id
JOIN customer AS c ON i.customer_id = c.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name, artist_name
ORDER BY amount_spent DESC;
```

### 2. Most Popular Genre for Each Country

Determine the most popular genre for each country based on purchases.

```sql
WITH most_popular_genre AS (
  SELECT g.name, i.billing_country, COUNT(il.quantity) AS purchase,
         ROW_NUMBER() OVER(PARTITION BY i.billing_country ORDER BY COUNT(il.quantity) DESC) AS row_no
  FROM genre AS g
  JOIN track AS t ON g.genre_id = t.genre_id
  JOIN invoice_line AS il ON t.track_id = il.track_id
  JOIN invoice AS i ON il.invoice_id = i.invoice_id
  GROUP BY g.name, i.billing_country
)
SELECT *
FROM most_popular_genre
WHERE row_no = 1;
```

### 3. Top Customer in Each Country

Return the country, top customer, and the amount spent.

```sql
WITH top_customer AS (
  SELECT c.customer_id, c.first_name, c.last_name, i.billing_country, SUM(il.unit_price * il.quantity) AS amount_spent,
         ROW_NUMBER() OVER(PARTITION BY i.billing_country ORDER BY SUM(il.unit_price * il.quantity) DESC) AS row_num
  FROM customer AS c
  JOIN invoice AS i ON c.customer_id = i.customer_id
  JOIN invoice_line AS il ON i.invoice_id = il.invoice_id
  GROUP BY c.customer_id, c.first_name, c.last_name, i.billing_country
)
SELECT *
FROM top_customer
WHERE row_num = 1;
