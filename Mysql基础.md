```shell
#连接数据库
mysql -uroot -p${password}
```

```mysql
-- 所有sql语句都需要用分号结尾，sql可以换行，当有分号时才认为一句sql语句结束
update mysql.user set authentication_string=password('123456') where user='root' and Host = 'localhost'; -- 修改密码

flush privileges; -- 刷新权限

show databases; -- 显示所有的数据库

use ${database}; -- 切换数据库 use 数据库名

show tables; -- 查看当前数据库的所有表

describe ${table} -- 查看表中的详情信息 describe 表名

create database ${database} -- 创建一个数据库 create database 数据库名
```

数据库语言类型

DDL(Database Define Language) 数据库定义语言

DML(Database Manage Language) 数据库操作语言

DQL(Database Query Language) 数据库查询语言

DCL(Database Crotrol Language) 数据库控制语言

### 1、操作数据库

#### 1.1、数据库的基本操作

* **创建数据库**

```mysql
CREATE DATABASE [IF NOT EXISTS] `hello`; -- 创建一个名为hello的数据库
```

* **删除数据库**

```mysql
DROP DATABASE IF EXISTS `hello`; -- 删除一个名为hello的数据库
```

* **使用数据库**

```mysql
USE `test`; -- 使用一个名为test的数据库 ``可以避免与mysql关键字混淆
```

* **查看数据库**

```mysql
SHOW DATABASES; -- 查看所有数据库
```

#### 1.2、数据库数据类型

> 数值类型

* tinyint    最小的数据 1个字节

* smallint   较小的数据 2个字节

* mediumint   3个字节

* **int   标准整数类型   4个字节   常用**
* bigint   较大的数据   8个字节
* float   单精度浮点类型    4个字节
* double   双精度浮点类型   8个字节
* decimal   字符串类型浮点数    常用于金融货币计算

> 字符串类型

* char   固定长度字符串   0~255
* **varchar   可变长度字符串   0~65535   常用**
* tinytext   微型文本   2^8-1
* **text   文本串   2^16-1   保存大文本**

> 时间日期类型

* date   日期格式   YYYY-MM-DD
* time    时间格式   HH:mm:ss
* **datetime   日期时间格式   YYYY-MM-DD HH:mm:ss   常用**
* timestamp   时间戳格式   1970.1.1到现在的毫秒数   全球统一
* year   年份

> null类型

* 无值，未知
* **==不要用null进行运算，运算结果都为null==**

#### 1.3、数据库的字段属性

* 主键

* unsigned   

* 自增

* zerofull（零填充）

* 默认

* 非空

* 更新

* 注释

  

>阿里巴巴表规范（每一张表必须存在以下5个字段）
>
>id   主键
>
>version   乐观锁
>
>is_delete   假删除
>
>gmt_create   创建时间
>
>gmt_update   更新时间

#### 1.4、数据库表的基本操作

* **创建表**

```sql
CREATE TABLE IF NOT EXISTS `表名` (
    `字段名` 列类型 [属性] [索引] [注释],
    `字段名` 列类型 [属性] [索引] [注释],
    ......
)[表类型][字符集设置][注释]

-- 例子如下
CREATE TABLE IF NOT EXISTS `user` (
  `id` INT(20) NOT NULL AUTO_INCREMENT COMMENT '用户id',
  `nickName` VARCHAR(20) NOT NULL DEFAULT '无' COMMENT '用户名',
  `account` VARCHAR(20) NOT NULL DEFAULT 'username' COMMENT '用户账号',
  `password` VARCHAR(15) NOT NULL DEFAULT '123456' COMMENT '用户密码',
  `isAdmin` INT(3) NOT NULL DEFAULT '0' COMMENT '是否为管理员',
  `is_delete` INT(3) NOT NULL DEFAULT '0' COMMENT '假删除标志位',
  `gmt_create` DATETIME DEFAULT NULL COMMENT '表创建日期时间',
  `gmt_update` DATETIME DEFAULT NULL COMMENT '表更新日期时间',
  `version` VARCHAR(10) DEFAULT NULL COMMENT '乐观锁',
   PRIMARY KEY (`id`)
)ENGINE=INNODB DEFAULT CHARSET=utf8
```

* **删除表**

```sql
DROP TABLE IF EXISTS `表名`;
```



