# Case Study 2-Pizza Runner

## Problem Statement: [Pizza Runner](https://8weeksqlchallenge.com/case-study-2/)

> [Schema file](SQLSchema/CaseStudy_2_PizzaRunner.sql)

> [SQL playground](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)

> [ER Image](images/CaseStudy2_ERDiagram.png)

### *Status: Completed*

## A. Pizza Metrics

### 1. How many pizzas were ordered?

```sql
SELECT
   COUNT([order_id]) AS number_of_pizzas 
FROM
   [dbo].[customer_orders]
```

How many unique customer orders were made?

```sql
SELECT
   COUNT(DISTINCT([order_id])) AS unique_cust_ord 
FROM
   [dbo].[customer_orders]
```

How many successful orders were delivered by each runner?

```sql
SELECT
   runner_id,
   COUNT(order_id) AS orders_delivered 
FROM
   [dbo].[runner_orders] 
WHERE
   duration NOT IN 
   (
      'null'
   )
GROUP BY
   runner_id
```

How many of each type of pizza was delivered?

```sql
ALTER TABLE [dbo].[pizza_names] ALTER COLUMN [pizza_name] NVARCHAR(50) 
SELECT
   p.[pizza_name],
   COUNT(o.order_id) AS ordered 
FROM
   [dbo].[pizza_names] p 
   LEFT JOIN
      [dbo].[customer_orders] o 
      ON p.[pizza_id] = o.[pizza_id] 
GROUP BY
   p.[pizza_name]
```

How many Vegetarian and Meat lovers were ordered by each customer?

```sql
SELECT
   p.[pizza_name],
   o.[customer_id],
   COUNT(o.order_id) AS ordered 
FROM
   [dbo].[pizza_names] p 
   LEFT JOIN
      [dbo].[customer_orders] o 
      ON p.[pizza_id] = o.[pizza_id] 
GROUP BY
   p.[pizza_name],
   o.[customer_id]
```

What was the maximum number of pizzas delivered in a single order?

```sql
WITH pizza_highest AS 
(
   SELECT
      order_id,
      COUNT(pizza_id) AS pizze_cnt 
   FROM
      [dbo].[customer_orders] 
   GROUP BY
      order_id 
)
SELECT
   top 1 MAX(pizze_cnt) AS highest_pizza_cnt,
   order_id 
FROM
   pizza_highest 
GROUP BY
   order_id 
ORDER BY
   1 DESC
```

For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

```sql
SELECT
   customer_id,
   SUM(
   CASE
      WHEN
         exclusions <> '' 
         OR extras <> '' 
      THEN
         1 
      ELSE
         0 
   END
) AS atleast_1_change, SUM(
   CASE
      WHEN
         exclusions = '' 
         AND 
         (
            extras = '' 
            OR extras IS NULL
         )
      THEN
         1 
      ELSE
         0 
   END
) AS no_changes 
FROM
   [dbo].[customer_orders] o 
   INNER JOIN
      [dbo].[runner_orders] r 
      ON r.[order_id] = o.order_id 
WHERE
   r.[duration] NOT IN 
   (
      'null'
   )
GROUP BY
   customer_id
```

How many pizzas were delivered that had both exclusions and extras?

```sql
SELECT
   COUNT(*) AS pizza_exclu_extra 
FROM
   [dbo].[customer_orders] o 
   INNER JOIN
      [dbo].[runner_orders] r 
      ON r.[order_id] = o.order_id 
WHERE
   r.[duration] NOT IN 
   (
      'null'
   )
   AND exclusions <> '' 
   AND extras <> '' 
   AND exclusions NOT IN 
   (
      'null'
   )
   AND extras NOT IN 
   (
      'null'
   )
```

What was the total volume of pizzas ordered for each hour of the day?

```sql
SELECT
   DATEPART(HOUR, [order_time]) AS hour_of_day,
   COUNT(*) AS pizzas_ordered 
FROM
   [dbo].[customer_orders] 
GROUP BY
   [order_time]
```

