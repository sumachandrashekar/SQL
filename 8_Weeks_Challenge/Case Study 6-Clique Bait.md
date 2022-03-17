# Case Study 6-Clique Bait

## Problem Statement: [Clique Bait](https://8weeksqlchallenge.com/case-study-6/)

![Schema file](SQLSchema/CaseStudy_6_Clique_Bait.sql)

![ER Image]()

## 2. Digital Analysis

Using the available datasets - answer the following questions using a single query for each one:

### 1. How many users are there?

```sql

```

### 2. How many cookies does each user have on average?

```sql

```

### 3. What is the unique number of visits by all users per month?

```sql

```

### 4. What is the number of events for each event type?

```sql

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

### 1. Which product had the most views, cart adds and purchases?

```sql

```

### 2. Which product was most likely to be abandoned?

```sql

```

### 3. Which product had the highest view to purchase percentage?

```sql

```

### 4. What is the average conversion rate from view to cart add?

```sql

```

### 5. What is the average conversion rate from cart add to purchase?

```sql

```

## 3. Campaigns Analysis

Generate a table that has 1 single row for every unique ==visit_id== record and has the following columns:

* ==user_id==
* visit_id
* visit_start_time: the earliest event_time for each visit
* page_views: count of page views for each visit
* cart_adds: count of product cart add events for each visit
* purchase: 1/0 flag if a purchase event exists for each visit
* campaign_name: map the visit to a campaign if the visit_start_time falls between the start_date and end_date
* impression: count of ad impressions for each visit
* click: count of ad clicks for each visit
* ***(Optional column)*** cart_products: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number)

Use the subsequent dataset to generate at least 5 insights for the Clique Bait team - bonus: prepare a single A4 infographic that the team can use for their management reporting sessions, be sure to emphasise the most important points from your findings.

Some ideas you might want to investigate further include:

* Identifying users who have received impressions during each campaign period and comparing each metric with other users who did not have an impression event
* Does clicking on an impression lead to higher purchase rates?
* What is the uplift in purchase rate when comparing users who click on a campaign impression versus users who do not receive an impression? What if we compare them with users who just an impression but do not click?
* What metrics can you use to quantify the success or failure of each campaign compared to eachother?