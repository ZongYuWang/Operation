## MySQL主从同步

### 1、MySQL主从复制的原理过程

- 在Slave服务器上执行start slave，开启主从复制开关，开始进行主从复制。

- 此时Slave服务器的IO线程会通过Master上授权的复制用户权限连接Master服务器，并请求从指定的Binlog日志文件的指定位置(日志文件名和位置就是在配置主从复制服务时执行change master命令时指定的)之后发送Binlog日志内容；

- Master服务器收到来自Slave服务器的I/O线程请求后，Master服务器上负责复制的IO线程根据Slave服务器的IO线程请求的信息分批读取指定Binlog日志文件指定位置之后的Binlog日志信息，然后返回给Slave端的IO线程。返回的信息除了Binlog日志内容外，还有在Master服务器端新的Binlog文件名，以及在新的Binlog中的下一个指定更新位置；

- 当Slave服务器的IO线程读取到来自Master服务器上IO线程发送日志内容及日志文件及位置点后，将binlog日志内容依次写入到Slave端自身的Relay Log(即中继日志)文件(MySQL-relay-bin.xxxxxx)的最末端，并将新的Binlog文件名和位置记录到master-info文件中，以便下一次读取Master端新Binlog日志时能够告诉Master服务器需要从新Binlog日志的哪个文件哪个位置开始请求新的Binlog日志内容；

- Slave服务器端的SQL线程会实时的检测本地的Relay Log中新增加的日志内容，然后及时把Log文件中的内容解析成在Master 端曾经执行的SQL语句内容，并在自身Slave服务器上按语句的顺序执行应用这些SQL语句，应用完毕后清理应用过的日志；

- 经过上面的过程，就可以确保在Master端和Slave端执行了同样的SQL语句，当复制状态正常情况下，Master端和Slave端的数据是完全一样的；

图：


- 主库通过记录binlog实现对从库的同步，binlog记录数据库的更新语句；
- 主库1个IO线程，从库1个IO线程和1个SQL线程来完成；
- 从库关键文件：master-info，relay-log，relay-log.info；
- 如果从库还想级联从库，需要打开log-bin和log-slave-updates参数；



### 2、在MySQL主服务器上模拟数据：
```ruby
MariaDB [(none)]> create database MyDB;
MariaDB [(none)]> use MyDB;

create table MyTB(
id int(4) not null AUTO_INCREMENT,
name char(20) not null,
primary key(id)
);

MariaDB [MyDB]> insert into MyTB(name) values('babyshen1');
MariaDB [MyDB]> insert into MyTB(name) values('babyshen2');

MariaDB [MyDB]> select * from MyTB;
+----+-----------+
| id | name      |
+----+-----------+
|  1 | babyshen1 |
|  2 | babyshen2 |
+----+-----------+

```

### 3、配置主从同步：
| MySQL主服务器IP | MySQL从服务器IP | 
| - | :-: | 
| 172.30.105.121 | 172.30.105.122|  

#### 3.1 修改MySQL配置文件：
主库配置log-bin和server-id参数，从库配置server-id，不能和主库及其他从库一样，一般不开启log-bin功能，配置参数后要重启数据库生效
##### MySQL主服务器：
```ruby
[root@MySQlL1-Master ~]# grep -E "server-id|log-bin" /etc/my.cnf 
log-bin=/ourdata/binlog/mysql-bin
server-id	= 1

[root@MySQlL1-Master ~]# mysql -usystem -pwangzongyu -e "show variables like 'log_bin';"
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | ON    |
+---------------+-------+

```
##### MySQL从服务器：
```ruby
[root@MySQL2-Slave ~]# grep -E "server-id|log-bin" /etc/my.cnf
log-bin=/ourdata/binlog/mysql-bin 
// 从服务器的log-bin可开可不开
server-id	= 2

```
#### 3.2 建立用于从库复制的账号：
登陆主库增加用于从库连接主库同步的账号，并授权replication slave同步的权限
##### MySQL主服务器：
```ruby
MariaDB [(none)]> grant replication slave on *.* to rep@'172.30.105.%' identified by "wangzongyu";
MariaDB [(none)]> flush privileges;

// REPLICATION SLAVE 这是MySQL同步的必须权限，不要授权all
// *.* 表示所有库所有表，也可以指定具体的库和表进行复制
// 使用%表示允许整个172.30.105.0网段以rep用户访问

[root@MySQlL1-Master ~]# vim /etc/sysconfig/iptables
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT
```


### 4、开始备份：
#### 4.1 锁表并获取Binlog信息：
登陆主库，整库锁表，窗口关闭即失效，超时参数到了也失效，然后show master status查看binlog的位置状态
##### MySQL主服务器：
```ruby
MariaDB [MyDB]> flush table with read lock;
// 在主库上加一个锁，此时主库不能写数据，包括也不能创建数据库，只能读库
// MySQL5.1 flush tables with read lock,tables会有一个s

MariaDB [MyDB]> show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000006 |     1933 |              |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)

```
`这个锁表命令的有效时间，在不同的引擎下，会受下面参数的控制，如果超过设置时间不操作会自动解锁`
```ruby
interactive_timeout = 60
wait_timeout = 60

MariaDB [MyDB]> show variables like '%timeout%';
+----------------------------+----------+
| Variable_name              | Value    |
+----------------------------+----------+
| interactive_timeout        | 28800    |
| wait_timeout               | 28800    |
+----------------------------+----------+

```

