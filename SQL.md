---
title: MySQL Note
---





# 关系模型 (Relational Model)

## 主键 (Primary Key)

- **主键**：能够通过某个字段唯一区分出不同的记录的字段

  

- **联合主键**: 通过多个字段唯一标识记录，即两个或更多的字段都设置为主键。

  - 允许有重复，只要不是所有主键列都重复即可

  - 没有必要的情况下尽量不使用，因为它给关系表带来了复杂度的上升。

    

- 主键不要带有业务含义，而应该使用 `BIGINT` 自增或者 `GUID` 类型。主键也不应该允许`NULL`。



## 外键 (Foreign Key)

- **外键**: 在一张表中，通过其字段，可以把数据与另一张表关联起来。

- 关系数据库通过外键可以实现一对多、多对多和一对一的关系。

  

- 定义外键:

  ```sql
  ALTER TABLE students
  ADD CONSTRAINT fk_class_id		 # 定义外键约束; 即为下面的外键设置命名
  FOREIGN KEY (class_id)					# 指定class_id为student表中的外键
  REFERENCES classes (id);			    # 指定class_id关联classes表中的id
  ```

- 通过定义外键约束，关系数据库可以保证无法插入无效的数据。

  - 如果classes表不存在id=99的记录，students表就无法插入class_id=99的记录。

    

- **外键约束会降低数据库的性能**

  - 因此, 公司都是把普通的列当作外键在用, 但并不明显地设置为外键

    

- 删除外键
  - 删除外键约束并没有删除外键这一列。删除列是通过`DROP COLUMN ...`实现的。

```sql
ALTER TABLE students
DROP FOREIGN KEY fk_class_id;
```





## 索引 (Index)

- **索引**: 关系数据库中对某一列或多个列的值进行预排序的数据结构。
  - 通过使用索引，可以让数据库系统不必扫描整个表，而是直接定位到符合条件的记录，这样就大大加快了查询速度.



- 创建索引:

  ```sql
  ALTER TABLE students
  ADD INDEX idx_score (score);	# idx_score: 索引名; (score): 对应的列名
  ```

- 索引如果有多列，可以在括号里依次写上:

  ```sql
  ALTER TABLE students
  ADD INDEX idx_name_score (name, score);
  ```

  

- **索引的效率**: 取决于索引列的值是否散列，即该列的值如果越互不相同，那么索引效率越高。

  - **主键自动索引**: 关系数据库会自动对其创建主键索引。使用主键索引的效率是最高的，因为主键会保证绝对唯一。

  - 可以对一张表创建多个索引。
  - 索引的优点是提高了查询效率，缺点是在插入、更新和删除记录时，需要同时修改索引. 
  - 因此，**索引越多，插入、更新和删除记录的速度就越慢**。
  - **索引是查询业务和修改业务之间的平衡**.



- **唯一索引**: 对于非主键的某些要求唯一的列, 可添加唯一索引/唯一性约束.

  - 添加唯一索引:

    ```sql
    ALTER TABLE students
    ADD UNIQUE INDEX uni_name (name);		# unique关键字代表唯一性
    ```

  - 只对某一列添加一个唯一约束而不创建唯一索引

    ```sql
    ALTER TABLE students
    ADD CONSTRAINT uni_name UNIQUE (name);   # uni_name为约束名; unique(name)为约束具体内容
    ```

    

​	

# 查询数据 (DQL)



## 基本查询 (select)

```sql
SELECT * FROM <表名>
```



## 条件查询 (where)

```sql
SELECT * FROM <表名> WHERE <条件表达式>
```

- 条件表达式: 
  - 可以通过and, or, not等连接多个表达式, 实现不同逻辑



## 投影查询 (select c1, c2, c3)

```sql
SELECT 列1 (as) c1, 列2, 列3 FROM 表1   # 可以把原始的列1重命名为c1; as可以省略; 选择只看列123

```



## 排序 (order by)

```sql
SELECT id, name, gender, score 
FROM students 
WHERE class_id = 1
ORDER BY score, name DESC;		# order by 实现排序; 通过添加desc可以实现倒序(默认正序); 先根据score, 再根据name
```





## 分页查询 (limit M offset N)

- 通过 `limit`, 可以设定返回的行数; 通过`offset`, 可以设定跳过的行数

```sql
SELECT id, name, gender, score
FROM students
ORDER BY score DESC
LIMIT 3 OFFSET 6;
```



- 分页公式:
  - `LIMIT` 总是设定为 `pageSize`；
  - `OFFSET` 计算公式为 `pageSize * (pageIndex - 1)`.



