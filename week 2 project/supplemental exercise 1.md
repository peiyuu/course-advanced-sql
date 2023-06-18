# Supplmental Exercise #1
```sql
-- get recipe ingredients
with ingredients as (
 select 
    recipe_id,
    recipe_name,
    flat_ingredients.index,
    trim(upper(replace(flat_ingredients.value, '"', ''))) as ingredient
from vk_data.chefs.recipe
, table(flatten(ingredients)) as flat_ingredients
)

-- get ingredients nutrition info
, ingredients_nutrition as (
select 
    trim(upper(replace(substring(ingredient_name, 1, charindex(',', ingredient_name)), ',', ''))) as ingredient_name,
    min(id) as first_record,
    max(calories) as calories,
    max(total_fat) as total_fat
from vk_data.resources.nutrition 
group by 1) 

-- filter for specific cookie recipes
, cookies_nutrition as (
select 
    recipe_id,
    recipe_name,
    ingredient,
    first_record,
    calories,
    total_fat
from ingredients i
left join ingredients_nutrition n on i.ingredient = n.ingredient_name
where recipe_name in ('birthday cookie', 'a perfect sugar cookie','honey oatmeal raisin cookies', 'frosted lemon cookies','snickerdoodles cinnamon cookies')
)

select
    recipe_name, 
    sum(c.calories) as total_calories, 
    sum(cast(replace(c.total_fat, 'g', '') as int)) as total_fat
from cookies_nutrition c
inner join vk_data.resources.nutrition n on r.first_record = n.id
group by 1 
order by 1
```
