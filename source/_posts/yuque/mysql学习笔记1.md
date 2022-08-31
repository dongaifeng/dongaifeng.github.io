---
title: mysql学习笔记1
urlname: zgto0n
date: '2019-11-11 14:13:03 +0800'
tags: []
categories: []
---

## my.ini  配置文件

服务端的端口号：port
安装目录：basedir
数据文件保存路径：datadir
![image.png](https://cdn.nlark.com/yuque/0/2019/png/462392/1573452989778-f8e69f40-fde3-414b-a55a-01effd656814.png#align=left&display=inline&height=367&name=image.png&originHeight=367&originWidth=457&size=162422&status=done&style=none&width=457)

开启   关闭
 stop mysql   / start mysql
登录/  退出
mysql -h localhost -P 3306 -u root -p  /   exit
环境变量
复制 bin 目录---右击计算机 ---属性---高级---环境变量---path
显示数据库
show database;
进入数据库 test
use test；
显示表
show tables  /   show tables from mysql  /   select database()  /   create table ()

### linux 下操作 mysql

#### 查看 mysql 数据库是否运行  

systemctl status mysqld

#### 进入数据库  

mysql -u 用户名 -p 密码

#### 查看数据库中的数据库  

show databases;

#### 显示数据库中的表  

show tables;

#### 添加用户  

create user 用户名称 identified by 密码

#### 授权某一个数据库的所有权限给指定用户  

grant all on 数据库名称 to 用户名称

## 查询

1  别名  SELECT ‘222’ AS  数字   /  SELECT name  姓名  FROM table
如果有特殊符号比如空格，#  就用引号括起来

2  去重操作
SELECT DISTINCT name FROM table

3   +  只能做运算   
被操作的数字   如果是字符串数字   则会转化成数字  
如果是字母或者   则转换成 0
如果是 null  跟任何拼接都是 null

4  拼接  concat
select concat(last_name, first_name) as  姓名

5  显示表结构： DESC  表名

6  判断数值为空
IFNULL(num, 0)    若果 num 为 null 的时候就输出 0

## 运算符  

<   >  =  !=   <>这是不等于   >=  <=  and   or  not（&&　 ||　　！）

### like： 模糊查询 要查询的字段 类似后面的条件

通配符
% 表示 0 个或者多个字符， 
\_  表示 任意单个字符

select \* from  table where name like "%a%" ;  选出名字里面包含 a 的
WHERE last*name LIKE '*$_%' ESCAPE '$';  ESCAPE 表是$后面的 _ 是字符  
WHERE last_name LIKE '_\_%';  也可使用 \ 专业字符

### between  and  在数值 1 和数值 2 之间的数字 包含临界值 不可调换顺序

```sql
SELECT
	*
FROM
	employees
WHERE
	employee_id BETWEEN 120 AND 100;
```

### in  找出 in 前面的列名中是 in 后面括号里的字符的，不支持通配符，类型必须一致

```sql
# 找出job_id是'IT_PROT' ,'AD_VP','AD_PRES' 的 员工名
SELECT
	last_name,
	job_id
FROM
	employees
WHERE
	job_id IN( 'IT_PROT' ,'AD_VP','AD_PRES');
```

```sql
#案例1：查询没有奖金的员工名和奖金率
SELECT
	last_name,
	commission_pct
FROM
	employees
WHERE
	commission_pct IS NULL;

  #案例1：查询有奖金的员工名和奖金率
SELECT
	last_name,
	commission_pct
FROM
	employees
WHERE
	commission_pct IS NOT NULL;
```

<=>安全等于。既可以判断 NULL 值，又可以判断普通的数值，可读性较低

### 排序查询

语法：
select 查询列表
from 表名
【where   筛选条件】
order by 排序的字段或表达式
asc 或者 desc;

asc 代表的是升序，可以省略,desc 代表的是降序

```sql
#1、按单个字段排序
SELECT * FROM employees ORDER BY salary DESC;

#案例：查询员工信息 按年薪升序
SELECT *,salary*12*(1+IFNULL(commission_pct,0)) 年薪
FROM employees
ORDER BY 年薪 ASC;

#案例：查询员工信息，要求先按工资降序，工资一样的再按employee_id升序
SELECT *
FROM employees
ORDER BY salary DESC,employee_id ASC;

#案例：查询员工名，并且按名字的长度降序
SELECT LENGTH(last_name),last_name
FROM employees
ORDER BY LENGTH(last_name) DESC;
```

LENGTH(name)函数 ： 输出 name 字段的长度

## 函数

### 常用函数

```sql
#1 length(str) 获取参数值的字节个数
SELECT LENGTH('john');

#2.concat() 拼接字符串
SELECT CONCAT(last_name,'_',first_name) 姓名 FROM employees;

#3.upper()大写,lower()小写
SELECT UPPER('john');
SELECT LOWER('joHn');

#4.substr(str, 开始下标, [截取个数])、substring()  截取从指定索引处后面所有字符
# 注意：索引从1开始
SELECT SUBSTR('李莫愁爱上了陆展元',7)  out_put;

#5.instr(str, 子串) 返回子串第一次出现的索引，如果找不到返回0
SELECT INSTR('杨不殷六侠悔爱上了殷六侠','殷') out_put;

#6.trim()  前后去空格
SELECT LENGTH(TRIM('    张翠山    ')) AS out_put;

#8.rpad(str, 要填充成的长度, 用这个填充) 用指定的字符实现右填充指定长度 lpad()左填充
SELECT RPAD('殷素素',12,'ab') AS out_put;

#9.replace(str, 要替换得, 替换成这个) 替换
SELECT REPLACE('周芷若周芷若周芷若周芷若张无忌爱上了周芷若','周芷若','赵敏') AS out_put;
```

### 数学函数

```sql
#round 四舍五入
SELECT ROUND(-1.55);

#ceil 向上取整,返回>=该参数的最小整数
SELECT CEIL(-1.02);

#floor 向下取整，返回<=该参数的最大整数
SELECT FLOOR(-9.99);

#truncate(num, 小数点后位数) 截断 小数点后几位
SELECT TRUNCATE(1.69999,1);

#mod(num, 除数)  取余
SELECT MOD(10,-3);
```

### 日期函数

```sql
#now 返回当前系统年月+时间
SELECT NOW();

#curdate 返回当前系统日期，不包含时间
SELECT CURDATE();

#curtime 返回当前时间，不包含日期
SELECT CURTIME();

#可以获取指定的部分，年、月、日、小时、分钟、秒
SELECT YEAR(NOW()) 年;
SELECT YEAR('1998-1-1') 年;
SELECT  YEAR(hiredate) 年 FROM employees;

SELECT MONTH(NOW()) 月;  11
SELECT MONTHNAME(NOW()) 月; 英文的

#str_to_date(date, 按这种格式去解析) 将字符通过指定的格式转换成日期
SELECT STR_TO_DATE('1998-3-2','%Y-%c-%d') AS out_put;  1998-03-02

#date_format(date, 解析成这种格式) 将日期转换成字符
SELECT DATE_FORMAT(NOW(),'%Y年%m月%d日') AS out_put;
```

### 流程控制函数

```sql
#1.if(判断条件, '返回true结果', '返回false结果')函数： 实现if else 的效果
SELECT IF(commission_pct IS NULL,'是null','不是null') AS 备注
FROM employees;

#2. case函数  实现switch case的效果
/*
case 要判断的字段或表达式
when 常量1 then 要显示的值1或语句1;
when 常量2 then 要显示的值2或语句2;
...
else 要显示的值n或语句n;
end
*/
SELECT salary 原始工资,department_id,
CASE department_id
WHEN 30 THEN salary*1.1
WHEN 40 THEN salary*1.2
ELSE salar
END AS 新工资
FROM employees;

// 另一种用法
SELECT salary,
CASE
WHEN salary>20000 THEN 'A'
WHEN salary>15000 THEN 'B'
WHEN salary>10000 THEN 'C'
ELSE 'D'
END AS 工资级别
FROM employees;
```

### 分组函数   用作统计数据的

sum 求和、avg 平均值、max 最大值 、min 最小值 、count 计算个数
特点：
1、sum、avg 一般用于处理数值型，max、min、count 可以处理任何类型
2、以上分组函数都忽略 null 值
3、可以和 distinct 搭配实现去重的运算
4、和分组函数一同查询的字段要求是 group by 后的字段

```sql
#1、简单 的使用
SELECT SUM(salary) FROM employees;
SELECT SUM(salary) 和,AVG(salary) 平均,MAX(salary) 最高,MIN(salary) 最低,COUNT(salary) 个数
FROM employees;

#4、和distinct搭配 实现去重
SELECT SUM(DISTINCT salary),SUM(salary) FROM employees;
# 统计不为空的行 的总数
SELECT COUNT(*) FROM employees
SELECT COUNT(1) FROM employees
```
