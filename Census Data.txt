-- View first table
select * from practice..Data1$;

-- View second table
select * from practice..Sheet1$;

-- Check the number of rows in both tables;
select count(*) from practice..Data1$;  --640 rows

select count(*) from practice..Sheet1$;  --640 rows

--To find out total population of India
select sum(population) from practice..Sheet1$;  --1210854977

-- To find average growth rate of country
select round(AVG(Growth)*100,1) from practice..Data1$;  --19.2%

-- To find average Sex-Ratio of country
select avg(Sex_Ratio) from practice..Data1$;  --945.43

--To find average literacy in country
select AVG(Literacy) from practice..Data1$;  --72.3

-- Top 3 state showing highest growth ratio
select top 3 State,avg(Growth)*100 as avg_growth from practice..Data1$ 
group by state 
order by avg_growth desc ;

--bottom 3 state showing lowest sex ratio
select top 3 state,avg(Sex_Ratio)  as avg_sex_ratio from practice..Data1$
group by State
order by avg_sex_ratio;

-- top and bottom 3 states in literacy state
drop table if exists #topstates;
create table #topstates
( state nvarchar(255),
  topstate float

  )

insert into #topstates
select state,round(avg(literacy),0) as avg_literacy_ratio from practice..Data1$
group by state order by avg_literacy_ratio desc;

select top 3 * from #topstates order by #topstates.topstate desc;


drop table if exists #bottomstates;
create table #bottomstates
( state nvarchar(255),
  bottomstate float

  )

insert into #bottomstates
select state,round(avg(literacy),0) avg_literacy_ratio from practice..Data1$
group by state order by avg_literacy_ratio desc;

select top 3 * from #bottomstates order by #bottomstates.bottomstate asc;

select * from (
select top 3 * from #topstates order by #topstates.topstate desc) a
union
select * from (
select top 3 * from #bottomstates order by #bottomstates.bottomstate asc) b;

-- Find total males and females
select d.State, sum(Males) as Total_Male, sum(Females) as Total_Female from
(select c.State, c.District , round(c.Population/(c.sex_ratio+1),0) as Males , round((c.Population * c.sex_ratio)/(c.sex_ratio+1),0) as Females
from(
select a.State, a.District ,a.Sex_Ratio/1000 as sex_ratio, b.Population from practice..Data1$ a
inner join practice..Sheet1$ b on a.District=b.District) c) d
group by d.State;

--Find total literate and illiterate people
select d.State, SUM(d.Literate_people) as total_literate_population, SUM(d.illiterate_People) as Total_illiterate_population from (
select c.State,c.District, round((c.Literacy_ratio*c.Population),0) as Literate_People,
round((1-c.Literacy_ratio)*c.Population,0) as Illiterate_People
from(
select a.State, a.District ,a.Literacy/100 as Literacy_ratio, b.Population from practice..Data1$ a
inner join practice..Sheet1$ b on a.District=b.District) c) d
group by d.State;

--Find top 3 districts from each state with highest literacy rate
select a.* from
(select district,state,literacy,rank() over(partition by state order by literacy desc) rnk from practice..Data1$) a
where a.rnk in (1,2,3) order by state;

-- Find population in previous census 
select sum(e.Current_census_population) as Current_census_population, SUM(e.previous_census_population) as previous_census_population  from(
select d.State,SUM(Current_census_population) as Current_census_population, SUM(Previous_census_population) as previous_census_population from (
select c.State,c.District,c.Population as Current_census_population, round(c.Population/(1+c.Growth),0) as Previous_census_population from(
select a.State, a.District ,a.Growth, b.Population from practice..Data1$ a
inner join practice..Sheet1$ b on a.District=b.District)c)d
group by d.State)e


--Find population density of each state
select a.State, round(sum(a.Population/a.Area_km2),1) as Population_Density from (
select State,Area_km2,Population from practice..Sheet1$)a
where a.State<>'NULL'
group by a.State
order by Population_Density desc;