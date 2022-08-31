---
title: mysql学习笔记3
urlname: vwvnrm
date: '2019-11-19 09:36:17 +0800'
tags: []
categories: []
---

## DML  数据操作语言

### 插入 insert

#### insert into  表名(列名, ...) values(值 1,....)

```sql
#插入的值的类型要与列的类型一致或兼容,不可以为null的列必须插入值
INSERT INTO beauty(id, NAME, sex, borndate, phone, photo, boyfriend_id)
VALUES(23, '赵丽颖', '女', '1990-12.14', '45646545', NULL, 3)

INSERT INTO beauty
VALUES(213, '赵丽颖', '女', '1990-12.14', '45646545', NULL, 3)
# 插入多行
INSERT INTO beauty
VALUES(23,'唐艺昕1','女','1990-4-23','1898888888',NULL,2)
,(25,'唐艺昕3','女','1990-4-23','1898888888',NULL,2);
#将boys表id<3 的列 插入到beauty表
INSERT INTO beauty(id,NAME,phone)
SELECT id,boyname,'1234567'
FROM boys WHERE id<3;
```

### insert into 表名 set 列名=值,列名=值,...

```sql
INSERT INTO beauty
SET id=19,NAME='刘涛',phone='999';
```

## 修改 update

### update  表名 set  列=新值,列=新值,...  where  筛选条件

```sql
UPDATE beauty SET phone = '13899888899'
WHERE NAME LIKE '唐%';
```

### update  表 1  别名  inner|left|right join  表 2  别名  on  连接条件  set  列=值，... where  筛选条件

```sql
#案例 修改张无忌的女朋友的手机号为114
UPDATE beauty b
INNER JOIN boys bo
ON b.boyfriend_id=bo.id
SET b.phone='110',bo.userCP='222'
WHERE bo.boyName='张无忌'
```

## 删除 delete

### delete from  表名  where  筛选条件

```sql
#案例：删除手机号以9结尾的女神信息
DELETE FROM beauty WHERE phone LIKE '%9';
```

### delete  表 1 的别名, 表 2 的别名  from  表 1  别名  inner|left|right join  表 2 on  连接条件  where  筛选条件

```sql
#案例：删除张无忌的女朋友的信息
DELETE b,bo
FROM beauty b
INNER JOIN boys bo
ON b.boyfriend_id=bo.id
WHERE bo.id=1
```

### truncate table  表名

```sql
#案例：将男神信息删除  相当于格式化
TRUNCATE TABLE boys ;
```

- delete 可以加 where 条件，truncate 不能加
- truncate 删除，效率高一丢丢
- 假如要删除的表中有自增长列，如果用 delete 删除后，再插入数据，自增长列的值从断点开始，而 truncate 删除后，再插入数据，自增长列的值从 1 开始。
- truncate 删除没有返回值，delete 删除有返回值
- truncate 删除不能回滚，delete 删除可以回滚

## DDL  数据定义语言

库和表的管理

### 库的管理

```sql
#创建库Books
CREATE DATABASE IF NOT EXISTS books ;
#修改库的名称
RENAME DATABASE books TO 新库名;
# 修改库的字符集
ALTER DATABASE books CHARACTER SET gbk;
#库的删除
DROP DATABASE IF EXISTS books;
```

### 表的管理

#### 创建表

create table 表名(
列名 列的类型【(长度) 约束】,
...
列名 列的类型【(长度) 约束】
)

```sql
CREATE TABLE book(
	id INT,#编号
	bName VARCHAR(20),#图书名
	price DOUBLE,#价格
	authorId  INT,#作者编号
	publishDate DATETIME#出版日期
);
```

#### 修改表  

alter table 表名 add|drop|modify|change column 列名 【列类型 约束】

```sql
#修改列名
ALTER TABLE book CHANGE COLUMN publishdate pubDate DATETIME;
#修改列的类型或约束
ALTER TABLE book MODIFY COLUMN pubdate TIMESTAMP;
#添加新列
ALTER TABLE author ADD COLUMN annual DOUBLE;
#删除列
ALTER TABLE book_author DROP COLUMN  annual;
#修改表名
ALTER TABLE author RENAME TO book_author;
#查看表
DESC book;
```

#### 删除表

```sql
DROP TABLE IF EXISTS book_author;
SHOW TABLES;
```

#### 复制表

```sql
#1.仅仅复制表的结构
CREATE TABLE new_table LIKE author;
#2.复制表的结构+数据
CREATE TABLE copy2
SELECT * FROM author;
#只复制部分数据
CREATE TABLE copy3
SELECT id,au_name
FROM author
WHERE nation='中国';
#仅仅复制某些字段
CREATE TABLE copy4
SELECT id,au_name
FROM author
WHERE 0;
```

## 数据类型

### 数值型

