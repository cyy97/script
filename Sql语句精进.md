> 

# Mysql 语句精进

安装数据库，然后导入tigerfive.sql。 sql见文档结尾部分



MySQL示例数据库模式由以下表组成：

- `customers`: 存储客户的数据。
- `products`: 存储汽车的数据。
- `productLines`: 存储产品类别数据。
- `orders`: 存储客户订购的销售订单。
- `orderDetails`: 存储每个销售订单的订单产品数据项。
- `payments`: 存储客户订单的付款数据信息。
- `employees`: 存储所有员工信息以及组织结构，例如，直接上级(谁向谁报告工作)。
- `offices`: 存储销售处数据，类似于各个分公司



表关系如下：

![](https://i.loli.net/2019/06/22/5d0dea99ed40a88500.jpg)



**！！！ 请先复习下列基础语法，然后进行select的精进操作**

---

## SQL语句分类：

SQL语句主要分为三大类

#### DDL语句
DDL是数据定义语句,是对数据库内部的对象进行创建、删除、修改等操作的语句.create、drop、alter等(DBA常用)

#### DML语句
DML是数据操作语句,指对数据库表记录的基本操作，insert、update、delete、select等(开发常用)

#### DCL语句
DCL是数据控制语句,用于控制不同数据段直接的许可和访问级别的语句.定义了数据库、表、字段、用户的访问权限和安全级别.主要是grant、revoke等(DBA常用)

## 数据库基本操作

```
数据库的增删改查
```
### 库操作
**创建库** 
```
mysql > create database db1;

```
### **查看库**
```
mysql > show databases; 
mysql > show create databasee db1;  //更详细的查看库
```
**使用库**
```
mysql > use db1;
```
### 表操作
**创建表**
```
mysql > create tables 表名(字段 类型(修饰符))
```
**查看表**
```
查看所有表：mysql > show tables ;
查看表记录：mysql > show table status like '表名' ；
查看表结构：mysql > desc '表名' ；
查看表记录：mysql > select * from  '表名' ;
```
**修改表**
```
修改表名：mysql > rename table  '源表名'  to  '新表名' ；
```
### 字段记录操作
**字段**
```
添加字段：mysql > alter table '表名'  add '字段' '修饰符';
删除字段：mysql > alter table '表名'  drop '字段' ；
修改字段：mysql > alter table  '表名' change '旧字段' '新字段'  '修饰符';
```
**记录**
```
添加记录：insert into '表名(记录，记录)'  values (),(),();
        ：insert into '表名'  values  (),(),();
修改记录：update '表名' set 字段=''  where 字段='';
删除记录：delete from '表名' where  '字段'='' ;
```
### 查询操作
**查询**
```
简单查询：select * from '表名';
避免重复：select distinct '字段' from '表名';
条件查询：select 字段,字段 from 表名 where id<=5(条件);
四则运算查询：select id,dep_id,id*dep_id from company.employee5  where id<=5;
定义显示格式一：select id*dep_id as "id and dep_id's sum" from company.employee5  where id<=5;
定义显示格式：SELECT CONCAT(name, ' annual salary: ', salary*14)  AS Annual_salary FROM employee5;   //定义格式+四个运算，CONCAT是关键字
多条件：select '字段，字段‘ from '表名' WHERRE '条件一' AND '条件二';
关键字between and：
select  '字段，字段' from '表名'  where 字段  BETWEEN
'条件一' AND '条件二';


----------
排序查询：
select '字段' from  '表名' ORDER BY  '排序字段'；//字段后加DESC正序，ASC反序

限制查询的记录数：
select '字段' from '表名' ORDER BY  '字段，DESC|ACS' LIMIT  '数字'; //数字有两种的是(5,从初始位置到第五个)(2,5,第三个开始，共显示五个)

使用集合的查询：
select COUNT(*)  from '表名'；
select COUNT(*) FROM '表名' WHERE dep_id=101;
select MAX(salary) FROM '表名';
select MIN(salary) FROM '表名';
select AVG(salary) FROM '表名';
select SUM(salary) FROM '表名';
select SUM(salary) FROM '表名' WHERE dep_id=101;

分组查询：
select '字段' from  '表名' group by 字段;   //可参考下列面试题

模糊查询：
select '字段' from  '表名'  LIKE  '关键字'；

正则表达式查询：
select * from '表名' where '字段'  REGEXP '关键字'； 

```
### 删库
****
```
删库：drop database '库名'；
删表：drop table   '表名'
```

## 高级操作
### **连接数据库**
```
# mysql -uroot  -p  -h10.18.44.209  -p3306

授权
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```
### **修改数据库密码**
```
# vim  /etc/my.cnf 追加
validate_password=off
# systemctl restart mysqld
方法一：
mysql > SET PASSWORD FOR  user3@'localhost'='new_password';
mysql > flush privileges;
方法二：
mysql > update mysql.user set password=password('newpassword') where user='root';
mysql > flush privileges;
方式三：
mysql > set password for 'root'@'localhost'=password('newpassord');
```
### **编译安装需要参数**
```

[root@mysql-5.7.17 ~]# cmake . \
-DWITH_BOOST=boost_1_59_0/ \                    指定boost目录
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql \       指定安装目录
-DSYSCONFDIR=/etc \                             配置文件位置  
-DMYSQL_DATADIR=/usr/local/mysql/data \         数据目录（非常重要 错误日志、数据存放位置）
-DINSTALL_MANDIR=/usr/share/man \               man手册
-DMYSQL_TCP_PORT=3306 \                         默认端口3306
-DMYSQL_UNIX_ADDR=/tmp/mysql.sock \             sock文件位置，用来做网络通信的  
-DDEFAULT_CHARSET=utf8 \                        字符集的支持  默认
-DEXTRA_CHARSETS=all \                          字符集的支持  扩展支持
-DDEFAULT_COLLATION=utf8_general_ci \           字符集的支持   设定默认排序规则
-DWITH_READLINE=1 \                             可以上下泛历史命令  
-DWITH_SSL=system \                             使用证书登陆   （安全 但是影响传输速度）      
-DWITH_EMBEDDED_SERVER=1 \                      编译嵌入式服务器支持  
-DENABLED_LOCAL_INFILE=1 \                      支持从本机导入
-DWITH_INNOBASE_STORAGE_ENGINE=1                默认引擎   数据如何存储的库

```
### msyql中now()、sysdate()、curdate()区别

```
SELECT date_sub(date_sub(date_format(now(),'%y-%m-%d '),interval extract( day from now())-1 day),interval 1 month)
上个月第一天
select date_sub(date_sub(date_format(now(),'%y-%m-%d'),interval extract(  day from now()) day),interval 0 month)
上个月最后一天

select date_sub(date_sub(date_format(now(),'%y-%m-%d'),interval extract(  day from now())-1 day),interval 0 month)
这个月第一天
```




### **Mysql日志管理**
```
error log    错误日志        排错    /var/log/mysqld.log【默认开启】
bin log      二进制日志      备份    增量备份 DDL DML DCL
Relay log    中继日志        复制    接收 replication master 
slow log     慢查询日志       调优    查询时间超过指定值

```

```
Error Log
log-error=/var/log/mysqld.log
```
```
Binary Log(用于备份恢复数据)
产生binlog日志：
log-bin=/var/log/mysql-bin/slave2
serve-id=2

# mkdir /var/log/mysql-bin/slave2
#chmod mysql.mysql /var/log/mysql-bin/slave
#systemctl restart mysqld


1. 重启mysqld 会截断旧日志产生新的日志
2. 刷新日志会截断旧日志产生新的日志
mysql> flush logs 
3. 删除所有binlog（禁用）
mysql> reset master
4. 删除部分日志
mysql> PURGE BINARY LOGS TO 'mysql-bin.010';
mysql> PURGE BINARY LOGS BEFORE '2016-04-02 22:46:26';
5. 暂停binlog日志功能（仅对当前会话生效）
mysql> SET SQL_LOG_BIN=0;
mysql> SET SQL_LOG_BIN=1;

读取binlog日志：
# mysqlbinlog mysql.000002

按datetime读取：
# mysqlbinlog mysql.000002 --start-datetime="2018-12-05 10:02:56"
# mysqlbinlog mysql.000002 --stop-datetime="2018-12-05 11:02:54"
# mysqlbinlog mysql.000002 --start-datetime="2018-12-05 10:02:56" --stop-datetime="2018-12-05 11:02:54" 
 
按position读取：
# mysqlbinlog mysql.000002 --start-position=260
# mysqlbinlog mysql.000002 --stop-position=260
# mysqlbinlog mysql.000002 --start-position=260 --stop-position=930

查看带加密的binlong日志
mysqlbinlog --base64-output=decode-rows  -v  日志文件

根据binlog恢复数据：
根据时间点恢复数据
# mysqlbinlog --start-datetime='2014-11-25 11:56:54'  --stop-datetime='2014-11-25 11:57:41'  tiger-bin.000001 | mysql -u root -p1

根据位置点恢复数据
# mysqlbinlog --start-position 106 --stop-position  527 tiger-bin.000001 | mysql -u root -p1  

刷新bin-log日志:
#mysqladmin  flush-logs

去除binlog加密：
transaction_isolation=repeatable-read
binlog_format=mixed

```
```
慢查询：
slow_query_log=1
slow_query_log_file=/var/log/mysql-slow/slow.log
long_query_time=3

# mkdir /var/log/mysql-slow/
# chown mysql.mysql /var/log/mysql-slow/
# systemctl restart mysqld

查看慢查询日志
测试:BENCHMARK(count,expr)
SELECT BENCHMARK(50000000,2*3);

```
### mysql中TIMESTAMPDIFF和TIMESTAMPADD函数的用法

今天在处理工单的时候,其中的一个需求是某商品的发货时效(即下单时间和发货时间的时间差),接触到了TIMESTAMPDIFF函数

**TIMESTAMPDIFF**
```
TIMERSTAMPDIFF语法:
TIMERSTAMPDIFF(interval,datetime_expr1,datetime_expr2)

说明:
该函数是返回datetime_expr1和datetime_expr2之间的整数差,其中单位有interval参数决定,interval的常用参数有:
FRAC_SECOND   时间间隔是毫秒
SECOND        时间间隔是秒
MINUTE        时间间隔是分钟
HOUR          时间间隔是小时
DAY           时间间隔是天
WEEK          时间间隔是周
MOUTH         时间间隔是月
QUARTER       时间间隔是季度
YEAR          时间间隔是年
```
示例:
```
MariaDB [(none)]> select timestampdiff(hour,'2018-06-05 09:00:00','2018-06-15 09:00:00');
+-----------------------------------------------------------------+
| timestampdiff(hour,'2018-06-05 09:00:00','2018-06-15 09:00:00') |
+-----------------------------------------------------------------+
|                                                             240 |
+-----------------------------------------------------------------+
1 row in set (0.00 sec)

```
示例二:
```
MariaDB [(none)]> select timestampdiff(day,'2018-06-05 09:00:00','2018-06-15 09:00:00');
+----------------------------------------------------------------+
| timestampdiff(day,'2018-06-05 09:00:00','2018-06-15 09:00:00') |
+----------------------------------------------------------------+
|                                                             10 |
+----------------------------------------------------------------+
1 row in set (0.00 sec)

```
**TIMESTAMPADD**
```
TIMESTAMPADD语法:
TIMESTAMPADD(interval,int_expr,datetime_expr)

说明:
将整型表达式int_expr 添加到日期或日期时间表达式 datetime_expr中。式中的interval和TIMESTAMPDIFF中列举的取值是一样的。
```
示例:
```
MariaDB [(none)]> select timestampadd(second,86400,'2018-06-15 09:00:00');
+--------------------------------------------------+
| timestampadd(second,86400,'2018-06-15 09:00:00') |
+--------------------------------------------------+
| 2018-06-16 09:00:00                              |
+--------------------------------------------------+
1 row in set (0.00 sec)

```
### msyql中CASE WHEN语法
```
MySQL中case when语句，用于计算条件列表并返回多个可能表达式之一。
```
CASE具有两种格式：简单CASE函数将某个表达式与一组简单表达式进行比较以确定结果。CASE搜索函数计算一组布尔表达式以确定结果。两种都支持可选的ELSE函数。

1）简单CASE函数
语法如下：
```
CASE input_expression  
WHEN when_expression THEN  
    result_expression [...n ] [  
ELSE  
    else_result_expression  
END  
```
参数介绍
```
input_expression是使用简单 CASE 格式时所计算的表达式。Input_expression 是任何有效的 Microsoft SQL Server 表达式。
WHEN when_expression使用简单 CASE 格式时 input_expression 所比较的简单表达式。When_expression 是任意有效的 SQL Server 表达式。Input_expression 和每个 when_expression 的数据类型必须相同，或者是隐性转换。
占位符，表明可以使用多个 WHEN when_expression THEN result_expression 子句或 WHEN Boolean_expression THEN result_expression 子句。
THEN result_expression 当 input_expression = when_expression 取值为 TRUE，或者 Boolean_expression 取值 TRUE 时返回的表达式。
result expression 是任意有效的 SQL Server 表达式。
ELSE else_result_expression当比较运算取值不为 TRUE 时返回的表达式。如果省略此参数并且比较运算取值不为 TRUE，CASE 将返回 NULL 值。else_result_expression 是任意有效的 SQL Server 表达式。else_result_expression 和所有 result_expression 的数据类型必须相同，或者必须是隐性转换。

简单 CASE 函数：返回结果值介绍：

计算 input_expression，然后按指定顺序对每个 WHEN 子句的 input_expression = when_expression 进行计算。
返回第一个取值为 TRUE 的 (input_expression = when_expression) 的 result_expression。如果没有取值为 TRUE 的 input_expression = when_expression，则当指定 ELSE 子句时 SQL Server 将返回 else_result_expression；若没有指定 ELSE 子句，则返回 NULL 值。
```
2)CASE搜索函数
语法如下：
```sql
CASE  
WHEN Boolean_expression THEN  
    result_expression [...n ] [  
ELSE  
    else_result_expression  
END  

```
参数介绍：
```
WHEN Boolean_expression 使用 CASE 搜索格式时所计算的布尔表达式。Boolean_expression 是任意有效的布尔表达式。结果类型从 result_expressions 和可选 else_result_expression 的类型集合中返回最高的优先规则类型。有关更多信息，请参见数据类型的优先顺序。

CASE 搜索函数：返回结果值介绍：
按指定顺序为每个 WHEN 子句的 Boolean_expression 求值。返回第一个取值为 TRUE 的 Boolean_expression 的 result_expression。
如果没有取值为 TRUE 的 Boolean_expression，则当指定 ELSE 子句时 SQL Server 将返回 else_result_expression；若没有指定 ELSE 子句，则返回 NULL 值。
```
3)CASE WHEN例子介绍
1、仅带简单case的select语句
```
select 
    CASE good_type
WHEN '0' THEN
    '食品类'
WHEN '1' THEN
    '饮料类'
WHEN '2' THEN
    '日用品'
WHEN '3' THEN
    '鲜果类'
END AS good_type_now,
good_type,user_id,user_name
FROM 
    express.t_main_order
```
```
+---------------+-----------+---------+-----------+
| good_type_now | good_type | user_id | user_name |
+---------------+-----------+---------+-----------+
| 食品类        |         0 | 1       | tina      |
| 食品类        |         0 | 2       | tige      |
| 饮料类        |         1 | 3       | five      |
| 日用品        |         2 | 4       | wate      |
| 鲜果类        |         3 | 5       | fiww      |
| 鲜果类        |         3 | 6       | www       |
| 鲜果类        |         3 | 7       | wfiw      |
+---------------+-----------+---------+-----------+

```
2、使用带有简单CASE函数和CASE搜索函数的select语句
在select语句中，CASE搜索函数允许根据比较值
```sql
select 
    CASE 
WHEN good_type<2 
    THEN '<2' 
WHEN good_type>=2 AND good_type<3 
    THEN '>=2 && <3' 
    ELSE '>=3' END AS good_now_type,
good_type,user_id,user_name 
FROM t_main_order;
```
```
+---------------+-----------+---------+-----------+
| good_now_type | good_type | user_id | user_name |
+---------------+-----------+---------+-----------+
| <2            |         0 | 1       | tina      |
| <2            |         0 | 2       | tige      |
| <2            |         1 | 3       | five      |
| >=2 && <3     |         2 | 4       | wate      |
| >=3           |         3 | 5       | fiww      |
| >=3           |         3 | 6       | www       |
| >=3           |         3 | 7       | wfiw      |
+---------------+-----------+---------+-----------+

```
3)CASE的其他用法
```sql
select 
    CASE 
WHEN good_type<2 
    THEN '<2' 
WHEN good_type>=2 AND good_type<3 
    THEN '>=2 && <3' ELSE '>=3' 
END AS good_now_type,
    count(*) AS num_count 
FROM t_main_order 
GROUP BY good_now_type 
ORDER BY num_count;
```
```
+---------------+-----------+
| good_now_type | num_count |
+---------------+-----------+
| >=2 && <3     |         1 |
| <2            |         3 |
| >=3           |         3 |
+---------------+-----------+

```
### left join、right join、inner join的区别
left join(左联接) 返回包括左表中的所有记录和右表中联结字段相等的记录 
right join(右联接) 返回包括右表中的所有记录和左表中联结字段相等的记录
inner join(等值连接) 只返回两个表中联结字段相等的行

