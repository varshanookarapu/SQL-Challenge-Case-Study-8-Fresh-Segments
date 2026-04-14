## Interest Analysis


**Question 1:** Which interests have been present in all month_year dates in our dataset?

```sql
-- To address this question first we need to understand how many distinct month_year dates are present in our data set 

SELECT COUNT(DISTINCT month_year) FROM fresh_segments.interest_metrics;

-- creating a base data set with interest id , interest name and month_year
WITH interests AS
(
SELECT   interest_id,month_year ,interest_name
FROM fresh_segments.interest_metrics im LEFT JOIN fresh_segments.interest_map imap
ON im.interest_id ::NUMERIC = imap.id 
ORDER BY interest_id, month_year
)

--querying the interest_id and interest names and filerting those interests whose month_year count is 14

SELECT interest_id, interest_name, COUNT( DISTINCT month_year) as months_count FROM 
interests 
GROUP BY interest_id ,interest_name
HAVING COUNT( DISTINCT month_year) IN (14)
ORDER BY interest_id :: NUMERIC

```
-- There are a total of 480 interests that have been present in all 14 month_year dates in our dataset 
<img width="1887" height="797" alt="image" src="https://github.com/user-attachments/assets/6c85a524-31f1-4a64-b664-7a2e9adbb30b" />
<img width="744" height="152" alt="image" src="https://github.com/user-attachments/assets/b4f497ec-e105-4cd0-98bd-c7839a5a5e96" />

---

**Question 2:** Using this same total_months measure - calculate the cumulative percentage of all records starting at 14 months - which total_months value passes the 90% cumulative percentage value?

```sql

-- creating a base data set with interest id , interest name and month_year
WITH interests AS
(
SELECT   interest_id,month_year ,interest_name
FROM fresh_segments.interest_metrics im LEFT JOIN fresh_segments.interest_map imap
ON im.interest_id ::NUMERIC = imap.id 
ORDER BY interest_id, month_year
),

--querying the interest_id and interest names and filerting those interests whose month_year count is 14
interest_month_count AS
(
SELECT interest_id, interest_name, COUNT( DISTINCT month_year) as months_count FROM 
interests 
GROUP BY interest_id ,interest_name
ORDER BY interest_id :: NUMERIC
),

interest_cumulative AS
(
SELECT months_count, COUNT(interest_id) as intrests_count 
FROM interest_month_count
GROUP BY months_count
ORDER BY months_count DESC
--WHERE months_count <  (SELECT MAX(months_count) FROM interest_month_count )
)

SELECT months_count, intrests_count , SUM(intrests_count) OVER(ORDER BY months_count DESC) as cumulative_count,
--main logic
 ROUND( SUM(intrests_count) OVER(ORDER BY months_count DESC)/ SUM(intrests_count)OVER() *100 ,2) ::NUMERIC as cumulative_percentage 

FROM interest_cumulative
```

---

**Question 3:** If we were to remove all interest_id values which are lower than the total_months value we found in the previous question - how many total data points would we be removing?

```sql
```

---

**Question 4:** Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed interest example for your arguments - think about what it means to have less months present from a segment perspective.

```sql
```

---

**Question 5:** After removing these interests - how many unique interests are there for each month?

```sql
```

---
