# Case Study #1 - Danny's Diner

> [Markdown style guide ref](https://www.markdownguide.org/basic-syntax/#:~:text=To%20bold%20text%2C%20add%20two,without%20spaces%20around%20the%20letters.&text=I%20just%20love%20**bold%20text**)

> [SQL formatter](https://www.freeformatter.com/sql-formatter.html#ad-output)

## Problem Statement: [Danny's Dinner](https://8weeksqlchallenge.com/case-study-1/)
> [Schema file]()

### 1. What is the total amount *each* customer spent at the restaurant?

```sql
SELECT
   m.customer_id,
   SUM(me.[price]) AS amt_spent 
FROM
   [dbo].[members] m 
   LEFT OUTER JOIN
      [dbo].[sales] s 
      ON s.[customer_id] = m.[customer_id] 
   LEFT OUTER JOIN
      [dbo].[menu] me 
      ON me.[product_id] = s.[product_id] 
GROUP BY
   m.customer_id
```

### 2. How many **days** has each customer visited the restaurant?

```sql
SELECT
   customer_id,
   COUNT(DISTINCT([order_date])) AS total_visits 
FROM
   [dbo].[sales] 
GROUP BY
   customer_id
```

### 3. What was the ***first item*** from the menu purchased by each customer?

```sql
WITH order_rank
     AS (SELECT m.[product_name],
                s.[customer_id],
                S.[order_date],
                Rank()
                  OVER (
                    partition BY s.[customer_id]
                    ORDER BY S.[order_date] ASC) AS date_rank
         FROM   [dbo].[menu] m
                LEFT JOIN [dbo].[sales] s
                       ON m.[product_id] = s.[product_id])
SELECT *
FROM   order_rank
WHERE  date_rank = 1
```

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
SELECT
   top 1 m.[product_name],
   COUNT(s.[product_id]) AS product_cnt 
FROM
   [dbo].[sales] s 
   LEFT JOIN
      [dbo].[menu] m 
      ON m.[product_id] = s.[product_id] 
GROUP BY
   m.[product_name] 
ORDER BY
   COUNT(s.[product_id]) DESC
```

### 5. Which item was the most popular for each customer?

```sql
WITH popular AS 
(
   SELECT
      s.[customer_id],
      m.[product_name],
      DENSE_RANK() OVER (PARTITION BY s.[customer_id] 
   ORDER BY
      COUNT(s.[product_id]) DESC) AS rnk 
   FROM
      [dbo].[sales] s 
      LEFT JOIN
         [dbo].[menu] m 
         ON m.[product_id] = s.[product_id] 
   GROUP BY
      s.[customer_id],
      m.[product_name] 
)
SELECT
   [customer_id],
   [product_name] 
FROM
   popular 
WHERE
   rnk = 1
```

### 6. Which item was purchased first by the customer after they became a member?

```sql
WITH first_purchase AS 
(
   SELECT
      m.[customer_id],
      me.[product_name],
      s.[order_date],
      m.[join_date],
      RANK() OVER (PARTITION BY m.[customer_id] 
   ORDER BY
      s.[order_date] ASC ) AS first_pur 
   FROM
      [dbo].[sales] s 
      LEFT JOIN
         [dbo].[members] m 
         ON s.[customer_id] = m.[customer_id] 
      LEFT JOIN
         [dbo].[menu] me 
         ON me.[product_id] = s.[product_id] 
   WHERE
      s.[order_date] >= m.[join_date] 
)
SELECT
   [customer_id],
   [product_name],
   [order_date],
   [join_date] 
FROM
   first_purchase 
WHERE
   first_pur = 1
```

### 7. Which item was purchased just before the customer became a member?

```sql
WITH first_purchase AS 
(
   SELECT
      m.[customer_id],
      me.[product_name],
      s.[order_date],
      m.[join_date],
      DENSE_RANK() OVER (PARTITION BY m.[customer_id] 
   ORDER BY
      s.[order_date] DESC ) AS first_pur 
   FROM
      [dbo].[sales] s 
      LEFT JOIN
         [dbo].[members] m 
         ON s.[customer_id] = m.[customer_id] 
      LEFT JOIN
         [dbo].[menu] me 
         ON me.[product_id] = s.[product_id] 
   WHERE
      s.[order_date] < m.[join_date] 
)
SELECT
   [customer_id],
   [product_name],
   [order_date],
   [join_date] 
FROM
   first_purchase 
WHERE
   first_pur = 1
```

### 8. What is the total items and amount spent for each member before they became a member?

```sql
SELECT
   m.[customer_id],
   COUNT(DISTINCT(me.[product_name])) AS total_items,
   SUM(me.[price]) AS amt_spent 
FROM
   [dbo].[sales] s 
   LEFT JOIN
      [dbo].[members] m 
      ON s.[customer_id] = m.[customer_id] 
   LEFT JOIN
      [dbo].[menu] me 
      ON me.[product_id] = s.[product_id] 
WHERE
   s.[order_date] < m.[join_date] 
GROUP BY
   m.[customer_id]
```

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql

```

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```sql

```
