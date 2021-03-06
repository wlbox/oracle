# 实验三：创建分区表
- 用户：system用户、new_userwl用户
## 实验内容：
- 本实验使用3个表空间：USERS,USERS02,USERS03。在表空间中创建两张表：订单表(orders)与订单详表(order_details)。

- 使用你自己的账号创建本实验的表，表创建在上述3个分区，自定义分区策略。

- 你需要使用system用户给你自己的账号分配上述分区的使用权限。你需要使用system用户给你的用户分配可以查询执行计划的权限。

- 表创建成功后，插入数据，数据能并平均分布到各个分区。每个表的数据都应该大于1万行，对表进行联合查询。

- 写出插入数据的语句和查询数据的语句，并分析语句的执行计划。

- 进行分区与不分区的对比实验。
## 实验步骤：
1、使用system用户给自己的用户new_userwl分配上述分区的使用权限：

![tu](./tupian/a.png)

2、用自己的账号new_userwl登录：

![tu](./tupian/b.png)

3、建立orders表并按订单日期进行分区：

![tu](./tupian/c.png)

```sql
SQL> CREATE TABLE orders 
(
 order_id NUMBER(10, 0) NOT NULL 
 , customer_name VARCHAR2(40 BYTE) NOT NULL 
 , customer_tel VARCHAR2(40 BYTE) NOT NULL 
 , order_date DATE NOT NULL 
 , employee_id NUMBER(6, 0) NOT NULL 
 , discount NUMBER(8, 2) DEFAULT 0 
 , trade_receivable NUMBER(8, 2) DEFAULT 0 
) 
TABLESPACE USERS 
PCTFREE 10 INITRANS 1 
STORAGE (   BUFFER_POOL DEFAULT ) 
NOCOMPRESS NOPARALLEL 
PARTITION BY RANGE (order_date) 
(
 PARTITION PARTITION_BEFORE_2016 VALUES LESS THAN (
 TO_DATE(' 2016-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 
 'NLS_CALENDAR=GREGORIAN')) 
 NOLOGGING 
 TABLESPACE USERS 
 PCTFREE 10 
 INITRANS 1 
 STORAGE 
( 
 INITIAL 8388608 
 NEXT 1048576 
 MINEXTENTS 1 
 MAXEXTENTS UNLIMITED 
 BUFFER_POOL DEFAULT 
) 
NOCOMPRESS NO INMEMORY  
, PARTITION PARTITION_BEFORE_2017 VALUES LESS THAN (
TO_DATE(' 2017-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN')) 
NOLOGGING 
TABLESPACE USERS02 
NOCOMPRESS NO INMEMORY  
, PARTITION PARTITION_BEFORE_2018 VALUES LESS THAN (
TO_DATE(' 2018-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN')) 
NOLOGGING 
TABLESPACE USERS03 
);
```
- 创建表 order_details 出现错误时的原因：

  此列列表的唯一关键字和主键不匹配的错误

- 具体原因：

![tu](./tupian/d.png)

- 解决办法:设置主键：

![tu](./tupian/e.png)

4、建立order_details表并按订单日期进行分区：

![tu](./tupian/f.png)

```sql
CREATE TABLE order_details 
(
id NUMBER(10, 0) NOT NULL 
, order_id NUMBER(10, 0) NOT NULL
, product_id VARCHAR2(40 BYTE) NOT NULL 
, product_num NUMBER(8, 2) NOT NULL 
, product_price NUMBER(8, 2) NOT NULL 
, CONSTRAINT order_details_fk1 FOREIGN KEY  (order_id)REFERENCES orders  (  order_id   )
ENABLE 
) 
TABLESPACE USERS 
PCTFREE 10 INITRANS 1 
STORAGE (   BUFFER_POOL DEFAULT ) 
NOCOMPRESS NOPARALLEL
PARTITION BY REFERENCE (order_details_fk1)
(-- PARTITION 2016
PARTITION PARTITION_BEFORE_2016 
NOLOGGING 
TABLESPACE USERS --必须指定表空间,否则会将分区存储在用户的默认表空间中
PCTFREE 10
INITRANS 1
STORAGE
(
INITIAL 8388608
NEXT 1048576
MINEXTENTS 1
MAXEXTENTS UNLIMITED
BUFFER_POOL DEFAULT
) 
NOCOMPRESS NO INMEMORY,-- PARTITION 2017 
PARTITION PARTITION_BEFORE_2017 
NOLOGGING 
TABLESPACE USERS02
PCTFREE 10
INITRANS 1
STORAGE
(
INITIAL 8388608
NEXT 1048576
MINEXTENTS 1
MAXEXTENTS UNLIMITED
BUFFER_POOL DEFAULT
) 
NOCOMPRESS NO INMEMORY,-- PARTITION 2018
PARTITION PARTITION_BEFORE_2018 
NOLOGGING 
TABLESPACE USERS03
PCTFREE 10
INITRANS 1
STORAGE
(
INITIAL 8388608
NEXT 1048576
MINEXTENTS 1
MAXEXTENTS UNLIMITED
BUFFER_POOL DEFAULT
) 
NOCOMPRESS NO INMEMORY  
);
```
5、建立好订单表(orders)与订单详表(order_details)的sqldeveloper中的截图：

