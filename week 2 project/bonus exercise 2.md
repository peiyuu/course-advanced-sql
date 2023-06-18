# Bonus Exercise #2
```sql
-- clean up us_cities
with us_cities_sanitized as (
select
    city_id,
    trim(upper(city_name)) as city_name,
    trim(upper(state_abbr)) as state_abbr,
    trim(upper(city_name)) || ', ' || trim(upper(state_abbr)) as city_location,
    lat,
    long,
    geo_location
from vk_data.resources.us_cities city
)

-- clean up supplier
, supplier_sanitized as (
select
    supplier_id,
    supplier_name,
    supplier_city || ', ' || supplier_state as supplier_location,
    trim(upper(supplier_city)) as supplier_city,
    trim(upper(supplier_state)) as supplier_state
from vk_data.suppliers.supplier_info 
) 

-- get supplier geo_location
, supplier_geography as (
select
    s.supplier_id,
    s.supplier_name,
    s.supplier_location,
    us_cities.geo_location
from supplier_sanitized s 
left join us_cities_sanitized us_cities
    on s.supplier_city = us_cities.city_name and s.supplier_state = us_cities.state_abbr
)

-- self join to get back up supplier and calculate distance
, supplier_distance_miles as (
select
    s1.supplier_id,
    s1.supplier_name, 
    s1.supplier_location as location_main, 
    s2.supplier_location as location_backup,
    st_distance(s1.geo_location, s2.geo_location) as distance_measure, 
    rank() over (partition by s1.supplier_id order by distance_measure asc) as rk
from supplier_geography s1
join supplier_geography s2 
where s1.supplier_id != s2.supplier_id
)

select
    supplier_id,
    supplier_name,
    location_main,
    location_backup, 
    round(distance_measure / 1609) as travel_miles
from supplier_distance_miles 
where rk = 1 
```
