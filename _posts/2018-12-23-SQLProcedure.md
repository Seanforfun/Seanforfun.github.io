---
layout: post
title:  "MySql|Procedure存储过程"
date:   2018-12-23 10:13
author: Botao Xiao
categories: MySQL
comment: true
description: 存储过程我个人认为是一种解除业务和数据库操作之间的耦合的一种方式。同时存储过程只会在其第一次被调用的时候被编译，所以速度也会有所加快。
---
存储过程我个人认为是一种解除业务和数据库操作之间的耦合的一种方式。同时存储过程只会在其第一次被调用的时候被编译，所以速度也会有所加快。

### 存储过程 简单，安全，高性能
1. 存储过程简单来说，就是为以后的使用而保存的一条或多条MySQL语句的集合。
2. 我们将处理封装在容易使用的单元中，简化复杂的操作。
3. 在测试和开发的过程中我们遵循实用相同的存储过程，保证了数据的一致性。
4. 简化了对变动的管理，如果表名，列名或者逻辑业务发生了变化，我们只需要改变存储过程即可，使用它的人甚至东瀑布需要看到这样的变化。

#### 创建存储过程 CREATE PROCEDURE 存储过程名
1. 存储过程的添加，调用和删除
```SQL
# 创建一个存储过程，用于计算平均工资
CREATE PROCEDURE avgprice()
BEGIN
    SELECT AVG(salary) AS average_salary
    FROM INFO;
END;
# 调用存储过程
CALL avgprice()；
# 删除存储过程
DROP PROCEDURE avgprice;
# 如果存在就删除
DROP PROCEDURE avgsalary IF EXISTS;
```

#### 使用参数
所有的参数都应该以@开头，在存储过程中传入参数，而变量则是内存中的一个特定的位置。
```Sql
# 创建存储过程，声明变量，声明存储过程。
# IN:传递给存储过程
# OUT：从存储过程传出
# INOUT:对存储过程传入和传出
# INTO： 在BEGIN和END之间写出存储过程的代码，用来检索值，保存到相应的变量。通过INTO指定关键字。
CREATE PROCEDURE salarys(
OUT sl DECIMAL(8,2),
OUT sh DECIMAL(8,2),
OUT sa DECIMAL(8,2)
)
BEGIN
SELECT MAX(salary) INTO sh FROM info;
SELECT MIN(salary) INTO sl FROM info;
SELECT AVG(salary) INTO sa FROM info;
END;
# 调用存储过程，我个人感觉就是创建一块内存，将查找出的值通过存储过程的调用将值附给自己定义的变量。
# 定义了三个变量（实际上是三块内存空间，我们传入三个变量）
# 运行该程序实行存储过程。
CALL salarys(@lowsalary, @highsalary, @avgsalary);
# 通过SELECT语句调用存储过程
SELECT @lowsalary, @highsalary, @avgsalary;
+------------+-------------+------------+
| @lowsalary | @highsalary | @avgsalary |
+------------+-------------+------------+
| 10000.00   | 121212.00   | 77803.00   |
+------------+-------------+------------+
# 使用IN, OUT传入传出变量,接收一个参数，将计算所得到的值传入total；
CREATE PROCEDURE totalsalary(
IN month INT,
OUT total DOUBLE
)
BEGIN
SELECT SUM(info.salary * month)
INTO total
FROM info;
END;
# 调用存储过程,传入一个值，并将返回值传入@total，此处要注意，变量是一个值，不能返回多个值。
CALL totalsalary(12, @total);
# 获取@total的值
SELECT @total；
+---------+
| @total  |
+---------+
| 3734544 |
+---------+
```

