## MySQL多实例


### 1、什么是MySQL多实例：
简单的说，就是在一台机器上开启多个不同的服务端口(如:3306,3306),运行多个MySQL服务进程，这些服务进程通过不同的socket监听不同的服务端口来提供各自的服务。
  
这些MySQL多实例共用一套MySQL安装程序，使用不同(也可以相同)的my.cnf配置文件、启动程序，数据文件。在提供服务时，多实例MySQL在逻辑上看来是各自独立的，多个实例的自身是根据配置文件对应的设定值，来取得服务的相关硬件资源多少。

### 2、MySQL多实例的作用与问题：
#### 2.1 有效利用服务器资源
当单个服务器资源有剩余时，可以充分利用剩余的资源提供更多的服务。
#### 2.2 节约服务器资源
当公司资金紧张，但是数据库又需要各自尽量独立提供服务，而且，需要主从同步等技术时，多实例就再好不过了
#### 2.3 资源互相抢占问题
当某个服务实例并发很高或者有慢查询时，整个实例会消耗更多的内存、CPU、磁盘、IO资源，导致服务器上的其他的实例提供服务的质量下降。

### 3、MySQL多实例生产应用场景：
#### 3.1 资金紧张型公司的选择：   
当公司业务访问量不太大，又舍不得花钱，但又希望不同业务的数据库服务各自尽量独立的提供服务互相不受影响时。
#### 3.2 并发访问不是特别大的的业务：  
当公司业务访问量不太大的时候，服务器的资源基本都是浪费的，这时就很适合多实例的应用。
#### 3.3 门户网站应用MySQL多实例场景：   
百度搜索引擎的数据库就是多实例，一般是从库，内存96G，跑3-4个实例，新浪网也是用的多实例，内存48G左右，门户网站使用多实例的目的是配硬件好的服务器，节省IDC机柜空间，同时，跑多实例让硬件资源不浪费。

### 4、MySQL多实例常见配置方案
#### 4.1 多配制文件部署方案：
`通过配置多个配置文件及多个启动程序来实现多实例的方案`
```ruby
[root@mysql ~]# tree /mydata/data/
/mydata/data/
├── 3306
│   ├── data     //3306实例的数据文件
│   ├── my.cnf   //3306实例的配置文件
│   ├── mysql    //实例的启动文件
└── 3307
    ├── data
    ├── my.cnf
    ├── mysql

```
#### 4.2 单一配置文件部署方案：

```ruby
/etc/my.cnf   // 统一的一个配置文件
/usr/local/mysql/bin/mysqld_multi  // 启动/关闭多实例
```

### 5、安装多实例MySQL数据库：

#### 5.1 安装好MySQL并安装依赖包：
`为突出本节实验内容直接采用已经编译好的MySQL软件包(mariadb-5.5.56-linux-x86_64.tar.gz)来使用`
```ruby
[root@mysql ~]# yum install ncurses-devel
[root@mysql ~]# yum install libaio-devel 

[root@mysql ~]# tar xvf mariadb-5.5.56-linux-x86_64.tar.gz
[root@mysql ~]# mv mariadb-5.5.56-linux-x86_64 /usr/local/mysql

[root@mysql ~]# mkdir -p /mydata/data  // 这个目录应该是安装MariaDB部分的LVM逻辑卷，为了演示接下来的实验内容，直接创建一个目录
```
当然安装MySQL部分可以使用MySQL(MariaDB)安装章节中的任意一种方式，如果是编译安装，则到make install之后截止；

#### 5.2 建立MySQL账号：
```ruby
[root@mysql ~]# groupadd -r mysql
[root@mysql ~]# useradd -g mysql -r -s /sbin/nologin -M -d /mydata/data mysql
```

#### 5.3 创建MySQL多实例的数据文件目录：
```ruby
[root@mysql ~]# mkdir -p /mydata/data/{3306,3307}/data
[root@mysql ~]# tree /mydata/data/
/mydata/data/    //总的多实例目录
├── 3306         //3306实例目录
│   └── data     //3306实例的数据文件目录
└── 3307         //3307实例目录
    └── data

4 directories, 0 files

[root@mysql ~]# chown -R mysql.mysql /mydata/data/
```
#### 5.4 多配制文件方案的实例配置文件：
`单一配置文件部署方案的实例配置文件见下方`

