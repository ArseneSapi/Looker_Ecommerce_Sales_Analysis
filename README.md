# Goal of the project

In this project, the objective was to make a review of data sets of the compbany named Looker Ecommerce to answer the following questions.
- What is the total revenue generated over different time periods (yearly)?
- What are the sales trends over time (monthly & yearly)?
- Which product categories generate the most revenue (TOP 10)?
- Which products have the highest Sales Revenue and salesVolume(quantity)?
- What is the average order revenue per year?
- What are customers distribution based on their country location and their purchases?
- How many customers stop buying after their first purchase?
- What proportion of sales comes from new customers compared to returning customers?
- Which departement and category have the highest sales volume and their revenue?
- What are the sources of traffic (organic, Search, Facebook, Email, Display) that drive the most sales?
- What is the conversion rate across different source of traffic? What is the cancellation rate across different source of traffic?
- What are the gross profit and margins for different product categories?
- What are the gross profit and margins over different time periods?

## Technology used 
### SQL
The project was entirely completed with SQL especially MySQL

## Data source and contents
Data was a set of seven tables as follow :
- distributions centers : list of all wareehouse and their location from where items are sent to customers
- events : It is a lists of all internet session during which search has been made for looker ecommerce products
- inventory_itemns : products available in stocks
- Orders : list of orders made by customers
- Orders items : list of items ordered through on orders
- Products : list of produts ant their caracteristics
- users : list of users of company services

The data sets are from Kaggle. They can be downloaded through the following link:
https://www.kaggle.com/datasets/mustafakeser4/looker-ecommerce-bigquery-dataset

## Analysis
### 1. What is the total revenue generated over different time periods (yearly)?

