## SQL语句的分类

### 1、数据查询语言(DQL)：
DQL全程Data Query Language，用以从表中获得数据，确定数据怎样再应用程序给出，SELECT是DQL用得最多的，其他的DQL有WHERE，ORDER BY，GROUP BY和HAVING，这些DQL保留字常与其他的SQL语句一起使用
#### 1.1 SELECT
```ruby
MariaDB [(none)]> select user,host,password from mysql.user order by user;
```
```ruby
MariaDB [babydb]> select id,name from student where id=1;
+----+----------+
| id | name     |
+----+----------+
|  1 | babyshen |
+----+----------+
1 row in set (0.00 sec)

```
- 查看当前使用的数据库
```ruby
MariaDB [(none)]> select database();
+------------+
| database() |
+------------+
| NULL       |
+------------+
1 row in set (0.01 sec)

MariaDB [(none)]> use babyshen;
MariaDB [babyshen]> select database();
+------------+
| database() |
+------------+
| babyshen   |
+------------+
1 row in set (0.00 sec)
```


### 2、数据操作语言(DML):
DML全程Data Manipulation Language，其语句包括INSERT、UPDATE、DELETE，他们分别用于添加、修改、删除表中的行(数据)。
#### 2.1 INSERT
```ruby
 create table `student`(
`id` int(4) not null auto_increment,
`name` char(20) not null,
primary key (`id`)
);

MariaDB [babydb]> insert into student(id,name) values(1,'babyshen');
MariaDB [babydb]> select * from student;
+----+----------+
| id | name     |
+----+----------+
|  1 | babyshen |
+----+----------+
1 row in set (0.00 sec)

MariaDB [babydb]> insert into student(name) values('wangzy');
MariaDB [babydb]> select * from student;
+----+----------+
| id | name     |
+----+----------+
|  1 | babyshen |
|  2 | wangzy   |
+----+----------+
2 rows in set (0.00 sec)

MariaDB [babydb]> insert into student values(4,'baby'),(6,'shen');
MariaDB [babydb]> select * from student;
+----+----------+
| id | name     |
+----+----------+
|  1 | babyshen |
|  2 | wangzy   |
|  4 | baby     |
|  6 | shen     |
+----+----------+
4 rows in set (0.00 sec)

// 我们平时登陆网站发帖子，实质上都是调用网站的程序连接MySQL数据库，通过上述的insert sql语句把帖子博文数据存入数据库的
```
#### 2.2 DELETE
```ruby
MariaDB [(none)]> delete from mysql.user where user='';
// 注意where条件
```
#### 2.3 truncate
```ruby
create table test(
id int(4) not null,
name varchar(20) not null
);
MariaDB [babydb]> insert into test(id,name) values(20,'baby'),(30,'shen');

MariaDB [babydb]> select * from test;
+----+------+
| id | name |
+----+------+
| 20 | baby |
| 30 | shen |
+----+------+
2 rows in set (0.00 sec)

MariaDB [babydb]> truncate table test;
MariaDB [babydb]> select * from test;
Empty set (0.00 sec)

```
`truncate table test和delete from test的区别：`
```ruby
truncate table test:更快，清空物理文件；
delete from test：逻辑清楚，按行删除；
```

### 3、事务处理语言(TPL)：
它的语句能确保被DML语句影响的表的所有行及时得以更新，TPL语句包括BEGIN、TRANSACTION、COMMIT和ROLLBACK

### 4、数据控制语言(DCL)：
DCL全程Data Control Language，它的语句通过GRANT或REVOKE获得许可；