举例如下： 

--------------------------------------------
表A记录如下：
aID　　　　　aNum
1　　　　　a20050111
2　　　　　a20050112
3　　　　　a20050113
4　　　　　a20050114
5　　　　　a20050115

表B记录如下:
bID　　　　　bName
1　　　　　2006032401
2　　　　　2006032402
3　　　　　2006032403
4　　　　　2006032404
8　　　　　2006032408

--------------------------------------------

#### 1.left join
sql语句如下: 
select * from A
left join B 
on A.aID = B.bID

结果如下:
aID　　　　　aNum　　　　　bID　　　　　bName
1　　　　　a20050111　　　　1　　　　　2006032401
2　　　　　a20050112　　　　2　　　　　2006032402
3　　　　　a20050113　　　　3　　　　　2006032403
4　　　　　a20050114　　　　4　　　　　2006032404
5　　　　　a20050115　　　　NULL　　　　　NULL

（所影响的行数为 5 行）
结果说明:
left join是以A表的记录为基础的,A可以看成左表,B可以看成右表,left join是以左表为准的.
换句话说,左表(A)的记录将会全部表示出来,而右表(B)只会显示符合搜索条件的记录(例子中为: A.aID = B.bID).
B表记录不足的地方均为NULL.

--------------------------------------------
#### 2.right join
sql语句如下: 
select * from A
right join B 
on A.aID = B.bID