* **查看库表sql定义语句**

```sql
SHOW CREATE database `test`; -- 查看test数据库的定义语句
SHOW CREATE TABLE `user`; -- 查看user表的定义语句
DESC `user` -- 显示user表的结构
```



#### 1.5、数据表的类型

|            | MYISAM | INNODB         |
| ---------- | ------ | -------------- |
| 事务支持   | 不支持 | 支持           |
| 数据行锁定 | 不支持 | 支持           |
| 外键约束   | 不支持 | 支持           |
| 全文索引   | 支持   | 不支持         |
| 表空间大小 | 较小   | 较大，约为两倍 |

>  mysql数据库实质上是一个文件夹，文件夹内的各种文件就是数据表

* **InnoDB 的文件类型**

*.frm (表的结构定义)

*.ibd

* **Myisam文件类型**

*.frm (表的结构定义)

*.MYD

*.MYI

* 修改字符集编码为utf8的方式
  1. 建表时加上CHARSET=utf8
  2. 在my.ini文件中写入character-set-server=utf8

#### 1.6、修改删除表

```sql
ALTER TABLE `user` RENAME AS `user_test`; -- 将数据表user重命名为user_test

ALTER TABLE `user` ADD	`sex` VARCHAR(2) NOT NULL DEFAULT '男' COMMENT '性别'; -- 向user表添加一个名字为sex的新字段并设置相关属性

ALTER TABLE `user` MODIFY `sex` VARCHAR(2) NOT NULL DEFAULT '女' COMMENT '性别'; -- 将user表的age字段值默认设置修改为'女'（修改约束）

ALTER TABLE `user` CHANGE `sex` `c_sex` VARCHAR(5) NOT NULL DEFAULT '女' COMMENT '性别'; -- 将user表的sex字段重命名为c_sex并修改约束

ALTER TABLE `user` DROP `sex`; -- 从user表中删除sex字段
```

### 2、Mysql数据管理

#### 2.1、外键

```sql
/*
将user表（从表）的roleId字段作为外键引用role表(主表)的roleId字段
*/
CREATE TABLE IF NOT EXISTS `user` (
    `id` INT(20) NOT NULL AUTO_INCREMENT COMMENT '用户id',
    `nickName` VARCHAR(20) NOT NULL DEFAULT '无' COMMENT '用户名', 
    `account` VARCHAR(20) NOT NULL DEFAULT 'username' COMMENT '用户账号', 
    `password` VARCHAR(15) NOT NULL DEFAULT '123456' COMMENT '用户密码', 
    `isAdmin` INT(3) NOT NULL DEFAULT '0' COMMENT '是否为管理员', 
    `roleId` INT(20) NOT NULL COMMENT '角色id（外键）', 
    `is_delete` INT(3) NOT NULL DEFAULT '0' COMMENT '假删除标志位', 
    `gmt_create` DATETIME DEFAULT NULL COMMENT '表创建日期时间', 
    `gmt_update` DATETIME DEFAULT NULL COMMENT '表更新日期时间', 
    `version` VARCHAR(10) DEFAULT NULL COMMENT '乐观锁', 
    PRIMARY KEY (`id`), 
    KEY `FK_roleId` (`roleId`), 
    CONSTRAINT `FK_roleId` FOREIGN KEY (`roleId`) REFERENCES `role`(`roleId`) )ENGINE=INNODB DEFAULT CHARSET=utf8; 
    
/* 第二种关联外键方法，建表后再添加外键约束 */
CREATE TABLE IF NOT EXISTS `user` (
    `id` INT(20) NOT NULL AUTO_INCREMENT COMMENT '用户id',
    `nickName` VARCHAR(20) NOT NULL DEFAULT '无' COMMENT '用户名', 
    `account` VARCHAR(20) NOT NULL DEFAULT 'username' COMMENT '用户账号', 
    `password` VARCHAR(15) NOT NULL DEFAULT '123456' COMMENT '用户密码', 
    `isAdmin` INT(3) NOT NULL DEFAULT '0' COMMENT '是否为管理员', 
    `roleId` INT(20) NOT NULL COMMENT '角色id（外键）', 
    `is_delete` INT(3) NOT NULL DEFAULT '0' COMMENT '假删除标志位', 
    `gmt_create` DATETIME DEFAULT NULL COMMENT '表创建日期时间', 
    `gmt_update` DATETIME DEFAULT NULL COMMENT '表更新日期时间', 
    `version` VARCHAR(10) DEFAULT NULL COMMENT '乐观锁', 
    PRIMARY KEY (`id`)
)ENGINE=INNODB DEFAULT CHARSET=utf8; 

ALTER TABLE `user` ADD CONSTRAINT `FK_roleId` FOREIGN KEY (`roleId`) REFERENCES `role` (`roleId`); -- 给user表添加一个约束FK_roleId将roleId字段作为外键引用role表的roleId字段

-- 主表因为被引用，当从表存在与主表关联的外键时，主表无法被删除
-- 开发中不建议使用物理外键，因为数据库过多时会造成引用混乱
```



#### 2.2、DML语言 (操作字段数据)

* 添加

  ```sql
  -- 插入语句（添加）
  -- insert into `表名`([字段1],[字段2],[字段3],...) values ('行1字段1值','行1字段2值','行1字段3值'),('行2字段1值','行2字段2值','行2字段3值');
  INSERT INTO `user` (`nickName`,`account`) VALUES ('张三','zhangsan'),('李四','lisi'); -- 向user表中插入两行数据（nickName列和account列设置值，其它列取默认值）
  ```

  

* 删除

  > delete命令

  ```sql
  -- 删除语句 DELETE
  -- DELETE FROM `表名` WHERE [筛选条件]
  DELETE FROM `user`; -- 不指定筛选条件,删除所有user表数据（数据库不重启时自增量不清0）
  DELETE FROM `user` WHERE id>=2; -- 删除user表id大于等于2的行
  ```

  > truncate命令(清空)

  ```sql
  TRUNCATE `user`; -- 清空user表的所有数据（自增量清0）
  ```

  

* 修改

  ```sql
  -- 更新语句（修改）UPDATE
  -- UPDATE `表名` SET `字段`='新字段值',[`字段`='新字段值'] WHERE [筛选条件] 
  
  /*
   携带条件的更新语句
   */
  UPDATE	`user` SET `nickName`='王五' WHERE `id`='2'; -- 将user表的id为2的行的nickName字段修改为王五
  
  /*
   不携带条件的更新语句
   */
  UPDATE	`user` SET `nickName`='王五'; -- 没有携带查询条件时，默认将user表所有行的nickName
  
  /*
   同时修改指定行的多个字段
   */
  UPDATE	`user` SET `nickName`='王五',`password`='ww123456' WHERE `id`='1'; 
  ```

  where语句的执行

  | 操作符                    | 含义         | 范围         | 结果  |
  | ------------------------- | ------------ | ------------ | ----- |
  | =                         | 等于         | 10=5         | false |
  | <>或!=                    | 不等于       | 10!=5        | true  |
  | >                         | 大于         | 10>5         | true  |
  | <                         | 小于         | 10<5         | false |
  | <=                        | 小于等于     | 10<=5        | false |
  | >=                        | 大于等于     | 10>=5        | true  |
  | BETWEEN [start] AND [end] | 在某个范围内 | [5,10]       |       |
  | AND                       | 并集         | 10>5 and 2>3 | false |
  | OR                        | 交集         | 10>5 or 2>3  | true  |

  

### 3、DQL查询数据

#### 3.1、基本查询

```sql
-- 查询语句 SELECT
/*
 SELECT [字段1,字段2,...] {DISTINCT}
 FROM `表名`
 WHERE [筛选条件]
 GROUP BY [字段1,字段2,...]
 HAVING [分组过滤条件]
 ORDER BY [字段] [排序方式(DESC或ASC)]
 LIMIT 偏移量 页数数目
 */

-- 查询所有字段
SELECT * FROM `student`; -- 查询student表的所有列

-- 查询指定字段
SELECT `studentname`,`address` FROM `student`; -- 查询student表的studentname列和address列

-- 给查询字段起别名
SELECT `studentname` AS 名字,`address` AS 地址 FROM `student`; -- 查询student表的studentname列和address列并分别将表头重命名为'名字'和'地址'

-- concat函数
SELECT CONCAT('学生名：',studentname),CONCAT('学生地址：',address) FROM `student`;  -- 查询student表的studentname列和address列并在字段值前拼接字符串

-- 去重查询
SELECT DISTINCT `studentno` FROM `result`; -- 从result表中查出studentno数据列并进行去重

-- 查询mysql系统版本
SELECT VERSION();

-- 查询计算值
SELECT (1000-1)*3 AS 计算结果;

-- 查询变量
SELECT @@auto_increment_increment; -- 查询自增步长变量
```

