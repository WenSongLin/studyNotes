# MySQL

# 1. 数据库  DB

> database  数据库  存储不同 类型的数据。

```java
为什么要学习数据库? 项目中?
    用户--->页面
       ----->控制器(接收前台页面提交的数据)
       --->service(业务逻辑处理层)
       --->dao(数据持久层)
       --->DB(mysql oracle pot db2  sqlite)
  持久化保存用户数据，  
```

```java
数据库的分类:
   1. 从存储位置划分:
        1.1 基于磁盘(文件) --->mysql oracle sqlserver  持久化 IO 效率偏慢
        1.2 基于缓存----> redis   不能完全保证持久化  效率快
         
 mysql：
      db--->table--->字段 类型  数据
   2.从关系上划分:
       2.1 关系型数据库:  表与表  字段与字段之间  有关系  mysql oracle sqlserver
       2.2 非关系型数据库NOSQL(not only sql): key:value   redis       
```



```java
数据库系统:
   1. 数据库
   2. 硬件
   3. 软件: dbms 数据库管理系统软件。
   4. 人员  DBA  
```



# 2. DBMS

> 数据库管理系统。目前需要学习的是关系型数据库管理系统。 ==简称RDBMS.==  软件---> 服务端的程序。

```java
Oracle:  Scott/orcl  tiger  政府项目  
 9i  11G  12C   租赁   
```



# 3. MySQL

```java
mysql 8 
   常用: 5.5+  jdk:15  学习: 5.7
       
下载:  https://dev.mysql.com/downloads/windows/installer/8.0.html
用户名: root
密码:  root
端口号: 3306
    
 my.ini  mysql的核心配置文件。   
```



# 4. 操作

## 4.1 指令

> 客户端连接服务端。

> cmd通过命令行操作服务端， 配置mysql的环境  在path里面配置mysql的bin目录。

```bash
# TCP/IP: 连接特定计算机里面的软件程序。  ip 3306 root  root
# mysql -hip -uroot -p
# mysql -h127.0.0.1 -uroot -p  连接本机的服务: C:\Users\DELL>mysql -uroot -p

#mysql DBMS 里面的组成部分:
##   很多数据库DB
##    每个DB--->table--->字段  类型  数据
```

```bash
# mysql> show databases; 查看所有的数据库


# mysql> use mysql; 使用指定数据库

# mysql> show tables;  查看指定库里面所有的表
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| columns_priv              |
| db                        |
| engine_cost               |
| event

# mysql> desc user;  查看表结构

mysql> show create database mydb; # 查看创建的数据库的基本结构
+----------+-----------------------------------------------------------------+
| Database | Create Database                                                 |
+----------+-----------------------------------------------------------------+
| mydb     | CREATE DATABASE `mydb` /*!40100 DEFAULT CHARACTER SET latin1 */ |
+----------+-----------------------------------------------------------------+
```





## 4.2 SQL

> 结构化查询语言。 structured query  language

```mysql
# 分为4类:
1. DDL data definition language  数据定义语言  create alter  drop  truncate
2. DML 数据操作语言 insert  delete   update
3. DQL 数据查询语言 select
4. DCL 数据控制语言 start transaction  begin  commit rollback  grant
```





```mysql
mysql> select database();  # 查看当前正在操作的数据库
+------------+
| database() |
+------------+
| mysql      |
+------------+

select * from 表名;   * 通配的表里面所有的字段
select 字段,字段,.... from 表名; 
# mysql> select * from user;   关键字  数据  不区分大小写
mysql> select host,user from user;
+-----------+---------------+
| host      | user          |
+-----------+---------------+
| localhost | mysql.session |
| localhost | mysql.sys     |
| localhost | root          |
+-----------+---------------+
```



# 5. DDL

> 创建库    创建表    删库 删表      修改表结构

```mysql
create database 数据库名称 ; #名字无法修改的
# mysql> create database mydb

mysql> drop database mydb; # 删除指定的库
```



```mysql
create table 表名(
   字段名称1 字段类型 [约束],
   字段1 字段类型 [约束],
    ....
   字段n 字段类型 [约束]
);

# 创建用户信息表 tb_userinfo  t_userinfo  userinfo  
 # 必不可少的3个字段  id  create_time  update_time
create table tb_userinfo( 
  id  int(4),
  name varchar(20),
  age tinyint(2) unsigned,
  gender  char(1),
  birthday date,
  balance decimal(10,3),
  create_time  datetime,
  update_time datetime
);

属性: java程序class   基本  引用
映射关系:
   表--->class
   表字段--->类属性
   表字段类型--->类属性的数据类型
```



