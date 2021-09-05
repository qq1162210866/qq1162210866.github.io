### 一些重要的SQL命令：

- **SELECT** - 从数据库中提取数据
- **UPDATE** - 更新数据库中的数据
- **DELETE** - 从数据库中删除数据
- **INSERT INTO** - 向数据库中插入新数据
- **CREATE DATABASE** - 创建新数据库
- **ALTER DATABASE** - 修改数据库
- **CREATE TABLE** - 创建新表
  - **NOT NULL** - 指示某列不能存储 NULL 值。
  - **UNIQUE** - 保证某列的每行必须有唯一的值。
  - **PRIMARY KEY** - NOT NULL 和 UNIQUE 的结合。确保某列（或两个列多个列的结合）有唯一标识，有助于更容易更快速地找到表中的一个特定的记录。
  - **FOREIGN KEY** - 保证一个表中的数据匹配另一个表中的值的参照完整性。
  - **CHECK** - 保证列中的值符合指定的条件。
  - **DEFAULT** - 规定没有给列赋值时的默认值。
- **ALTER TABLE** - 变更（改变）数据库表
- **DROP TABLE** - 删除表
- **CREATE INDEX** - 创建索引（搜索键）
- **DROP INDEX** - 删除索引

### 需要注意的点

* SQL 使用单引号来环绕文本值，如果是数值字段，请不要使用引号。
* 通过使用 SQL，可以为表名称或列名称指定别名。基本上，创建别名是为了让列名称的可读性更强。
* 更新一个包含索引的表需要比更新一个没有索引的表花费更多的时间，这是由于索引本身也需要更新。

### 关键字用法

* SELECT DISTINCT 语句。在表中，一个列可能会包含多个重复值，有时您也许希望仅仅列出不同（distinct）的值。

  DISTINCT 关键词用于返回唯一不同的值。

* WHERE语句。WHERE 子句用于提取那些满足指定条件的记录。

  * BETWEEN：在某个范围内`Select * from emp where sal between 1500 and 3000;`
  * LIKE：搜索某种模式`Select * from emp where ename like 'M%';`（通配符类似正则表达式）
  * MySQL 中使用 REGEXP 或 NOT REGEXP 运算符 (或 RLIKE 和 NOT RLIKE) 来操作正则表达式。`SELECT * FROM Websites WHERE name REGEXP '^[GFs]';`

  ![image-20210601165337182](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210601165337182.png)

  * IN：指定针对某个列的多个可能值`Select * from emp where sal in (5000,3000,1500);`

* AND & OR 运算符。如果第一个条件和第二个条件都成立，则 AND 运算符显示一条记录。如果第一个条件和第二个条件中只要有一个成立，则 OR 运算符显示一条记录。

* ORDER BY关键字。 关键字用于对结果集按照一个列或者多个列进行排序。(多列时，先排前面字段，再拍后面字段)ORDER BY 关键字默认按照升序对记录进行排序。如果需要按照降序对记录进行排序，您可以使用 DESC 关键字。`SELECT * FROM Websites ORDER BY alexa DESC;`

* SELECT TOP 子句。MYSQL中为LIMIT。`SELECT * FROM Websites LIMIT 2;`SELECT TOP 子句用于规定要返回的记录的数目。

* JOIN语句。SQL join 用于把来自两个或多个表的行结合起来。**on** 条件是在生成临时表时使用的条件，它不管 **on** 中的条件是否为真，都会返回左边表中的记录。**where** 条件是在临时表生成好后，再对临时表进行过滤的条件。

  * **INNER JOIN**：如果表中有至少一个匹配，则返回行
  * **LEFT JOIN**：即使右表中没有匹配，也从左表返回所有的行。LEFT JOIN语句后面的表被称为右表。
  * **RIGHT JOIN**：即使左表中没有匹配，也从右表返回所有的行
  * **FULL JOIN**：只要其中一个表中存在匹配，则返回行

* UNION 操作符合并两个或多个 SELECT 语句的结果。UNION 内部的每个 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每个 SELECT 语句中的列的顺序必须相同。UNION ALL用于将不同表中相同列中查询的数据展示出来；（包括重复数据）

* SELECT INTO 语句从一个表复制数据，然后把数据插入到另一个新表中。

* INSERT INTO SELECT 语句从一个表复制数据，然后把数据插入到一个已存在的表中。

* TRUNCATE TABLE 语句。仅仅删除表内的数据，但并不删除表本身。

* AUTO INCREMENT 字段。我们通常希望在每次插入新记录时，自动地创建主键字段的值。

```sql
  CREATE TABLE Persons
  (
  ID int NOT NULL AUTO_INCREMENT,
  LastName varchar(255) NOT NULL,
  FirstName varchar(255),
  Address varchar(255),
  City varchar(255),
  PRIMARY KEY (ID)
  )
```

### SQL函数

* AVG() 函数返回数值列的平均值。
* COUNT() 函数返回匹配指定条件的行数。
  * COUNT(column_name) 函数返回指定列的值的数目
  * COUNT(*) 函数返回表中的记录数
  * COUNT(DISTINCT column_name) 函数返回指定列的不同值的数目
* FIRST() 函数返回指定的列中第一个记录的值。
* LAST() 函数返回指定的列中最后一个记录的值。
* MAX() 函数返回指定列的最大值。
* MIN() 函数返回指定列的最小值。
* SUM() 函数返回数值列的总数。
* GROUP BY 语句用于结合聚合函数，根据一个或多个列对结果集进行分组。
* HAVING 子句。在 SQL 中增加 HAVING 子句原因是，WHERE 关键字无法与聚合函数一起使用。HAVING 子句可以让我们筛选分组后的各组数据。
* EXISTS 运算符用于判断查询子句是否有记录，如果有一条或多条记录存在返回 True，否则返回 False。
* UCASE() 函数把字段的值转换为大写。
* LCASE() 函数把字段的值转换为小写。
* MID() 函数用于从文本字段中提取字符。
* LEN() 函数返回文本字段中值的长度。
* ROUND() 函数用于把数值字段舍入为指定的小数位数。
* NOW() 函数返回当前系统的日期和时间。
* FORMAT() 函数用于对字段的显示进行格式化。
* DATEDIFF() 函数返回两个日期之间的天数。用前一个参数减去后一个参数。

## 缓存

\# Write your MySQL query statement below

select  count(distinct sender_id send_to_id)/count(distinct requester_id accepter_id) from FriendRequest,RequestAccepted



select product_id,sum(rest),sum(paid),sum(canceled),sum(refunded) from Invoice group by product_id;





select distinct

a.follower,

count(f.followee) as num

from 

(select distinct followee as follower

from follow 

where

followee in (select distinct follower from follow)) as a

left join follow as f on a.follower = f.followee

group by f.followee

order by a.follower







select distinct

a1.player_id

from Activity as a1,Activity as a2

where a1.event_date = a2.event_date-1





select 

b.book_id

from Orders as o

left join Books as b

on o.book_id = b.book_id

where datediff(2019-06-23,b.available_from)>30 

and datediff(o.dispatch_date,2018-06-23)>0

group by book_id