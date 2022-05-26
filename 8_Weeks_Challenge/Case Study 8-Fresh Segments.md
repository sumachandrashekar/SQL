# Case Study 8 - Fresh Segments

## Problem Statement: [Fresh Segments](https://8weeksqlchallenge.com/case-study-8/)

![Schema file](SQLSchema/CaseStudy_8_FreshSegments.sql)

### Status: In Progress

The following questions can be considered key business questions that are required to be answered for the Fresh Segments team.

Most questions can be answered using a single query however some questions are more open ended and require additional thought and not just a coded solution!

## Data Exploration and Cleansing

### 1. Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month

```sql
ALTER TABLE fresh_segments.interest_metrics ALTER month_year TYPE DATE USING month_year::DATE;
```

### 2. What is count of records in the fresh_segments.interest_metrics for each month_year value sorted in chronological order (earliest to latest) with the null values appearing first?

```sql
SELECT
   month_year,
   COUNT(*) 
FROM
   fresh_segments.interest_metrics 
GROUP BY
   month_year 
ORDER BY
   month_year NULLS FIRST;
```

### 3. What do you think we should do with these null values in the fresh_segments.interest_metrics

```sql
SELECT
   ROUND(100 * (SUM(
   CASE
      WHEN
         interest_id IS NULL 
      THEN
         1 
   END
) * 1.0 / COUNT(*)), 2) AS null_perc 
FROM
   fresh_segments.interest_metrics
```

### 4. How many interest_id values exist in the fresh_segments.interest_metrics table but not in the fresh_segments.interest_map table? What about the other way around?

```sql
SELECT
   COUNT(DISTINCT map.id) AS map_id_count,
   COUNT(DISTINCT metrics.interest_id) AS metrics_id_count,
   SUM(
   CASE
      WHEN
         map.id IS NULL 
      THEN
         1 
   END
) AS not_in_metric, SUM(
   CASE
      WHEN
         metrics.interest_id IS NULL 
      THEN
         1 
   END
) AS not_in_map 
FROM
   fresh_segments.interest_map map 
   FULL OUTER JOIN
      fresh_segments.interest_metrics metrics 
      ON metrics.interest_id = map.id;
```

### 5. Summarize the id values in the fresh_segments.interest_map by its total record count in this table

```sql
SELECT
   id,
   interest_name,
   COUNT(*) 
FROM
   fresh_segments.interest_map map 
   JOIN
      fresh_segments.interest_metrics metrics 
      ON map.id = metrics.interest_id 
GROUP BY
   id,
   interest_name 
ORDER BY
   COUNT DESC,
   id;
```

### 6 What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where interest_id = 21246 in your joined output and include all columns from fresh_segments. interest_metrics and all columns from fresh_segments.interest_map except from the id column.

```sql
SELECT
   * 
FROM
   fresh_segments.interest_map map 
   INNER JOIN
      fresh_segments.interest_metrics metrics 
      ON map.id = metrics.interest_id 
WHERE
   metrics.interest_id = 21246 
   AND metrics._MONTH IS NOT NULL
```

### 7. Are there any records in your joined table where the month_year value is before the created_at value from the fresh_segments.interest_map table? Do you think these values are valid and why?

```sql
SELECT
   COUNT(*) 
FROM
   fresh_segments.interest_map map 
   INNER JOIN
      fresh_segments.interest_metrics metrics 
      ON map.id = metrics.interest_id 
WHERE
   metrics.month_year < map.created_at::DATE;
```

## Interest Analysis

### 1. Which interests have been present in all month_year dates in our dataset?

```sql
SELECT
   COUNT(DISTINCT month_year) AS unique_month_year_count,
   COUNT(DISTINCT interest_id) AS unique_interest_id_count 
FROM
   fresh_segments.interest_metrics;
WITH interest_cte AS 
(
   SELECT
      interest_id,
      COUNT(DISTINCT month_year) AS total_months 
   FROM
      fresh_segments.interest_metrics 
   WHERE
      month_year IS NOT NULL 
   GROUP BY
      interest_id 
)
SELECT
   c.total_months,
   COUNT(DISTINCT c.interest_id) 
FROM
   interest_cte c 
WHERE
   total_months = 14 
GROUP BY
   c.total_months 
ORDER BY
   COUNT DESC;
```

### 2. Using this same total_months measure - calculate the cumulative percentage of all records starting at 14 months - which total_months value passes the 90% cumulative percentage value?

```sql
WITH cte_interest_months AS 
(
   SELECT
      interest_id,
      MAX(DISTINCT month_year) AS total_months 
   FROM
      fresh_segments.interest_metrics 
   WHERE
      interest_id IS NOT NULL 
   GROUP BY
      interest_id 
)
,
cte_interest_counts AS 
(
   SELECT
      total_months,
      COUNT(DISTINCT interest_id) AS interest_count 
   FROM
      cte_interest_months 
   GROUP BY
      total_months 
)
SELECT
   total_months,
   interest_count,
   ROUND(100 * SUM(interest_count) OVER (
ORDER BY
   total_months DESC) / 	-- Create running total field using cumulative values of interest count
(SUM(INTEREST_COUNT) OVER ()), 2) AS cumulative_percentage 
FROM
   cte_interest_counts;
```

### 3. If we were to remove all interest_id values which are lower than the total_months value we found in the previous question - how many total data points would we be removing?

```sql

```

### 4 Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed interest example for your arguments - think about what it means to have less months present from a segment perspective.

```sql

```

### 5. After removing these interests - how many unique interests are there for each month?

```sql

```

## Segment Analysis

### 1. Using our filtered dataset by removing the interests with less than 6 months worth of data, which are the top 10 and bottom 10 interests which have the largest composition values in any month_year? Only use the maximum composition value for each interest but you must keep the corresponding month_year

```sql

```

### 2. Which 5 interests had the lowest average ranking value?

```sql

```

### 3. Which 5 interests had the largest standard deviation in their percentile_ranking value?

```sql

```

### 4. For the 5 interests found in the previous question - what was minimum and maximum percentile_ranking values for each interest and its corresponding year_month value? Can you describe what is happening for these 5 interests?

```sql

```

### 5. How would you describe our customers in this segment based off their composition and ranking values? What sort of products or services should we show to these customers and what should we avoid?

```sql

```

## Index Analysis

The ==index_value== is a measure which can be used to reverse calculate the average composition for Fresh Segments’ clients.

Average composition can be calculated by dividing the composition column by the index_value column rounded to 2 decimal places.

### 1. What is the top 10 interests by the average composition for each month?

```sql

```

### 2. For all of these top 10 interests - which interest appears the most often?

```sql

```

### 3. What is the average of the average composition for the top 10 interests for each month?

```sql

```

### 4. What is the 3 month rolling average of the max average composition value from September 2018 to August 2019 and include the previous top ranking interests in the same output shown below.

```sql

```

Provide a possible reason why the max average composition might change from month to month. Could it signal something is not quite right with the overall business model for Fresh Segments?