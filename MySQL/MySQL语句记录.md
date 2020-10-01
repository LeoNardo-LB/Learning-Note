# MySQL 语句记录

#### 统计每一个部门工资最高员工的信息
```mysql
select ename, salary, did
from t_employee e
where salary = (
    select max(salary)
    from t_employee e2
    where e.did = e2.did);
```

**分析**

1.  在外层from中，记录定位到了一个did，比如did=1，则开始子查询
2.  子查询过程中，首先选中e2.did与e.did相等的那些部门，选中其中最大值，然后作为外层from的where条件返回，
3.  此时就会在外层did=1的记录中选择 薪水与子查询返回结果（最大值）相等的 ename，salary，did



#### 查询" 01 "课程比" 02 "课程成绩高的学生的信息及课程分数

```mysql
select *
from (
         select t1.sid, classA, classB
         from (select sid, score classA from sc where cid = '01') t1,
              (select sid, score classB from sc where cid = '02') t2
         where t1.sid = t2.sid
           and classA > classB
     ) r
         inner join student s on r.sid = s.sid;
```

**分析**

