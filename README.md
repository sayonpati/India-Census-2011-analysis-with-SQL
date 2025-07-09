# India Census 2011 analysis with SQL

---- Find the total Population of India

Select FORMAT(sum(population),'N') as Total_Population from population;


---- Find the average growth rate in India as compared to the previous census

select round(avg(growth),2) Average_growth from literacy;


---- Find the average growth rate for each state as compared to the previous census

select state, round(avg(growth),2) as Average_growth from literacy
group by state;


---- Find the details of Jharkhand and Bihar

select * from literacy
join population on population.district = literacy.district
where literacy.state in ('Jharkhand','Bihar');


---- Find the previous and current population density for each state

select a.state,
round((population*(100/(100+growth)))/a.area,2) as previous_census_population_density,
current_population_density 
from (
  select c.district, c.state, c.growth, c.area, c.population,
  round(c.population/c.area,2) as current_population_density 
  from (
    select l.district, l.state, growth, p.area_km2 as area, p.population
    from literacy l
    join population p on p.district = l.district
  ) c
) a;


---- Find top 3 districts from each state with highest literacy rate

-- Using Windows Function

select a.* from (
  select district, state, literacy,
  rank() over(partition by state order by literacy desc) as districtwise_literacy
  from literacy
) a
where a.districtwise_literacy in (1,2,3)
order by state;

-- Using Common Table Expression (CTE) and Windows Function

with Top_3 as (
  select district, state, literacy,
  rank() over(partition by state order by literacy desc) as districtwise_growth
  from literacy
  order by state, districtwise_growth
)
select * from Top_3
where districtwise_growth <= 3;

---- Find the top 3 districts in each state with the highest growth rate

--- Using Windows Function
select a.* from (
  select district, state, growth,
  rank() over(partition by state order by growth desc) as districtwise_growth
  from literacy
) a
where a.districtwise_growth in (1,2,3)
order by state;


--- Using Common Table Expression and Windows Function (CTE)
with Top_3 as (
  select district, state, growth,
  rank() over(partition by state order by growth desc) as districtwise_growth
  from literacy
  order by state, districtwise_growth
)
select * from top_3
where districtwise_growth <= 3;


---- Find the population in the previous census

select e.state,
format(sum(e.previous_census_population),'N') as statewide_previous_census,
format(sum(e.current_sensus_population),'N') as statewide_current_sensus
from (
  Select c.district, c.state, round(d*(100/(100+c.growth)),0) as previous_census_population,
  d as current_sensus_population
  from (
    select l.district, l.state, l.growth, p.population as d
    from literacy as l
    join population p on p.district = l.district
  ) as c
) as e
group by e.state;


---- Find the total literate population for each state

select l.state,
format(sum(round(literacy/100 * population)),'N') as literate_population,
format(sum(round((1-literacy/100) * population)),'N') as illiterate_population
from literacy as l
join population p on p.district = l.district
group by l.state;

--- Using Nested Query

select c.state,
format(sum(literate_people),'N') as total_literate_pop,
format(sum(illiterate_people),'N') as total_illiterate_pop
from (
  select d.district, d.state,
  round(d.literacy_ratio * d.population, 0) as literate_people,
  round((1 - d.literacy_ratio) * d.population, 0) as illiterate_people
  from (
    select l.district, l.state, l.literacy / 100 as literacy_ratio, p.population
    from literacy l
    inner join population p on l.district = p.district
  ) d
) c
group by c.state;


---- Find the number of males and females for each state

select l.state,
format(round(sum(1000/(sex_ratio+1000)*population)),'N') as Males,
format((sum(sex_ratio/(sex_ratio+1000)*population)),'N') as Females
from literacy as l
join population as p on p.district = l.district
group by l.state;

--- Using Nested Query

select l.state, l.males, l.females 
from (
  select round(sum(1000/(sex_ratio+1000)*population)) as Males,
  round(sum(sex_ratio/(sex_ratio+1000)*population)) as Females
  from (
    select l.district, l.state, sex_ratio/1000, p.population
    from literacy as l
    join population as p on p.district = l.district
  ) as c
) as d
group by l.state;


---- Find the states with an average literacy ratio greater than 90

select state, round(avg(literacy),2) as average_literacy
from literacy
group by state
having average_literacy > 90
order by average_literacy;


---- Top 3 state showing highest growth ratio

select state, round(avg(growth),2) as growth_rate
from literacy
group by state
order by growth_rate desc
limit 3;

select * from (
  select state, round(avg(literacy),2) as literacy_rate
  from literacy
  group by state
  order by literacy_rate desc
  limit 3
) as top3;


---- States starting with the letter a

select distinct state from literacy
where state like 'a%' or state like '%d';


---- Find the number of females for the district with the highest literacy rate

select literacy.district, literacy.state,
round((sex_ratio/(sex_ratio+1000)*population)) as Females,
population
from literacy
join population on population.district = literacy.district
order by literacy desc
limit 1;
