---
title: mysql学习笔记4
urlname: xv3uy5
date: '2019-11-21 09:08:49 +0800'
tags: []
categories: []
---

## 视图

虚拟表，和普通表一样使用，mysql5.1 版本出现的新特性，是通过表动态生成的数据
只保存 sql 逻辑，一般只作为查   不增删改。

#### 创建视图：create view 视图名 as 查询语句

```sql
CREATE VIEW my_view
AS
SELECT last_name,department_name,job_title
FROM employees e
JOIN departments d
ON e.department_id  = d.department_id
JOIN jobs j
ON j.job_id  = e.job_id;
```

#### 修改视图：create or replace view   视图名 as 查询语句

```sql
CREATE OR REPLACE VIEW v2
AS
SELECT AVG(salary) AS ag, `job_id`
FROM `employees`
GROUP BY job_id;

ALTER VIEW v2
AS
SELECT *
FROM `employees`
GROUP BY `job_id`
HAVING last_name LIKE '%a%'
```

#### 删除视图：drop view 视图名 1, 视图名 2,...;

```sql
DROP VIEW v1,v2;
```

#### 查看视图：

DESC myv3; 
SHOW CREATE VIEW myv3;

#### 更新视图

具备以下特点的视图不允许更新：

- 包含以下关键字的 sql 语句：分组函数、distinct、group  by、having、union 或者 union all
- Select 中包含子查询，jion 连接的
- where 子句的子查询引用了 from 子句中的表

### 变量

#### 系统变量：由系统定义，属于服务器层面

全局变量：需要添加 global 关键字，针对所有，但不能跨重启
会话变量：需要添加 session 关键字，如果不写，默认会话级别，当前会话有效

```sql
#①查看
SHOW GLOBAL VARIABLES; #全局
SHOW SESSION VARIABLES; #会话
SHOW SESSION VARIABLES LIKE '%char%';
#查看指定的会话变量的值
SELECT @@global.autocommit;
SELECT @@session.tx_isolation;
#赋值
SET @@global.autocommit=0;
SET GLOBAL autocommit=0;
SET @@session.tx_isolation='read-uncommitted';
SET SESSION tx_isolation='read-committed';
```

#### 自定义变量

用户变量：针对当前会话

```sql
#声明并初始化： SET @变量名=值; | SET @变量名:=值; SELECT @变量名:=值;
SET @var1="a";
SET @var2:="b";
SELECT @var3:="c";

#赋值 方法同声明变量一样 另外：SELECT 字段 INTO @变量名 FROM 表;

#查看使用： SELECT @变量名;

#局部变量声明
```

局部变量：仅仅在定义它的 begin end 块中有效，应用在 begin end 中的第一句话

- 声明
  - DECLARE 变量名   数据类型;
  - DECLARE 变量名   数据类型   DEFAULT 值;
- 赋值（更新变量的值）
  - SET 局部变量名=值;
  - SET 局部变量名:=值;
  - SELECT 局部变量名:=值;
  - SELECT 字段 INTO 具备变量名   FROM  表;
- 使用（查看变量的值）
  - SELECT 局部变量名;

```sql
#局部变量
DECLARE m INT DEFAULT 1;
DECLARE n INT DEFAULT 1;
DECLARE SUM INT;
SET SUM=m+n;
SELECT SUM;
```

## 存储过程

一组预先编译好的 SQL 语句的集合，理解成批处理语句

### 创建

CREATE  PROCEDURE  存储过程名(参数列表)
BEGIN
     存储过程体（一组合法的 SQL 语句）
END
要点：

- 参数列表包含三部分：参数模式   参数名   参数数据类型
  - 例如：in  my_name  varchar(20)
- 参数模式有三种：
  - in  传入的参数
  - out  返回的参数
  - inout  即可以作为传入也可以作为返回
- 如果存储过程体仅仅只有一句话，begin end 可以省略
- 存储过程的结尾可以使用 delimiter 重新设置结尾符号(;)。存储过程体中的每条 sql 语句的结尾要求必须加分号。
  - delimiter $  设置$  为结尾符号替代分号。

### 调用

CALL 存储过程名(实参列表);

```sql
#创建无参存储过程
DELIMITER $
CREATE PROCEDURE myp1()
BEGIN
	INSERT INTO admin(username,`password`)
	VALUES('john1','0000'),('lily','0000');
END $
#调用
CALL myp1()$

#创建存储过程实现，用户是否登录成功
CREATE PROCEDURE myp4(IN username VARCHAR(20),IN PASSWORD VARCHAR(20))
BEGIN
	DECLARE result INT DEFAULT 0;#声明局部变量并初始化
	SELECT COUNT(*) INTO result#赋值
	FROM admin
	WHERE admin.username = username
	AND admin.password = PASSWORD;

	SELECT IF(result>0,'成功','失败');#使用
END $
#调用
CALL myp3('张飞','8888')$

#根据输入的女神名，返回对应的男神名和魅力值
CREATE PROCEDURE myp7(IN beautyName VARCHAR(20),OUT boyName VARCHAR(20),OUT usercp INT)
BEGIN
	SELECT boys.boyname ,boys.usercp INTO boyname,usercp
	FROM boys
	RIGHT JOIN
	beauty b ON b.boyfriend_id = boys.id
	WHERE b.name=beautyName ;
END $
#调用
CALL myp7('小昭',@name,@cp)$
SELECT @name,@cp$

#传入a和b两个值，最终a和b都翻倍并返回
CREATE PROCEDURE myp8(INOUT a INT ,INOUT b INT)
BEGIN
	SET a=a*2;
	SET b=b*2;
END $
#调用
SET @m=10$
SET @n=20$
CALL myp8(@m,@n)$
SELECT @m,@n$
```

