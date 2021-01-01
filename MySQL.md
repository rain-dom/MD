# 基础

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



### 存储引擎



不同的存储引擎，数据文件和索引文件存放的位置是不同的，因此有了分类：
**聚簇索引**：数据和文件放在一起：innodb
.frm：存放的是表结构
ibd：存放数据文件和索引文件

>  注意：mysql的innodb存储引擎默认情况下会把所有的数据文件放到表空间中，不会为每一个单独的表保存一份数据文件，如果需要将每一个表单独使用文件保存，设置如下属性：set global innodb_file_per table=on；

![聚簇索引](E:\deng\deng\MD\image\聚簇索引.png)

**非聚簇索引**：数据和索引单独一个文件：MyISAM

> 最底层的叶子节点存储的是文件指针，再去数据文件中定位拿取

#### **InnoDB--B+Tree**

聚簇索引

叶子节点直接放置数据

* 必须有主键

#### **MYISAM--B+Tree**

非聚簇索引

叶子节点存放指针





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



## 事务处理

savepoint test1

rollback to test1

### 事物的ACID属性

* 最终为了保持一致性

![事务的ACID属性](E:\deng\image\事务的ACID属性.png)

### 锁的机制

为了解决并发访问时数据不一致问题，给数据加锁时需考虑"粒度"的问题：

一般情况下，锁的粒度越小，效率越高。粒度越大，效率越低。大部分情况用行级锁



## 数据库设计三范式

![第一范式](E:\deng\image\第一范式.png)

![第二范式](E:\deng\image\第二范式.png)

![第三范式](E:\deng\image\第三范式.png)



## mysql的锁机制

### 1、MySQL锁的基本介绍

**锁是计算机协调多个进程或线程并发访问某一资源的机制。**

MyISAM和MEMORY存储引擎采用的是表级锁（table-level locking）；

InnoDB存储引擎既支持行级锁（row-level locking），也支持表级锁，但默认情况下是采用行级锁。 

​		**表级锁：**开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低。 
​		**行级锁：**开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。  

表级锁更适合于以查询为主，只有少量按索引条件更新数据的应用，如Web应用；

行级锁则更适合于有大量按索引条件并发更新少量不同数据，同时又有 并发查询的应用，如一些在线事务处理（OLTP）系统。 

### 2、MyISAM表锁

MySQL的表级锁有两种模式：**表共享读锁（Table Read Lock）**和**表独占写锁（Table Write Lock）**。  

* 对MyISAM表的读操作，不会阻塞其他用户对同一表的读请求，但会阻塞对同一表的写请求；

* 对 MyISAM表的写操作，则会阻塞其他用户对同一表的读和写操作；MyISAM表的读操作与写操作之间，以及写操作之间是串行的！ 

建表语句：

```sql
CREATE TABLE `mylock` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `NAME` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;--要设置为MyISAM,因为默认是InnoDB

INSERT INTO `mylock` (`id`, `NAME`) VALUES ('1', 'a');
INSERT INTO `mylock` (`id`, `NAME`) VALUES ('2', 'b');
INSERT INTO `mylock` (`id`, `NAME`) VALUES ('3', 'c');
INSERT INTO `mylock` (`id`, `NAME`) VALUES ('4', 'd');
```

**MyISAM写锁阻塞读的案例：**

​		当一个线程获得对一个表的写锁之后，只有持有锁的线程可以对表进行更新操作。其他线程的读写操作都会等待，直到锁释放为止。

|                           session1                           |                         session2                          |
| :----------------------------------------------------------: | :-------------------------------------------------------: |
|       获取表的write锁定<br />lock table mylock write;        |                                                           |
| 当前session对表的查询，插入，更新操作都可以执行<br />select * from mylock;<br />insert into mylock values(5,'e'); | 当前session对表的查询会被阻塞<br />select * from mylock； |
|                释放锁：<br />unlock tables；                 |          当前session能够立刻执行，并返回对应结果          |

**MyISAM读阻塞写的案例：**