- 当`offset` 大于实际行数: 返回空记录, 不报错

- **offset可省略 (MySQL)**: `LIMIT 15 OFFSET 30` 可简写成 `LIMIT 30, 15`。



- 使用`LIMIT <M> OFFSET <N>`分页时，随着`N`越来越大，查询效率也会越来越低。



## 聚合查询 (aggregate function)





### 基本聚合

- 对于统计总数、平均数这类计算，SQL提供了专门的聚合函数进行查询

```sql
SELECT COUNT(*) FROM students;
```



- 函数:
  - `count`, `sum`, `avg`
  - `min`, `max`: 注意这两个函数不要求其中的列为数值型



- 例: 查询男生平均成绩

```sql
SELECT AVG(score) average 
FROM students 
WHERE gender = 'M';
```



- 如果聚合查询的`WHERE`条件没有匹配到任何行
  - `COUNT()`会返回0
  - `SUM()`、`AVG()`、`MAX()`和`MIN()`会返回`NULL`



- 例: 聚合查询查询页数:

  ```sql
  SELECT CEILING(COUNT(*) / 3) 
  FROM students;
  ```

  

### 分组聚合 (group by)

- 使用group by 可返回根据某一列值分组后, 组内的统计结果
- 这里select的标准写法是, `聚合键` 在前, 函数项在后

```sql
SELECT class_id, COUNT(*) num 
FROM students 
GROUP BY class_id;
```



## 多表查询

```sql
SELECT
    s.id sid,
    s.name,
    s.gender,
    s.score,
    c.id cid,
    c.name cname				
FROM students s, classes c						# 通过起别名来避免重名, 且减少码量
WHERE s.gender = 'M' AND c.id = 1;		# 可对各自的表应用filter
```





## 连接查询 (join)

- 连接查询时, 两(多)个表会被连起来形成一个中间表, 相当于在这个中间表上操作:

```sql
SELECT s.id, s.name, s.class_id, c.name class_name, s.gender, s.score
FROM students s
INNER JOIN classes c		# from 表1 join 表2 on 表1.c = 表2.c
ON s.class_id = c.id;
```



- `join` 默认使用的是`inner join` 的操作, 但其实有多种不同的join方式 (因为列数据不唯一导致):

  - `inner join`: 两表都有的
  - `left (outer) join`: 左表所有记录; 匹配上右表的才有右表的数据
  - `right (outer) join`: 右表所有记录; 匹配上左表的才有左表的数据

  - `full (outer) join`: 左右表所有记录; 匹配上的部分才同时有两表的数据
  - `cross join`: 笛卡尔集





# 修改数据 (DML)



- 关系数据库的基本操作就是增删改查，即CRUD：Create、Retrieve、Update、Delete

- 增、删、改，对应的SQL语句分别是：

  - `INSERT`：插入新记录；

  - `UPDATE`：更新已有记录；

  - `DELETE`：删除已有记录。



## Insert

- `INSERT`语句的基本语法是：
  - `id`一般是auto increment的, 不用我们插入
  - 同理, 有默认值的字段也可以不插入

```sql
INSERT INTO <表名> (字段1, 字段2, ...) VALUES (值1, 值2, ...);
```



- **一次性添加多条记录**: 只需要在`VALUES`子句中指定多个记录值，每个记录是由`(...)`包含的一组值：

```sql
INSERT INTO students (class_id, name, gender, score) VALUES
  (1, '大宝', 'M', 87),
  (2, '二宝', 'M', 81);
```



## Update

- `UPDATE`语句的基本语法是：

```sql
UPDATE <表名> 
SET 字段1=值1, 字段2=值2, ... 		# 设定要更新的行的值
WHERE ...;										 # where筛选要更新的行
```



- 可以一次更新多条记录：

```sql
UPDATE students 
SET name='小牛', score=77 
WHERE id>=5 AND id<=7;
```



- 更新字段时可以使用表达式。例如，把所有80分以下的同学的成绩加10分：

```sql
UPDATE students 
SET score=score+10 
WHERE score<80;
```



- 如果`WHERE`条件没有匹配到任何记录，`UPDATE`语句不会报错，也不会有任何记录被更新。

- `UPDATE`语句可以没有`WHERE`条件, 这时，整个表的所有记录都会被更新.

- 因此, 小心使用`update`. 先查询相应数据是否是想要的, 检查后再更新时比较好的做法.



