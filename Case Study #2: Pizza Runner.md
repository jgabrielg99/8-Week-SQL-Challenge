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
* Remove null values and replace with an empty string ''.
```sql
CREATE TEMP TABLE customer_orders_temp AS
SELECT
  order_id,
  customer_id,
  pizza_id,
  CASE
    WHEN exclusions IS NULL OR exclusions LIKE 'null' THEN ''
    ELSE exclusions
    END AS exclusions,
  CASE
    WHEN extras IS NULL OR extras LIKE 'null' THEN ''
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
* In the `pickup_time` column, remove null values and replace with a empty string ''
* In the `distance`, remove 'km' and null values and replace a empty string ''
* In the `duration`, remove 'minutes', 'minute', 'mins' and null values and replace a empty string ''
* In the `cancellation`, remove null values and replace a empty string ''

```sql
CREATE TEMP TABLE runner_orders_temp AS 
SELECT 
  order_id,
  runner_id,
  CASE
    WHEN pickup_time LIKE 'null' THEN ''
    ELSE pickup_time
  END AS pickup_time,
  CASE 
    WHEN distance LIKE 'null' THEN ''
    WHEN distance LIKE '%km' THEN TRIM('km' FROM distance)
    ELSE distance
  END AS distance,
  CASE
    WHEN duration LIKE 'null' THEN ''
    WHEN duration LIKE '%minutes' THEN TRIM('minutes' FROM duration)
    WHEN duration LIKE '%minute' THEN TRIM('minute' FROM duration)
    WHEN duration LIKE '$mins' THEN TRIM('mins' FROM duration)
    ELSE duration
  END AS duration,
  CASE
    WHEN cancellation IS NULL OR cancellation LIKE 'null' THEN ''
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
```sql
SELECT 
  c.order_id,
  COUNT(c.pizza_id) AS order_count
FROM customer_orders c
JOIN runner_orders r
  ON c.order_id = r.order_id
WHERE r.distance IS NOT NULL
GROUP BY c.order_id
ORDER BY order_count DESC
```
---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)
#### Steps:
#### Solution:
| order_id | order_count |
| -------- | ----------- |
| 4        | 3           |
| 3        | 2           |
| 10       | 2           |
| 1        | 1           |
| 7        | 1           |
| 8        | 1           |
| 9        | 1           |
| 6        | 1           |
| 2        | 1           |
| 5        | 1           |

**Question 7: For each customer, how many delivered pizzas had at least 1 change and how many how no changes?**
```sql
SELECT 
  c.customer_id,
  SUM(
    CASE WHEN COALESCE(c.exclusions, '') <> '' OR COALESCE(c.extras, '') <> '' THEN 1
    ELSE 0
  END) AS with_changes,
  SUM(
    CASE WHEN COALESCE(c.exclusions, '') = '' AND COALESCE(c.extras, '') = '' THEN 1
    ELSE 0
  END) AS no_change
FROM customer_orders_temp AS c
JOIN runner_orders_temp AS r
  ON c.order_id = r.order_id
WHERE r.cancellation NOT LIKE '%Cancellation'
GROUP BY c.customer_id
ORDER BY c.customer_id;
```
#### Steps:
#### Solution:
| customer_id | with_changes | no_change |
| ----------- | ------------ | --------- |
| 101         | 0            | 2         |
| 102         | 0            | 3         |
| 103         | 3            | 0         |
| 104         | 2            | 1         |
| 105         | 1            | 0         |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)

**Question 8: How many pizzas were delivered that had both exclusions and extras?**
```sql
SELECT 
  c.customer_id,
  SUM(
    CASE WHEN COALESCE(c.exclusions, '') <> '' AND COALESCE(c.extras, '') <> '' THEN 1
    ELSE 0
  END) AS with_exclusions_extras
FROM customer_orders_temp c
JOIN runner_orders_temp r
  ON c.order_id = r.order_id
WHERE r.cancellation NOT LIKE '%Cancellation'
  AND c.exclusions <> ''
  AND c.extras <> ''
GROUP BY c.customer_id
```
#### Steps:
#### Solution:
| customer_id | with_exclusions_extras |
| ----------- | ---------------------- |
| 104         | 1                      |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)

**Question 9: What was the total volume of pizzas ordered for each hour of the day?**
```sql
SELECT
  EXTRACT(HOUR FROM order_time) as hour,
  COUNT(order_id)
FROM customer_orders_temp
GROUP BY hour
ORDER BY hour
```

#### Steps:
#### Solution:
| hour | count |
| ---- | ----- |
| 11   | 1     |
| 13   | 3     |
| 18   | 3     |
| 19   | 1     |
| 21   | 3     |
| 23   | 3     |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)