结果如下:
aID　　　　　aNum　　　　　bID　　　　　bName
1　　　　　a20050111　　　　1　　　　　2006032401
2　　　　　a20050112　　　　2　　　　　2006032402
3　　　　　a20050113　　　　3　　　　　2006032403
4　　　　　a20050114　　　　4　　　　　2006032404
NULL　　　　　NULL　　　　　8　　　　　2006032408

（所影响的行数为 5 行）
结果说明:
仔细观察一下,就会发现,和left join的结果刚好相反,这次是以右表(B)为基础的,A表不足的地方用NULL填充.

--------------------------------------------

#### 3.inner join
sql语句如下: 
select * from A
innerjoin B 
on A.aID = B.bID

结果如下:
aID　　　　　aNum　　　　　bID　　　　　bName
1　　　　　a20050111　　　　1　　　　　2006032401
2　　　　　a20050112　　　　2　　　　　2006032402
3　　　　　a20050113　　　　3　　　　　2006032403
4　　　　　a20050114　　　　4　　　　　2006032404

结果说明:
很明显,这里只显示出了 A.aID = B.bID的记录.这说明inner join并不以谁为基础,它只显示符合条件的记录.

--------------------------------------------
注: 
LEFT JOIN操作用于在任何的 FROM 子句中，组合来源表的记录。使用 LEFT JOIN 运算来创建一个左边外部联接。左边外部联接将包含了从第一个（左边）开始的两个表中的全部记录，即使在第二个（右边）表中并没有相符值的记录。