#### 3.2、条件查询

> 逻辑运算符

| 运算符    | 语法          | 描述 |
| --------- | ------------- | ---- |
| and   &&  | a and b a&&b  | 并集 |
| or   \|\| | a or b a\|\|b | 交集 |
| Not   ！  | not a   !a    | 非   |



```sql
-- 条件查询 SELECT...WHERE...
-- SELECT `字段名1`,[字段名2]`,... FROM 表名 WHERE [筛选条件]
SELECT `studentno`,`studentresult` FROM result WHERE `studentresult`>=70 AND `studentresult`<=90; -- 查询result表studentresult字段值位于70到90区间内的studentno列和studentresult列

SELECT `studentno`,`studentresult` FROM result WHERE NOT studentno=1000; -- 查询result表studentno字段值不等于1000的studentno列和studentresult列
```

> 模糊查询：比较运算符

| 运算符      | 语法              | 描述                                  |
| ----------- | ----------------- | ------------------------------------- |
| IS NULL     | a is null         | 如果操作符为null，结果为真            |
| IS NOT NULL | a is not null     | 如果操作符不为null，结果为真          |
| BETWEEN     | a between b and c | 若a在b和c之间，结果为真               |
| **Like**    | a like b          | SQL匹配，如果a匹配b，结果为真         |
| In          | a in (a1,a2,a3)   | 包含，a被包含于(a1,a2,a3)中，结果为真 |

```sql
-- like和%或_结合的模糊查询(%表示任意个数的任意字符 _表示一个任意字符)

SELECT * FROM student WHERE `studentname` LIKE '赵%'; -- 查询student表中studentname开头为赵字的所有列

SELECT * FROM student WHERE `studentname` LIKE '%江%'; -- 查询student表中studentname中间包含江字的所有列

-- in
SELECT * FROM student WHERE `studentno` IN (1000,1003); -- 查询student表中studentno包含1000或1003的所有列

-- IS NULL
SELECT * FROM student WHERE `studentno` IN (1000,1003); -- 查询student表中studentno包含1000或1003的所有列
```

#### 3.3、联表查询

![](C:\Users\HuQiaoDong\Desktop\笔记\images\join.png)

```sql
-- 联表查询
/*
 联表查询思路分析
 1.分析需要查询的字段来自哪些表
 2.确定使用哪种连接查询？共7种连接查询
 3.确定多表之间的交叉数据（哪些数据值是相等的）
 */
-- join on(条件判断) 连接查询
-- where 等值查询
 
-- inner join联表查询
SELECT s.studentno,s.studentname,g.gradename FROM student AS s INNER JOIN grade AS g WHERE s.gradeid=g.gradeid; -- 从student表和grade表中查出student表的gradeid值等于grade表的gradeid值的studentno，studentname，gradename列

-- 三表联表查询，查找学生的成绩单（学号、名字、学科名、成绩）
/*
 思路分析
 1.学号、名字来自student表，学科名来自subject表，成绩来自result表，需要从三个表中拿数据
 2.查询的是成绩单，因此result表作为主表
 3.student表和result表的交叉数据是studentno，result表和subject表的交叉数据是subjectno
 */
SELECT s.studentno,studentname,subjectname,r.studentresult FROM student s INNER JOIN result r ON s.`studentno`=r.`studentno` INNER JOIN `subject` sub ON r.subjectno=sub.subjectno;

-- 查询学员所属的年级（学号、名字、年级名称）
/*
 思路分析
 1.学号和名字来自student表，年级名称来自grade表
 2.查询的是学员的年级信息，主表为student表
 3.student表和grade表的交叉数据是gradeid
 */
SELECT `studentno`,`studentname`,`gradename` FROM student INNER JOIN grade ON student.`gradeid`=grade.`gradeid`;

