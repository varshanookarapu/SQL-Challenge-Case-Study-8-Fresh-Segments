# Case Study 8 : Fresh Segments

## Data Exploration and Cleansing

**Question 1:** Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month
```sql

-- Check
SELECT _month as month , _year as year , 
--used concatenation here as start of month is always 01 , you'll notice the format that we have given in the fuction and the format that it outputs is different because PostgreSQL stores dates internally in ISO format (YYYY-MM-DD)
TO_DATE(month_year  ||  '-01' , 'MM-YYYY-DD')  as month_year , interest_id,composition,index_value, ranking,percentile_ranking

FROM fresh_segments.interest_metrics LIMIT 10;

-- Actual Answer , updating the data type as well as the month year values
ALTER TABLE fresh_segments.interest_metrics
ALTER COLUMN month_year TYPE DATE 
USING TO_DATE(month_year || '-01', 'MM-YYYY-DD');

SELECT * FROM fresh_segments.interest_metrics  LIMIT 5; 
```
<img width="1805" height="340" alt="image" src="https://github.com/user-attachments/assets/2d93d6fe-106b-4438-960b-69de68d1baac" />

**Question 2:** What is count of records in the fresh_segments.interest_metrics for each month_year value sorted in chronological order (earliest to latest) with the null values appearing first?

```sql
SELECT month_year , COUNT(*) AS count_of_records
FROM fresh_segments.interest_metrics 
GROUP BY month_year 
ORDER BY month_year DESC
```
<img width="1190" height="766" alt="image" src="https://github.com/user-attachments/assets/34744ea2-af9c-4292-a4da-305856769b27" />
---

**Question 3:** What do you think we should do with these null values in the fresh_segments.interest_metrics

```sql
I believe we should not consider the null values for the analyssis and remove them as the interest id is also null and we would not be able to map with the interest names 
```
---

**Question 4:** How many interest_id values exist in the fresh_segments.interest_metrics table but not in the fresh_segments.interest_map table? What about the other way around?

```sql
SELECT  COUNT(DISTINCT interest_id) as interest_id_count_in_interst_metrics
FROM fresh_segments.interest_metrics  im
LEFT JOIN fresh_segments.interest_map  imap 
ON im.interest_id :: NUMERIC = imap.id ;


SELECT  COUNT(DISTINCT id) as interest_id_count_in_interst_map
FROM fresh_segments.interest_metrics  im
RIGHT JOIN fresh_segments.interest_map  imap 
ON im.interest_id :: NUMERIC = imap.id ;

```
<img width="483" height="281" alt="image" src="https://github.com/user-attachments/assets/ce9cb8e3-f3b5-4127-9a4e-6c076b653b52" />

---

**Question 5:** Summarise the id values in the fresh_segments.interest_map by its total record count in this table

This question was a bit ambiguous to me I am assuming they wanted to join both the tables , summarize the interest_id values and their record counts.
```sql
SELECT interest_id,interest_name,COUNT(*) AS records_count
FROM fresh_segments.interest_metrics  im
LEFT JOIN fresh_segments.interest_map  imap 
ON im.interest_id :: NUMERIC = imap.id 
WHERE interest_id IS NOT NULL
GROUP BY interest_id ,interest_name
ORDER BY 3 DESC'

-- But then again if I take the question literally then this is  the following query
SELECT id,interest_name,COUNT(*) AS records_count
FROM 
fresh_segments.interest_map 
WHERE id IS NOT NULL
GROUP BY id ,interest_name
ORDER BY 3 DESC
```
<img width="1898" height="601" alt="image" src="https://github.com/user-attachments/assets/792a46c6-184e-4a6e-8961-6bdfc71b489f" />
<img width="1781" height="587" alt="image" src="https://github.com/user-attachments/assets/50fd4274-abc6-4140-886e-22921f97bcc3" />

---

**Question 6:** What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where interest_id = 21246 in your joined output and include all columns from fresh_segments.interest_metrics and all columns from fresh_segments.interest_map except from the id column.
Used LEFT JOIN for the analysis
```sql
SELECT  _month as month, _year as year, month_year, interest_id,composition,index_value,ranking,percentile_ranking,interest_name,interest_summary,created_at,last_modified
FROM fresh_segments.interest_metrics  im
LEFT JOIN fresh_segments.interest_map  imap 
ON im.interest_id :: NUMERIC = imap.id 
WHERE interest_id = '21246'
-- can filter out the null values bu adding "AND interest_id IS NOT NULL" to the WHERE clause as the interest id is avaliable but  the date and year values are null for further analysis
```
<img width="1901" height="571" alt="image" src="https://github.com/user-attachments/assets/9061f646-c212-4e7c-96b7-a680e77d3c51" />

---


**Question 7:** Are there any records in your joined table where the month_year value is before the created_at value from the fresh_segments.interest_map table? Do you think these values are valid and why?

```sql
ALTER TABLE fresh_segments.interest_metrics ALTER COLUMN month_year TYPE DATE USING TO_DATE(month_year || '-01', 'MM-YYYY-DD');

SELECT interest_id, month_year ,created_at :: DATE FROM fresh_segments.interest_metrics im LEFT JOIN fresh_segments.interest_map imap ON im.interest_id ::NUMERIC = imap.id WHERE month_year < created_at :: DATE LIMIT 10

-- scenario where month_year > created_at
SELECT   interest_id, month_year ,created_at :: DATE
FROM fresh_segments.interest_metrics im LEFT JOIN fresh_segments.interest_map imap
ON im.interest_id ::NUMERIC = imap.id 
WHERE month_year > created_at :: DATE AND interest_id ='1'


```
---

My interpretation is that the created_at value represents the date when the interest was created, whereas the month_year value from when the interest is being tracked
I do see cases where the month_year value is earlier than the created_at value. However, since the created_at date falls within the same month as month_year, I believe these values are still valid, and the analysis would not be affected.

but if we interpert month_year as start of month and the month_year < created_at it means they started tracking long before the interest was created. then the values would be invalid.

<img width="1726" height="513" alt="image" src="https://github.com/user-attachments/assets/5d3ec34e-cb61-460c-9034-a0fe07c9372f" />

In the following scenario , the metrics was created in May 2016 , but it has been tracked over time hece we see the month_year to in the years 2019 , 2018  etc. 
<img width="1549" height="664" alt="image" src="https://github.com/user-attachments/assets/9d0411ad-e549-408b-a9cf-66e18bc6920b" />