​		一个session使用lock table给表加读锁，这个session可以锁定表中的记录，但更新和访问其他表都会提示错误，同时，另一个session可以查询表中的记录，但更新就会出现**锁等待**。

|                           session1                           |                           session2                           |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|        获得表的read锁定<br />lock table mylock read;         |                                                              |
|   当前session可以查询该表记录：<br />select * from mylock;   |   当前session可以查询该表记录：<br />select * from mylock;   |
| 当前session不能查询没有锁定的表<br />select * from person<br />Table 'person' was not locked with LOCK TABLES | 当前session可以查询或者更新未锁定的表<br />select * from mylock<br />insert into person values(1,'zhangsan'); |
| 当前session插入或者更新表会提示错误<br />insert into mylock values(6,'f')<br />Table 'mylock' was locked with a READ lock and can't be updated<br />update mylock set name='aa' where id = 1;<br />Table 'mylock' was locked with a READ lock and can't be updated | 当前session插入数据会等待获得锁<br />insert into mylock values(6,'f'); |
|                  释放锁<br />unlock tables;                  |                       获得锁，更新成功                       |

### 注意:

**MyISAM在执行查询语句之前，会自动给涉及的所有表加读锁，在执行更新操作前，会自动给涉及的表加写锁，这个过程并不需要用户干预，因此用户一般不需要使用命令来显式加锁，上例中的加锁时为了演示效果。**

**MyISAM的并发插入问题**

MyISAM表的读和写是串行的，这是就总体而言的，在一定条件下，MyISAM也支持查询和插入操作的并发执行

MyISAM存储引擎有一个系统变量concurrent_insert，专门用来控制并发插入的行为，值分别为0，1,2

​	值为0时，不允许并发插入

​	值为1时，如果表中间没有被删除的行(空洞)，MyISAM允许一个进程读表的同时，另一个进程从表尾插入记录

​	值为2时，无论表中有没有空洞，都允许在表尾并发插入记录

|                           session1                           |                           session2                           |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|   获取表的read local锁定<br />lock table mylock read local   |                                                              |
| 当前session不能对表进行更新或者插入操作<br />insert into mylock values(6,'f')<br />Table 'mylock' was locked with a READ lock and can't be updated<br />update mylock set name='aa' where id = 1;<br />Table 'mylock' was locked with a READ lock and can't be updated |    其他session可以查询该表的记录<br />select* from mylock    |
| 当前session不能查询没有锁定的表<br />select * from person<br />Table 'person' was not locked with LOCK TABLES | 其他session可以进行插入操作，但是更新会阻塞<br />update mylock set name = 'aa' where id = 1; |
|          当前session不能访问其他session插入的记录；          |                                                              |
|                  释放锁资源：unlock tables                   |               当前session获取锁，更新操作完成                |
|           当前session可以查看其他session插入的记录           |                                                              |

 可以通过检查table_locks_waited和table_locks_immediate状态变量来分析系统上的表锁定争夺： 

```sql
mysql> show status like 'table%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| Table_locks_immediate | 352   |
| Table_locks_waited    | 2     |
+-----------------------+-------+
--如果Table_locks_waited的值比较高，则说明存在着较严重的表级锁争用情况。
```

**InnoDB锁**

**1、事务及其ACID属性**

事务是由一组SQL语句组成的逻辑处理单元，事务具有4属性，通常称为事务的ACID属性。

原子性（Actomicity）：事务是一个原子操作单元，其对数据的修改，要么全都执行，要么全都不执行。ONdo log 实现
一致性（Consistent）：在事务开始和完成时，数据都必须保持一致状态。
隔离性（Isolation）：数据库系统提供一定的隔离机制，保证事务在不受外部并发操作影响的“独立”环境执行。锁机制
持久性（Durable）：事务完成之后，它对于数据的修改是永久性的，即使出现系统故障也能够保持。redo log

**2、并发事务带来的问题**