-- 查询科目所属的年级（科目名、年级名）
 /*
 思路分析
 1.科目名来自subject表，年级名来自grade表
 2.查询的主体对象是科目，主表为subject表
 3.subject表和grade表的交叉数据是gradeid
 */
 SELECT `subjectname`,`gradename` FROM `subject` LEFT JOIN grade ON `subject`.`gradeid`=grade.`gradeid`;
 
 -- 查询参加了数据结构考试的同学信息（科目名、学号、名字、年级名称、分数）
SELECT subjectname,s.studentno,studentname,gradename,studentresult 
FROM student s 
INNER JOIN `result` r 
ON r.`studentno`=s.`studentno`  
INNER JOIN `subject` sub 
ON sub.`subjectno`=r.`subjectno` 
INNER JOIN `grade` g 
ON s.gradeid=g.gradeid
WHERE subjectname='数据库结构-1'  
```

#### 3.4、分页和排序

```sql
-- 排序 ORDERBY 字段 排序方式[ASC,DESC]
-- SELECT subjectname,s.studentno,studentname,gradename,studentresult 
FROM student s 
INNER JOIN `result` r 
ON r.`studentno`=s.`studentno`  
INNER JOIN `subject` sub 
ON sub.`subjectno`=r.`subjectno` 
INNER JOIN `grade` g 
ON s.gradeid=g.gradeid
WHERE subjectname='数据库结构-1' 

-- 分页 LIMIT 起始值,页面大小
-- LIMIT 0,5    1~5
-- LIMIT 1,5    2~6
SELECT subjectname,s.studentno,studentname,gradename,studentresult 
FROM student s 
INNER JOIN `result` r 
ON r.`studentno`=s.`studentno`  
INNER JOIN `subject` sub 
ON sub.`subjectno`=r.`subjectno` 
INNER JOIN `grade` g 
ON s.gradeid=g.gradeid
WHERE subjectname='数据库结构-1' 
ORDER BY studentresult DESC
LIMIT 0,5
-- 第n页 LIMIT (n-1)*pageSize,pageSize
-- （n-1）*pageSize = 起始值
-- 数据总数/页面大小 = 总页数

-- 查询JAVA第一学年 课程成绩排名前十的学生，并且分数要大于80的学生信息（学号，姓名，课程名称，分数）
/*
 思路分析：
 1.学号、姓名来自学生表，课程名称来自学科表，分数来自成绩表
 2.要查的是学生信息，主表为学生表
 3.学生表和成绩表的交叉数据为studentno，成绩表和学科表的交叉数据为subjectno，先关联学生表和成绩表，再关联学科表
 4.结果集的学生分数要大于80且只查JAVA第一学年学科且学生排名前十（降序，分页取前十条）
 */
SELECT s.studentno,studentname,subjectname,studentresult
FROM student s
INNER JOIN result r
ON s.`studentno`=r.`studentno`
INNER JOIN `subject` sub
ON r.`subjectno`=sub.`subjectno`
WHERE subjectname='Java程序设计-1'
AND studentresult>80
ORDER BY studentresult DESC
LIMIT 0,10;
```

#### 3.5  子查询

```sql
-- 查询数据库结构-1的所有考试结果（学号，科目编号，成绩），降序排列
-- 方式一 连接查询
SELECT studentno,s.subjectname,studentresult
FROM `subject` s
INNER JOIN result r
ON s.`subjectno`=r.`subjectno`
WHERE subjectname='数据库结构-1'
ORDER BY r.studentresult DESC;

-- 方式二 子查询（由里向外）
SELECT studentno,subjectno,studentresult 
FROM result
WHERE subjectno=(
 SELECT subjectno 
 FROM `subject` 
 WHERE  subjectname='数据库结构-1'
)
ORDER BY studentresult DESC;

-- 高等数学-2学科分数不小于80分的学生的学号和姓名
-- 方式一 联表查询
SELECT s.studentno,studentname
FROM student s
INNER JOIN result r
ON s.`studentno`=r.`studentno`
INNER JOIN `subject` sub 
ON sub.`subjectno`=r.`subjectno`
WHERE studentresult>='80' AND sub.subjectname='高等数学-2';

-- 方式二 联表+子查询
SELECT s.studentno,studentname
FROM student s
INNER JOIN result r
ON s.`studentno`=r.`studentno`
WHERE r.`subjectno`=(
 SELECT subjectno 
 FROM `subject`
 WHERE subjectname='高等数学-2'
) AND studentresult>='80';

