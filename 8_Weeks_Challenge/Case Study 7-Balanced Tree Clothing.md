# Case Study 7 - Balanced Tree Clothing

## Problem Statement: [Balanced Tree Clothing](https://8weeksqlchallenge.com/case-study-7/)

[Schema file](SQLSchema/CaseStudy_7_BalancedTreeClothing.sql)

[SQL Playground](https://www.db-fiddle.com/f/dkhULDEjGib3K58MvDjYJr/8)

### PostgreSQL used for data analysis

The following questions can be considered key business questions and metrics that the Balanced Tree team requires for their monthly reports.

Each question can be answered using a single query - but as you are writing the SQL to solve each individual problem, keep in mind how you would generate all of these metrics in a single SQL script which the Balanced Tree team can run each month.

## High Level Sales Analysis

### 1. What was the total quantity sold for all products?

````sql
SELECT
   SUM(qty) AS total_sold 
FROM
   balanced_tree.sales
````

### 2. What is the total generated revenue for all products before discounts?

```sql
SELECT
   SUM(qty*price) AS revenue_beforediscount 
FROM
   balanced_tree.sales
```

### 3. What was the total discount amount for all products?

```sql
SELECT
   SUM(discount) AS total_discount 
FROM
   balanced_tree.sales
```

## Transaction Analysis

### 1. How many unique transactions were there?

```sql
SELECT
   COUNT(DISTINCT(txn_id)) AS distinct_transactuons 
FROM
   balanced_tree.sales
```

### 2. What is the average unique products purchased in each transaction?

```sql
WITH purchases AS 
(
   SELECT
      txn_id,
      COUNT(DISTINCT prod_id) AS number_of_products 
   FROM
      balanced_tree.sales 
   GROUP BY
      txn_id 
)
SELECT
   ROUND(AVG(number_of_products),0) AS avg_products_purchased 
FROM
   purchases
```

### 3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?

```sql
WITH rev AS 
(
   SELECT
      txn_id,
      (
         qty*price
      )
       - discount AS revenue 
   FROM
      balanced_tree.sales 
   GROUP BY
      1,
      2 
)
SELECT
   txn_id,
   PERCENTILE_CONT(0.25) WITHIN GROUP (
ORDER BY
   revenue ASC) AS per_25,
   PERCENTILE_CONT(0.5) WITHIN GROUP (
ORDER BY
   revenue ASC) AS per_05,
   PERCENTILE_CONT(0.75) WITHIN GROUP (
ORDER BY
   revenue ASC) AS per_75 
FROM
   rev 
GROUP BY
   txn_id
```

### 4. What is the average discount value per transaction?

```sql
SELECT
   txn_id,
   round(AVG(discount), 0) AS avg_discount 
FROM
   balanced_tree.sales 
GROUP BY
   1
```

### 5. What is the percentage split of all transactions for members vs non-members?

```sql
SELECT
   MEMBER,
   COUNT(DISTINCT txn_id) AS total_transactions,
   100 * COUNT(DISTINCT txn_id) / (
   SELECT
      COUNT(DISTINCT txn_id) 
   FROM
      balanced_tree.sales) AS percentage 
   FROM
      balanced_tree.sales 
   GROUP BY
      MEMBER
```

### 6. What is the average revenue for member transactions and non-member transactions?

```sql
SELECT
   MEMBER,
   AVG((qty*price) - discount) AS avg_revenue 
FROM
   balanced_tree.sales 
GROUP BY
   MEMBER
```

## Product Analysis

### 1. What are the top 3 products by total revenue before discount?

```sql
WITH reven AS 
(
   SELECT
      p.product_name,
      SUM(s.qty*s.price) AS revenue_prior_discount 
   FROM
      balanced_tree.sales s 
      INNER JOIN
         balanced_tree.product_details p 
         ON s.prod_id = p.product_id 
   GROUP BY
      p.product_name 
   ORDER BY
      SUM(s.qty*s.price) DESC 
)
SELECT
   * 
FROM
   reven LIMIT 3
```

### 2. What is the total quantity, revenue and discount for each segment?

```sql
SELECT
   p.segment_name,
   SUM(s.qty) AS quantity,
   SUM((s.qty*s.price) - s.discount) AS revenue,
   SUM(s.discount) AS discount 
FROM
   balanced_tree.sales s 
   INNER JOIN
      balanced_tree.product_details p 
      ON s.prod_id = p.product_id 
GROUP BY
   p.segment_name
```

### 3. What is the top selling product for each segment?

```sql
WITH cte AS 
(
   SELECT
      p.segment_name,
      p.product_name,
      SUM(s.qty) AS quantity,
      SUM((s.qty*s.price) - s.discount) AS revenue,
      SUM(s.discount) AS discount 
   FROM
      balanced_tree.sales s 
      INNER JOIN
         balanced_tree.product_details p 
         ON s.prod_id = p.product_id 
   GROUP BY
      p.segment_name,
      p.product_name 
)
,
product_partition AS 
(
   SELECT
      segment_name,
      product_name,
      DENSE_RANK() OVER (PARTITION BY segment_name 
   ORDER BY
      revenue DESC) AS rnk 
   FROM
      cte 
)
SELECT
   * 
FROM
   product_partition 
WHERE
   rnk = 1 
ORDER BY
   segment_name,
   product_name
```

### 4. What is the total quantity, revenue and discount for each category?

```sql
SELECT
   p.category_name,
   SUM(s.qty) AS quantity,
   SUM((s.qty*s.price) - s.discount) AS revenue,
   SUM(s.discount) AS discount 
FROM
   balanced_tree.sales s 
   INNER JOIN
      balanced_tree.product_details p 
      ON s.prod_id = p.product_id 
GROUP BY
   p.category_name
```

### 5. What is the top selling product for each category?

```sql
WITH cte AS 
(
   SELECT
      p.category_name,
      p.product_name,
      SUM(s.qty) AS quantity,
      SUM((s.qty*s.price) - s.discount) AS revenue,
      SUM(s.discount) AS discount 
   FROM
      balanced_tree.sales s 
      INNER JOIN
         balanced_tree.product_details p 
         ON s.prod_id = p.product_id 
   GROUP BY
      p.category_name,
      p.product_name 
)
,
product_partition AS 
(
   SELECT
      category_name,
      product_name,
      DENSE_RANK() OVER (PARTITION BY category_name 
   ORDER BY
      revenue DESC) AS rnk 
   FROM
      cte 
)
SELECT
   category_name,
   product_name 
FROM
   product_partition 
WHERE
   rnk = 1 
ORDER BY
   category_name,
   product_name
```

### 6. What is the percentage split of revenue by product for each segment?

```sql
WITH product_sales AS 
(
   SELECT
      prod.segment_name,
      prod.product_name,
      SUM((sal.qty*sal.price) - sal.discount) AS total_rev_product 
   FROM
      balanced_tree.sales sal 
      LEFT JOIN
         balanced_tree.product_details prod 
         ON sal.prod_id = prod.product_id 
   GROUP BY
      prod.segment_name,
      prod.product_name 
)
SELECT
   segment_name,
   product_name,
   round(100 * (total_rev_product / SUM(total_rev_product) OVER (PARTITION BY segment_name)), 2) AS rev_percent_seg 
FROM
   product_sales
```

### 7. What is the percentage split of revenue by segment for each category?

```sql
WITH segment_sales AS 
(
   SELECT
      prod.segment_name,
      prod.category_name,
      SUM((sal.qty*sal.price) - sal.discount) AS total_rev_segment 
   FROM
      balanced_tree.sales sal 
      LEFT JOIN
         balanced_tree.product_details prod 
         ON sal.prod_id = prod.product_id 
   GROUP BY
      prod.segment_name,
      prod.category_name 
)
SELECT
   category_name,
   segment_name,
   round(100 * (total_rev_segment / SUM(total_rev_segment) OVER (PARTITION BY category_name)), 2) AS rev_percent_cat 
FROM
   segment_sales
```

### 8. What is the percentage split of total revenue by category?

```sql
SELECT
   prod.category_name,
   round(100 *( SUM((sal.qty*sal.price) - sal.discount) / SUM(SUM((sal.qty*sal.price) - sal.discount)) OVER () ), 2) AS perct 
FROM
   balanced_tree.sales sal 
   LEFT JOIN
      balanced_tree.product_details prod 
      ON sal.prod_id = prod.product_id 
GROUP BY
   1
```

### 9. What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)

```sql
SELECT
   prod.product_name,
   COUNT(DISTINCT sal.txn_id) AS transac_cnt,
   round(100*( COUNT(DISTINCT sal.txn_id)::NUMERIC / (
   SELECT
      COUNT(DISTINCT txn_id) 
   FROM
      sales)), 2) AS percentage 
   FROM
      sales sal 
      LEFT JOIN
         product_details prod 
         ON sal.prod_id = prod.product_id 
   GROUP BY
      prod.product_name
```

### 10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?

```sql
WITH three_prod AS 
(
   SELECT
      sal.txn_id 
   FROM
      sales sal 
   GROUP BY
      sal.txn_id 
   HAVING
      COUNT(sal.prod_id) >= 3 
)
,
prod_aggr AS 
(
   SELECT
      string_agg(prod.product_name, ',' 
   ORDER BY
      product_name) AS prod_combo 
   FROM
      sales sal 
      LEFT JOIN
         product_details prod 
         ON sal.prod_id = prod.product_id 
   WHERE
      txn_id IN 
      (
         SELECT
            * 
         FROM
            three_prod
      )
   GROUP BY
      txn_id 
)
,
ranking AS 
(
   SELECT
      prod_combo,
      COUNT(*),
      DENSE_RANK() OVER (
   ORDER BY
      COUNT(prod_combo) DESC) AS rnk 
   FROM
      prod_aggr 
   GROUP BY
      prod_combo 
)
SELECT
   * 
FROM
   ranking 
WHERE
   rnk = 1

```

## Reporting Challenge

Write a single SQL script that combines all of the previous questions into a scheduled report that the Balanced Tree team can run at the beginning of each month to calculate the previous month’s values.

Imagine that the Chief Financial Officer (which is also Danny) has asked for all of these questions at the end of every month.

He first wants you to generate the data for January only - but then he also wants you to demonstrate that you can easily run the same analysis for February without many changes (if at all).

Feel free to split up your final outputs into as many tables as you need - but be sure to explicitly reference which table outputs relate to which question for full marks :)

## Bonus Challenge

Use a single SQL query to transform the *product_hierarchy* and *product_prices* datasets to the product_details table.

Hint: you may want to consider using a recursive CTE to solve this problem!

```sql
with recursive prod_details as (
  select id, parent_id, level_text, level_name
  from product_hierarchy 
  
  union all
  select ph.id, ph.parent_id+1, ph.level_text, ph.level_name
  from product_hierarchy ph
  inner join prod_details pd on pd.parent_id=ph.id

 
  )
  ,prod_price as (
    select pd.*, pp.price
    from prod_details pd 
    left join product_prices pp on pd.id=pp.id
    )
select  max(case when (level_name='Category') then level_text else NULL end) as Category,
max(case when (level_name='Segment') then level_text else NULL end) as Segment,
max(case when (level_name='Style') then level_text else NULL end) as Style,
 price
from prod_price
group by price
order by 2
```
