# Examining Bikeway Networks dataset (the dataset is pulled from Mode analytics)

The original file can be found [here](https://data.sfgov.org/Transportation/SFMTA-Bikeway-Network/ygmz-vaxd). While most of my data work was in python,
I ran some queries in order to understand what sort of data I was dealing with. 


1. Total bikeway length per year – essentially this is a cumulative sum of bikeway way lengths per year.

I'm glad I pulled this one because it's here I realized that it's a bad idea to just compare bike lane accidents by year - there's clearly a difference
and bike lane length is a variable.
I started to standardize accidents counts by % of road available that year, as a result.

Since bike lanes that existed before 1971 do not have a install year, I impute 1970 for them
```sql
SELECT t2.year,
       sum(bikeway_length) over (
                                 ORDER BY t2.year)
FROM
  (SELECT t1.year AS year,
          sum(length) AS bikeway_length
   FROM
     (SELECT CASE
                 WHEN install_yr = 0
                      OR install_yr is NULL then 1970
                 ELSE install_yr
             END AS year,
             length
      FROM public.bikeway_networks) t1
   GROUP BY t1.year) t2
 ```
3. Percent of commented cases from 311 (311 being a hotline to report cars being in blocked lanes) - wanted to see if there was a relationship between comments and area
```sql
select t1.neighborhoods_sffind_boundaries, 
  round(SUM(case when description is not null then 1 else 0 end)/COUNT(*),3)*100 
  as percent_commented 
  from public.sf311_cases t1
  group by t1.neighborhoods_sffind_boundaries 
```

4. Select counts of how many bike lanes per 'symbology' (or bike lane) type. A symbology types include a seperated bike lane, and a regular old bike lane
```sql
select symbology, count(id) from public.bikeway_networks
  group by symbology
```

4. Percent of a bike lane type with a sharrow (or lane marking that it is a bike lane)
```sql
select symbology, sum(sharrow)/count(*) as percent_with_sharrow from  public.bikeway_networks
  group by symbology
```

