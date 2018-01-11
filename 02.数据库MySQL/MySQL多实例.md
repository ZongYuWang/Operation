## MySQL多实例

### MySQL多实例常见配置方案
#### 多配制文件部署方案
`通过配置多个配置文件及多个启动程序来实现多实例的方案`



### 安装多实例MySQL数据库：

#### 安装好MySQL并安装依赖包：
`为突出本节实验内容直接采用已经编译好的MySQL软件包(mariadb-5.5.56-linux-x86_64.tar.gz)来使用`
```ruby
[root@mysql ~]# yum install ncurses-devel
[root@mysql ~]# yum install libaio-devel 

[root@mysql ~]# tar xvf mariadb-5.5.56-linux-x86_64.tar.gz
[root@mysql ~]# mv mariadb-5.5.56-linux-x86_64 /usr/local/mysql

[root@mysql ~]# mkdir -p /mydata/data  // 这个目录应该是安装MariaDB部分的LVM逻辑卷，为了演示接下来的实验内容，直接创建一个目录
```
当然安装MySQL部分可以使用MySQL(MariaDB)安装章节中的任意一种方式，如果是编译安装，则到make install之后截止；

#### 建立MySQL账号：
```ruby
[root@mysql ~]# groupadd -r mysql
[root@mysql ~]# useradd -g mysql -r -s /sbin/nologin -M -d /mydata/data mysql
```

#### 创建MySQL多实例的数据文件目录：
```ruby
[root@mysql ~]# mkdir -p /mydata/data/{3306,3307}/data
[root@mysql ~]# tree /mydata/data/
/mydata/data/    //总的多实例目录
├── 3306         //3306实例目录
│   └── data     //3306实例的数据文件目录
└── 3307         //3307实例目录
    └── data

4 directories, 0 files

```
配置3306实例
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
配置3307实例
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

[root@mysql ~]# chown -R mysql.mysql /mydata/data/
```

初始化MySQL多实例：
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

MySQL多实例启动脚本：
```rubby
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
      
```

启动和关闭3306实例：
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

添加环境变量：
```ruby
[root@mysql ~]# vim /etc/profile
export PATH=/usr/local/mysql/bin:$PATH
[root@mysql ~]# source /etc/profile

```
#### 为MySQL的root账号添加密码：
```ruby
[root@mysql ~]# /usr/local/mysql/bin/mysqladmin -u root -S /mydata/data/3306/mysql.sock password '123456'
[root@mysql ~]# mysql -S /mydata/data/3306/mysql.sock -p

```

启动和关闭3307实例：
```ruby
[root@mysql ~]# cp /mydata/data/3306/mysql /mydata/data/3307/
// 需要修改一下mysql脚本中的port=3307
[root@mysql ~]# chown mysql.mysql /mydata/data/3307/mysql
[root@mysql 3307]# /mydata/data/3307/mysql start
MySQL is running...

```

查看MySQL多实例并测试：
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