相对于串行处理来说，并发事务处理能大大增加数据库资源的利用率，提高数据库系统的事务吞吐量，从而可以支持更多用户的并发操作，但与此同时，会带来一下问题：

**脏读**： 一个事务正在对一条记录做修改，在这个事务并提交前，这条记录的数据就处于不一致状态；这时，另一个事务也来读取同一条记录，如果不加控制，第二个事务读取了这些“脏”的数据，并据此做进一步的处理，就会产生未提交的数据依赖关系。这种现象被形象地叫做“脏读” 

**不可重复读**：一个事务在读取某些数据已经发生了改变、或某些记录已经被删除了！这种现象叫做“不可重复读”。 

**幻读**： 一个事务按相同的查询条件重新读取以前检索过的数据，却发现其他事务插入了满足其查询条件的新数据，这种现象就称为“幻读” 

上述出现的问题都是数据库读一致性的问题，可以通过事务的隔离机制来进行保证。

数据库的事务隔离越严格，并发副作用就越小，但付出的代价也就越大，因为事务隔离本质上就是使事务在一定程度上串行化，需要根据具体的业务需求来决定使用哪种隔离级别

|                             | 脏读 | 不可重复读 | 幻读 |
| :-------------------------: | :--: | :--------: | :--: |
|      read uncommitted       |  √   |     √      |  √   |
|       read committed        |      |     √      |  √   |
| repeatable read（默认级别） |      |            |  √   |
|        serializable         |      |            |      |

 可以通过检查InnoDB_row_lock状态变量来分析系统上的行锁的争夺情况： 

```sql
mysql> show status like 'innodb_row_lock%';
+-------------------------------+-------+
| Variable_name                 | Value |
+-------------------------------+-------+
| Innodb_row_lock_current_waits | 0     |
| Innodb_row_lock_time          | 18702 |
| Innodb_row_lock_time_avg      | 18702 |
| Innodb_row_lock_time_max      | 18702 |
| Innodb_row_lock_waits         | 1     |
+-------------------------------+-------+
--如果发现锁争用比较严重，如InnoDB_row_lock_waits和InnoDB_row_lock_time_avg的值比较高
```

**3、InnoDB的行锁模式及加锁方法**

​		**共享锁（s）**：又称读锁。允许一个事务去读一行，阻止其他事务获得相同数据集的排他锁。若事务T对数据对象A加上S锁，则事务T可以读A但不能修改A，其他事务只能再对A加S锁，而不能加X锁，直到T释放A上的S锁。这保证了其他事务可以读A，但在T释放A上的S锁之前不能对A做任何修改。
​		**排他锁（x）**：又称写锁。允许获取排他锁的事务更新数据，阻止其他事务取得相同的数据集共享读锁和排他写锁。若事务T对数据对象A加上X锁，事务T可以读A也可以修改A，其他事务不能再对A加任何锁，直到T释放A上的锁。

​		mysql InnoDB引擎默认的修改数据语句：**update,delete,insert都会自动给涉及到的数据加上排他锁，select语句默认不会加任何锁类型**，如果

加排他锁可以使用select …for update语句，

加共享锁可以使用select … lock in share mode语句。**所以加过排他锁的数据行在其他事务种是不能修改数据的，也不能通过for update和lock in share mode锁的方式查询数据，但可以直接通过select …from…查询数据，因为普通查询没有任何锁机制。** 

#### **InnoDB行锁实现方式**

​		InnoDB行锁是通过给**索引**上的索引项加锁来实现的，这一点MySQL与Oracle不同，后者是通过在数据块中对相应数据行加锁来实现的。InnoDB这种行锁实现特点意味着：只有通过索引条件检索数据，InnoDB才使用行级锁，**否则，InnoDB将使用表锁！**  

1、在不通过索引条件查询的时候，innodb使用的是表锁而不是行锁

```sql
create table tab_no_index(id int,name varchar(10)) engine=innodb;
insert into tab_no_index values(1,'1'),(2,'2'),(3,'3'),(4,'4');
```

