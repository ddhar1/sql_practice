Questions from here: https://quip.com/2gwZArKuWk7W

# The Best Medium-Hard Data Analyst SQL Interview Questions  

# 1. Month over Month change for MAU

>*Context:* Oftentimes it's useful to know how much a key metric, such as monthly active users, changes between months. Say we have a table logins in the form: 
>
>| user_id | date       |  
>|---------|------------|  
>| 1       | 2018-07-01 |  
>| 234     | 2018-07-02 |  
>| 3       | 2018-07-02 |  
>| 1       | 2018-07-02 |  
>| ...     | ...        |  
>| 234     | 2018-10-04 |  
>
>*Task*: Find the MoM percentage change for MAU. 


### Questions I would initially ask:**  
* do you mean average MoM change? Or just between two months?
* What is the datatype of datetime
* rounding?
* clarify month over month equation
Assumptions:
* assume just from month to month over time
* writing 
* first thing we would need is number of members per month
* then we would need to order by month, and somehow get the month over month change - i would imagine a window function would be useful for this, or a self join
* we can make the first thing we do a cte

```
with mau_monthly as (
select month(date) as month, year(date) as year, count(*) as mau
from logins
group by month(date), year(date) 
)

select month, year, mau, ( lag(mau, 1) over( order by year, month ) - mau*1.0 ) / lag(mau, 1) over( order by year, month ) as mua_perc
```

## 2. Tree Structure Labeling
>*Context:* Say you have a table tree with a column of nodes and a column corresponding parent nodes 
>
>node   parent  
>1       2  
>2       5  
>3       5  
>4       3  
>5       NULL   
>
>*Task:* Write SQL such that we label each node as a “leaf”, “inner” or “Root” node, such that for the nodes above we get: 

>node    label   
>1       Leaf  
>2       Inner  
>3       Inner  
>4       Leaf  
>5       Root  


- does this require recursion? Do we know the height of the tree?
- if the parent column is null we automatically know the node is a root. 
- If the node doesn't has a parent node but isn't in the parent column itself, then we know its' a leaf
- if it's none of these, then we know it's a inner node

select node, case when node not in (select parent from tree ) then 'leaf' when parent is null then 'root' else 'inner' end as label from tree

## 3. Retained Users per Month (multi-part)

## Histograms:
>*Context:* Say we have a table sessions where each row is a video streaming session with length in seconds: 
>
>| session_id | length_seconds |  
>|------------|----------------|  
>| 1          | 23             |  
>| 2          | 453            |  
>| 3          | 27             |  
>| ..         | ..             |  
>
>*Task:* Write a query to count the number of sessions that fall into bands of size 5, i.e. for the above snippet, produce something akin to: 
>
>| bucket  | count |
>|---------|-------|
>| 20-25   | 2     |
>| 450-455 | 1     |
>
>Get complete credit for the proper string labels (“5-10”, etc.) but near complete credit for something that is communicable as the bin. 

### Thought process
* I can use 