语法：FROM table1 LEFT JOIN table2 ON table1.field1 compopr table2.field2

说明：table1, table2参数用于指定要将记录组合的表的名称。
field1, field2参数指定被联接的字段的名称。且这些字段必须有相同的数据类型及包含相同类型的数据，但它们不需要有相同的名称。
compopr参数指定关系比较运算符："="， "<"， ">"， "<="， ">=" 或 "<>"。
如果在INNER JOIN操作中要联接包含Memo 数据类型或 OLE Object 数据类型数据的字段，将会发生错误. 

### 其他常用函数
```
DATE_FORMAT
GROUP BY
ORDER BY
LIMIT
AS
DATE
UNION
UNION ALL
AND
IN

```


## 数据库备份
### 基本概念

```
备份过程中必须考虑因素：
1. 数据的一致性
2. 服务的可用性

======================================

逻辑备份： 备份的是建表、建库、插入等操作所执行SQL语句（DDL DML DCL），适用于中小型数据库，效率相对较低。
mysqldump
binlog
mydumper
phpmyadmin

物理备份： 直接复制数据库文件，适用于大型数据库环境，不受存储引擎的限制，但不能恢复到不同的MySQL版本。
tar,cp
mysqlhotcopy  只能用于备份MyISAM。
xtrabackup
inbackup
lvm snapshot

======================================

完全备份
增量备份
差异备份
======================================
```
```


```