|                           session1                           |                           session2                           |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| set autocommit=0<br />select * from tab_no_index where id = 1; | set autocommit=0<br />select * from tab_no_index where id =2 |
|      select * from tab_no_index where id = 1 for update      |                                                              |
|                                                              |     select * from tab_no_index where id = 2 for update;      |

session1只给一行加了排他锁，但是session2在请求其他行的排他锁的时候，会出现锁等待。原因是在没有索引的情况下，innodb只能使用表锁。

2、**创建带索引**的表进行条件查询，innodb使用的是行锁

```sql
create table tab_with_index(id int,name varchar(10)) engine=innodb;
alter table tab_with_index add index id(id);
insert into tab_with_index values(1,'1'),(2,'2'),(3,'3'),(4,'4');
```

|                           session1                           |                           session2                           |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| set autocommit=0<br />select * from tab_with_indexwhere id = 1; | set autocommit=0<br />select * from tab_with_indexwhere id =2 |
|     select * from tab_with_indexwhere id = 1 for update      |                                                              |
|                                                              |     select * from tab_with_indexwhere id = 2 for update;     |

3、由于mysql的行锁是针对索引加的锁，不是针对记录加的锁，所以虽然是访问不同行的记录，但是如果是使用相同的索引键，是会出现冲突的。

```sql
alter table tab_with_index drop index id;
insert into tab_with_index  values(1,'4');
```

|                           session1                           |                           session2                           |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|                       set autocommit=0                       |                       set autocommit=0                       |
| select * from tab_with_index where id = 1 and name='1' for update |                                                              |
|                                                              | select * from tab_with_index where id = 1 and name='4' for update<br />虽然session2访问的是和session1不同的记录，但是因为使用了相同的索引，所以需要等待锁 |

### 总结

**对于MyISAM的表锁，主要讨论了以下几点：** 
（1）共享读锁（S）之间是兼容的，但共享读锁（S）与排他写锁（X）之间，以及排他写锁（X）之间是互斥的，也就是说读和写是串行的。  
（2）在一定条件下，MyISAM允许查询和插入并发执行，我们可以利用这一点来解决应用中对同一表查询和插入的锁争用问题。 
（3）MyISAM默认的锁调度机制是写优先，这并不一定适合所有应用，用户可以通过设置LOW_PRIORITY_UPDATES参数，或在INSERT、UPDATE、DELETE语句中指定LOW_PRIORITY选项来调节读写锁的争用。 
（4）由于表锁的锁定粒度大，读写之间又是串行的，因此，如果更新操作较多，MyISAM表可能会出现严重的锁等待，可以考虑采用InnoDB表来减少锁冲突。

**对于InnoDB表，本文主要讨论了以下几项内容：** 
（1）InnoDB的行锁是基于索引实现的，如果不通过索引访问数据，InnoDB会使用表锁。 
（2）在不同的隔离级别下，InnoDB的锁机制和一致性读策略不同。

在了解InnoDB锁特性后，用户可以通过设计和SQL调整等措施减少锁冲突和死锁，包括：

- 尽量使用较低的隔离级别； 精心设计索引，并尽量使用索引访问数据，使加锁更精确，从而减少锁冲突的机会；
- 选择合理的事务大小，小事务发生锁冲突的几率也更小；
- 给记录集显式加锁时，最好一次性请求足够级别的锁。比如要修改数据的话，最好直接申请排他锁，而不是先申请共享锁，修改时再请求排他锁，这样容易产生死锁；
- 不同的程序访问一组表时，应尽量约定以相同的顺序访问各表，对一个表而言，尽可能以固定的顺序存取表中的行。这样可以大大减少死锁的机会；
- 尽量用相等条件访问数据，这样可以避免间隙锁对并发插入的影响； 不要申请超过实际需要的锁级别；除非必须，查询时不要显示加锁；
- 对于一些特定的事务，可以使用表锁来提高处理速度或减少死锁的可能。



## 事务

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



## 日志

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



## 索引

每条数据所在的物理位置基本不在一起

如果没有索引，想找到第六行的文件需要6次磁盘IO（寻道，旋转）

但有索引

![索引](E:\deng\deng\MD\image\索引.png)

create index iname on table book(id);

drop index iname



不同的存储引擎，数据文件和索引文件存放的位置是不同的，因此有了分类：
**聚簇索引**：数据和文件放在一起：innodb
.frm：存放的是表结构
ibd：存放数据文件和索引文件

>  注意：mysql的innodb存储引擎默认情况下会把所有的数据文件放到表空间中，不会为每一个单独的表保存一份数据文件，如果需要将每一个表单独使用文件保存，设置如下属性：set global innodb_file_per table=on；

![聚簇索引](E:\deng\deng\MD\image\聚簇索引.png)

**非聚簇索引**：数据和索引单独一个文件：MyISAM

> 最底层的叶子节点存储的是文件指针，再去数据文件中定位拿取

![非聚簇索引](E:\deng\deng\MD\image\非聚簇索引.png)

**存储引擎是表级别的（不同表可以使用不同的存储引擎）**

#### 基础知识

磁盘块通常为4k,称为一页，磁盘预读会读页的整数倍
mysql默认读16k

* 主键最好是自增的，比如3,4存储在一个磁盘块里，你手动输入了一个3.5，这会导致页分裂。

![索引基础](E:\deng\image\索引基础.png)

![数据库流程](E:\deng\image\mysql架构图.png)



#### 数据结构选择

**为什么最后选择了B+树**

哈希表：

* 适合等值查询（只查一对数据），因为一个数据索引对应唯一的hash值
* 表中数据为无序数据（范围查找时浪费时间）,而企业中较多范围查询
* 且使用时需将全部数据加载到内存

**二叉树**：树身过长影响查询性能

**AVL树**：插入元素时会进行1到N次的旋转，严重影响插入的性能

**红黑树**：损失了部分查询性能，来提高插入性能

* 树深无法控制或插入数据的性能较低（旋转树）

**B树**

磁盘块中包含了指针、键值和数据

当data占用数据较大时，会导致树身增大，增大了查询时磁盘的io次数，进而影响查询性能





**B+tree**

每个磁盘块包含更多的节点，降低了树的高度，同时区间变多，加快了检索速度

所有叶子节点间是链式结构，因此既可以根据主键范围查找和分页查找，也可以从根节点随机查找。

![B+树](E:\deng\image\B+树.png)

**B+Tree索引的性能分析**

·一般使用磁盘I/0次数评价索引结构的优劣
·预读：磁盘一般会顺序向后读取一定长度的数据（页的整数倍）放入内存
·局部性原理：当一个数据被用到时，其附近的数据也通常会马上被使用
·B+Tree节点的大小设为等于一个页，每次新建节点直接申请一个页的空间，这样就保证一个节点物理上也存储在一个页里，就实现了一个节点的载入只需一次I/0
·B+Tree的度d一般会超过100，因此h非常小（一般为3到5之间）



>  主键最好是自增的，比如3,4存储在一个磁盘块里，你手动输入了一个3.5，这会导致页分裂。



存储引擎是表级别的（不同表可以使用不同的存储引擎）

#### **InnoDB--B+Tree**

聚簇索引

叶子节点直接放置数据

·数据文件本身就是索引文件
·表数据文件本身就是按B+Tree组织的一个索引结构文件
·聚集索引一叶节点包含了完整的数据记录
·为什么InnoDB表必须有主键，并且推荐使用整型的自增主键？

> 数据在空间上连续，避免页分裂，检索数据更快，

·为什么非主键索引结构叶子节点存储的是主键值？（一致性和节省存储空间）

#### **MYISAM--B+Tree**

非聚簇索引

叶子节点存放指针，文件位置的指针
因此要多一次IO

#### 联合索引

先从第一个比大小，相同再比后一个

![联合索引](E:\deng\deng\MD\image\联合索引.png)

#### 分类

##### 主键索引

主键

