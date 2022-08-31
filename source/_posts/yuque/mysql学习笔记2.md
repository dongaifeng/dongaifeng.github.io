---
title: mysql学习笔记2
urlname: guz9po
date: '2019-11-13 15:07:37 +0800'
tags: []
categories: []
---

## 分组查询，按照 group by 后面的字段分组

select  分组函数，某一列（必须和 group by 后面的字段 by  一致） from 表名称
where 筛选条件
group by 分组的字段
order by 排序的字段

```sql
/*
特点：
和分组函数一同查询的字段必须是group by后出现的字段
筛选分为两类：分组前筛选和分组后筛选
	分组前筛选	用关键字where 从原始表查
	分组后筛选	用having  从group by后的结果集查
分组可以按单个字段也可以按多个字段
*/
SELECT AVG(salary),job_id
FROM employees
GROUP BY job_id;

#案例2：查询有奖金的每个领导手下员工的平均工资
SELECT AVG(salary),manager_id
FROM employees
WHERE commission_pct IS NOT NULL
GROUP BY manager_id;


#3、分组后筛选 案例：查询哪个部门的员工个数>5  having 后面跟筛选条件 where只能放到from后面
SELECT COUNT(*),department_id
FROM employees
GROUP BY department_id
HAVING COUNT(*)>5;
```

## 多表查询

多表查询，查询的字段来自多个表

### 等值连接

① 多表等值连接的结果为多表的交集部分
② n 表连接，至少需要 n-1 个连接条件
③ 多表的顺序没有要求
④ 一般需要为表起别名
⑤ 可以搭配前面介绍的所有子句使用，比如排序、分组、筛选

```sql
#案例：查询员工名和对应的部门名
SELECT last_name,department_name
FROM employees AS e,departments AS d
WHERE e.`department_id`=d.`department_id`;

#案例：查询有奖金的员工名、部门名
SELECT last_name,department_name,commission_pct
FROM employees e,departments d
WHERE e.`department_id`=d.`department_id`
AND e.`commission_pct` IS NOT NULL;

#案例：查询每个工种的工种名和员工的个数，并且按员工个数降序
SELECT job_title,COUNT(*)
FROM employees e,jobs j
WHERE e.`job_id`=j.`job_id`
GROUP BY job_title
ORDER BY COUNT(*) DESC;

#案例：查询员工名、部门名和所在的城市
SELECT last_name,department_name,city
FROM employees e,departments d,locations l
WHERE e.`department_id`=d.`department_id`
AND d.`location_id`=l.`location_id`
AND city LIKE 's%'
ORDER BY department_name DESC;
```

### 非等值连接

```sql
#案例1：查询员工的工资和工资级别
SELECT salary,grade_level
FROM employees e,job_grades g
WHERE salary BETWEEN g.`lowest_sal` AND g.`highest_sal`
AND g.`grade_level`='A';
```

### 自连接

```sql
#案例：查询 员工名和上级的名称
# 自己查自己 给一张表起两个别名 相当于两个表
SELECT e.employee_id,e.last_name,m.employee_id,m.last_name
FROM employees e,employees m
WHERE e.`manager_id`=m.`employee_id`;
```

## 99 语法

多表查询

### 内连接

select  要查询的列   from  表 1 as  别名   inner join  表 2  as  别名
on  连接条件
where  筛选条件
group by  以这个字段进行分组
having  分组后的筛选条件
order by    排序字段

```sql
#案例.查询名字中包含e的员工名和工种名（添加筛选）
SELECT last_name,job_title
FROM employees e
INNER JOIN jobs j
ON e.`job_id`=  j.`job_id`
WHERE e.`last_name` LIKE '%e%';

#案例.查询哪个部门的员工个数>3的部门名和员工个数，并按个数降序（添加排序）
SELECT COUNT(*) 个数,department_name
FROM employees e
INNER JOIN departments d
ON e.`department_id`=d.`department_id`
GROUP BY department_name
HAVING COUNT(*)>3
ORDER BY COUNT(*) DESC;

 #查询工资级别的个数>20的个数，并且按工资级别降序  非等值连接
 SELECT COUNT(*),grade_level
 FROM employees e
 inner JOIN job_grades g
 ON e.`salary` BETWEEN g.`lowest_sal` AND g.`highest_sal`
 GROUP BY grade_level
 HAVING COUNT(*)>20
 ORDER BY grade_level DESC;

  #查询姓名中包含字符k的员工的名字、上级的名字  自连接
  SELECT a.last_name,b.last_name,a.salary
  FROM `employees` a
  INNER JOIN `employees` b
  ON a.`manager_id` = b.`employee_id`
  WHERE a.salary BETWEEN 10000 AND 20000
  AND a.last_name LIKE '%k%'
```

### 外连接

用于查询一个表中有，另一个表没有的记录
  特点：

- 外连接的查询结果为主表中的所有记录，如果从表中有和它匹配的，则显示匹配的值，否则显示 null。
- 左外连接，left join 左边的是主表，右外连接，right join 右边的是主表
- 左外和右外交换两个表的顺序，可以实现同样的效果
- 全外连接=内连接的结果+表 1 中有但表 2 没有的+表 2 中有但表 1 没有的

