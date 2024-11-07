What are your risk areas? Identify and describe them.
* missing or incomplete data
* imposible data e.g negative unitssold
* duplicates
* incorrect data formats 

QA Process:
Describe your QA process and include the SQL queries used to execute it.

--1, check for duplicates of fullvisitorid and productsku

select fullvisitorid 
from all_sessions ----15134

select distinct fullvisitorid 
from all_sessions ---14223

/*so as we can see the fullvistorid has some duplicates
for further investigation lets check why it has duplicates */


select fullvisitorid, count(productsku)
from all_sessions
group by fullvisitorid
having count(productsku) > 1
order by count(productsku) desc ---794

/* so there are 794 visitors who ordered more than 1 products it
 means we have fullvisitorid which is more than 1 session*/

 select * from analytics --4,301,122
 
 select distinct * from analytics--1,739,308

--In the anlytics table we have duplicates.

/*we can check every column like that and it will give us insights what thos duplicates 
  are happening remember not all duplicates are dirty data*/

--2, check for nulls
/* as we discovered when we clean the data there are plenty of columns and raws within the tables with
  null*/
  
  select * 
  from analytics
  where unitssold is null--4,205,975
 
 /* so we can se there are more than 4 milion and we can check like that for 
     all columns and we can use some sql queries to remove those null and reolace them we 
	 0 or n/a something that can represent the nulls*/

select distinct
	 coalesce(unitssold,0) as unitssold,
	 coalesce(pageviews::integer,0) as pageviews,
	 coalesce((timeonsite * interval '1 second')::time,'00:00:00') as timeonsite,
	 coalesce(bounces, 0) as bounces,
	 coalesce(revenue::bigint, 0) as revenue
from analytics

/* as we can see in the analytics table we nulls in 5 columns and clean thhose nulls with that query
   and assure the data is valide*/

 --3, impossible data and incorrect formating
  
  select * 
  from analytics
  where unitssold < 0  -- 1 raw with -89 units sold and thats impossible

 select *
 from all_sessions
 where city in ('not available in demo dataset','(not set)' )-- 8656

 select *
 from all_sessions
 where country in ('not available in demo dataset','(not set)' )--24

 /* as we can see there many countries and cities with missing data*/

 --4, wrong data formats e.g date, time 

 select 
  (time * interval'1 second')::time as time,   --to change the data format in to time
	case when timeonsite is null then '00:00:00'
	       else (timeonsite * interval '1 second')::time end as timeonsite,
		   to_date(date::text,'yyyymmdd')::date as date, --to change the data format in to date format
		    ecommerceaction_type::integer
	from all_sessions


QA Process:
Describe your QA process and include the SQL queries used to execute it.
