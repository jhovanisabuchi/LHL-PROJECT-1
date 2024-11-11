Answer the following questions and provide the SQL queries used to find the answer.

    
# Question 1: Which cities and countries have the highest level of transaction revenues on the site?""
```SQL Queries:

with total_revenue as (      -- create cte to calculate total revenue and modify the missing ata in the city and country
select  
      case when city = 'not available in demo dataset' then country 
	         else city end as city,
	  case when city = 'New York' and country= 'Canada' then 'United states'
	        else country end as country,
	  sum(totaltransactionrevenue) as total_transaction_rev

from all_sessions
where totaltransactionrevenue is not null
group by city, country
order by  total_transaction_rev desc
   )

select 
      rank() over (order by total_transaction_rev desc) as rank, --rank the cities and countries by the transaction to find the higher transaction 
      city,
	  country,
	  to_char(total_transaction_rev,'9,999,999,999') as total_transaction_rev

from total_revenue
where city != country ---- to exclude raws with missing city and i used the country name to handle the nulls and missing data
order by total_transaction_rev desc
limit 10     -- identify the top 10 highest total revenue```



***Answer:
![Screenshot 2024-11-11 100702](https://github.com/user-attachments/assets/990cd645-096d-40f3-a16b-92b96c72241e)


## Question 2: What is the average number of products ordered from visitors in each city and country?**

```SQL Queries:
select 
     case when city = 'not available in demo dataset' then country 
	         else city end as city,
	  case when city = 'New York' and country= 'Canada' then 'United states'
	        else country end as country, -- to correct some miscollected data and missing data
		Round(avg(ordered_quantity),2) as avg_order --to minimize many decimal points
from all_sessions s
   join products p
    using (productsku)
where
     ordered_quantity is not null and --filter nulls and missing data
	 ordered_quantity > 0 and
	 city not in ('(not set)', 'not available in demo dataset') and 
	 country not in ('(not set)', 'not available in demo dataset')
group by  city, country
order by  avg_order desc
;```



Answer:

![Screenshot 2024-11-07 034615](https://github.com/user-attachments/assets/537ebe84-0886-490e-91dd-0df4337b89e8)




# Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?


```SQL Queries:
total_order_location as --create cte to calculate the sum of orders grouped by category within defined city and country
             (select distinct
		           v2productcategory as category,
		          sum(ordered_quantity) as total_sales,
		         extract(month from(to_date(date::text,'yyyymmdd'))) as month,-- extract month and year from the date column
		  	      extract(year from(to_date(date::text,'yyyymmdd'))) as year,
		          city,
		           country
		     from all_sessions s
                 join products p
                   using (productsku)
            where city not in ('(not set)', 'not available in demo dataset') and -- exclude raws with missing data
			        v2productcategory not in ('(not set)')
             group by v2productcategory,
      	             extract(month from(to_date(date::text,'yyyymmdd'))),
			 	      extract(year from(to_date(date::text,'yyyymmdd'))),
		              city ,
		               country
            order by   sum(ordered_quantity) desc
                     ),
                                   -- create cte to rank the raws in order oof thier total_sales descending and ascending 
                                    --to find where those city and country are interested and least interested
  ranked_sales as (
            select  
               category,
	             month,
	              year,
	              total_sales,
	             city,
	            country,
	            rank() over (partition by city,country order by total_sales desc) as top_sales, --this ranks the raws with top_sales at the top
	            dense_rank() over (partition by city,country order by total_sales asc) as bottom_sales --this ranks the raws with low_sales at the top 

          from total_order_location )

select   
      category,
	   month,
	   year,
	   total_sales,
	    city,
	   country
  from ranked_sales 
  where (top_sales <= 3 or bottom_sales <= 3) and total_sales > 0 -- now fetch catagories which are more popular and least popular 
                                                                    --by visitors in each city and country
  order by country, city, top_sales, bottom_sales
  ;```
       



Answer:

![Screenshot 2024-11-07 034908](https://github.com/user-attachments/assets/16b41dbc-fa01-4496-9140-28d2cba59f8f)




# Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


```SQL Queries:
with total_product_sales as 
      (select distinct v2productname as productname,
              country,
			  city ,
              sum(ordered_quantity) as total_sales
	
         from all_sessions s
            join products p
             on s.productsku = p.productsku
         where city is not null and 
               city  not in( 'not available in demo dataset','(not set)')

          group by country, city, v2productname
          order by total_sales desc),

ranked_sales as (
    select  productname,
	        country,
             city,
              total_sales,
		       rank() over (partition by country, city order by total_sales desc) as sales_rank
        from  total_product_sales
	        )
select productname,
       country,
	   city,
	   to_char(total_sales, '9,999,999,999') as total_sales
from ranked_sales
where sales_rank = 1 and
      total_sales > 0
order by total_sales desc
;```




***Answer:
![Screenshot 2024-11-07 035354](https://github.com/user-attachments/assets/d6340fea-06cd-4527-abcb-39d2ff0ce67c)





# Question 5: Can we summarize the impact of revenue generated from each city/country?**

```SQL Queries:
with revenue as (
     select fullvisitorid,
           visitid::integer,
        (unitprice::bigint/1000000) as unitprice,-- change unitprice to bigint and divide by 1000000 as mentioned in the instruction
		    unitssold,
        (unitprice::bigint/1000000 * unitssold) as sales_revenue
from analytics
 where unitssold is not null and 
       unitprice is not null
	   and unitprice > 0 and 
	   unitssold  > 0-- exclude raws with nulls and 0 to avoid miscalculation
     )
 
select city,
     country, 
     to_char(sum(sales_revenue),'9,999,999,999') as total_revenue
from all_sessions s
  join revenue r
    using (visitid)
where city not in ('not available in demo dataset', '(not set)')
         and country not in ('(not set)')-- to exclude raws with missing data in city
group by city,
         country
order by total_revenue desc 
;```

;

***Answer:
![Screenshot 2024-11-07 112053](https://github.com/user-attachments/assets/68fa8d5a-4ee5-4786-ba38-ead4a5064813)

--6, ERD generated for the data base 
![Screenshot 2024-11-07 130816](https://github.com/user-attachments/assets/70b78a6b-0c33-4f14-a2a0-c1c6673bc559)