##### 3306实例配置文件：
```ruby
[root@mysql ~]# grep -v "^#"  /usr/local/mysql/support-files/my-large.cnf | grep -v "^$" >> /mydata/data/3306/my.cnf

[root@mysql ~]# vim /mydata/data/3306/my.cnf
[client]
port            = 3306
socket          = /mydata/data/3306/mysql.sock
[mysqld]
port            = 3306
socket          = /mydata/data/3306/mysql.sock
datadir = /mydata/data/3306/data
log-error = /mydata/data/3306/error.log

log-slow-queries=/mydata/data/3306/queries-slow.log
pid-file=/mydata/data/3306/mysqld.pid
log-bin=/mydata/data/3306/bin-log
server-id = 1

```
##### 3307实例配置文件：
`[root@mysql ~]# sed -i s/3306/3307/g /mydata/data/3307/my.cnf`
```ruby
[root@mysql1 ~]# grep -v "^#"  /usr/local/mysql/support-files/my-large.cnf | grep -v "^$" >> /mydata/data/3307/my.cnf

[root@mysql ~]# vim /mydata/data/3307/my.cnf 
[client]
port            = 3307
socket          = /mydata/data/3307/mysql.sock
[mysqld]
port            = 3307
socket          = /mydata/data/3307/mysql.sock
datadir = /mydata/data/3307/data
log-error = /mydata/data/3307/error.log

log-slow-queries=/mydata/data/3307/queries-slow.log
pid-file=/mydata/data/3307/mysqld.pid
log-bin=/mydata/data/3307/bin-log
server-id = 2
```
```ruby
[root@mysql ~]# tree /mydata/data/
/mydata/data/
├── 3306
│   ├── data
│   └── my.cnf
└── 3307
    ├── data
    └── my.cnf

4 directories, 2 files

```

#### 5.5 单一配置文件部署方案的实例配置文件:

```ruby
[root@mysql ~]# vim /mydata/data/my.cnf配置文件样例：

[mysqld_multi]
mysqld     = /usr/local/mysql/bin/mysqld_safe
mysqladmin = /usr/local/mysql/bin/mysqladmin
log = /mydata/data/multi.log
#user       = multi_admin  // 此帐户用于多实例关闭时使用，需要在每个实例上创建并授权
#password   = my_password  // 使用统一的密码便于管理


[mysqld3306]
socket=/mydata/data/3306/mysql.sock
port = 3306
pid-file=/mydata/data/3306/mysqld.pid
datadir=/mydata/data/3306/data/
basedir=/usr/local/mysql
user = mysql
server-id=3306
log-error = /mydata/data/3306/error.log
#log-bin=/mydata/data/3306/log/bin-log/bin-log
#log-bin-index = /mydata/data/3306/log/bin-log/master-bin.index
#log-error=/mydata/data/3306/log/mysqld.log

[mysqld3307]
socket=/mydata/data/3307/mysql.sock
port = 3307
pid-file=/mydata/data/3307/mysqld.pid
datadir=/mydata/data/3307/data/
basedir=/usr/local/mysql
user = mysql
server-id=3307
log-error = /mydata/data/3307/error.log
#log-bin=/mydata/data/3307/log/bin-log/bin-log
#log-bin-index = /mydata/data/3307/log/bin-log/master-bin.index
#log-error=/mydata/data/3307/log/mysqld.log

```

#### 5.6 初始化MySQL多实例：
`多配制文件方案和单一配置文件方案都需要初始化`
```ruby
[root@mysql ~]# cd /usr/local/mysql/scripts/

[root@mysql scripts]# ./mysql_install_db --basedir=/usr/local/mysql/ --datadir=/mydata/data/3306/data/ --user=mysql
WARNING: The host 'mysql' could not be looked up with resolveip.
This probably means that your libc libraries are not 100 % compatible
with this binary MariaDB version. The MariaDB daemon, mysqld, should work
normally with the exception that host name resolving will not work.
This means that you should use IP addresses instead of hostnames
when specifying MariaDB privileges !
Installing MariaDB/MySQL system tables in '/mydata/data/3306/data/' ...
180104 23:42:58 [Note] /usr/local/mysql//bin/mysqld (mysqld 5.5.56-MariaDB) starting as process 1559 ...
OK
Filling help tables...
180104 23:42:59 [Note] /usr/local/mysql//bin/mysqld (mysqld 5.5.56-MariaDB) starting as process 1568 ...
OK
......

[root@mysql scripts]# ./mysql_install_db --basedir=/usr/local/mysql/ --datadir=/mydata/data/3307/data/ --user=mysql
```

#### 5.7 MySQL多实例启动：
##### 添加环境变量：
```ruby
[root@mysql ~]# vim /etc/profile
export PATH=/usr/local/mysql/bin:$PATH
[root@mysql ~]# source /etc/profile

```

##### 多配制文件方案启动/关闭脚本：
```ruby
#!/bin/bash

#init
port=3306
mysql_user="root"
mysql_pwd=""
CmdPath="/usr/local/mysql/bin"
mysql_sock="/mydata/data/${port}/mysql.sock"

#startup function
function_start_mysql()
{
 if [ ! -e "$mysql_sock" ];then
  printf "Starting MySQL...\n"
  /bin/sh ${CmdPath}/mysqld_safe --defaults-file=/mydata/data/${port}/my.cnf 2>&1 > /dev/null &
 else
  printf "MySQL is running...\n"
 exit
 fi
}


#stop function
function_stop_mysql()
{
      if [ ! -e "$mysql_sock" ];then
        printf "MySQL is stopped...\n"
        exit
      else
        printf "Stoping MySQL...\n"
        ${CmdPath}/mysqladmin -u ${mysql_user} -p${mysql_pwd} -S /mydata/data/${port}/mysql.sock shutdown
      fi
}

#restart function
function_restart_mysql()
{
  printf "Restarting MySQL...\n"
  function_stop_mysql
  sleep 2
  function_start_mysql
}

case $1 in
start)
 function_start_mysql
;;
stop)
 function_stop_mysql
;;
restart)
 function_restart_mysql
;;
*)
  printf "Usage: /data/${port}/mysql {start|stop|restart}\n"
esac      
      
```

