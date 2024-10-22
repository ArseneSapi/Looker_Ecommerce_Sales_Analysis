# Goal of the project

In this project, the objective was to make a review of data sets of the compbany named Looker Ecommerce to answer the following questions.
- What is the total revenue generated over different time periods (yearly)?
- What are the sales trends over time (monthly & yearly)?
- Which product categories generate the most revenue (TOP 10)?
- Which products have the highest Sales Revenue and salesVolume(quantity)?
- What is the average order revenue per customer per year?
- What are segment customers distribution based on their country location and their purchases?
- How many customers stop buying after their first purchase?
- What proportion of sales comes from new customers compared to returning customers?
- Which departement and category have the highest sales volume and their revenue?
- What are the sources of traffic (organic, Search, Facebook, Email, Display) that drive the most sales?
- What is the conversion rate across different source of traffic?
- What is the cancellation rate across different source of traffic?
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
### What is the total revenue generated over different time periods (yearly)?

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





- What are the sales trends over time (monthly & yearly)?
- Which product categories generate the most revenue (TOP 10)?
- Which products have the highest Sales Revenue and salesVolume(quantity)?
- What is the average order revenue per customer per year?
- What are segment customers distribution based on their country location and their purchases?
- How many customers stop buying after their first purchase?
- What proportion of sales comes from new customers compared to returning customers?
- Which departement and category have the highest sales volume and their revenue?
- What are the sources of traffic (organic, Search, Facebook, Email, Display) that drive the most sales?
- What is the conversion rate across different source of traffic?
- What is the cancellation rate across different source of traffic?
- What are the gross profit and margins for different product categories?
- What are the gross profit and margins over different time periods?




 







