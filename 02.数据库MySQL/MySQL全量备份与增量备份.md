## 二、其他备份：


## 一、MySQLdump备份

### 1、MySQLdump备份命令参数:

#### 1.1 -A参数(备份所有库)：
&emsp;&emsp;-A是备份所有的库，后面不能再指定库，再指定某个库babydb就引起了冲突
```ruby
// 错误的备份命令：

[root@mysql ~]# mysqldump -usystem -pwangzongyu -A -B babydb > /opt/student.sql
```
#### 1.2 -B参数(增加了创建数据库和连接数据库的命令)：
```ruby
[root@mysql ~]# mysqldump -usystem -p'wangzongyu' -B BabyDB > /opt/BabyDB_back_B.sql
[root@mysql ~]# mysqldump -usystem -p'wangzongyu'  BabyDB > /opt/BabyDB_back.sql
```
`加B与不加B的区别：`
```ruby
[root@mysql ~]# diff /opt/BabyDB_back_B.sql /opt/BabyDB_back.sql 
17,24d16
< --
< -- Current Database: `BabyDB`
< --
< CREATE DATABASE /*!32312 IF NOT EXISTS*/ `BabyDB` /*!40100 DEFAULT CHARACTER SET gbk */; 
< USE `BabyDB`;
35c27

// 加-B参数的作用就是增加了创建数据库和连接数据库的命令了
// 加-B参数在恢复数据库的时候就可以不用指定数据库了

[root@mysql ~]# mysql -usystem -p'wangzongyu' < /opt/BabyDB_back_B.sql
```

#### 1.3 --compact参数(用于测试)：
```ruby
--compact     Give less verbose output (useful for debugging). Disables
              structure comments and header/footer constructs.  Enables
              options --skip-add-drop-table --skip-add-locks
              --skip-comments --skip-disable-keys --skip-set-charset.

[root@mysql ~]# mysqldump -usystem -p'wangzongyu' --compact -B BabyDB > /opt/BabyDB_back_B_compact.sql

// 用于调试，因为忽略了drop table、忽略key、忽略锁
```
#### 1.4 -d参数(只备份表的结构，不含数据)：
```ruby
[root@mysql ~]# mysqldump -usystem -p'wangzongyu' -d babydb 

// 备份babydb库的所有表的结构
```
```ruby
[root@mysql ~]# mysqldump -usystem -p'wangzongyu' -d babydb course --compact
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `course` (
  `Cnum` int(10) NOT NULL COMMENT '课程号',
  `Cname` varchar(64) NOT NULL COMMENT '课程名',
  `Ccredit` tinyint(2) NOT NULL COMMENT '学分',
  PRIMARY KEY (`Cnum`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
/*!40101 SET character_set_client = @saved_cs_client */;

```
#### 1.5 -t参数(只备份数据)：
```ruby
[root@mysql ~]# mysqldump -usystem -p'wangzongyu' --compact -t babydb student
INSERT INTO `student` VALUES (1,'小军','男',30,'计算机网络'),(2,'乐乐','男',30,'computer applica'),(3,'神童','男',28,'物流管理'),(4,'博文','男',29,'computer applica'),(5,'婷婷','女',26,'计算机科学与技术'),(6,'丽丽','女',22,'护士');
```
#### 1.6 -F参数(刷新binlog/切割binlog)：
&emsp;&emsp; binlog里面的东西其实跟备份是重复的，所以加上-F 切割一下binlog，是为了binlog之后的是新产生的，切割之前的是数据库备份中已经有的
```ruby
[root@mysql ~]# mysqldump -usystem -p'wangzongyu' -F babydb > /opt/babydb_back.sql
```

