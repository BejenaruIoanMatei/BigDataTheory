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




1. Normalizarea 1NF-3NF

2. Denormalizarea pentru Big Data

3. Star Schema vs Snowflake Schema vs Medallion Schema

4. Concepte de SQL pentru Big Data

5. Cum functioneaza un query in spate si cum poate fi optimizat ? Execution Plan ce este ?