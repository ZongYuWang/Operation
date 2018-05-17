## CentOS6安装配置Zabbix2.4监控

&emsp;&emsp;自从Zabbix3.0需要PHP5.4.0或者更高的版本，因为在RHCL6中PHP的最新版本是5.3.3，所以因为PHP的版本问题，Zabbix前端在RHCL6中已经不再支持
&emsp;&emsp;在多数情况下，Zabbix-server和frontend(前端)都安装在同一个机器上，当从2.2版本升级到3.0时Zabbix-server将会升级数据库(而且也没有办法回滚数据库的更改)并且frontend将停止工作，所以很多用户将被迫使用第三方工具升级PHP，这就也是Zabbix-server在RHEL6中也被弃用的原因
&emsp;&emsp;如果你想使用Zabbix-frontend在RHEL6上而且使用了第三方工具升级了PHP，你还需要在/etc/yum.repos.d/zabbix.repo中开启zabbix-deprecated(set enabled=1)

|主机名 | IP地址 | 备注 
| - | :-: | :- 
| zabbix-NMS | 192.168.67.101| NMS端(zabbix-server、zabbix-web、zabbix-agent) 
| node1 | 192.168.67.102 | zabbix-agent   

`NMS：Network Monitor System `

### 1、Zabbix基础说明

#### 1.1 zabbix-agent说明：
zabbix有专用的agent的监控工具，可以监控主机以及网络设备；
#### 1.2 zabbix-database(存储)说明：
zabbix使用的是MySQL5.0+、PgSQL(postgreSQL)8.1+、Oracle10+、DB2 9.7+、SQLite3.3.5+存储
#### 1.3 zabbix架构中的组件：
- zabbix-server:C语言
- zabbix-agent：C语言
- zabbix-web(GUI)：用于实现zabbix设定和展示
- zabbix-proxy：分布式监控环境中的专用组件
- zabbix-database：MySQL、PgSQL(postgreSQL)、Oracle、DB2、SQLite

### 2、Zabbix硬件资源说明

| 监控规模 | 平台 | CPU/memory | DataBase | 监控节点数 
| - | :-: | :- 
| Small | CentOS | Virtual Appliance | MySQL InnoDB | 20
| Sedium | CentOS | 2 CPU cores/2GB | MySQL5.0+ InnoDB | 500
| Large | RedHat Enterprise Linux | 4 CPU cores/8GB | RAID10 MySQL InnoDB or PostgreSQL | >1000
| Very large | RedHat Enterprise Linux | 8 CPU cores/16GB | Fast RAID10 MySQL InnoDB or PostgreSQL | >10000

