## MySQL全量备份与增量备份

### 1、全量备份：
全量数据就是数据库中所有的数据，全量备份就是把数据库中所有的数据进行备份

#### 1.1 备份所有库：
```ruby
[root@MySQlL1-Master ~]# mysqldump -usystem -pwangzongyu --default-character-set=utf8 -F -B -A -x --master-data=1 --events |gzip > /MySQL_BACK/mysqlbak_$(date +%F).sql.gz

// -F 会刷新binlog,也就是切割了binlog
// -x 锁表，防止备份的时候新的数据写入，但是可读 = --lock-all-tables = --single-transaction

[root@MySQlL1-Master MySQL_BACK]# gzip -d mysqlbak_2018-02-01.sql.gz
[root@MySQlL1-Master MySQL_BACK]# grep -E -v "#|\/|^$|--" mysqlbak_2018-02-01.sql
```
`锁表命令详解：`
```ruby
InnoDB引擎备份：
mysqldump -u$MYUSER -p$MYPASS -F -B -A --single-transaction |gzip > $DATA_FILE

MyISAM引擎备份：
mysqldump -u$MYUSER -p$MYPASS -F -B -A --lock-all-tables |gzip > $DATA_FILE

// 也可以直接使用-x参数，锁表默认是关闭的
```

#### 1.2 备份一个库：
```ruby
[root@MySQlL1-Master ~]# mysqldump -usystem -pwangzongyu --default-character-set=utf8 -F -B -x --master-data=1 --events MyDB|gzip > /MySQL_BACK/mysqlbak_$(date +%F).sql.gz
// MyDB是需要备份的库
```
`错误的备份命令：`
```ruby
[root@mysql ~]# mysqldump -usystem -pwangzongyu -A -B babydb > /opt/student.sql
//-A是备份所有的库，后面不能再指定库，再指定某个库babydb就引起了冲突
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
##### 编辑sql文件，将误删数据库的语句(drop database MyDB)删除
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
