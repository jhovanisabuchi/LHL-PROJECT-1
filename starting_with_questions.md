Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


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
	  total_transaction_rev

from total_revenue
where city != country ---- to exclude raws with missing city and i used the country name to handle the nulls and missing data
order by total_transaction_rev desc
limit 10     -- identify the top 10 highest total revenue


Answer:



![Screenshot 2024-11-07 033343](https://github.com/user-attachments/assets/ca080d1c-b997-405c-bb87-3ad8cd743858)


**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:
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
;



Answer:





**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:
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
  ;
       



Answer:





**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:
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
	   total_sales
from ranked_sales
where sales_rank = 1 and
      total_sales > 0
order by total_sales desc
;




Answer:





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:
with revenue as (
     select fullvisitorid,
           visitid::integer,
        (unitprice::bigint/1000000) as unitprice,-- change unitprice to bigint and divide by 1000000 as mentioned in the instruction
		    unitssold,
        (unitprice::bigint/1000000 * unitssold) as sales_revenue
from analytics
 where unitssold is not null and 
       unitprice is not null
	   and unitprice > 0 -- exclude raws with nulls and 0 to avoid miscalculation
     )
 
select city,
     country, 
     sum(sales_revenue) as total_revenue
from all_sessions s
  join revenue r
    using (visitid)
where city not in ('not available in demo dataset', '(not set)')
         and country not in ('(not set)')-- to exclude raws with missing data in city
group by city,
         country
order by total_revenue desc 
;





Answer:







