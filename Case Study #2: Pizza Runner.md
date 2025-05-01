# Case Study #2: Pizza Runner
![SQL 2](https://github.com/user-attachments/assets/696fc26e-c1e9-4e31-8099-322757e986ed)

## Table of Contents
  - [Business Task](#business-task)
  - [Entity Relationship Diagram](#entity-relationship-diagram)
  - [Questions and Solutions](#questions-and-solutions)

## Business Task
Danny is expanding his new Pizza Empire and he plans to Uberize it, so Pizza Runner was launched! 

Danny started by recruiting 'runner' to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny's house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

Danny prepared an entity relationship diagram of his database design but requires further assistance to clean his data and apply some basic calculations so he can better direct his runners and optimise Pizza Runner's operation.

## Entity Relationship Diagram
![image](https://github.com/user-attachments/assets/37f7bfbc-263e-4dc1-95dd-adade7a960fc)

## Data Cleaning and Transformation

### Table: customer_orders
* The `customer_order` table below is not workable in its current state
  * There are missing and null values in the `exclusions` column
  * There are missing and null values in the `extras`
 ![image](https://github.com/user-attachments/assets/a177bfc3-02cf-442a-b85f-ee3b1dea743e)

#### Steps:
* Create a temporary table copy of the `customer_order` table
* Remove null values and replace with a blank space ' '.
```sql
CREATE TEMP TABLE customer_orders_temp AS
SELECT
  order_id,
  customer_id,
  pizza_id,
  CASE
    WHEN exclusions IS NULL OR exclusions LIKE 'null' THEN ' '
    ELSE exclusions
    END AS exclusions,
  CASE
    WHEN extras IS NULL OR extras LIKE 'null' THEN ' '
    ELSE extras
    END AS extras,
  order_time
FROM pizza_runner.customer_orders
```
The new `customer_orders_temp` table looks like this
![image](https://github.com/user-attachments/assets/3bf4f986-b4a8-4f05-a841-5562f421e428)

### Table: runner_orders
* The `runner_orders` table below also needs some cleaning
  * There are null values in the `pickup_time` column
  * There are inconsistent values in the `distance` column
  * There are inconsitent values in the `duration` column
  * There are missing and null values in the `cancellation` column
 ![image](https://github.com/user-attachments/assets/8b8e224c-c949-4400-a36c-9d7a2bee45bf)

#### Steps: 
* Create a temporary table copy of the `runner_orders` table
* In the `pickup_time` column, remove null values and replace with a blank space ' '
* In the `distance`, remove 'km' and null values and replace a blank space ' '
* In the `duration`, remove 'minutes', 'minute', 'mins' and null values and replace a blank space ' '
* In the `cancellation`, remove null values and replace a blank space ' '

```sql
CREATE TEMP TABLE runner_orders_temp AS 
SELECT 
	order_id,
    runner_id,
    CASE
    	WHEN pickup_time LIKE 'null' THEN ' '
        ELSE pickup_time
     END AS pickup_time,
     CASE 
     	WHEN distance LIKE 'null' THEN ' '
        WHEN distance LIKE '%km' THEN TRIM('km' FROM distance)
        ELSE distance
     END AS distance,
     CASE
     	WHEN duration LIKE 'null' THEN ' '
        WHEN duration LIKE '%minutes' THEN TRIM('minutes' FROM duration)
        WHEN duration LIKE '%minute' THEN TRIM('minute' FROM duration)
        WHEN duration LIKE '$mins' THEN TRIM('mins' FROM duration)
        ELSE duration
      END AS duration,
      CASE
      	WHEN cancellation IS NULL OR cancellation LIKE 'null' THEN ' '
        ELSE cancellation
       END AS cancellation
FROM runner_orders
```

![image](https://github.com/user-attachments/assets/618ebb11-ddd6-4253-8d9f-39034b1d11e8)

## A. Pizza Metrics
**Question 1: How many pizzas were ordered?**
```sql
SELECT 
	COUNT(*) AS order_count
FROM customer_orders_temp
```
#### Steps:
#### Solution:

| order_count |
| ----------- |
| 14          |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/3417#)

**Question 2: How many unique customer orders were made**
```sql
SELECT 
	COUNT(DISTINCT customer_id) as unique_customers
FROM customer_orders_temp
```
#### Steps:
#### Solution:

| unique_customers |
| ---------------- |
| 5                |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/3417#)

**Question 3: How many successful orders were delivered by each runner?**
```sql
SELECT
	runner_id,
	COUNT(*) 
FROM runner_orders_temp
WHERE cancellation NOT LIKE '%Cancellation'
GROUP BY runner_id
ORDER BY runner_id
```
#### Steps:
#### Solution:

| runner_id | count |
| --------- | ----- |
| 1         | 4     |
| 2         | 3     |
| 3         | 1     |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/3417#)

**Question 4: How many of each type of pizza was delivered?**
```sql
SELECT 
	p.pizza_name,
    COUNT(c.*)
FROM customer_orders_temp c
JOIN pizza_names p
	ON c.pizza_id = p.pizza_id
JOIN runner_orders_temp r
	ON c.order_id = r.order_id
 WHERE cancellation NOT LIKE '%Cancellation'
GROUP BY p.pizza_name
```
#### Steps:
#### Solution:

| pizza_name | count |
| ---------- | ----- |
| Meatlovers | 9     |
| Vegetarian | 3     |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/3417#)

**Question 5: How many Vegetarian and Meatlovers were ordered by each customer?**
```sql
SELECT
  customer_id,
  COUNT(CASE WHEN pizza_name = 'Meatlovers' THEN 1 END) AS meatlovers_count,
  COUNT(CASE WHEN pizza_name = 'Vegetarian' THEN 1 END) AS vegetarian_count
FROM
  runner_orders_temp r
JOIN customer_orders_temp c
  ON r.order_id = c.order_id
JOIN pizza_names p
  ON c.pizza_id = p.pizza_id
GROUP BY customer_id
ORDER BY customer_id
```
#### Steps:
#### Solution:

| customer_id | meatlovers_count | vegetarian_count |
| ----------- | ---------------- | ---------------- |
| 101         | 2                | 1                |
| 102         | 2                | 1                |
| 103         | 3                | 1                |
| 104         | 3                | 0                |
| 105         | 0                | 1                |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/3417#)

**Question 6: What was the maximum number of pizzas delivered in a single order?**
#### Steps:
#### Solution:

**Question 7: For each customer, how many delivered pizzas had at least 1 change and how many how no changes?**
#### Steps:
#### Solution:

**Question 8: How many pizzas were delivered that had both exclusions and extras?**
#### Steps:
#### Solution:

**Question 9: What was the total volume of pizzas ordered for each hour of the day?**
#### Steps:
#### Solution:

**Question 10: What was the volume of orders for each day of the week?**
#### Steps:
#### Solution:





