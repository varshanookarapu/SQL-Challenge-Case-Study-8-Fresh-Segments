# Case Study 8 : Fresh Segments

## Data Exploration and Cleansing

**Question 1:** Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month
```sql
SELECT _month as month , _year as year , 
--used concatenation here as start of month is always 01 , you'll notice the format that we have given in the fuction and the format that it outputs is different because PostgreSQL stores dates internally in ISO format (YYYY-MM-DD)
TO_DATE(month_year  ||  '-01' , 'MM-YYYY-DD')  as month_year , interest_id,composition,index_value, ranking,percentile_ranking

FROM fresh_segments.interest_metrics LIMIT 10;
```
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

```sql
SELECT id, COUNT(*) as count_of_records FROM fresh_segments.interest_map
GROUP BY id
ORDER BY id
```
---

**Question 6:** What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where interest_id = 21246 in your joined output and include all columns from fresh_segments.interest_metrics and all columns from

```sql
```
---

**Question 7:** Are there any records in your joined table where the month_year value is before the created_at value from the fresh_segments.interest_map table? Do you think these values are valid and why?

```sql
```
---
