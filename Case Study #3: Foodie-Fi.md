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

---

[View on DB Fiddle](https://www.db-fiddle.com/f/rHJhRrXy5hbVBNJ6F6b9gJ/16)

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
#### Steps:
#### Solution:

**Question 2: What is the monthly distribution of `trial` plan `start_date` values for our dataset? Use the start of the month as the group by value**
#### Steps:
#### Solution:

**Question 3: What plan `start_date` values occur after the year 2020 for our dataset? Show the breakdown by count of events for each `plan_name`.**
#### Steps:
#### Solution:

**Question 4: What is the customer count and percentage of customers who have churned rounded to 1 decimal place?**
#### Steps:
#### Solution:

**Question 5: How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?**
#### Steps:
#### Solution:

**Question 6: What is the number and percentage of customer plans after their initial free trail?**
#### Steps:
#### Solution:

**Question 7: What is the customer count and percentage breakdown of all 5 `plan_name` values at `2020-12-31`**
#### Steps:
#### Solution:

**Question 8: How many customers have upgraded to an annual plan in 2020?**
#### Steps:
#### Solution:

**Question 9: How many days on average does it take for a customer to upgrade to an annual plan from the day they join Foodie-Fi?**
#### Steps:
#### Solution:

**Question 10: Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days, etc.)**
#### Steps:
#### Solution:

**Question 11: How many customers downgraded from a pro monthly to a basic monthly plan in 2020?**
#### Steps:
#### Solution:


