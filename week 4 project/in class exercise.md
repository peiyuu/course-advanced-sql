``` sql
-- metada exploration
-- select *
-- from snowflake_sample_data.information_schema.tables
-- where table_schema = 'TPCH_SF1
-- select *
-- from vk_data.information_schema.tables
-- where table_name = 'RECIPE'

-- -- query the columns view in information_schema
-- select *
-- from vk_data.information_schema.columns
-- where table_name = 'RECIPE'

-- -- find all tables with an email column
-- select *
-- from vk_data.information_schema.columns
-- where column_name ilike '%email%'

-- -- find all tables with more than 10000 rows
-- select *
-- from vk_data.information_schema.tables
-- where row_count >= 10000

-- -- find all GEOGRAPHY data in the database
-- select *
-- from vk_data.information_schema.columns
-- where data_type = 'GEOGRAPHY'

with daily_orders as (
select
   o_orderdate as order_date
    , r_name
    , count(distinct o_orderkey) as order_count 
    , sum(l_quantity) as total_quantity
from lineitem
inner join orders on l_orderkey = o_orderkey
inner join customer on c_custkey = o_custkey
left join nation on c_nationkey = n_nationkey
left join region on r_regionkey = n_regionkey
where lower(r_name) in ('europe', 'asia', 'africa') and date_part(year, order_date) = '1992'
group by order_date, r_name
), 

highest_order_count as (
select
    order_date
    , r_name as region_with_highest_order_count
    , order_count
from daily_orders
qualify rank() over (partition by order_date order by order_count desc) = 1
) 

, highest_order_quantity as (
select 
     order_date
    , r_name as region_with_highest_order_quantity
    , total_quantity
from daily_orders
qualify rank() over (partition by order_date order by total_quantity desc) = 1
)

select 
    c.order_date
    , region_with_highest_order_count
    , order_count
    , region_with_highest_order_quantity
    , round(total_quantity,0) as total_quantity
from highest_order_count c 
inner join highest_order_quantity q on c.order_date = q.order_date
order by order_date asc
```
