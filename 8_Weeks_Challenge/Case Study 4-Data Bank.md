# Case Study 4-Data Bank

## Problem Statement: [Data Bank](https://8weeksqlchallenge.com/case-study-4/)

> [Schema file](SQLSchema/CaseStudy_4_Data_Bank.sql)

![ER Image](images/CaseStudy4_ERDiagram.png)

## A. Customer Nodes Exploration

### 1. How many unique nodes are there on the Data Bank system?

```sql
SELECT
   COUNT(DISTINCT node_id) AS unique_nodes 
FROM
   data_bank.customer_nodes
```

### 2. What is the number of nodes per region?

```sql
SELECT
   r.region_name,
   COUNT(node_id) AS number_of_node 
FROM
   data_bank.customer_nodes n 
   LEFT JOIN
      data_bank.regions r 
      ON n.region_id = r.region_id 
GROUP BY
   r.region_name
```

### 3. How many customers are allocated to each region?

```sql
SELECT
   n.region_name,
   COUNT(cn.customer_id) AS customer_count 
FROM
   data_bank.customer_nodes cn 
   LEFT JOIN
      data_bank.regions n 
      ON cn.region_id = n.region_id 
GROUP BY
   n.region_name
```

### 4. How many days on average are customers reallocated to a different node?

```sql
WITH diff AS 
(
   SELECT
      cn.customer_id,
      cn.node_id,
      cn.start_date,
      cn.end_date,
      cn.end_date - cn.start_date AS date_diff 
   FROM
      data_bank.customer_nodes cn 
      LEFT JOIN
         data_bank.regions n 
         ON cn.region_id = n.region_id 
   WHERE
      cn.end_date != '9999-12-31' 
)
,
datesum AS 
(
   SELECT
      customer_id,
      node_id,
      SUM(date_diff) AS sum_date_diff 
   FROM
      diff 
   GROUP BY
      customer_id,
      node_id 
)
SELECT
   round(AVG(sum_date_diff)) AS avg_reallocation 
FROM
   datesum
```

### 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?

```sql
WITH diff AS 
(
   SELECT
      cn.customer_id,
      cn.node_id,
      n.region_name,
      cn.start_date,
      cn.end_date,
      cn.end_date - cn.start_date AS date_diff 
   FROM
      data_bank.customer_nodes cn 
      LEFT JOIN
         data_bank.regions n 
         ON cn.region_id = n.region_id 
   WHERE
      cn.end_date != '9999-12-31' 
)
,
datesum AS 
(
   SELECT
      customer_id,
      node_id,
      region_name,
      SUM(date_diff) AS sum_date_diff 
   FROM
      diff 
   GROUP BY
      customer_id,
      node_id,
      region_name 
)
SELECT
   region_name,
   PERCENTILE_CONT(0.5) WITHIN GROUP (
ORDER BY
   sum_date_diff) AS median_reallocation,
   PERCENTILE_CONT(0.8) WITHIN GROUP (
ORDER BY
   sum_date_diff) AS eightpercentile_reallocation,
   PERCENTILE_CONT(0.95) WITHIN GROUP (
ORDER BY
   sum_date_diff) AS ninethypercentile_reallocation 
FROM
   datesum 
GROUP BY
   region_name
```

## B. Customer Transactions

### 1. What is the unique count and total amount for each transaction type?

```sql
SELECT
   t.txn_type,
   SUM(t.txn_amount) AS total_amount,
   COUNT(*) AS transaction_count 
FROM
   data_bank.customer_transactions t 
GROUP BY
   t.txn_type
```

### 2. What is the average total historical deposit counts and amounts for all customers?

```sql
WITH total_amt AS 
(
   SELECT
      t.customer_id,
      SUM(t.txn_amount) AS total_amount,
      COUNT(*) AS transaction_count 
   FROM
      data_bank.customer_transactions t 
   WHERE
      t.txn_type = 'deposit' 
   GROUP BY
      t.customer_id 
)
SELECT
   AVG(total_amount) AS avg_amount,
   round(AVG(transaction_count)) AS avg_tran_count 
FROM
   total_amt
```

### 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?