### 5、数据定义语言(DDL)：
DDL全程Data Definition Language，其语句包括CREATE和DROP，在数据库中创建新表或删除表，为表加入索引等；
#### 5.1 CREATE
```ruby
INT(M)型：正常大小整数类型
CHAR(M)型：定长字符串类型，当存储时，总是用空格填满右边到指定的长度
VARCHAR型:变长字符串类型
```
```ruby
方式一：

create table student(
id int(4) not null,
name char(20) not null,
age tinyint(2) not null default '0',
dept varchar(16) default null
);

方式二：
create table `student`(
`id` int(4) not null,
`name` char(20) not null,
`age` tinyint(2) not null default '0',
`dept` varchar(16) default null
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
// MySQL5.1和MySQL5.5环境的默认建表语句中的引擎不同，如果希望控制表的引擎，就要在建表语句中指定引擎；
// MySQL5.1及以前默认的引擎是MyISAM,MySQL5.5.5以后的默认引擎为InnoDB；


方式三：建表的同时创建索引
CREATE TABLE `subject_comment_manager` (
`subject_comment_manager_id` bigint(12) NOT NULL auto_increment COMMENT '主键',
`subject_type` tinyint(2) NOT NULL COMMENT '素材类型',
`subject_primary_key` varchar(255) NOT NULL COMMENT '素材的主键',
`subject_title` varchar(255) NOT NULL COMMENT '素材的名称',
`edit_user_nick` varchar(64) default NULL COMMENT '修改人',
`edit_user_time` timestamp NULL default NULL COMMENT '修改时间',
`edit_comment` varchar(255) default NULL COMMENT '修改的理由',
`state` tinyint(1) NOT NULL default '1' COMMENT '0代表关闭，1代表正常',
PRIMARY KEY (`subject_comment_manager_id`),
KEY `IDX_PRIMARYKEY` (`subject_primary_key`(32)), 

//括号内的32表示对前32个字符做前缀索引。
KEY `IDX_SUBJECT_TITLE` (`subject_title`(32))
KEY `index_nick_type` (`edit_user_nick`(32),`subject_type`)

//联合索引，此行为新加的，用于给大家讲解的。实际表语句内没有此行。
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

// COMMENT是注释作用；
```

### 6、使用alter命令实现增删改表的字段
[使用alter命令创建、修改、删除索引请点击此链接](www.baidu.com)
#### 6.1 命令语法及默认添加演示：
```ruby
命令语法：alter table 表名 add 字段 类型 其他;

原始表：
create table test(
id int(4) not null,
name varchar(20) not null
);

在表test中添加字段 sex，age，qq，类型分别为char(4)，int(4)，varchar(15)：

MariaDB [babydb]> alter table test add qq varchar(15);
MariaDB [babydb]> desc test;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(4)      | NO   |     | NULL    |       |
| name  | varchar(20) | NO   |     | NULL    |       |
| qq    | varchar(15) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
3 rows in set (0.02 sec)

MariaDB [babydb]> alter table test add age int(4) first;
MariaDB [babydb]> desc test;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| age   | int(4)      | YES  |     | NULL    |       |
| id    | int(4)      | NO   |     | NULL    |       |
| name  | varchar(20) | NO   |     | NULL    |       |
| qq    | varchar(15) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
4 rows in set (0.00 sec)
```
#### 6.2 生产环境多个复杂添加修改多字段信息的案例：
```ruby
添加两个字段：
MariaDB [babydb]> alter table `test` add classID varchar(100) default NULL comment '测试classID', add schoolID varchar(200) default NULL comment '测试schoolID';
MariaDB [babydb]> desc test;
+----------+--------------+------+-----+---------+-------+
| Field    | Type         | Null | Key | Default | Extra |
+----------+--------------+------+-----+---------+-------+
| age      | int(4)       | YES  |     | NULL    |       |
| id       | int(4)       | NO   |     | NULL    |       |
| name     | varchar(20)  | NO   |     | NULL    |       |
| qq       | varchar(15)  | YES  |     | NULL    |       |
| classID  | varchar(100) | YES  |     | NULL    |       |
| schoolID | varchar(200) | YES  |     | NULL    |       |
+----------+--------------+------+-----+---------+-------+
6 rows in set (0.00 sec)


修改字段：
change命令修改：
MariaDB [babydb]> alter table test change classID classID int(50) comment '修改类型';
MariaDB [babydb]> desc test;
+----------+--------------+------+-----+---------+-------+
| Field    | Type         | Null | Key | Default | Extra |
+----------+--------------+------+-----+---------+-------+
| age      | int(4)       | YES  |     | NULL    |       |
| id       | int(4)       | NO   |     | NULL    |       |
| name     | varchar(20)  | NO   |     | NULL    |       |
| qq       | varchar(15)  | YES  |     | NULL    |       |
| classID  | int(50)      | YES  |     | NULL    |       |
| schoolID | varchar(200) | YES  |     | NULL    |       |
+----------+--------------+------+-----+---------+-------+
6 rows in set (0.00 sec)

modify命令修改：
MariaDB [babydb]> alter table test modify age char(4) after name;
MariaDB [babydb]> desc test;
+----------+--------------+------+-----+---------+-------+
| Field    | Type         | Null | Key | Default | Extra |
+----------+--------------+------+-----+---------+-------+
| id       | int(4)       | NO   |     | NULL    |       |
| name     | varchar(20)  | NO   |     | NULL    |       |
| age      | char(4)      | YES  |     | NULL    |       |
| qq       | varchar(15)  | YES  |     | NULL    |       |
| classID  | int(50)      | YES  |     | NULL    |       |
| schoolID | varchar(200) | YES  |     | NULL    |       |
+----------+--------------+------+-----+---------+-------+
6 rows in set (0.00 sec)
```

