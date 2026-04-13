# Case Study 8 : Fresh Segments

## Data Exploration and Cleansing

```sql
SELECT _month as month , _year as year , 
--used concatenation here as start of month is always 01 , you'll notice the format that we have given in the fuction and the format that it outputs is different because PostgreSQL stores dates internally in ISO format (YYYY-MM-DD)
TO_DATE(month_year  ||  '-01' , 'MM-YYYY-DD')  as month_year , interest_id,composition,index_value, ranking,percentile_ranking

FROM fresh_segments.interest_metrics LIMIT 10;
```

