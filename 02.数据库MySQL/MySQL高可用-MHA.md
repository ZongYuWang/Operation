## MySQL高可用-MHA
[MHA官方配置文档](https://www.percona.com/blog/2016/09/02/mha-quickstart-guide/)    
&emsp;&emsp;  MHA(Master HA)是一款开源的MySQL的高可用程序，它为MySQL主从复制架构提供了automating master failover功能，MHA在监控到master节点故障时，会提供其中拥有最新数据的slave节点成为新的master节点，在此期间，MHA会通过于其它从节点获取额外信息来避免一致性方面的问题，MHA还提供了master节点的在线切换功能，即按需切换master/slave节点(也就是对主节点进行监控，可实现自动故障转移至其它从节点；通过提升某一从节点为新的主节点)；


MHA服务有两种角色，MHA Manager（管理节点）和MHA Node（数据节点）:
- MHA Manager：通常单独部署在一台独立机器上管理多个master/slave集群，每个master/slave集群称作一个application；
- MHA Node：运行在每台MySQL服务器上(master/slave/manager)，它通过监控具备解析和清理logs功能的脚本来加快故障转移；        
`Manager节点不能放在MySQL Master节点上，应该放在MySQL 的Slave节点上，如果放在Master上，Master机器宕机之后，那么MHA的Manager也就不能使用了`

### MHA的工作原理：
- 从宕机崩溃的master保存二进制日志事件（binlog events）;     
- 识别含有最新更新的slave；          
- 应用差异的中继日志（relay log）到其他的slave；      
- 应用从master保存的二进制日志事件（binlog events）；       
- 提升一个slave为新的master；       
- 使其他的slave连接新的master进行复制；      

### MHA-Manager节点组件：
       
- masterha_check_ssh:MHA依赖的SSH环境监测工具；
- masterha_check_repl:MySQL复制环境监测工具；
- masterha_manager:MHA服务主程序；
- masterha_check_status:MHA运行状态探测工具；
- masterha_master_monitor:MySQL master节点可用性监测工具；
- masterha_master_switch:master节点切换工具；
- masterha_conf_host:添加或删除配置的节点；
- masterha_stop:关闭MHA服务的工具；

### MHA-Node节点组件：
- save_binary_logs:保存和复制master的二进制日志；
- apply_diff_relay_logs:识别差异的中继日志事件并应用于其它salve；
- filter_mysqlbinlogs:去除不必要的ROLLBACK事件（MHA已不再使用这个工具）；
- purge_relay_logs:清除中继日志（不会阻塞SQL线程）；
##### 自定义扩展：
- secondary_check_script:通过多条网络路由检测master的可用性；
- master_ip_failover_script:自动切换时vip管理的脚本（非必须），如果使用keepalived，可以自己编写脚本完成对vip的管理，比如监控mysql，如果mysql异常，我们停止keepalived就行，这样vip就会自动漂移；
- shutdown_script:强制关闭master节点；
- init_conf_load_script:加载初始配置参数；
- master_ip_online_change_script:更在线切换时vip的管理，不是必须，同样可以可以自行编写简单的shell完成；
- power_manager :故障发生后关闭主机的脚本（非必须）；
- send_report :因故障切换后发送报警的脚本（非必须），可自行编写简单的shell完成；

### 1、环境准备：
操作系统：CentOS7    
数据库版本：Server version: 5.5.20
#### 1.1 服务器准备：
一个MHA复制集群中必须最少有三台数据库服务器，一主二从，即一台充当Master，一台充当备用Master，另外一台充当从库,其中把被Master的slave节点设置为半同步复制      

角色 | IP地址 | 主机名 | Server_ID | 类型 | 
|- | :-: | :-: | :-: | :-: | 
Manager | 172.30.105.103 | manager | | MHA管理节点 | 
MySQL-Master | 172.30.105.104 | master | 104 | 写操作|
MySQL-Slave | 172.30.105.105 | slave1 | 105 | 读操作(半同步节点)|
MySQL-Slave | 172.30.105.106 | slave2 | 106 | 读操作|

#### 1.2 配置主机名并安装net-tools：
```ruby
[root@localhost ~]# hostnamectl set-hostname Manager
// 这种配置方式永久生效

[root@localhost ~]# yum install net-tools
// 三台数据库服务器都要安装，配置VIP的时候使用ifconfig命令



```

### 2、配置主从同步：
[主从同步以及半同步文档](https://github.com/ZongYuWang/Operation/blob/master/02.%E6%95%B0%E6%8D%AE%E5%BA%93MySQL/MySQL%E4%B8%BB%E4%BB%8E%E5%90%8C%E6%AD%A5.md)
- 为了尽可能的减少主库硬件损坏宕机造成的数据丢失，因此在配置MHA的同时建议配置成MySQL 5.5的半同步复制；
- binlog-do-db 和 replicate-ignore-db 设置必须相同，因为MHA 在启动时候会检测过滤规则，如果过滤规则不同，MHA 不启动监控和故障转移；    

#### 2.1 MySQL的主服务器和从服务器都要开启binlog和realylog：
`也就是万一主库宕了，其他的从库可以升级为主库，老的主库可以充当从库`
```ruby
innodb_file_per_table = 1
skip_name_resolve = 1
log-bin = /mysqlbinlogs/mysql-bin
relay-log = /mysqlbinlogs/relay-bin
server-id = xxx


```
#### 2.2 MySQL从服务器设置只读模式：
```ruby
[root@MySQL2-Slave1 ~]# mysql -uroot -pwangzongyu -e 'set global read_only=1'
[root@MySQL2-Slave2 ~]# mysql -uroot -pwangzongyu -e 'set global read_only=1'

// 两台slave服务器设置read_only（从库对外提供读服务，只所以没有写进配置文件，是因为随时slave会提升为master）
// read_only参数对对super权限的用户无效(也就是授权不能指定super或all privileges权限)
```
#### 2.3 MySQL主服务器上创建监控用户信息：
```ruby
mysql> GRANT ALL ON *.* TO 'mhauser'@'172.30.105.%' IDENTIFIED BY 'wangzongyu';
mysql> FLUSH PRIVILEGES;

```
#### 2.4 MySQL主从同步账号信息：
```ruby
mysql> grant all on *.* to repuser@'172.30.105.%' identified by 'wangzongyu';
mysql> FLUSH PRIVILEGES;



```


### 3、配置每台主机的无秘钥登陆：
`配置每台主机都能无秘钥登陆到其他主机，需要互相都可以登陆，可以在任意主机上操作`    
一定要检查每台机器的SElinux，都配置正确但是selinux开启会影响无秘钥连接
```ruy

172.30.105.120  Manager       
172.30.105.121  MySQL1-Master 
172.30.105.122  MySQL2-Slave1 
172.30.105.123  MySQL2-Slave2

// 必须使用主机名，所以需要在/etc/hosts中解析，每个主机都需要配置
```
```ruby
[root@Manager ~]# ssh-keygen -t rsa -P ''
[root@manager ~]# ssh-keygen -t rsa -P ''
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:dhviTOqVR8nU0TBLp3BWNJIwBYWY5OPELb2C8wRRDFY root@manager
The key's randomart image is:
+---[RSA 2048]----+
|      +*Eo=*@*=  |
|     . ++o.B.B.. |
|      . * + +    |
|       = = o     |
|      o S B      |
|       X * o     |
|      . * o      |
|     . . .       |
|      .          |
+----[SHA256]-----+

```
```ruby
[root@Manager ~]# cat .ssh/id_rsa.pub >> .ssh/authorized_keys
[root@Manager ~]# chmod go= .ssh/authorized_keys 
[root@Manager ~]# ll .ssh/authorized_keys 
-rw-------. 1 root root 394 Jan 20 03:53 .ssh/authorized_keys

[root@Manager ~]# scp -p .ssh/id_rsa .ssh/authorized_keys master:/root/.ssh/
[root@Manager ~]# scp -p .ssh/id_rsa .ssh/authorized_keys slave1:/root/.ssh/
[root@Manager ~]# scp -p .ssh/id_rsa .ssh/authorized_keys slave2:/root/.ssh/
```
### 4、安装配置MHA-Manager节点：

#####  Manager-172.30.105.120服务器 : 
[MHA软件包下载](https://code.google.com/archive/p/mysql-master-ha/downloads)      
`MHA-Manager和MHA-Node的版本号不一定要完全匹配`
 
#### 4.1 安装依赖包：
`Manager节点需要安装Manager包和Node包`
```ruby
[root@manager ~]# rpm -ivh http://mirror.utexas.edu/epel/7Server/x86_64/Packages/e/epel-release-7-11.noarch.rpm
[root@manager ~]# rpm -qa|grep epel
epel-release-7-11.noarch
[root@manager ~]# rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
[root@manager ~]# yum install -y \
perl-DBD-MySQL \
perl-Config-Tiny \
perl-Log-Dispatch \
perl-Parallel-ForkManager \
perl-Time-HiRes \
perl-ExtUtils-CBuilder \
perl-ExtUtils-MakeMaker \
cpan
```
#### 4.2 安装mha4mysql-manager：
##### RPM方式安装(推荐使用这种安装方式)：
```ruby
[root@manager ~]# yum install -y mha4mysql-node-0.54-0.el6.noarch.rpm
[root@manager ~]# yum install mha4mysql-manager-0.55-0.el6.noarch.rpm 

// 注意安装顺序，要先安装node包再安装manager包
```

##### 编译方式安装：
`(这种安装方式在后续的部署中总是出现问题)`

```ruby

[root@manager ~]# tar xf mha4mysql-manager-0.54.tar.gz 
[root@manager ~]# cd mha4mysql-manager-0.54
[root@manager mha4mysql-manager-0.54]# perl Makefile.PL
[root@manager mha4mysql-manager-0.54]# make && make install
```
#### 4.3 配置MHA-Manager节点：
[app1.cnf下载地址](https://github.com/ZongYuWang/File/tree/master/File/MySQL/MySQL_MHA)       
[master_ip_failover下载地址](https://github.com/ZongYuWang/File/tree/master/File/MySQL/MySQL_MHA)      
```ruby
master_ip_failover文件需要修改如下部分：

my $vip = '172.30.105.203/24';
my $key = '1';
my $ssh_start_vip = "/sbin/ifconfig ens33:$key $vip";
my $ssh_stop_vip = "/sbin/ifconfig ens33:$key down";
```
[master_ip_online_change](https://github.com/ZongYuWang/File/tree/master/File/MySQL/MySQL_MHA)     
```ruby
master_ip_online_change文件需要修改如下部分：

my $vip = '172.30.105.203/24';  # Virtual IP 
my $key = "1";
my $ssh_start_vip = "/sbin/ifconfig ens33:$key $vip";
my $ssh_stop_vip = "/sbin/ifconfig ens33:$key down";
my $ssh_user = "root";
my $new_master_password='wangzongyu';
my $orig_master_password='wangzongyu';

```
[send_report](https://github.com/ZongYuWang/File/tree/master/File/MySQL/MySQL_MHA)       
`将这四个文件分别放到下面app1.cnf配置文件的指定目录中`     



```ruby
配置文件分为：
global配置：作用是为各application提供默认配置；
application配置：
```
```ruby
[root@manager ~]# mkdir /etc/masterha
[root@manager ~]# vim /etc/masterha/app1.cnf

[server default]
user=mhauser            
//也就是能登陆各mysql主机 并能管理mysql的用户，可以使用root

password=wangzongyu
manager_workdir=/data/masterha/app1                       
//设置manager的工作目录，这个目录会自动创建

manager_log=/data/masterha/manager.log                    
//设置manager的日志

remote_workdir=/data/masterha/app1                                                
//设置远端mysql在发生切换时binlog的保存位置

master_ip_failover_script= /usr/bin/master_ip_failover             
//设置自动failover时候的切换脚本

master_ip_online_change_script= /usr/bin/master_ip_online_change             
//设置手动切换时候的切换脚本

report_script=/usr/bin/send_report                             
//设置发生切换后发送的报警的脚本

secondary_check_script= /usr/bin/masterha_secondary_check -s slave1 -s master
shutdown_script=""                                     
//设置故障发生后关闭故障主机脚本（该脚本的主要作用是关闭主机放在发生脑裂,这里没有使用）

ssh_user=root                        
//设置ssh的登录用户名，不需要密码，以为已经做了无秘钥登陆；

repl_user=repluser                 
//设置复制环境中的复制用户名
repl_password=wangzongyu
ping_interval=1

[server1]
hostname=172.30.105.104
#ssh_port=22                
// 如果ssh使用的不是默认端口，则需要修改；

master_binlog_dir=/mysqlbinlogs       
//设置master 保存binlog的位置，以便MHA可以找到master的日志

[server2]
hostname=172.30.105.105
candidate_master=1                 
//设置为候选master，如果设置该参数以后，发生主从切换以后将会将此从库提升为主库，即使这个主库不是集群中事件最新的slave

check_repl_delay=0                 
//默认情况下如果一个slave落后master 100M的relay logs的话，MHA将不会选择该slave作为一个新的master，因为对于这个slave的恢复需要花费很长时间;
// 通过设置check_repl_delay=0,MHA触发切换在选择一个新的master的时候将会忽略复制延时，这个参数对于设置了candidate_master=1的主机非常有用，因为这个候选主在切换的过程中一定是新的master

master_binlog_dir=/mysqlbinlogs

[server3]
hostname=172.30.105.106
master_binlog_dir=/mysqlbinlogs
no_master=1              
// 该主机不允许变为MySQL Master节点；

```
```ruby
[root@manager ~]# chmod +x /usr/bin/master_ip_failover
[root@manager ~]# chmod +x /usr/bin/master_ip_online_change
[root@manager ~]# chmod +x /usr/bin/send_report
```


### 5、安装配置MHA-Master节点和MHA-Slave节点：
`MHA-Master节点只需要安装Node包`
##### Master-172.30.105.104：
```ruby
[root@master ~]# yum install mha4mysql-node-0.54-0.el6.noarch.rpm 
```

##### Slave1-172.30.105.105：
```ruby
[root@slave1 ~]# yum install -y mha4mysql-node-0.54-0.el6.noarch.rpm
```

##### Slave2-172.30.105.106：
```ruby
[root@slave2 ~]# yum install -y mha4mysql-node-0.54-0.el6.noarch.rpm

```
#### 5.1 两台Slave节点上设置relay log的清除方式:
```ruby
[root@slave1 ~]# mysql -uroot -pwangzongyu -e 'set global relay_log_purge=0'
```
```ruby
[root@slave2 ~]# mysql -uroot -pwangzongyu -e 'set global relay_log_purge=0'
```
- 在默认情况下，从服务器上的中继日志会在SQL线程执行完毕后被自动删除。但是在MHA环境中，这些中继日志在恢复其他从服务器时可能会被用到，因此需要禁用中继日志的自动删除功能。定期清除中继日志需要考虑到复制延时的问题。在ext3的文件系统下，删除大的文件需要一定的时间，会导致严重的复制延时。为了避免复制延时，需要暂时为中继日志创建硬链接，因为在linux系统中通过硬链接删除大文件速度会很快。（在mysql数据库中，删除大表时，通常也采用建立硬链接的方式）
- MHA节点中包含了pure_relay_logs命令工具，它可以为中继日志创建硬链接，执行SET GLOBAL relay_log_purge=1,等待几秒钟以便SQL线程切换到新的中继日志，再执行SET GLOBAL relay_log_purge=0
```ruby
pure_relay_logs脚本参数如下所示：
--user mysql               
//  用户名

--password mysql      
// 密码

--port                           
// 端口号

--workdir                    
// 指定创建relay log的硬链接的位置，默认是/var/tmp，由于系统不同分区创建硬链接文件会失败，故需要执行硬链接具体位置，成功执行脚本后，硬链接的中继日志文件被删除

--disable_relay_log_purge        
// 默认情况下，如果relay_log_purge=1，脚本会什么都不清理，自动退出;
// 当relay_log_purge=1的情况下会将relay_log_purge设置为0；清理relay log之后，最后将参数设置为OFF；
```
##### 两台Slave服务器中设置定时清理relay脚本：
```ruby
#!/bin/bash
user=root
passwd=123456
port=3306
log_dir='/data/masterha/log'
work_dir='/mysqlbinlogs'
purge='/usr/bin/purge_relay_logs'

if [ ! -d $log_dir ]
then
   mkdir $log_dir -p
fi

$purge --user=$user --password=$passwd --disable_relay_log_purge --port=$port --workdir=$work_dir >> $log_dir/purge_relay_logs.log 2>&1

[root@slave2 ~]# crontab -l
0 4 * * * /bin/bash /root/purge_relay_log.sh
```
```ruby
// 使用purge_relay_logs脚本删除中继日志不会阻塞SQL线程，下面手动执行一下看什么情况：

[root@slave2 ~]# find / -name relay-bin*
/mysqlbinlogs/relay-bin.000002
/mysqlbinlogs/relay-bin.index
/mysqlbinlogs/relay-bin.000001

```
```ruby
[root@slave2 ~]# purge_relay_logs --user=system --password=wangzongyu --host=localhost --port=3306 -disable_relay_log_purge --workdir=/mysqlbinlogs/
2018-02-08 05:48:59: purge_relay_logs script started.
 Found relay_log.info: /mysqldata/relay-log.info
 Removing hard linked relay log files relay-bin* under /mysqlbinlogs/.. done.
 Current relay log file: /mysqlbinlogs/relay-bin.000002
 Archiving unused relay log files (up to /mysqlbinlogs/relay-bin.000001) ...
 Old relay logs not found. No need to archive right now.
 Executing SET GLOBAL relay_log_purge=1; FLUSH LOGS; sleeping a few seconds so that SQL thread can delete older relay log files (if it keeps up); SET GLOBAL relay_log_purge=0; .. ok.
 Removing hard linked relay log files relay-bin* under /mysqlbinlogs/.. done.
2018-02-08 05:49:03: All relay log purging operations succeeded.

[root@slave2 ~]# find / -name relay-bin*
// 就被清空了
```


### 6、检查MHA集群的SSH配置：
检测各节点间ssh互相通信配置是否正常:
```ruby
[root@manager ~]# masterha_check_ssh --conf=/etc/masterha/app1.cnf 
Sun Feb 11 22:26:10 2018 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Sun Feb 11 22:26:10 2018 - [info] Reading application default configurations from /etc/masterha/app1.cnf..
Sun Feb 11 22:26:10 2018 - [info] Reading server configurations from /etc/masterha/app1.cnf..
Sun Feb 11 22:26:10 2018 - [info] Starting SSH connection tests..
Sun Feb 11 22:26:13 2018 - [debug] 
Sun Feb 11 22:26:11 2018 - [debug]  Connecting via SSH from root@172.30.105.106(172.30.105.106:22) to root@172.30.105.104(172.30.105.104:22)..
Sun Feb 11 22:26:12 2018 - [debug]   ok.
Sun Feb 11 22:26:12 2018 - [debug]  Connecting via SSH from root@172.30.105.106(172.30.105.106:22) to root@172.30.105.105(172.30.105.105:22)..
Sun Feb 11 22:26:13 2018 - [debug]   ok.
Sun Feb 11 22:26:13 2018 - [debug] 
Sun Feb 11 22:26:11 2018 - [debug]  Connecting via SSH from root@172.30.105.105(172.30.105.105:22) to root@172.30.105.104(172.30.105.104:22)..
Sun Feb 11 22:26:12 2018 - [debug]   ok.
Sun Feb 11 22:26:12 2018 - [debug]  Connecting via SSH from root@172.30.105.105(172.30.105.105:22) to root@172.30.105.106(172.30.105.106:22)..
Sun Feb 11 22:26:12 2018 - [debug]   ok.
Sun Feb 11 22:26:13 2018 - [debug] 
Sun Feb 11 22:26:10 2018 - [debug]  Connecting via SSH from root@172.30.105.104(172.30.105.104:22) to root@172.30.105.105(172.30.105.105:22)..
Sun Feb 11 22:26:11 2018 - [debug]   ok.
Sun Feb 11 22:26:11 2018 - [debug]  Connecting via SSH from root@172.30.105.104(172.30.105.104:22) to root@172.30.105.106(172.30.105.106:22)..
Sun Feb 11 22:26:12 2018 - [debug]   ok.
Sun Feb 11 22:26:13 2018 - [info] All SSH connection tests passed successfully.

```
### 7、检查MHA集群的主从同步配置：
```ruby
[root@manager ~]# masterha_check_repl --conf=/etc/masterha/app1.cnf
```

- FAQ1：
```ruby
Can't exec "mysqlbinlog": No such file or directory at /usr/share/perl5/vendor_perl/MHA/BinlogManager.pm line 99.
mysqlbinlog version not found!

解决办法：
[root@master ~]# ln -s /usr/local/mysql/bin/mysqlbinlog /usr/bin/mysqlbinlog
[root@slave1 ~]# ln -s /usr/local/mysql/bin/mysqlbinlog /usr/bin/mysqlbinlog
[root@slave2 ~]# ln -s /usr/local/mysql/bin/mysqlbinlog /usr/bin/mysqlbinlog
```
- FAQ2：
```ruby
Creating directory /data/masterha/app1.. done.
  Checking slave recovery environment settings..
    Opening /mydata/data/relay-log.info ... ok.
    Relay log found at /ourdata/binlog, up to relay-bin.000002
    Temporary relay log file is /ourdata/binlog/relay-bin.000002
    Testing mysql connection and privileges..sh: mysql: command not found
mysql command failed with rc 127:0!
 at /usr/bin/apply_diff_relay_logs line 367
	main::check() called at /usr/bin/apply_diff_relay_logs line 486
	eval {...} called at /usr/bin/apply_diff_relay_logs line 466
	main::main() called at /usr/bin/apply_diff_relay_logs line 112

解决办法：
[root@master ~]# ln -s /usr/local/mysql/bin/mysql /usr/bin/mysql
[root@slave1 ~]# ln -s /usr/local/mysql/bin/mysql /usr/bin/mysql
[root@slave2 ~]# ln -s /usr/local/mysql/bin/mysql /usr/bin/mysql
```

```ruby
[root@manager ~]# masterha_check_repl --conf=/etc/masterha/app1.cnf
Sun Feb 11 22:30:11 2018 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Sun Feb 11 22:30:11 2018 - [info] Reading application default configurations from /etc/masterha/app1.cnf..
Sun Feb 11 22:30:11 2018 - [info] Reading server configurations from /etc/masterha/app1.cnf..
Sun Feb 11 22:30:11 2018 - [info] MHA::MasterMonitor version 0.55.
Sun Feb 11 22:30:12 2018 - [info] Dead Servers:
Sun Feb 11 22:30:12 2018 - [info] Alive Servers:
Sun Feb 11 22:30:12 2018 - [info]   172.30.105.104(172.30.105.104:3306)
Sun Feb 11 22:30:12 2018 - [info]   172.30.105.105(172.30.105.105:3306)
Sun Feb 11 22:30:12 2018 - [info]   172.30.105.106(172.30.105.106:3306)
Sun Feb 11 22:30:12 2018 - [info] Alive Slaves:
Sun Feb 11 22:30:12 2018 - [info]   172.30.105.105(172.30.105.105:3306)  Version=5.5.20-log (oldest major version between slaves) log-bin:enabled
Sun Feb 11 22:30:12 2018 - [info]     Replicating from 172.30.105.104(172.30.105.104:3306)
Sun Feb 11 22:30:12 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Sun Feb 11 22:30:12 2018 - [info]   172.30.105.106(172.30.105.106:3306)  Version=5.5.20-log (oldest major version between slaves) log-bin:enabled
Sun Feb 11 22:30:12 2018 - [info]     Replicating from 172.30.105.104(172.30.105.104:3306)
Sun Feb 11 22:30:12 2018 - [info]     Not candidate for the new Master (no_master is set)
Sun Feb 11 22:30:12 2018 - [info] Current Alive Master: 172.30.105.104(172.30.105.104:3306)
Sun Feb 11 22:30:12 2018 - [info] Checking slave configurations..
Sun Feb 11 22:30:12 2018 - [info]  read_only=1 is not set on slave 172.30.105.105(172.30.105.105:3306).
Sun Feb 11 22:30:12 2018 - [info]  read_only=1 is not set on slave 172.30.105.106(172.30.105.106:3306).
Sun Feb 11 22:30:12 2018 - [info] Checking replication filtering settings..
Sun Feb 11 22:30:12 2018 - [info]  binlog_do_db= , binlog_ignore_db= 
Sun Feb 11 22:30:12 2018 - [info]  Replication filtering check ok.
Sun Feb 11 22:30:12 2018 - [info] Starting SSH connection tests..
Sun Feb 11 22:30:16 2018 - [info] All SSH connection tests passed successfully.
Sun Feb 11 22:30:16 2018 - [info] Checking MHA Node version..
Sun Feb 11 22:30:17 2018 - [info]  Version check ok.
Sun Feb 11 22:30:17 2018 - [info] Checking SSH publickey authentication settings on the current master..
Sun Feb 11 22:30:17 2018 - [info] HealthCheck: SSH to 172.30.105.104 is reachable.
Sun Feb 11 22:30:17 2018 - [info] Master MHA Node version is 0.54.
Sun Feb 11 22:30:17 2018 - [info] Checking recovery script configurations on the current master..
Sun Feb 11 22:30:17 2018 - [info]   Executing command: save_binary_logs --command=test --start_pos=4 --binlog_dir=/mysqlbinlogs --output_file=/data/masterha/app1/save_binary_logs_test --manager_version=0.55 --start_file=mysql-bin.000008 
Sun Feb 11 22:30:17 2018 - [info]   Connecting to root@172.30.105.104(172.30.105.104).. 
  Creating /data/masterha/app1 if not exists..    ok.
  Checking output directory is accessible or not..
   ok.
  Binlog found at /mysqlbinlogs, up to mysql-bin.000008
Sun Feb 11 22:30:18 2018 - [info] Master setting check done.
Sun Feb 11 22:30:18 2018 - [info] Checking SSH publickey authentication and checking recovery script configurations on all alive slave servers..
Sun Feb 11 22:30:18 2018 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user='mhauser' --slave_host=172.30.105.105 --slave_ip=172.30.105.105 --slave_port=3306 --workdir=/data/masterha/app1 --target_version=5.5.20-log --manager_version=0.55 --relay_log_info=/mysqldata/mysql/relay-log.info  --relay_dir=/mysqldata/mysql/  --slave_pass=xxx
Sun Feb 11 22:30:18 2018 - [info]   Connecting to root@172.30.105.105(172.30.105.105:22).. 
Creating directory /data/masterha/app1.. done.
  Checking slave recovery environment settings..
    Opening /mysqldata/mysql/relay-log.info ... ok.
    Relay log found at /mysqlbinlogs, up to relay-bin.000003
    Temporary relay log file is /mysqlbinlogs/relay-bin.000003
    Testing mysql connection and privileges.. done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Sun Feb 11 22:30:18 2018 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user='mhauser' --slave_host=172.30.105.106 --slave_ip=172.30.105.106 --slave_port=3306 --workdir=/data/masterha/app1 --target_version=5.5.20-log --manager_version=0.55 --relay_log_info=/mysqldata/mysql/relay-log.info  --relay_dir=/mysqldata/mysql/  --slave_pass=xxx
Sun Feb 11 22:30:18 2018 - [info]   Connecting to root@172.30.105.106(172.30.105.106:22).. 
Creating directory /data/masterha/app1.. done.
  Checking slave recovery environment settings..
    Opening /mysqldata/mysql/relay-log.info ... ok.
    Relay log found at /mysqlbinlogs, up to relay-bin.000002
    Temporary relay log file is /mysqlbinlogs/relay-bin.000002
    Testing mysql connection and privileges.. done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Sun Feb 11 22:30:19 2018 - [info] Slaves settings check done.
Sun Feb 11 22:30:19 2018 - [info] 
172.30.105.104 (current master)
 +--172.30.105.105
 +--172.30.105.106

Sun Feb 11 22:30:19 2018 - [info] Checking replication health on 172.30.105.105..
Sun Feb 11 22:30:19 2018 - [info]  ok.
Sun Feb 11 22:30:19 2018 - [info] Checking replication health on 172.30.105.106..
Sun Feb 11 22:30:19 2018 - [info]  ok.
Sun Feb 11 22:30:19 2018 - [info] Checking master_ip_failover_script status:
Sun Feb 11 22:30:19 2018 - [info]   /usr/bin/master_ip_failover --command=status --ssh_user=root --orig_master_host=172.30.105.104 --orig_master_ip=172.30.105.104 --orig_master_port=3306 


IN SCRIPT TEST====/sbin/ifconfig eth0:1 down==/sbin/ifconfig eth0:1 172.30.105.120/24===

Checking the Status of the script.. OK 
Sun Feb 11 22:30:19 2018 - [info]  OK.
Sun Feb 11 22:30:19 2018 - [warning] shutdown_script is not defined.
Sun Feb 11 22:30:19 2018 - [info] Got exit code 0 (Not master dead).

MySQL Replication Health is OK.

```

### 8、开启和关闭MHA Manager：

#### 8.1 开启MHA Manager监控：
```ruby
[root@manager ~]# nohup masterha_manager --conf=/etc/masterha/app1.cnf --remove_dead_master_conf --ignore_last_failover < /dev/null > /data/masterha/app1/manager.log 2>&1 & 

// --remove_dead_master_conf:当发生主从切换后，老的主库的ip将会从配置文件中移除，也就是这个配置文件中/etc/masterha/app1.cnf自动删除老的master服务器的配置信息
// --manger_log:日志存放位置
// --ignore_last_failover：在默认情况下，如果MHA检测到连续发生宕机，且两次宕机间隔不足8小时的话，则不会进行Failover,之所以这样限制是为了避免ping-pong效应,
该参数代表忽略上次MHA触发切换产生的文件，默认情况下，MHA发生切换后会在日志目录，也就是上面设置的/data/masterha/app1/产生app1.failover.complete文件，
下次再次切换的时候如果发现该目录下存在该文件将不允许触发切换，除非在第一次切换后收到删除该文件，为了方便，这里设置为--ignore_last_failover
```
#### 8.2 查看MHA运行状态：
```ruby
[root@manager ~]# masterha_check_status --conf=/etc/masterha/app1.cnf 
app1 (pid:44264) is running(0:PING_OK), master:172.30.105.104

查看启动日志：
[root@manager ~]# tail -f /data/masterha/app1/manager.log

[root@manager ~]# masterha_check_status --conf=/etc/masterha/app1.cnf 
app1 monitoring program is now on initialization phase(10:INITIALIZING_MONITOR). Wait for a while and try checking again.
// 有时候会显示这个，那么请稍等一会再重试
```
#### 8.3 关闭MHA Manager监控：
```ruby
[root@manager ~]# masterha_stop --conf=/etc/masterha/app1.cnf
Stopped app1 successfully.
```


### 9、采用手动配置VIP方式
- 通过keepalived的方式管理VIP
- 通过脚本的方式管理VIP(使用脚本管理vip的话，需要手动在master服务器上绑定一个vip)
```ruby
[root@master ~]# ifconfig ens33:1 172.30.105.203
[root@master ~]# ip a
......
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    link/ether 00:0c:29:22:c8:47 brd ff:ff:ff:ff:ff:ff
    inet 172.30.105.104/24 brd 172.30.105.255 scope global ens33
       valid_lft forever preferred_lft forever
    inet 172.30.105.203/16 brd 172.30.255.255 scope global ens33:1
       valid_lft forever preferred_lft forever
    inet6 fe80::eb80:7f7f:ea12:7d07/64 scope link 
       valid_lft forever preferred_lft forever

```

### 10、测试主数据库的自动切换：

[数据库测试详解]()

#### 10.1 在主库(172.30.105.104)上生成10W条记录：
```ruby
[root@master ~]# /usr/local/sysbench/bin/sysbench --test=oltp \
--oltp-table-size=100000 \
--oltp-read-only=off \
--init-rng=on \
--num-threads=16 \
--max-requests=0 \
--oltp-dist-type=uniform \
--max-time=1800 \
--mysql-user=root \
--mysql-password=wangzongyu \
--mysql-socket=/tmp/mysql.sock \
--db-driver=mysql \
--mysql-table-engine=innodb \
--oltp-test-mode=complex prepare

sysbench v0.4.8:  multi-threaded system evaluation benchmark

Creating table 'sbtest'...
Creating 1000000 records in table 'sbtest'...
 
```
#### 10.2 从库(172.30.105.106)关闭的IO线程：
`不要等master上的10W条记录都生成，就要关闭IO线程，目的是使从库落后于主库，另一台半同步的slave中并没有停止IO线程，所以可以继续接收日志`
```ruby

mysql> stop slave io_thread;

```

#### 10.3 在主库(172.30.105.104)上进行压力测试会产生大量的Binlog：

```ruby
[root@master ~]# /usr/local/sysbench/bin/sysbench --test=oltp \
--oltp-table-size=100000 \
--oltp-read-only=off  \
--num-threads=16 \
--max-requests=0 \
--oltp-dist-type=uniform \
--max-time=180 \
--mysql-user=root \
--mysql-password=wangzongyu \
--mysql-socket=/tmp/mysql.sock \
--db-driver=mysql \
--mysql-table-engine=innodb \
--oltp-test-mode=complex run 

sysbench v0.4.8:  multi-threaded system evaluation benchmark

WARNING: Preparing of "BEGIN" is unsupported, using emulation
(last message repeated 15 times)
Running the test with following options:
Number of threads: 16

Doing OLTP test.
Running mixed OLTP test
Using Uniform distribution
Using "BEGIN" for starting transactions
Using auto_inc on the id column
Threads started!
Time limit exceeded, exiting...
(last message repeated 15 times)
Done.

OLTP test statistics:
    queries performed:
        read:                            97930
        write:                           34975
        other:                           13990
        total:                           146895
    transactions:                        6995   (38.81 per sec.)
    deadlocks:                           0      (0.00 per sec.)
    read/write requests:                 132905 (737.46 per sec.)
    other operations:                    13990  (77.63 per sec.)

Test execution summary:
    total time:                          180.2194s
    total number of events:              6995
    total time taken by event execution: 2882.7302
    per-request statistics:
         min:                            0.0031s
         avg:                            0.4121s
         max:                            2.0698s
         approx.  95 percentile:         1.7539s

Threads fairness:
    events (avg/stddev):           437.1875/4.00
    execution time (avg/stddev):   180.1706/0.06

```

#### 10.4 开启从库(172.30.105.106)的IO线程，追赶落后的Master的Binlog
```ruby
mysql> start slave io_thread;

```
#### 10.5 停止掉主库的MySQL进程，模拟主库发生故障，进行自动的failover操作：
```ruby
[root@manager ~]# tail -f /data/masterha/manager.log 
Mon Feb 12 20:56:08 2018 - [info] Set secondary check script: /usr/bin/masterha_secondary_check -s slave1 -s master
Mon Feb 12 20:56:08 2018 - [info] Starting ping health check on 172.30.105.104(172.30.105.104:3306)..
Mon Feb 12 20:56:08 2018 - [info] Ping(SELECT) succeeded, waiting until MySQL doesn't respond..
Mon Feb 12 21:26:41 2018 - [warning] Got timeout on MySQL Ping child process and killed it! at /usr/share/perl5/vendor_perl/MHA/HealthCheck.pm line 384.
Mon Feb 12 21:26:41 2018 - [info] Executing SSH check script: save_binary_logs --command=test --start_pos=4 --binlog_dir=/mysqlbinlogs --output_file=/data/masterha/app1/save_binary_logs_test --manager_version=0.55 --binlog_prefix=mysql-bin
Mon Feb 12 21:26:41 2018 - [info] Executing seconary network check script: /usr/bin/masterha_secondary_check -s slave1 -s master  --user=root  --master_host=172.30.105.104  --master_ip=172.30.105.104  --master_port=3306
Mon Feb 12 21:26:42 2018 - [info] HealthCheck: SSH to 172.30.105.104 is reachable.
Mon Feb 12 21:26:43 2018 - [info] Ping(SELECT) succeeded, waiting until MySQL doesn't respond..
Mon Feb 12 21:26:44 2018 - [warning] Got timeout on Secondary Check child process and killed it! at /usr/share/perl5/vendor_perl/MHA/HealthCheck.pm line 384.
Master is reachable from slave1!

Mon Feb 12 21:29:27 2018 - [warning] Got timeout on MySQL Ping child process and killed it! at /usr/share/perl5/vendor_perl/MHA/HealthCheck.pm line 384.
Mon Feb 12 21:29:27 2018 - [info] Executing seconary network check script: /usr/bin/masterha_secondary_check -s slave1 -s master  --user=root  --master_host=172.30.105.104  --master_ip=172.30.105.104  --master_port=3306
Mon Feb 12 21:29:27 2018 - [info] Executing SSH check script: save_binary_logs --command=test --start_pos=4 --binlog_dir=/mysqlbinlogs --output_file=/data/masterha/app1/save_binary_logs_test --manager_version=0.55 --binlog_prefix=mysql-bin
Master is reachable from slave1!
Mon Feb 12 21:29:28 2018 - [warning] Master is reachable from at least one of other monitoring servers. Failover should not happen.
Mon Feb 12 21:29:28 2018 - [info] HealthCheck: SSH to 172.30.105.104 is reachable.
Mon Feb 12 21:29:29 2018 - [info] Ping(SELECT) succeeded, waiting until MySQL doesn't respond..
Mon Feb 12 21:29:52 2018 - [warning] Got timeout on MySQL Ping child process and killed it! at /usr/share/perl5/vendor_perl/MHA/HealthCheck.pm line 384.
Mon Feb 12 21:29:52 2018 - [info] Executing seconary network check script: /usr/bin/masterha_secondary_check -s slave1 -s master  --user=root  --master_host=172.30.105.104  --master_ip=172.30.105.104  --master_port=3306
Mon Feb 12 21:29:52 2018 - [info] Executing SSH check script: save_binary_logs --command=test --start_pos=4 --binlog_dir=/mysqlbinlogs --output_file=/data/masterha/app1/save_binary_logs_test --manager_version=0.55 --binlog_prefix=mysql-bin
Mon Feb 12 21:29:52 2018 - [info] Ping(SELECT) succeeded, waiting until MySQL doesn't respond..
Master is reachable from slave1!
Mon Feb 12 21:29:52 2018 - [warning] Master is reachable from at least one of other monitoring servers. Failover should not happen.
Mon Feb 12 21:29:52 2018 - [info] HealthCheck: SSH to 172.30.105.104 is reachable.
Mon Feb 12 21:31:14 2018 - [warning] Got error on MySQL select ping: 2006 (MySQL server has gone away)
Mon Feb 12 21:31:14 2018 - [info] Executing seconary network check script: /usr/bin/masterha_secondary_check -s slave1 -s master  --user=root  --master_host=172.30.105.104  --master_ip=172.30.105.104  --master_port=3306
Mon Feb 12 21:31:14 2018 - [info] Executing SSH check script: save_binary_logs --command=test --start_pos=4 --binlog_dir=/mysqlbinlogs --output_file=/data/masterha/app1/save_binary_logs_test --manager_version=0.55 --binlog_prefix=mysql-bin
Monitoring server slave1 is reachable, Master is not reachable from slave1. OK.
Mon Feb 12 21:31:14 2018 - [info] HealthCheck: SSH to 172.30.105.104 is reachable.
Mon Feb 12 21:31:15 2018 - [warning] Got error on MySQL connect: 2003 (Can't connect to MySQL server on '172.30.105.104' (111))
Mon Feb 12 21:31:15 2018 - [warning] Connection failed 1 time(s)..
Mon Feb 12 21:31:16 2018 - [warning] Got error on MySQL connect: 2003 (Can't connect to MySQL server on '172.30.105.104' (111))
Mon Feb 12 21:31:16 2018 - [warning] Connection failed 2 time(s)..
Monitoring server master is reachable, Master is not reachable from master. OK.
Mon Feb 12 21:31:16 2018 - [info] Master is not reachable from all other monitoring servers. Failover should start.
Mon Feb 12 21:31:17 2018 - [warning] Got error on MySQL connect: 2003 (Can't connect to MySQL server on '172.30.105.104' (111))
Mon Feb 12 21:31:17 2018 - [warning] Connection failed 3 time(s)..
Mon Feb 12 21:31:17 2018 - [warning] Master is not reachable from health checker!
Mon Feb 12 21:31:17 2018 - [warning] Master 172.30.105.104(172.30.105.104:3306) is not reachable!
Mon Feb 12 21:31:17 2018 - [warning] SSH is reachable.
Mon Feb 12 21:31:17 2018 - [info] Connecting to a master server failed. Reading configuration file /etc/masterha_default.cnf and /etc/masterha/app1.cnf again, and trying to connect to all servers to check server status..
Mon Feb 12 21:31:17 2018 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Mon Feb 12 21:31:17 2018 - [info] Reading application default configurations from /etc/masterha/app1.cnf..
Mon Feb 12 21:31:17 2018 - [info] Reading server configurations from /etc/masterha/app1.cnf..
Mon Feb 12 21:31:18 2018 - [info] Dead Servers:
Mon Feb 12 21:31:18 2018 - [info]   172.30.105.104(172.30.105.104:3306)
Mon Feb 12 21:31:18 2018 - [info] Alive Servers:
Mon Feb 12 21:31:18 2018 - [info]   172.30.105.105(172.30.105.105:3306)
Mon Feb 12 21:31:18 2018 - [info]   172.30.105.106(172.30.105.106:3306)
Mon Feb 12 21:31:18 2018 - [info] Alive Slaves:
Mon Feb 12 21:31:18 2018 - [info]   172.30.105.105(172.30.105.105:3306)  Version=5.5.20-log (oldest major version between slaves) log-bin:enabled
Mon Feb 12 21:31:18 2018 - [info]     Replicating from 172.30.105.104(172.30.105.104:3306)
Mon Feb 12 21:31:18 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Mon Feb 12 21:31:18 2018 - [info]   172.30.105.106(172.30.105.106:3306)  Version=5.5.20-log (oldest major version between slaves) log-bin:enabled
Mon Feb 12 21:31:18 2018 - [info]     Replicating from 172.30.105.104(172.30.105.104:3306)
Mon Feb 12 21:31:18 2018 - [info]     Not candidate for the new Master (no_master is set)
Mon Feb 12 21:31:18 2018 - [info] Checking slave configurations..
Mon Feb 12 21:31:18 2018 - [info]  read_only=1 is not set on slave 172.30.105.105(172.30.105.105:3306).
Mon Feb 12 21:31:18 2018 - [warning]  relay_log_purge=0 is not set on slave 172.30.105.105(172.30.105.105:3306).
Mon Feb 12 21:31:18 2018 - [info]  read_only=1 is not set on slave 172.30.105.106(172.30.105.106:3306).
Mon Feb 12 21:31:18 2018 - [warning]  relay_log_purge=0 is not set on slave 172.30.105.106(172.30.105.106:3306).
Mon Feb 12 21:31:18 2018 - [info] Checking replication filtering settings..
Mon Feb 12 21:31:18 2018 - [info]  Replication filtering check ok.
Mon Feb 12 21:31:18 2018 - [info] Master is down!
Mon Feb 12 21:31:18 2018 - [info] Terminating monitoring script.
Mon Feb 12 21:31:18 2018 - [info] Got exit code 20 (Master dead).
Mon Feb 12 21:31:18 2018 - [info] MHA::MasterFailover version 0.55.
Mon Feb 12 21:31:18 2018 - [info] Starting master failover.
Mon Feb 12 21:31:18 2018 - [info] 
Mon Feb 12 21:31:18 2018 - [info] * Phase 1: Configuration Check Phase..
Mon Feb 12 21:31:18 2018 - [info] 
Mon Feb 12 21:31:19 2018 - [info] Dead Servers:
Mon Feb 12 21:31:19 2018 - [info]   172.30.105.104(172.30.105.104:3306)
Mon Feb 12 21:31:19 2018 - [info] Checking master reachability via mysql(double check)..
Mon Feb 12 21:31:19 2018 - [info]  ok.
Mon Feb 12 21:31:19 2018 - [info] Alive Servers:
Mon Feb 12 21:31:19 2018 - [info]   172.30.105.105(172.30.105.105:3306)
Mon Feb 12 21:31:19 2018 - [info]   172.30.105.106(172.30.105.106:3306)
Mon Feb 12 21:31:19 2018 - [info] Alive Slaves:
Mon Feb 12 21:31:19 2018 - [info]   172.30.105.105(172.30.105.105:3306)  Version=5.5.20-log (oldest major version between slaves) log-bin:enabled
Mon Feb 12 21:31:19 2018 - [info]     Replicating from 172.30.105.104(172.30.105.104:3306)
Mon Feb 12 21:31:19 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Mon Feb 12 21:31:19 2018 - [info]   172.30.105.106(172.30.105.106:3306)  Version=5.5.20-log (oldest major version between slaves) log-bin:enabled
Mon Feb 12 21:31:19 2018 - [info]     Replicating from 172.30.105.104(172.30.105.104:3306)
Mon Feb 12 21:31:19 2018 - [info]     Not candidate for the new Master (no_master is set)
Mon Feb 12 21:31:19 2018 - [info] ** Phase 1: Configuration Check Phase completed.
Mon Feb 12 21:31:19 2018 - [info] 
Mon Feb 12 21:31:19 2018 - [info] * Phase 2: Dead Master Shutdown Phase..
Mon Feb 12 21:31:19 2018 - [info] 
Mon Feb 12 21:31:19 2018 - [info] Forcing shutdown so that applications never connect to the current master..
Mon Feb 12 21:31:19 2018 - [info] Executing master IP deactivatation script:
Mon Feb 12 21:31:19 2018 - [info]   /usr/bin/master_ip_failover --orig_master_host=172.30.105.104 --orig_master_ip=172.30.105.104 --orig_master_port=3306 --command=stopssh --ssh_user=root  


IN SCRIPT TEST====/sbin/ifconfig ens33:1 down==/sbin/ifconfig ens33:1 172.30.105.203/24===

Disabling the VIP on old master: 172.30.105.104 
SIOCSIFFLAGS: Cannot assign requested address
Mon Feb 12 21:31:20 2018 - [info]  done.
Mon Feb 12 21:31:20 2018 - [warning] shutdown_script is not set. Skipping explicit shutting down of the dead master.
Mon Feb 12 21:31:22 2018 - [info] * Phase 2: Dead Master Shutdown Phase completed.
Mon Feb 12 21:31:22 2018 - [info] 
Mon Feb 12 21:31:22 2018 - [info] * Phase 3: Master Recovery Phase..
Mon Feb 12 21:31:22 2018 - [info] 
Mon Feb 12 21:31:22 2018 - [info] * Phase 3.1: Getting Latest Slaves Phase..
Mon Feb 12 21:31:22 2018 - [info] 
Mon Feb 12 21:31:22 2018 - [info] The latest binary log file/position on all slaves is mysql-bin.000011:12308339
Mon Feb 12 21:31:22 2018 - [info] Latest slaves (Slaves that received relay log files to the latest):
Mon Feb 12 21:31:22 2018 - [info]   172.30.105.105(172.30.105.105:3306)  Version=5.5.20-log (oldest major version between slaves) log-bin:enabled
Mon Feb 12 21:31:22 2018 - [info]     Replicating from 172.30.105.104(172.30.105.104:3306)
Mon Feb 12 21:31:22 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Mon Feb 12 21:31:22 2018 - [info]   172.30.105.106(172.30.105.106:3306)  Version=5.5.20-log (oldest major version between slaves) log-bin:enabled
Mon Feb 12 21:31:22 2018 - [info]     Replicating from 172.30.105.104(172.30.105.104:3306)
Mon Feb 12 21:31:22 2018 - [info]     Not candidate for the new Master (no_master is set)
Mon Feb 12 21:31:22 2018 - [info] The oldest binary log file/position on all slaves is mysql-bin.000011:12308339
Mon Feb 12 21:31:22 2018 - [info] Oldest slaves:
Mon Feb 12 21:31:22 2018 - [info]   172.30.105.105(172.30.105.105:3306)  Version=5.5.20-log (oldest major version between slaves) log-bin:enabled
Mon Feb 12 21:31:22 2018 - [info]     Replicating from 172.30.105.104(172.30.105.104:3306)
Mon Feb 12 21:31:22 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Mon Feb 12 21:31:22 2018 - [info]   172.30.105.106(172.30.105.106:3306)  Version=5.5.20-log (oldest major version between slaves) log-bin:enabled
Mon Feb 12 21:31:22 2018 - [info]     Replicating from 172.30.105.104(172.30.105.104:3306)
Mon Feb 12 21:31:22 2018 - [info]     Not candidate for the new Master (no_master is set)
Mon Feb 12 21:31:22 2018 - [info] 
Mon Feb 12 21:31:22 2018 - [info] * Phase 3.2: Saving Dead Master's Binlog Phase..
Mon Feb 12 21:31:22 2018 - [info] 
Mon Feb 12 21:31:22 2018 - [info] Fetching dead master's binary logs..
Mon Feb 12 21:31:22 2018 - [info] Executing command on the dead master 172.30.105.104(172.30.105.104:3306): save_binary_logs --command=save --start_file=mysql-bin.000011  --start_pos=12308339 --binlog_dir=/mysqlbinlogs --output_file=/data/masterha/app1/saved_master_binlog_from_172.30.105.104_3306_20180212213118.binlog --handle_raw_binlog=1 --disable_log_bin=0 --manager_version=0.55
  Creating /data/masterha/app1 if not exists..    ok.
 Concat binary/relay logs from mysql-bin.000011 pos 12308339 to mysql-bin.000011 EOF into /data/masterha/app1/saved_master_binlog_from_172.30.105.104_3306_20180212213118.binlog ..
  Dumping binlog format description event, from position 0 to 107.. ok.
  Dumping effective binlog data from /mysqlbinlogs/mysql-bin.000011 position 12308339 to tail(12308358).. ok.
 Concat succeeded.
Mon Feb 12 21:31:26 2018 - [info] scp from root@172.30.105.104:/data/masterha/app1/saved_master_binlog_from_172.30.105.104_3306_20180212213118.binlog to local:/data/masterha/app1/saved_master_binlog_from_172.30.105.104_3306_20180212213118.binlog succeeded.
Mon Feb 12 21:31:26 2018 - [info] HealthCheck: SSH to 172.30.105.105 is reachable.
Mon Feb 12 21:31:28 2018 - [info] HealthCheck: SSH to 172.30.105.106 is reachable.
Mon Feb 12 21:31:28 2018 - [info] 
Mon Feb 12 21:31:28 2018 - [info] * Phase 3.3: Determining New Master Phase..
Mon Feb 12 21:31:28 2018 - [info] 
Mon Feb 12 21:31:28 2018 - [info] Finding the latest slave that has all relay logs for recovering other slaves..
Mon Feb 12 21:31:28 2018 - [info] All slaves received relay logs to the same position. No need to resync each other.
Mon Feb 12 21:31:28 2018 - [info] Searching new master from slaves..
Mon Feb 12 21:31:28 2018 - [info]  Candidate masters from the configuration file:
Mon Feb 12 21:31:28 2018 - [info]   172.30.105.105(172.30.105.105:3306)  Version=5.5.20-log (oldest major version between slaves) log-bin:enabled
Mon Feb 12 21:31:28 2018 - [info]     Replicating from 172.30.105.104(172.30.105.104:3306)
Mon Feb 12 21:31:28 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Mon Feb 12 21:31:28 2018 - [info]  Non-candidate masters:
Mon Feb 12 21:31:28 2018 - [info]   172.30.105.106(172.30.105.106:3306)  Version=5.5.20-log (oldest major version between slaves) log-bin:enabled
Mon Feb 12 21:31:28 2018 - [info]     Replicating from 172.30.105.104(172.30.105.104:3306)
Mon Feb 12 21:31:28 2018 - [info]     Not candidate for the new Master (no_master is set)
Mon Feb 12 21:31:28 2018 - [info]  Searching from candidate_master slaves which have received the latest relay log events..
Mon Feb 12 21:31:28 2018 - [info] New master is 172.30.105.105(172.30.105.105:3306)
Mon Feb 12 21:31:28 2018 - [info] Starting master failover..
Mon Feb 12 21:31:28 2018 - [info] 
From:
172.30.105.104 (current master)
 +--172.30.105.105
 +--172.30.105.106

To:
172.30.105.105 (new master)
 +--172.30.105.106
Mon Feb 12 21:31:28 2018 - [info] 
Mon Feb 12 21:31:28 2018 - [info] * Phase 3.3: New Master Diff Log Generation Phase..
Mon Feb 12 21:31:28 2018 - [info] 
Mon Feb 12 21:31:28 2018 - [info]  This server has all relay logs. No need to generate diff files from the latest slave.
Mon Feb 12 21:31:28 2018 - [info] Sending binlog..
Mon Feb 12 21:31:29 2018 - [info] scp from local:/data/masterha/app1/saved_master_binlog_from_172.30.105.104_3306_20180212213118.binlog to root@172.30.105.105:/data/masterha/app1/saved_master_binlog_from_172.30.105.104_3306_20180212213118.binlog succeeded.
Mon Feb 12 21:31:29 2018 - [info] 
Mon Feb 12 21:31:29 2018 - [info] * Phase 3.4: Master Log Apply Phase..
Mon Feb 12 21:31:29 2018 - [info] 
Mon Feb 12 21:31:29 2018 - [info] *NOTICE: If any error happens from this phase, manual recovery is needed.
Mon Feb 12 21:31:29 2018 - [info] Starting recovery on 172.30.105.105(172.30.105.105:3306)..
Mon Feb 12 21:31:29 2018 - [info]  Generating diffs succeeded.
Mon Feb 12 21:31:29 2018 - [info] Waiting until all relay logs are applied.
Mon Feb 12 21:31:29 2018 - [info]  done.
Mon Feb 12 21:31:31 2018 - [info] Getting slave status..
Mon Feb 12 21:31:31 2018 - [info] This slave(172.30.105.105)'s Exec_Master_Log_Pos equals to Read_Master_Log_Pos(mysql-bin.000011:12308339). No need to recover from Exec_Master_Log_Pos.
Mon Feb 12 21:31:31 2018 - [info] Connecting to the target slave host 172.30.105.105, running recover script..
Mon Feb 12 21:31:31 2018 - [info] Executing command: apply_diff_relay_logs --command=apply --slave_user='mhauser' --slave_host=172.30.105.105 --slave_ip=172.30.105.105  --slave_port=3306 --apply_files=/data/masterha/app1/saved_master_binlog_from_172.30.105.104_3306_20180212213118.binlog --workdir=/data/masterha/app1 --target_version=5.5.20-log --timestamp=20180212213118 --handle_raw_binlog=1 --disable_log_bin=0 --manager_version=0.55 --slave_pass=xxx
Mon Feb 12 21:31:31 2018 - [info] 
Applying differential binary/relay log files /data/masterha/app1/saved_master_binlog_from_172.30.105.104_3306_20180212213118.binlog on 172.30.105.105:3306. This may take long time...
Applying log files succeeded.
Mon Feb 12 21:31:31 2018 - [info]  All relay logs were successfully applied.
Mon Feb 12 21:31:31 2018 - [info] Getting new master's binlog name and position..
Mon Feb 12 21:31:31 2018 - [info]  mysql-bin.000004:107
Mon Feb 12 21:31:31 2018 - [info]  All other slaves should start replication from here. Statement should be: CHANGE MASTER TO MASTER_HOST='172.30.105.105', MASTER_PORT=3306, MASTER_LOG_FILE='mysql-bin.000004', MASTER_LOG_POS=107, MASTER_USER='repuser', MASTER_PASSWORD='xxx';
Mon Feb 12 21:31:31 2018 - [info] Executing master IP activate script:
Mon Feb 12 21:31:31 2018 - [info]   /usr/bin/master_ip_failover --command=start --ssh_user=root --orig_master_host=172.30.105.104 --orig_master_ip=172.30.105.104 --orig_master_port=3306 --new_master_host=172.30.105.105 --new_master_ip=172.30.105.105 --new_master_port=3306 --new_master_user='mhauser' --new_master_password='wangzongyu'  
Unknown option: new_master_user
Unknown option: new_master_password


IN SCRIPT TEST====/sbin/ifconfig ens33:1 down==/sbin/ifconfig ens33:1 172.30.105.203/24===

Enabling the VIP - 172.30.105.203/24 on the new master - 172.30.105.105 
Mon Feb 12 21:31:32 2018 - [info]  OK.
Mon Feb 12 21:31:32 2018 - [info] ** Finished master recovery successfully.
Mon Feb 12 21:31:32 2018 - [info] * Phase 3: Master Recovery Phase completed.
Mon Feb 12 21:31:32 2018 - [info] 
Mon Feb 12 21:31:32 2018 - [info] * Phase 4: Slaves Recovery Phase..
Mon Feb 12 21:31:32 2018 - [info] 
Mon Feb 12 21:31:32 2018 - [info] * Phase 4.1: Starting Parallel Slave Diff Log Generation Phase..
Mon Feb 12 21:31:32 2018 - [info] 
Mon Feb 12 21:31:32 2018 - [info] -- Slave diff file generation on host 172.30.105.106(172.30.105.106:3306) started, pid: 3938. Check tmp log /data/masterha/app1/172.30.105.106_3306_20180212213118.log if it takes time..
Mon Feb 12 21:31:33 2018 - [info] 
Mon Feb 12 21:31:33 2018 - [info] Log messages from 172.30.105.106 ...
Mon Feb 12 21:31:33 2018 - [info] 
Mon Feb 12 21:31:32 2018 - [info]  This server has all relay logs. No need to generate diff files from the latest slave.
Mon Feb 12 21:31:33 2018 - [info] End of log messages from 172.30.105.106.
Mon Feb 12 21:31:33 2018 - [info] -- 172.30.105.106(172.30.105.106:3306) has the latest relay log events.
Mon Feb 12 21:31:33 2018 - [info] Generating relay diff files from the latest slave succeeded.
Mon Feb 12 21:31:33 2018 - [info] 
Mon Feb 12 21:31:33 2018 - [info] * Phase 4.2: Starting Parallel Slave Log Apply Phase..
Mon Feb 12 21:31:33 2018 - [info] 
Mon Feb 12 21:31:33 2018 - [info] -- Slave recovery on host 172.30.105.106(172.30.105.106:3306) started, pid: 3940. Check tmp log /data/masterha/app1/172.30.105.106_3306_20180212213118.log if it takes time..
Mon Feb 12 21:32:43 2018 - [info] 
Mon Feb 12 21:32:43 2018 - [info] Log messages from 172.30.105.106 ...
Mon Feb 12 21:32:43 2018 - [info] 
Mon Feb 12 21:31:33 2018 - [info] Sending binlog..
Mon Feb 12 21:31:34 2018 - [info] scp from local:/data/masterha/app1/saved_master_binlog_from_172.30.105.104_3306_20180212213118.binlog to root@172.30.105.106:/data/masterha/app1/saved_master_binlog_from_172.30.105.104_3306_20180212213118.binlog succeeded.
Mon Feb 12 21:31:34 2018 - [info] Starting recovery on 172.30.105.106(172.30.105.106:3306)..
Mon Feb 12 21:31:34 2018 - [info]  Generating diffs succeeded.
Mon Feb 12 21:31:34 2018 - [info] Waiting until all relay logs are applied.
Mon Feb 12 21:32:42 2018 - [info]  done.
Mon Feb 12 21:32:42 2018 - [info] Getting slave status..
Mon Feb 12 21:32:42 2018 - [info] This slave(172.30.105.106)'s Exec_Master_Log_Pos equals to Read_Master_Log_Pos(mysql-bin.000011:12308339). No need to recover from Exec_Master_Log_Pos.
Mon Feb 12 21:32:42 2018 - [info] Connecting to the target slave host 172.30.105.106, running recover script..
Mon Feb 12 21:32:42 2018 - [info] Executing command: apply_diff_relay_logs --command=apply --slave_user='mhauser' --slave_host=172.30.105.106 --slave_ip=172.30.105.106  --slave_port=3306 --apply_files=/data/masterha/app1/saved_master_binlog_from_172.30.105.104_3306_20180212213118.binlog --workdir=/data/masterha/app1 --target_version=5.5.20-log --timestamp=20180212213118 --handle_raw_binlog=1 --disable_log_bin=0 --manager_version=0.55 --slave_pass=xxx
Mon Feb 12 21:32:42 2018 - [info] 
Applying differential binary/relay log files /data/masterha/app1/saved_master_binlog_from_172.30.105.104_3306_20180212213118.binlog on 172.30.105.106:3306. This may take long time...
Applying log files succeeded.
Mon Feb 12 21:32:42 2018 - [info]  All relay logs were successfully applied.
Mon Feb 12 21:32:42 2018 - [info]  Resetting slave 172.30.105.106(172.30.105.106:3306) and starting replication from the new master 172.30.105.105(172.30.105.105:3306)..
Mon Feb 12 21:32:42 2018 - [info]  Executed CHANGE MASTER.
Mon Feb 12 21:32:42 2018 - [info]  Slave started.
Mon Feb 12 21:32:43 2018 - [info] End of log messages from 172.30.105.106.
Mon Feb 12 21:32:43 2018 - [info] -- Slave recovery on host 172.30.105.106(172.30.105.106:3306) succeeded.
Mon Feb 12 21:32:43 2018 - [info] All new slave servers recovered successfully.
Mon Feb 12 21:32:43 2018 - [info] 
Mon Feb 12 21:32:43 2018 - [info] * Phase 5: New master cleanup phase..
Mon Feb 12 21:32:43 2018 - [info] 
Mon Feb 12 21:32:43 2018 - [info] Resetting slave info on the new master..
Mon Feb 12 21:32:43 2018 - [info]  172.30.105.105: Resetting slave info succeeded.
Mon Feb 12 21:32:43 2018 - [info] Master failover to 172.30.105.105(172.30.105.105:3306) completed successfully.
Mon Feb 12 21:32:43 2018 - [info] Deleted server1 entry from /etc/masterha/app1.cnf .
Mon Feb 12 21:32:43 2018 - [info] 

----- Failover Report -----

app1: MySQL Master failover 172.30.105.104 to 172.30.105.105 succeeded

Master 172.30.105.104 is down!

Check MHA Manager logs at manager:/data/masterha/manager.log for details.

Started automated(non-interactive) failover.
Invalidated master IP address on 172.30.105.104.
The latest slave 172.30.105.105(172.30.105.105:3306) has all relay logs for recovery.
Selected 172.30.105.105 as a new master.
172.30.105.105: OK: Applying all logs succeeded.
172.30.105.105: OK: Activated master IP address.
172.30.105.106: This host has the latest relay log events.
Generating relay diff files from the latest slave succeeded.
172.30.105.106: OK: Applying all logs succeeded. Slave started, replicating from 172.30.105.105.
172.30.105.105: Resetting slave info succeeded.
Master failover to 172.30.105.105(172.30.105.105:3306) completed successfully.
Mon Feb 12 21:32:43 2018 - [info] Sending mail..


```
`切换成功之后，在/etc/masterha/app1.cnf配置文件中老的Master信息会自动删除`

#### 10.6 整个自动切换过程如下如下：
`可通过日志观察切换过程`    
- 配置文件检查阶段，这个阶段会检查整个集群配置文件配置；
- 宕机的master处理，这个阶段包括虚拟ip摘除操作，主机关机操作（这个我这里还没有实现，需要研究）；
- 复制dead maste和最新slave相差的relay log，并保存到MHA Manger具体的目录下；
- 识别含有最新更新的slave；
- 应用从master保存的二进制日志事件（binlog events）；
- 提升一个slave为新的master进行复制；
- 使其他的slave连接新的master进行复制；
- 最后启动MHA Manger监控；

VIP已经切换到了Slave1节点：
```ruby
[root@slave1 ~]# ip a
......
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    link/ether 00:0c:29:bb:f4:cc brd ff:ff:ff:ff:ff:ff
    inet 172.30.105.105/24 brd 172.30.105.255 scope global ens33
       valid_lft forever preferred_lft forever
    inet 172.30.105.203/16 brd 172.30.255.255 scope global ens33:1
       valid_lft forever preferred_lft forever
    inet6 fe80::e2ad:ab11:3077:e697/64 scope link 
       valid_lft forever preferred_lft forever

```
- 发生主从切换后，MHAmanager服务会自动停掉，且在manager_workdir目录下面生成文件app1.failover.complete，若要启动MHA，必须先确保无此文件
```ruby
[root@manager app1]# pwd
/data/masterha/app1
[root@manager app1]# ll
total 4
-rw-r--r--. 1 root root   0 Feb 11 23:33 app1.failover.complete
```
```ruby
[root@manager ~]# masterha_check_status --conf=/etc/masterha/app1.cnf 
app1 (pid:28127) is running(0:PING_OK), master:172.30.105.105
```

收到的邮件截图：     
![](https://github.com/ZongYuWang/image/blob/master/MySQL/MySQL_MHA1.png)


### 11、测试主数据库的手动切换：
&emsp;&emsp;手动切换这种场景意味着在业务上没有启用MHA自动切换功能，当主服务器故障时，人工手动调用MHA来进行故障切换操作      
`手动切换操作同自动切换操作，只是不能启动MHA Manager监控，也要模拟MySQL主库故障`      

```ruby
[root@manager ~]# masterha_stop --conf=/etc/masterha/app1.cnf
```

```ruby
[root@manager ~]# masterha_master_switch --master_state=dead --conf=/etc/masterha/app1.cnf --dead_master_host=172.30.105.104 --dead_master_port=3306 --new_master_host=172.30.105.105 --new_master_port=3306 --ignore_last_failover && tail -f  /data/masterha/manager.log 

--dead_master_ip=<dead_master_ip> is not set. Using 172.30.105.104.
Mon Feb 12 08:58:20 2018 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Mon Feb 12 08:58:20 2018 - [info] Reading application default configurations from /etc/masterha/app1.cnf..
Mon Feb 12 08:58:20 2018 - [info] Reading server configurations from /etc/masterha/app1.cnf..
Mon Feb 12 08:58:20 2018 - [info] MHA::MasterFailover version 0.55.
Mon Feb 12 08:58:20 2018 - [info] Starting master failover.
Mon Feb 12 08:58:20 2018 - [info] 
Mon Feb 12 08:58:20 2018 - [info] * Phase 1: Configuration Check Phase..
Mon Feb 12 08:58:20 2018 - [info] 
Mon Feb 12 08:58:21 2018 - [info] Dead Servers:
Mon Feb 12 08:58:21 2018 - [info]   172.30.105.104(172.30.105.104:3306)
Mon Feb 12 08:58:21 2018 - [info] Checking master reachability via mysql(double check)..
Mon Feb 12 08:58:21 2018 - [info]  ok.
Mon Feb 12 08:58:21 2018 - [info] Alive Servers:
Mon Feb 12 08:58:21 2018 - [info]   172.30.105.105(172.30.105.105:3306)
Mon Feb 12 08:58:21 2018 - [info]   172.30.105.106(172.30.105.106:3306)
Mon Feb 12 08:58:21 2018 - [info] Alive Slaves:
Mon Feb 12 08:58:21 2018 - [info]   172.30.105.105(172.30.105.105:3306)  Version=5.5.20-log (oldest major version between slaves) log-bin:enabled
Mon Feb 12 08:58:21 2018 - [info]     Replicating from 172.30.105.104(172.30.105.104:3306)
Mon Feb 12 08:58:21 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Mon Feb 12 08:58:21 2018 - [info]   172.30.105.106(172.30.105.106:3306)  Version=5.5.20-log (oldest major version between slaves) log-bin:enabled
Mon Feb 12 08:58:21 2018 - [info]     Replicating from 172.30.105.104(172.30.105.104:3306)
Mon Feb 12 08:58:21 2018 - [info]     Not candidate for the new Master (no_master is set)
Master 172.30.105.104 is dead. Proceed? (yes/NO): yes

// 是否确定Master(172.30.105.104)已经宕机，输入yes

Mon Feb 12 08:58:25 2018 - [info] ** Phase 1: Configuration Check Phase completed.
Mon Feb 12 08:58:25 2018 - [info] 
Mon Feb 12 08:58:25 2018 - [info] * Phase 2: Dead Master Shutdown Phase..
Mon Feb 12 08:58:25 2018 - [info] 
Mon Feb 12 08:58:25 2018 - [info] HealthCheck: SSH to 172.30.105.104 is reachable.
Mon Feb 12 08:58:25 2018 - [info] Forcing shutdown so that applications never connect to the current master..
Mon Feb 12 08:58:25 2018 - [info] Executing master IP deactivatation script:
Mon Feb 12 08:58:25 2018 - [info]   /usr/bin/master_ip_failover --orig_master_host=172.30.105.104 --orig_master_ip=172.30.105.104 --orig_master_port=3306 --command=stopssh --ssh_user=root  


IN SCRIPT TEST====/sbin/ifconfig ens33:1 down==/sbin/ifconfig ens33:1 172.30.105.203/24===

Disabling the VIP on old master: 172.30.105.104 
Mon Feb 12 08:58:26 2018 - [info]  done.
Mon Feb 12 08:58:26 2018 - [warning] shutdown_script is not set. Skipping explicit shutting down of the dead master.
Mon Feb 12 08:58:26 2018 - [info] * Phase 2: Dead Master Shutdown Phase completed.
Mon Feb 12 08:58:26 2018 - [info] 
Mon Feb 12 08:58:26 2018 - [info] * Phase 3: Master Recovery Phase..
Mon Feb 12 08:58:26 2018 - [info] 
Mon Feb 12 08:58:26 2018 - [info] * Phase 3.1: Getting Latest Slaves Phase..
Mon Feb 12 08:58:26 2018 - [info] 
Mon Feb 12 08:58:26 2018 - [info] The latest binary log file/position on all slaves is mysql-bin.000010:12976817
Mon Feb 12 08:58:26 2018 - [info] Latest slaves (Slaves that received relay log files to the latest):
Mon Feb 12 08:58:26 2018 - [info]   172.30.105.105(172.30.105.105:3306)  Version=5.5.20-log (oldest major version between slaves) log-bin:enabled
Mon Feb 12 08:58:26 2018 - [info]     Replicating from 172.30.105.104(172.30.105.104:3306)
Mon Feb 12 08:58:26 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Mon Feb 12 08:58:26 2018 - [info]   172.30.105.106(172.30.105.106:3306)  Version=5.5.20-log (oldest major version between slaves) log-bin:enabled
Mon Feb 12 08:58:26 2018 - [info]     Replicating from 172.30.105.104(172.30.105.104:3306)
Mon Feb 12 08:58:26 2018 - [info]     Not candidate for the new Master (no_master is set)
Mon Feb 12 08:58:26 2018 - [info] The oldest binary log file/position on all slaves is mysql-bin.000010:12976817
Mon Feb 12 08:58:26 2018 - [info] Oldest slaves:
Mon Feb 12 08:58:26 2018 - [info]   172.30.105.105(172.30.105.105:3306)  Version=5.5.20-log (oldest major version between slaves) log-bin:enabled
Mon Feb 12 08:58:26 2018 - [info]     Replicating from 172.30.105.104(172.30.105.104:3306)
Mon Feb 12 08:58:26 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Mon Feb 12 08:58:26 2018 - [info]   172.30.105.106(172.30.105.106:3306)  Version=5.5.20-log (oldest major version between slaves) log-bin:enabled
Mon Feb 12 08:58:26 2018 - [info]     Replicating from 172.30.105.104(172.30.105.104:3306)
Mon Feb 12 08:58:26 2018 - [info]     Not candidate for the new Master (no_master is set)
Mon Feb 12 08:58:26 2018 - [info] 
Mon Feb 12 08:58:26 2018 - [info] * Phase 3.2: Saving Dead Master's Binlog Phase..
Mon Feb 12 08:58:26 2018 - [info] 
Mon Feb 12 08:58:26 2018 - [info] Fetching dead master's binary logs..
Mon Feb 12 08:58:26 2018 - [info] Executing command on the dead master 172.30.105.104(172.30.105.104:3306): save_binary_logs --command=save --start_file=mysql-bin.000010  --start_pos=12976817 --binlog_dir=/mysqlbinlogs --output_file=/data/masterha/app1/saved_master_binlog_from_172.30.105.104_3306_20180212085820.binlog --handle_raw_binlog=1 --disable_log_bin=0 --manager_version=0.55
  Creating /data/masterha/app1 if not exists..    ok.
 Concat binary/relay logs from mysql-bin.000010 pos 12976817 to mysql-bin.000010 EOF into /data/masterha/app1/saved_master_binlog_from_172.30.105.104_3306_20180212085820.binlog ..
  Dumping binlog format description event, from position 0 to 107.. ok.
  Dumping effective binlog data from /mysqlbinlogs/mysql-bin.000010 position 12976817 to tail(12976836).. ok.
 Concat succeeded.
saved_master_binlog_from_172.30.105.104_3306_20180212085820.binlog                              100%  126   150.2KB/s   00:00    
Mon Feb 12 08:58:28 2018 - [info] scp from root@172.30.105.104:/data/masterha/app1/saved_master_binlog_from_172.30.105.104_3306_20180212085820.binlog to local:/data/masterha/app1/saved_master_binlog_from_172.30.105.104_3306_20180212085820.binlog succeeded.
Mon Feb 12 08:58:28 2018 - [info] HealthCheck: SSH to 172.30.105.105 is reachable.
Mon Feb 12 08:58:29 2018 - [info] HealthCheck: SSH to 172.30.105.106 is reachable.
Mon Feb 12 08:58:29 2018 - [info] 
Mon Feb 12 08:58:29 2018 - [info] * Phase 3.3: Determining New Master Phase..
Mon Feb 12 08:58:29 2018 - [info] 
Mon Feb 12 08:58:29 2018 - [info] Finding the latest slave that has all relay logs for recovering other slaves..
Mon Feb 12 08:58:29 2018 - [info] All slaves received relay logs to the same position. No need to resync each other.
Mon Feb 12 08:58:29 2018 - [info] 172.30.105.105 can be new master.
Mon Feb 12 08:58:29 2018 - [info] New master is 172.30.105.105(172.30.105.105:3306)
Mon Feb 12 08:58:29 2018 - [info] Starting master failover..
Mon Feb 12 08:58:29 2018 - [info] 
From:
172.30.105.104 (current master)
 +--172.30.105.105
 +--172.30.105.106

To:
172.30.105.105 (new master)
 +--172.30.105.106

Starting master switch from 172.30.105.104(172.30.105.104:3306) to 172.30.105.105(172.30.105.105:3306)? (yes/NO): yes

// 从Slave1(172.30.105.105)启动master，输入yes

Mon Feb 12 08:58:36 2018 - [info] New master decided manually is 172.30.105.105(172.30.105.105:3306)
Mon Feb 12 08:58:36 2018 - [info] 
Mon Feb 12 08:58:36 2018 - [info] * Phase 3.3: New Master Diff Log Generation Phase..
Mon Feb 12 08:58:36 2018 - [info] 
Mon Feb 12 08:58:36 2018 - [info]  This server has all relay logs. No need to generate diff files from the latest slave.
Mon Feb 12 08:58:36 2018 - [info] Sending binlog..
saved_master_binlog_from_172.30.105.104_3306_20180212085820.binlog                              100%  126     0.5KB/s   00:00    
Mon Feb 12 08:58:37 2018 - [info] scp from local:/data/masterha/app1/saved_master_binlog_from_172.30.105.104_3306_20180212085820.binlog to root@172.30.105.105:/data/masterha/app1/saved_master_binlog_from_172.30.105.104_3306_20180212085820.binlog succeeded.
Mon Feb 12 08:58:37 2018 - [info] 
Mon Feb 12 08:58:37 2018 - [info] * Phase 3.4: Master Log Apply Phase..
Mon Feb 12 08:58:37 2018 - [info] 
Mon Feb 12 08:58:37 2018 - [info] *NOTICE: If any error happens from this phase, manual recovery is needed.
Mon Feb 12 08:58:37 2018 - [info] Starting recovery on 172.30.105.105(172.30.105.105:3306)..
Mon Feb 12 08:58:37 2018 - [info]  Generating diffs succeeded.
Mon Feb 12 08:58:37 2018 - [info] Waiting until all relay logs are applied.
Mon Feb 12 08:58:37 2018 - [info]  done.
Mon Feb 12 08:58:37 2018 - [info] Getting slave status..
Mon Feb 12 08:58:37 2018 - [info] This slave(172.30.105.105)'s Exec_Master_Log_Pos equals to Read_Master_Log_Pos(mysql-bin.000010:12976817). No need to recover from Exec_Master_Log_Pos.
Mon Feb 12 08:58:37 2018 - [info] Connecting to the target slave host 172.30.105.105, running recover script..
Mon Feb 12 08:58:37 2018 - [info] Executing command: apply_diff_relay_logs --command=apply --slave_user='mhauser' --slave_host=172.30.105.105 --slave_ip=172.30.105.105  --slave_port=3306 --apply_files=/data/masterha/app1/saved_master_binlog_from_172.30.105.104_3306_20180212085820.binlog --workdir=/data/masterha/app1 --target_version=5.5.20-log --timestamp=20180212085820 --handle_raw_binlog=1 --disable_log_bin=0 --manager_version=0.55 --slave_pass=xxx
Mon Feb 12 08:58:37 2018 - [info] 
Applying differential binary/relay log files /data/masterha/app1/saved_master_binlog_from_172.30.105.104_3306_20180212085820.binlog on 172.30.105.105:3306. This may take long time...
Applying log files succeeded.
Mon Feb 12 08:58:37 2018 - [info]  All relay logs were successfully applied.
Mon Feb 12 08:58:37 2018 - [info] Getting new master's binlog name and position..
Mon Feb 12 08:58:37 2018 - [info]  mysql-bin.000003:107
Mon Feb 12 08:58:37 2018 - [info]  All other slaves should start replication from here. Statement should be: CHANGE MASTER TO MASTER_HOST='172.30.105.105', MASTER_PORT=3306, MASTER_LOG_FILE='mysql-bin.000003', MASTER_LOG_POS=107, MASTER_USER='repuser', MASTER_PASSWORD='xxx';
Mon Feb 12 08:58:37 2018 - [info] Executing master IP activate script:
Mon Feb 12 08:58:37 2018 - [info]   /usr/bin/master_ip_failover --command=start --ssh_user=root --orig_master_host=172.30.105.104 --orig_master_ip=172.30.105.104 --orig_master_port=3306 --new_master_host=172.30.105.105 --new_master_ip=172.30.105.105 --new_master_port=3306 --new_master_user='mhauser' --new_master_password='wangzongyu'  
Unknown option: new_master_user
Unknown option: new_master_password


IN SCRIPT TEST====/sbin/ifconfig ens33:1 down==/sbin/ifconfig ens33:1 172.30.105.203/24===

Enabling the VIP - 172.30.105.203/24 on the new master - 172.30.105.105 
Mon Feb 12 08:58:38 2018 - [info]  OK.
Mon Feb 12 08:58:38 2018 - [info] ** Finished master recovery successfully.
Mon Feb 12 08:58:38 2018 - [info] * Phase 3: Master Recovery Phase completed.
Mon Feb 12 08:58:38 2018 - [info] 
Mon Feb 12 08:58:38 2018 - [info] * Phase 4: Slaves Recovery Phase..
Mon Feb 12 08:58:38 2018 - [info] 
Mon Feb 12 08:58:38 2018 - [info] * Phase 4.1: Starting Parallel Slave Diff Log Generation Phase..
Mon Feb 12 08:58:38 2018 - [info] 
Mon Feb 12 08:58:38 2018 - [info] -- Slave diff file generation on host 172.30.105.106(172.30.105.106:3306) started, pid: 79322. Check tmp log /data/masterha/app1/172.30.105.106_3306_20180212085820.log if it takes time..
Mon Feb 12 08:58:39 2018 - [info] 
Mon Feb 12 08:58:39 2018 - [info] Log messages from 172.30.105.106 ...
Mon Feb 12 08:58:39 2018 - [info] 
Mon Feb 12 08:58:38 2018 - [info]  This server has all relay logs. No need to generate diff files from the latest slave.
Mon Feb 12 08:58:39 2018 - [info] End of log messages from 172.30.105.106.
Mon Feb 12 08:58:39 2018 - [info] -- 172.30.105.106(172.30.105.106:3306) has the latest relay log events.
Mon Feb 12 08:58:39 2018 - [info] Generating relay diff files from the latest slave succeeded.
Mon Feb 12 08:58:39 2018 - [info] 
Mon Feb 12 08:58:39 2018 - [info] * Phase 4.2: Starting Parallel Slave Log Apply Phase..
Mon Feb 12 08:58:39 2018 - [info] 
Mon Feb 12 08:58:39 2018 - [info] -- Slave recovery on host 172.30.105.106(172.30.105.106:3306) started, pid: 79325. Check tmp log /data/masterha/app1/172.30.105.106_3306_20180212085820.log if it takes time..
saved_master_binlog_from_172.30.105.104_3306_20180212085820.binlog                              100%  126     2.7KB/s   00:00    
Mon Feb 12 08:58:41 2018 - [info] 
Mon Feb 12 08:58:41 2018 - [info] Log messages from 172.30.105.106 ...
Mon Feb 12 08:58:41 2018 - [info] 
Mon Feb 12 08:58:39 2018 - [info] Sending binlog..
Mon Feb 12 08:58:40 2018 - [info] scp from local:/data/masterha/app1/saved_master_binlog_from_172.30.105.104_3306_20180212085820.binlog to root@172.30.105.106:/data/masterha/app1/saved_master_binlog_from_172.30.105.104_3306_20180212085820.binlog succeeded.
Mon Feb 12 08:58:40 2018 - [info] Starting recovery on 172.30.105.106(172.30.105.106:3306)..
Mon Feb 12 08:58:40 2018 - [info]  Generating diffs succeeded.
Mon Feb 12 08:58:40 2018 - [info] Waiting until all relay logs are applied.
Mon Feb 12 08:58:40 2018 - [info]  done.
Mon Feb 12 08:58:40 2018 - [info] Getting slave status..
Mon Feb 12 08:58:40 2018 - [info] This slave(172.30.105.106)'s Exec_Master_Log_Pos equals to Read_Master_Log_Pos(mysql-bin.000010:12976817). No need to recover from Exec_Master_Log_Pos.
Mon Feb 12 08:58:40 2018 - [info] Connecting to the target slave host 172.30.105.106, running recover script..
Mon Feb 12 08:58:40 2018 - [info] Executing command: apply_diff_relay_logs --command=apply --slave_user='mhauser' --slave_host=172.30.105.106 --slave_ip=172.30.105.106  --slave_port=3306 --apply_files=/data/masterha/app1/saved_master_binlog_from_172.30.105.104_3306_20180212085820.binlog --workdir=/data/masterha/app1 --target_version=5.5.20-log --timestamp=20180212085820 --handle_raw_binlog=1 --disable_log_bin=0 --manager_version=0.55 --slave_pass=xxx
Mon Feb 12 08:58:40 2018 - [info] 
Applying differential binary/relay log files /data/masterha/app1/saved_master_binlog_from_172.30.105.104_3306_20180212085820.binlog on 172.30.105.106:3306. This may take long time...
Applying log files succeeded.
Mon Feb 12 08:58:40 2018 - [info]  All relay logs were successfully applied.
Mon Feb 12 08:58:40 2018 - [info]  Resetting slave 172.30.105.106(172.30.105.106:3306) and starting replication from the new master 172.30.105.105(172.30.105.105:3306)..
Mon Feb 12 08:58:40 2018 - [info]  Executed CHANGE MASTER.
Mon Feb 12 08:58:40 2018 - [info]  Slave started.
Mon Feb 12 08:58:41 2018 - [info] End of log messages from 172.30.105.106.
Mon Feb 12 08:58:41 2018 - [info] -- Slave recovery on host 172.30.105.106(172.30.105.106:3306) succeeded.
Mon Feb 12 08:58:41 2018 - [info] All new slave servers recovered successfully.
Mon Feb 12 08:58:41 2018 - [info] 
Mon Feb 12 08:58:41 2018 - [info] * Phase 5: New master cleanup phase..
Mon Feb 12 08:58:41 2018 - [info] 
Mon Feb 12 08:58:41 2018 - [info] Resetting slave info on the new master..
Mon Feb 12 08:58:41 2018 - [info]  172.30.105.105: Resetting slave info succeeded.
Mon Feb 12 08:58:41 2018 - [info] Master failover to 172.30.105.105(172.30.105.105:3306) completed successfully.
Mon Feb 12 08:58:41 2018 - [info] 

----- Failover Report -----

app1: MySQL Master failover 172.30.105.104 to 172.30.105.105 succeeded

Master 172.30.105.104 is down!

Check MHA Manager logs at manager for details.

Started manual(interactive) failover.
Invalidated master IP address on 172.30.105.104.
The latest slave 172.30.105.105(172.30.105.105:3306) has all relay logs for recovery.
Selected 172.30.105.105 as a new master.
172.30.105.105: OK: Applying all logs succeeded.
172.30.105.105: OK: Activated master IP address.
172.30.105.106: This host has the latest relay log events.
Generating relay diff files from the latest slave succeeded.
172.30.105.106: OK: Applying all logs succeeded. Slave started, replicating from 172.30.105.105.
172.30.105.105: Resetting slave info succeeded.
Master failover to 172.30.105.105(172.30.105.105:3306) completed successfully.
Mon Feb 12 08:58:41 2018 - [info] Sending mail..
Unknown option: conf
IN SCRIPT TEST====/sbin/ifconfig ens33:1 down==/sbin/ifconfig ens33:1 172.30.105.203/24===

Checking the Status of the script.. OK 
Mon Feb 12 08:39:44 2018 - [info]  OK.
Mon Feb 12 08:39:44 2018 - [warning] shutdown_script is not defined.
Mon Feb 12 08:39:44 2018 - [info] Set master ping interval 1 seconds.
Mon Feb 12 08:39:44 2018 - [info] Set secondary check script: /usr/bin/masterha_secondary_check -s slave1 -s master
Mon Feb 12 08:39:44 2018 - [info] Starting ping health check on 172.30.105.104(172.30.105.104:3306)..
Mon Feb 12 08:39:44 2018 - [info] Ping(SELECT) succeeded, waiting until MySQL doesn't respond..
Mon Feb 12 08:46:19 2018 - [info] Got terminate signal. Exit.


// 手动切换有两处需要输入yes
```


### 12、测试主数据库的在线切换：
&emsp;&emsp; 在许多情况下， 需要将现有的主服务器迁移到另外一台服务器上。 比如主服务器硬件故障，RAID 控制卡需要重建，将主服务器移到性能更好的服务器上等等。维护主服务器引起性能下降， 导致停机时间至少无法写入数据。 另外， 阻塞或杀掉当前运行的会话会导致主主之间数据不一致的问题发生。 MHA 提供快速切换和优雅的阻塞写入，这个切换过程只需要 0.5-2s 的时间，这段时间内数据是无法写入的。在很多情况下，0.5-2s 的阻塞写入是可以接受的。因此切换主服务器不需要计划分配维护时间窗口     

#### 12.1 在线切换的时候应用架构需要考虑以下两个问题：
- 自动识别master和slave的问题（master的机器可能会切换），如果采用了vip的方式，基本可以解决这个问题;   
- 负载均衡的问题（可以定义大概的读写比例，每台机器可承担的负载比例，当有机器离开集群时，需要考虑这个问题）；    

#### 12.2 切换的必备条件：
为了保证数据完全一致性，在最快的时间内完成切换，MHA的在线切换必须满足以下条件才会切换成功，否则会切换失败。
- 所有slave的IO线程都在运行
- 所有slave的SQL线程都在运行
- 所有的show slave status的输出中Seconds_Behind_Master参数小于或者等于running_updates_limit秒，如果在切换过程中不指定running_updates_limit,那么默认情况下running_updates_limit为1秒。
- 在master端，通过show processlist输出，没有一个更新花费的时间大于running_updates_limit秒。

#### 12.3 在线切换步骤如下：

##### 停止MHA Manager监控：
```ruby
[root@manager ~]# masterha_stop --conf=/etc/masterha/app1.cnf
```

##### 进行在线切换：
```ruby
[root@manager ~]# masterha_master_switch --conf=/etc/masterha/app1.cnf --master_state=alive --new_master_host=172.30.105.105 --new_master_port=3306 --orig_master_is_new_slave --running_updates_limit=10000 

参数说明：
// --orig_master_is_new_slave 切换时加上此参数是将原master变为slave节点，如果不加此参数，原来的 master 将不启动
// --running_updates_limit=10000,故障切换时,候选master 如果有延迟的话， mha 切换不能成功;
加上此参数表示延迟在此时间范围内都可切换（单位为s），但是切换的时间长短是由recover时relay日志的大小决定
```
[master_ip_online_change脚本下载](https://github.com/ZongYuWang/File/tree/master/File/MySQL/MySQL_MHA)         
`在线进行切换需要调用到master_ip_online_change这个脚本`      

```ruby
[root@manager ~]# chmod +x /usr/bin/master_ip_online_change

```

```ruby
[root@manager ~]# tail -f /data/masterha/manager.log 

Mon Feb 12 03:11:48 2018 - [info] MHA::MasterRotate version 0.55.
Mon Feb 12 03:11:48 2018 - [info] Starting online master switch..
Mon Feb 12 03:11:48 2018 - [info] 
Mon Feb 12 03:11:48 2018 - [info] * Phase 1: Configuration Check Phase..
Mon Feb 12 03:11:48 2018 - [info] 
Mon Feb 12 03:11:48 2018 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Mon Feb 12 03:11:48 2018 - [info] Reading application default configurations from /etc/masterha/app1.cnf..
Mon Feb 12 03:11:48 2018 - [info] Reading server configurations from /etc/masterha/app1.cnf..
Mon Feb 12 03:11:49 2018 - [info] Current Alive Master: 172.30.105.104(172.30.105.104:3306)
Mon Feb 12 03:11:49 2018 - [info] Alive Slaves:
Mon Feb 12 03:11:49 2018 - [info]   172.30.105.105(172.30.105.105:3306)  Version=5.5.20-log (oldest major version between slaves) log-bin:enabled
Mon Feb 12 03:11:49 2018 - [info]     Replicating from 172.30.105.104(172.30.105.104:3306)
Mon Feb 12 03:11:49 2018 - [info]     Primary candidate for the new Master (candidate_master is set)
Mon Feb 12 03:11:49 2018 - [info]   172.30.105.106(172.30.105.106:3306)  Version=5.5.20-log (oldest major version between slaves) log-bin:enabled
Mon Feb 12 03:11:49 2018 - [info]     Replicating from 172.30.105.104(172.30.105.104:3306)
Mon Feb 12 03:11:49 2018 - [info]     Not candidate for the new Master (no_master is set)

It is better to execute FLUSH NO_WRITE_TO_BINLOG TABLES on the master before switching. Is it ok to execute on 172.30.105.104(172.30.105.104:3306)? (YES/no): YES
Mon Feb 12 03:12:03 2018 - [info] Executing FLUSH NO_WRITE_TO_BINLOG TABLES. This may take long time..
Mon Feb 12 03:12:03 2018 - [info]  ok.
Mon Feb 12 03:12:03 2018 - [info] Checking MHA is not monitoring or doing failover..
Mon Feb 12 03:12:03 2018 - [info] Checking replication health on 172.30.105.105..
Mon Feb 12 03:12:03 2018 - [info]  ok.
Mon Feb 12 03:12:03 2018 - [info] Checking replication health on 172.30.105.106..
Mon Feb 12 03:12:03 2018 - [info]  ok.
Mon Feb 12 03:12:03 2018 - [info] 172.30.105.105 can be new master.
Mon Feb 12 03:12:03 2018 - [info] 
From:
172.30.105.104 (current master)
 +--172.30.105.105
 +--172.30.105.106

To:
172.30.105.105 (new master)
 +--172.30.105.106
 +--172.30.105.104

Starting master switch from 172.30.105.104(172.30.105.104:3306) to 172.30.105.105(172.30.105.105:3306)? (yes/NO): yes
Mon Feb 12 03:12:08 2018 - [info] Checking whether 172.30.105.105(172.30.105.105:3306) is ok for the new master..
Mon Feb 12 03:12:08 2018 - [info]  ok.
Mon Feb 12 03:12:08 2018 - [info] 172.30.105.104(172.30.105.104:3306): SHOW SLAVE STATUS returned empty result. To check replication filtering rules, temporarily executing CHANGE MASTER to a dummy host.
Mon Feb 12 03:12:08 2018 - [info] 172.30.105.104(172.30.105.104:3306): Resetting slave pointing to the dummy host.
Mon Feb 12 03:12:08 2018 - [info] ** Phase 1: Configuration Check Phase completed.
Mon Feb 12 03:12:08 2018 - [info] 
Mon Feb 12 03:12:08 2018 - [info] * Phase 2: Rejecting updates Phase..
Mon Feb 12 03:12:08 2018 - [info] 
Mon Feb 12 03:12:08 2018 - [info] Executing master ip online change script to disable write on the current master:
Mon Feb 12 03:12:08 2018 - [info]   /usr/bin/master_ip_online_change --command=stop --orig_master_host=172.30.105.104 --orig_master_ip=172.30.105.104 --orig_master_port=3306 --orig_master_user='mhauser' --orig_master_password='wangzongyu' --new_master_host=172.30.105.105 --new_master_ip=172.30.105.105 --new_master_port=3306 --new_master_user='mhauser' --new_master_password='wangzongyu'  
Unknown option: orig_master_password
Unknown option: new_master_password
Mon Feb 12 03:12:08 2018 861438 Set read_only on the new master.. ok.
Mon Feb 12 03:12:08 2018 865131 Set read_only=1 on the orig master.. ok.
Disabling the VIP on old master: 172.30.105.104 
Mon Feb 12 03:12:09 2018 232295 Killing all application threads..
Mon Feb 12 03:12:09 2018 232349 done.
Mon Feb 12 03:12:09 2018 - [info]  ok.
Mon Feb 12 03:12:09 2018 - [info] Locking all tables on the orig master to reject updates from everybody (including root):
Mon Feb 12 03:12:09 2018 - [info] Executing FLUSH TABLES WITH READ LOCK..
Mon Feb 12 03:12:09 2018 - [info]  ok.
Mon Feb 12 03:12:09 2018 - [info] Orig master binlog:pos is mysql-bin.000010:279.
Mon Feb 12 03:12:09 2018 - [info]  Waiting to execute all relay logs on 172.30.105.105(172.30.105.105:3306)..
Mon Feb 12 03:12:09 2018 - [info]  master_pos_wait(mysql-bin.000010:279) completed on 172.30.105.105(172.30.105.105:3306). Executed 0 events.
Mon Feb 12 03:12:09 2018 - [info]   done.
Mon Feb 12 03:12:09 2018 - [info] Getting new master's binlog name and position..
Mon Feb 12 03:12:09 2018 - [info]  mysql-bin.000003:107
Mon Feb 12 03:12:09 2018 - [info]  All other slaves should start replication from here. Statement should be: CHANGE MASTER TO MASTER_HOST='172.30.105.105', MASTER_PORT=3306, MASTER_LOG_FILE='mysql-bin.000003', MASTER_LOG_POS=107, MASTER_USER='repuser', MASTER_PASSWORD='xxx';
Mon Feb 12 03:12:09 2018 - [info] Executing master ip online change script to allow write on the new master:
Mon Feb 12 03:12:09 2018 - [info]   /usr/bin/master_ip_online_change --command=start --orig_master_host=172.30.105.104 --orig_master_ip=172.30.105.104 --orig_master_port=3306 --orig_master_user='mhauser' --orig_master_password='wangzongyu' --new_master_host=172.30.105.105 --new_master_ip=172.30.105.105 --new_master_port=3306 --new_master_user='mhauser' --new_master_password='wangzongyu'  
Unknown option: orig_master_password
Unknown option: new_master_password
Mon Feb 12 03:12:09 2018 312079 Set read_only=0 on the new master.
Enabling the VIP - 172.30.105.203/24 on the new master - 172.30.105.105 
Mon Feb 12 03:12:09 2018 - [info]  ok.
Mon Feb 12 03:12:09 2018 - [info] 
Mon Feb 12 03:12:09 2018 - [info] * Switching slaves in parallel..
Mon Feb 12 03:12:09 2018 - [info] 
Mon Feb 12 03:12:09 2018 - [info] -- Slave switch on host 172.30.105.106(172.30.105.106:3306) started, pid: 60867
Mon Feb 12 03:12:09 2018 - [info] 
Mon Feb 12 03:12:10 2018 - [info] Log messages from 172.30.105.106 ...
Mon Feb 12 03:12:10 2018 - [info] 
Mon Feb 12 03:12:09 2018 - [info]  Waiting to execute all relay logs on 172.30.105.106(172.30.105.106:3306)..
Mon Feb 12 03:12:09 2018 - [info]  master_pos_wait(mysql-bin.000010:279) completed on 172.30.105.106(172.30.105.106:3306). Executed 0 events.
Mon Feb 12 03:12:09 2018 - [info]   done.
Mon Feb 12 03:12:09 2018 - [info]  Resetting slave 172.30.105.106(172.30.105.106:3306) and starting replication from the new master 172.30.105.105(172.30.105.105:3306)..
Mon Feb 12 03:12:09 2018 - [info]  Executed CHANGE MASTER.
Mon Feb 12 03:12:09 2018 - [info]  Slave started.
Mon Feb 12 03:12:10 2018 - [info] End of log messages from 172.30.105.106 ...
Mon Feb 12 03:12:10 2018 - [info] 
Mon Feb 12 03:12:10 2018 - [info] -- Slave switch on host 172.30.105.106(172.30.105.106:3306) succeeded.
Mon Feb 12 03:12:10 2018 - [info] Unlocking all tables on the orig master:
Mon Feb 12 03:12:10 2018 - [info] Executing UNLOCK TABLES..
Mon Feb 12 03:12:10 2018 - [info]  ok.
Mon Feb 12 03:12:10 2018 - [info] Starting orig master as a new slave..
Mon Feb 12 03:12:10 2018 - [info]  Resetting slave 172.30.105.104(172.30.105.104:3306) and starting replication from the new master 172.30.105.105(172.30.105.105:3306)..
Mon Feb 12 03:12:10 2018 - [info]  Executed CHANGE MASTER.
Mon Feb 12 03:12:10 2018 - [info]  Slave started.
Mon Feb 12 03:12:10 2018 - [info] All new slave servers switched successfully.
Mon Feb 12 03:12:10 2018 - [info] 
Mon Feb 12 03:12:10 2018 - [info] * Phase 5: New master cleanup phase..
Mon Feb 12 03:12:10 2018 - [info] 
Mon Feb 12 03:12:10 2018 - [info]  172.30.105.105: Resetting slave info succeeded.
Mon Feb 12 03:12:10 2018 - [info] Switching master to 172.30.105.105(172.30.105.105:3306) completed successfully.
IN SCRIPT TEST====/sbin/ifconfig ens33:1 down==/sbin/ifconfig ens33:1 172.30.105.203/24===

Checking the Status of the script.. OK 
Mon Feb 12 03:03:55 2018 - [info]  OK.
Mon Feb 12 03:03:55 2018 - [warning] shutdown_script is not defined.
Mon Feb 12 03:03:55 2018 - [info] Set master ping interval 1 seconds.
Mon Feb 12 03:03:55 2018 - [info] Set secondary check script: /usr/bin/masterha_secondary_check -s slave1 -s master
Mon Feb 12 03:03:55 2018 - [info] Starting ping health check on 172.30.105.104(172.30.105.104:3306)..
Mon Feb 12 03:03:55 2018 - [info] Ping(SELECT) succeeded, waiting until MySQL doesn't respond..
Mon Feb 12 03:09:23 2018 - [info] Got terminate signal. Exit.
```

#### 12.4 MHA在线切换的大概过程：
- 检测复制设置和确定当前主服务器       
- 确定新的主服务器       
- 阻塞写入到当前主服务器       
- 等待所有从服务器赶上复制        
- 授予写入到新的主服务器      
- 重新设置从服务器      

#### 12.5 修复宕机的Master
`但是上面的在线切换已经把老的主库设置为从库了，指向了172.30.105.105`
```ruby
mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.30.105.105
                  Master_User: repuser
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000003
          Read_Master_Log_Pos: 107
               Relay_Log_File: relay-bin.000002
                Relay_Log_Pos: 253
        Relay_Master_Log_File: mysql-bin.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes

```
&emsp;&emsp; 通常情况下自动切换以后，原master可能已经废弃掉，待原master主机修复后，如果数据完整的情况下，可能想把原来master重新作为新主库的slave，这时我们可以借助当时自动切换时刻的MHA日志来完成对原master的修复。

```ruby
[root@manager ~]# grep -i "All other slaves should start" /data/masterha/manager.log

Sun Feb 11 23:30:52 2018 - [info]  All other slaves should start replication from here. Statement should be: CHANGE MASTER TO MASTER_HOST='172.30.105.105', MASTER_PORT=3306, MASTER_LOG_FILE='mysql-bin.000003', MASTER_LOG_POS=107, MASTER_USER='repuser', MASTER_PASSWORD='xxx';
Mon Feb 12 00:32:01 2018 - [info]  All other slaves should start replication from here. Statement should be: CHANGE MASTER TO MASTER_HOST='172.30.105.105', MASTER_PORT=3306, MASTER_LOG_FILE='mysql-bin.000003', MASTER_LOG_POS=107, MASTER_USER='repuser', MASTER_PASSWORD='xxx';

```
&emsp;&emsp; 获取上述信息以后，就可以直接在修复后的master上执行change master to相关操作，重新作为从库了

##### 以上的三种切换方式无论使用哪种，VIP也都应该自动跳转到新的Master