-- 方式三 嵌套子查询
 SELECT s.studentno,studentname
 FROM student s
 WHERE studentno = (
  SELECT studentno 
  FROM result 
  WHERE studentresult>=80 
  AND subjectno = (
   SELECT subjectno
   FROM `subject`
   WHERE subjectname='高等数学-2'
  )
 )
 
 -- 查询C语言-1前5名同学的成绩的信息（学号，姓名，分数）
  -- 方式一 联表查询
 SELECT s.studentno,studentname,studentresult
 FROM student s
 INNER JOIN result r
 ON s.`studentno`=r.`studentno`
 INNER JOIN `subject` sub
 ON r.`subjectno`=sub.`subjectno`
 WHERE subjectname='C语言-1'
 ORDER BY studentresult DESC
 LIMIT 0,5;
 
 -- 方式二 嵌套子查询
 SELECT s.studentno,studentname,studentresult
 FROM student s
 INNER JOIN result r
 ON s.`studentno`=r.`studentno`
 WHERE r.subjectno = (
  SELECT subjectno
  FROM `subject`
  WHERE subjectname='C语言-1'
 )
 ORDER BY studentresult DESC
 LIMIT 0,5;
```

#### 3.6、分组和过滤

```sql
-- 查询不同课程的平均分，最高分，最低分，并且平均分大于80的数据
SELECT subjectname,AVG(studentresult) AS 平均分,MAX(studentresult),MIN(studentresult)
FROM result r
INNER JOIN `subject` sub
ON r.`subjectno`=sub.`subjectno`
GROUP BY r.subjectno
HAVING 平均分>=80;
```



### 4、Mysql函数

#### 4.1、一般函数

#### 4.2、聚合函数

| 函数名称 | 描述   |
| -------- | ------ |
| COUNT()  | 计数   |
| SUM()    | 求和   |
| AVG()    | 求平均 |
| MIN()    | 最小值 |
| MAX()    | 最大值 |
|          |        |

```sql
-- COUNT()查询一个表中有多少个记录
SELECT COUNT(studentno) FROM student; -- 指定列行数，会忽略所有null值
SELECT COUNT(*) FROM student; -- 所有列行数，不会忽略null值
SELECT COUNT(1) FROM student; -- 所有列行数，不会忽略null值

-- SUM()求和
SELECT SUM(studentresult) AS 总分 FROM result; -- 求学生成绩总值

-- AVG()求平均
SELECT AVG(studentresult) AS 平均分 FROM result; -- 求学生成绩平均值

-- MAX()求最大值
SELECT MAX(studentresult) AS 最高成绩 FROM result; -- 求学生最高成绩

-- MIN()求最小值
SELECT MIN(studentresult) AS 最低成绩 FROM result; -- 求学生最低成绩

```

### 5、事务

#### 5.1、事务基础

概念：将一组SQL放在一个批次中去执行

> 事务原则：ACID原则（A：原子性  C：一致性 I：隔离性 D：持久性）

**原子性（Atomicity）**

==要么都执行成功，要么都执行失败==

事件1：A有1000块钱，给B转200块钱 

事件2：B有200块钱，收到A转的200块钱

如果事件1执行成功，事件2执行失败，就破坏了原子性原则

**一致性（Consistency）**

==事务提交前后的数据完整性要保持一致==

事件1：A有1000块钱，给B转200块钱 

事件2：B有200块钱，收到A转的200块钱

必须保证A的钱加上B的钱总值为1200块钱

**持久性(Durability)**

事务一旦提交则不可逆，被持久化到数据库中

**隔离性(Isolation)**

多个用户并发访问数据库时，数据库为每一个用户单独开启一个事务，不被其它事务的操作所干扰，事务与事务之间相互隔离

> 隔离所导致的一些问题

**脏读：**

一个事务读取了另外一个事务未提交的数据

**不可重复读：**

在一个事务内读取表中的某一行数据，多次读取结果不同（中途发生过写入或修改）

**幻读：**

指一个事务内读到了别的事务插入的数据，导致前后读取结果不一致的情况。

#### 5.2、事务测试

```sql
-- mysql事务自动开启

-- 开启事务提交
SET autocommit = 0;

-- 关闭事务提交
SET autocommit = 1;


```