```mysql
mysql> desc tb_userinfo;
+-------------+---------------------+------+-----+---------+-------+
| Field       | Type                | Null | Key | Default | Extra |
+-------------+---------------------+------+-----+---------+-------+
| id          | int(4)              | YES  |     | NULL    |       |
| name        | varchar(20)         | YES  |     | NULL    |       |
| age         | tinyint(2) unsigned | YES  |     | NULL    |       |
| gender      | char(1)             | YES  |     | NULL    |       |
| birthday    | date                | YES  |     | NULL    |       |
| balance     | decimal(10,3)       | YES  |     | NULL    |       |
| create_time | datetime            | YES  |     | NULL    |       |
| update_time | datetime            | YES  |     | NULL    |       |
+-------------+---------------------+------+-----+---------+-------+
```



## ==5.1 数据类型==

> 1. 整数类型

```mysql
tinyint(m)==>byte  年龄 -128-127   unsigned 0-255 
  tinyint(1)---> 1  0 boolean 
int(m)==>int  id      11  int(4)---> 限制列宽  1000000
bigint(m)==>long  id  时间的毫秒数  
unsigned: 标识数值必须是一个正整数。
zerofill: 以0填充。   int(4) 0100 10000
```





> 2. 小数类型

```mysql
float/ double / BigDecimal 小数

float(m,n)  m: 限定小数总位数。  n: 限定小数点之后的位数
double(m,n)
decimal(m,n) 定点数  BigDecimal
```



> 3. 字符类型

```mysql
char/String
-- 存储字符串的数据     n: 限定字符个数 
char(n)
varchar(n) 255   

char(3)  vs varchar(3)
char(3): 'a__' 查询的时候 先trim 再展示数据   性别  手机号  身份证号 
varchar(3): 'a'  可变字符串

text
longtext
```



> 4. 日期类型

```mysql
-- 年月日 date
-- 时分秒 time
-- 年月日  时分秒  datetime  timestamp  bigint
-- 年 year(n) n=2/4  2021
```



## 5.2 alter

```mysql
-- 修改表结构
-- 1. 新增新的字段
alter table tb_userinfo add image varchar(100);
-- 2.删除指定字段
alter table tb_userinfo drop image;
-- 3. 修改字段名称/字段类型
alter table tb_userinfo change name username varchar(30);
-- 4. 修改字段类型
alter table tb_userinfo modify username char(30);

-- 5. 修改表名
alter table tb_userinfo rename userinfo;

-- 6. 删除表
drop table userinfo;
```





# ==6. DML==

> 1. 新增  insert     一次新增一条记录  1条

```mysql
-- 1. 对表的列全部赋值(不太推荐) 
insert into 表名 values (数据1,数据2....数据n),(数据1,数据2....数据n); 
insert into tb_userinfo values (1,'jim',20,'m','2020-01-01',40000,'2020-01-01 12:00:00','2020-01-01 12:00:00');

mysql> insert into tb_userinfo values (1,'jim',-10,'男','2020-01-01',40000,'2020-01-01 12:00:00','2020-01-01 12:00:00');
ERROR 1264 (22003): Out of range value for column 'age' at row 1

mysql> insert into tb_userinfo values (1,'jim',20,'男','2020-01-01',40000,'2020-01-01 12:00:00','2020-01-01 12:00:00');
ERROR 1366 (HY000): Incorrect string value: '\xC4\xD0' for column 'gender' at row 1
```

```mysql
-- 2. 对指定字段赋值
insert into 表名 (字段1,...字段n) values (数据1,...数据n),(数据1,...数据n);
insert into tb_userinfo (id,name,balance,create_time) values (2,'tom',49999,now());

mysql> insert into tb_userinfo (id,name,balance,create_time) values (2,'tom',49999,now()),(3,'lily',635325,now());
```





> 2. 删除  delete  >=0

```mysql
-- delete from 表名 [where 条件1 and/or ];  清空表数据  多行记录受影响
mysql> delete from tb_userinfo where name='jim';
mysql> delete from tb_userinfo where name='tom' or id=3;
```





> 3. 修改  update  >=0