```sql
WITH yearlySales AS (
  SELECT 
     YEAR(created_at) AS yr,
     ROUND(SUM(sale_price), 2) AS salesRevenue
  FROM order_items
  WHERE status ='Complete'
  GROUP BY yr 
  ORDER BY yr ASC),

salesdiff AS (
  SELECT 
      yr,
      salesRevenue AS yearlySales,
      LAG(salesRevenue) OVER( ORDER BY yr) AS previousYearlySales,
      ROUND((salesRevenue - LAG(salesRevenue) OVER(ORDER BY yr)),2) AS yearlysalesDiff
  FROM yearlySales)

SELECT 
    yr,
    yearlySales,
    previousYearlySales,
    yearlysalesDiff,
    ROUND((yearlysalesDiff/previousYearlySales)*100,2) AS salesdiffPercentage
FROM salesdiff
```
### 2. What are the sales trends over time (monthly & yearly)?
```sql
with monthlyRevenue AS (
    SELECT 
        YEAR(created_at) AS 'year',
        MONTH(created_at) AS 'month',
        ROUND(SUM(sale_price), 2)  AS monthlyRevenue
    FROM order_items 
    WHERE status ='Complete' 
    GROUP BY year, month
    ORDER BY year ASC, month ASC)

SELECT 
    year,
    month,
    monthlyRevenue AS 'monthlyRevenue($)',
    LAG(monthlyRevenue) OVER( ORDER BY year, month) 	AS 'previousmonthlyRevenue($)',
    ROUND(monthlyRevenue - LAG(monthlyRevenue) OVER( ORDER BY year, month), 2) AS 'monthlyRevenueDiff($)',
    ROUND((ROUND(monthlyRevenue - LAG(monthlyRevenue) OVER( ORDER BY year, month), 2))/
    (LAG(monthlyRevenue) OVER( ORDER BY year, month))*100, 2)   AS 'diffRevenuePercent(%)'
FROM monthlyRevenue
```
### 3. Which product categories generate the most revenue (TOP 10)?
```sql
SELECT 
    p.category as category,
    ROUND(SUM(o.sale_price), 2)  AS Revenue
FROM order_items o
JOIN products p ON p.id=o.product_id
WHERE o.status ='Complete' 
GROUP BY category
ORDER BY Revenue DESC
LIMIT 10;
```
### 4. Which products (top 25) have the highest Sales Revenue and salesVolume(quantity)?
```sql
SELECT 
    o.product_id,
    p.name,
    ROUND(SUM(o.sale_price), 2)  AS salesRevenue,
    COUNT(o.product_id) AS salesVolume
FROM order_items o
JOIN products p ON p.id=o.product_id
WHERE o.status ='Complete' 
GROUP BY o.product_id, p.name
ORDER BY salesRevenue DESC
LIMIT 25
```
### 5. What is the average order revenue per year?
```sql
SELECT 
    year(created_at) AS 'year',
    COUNT(distinct user_id) AS totalUsers,
    ROUND(SUM(sale_price), 2)  AS 'salesRevenue($)',
    ROUND((ROUND(SUM(sale_price), 2)) / (COUNT(distinct user_id)), 2) AS 'AvgRevPerCustomer($)'
FROM order_items
WHERE status ='Complete' 
GROUP BY year
ORDER BY year ASC
```
### 6. What are customers distribution based on their country location and their purchases?
```sql
SELECT 
    u.country,
    COUNT(distinct o.user_id) AS totalUsers,
    ROUND(SUM(o.sale_price), 2)  AS salesRevenue
    FROM order_items o
JOIN users u ON u.id=o.user_id
WHERE o.status ='Complete' 
GROUP BY u.country
ORDER BY  salesRevenue DESC
```
### 7. What percentage of customers stop buying after their first purchase?
```sql
WITH firstPurchase  AS ( 
    SELECT 
        user_id,
        MIN(created_at)  AS firstPurchase
    FROM orders
    WHERE status ='Complete'
    GROUP BY user_id),

ordersCompleted AS (
    SELECT
        *
    FROM orders
    WHERE status ='Complete'),

userLost as (
    SELECT 
        f.user_id, 
        SUM(CASE WHEN o.created_at=f.firstPurchase THEN 1 ELSE 0 END) AS firstPurchase,
        SUM(CASE WHEN o.created_at>f.firstPurchase  THEN 1 ELSE 0 END) AS otherPurchase
    FROM firstPurchase f
    JOIN ordersCompleted o on o.user_id=f.user_id 
    GROUP BY f.user_id
    HAVING otherPurchase<1)

SELECT
    (COUNT(user_id)/(SELECT COUNT(DISTINCT user_id) FROM looker_ecommerce.orders
      WHERE status ='complete'))*100 AS 'churn(%)'
FROM userLost 
```
### 8. What proportion of sales comes from new customers compared to returning customers?
```sql
WITH firstPurchase  AS (
      SELECT 
          user_id, 
          MIN(created_at)  AS firstPurchase
      FROM order_items
      GROUP BY user_id)

SELECT
    YEAR(o.created_at) AS year,
    ROUND(SUM(o.sale_price),2)  AS 'totalSales($)',
    ROUND(SUM(CASE WHEN o.created_at=f.firstPurchase THEN o.sale_price ELSE 0 END),2) AS 'newCustomerSales($)',
    ROUND(SUM(CASE WHEN o.created_at=f.firstPurchase THEN o.sale_price ELSE 0 END)/SUM(o.sale_price)*100,2) AS 'newCustomerSalesProp(%)',
    ROUND(SUM(CASE WHEN o.created_at>f.firstPurchase THEN o.sale_price ELSE 0 END),2) AS 'ReturningCustomerSales($)',
    ROUND(SUM(CASE WHEN o.created_at>f.firstPurchase THEN o.sale_price ELSE 0 END)/SUM(o.sale_price)*100,2) AS 'ReturningCustomerSalesProp(%)'
FROM firstPurchase f 
RIGHT JOIN order_items o ON o.user_id=f.user_id
WHERE o.status ='Complete'
GROUP BY year
ORDER BY year ASC
```
### 9. Which departement and category have the highest sales volume and their revenue?
```sql
SELECT 
    p.department AS department,
    p.category AS category,
    COUNT(o.product_id) AS SalesVolume,
    ROUND(SUM(o.sale_price), 2)  AS salesRevenue
FROM order_items o
JOIN products p ON p.id=o.product_id
WHERE o.status ='Complete' 
GROUP BY department,  category
ORDER BY SalesVolume DESC
LIMIT 10
```
### 10. What are the sources of traffic (organic, Search, Facebook, Email, Display) that drive the most sales?
```sql
WITH traffic AS (
      SELECT 
          u.traffic_source AS traffic_source,
          ROUND(SUM(o.sale_price), 2)  AS salesRevenue
      FROM order_items o
      JOIN orders s ON s.order_id=o.order_id
      JOIN users u ON u.created_at=s.created_at
      WHERE s.status ='Complete'
      GROUP BY traffic_source
      ORDER BY  salesRevenue DESC)

SELECT
    traffic_source,
    salesRevenue,
    ROUND(salesRevenue/(SELECT ROUND(SUM(sale_price), 2) FROM order_items WHERE status ='Complete')*100, 2) AS totalSales_percentage
FROM traffic
```
### 11. What is the conversion rate across different source of traffic? What is the cancellation rate across different source of traffic?
```sql
WITH traffic AS (
      SELECT
          traffic_source,
          COUNT(id) AS usersSession
      FROM users
      GROUP BY  traffic_source),

conversion AS (
      SELECT 
          u.traffic_source AS traffic_source,
          COUNT(o.order_id) AS orders
      FROM orders o
      JOIN users u ON u.created_at=o.created_at
      GROUP BY traffic_source),

cancelation AS (
      SELECT 
          u.traffic_source AS traffic_source,
          COUNT(o.order_id) AS ordersCancelled
      FROM orders o
      JOIN users u ON u.created_at=o.created_at
      WHERE o.status='cancelled'
      GROUP BY traffic_source),

returnOrders AS (
      SELECT 
          u.traffic_source AS traffic_source,
          COUNT(o.order_id) AS ordersReturned
      FROM orders o
      JOIN users u ON u.created_at=o.created_at
      WHERE o.status='returned'
      GROUP BY traffic_source)

SELECT 
    c.traffic_source,
    t.usersSession,
    c.orders,
    ROUND(c.orders/t.usersSession*100, 2) AS 'conversion_rate(%)',
    n.ordersCancelled AS ordersCancelled,
    ROUND(n.ordersCancelled /c.orders*100, 2) AS 'cancellation_rate(%)',
    r.ordersReturned AS ordersReturned,
    ROUND(r.ordersReturned /c.orders*100, 2) AS 'return_rate(%)'
FROM traffic t
JOIN conversion c ON c.traffic_source=t.traffic_source
JOIN cancelation n ON n.traffic_source=t.traffic_source
JOIN returnOrders r ON r.traffic_source=t.traffic_source
```
### 12. What are the gross profit and margins for different product category?
```sql
WITH salesRevenue AS (
      SELECT 
          p.category as category,
          ROUND(SUM(o.sale_price), 2)  AS salesRevenue
      FROM order_items o
      JOIN products p ON p.id=o.product_id
      WHERE o.status ='Complete' 
      GROUP BY category),

costs AS (
      SELECT 
          p.category as category,
          ROUND(SUM(p.cost), 2)  AS cogs
      FROM products p
      JOIN order_items o  ON p.id=o.product_id
      WHERE o.status ='Complete' 
      GROUP BY category)

SELECT
    s.category,
    s.salesRevenue,
    c.cogs,
    ROUND((s.salesRevenue - c.cogs), 2) AS grossProfit,
    ROUND((s.salesRevenue - c.cogs) /s.salesRevenue*100, 2) AS grossprofitMargin,
    ROUND(c.cogs/s.salesRevenue*100, 2) AS SalesCostPrecentage
FROM salesRevenue s
JOIN costs c ON c.category=s.category
ORDER BY grossprofitMargin DESC
```
### 13. What are the gross profit and margins over different time period?
```sql
WITH salesRevenue AS (
      SELECT 
          YEAR(created_at) AS yr,
          ROUND(SUM(o.sale_price), 2)  AS salesRevenue
      FROM order_items o
      JOIN products p ON p.id=o.product_id
      WHERE o.status ='Complete' 
      GROUP BY yr),

costs AS (
      SELECT 
          YEAR(created_at) AS yr,
          ROUND(SUM(p.cost), 2)  AS cogs
      FROM products p
      JOIN order_items o  ON p.id=o.product_id
      WHERE o.status ='Complete' 
      GROUP BY yr)

SELECT
    s.yr,
    s.salesRevenue,
    c.cogs,
    ROUND((s.salesRevenue - c.cogs), 2) AS grossProfit,
    ROUND(((s.salesRevenue - c.cogs) /(s.salesRevenue))*100, 2) AS grossprofitMargin,
    ROUND(c.cogs/s.salesRevenue*100, 2) AS SalesCostPrecentage
FROM salesRevenue s
JOIN costs c ON c.yr=s.yr
ORDER BY yr ASC
```




 







