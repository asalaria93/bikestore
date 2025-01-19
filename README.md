# Data Cleaning and Analysis Project

This project demonstrates a comprehensive approach to data cleaning, analysis, and generating insights using SQL. Below, you will find a detailed description of the steps taken, the purpose of each query, and the key results.

## Table of Contents
1. [Introduction](#introduction)
2. [Data Cleaning](#data-cleaning)
3. [Data Analysis](#data-analysis)
4. [Key Queries](#key-queries)
    - [Join Operations](#join-operations)
    - [Customer Insights](#customer-insights)
    - [Revenue Analysis](#revenue-analysis)
    - [Top-Selling Products and Categories](#top-selling-products-and-categories)
5.  [Results](#results)

---

## Introduction
This repository contains SQL scripts that clean and analyze sales data. The data is sourced from multiple tables such as `customers`, `orders`, `products`, `order_items`, and `stocks`. The main objectives include:

- Cleaning and standardizing the dataset.
- Deriving meaningful insights, such as identifying loyal customers, premium customers, revenue contributors, and top-selling products.

## Data Cleaning
1. **Replace NULL Values in `customers`:**
    ```sql
    UPDATE customers SET Phone = '000000' WHERE phone IS NULL;
    ```
    Ensures no NULL values exist in the `Phone` column by replacing them with a placeholder value (`000000`).

2. **Check for NULL Values in Tables:**
    - `order_items`
    - `products`

    Queries identify any NULL values in critical columns such as `order_id`, `product_id`, and `list_price`.

## Data Analysis

The following analytical steps are performed to extract insights:

- **Joining Data:** Orders are joined with customer details to create comprehensive datasets.
- **View Creation:** Reusable views are created for simplified query execution and better readability.
- **Customer Segmentation:** Identification of loyal and premium customers.
- **Revenue Analysis:** Calculation of total revenue and top revenue-generating products/categories.
- **Sales Analysis:** Identification of top-selling products by quantity and revenue.

## Key Queries

### Join Operations
Joined the `orders` and `customers` tables to create a unified dataset for analysis:
```sql
SELECT o.order_id, o.customer_id, o.order_status, o.required_date, o.shipped_date, o.store_id, o.staff_id,
       c.first_name, c.last_name, c.phone, c.email, c.street, c.city, c.state, c.zip_code
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
UNION
SELECT o.order_id, o.customer_id, o.order_status, o.required_date, o.shipped_date, o.store_id, o.staff_id,
       c.first_name, c.last_name, c.phone, c.email, c.street, c.city, c.state, c.zip_code
FROM orders o
RIGHT JOIN customers c ON o.customer_id = c.customer_id;

```
### Customer Insights
#### Loyal Customers
Identified customers who made multiple purchases in the last year:
```sql
SELECT first_name, last_name, customer_id, email, COUNT(*) AS purchase_count
FROM fullpurchase_data4
WHERE shipped_date BETWEEN '2017-04-01' AND '2018-04-01'
GROUP BY first_name, last_name, customer_id, email
HAVING COUNT(*) > 1
ORDER BY purchase_count DESC;
```

#### Premium Customers
Identified customers who either purchased items worth over $5000 or bought multiple items:
```sql
WITH premium_customer1 AS (
    SELECT first_name, last_name, email, shipped_date,
           CASE WHEN final_cost > 5000 THEN 'yes' ELSE 'no' END AS premium_customer
    FROM fullpurchase_data4
    WHERE shipped_date BETWEEN '2017-04-01' AND '2018-04-01')
SELECT DISTINCT first_name, last_name, email, shipped_date
FROM premium_customer1
WHERE premium_customer = 'yes';
```

### Revenue Analysis
#### Total Revenue
Calculated total revenue for the last year:
```sql
SELECT ROUND(SUM(COALESCE(quantity, 0) * COALESCE(list_price, 0) * (1 - COALESCE(discount, 0))), 2) AS total_revenue
FROM fullpurchase_data4
WHERE shipped_date BETWEEN '2017-04-01' AND '2018-04-01';
```

#### Top-Contributing Products and Categories
Identified products and categories contributing the most to revenue:
```sql
SELECT product_id, ROUND(SUM((quantity * list_price) * (1 - discount)), 2) AS revenue
FROM fullpurchase_data4
GROUP BY product_id
ORDER BY revenue DESC
LIMIT 10;
```

### Top-Selling Products and Categories
#### By Quantity
```sql
SELECT product_id, SUM(quantity) AS total_quantity
FROM fullpurchase_data4
GROUP BY product_id
ORDER BY total_quantity DESC
LIMIT 10;
```

#### By Revenue
```sql
SELECT product_id, ROUND(SUM((quantity * list_price) * (1 - discount)), 2) AS revenue
FROM fullpurchase_data4
GROUP BY product_id
ORDER BY revenue DESC
LIMIT 10;
```


## Results
- **Loyal Customers:** Customers who purchased multiple times in the last year.
- **Premium Customers:** Customers who spent over $5000 in the last year.
- **Revenue Insights:** Total revenue generated and top contributors by product and category.
- **Sales Insights:** Top-selling products by quantity and revenue.