- 在使用MySQL这类真正的关系数据库时，`UPDATE`语句会返回更新的行数以及`WHERE`条件匹配的行数。



## Delete

- `DELETE`语句的基本语法是：

```sql
DELETE FROM <表名> WHERE ...;
```



- `where` 的使用原理同 `UPDATE`



- 要特别小心的是，和`UPDATE`类似，不带`WHERE`条件的`DELETE`语句会删除整个表的数据

```sql
DELETE FROM students;
```



# 管理数据 (MySQL DML)



## 库 (database)

- 列出所有数据库: 

  ```sql
  SHOW DATABASES;
  ```

  - `information_schema`、`mysql`、`performance_schema`和`sys`是系统库，不要去改动它们。其他的是用户创建的数据库。



- 创建新数据库

  ```sql
  CREATE DATABASE test;
  ```

  

- 删除数据库

  ```sql
  DROP DATABASE test;
  ```



- 切换到指定数据库

  ```sql
  USE test;
  ```

  

## 表 (table)



### 基础

- 列出当前数据库的所有表

  ```sql
  SHOW TABLES;
  ```

  

- 查看一个表的结构

  ```sql
  DESC students;
  ```

  

- 查看创建表的SQL语句：

  ```sql
  SHOW CREATE TABLE students;
  ```

  

- 删除表:

  ```sql
  DROP TABLE students;
  ```

  

### DML重点: 创建表/修改表

- 新增一列:

  ```sql
  ALTER TABLE students ADD COLUMN birth VARCHAR(10) NOT NULL;
  ```

  

- 修改一列

  ```sql
  ALTER TABLE students CHANGE COLUMN birth birthday VARCHAR(20) NOT NULL;
  ```

  

- 删除一列

  ```sql
  ALTER TABLE students DROP COLUMN birthday;
  ```

  

- 创建表: 暂时略; 涉及到太多东西



# 实用语句

## 插入或替换 (REPLACE INTO)

- 如果我们希望插入一条新记录（`INSERT`），但如果记录已经存在，就先删除原记录，再插入新记录。此时，可以使用`REPLACE`语句，这样就不必先查询，再决定是否先删除再插入：

```sql
REPLACE INTO students (id, class_id, name, gender, score) VALUES (1, 1, '小明', 'F', 99);
```



## 插入或更新 (ON DUPLICATE KEY UPDATE)

- 如果我们希望插入一条新记录（INSERT），但如果记录已经存在，就更新该记录.
  - 此时，可以使用`INSERT INTO ... ON DUPLICATE KEY UPDATE ...`

```sql
INSERT INTO students (id, class_id, name, gender, score) 
VALUES (1, 1, '小明', 'F', 99) 
ON DUPLICATE KEY UPDATE name='小明', gender='F', score=99;
```



- 相比插入或替换, 这样做的好处是, **不会改变主键**



## 插入或忽略 (INSERT IGNORE INTO ...)

- 如果我们希望插入一条新记录（INSERT），但如果记录已经存在，就啥事也不干直接忽略

```sql
INSERT IGNORE INTO students (id, class_id, name, gender, score) VALUES (1, 1, '小明', 'F', 99);
```



## 快照

- **快照**: 复制一份当前表的数据到一个新表

```sql
-- 对class_id=1的记录进行快照，并存储为新表students_of_class1:
CREATE TABLE students_of_class1 SELECT * FROM students WHERE class_id=1;
```



## 写入查询结果集

- 如果查询结果集需要写入到表中，可以结合`INSERT`和`SELECT`，将`SELECT`语句的结果集直接插入到指定表中。

  

- 创建一个统计成绩的表`statistics`，记录各班的平均成绩：

```sql
CREATE TABLE statistics (
    id BIGINT NOT NULL AUTO_INCREMENT,
    class_id BIGINT NOT NULL,
    average DOUBLE NOT NULL,
    PRIMARY KEY (id)
);
```



  - 结合`INSERT`和`SELECT`，将`SELECT`语句的结果集直接插入到指定表中。

```sql
INSERT INTO statistics (class_id, average) 
SELECT class_id, AVG(score) FROM students GROUP BY class_id;   # 这一行的查询结果替代了values()
```



## 强制使用指定索引

- 在查询的时候，数据库系统会自动分析查询语句，并选择一个最合适的索引。
- 然而, 很多时候，数据库系统的查询优化器并不一定总是能使用最优索引。如果我们知道如何选择索引，可以使用`FORCE INDEX`强制查询使用指定的索引。

