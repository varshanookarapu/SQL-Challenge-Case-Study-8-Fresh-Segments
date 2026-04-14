## Segment Analysis
---
**Question 1:** Using our filtered dataset by removing the interests with less than 6 months worth of data, which are the top 10 and bottom 10 interests which have the largest composition values in any month_year? Only use the maximum composition value for each interest but you must keep the corresponding month_year


```sql

-- creating a base data set with interest id , interest name and month_year
WITH interests AS
(
SELECT   interest_id,month_year ,interest_name ,composition,index_value,ranking ,percentile_ranking
FROM fresh_segments.interest_metrics im LEFT JOIN fresh_segments.interest_map imap
ON im.interest_id ::NUMERIC = imap.id 
ORDER BY interest_id, month_year
),

interest_month_count AS
(
SELECT interest_id, interest_name, COUNT( DISTINCT month_year) as months_count FROM 
interests 
GROUP BY interest_id ,interest_name
ORDER BY interest_id :: NUMERIC
),

filtered_interests AS (
    SELECT interest_id
    FROM interest_month_count
    WHERE months_count >= 6
),

```

```sql
-- top 10 interests with largest composition values in any month year
top_ten_interests AS
(
SELECT 
    i.month_year,i.interest_id ,i.interest_name,i.composition, RANK() OVER(PARTITION BY month_year ORDER BY composition DESC) as rank
FROM interests i
JOIN filtered_interests f
    ON i.interest_id = f.interest_id
ORDER BY i.month_year
)

SELECT * FROM top_ten_interests  WHERE rank = 1
```

<img width="1890" height="746" alt="image" src="https://github.com/user-attachments/assets/7ae2b576-7fad-4824-a298-4fcc33a24b84" />

```sql
-- Bottom 10 interests with largest composition values in any month year
bottom_ten_interests AS
(
SELECT 
    i.month_year,i.interest_id ,i.interest_name,i.composition, RANK() OVER(PARTITION BY month_year ORDER BY composition ASC) as rank
FROM interests i
JOIN filtered_interests f
    ON i.interest_id = f.interest_id
ORDER BY i.month_year
)

SELECT * FROM bottom_ten_interests  WHERE rank = 1
```
<img width="1883" height="790" alt="image" src="https://github.com/user-attachments/assets/fc231128-d007-49c1-807f-1d7eaf567ec8" />
