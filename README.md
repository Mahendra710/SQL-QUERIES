# SQL-QUERIES
## Table of Contents
- [Query-1](#Query-1)
- [Query-2](#Query-2)
- [Query-3](#Query-3)
- [Query-4](#Query-4)
- [Query-5](#Query-5)

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

## Query-3
### STATEMENT 
- Write a sql query to return the footer values from input table, meaning all the last non null values from each field as shown in expected output.								

![image](https://github.com/user-attachments/assets/bd106eb1-df04-48af-9b01-154514f0a65a)

```
DROP TABLE IF EXISTS FOOTER;
CREATE TABLE FOOTER 
(
	id 			INT PRIMARY KEY,
	car 		VARCHAR(20), 
	length 		INT, 
	width 		INT, 
	height 		INT
);

INSERT INTO FOOTER VALUES (1, 'Hyundai Tucson', 15, 6, NULL);
INSERT INTO FOOTER VALUES (2, NULL, NULL, NULL, 20);
INSERT INTO FOOTER VALUES (3, NULL, 12, 8, 15);
INSERT INTO FOOTER VALUES (4, 'Toyota Rav4', NULL, 15, NULL);
INSERT INTO FOOTER VALUES (5, 'Kia Sportage', NULL, NULL, 18); 

SELECT * FROM FOOTER;
```
### SOLUTION 
```
select * from (
			select TOP 1 car
			from FOOTER 
			where car is not null 
			order by id desc) car
cross join (select TOP 1 length
			from FOOTER 
			where length is not null 
			order by id desc) length
cross join (select TOP 1 width
			from FOOTER 
			where width is not null 
			order by id desc) width
cross join (select TOP 1 height
			from FOOTER 
			where height is not null 
			order by id desc) height
```

## Query-4
### PROBLEM STATEMENT
```
drop table if exists Q4_data;
create table Q4_data
(
	id			int,
	name		varchar(20),
	location	varchar(20)
);
insert into Q4_data values(1,null,null);
insert into Q4_data values(2,'David',null);
insert into Q4_data values(3,null,'London');
insert into Q4_data values(4,null,null);
insert into Q4_data values(5,'David',null);

select * from Q4_data;
```
- Derive expected output
- EXPECTED OUTPUT - 1 \
 ![image](https://github.com/user-attachments/assets/86e2068c-7544-4193-ad05-513b91c2f221)

- EXPECTED OUTPUT - 2 \  
![image](https://github.com/user-attachments/assets/58be4d16-4d5f-4fae-9282-00992178facc)


### SOLUTION
```
--- First expected Output 
select min(id) as ID ,min(name) as Name ,min(location) as Location
from Q4_data

---- Second Expected Output

select max(id) as ID , max(name) as Name , max(location) as Location
from Q4_data
```

## Query-5
### PROBLEM STATEMENT
- Using the given Salary, Income and Deduction tables, first write an sql query to populate the Emp_Transaction table as shown below and then generate a salary report as shown. 
- ![image](https://github.com/user-attachments/assets/5b823671-0975-4246-b1f9-09066508046e) 
- ![image](https://github.com/user-attachments/assets/782fdc30-f249-4eef-b94c-deda2a74f992) 
- ![image](https://github.com/user-attachments/assets/419c0eb7-8121-4618-a9db-65c31d7ad04b)
```
drop table if exists salary;
create table salary
(
	emp_id		int,
	emp_name	varchar(30),
	base_salary	int
);
insert into salary values(1, 'Rohan', 5000);
insert into salary values(2, 'Alex', 6000);
insert into salary values(3, 'Maryam', 7000);


drop table if exists income;
create table income
(
	id			int,
	income		varchar(20),
	percentage	int
);
insert into income values(1,'Basic', 100);
insert into income values(2,'Allowance', 4);
insert into income values(3,'Others', 6);


drop table if exists deduction;
create table deduction
(
	id			int,
	deduction	varchar(20),
	percentage	int
);
insert into deduction values(1,'Insurance', 5);
insert into deduction values(2,'Health', 6);
insert into deduction values(3,'House', 4);


drop table if exists emp_transaction;
create table emp_transaction
(
	emp_id		int,
	emp_name	varchar(50),
	trns_type	varchar(20),
	amount		numeric
);
select * from salary;
select * from income;
select * from deduction;
select * from emp_transaction;
```

- 
![image](https://github.com/user-attachments/assets/4bf3d208-4acd-4818-b7b6-69714b1d03e5)

-
![image](https://github.com/user-attachments/assets/e8666999-7ac7-4039-845c-30ead5fbc9fa)

### SOLUTION 
```
insert into emp_transaction
SELECT 
    s.emp_id, 
    s.emp_name, 
    x.trans_type,
    (s.base_salary * x.percentage) / 100 AS amount
FROM salary s
CROSS JOIN (
    SELECT deduction AS trans_type, percentage FROM deduction
    UNION 
    SELECT income AS trans_type, percentage FROM income
) x;

--- using case when
select emp_name, sum (case when trns_type = 'Allowance' then amount else 0 end ) as Allowance,
 sum (case when trns_type = 'Basic' then amount else 0 end ) as Basic,
 sum (case when trns_type = 'Others' then amount else 0 end ) as Others,
 SUM(CASE WHEN trns_type IN ('Basic', 'Allowance', 'Others') THEN amount ELSE 0 END) AS Gross,
 sum (case when trns_type = 'Insurance' then amount else 0 end ) as Insurance,
 sum (case when trns_type = 'Health' then amount else 0 end ) as Health,
 sum (case when trns_type = 'House' then amount else 0 end ) as House,
 sum(case when trns_type in ('Insurance','Health','House') then amount else 0 end ) as total_deduction,
 SUM(CASE WHEN trns_type IN ('Basic', 'Allowance', 'Others') THEN amount ELSE 0 END) 
    - SUM(CASE WHEN trns_type IN ('Insurance', 'Health', 'House') THEN amount ELSE 0 END) AS Net_Salary
from emp_transaction
group by emp_id, emp_name

---- Using pivot

SELECT emp_name, 
       Allowance, 
       Basic, 
       Others, 
       (Basic + Allowance + Others) AS Gross,  -- Gross Calculation
       Insurance, 
       Health, 
       House, 
       (Insurance + Health + House) AS total_deduction,  -- Total Deduction Calculation
       (Basic + Allowance + Others) - (Insurance + Health + House) AS Net_Salary  -- Net Salary Calculation
FROM (
    -- Source Data
    SELECT emp_name, trns_type, amount
    FROM emp_transaction
) src
PIVOT (
    SUM(amount) 
    FOR trns_type IN (Basic, Allowance, Others, Insurance, Health, House)
) AS pvt;

```
