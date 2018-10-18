# 查询1：
```sql
SELECT d.department_name，count(e.job_id)as "部门总人数"，
avg(e.salary)as "平均工资"
from hr.departments d，hr.employees e
where d.department_id = e.department_id
and d.department_name in ('IT'，'Sales')
GROUP BY department_name;
```

- 查询结果：

![tu](./1.png)

- 执行计划：

其中：cost=2，rows=20,Predicate Information(谓词信息）中有一次索引access，一次全表搜索filter。

![tu](./1.1.png)

![tu](./1.11.png)

# 查询2：
```
SELECT d.department_name，count(e.job_id)as "部门总人数"，
avg(e.salary)as "平均工资"
FROM hr.departments d，hr.employees e
WHERE d.department_id = e.department_id
GROUP BY department_name
HAVING d.department_name in ('IT'，'Sales');
```

- 查询结果：

![tu](./3.png)

- 执行计划：

其中：cost=5，rows=106,Predicate Information(谓词信息）中有一次索引access，两次全表搜索filter。

![tu](./2.2.png)

![tu](./2.22.png)

从分析两个SQL语句可以看，查询1是先过滤后汇总（where子句），参与汇总与计算的数据量少。而查询2是先汇总后过滤（having子句），参与汇总与计算的数据量多。且通过对以上两个查询的执行计划进行比较，查询1中cost=2，查询2中cost=5，二者相比，查询1中的cost更小，即：它的成本更低，因为成本越低越好，所以查询1的QL语句是最优的。

- 对查询1进行优化指导：

![tu](./2.png)

```
GENERAL INFORMATION SECTION
-------------------------------------------------------------------------------
Tuning Task Name   : staName35891
Tuning Task Owner  : HR
Tuning Task ID     : 11
Workload Type      : Single SQL Statement
Execution Count    : 1
Current Execution  : EXEC_1
Execution Type     : TUNE SQL
Scope              : COMPREHENSIVE
Time Limit(seconds): 1800
Completion Status  : COMPLETED
Started at         : 10/16/2018 11:27:51
Completed at       : 10/16/2018 11:27:52

-------------------------------------------------------------------------------
Schema Name   : HR
Container Name: PDBORCL
SQL ID        : gv51ddmnrkrj6
SQL Text      : SELECT d.department_name，count(e.job_id)as "部门总人数",
                avg(e.salary)as "平均工资"
                from hr.departments d,hr.employees e
                where d.department_id = e.department_id
                and d.department_name in ('IT','Sales')
                GROUP BY department_name

-------------------------------------------------------------------------------
FINDINGS SECTION (1 finding)
-------------------------------------------------------------------------------

1- Index Finding (see explain plans section below)
--------------------------------------------------
  通过创建一个或多个索引可以改进此语句的执行计划。

  Recommendation (estimated benefit: 59.99%)
  ------------------------------------------
  - 考虑运行可以改进物理方案设计的访问指导或者创建推荐的索引。
    create index HR.IDX$$_000B0001 on HR.DEPARTMENTS("DEPARTMENT_NAME","DEPARTM
    ENT_ID");

  Rationale
  ---------
    创建推荐的索引可以显著地改进此语句的执行计划。但是, 使用典型的 SQL 工作量运行 "访问指导"
    可能比单个语句更可取。通过这种方法可以获得全面的索引建议案, 包括计算索引维护的开销和附加的空间消耗。

-------------------------------------------------------------------------------
```