-主键是一种唯一性索引，但它必须定为PRIMARY KEY，每个表只能有一个主键。

如果没有会生成rowid

##### 唯一索引

索引列的所有值都只能出现一次，即必须唯一，值可以为空。

##### 普通索引

* 普通索引会导致回表问题，遍历2次B+树
  * 用ASC码排序
  * 使用**覆盖索引**优化
  * select id from emp where name ="";
  * select id 而不是*，name的B+树中就有id，直接找到，不会发生回表

##### 全文索引

MYISAM支持，Innodb在5.6后支持

全文索引的索引类型为FULTEXT.全文索引可以在varchar、char、text类型的列上创建组合索引

##### 组合索引

经常查询的列建组合索引

涉及**最左匹配**

比如要建name、age的组合索引，考虑空间问题，age索引占据的空间小于name索引。所以建 name ,age 和 age 的索引

#### 面试难点

##### 回表

比如用创建索引的是其它字段（比如name）时，叶子节点中放的是主键，再根据主键索引找数据

因此经过两次遍历 

> 这么设定的原因在于，如果普通索引也放置数据，那么就需要解决数据一致性的问题

##### 覆盖索引

如果查询的值为索引内容，则成为覆盖索引

* 使用**覆盖索引**优化回表问题
* select id from emp where name ="";
* select id 而不是*，name的B+树中就有id，直接找到，不会发生回表

##### 最左前缀



**最左匹配**

组合索引 只有在搜最左的数据时才生效

##### 索引下推

select * from table where name="zhangsan" and age=10

普通情况下先通过张三找到对应的主键，再通过主键找到age返回信息

索引下推时，匹配name的同时匹配age=10的情况，只将"zhangsan" and age=10的数据主键返回







![存储引擎](E:\deng\image\存储引擎.png)

#### 索引维护

![索引维护](E:\deng\image\索引维护.png)



# mysql调优

## 基本命令

| 命令                     | 功能            |
| ------------------------ | --------------- |
| set profiling=1          | 开启查看时间    |
| show profiles            | 查看执行时间    |
| show profile for query n | 看第n个命令具体 |
| show profile all         | 所有耗时        |

## 性能监控



## schema与数据类型优化

### 数据类型优化

在可以正确存储数据的类型下，应该尽可能小。

避免null



#### 整数

TINYINT，SMALLINT，MEDIUMINT，INT，BIGINT
分别使用8，16，24，32，64位存储空间。

尽量使用满足需求的最小数据类型



#### 字符与字符串

char长度固定，最大长度255，会自动删除末尾的空格，

varchar可变程度

按照查询速度：char>varchar>text

**优化**：

1. 使用最小符合要求的长度
2. 基本使用varchar
3. char存储波动不大的数据



#### 日期

不要用字符串存日期

**datetime:**

* 占用8字节
* 与时区无关
* 保存到毫秒

**timestamp**：

* 占用4字节
* 时间范围1970—2038
* 精度到秒
* 采用整形存储

**date:**

* 占用3字节
* 可以利用日期时间函数计算

#### 枚举

数据更加紧凑

#### 特殊类型

存ip可以使用INET_ATON()和INET_NTOA函数在这两种表示方法之间转换

案例：

select inet_aton('1.1.1.1')

select inet_ntoa(16843009)

### 范式与反范式



# mysql执行计划

​       使用explain+SQL语句来模拟优化器执行SQL查询语句，从而知道mysql是如何处理sql语句的。

​	   官网地址： https://dev.mysql.com/doc/refman/5.5/en/explain-output.html 

## 1、执行计划中包含的信息

|    Column     |                    Meaning                     |
| :-----------: | :--------------------------------------------: |
|      id       |            The `SELECT` identifier             |
|  select_type  |               The `SELECT` type                |
|     table     |          The table for the output row          |
|  partitions   |            The matching partitions             |
|     type      |                 The join type                  |
| possible_keys |         The possible indexes to choose         |
|      key      |           The index actually chosen            |
|    key_len    |          The length of the chosen key          |
|      ref      |       The columns compared to the index        |
|     rows      |        Estimate of rows to be examined         |
|   filtered    | Percentage of rows filtered by table condition |
|     extra     |             Additional information             |

