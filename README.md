# SQL-QUERIES
## Table of Contents
- [Query-1](#Query-1)
- [Query-2](#Query-2)

## Query-1 
### PROBLEM STATEMENT
- For pairs of brands in the same year (e.g. apple/samsung/2020 and samsung/apple/2020) 
    - if custom1 = custom3 and custom2 = custom4 : then keep only one pair

- For pairs of brands in the same year 
    - if custom1 != custom3 OR custom2 != custom4 : then keep both pairs

- For brands that do not have pairs in the same year : keep those rows as well
*/


``` DROP TABLE IF EXISTS brands;
CREATE TABLE brands 
(
    brand1      VARCHAR(20),
    brand2      VARCHAR(20),
    year        INT,
    custom1     INT,
    custom2     INT,
    custom3     INT,
    custom4     INT
);
INSERT INTO brands VALUES ('apple', 'samsung', 2020, 1, 2, 1, 2);
INSERT INTO brands VALUES ('samsung', 'apple', 2020, 1, 2, 1, 2);
INSERT INTO brands VALUES ('apple', 'samsung', 2021, 1, 2, 5, 3);
INSERT INTO brands VALUES ('samsung', 'apple', 2021, 5, 3, 1, 2);
INSERT INTO brands VALUES ('google', NULL, 2020, 5, 9, NULL, NULL);
INSERT INTO brands VALUES ('oneplus', 'nothing', 2020, 5, 9, 6, 3);

SELECT * FROM brands;
```


### SOLUTION

```
With CTE as (
SELECT *,
CASE when brand1 <brand2 then CONCAT(brand1,brand2,year) 
     else CONCAT(brand2,brand1,year) end as pair_ID
FROM brands),
    cte_rn as (
     select *, ROW_NUMBER() over(PARTITION BY PAIR_ID ORDER BY PAIR_ID) AS RN
	 FROM CTE)

SELECT *
FROM cte_rn
WHERE RN= 1 
OR (custom1 <> custom3 AND custom2 <>custom4)
```

## Query-2
### PROBLEM STATEMENT
- A ski resort company is planning to construct a new ski slope using a pre-existing network of 
mountain huts and trails between them. A new slope has to begin at one of the mountain huts, 
have a middle station at another hut connected with the first one by a direct trail, and end at 
the third mountain hut which is also connected by a direct trail to the second hut. The altitude 
of the three huts chosen for constructing the ski slope has to be strictly decreasing.
- You are given two SQL tables, mountain_huts and trails, with the following structure:
```
drop table if exists mountain_huts;
create table mountain_huts 
(
	id 			integer not null unique,
	name 		varchar(40) not null unique,
	altitude 	integer not null
);
insert into mountain_huts values (1, 'Dakonat', 1900);
insert into mountain_huts values (2, 'Natisa', 2100);
insert into mountain_huts values (3, 'Gajantut', 1600);
insert into mountain_huts values (4, 'Rifat', 782);
insert into mountain_huts values (5, 'Tupur', 1370);

drop table if exists trails;
create table trails 
(
	hut1 		integer not null,
	hut2 		integer not null
);
insert into trails values (1, 3);
insert into trails values (3, 2);
insert into trails values (3, 5);
insert into trails values (4, 5);
insert into trails values (1, 5);

select * from mountain_huts;
select * from trails;
``` 
- Each entry in the table trails represents a direct connection between huts with IDs hut1 and 
hut2. Note that all trails are bidirectional.
- Create a query that finds all triplets(startpt,middlept,endpt) representing the mountain huts 
that may be used for construction of a ski slope.
- Output returned by the query can be ordered in any way

-- Assume that:
- there is no trail going from a hut back to itself;
-  for every two huts there is at most one direct trail connecting them;
-  each hut from table trails occurs in table mountain_huts

### SOLUTION
```
with cte_trail1 as (
				Select t.hut1 as start_hut,mh.name as start_hut_name, mh.altitude as start_hut_altitude,
				t.hut2 as end_hut
				from trails t
				join mountain_huts mh on t.hut1 = mh.id),
     cte_trail2 as (
				select start_hut,start_hut_name,start_hut_altitude,end_hut,mh.name as end_hut_name, mh.altitude as end_hut_altitude
				from cte_trail1 ct1
				join mountain_huts mh on ct1.end_hut= mh.id),

     cte_final as (
				select  case when start_hut_altitude > end_hut_altitude then start_hut else end_hut end as start_hut,
				case when start_hut_altitude > end_hut_altitude then start_hut_name else end_hut_name end as start_hut_name,
				case when start_hut_altitude > end_hut_altitude then end_hut else start_hut end as end_hut,
				case when start_hut_altitude > end_hut_altitude then end_hut_name else start_hut_name end as end_hut_name

				from cte_trail2)

select cf1.start_hut_name as startpoint, cf1.end_hut_name as middlepoint , cf2.end_hut_name as endpoint
from cte_final cf1
join cte_final cf2 on cf1.end_hut = cf2.start_hut
```