## SELECT精进操作开始

模拟企业实际数据库结构进行select练习

### 示例数据库表结构

```
CREATE TABLE `customers` (
  `customerNumber` int(11) NOT NULL,
  `customerName` varchar(50) NOT NULL,
  `contactLastName` varchar(50) NOT NULL,
  `contactFirstName` varchar(50) NOT NULL,
  `phone` varchar(50) NOT NULL,
  `addressLine1` varchar(50) NOT NULL,
  `addressLine2` varchar(50) DEFAULT NULL,
  `city` varchar(50) NOT NULL,
  `state` varchar(50) DEFAULT NULL,
  `postalCode` varchar(15) DEFAULT NULL,
  `country` varchar(50) NOT NULL,
  `salesRepEmployeeNumber` int(11) DEFAULT NULL,
  `creditLimit` decimal(10,2) DEFAULT NULL,
  PRIMARY KEY (`customerNumber`),
  KEY `salesRepEmployeeNumber` (`salesRepEmployeeNumber`),
  CONSTRAINT `customers_ibfk_1` FOREIGN KEY (`salesRepEmployeeNumber`) REFERENCES `employees` (`employeeNumber`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;


CREATE TABLE `employees` (
  `employeeNumber` int(11) NOT NULL,
  `lastName` varchar(50) NOT NULL,
  `firstName` varchar(50) NOT NULL,
  `extension` varchar(10) NOT NULL,
  `email` varchar(100) NOT NULL,
  `officeCode` varchar(10) NOT NULL,
  `reportsTo` int(11) DEFAULT NULL,
  `jobTitle` varchar(50) NOT NULL,
  PRIMARY KEY (`employeeNumber`),
  KEY `reportsTo` (`reportsTo`),
  KEY `officeCode` (`officeCode`),
  CONSTRAINT `employees_ibfk_1` FOREIGN KEY (`reportsTo`) REFERENCES `employees` (`employeeNumber`),
  CONSTRAINT `employees_ibfk_2` FOREIGN KEY (`officeCode`) REFERENCES `offices` (`officeCode`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;


CREATE TABLE `items` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `item_no` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=10 DEFAULT CHARSET=utf8;


CREATE TABLE `offices` (
  `officeCode` varchar(10) NOT NULL,
  `city` varchar(50) NOT NULL,
  `phone` varchar(50) NOT NULL,
  `addressLine1` varchar(50) NOT NULL,
  `addressLine2` varchar(50) DEFAULT NULL,
  `state` varchar(50) DEFAULT NULL,
  `country` varchar(50) NOT NULL,
  `postalCode` varchar(15) NOT NULL,
  `territory` varchar(10) NOT NULL,
  PRIMARY KEY (`officeCode`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;


CREATE TABLE `orderdetails` (
  `orderNumber` int(11) NOT NULL,
  `productCode` varchar(15) NOT NULL,
  `quantityOrdered` int(11) NOT NULL,
  `priceEach` decimal(10,2) NOT NULL,
  `orderLineNumber` smallint(6) NOT NULL,
  PRIMARY KEY (`orderNumber`,`productCode`),
  KEY `productCode` (`productCode`),
  CONSTRAINT `orderdetails_ibfk_1` FOREIGN KEY (`orderNumber`) REFERENCES `orders` (`orderNumber`),
  CONSTRAINT `orderdetails_ibfk_2` FOREIGN KEY (`productCode`) REFERENCES `products` (`productCode`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;


CREATE TABLE `orders` (
  `orderNumber` int(11) NOT NULL,
  `orderDate` date NOT NULL,
  `requiredDate` date NOT NULL,
  `shippedDate` date DEFAULT NULL,
  `status` varchar(15) NOT NULL,
  `comments` text,
  `customerNumber` int(11) NOT NULL,
  PRIMARY KEY (`orderNumber`),
  KEY `customerNumber` (`customerNumber`),
  CONSTRAINT `orders_ibfk_1` FOREIGN KEY (`customerNumber`) REFERENCES `customers` (`customerNumber`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;


CREATE TABLE `payments` (
  `customerNumber` int(11) NOT NULL,
  `checkNumber` varchar(50) NOT NULL,
  `paymentDate` date NOT NULL,
  `amount` decimal(10,2) NOT NULL,
  PRIMARY KEY (`customerNumber`,`checkNumber`),
  CONSTRAINT `payments_ibfk_1` FOREIGN KEY (`customerNumber`) REFERENCES `customers` (`customerNumber`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;


CREATE TABLE `productlines` (
  `productLine` varchar(50) NOT NULL,
  `textDescription` varchar(4000) DEFAULT NULL,
  `htmlDescription` mediumtext,
  `image` mediumblob,
  PRIMARY KEY (`productLine`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;


CREATE TABLE `products` (
  `productCode` varchar(15) NOT NULL DEFAULT '' COMMENT '产品代码',
  `productName` varchar(70) NOT NULL COMMENT '产品名称',
  `productLine` varchar(50) NOT NULL COMMENT '产品线',
  `productScale` varchar(10) NOT NULL,
  `productVendor` varchar(50) NOT NULL,
  `productDescription` text NOT NULL,
  `quantityInStock` smallint(6) NOT NULL COMMENT '库存',
  `buyPrice` decimal(10,2) NOT NULL COMMENT '价格',
  `MSRP` decimal(10,2) NOT NULL COMMENT '建议零售价',
  PRIMARY KEY (`productCode`),
  KEY `productLine` (`productLine`),
  CONSTRAINT `products_ibfk_1` FOREIGN KEY (`productLine`) REFERENCES `productlines` (`productLine`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;



DROP TABLE IF EXISTS `tokens`;
CREATE TABLE `tokens` (
  `s` varchar(6) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```