**id**

select查询的序列号，包含一组数字，表示查询中执行select子句或者操作表的顺序

id号分为三种情况：

​		1、如果id相同，那么执行顺序从上到下

```sql
explain select * from emp e join dept d on e.deptno = d.deptno join salgrade sg on e.sal between sg.losal and sg.hisal;
```

​		2、如果id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行

```sql
explain select * from emp e where e.deptno in (select d.deptno from dept d where d.dname = 'SALES');
```

​		3、id相同和不同的，同时存在：相同的可以认为是一组，从上往下顺序执行，在所有组中，id值越大，优先级越高，越先执行

```sql
explain select * from emp e join dept d on e.deptno = d.deptno join salgrade sg on e.sal between sg.losal and sg.hisal where e.deptno in (select d.deptno from dept d where d.dname = 'SALES');
```

**select_type**

主要用来分辨查询的类型，是普通查询还是联合查询还是子查询

| `select_type` Value  |                           Meaning                            |
| :------------------: | :----------------------------------------------------------: |
|        SIMPLE        |        Simple SELECT (not using UNION or subqueries)         |
|       PRIMARY        |                       Outermost SELECT                       |
|        UNION         |         Second or later SELECT statement in a UNION          |
|   DEPENDENT UNION    | Second or later SELECT statement in a UNION, dependent on outer query |
|     UNION RESULT     |                      Result of a UNION.                      |
|       SUBQUERY       |                   First SELECT in subquery                   |
|  DEPENDENT SUBQUERY  |      First SELECT in subquery, dependent on outer query      |
|       DERIVED        |                        Derived table                         |
| UNCACHEABLE SUBQUERY | A subquery for which the result cannot be cached and must be re-evaluated for each row of the outer query |
|  UNCACHEABLE UNION   | The second or later select in a UNION that belongs to an uncacheable subquery (see UNCACHEABLE SUBQUERY) |

```sql
--sample:简单的查询，不包含子查询和union
explain select * from emp;

--primary:查询中若包含任何复杂的子查询，最外层查询则被标记为Primary
explain select staname,ename supname from (select ename staname,mgr from emp) t join emp on t.mgr=emp.empno ;

--union:若第二个select出现在union之后，则被标记为union
explain select * from emp where deptno = 10 union select * from emp where sal >2000;

--dependent union:跟union类似，此处的depentent表示union或union all联合而成的结果会受外部表影响
explain select * from emp e where e.empno  in ( select empno from emp where deptno = 10 union select empno from emp where sal >2000)

--union result:从union表获取结果的select
explain select * from emp where deptno = 10 union select * from emp where sal >2000;

--subquery:在select或者where列表中包含子查询
explain select * from emp where sal > (select avg(sal) from emp) ;

--dependent subquery:subquery的子查询要受到外部表查询的影响
explain select * from emp e where e.deptno in (select distinct deptno from dept);

--DERIVED: from子句中出现的子查询，也叫做派生类，
explain select staname,ename supname from (select ename staname,mgr from emp) t join emp on t.mgr=emp.empno ;

--UNCACHEABLE SUBQUERY：表示使用子查询的结果不能被缓存
 explain select * from emp where empno = (select empno from emp where deptno=@@sort_buffer_size);
 
--uncacheable union:表示union的查询结果不能被缓存：sql语句未验证
```

**table**

对应行正在访问哪一个表，表名或者别名，可能是临时表或者union合并结果集
		1、如果是具体的表名，则表明从实际的物理表中获取数据，当然也可以是表的别名

​		2、表名是derivedN的形式，表示使用了id为N的查询产生的衍生表

​		3、当有union result的时候，表名是union n1,n2等的形式，n1,n2表示参与union的id

### **type**-访问类型