### 7、修改表名：

#### 7.1 rename法：
```ruby
语法：rename table 原表名 to 新表名
MariaDB [babydb]> rename table test to test_new;
```
#### 7.2 alter法：
```ruby
MariaDB [babydb]> alter table test rename to test_new;
```

### 8、多表查询：
建立几个关联表，要实现多表连表查询，就需要有关联表及数据
```ruby
创建数据库：

MariaDB [babydb]> create database babydb;
```

```ruby
学生表：
学号-主键，姓名，性别，年龄，所在系

create table student(
Snum int(10) NOT NULL COMMENT '学号',
Sname varchar(16) NOT NULL COMMENT '姓名',
Ssex char(2) NOT NULL COMMENT '性别',
Sage tinyint(2) NOT NULL default '0' COMMENT '学生年龄',
Sdept varchar(16) default NULL COMMENT '学生所在系别',
PRIMARY KEY (Snum),
key index_Sname(Sname)

) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```
```ruby
课程表：
课程号-主键，课程名，学分

create table course(
Cnum int(10) NOT NULL COMMENT '课程号',
Cname varchar(64) NOT NULL COMMENT '课程名',
Ccredit tinyint(2) NOT NULL COMMENT '学分',
PRIMARY KEY (Cnum)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```
```ruby
选课表：

create table sc (
SCid int(12) NOT NULL auto_increment COMMENT '主键',
Cnum int(10) NOT NULL COMMENT '课程号',
Snum int(10) NOT NULL COMMENT '学号',
Grade tinyint(2) NOT NULL COMMENT '学生成绩',
PRIMARY KEY (SCid)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

```ruby
学生表插入数据：

insert into student values (0001,'小军','男',30,'计算机网络');
insert into student values (0002,'乐乐','男',30,'computer application');
insert into student values (0003,'神童','男',28,'物流管理');
insert into student values(0004,'博文','男',29,'computer application');
insert into student values(0005,'婷婷','女',26,'计算机科学与技术');
insert into student values(0006,'丽丽','女',22,'护士');


MariaDB [babydb]> select * from student;
+------+--------+------+------+--------------------------+
| Snum | Sname  | Ssex | Sage | Sdept                    |
+------+--------+------+------+--------------------------+
|    1 | 小军   | 男   |   30 | 计算机网络               |
|    2 | 乐乐   | 男   |   30 | computer applica         |
|    3 | 神童   | 男   |   28 | 物流管理                 |
|    4 | 博文   | 男   |   29 | computer applica         |
|    5 | 婷婷   | 女   |   26 | 计算机科学与技术         |
|    6 | 丽丽   | 女   |   22 | 护士                     |
+------+--------+------+------+--------------------------+
6 rows in set (0.00 sec)

```

```ruby
插入课程数据：

insert into course values(1001,'Linux运维',3);
insert into course values(1002,'Linux架构师',5);
insert into course values(1003,'MySQL高级DBA',4);
insert into course values(1004,'Python运维开发',4);
insert into course values(1005,'Java web开发',3);

MariaDB [babydb]> select * from course;
+------+--------------------+---------+
| Cnum | Cname              | Ccredit |
+------+--------------------+---------+
| 1001 | Linux运维          |       3 |
| 1002 | Linux架构师        |       5 |
| 1003 | MySQL高级DBA       |       4 |
| 1004 | Python运维开发     |       4 |
| 1005 | Java web开发       |       3 |
+------+--------------------+---------+
5 rows in set (0.00 sec)
```
```ruby
插入选课数据：

