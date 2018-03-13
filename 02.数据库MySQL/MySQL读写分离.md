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


## PHP程序实现读写分离：
### 1、通过判断SQL语句：
&emsp;&emsp;这种情况是开发和项目经理或者部门主管，拿到运维部门给好的读写数据库连接地址，提前定义好数据库的操作类程序文件，然后编写开发文档，让所有开发人员，统一调用这个类来执行SQL语句；    
&emsp;&emsp;封装的内部通过SQL语句里的select关键字来判断，是读取数据还是更新写入，来区分连接读取库还是写入库； `(update users set select='wangzy' where id=1;)(select * from users;)`

### 2、通过调用方法实现读写分离：
&emsp;&emsp;拿到运维部门给的读写库地址，项目经理或主管封装好数据库操作类程序文件，然后编写文档，告知开发人员或开会定义，写入数据调用操作方法和读取数据调取方法，从而实现数据读写分离功能；


## Amoeba实现读写分离
&emsp;&emsp;这个软件致力于MySQL的分布式数据库前端代理层，它主要在应用层访问MySQL的 时候充当SQL路由功能，专注于分布式数据库代理层（Database Proxy）开发。座落与 Client、DB Server(s)之间,对客户端透明。具有负载均衡、高可用性、SQL 过滤、读写分离、可路由相关的到目标数据库、可并发请求多台数据库合并结果。 通过Amoeba你能够完成多数据源的高可用、负载均衡、数据切片的功能，目前Amoeba已在很多 企业的生产线上面使用。

服务器角色 | 系统版本 | IP地址 | 备注 | 
|:- | :-: | :-: | :- | 
master主数据库 | CentOS7 | 172.30.105.104 | MySQL的写操作|
slave从数据库| CentOS7| 172.30.105.105 | MySQL的读操作|
Amoeba服务器| CentOS7| 172.30.105.106 | Amoeba服务器，无需安装MySQL|
测试Client端 |CentOS6 |172.30.105.123 | 连接Amoeba测试 |

