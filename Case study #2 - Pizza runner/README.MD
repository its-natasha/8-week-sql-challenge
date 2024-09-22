# Case study #2 Pizza Runner
# A.Pizza Metrics

#### Cleaning up customer_order table:

#### Code
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

### Steps taken: 
- Replace IS NULL or NULL values as an empty string 
- Use `strip()` to remove white space
- Use `lower()` to update all characters to consistent lower case 

```

#### Cleaning up runner_orders table:

### Code

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

#### Code:
```
SELECT COUNT (DISTINCT CUSTOMER_ID)
FROM customer_orders
```

#### Steps taken:
- Use `DISTINCT` to extract unique values
- Use `COUNT` to count number of values in a row


### How many successful orders were delivered by each runner?


#### How many of each type of pizza was delivered?
#### How many Vegetarian and Meatlovers were ordered by each customer?
#### What was the maximum number of pizzas delivered in a single order?
#### For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
#### How many pizzas were delivered that had both exclusions and extras?
#### What was the total volume of pizzas ordered for each hour of the day?
#### What was the volume of orders for each day of the week?