#### 4.2 将现有数据做一个全备份：
新打开窗口，linux命令行备份或导出原有的数据库数据，并拷贝到从库所在的服务器目录，如果数据量很大，并且允许停机，可以停机打包，而不用mysqldump     
`锁上表之后，mysqldump会自动拿到位置点`    
##### 使用--master-data=2：
```ruby
[root@MySQlL1-Master ~]# mysqldump -usystem -pwangzongyu -A -B --events|gzip > /opt/MyDB.sql.gz

// --event：
// 如果数据量比较小的话，就不用压缩备份了

[root@MySQlL1-Master ~]# mysqldump -usystem -pwangzongyu -A -B --events --master-data=2 > /opt/MyDB.sql

[root@MySQlL1-Master ~]# vim /opt/MyDB.sql
-- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000006', MASTER_LOG_POS=1933;
// 此行是被注释的

验证：
// mysqldump之后再看看Binlog信息，Binlog信息不能改变，如果改变了说明没有锁住表
MariaDB [MyDB]> show master status;

```
##### 使用--master-data=1
```ruby
[root@MySQlL1-Master ~]# mysqldump -usystem -pwangzongyu -A -B --events --master-data=1 > /opt/MyDB_master1.sq

[root@MySQlL1-Master ~]# vim /opt/MyDB_master1.sql
CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000006', MASTER_LOG_POS=1933;
// 这条没有被注释

```


#### 4.3 解开锁并将全备份的文件拷贝至从服务器：
解锁主库，把主库导出的原有数据恢复到从库
```ruby
MariaDB [MyDB]> unlock tables;

[root@MySQlL1-Master ~]# scp /opt/MyDB.sql 172.30.105.122:/opt

[root@MySQL2-Slave ~]# mysql -usystem -pwangzongyu < /opt/MyDB.sql
```
#### 4.4 配置从库连接主库
根据主库的show master status查看binlog的位置状态，在从库执行change master to...语句
##### ##### 使用--master-data=2的情况：
```ruby
MariaDB [(none)]> CHANGE MASTER TO
     MASTER_HOST='172.30.105.121',
     MASTER_PORT=3306,
     MASTER_USER='rep',
     MASTER_PASSWORD='wangzongyu',
     MASTER_LOG_FILE='mysql-bin.000006',
     MASTER_LOG_POS=1933;
     
```
##### 使用--master-data=1的情况：
```ruby
MariaDB [(none)]> CHANGE MASTER TO
          MASTER_HOST='172.30.105.121',
          MASTER_PORT=3306,
          MASTER_USER='rep',
          MASTER_PASSWORD='wangzongyu';
          
// 上面mysqldump使用--master-data=1备份之后，备份文件会自动记录Binlog的file文件和POS位置点
```

#### 4.5 启动从服务器节点并查看主从复制状态：
从库开启同步开关，同步之后，检查同步状态，并在主库进行更新测试
```ruby
MariaDB [(none)]> start slave;

MariaDB [(none)]> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.30.105.121
                  Master_User: rep
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000006
          Read_Master_Log_Pos: 1933
               Relay_Log_File: MySQL2-Slave-relay-bin.000002
                Relay_Log_Pos: 529
        Relay_Master_Log_File: mysql-bin.000006
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
......
            Seconds_Behind_Master: 0
            // 落后主库的秒数一般是0，但也不完全准确
......

```
#### 4.6 查看相关文件的内容：
##### 从服务器上查看Relay Log:
```ruby
[root@MySQL2-Slave ~]# ll /mydata/data/ | grep relay
-rw-rw----. 1 mysql mysql      301 Jan 31 22:07 MySQL2-Slave-relay-bin.000001
-rw-rw----. 1 mysql mysql      529 Jan 31 22:07 MySQL2-Slave-relay-bin.000002
-rw-rw----. 1 mysql mysql       64 Jan 31 22:07 MySQL2-Slave-relay-bin.index

[root@MySQL2-Slave ~]# mysqlbinlog /mydata/data/MySQL2-Slave-relay-bin.000001

```


##### 从服务器上查看master.info：
```ruby
[root@MySQL2-Slave ~]# cat /mydata/data/master.info 
18
mysql-bin.000006
1933
172.30.105.121
rep
wangzongyu
3306
60
0

0
1800.000

0

```

##### 从服务器上查看relay-log.info:
`在写SQL线程的时候，这个文件会记录读取relay-log文件的时候，读取到哪个位置点了，会读一点往SQL中写一点`
```ruby
./MySQL2-Slave-relay-bin.000001
4
mysql-bin.000006
1933

```
### 5、生产场景快速配置MySQL主从复制方案
- 安装好要配置从库的数据库，配置好log-bin和server-id参数；
- 无需配置主库的my.cnf文件，主库的log-bin和server-id参数默认就是配置好的；
- 登陆主库增加用于从库连接主库同步的账号，并授权replication slave同步的权限；
- 使用半夜mysqldump带--master-data=1备份的全备数据恢复到从库(如果没有全备份，那么就需要选择不影响业务的情况下，锁表或者停库进行一次全备份)；
- 在从库执行change master to...语句，无需binlog文件及对应位置点；
- 从库开启同步开关，start slave；
- 从库show slave status\G，检查同步状态，并在主库进行更新测试；