```mysql
-- update 表名 set 字段1=新值1,...字段n=新值n [where 条件1 and/or ] ;
mysql> update tb_userinfo set age=18,gender='F';
mysql> update tb_userinfo set balance = 1665327,birthday='2021-10-10',update_time=now() where id=2;
```



> 不能存储中文

```mysql
mysql> alter database mydb character set utf8;
Query OK, 1 row affected (0.00 sec)

mysql> show create database mydb;
+----------+---------------------------------------------------------------+
| Database | Create Database                                               |
+----------+---------------------------------------------------------------+
| mydb     | CREATE DATABASE `mydb` /*!40100 DEFAULT CHARACTER SET utf8 */ |
+----------+---------------------------------------------------------------+

-- 用户表是在修改编码格式之前创建的 依然不能添加中文
mysql> insert into tb_userinfo (name) values ('张三');
ERROR 1366 (HY000): Incorrect string value: '\xD5\xC5\xC8\xFD' for column 'name' at row 1
mysql> create table a(name varchar(10));
Query OK, 0 rows affected (0.02 sec)

mysql> insert into a values ('张三');
Query OK, 1 row affected (0.01 sec)

mysql> select * from a;
+------+
| name |
+------+
| 张三 |
+------+
```



```mysql
-- 修改方式:  修改mysql的服务  my.ini
 66 default-character-set=utf8
 100 character-set-server=utf8
    
 重新加载核心配置文件----> 重启服务
mysql> show variables like '%character%';    
```



# 7. 可视化客户端

> navicat      sqlyog      mysqlfront

```mysql
navicat客户端连接mysql的服务:
   ip  3306
   用户名
   密码 
```



# 8. 约束

> 使用一些规则限制字段的数据，保证数据更加具有规范性。

```mysql
1. 行级约束  限定一个字段    not  null  unique   default  primary key
2. 表级约束  与多个字段有关 primary key  foreign key
```



> 1. 非空约束  not  null   字段不能为null

```mysql
create TABLE a(
 id int not null,
`name` VARCHAR(20) not null,
 age TINYINT(2)
);


alter TABLE a change name name varchar(20) not null;

insert into a (name) values ('jim1')
> 1364 - Field 'id' doesn't have a default value  
```



> 2. 唯一性约束 unique  保证字段值的唯一性

```mysql
-- 唯一性约束  unique  也称为唯一性索引 自带索引。查询效率较高

alter TABLE a change name name varchar(20) not null unique;

id	int(11)	NO	PRI		 
name	varchar(20)	yes	UNI		
age	tinyint(2)	YES			

insert into a (id,name) values (2,'jim')
> 1062 - Duplicate entry 'jim' for key 'name'
-- 排除null值 
```



> 默认约束  default  给字段默认数据。

```mysql
alter TABLE a CHANGE age age TINYINT(2) DEFAULT 18;

-- create TABLE a(
--  id int not null UNIQUE,
-- `name` VARCHAR(20) not null UNIQUE,
--  age TINYINT(2) default 18
-- );
```



> 主键约束   primary key==> not null+unique    列的数据必须是非空且唯一   自带索引

```mysql
-- 在一张表里面  有且只有1个主键约束。 哪一个列可以充当主键列? 任意一个字段都可以充当主键列
-- 使用最多： id   int/bigint /varchar   一般都是一个字段充当主键列
-- 特殊情况: 多个字段同时充当一个主键列 (联合主键 -->中间表)

-- 表设计的三大范式:
1. 列不可再分 保证列的原子性
    收货地址:  省份  城市  区   河南省郑州市高新区
2.  遵循第一范式基础之上  保证每行记录唯一性  一般都是结合主键约束。
3.  遵循第一,二范式基础之上  尽可能避免数据的冗余   排除外键列数据(主键列数据)
    订单表: 求总金额 buyNum*price 
```

```mysql
CREATE TABLE b(
 id int(3) PRIMARY KEY,
 username varchar(20) unique,
 age int
);

-- 当主键列的类型是整型的时候  一般都会结合 自增去使用  auto_increment
-- 初始值: 1   每次也是+1
-- CREATE TABLE b(
--  id int(3) PRIMARY KEY auto_increment,
--  username varchar(20) unique,
--  age int
-- );
alter TABLE b change id id int(3) auto_increment;  -- 设置为自动增长

-- 获得上一次自增的id
select LAST_INSERT_ID();

-- 设置自增的初始值为1000
alter table c auto_increment 1000;

-- 修改自增的步长 +5  修改全局的步长
set GLOBAL auto_increment_increment=5
```