#### 使用智能存储过程
所谓的智能存储过程我理解为就是创建一个函数，在其中使用一些逻辑判断。
```SQL
# 创建一个存储过程，在存储过程中定义了一些变量
# 根据接收到的布尔值决定是否应该进行扣税
CREATE PROCEDURE aftertaxsalary(
IN month INT,
IN taxable BOOLEAN,
IN id INT,
OUT result DOUBLE
)COMMENT 'Get after tax salary'
BEGIN
	-- Declear a variable used to save salary before tax --
	DECLARE total DOUBLE;
	-- Declare tax rate --
	DECLARE taxrate DOUBLE DEFAULT 0.8;
	SELECT SUM(month * salary)
	INTO total
	FROM info
	WHERE info.id = id;
	-- Add logical --
	IF taxable THEN
		SELECT total * taxrate INTO total;
	END IF;
	-- Assign result to variable --
	SELECT total INTO result;
END;
# 检查存储过程的创建信息
SHOW CREATE PROCEDURE aftertaxsalary;
# 显示所有的存储过程信息
SHOW PROCEDURE STATUS;
+------+----------------+-----------+----------------+---------------------+---------------------+---------------+----------------------+----------------------+----------------------+--------------------+
| Db   | Name           | Type      | Definer        | Modified            | Created             | Security_type | Comment              | character_set_client | collation_connection | Database Collation |
+------+----------------+-----------+----------------+---------------------+---------------------+---------------+----------------------+----------------------+----------------------+--------------------+
| test | aftertaxsalary | PROCEDURE | root@localhost | 2018-10-23 11:50:08 | 2018-10-23 11:50:08 | DEFINER       | Get after tax salary | utf8                 | utf8_general_ci      | utf8_general_ci    |
| test | salarys        | PROCEDURE | root@localhost | 2018-10-23 11:02:22 | 2018-10-23 11:02:22 | DEFINER       |                      | utf8                 | utf8_general_ci      | utf8_general_ci    |
| test | totalsalary    | PROCEDURE | root@localhost | 2018-10-23 11:22:30 | 2018-10-23 11:22:30 | DEFINER       |                      | utf8                 | utf8_general_ci      | utf8_general_ci    |
+------+----------------+-----------+----------------+---------------------+---------------------+---------------+----------------------+----------------------+----------------------+--------------------+
```

#### 游标 CURSOR，实际上就是存储过程中的FOR循环
使用游标对数据进行遍历
```sql
# 创建一个游标，打开，关闭游标。
CREATE PROCEDURE processsalary()
BEGIN
	DECLARE temp DOUBLE;
	-- 定义一个游标，用于遍历一个列的集合
	DECLARE salaryvalue CURSOR
	FOR SELECT info.salary FROM info;
	# 打开游标
	OPEN salaryvalue;
	FETCH salaryvalue INTO temp;
	# 关闭游标
	CLOSE salaryvalue;
END;
# 创建一个表，向其中加入数据
CREATE PROCEDURE insertvalue(
IN salary DOUBLE
)
BEGIN
	INSERT INTO copy1 (salary) values (salary);
END;
# 遍历一列数据
CREATE PROCEDURE iteratesalary()
BEGIN
	DECLARE o DOUBLE;
	DECLARE done BOOLEAN DEFAULT false;
	DECLARE cur CURSOR FOR SELECT salary FROM info;
	DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done = true;
	-- 如果表不存在，创建张新的表
	CREATE TABLE IF NOT EXISTS copy1(id INT(11) not null auto_increment primary key, salary DOUBLE);
	OPEN cur;
	REPEAT
		FETCH cur INTO o;
		-- 在存储过程中调用别的存储过程
		CALL insertvalue(o);
	-- 直到done=false我们结束循环
	UNTIL done END REPEAT;
	CLOSE cur;
END;
```

### 触发器TRIGGER
所有的MySQL语句和存储过程都需要被调用才能执行，但是我们可以通过触发器来让一些语句可以直接被调用。
1. 唯一的触发器名。
2. 触发器关联的表。
3. 触发器影响的活动（DELETE, UPDATE, INSERT）
4. 触发器执行的时机（BEFORE, AFTER）, BEFORE一般是用于净化数据。
```SQL
# 创建一个触发器，向log表中插入当前的时间。
CREATE PROCEDURE insertlog(
IN text VARCHAR(200)
)
BEGIN
    CREATE TABLE IF NOT EXISTS log(id INT NOT NULL PRIMARY KEY AUTO_INCREMENT, log varchar(200));
    INSERT INTO log (log) VALUES (text);
END;
# 创建一个触发器，在info表被修改后调用insertlog存储过程
CREATE TRIGGER infoupdate
AFTER UPDATE ON info
FOR EACH ROW
CALL insertlog(now());
# 使用DELETE触发器，可以调用一张OLD表，这张表是只读的，并不能修改。
# 我们可以利用delete触发器备份数据。
CREATE TRIGGER deleteinfo
AFTER DELETE ON info
FOR EACH ROW
BEGIN
    INSERT INTO log (log) VALUES (old.salary);
END;
+----+---------------------+
| id | log                 |
+----+---------------------+
|  1 | 2018-10-23 14:18:28 |
|  2 | 121212              |
+----+---------------------+
```