![tu](./tupian/g.png)

6、向orders表中插入数据：

- 代码：
```sql
declare 
  m integer; begin 
--输出开始时间 
  dbms_output.put_line('start:'||sysdate); 
  m:=9998;
--循环插入的数据量 
  for i in 1..4999 loop 
   m:=m+1; insert into ORDERS (ORDER_ID,CUSTOMER_NAME,CUSTOMER_TEL,ORDER_DATE,EMPLOYEE_ID,DISCOUNT,TRADE_RECEIVABLE) values (m,'user'||m,'10000',to_date('2017-05-04 00:00:00', 'SYYYY-MM-DD HH24:MI:SS'),001,15,29);
    commit; 
  end loop; --输出结束时间 
  dbms_output.put_line('end:'||sysdate); 
end;
```
- 执行结果：

![tu](./tupian/a1.png)

- 查询orders表插入的总共记录数：

![tu](./tupian/b1.png)

   通过修改m值以及时间达到平均插入到不同的分区里面。总共的数据为14997条，其中每一分区含有4999条记录。

7、向order_details表中插入数据：
- 代码：
```sql
declare 
  m integer; begin 
--输出开始时间 
  dbms_output.put_line('start:'||sysdate); 
  m:=0;
--循环插入的数据量 
  for i in 1..4999 loop 
   m:=m+1; insert into ORDER_DETAILS (ID,ORDER_ID,PRODUCT_ID,PRODUCT_NUM,PRODUCT_PRICE) values (m,m,'product'||m,15,29);
    commit; 
  end loop; --输出结束时间 
  dbms_output.put_line('end:'||sysdate); 
end;
```
- 执行结果：

![tu](./tupian/c1.png)

- 查询order_details表插入的总共记录数：

![tu](./tupian/d1.png)

  通过对order_id的值进行改变，实现关联order表中的外键，并分配到各个分区里面，向order_details表中共插入14997条数据。

8、对表进行联合查询：
- 代码：
```sql
select 
    orders.order_id as AID,
    orders.customer_name as customer_name,
    order_details.order_id as BID,
    ORDER_DETAILS.PRODUCT_ID as product_idfrom
    ORDERSINNER JOIN ORDER_DETAILS ON (orders.order_id=order_details.order_id);
 ```
- 执行结果：

![tu](./tupian/e1.png)

9、分析语句的执行计划：

- 执行计划代码：
```sql
EXPLAIN plan for
select 
    orders.order_id as AID,
    orders.customer_name as customer_name,
    order_details.order_id as BID,
    ORDER_DETAILS.PRODUCT_ID as product_id
from
    ORDERS
INNER JOIN ORDER_DETAILS ON (orders.order_id=order_details.order_id);

select * from table(dbms_xplan.display());
```
- 执行计划结果:

![tu](./tupian/f1.png)

- 执行计划结论：

由执行结果可知：最先执行的是TABLE ACCESS FULL，意思为对order_details表进行全表扫描。

然后其次执行的是PARTITION REFERENCE ALL，对分区进行引用。

然后对order_id进行索引唯一扫描，因为为order_details的外键。

又因为使用了join，所以又进行了NESTED LOOPS连接查询。

再对orders表进行TABLE ACCESS BY GLOBAL INDEX ROWID，即rowid与索引的扫描，找出符合条件的元素。最后将数据查出。

10、分区与不分区的对比：

分区：就是把一张表的数据分成N多个区块，这些区块可以在同一个磁盘上，也可以在不同的磁盘上。

分区的优点：

a、改善查询性能：对分区对象的查询可以仅搜索自己关心的分区，提高检索速度。

b、增强可用性：如果表的某个分区出现故障，表在其他分区的数据仍然可用；

c、维护方便：如果表的某个分区出现故障，需要修复数据，只修复该分区即可；

d、均衡I/O：可以把不同的分区映射到磁盘以平衡I/O，改善整个系统性能。

不分区的缺点：

当表中的数据量不断增大时，查询数据的速度就会变得越来越慢，应用程序的性能就会下降等。

例如：在实际开发中，往往一个表里的数据量特别大。上百万千万的数量级。如果不进行分区，即：将这些数据放在一个物理文件内（就是表的物理存储文件）实在太大
了。为了避免这种情况，我们就可以使用表分区。