### 删除

```sql
DROP PROCEDURE p1; # 一次只能删除一个
SHOW CREATE PROCEDURE  myp2; #查看存储过程
```

## 函数

跟存储过程一样。
存储过程   可以有任意个返回值，  适合   批量插入，批量更新。
函数   有且只有一个返回值，  适合提交处理数据后返回一个结果。

### 创建

CREATE FUNCTION  函数名(参数列表)  RETURNS  返回类型
BEGIN
     函数体
END

- 参数列表包含两部分：参数名   参数数据类型
- 函数体   必须有 return  语句

```sql
#返回公司的员工个数
CREATE FUNCTION myf1() RETURNS INT
BEGIN
	DECLARE c INT DEFAULT 0;#定义局部变量
	SELECT COUNT(*) INTO c#赋值
	FROM employees;
	RETURN c;
END $

SELECT myf1()$ #调用

#根据部门名，返回该部门的平均工资
CREATE FUNCTION myf3(deptName VARCHAR(20)) RETURNS DOUBLE
BEGIN
	DECLARE sal DOUBLE ;
	SELECT AVG(salary) INTO sal
	FROM employees e
	JOIN departments d ON e.department_id = d.department_id
	WHERE d.department_name=deptName;
	RETURN sal;
END $

SELECT myf3('IT')$
SHOW CREATE FUNCTION myf3;  #查看函数
DROP FUNCTION myf3;  #删除函数
```

## 流程控制

###   分支结构

#### if 函数： SELECT if(条件，值 1，值 2)

#### case 结构

- 用法一：类似于 switch

case  表达式
  when  值 1  then  结果 1 或语句 1(如果是语句，需要加分号) 
  when  值 2  then  结果 2 或语句 2(如果是语句，需要加分号)
  ...
  else  结果 n 或语句 n(如果是语句，需要加分号)
  end 【case】（如果是放在 begin end 中需要加上 case，如果放在 select 后面不需要）

- 用法二：类似于多重 if

case 
  when  条件 1 then  结果 1 或语句 1(如果是语句，需要加分号) 
  when  条件 2 then  结果 2 或语句 2(如果是语句，需要加分号)
  ...
  else  结果 n 或语句 n(如果是语句，需要加分号)
  end 【case】（如果是放在 begin end 中需要加上 case，如果放在 select 后面不需要）

```sql
DELIMITER $
CREATE PROCEDURE test_case(IN score INT)
BEGIN
	CASE
	WHEN score>=90 AND score<=100 THEN SELECT "A";
	WHEN score>=80                THEN SELECT "B";
	WHEN score>=60		      			THEN SELECT "C";
	ELSE SELECT "D";
	END CASE;
END $
CALL test_case(99)$
```

#### if elseif 结构：只能应用在 begin end 中

if  条件 1  then  语句 1;
elseif  条件 2  then  语句 2;
....
else 语句 n;
end if;

```sql
CREATE FUNCTION test_if(score FLOAT) RETURNS CHAR
BEGIN
	DECLARE ch CHAR DEFAULT 'A';
	IF score>90 THEN SET ch='A';
	ELSEIF score>80 THEN SET ch='B';
	ELSEIF score>60 THEN SET ch='C';
	ELSET ch='D';
	END IF;
	RETURN ch;
END $
SELECT test_if(87)$
```

### 循环结构

三种：while  loop  repeat
iterate  类似于 continue  继续，结束本次循环，继续下一次
leave 类似于  break，跳出，结束当前所在的循环

#### while 语法：

【标签:】while 循环条件 do
         循环体;
end while【 标签】;

#### loop 语法：

【标签:】loop
         循环体;
end loop 【标签】;

#### repeat 语法：

【标签：】repeat
        循环体;
until 结束循环的条件
end repeat 【标签】;

```sql
#1.没有添加循环控制语句
#案例：批量插入，根据次数插入到admin表中多条记录
CREATE PROCEDURE pro_while1(IN insertCount INT)
BEGIN
	DECLARE i INT DEFAULT 1;
	WHILE i<=insertCount DO
		INSERT INTO admin(username,`password`) VALUES(CONCAT('Rose',i),'666');
		SET i=i+1;
	END WHILE;

END $
CALL pro_while1(100)$

#2.添加leave语句
#案例：批量插入，根据次数插入到admin表中多条记录，如果次数>20则停止
TRUNCATE TABLE admin$
DROP PROCEDURE test_while1$
CREATE PROCEDURE test_while1(IN insertCount INT)
BEGIN
	DECLARE i INT DEFAULT 1;
	a:WHILE i<=insertCount DO
		INSERT INTO admin(username,`password`) VALUES(CONCAT('xiaohua',i),'0000');
		IF i>=20 THEN LEAVE a;
		END IF;
		SET i=i+1;
	END WHILE a;
END $
CALL test_while1(100)$

#3.添加iterate语句
#案例：批量插入，根据次数插入到admin表中多条记录，只插入偶数次
TRUNCATE TABLE admin$
DROP PROCEDURE test_while1$
CREATE PROCEDURE test_while1(IN insertCount INT)
BEGIN
	DECLARE i INT DEFAULT 0;
	a:WHILE i<=insertCount DO
		SET i=i+1;

		IF MOD(i,2)!=0 THEN ITERATE a;
		END IF;

		INSERT INTO admin(username,`password`) VALUES(CONCAT('xiaohua',i),'0000');
	END WHILE a;
END $
CALL test_while1(100)$
```