```sql
 #案例1：查询哪个部门没有员工  左外
 SELECT d.*,e.employee_id
 FROM departments d
 LEFT OUTER JOIN employees e
 ON d.`department_id` = e.`department_id`
 WHERE e.`employee_id` IS NULL;
 #右外
 SELECT d.*,e.employee_id
 FROM employees e
 RIGHT OUTER JOIN departments d
 ON d.`department_id` = e.`department_id`
 WHERE e.`employee_id` IS NULL;
 #全外
 USE girls;
 SELECT b.*,bo.*
 FROM beauty b
 FULL OUTER JOIN boys bo
 ON b.`boyfriend_id` = bo.id;
 #交叉连接
 SELECT b.*,bo.*
 FROM beauty b
 CROSS JOIN boys bo;
```

### 子查询

出现在其他语句中的 select 语句，称为子查询或内查询，外部的查询语句，称为主查询或外查询

#### where,having 后面

标量子查询(单行子查询)  列子查询(多行子查询)  行子查询(多行多列)
① 子查询放在小括号内
② 子查询一般放在条件的右侧
③ 标量子查询，一般搭配着单行操作符使用：> < >= <= = <>
列子查询，一般搭配着多行操作符使用：in、any/some、all

- in  等于后面括号内的列表中任意一个就行，not in 是不等于后面括号内任意一个
- any，some    a>any(10,20,30)   a 大于后面列表任意一个就行   可用 min 代替
- all   a>all(10,20,30) a 大于后面列表所有的   可用 max 代替

④ 子查询的执行优先于主查询执行，主查询的条件用到了子查询的结果

```sql
#案例：谁的工资比 Abel 高?
SELECT *
FROM employees
WHERE salary  >(
	SELECT salary
	FROM employees
	WHERE last_name = 'Abel'
);

#案例2：返回job_id与141号员工相同，salary比143号员工多的员工 姓名，job_id 和工资
SELECT last_name,job_id,salary
FROM employees
WHERE job_id = (
	SELECT job_id
	FROM employees
	WHERE employee_id = 141
) AND salary>(
	SELECT salary
	FROM employees
	WHERE employee_id = 143
);

#案例：返回location_id是1400或1700的部门中的所有员工姓名
SELECT last_name
FROM employees
WHERE department_id  <>ALL(
	SELECT DISTINCT department_id
	FROM departments
	WHERE location_id IN(1400,1700)
);

#案例：返回其它部门中比job_id为‘IT_PROG’部门所有工资都低的员工的员工号、姓名、job_id 以及salary
SELECT last_name,employee_id,job_id,salary
FROM employees
WHERE salary<(
	SELECT MIN( salary)
	FROM employees
	WHERE job_id = 'IT_PROG'
) AND job_id<>'IT_PROG';

#案例：查询员工编号最小并且工资最高的员工信息  行子查询（结果集一行多列或多行多列）
SELECT *
FROM employees
WHERE employee_id=(
	SELECT MIN(employee_id)
	FROM employees
)AND salary=(
	SELECT MAX(salary)
	FROM employees
);

SELECT *
FROM `employees`
WHERE (employee_id, salary) = (
	SELECT MIN(`employee_id`),MAX(salary)
	FROM `employees`
)
```

#### select 后面，仅仅支持标量子查询

```sql
 #案例：查询员工号=102的部门名
SELECT (
	SELECT department_name,e.department_id
	FROM departments d
	INNER JOIN employees e
	ON d.department_id=e.department_id
	WHERE e.employee_id=102
) 部门名;
```

#### from 后面，将子查询结果充当一张表，要求必须起别名

```sql
#案例：查询每个部门的平均工资的工资等级
SELECT  ag_dep.*,g.`grade_level`
FROM (
	SELECT AVG(salary) ag,department_id
	FROM employees
	GROUP BY department_id
) ag_dep
INNER JOIN job_grades g
ON ag_dep.ag BETWEEN lowest_sal AND highest_sal;
```

#### exists 后面（相关子查询）

```sql
#案例：查询有员工的部门名
SELECT department_name
FROM departments d
WHERE EXISTS(
	SELECT *
	FROM employees e
	WHERE d.`department_id`=e.`department_id`
);
```

### 分页查询

#### 语法

select 查询列表
from 表
【join type】 join 表 2
on 连接条件
where 筛选条件
group by 分组字段
having 分组后的筛选
order by 排序的字段】
limit 【offset,】size;
offset 要显示条目的起始索引（起始索引从 0 开始）
size 要显示的条目个数

#### 特点

    ①limit语句放在查询语句的最后
    ②公式： limit (page-1)*size,size;
    要显示的页数 page，每页的条目数size

```sql
SELECT * FROM  employees LIMIT 5;
```

### 联合查询

将多条查询语句的结果合并成一个结果

#### 语法

查询语句 1
union
查询语句 2
union
...

#### 特点

1、要求多条查询语句的查询列数是一致的！
2、要求多条查询语句的查询的每一列的类型和顺序最好一致
3、union 关键字默认去重，如果使用 union all 可以包含重复项

```sql
#案例：查询中国用户中男性的信息以及外国用户中年男性的用户信息
SELECT id,cname FROM t_ca WHERE csex='男'
UNION ALL
SELECT t_id,tname FROM t_ua WHERE tGender='male';
```
