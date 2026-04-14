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