```mysql
-- mysql维护主键列数据  varchar      uuid()
-- UUID
insert into a(id,name) VALUES (uuid(),'v');
select * from a;
```



```mysql
-- 联合主键  多个字段同时充当主键列   中间表
create TABLE a(
 id int, 
 `name` VARCHAR(20),
 age TINYINT(2) default 18,
 PRIMARY key (id,name)
);
-- 只要有一个字段不重复  就可以成功添加。
insert into a(id,name) VALUES (1,'a')
> 1062 - Duplicate entry '1-a' for key 'PRIMARY'  
```





> 外键约束  foreign key   有外键列 ==(不要使用它)==

```mysql
-- 前提: 有多张表。
-- 要求: 外键列的数据要严格参照主键列的数据。 外键列都是在子表/从表
-- 一对一
-- 一对多
-- 多对多
```



> 一对一

```mysql
create TABLE tb_userinfo(
 id int PRIMARY key auto_increment COMMENT'用户id',
 username varchar(20) unique COMMENT'用户name',
 age TINYINT(2) UNSIGNED,
 image varchar(100),
 birthday date,
 balance DECIMAL(10,3),
 create_time TIMESTAMP,
 update_time TIMESTAMP
);
-- mysql  表  java  类  映射

create TABLE tb_role(
	id int PRIMARY key auto_increment,
	rolename varchar(20),
	remark varchar(100),
	create_time TIMESTAMP,
	update_time TIMESTAMP
);

-- 给用户分配角色
-- 用户(从表)的角色(外键列)参照角色表(主表)的。
-- 外键列的数据要严格参照主键列的数据。
-- tb_userinfo rid(外键列)---> 严格参照 tb_role(id)
-- 添加外键约束 维护多表的关系

alter TABLE tb_userinfo 
 add CONSTRAINT FOREIGN key (rid) REFERENCES tb_role(id);
 
 tb_userinfo_ibfk_1	rid	mydb	tb_role	id	RESTRICT	RESTRICT

```

```mysql
-- 1. 新增子表记录
insert into tb_userinfo (username,age,rid) values ('admin',20,100)
> 1452 - Cannot add or update a child row: a foreign key constraint fails (`mydb`.`tb_userinfo`, CONSTRAINT `tb_userinfo_ibfk_1` FOREIGN KEY (`rid`) REFERENCES `tb_role` (`id`))

-- 2. 修改子表的记录
update tb_userinfo set rid =100 where id=5
> 1452 - Cannot add or update a child row: a foreign key constraint fails (`mydb`.`tb_userinfo`, CONSTRAINT `tb_userinfo_ibfk_1` FOREIGN KEY (`rid`) REFERENCES `tb_role` (`id`))

-- 3. 删除子表数据
delete from tb_userinfo where id=5;
```



```mysql
--  操作主表 insert update  不会影响子表
-- insert into tb_role (rolename) values ('aaa');
-- update tb_role set  remark = 'aaaaa' where id=3;

-- 1. 外键约束的特性: 
     1.1 restrict  删除/更新/新增 严格受限
     1.2 SET NULL  (前提: 外键列允许为null)   将外键列设置为null  删除主键列数据
     1.3 CASCADE 级联   删除主表以及遍历式删除子表的记录

-- 删除主表的数据(子表是否参照这个记录)
delete from tb_role where id=1
> 1451 - Cannot delete or update a parent row: a foreign key constraint fails (`mydb`.`tb_userinfo`, CONSTRAINT `tb_userinfo_ibfk_1` FOREIGN KEY (`rid`) REFERENCES `tb_role` (`id`))
```



> 外键列还是存在的  但是不需要添加外键约束。 弱化外键。 代码层面



> 一对多

```mysql
-- 一个用户有多个收货地址
-- 在地址表里面  uid  外键列  严格参照用户信息表的id的数据
```



> 多对多---> 中间表

```mysql
角色和权限
 tb_role_per
 
```



# 9. DQL



```mysql
select * from 表名;
-- 语法:
  SELECT 
    DISTINCT 指定字段
  FROM 
    表1,表2...表n
[
  WHERE 条件 -- 过滤下来符合条件的记录
  GROUP BY 字段1,字段2  -- 分组查询
  HAVING 条件 -- 对分组之后的记录进行过滤
  ORDER BY 字段1 ASC/DESC -- 根据指定字段进行升序或者降序排列
  LIMIT ?/?,?  -- 结果限定  分页查询
]    
```



## 1. WHERE 条件查询

```mysql
=、!=、<>、<、<=、>、>=；
BETWEEN…AND；是否满足一个区间范围  >=  <=
IN(set)；条件的集合
IS NULL；
AND； 连接多个条件的查询
OR；or  满足其中一个条件就可以
NOT；
```

```mysql
-- 查询学生性别为女，并且年龄15的记录
-- SELECT sid,sname,age FROM stu WHERE (gender='female' AND age=15);

-- 查询学号为S_1001，S_1002，S_1003的记录
-- SELECT * FROM stu WHERE (sid ='S_1001') OR (sid ='S_1002') OR sid ='S_1003';
-- SELECT * FROM stu WHERE sid in ('S_1001','S_1002','S_1003');

-- 查询学号不是S_1001，S_1002，S_1003的记录
-- SELECT * FROM stu WHERE sid !='S_1001' AND sid !='S_1002' AND sid !='S_1003';
-- SELECT * FROM stu WHERE  sid not in ('S_1001','S_1002','S_1003');

-- 查询年龄为null的记录
-- SELECT * FROM stu WHERE  age  is not null;

-- 查询年龄在20到40之间的学生记录
-- SELECT * FROM stu WHERE  age>=20 AND age<=40; 
-- SELECT * FROM stu WHERE  age BETWEEN 20 AND 40;

-- 查询性别非男的学生记录
-- SELECT * FROM stu WHERE gender!='male' or gender is null;
```



## 2. 模糊查询 like

```mysql
-- 搜索商品  模糊匹配
-- 查询姓名由5个字母构成的学生记录  通配符: _ 
-- SELECT * FROM stu WHERE sname like '_____';

-- 查询姓名以“z”开头的学生记录  通配符: %
-- SELECT * FROM stu WHERE sname LIKE 'z%';

-- 查询姓名中第2个字母为“i”的学生记录
-- SELECT * FROM stu WHERE sname LIKE '_i%';

-- 查询姓名中包含“a”字母的学生记录
SELECT * FROM stu WHERE sname LIKE '%a%';
```



## 3. 字段控制查询

> 去重  distinct

```mysql
-- 查询员工表里面所有的职位
-- DISTINCT 一般单列会多一些 
-- SELECT DISTINCT * FROM emp;
-- SELECT DISTINCT gender,AGE FROM STU ;
```



> null 值的判断

```mysql
-- 给员工发工资
-- 有一个字段 为null  执行算术运算  最后的结果还是为null
-- 判断: 为null 使用一个默认值进行替换  否则 就使用本身数据   ifnull(字段,默认值)

SELECT empno,ename, sal,comm,(sal+IFNULL(comm,0))  FROM emp;
```

> 别名操作

```mysql
-- 别名查询: 表 字段 AS
-- SELECT emp.empno,emp.ename, emp.sal,comm,(sal+IFNULL(comm,0))  total  FROM emp e;
```





## 4. 排序 order by

```mysql
-- 根据某个字段或者某些字段进行升序或者降序排序
-- 默认排序: 升序排列 ASC  降序:  DESC 
-- 4. 排序 ORDER BY
-- 查询所有学生记录，按年龄降序排序
-- SELECT * FROM STU WHERE gender='male' ORDER BY age DESC;
-- 查询所有雇员，按月薪降序排序，如果月薪相同时，按编号升序排序
-- SELECT * FROM emp ORDER BY sal DESC, empno DESC;

```



## 5. 分组查询 group by

```mysql
-- 组函数
COUNT()：统计指定列不为NULL的记录行数；
-- 统计表的记录数
MAX()：计算指定列的最大值，如果指定列是字符串类型，那么使用字符串排序运算；
MIN()：计算指定列的最小值，如果指定列是字符串类型，那么使用字符串排序运算；
SUM()：计算指定列的数值和，如果指定列类型不是数值类型，那么计算结果为0；
AVG()：计算指定列的平均值，如果指定列类型不是数值类型，那么计算结果为0；

-- count(字段)/count(*)/count(1)
-- 使用分组函数  结果有且只有一个
-- SELECT COUNT(*) totalCount,COUNT(mgr),COUNT(comm),count(1),COUNT(empno) from emp;
-- SELECT max(sal),min(sal),avg(sal),sum(sal)/count(*),sum(sal) from emp;
-- SELECT * from emp WHERE sal = (SELECT MAX(sal) from emp);

```