**Question 10: What was the volume of orders for each day of the week?**
```sql
SELECT 
  TO_CHAR(order_time, 'Day') AS day_of_week,
  COUNT(order_id) AS total_pizzas_ordered
FROM customer_orders_temp
GROUP BY TO_CHAR(order_time, 'Day')
ORDER BY MIN(EXTRACT(DOW FROM order_time))
```
#### Steps:
#### Solution:
| day_of_week | total_pizzas_ordered |
| ----------- | -------------------- |
| Wednesday   | 5                    |
| Thursday    | 3                    |
| Friday      | 1                    |
| Saturday    | 5                    |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)

## B. Runner and Customer Experience

**Question 1: How many runners signed up for each 1 week period? (i.e. week starts 2020-01-01)**
```sql
    SELECT 
    	EXTRACT(WEEK FROM registration_date) as week,
        COUNT(runner_id)
    FROM runners
    GROUP BY week
    ORDER BY week
```
#### Steps:
#### Solution:
| week | count |
| ---- | ----- |
| 1    | 2     |
| 2    | 1     |
| 3    | 1     |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)

**Question 2: What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?**
```sql
    SELECT
    	r.runner_id,
       	AVG(EXTRACT(EPOCH FROM (CAST(r.pickup_time AS TIMESTAMP) - c.order_time)) / 60) as avg_pickup_minutes
    FROM customer_orders_temp c
    JOIN runner_orders_temp r
    	ON c.order_id = r.order_id
    WHERE r.cancellation NOT LIKE '%Cancellation'
    GROUP BY r.runner_id
    ORDER BY r.runner_id
```
#### Steps:
#### Solution:
| runner_id | avg_pickup_minutes |
| --------- | ------------------ |
| 1         | 15.677777777777777 |
| 2         | 23.720000000000002 |
| 3         | 10.466666666666667 |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)

**Question 3: Is there any relationship between the number of pizzas and how long the order takes to prepare?**
```sql
    WITH prep_time_cte AS(
      SELECT
          c.order_id,
      		COUNT(c.order_id) as pizza_count,
          EXTRACT(EPOCH FROM(CAST(r.pickup_time AS TIMESTAMP) - c.order_time))/60 as prep_time
      FROM customer_orders_temp c
      JOIN runner_orders_temp r
          ON c.order_id = r.order_id
      WHERE r.cancellation NOT LIKE '%Cancellation'
      GROUP BY c.order_id, r.pickup_time, c.order_time
      ORDER BY c.order_id
      )
      
    SELECT
    	pizza_count,
        AVG(prep_time) AS avg_prep_time
    FROM prep_time_cte
    WHERE prep_time > 1
    GROUP BY pizza_count
```
#### Steps:
#### Solution:
| pizza_count | avg_prep_time      |
| ----------- | ------------------ |
| 3           | 29.283333333333335 |
| 2           | 18.375             |
| 1           | 12.356666666666666 |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)

**Question 4: What was the average distance travelled for each customer?**

#### Steps:
#### Solution:

**Question 5: What was the difference between the longest and shortest delivery times for all orders?**
#### Steps:
#### Solution:

**Question 6: What was the average speed for each runner for each delivery and do you notice any trend for these values?**
#### Steps:
#### Solution:

**Question 7: What is the successful delivery percentage for each runner?**
#### Steps:
#### Solution:

## C. Ingredient Optimization

**Question 1: What are the standard ingredients for each pizza?**
#### Steps:
#### Solution:

**Question 2: What was the msot commonly added extra?**
#### Steps:
#### Solution:

**Question 3: What was the most common exclusion?**
#### Steps:
#### Solution:

**Question 4: Generate an order item for each record in the `customer_orders` table in the format of one of the following:
* Meat Lovers
* Meat Lovers - Exclude Beef
* Meat Lovers - Extra Bacon
* Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers**
#### Steps:
#### Solution:

**Question 5: Generate an alphabetically ordered comma separated ingredient list for each pizza order form the `customer_orders` table and add a `2x` in front of any relevant ingredients
* For example: `"Meat Lovers: 2xBacon, Beef, ... , Salami"`**
#### Steps:
#### Solution:

**Question 6: What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?**
#### Steps:
#### Solution:

## D. Pricing and Ratings

**Question 1: If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?**
#### Steps:
#### Solution:

**Question 2: What if there was an additional $1 charge for any pizza extras?
* Add cheese is $1 extra**
#### Steps:
#### Solution:

**Question 3: The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 and 5.**
#### Steps:
#### Solution:

**Question 4: Using your newly generated table - can you join all of the information together to form a table which has the following information for successul deliveries?
* `customer_id`
* `order_id`
* `runner_id`
* `rating`
* `order_time`
* `pickup_time`
* Time between order and pickup
* Delivery duration
* Average speed
* Total number of pizzas**
#### Steps:
#### Solution:

**Question 5: If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometer traveled - how much money does Pizza Runner have left over after these deliveries?**
#### Steps:
#### Solution:





