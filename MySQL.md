# MySQL

### 基础知识



![数据库流程](E:\deng\image\数据库流程.png)

### 进入方式

* cmd: 
  * 配置环境变量C:\Program Files\MySQL\MySQL Server 5.5\bin
  * mysql -u账号 -p

~~~mysql


#drop table java成绩表;
USE dzp1;
DELETE FROM `java成绩表` WHERE NAME='李四';
#查询	-取别名
SELECT NAME m,results 分数 FROM `java成绩表` WHERE results>=60;

#删除
DELETE FROM students2

~~~

### sql语句分类

#### DQL

数据库查询语言（DQL）:对表的查询语句。select

#### DDL

数据库定义语言（DDL）:create、drop、alter、show

~~~mysql
#创建库
CREATE DATABASE dzp1 character set utf8;
USE dzp1;	#选中库
#查看库编码
show databases;
show create database dzp;
#修改库编码
alter database dzp character set utf8;

#创建表
CREATE TABLE java成绩表(
	姓名 VARCHAR(40),
	班级 VARCHAR(20),
	java成绩 FLOAT);
#查看表结构ALTER TABLE bank ADD birthday DATE;
desc table
	#查看表编码
	show create table bank;
#修改表
rename table bank to aaa;	#改表名
	#添加字段
	ALTER TABLE bank ADD gender VARCHAR(4);
	#删除字段
	ALTER TABLE bank DROP gender;
	#修改字段-变数据长度
	ALTER TABLE bank CHANGE username nname VARCHAR(50)
	#添加DATE字段
	ALTER TABLE bank ADD birthday DATE;#datetime带时分秒
~~~

#### DML

数据库操作语言（DML）:insert、delete、update、select

~~~mysql
#添加日期
UPDATE bank SET birthday='2000-2-28' WHERE id = 1;
#增加数据
INSERT INTO `java成绩表` VALUES('张三','dt55',90.5),('李四','mike',80),('王五','rose',30)
#插入数据2
INSERT INTO `java成绩表`(username,gender) values('zhangfei','男')
#插入数据3
INSERT INTO `java成绩表` set username='zhangfei',gender='男
#更新数据
UPDATE students SET id=0,stuName='zhangsan',weight=50 WHERE id=1;
~~~



### 数据库备份与还原

cmd：

~~~mysql
mysqldump -uroot -p dt_account > d:\dt_account_back.sql
#还原库 > 新建库 >
source d:\dt_account_back.sql
~~~

sqlyog: 打开sql > 全选F8



### 数据类型

字符串：varchar

整数：tinyint、int、bigint

小数：float

* 关键字

  * 默认值：default

  * 主键：primary key，不能重复，唯一

  * 自动增长：auto_increment ，尽量作用于int

  * 唯一键：unique

    