`启动和关闭3306实例：`
```ruby
[root@mysql 3306]# chmod +x mysql
[root@mysql 3306]# /mydata/data/3306/mysql start
Starting MySQL...
[root@mysql 3306]# netstat -tlnp |grep 3306
tcp        0      0 0.0.0.0:3306                0.0.0.0:*                   LISTEN      2566/mysqld         
[root@mysql 3306]# 
[root@mysql 3306]# /mydata/data/3306/mysql stop
Stoping MySQL...
Enter password:   // 关闭需要密码的

```

`启动和关闭3307实例：`
```ruby
[root@mysql ~]# cp /mydata/data/3306/mysql /mydata/data/3307/
// 需要修改一下mysql脚本中的port=3307
[root@mysql ~]# chown mysql.mysql /mydata/data/3307/mysql
[root@mysql 3307]# /mydata/data/3307/mysql start
MySQL is running...

```

`查看MySQL多实例并测试：`
```ruby
[root@mysql 3307]# netstat -tlnp | grep 330*
tcp        0      0 0.0.0.0:3307                0.0.0.0:*                   LISTEN      3382/mysqld         
tcp        0      0 0.0.0.0:3306                0.0.0.0:*                   LISTEN      2932/mysqld 
```
```ruby
[root@mysql 3307]# mysql -S /mydata/data/3307/mysql.sock 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 1
Server version: 5.5.56-MariaDB MariaDB Server

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.01 sec)

MariaDB [(none)]> create database db_3307;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| db_3307            |
| mysql              |
| performance_schema |
| test               |
+--------------------+
5 rows in set (0.00 sec)

MariaDB [(none)]> system mysql -S /mydata/data/3306/mysql.sock -p   //加上system可以执行系统命令
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 5
Server version: 5.5.56-MariaDB MariaDB Server

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.01 sec)

MariaDB [(none)]> create database db_3306;
Query OK, 1 row affected (0.02 sec)

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| db_3306            |
| mysql              |
| performance_schema |
| test               |
+--------------------+
5 rows in set (0.00 sec)

MariaDB [(none)]> exit;
Bye
MariaDB [(none)]> 

```

##### 单一配置文件部署方案启动/关闭脚本：
```ruby
[root@mysql ~]# mysqld_multi --defaults-file=/mydata/data/my.cnf report
Reporting MySQL servers
MySQL server from group: mysqld3306 is not running
MySQL server from group: mysqld3307 is not running

[root@mysql ~]# mysqld_multi --defaults-file=/mydata/data/my.cnf start 3306 --log=/tmp/mysqld_multi.log

[root@mysql ~]# mysqld_multi --defaults-file=/mydata/data/my.cnf report
Reporting MySQL servers
MySQL server from group: mysqld3306 is running
MySQL server from group: mysqld3307 is not running

[root@mysql ~]# mysqld_multi --defaults-file=/mydata/data/my.cnf start 3307 --log=/tmp/mysqld_multi.log
[root@mysql ~]# mysqld_multi --defaults-file=/mydata/data/my.cnf report
Reporting MySQL servers
MySQL server from group: mysqld3306 is running
MySQL server from group: mysqld3307 is running

[root@mysql ~]# mysqld_multi --defaults-file=/mydata/data/my.cnf stop 3306,3307
[root@mysql ~]# mysqld_multi --defaults-file=/mydata/data/my.cnf report
Reporting MySQL servers
MySQL server from group: mysqld3306 is not running
MySQL server from group: mysqld3307 is not running

```


##### 为MySQL的root账号添加密码：
`多配制文件方案设置root账号密码：`
```ruby
[root@mysql ~]# /usr/local/mysql/bin/mysqladmin -u root -S /mydata/data/3306/mysql.sock password '123456'
[root@mysql ~]# mysql -S /mydata/data/3306/mysql.sock -p

```
`单一配置文件部署方案设置root账号密码：`
```ruby
[root@mysql ~]# mysqld_multi --defaults-file=/mydata/data/my.cnf start
[root@mysql ~]# mysqld_multi --defaults-file=/mydata/data/my.cnf report
Reporting MySQL servers
MySQL server from group: mysqld3306 is running
MySQL server from group: mysqld3307 is running

[root@mysql ~]# /usr/local/mysql/bin/mysqladmin -u root -S /mydata/data/3306/mysql.sock password '123456'

[root@mysql ~]# mysql -u root -p -S /mydata/data/3306/mysql.sock 
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 4
Server version: 5.5.56-MariaDB MariaDB Server

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 

```