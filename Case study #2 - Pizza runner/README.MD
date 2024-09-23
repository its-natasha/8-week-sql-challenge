# Case study #2 Pizza Runner

Table of contents
1. Part A.Pizza Metrics
2. Part B.Pizza Metrics
3. Part c.Pizza Metrics

### 1. Part A.Pizza Metrics

#### Cleaning the tables: Query: 
```
WITH customer_orders AS (SELECT 
 	order_id,
	customer_id, 
        pizza_id,
	CASE WHEN exclusions = 'null' THEN '' ELSE 	   exclusions END AS exclusions,
        CASE WHEN lower(TRIM(extras)) = 'null' OR extras IS NULL THEN '' ELSE TRIM(extras) END AS extras,
        order_time
FROM
pizza_runner.customer_orders),

runner_orders AS (
SELECT 
    order_id,
    runner_id,
    CASE WHEN pickup_time IS NULL or lower(pickup_time) = 'null' THEN '' ELSE pickup_time END AS      pickup_time,
    REPLACE(REPLACE(LOWER(distance), 'km', ''), 'null', '') AS distance,
    CASE 
    	WHEN duration LIKE '%km' THEN TRIM(duration, 'km') 
        WHEN duration LIKE '%mins' THEN TRIM(duration, 'mins')
        WHEN duration LIKE '%minute' THEN TRIM(duration, 'minute')
        WHEN duration LIKE '%minutes' THEN TRIM(duration, 'minutes')
        WHEN duration IS NULL or duration = 'null' THEN '' 
        ELSE duration END AS duration,
    CASE
    	WHEN trim(lower(cancellation)) LIKE '%cancel%' THEN 'cancellation'
        WHEN cancellation IS NULL or cancellation = 'null' THEN ''
        ELSE cancellation END AS cancellation
FROM pizza_runner.runner_orders),

pizza_names AS (
  SELECT * 
  FROM pizza_runner.pizza_names)
```

# Solutions
---

#### 1. How many pizzas were ordered?
14

#### 1.2 Query 
```
SELECT COUNT(pizza_id)
FROM customer_orders
```

#### 1.3 Query steps:
- Use the `COUNT` function to determine the total number of rows with values in the `pizza_id` column.
---

#### 2. How many unique customer orders were made?
- 5

#### 2.1 Query
```
SELECT COUNT (DISTINCT CUSTOMER_ID)
FROM customer_orders
```

#### 2.2 Query steps:
- Use `DISTINCT` to extract unique values.
- Use `COUNT` to count the number of values in each row.
---

#### 3. How many successful orders were delivered by each runner?

#### 3.1 Query 
```
SELECT 
    ro.runner_id, 
    COUNT(pickup_time) as successful_deliveries
FROM customer_orders co
LEFT JOIN runner_orders ro
ON co.order_id = ro.order_id
WHERE cancellation <> 'cancellation'
GROUP BY runner_id
```

#### 3.2 Query steps:
- Join the `customer_orders` table with the `runner_orders` table.
- Use `WHERE` on the `cancellation` column to filter out any rows containing 'cancellation'.
- Use `GROUP BY` to aggregate the data by runner.
---

#### 4. How many of each type of pizza was delivered?

#### 4.1 Query:
```
SELECT 
    pn.pizza_id,
    pn.pizza_name,
    COUNT(ro.pickup_time) AS delivered_pizza
FROM runner_orders ro
LEFT JOIN customer_orders co
ON ro.order_id = co.order_id
LEFT JOIN pizza_names pn
ON co.pizza_id = pn.pizza_id
WHERE cancellation <> 'cancellation'
GROUP BY pn.pizza_id, pn.pizza_name
```
#### 4.2 Query steps:
- Perform a left join between `pizza_orders` and `pizza_name` using `order_id`.
- Count the number of vegetarian pizza orders and the number of meat lover pizza orders, grouping the results by `customer_id`.
---

#### 5. How many Vegetarian and Meatlovers were ordered by each customer?