What was the volume of orders for each day of the week?

```sql
SELECT
   DATENAME(WEEKDAY, [order_time]) AS day_of_week,
   COUNT(*) AS pizzas_ordered 
FROM
   [dbo].[customer_orders] 
GROUP BY
   [order_time]
```

## B. Runner and Customer Experience

How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

```sql
SELECT
   DATE_PART('WEEK', registration_date) AS REGISTRATION_WEEK,
   COUNT(runner_id) AS number_of_registrations 
FROM
   pizza_runner.runners 
GROUP BY
   1
```

What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

```sql
WITH runner_time AS 
(
   SELECT
      ro.runner_id,
      EXTRACT(EPOCH 
   FROM
      (
         ro.pickup_time::TIMESTAMP - co.order_time
      )
) / 60 AS pickup_minutes 
   FROM
      runner_orders ro 
      LEFT JOIN
         customer_orders co 
         ON ro.order_id = co.order_id 
   WHERE
      pickup_time NOT IN
      (
         'null'
      )
)
SELECT
   AVG(pickup_minutes) AS avg_pickup 
FROM
   runner_time
```

Is there any relationship between the number of pizzas and how long the order takes to prepare?

```sql
WITH prep_time AS 
(
   SELECT
      co.order_id,
      COUNT(pizza_id) AS pizzas_ordered,
      EXTRACT(EPOCH 
   FROM
      (
         ro.pickup_time::TIMESTAMP - co.order_time
      )
) / 60 AS pickup_minutes 
   FROM
      runner_orders ro 
      LEFT JOIN
         customer_orders co 
         ON ro.order_id = co.order_id 
   WHERE
      pickup_time NOT IN
      (
         'null'
      )
   GROUP BY
      1,
      3 
)
SELECT
   pizzas_ordered,
   AVG(pickup_minutes) 
FROM
   prep_time 
GROUP BY
   1 
ORDER BY
   pizzas_ordered DESC
```

What was the average distance traveled for each customer?

```sql
WITH distance_conversion AS 
(
   SELECT
      co.customer_id,
      co.order_id,
      (
(regexp_matches(ro.distance, '[0-9]+\.?[0-9]*'))[1]::NUMERIC
      )
      AS dis_travelled 
   FROM
      pizza_runner.customer_orders co 
      LEFT JOIN
         pizza_runner.runner_orders ro 
         ON co.order_id = ro.order_id 
   WHERE
      ro.distance != 'null' 
   GROUP BY
      co.customer_id,
      ro.distance,
      co.order_id 
)
SELECT
   customer_id,
   round(AVG(dis_travelled), 2) AS avg_distance 
FROM
   distance_conversion 
GROUP BY
   customer_id
```

What was the difference between the longest and shortest delivery times for all orders?

```sql
with duration_con as (
  select order_id,((regexp_matches(duration, '[0-9]+\.?[0-9]*'))[1]::NUMERIC) as delivery_time
from
pizza_runner.runner_orders
  )
  select max(delivery_time)-min(delivery_time) as delivery_diff
  from 
  duration_con
```

What was the average speed for each runner for each delivery and do you notice any trend for these values?

```sql
WITH prep_time AS 
(
   SELECT
      co.order_id,
      co.customer_id,
      ro.runner_id,
      EXTRACT(EPOCH 
   FROM
      (
         ro.pickup_time::TIMESTAMP - co.order_time
      )
) / 60 AS pickup_minutes 
   FROM
      runner_orders ro 
      LEFT JOIN
         customer_orders co 
         ON ro.order_id = co.order_id 
   WHERE
      pickup_time NOT IN
      (
         'null'
      )
   GROUP BY
      1,
      2,
      3,
      4 
)
SELECT
   * 
FROM
   prep_time 
ORDER BY
   customer_id
```

What is the successful delivery percentage for each runner?

