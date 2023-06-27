# Exercise
``` sql
-- my optimization strategy is to limit the primary table we will be pulling from to only include necessary fields and filter for event and recipe_id fields. 
-- removed any order by's in the query
-- my query plan revealed that joins were the most costly nodes, so I removed the join/max functions and used rank instead to find the most viewed recipe by date 
-- in the final query, only one join is used. 

with event_info as (
select
     distinct a.session_id
    , a.event_timestamp
    , a.event_details
    , flat_event_details.key
    , flat_event_details.value as event_activity
from VK_DATA.EVENTS.WEBSITE_ACTIVITY a
, table(flatten(input => parse_json(event_details))) as flat_event_details
where key in ('event', 'recipe_id') 
) 

, daily_session_summary  as (
select 
    session_id
    -- convert timestamp to date 
    , event_timestamp::date as session_date
    , timediff(second, min(event_timestamp), max(event_timestamp)) as session_duration_seconds
    , sum(case when trim(event_activity) = 'search' then 1 else 0 end) as search_count
    , sum(case when trim(event_activity) = 'view_recipe' then 1 else 0 end) as view_count
from event_info e
group by 
    session_id
    , session_date
-- order by session_id
)

-- find most popular recipe
, daily_recipe_view_count as (
select
      event_timestamp::date as session_date
     , event_activity as recipe_id
     , rank() over (partition by session_date order by count(*) desc) as rk
from event_info
where key = 'recipe_id' 
group by
    session_date
    , event_activity
) 

-- find recipe id 
, most_viewed_recipe_id as (
select 
    session_date
    , recipe_id 
from daily_recipe_view_count
where rk = 1
)

-- create final table aggregates and left join recipe table to daily session summary
select 
    d.session_date
    , count(distinct session_id) as total_unique_event_sessions
    -- round to decimals
    , round(avg(session_duration_seconds), 2) as avg_session_length_seconds
    , round(div0(sum(search_count),sum(view_count)),2) as avg_search_counts
    , listagg(distinct recipe_id, ', ') within group (order by recipe_id asc) as recipe_id
from daily_session_summary d
-- use left join to get the recipe_id from the right table 
left join most_viewed_recipe_id m on d.session_date = m.session_date
group by d.session_date
-- order by d.session_date desc```
