# Probleme de Leetcode

1. Duplicate Emails

```sql
select
    email as Email
from Person
group by email
having count(id) > 1; 
```

2. Customers who never order

```sql
select
    name as Customers
from Customers c
left join Orders o
    on c.id = o.customerId
where o.customerId is null;
```

3. Employees earning more than their managers

```sql
select
    e2.name as Employee
from Employee e1
inner join Employee e2
    on e1.id = e2.managerId
where e2.salary > e1.salary;
```