~~~mysql
create table students(
	id int(20) auto_increment primary key comment '学生编号'）,
    gender varchar(2) default '男' comment '性别',	#设置默认
    className varchar(40) not null comment '班级'，#设置非空	
    phoneNum varchar(20) unique comment '手机号'	#唯一
#清空表，自增重置
truncate table students
  
~~~



### 函数

#### 排序

order by

* 升序：asc
* 降序：desc

~~~mysql
#降序
SELECT * FROM users ORDER BY idcard DESC
#升序
SELECT * FROM users ORDER BY idcard ASC
~~~

#### 聚合函数

select  函数名(字段）from 表名

~~~mysql
#找最大（最小）值
SELECT MAX(javaScore) 最高分 FROM users;
SELECT MIN(javaScore) 最低分 FROM users;
#求和
SELECT SUM(javaScore) 最低分 FROM users;
#求平均
SELECT AVG(javaScore) 最低分 FROM users;
#统计记录
SELECT COUNT(*) 最低分 FROM users;
~~~

#### 时间日期

![时间日期](E:\deng\image\时间日期.png)

~~~mysql
SELECT personName,DATE_FORMAT(birthday,'%Y-%m-%d %H:%i:%s') FROM `person`;

~~~



#### 数学函数

~~~mysql
select ceil();	#向上取整
select floor();	#向下取整
select rand()	#（0,1）的小数
~~~

### 查询

#### 同时查多条记录

~~~mysql
select * from emp where id in(1,2,4)
~~~

#### 分组查询

group by	-	having

~~~mysql
#group by 后应该接 having
SELECT goodCategory FROM goods GROUP BY goodCategory having goodCategory='数码'
~~~

#### 分页

limit 起始下标,每页数据量

~~~mysql
SELECT * FROM goods LIMIT (pageNo-1)*pageSize,pagesize;
~~~

#### 子查询

~~~mysql
SELECT u.`username`,temp.minscore FROM users u,(SELECT MIN(javaScore) minscore FROM users) temp WHERE u.`javaScore`=temp.minscore

~~~

#### 模糊查询

like '%王%'  分前后左右

### 多表查询

* union 自动去重，合并两个查询结果
  * union all 不去重

~~~mysql
#select * from 表n where 条件 
SELECT u.`username`,temp.minscore FROM users u,(SELECT MIN(javaScore) minscore FROM users) temp WHERE u.`javaScore`=temp.minscore
~~~

#### 内连接

表1 inner join 表2 on 条件(多表间有关联的部分) where 条件（无关联的部分）

~~~mysql
SELECT * FROM book b INNER JOIN publisher p ON b.`P_ID`=p.`P_ID` WHERE p.`name`='长江传媒'

~~~

#### 外连接

~~~mysql
#左外连接-以左表为主
selectSELECT * FROM book b INNER JOIN publisher p ON b.`P_ID`=p.`P_ID`
~~~





### 其它命令

#### 复制表

~~~mysql
create table aaa(
	select * from publisher
)
~~~

#### 插入数据

insert into aaa select *from aaa where id =3;





### 视图

#### 创建视图

~~~mysql
#创建视图
CREATE VIEW view_all 
	AS SELECT e.ename,e.salary,e.phone,d.deptname
	FROM dept d INNER JOIN emp e ON d.id = e.deptid	
	
#删除视图
DROP VIEW view_all
~~~

### 数据库建模

file-->new Model-->physical data modal





### 事务

* 增删改需满足事务性

#### 事务处理

savepoint test1

rollback to test1

#### 事物的ACID属性

* 最终为了保持一致性

![事务的ACID属性](E:\deng\image\事务的ACID属性.png)

#### 锁的机制

为了解决并发访问时数据不一致问题，给数据加锁时需考虑"粒度"的问题：

一般情况下，锁的粒度越小，效率越高。粒度越大，效率越低。大部分情况用行级锁



1、打开mysql的命令行，将自动提交事务给关闭

```sql
--查看是否是自动提交 1表示开启，0表示关闭
select @@autocommit;
--设置关闭
set autocommit = 0;
```

2、数据准备

```sql
--创建数据库
create database tran;
--切换数据库 两个窗口都执行
use tran;
--准备数据
 create table psn(id int primary key,name varchar(10)) engine=innodb;
--插入数据
insert into psn values(1,'zhangsan');
insert into psn values(2,'lisi');
insert into psn values(3,'wangwu');
commit;
```

3、测试事务

```sql
--事务包含四个隔离级别：从上往下，隔离级别越来越高，意味着数据越来越安全
read uncommitted; 	--读未提交
read commited;		--读已提交
repeatable read;	--可重复读
(seriable)			--序列化执行，串行执行
--产生数据不一致的情况：
脏读
不可重复读
幻读
```

| 隔离级别 | 异常情况 |            | 异常情况 |
| -------- | -------- | ---------- | -------- |
| 读未提交 | 脏读     | 不可重复读 | 幻读     |
| 读已提交 |          | 不可重复读 | 幻读     |
| 可重复读 |          |            | 幻读     |
| 序列化   |          |            |          |



4、测试1：脏读 read uncommitted

```sql
set session transaction isolation level read uncommitted;--设置隔离级别
A:start transaction;--开启事务
A:select * from psn;
B:start transaction;
B:select * from psn;

A:update psn set name='msb';--更新操作
A:selecet * from psn
B:select * from psn;  --读取的结果msb。产生脏读，因为A事务并没有commit，读取到了不存在的数据
A:commit;
B:select * from psn; --读取的数据是msb,因为A事务已经commit，数据永久的被修改
```

5、测试2：当使用read committed的时候，就不会出现脏读的情况了，当时会出现不可重复读的问题

```sql
set session transaction isolation level read committed;
A:start transaction;
A:select * from psn;
B:start transaction;
B:select * from psn;
--执行到此处的时候发现，两个窗口读取的数据是一致的
A:update psn set name ='zhangsan' where id = 1;
A:select * from psn;
B:select * from psn;
--执行到此处发现两个窗口读取的数据不一致，B窗口中读取不到更新的数据
A:commit;
A:select * from psn;--读取到更新的数据
B:select * from psn;--也读取到更新的数据
--发现同一个事务中多次读取数据出现不一致的情况
```

6、测试3：当使用repeatable read的时候(按照上面的步骤操作)，就不会出现不可重复读的问题，但是会出现幻读的问题

```sql
set session transaction isolation level repeatable read;
A:start transaction; --开始事务
A:select * from psn;
B:start transaction;
B:select * from psn;
--此时两个窗口读取的数据是一致的
A:insert into psn values(4,'sisi');
A:commit;
A:select * from psn;--读取到添加的数据
B:select * from psn;--读取不到添加的数据
B:insert into psn values(4,'sisi');--报错，无法插入数据
--此时发现读取不到数据，但是在插入的时候不允许插入，出现了幻读，设置更高级别的隔离级别即可解决
```



总结：

​	现在学习的是数据库级别的事务，需要掌握的就是事务的隔离级别和产生的数据不一致的情况

后续会学习声明式事务及事务的传播特性以及分布式事务



### 索引

create index iname on table book(id);

drop index iname



不同的存储引擎，数据文件和索引文件存放的位置是不同的，因此有了分类：
聚簇索引：数据和文件放在一起：innodb
.frm：存放的是表结构
ibd：存放数据文件和索引文件
注意：mysql的innodb存储引擎默认情况下会把所有的数据文件放到表空间中，不会为每一个单独的表保存一份数据文件，如果需要将每一个表单独使用文件保存，设置如下属性：set global innodb_file_per table=on；非聚簇索引：数据和索引单独一个文件：MyISAM



#### 基础知识

![索引基础](E:\deng\image\索引基础.png)

![数据库流程](E:\deng\image\mysql架构图.png)



#### 数据结构

为什么最后选择了B+树

哈希表：

* 适合等值查询
* 表中数据为无序数据（范围查找时浪费时间）,而企业中较多范围查询
* 且使用时需将全部数据加载到内存

树

**AVL树**插入元素时会进行1到N次的旋转，严重影响插入的性能

**红黑树**损失了部分查询性能，来提高插入性能

* 树深无法控制或插入数据的性能较低（旋转树）

**B树**

磁盘块中包含了指针、键值和数据

当data占用数据较大时，会导致树身增大，增大了查询时磁盘的io次数，进而影响查询性能

![B树](E:\deng\image\B树.png)





##### B+tree

每个磁盘块包含更多的节点，降低了树的高度，同时区间变多，加快了检索速度

所有叶子节点间是链式结构，因此既可以根据主键范围查找和分页查找，也可以从根节点随机查找。

![B+树](E:\deng\image\B+树.png)

主键最好是自增的，比如3,4存储在一个磁盘块里，你手动输入了一个3.5，这会导致页分裂。





**InnoDB--B+Tree**

叶子节点直接放置数据

**MYISAM--B+Tree**

叶子节点存放指针





#### 分类

##### 主键索引

主键

-主键是一种唯一性索引，但它必须定为PRIMARY KEY，每个表只能有一个主键。

如果没有会生成rowid

##### 唯一索引

索引列的所有值都只能出现一次，即必须唯一，值可以为空。

##### 普通索引

* 普通索引会导致回表问题，遍历2次B+树
  * 使用**覆盖索引**优化
  * select id from emp where name ="";
  * select id 而不是*，name的B+树中就有id，直接找到，不会发生回表

##### 全文索引

MYISAM支持，Innodb在5.6后支持

全文索引的索引类型为FULTEXT.全文索引可以在varchar、char、text类型的列上创建组合索引

##### 组合索引

经常查询的列建组合索引

涉及**最左匹配**

​	考虑空间问题，age索引占据的空间小于name索引

#### 面试难点

##### 回表

比如用创建索引的是其它字段时，叶子节点中放的是主键，再根据主键索引找数据

经过两次遍历 

##### 覆盖索引

* 使用**覆盖索引**优化
* select id from emp where name ="";
* select id 而不是*，name的B+树中就有id，直接找到，不会发生回表

##### 最左前缀



##### 索引下推

select * from table where name="zhangsan" and age=10

普通情况下先通过张三找到对应的主键，再通过主键找到age返回信息

索引下推时，匹配name的同时匹配age=10的情况，只将"zhangsan" and age=10的数据主键返回







![存储引擎](E:\deng\image\存储引擎.png)

#### 索引维护

![索引维护](E:\deng\image\索引维护.png)



### 日志

**Redo LOG**——innodb存储引擎的日志文件

保证持久性

![小黑板](E:\deng\image\小黑板.png)

![日志](E:\deng\image\日志.png)

**Undo log**——回滚日志

实现事务原子性

在操作任何数据之前，首先将数据备份到一个地方（这个存储数据备份的地方称为UndoLog）。然后进行数据的修改。

如果出现了错误或者用户执行了ROLLBACK语句，系统可以利用Undo Log中的备份将数据恢复到事务开始之前的状态
注意：undo log是逻辑日志，可以理解为：

​	你执行delete，undolog就会执行一条insert

**binlog**

归属于服务端而不是存储引擎

追加写的，不会覆盖之前的日志信息

#### 更新流程

![更新流程](E:\deng\image\更新流程.png)

**为了保持数据一致性**，将redolog分成两阶段提交。