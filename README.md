# SQL-QUERIES
## Table of Contents
- [Query-1](#Query-1)
- [Query-2](#Query-2)
- [Query-3](#Query-3)
- [Query-4](#Query-4)
- [Query-5](#Query-5)
- [Query-6](#Query-6)
- [Query-7](#Query-7)
- [Query-8](#Query-8)
- [Query-9](#Query-9)
- [Query-10](#Query-10)
- [Query-11](#Query-11)
- [Query-12](#Query-12)
- [Query-13](#Query-13)

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

## Query-6
### PROBLEM STATEMENT
- You are given a table having the marks of one student in every test. 
- You have to output the tests in which the student has improved his performance. 
- For a student to improve his performance he has to score more than the previous test.
- Provide 2 solutions, one including the first test score and second excluding it.
```
drop table if exists  student_tests;
create table student_tests
(
	test_id		int,
	marks		int
);
insert into student_tests values(100, 55);
insert into student_tests values(101, 55);
insert into student_tests values(102, 60);
insert into student_tests values(103, 58);
insert into student_tests values(104, 40);
insert into student_tests values(105, 50);

select * from student_tests;
```

![image](https://github.com/user-attachments/assets/9d3658af-89dc-4a82-9e68-de0f595a2ab1)   ![image](https://github.com/user-attachments/assets/0e881001-56d6-4b78-9588-b71d7b0bfda1)
 

### SOLUTION

```
--- First Output
select test_id, marks
from (
select test_id, marks, lag(marks,1,0) over(order by test_id) as privious_marks
from student_tests) x
where x.marks > x.privious_marks

-- second output
select test_id, marks
from (
select test_id, marks, lag(marks,1,marks) over(order by test_id) as privious_marks
from student_tests) x
where x.marks > x.privious_marks

```

## Query-7

### PROBLEM STATEMENT
- In the given input table DAY_INDICATOR field indicates the day of the week with the first character being Monday, followed by Tuesday and so on.
- Write a query to filter the dates column to showcase only those days where day_indicator character for that day of the week is 1
- 
![image](https://github.com/user-attachments/assets/5e0d6519-7438-4e5a-a324-eb62f2c70b3f)   ![image](https://github.com/user-attachments/assets/de6bce6c-d163-4f69-905c-fc373c81969b)


```
-- Micrososft SQL Server
drop table if exists Day_Indicator;
create table Day_Indicator
(
	Product_ID 		varchar(10),	
	Day_Indicator 	varchar(7),
	Dates			date
);
insert into Day_Indicator values ('AP755', '1010101', CONVERT(DATE,'04-Mar-2024', 102));
insert into Day_Indicator values ('AP755', '1010101', CONVERT(DATE,'05-Mar-2024', 102));
insert into Day_Indicator values ('AP755', '1010101', CONVERT(DATE,'06-Mar-2024', 102));
insert into Day_Indicator values ('AP755', '1010101', CONVERT(DATE,'07-Mar-2024', 102));
insert into Day_Indicator values ('AP755', '1010101', CONVERT(DATE,'08-Mar-2024', 102));
insert into Day_Indicator values ('AP755', '1010101', CONVERT(DATE,'09-Mar-2024', 102));
insert into Day_Indicator values ('AP755', '1010101', CONVERT(DATE,'10-Mar-2024', 102));
insert into Day_Indicator values ('XQ802', '1000110', CONVERT(DATE,'04-Mar-2024', 102));
insert into Day_Indicator values ('XQ802', '1000110', CONVERT(DATE,'05-Mar-2024', 102));
insert into Day_Indicator values ('XQ802', '1000110', CONVERT(DATE,'06-Mar-2024', 102));
insert into Day_Indicator values ('XQ802', '1000110', CONVERT(DATE,'07-Mar-2024', 102));
insert into Day_Indicator values ('XQ802', '1000110', CONVERT(DATE,'08-Mar-2024', 102));
insert into Day_Indicator values ('XQ802', '1000110', CONVERT(DATE,'09-Mar-2024', 102));
insert into Day_Indicator values ('XQ802', '1000110', CONVERT(DATE,'10-Mar-2024', 102));
```

### SOLUTION
```
SET DateFirst 1

select Product_ID, Day_Indicator, Dates
from (
      select *,
      case when SUBSTRING( Day_Indicator,DATEPART ( WEEKDAY,Dates),1) =1 then 1 else 0 end as x
      from Day_Indicator) as t
where x= 1
```
## Query-8

### PROBLEM STATEMENT 
- In the given input table, there are rows with missing JOB_ROLE values. Write a query to fill in those blank fields with appropriate values.
- Assume row_id is always in sequence and job_role field is populated only for the first skill.
- 
![image](https://github.com/user-attachments/assets/29b08f3c-fd34-4e95-ad59-446aee5be58a)

```
drop table if exists job_skills;
create table job_skills
(
	row_id		int,
	job_role	varchar(20),
	skills		varchar(20)
);
insert into job_skills values (1, 'Data Engineer', 'SQL');
insert into job_skills values (2, null, 'Python');
insert into job_skills values (3, null, 'AWS');
insert into job_skills values (4, null, 'Snowflake');
insert into job_skills values (5, null, 'Apache Spark');
insert into job_skills values (6, 'Web Developer', 'Java');
insert into job_skills values (7, null, 'HTML');
insert into job_skills values (8, null, 'CSS');
insert into job_skills values (9, 'Data Scientist', 'Python');
insert into job_skills values (10, null, 'Machine Learning');
insert into job_skills values (11, null, 'Deep Learning');
insert into job_skills values (12, null, 'Tableau');

select * from job_skills;

```

### SOLUTION 

```
with cte as (
			select *, sum (case when job_role is null then 0 else 1 end) over( order by row_id) as job_segment
			from job_skills)

select row_id, FIRST_VALUE(job_role) over(partition by job_segment order by row_id) as job_role, skills
from cte
```

## Query-9
### PROBLEM STATEMENT
- Write an sql query to merge products per customer for each day as shown in expected output.
- 
![image](https://github.com/user-attachments/assets/ed386618-1719-4b70-a111-1d806a74adc8)       ![image](https://github.com/user-attachments/assets/2cc480b6-e461-44df-ba84-6ff10bc335a8)


### SOLUTION
```
select dates, cast(product_id as varchar) as products
from orders
union
select dates, STRING_AGG( product_id, ',') as products
from orders
group by dates,customer_id
order by dates, products
```

## Query-10
### PROBLEM STATEMENT

![Video_Q10_Problem_Statement](https://github.com/user-attachments/assets/268a7bb8-49b0-4f89-a411-7c80c3bb9c05)

### SOLUTION

```
with t as (select client,auto,repair_date, MAX(case when indicator= 'level' then value end) as level,MAX( case when indicator ='velocity' then value end) as velocity
from auto_repair
group by client,auto,repair_date
),
t1 as (
select velocity,level, count (*) as count1
from t
group by level,velocity
)
select velocity, COALESCE(Max(case when level= 'good' then count1 end),0) as good,
COALESCE(Max(case when level= 'regular' then count1 end),0) as regular,
COALESCE(Max(case when level= 'wrong' then count1 end),0) as wrong
from t1
group by velocity
order by velocity
```

## Query-11
### PROBLEM STATEMENT
![image](https://github.com/user-attachments/assets/0d27cd98-d970-447f-9627-6d3bdc11f9ae)

### SOLUTION
```
WITH CTE AS (
    SELECT 
        hotel,
        year, 
        rating, 
        CAST(ROUND(AVG(rating) OVER (
            PARTITION BY hotel 
            ORDER BY year 
            ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
        ), 2) AS DECIMAL(10,2)) AS overall_hotel_avg
    FROM hotel_ratings
),
t as (SELECT 
    hotel,
    year,
    rating,
    overall_hotel_avg,
    ABS(rating - overall_hotel_avg) AS rating_deviation, RANK() over (partition by hotel order by ABS(rating - overall_hotel_avg) desc) as rnk
FROM CTE
)
select hotel, year, rating
from t 
where rnk >1
order by hotel,year
```

## Query-12
### PROBLEM STATEMENT
![image](https://github.com/user-attachments/assets/917f3a63-fdb1-4f20-96df-b7ba4769e9bf)

### SOLUTION
```
WITH cte AS (
    -- Step 1: Assign team numbers to direct reports of Elon
    SELECT 
        employee, 
        manager,
        ROW_NUMBER() OVER (ORDER BY employee) AS team
    FROM company 
    WHERE manager = (SELECT employee FROM company WHERE manager IS NULL) 

    UNION ALL

    -- Step 2: Recursively assign employees to the same team as their manager
    SELECT 
        e.employee, 
        c.employee AS manager,
        c.team AS team
    FROM cte c
    INNER JOIN company e ON c.employee = e.manager
)
SELECT 
    'Team ' + CAST(team AS VARCHAR) AS TEAMS, 
    (SELECT employee FROM company WHERE manager IS NULL) +',' + STRING_AGG(employee, ', ') AS MEMBERS
FROM cte
GROUP BY team
ORDER BY team;

``

## Query-13
### PROBLEM STATEMENT
![image](https://github.com/user-attachments/assets/42277cd7-c5dc-4543-b90f-8d86e11d3447)

### SOLUTION
```
with cte as( 
SELECT manager,count(*) as no_of_employees
FROM EMPLOYEE_MANAGERS
group by manager
)
select em.name as Manager, ct.no_of_employees
from employee_managers em
join cte ct on ct.manager= em.id
order by ct.no_of_employees desc
```