```sql
SELECT * FROM students 
FORCE INDEX (idx_class_id)  # 索引idx_class_id必须存在
WHERE class_id = 1 ORDER BY id DESC;
```



# 事务 (Transaction)



## 简介

- 在执行SQL语句的时候，某些业务要求，一系列操作必须全部执行，而不能仅执行一部分。
  - 例如, 转账: A转给B 100. 假如中间出了错, A转出了, B没收到, 这样的情况是我们不希望见到的. 
  - 使用事务保证了在这种情况下, 数据库会回滚 (roll back), 那100块钱会回到A的账上.



- **事务**: 把多条语句作为一个整体进行操作的功能.

  - 事务可以确保该事务范围内的所有操作都可以全部成功或者全部失败。如果事务失败，那么效果就和没有执行这些SQL一样，不会对数据库数据有任何改动。

    

- 特性: **ACID**

  - A：Atomic，原子性，将所有SQL作为原子工作单元执行，要么全部执行，要么全部不执行；

  - C：Consistent，一致性，事务完成后，所有数据的状态都是一致的，即A账户只要减去了100，B账户则必定加上了100；

  - I：Isolation，隔离性，如果有多个事务并发执行，每个事务作出的修改必须与其他事务隔离；

  - D：Duration，持久性，即事务完成后，对数据库数据的修改被持久化存储。



- **隐式事务**: 对于单条SQL语句，数据库系统自动将其作为一个事务执行。

- **显式事务**: 使用`BEGIN`开启一个事务，使用`COMMIT`提交一个事务

  - `COMMIT`是指提交事务，即试图把事务内的所有SQL所做的修改永久保存。如果`COMMIT`语句执行失败了，整个事务也会失败。

    ```sql
    BEGIN;
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;
    COMMIT; 
    ```

    

  - 有些时候，我们希望主动让事务失败，这时，可以用`ROLLBACK`回滚事务，整个事务会失败：

    ```sql
    BEGIN;
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;
    ROLLBACK;
    ```

    

- 隔离级别:

  - 对于两个并发执行的事务，如果涉及到操作同一条记录的时候，可能会发生问题。因为并发操作会带来数据的不一致性，包括脏读、不可重复读、幻读等。

  - 数据库系统提供了隔离级别来让我们有针对性地选择事务的隔离级别，避免数据不一致的问题。

    

| Isolation Level  | 脏读（Dirty Read） | 不可重复读（Non Repeatable Read） | 幻读（Phantom Read） |
| :--------------- | :----------------- | :-------------------------------- | :------------------: |
| Read Uncommitted | Yes                | Yes                               |         Yes          |
| Read Committed   | -                  | Yes                               |         Yes          |
| Repeatable Read  | -                  | -                                 |         Yes          |
| Serializable     | -                  | -                                 |          -           |



## Read Uncommitted

- Read Uncommitted是隔离级别最低的一种事务级别。
  - 在这种隔离级别下，一个事务会读到另一个事务更新后但未提交的数据;
  - 如果另一个事务回滚，那么当前事务读到的数据就是脏数据，这就是脏读 `（Dirty Read）`。



## Read Committed

- 在Read Committed隔离级别下，一个事务可能会遇到不可重复读（Non Repeatable Read）的问题。
  - 不可重复读是指，在一个事务内，多次读同一数据，在这个事务还没有结束时，如果另一个事务恰好修改了这个数据，那么，在第一个事务中，两次读取的数据就可能不一致。



- 因此，在Read Committed隔离级别下，事务应避免重复读同一条记录，因为很可能读到的结果不一致。



## Repeatable Read

- 在Repeatable Read隔离级别下，一个事务可能会遇到幻读（Phantom Read）的问题。
  - 幻读是指，在一个事务中，第一次查询某条记录，发现没有，但是，当试图更新这条不存在的记录时，竟然能成功，并且，再次读取同一条记录，它就神奇地出现了。 (**时灵时不灵**)



## Serializable

- Serializable是最严格的隔离级别。在Serializable隔离级别下，所有事务按照次序依次执行，因此，脏读、不可重复读、幻读都不会出现。



- 虽然Serializable隔离级别下的事务具有最高的安全性，但是，由于事务是串行执行，所以效率会大大下降，应用程序的性能会急剧降低。**如果没有特别重要的情景，一般都不会使用Serializable隔离级别。**



## 默认隔离级别

- 如果没有指定隔离级别，数据库就会使用默认的隔离级别。在MySQL中，如果使用InnoDB，默认的隔离级别是**Repeatable Read**。