### 表关系

```
表关系
- customers`: 存储客户的数据。
- `products`: 存储汽车的数据。
- `productLines`: 存储产品类别数据。
- `orders`: 存储客户订购的销售订单。
- `orderdetails`: 存储每个销售订单的订单产品数据项。
- `payments`: 存储客户订单的付款数据信息。
- `employees`: 存储所有员工信息以及组织结构，例如，直接上级(谁向谁报告工作)。
- `offices`: 存储销售处数据，类似于各个分公司

productlines.productLine=products.productLine
products.productCode=orderdetails.productCode
orderdetails.orderNumber=orders.orderNumber
orders.customerNumber=customers.customerNumber
customers.customerNumber=payments.customerNumber
customers.salesRepEmployeeNumber=employees.employeeNumber
employees.officeCode=offices.officeCode


```



### 单表查询

```
过滤条件 where、AND、OR、BETWEEN、LIKE、IN、LIMIT、IS NULL

> show tables;
> show create table customers;                                   //查看表结构
> select * from customers limit 1 ;                              //用*记得加约束条件LIMIT

> select customerNumber,contactLastName,city,creditLimit from customers where customers.customerNumber>490 ;                                   //WHERE条件

> select customerNumber,contactLastName,city,creditLimit from customers where customers.city='London' ;                                        //WHERE条件

