## 设置及修改MySQL的root用户密码

### 1、设定密码
```ruby
单实例：
[root@localhost ~]# mysqladmin -u root password 'wangzongyu'

多实例：
[root@mysql ~]# mysqladmin -u root -p'123456' password'wangzongyu' -S /mydata/data/3306/mysql.sock
// password后面有一个空格
```
### 2、修改管理员root用户密码
- 必须指定where条件；
- 必须使用password(）函数来加密更改密码；

#### 2.1 方式一：
```ruby
MariaDB [(none)]> update mysql.user set password='wang' where user='root' and host='localhost';
Query OK, 1 row affected (0.03 sec)
Rows matched: 1  Changed: 1  Warnings: 0

MariaDB [(none)]> select user,host,password from mysql.user;
+------+-----------+----------+
| user | host      | password |
+------+-----------+----------+
| root | localhost | wang     |
| root | mysql     |          |
| root | 127.0.0.1 |          |
| root | ::1       |          |
|      | localhost |          |
|      | mysql     |          |
+------+-----------+----------+
6 rows in set (0.00 sec)

// password='wang'这样在mysql.user表中是明文的存在的，使用明文是不能登陆的

密码变为密文形式存在(password=password('wang'))：
MariaDB [(none)]> update mysql.user set password=password('wang') where user='root' and host='localhost';
MariaDB [(none)]> flush privileges;

```

#### 2.2 方式二：
`此法不适合 --skip-grant-tables方式修改密码`
```ruby
MariaDB [(none)]> set password=password('wangzongyu');
MariaDB [(none)]> flush privileges;

```
## 找回丢失的MySQL的root用户密码
### 1、停止mysql服务
```ruby
[root@mysql ~]# service mysqld stop
Shutting down MySQL.... SUCCESS! 

```
### 2、忽略授权登陆
`使用--skip-grant-tables启动mysql，忽略授权登陆验证`
```ruby
[root@mysql ~]# mysqld_safe --skip-grant-tables --user=mysql &
[1] 17880
[root@mysql ~]# 180107 00:29:01 mysqld_safe Logging to '/mydata/data/mysql.err'.
180107 00:29:01 mysqld_safe Starting mysqld daemon with databases from /mydata/data

[root@mysql ~]# mysql
......
MariaDB [(none)]> 

修改密码：
MariaDB [(none)]> update mysql.user set password=password('wangzongyu') where user='root' and host='localhost';
MariaDB [(none)]> flush privileges;

mysql进程中有--skip-grant-tables：
[root@mysql ~]# ps -ef | grep mysql
root     17880 17832  0 00:29 pts/0    00:00:00 /bin/sh /usr/local/mysql/bin/mysqld_safe --skip-grant-tables --user=mysql
mysql    18210 17880  0 00:29 pts/0    00:00:01 /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/mydata/data --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --skip-grant-tables --log-error=/mydata/data/mysql.err --pid-file=mysql.pid --socket=/tmp/mysql.sock --port=3306
root     18239 17832  0 00:43 pts/0    00:00:00 grep mysql

[root@mysql ~]# service mysqld stop
Shutting down MySQL.. SUCCESS! 
[1]+  Done                    mysqld_safe --skip-grant-tables --user=mysql
[root@mysql ~]# service mysqld start
Starting MySQL.180107 01:52:15 mysqld_safe Logging to '/mydata/data/mysql.err'.
180107 01:52:15 mysqld_safe Starting mysqld daemon with databases from /mydata/data
. SUCCESS! 

```

## MySQL修改授权

### 1、授权用户权限
#### 1.1 改表法：
```ruby
mysql> use mysql;
Database changed
mysql> update user set host='%' where user='root';
mysql> select host,user,password from user;
mysql> FLUSH PRIVILEGES;
```
#### 1.2 授权法：
```ruby
mysql>GRANT ALL ON *.* TO 'root'@'%' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;
// ALL表示文章末尾提到的所有的用户权限
//*.*第一个*可以改为库名，表示授权可以连接哪个库
mysql> FLUSH PRIVILEGES;

grant select ,update on *.* to user1 [with grant option ]
// 末尾加上"WITH GRANT OPTION"表示user1可以将select ,update权限传递给其他用户如user2
```
#### 1.3 create和grant配合法：
```ruby
首先创建用户babyshen及密码babypass
MariaDB [(none)]> create user 'babyshen'@'localhost' identified by 'babypass';
授权localhost主机上通过用户babyshen管理所有数据库的所有权限
MariaDB [(none)]> grant all on *.* to 'babyshen'@'localhost';

以上两条命令相当于下面一条：
MariaDB [(none)]> grant all on *.* to 'babyshen'@'localhost' identified by 'babypass';

```
| grant | all privileges | on dbname.*  | to username@localhost | identified by 'passwd'|
| ------------- |:---------:| :-----:|:-----:|:-----:|
| 授权命令 | 对应权限 | 目标：库和表 | 用户名和客户端主机 | 用户密码|

##### 创建一个新的用户，不分配权限默认是USAGE权限
```ruby
MariaDB [(none)]> create user baby@localhost identified by 'wangzongyu';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> show grants for baby@localhost;
+-------------------------------------------------------------------------------------------------------------+
| Grants for baby@localhost                                                                                   |
+-------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'baby'@'localhost' IDENTIFIED BY PASSWORD '*9676FBC3B6BA6FB6672DA4AD25939828D969EA19' |
+-------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

```

### 2、授权局域网内主机远程连接数据库：
#### 2.1 百分号(%)匹配法：
```ruby
MariaDB [(none)]> grant all on *.* to babyshen@'172.30.105.%' identified by 'wangzongyu';

```
#### 2.2 子网掩码匹配法：
```ruby
MariaDB [(none)]> grant all on *.* to babyshen@'172.30.105.0/255.255.255.0' identified by 'wangzongyu';

```

### 3、使用revoke收回权限：
```ruby
MariaDB [(none)]> grant all on *.* to 'babyshen'@'localhost' identified by 'babypass';
MariaDB [(none)]> show grants for babyshen@localhost;
+--------------------------------------------------------------------------------------------------------------------------+
| Grants for babyshen@localhost                                                                                            |
+--------------------------------------------------------------------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'babyshen'@'localhost' IDENTIFIED BY PASSWORD '*8F6A81A195FBFAFAF3C30295738E1E29B4E7B6B1' |
+--------------------------------------------------------------------------------------------------------------------------+

MariaDB [(none)]> REVOKE INSERT ON *.* FROM 'babyshen'@'localhost';
MariaDB [(none)]> show grants for babyshen@localhost\G;
*************************** 1. row ***************************
Grants for babyshen@localhost: GRANT SELECT, UPDATE, DELETE, CREATE, DROP, RELOAD, SHUTDOWN, PROCESS, FILE, REFERENCES, INDEX, ALTER, SHOW DATABASES, SUPER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER, CREATE TABLESPACE ON *.* TO 'babyshen'@'localhost' IDENTIFIED BY PASSWORD '*8F6A81A195FBFAFAF3C30295738E1E29B4E7B6B1'

```
```ruby
[root@mysql ~]# mysql -u system -pwangzongyu -e "show grants for babyshen@localhost;" | grep -i grant
Grants for babyshen@localhost
GRANT SELECT, UPDATE, DELETE, CREATE, DROP, RELOAD, SHUTDOWN, PROCESS, FILE, REFERENCES, INDEX, ALTER, SHOW DATABASES, SUPER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER, CREATE TABLESPACE ON *.* TO 'babyshen'@'localhost' IDENTIFIED BY PASSWORD '*8F6A81A195FBFAFAF3C30295738E1E29B4E7B6B1'
```
- 查看mysql的所有权限
```ruby
mysql -u system -pwangzongyu -e "show grants for babyshen@localhost;" | grep -i grant | tail -1 | tr ',' '\n' >> mysql_grant.txt

 SELECT
 INSERT
 UPDATE
 DELETE
 CREATE
 DROP
 RELOAD
 SHUTDOWN
 PROCESS
 FILE
 REFERENCES
 INDEX
 ALTER
 SHOW DATABASES
 SUPER
 CREATE TEMPORARY TABLES
 LOCK TABLES
 EXECUTE
 REPLICATION SLAVE
 REPLICATION CLIENT
 CREATE VIEW
 SHOW VIEW
 CREATE ROUTINE
 ALTER ROUTINE
 CREATE USER
 EVENT
 TRIGGER
```
常规情况下授权select、insert、update、delete4个权限即可，有的开源软件，如BBS，还需要create、drop等比较危险的权限