```sql
SELECT
   runner_id,
   ROUND(100 * SUM( 
   CASE
      WHEN
         distance = 'null' 
      THEN
         0 
      ELSE
         1 
   END
) / COUNT(*), 0) AS success_perc 
FROM
   runner_orders 
GROUP BY
   runner_id 
ORDER BY
   runner_id
```

C. Ingredient Optimization

What are the standard ingredients for each pizza?

```sql
WITH ingredient AS 
(
   SELECT
(REGEXP_SPLIT_TO_TABLE(toppings, '[,\s]+')::INTEGER) AS toppings_id,
      COUNT(*) AS pizza_count 
   FROM
      pizza_runner.pizza_recipes 
   GROUP BY
      1 
   ORDER BY
      2 DESC 
)
SELECT
   pt.topping_name,
   count_toppings 
FROM
   ingredient i 
   LEFT JOIN
      pizza_runner.pizza_toppings pt 
      ON pt.topping_id = i.toppings_id 
ORDER BY
   count_toppings DESC
```

What was the most commonly added extra?

```sql
WITH ingredient AS 
(
   SELECT
(REGEXP_SPLIT_TO_TABLE(toppings, '[,\s]+')::INTEGER) AS toppings_id,
      COUNT(*) AS count_toppings 
   FROM
      pizza_runner.pizza_recipes 
   GROUP BY
      1 
   ORDER BY
      2 DESC 
)
SELECT
   pt.topping_name 
FROM
   ingredient i 
   LEFT JOIN
      pizza_runner.pizza_toppings pt 
      ON pt.topping_id = i.toppings_id 
ORDER BY
   count_toppings DESC
```

What was the most common exclusion?

```sql
SELECT
   exclusions,
   COUNT(*) AS exclusion_occurrences 
FROM
   pizza_runner.customer_orders co 
WHERE
   exclusions != 'null' 
GROUP BY
   exclusions 
ORDER BY
   2 DESC LIMIT 1
```

Generate an order item for each record in the customers_orders table in the format of one of the following:

* Meat Lovers
* Meat Lovers - Exclude Beef
* Meat Lovers - Extra Bacon
* Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

```sql
select order_id, customer_id, pizza_id, 
case when 
pt.topping_name!='Bacon' and pt.topping_name!='Cheese' and co.
case when 
from customer_orders co
left join pizza_name pn on co.pizza_id=pn.pizza_id
left join pizza_recipes pr on pn.pizza_id=pr.pizza_id
left join pizza_toppings pt on pr.toppings=pt.topping_id
where pn.pizza_name ='MeatLovers'
*/
```

Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"

```sql

```

What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

```sql
SELECT
   pt.topping_name,
   (
      REGEXP_SPLIT_TO_TABLE(toppings, '[,\s]+')::INTEGER
   )
   AS toppings_id,
   COUNT(pt.topping_id) AS topping_cnt 
FROM
   customer_orders co 
   LEFT JOIN
      runner_orders ro 
      ON co.order_id = ro.order_id 
   LEFT JOIN
      pizza_names pn 
      ON co.pizza_id = pn.pizza_id 
   LEFT JOIN
      pizza_recipes pr 
      ON pn.pizza_id = pr.pizza_id 
   LEFT JOIN
      pizza_toppings pt 
      ON pr.toppings = pt.topping_name 
WHERE
   ro.distance NOT IN 
   (
      'null'
   )
GROUP BY
   1,
   2
```

D. Pricing and Ratings

If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?

```sql

```

What if there was an additional $1 charge for any pizza extras?
Add cheese is $1 extra

```sql

```

The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.

```sql

```

Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
customer_id
order_id
runner_id
rating
order_time
pickup_time
Time between order and pickup
Delivery duration
Average speed
Total number of pizzas

```sql

```

If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometer traveled - how much money does Pizza Runner have left over after these deliveries?

```sql

```

E. Bonus Questions

If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?

```sql

```
