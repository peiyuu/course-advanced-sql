# Exercise #1
```sql
-- created customer_info cte to identify eligible customers and get the geo_location of customers
-- joined supplier_info to us_cities table to find the geo_location of suppliers
-- cross joined both tables and used st_distance to calculate the distance between customer address and supplier
-- divided distance by 1609 to conver into miles. Used a window function to determine the closest supplier

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

# Exercise #2 - use of pivot & flatten
```sql
-- reused customer_info cte from exercise 1 to identify eligible customers based on location
-- joined customer_info, customer_survey, and recipe_tags together to find customer preferences from survey
-- used row_number to find top 3 preferences
-- used pivot function to pivot rows into separate columns 
-- flattened json values from the chefs_recipes table to get a list of tags and their corresponding recipes
-- used min function to return one recipe
-- joined pivot and recipe tables to return a list of customers, food preferences, and suggested recipe

with customer_info as (
select
    c.customer_id
    , c.first_name
    , c.last_name
    , c.email
from VK_DATA.CUSTOMERS.CUSTOMER_DATA c
inner join VK_DATA.CUSTOMERS.CUSTOMER_ADDRESS a
    on c.customer_id = a.customer_id
inner join VK_DATA.RESOURCES.US_CITIES cities
    on trim(lower(a.customer_state)) = lower(cities.state_abbr) 
    and trim(lower(a.customer_city)) = lower(cities.city_name)
where a.customer_city is not null and a.customer_state is not null
)
, preferences as (
select 
    c.customer_id
    , c.email
    , c.first_name
    , trim(r.tag_property) as tag 
    , row_number() over (partition by c.customer_id order by tag) as rk
from customer_info c
inner join VK_DATA.CUSTOMERS.CUSTOMER_SURVEY s
    on c.customer_id = s.customer_id
left join VK_DATA.RESOURCES.RECIPE_TAGS  r 
    on s.tag_id = r.tag_id
where is_active ='TRUE'
) 

, pivot as (
select 
   *
from preferences
pivot( max(tag)
    for rk in (1,2,3))
    as pivot_values(customer_id, email, first_name, food_pref_1, food_pref_2, food_pref_3)
)

, one_recipe as (
select 
    min(recipe_name) as suggested_recipe
    , ltrim(trim(flat_tag_list.value, '"'), ' ') as tag
from VK_DATA.CHEFS.RECIPE
, table(flatten(tag_list)) as flat_tag_list
group by 2
)

select
 p.customer_id
 , p.email
 , p.first_name
 , p.food_pref_1
 , p.food_pref_2
 , p.food_pref_3
 , r.suggested_Recipe
from pivot p
join one_recipe r on p.food_pref_1 = r.tag
order by email 
```
