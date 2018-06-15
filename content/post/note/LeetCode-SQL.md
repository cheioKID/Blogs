---
date: 2017-03-29
title: "LeetCode mySql"
tags:
    - mySql
    - Database Principles
categories:
    - SQL
comment: true
---

## 175 
```SQL  
SELECT FirstName, LAStName, City, State
FROM person 
LEFT JOIN address ON person.personID = address.personID;
```
> 左外连接，除去address中person没有的元祖 ```  

## 176 
```SQL
SELECT max(Salary) AS SecondHighestSalary
FROM Employee 
WHERE salary NOT IN (SELECT max(Salary) FROMEmployee);
```
> 子查询得到最大值，外层查询的最大值满足小于子查询的最大值 ```  

## 177 
```SQL
CREATE FUNCTION getNthHighestSalary(N INT)
RETURNS INT
BEGIN
	RETURN(SELECT DISTINCT c.salary
	FROM(SELECT a.salary AS salary, (SELECT 		COUNT(DISTINCT b.salary) FROM employee AS bWHERE b.salary >= a.salary) AS _row 
	FROM employee AS a) AS c
	WHEREc._row = n);
END 
```
> 和rank那题思路类似，如果表自身连接，有n条不重复的记录大于等于某个值，那么某个值就是第n高的记录，外层查询的范围是不相关子查询的结果，注意列的别名必须匹配外层查询的字段，而且子查询返回的表必须有别名才能在外层查询中查询，否则会报错
```SQL
CREATE FUNCTION getNthHighestSalary(N INT)
RETURNS INT
BEGIN
	SET n = n-1;
	RETURN(SELECT DISTINCT Salary 
	FROM employee ORDER BY salary DESC limit n,1);
END
```
> 因为limit的第一个参数是某行相对第一行的偏移量，所以函数参数n要先减一，返回结果为salary从高到低排序，第n行记录，只取一行 


## 178 
```SQL
SELECT Score,(SELECT COUNT(DISTINCT score) 
	FROM scores WHEREscore >= a.score) AS Rank
FROM scores AS a
ORDER BY score DESC
```
> 外层查询遍历表scores的记录，每一次获得一条记录的score值x，传递到相关子查询，相关子查询遍历得到n条不重复的记录的score值小于等于x，那么该score值x的rank就为n，因此得到score值对应的rank，然后按score值降序

## 180
```SQL  
SELECT DISTINCT a.num AS ConsecutiveNums
FROM logs AS a, logs AS b, logs AS c
WHERE a.id + 1 = b.id AND b.id + 1 = c.id 
AND a.num = b.num AND b.num = c.num
```
> a表与b表连接，因为是连续重复的，所以id值的差都为1，重复三行只需要表连接两次。如果记录连续重复超过3行，结果会出现重复所以要加distinct

## 181 
```SQL
SELECT a.name AS Employee
FROM Employee AS a, Employee AS b
WHERE a.ManagerId = b.Id AND a.Salary >b.salary;
```
> 等值连接，表自身连接 

## 182 
```SQL  
SELECT Email
FROM Person
GROUP BY Email
HAVING COUNT(Email) >= 2
```
> 查找重复的邮箱，只要count(邮箱)>=2

## 183 
```SQL  
SELECT name AS Customers
FROM Customers
WHERE Customers.ID NOT IN
(SELECT CustomerId FROM Orders);
```
> CustomersId在Order中无记录就是没点餐

## 184 
```SQL  
SELECT Department.name AS Department, 
Employee.name AS Employee, Salary
FROM Department, Employee
WHERE Employee.DepartmentId = Department.Id
AND (DepartmentId, Salary) IN
	(SELECTDepartmentId, MAX(Salary) FROM Employee GROUP BY DepartmentId);
```
> 子查询获得以部门号分组的salary最大值和对应的部门号，外层查询满足WHERE中条件的结果应在子查询结果集中 

## 185 
```SQL  
SELECT b.name AS Department, a.name ASEmployee, Salary
FROM employee AS a, department AS b
WHERE a.departmentId = b.id
AND (SELECT COUNT(DISTINCT salary) FROM employee WHERE departmentid = a.departmentid AND salary > a.salary) < 3
ORDER BY a.departmentID ASC, a.salary DESC
```
> 两表等值连接，相关子查询得到大于某个salary值的count不超过三，就可以得到前3高的salary，最后按departmentID升序(默认升序)，按salary降序 

## 196 
```SQL  
DELETE b FROM person AS a, person AS b
WHERE a.email = b.email AND a.id < b.id
```
> person表中email值相同的元组自身连接，如果连接后的记录存在email相同但是id不同，则删除id值更大的那一条记录  

## 197 
```SQL  
SELECT b.Id FROM weather AS a, weather AS b
WHERE TO_DAYS(a.date) - TO_DAYS(b.date) =-1 
AND a.temperature < b.temperature
```
> weather表自身连接，TO_DAYS函数将date类型转换成一个int类型的数，得出日期之间的偏移量，如果a中某记录比b的日期早一天，且a中记录的temperature小于b中temperature，则b中记录在结果集中

## 262 
```SQL  
SELECT request_at AS Day, 
ROUND(SUM(CASE WHEN status LIKE'cancelled_%' THEN 1 ELSE 0 END)/COUNT(status),2) AS 'Cancellation Rate'
FROM trips, users
WHERE client_id = users_id AND banned ='no' AND request_at BETWEEN '2013-10-01' AND '2013-10-03'
GROUP BY request_at 
```
> 先两表等值连接，满足条件的记录中分组计算取消比率，可以用sum函数（或者也可以用count(if())但是没试过），得出cancalled的记录数量，除以status的总数就是取消比率