### 3、获取Zabbix软件包：
[zabbix软件包下载](https://repo.zabbix.com/zabbix)

&emsp;&emsp;例如在CentOS6-X86_64系统中安装zabbix，yum源默认是没有zabbix软件包的
&emsp;&emsp;可以直接下载响应的zabbix-release-xxx.noarch.rpm软件包安装，这样会在/etc/yum.repos.d下生成zabbix.repo源，这个源中的baseurl地址同时也指向了[zabbix相关软件包的yum源地址](http://repo.zabbix.com/zabbix/3.4/rhel/6/$basearch/)
&emsp;&emsp;也可以直接把所有的软件包下载至服务器中，根据需要选择相关的软件包

### 4、安装Zabbix-NMS端(监控端)

#### 4.1 安装Zabbix-database（数据库）：
`这里使用编译安装MySQL5.5版本` 

| Software | Version | Comments 
| - | :-: | :- 
| MySQL | 5.0.3 or later | Required if MySQL is used as Zabbix backend database. InnoDB engine is required.MariaDB also works with Zabbix.
| Oracle | 10g or later | Required if Oracle is used as Zabbix backend database.
| PostgreSQL | 8.1 or later | Required if PostgreSQL is used as Zabbix backend database.It is suggested to use at least PostgreSQL 8.3
| IBM DB2 | 9.7 or later | Required if IBM DB2 is used as Zabbix backend database.
| SQLite | 3.3.5 or later | SQLite is only supported with Zabbix proxies. Required if SQLite is used as Zabbix proxy database.

#### 4.2 安装Zabbix相关软件包：
`NMS端既是zabbix-server端，又是zabbix-web端，又是zabbix-agent端(自己监控自己)` 
##### RPM方式安装：
```ruby
[root@zabbix-NMS zabbix]# rpm -ivh zabbix-release-2.4-1.el6.noarch.rpm
```
zabbix-server需要安装的组件：
```ruby
[root@zabbix-NMS zabbix]# yum install zabbix-2.4.8-1.el6.x86_64.rpm zabbix-server-2.4.8-1.el6.x86_64.rpm zabbix-server-mysql-2.4.8-1.el6.x86_64.rpm zabbix-get-2.4.8-1.el6.x86_64.rpm
```
zabbix-web需要安装的组件：
```ruby
[root@zabbix-NMS zabbix]# yum install -y zabbix-web-2.4.8-1.el6.noarch.rpm zabbix-web-mysql-2.4.8-1.el6.noarch.rpm  

```
```ruby
// 安装zabbix-web之后其实就是安装了一个HTTP和PHP
// 所以在部署之前不需要自己单独安装Apache和PHP环境

[root@zabbix-NMS ~]# ls /etc/httpd/
conf  conf.d  logs  modules  run
[root@zabbix-NMS ~]# ls /etc/httpd/conf.d/
php.conf  README  welcome.conf  zabbix.conf
```
```ruby
[root@zabbix-NMS ~]# php -v
PHP 5.3.3 (cli) (built: Mar 22 2017 12:27:09) 
```
zabbix-agent需要安装的组件：
```ruby
[root@zabbix-NMS zabbix]# yum install zabbix-agent-2.4.8-1.el6.x86_64.rpm zabbix-sender-2.4.8-1.el6.x86_64.rpm 

```
```ruby
[root@zabbix-NMS ~]# rpm -ql zabbix-server
/etc/init.d/zabbix-server
/etc/logrotate.d/zabbix-server
/etc/zabbix/zabbix_server.conf
/usr/lib/zabbix/alertscripts
/usr/lib/zabbix/externalscripts
/usr/share/man/man8/zabbix_server.8.gz
```
##### 编译方式安装：
```ruby
https://www.cnblogs.com/galengao/p/5756683.html
```

### 5、创建数据库用户并导入数据库表结构
#### 5.1 创建使用的数据库：
```ruby
mysql> CREATE DATABASE zabbix CHARACTER SET utf8;
mysql> GRANT ALL on zabbix.* TO 'zbuser'@'192.168.%.%' IDENTIFIED BY 'zbpass';
// 创建一个用户，让用户通过zabbix-web只能访问上面创建的数据局
mysql> GRANT ALL on zabbix.* TO 'zbuser'@'node1' IDENTIFIED BY 'zbpass';
mysql> FLUSH PRIVILEGES;

```
#### 5.2 导入数据库：
```ruby
[root@zabbix-NMS ~]# ls /usr/share/doc/zabbix-server-mysql-2.4.8/create/
data.sql  images.sql  schema.sql
[root@zabbix-NMS create]# mysql zabbix < schema.sql 
[root@zabbix-NMS create]# mysql zabbix < images.sql 
[root@zabbix-NMS create]# mysql zabbix < data.sql
```

### 6、配置Zabbix-NMS端(监控端)

#### 6.1 配置并启动zabbix_server：
```ruby
[root@zabbix-NMS ~]# vim /etc/zabbix/zabbix_server.conf

ListenPort=10051
LogFileSize=0  // 0表示不做日志滚动，1表示到1G之后自动日志滚动
DBHost=192.168.67.101  // 如果是在本机上，建议使用localhost
DBName=zabbix
DBUser=zbuser
DBPassword=zbpass
DBSocket=/tmp/mysql.sock
// 上面的DBHost为localhost时，那么这个DBsocket也要改成mysql的sock位置
// 现在不是使用的localhost，所以不需要修改，只要不是使用的localhost，这项改不改都不生效

```

```ruby
[root@zabbix-NMS ~]# service zabbix-server start
Starting Zabbix server:                                    [  OK  ]
[root@zabbix-NMS ~]# chkconfig --level 12345 zabbix-server on
```
#### 6.2 配置zabbix的php选项：
```ruby
[root@zabbix-NMS conf.d]# vim /etc/httpd/conf.d/zabbix.conf
<IfModule mod_php5.c>
        php_value max_execution_time 300
        php_value memory_limit 128M
        php_value post_max_size 16M
        php_value upload_max_filesize 2M
        php_value max_input_time 300
        php_value date.timezone Asia/Chongqing
</IfModule>

```
#### 6.3 启动zabbix-web：
```ruby
[root@zabbix-NMS conf]# service httpd restart
```
### 7、 Zabbix-web页面访问
```ruby
http://192.168.67.101/zabbix/

登陆用户名：admin
登陆用户名密码：zabbix
```
![](https://github.com/ZongYuWang/image/blob/master/Zabbix/Zabbix1.png)


### 8、zabbix-agent配置并启动
`zabbix也需要自己监控自己的运行状况`

```ruby
[root@zabbix-NMS ~]# vim /etc/zabbix/zabbix_agentd.conf

Server=127.0.0.1,192.168.67.101  // 这是授权，标示哪些Zabbix-Server可以来获取数据
ServerActive=127.0.0.1,192.168.67.101
// Zabbix-agent也可以将监控信息发送给Zabbix-server端，这里的IP地址指向的Zabbix-Server端
// 127.0.0.1不要去掉，Zabbix-server和Zabbix-agent在同一台机器上的时候不要去掉127.0.0.1
// 该机器单独作为zabbix-agent的时候可以去掉127.0.0.1
Hostname=Zabbix-NMS  // 主机名在所有的被监控端中必须唯一

```
```ruby
[root@zabbix-NMS ~]# service zabbix-agent start
Starting Zabbix agent:                                     [  OK  ]

```
![](https://github.com/ZongYuWang/image/blob/master/Zabbix/Zabbix2.png)

### 9、添加监控主机(node1)
```ruby
[root@node1 zabbix]# yum install zabbix-2.4.8-1.el6.x86_64.rpm zabbix-agent-2.4.8-1.el6.x86_64.rpm zabbix-sender-2.4.8-1.el6.x86_64.rpm 
```
```ruby
[root@node1 ~]# ls /etc/zabbix/
zabbix_agentd.conf  zabbix_agentd.d

[root@node1 ~]# vim /etc/zabbix/zabbix_agentd.conf
Server=192.168.67.101
ServerActive=192.168.67.101
Hostname=node1
```
```ruby
[root@node1 ~]# service zabbix-agent start
Starting Zabbix agent:                                     [  OK  ]
```

![](https://github.com/ZongYuWang/image/blob/master/Zabbix/Zabbix3.png)     
![](https://github.com/ZongYuWang/image/blob/master/Zabbix/Zabbix4.png)     
![](https://github.com/ZongYuWang/image/blob/master/Zabbix/Zabbix5.png)    