```sql
WITH customer_txn AS 
(
   SELECT
      date_part('month', ct.txn_date) AS transaction_month,
      ct.customer_id,
      SUM(
      CASE
         WHEN
            ct.txn_type = 'deposit' 
         THEN
            1 
         ELSE
            0 
      END
) AS deposit, SUM(
      CASE
         WHEN
            ct.txn_type = 'purchase' 
         THEN
            1 
         ELSE
            0 
      END
) AS purchase, SUM(
      CASE
         WHEN
            ct.txn_type = 'withdrawal' 
         THEN
            1 
         ELSE
            0 
      END
) AS withdrawal 
   FROM
      data_bank.customer_transactions ct 
      LEFT JOIN
         data_bank.customer_nodes cn 
         ON ct.customer_id = cn.customer_id 
      LEFT JOIN
         data_bank.regions r 
         ON r.region_id = cn.region_id 
   GROUP BY
      ct.txn_date, ct.customer_id 
)
SELECT
   transaction_month,
   COUNT(DISTINCT customer_id) AS total_customers 
FROM
   customer_txn 
WHERE
   deposit > 1 
   AND 
   (
      purchase >= 1 
      OR withdrawal >= 1
   )
GROUP BY
   transaction_month
```

### 4. What is the closing balance for each customer at the end of the month?

```sql ---Rework using this website: <https://stackoverflow.com/questions/42054472/opening-and-closing-balance-query>
---You have calculated for each month instead of running calculation
WITH CREDIT_DEBIT AS (
  select cn.customer_id, EXTRACT(MONTH FROM ct.txn_date) AS MONTH_NUMBER,
case when ct.txn_type='withdrawal' or ct.txn_type='purchase' then (-1.0)*sum(ct.txn_amount) end as debit,
case when ct.txn_type='deposit' then sum(ct.txn_amount) end as credit
   FROM data_bank.customer_transactions ct 
   left join data_bank.customer_nodes cn on ct.customer_id=
   cn.customer_id
      LEFT JOIN
         data_bank.regions n 
         ON cn.region_id = n.region_id
 group by     cn.customer_id, EXTRACT(MONTH FROM ct.txn_date),
 ct.txn_type
 ORDER BY 1,2 ASC
         
  )
  SELECT customer_id, MONTH_NUMBER, SUM(debit)-SUM(credit) as Closing_Balance ---write wind func here
  from CREDIT_DEBIT
  group by 1,2 
  order by 1,2 asc
   
```

### 5. What is the percentage of customers who increase their closing balance by more than 5%?

```sql

```

## C. Data Allocation Challenge

To test out a few different hypotheses - the Data Bank team wants to run an experiment where different groups of customers would be allocated data using 3 different options:

Option 1: data is allocated based off the amount of money at the end of the previous month
Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days
Option 3: data is updated real-time
For this multi-part challenge question - you have been requested to generate the following data elements to help the Data Bank team estimate how much data will need to be provisioned for each option:

running customer balance column that includes the impact each transaction
customer balance at the end of each month
minimum, average and maximum values of the running balance for each customer
Using all of the data available - how much data would have been required for each option on a monthly basis?

```sql

```

## D. Extra Challenge

Data Bank wants to try another option which is a bit more difficult to implement - they want to calculate data growth using an interest calculation, just like in a traditional savings account you might have with a bank.

If the annual interest rate is set at 6% and the Data Bank team wants to reward its customers by increasing their data allocation based off the interest calculated on a daily basis at the end of each day, how much data would be required for this option on a monthly basis?

Special notes:

Data Bank wants an initial calculation which does not allow for compounding interest, however they may also be interested in a daily compounding interest calculation so you can try to perform this calculation if you have the stamina!
Extension Request
The Data Bank team wants you to use the outputs generated from the above sections to create a quick Powerpoint presentation which will be used as marketing materials for both external investors who might want to buy Data Bank shares and new prospective customers who might want to bank with Data Bank.

Using the outputs generated from the customer node questions, generate a few headline insights which Data Bank might use to market it’s world-leading security features to potential investors and customers.

With the transaction analysis - prepare a 1 page presentation slide which contains all the relevant information about the various options for the data provisioning so the Data Bank management team can make an informed decision.

```sql

```