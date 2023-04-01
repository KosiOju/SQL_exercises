# Health Analytics Mini Case Study
---
## Summary
    The following mini-case study is from the serious sql data with danny course.

We’ve just received an urgent request from the General Manager of Analytics at Health Co requesting our assistance with their analysis of the **health.user_logs** dataset.

The Health Co analytics team have shared with us their SQL script - they unfortunately ran into a few bugs that they couldn’t fix!

We’ve been asked to quickly debug their SQL script and use the resulting query outputs to quickly answer a few questions that the GM has requested for a board meeting about their active users.

## Business Questions
Before we start digging into the SQL script - let’s cover the business questions that we need to help the GM answer!

1. How many unique users exist in the logs dataset?
2. How many total measurements do we have per user on average?
3. What about the median number of measurements per user?
4. How many users have 3 or more measurements?
5. How many users have 1,000 or more measurements?

*Looking at the logs data - what is the number and percentage of the active user base who:*

6. Have logged blood glucose measurements?
7. Have at least 2 types of measurements?
8. Have all 3 measures - blood glucose, weight and blood pressure?

*For users that have blood pressure measurements:*

9. What is the median systolic/diastolic blood pressure values?

## SQL Script

```sql
-- 1. How many unique users exist in the logs dataset?
SELECT COUNT(DISTINCT id)
FROM health.user_logs;

-- for questions 2-8 we created a temporary table
DROP TABLE IF EXISTS user_measure_count;
CREATE TEMP TABLE user_measure_count AS (
SELECT
    id,
    COUNT(*) AS measure_count,
    COUNT(DISTINCT measure) as unique_measures
  FROM health.user_logs
  GROUP BY 1 ); 
--554

-- 2. How many total measurements do we have per user on average?
SELECT
  ROUND(AVG(measure_count::NUMERIC), 2)
FROM user_measure_count;
--79.23

-- 3. What about the median number of measurements per user?
SELECT
 PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_count) AS median_value
FROM user_measure_count;
--2

-- 4. How many users have 3 or more measurements?
SELECT
  COUNT(*)
FROM user_measure_count
WHERE measure_count >= 3;
--209

-- 5. How many users have 1,000 or more measurements?
SELECT
  COUNT(id)
FROM user_measure_count
WHERE measure_count >= 1000;

-- 6. Have logged blood glucose measurements?
SELECT
  COUNT (DISTINCT id)
FROM health.user_logs
WHERE measure = 'blood_glucose';
--325

-- 7. Have at least 2 types of measurements?
SELECT
  COUNT (*)
FROM user_measure_count
WHERE unique_measures >= 2;
--204

-- 8. Have all 3 measures - blood glucose, weight and blood pressure?
SELECT
  COUNT (*)
FROM user_measure_count
WHERE unique_measures = 3;
--50


-- 9.  What is the median systolic/diastolic blood pressure values?
SELECT
  PERCENTILE_CONT(0.5) WITHIN (ORDER BY systolic) AS median_systolic
  PERCENTILE_CONT(0.5) WITHIN (ORDER BY diastolic) AS median_diastolic
FROM health.user_logs
WHERE measure is blood_pressure;
--median_systolic: 126
--median_diastolic: 79
```