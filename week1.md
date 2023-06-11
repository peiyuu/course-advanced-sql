# Exercise #1
```sql
-- created customer_info cte to identify eligible customers and get the geo_location of customers
-- joined supplier_info to us_cities table to find the geo_location of suppliers
-- cross joined both tables together and used st_distance to calculate difference between customer address and supplier. Divided the difference by 1609 to convert it into miles. Used a window function to determine the closest supplier

with customer_info as (
select    
    c.customer_id
    , c.first_name
    , c.last_name
    , c.email
    , trim(lower(a.customer_city)) as customer_city
    , trim(lower(a.customer_state)) as customer_state
    , cities.geo_location 
from VK_DATA.CUSTOMERS.CUSTOMER_DATA c
inner join VK_DATA.CUSTOMERS.CUSTOMER_ADDRESS a
    on c.customer_id = a.customer_id
inner join VK_DATA.RESOURCES.US_CITIES cities
    on trim(lower(a.customer_state)) = lower(cities.state_abbr) 
    and trim(lower(a.customer_city)) = lower(cities.city_name)
where a.customer_city is not null and a.customer_state is not null
)

, supplier_geo_location as (
    select 
        s.supplier_id
        , s.supplier_name
        , s.supplier_city
        , s.supplier_state
        , c.geo_location
    from VK_DATA.SUPPLIERS.SUPPLIER_INFO s
    inner join VK_DATA.RESOURCES.US_CITIES c 
        on trim(lower(s.supplier_city)) = lower(c.city_name) and trim(lower(s.supplier_state)) = lower(c.state_abbr)
) 

, distance as (
select 
   c.customer_id
    , c.first_name
    , c.last_name
    , c.email
    , s.supplier_id
    , s.supplier_name
    , st_distance(c.geo_location, s.geo_location) / 1609  as distance_in_miles
    , row_number() over (partition by customer_id order by distance_in_miles asc) as rk
from customer_info c 
cross join supplier_geo_location s
) 

select 
     customer_id
    , first_name
    , last_name
    , email
    , supplier_id
    , supplier_name
    , distance_in_miles
from distance
where rk = 1 
order by last_name, first_name
```
