### SQL avansat - Partea de Teorie

Fundamente din SQL:

CRUD Operations -> Create, Read, Update, Delete

Data Types -> INTEGER, VARCHAR, DATE, BOOLEAN + more

CONSTRAINTS:

-   PRIMARY KEY: ensures the uniqueness of a column's values

-   FOREIGN KEY: maintains referential integrity between two tables

-   NOT NULL: ensures a column must have a value

-   UNIQUE: ensures unique values in a column

INDEXES -> database objects used to speed up data retrieval

AGGREGATE FUNCTIONS -> SUM, AVG, COUNT, MIN, MAX

GROUP BY -> group data based on specific columns

ORDER BY (column)

SUBQUERY -> a query embedded within another query

```sql
SELECT *
FROM table
WHERE ceva = (
    SELECT *
    FROM table
    ETC..
);
```

VIEWS -> virtual tables created by queries and can be used like regular tables 

    -> simplify complex queries and restrict access to sensitive data

```sql
CREATE VIEW view_name AS
select
from
where...;
```

TRANSACTIONS -> sequences of one or more SQL operations treated as a single unit

    -> These operations either ALL SUCCEED or ALL FAIL

- reffered to ACID properties: Atomicity, Consistency, Isolation, Durability

    -> reliability & corectness of database operations

```SQL
BEGIN TRANSACTION;
--SQL STATEMENTS (INSERT, UPDATE, DELETE, ETC)
COMMIT; -- TO SAVE CHANGES PERM
ROLLBACK; -- TO UNDO THE CHANGES
```

NORMALIZATION -> organizing data in a database to eliminate redundancy and improve data quality and integrity

    -> defining relationships between tables

STORED PROCEDURES -> precompiled SQL code that can be stored and executed on the database server 

    -> improving performance & security

TRIGGERS -> database objects that automatically execute in response to specific events like some mods in the data

    -> data consistency & integrity

```sql
CREATE [OR REPLACE] TRIGGER name
{BEFORE|AFTER|INSTEAD OF} {INSERT|UPDATE|DELETE}
ON table
[FOR EACH ROW]
BEGIN
--SQL STATEMENTS TO BE EXECUTED WHEN THE TRIGGER IS ACTIVATED
END;
```

### SQL Joins

-    ANTI JOIN -> LEFT JOIN sa zic a.id = b.id cu a tabela din stanga. Si ma uit sa vad ce NU este in join, adica unde am b.id = NULL

Cum functioneaza baza de date:

1. FROM & JOIN -> se colecteaza datele din tabele

2. GROUP BY -> se impart randurile in buckets bazate pe coloanele care nu sunt in functia de agregare, dar sunt in select

3. SUM(AMOUNT) -> se calculeaza totalul pentru fiecare bucket (grup)

4. SELECT (INCLUSIV CASE) -> abia acum se uita la rezultat si aplica regulile din CASE

```sql
with febProds as (
	select
		p.Product,
        s.Amount,
        s.SaleDate
	from products p 
    left join sales s on p.PID = s.PID and s.SaleDate = '2022-2-1'
)
select
	Product,
	sum(Amount) as 'total',
    case
		when sum(Amount) > 0 then 'yes'
        else 'no'
	end as 'shipped'
from febProds
group by Product;

```

Deci daca am un CASE in select nu trebuie bagat in GROUP BY

### Window Functions

```sql
function() over (
    partition by column1
    order by column2 
) as SOMETHING
```

Cumulative Sales: 

```sql
select
    date_id,
    sales,
    sum(sales) over (
        order by date_id asc
        rows unbounded preceding
    ) as cum_sales
from orders;
```

3-Day Average:

```sql
select
    date_id,
    sales,
    avg(sales) over (
        order by date_id asc
        rows between 2 preceding
        and current row
    ) as 3dayAvg
from orders;
```


1. Normalizarea 1NF-3NF

2. Denormalizarea pentru Big Data

3. Star Schema vs Snowflake Schema vs Medallion Schema

4. Concepte de SQL pentru Big Data

5. Cum functioneaza un query in spate si cum poate fi optimizat ? Execution Plan ce este ?

Leetcode SQL: 1581, 197