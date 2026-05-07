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

4. Delete duplicate emails

```sql
delete
    p2
from Person p1
join Person p2
    on p1.email = p2.email
where p2.id > p1.id;
```

5. Second highest salary

```sql
select
    (select
        distinct salary
    from Employee
    order by salary desc
    limit 1 offset 1) as SecondHighestSalary;
```

6. Rank scores

```sql
select
    score,
    dense_rank() over (
        order by score desc
    )
from Scores;
```

7. Department highest salary

```sql
with top_dep as (
    select
        name,
        salary,
        departmentId,
        dense_rank() over (
            partition by departmentId
            order by salary desc
        ) as clasament
    from Employee
)
select
    d.name as Department,
    td.name as Employee,
    td.salary as Salary
from top_dep td
inner join Department d
    on td.departmentId = d.id
where td.clasament = 1;
```

8. Managers with at least 5 direct reports

```sql
with reports_5 as (
    select
        managerId,
        count(managerId) as n_reports
    from Employee
    group by managerId
    having count(managerId) >=5
)
select
    e.name
from reports_5 r
inner join Employee e
    on r.managerId = e.id;
```

9. Monthly transactions 1

```sql
select
    date_format(trans_date, "%Y-%m") as 'month',
    country,
    count(*) as trans_count,
    sum(
        case
            when state = 'approved' then 1
            else 0
        end
    ) as approved_count,
    sum(amount) as trans_total_amount,
    sum(
        case
            when state = 'approved' then amount
            else 0
        end
    ) as approved_total_amount
from Transactions
group by
    month,
    country;
```

10. Exchange seats

```sql
select
    case
        when id % 2 <> 0 and id + 1 <= (select max(id) from Seat)
            then id + 1
        when id % 2 = 0
            then id - 1
        else id
    end as id,
    student
from Seat
order by id asc;
```