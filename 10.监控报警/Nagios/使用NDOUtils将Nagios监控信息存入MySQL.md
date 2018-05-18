
## 使用NDOUtils将Nagios监控信息存入MySQL    

![](https://github.com/ZongYuWang/image/tree/master/Nagios/Nagios-NDOUtils1.png)

### 1、安装配置MySQL：
```ruby
# yum install mysql mysql-server mysql-devel perl-DBD-MySQL
mysql> USE mysql;
mysql> update user set Password=password('newpassword') where User='root';
mysql> flush privileges;
mysql> create database nagios;
mysql> quit
```

### 2、安装ndoitils：
```ruby
#下面是yum安装mysql之后，编译ndoitils的方式:
[root@localhost ~]# mkdir /nagios
[root@localhost ~]# cd /nagios/
[root@localhost nagios]# tar xvf ndoutils-2.0.0.tar.gz
[root@localhost nagios]# cd ndoutils-2.0.0
[root@localhost ndoutils-2.0.0]# ./configure --prefix=/usr/local/nagios LDFLAGS=-L/usr/lib64 --with-mysql-inc=/usr/include/mysql --with-mysql-lib=/usr/lib64/mysql --enable-mysql --disable-pgsql --with-ndo2db-user=nagios --with-ndo2db-group=nagios
# make

#下面是编译安装mysql之后，编译ndoutils的方式:
[root@localhost ndoutils-2.0.0]# ./configure --prefix=/usr/local/nagios --with-mysql-inc=/usr/local/mysql/include/ --with-mysql-lib=/usr/local/mysql/lib/ --with-mysql=/usr/local/mysql/ --enable-mysql --disable-pgsql --with-ndo2db-user=nagios --with-ndo2db-group=nagios
#make
【说明】make完之后，不要make install
```
`make可能会报如下错误:`
```ruby
[root@localhost ndoutils-2.0.0]# make && echo ok
cd ./src && make
make[1]: Entering directory `/nagios/ndoutils-2.0.0/src'
gcc -fPIC -g -O2 -I/usr/include/mysql -DHAVE_CONFIG_H  -c -o io.o io.c
In file included from io.c:11:
../include/config.h:261:25: error: mysql/mysql.h: No such file or directory
../include/config.h:262:26: error: mysql/errmsg.h: No such file or directory
make[1]: *** [io.o] Error 1
make[1]: Leaving directory `/nagios/ndoutils-2.0.0/src'
make: *** [all] Error 2

解决办法：
[root@localhost ndoutils-2.0.0]# yum install -y mysql-devel
[root@localhost ndoutils-2.0.0]# make clean all
[root@localhost ndoutils-2.0.0]# make
```
### 3、配置ndoitils：
```ruby
[root@localhost ~]# cp config/{ndo2db.cfg-sample,ndomod.cfg-sample} /usr/local/nagios/etc  
[root@localhost ~]# mv /usr/local/nagios/etc/ndo2db.cfg-sample /usr/local/nagios/etc/ndo2db.cfg
[root@localhost ~]# mv /usr/local/nagios/etc/ndomod.cfg-sample /usr/local/nagios/etc/ndomod.cfg 
[root@localhost ~]# cp src/ndomod-3x.o /usr/local/nagios/bin/
[root@localhost ~]# cp src/ndo2db-3x /usr/local/nagios/bin/

[root@localhost ~]# chmod 644 /usr/local/nagios/etc/ndo*  
[root@localhost ~]# chown nagios:nagios /usr/local/nagios/etc/*
[root@localhost ~]# chown nagios:nagios /usr/local/nagios/bin/*
```
```ruby
[root@localhost ~]# vim /usr/local/nagios/etc/nagios.cfg 
event_broker_options=-1
broker_module=/usr/local/nagios/bin/ndomod-3x.o config_file=/usr/local/nagios/etc/ndomod.cfg
【说明】broker_module和config_file要写在同一行上，写两行nagios启动会有问题

[root@localhost ~]# vim /usr/local/nagios/etc/ndo2db.cfg 
socket_type=tcp
db_servertype=mysql
db_host=localhost
db_port=3306
db_name=nagios
db_prefix=nagios_
db_user=root
db_pass=wangzongyu

[root@localhost ~]# vim /usr/local/nagios/etc/ndomod.cfg 
output_type=tcpsocket
output=127.0.0.1
#output=/usr/local/nagios/var/ndo.sock   //注销此行

mysql> create database nagios;
mysql> GRANT SELECT,INSERT,UPDATE,DELETE ON nagios.* TO root@localhost IDENTIFIED BY 'wangzongyu';
[root@localhost ndoutils-2.0.0]#  cd db/
[root@localhost db]# ./installdb -u root -p wangzongyu -h localhost -d nagios
【说明】-d表示目标数据库
DBD::mysql::db do failed: Table 'nagios.nagios_dbversion' doesn't exist at ./installdb line 51.
** Creating tables for version 2.0.1
     Using mysql.sql for installation...
** Updating table nagios_dbversion
Done!

```
`可能会出现如下错误：`
```ruby
[root@localhost db]# ./installdb -u root -p wangzongyu -h localhost -d nagios
DBI connect('database=nagios;host=localhost','root',...) failed: Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2) at ./installdb line 41

解决办法：
[root@localhost ~]# find / -name mysql.sock
/tmp/mysql.sock
[root@localhost ~]# mkdir /var/lib/mysql/
[root@localhost ~]# ln -s /tmp/mysql.sock /var/lib/mysql/mysql.sock
```

### 4、设置ndo2db开机自启动：
```ruby

[root@localhost ndoutils-2.0.0]# cd /nagios_soft/ndoutils-2.0.0
[root@localhost ndoutils-2.0.0]# cp ./daemon-init /etc/init.d/ndo2db
[root@localhost ndoutils-2.0.0]# vim /etc/init.d/ndo2db 
    Ndo2dbBin=/usr/local/nagios/bin/ndo2db-3x
[root@localhost ndoutils-2.0.0]# chmod +x /etc/init.d/ndo2db 
[root@localhost ndoutils-2.0.0]# service ndo2db start
```
#### 4.1 FAQ：
```ruby
[root@localhost ndoutils-2.0.0]# service ndo2db start
Starting ndo2db:/usr/local/nagios/bin/ndo2db: error while loading shared libraries: libmysqlclient.so.18: cannot open shared object file: No such file or directory
done.

解决办法：
[root@localhost ~]# ln -s /usr/local/mysql/lib/* /usr/lib64
```
#### 4.2 检查开启的端口：
```ruby
[root@localhost ~]# netstat -antup | grep 5668
tcp        0      0 0.0.0.0:5668                0.0.0.0:*                   LISTEN      121725/ndo2db-3x 
```
```ruby
[root@localhost ~]# service ndo2db stop
Stopping ndo2db: head: cannot open `/usr/local/nagios/var/ndo2db.lock' for reading: No such file or directory
done.
【说明】关闭ndo2db会存在问题
```
[ndo2pnp.pl下载地址]( https://github.com/ZongYuWang/File/tree/master/File/Nagios )
```ruby

[root@localhost ~]# cd /usr/local/nagios/libexec/
[root@localhost ~]# chmod +x ndo2pnp.pl  // 将附件中的ndo2pnp.pl上传到上面的目录中

[root@localhost libexec]# ./ndo2pnp.pl --help
Usage :
  -h  --help            Display this message.
      --version         Display version then exit.
  -v  --verbose         Verbose run.
  -u  --user <NDOUSER>  Log on to database with <NDOUSER> (default root).
  -p  --pass <PASSWD>   Use <PASSWD> to logon (default root).
  -t  --type <DBTYPE>   Change database type (default mysql).
      --host <DBHOST>   Use <DBHOST> (default localhost).
      --dbname <DB>     Use <DB> for ndo database name (default ndoutils).
      --list-machine    Display machine definition in ndo database.
      --list-service    Show services defined.
      --export-as-pnp   Export ndo content as a bulk file used by process_perfdata.pl.

```

#### 4.3 查看存入数据库的数据：
```ruby
[root@localhost libexec]# ./ndo2pnp.pl -u root -p Tianjin_sunvsoft.2017! --dbname nagios --list-service
Hostname                       | Service
-------------------------------+-------------------
192.168.1.9                    | IoInfo
192.168.1.9                    | MemInfo
192.168.1.9                    | PartitionInfo
192.168.1.9                    | RootPartitionInfo
192.168.1.9                    | System Load
B2B-DB                         | IoInfo
B2B-DB                         | MemInfo
B2B-DB                         | PartitionInfo
B2B-DB                         | RootPartitionInfo
B2B-DB                         | System Load
B2B-WEB                        | IoInfo
B2B-WEB                        | MemInfo
B2B-WEB                        | PartitionInfo
B2B-WEB                        | RootPartitionInfo
B2B-WEB                        | System Load
```

#### 4.4 查看连接数据库的日志：
```ruby
[root@localhost ~]# tail -f 1000 /usr/local/nagios/var/nagios.log
tail: cannot open `1000' for reading: No such file or directory
==> /usr/local/nagios/var/nagios.log <==
[1508226586] ndomod: Successfully flushed 92 queued items to data sink.
[1508226586] ndomod: Error writing to data sink!  Some output may get lost...
[1508226586] ndomod: Please check remote ndo2db log, database connection or SSL Parameters
[1508226597] Warning: Return code of 255 for check of service 'MemInfo' on host 'Import-buyer-DB-backup' was out of bounds.
[1508226597] Warning: Return code of 255 for check of service 'PartitionInfo' on host 'Import-buyer-DB1' was out of bounds.
[1508226602] ndomod: Successfully reconnected to data sink!  0 items lost, 243 queued items to flush.
[1508226602] ndomod: Successfully flushed 243 queued items to data sink.
```
