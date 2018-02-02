
[MySQL异步复制(默认方式)](#MySQL异步复制(默认方式)) 


##  MySQL异步复制(默认方式)

### 1、MySQL主从复制的原理过程

- 在Slave服务器上执行start slave，开启主从复制开关，开始进行主从复制。

- 此时Slave服务器的IO线程会通过Master上授权的复制用户权限连接Master服务器，并请求从指定的Binlog日志文件的指定位置(日志文件名和位置就是在配置主从复制服务时执行change master命令时指定的)之后发送Binlog日志内容；

- Master服务器收到来自Slave服务器的I/O线程请求后，Master服务器上负责复制的IO线程根据Slave服务器的IO线程请求的信息分批读取指定Binlog日志文件指定位置之后的Binlog日志信息，然后返回给Slave端的IO线程。返回的信息除了Binlog日志内容外，还有在Master服务器端新的Binlog文件名，以及在新的Binlog中的下一个指定更新位置；

- 当Slave服务器的IO线程读取到来自Master服务器上IO线程发送日志内容及日志文件及位置点后，将binlog日志内容依次写入到Slave端自身的Relay Log(即中继日志)文件(MySQL-relay-bin.xxxxxx)的最末端，并将新的Binlog文件名和位置记录到master-info文件中，以便下一次读取Master端新Binlog日志时能够告诉Master服务器需要从新Binlog日志的哪个文件哪个位置开始请求新的Binlog日志内容；

- Slave服务器端的SQL线程会实时的检测本地的Relay Log中新增加的日志内容，然后及时把Log文件中的内容解析成在Master 端曾经执行的SQL语句内容，并在自身Slave服务器上按语句的顺序执行应用这些SQL语句，应用完毕后清理应用过的日志；

- 经过上面的过程，就可以确保在Master端和Slave端执行了同样的SQL语句，当复制状态正常情况下，Master端和Slave端的数据是完全一样的；   

![](https://github.com/ZongYuWang/image/blob/master/MySQL4.png)    

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
`也可以不在此处做锁表操作，在下面的mysqldump命令中指定-x参数锁表`
```ruby
[root@MySQlL1-Master ~]# mysqldump -usystem -pwangzongyu -A -B --master-data=1 -x --events > /opt/mysql_fullback.sql
// -x是锁表 ，类似于登陆数据库执行 flush table with read lock
```


#### 4.2 将现有数据做一个全备份：
新打开窗口，linux命令行备份或导出原有的数据库数据，并拷贝到从库所在的服务器目录，如果数据量很大，并且允许停机，可以停机打包，而不用mysqldump     
`锁上表之后，mysqldump会自动拿到位置点`    
##### 使用--master-data=2：
```ruby
[root@MySQlL1-Master ~]# mysqldump -usystem -pwangzongyu -A -B --events|gzip > /opt/MyDB.sql.gz

// --event：mysqldump默认是不备份事件表的,只有加了--events 才会不警告
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
[root@MySQlL1-Master ~]# mysqldump -usystem -pwangzongyu -A -B --events --master-data=1 > /opt/MyDB_master1.sql

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
##### 使用--master-data=2的情况：
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

### 6、主从备份自动化脚本：
MySQL主服务器和MySQL从服务器需要做好时钟同步，因为备份文件都是按照时间备份的
##### MySQL主服务器：
```ruby
[root@MySQlL1-Master ~]# vim mysql_master_fullback.sh

#!/bin/sh

##################################################
# This scripts is create by wangzongyu
# wangzongyu QQ:479414941
##################################################

MYUSER="system"
MYPASSWORD="wangzongyu"
MYSOCK=/tmp/mysql.sock


DATA_PATH=/mysql_back && mkdir -p $DATA_PATH
LOG_FILE=${DATA_PATH}/mysqllogs_`date +%F`.log
DATA_FILE=${DATA_PATH}/mysql_backup_`date +%F`.sql.gz
MYSQL_PATH=/usr/local/mysql/bin
MYSQL_CMD="$MYSQL_PATH/mysql -u$MYUSER -p$MYPASSWORD -S$MYSOCK"

MYSQL_DUMP="$MYSQL_PATH/mysqldump -u$MYUSER -p$MYPASSWORD -S$MYSOCK -A -B --master-data=1 --single-transaction -e"

${MYSQL_DUMP} | gzip > $DATA_FILE

```
```ruby
检验查看备份文件：

[root@MySQlL1-Master ~]# ls /mysql_back/
mysql_backup_2018-02-01.sql.gz
[root@MySQlL1-Master mysql_back]# gzip -d mysql_backup_2018-02-01.sql.gz 
[root@MySQlL1-Master mysql_back]# vim mysql_backup_2018-02-01.sql 
CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=1170;
```
##### MySQL从服务器：
`不停主库一键批量创建从库，也就是把上面主库的备份文件分发到想做从库的机器上，可以多台`

```ruby
[root@MySQL2-Slave ~]# mkdir /mysql_back 
// 主库全备份文件拷贝至这个目录中，这个目录在脚本中也需要，也可以自行修改成其他目录

[root@MySQlL1-Master mysql_back]# scp mysql_backup_2018-02-01.sql.gz 172.30.105.122:/mysql_back/
// 将主库的全备份文件分发至从服务器

```
```ruby
#!/bin/sh

##################################################
# This scripts is create by wangzongyu
# wangzongyu QQ:479414941
##################################################

echo -e "\033[41;36m installing software \033[0m"
yum install -y mail
echo -e "\033[41;36m package is installed done \033[0m"


MYUSER="system"
MYPASSWORD="wangzongyu"
MYSOCK=/tmp/mysql.sock

DATA_PATH=/mysql_back #主库的备份文件需要拷贝至这个目录下
LOG_FILE=${DATA_PATH}/mysqllogs_`date +%F`.log
MYSQL_PATH=/usr/local/mysql/bin
MYSQL_CMD="$MYSQL_PATH/mysql -u$MYUSER -p$MYPASSWORD -S $MYSOCK"

#recover
cd ${DATA_PATH}
gzip -d mysql_backup_`date +%F`.sql.gz
$MYSQL_CMD < mysql_backup_`date +%F`.sql

#config slave
$MYSQL_CMD -e << EOF
CHANGE MASTER TO
MASTER_HOST='172.30.105.121',
MASTER_PORT=3306,
MASTER_USER='rep',
MASTER_PASSWORD='wangzongyu';
EOF

$MYSQL_CMD -e "start slave;"
sleep 20
$MYSQL_CMD -e "show slave status\G"|egrep "IO_Running|SQL_Running">$LOG_FILE

#mail -s "mysql slave result" 479414941@qq.com < $LOG_FILE
```
### 7、相关MySQL主从复制技术技巧概览：
#### 7.1 登陆数据库查看MySQL线程同步状态：
##### MySQL主服务器：
```ruby
MariaDB [MyDB]> show processlist\G
*************************** 1. row ***************************
      Id: 12
    User: system
    Host: localhost
      db: MyDB
 Command: Query
    Time: 0
   State: NULL
    Info: show processlist
Progress: 0.000
*************************** 2. row ***************************
      Id: 17
    User: rep
    Host: 172.30.105.122:41056
      db: NULL
 Command: Binlog Dump
    Time: 873
   State: Master has sent all binlog to slave; waiting for binlog to be updated
    Info: NULL
Progress: 0.000

```
##### MySQL从服务器：
```ruby
MariaDB [MyDB]> show processlist\G
*************************** 1. row ***************************
      Id: 10
    User: system
    Host: localhost
      db: MyDB
 Command: Query
    Time: 0
   State: NULL
    Info: show processlist
Progress: 0.000
*************************** 2. row ***************************
      Id: 22
    User: system user
    Host: 
      db: NULL
 Command: Connect
    Time: 924
   State: Waiting for master to send event
    Info: NULL
Progress: 0.000
*************************** 3. row ***************************
      Id: 23
    User: system user
    Host: 
      db: NULL
 Command: Connect
    Time: 924
   State: Slave has read all relay log; waiting for the slave I/O thread to update it
    Info: NULL
Progress: 0.000

```
#### 7.2 主从复制主线程状态
下面列出了主服务器的Binlog Dump线程的State列的最常见的状态
- Sending binlog event to slave
二进制日志由各种事件组成，一个事件通常为一个更新加一些其它信息。线程已经从二进制日志读取了一个事件并且正将它发送到从服务器。

- Finished reading one binlog; switching to next binlog
线程已经读完二进制日志文件并且正打开下一个要发送到从服务器的日志文件。

- Has sent all binlog to slave; waiting for binlog to be updated
线程已经从二进制日志读取所有主要的更新并已经发送到了从服务器。线程现在正空闲，等待由主服务器上新的更新导致的出现在二进制日志中的新事件。

- Waiting to finalize termination
线程停止时发生的一个很简单的状态。

#### 7.2 主从复制从I/O线程状态
下面列出了从服务器的I/O线程的State列的最常见的状态    

- Connecting to master
线程正试图连接主服务器。

- Checking master version
建立同主服务器之间的连接后立即临时出现的状态。

- Registering slave on master
建立同主服务器之间的连接后立即临时出现的状态。

- Requesting binlog dump
建立同主服务器之间的连接后立即临时出现的状态。线程向主服务器发送一条请求，索取从请求的二进制日志文件名和位置开始的二进制日志的内容。

- Waiting to reconnect after a failed binlog dump request
如果二进制日志转储请求失败(由于没有连接)，线程进入睡眠状态，然后定期尝试重新连接。可以使用–master-connect-retry选项指定重试之间的间隔。

- Reconnecting after a failed binlog dump request
线程正尝试重新连接主服务器。

- Waiting for master to send event
线程已经连接上主服务器，正等待二进制日志事件到达。如果主服务器正空闲，会持续较长的时间。如果等待持续slave_read_timeout秒，则发生超时。此时，线程认为连接被中断并企图重新连接。

- Queueing master event to the relay log
线程已经读取一个事件，正将它复制到中继日志供SQL线程来处理。

- Waiting to reconnect after a failed master event read
读取时(由于没有连接)出现错误，线程企图重新连接前将睡眠master-connect-retry秒。

- Reconnecting after a failed master event read
线程正尝试重新连接主服务器，当连接重新建立后，状态变为Waiting for master to send event。

- Waiting for the slave SQL thread to free enough relay log space
正使用一个非零relay_log_space_limit值，中继日志已经增长到其组合大小超过该值。I/O线程正等待直到SQL线程处理中继日志内容并删除部分中继日志文件来释放足够的空间。

- Waiting for slave mutex on exit
线程停止时发生的一个很简单的状态。

### 8、主从同步灾难恢复：
- 主库的MySQL服务故障，这样可以把主库的Binlog日志拷贝至从库上，补全从库的Binlog，然后把这个从库提升为主库，如果是一主多从的架构，那么要看每个从库的master-info ` cat /mydata/data/master.info` 信息哪个更接近最新
- 主库的MySQL服务器宕机

#### 8.1 确保所有的relay log全部更新完毕
```ruby
在每个从库执行：
MariaDB [(none)]> stop slave io_thread;
MariaDB [(none)]> show processlist\G 
直到看到State: Slave has read all relay log;表示从库更新都执行完毕；
```
#### 8.2 登陆从库：
```ruby
[root@MySQL2-Slave ~]# mysql -usystem -pwangzongyu
MariaDB [(none)]> stop slave;
MariaDB [(none)]> reset master;
```
#### 8.3 从服务器进入数据目录，删除master.info和relay-log.info：
```ruby
[root@MySQL2-Slave ~]# cd /mydata/data/
[root@MySQL2-Slave data]# rm -rf master.info relay-log.info
```
#### 8.4 提升从库为主库：
```ruy
开启：log-bin=/ourdata/binlog/mysql-bin

// 如果存在read-only和log-slave-updates等一定要注释
[root@MySQL2-Slave ~]# service mysqld restart
```

#### 8.5 补全新主库的Binlog：
如果原主库服务器没有宕机，需要去原主库拉取Binlog补全新主库

#### 8.6 其他从库操作：
```ruby
MariaDB [(none)]> stop slave;
MariaDB [(none)]> change master to master_host='172.30.105.121';
MariaDB [(none)]> start slave;
MariaDB [(none)]> show slave status\G

```
`或者直接使用半同步功能，直接选择做了实时同步那个从库提升为主库`

### 9、主从不同步解决方案：
#### 9.1 使用set global sql_slave_skip_counter：
```ruby
[root@MySQL2-Slave ~]# echo "stop slave; set global sql_slave_skip_counter=1; slave start;" | mysql -usystem -pwangzongyu 
```
#### 9.2 重新做SLAVE：
重新配置一遍slave端



## MySQL的级联同步
- 当前从库还要作为其他从库的主库，也就是级联同步
- 把从库作为备份服务器时需要开启binlog

### 1、MySQL从库记录binlog的方法：
```ruby
log-bin=/ourdata/binlog/mysql-bin
log-slave-updates
// 上面这两个必须开启

expire_logs_days = 7 
// 只留最近7天的binlog日志
// find /ourdata/binlog/ -type f -name "mysql-bin.000* -mtime +7 | xargs rm -f

```

## MySQL双主及多主同步

### 1、解决主键自增长变量冲突：
```ruby

Master1:
[mysqld]
auto_increment_increment = 2  
// 自增ID的间隔，如1,3,5
auto_increment_offset =1
// ID的初始位置

Master2:
auto_increment_increment = 2  
auto_increment_offset =2

// 两个主如果都可以写权限，那么ID可能不连续，Master1写1,3,5，然后Master2写的ID是6,8,10，再可能Master又来写的请求，ID就从11开始
```
### 2、配置双主模式：
##### MySQL主服务器：
```ruby
[root@MySQlL1-Master ~]# vim /etc/my.cnf
[mysqld]
auto_increment_increment = 2
auto_increment_offset =1
log-slave-updates
log-bin=/ourdata/binlog/mysql-bin
```

##### MySQL从服务器：
```ruby
[root@MySQL2-Slave ~]# vim /etc/my.cnf 
[mysqld]
auto_increment_increment = 2
auto_increment_offset =2
log-slave-updates
log-bin=/ourdata/binlog/mysql-bin

```
`其他的过程和MySQL主从同步一致，此处不再赘述`

### 3、测试：
```ruby
Master1：
MariaDB [(none)]> create database My_DB;
MariaDB [(none)]> use My_DB;
MariaDB [(none)]> create table MyTB(
id int(4) not null AUTO_INCREMENT,
name char(20) not null,
primary key(id)
);
// 创建一个自增的表

MariaDB [My_DB]> insert into MyTB(name) values('wangzy1');
MariaDB [My_DB]> insert into MyTB(name) values('wangzy2');
MariaDB [My_DB]> select * from MyTB;
+----+---------+
| id | name    |
+----+---------+
|  1 | wangzy1 |
|  3 | wangzy2 |
+----+---------+

Master2：
MariaDB [(none)]> use My_DB;
MariaDB [My_DB]> insert into MyTB(name) values('wangzy3');
MariaDB [My_DB]> insert into MyTB(name) values('wangzy4');
MariaDB [My_DB]> select * from MyTB;
+----+---------+
| id | name    |
+----+---------+
|  1 | wangzy1 |
|  3 | wangzy2 |
|  4 | wangzy3 |
|  6 | wangzy4 |
+----+---------+

```

## MySQL半同步复制
优点：确保至少一个从库和主库的数据一致；
缺点：主从之间网络延迟，或者从库有问题的时候，用户体验很差，当然也可以设置超时时间=10秒；

### 1、安装插件
##### MySQL主服务器：
```ruby
MariaDB [(none)]> help install;
MariaDB [(none)]> INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';
MariaDB [(none)]> show plugins;
+--------------------------------+--------+--------------------+--------------------+---------+
| Name                           | Status | Type               | Library            | License |
+--------------------------------+--------+--------------------+--------------------+---------+
| rpl_semi_sync_master           | ACTIVE | REPLICATION        | semisync_master.so | GPL     |
+--------------------------------+--------+--------------------+--------------------+---------+

```
##### MySQL从服务器：
```ruby
MariaDB [(none)]> INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';
MariaDB [(none)]> show plugins;
+--------------------------------+--------+--------------------+-------------------+---------+
| Name                           | Status | Type               | Library           | License |
+--------------------------------+--------+--------------------+-------------------+---------+
| rpl_semi_sync_slave            | ACTIVE | REPLICATION        | semisync_slave.so | GPL     |
+--------------------------------+--------+--------------------+-------------------+---------+

```

### 2、开启半同步复制变量：
##### MySQL主服务器：
```ruby
MariaDB [(none)]> SHOW GLOBAL VARIABLES LIKE '%semi%';
+------------------------------------+-------+
| Variable_name                      | Value |
+------------------------------------+-------+
| rpl_semi_sync_master_enabled       | OFF   |
| rpl_semi_sync_master_timeout       | 10000 |
| rpl_semi_sync_master_trace_level   | 32    |
| rpl_semi_sync_master_wait_no_slave | ON    |
+------------------------------------+-------+

MariaDB [(none)]> SET GLOBAL rpl_semi_sync_master_enabled=1;
// 需要将rpl_semi_sync_master_enabled设置为ON，开启半同步复制


MariaDB [(none)]> SHOW GLOBAL STATUS LIKE '%semi%';
+--------------------------------------------+-------+
| Variable_name                              | Value |
+--------------------------------------------+-------+
| Rpl_semi_sync_master_clients               | 0     |
| Rpl_semi_sync_master_net_avg_wait_time     | 0     |
| Rpl_semi_sync_master_net_wait_time         | 0     |
| Rpl_semi_sync_master_net_waits             | 0     |
| Rpl_semi_sync_master_no_times              | 0     |
| Rpl_semi_sync_master_no_tx                 | 0     |
| Rpl_semi_sync_master_status                | OFF   |
| Rpl_semi_sync_master_timefunc_failures     | 0     |
| Rpl_semi_sync_master_tx_avg_wait_time      | 0     |
| Rpl_semi_sync_master_tx_wait_time          | 0     |
| Rpl_semi_sync_master_tx_waits              | 0     |
| Rpl_semi_sync_master_wait_pos_backtraverse | 0     |
| Rpl_semi_sync_master_wait_sessions         | 0     |
| Rpl_semi_sync_master_yes_tx                | 0     |
+--------------------------------------------+-------+

// Rpl_semi_sync_master_clients是有多少个半同步节点，还没开始配置，所以目前为0
```
##### MySQL从服务器：
```ruby
MariaDB [(none)]> SHOW GLOBAL VARIABLES LIKE '%semi%';
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| rpl_semi_sync_slave_enabled     | OFF   |
| rpl_semi_sync_slave_trace_level | 32    |
+---------------------------------+-------+

MariaDB [(none)]> SET GLOBAL rpl_semi_sync_slave_enabled=1;
// 需要将rpl_semi_sync_slave_enabled设置为ON，开启半同步复制
```
### 3、检查测试：
#### 3.1 通过增删改查看主库的变量值的变化：
##### MySQL从服务器：
```ruby
MariaDB [(none)]> slave start;
```
##### MySQL主服务器：
```ruby
MariaDB [(none)]> SHOW GLOBAL STATUS LIKE '%semi%';
+--------------------------------------------+-------+
| Variable_name                              | Value |
+--------------------------------------------+-------+
| Rpl_semi_sync_master_clients               | 1     |
| Rpl_semi_sync_master_net_avg_wait_time     | 0     |
| Rpl_semi_sync_master_net_wait_time         | 0     |
| Rpl_semi_sync_master_net_waits             | 0     |
| Rpl_semi_sync_master_no_times              | 0     |
| Rpl_semi_sync_master_no_tx                 | 0     |
| Rpl_semi_sync_master_status                | ON    |
| Rpl_semi_sync_master_timefunc_failures     | 0     |
| Rpl_semi_sync_master_tx_avg_wait_time      | 0     |
| Rpl_semi_sync_master_tx_wait_time          | 0     |
| Rpl_semi_sync_master_tx_waits              | 0     |
| Rpl_semi_sync_master_wait_pos_backtraverse | 0     |
| Rpl_semi_sync_master_wait_sessions         | 0     |
| Rpl_semi_sync_master_yes_tx                | 0     |
+--------------------------------------------+-------+

// Rpl_semi_sync_master_clients这项的值已经变成了1

```
```ruby
MariaDB [(none)]> create database MYDB;
MariaDB [(none)]> use MYDB;
MariaDB [MYDB]> create table TB1 (id int,name char(30));

MariaDB [MYDB]> SHOW GLOBAL STATUS LIKE '%semi%';
+--------------------------------------------+-------+
| Variable_name                              | Value |
+--------------------------------------------+-------+
| Rpl_semi_sync_master_clients               | 1     |
| Rpl_semi_sync_master_net_avg_wait_time     | 2034  |
| Rpl_semi_sync_master_net_wait_time         | 4068  |
| Rpl_semi_sync_master_net_waits             | 2     |
| Rpl_semi_sync_master_no_times              | 0     |
| Rpl_semi_sync_master_no_tx                 | 0     |
| Rpl_semi_sync_master_status                | ON    |
| Rpl_semi_sync_master_timefunc_failures     | 0     |
| Rpl_semi_sync_master_tx_avg_wait_time      | 1949  |
| Rpl_semi_sync_master_tx_wait_time          | 3898  |
| Rpl_semi_sync_master_tx_waits              | 2     |
| Rpl_semi_sync_master_wait_pos_backtraverse | 0     |
| Rpl_semi_sync_master_wait_sessions         | 0     |
| Rpl_semi_sync_master_yes_tx                | 2     |
+--------------------------------------------+-------+

// Rpl_semi_sync_master_net_waits=2 正好使用了两条语句，所以wait值是2
// Rpl_semi_sync_master_net_wait_time=4068，总的等待时间是4068毫秒
// Rpl_semi_sync_master_net_avg_wait_time=2034 每条语句需要等待2034秒，大约2秒钟
```
#### 3.2 通过tcpdump抓包比较半同步复制与异步复制的区别：
##### 半同步复制：
```ruby
主库(172.30.105.121)插入一条数据会同步到从库(172.30.105.122)：
MariaDB [MyDB]> insert into MyTB(name) values('wangzy102');

[root@MySQlL1-Master ~]# tcpdump -n -i eth0 host 172.30.105.121 and 172.30.105.122
// 表示抓取172.30.105.121与172.30.105.122之间的数据包

20:55:58.347048 IP 172.30.105.121.mysql > 172.30.105.122.41056: Flags [P.], seq 4254710471:4254710727, ack 1486459343, win 114, options [nop,nop,TS val 156958776 ecr 155948401], length 256
20:55:58.349664 IP 172.30.105.122.41056 > 172.30.105.121.mysql: Flags [P.], seq 1:30, ack 256, win 165, options [nop,nop,TS val 156158332 ecr 156958776], length 29
20:55:58.349922 IP 172.30.105.121.mysql > 172.30.105.122.41056: Flags [.], ack 30, win 114, options [nop,nop,TS val 156958779 ecr 156158332], length 0

20:56:03.353127 ARP, Request who-has 172.30.105.122 tell 172.30.105.121, length 28
20:56:03.353798 ARP, Request who-has 172.30.105.121 tell 172.30.105.122, length 46
20:56:03.353816 ARP, Reply 172.30.105.121 is-at 00:0c:29:54:7a:ef, length 28
20:56:03.353867 ARP, Reply 172.30.105.122 is-at 00:0c:29:da:65:76, length 46

```
##### 异步复制：
```ruby
主库(172.30.105.121)插入一条数据会同步到从库(172.30.105.122)：
MariaDB [MyDB]> insert into MyTB(name) values('wangzy102');

[root@MySQlL1-Master ~]# tcpdump -n -i eth0 host 172.30.105.121 and 172.30.105.122

18:08:01.910215 IP 172.30.105.121.mysql > 172.30.105.122.41055: Flags [P.], seq 2094180227:2094180475, ack 4278786909, win 114, options [nop,nop,TS val 146882340 ecr 145287155], length 248
18:08:01.911020 IP 172.30.105.122.41055 > 172.30.105.121.mysql: Flags [.], ack 248, win 148, options [nop,nop,TS val 146368186 ecr 146882340], length 0

18:08:06.922866 ARP, Request who-has 172.30.105.122 tell 172.30.105.121, length 28
18:08:06.922968 ARP, Request who-has 172.30.105.121 tell 172.30.105.122, length 46
18:08:06.922988 ARP, Reply 172.30.105.121 is-at 00:0c:29:54:7a:ef, length 28
18:08:06.925632 ARP, Reply 172.30.105.122 is-at 00:0c:29:da:65:76, length 46
```