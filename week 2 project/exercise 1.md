# Exercise #1
``` sql
-- clean up us cities
with us_cities_sanitized as (
select 
    -- perform string formatting in cte instead of join
    upper(trim(state_abbr)) as state_abbr, 
    lower(trim(city_name)) as city_name,
    geo_location
from vk_data.resources.us_cities

)

-- get chicago location
, chicago as (
select geo_location
from us_cities_sanitized
where city_name = 'chicago' and state_abbr = 'IL'
)

-- get gary location
, gary as (
select geo_location
from us_cities_sanitized
where city_name = 'gary' and state_abbr = 'IN'
)

-- clean up customers
, customer_info_sanitized as (
select 
    c.customer_id,
    first_name || ' ' || last_name as customer_name,
    lower(trim(ca.customer_city)) as customer_city,
    upper(rtrim(ltrim(ca.customer_state))) as customer_state
from vk_data.customers.customer_address ca
inner join vk_data.customers.customer_data c on ca.customer_id = c.customer_id
-- include where statement here to limit query size
where customer_state in ('CA', 'KY', 'TX')
)

-- replace subquery with cte
, customer_active_food_pref as (
select 
    customer_id,
    count(*) as food_pref_count
from vk_data.customers.customer_survey
where is_active = true
group by 1
) 

select 
    c.customer_name,
    c.customer_city,
    c.customer_state,
    s.food_pref_count,
    (st_distance(us.geo_location, chicago.geo_location) / 1609)::int as chicago_distance_miles,
    (st_distance(us.geo_location, gary.geo_location) / 1609)::int as gary_distance_miles 
from customer_info_sanitized c
left join us_cities_sanitized us on c.customer_state = us.state_abbr and c.customer_city = us.city_name
inner join customer_active_food_pref s on c.customer_id = s.customer_id
cross join chicago 
cross join gary
-- replace multiple or statements with ilike any
where us.city_name ilike any ('%concord%', '%georgetown%', '%ashland%','%oakland%','%pleasant hill%', '%arlington%', '%brownsville%')
``` 