```mysql
-- 查询每个部门的部门编号和每个部门的工资和：
-- SELECT DISTINCT deptno FROM emp;
-- SELECT deptno,sum(sal) sum  FROM emp GROUP BY deptno;
-- 查询每个部门的部门编号以及每个部门的人数：
-- SELECT deptno,count(*) cout  FROM emp GROUP BY deptno;

-- 查询每个部门的部门编号以及每个部门员工工资大于1500的人数：
-- SELECT deptno,count(*) cout  FROM emp WHERE sal>1500 GROUP BY deptno;
```



> having

```mysql
-- 查询员工薪资>1500
-- where 与 having 的作用一样 都是过滤下来符合条件的记录
-- SELECT * from emp where sal>1500 ;
-- SELECT * from emp HAVING sal>1500 ;

-- 查询工资总和大于9000的部门编号以及工资和：
-- 1. WHERE 不能与组函数一块使用  having可以
-- 2. 位置: WHERE 在分组之前  having  分组之后  (对分组之后的数据进行过滤)

SELECT deptno,sum(sal) sum FROM emp GROUP BY deptno HAVING sum>9000 ORDER BY sum;
```



## 6. 关联查询

> 1. 等值连接

```mysql
-- 等值连接
-- 查询员工信息，要求显示员工号，姓名，月薪，部门名称 
-- 14*4  笛卡尔集的数据  过滤掉笛卡尔积的数据  条件 where/having
-- 规则: 2张表  至少有1个条件  3-->2  
SELECT 
 e.empno,e.ename,e.sal,d.dname
 FROM
 emp AS e,dept AS d 
 WHERE e.deptno=d.deptno;
```

> 2. 不等值连接

```mysql
-- 查询员工信息，要求显示：员工号，姓名，月薪，薪水的级别
-- 14*5=70 
 SELECT
  e.empno,e.ename,e.sal,s.GRADE
 FROM
 emp AS e,salgrade AS s
 WHERE e.sal BETWEEN s.LowSAL AND s.HISAL ORDER BY e.sal DESC;
```

```mysql
-- 查询员工信息， 展示员工基本信息以及员工所在的部门名称 以及薪水的级别
SELECT
  e.*,d.dname,s.grade
FROM
emp e,dept d,salgrade s
WHERE e.deptno=d.deptno AND e.sal BETWEEN s.LowSAL AND s.HISAL ORDER BY e.sal DESC;
```



> 3. 外连接

```mysql
-- 按部门统计员工数，部门号，部门名称，人数
-- SELECT
--  d.deptno,d.dname, COUNT(*) '每个部门的人数'
-- FROM
-- dept d , emp e
-- WHERE d.deptno=e.deptno
-- GROUP BY d.deptno;
-- 外连接
-- 内外连接 inner join on/where 等同于关联查询
-- 左外/右外连接  left join/right join  on
-- 以左表为基准 右表没有的数据使用null/0填充
-- 以右表为基准 左表没有的数据使用null/0填充
-- SELECT
--  d.deptno,d.dname, COUNT(*) '每个部门的人数'
-- FROM
-- dept d INNER JOIN  emp e on d.deptno=e.deptno where 1=1
-- GROUP BY d.deptno;

SELECT
 d.deptno,d.dname, COUNT(e.empno) '每个部门的人数'
FROM
dept d LEFT JOIN emp e on d.deptno=e.deptno where 1=1
GROUP BY d.deptno;

```



> 4. 自连接

```mysql
-- 自连接
-- 查询员工姓名和员工的老板的名称
-- emp: 看成多张表进行操作
-- 看条件定: 员工表e  老板表boss
-- SELECT
--  e.empno,e.ename,e.mgr,boss.empno AS bid, boss.ename AS bname
-- FROM
-- emp e ,emp AS boss
-- WHERE e.mgr=boss.empno;
-- 
-- SELECT
--  e.empno,e.ename,e.mgr,boss.empno AS bid, boss.ename AS bname
-- FROM
-- emp e  INNER JOIN emp AS boss
-- WHERE e.mgr=boss.empno;

-- SELECT
--  e.empno,e.ename,e.mgr,boss.empno AS bid, boss.ename AS bname
-- FROM
-- emp e LEFT JOIN emp AS boss
-- on e.mgr=boss.empno;
```



