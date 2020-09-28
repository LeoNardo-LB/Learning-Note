# SQL语句执行顺序

1.  from
2.  join
3.  on
4.  where
5.  group by
6.  (分组函数，分组函数中的distinct)
7.  having
8.  select
9.  distinct
10.  order by
11.  limit
12.  union

>   注意：
>
>   -   表起起别名，会覆盖原有表明，之后只能使用表的别名
>   -   字段起别名，只有执行迅速在select之后的才能使用别名，而且别名不会覆盖（可以使用原有字段）
>
>   MySQL对起别名进行了优化，虽然group by, having 的执行顺序在select 之前，但别名依旧可以使用（其他数据库不可）