# Case Study #1 - Danny's Diner

> [Markdown style guide ref](https://www.markdownguide.org/basic-syntax/#:~:text=To%20bold%20text%2C%20add%20two,without%20spaces%20around%20the%20letters.&text=I%20just%20love%20**bold%20text**)
 
## Problem Statement: [Danny's Dinner](https://8weeksqlchallenge.com/case-study-1/)

### 1. What is the total amount *each* customer spent at the restaurant?

```sql
SELECT M.CUSTOMER_ID, SUM(ME.[price]) AS AMT_SPENT
FROM [dbo].[members] M
LEFT OUTER JOIN [dbo].[sales] S ON S.[customer_id]=M.[customer_id]
LEFT OUTER JOIN [dbo].[menu] ME ON ME.[product_id]=S.[product_id]
GROUP BY M.CUSTOMER_ID
```

### 2. How many **days** has each customer visited the restaurant?

```sql
SELECT CUSTOMER_ID, COUNT(DISTINCT([order_date])) AS TOTAL_VISITS
FROM [dbo].[sales]
GROUP BY CUSTOMER_ID
```

### 3. What was the ***first item*** from the menu purchased by each customer?

```sql


```

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql

```

### 5. Which item was the most popular for each customer?

```sql

```

### 6. Which item was purchased first by the customer after they became a member?

```sql

```

### 7. Which item was purchased just before the customer became a member?

```sql

```

### 8. What is the total items and amount spent for each member before they became a member?

```sql

```

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql

```

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```sql

```
