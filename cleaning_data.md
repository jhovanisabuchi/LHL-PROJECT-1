## By cleaning this data i will be :
- Removing Duplicates
- Handling Missing Data
- correcting data inconsistency
- Ensuring Data Accuracy
- Standardizing Units and Formats
-  Handling Data Redundancy




# 1, Cleaning the all_sessions table 

```sql
select distinct
    fullvisitorid,
	channelgrouping,
	(time * interval'1 second')::time as time,
	case when country = '(not set)' then 'unkown'
	    else country end as country,
    case when city = '(not set)' then country
	     when city = 'not available in demo dataset' then country
		  else city end as city,
	case when totaltransactionrevenue is null then '0'
		 else totaltransactionrevenue end as totaltransactionrevenue,
	case when transactions is null then '0'
	  else transactions end as transactions,
	case when timeonsite is null then '00:00:00'
	       else (timeonsite * interval '1 second')::time end as timeonsite,
	  pageviews,
	  to_date(date::text,'yyyymmdd')::date as date,
	  visitid,
	  type,
	  (productprice/1000000) as productprice,
	  productsku,
	  initcap(v2productname) as v2productname,
	  initcap(v2productcategory) as v2productcategory,
	  coalesce(currencycode,'USD') as currencycode,
	  pagetitle,
	  pagepathlevel1,
	  ecommerceaction_type::integer,
	  ecommerceaction_step,
	  coalesce (ecommerceaction_option,'(not specified)') as  ecommerceaction_option
from all_sessions
;
```

## 2, Cleaning the anzlytics btable. 

```sql
select distinct
     visitnumber,
	 visitid, 
	 ((interval'1 second') * visitstarttime::integer)::time  as visitstarttime,
	 to_date(date::text, 'yyyymmdd')::date as date,
	 fullvisitorid,
	 channelgrouping,
	 socialengagementtype,
	 coalesce(unitssold,0) as unitssold,
	 coalesce(pageviews::integer,0) as pageviews,
	 coalesce((timeonsite * interval '1 second')::time,'00:00:00') as timeonsite,
	 coalesce(bounces, 0) as bounces,
	 coalesce(revenue::bigint, 0) as revenue,
	 (unitprice/1000000) as unitprice
from analytics
;
```

## 3, Cleaning the products table

```sql
select distinct
     productsku,
	 trim(leading from initcap(productname)) as productname,
	 ordered_quantity,
	 stock_level,
	 restocking_leadtime,
	 coalesce(sentimentscore,0) as  sentimentscore,
	 coalesce(sentimentmagnitude,0) as  sentimentmagnitude
from products
;
```

# 4, Cleaning the sales_report table

```sql
select 
   productsku,
   totalordered,
   trim(leading from initcap(productname)) as productname,
   stock_level,
   restocking_leadtime,
   sentimentscore,
   sentimentmagnitude,
   radio
 from sales_report
 ;
 ```


