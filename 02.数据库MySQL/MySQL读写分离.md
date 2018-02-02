## MySQL读写分离

### 1、MySQL实现读写分离方案：
`通过MySQL的主从服务器架构实现读写分离可实现分担网站压力`
#### 1.1 通过程序实现读写分离：
PHP和Java程序都可以通过设置多个连接文件轻松的实现对数据库的读写分离，即当select时，就去连接读库的链接文件，当update、insert、delete时就连接写库的链接文件；
#### 1.2 通过代理软件实现读写分离：
MySQL-Proxy、Amoeba等代理软件也可以实现读写分离功能，代理软件目前存在缺陷所以不能应用于正式线上环境中，可用于测试环境；

#### 1.3 开发DBProxy
门户网站会使用分布式DBProxy(读写分离、Hash负载均衡、健康检查)

### 2、实现MySQL主从读写分离：

#### 2.1 方案一：主从使用相同的账号：
```ruby
方法一：主库的username用户同步到从库，然后再收回INSERT,UPDATE,DELETE权限

主库：
MariaDB [(none)]> GRANT SELECT,INSERT,UPDATE,DELETE on mydb.* to 'username'@'172.30.105.%' identified by 'userpassword';
// 特殊业务可能权限会略多，如果业务安全性要求不高，也可以all权限；
从库：
MariaDB [(none)]> REVOKE INSERT,UPDATE,DELETE ON mydb.* FROM 'username'@'localhost';
// REVOKE回收权限，也可以结合read-only参数共同做

方法二：如果不收回从库的INSERT,UPDATE,DELETE权限，设置read-only参数确保从库只读

```
#### 2.1 方案二：主从使用不同的账号：
```ruby
主库：
MariaDB [(none)]> GRANT SELECT,INSERT,UPDATE,DELETE on mydb.* to 'user_w'@'172.30.105.%' identified by 'user_w';
从库：
MariaDB [(none)]> GRANT SELECT on mydb.* to 'user_r'@'172.30.105.%' identified by 'user_r';

// 风险：也可以使用主库的user_w连接从库，还是需要设置read-only参数确保从库只读

```

#### 方案三：通过忽略授权表的方式防止数据写从库的方法：
一般在生产环境都会采用忽略授权表的方式同步，然后对从服务器上的用户授权select读权限，不同步mysql库，这样就保证主库和从库相同的用户可以授权不用的权限      
不同步mysql库，主从库分别进行如下授权：
```ruby
replicate-wild-ignore-table = mysql.%
replicate-wild-ignore-table = test.%
replicate-wild-ignore-table = information_schema.%
```
```ruby
主库：
MariaDB [(none)]> GRANT SELECT,INSERT,UPDATE,DELETE on mydb.* to 'username'@'172.30.105.%' identified by 'userpassword';
从库：
MariaDB [(none)]> GRANT SELECT on mydb.* to 'username'@'172.30.105.%' identified by 'userpassword';
// 缺陷：当一个从库想提升为主库的时候，连接用户权限的问题
// 解决办法：需要单独找一个从库专门负责准备接替主库
```
#### 方案四：通过read-only参数防止数据写从库的办法：
除了上面在从库仅做SELECT的授权外，还可以在slave服务器启动选项增加参数或者在my.cnf配置文件中加read-only参数来确保从库只读，当然授权用户和read-only参数二者同时操作效果更佳，这也是生产环境中使用的方案；     
read-only参数可以让slave服务器只允许来自slave服务器线程或具有Super权限的用户的更新，可以确保slave服务器不接受来自普通用户的更新，slave服务器启动选项增加--read-only也是同样功能。
`read-only参数对super权限的用户无效，不起作用(也就是授权不能指定super或all privileges权限)`
```ruby
[root@MySQL2-Slave ~]# vim /etc/my.cnf
[mysqld]
read-only
port            = 3306
socket          = /tmp/mysql.sock
// 需要重启服务参数才生效

登陆超级用户新建一个其他的用户wangzy：
MariaDB [MyDB]> grant select,insert,update,delete on *.* to 'wangzy'@'localhost' identified by 'wangzongyu';
MariaDB [MyDB]> flush privileges;

[root@MySQL2-Slave ~]# mysql -uwangzy -pwangzongyu
MariaDB [(none)]> use MyDB;
MariaDB [MyDB]> insert into MyTB(name) values('babyshen9');
ERROR 1290 (HY000): The MariaDB server is running with the --read-only option so it cannot execute this statement

// 可以看出设置了read-only参数之后，不能写入数据了

```