> 5. 关联查询与外连接混合使用

```mysql
-- 查询的员工基本信息:
-- 展示部门名称 上级领导的名称 薪资级别
SELECT
 e.empno,e.ename,e.sal,e.mgr,d.dname,b.ename '老板名称',s.GRADE
FROM
emp e LEFT JOIN emp AS b ON e.mgr=b.empno,
dept d,salgrade s 
WHERE  e.deptno=d.deptno AND e.sal BETWEEN s.LowSAL AND s.HISAL
ORDER BY e.sal;


SELECT
 e.empno,e.ename,e.sal,e.mgr,d.dname,b.ename '老板名称',s.GRADE
FROM
emp e LEFT JOIN emp AS b ON e.mgr=b.empno
LEFT JOIN dept d on e.deptno=d.deptno
LEFT JOIN salgrade s on e.sal BETWEEN s.LowSAL AND s.HISAL
ORDER BY e.sal;
```



> 6. 子查询

```mysql
-- 查询工资为20号部门平均工资的员工信息
-- SELECT avg(sal) FROM emp WHERE deptno=20;
-- SELECT * from emp WHERE sal=2175;
-- SELECT * from emp WHERE sal=(SELECT avg(sal) FROM emp WHERE deptno=20);
-- SELECT * from emp WHERE sal in(SELECT max(sal) from emp);
```



> 7. 集合运算

```mysql
-- union all
-- union 
-- 场景: 分表  mycat 
-- 用户表的信息: user1 user2  user3
-- 多个合并一起
-- UNION ALL 不会去除重复的行记录
-- UNION  会去除重复的行记录

-- DISTINCT
-- SELECT DISTINCT * from
-- (SELECT * from user1
-- UNION  ALL
-- SELECT * from user2
-- UNION  ALL
-- SELECT * from user3) AS a;

SELECT * from user1
UNION 
SELECT * from user2
UNION  
SELECT * from user3;
```



> 8. 分页查询

```mysql
-- 上一页(当前第几页-1)   下一页(当前第几页+1)  尾页totalPage  一共totalPage页  跳转到多少页
-- 每页展示的商品数量: size=20
-- 商品表里面90个商品--->有90行记录 count(*)
-- totalPage= 5页 
-- 分页查询员工表信息
-- 每页展示5个员工----> 14/5  3页
-- SELECT COUNT(*) FROM emp;
-- 第一页:  LIMIT size; 默认从第一条记录开始查看 查size条
select * from emp  LIMIT 0,5;
-- 第2页 LIMIT start,size; 从第start条开始查询  查size条  行记录有索引 1-0
select * from emp  LIMIT 5,5;

select * from emp  LIMIT 10,5;

-- pageSize=5
-- 用户只知道提交请求 page=1
select * from 表名  limit (page-1)*pageSize,pageSize;
```





# 10. 函数

```mysql
-- INSERT into aaa (name,create_time) values ('张三',UNIX_TIMESTAMP());
-- update aaa set name='张三丰', update_time=UNIX_TIMESTAMP()  where id = 1;
-- 格式化毫秒数数据---> yyyy-MM-dd HH:mm:ss
-- select id,name,create_time, FROM_UNIXTIME(create_time,'%Y-%m-%d %H:%i:%S') dateStr from aaa;

SELECT abs(-1);
SELECT ename, LOWER(ename) from emp;
```



# 11. 数据备份

```mysql
1. 物理备份
C:\ProgramData\MySQL\MySQL Server 5.7\Data

2. 命令行
导出:
C:\Users\DELL>mysqldump -uroot -proot mydb > D:\a.sql
C:\Users\DELL>mysqldump -uroot -proot mydb aaa user1 > D:\a.sql

导入: 获得连接状态下
mysql> source d:\a.sql

3. 客户端工具
```



# 12. 数据库引擎

```mysql
-- show VARIABLES like '%STORAGE%';
1. myisam
   1. 文件 一张表要通过3个文件维护
   2. 256TB  节约空间，速度较快(小而快) 不支持事务
   
2. INNODB
   1. 一张表需要2个文件维护  
   2. 64tb 
   3. 支持事务 索引
   
3. memory
    基于缓存 查询效率最快 
4. archive 
```



