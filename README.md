# Case-Study-1---Danny-s-Diner---8-Week-SQL-Challenge
<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/175824201-492e580d-074f-4357-98eb-31e0c09f8545.png" />
</p>

### Introduction

Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.
  
### Problem Statement

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

### Datasets
Danny has shared with you 3 key datasets for this case study:

  - sales
  - menu
  - members

<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/175824722-7f9e9b2f-4e6b-4800-899b-b7150a0faa9e.png" />
</p>

### Questions
#### 1. What is the total amount each customer spent at the restaurant?

~~~~sql
SELECT
  sales.customer_id,
  SUM(menu.price) AS total_sales
FROM 
  dannys_diner.sales
LEFT JOIN 
  dannys_diner.menu ON sales.product_id=menu.product_id
GROUP BY
  sales.customer_id;
~~~~
 
 <p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/175825191-382b916f-906f-489e-a899-d77c8755bb38.png" />
</p>


#### 2. How many days has each customer visited the restaurant?

~~~~sql
SELECT
  customer_id,
  COUNT (DISTINCT order_date) as n_of_visits
FROM 
  dannys_diner.sales
GROUP BY
   customer_id
~~~~

#### 3. What was the first item from the menu purchased by each customer?

~~~~sql
WITH temp_t AS (
SELECT
  sales.customer_id,
  sales.order_date,
  DENSE_RANK () OVER (PARTITION BY sales.customer_id ORDER BY sales.order_date) AS ordered_dates,
  menu.product_name
FROM 
  dannys_diner.sales
INNER JOIN 
  dannys_diner.menu on sales.product_id=menu.product_id
  )

SELECT DISTINCT 
  customer_id,
  product_name
FROM 
  temp_t
WHERE 
  ordered_dates= 1
~~~~

#### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

~~~~sql
WITH popular_meal AS (
SELECT
  sales.product_id,
  count (sales.product_id) as n_of_purchases,
  menu.product_name
FROM 
  dannys_diner.sales
INNER JOIN 
  dannys_diner.menu ON sales.product_id=menu.product_id
GROUP BY 
  sales.product_id, menu.product_name
  )

SELECT
  n_of_purchases,
  product_name
FROM 
  popular_meal
ORDER BY 
  n_of_purchases DESC
LIMIT 1
~~~~

#### 5. Which item was the most popular for each customer?

~~~~sql
WITH customer_cte AS (
  SELECT
    sales.customer_id,
    menu.product_name,
    COUNT(sales.*) AS item_quantity,
    DENSE_RANK() OVER (PARTITION BY sales.customer_id ORDER BY COUNT(sales.*) DESC) AS item_rank
  FROM 
    dannys_diner.sales
  INNER JOIN
   dannys_diner.menu on sales.product_id=menu.product_id
  GROUP BY
    sales.customer_id, menu.product_name
)

SELECT
  customer_id,
  product_name,
  item_quantity
FROM 
  customer_cte
WHERE 
  item_rank = 1
~~~~

#### 6. Which item was purchased first by the customer after they became a member?

~~~~sql
 WITH member_sales_cte AS (
  SELECT
    sales.customer_id,
    sales.order_date,
    menu.product_name,
    DENSE_RANK() OVER (PARTITION BY sales.customer_id ORDER BY sales.order_date) AS order_rank
  FROM 
    dannys_diner.sales
  INNER JOIN 
    dannys_diner.menu ON sales.product_id = menu.product_id
  INNER JOIN 
    dannys_diner.members ON sales.customer_id = members.customer_id
  WHERE
    sales.order_date >= members.join_date::DATE
)
SELECT DISTINCT
  customer_id,
  order_date,
  product_name
FROM 
  member_sales_cte
WHERE 
  order_rank = 1
~~~~

#### 7. Which item was purchased just before the customer became a member?

~~~~sql
WITH member_sales_cte AS (
  SELECT
    sales.customer_id,
    sales.order_date,
    menu.product_name,
    RANK() OVER (PARTITION BY sales.customer_id ORDER BY sales.order_date DESC) AS order_rank
  FROM 
    dannys_diner.sales
  INNER JOIN 
    dannys_diner.menu ON sales.product_id = menu.product_id
  INNER JOIN 
    dannys_diner.members ON sales.customer_id = members.customer_id
  WHERE
    sales.order_date < members.join_date::DATE)
    
 SELECT 
  customer_id,
  order_date,
  product_name
FROM 
  member_sales_cte
WHERE 
  order_rank = 1
~~~~
#### 8. What is the total items and amount spent for each member before they became a member?

~~~~sql
WITH member_sales AS (
  SELECT
    sales.customer_id,
    sales.order_date,
    menu.product_id,
    menu.price,
    members.join_date
  FROM 
  dannys_diner.sales
  INNER JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
  INNER JOIN dannys_diner.members
    ON sales.customer_id = members.
  WHERE
    sales.order_date < members.join_date::DATE)
    
SELECT
  customer_id,
  SUM (price) AS total_spending,
  COUNT (product_id) AS n_number_of_products
FROM 
  member_sales
GROUP BY 
  customer_id
~~~~

#### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

~~~~sql

~~~~

#### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

~~~~sql

~~~~