insert into sc (Snum,Cnum,Grade) values (0001,1001,4);
insert into sc (Snum,Cnum,Grade) values (0001,1002,3);
insert into sc (Snum,Cnum,Grade) values (0001,1003,1);
insert into sc (Snum,Cnum,Grade) values (0001,1004,6);
insert into sc (Snum,Cnum,Grade) values (0002,1001,3);
insert into sc (Snum,Cnum,Grade) values (0002,1002,2);
insert into sc (Snum,Cnum,Grade) values (0002,1003,2);
insert into sc (Snum,Cnum,Grade) values (0002,1004,8);
insert into sc (Snum,Cnum,Grade) values (0003,1001,4);
insert into sc (Snum,Cnum,Grade) values (0003,1002,4);
insert into sc (Snum,Cnum,Grade) values (0003,1003,2);
insert into sc (Snum,Cnum,Grade) values (0003,1001,8);
insert into sc (Snum,Cnum,Grade) values (0004,1001,1);
insert into sc (Snum,Cnum,Grade) values (0004,1002,1);
insert into sc (Snum,Cnum,Grade) values (0004,1003,2);
insert into sc (Snum,Cnum,Grade) values (0004,1004,3);
insert into sc (Snum,Cnum,Grade) values (0005,1001,5);
insert into sc (Snum,Cnum,Grade) values (0005,1002,3);
insert into sc (Snum,Cnum,Grade) values (0005,1003,2);
insert into sc (Snum,Cnum,Grade) values (0005,1004,9);


MariaDB [babydb]> select * from sc;
+------+------+------+-------+
| SCid | Cnum | Snum | Grade |
+------+------+------+-------+
|    1 | 1001 |    1 |     4 |
|    2 | 1002 |    1 |     3 |
|    3 | 1003 |    1 |     1 |
|    4 | 1004 |    1 |     6 |
|    5 | 1001 |    2 |     3 |
|    6 | 1002 |    2 |     2 |
|    7 | 1003 |    2 |     2 |
|    8 | 1004 |    2 |     8 |
|    9 | 1001 |    3 |     4 |
|   10 | 1002 |    3 |     4 |
|   11 | 1003 |    3 |     2 |
|   12 | 1001 |    3 |     8 |
|   13 | 1001 |    4 |     1 |
|   14 | 1002 |    4 |     1 |
|   15 | 1003 |    4 |     2 |
|   16 | 1004 |    4 |     3 |
|   17 | 1001 |    5 |     5 |
|   18 | 1002 |    5 |     3 |
|   19 | 1003 |    5 |     2 |
|   20 | 1004 |    5 |     9 |
+------+------+------+-------+
20 rows in set (0.00 sec)

```
```ruby
MariaDB [babydb]> select student.Snum,student.Sname,course.Cname,sc.Grade from student,course,sc where student.Snum=sc.Snum and course.Cnum=sc.Cnum;

+------+--------+--------------------+-------+
| Snum | Sname  | Cname              | Grade |
+------+--------+--------------------+-------+
|    1 | 小军   | Linux运维          |     4 |
|    2 | 乐乐   | Linux运维          |     3 |
|    3 | 神童   | Linux运维          |     4 |
|    3 | 神童   | Linux运维          |     8 |
|    4 | 博文   | Linux运维          |     1 |
|    5 | 婷婷   | Linux运维          |     5 |
|    1 | 小军   | Linux架构师        |     3 |
|    2 | 乐乐   | Linux架构师        |     2 |
|    3 | 神童   | Linux架构师        |     4 |
|    4 | 博文   | Linux架构师        |     1 |
|    5 | 婷婷   | Linux架构师        |     3 |
|    1 | 小军   | MySQL高级DBA       |     1 |
|    2 | 乐乐   | MySQL高级DBA       |     2 |
|    3 | 神童   | MySQL高级DBA       |     2 |
|    4 | 博文   | MySQL高级DBA       |     2 |
|    5 | 婷婷   | MySQL高级DBA       |     2 |
|    1 | 小军   | Python运维开发     |     6 |
|    2 | 乐乐   | Python运维开发     |     8 |
|    4 | 博文   | Python运维开发     |     3 |
|    5 | 婷婷   | Python运维开发     |     9 |
+------+--------+--------------------+-------+
20 rows in set (0.01 sec)

```
