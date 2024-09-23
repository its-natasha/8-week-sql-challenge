# Case study #2 Pizza Runner

There are 3parts to this case study, click on the following links to skip to Part A, B or C:
- Part A.Pizza Metrics
- Part B.Pizza Metrics
- Part c.Pizza Metrics

# Part A.Pizza Metrics

#### Cleaning up customer_order table:

##### Code
```
WITH customer_orders AS (SELECT 
	order_id,
	customer_id, 
    	pizza_id,
	CASE WHEN exclusions = 'null' THEN '' ELSE 	   exclusions END AS exclusions,
    	CASE WHEN lower(TRIM(extras)) = 'null' OR extras IS NULL THEN '' ELSE TRIM(extras) END AS extras,
    	order_time
	FROM
	pizza_runner.customer_orders)

SELECT * FROM customer_orders

##### Steps taken: 
- Replace IS NULL or NULL values as an empty string 
- Use `strip()` to remove white space
- Use `lower()` to update all characters to consistent lower case 

```

#### Cleaning up runner_orders table:

##### Code

```
SELECT 
    order_id,
    runner_id,
    CASE WHEN pickup_time IS NULL or lower(pickup_time) = 'null' THEN '' ELSE pickup_time END AS pickup_time,
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
FROM pizza_runner.runner_orders
```

# Solutions
### 1. How many pizzas were ordered?
- 14

#### Code:
```
SELECT COUNT(pizza_id)
FROM customer_orders
```

#### Steps taken:
- Use `count` to count the total number of rows with values in the pizza_id column



### 2. How many unique customer orders were made?
- 5

#### 2.1 Code solution:
```
SELECT COUNT (DISTINCT CUSTOMER_ID)
FROM customer_orders
```

#### 2.2 Steps taken:
- Use `DISTINCT` to extract unique values
- Use `COUNT` to count number of values in a row


### 3. How many successful orders were delivered by each runner?
We need to count all of the deliveries made by each runner, and exclude the count of cancellations. In the following section, if a runner has "picked up" a delivery and there was no cancellation, this will be considered a successful delivery.

#### 3.1 Code solution:
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

#### 3.2 Steps:
- Join the `customer_orders` table to `runner_orders` table
- Use `WHERE` on `cancellation` column to filter out any row that says 'cancellation'
- Use `GROUP BY` to agregate the data by runners

#### 4. How many of each type of pizza was delivered?

#### 4.1 Code solution:
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
#### 4.2 Steps:
- Left join `pizza_orders` and `pizza_name` by `order_id`
- COUNT number of vegetarian pizza orders and COUNT number of meatlover pizza orders and GROUP BY customer_id


#### 5. How many Vegetarian and Meatlovers were ordered by each customer?

#### 5.1 Code solution:

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


#### 6. What was the maximum number of pizzas delivered in a single order?
Count number of pizza's deliverd by order_id and filter out cancellations

#### 6.1 Code solution:
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


#### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

#### 7.2 steps
- COUNT number of blanks `''` for exclusions and extras to determine how many pizza did not have any changes
- COUNT number of non-blanks for exclusions and extras to detminer how many pizzas did have a change
- Use GROUP BY to arrange customer_ids into groups

#### 7.1 Code solution:
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

#### 8. How many pizzas were delivered that had both exclusions and extras?

8.1 Code solution:
```
SELECT
    COUNT(CASE WHEN co.exclusions <> '' AND co.extras <> '' THEN 1 END) AS original_pizza
FROM customer_orders co
LEFT JOIN runner_orders ro
ON co.order_id = ro.order_id
WHERE ro.cancellation NOT LIKE '%cancel'
```

#### 9. What was the total volume of pizzas ordered for each hour of the day?
COUNT number of pizza's ordered per hour per day. After conducting research, 

##### 9.1 Code solution:

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


#### 10. What was the volume of orders for each day of the week?

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
    CASE WHEN pickup_time IS NULL or lower(pickup_time) = 'null' THEN '' ELSE pickup_time END AS pickup_time,
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
