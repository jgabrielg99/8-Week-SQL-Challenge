# Case Study #3: Foodie-Fi
![image](https://github.com/user-attachments/assets/91c5ff13-bc55-46bc-b5af-d88757d21106)


## Table of Contents
  - [Business Task](#business-task)
  - [Entity Relationship Diagram](#entity-relationship-diagram)
  - [Questions and Solutions](#questions-and-solutions)

## Business Task

Danny and a few of his smart friends launched a new on-demand streaming service strictly devotes to food related content: Foodie-Fi. 

This case study focuses on using subscription style digital data to answer important business questions, gain insight on customer journeys, payments, and business metrics.

We have 2 key datasets to work with
  * plans
  * subscriptions

## Entity Relationship Diagram
![image](https://github.com/user-attachments/assets/f18a1de5-6d2c-4f25-a0b3-23aa3a3360be)

## Questions and Solutions
## A. Customer Journey
** Based on the customer samples provided from the `subscriptions` table, write a brief description about each customer's onboarding journey. 
```sql
SELECT 
	s.customer_id,
    p.plan_name,
    s.start_date
FROM subscriptions s
JOIN plans p 
	ON s.plan_id = p.plan_id
WHERE s.customer_id IN (1,2,11,13,15,16,18,19)
ORDER BY s.customer_id
```
| customer_id | plan_name     | start_date |
| ----------- | ------------- | ---------- |
| 1           | trial         | 2020-08-01 |
| 1           | basic monthly | 2020-08-08 |
| 2           | trial         | 2020-09-20 |
| 2           | pro annual    | 2020-09-27 |
| 11          | trial         | 2020-11-19 |
| 11          | churn         | 2020-11-26 |
| 13          | trial         | 2020-12-15 |
| 13          | basic monthly | 2020-12-22 |
| 13          | pro monthly   | 2021-03-29 |
| 15          | trial         | 2020-03-17 |
| 15          | pro monthly   | 2020-03-24 |
| 15          | churn         | 2020-04-29 |
| 16          | trial         | 2020-05-31 |
| 16          | basic monthly | 2020-06-07 |
| 16          | pro annual    | 2020-10-21 |
| 18          | trial         | 2020-07-06 |
| 18          | pro monthly   | 2020-07-13 |
| 19          | trial         | 2020-06-22 |
| 19          | pro monthly   | 2020-06-29 |
| 19          | pro annual    | 2020-08-29 |

Customer 1: This customer started their `trial` period on August 1, 2020. At the end of their week-long trial, they decided to subscribe to the `basic monthly` plan.

Customer 2: Following their `trial` which began on September 20, 2020, Customer 2 subscribed to the `pro annual` plan.

Customer 11: This customer began their `trial` period on November 19, 2020 and decided to cancel the service at the end of the trial. 

Customer 13: Following their `trial` which began on December 15, 2020, this customer initially subscribed to the `basic monthly` at the end of their trial, before ultimately upgrading to the `pro monthly` plan a few months later on March 29, 2021.

Customer 15: At the end of Customer 15's `triaL` period which began on March 17, 2020, they subcribed to the `pro monthly` plan. However, after approximately a month, this customer cancelled their services. 

Customer 16: Customer 16 began this `trial` period on May 31, 2020. Following their trial, they subscribed to the `basic monthly` plan before upgrading to `pro annual` several months later on October 21, 2020.

Customer 18: This customer began their `trial` on July 6, 2020 and subcribed to the `pro monthly` plan at the end of their trial. 

Customer 19: After Customer 19's `trial` period which began on June 22, 2020, they subscribed to the `pro monthly` plan before upgrade to `pro annual` two months later. 

## B. Data Analysis Questions:
**Question 1: How many customers has Foodie-Fi ever had?**
```sql
SELECT
	COUNT(DISTINCT customer_id)
FROM subscriptions
```
#### Steps:
* Use **COUNT DISTINCT** to count only each unique `customer_id`
  
#### Solution:
| count |
| ----- |
| 1000  |

**Question 2: What is the monthly distribution of `trial` plan `start_date` values for our dataset? Use the start of the month as the group by value**
```sql
SELECT
	DATE_PART('month', start_date) AS month_date,
	COUNT(s.customer_id) as customer_count
FROM subscriptions s
JOIN plans p
	ON s.plan_id = p.plan_id
WHERE p.plan_name = 'trial'
GROUP BY DATE_PART('month', start_date)
ORDER BY month_date
```
#### Steps:
* The **DATE_PART** clause will extract each 'month' value from the `start_date` column
* **COUNT** the number of `customer_id`
* JOIN the `plans` table on the `subscriptions` table to be able to filter to only show results where the `plan_name` is 'trial'
	* Alternatively, you could forego the join and filter by `plan_id` = 0 (demonstrated in question 4); however for scalability purposed, if there were a higher quantity of `plan_id` values, joining is the way to go 

#### Solution:
| month_date | customer_count |
| ---------- | -------------- |
| 1          | 88             |
| 2          | 68             |
| 3          | 94             |
| 4          | 81             |
| 5          | 88             |
| 6          | 79             |
| 7          | 89             |
| 8          | 88             |
| 9          | 87             |
| 10         | 79             |
| 11         | 75             |
| 12         | 84             |

**Question 3: What plan `start_date` values occur after the year 2020 for our dataset? Show the breakdown by count of events for each `plan_name`.**
```sql
SELECT
	p.plan_id,
	p.plan_name,
	COUNT(s.customer_id)
FROM subscriptions s
JOIN plans p
	ON s.plan_id = p.plan_id
WHERE s.start_date > '2020-12-31'
GROUP BY p.plan_name, p.plan_id
ORDER BY p.plan_id
```
#### Steps:
* **COUNT** the number of `customer_id` values
* Then filter to only return results where the `start_date` is greater than December 31, 2020
 
#### Solution:
| plan_id | plan_name     | count |
| ------- | ------------- | ----- |
| 1       | basic monthly | 8     |
| 2       | pro monthly   | 60    |
| 3       | pro annual    | 63    |
| 4       | churn         | 71    |


**Question 4: What is the customer count and percentage of customers who have churned rounded to 1 decimal place?**
```sql
SELECT 
	COUNT(s.customer_id) AS churned_count,
    ROUND(100.0 * COUNT(s.customer_id)/
          (SELECT COUNT(DISTINCT customer_id) FROM subscriptions), 1) AS churn_percentage
FROM subscriptions s
WHERE s.plan_id = 4 -- Churned plan_id is 4
```
#### Steps:
* Use **COUNT** to count the `customer_id` values
* To find the percentage, divide the number of customers who have churned by the total number of customers. To do this, use a subquery in the denominator and the filtered results in the numerator
* **ROUND** the percentage value to 1 decimal place
* Filter the results to only return `plan_id` = 4 -- churned

#### Solution:
| churned_count | churn_percentage |
| ------------- | ------------- |
| 307           | 30.7          |

**Question 5: How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?**
```sql
WITH plan_ranked AS (
  SELECT 
  	ROW_NUMBER() OVER(PARTITION BY s.customer_id
                  ORDER BY s.start_date) AS rank,
  	s.customer_id,
  	p.plan_name
  FROM subscriptions s
  JOIN plans p
  	ON s.plan_id = p.plan_id
  )
  
SELECT 
	COUNT(CASE
          WHEN rank = 2 AND plan_name = 'churn' THEN 1
          ELSE 0
          END) AS churned_customers,
    ROUND(100.0 * COUNT(CASE
          WHEN rank = 2 AND plan_name = 'churn' THEN 1
          ELSE 0
          END) / (SELECT COUNT(DISTINCT customer_id) FROM subscriptions), 0) AS churn_perc
FROM plan_ranked
WHERE plan_name = 'churn'
    AND rank = 2
```
#### Steps:
#### Solution:
| churned_customers | churn_perc |
| ----------------- | ---------- |
| 92                | 9          |


**Question 6: What is the number and percentage of customer plans after their initial free trail?**
```sql
WITH after_trial AS (
  SELECT
  	customer_id,
  	plan_id,
	LEAD(plan_id) OVER(PARTITION BY(customer_id) ORDER BY plan_id) as resub_plan_id
  FROM subscriptions
)

SELECT
	resub_plan_id,
    COUNT(customer_id) AS plan_count,
    ROUND(100.0 * COUNT(customer_id) / (SELECT COUNT(DISTINCT customer_id) FROM subscriptions), 1) AS plan_percentage
FROM after_trial
WHERE resub_plan_id IS NOT NULL
	AND plan_id =0
GROUP BY resub_plan_id
ORDER BY resub_plan_id
```
#### Steps:
#### Solution:
| resub_plan_id | plan_count | plan_percentage |
| ------------- | ---------- | --------------- |
| 1             | 546        | 54.6            |
| 2             | 325        | 32.5            |
| 3             | 37         | 3.7             |
| 4             | 92         | 9.2             |

**Question 7: What is the customer count and percentage breakdown of all 5 `plan_name` values at `2020-12-31`**
```sql
WITH next_date AS (
  SELECT 
  	customer_id,
  	plan_id,
  	start_date,
  	LEAD(start_date) OVER(PARTITION BY customer_id ORDER BY start_date) AS next_date
  FROM subscriptions
  WHERE start_date <= '2020-12-31'
)

SELECT 
	plan_id,
    COUNT(customer_id) AS plan_count,
    ROUND(100.0 * COUNT(DISTINCT customer_id) / (SELECT COUNT(DISTINCT customer_id) FROM subscriptions), 1) AS plan_percentage
FROM next_date
WHERE next_date IS NULL
GROUP BY plan_id
ORDER BY plan_id
```
#### Steps:
#### Solution:
| plan_id | plan_count | plan_percentage |
| ------- | ---------- | --------------- |
| 0       | 19         | 1.9             |
| 1       | 224        | 22.4            |
| 2       | 326        | 32.6            |
| 3       | 195        | 19.5            |
| 4       | 236        | 23.6            |

**Question 8: How many customers have upgraded to an annual plan in 2020?**
```sql
SELECT
	COUNT(DISTINCT customer_id) AS annual_plans
FROM subscriptions
WHERE plan_id = 3
	AND start_date <= '2020-12-31'
```
#### Steps:
#### Solution:
| annual_plans |
| ------------ |
| 195          |


**Question 9: How many days on average does it take for a customer to upgrade to an annual plan from the day they join Foodie-Fi?**
```sql
WITH trial_date AS (
  SELECT 
  	customer_id, 
  	start_date AS trial_date
  FROM subscriptions
  WHERE plan_id = 0
  ),
 annual_date AS (
   SELECT
   	customer_id,
   	start_date AS annual_date
   FROM subscriptions
   WHERE plan_id = 3
   )
   
SELECT 
	ROUND(AVG(a.annual_date - t.trial_date), 0)  AS avg_upgrade_days
FROM trial_date t
JOIN annual_date a
	ON t.customer_id = a.customer_id
```
#### Steps:
#### Solution:
| avg_upgrade_days |
| ---------------- |
| 105              |

**Question 10: Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days, etc.)**
#### Steps:
#### Solution:

**Question 11: How many customers downgraded from a pro monthly to a basic monthly plan in 2020?**
#### Steps:
#### Solution:


