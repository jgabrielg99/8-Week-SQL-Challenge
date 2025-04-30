# Case Study #1: Danny's Diner
![SQL 1](https://github.com/user-attachments/assets/2e5427a4-aec5-4935-a48c-c3abc83d5145)

## Table of Contents
  - [Business Task](#business-task)
  - [Questions and Solutions](#questions-and-solutions)

## Business Task

Danny wants to use data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they've spent and also which menu items are their favorite.

We have 3 key datasets to work with
  * sales
  * menu
  * members

## Questions and Solutions

** Question 1: What is the total amount each customer spent at the restaurant?
```sql
SELECT 
	s.customer_id,
    SUM(m.price) as total_amount
FROM sales s
INNER JOIN menu m
	ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id ASC;
```

#### Steps:
  * Use **INNER JOIN** to combine the 'sales' and 'menu' tables to access 'sales.customer_id' and 'menu.price'
  * Use **SUM** to calculate the total amount spent by each customer
  * Group the aggregated results by 'sales.customer_id'

#### Solution
| customer_id | total_amount |
| ----------- | ------------ |
| A           | 76           |
| B           | 74           |
| C           | 36           |

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)

---

** Question 2. How many days has each customer visited the restaurant?
```sql
SELECT 
	customer_id,
	COUNT(DISTINCT order_date)
FROM dannys_diner.sales
GROUP BY customer_id;
```

#### Steps:
  * Use **COUNT(DISTINCT)** to calculate the number of unique dates (just in case a customer placed more than one order on the same date) customers placed an order

#### Solution

| customer_id | count |
| ----------- | ----- |
| A           | 4     |
| B           | 6     |
| C           | 2     |

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)

---

** Question 3. What was the first item from the menu purchased by each customer?
```sql
SELECT 
  	s.customer_id,
    s.order_date,
    m.product_name
FROM sales s
JOIN menu m
	ON s.product_id = m.product_id
WHERE (s.customer_id, s.order_date) IN (
  	SELECT customer_id, MIN(order_date)
  	FROM sales
  	GROUP BY customer_id
  );
```


#### Steps:
  * Using a subquery, create a temporary table that finds the first (**MIN**) 'order_date' for each customer
  * Use **JOIN** to merge 'sales' and 'menu' to match 'product_name' with 'customer_id'

#### Solution

| customer_id | order_date | product_name |
| ----------- | ---------- | ------------ |
| A           | 2021-01-01 | sushi        |
| B           | 2021-01-01 | curry        |
| A           | 2021-01-01 | curry        |
| C           | 2021-01-01 | ramen        |
| C           | 2021-01-01 | ramen        |

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)

---

** Question 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
SELECT 
	m.product_name,
    COUNT(s.product_id) AS most_ordered
FROM sales s
JOIN menu m
	ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY most_ordered DESC;

SELECT 
	s.customer_id,
    COUNT(m.product_name)
FROM sales s
JOIN menu m
	ON s.product_id = m.product_id
WHERE m.product_name = 'ramen'
GROUP BY s.customer_id
ORDER BY s.customer_id ASC;
```
    
#### Steps:
#### Solution

| product_name | most_ordered |
| ------------ | ------------ |
| ramen        | 8            |
| curry        | 4            |
| sushi        | 3            |

---

| customer_id | count |
| ----------- | ----- |
| A           | 3     |
| B           | 2     |
| C           | 3     |

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)
---

** Question 5. Which item was the most popular for each customer?
```sql
with most_popular AS (
  SELECT 
  	s.customer_id,
  	m.product_name, 
  	COUNT(m.product_id) AS order_count,
  	DENSE_RANK() OVER(
      PARTITION BY s.customer_id
      ORDER BY COUNT(s.customer_id) DESC) AS rank
  FROM sales s
  JOIN menu m
  	ON s.product_id = m.product_id
  GROUP BY s.customer_id, m.product_name
  )
  
 SELECT 
  	customer_id,
    product_name,
    order_count
 FROM most_popular
 WHERE rank = 1
```
 
#### Steps:
  * Create a Common Table Expression (CTE). Within the CTE using DENSE_RANK() ensures there are no gaps in rankings. **PARTITION BY** groups the data by 'customer_id', then is **ORDERED BY** 'customer_id' descending.
  * Filter the data with the **WHERE** clause to find the most popular order from each customer
    
#### Solution
| customer_id | product_name | order_count |
| ----------- | ------------ | ----------- |
| A           | ramen        | 3           |
| B           | ramen        | 2           |
| B           | curry        | 2           |
| B           | sushi        | 2           |
| C           | ramen        | 3           |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)

** Question 6. Which item was purchased first by the customer after they became a member?
```sql
with joined_member AS (
  SELECT
  	s.customer_id,
  	s.order_date,
  	s.product_id,
  	mem.join_date,
  	ROW_NUMBER() OVER (
      PARTITION BY s.customer_id
      ORDER BY s.order_date) AS row_num
  FROM sales s
  JOIN members mem
  	ON s.customer_id = mem.customer_id
  WHERE s.order_date > mem.join_date
)

SELECT 
	customer_id,
    product_name
FROM joined_member jm
JOIN menu m
	ON jm.product_id = m.product_id
WHERE row_num = 1
ORDER BY customer_id ASC;
```


#### Steps:
#### Solution

** Question 7. Which item was purchased just before the customer became a member?
```sql
with before_member AS (
  SELECT
  	s.customer_id,
  	s.order_date,
  	s.product_id,
  	mem.join_date,
  	ROW_NUMBER() OVER (
      PARTITION BY s.customer_id
      ORDER BY s.order_date DESC) AS row_num
  FROM sales s
  JOIN members mem
  	ON s.customer_id = mem.customer_id
  WHERE s.order_date < mem.join_date
)

SELECT 
	customer_id,
    product_name
FROM before_member bm
JOIN menu m
	ON bm.product_id = m.product_id
WHERE row_num = 1
ORDER BY customer_id ASC;

```
#### Steps:
#### Solution





