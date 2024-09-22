# Case study #1: Danny's Diner

## Problem statement
Danny seriously loves Japanese food. He's taken a bold move and embarked on a risky venture, opening up Danny's Diner. After a few sales, he has now collected sufficient data about his sales and customers. Danny wants to use the data to understand customer visiting patterns, how much money they’ve spent, and which menu items are their favorite.

## Case study questions 

### 1. What is the total amount each customer spent at the restaurant?

```
SELECT s.customer_id, sum(m.price) AS total_spent
FROM dannys_diner.sales s
LEFT JOIN dannys_diner.menu m
ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id
```
### Insights
+ Customer A spent $75.
+ Customer B spent $74.
+ Customer C spent $36.

### 2. How many days has each customer visited the restaurant?

```
SELECT customer_id, COUNT(DISTINCT order_date) AS days_visited
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY customer_id ASC;
```

### Insights:
+ Customer A visited on 4 different days.
+ Customer B visited on 6 different days.
+ Customer C visited on 2 different days.

Note: Based on the questions criteria, the number of visits is determined by the number of days a customer visited, not by the total number of visits within those days.

## 3. What was the first item from the menu purchased by each customer?

```
SELECT *
FROM (SELECT s.*, m.product_name, RANK() OVER (PARTITION BY customer_id ORDER BY order_date ASC) AS purchase_order_rank,
ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date ASC) as RN
FROM dannys_diner.sales s
LEFT JOIN dannys_diner.menu m
ON s.product_id = m.product_id) AS sales
WHERE purchase_order_rank = 1 OR rn = 1
ORDER BY customer_id;
```

### Insights
+ Customer A bought curry and sushi.
+ Customer B bought curry.
+ Customer C bought ramen (x2).

## 4. What is the most purchased item on the menu and how many times was it purchased by all customers? 

```
WITH orders AS (SELECT s.product_id, m.product_name, count(s.order_date) as times_ordered
FROM dannys_diner.sales s
LEFT JOIN dannys_diner.menu m
ON s.product_id = m.product_id
GROUP BY s.product_id, m.product_name)

SELECT *
FROM orders
WHERE times_ordered = 
(SELECT max(times_ordered)
FROM orders)
```

### Insights
+ Ramen has been purchased 8 times

## 5. Which item was the most popular for each customer?

```
SELECT *
FROM (SELECT *, RANK() OVER (PARTITION BY customer_id ORDER BY orders DESC) as most_ordered_item
FROM (SELECT s.customer_id, m.product_name, COUNT(s.order_date) as orders
FROM dannys_diner.sales s
LEFT JOIN dannys_diner.menu m
ON s.product_id = m.product_id
GROUP BY s.customer_id, m.product_name
ORDER BY s.customer_id ASC) as orders_table) as ranked_tabled
WHERE most_ordered_item = 1
```

### Insights
+ Customer A purchased ramen the most frequently.
+ Customer B purchased sushi, curry, and ramen an equal number of times, which was the most frequent choice for each.
+ Customer C purchased ramen the most frequently.

Note: When a customer had ordered a menu item the same number of times, each menu item was given equal weighting.

## 6. Which item was purchased first by the customer after they became a member?

```
SELECT *
FROM (SELECT s.*, menu.product_name, m.join_date,
RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date DESC) as order_of_purchases
FROM dannys_diner.sales s
INNER JOIN dannys_diner.members m
ON s.customer_id = m.customer_id
INNER JOIN dannys_diner.menu menu
ON s.product_id = menu.product_id
WHERE order_date > join_date) as ranked_table
WHERE order_of_purchases = 1;
```

### Insights
+ Customer A’s first purchase as a member was ramen (x2).
+ Customer B's first purchase as a member was ramen.

## 7. Which item was purchased just before the customer became a member?

```
SELECT *
FROM (SELECT s.*, menu.product_name, m.join_date,
RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date DESC) as order_of_purchases
FROM dannys_diner.sales s
INNER JOIN dannys_diner.members m
ON s.customer_id = m.customer_id
INNER JOIN dannys_diner.menu menu
ON s.product_id = menu.product_id
WHERE order_date < join_date) as ranked_table
WHERE order_of_purchases = 1;
```

### Insights
+ Customer A's last purchase before becoming a member was sushi and curry.
+ Customer B's last purchase before becoming a member was sushi.