> select customerNumber,contactLastName,city,creditLimit from customers where customers.customerNumber between 480 and 490 ;                   //BETWEEN AND 查询

> select customerNumber,contactLastName,city,creditLimit from customers where customers.customerNumber>480 and customers.customerNumber<490 ;  //AND多条件查询

> select * from customers where customers.phone like '%13%' ;    //LIKE模糊查询

> select customerNumber,contactLastName,city,creditLimit from customers where customers.customerNumber=480 or customers.customerNumber=489;    //OR查询

> select customerNumber,contactLastName,city,creditLimit from customers where customers.customerNumber IN (489,481,486,487);                   //IN查询
> 

排序 ORDER BY
select customerNumber,contactLastName,city,creditLimit from customers where customers.customerNumber IN (489,481,486,487) ORDER BY customers.customerNumber;  //ORDER BY

分组 GROUP BY 、HAVING
SELECT status,count(orderNumber) FROM  orders GROUP BY status;   //GROUP BY分组
SELECT status,count(orderNumber) FROM  orders GROUP BY status having count(orderNumber) = 6;                                                             //HAVING
SELECT status FROM  orders GROUP BY status having count(orderNumber) = 6; 

```



### 联合查询（两表）

```
联合查询(多表查询) INNER JOIN、LEFT JOIN、RIGHT JOIN、
不带条件的联合查询
select * from productlines as cs left join products as ps on  cs.productLine=ps.productLine limit 1\G
select * from products left join orderdetails on products.productCode=orderdetails.productCode limit 1\G
 select * from orderdetails left join orders on orderdetails.orderNumber=orders.orderNumber limit 1 \G
select * from orders left join customers on orders.customerNumber=customers.customerNumber limit 1 \G 
select * from customers left join payments on customers.customerNumber=payments.customerNumber limit 1 \G
select * from customers left join employees on customers.salesRepEmployeeNumber=employees.employeeNumber  limit 1 \G
select * from employees left join offices on employees.officeCode=offices.officeCode limit 1 \G


带条件的联合查询
select * from employees left join offices on employees.officeCode=offices.officeCode where email like '%tiger%';


```



### 多表联合查询

```
select * from customers left join orders on orders.customerNumber=customers.customerNumber left join orderdetails on orderdetails.orderNumber=orders.orderNumber  where  orderdetails.productCode IN (select products.productCode from products where products.buyPrice>100);             //三表联合加子查询
```

