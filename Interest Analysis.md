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
This question required a bit of math and an understanding of what cumulative percentages actually are. In simple terms, 
**cumulative percentage means that as we move through values step by step, we measure what percentage of the total we have covered so far.**
Based on that, we currently have the count of months and the record counts for those months. To address this question, we calculate the cumulative count — i.e., the first row is the first row value, the second row is the first row value plus the second row value, and so on.

Now, the question asks us to calculate the cumulative percentage:

cumulative percentage = (cumulative count / total record count) × 100 -- for every row 
We get the cumulative count using: SUM(intrests_count) OVER(ORDER BY month_count DESC)
and we get the total record count using:SUM(intrests_count) OVER() -- we are using window function here because we want the total interests count repeated for every row.
By substituting these values and rounding them, we obtain the cumulative percentages.

To address the second part of the question, The cumulative percentage crosses 90% at months_count = 6.
This means that when we include all records with months_count greater than or equal to 6, we have covered approximately 90% of the total data.
<img width="1774" height="762" alt="image" src="https://github.com/user-attachments/assets/4df18e7f-61ab-40f8-a196-525f9d52177d" />

---

**Question 3:** If we were to remove all interest_id values which are lower than the total_months value we found in the previous question - how many total data points would we be removing?

```sql
-- From the previous question the total_months value where the cumulative percentage crosses 90% is 6

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

-- data points to be removed 
SELECT SUM(intrests_count) as data_points_to_be_removed  FROM interest_cumulative
WHERE months_count  <6

-- individual  breakdown
SELECT months_count, intrests_count 
FROM interest_cumulative
WHERE months_count  <6 ;

```
<img width="410" height="151" alt="image" src="https://github.com/user-attachments/assets/c01db49a-70a9-4372-8abf-f87eeb529392" />

<img width="1066" height="317" alt="image" src="https://github.com/user-attachments/assets/b622fc22-94fd-4e6a-abe0-339e1a86950d" />

---

**Question 4:** Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed interest example for your arguments - think about what it means to have less months present from a segment perspective.

```sql

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
)

-- We need to compare these two scenarios
SELECT interest_id, interest_name ,months_count FROM interest_month_count WHERE months_count IN (5,4,3,2,1)

SELECT interest_id, interest_name ,months_count FROM interest_month_count WHERE months_count IN (14)
```
From a business perspective, interests that appear in all 14 months (e.g., wedding planners, vacation planners, NBA fans) show stable, long-term behavior. These interests show users main preferences .

compared to interests that appear for only 1–5 months (e.g., anime fans, space enthusiasts) are short-term or inconsistent. They may reflect temporary trends, seasonal activity, or changing interests.

Removing these short-term interests reduces noise and makes the data more focused on stable patterns. However, it also means losing information about new or trending interests, so it depends on whether the goal is long-term analysis or capturing change over time.

---

**Question 5:** After removing these interests - how many unique interests are there for each month?

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
),

-- this is where we are filtering all the interest ids where the months count is >  which makes up for the 90% of the dataset 
filtered_interests AS (
    SELECT interest_id 
    FROM interest_month_count
    WHERE months_count >= 6   
)
-- SELECT SUM(intrests_count) as data_points_to_be_removed  FROM interest_cumulative
-- WHERE months_count  >=6

SELECT month_year ,  COUNT(fi.interest_id) as unique_intrests_count 
FROM filtered_interests fi JOIN interests i 
ON fi.interest_id = i.interest_id 
WHERE month_year IS NOT NULL
GROUP BY i.month_year
ORDER BY i.month_year

```
<img width="1107" height="705" alt="image" src="https://github.com/user-attachments/assets/a0f9c4c9-3f3b-49d5-baaa-35174736fc1b" />

---