#### 5.1 Query:
```
SELECT
  co.customer_id, 
  COUNT(CASE WHEN pn.pizza_name = 'Vegetarian' THEN 1 END) AS vegetarian,
  COUNT(CASE WHEN pn.pizza_name = 'Meatlovers' THEN 1 END) AS meatlovers
FROM customer_orders co
LEFT JOIN pizza_names pn
ON co.pizza_id = pn.pizza_id
GROUP BY customer_id
ORDER BY customer_id
```

#### 5.2 Query steps:
- Use `COUNT` and `CASE WHEN` to flag a `1` when a customer orders a vegetarian pizza.
- Use `COUNT` and `CASE WHEN` to flag a `1` when a customer orders a meat lover pizza.
- Use `GROUP BY` to organise the results based on `customer_id` and the type of pizza ordered.
---


#### 6. What was the maximum number of pizzas delivered in a single order?

#### 6.1 Query:
```
SELECT 
  co.order_id, 
  COUNT (co.order_id) as order_count
FROM customer_orders co
LEFT JOIN runner_orders as ro
ON co.order_id = ro.order_id
WHERE cancellation NOT LIKE '%cancel'
GROUP by co.order_id
ORDER BY order_count DESC
LIMIT 1
```

#### 6.2 Steps:
- Instead of using the `MAX()` function, opted for `ORDER BY` with `LIMIT`.
- Count the total number of pizza orders.
- Use `WHERE` to filter out cancellations.
- Arrange the results in descending order with `ORDER BY`.
- Use `LIMIT` to display only the top result.

---

#### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

#### 7.1 Query:
```
SELECT
    co.customer_id,
    COUNT(CASE WHEN co.exclusions = '' AND co.extras = '' THEN 1 END) AS original_pizza,
    COUNT (CASE WHEN co.exclusions <> '' OR co.extras <> '' THEN 1 END) AS pizza_changes
FROM customer_orders co
LEFT JOIN runner_orders ro
ON co.order_id = ro.order_id
WHERE ro.cancellation NOT LIKE '%cancel'
GROUP BY co.customer_id
```

#### 7.2 Query steps:
- Count the number of blank entries (`''`) for `exclusions` and `extras` to determine how many pizzas had no changes.
- Count the number of non-blank entries for `exclusions` and `extras` to identify how many pizzas had changes.
- Use `GROUP BY` to organize the results by `customer_id`.
---

#### 8. How many pizzas were delivered that had both exclusions and extras?

#### 8.1 Query:
```
SELECT
    COUNT(CASE WHEN co.exclusions <> '' AND co.extras <> '' THEN 1 END) AS original_pizza
FROM customer_orders co
LEFT JOIN runner_orders ro
ON co.order_id = ro.order_id
WHERE ro.cancellation NOT LIKE '%cancel'
```

#### 8.2 Query steps:
- Count the number of cells with values in `exclusions` and `extras`.
- Use `WHERE` to filter out cancellations.
---

#### 9. What was the total volume of pizzas ordered for each hour of the day?

##### 9.1 Query:

```
SELECT
 hour,
 order_time,
 order_date,
COUNT(order_id) AS volume
FROM (
SELECT *,
EXTRACT (HOUR from order_time) as hour,
CAST(order_time AS DATE) as order_date
FROM customer_orders) AS subquery
GROUP BY hour, order_date, order_time
ORDER BY order_date
```

##### 9.2 Query steps:
- Use `EXTRACT` to retrieve the hours from `order_time`.
- Use `CAST` to obtain the date from `order_time`.
- Use `GROUP BY` to calculate the count based on hour, order date, and `order_time`.
---

#### 10. What was the volume of orders for each day of the week?

#### 10.1 Query:
```
SELECT
  CAST(order_time AS DATE) AS order_date,
  TO_CHAR(order_time, 'day') as order_day,
  COUNT(order_id) AS volume
FROM customer_orders
GROUP BY order_date, order_day
ORDER BY order_date
```

#### 10.2 Query steps:
- Use `CAST` to extract date from order_date

---
