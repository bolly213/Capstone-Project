# Capstone-Project 

## Table of Contents

- [Project Overview](#project-overview)
- [Data Source](#data-source)
- [Tools](#tools)
- [Data Preparation](#data-preparation)
- [Data Analysis](#data-analysis)
- [Explorative Data Analysis](#explorative-data-analysis)
- [Results And Findings](#results-and-findings)
- [Recommendations](#recommendations)

### Project Overview

My final capstone project at AltSchool Africa involved analysing data from the Chinook Database. It involves analysing the highest selling artist, highest sales with trend over time, customer purchase  and the purchase patterns of the high and medium value customers, popularity of each genre based over different time period and the customer lifetime value (CLV)

![Chinook Sales Dashboard](https://github.com/user-attachments/assets/0d9aa286-88c9-44b0-8d9f-59cdecc7ee46)

### Data Source

Chinook Database: Modeled after a digital media store, it contains details on Artist, and their sales' details, customers and their purchase characteristics.

### Tools

- SQL Server - Data Analysis
- PowerBI - Data Visualization & Creating Report [Download here](https://microsoft.com)
- Microsoft Powerpoint - Creating Report and Presentation [Download here](https://microsoft.com)

### Data Preparation

- Made use of https://sqliteviz.com/app/#/workspace platform to load and analyse the chinook database on sqlite.
- Explore the schema and understand the relationships between tables.
- Documented the key tables and fields.

### Explorative Data Analysis

- Top-Selling Artists: Identify artists with the highest sales and analyze their sales trends over time.
- Customer Purchase Patterns: Segment customers based on purchase behavior and identify key characteristics of high-value customers.
- Genre Popularity: Determine the most popular music genres and analyze the change in genre popularity over different time periods.
- Sales Over Time: Analyze monthly and yearly sales trends, including seasonal effects and significant sales events.
- Customer Lifetime Value (CLV): Calculate the lifetime value of customers based on their purchase history

### Data Analysis

- Highest Selling Artist

```sql
select ar.Name, strftime('%Y-%m', inv.InvoiceDate) AS Month, sum(ii.UnitPrice * ii.Quantity) as highest_sales
from artists ar
join albums al on ar.ArtistId = al.ArtistId
join tracks tr on al.AlbumId = tr.AlbumId
join invoice_items ii on tr.TrackId = ii.TrackId
join invoices inv on ii.InvoiceId = inv.InvoiceId
group by ar.Name, Month
order by highest_sales DESC
```

- Top-Selling Artists: Identify artists with the highest sales and analyze their sales trends over time

```sql
WITH ArtistSales AS (
SELECT ar.Name AS ArtistName, 
           strftime('%Y', i.InvoiceDate) AS Year, 
           ROUND(SUM(il.UnitPrice * il.Quantity), 2) AS YearlySales
    FROM artists ar
    JOIN albums al on ar.ArtistId = al.ArtistId
    JOIN tracks t on al.AlbumId = t.AlbumId
    JOIN invoice_items il on t.TrackId = il.TrackId
    JOIN invoices i on il.InvoiceId = i.InvoiceId
    GROUP BY ar.Name, Year
```

- Customer Purchase

```sql
SELECT c.CustomerId, c.FirstName,c.LastName,
COUNT(i.InvoiceId) as Total_Number_Of_Invoices,
ROUND(SUM(i.Total),2) AS Total_Amount_Spent,
ROUND(AVG(i.Total),2) AS Average_Amount_Spent
FROM customers c
JOIN invoices i ON c.CustomerId = i.CustomerId
GROUP BY c.CustomerId
```

- Customer Purchase Patterns: Segment customers based on purchase behavior and identify key characteristics of high-value customers

```sql
WITH CustomerPurchases AS (
SELECT c.CustomerId, c.FirstName,c.LastName,
COUNT(i.InvoiceId) as Total_Number_Of_Invoices,
ROUND(SUM(i.Total),2) AS Total_Amount_Spent,
ROUND(AVG(i.Total),2) AS Average_Amount_Spent
FROM customers c
JOIN invoices i ON c.CustomerId = i.CustomerId
GROUP BY c.CustomerId
),
HighValueCustomers AS (SELECT *,
CASE 
WHEN Total_Amount_Spent > 40 THEN 'High Value Customers'
WHEN Total_Amount_Spent BETWEEN 30 AND 40 THEN 'Medium Value Customers'
ELSE 'Low Value Customers'
END AS CustomerSegment
FROM CustomerPurchases
)
SELECT CustomerId, FirstName, LastName, Total_Number_Of_Invoices, Total_Amount_Spent, Average_Amount_Spent, CustomerSegment
FROM HighValueCustomers
ORDER BY Total_Amount_Spent DESC;
```

- Genre Popularity: Determine the most popular music genres and analyze the change in genre popularity over different time periods

```sql
WITH Invoice_Details AS (
SELECT 
inv.InvoiceDate,
il.TrackId,
il.UnitPrice * il.Quantity AS Revenue
FROM invoices inv
JOIN invoice_items il ON inv.InvoiceId = il.InvoiceId
),
Genre_Revenue AS (
SELECT 
strftime('%Y', id.InvoiceDate) AS Year,
gen.Name AS Genre,
ROUND(SUM(id.Revenue),2) AS Total_Revenue,
COUNT(DISTINCT id.TrackId) AS Tracks_Sold
FROM Invoice_Details id
JOIN tracks tr ON id.TrackId = tr.TrackId
JOIN genres gen ON tr.GenreId = gen.GenreId
GROUP BY Year, Genre
)
SELECT 
Year,
Genre,
Total_Revenue,
Tracks_Sold,
RANK() OVER (PARTITION BY Year ORDER BY Total_Revenue DESC) AS Rank_Of_Genre
FROM Genre_Revenue
ORDER BY Year, Total_Revenue DESC
```

- Sales Over Time: Analyze monthly and yearly sales trends, including seasonal effects and significant sales events

```sql
SELECT 
strftime('%Y', InvoiceDate) as Year,
strftime('%m', InvoiceDate) as Month,
ROUND(SUM(Total),2) as Total_Revenue
FROM invoices
GROUP BY Year, Month
ORDER BY Year, Month
```

- Customer Lifetime Value (CLV)

```sql
WITH customer_revenue AS (
    SELECT 
        c.CustomerId,
        c.FirstName,
        c.LastName,
        SUM(i.Total) AS TotalRevenue,
        NTILE(3) OVER (ORDER BY SUM(i.Total)) AS Segment
    FROM 
        customers c
    JOIN 
        invoices i ON c.CustomerId = i.CustomerId
    GROUP BY 
        c.CustomerId
)
SELECT 
    CASE 
        WHEN Segment = 1 THEN 'Low'
        WHEN Segment = 2 THEN 'Medium'
        WHEN Segment = 3 THEN 'High'
    END AS Segment,
    AVG(TotalRevenue) AS AverageCLV,
    COUNT(*) AS CustomerCount
FROM 
    customer_revenue
GROUP BY 
    Segment
ORDER BY 
    AverageCLV DESC;
```

### Results And Findings

- Top-Selling Artists: Identify artists with the highest sales and analyze their sales trends over time
  
  - Iron Maiden consistently shows strong sales performance across the years, with notable peaks in 2010 and 2012
  - U2 enters the chart in 2010 with high sales figures and maintains a  strong presence through 2013
  - Some artists like Deep Purple and Eric Clapton show more consistent 
sale figures over the years.
  - There's significant variability in sales for most artists from year to year
 
- Customer Purchase Patterns: Segment customers based on purchase behavior and identify key characteristics of high-value customers
  
  -  High Value Customers were categorized as customers with their total spent 
amount  over or equals to  $40 while those that spent below $40 were 
categorized as Medium and Low Value Customers
  - All High Value Customers shown have made 7 purchases each 
  - High Value Customers (HVC) were found in 7 countries, with USA having the 
highest number(4) 
  - HVCs were found in 7 out of the 59 cities and 6 of the 30 states were 
purchases were made

- Genre Popularity: Determine the most popular music genres and analyze the change in genre popularity over different time periods

  - Rock dominates the music market in 2009, generating the highest revenue of $178.2 from 180 tracks sold 
  - Latin and Alternative & Punk follow as the second and third most popular genres, respectively 
  - There's a very distinct gap between the Rock Genre and others, and also another significant gap between the Metal,  Latin and Alternative & Punk and others like Jazz, Blues, and R&B/Soul in terms of revenue and tracks sold

- Sales Over Time: Analyze monthly and yearly sales trends, including seasonal effects and significant sales events

  - Highest Monthly Sales were recorded in January ($201.12) and June ($201.10), while February ($187.20) and November ($186.24) recorded the lowest monthly sales

  - Year 2009 and 2013 recorded the Lowest Monthly Sales at $449.45 and 
$450.58 respectively while the Highest Monthly Sales were recorded in 
year 2010 and 2012 at $481.45 and $477.53 respectively

- Customer Lifetime Value (CLV)
  - The High segment, despite having the fewest customers (19), generates the highest total CLV ($812.78) and has the highest average CLV ($42.78)
  - The Medium and Low segments have an equal number of customers (20 each), but the Medium segment generates slightly more value
  - There's a clear distinction between the High segment and the other two segments in terms of average CLV, with the High segment customers being significantly more valuable
 
### Recommendations

Based on the analysis, 
- Implement and invest in strategies to retain high-CLV customers, such as loyalty programs, exclusive offers, and personalized communication.
- Identify opportunities to upsell or cross-sell relevant products or services to high-CLV customers
