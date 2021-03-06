# Case Study 6-Clique Bait

## Problem Statement: [Clique Bait](https://8weeksqlchallenge.com/case-study-6/)

![Schema file](SQLSchema/CaseStudy_6_Clique_Bait.sql)

![SQL playground](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/17)

### Status: Completed

## 2. Digital Analysis

Using the available datasets - answer the following questions using a single query for each one:

### 1. How many users are there?

```sql
SELECT
   COUNT(DISTINCT user_id) 
FROM
   clique_bait.users
```

### 2. How many cookies does each user have on average?

```sql
WITH cte AS 
(
   SELECT
      user_id,
      COUNT(cookie_id) AS nocookie 
   FROM
      clique_bait.users 
   GROUP BY
      user_id 
)
SELECT
   round(AVG(nocookie), 0) AS avg_cookies 
FROM
   cte
```

### 3. What is the unique number of visits by all users per month?

```sql
SELECT
   COUNT(DISTINCT visit_id) AS unique_visits,
   EXTRACT(MONTH 
FROM
   event_time) AS visti_month 
FROM
   clique_bait.events 
GROUP BY
   EXTRACT(MONTH 
FROM
   event_time) 
ORDER BY
   2 
```

### 4. What is the number of events for each event type?

```sql
SELECT
   event_type,
   COUNT(*) AS number_of_events 
FROM
   clique_bait.events 
GROUP BY
   event_type 
ORDER BY
   event_type
```

### 5. What is the percentage of visits which have a purchase event?

```sql

```

### 6. What is the percentage of visits which view the checkout page but do not have a purchase event?

```sql

```

### 7. What are the top 3 pages by number of views?

```sql

```

### 8. What is the number of views and cart adds for each product category?

```sql

```

### 9. What are the top 3 products by purchases?

```sql

```

## 3. Product Funnel Analysis

Using a single SQL query - create a new output table which has the following details:

* How many times was each product viewed?
* How many times was each product added to cart?
* How many times was each product added to a cart but not purchased (abandoned)?
* How many times was each product purchased?

Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.

Use your 2 new output tables - answer the following questions:

### First table code:

```sql

```

### Second table code: 

```sql

```

The above two tables can answer the questions asked below:
### 1. Which product had the most views, cart adds and purchases?
### 2. Which product was most likely to be abandoned?
### 3. Which product had the highest view to purchase percentage?
### 4. What is the average conversion rate from view to cart add?
### 5. What is the average conversion rate from cart add to purchase?

## 3. Campaigns Analysis

Generate a table that has 1 single row for every unique ==visit_id== record and has the following columns:

* user_id
* visit_id
* visit_start_time: the earliest event_time for each visit
* page_views: count of page views for each visit
* cart_adds: count of product cart add events for each visit
* purchase: 1/0 flag if a purchase event exists for each visit
* campaign_name: map the visit to a campaign if the visit_start_time falls between the start_date and end_date
* impression: count of ad impressions for each visit
* click: count of ad clicks for each visit
* ***(Optional column)*** cart_products: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number)

Use the subsequent dataset to generate at least 5 insights for the Clique Bait team - bonus: prepare a single A4 infographic that the team can use for their management reporting sessions, be sure to emphasize the most important points from your findings.

Some ideas you might want to investigate further include:

* Identifying users who have received impressions during each campaign period and comparing each metric with other users who did not have an impression event
* Does clicking on an impression lead to higher purchase rates?
* What is the uplift in purchase rate when comparing users who click on a campaign impression versus users who do not receive an impression? What if we compare them with users who just an impression but do not click?
* What metrics can you use to quantify the success or failure of each campaign compared to each other?