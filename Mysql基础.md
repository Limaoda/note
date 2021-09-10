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

```sql
-- 查询语句 SELECT
-- SELECT [字段] FROM `表名`
-- 查询所有字段
SELECT * FROM `student`; -- 查询student表的所有列

-- 查询指定字段
SELECT `studentname`,`address` FROM `student`; -- 查询student表的studentname列和address列

-- 给查询字段起别名
SELECT `studentname` AS 名字,`address` AS 地址 FROM `student`; -- 查询student表的studentname列和address列并分别将表头重命名为'名字'和'地址'

-- concat函数
SELECT CONCAT('学生名：',studentname),CONCAT('学生地址：',address) FROM `student`;  -- 查询student表的studentname列和address列并在字段值前拼接字符串
```

