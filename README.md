# SQL-QUERIES

## Q-1 
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
OR (custom1 <> custom3 AND custom2 <>custom4)```