type显示的是访问类型，访问类型表示我是以何种方式去访问我们的数据，最容易想的是全表扫描，直接暴力的遍历一张表去寻找需要的数据，效率非常低下，访问的类型有很多，效率从最好到最坏依次是：

system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL 

一般情况下，得保证查询至少达到range级别，最好能达到ref

```sql
--all:全表扫描，一般情况下出现这样的sql语句而且数据量比较大的话那么就需要进行优化。
explain select * from emp;

--index：全索引扫描这个比all的效率要好，主要有两种情况，一种是当前的查询时覆盖索引，即我们需要的数据在索引中就可以索取，或者是使用了索引进行排序，这样就避免数据的重排序
explain  select empno from emp;

--range：表示利用索引查询的时候限制了范围，在指定范围内进行查询，这样避免了index的全索引扫描，适用的操作符： =, <>, >, >=, <, <=, IS NULL, BETWEEN, LIKE, or IN() 
explain select * from emp where empno between 7000 and 7500;

--index_subquery：利用索引来关联子查询，不再扫描全表
explain select * from emp where emp.job in (select job from t_job);

--unique_subquery:该连接类型类似与index_subquery,使用的是唯一索引
 explain select * from emp e where e.deptno in (select distinct deptno from dept);
 
--index_merge：在查询过程中需要多个索引组合使用，没有模拟出来

--ref_or_null：对于某个字段即需要关联条件，也需要null值的情况下，查询优化器会选择这种访问方式
explain select * from emp e where  e.mgr is null or e.mgr=7369;

--ref：使用了非唯一性索引进行数据的查找
 create index idx_3 on emp(deptno);
 explain select * from emp e,dept d where e.deptno =d.deptno;

--eq_ref ：使用唯一性索引进行数据查找
explain select * from emp,emp2 where emp.empno = emp2.empno;

--const：这个表至多有一个匹配行，
explain select * from emp where empno = 7369;
 
--system：表只有一行记录（等于系统表），这是const类型的特例，平时不会出现
```

 **possible_keys** 

​        显示可能应用在这张表中的索引，一个或多个，查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询实际使用

```sql
explain select * from emp,dept where emp.deptno = dept.deptno and emp.deptno = 10;
```

**key**

​		实际使用的索引，如果为null，则没有使用索引，查询中若使用了覆盖索引，则该索引和查询的select字段重叠。

```sql
explain select * from emp,dept where emp.deptno = dept.deptno and emp.deptno = 10;
```

**key_len**

表示索引中使用的字节数，可以通过key_len计算查询中使用的索引长度，在不损失精度的情况下长度越短越好。

```sql
explain select * from emp,dept where emp.deptno = dept.deptno and emp.deptno = 10;
```

**ref**

显示索引的哪一列被使用了，如果可能的话，是一个常数

```sql
explain select * from emp,dept where emp.deptno = dept.deptno and emp.deptno = 10;
```

**rows**

根据表的统计信息及索引使用情况，大致估算出找出所需记录需要读取的行数，此参数很重要，直接反应的sql找了多少数据，在完成目的的情况下越少越好

```sql
explain select * from emp;
```

**extra**

包含额外的信息。

```sql
--using filesort:说明mysql无法利用索引进行排序，只能利用排序算法进行排序，会消耗额外的位置
explain select * from emp order by sal;

--using temporary:建立临时表来保存中间结果，查询完成之后把临时表删除
explain select ename,count(*) from emp where deptno = 10 group by ename;

--using index:这个表示当前的查询时覆盖索引的，直接从索引中读取数据，而不用访问数据表。如果同时出现using where 表名索引被用来执行索引键值的查找，如果没有，表面索引被用来读取数据，而不是真的查找
explain select deptno,count(*) from emp group by deptno limit 10;

--using where:使用where进行条件过滤
explain select * from t_user where id = 1;

--using join buffer:使用连接缓存，情况没有模拟出来

--impossible where：where语句的结果总是false
explain select * from emp where empno = 7469;
```



## 索引优化