#### 整型

分类：tinyint、smallint、mediumint、int/integer、bigint

- 默认是有符号，如果想设置无符号，需要添加 unsigned 关键字
- 如果插入的数值超出了整型的范围,会报 out of range 异常，并且插入临界值
- 有默认的长度，长度代表了显示的最大宽度，如果不够会用 0 在左边填充，但必须搭配 zerofill 使用！

```sql
CREATE TABLE table_name(
		t1 INT(7) ZEROFILL,
  	t2 FLOAT,
		t3 INT(7) ZEROFILL
);
```

#### 小数

浮点型：float(M, D)   double(M, D)        M，D 根据插入的数值来决定
定点型：dec(M，D)   decimal(M,D)        M 默认为 10，D 默认为 0，精确度较高
M：整数部位+小数部位，D：小数部位，如果超过范围，则插入临界值

### 字符型

- 较短的文本：
  - char  固定长度的字符。写法：char(M)。  M 为最大的字符数，可省略，默认为 1。  比较耗费空间   效率高
  - varchar  可变长度。M 不可省略，节省空间，效率低。
- 较长的文本：
  - text，blob(较大的二进制)
- 其他：
  - binary 和 varbinary 用于保存较短的二进制
  - enum 用于保存枚举
  - set 用于保存集合

### 日期型

- date 保存日期
- time 只保存时间
- year  只保存年
- datetime  保存日期+时间   不受时区影响
- timestamp  保存日期+时间   受时区影响

## 约束

用于限制表中的数据，为了保证表中的数据的准确和可靠性
分类：六大约束

-     NOT NULL：非空，用于保证该字段的值不能为空，比如姓名、学号等
-     DEFAULT:默认，用于保证该字段有默认值，比如性别
-     PRIMARY KEY:主键，用于保证该字段的值具有唯一性，并且非空，比如学号、员工编号等
-     UNIQUE:唯一，用于保证该字段的值具有唯一性，可以为空，可以设置多个。比如座位号 
-     CHECK:检查约束【mysql中不支持】，比如年龄、性别
-     FOREIGN KEY:外键，用于限制两个表的关系，用于保证该字段的值必须来自于主表的关联列的值，在从表添加外键约束，用于引用主表中某列的值，比如学生表的专业编号，员工表的部门编号，员工表的工种编号

```sql
# 列级约束：六大约束语法上都支持，但外键约束没有效果
CREATE TABLE stuinfo(
	id INT PRIMARY KEY,#主键
	stuName VARCHAR(20) NOT NULL UNIQUE,#非空
	gender CHAR(1) CHECK(gender='男' OR gender ='女'),#检查
	seat INT UNIQUE,#唯一
	age INT DEFAULT 18,#默认约束
	majorId INT REFERENCES major(id)#外键
);

# 表级约束：除了非空、默认，其他的都支持
# 【constraint 约束名】 约束类型(字段名)
CREATE TABLE stuinfo(
	id INT,
	stuname VARCHAR(20),
	gender CHAR(1),
	seat INT,
	age INT,
	majorid INT,
	CONSTRAINT pk PRIMARY KEY(id),#主键
	CONSTRAINT uq UNIQUE(seat),#唯一键
	CONSTRAINT ck CHECK(gender ='男' OR gender  = '女'),#检查
	CONSTRAINT fk_stuinfo_major FOREIGN KEY(majorid) REFERENCES major(id)#外键
);

# 添加非空约束
ALTER TABLE stuinfo MODIFY COLUMN stuname VARCHAR(20)  NOT NULL;
# 添加默认约束
ALTER TABLE stuinfo MODIFY COLUMN age INT DEFAULT 18;
```

主键：不能为 null，最多一个，从表外键类型必须和主表的一致，一般是主表的主键和唯一列。先插入主表，先删除从表。

### 标识列，自增长，序号

标识列可以通过 SET auto_increment_increment=3;设置步长。可以通过 手动插入值，设置起始值

```sql
CREATE TABLE tab_identity(
	id INT AUTO_INCREMENT
);
```

## TCL  事务

一个或一组 sql 语句组成一个执行单元，这个执行单元要么全部执行，要么全部不执行。

```sql

SET autocommit=0;  #设置自动提交功能为禁用
START TRANSACTION; #开启事务

#编写一组事务的语句
UPDATE account SET balance = 1000 WHERE username='张无忌';
UPDATE account SET balance = 1000 WHERE username='赵敏';

ROLLBACK; #回滚事务 回滚也代表结束事务
commit; #结束事务

#演示savepoint 的使用
SET autocommit=0;
START TRANSACTION;
DELETE FROM account WHERE id=25;
SAVEPOINT a;#设置保存点
DELETE FROM account WHERE id=28;
ROLLBACK TO a;#回滚到保存点
```
