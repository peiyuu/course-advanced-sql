# Part I
``` sql
--- find automobile customers
with auto_customer as (
select c_custkey
from customer
where c_mktsegment = 'AUTOMOBILE'
)

-- find top 3 urgent orders from orders table
, top_3_urgent_orders as (
select 
    o_custkey
   , o_orderdate
   , o_orderkey 
   , o_totalprice
from orders
where o_orderpriority ilike '%URGENT%' 
group by 1,2,3,4
qualify rank() over (partition by o_custkey order by o_totalprice desc) <=3 
) 

-- find last order date, order numbers, and total spent
, orders_agg as (
select 
    o_custkey
    , max(o_orderdate) as last_order_date
    , listagg(o_orderkey, ', ') as order_numbers
    , sum(o_totalprice) as total_spent
from top_3_urgent_orders
group by o_custkey 
) 

-- find the most expensive parts of the 3 orders
, top_3_individual_parts as (
select
    c_custkey
    , l_partkey 
    , l_quantity
    , l_extendedprice 
    , rank() over (partition by c_custkey order by l_extendedprice desc) as rk 
from lineitem 
inner join top_3_urgent_orders on l_orderkey = o_orderkey
inner join auto_customer on o_custkey = c_custkey
group by c_custkey, l_partkey, l_quantity, l_extendedprice
qualify rk <= 3
)

-- pivot using case when and max 
select
    c_custkey
    , last_order_date
    , order_numbers
    , total_spent
    , max(case when rk = 1 then l_partkey else 0 end) as part_1_key
    , max(case when rk = 1 then l_quantity else 0 end) as part_1_quantity
    , max(case when rk = 1 then l_extendedprice else 0 end) as part_1_total_spent
    , max(case when rk = 2 then l_partkey else 0 end) as part_1_key
    , max(case when rk = 2 then l_quantity else 0 end) as part_1_quantity
    , max(case when rk = 2 then l_extendedprice else 0 end) as part_1_total_spent
    , max(case when rk = 3 then l_partkey else 0 end) as part_1_key
    , max(case when rk = 3 then l_quantity else 0 end) as part_1_quantity
    , max(case when rk = 3 then l_extendedprice else 0 end) as part_1_total_spent
from top_3_individual_parts 
inner join orders_agg on o_custkey = c_custkey
-- where c_custkey = '101405'
group by 1,2,3,4
limit 100
```
# Part II 
a. I don't agree with the query results. The query returns the total spent of the top 3 parts and not the top 3 orders. 
<br> b. It is easy to understand. 
<br> c. For readability, I would remove the joins at the end and use case when statements instead. 
