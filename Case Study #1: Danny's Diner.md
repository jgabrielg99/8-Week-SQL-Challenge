# Case Study #1: Danny's Diner
![SQL 1](https://github.com/user-attachments/assets/2e5427a4-aec5-4935-a48c-c3abc83d5145)

## Table of Contents
  - [Business Task](#business-task)
  - [Entity Relationship Diagram](#entity-relationship-diagram)
  - [Questions and Solutions](#questions-and-solutions)

## Business Task

Danny wants to use data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they've spent and also which menu items are their favorite.

We have 3 key datasets to work with
  * sales
  * menu
  * members

## Entity Relationship Diagram
![image](https://github.com/user-attachments/assets/b72c7387-37fb-49c1-866a-444a2a1bfaa8)


## Questions and Solutions

**Question 1: What is the total amount each customer spent at the restaurant?**
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
  * Use **INNER JOIN** to combine the `sales` and `menu` tables to access `sales.customer_id` and `menu.price`
  * Use **SUM** to calculate the total amount spent by each customer
  * Group the aggregated results by `sales.customer_id`

#### Solution
| customer_id | total_amount |
| ----------- | ------------ |
| A           | 76           |
| B           | 74           |
| C           | 36           |

* Customer A spent $76
* Customer B spent $74
* Customer C spent $36

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)
---

**Question 2. How many days has each customer visited the restaurant?**
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

* Customer A visited 4 times
* Customer B visited 6 times
* Customer C visited 2 times

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)
---

**Question 3. What was the first item from the menu purchased by each customer?**
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
  * Using a subquery, create a temporary table that finds the first (**MIN**) `order_date` for each customer
  * Use **JOIN** to merge `sales` and `menu` to match `product_name` with `customer_id`

#### Solution

| customer_id | order_date | product_name |
| ----------- | ---------- | ------------ |
| A           | 2021-01-01 | sushi        |
| A           | 2021-01-01 | curry        |
| B           | 2021-01-01 | curry        |
| C           | 2021-01-01 | ramen        |
| C           | 2021-01-01 | ramen        |

 * The first order customer A placed was sushi and curry
 * The first order customer B placed was curry
 * The first order customer C placed was ramen

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)
---

**Question 4. What is the most purchased item on the menu and how many times was it purchased by all customers?**
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
* To find the most purchased item, use the **COUNT** and **GROUP BY** clauses to total the number of orders for each product
	*Run this query to find the most popular item
*To find the how many times each customer ordered the most popular item, using the results from the previous query, **JOIN** `sales` and `menu` then filter to only include rows where 'ramen' was ordered.
* **COUNT** those rows and **GROUP BY** `customer_id` to calculate how many times each customer ordered ramen.

#### Solution

| product_name | most_ordered |
| ------------ | ------------ |
| ramen        | 8            |
| curry        | 4            |
| sushi        | 3            |

* Ramen is the most purchased item on the menu having been ordered 8 times


| customer_id | count |
| ----------- | ----- |
| A           | 3     |
| B           | 2     |
| C           | 3     |

* Customer A ordered ramen 3 times
* Customer B ordered ramen 2 times
* Customer C ordered ramen 3 times

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)
---

**Question 5. Which item was the most popular for each customer?**
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
  * Create a Common Table Expression (CTE). Within the CTE using DENSE_RANK() ensures there are no gaps in rankings. **PARTITION BY** divides the data by `customer_id`, then is **ORDERED BY** `customer_id` descending.
  * Filter the data with the **WHERE** clause to find the most popular order from each customer
    
#### Solution
| customer_id | product_name | order_count |
| ----------- | ------------ | ----------- |
| A           | ramen        | 3           |
| B           | ramen        | 2           |
| B           | curry        | 2           |
| B           | sushi        | 2           |
| C           | ramen        | 3           |

* Customer A ordered ramen the most amount of times
* Customer B ordered all 3 menu items the same number of times
* Customer C ordered ramen the most amount of times

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)
---

**Question 6. Which item was purchased first by the customer after they became a member?**
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
* Create a CTE with customer order data and use **ROW_NUMBER** to index each row.
	* **PARTITION BY** customer_id so the indexing resets for each customer and **ORDER BY** order_date ascending so the first row will be the earliest date
* Use a **WHERE** clause to a ensure the CTE returns orders that were place after the customer became a member
* Filter the results to include only the rows where the index is 1 to find the first purchase the new member made

#### Solution

| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | sushi        |

* The first item Customer A purchased after becoming a member was ramen
* The first item Customer B purchased after becoming a member was sushi

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)
---

**Question 7. Which item was purchased just before the customer became a member?**
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
* Create a CTE with customer order data and use **ROW_NUMBER** to index each row.
	* **PARTITION BY** customer_id so the indexing resets for each customer and **ORDER BY** order_date descending so the first row will be the latest date
* Use a **WHERE** clause to a ensure the CTE returns orders that were place prior to the customer becoming a member
* Filter the results to include only the rows where the index is 1 to find the last purchase the customer made

#### Solution
| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| B           | sushi        |

* The last item Customer A purchased before becoming a member was sushi
* The last item Customer B purchased before becoming a member was sushi

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)
---

**Question 8. What is the total items and amount spent for each member before they became a member?**
  ```sql  
    SELECT 
    	s.customer_id,
        COUNT(s.product_id) AS order_count,
        SUM(m.price) AS total_sales
    FROM sales s
    JOIN members mem
    	ON s.customer_id = mem.customer_id
        AND s.order_date < mem.join_date
    JOIN menu m
    	ON s.product_id = m.product_id
    GROUP BY s.customer_id
    ORDER BY s.customer_id
```
#### Steps:
* Use **COUNT** to tally up the number of `product_id` and **SUM** to add up prices of each item.
* Join `members` table on the `sales` table before filtering for only orders that were placed *before* the customer's join date.
* Join `menu` table on `sales` table to bring in `price`
* **GROUP BY** and **ORDER BY** `customer_id`
#### Solution

| customer_id | order_count | total_sales |
| ----------- | ----------- | ----------- |
| A           | 2           | 25          |
| B           | 3           | 40          |

* Customer A ordered 2 items and spent $25 before becoming a member
* Customer B ordered 3 items and spent $40 before becoming a member

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)
---

**Question 9. If each $1 spend equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**
```sql
with points as (
  SELECT 
	s.customer_id,
  	m.product_id,
    CASE 
    	WHEN s.product_id = 1 
        	THEN m.price * 20
        ELSE m.price *10
  	  END AS points
	FROM sales s
	JOIN menu m
		ON s.product_id = m.product_id)
        
SELECT 
	customer_id, 
    SUM(points)
FROM points
GROUP BY customer_id
ORDER BY customer_id
```

#### Steps:
* Within a CTE, use the **CASE** clause to calculate the number of points awarded for each `product_id`.
  * If each $1 spent is worth 10 points, the number of points awarded for an order is calculated by mulitplying the `m.price` by 10.
  * However, sushi has a 2x multiplier, so any order of sushi is calculated by multiplying `m.price` by 20.
* Use **SUM** to calculate the total number of points each customer has.

#### Solution

| customer_id | sum |
| ----------- | --- |
| A           | 860 |
| B           | 940 |
| C           | 360 |

* Customer A has 860 points
* Customer B has 940 points
* Customer C has 360 points

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)
---


**Question 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

```sql
with promo as (
  SELECT 
  	customer_id,
  	join_date,
  	join_date + 6 AS promo_end
  FROM members mem)
  
SELECT 
	s.customer_id,
    SUM(CASE
        WHEN m.product_name = 'sushi' THEN m.price * 20
        WHEN s.order_date BETWEEN promo.join_date AND promo.promo_end THEN m.price * 20
        ELSE m.price * 10
     END) AS points
FROM sales s
JOIN promo 
	ON s.customer_id = promo.customer_id
    AND promo.join_date <= s.order_date
    AND s.order_date <= '2021-01-31'
JOIN menu m 
	ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id
```

#### Steps:
* Create a CTE for the dates of the 2x points promotion
	* The promotion lasts entire week the customer joins as a member, so `join_date + 6` will encompass the customer's first week as a member
* Similar to the previous task, use **CASE** to apply the $1 * 10 points for ramen and curry, and $1 * 20 points for sushi
  	* However during the dates of the promotion, everything will receive a x2 multiplier - the addition **WHEN** clause includes this
* To calculate the number of points acquired by the end of January, set the cutoff date as January 31st (2021-01-31) when joining the `promo` CTE with the `sales` table. 

#### Solution

| customer_id | points |
| ----------- | ------ |
| A           | 1020   |
| B           | 320    |

* Customer A earned 1020 points during the month of January
* Customer B earned 320 points during the month of January

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)
---