### 1、配置两台服务器的主从同步：
[主从同步配置参考文档](https://github.com/ZongYuWang/Operation/blob/master/02.%E6%95%B0%E6%8D%AE%E5%BA%93MySQL/MySQL%E4%B8%BB%E4%BB%8E%E5%90%8C%E6%AD%A5.md)


### 2、基本环境安装：
#### 2.1 安装JDK：
[JDK下载路径](http://www.oracle.com/technetwork/java/javase/downloads/index.html)    
`目前Amoeba经验证在JavaTM SE 1.5和Java SE 1.6能正常运行`


```ruby
[root@amoeba ~]# rpm -ivh jdk-7u79-linux-x64.rpm 

```

```ruby
[root@amoeba ~]# java -version
java version "1.7.0_79"
Java(TM) SE Runtime Environment (build 1.7.0_79-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode)

```
```ruby
[root@amoeba ~]# ln -s /usr/java/jdk1.7.0_79/ /usr/java/jdk1.7
[root@amoeba ~]# vim /etc/profile
export JAVA_HOME=/usr/java/jdk1.7
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
[root@amoeba ~]# source /etc/profile

```
### 3、主从服务器上分别配置一个授权用户：
##### master主服务器(用于写操作)：
```ruby
mysql> grant select,insert,update,delete on *.* to amuser@'172.30.105.%' identified by 'ampasswd';
mysql> flush privileges;

```
```ruby
mysql> show grants for amuser@'172.30.105.%';
+-------------------------------------------------------------------------------------------------------------------------------------------+
| Grants for amuser@172.30.105.%                                                                                                            |
+-------------------------------------------------------------------------------------------------------------------------------------------+
| GRANT SELECT, INSERT, UPDATE, DELETE ON *.* TO 'amuser'@'172.30.105.%' IDENTIFIED BY PASSWORD '*82CA9FCF5A771A4DD4DF92BF30289F44A21095E1' |
+-------------------------------------------------------------------------------------------------------------------------------------------+

```
&emsp;&emsp;如果开起了主从同步功能，这时当主服务器创建一个用户后会自动同步到从服务器，所以我们要在从服务器上收回主服务器创建的用户的部分权限

##### slave从服务器(用于读操作)：
```ruby
mysql> REVOKE INSERT,UPDATE,DELETE ON *.* FROM 'amuser'@'172.30.105.%';

mysql> show grants for amuser@'172.30.105.%';
+-------------------------------------------------------------------------------------------------------------------+
| Grants for amuser@172.30.105.%                                                                                    |
+-------------------------------------------------------------------------------------------------------------------+
| GRANT SELECT ON *.* TO 'amuser'@'172.30.105.%' IDENTIFIED BY PASSWORD '*82CA9FCF5A771A4DD4DF92BF30289F44A21095E1' |
+-------------------------------------------------------------------------------------------------------------------+

```

### 4、安装和配置Amoeba：
[amoeba下载地址](http://sourceforge.net/projects/amoeba/files/)

#### 4.1 安装Amoeba：
```ruby
[root@amoeba ~]# mv amoeba-mysql-3.0.5-RC-distribution.zip /usr/local/
[root@amoeba ~]# cd /usr/local/
[root@amoeba local]# unzip amoeba-mysql-3.0.5-RC-distribution.zip 
[root@amoeba local]# mv amoeba-mysql-3.0.5-RC amoeba

```

#### 4.2 配置Amoeba：
```ruby
[root@amoeba ~]# vim /usr/local/amoeba/conf/amoeba.xml 
```
```ruby
<?xml version="1.0" encoding="gbk"?>

<!DOCTYPE amoeba:configuration SYSTEM "amoeba.dtd">
<amoeba:configuration xmlns:amoeba="http://amoeba.meidusa.com/">

	<proxy>
	
		<!-- service class must implements com.meidusa.amoeba.service.Service -->
		<service name="Amoeba for Mysql" class="com.meidusa.amoeba.mysql.server.MySQLService">
			<!-- port -->
			<property name="port">8066</property>
			
			<!-- bind ipAddress -->
		 
			<property name="ipAddress">172.30.105.106</property>
		      // 这个IP是对外提供的IP，比如WEB程序要连接这个IP
			
			<property name="connectionFactory">
				<bean class="com.meidusa.amoeba.mysql.net.MysqlClientConnectionFactory">
					<property name="sendBufferSize">128</property>
					<property name="receiveBufferSize">64</property>
				</bean>
			</property>
			
			<property name="authenticateProvider">
				<bean class="com.meidusa.amoeba.mysql.server.MysqlClientAuthenticator">
					
					<property name="user">root</property>		
					<property name="password">wangzongyu</property>
                    // amoeba对外提供的账号和密码，账号和密码可以随意设置
					
					<property name="filter">
						<bean class="com.meidusa.toolkit.net.authenticate.server.IPAccessController">
							<property name="ipFile">${amoeba.home}/conf/access_list.conf</property>
						</bean>
					</property>
				</bean>
			</property>
			
		</service>
		
		<runtime class="com.meidusa.amoeba.mysql.context.MysqlRuntimeContext">
			
			<!-- proxy server client process thread size -->
			<property name="executeThreadSize">128</property>
			
			<!-- per connection cache prepared statement size  -->
			<property name="statementCacheSize">500</property>
			
			<!-- default charset -->
			<property name="serverCharset">utf8</property>
			
			<!-- query timeout( default: 60 second , TimeUnit:second) -->
			<property name="queryTimeout">60</property>
		</runtime>
		
	</proxy>
	
	<!-- 
		Each ConnectionManager will start as thread
		manager responsible for the Connection IO read , Death Detection
	-->
	<connectionManagerList>
		<connectionManager name="defaultManager" class="com.meidusa.toolkit.net.MultiConnectionManagerWrapper">
			<property name="subManagerClassName">com.meidusa.toolkit.net.AuthingableConnectionManager</property>
		</connectionManager>
	</connectionManagerList>
	
		<!-- default using file loader -->
	<dbServerLoader class="com.meidusa.amoeba.context.DBServerConfigFileLoader">
		<property name="configFile">${amoeba.home}/conf/dbServers.xml</property>
	</dbServerLoader>
	
	<queryRouter class="com.meidusa.amoeba.mysql.parser.MysqlQueryRouter">
		<property name="ruleLoader">
			<bean class="com.meidusa.amoeba.route.TableRuleFileLoader">
				<property name="ruleFile">${amoeba.home}/conf/rule.xml</property>
				<property name="functionFile">${amoeba.home}/conf/ruleFunctionMap.xml</property>
			</bean>
		</property>
		<property name="sqlFunctionFile">${amoeba.home}/conf/functionMap.xml</property>
		<property name="LRUMapSize">1500</property>
		<property name="defaultPool">master</property>
		
		
		<property name="writePool">master</property>
		<property name="readPool">slave</property>
		
		// 定义写操作的池的名字(master)和读操作的池的名字(slave)，后面的配置文件要用到

		<property name="needParse">true</property>
	</queryRouter>
</amoeba:configuration>
                          
```

```ruby
需要提前创建一个amdb数据库，下面的配置文件中会使用
mysql> create database amdb;

// 当然这个库也会同步到从库中
```

```ruby
[root@amoeba ~]# vim /usr/local/amoeba/conf/dbServers.xml 
```
```ruby
<?xml version="1.0" encoding="gbk"?>

<!DOCTYPE amoeba:dbServers SYSTEM "dbserver.dtd">
<amoeba:dbServers xmlns:amoeba="http://amoeba.meidusa.com/">

		<!-- 
			Each dbServer needs to be configured into a Pool,
			If you need to configure multiple dbServer with load balancing that can be simplified by the following configuration:
			 add attribute with name virtual = "true" in dbServer, but the configuration does not allow the element with name factoryConfig
			 such as 'multiPool' dbServer   
		-->
		
	<dbServer name="abstractServer" abstractive="true">
		<factoryConfig class="com.meidusa.amoeba.mysql.net.MysqlServerConnectionFactory">
			<property name="connectionManager">${defaultManager}</property>
			<property name="sendBufferSize">64</property>
			<property name="receiveBufferSize">128</property>
				
			<!-- mysql port -->
			<property name="port">3306</property>
			
			<!-- mysql schema -->
			<property name="schema">amdb</property>
			// 这个库时提前创建好，存在的一个库，默认是test库
			
			<!-- mysql user -->
			<property name="user">amuser</property>
			
			<property name="password">ampasswd</property>
		</factoryConfig>

		<poolConfig class="com.meidusa.toolkit.common.poolable.PoolableObjectPool">
			<property name="maxActive">500</property>
			<property name="maxIdle">500</property>
			<property name="minIdle">1</property>
			<property name="minEvictableIdleTimeMillis">600000</property>
			<property name="timeBetweenEvictionRunsMillis">600000</property>
			<property name="testOnBorrow">true</property>
			<property name="testOnReturn">true</property>
			<property name="testWhileIdle">true</property>
		</poolConfig>
	</dbServer>

	<dbServer name="server1"  parent="abstractServer">
	// 父类是abstractServer，也就是使用abstractServer中定义的端口用户名和密码等
	
		<factoryConfig>
			<!-- mysql ip -->
			<property name="ipAddress">172.30.105.104</property>
		</factoryConfig>
	</dbServer>
	
	// 配置真实的mysqlserver
	
	
	<dbServer name="server2"  parent="abstractServer">
		<factoryConfig>
			<!-- mysql ip -->
			<property name="ipAddress">172.30.105.105</property>
		</factoryConfig>
	</dbServer>
	
	<dbServer name="master" virtual="true">
		<poolConfig class="com.meidusa.amoeba.server.MultipleServerPool">
			<!-- Load balancing strategy: 1=ROUNDROBIN , 2=WEIGHTBASED , 3=HA-->
			<property name="loadbalance">1</property>
			
			<!-- Separated by commas,such as: server1,server2,server1 -->
			<property name="poolNames">server1</property>
		</poolConfig>
	</dbServer>
	
	// amoeba.xml配置文件中定义的master是写的操作，定义master下面的server1是写数据库

        <dbServer name="slave" virtual="true">
                <poolConfig class="com.meidusa.amoeba.server.MultipleServerPool">
                        <!-- Load balancing strategy: 1=ROUNDROBIN , 2=WEIGHTBASED , 3=HA-->
                        <property name="loadbalance">1</property>

                        <!-- Separated by commas,such as: server1,server2,server1 -->
                        <property name="poolNames">server2</property>
                </poolConfig>
        </dbServer>

       // amoeba.xml配置文件中定义的slave是读的操作，定义slave下面的server2是读数据库

		
</amoeba:dbServers>

```

### 5、启动和关闭Amoeba：
#### 5.1 启动Amoeba:
```ruby
[root@amoeba ~]# /usr/local/amoeba/bin/launcher
 2018-03-08 04:00:33 [INFO] Project Name=Amoeba-MySQL, PID=39291 , starting...
log4j:WARN log4j config load completed from file:/usr/local/amoeba/conf/log4j.xml
2018-03-08 04:00:33,969 INFO  context.MysqlRuntimeContext - Amoeba for Mysql current versoin=5.1.45-mysql-amoeba-proxy-3.0.4-BETA
log4j:WARN ip access config load completed from file:/usr/local/amoeba/conf/access_list.conf
2018-03-08 04:00:34,298 INFO  net.ServerableConnectionManager - Server listening on /172.30.105.106:8066.
 2018-03-08 04:05:25 [INFO] Project Name=Amoeba-MySQL, PID=39598 , starting...
log4j:WARN log4j config load completed from file:/usr/local/amoeba/conf/log4j.xml
2018-03-08 04:05:25,990 INFO  context.MysqlRuntimeContext - Amoeba for Mysql current versoin=5.1.45-mysql-amoeba-proxy-3.0.4-BETA
log4j:WARN ip access config load completed from file:/usr/local/amoeba/conf/access_list.conf
2018-03-08 04:05:26,274 INFO  net.ServerableConnectionManager - Server listening on /172.30.105.106:8066.


 2018-03-08 04:21:02 [INFO] Project Name=Amoeba-MySQL, PID=40407 , starting...
log4j:WARN log4j config load completed from file:/usr/local/amoeba/conf/log4j.xml
2018-03-08 04:21:02,941 INFO  context.MysqlRuntimeContext - Amoeba for Mysql current versoin=5.1.45-mysql-amoeba-proxy-3.0.4-BETA
log4j:WARN ip access config load completed from file:/usr/local/amoeba/conf/access_list.conf
2018-03-08 04:21:03,219 INFO  net.ServerableConnectionManager - Server listening on /172.30.105.106:8066.
```
#### 5.1 关闭Amoeba:
```ruby
[root@amoeba ~]# /usr/local/amoeba/bin/shutdown 
```


#### FAQ:
```ruby
[root@amoeba ~]# /usr/local/amoeba/bin/launcher &
[1] 38856
[root@amoeba ~]# 
The stack size specified is too small, Specify at least 228k
Error: Could not create the Java Virtual Machine.
Error: A fatal exception has occurred. Program will exit.

[root@amoeba ~]# vim /usr/local/amoeba/jvm.properties
JVM_OPTIONS="-server -Xms1024m -Xmx1024m -Xss256k -XX:PermSize=16m -XX:MaxPermSize=96m"

```

### 6、测试连接：
#### 6.1 client连接Amoeba服务器：

```ruby
[root@client ~]# mysql -uroot -pwangzongyu -h172.30.105.106 -P8066
Warning: Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 915484425
Server version: 5.1.45-mysql-amoeba-proxy-3.0.4-BETA Source distribution

// 看此处的连接信息是5.1.45-mysql-amoeba-proxy-3.0.4-BETA

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```
`此处的password是连接Amoeba的密码，也就是在amoeba.xml配置文件中配置的密码，与mysql密码不同`

#### 6.2 测试读写分离效果：
##### master主数据库：
```ruby
[root@master ~]# mysql -u root -pwangzongyu

mysql> use amdb
mysql> create table amtb (id int(10) ,name varchar(10));

```

##### slave从数据库：
`然后停止从数据库的主从复制(便于观察结果)`

```ruby
[root@slave ~]# mysql -uroot -pwangzongyu
mysql> slave stop;

```

##### master主数据库上插入数据：
```ruby
mysql> insert into amtb values('1','wangzongyu');
```

##### slave从数据库上插入数据：
```ruby
mysql> use amdb;
mysql> insert into amtb values('2','baby');
mysql> insert into amtb values('3','shen');

```

##### 登陆到Amoeba服务器,进行读写分离的查看:
```ruby
[root@client ~]# mysql -uroot -pwangzongyu -h172.30.105.106 -P8066
mysql> use amdb;
mysql> select * from amtb;
+------+------+
| id   | name |
+------+------+
|    2 | baby |
|    3 | shen |
+------+------+

// select操作(读操作)只能查看到从数据库上的信息
```

##### 登陆到master主数据库查看：
`在主数据库中使用select读操作可以查看到主数据库上面的数据`
```ruby
mysql> use amdb;
mysql> select * from amtb;
+------+------------+
| id   | name       |
+------+------------+
|    1 | wangzongyu |
+------+------------+
1 row in set (0.00 sec)

```

##### 登陆Amoeba服务器插入一条数据，查看是不是插入到主服务器中：
```ruby
mysql> insert into amtb values('4','babyshen');

```
查看master主数据库：
```ruby
mysql> select * from amtb;
+------+------------+
| id   | name       |
+------+------------+
|    1 | wangzongyu |
|    4 | babyshen   |
+------+------------+

// 真的插入到了主服务器中
```
查看slave从数据库：
```ruby
mysql> select * from amtb;
+------+------+
| id   | name |
+------+------+
|    2 | baby |
|    3 | shen |
+------+------+

// 从服务器中没有新插入的数据
```

&emsp;&emsp;验证通过，使用Amoeba对Mysql读写分离成功。若此时开启从数据库主从复制(不要忘记开启主从复制)，则可以进行Mysql集群和负载均衡


## MySQL-Proxy实现读写分离
&emsp;&emsp;mysql-proxy是一个通过网络利用MySQL的网络协议，并且提供一个或多个MySQL服务器与一个或多个MySQL客户端相互沟通的应用程序，因为MySQL Proxy使用MySQL网络协议，所以它兼容任何MySQL客户端并且无需修改；     
&emsp;&emsp;除了基本的传递，MySQL Proxy可以在查询队列发送到服务器之前插入一些查询请求，也可以在服务器的应答中将对应的应答删除，这个功能可以使得管理员可以对每个查询进行跟踪并获取报告，例如，检测其执行时间或其他调试信息，并分别记录结果，同时还能将正确的应答返回给客户端；      
&emsp;&emsp;虽然mysql proxy功能强大，但目前仍然处于alpha版本，存在许多问题，不建议生产环境使用；    

![](https://github.com/ZongYuWang/image/blob/master/MySQL/MySQL-Proxy1.png) 



服务器角色 | 系统版本 | IP地址 | 备注 | 
|:- | :-: | :-: | :- | 
master主数据库 | CentOS7 | 172.30.105.104 | MySQL的写操作|
slave从数据库| CentOS7| 172.30.105.105 | MySQL的读操作|
MySQL-Proxy服务器| CentOS7| 172.30.105.106 | MySQL-Proxy服务器，无需安装MySQL|
测试Client端1 |CentOS6 |172.30.105.121 | 连接MySQL-Proxy服务器测试 |
测试Client端2 |CentOS6 |172.30.105.122 | 连接MySQL-Proxy服务器测试 |
测试Client端3 |CentOS6 |172.30.105.123 | 连接MySQL-Proxy服务器测试 |


### 1、配置两台服务器的主从同步：
[主从同步配置参考文档](https://github.com/ZongYuWang/Operation/blob/master/02.%E6%95%B0%E6%8D%AE%E5%BA%93MySQL/MySQL%E4%B8%BB%E4%BB%8E%E5%90%8C%E6%AD%A5.md)

### 2、MySQL主从服务器创建mysql-proxy的链接账号：

`主从服务器上分别配置一个授权用户`
##### master主服务器(写操作)：
```ruby
mysql> grant select,insert,update,delete on *.* to userproxy@'172.30.105.%' identified by 'pwdproxy';
mysql> flush privileges;

```
##### slave从服务器(读操作)：
```ruby
mysql> REVOKE INSERT,UPDATE,DELETE ON *.* FROM 'userproxy'@'172.30.105.%';

// 将主服务器同步过来的账号权限收回INSERT,UPDATE,DELETE权限
```

### 3、基础环境安装：
#### 3.1 安装依赖包：
```ruby
[root@mysql-proxy ~]# yum install lua lua-devel libevent libevent-devel glib2 glib2-devel  libffi  libffi-devel zlib zlib-devel gcc

```
#### 3.2 编译安装glib软件包：
```ruby
[root@mysql-proxy ~]# mkdir -p /mysql-proxy/tools
[root@mysql-proxy ~]# cd /mysql-proxy/tools/
[root@mysql-proxy tools]# wget http://ftp.gnome.org/pub/gnome/sources/glib/2.42/glib-2.42.0.tar.xz

```
```ruby
[root@mysql-proxy tools]# tar xf glib-2.42.0.tar.xz 
[root@mysql-proxy tools]# cd glib-2.42.0
[root@mysql-proxy glib-2.42.0]# ./configure && make && make install
```

```ruby
[root@mysql-proxy ~]# mv /usr/lib64/libglib-2.0.so.0 /usr/lib64/libglib-2.0.so.0.bak
[root@mysql-proxy ~]# mv /usr/lib64/libglib-2.0.so /usr/lib64/libglib-2.0.so.bak
[root@mysql-proxy ~]# ln -s /usr/local/lib/libglib-2.0.so.0.4200.0 /usr/lib64/libglib-2.0.so.0
[root@mysql-proxy ~]# ln -s /usr/local/lib/libglib-2.0.so.0.4200.0 /usr/lib64/libglib-2.0.so

```

### 4、安装MySQL-Proxy：
#### 4.1 使用RPM安装mysql-proxy：
[mysql-proxy下载地址](https://pkgs.org/download/mysql-proxy)
```ruby
[root@mysql-proxy tools]# rpm -ivh mysql-proxy-0.8.5-2.el7.x86_64.rpm 
Preparing...                          ################################# [100%]
Updating / installing...
   1:mysql-proxy-0.8.5-2.el7          ################################# [100%]

```
```ruby
wget http://dev.mysql.com/get/Downloads/MySQL-Proxy/mysql-proxy-0.8.5-linux-glibc2.3-x86-64bit.tar.gz
tar xf mysql-proxy-0.8.5-linux-glibc2.3-x86-64bit.tar.gz
[root@mysql-proxy tools]# cd mysql-proxy-0.8.5-linux-glibc2.3-x86-64bit
[root@mysql-proxy mysql-proxy-0.8.5-linux-glibc2.3-x86-64bit]# cp share/doc/mysql-proxy/rw-splitting.lua /usr/lib64/mysql-proxy/lua/proxy/

// 下载编译mysql-proxy-0.8.5-linux-glibc2.3-x86-64bit.tar.gz包不需要编译，目的是需要里面的读写分离脚本(rw-splitting.lua)
```

#### 4.1 配置mysql-proxy：
```ruby
[root@mysql-proxy ~]# rpm -qc mysql-proxy
/etc/sysconfig/mysql-proxy
[root@mysql-proxy ~]# cp /etc/sysconfig/mysql-proxy /etc/sysconfig/mysql-proxy.back
```
```ruby
[root@mysql-proxy ~]# vim /etc/sysconfig/mysql-proxy 

# Options for mysql-proxy 
ADMIN_USER="userproxy"
ADMIN_PASSWORD="pwdproxy"
#ADMIN_ADDRESS=""
#ADMIN_LUA_SCRIPT="/usr/lib64/mysql-proxy/lua/proxy/rw-splitting.lua"
#PROXY_ADDRESS=""
PROXY_USER="mysql-proxy"
PROXY_OPTIONS="--daemon --log-level=info --log-use-syslog --plugins=proxy --plugins=admin --proxy-backend-addresses=172.30.105.104:3306 --proxy-read-only-backend-addresses=172.30.105.105:3306 --proxy-lua-script=/usr/lib64/mysql-proxy/lua/proxy/rw-splitting.lua"

// ADMIN的账号和密码就是上面mysql主从服务器中配置的授权账号和密码
```

### 5、启动MySQL-Proxy：
```ruby
[root@mysql-proxy sysconfig]# systemctl status mysql-proxy
● mysql-proxy.service - SYSV: mysql-proxy is a proxy daemon for mysql
   Loaded: loaded (/etc/rc.d/init.d/mysql-proxy; bad; vendor preset: disabled)
   Active: active (running) since Mon 2018-03-12 03:41:34 EDT; 11s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 47181 ExecStart=/etc/rc.d/init.d/mysql-proxy start (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/mysql-proxy.service
           └─47190 mysql-proxy --daemon --log-level=info --log-use-syslog --plugins=proxy --plugins=admin --proxy-...

Mar 12 03:41:34 amoeba systemd[1]: Starting SYSV: mysql-proxy is a proxy daemon for mysql...
Mar 12 03:41:34 amoeba mysql-proxy[47181]: /etc/rc.d/init.d/mysql-proxy: line 20: [: =: unary operator expected
Mar 12 03:41:34 amoeba mysql-proxy[47181]: Starting mysql-proxy: [  OK  ]
Mar 12 03:41:34 amoeba systemd[1]: Started SYSV: mysql-proxy is a proxy daemon for mysql.
Mar 12 03:41:34 amoeba mysql-proxy[47190]: 2018-03-12 03:41:34: (critical) plugin proxy 0.8.5 started
Mar 12 03:41:34 amoeba mysql-proxy[47190]: 2018-03-12 03:41:34: (critical) plugin admin 0.8.5 started
Mar 12 03:41:34 amoeba mysql-proxy[47190]: 2018-03-12 03:41:34: (message) proxy listening on port :4040
Mar 12 03:41:34 amoeba mysql-proxy[47190]: 2018-03-12 03:41:34: (message) added read/write backend: 172.30.10...:3306
Mar 12 03:41:34 amoeba mysql-proxy[47190]: 2018-03-12 03:41:34: (message) added read-only backend: 172.30.105...:3306
Mar 12 03:41:34 amoeba mysql-proxy[47190]: 2018-03-12 03:41:34: (message) admin-server listening on port :4041
Hint: Some lines were ellipsized, use -l to show in full.

```
```ruby
[root@mysql-proxy ~]# ps -ef | grep mysql
mysql-p+  47190      1  0 03:41 ?        00:00:00 mysql-proxy --daemon --log-level=info --log-use-syslog --plugins=proxy --plugins=admin --proxy-backend-addresses=172.30.105.104:3306 --proxy-read-only-backend-addresses=172.30.105.105:3306 --proxy-lua-script=/usr/local/mysql-proxy/share/doc/mysql-proxy/rw-splitting.lua --pid-file=/var/run/mysql-proxy.pid --user=mysql-proxy --admin-username=admin --admin-lua-script=/usr/lib64/mysql-proxy/lua/admin.lua --admin-password=admin

```

```ruby
[root@mysql-proxy ~]# netstat -tlnp | grep mysql-proxy
tcp        0      0 0.0.0.0:4040            0.0.0.0:*               LISTEN      47190/mysql-proxy   
tcp        0      0 0.0.0.0:4041            0.0.0.0:*               LISTEN      47190/mysql-proxy 

// 4041是管理端口
```

### 6、测试：
#### 6.1 Client端连接MySQL-Proxy服务器测试：
```ruby
[root@client ~]# mysql -u userproxy -h172.30.105.106 -P4040 -ppwdproxy
Warning: Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 36
Server version: 5.5.20-log Source distribution

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 

```

#### 6.2 读操作测试：
##### master主服务器：
```ruby
mysql> create database myproxy;
mysql> use myproxy;
mysql> create table mytb (id int(10) ,name varchar(10));

```

##### master从服务器：
```ruby
mysql> slave stop;

// 将上面主服务器上创建的库和表同步到从服务器之后，然后关闭主从同步
```

`关闭主从同步为了观察读写分离效果，实际生产环境中不要关闭，实验测试完成还需要开启主从同步`
##### master主服务器：
```ruby
mysql> insert into mytb values('1','myproxy1');

// 在主服务器上插入一条数据
```

##### slave从服务器：
```ruby
mysql> use myproxy;
mysql> insert into mytb values('2','myproxy2');
mysql> insert into mytb values('3','myproxy3');

// 在从服务器上插入两条数据
```
```ruby
mysql> select * from mytb;
+------+----------+
| id   | name     |
+------+----------+
|    1 | myproxy1 |
+------+----------+
2 rows in set (0.00 sec)

```

##### 客户端执行查询操作：
```ruby
mysql> use myproxy;
Database changed
mysql> select * from mytb;
+------+----------+
| id   | name     |
+------+----------+
|    1 | myproxy1 |
+------+----------+
1 row in set (0.01 sec)

// mysql-proxy代理会检测客户端连接，当连接没有超过min_idle_connections预设值时，不会进行读写分离，即查询操作会发生到主库上;

```
```ruby
if not proxy.global.config.rwsplit then
        proxy.global.config.rwsplit = {
                min_idle_connections = 1,   
                // 修改最小连接数为1
                max_idle_connections = 2,
                // 修改最大连接数为2

                is_debug = false
        }
end

// 经过测试，当2个客户端连接时，查询操作还是查询的主库，当第三个之后的客户端查询时，查询从库；
```
```ruby
client3:

mysql> select * from mytb;
+------+----------+
| id   | name     |
+------+----------+
|    2 | myproxy2 |
|    3 | myproxy3 |
+------+----------+

```

#### 6.3 写操作测试：
##### 客户端执行写操作：
```ruby
mysql> insert into mytb values('10','myproxy10');

```
##### master主服务器：
```ruby
mysql> use myproxy;
mysql> select * from mytb;
+------+-----------+
| id   | name      |
+------+-----------+
|    1 | myproxy1  |
|    4 | myproxy4  |
|   10 | myproxy10 |
+------+-----------+

```
##### slave从服务器：
```ruby
mysql> use myproxy;
mysql> select * from mytb;
+------+----------+
| id   | name     |
+------+----------+
|    2 | myproxy2 |
|    3 | myproxy3 |
+------+----------+

// 写操作写入到了主服务器中，从服务器中并没有写入
```