#### 1.7 --master-data=1/2(自动获取备份的pos位置点)：
`如果指定了--mater-data，那么就不需要指定-F参数切割日志了`      
[参数使用案例详见](https://github.com/ZongYuWang/Operation/blob/master/02.%E6%95%B0%E6%8D%AE%E5%BA%93MySQL/MySQL%E4%B8%BB%E4%BB%8E%E5%90%8C%E6%AD%A5.md)


#### 1.8 -x参数(锁表)：
&emsp;&emsp;锁表是为了防止备份的时候新的数据写入，但是可读，-x参数等价于使用lock-all-tables = --single-transaction
```ruby
InnoDB引擎备份：
mysqldump -u$MYUSER -p$MYPASS -F -B -A --single-transaction |gzip > $DATA_FILE

MyISAM引擎备份：
mysqldump -u$MYUSER -p$MYPASS -F -B -A --lock-all-tables |gzip > $DATA_FILE

// 也可以直接使用-x参数，锁表默认是关闭的
```

#### 1.9 -R参数(备份存储过程和函数)：


#### 1.10 --triggres参数(备份触发器)：

- gzip压缩参数：
```ruby
[root@mysql ~]# mysqldump -usystem -p'wangzongyu' -B BabyDB|gzip > /opt/BabyDB_back_B.sql.gz
```



### 2、备份库：
#### 2.1 备份一个库：
```ruby
[root@mysql ~]# mysqldump -usystem -p'wangzongyu' -B BabyDB|gzip > /opt/BabyDB_back_B.sql.gz
```
#### 2.2 备份多个库：
##### 多个库使用同一个备份文件：
```ruby
[root@mysql opt]# mysqldump -usystem -p'wangzongyu' -B BabyDB BABYDB|gzip > /opt/baby_back_B.sql.gz
```
##### 多个库使用各自的备份文件：
&emsp;&emsp;分库备份的意义：     
有时一个企业的数据库里会有多个库(www、bbs、blog),但是出问题的时候很可能是某一个库，如果在备份的时候把所有的库都备份成了一个数据文件的话，恢复某一个库的数据时就比较麻烦；
- 方法一：
```ruby
[root@mysql ~]# mysql -usystem -p'wangzongyu' -e "show databases;"|grep -Evi "database|information|performance"|sed  's#^#mysqldump -usystem -p'wngzongyu' -B #g'
mysqldump -usystem -pwngzongyu -B BABYDB
mysqldump -usystem -pwngzongyu -B BABYDB_utf8
mysqldump -usystem -pwngzongyu -B BabyDB
mysqldump -usystem -pwngzongyu -B babydb
mysqldump -usystem -pwngzongyu -B babyshen_gbk
mysqldump -usystem -pwngzongyu -B babyshen_utf8
mysqldump -usystem -pwngzongyu -B mysql
mysqldump -usystem -pwngzongyu -B test
```
```ruby
[root@mysql ~]# mysql -usystem -p'wangzongyu' -e"show databases;" | grep -Eiv "database|infor|perfor" | sed -r 's#^([a-zA-Z].*$)#mysqldump -usystem -p'wangzongyu' --event -B \1|gzip > /opt/back/\1.sql.gz#g'|/bin/bash 

[root@mysql back]# ll
total 168
-rw-r--r--. 1 root root   1657 Jan 21 07:05 babydb.sql.gz
-rw-r--r--. 1 root root    536 Jan 21 07:05 BabyDB.sql.gz
-rw-r--r--. 1 root root    760 Jan 21 07:05 BABYDB.sql.gz
-rw-r--r--. 1 root root    759 Jan 21 07:05 BABYDB_utf8.sql.gz
-rw-r--r--. 1 root root    542 Jan 21 07:05 babyshen_gbk.sql.gz
-rw-r--r--. 1 root root    542 Jan 21 07:05 babyshen_utf8.sql.gz
-rw-r--r--. 1 root root 139633 Jan 21 07:05 mysql.sql.gz
-rw-r--r--. 1 root root    532 Jan 21 07:05 test.sql.gz
```
- 方法二：
```ruby
[root@mysql ~]# vim fenku.sh


for dbname in `mysql -usystem -p'wangzongyu' -e"show databases;" | grep -Eiv "database|infor|perfor"`
do
    mysqldump -usystem -p'wangzongyu' --event -B $dbname|gzip > /opt/back/${dbname}_bak.sql.gz
done

```

### 3、备份表：
#### 3.1 备份单个表：
```ruby
[root@mysql ~]# mysqldump -usystem -p'wangzongyu' babydb student > /opt/baby_table.sql

```
#### 3.2 备份多个表：
##### 多个表使用同一个备份文件：
```ruby
[root@mysql ~]# mysqldump -usystem -p'wangzongyu' babydb student course > /opt/two_table.sql
```
##### 多个表使用各自的备份文件：
&emsp;&emsp;分表备份的意义：     
一个库里有大表和小表，有时可能需要只恢复一个小表，上述的多表备份文件很难拆开，就会像没有分库那样导致恢复某一个小表很麻烦；
```ruby
[root@mysql ~]# mysqldump -usystem -p'wangzongyu' babydb student > /opt/babydb_sutdent.sql
[root@mysql ~]# mysqldump -usystem -p'wangzongyu' babydb course > /opt/badbdb_course.sql

```


### 1、全量备份：
全量数据就是数据库中所有的数据，全量备份就是把数据库中所有的数据进行备份

#### 1.1 备份所有库：
##### MyISAM:
```ruby
[root@MySQlL1-Master ~]# /usr/local/mysql/bin/mysqldump \
--user=root \
--password=wangzongyu \
--all-databases \
--flush-privileges \
--lock-all-tables \
--master-data=1 \
--flush-logs \
--triggers \
--routines \
--events \
--hex-blob | gzip > /MySQL_BACK/mysqlbak_$(date +%F).sql.gz

```
##### InnoDB:
```ruby
[root@MySQlL1-Master ~]# /usr/local/mysql/bin/mysqldump \
--user=root \
--password=wangzongyu \
--all-databases \
--flush-privileges \
--single-transaction \
--master-data=1 \
--flush-logs \
--triggers \
--routines \
--events \
--hex-blob | gzip > /MySQL_BACK/mysqlbak_$(date +%F).sql.gz
```


```ruby
[root@MySQlL1-Master MySQL_BACK]# gzip -d mysqlbak_2018-02-01.sql.gz
[root@MySQlL1-Master MySQL_BACK]# grep -E -v "#|\/|^$|--" mysqlbak_2018-02-01.sql
```

#### 1.2 备份一个库：
```ruby
[root@MySQlL1-Master ~]# mysqldump -usystem -pwangzongyu --default-character-set=utf8 -F -B -x --master-data=1 --events MyDB|gzip > /MySQL_BACK/mysqlbak_$(date +%F).sql.gz
// MyDB是需要备份的库
```

### 2、增量备份
增量数据是从上次全量备份之后，更新的新数据，对于MySQL来说，Binlog日志就是MySQL的增量数据；

### 3、企业全量备份和增量备份的频率：
- 中小企业：全量一般是每天一次，业务流量低估执行全备，备份时会锁表；
- 大型公司：周备，每周六0点一次全备份，下周日至下周六0点前都是增量备份；

一般由人为(或程序)逻辑的方式在数据库执行的SQL语句等误操作，需要增量恢复，因为此时，所有的从库也执行了误操作语句

## 模拟企业数据丢失应用场景：    

![](https://github.com/ZongYuWang/image/blob/master/MySQL3.png)    

### 1、 0点做一次全备份：
```ruby
[root@MySQlL1-Master ~]# mkdir /MySQL_FULLBACK

[root@MySQlL1-Master ~]# mysqldump -usystem -pwangzongyu -F -B --master-data=2 -x MyDB|gzip > /MySQL_FULLBACK/fullback_$(date +%F).sql.gz

// 备份的时候指定-F做一下Binlog日志的切割
```
### 2、 0:00-0:10有新的数据写入：
```ruby
MariaDB [(none)]> use MyDB;
MariaDB [MyDB]> select * from MyTB;
+----+-----------+
| id | name      |
+----+-----------+
|  1 | babyshen1 |
|  2 | babyshen2 |
|  3 | babyshen3 |
+----+-----------+
MariaDB [MyDB]> insert into MyTB(name) values('babyshen5');
MariaDB [MyDB]> insert into MyTB(name) values('babyshen6');

```
### 3、 模拟用户破坏数据库：
#### 3.1  0:10 某个同事误删了MyDB数据库
```ruby
MariaDB [(none)]> drop database MyDB;
```
#### 3.2 发现故障并排查原因：    
数据库出问题10分钟后，公司的网站运营人员报网站故障，联系运维人员解决，接到故障之后运维DBA或开发人员查看网站报错或查看后台日志，显示连接不上MyDB数据库，然后登陆数据库发现MyDB数据库没有了，经过询问，有人在10分钟之前误删了一个数据库，至此，问题原因找到，准备开始恢复数据库(所以设定权限是多么重要的！)

### 4、 检查并修改增量数据库文件：
通过防火墙禁止WEB等应用向主库写数据或者锁表，让主库暂时停止更新，然后在进行恢复

#### 4.1 检查全备及Binlog日志：
##### 检查凌晨全备：
```ruby
[root@MySQlL1-Master ~]# ll /MySQL_FULLBACK/
-rw-r--r--. 1 root root 857 Feb  2 10:52 fullback_2018-02-02.sql.gz
[root@MySQlL1-Master MySQL_FULLBACK]# gzip -d fullback_2018-02-02.sql.gz 
[root@MySQlL1-Master MySQL_FULLBACK]# grep -i "change" fullback_2018-02-02.sql 
-- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000011', MASTER_LOG_POS=245;

```

```ruby
[root@MySQlL1-Master MySQL_FULLBACK]# mysqladmin -usystem -pwangzongyu flush-logs

// 刷新一下Binlog，这样恢复的就只有mysql-bin.000011，新的数据会写入到mysql-bin.000012中，这种是企业业务不允许锁表的情况下
```
##### 将Binlog日志转换成sql文件：
```ruby
[root@MySQlL1-Master ~]# cp /ourdata/binlog/mysql-bin.000011 /MySQL_FULLBACK/
[root@MySQlL1-Master ~]# mysqlbinlog -d MyDB /MySQL_FULLBACK/mysql-bin.000011 > mysql-bin.000011.sql

```
#### 4.2 编辑sql文件，将误删数据库的语句(drop database MyDB)删除
```ruby
[root@MySQlL1-Master ~]# vim mysql-bin.000011.sql
......
# at 721
#180202 10:56:52 server id 1  end_log_pos 807   Query   thread_id=13    exec_time=0     error_code=0
SET TIMESTAMP=1517540212/*!*/;
drop database MyDB
/*!*/;
......

```
### 5、全备恢复过程：
```ruby
[root@MySQlL1-Master ~]# mysql -usystem -pwangzongyu < /MySQL_FULLBACK/fullback_2018-02-02.sql
```

### 6、增量恢复过程：
```ruby
[root@MySQlL1-Master ~]# mysql -usystem -pwangzongyu < /root/mysql-bin.000011.sql
```