## 8. What is the total items and amount spent for each member before they became a member?

```
SELECT s.customer_id, count(s.order_date) as count_of_purchases, SUM(price) AS total_spend
FROM dannys_diner.sales s
INNER JOIN dannys_diner.members membs
ON s.customer_id = membs.customer_id
LEFT JOIN dannys_diner.menu m 
ON s.product_id = m.product_id
WHERE s.order_date < membs.join_date 
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

### Insights
+ Since becoming a member, Customer A has made 2 purchases totaling $25.
+ Since becoming a member, Customer B has made 3 purchases totaling $40.

## 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```
SELECT customer_id, sum(points_earned) as points_earned
FROM (
SELECT s.*, m.*,CASE WHEN m.product_name = 'sushi' THEN m.price * 10 * 2 ELSE m.price * 10 END AS points_earned
FROM dannys_diner.sales s
LEFT JOIN dannys_diner.menu m
ON s.product_id = m.product_id) as points_table
GROUP BY customer_id
ORDER BY customer_id
```

### Insights
+ Customer A has earned 860 points.
+ Customer B has earned 940 points.
+ Customer C has earned 360 points.

## 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```
WITH cte1 AS (
SELECT membs.customer_id, membs.promotion_start_date, membs.promotion_end_date, s.order_date, m.product_id, m.price, m.product_name
FROM (SELECT customer_id, join_date as promotion_start_date,
join_date + INTERVAL '7 days' AS promotion_end_date
FROM dannys_diner.members) AS membs
LEFT JOIN dannys_diner.sales s
ON membs.customer_id = s.customer_id
LEFT JOIN dannys_diner.menu m
ON s.product_id = m.product_id),

cte2 AS (SELECT *, CASE WHEN product_name = 'sushi' THEN price * 10 * 2 ELSE price * 10 END AS old_points, CASE WHEN order_date BETWEEN promotion_start_date and promotion_end_date THEN price * 10 * 2 WHEN product_name = 'sushi' THEN price * 10 * 2 ELSE price * 10 END AS new_points,CASE WHEN order_date BETWEEN promotion_start_date AND promotion_end_date THEN 1 ELSE 0 END as eligibility_for_bonus_points
FROM cte1)
    
SELECT customer_id, sum(new_points) 
FROM cte2
WHERE order_date < '2021-02-01'
GROUP BY customer_id
```

### Assumptions
+ Customers begin accruing points even before they officially join as members.
+ During the week they join as members, customers earn double points on all items from the menu.

### Insights
+ Customer A has earned 1,370 points 
+ Customer B has earned 940 points

## Casey study bonus questions

## 1. Recreate the following table:
```
SELECT s.customer_id, s.order_date as order_date, m.product_name, m.price,
CASE WHEN membs.join_date IS NULL then 'N' WHEN s.order_date < membs.join_date THEN 'N' ELSE 'Y' END AS member FROM dannys_diner.sales s
LEFT JOIN dannys_diner.menu m
ON s.product_id = m.product_id
LEFT JOIN dannys_diner.members membs
on s.customer_id = membs.customer_id
ORDER BY s.customer_id, s.order_date, m.price DESC
```

## 2. Rank the following table
```
WITH cte1 as (
SELECT s.customer_id, s.order_date as order_date, m.product_name, m.price,
CASE WHEN membs.join_date IS NULL then 'N' WHEN s.order_date < membs.join_date THEN 'N' ELSE 'Y' END AS member FROM dannys_diner.sales s
LEFT JOIN dannys_diner.menu m
ON s.product_id = m.product_id
LEFT JOIN dannys_diner.members membs
on s.customer_id = membs.customer_id
ORDER BY s.customer_id, s.order_date, m.price DESC)

SELECT *, CASE WHEN member = 'Y' THEN ROW_NUMBER() OVER (PARTITION BY customer_id, member ORDER BY order_date ASC) ELSE NULL END AS ranking
FROM cte1
```

### Summary of insights:

+ Ramen has been ordered 8 times and is the most popular dish on the menu.
+ Customer B is the most frequent visitor.
+ After becoming members, customers often choose ramen as their first purchase.
+ Customer A has spent $75, customer B spent $74, customer C has spent $36.




