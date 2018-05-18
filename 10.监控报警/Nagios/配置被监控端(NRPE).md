## 配置被监控端(NRPE)  

### 1、NRPE简介：

&emsp;&emsp;Nagios监控远程主机的方法有多种，其方式包括SNMP、NRPE、SSH和NCSA等。这里介绍其通过NRPE监控远程Linux主机的方式。
&emsp;&emsp;NRPE（Nagios Remote Plugin Executor）是用于在远端服务器上运行检测命令的守护进程，它用于让Nagios监控端基于安装的方式触发远端主机上的检测命令，并将检测结果输出至监控端。而其执行的开销远低于基于SSH的检测方式，而且检测过程并不需要远程主机上的系统帐号等信息，其安全性也高于SSH的检测方式。

![](https://github.com/ZongYuWang/image/blob/master/Nagios/Nagios-NRPE1.png)

### 2、配置监控端

#### 2.1 安装NRPE
```ruby
# tar -zxvf nrpe-2.12.tar.gz
# cd nrpe-2.12
# ./configure --with-nrpe-user=nagios \
     --with-nrpe-group=nagios \
     --with-nagios-user=nagios \
     --with-nagios-group=nagios \
     --enable-command-args \
     --enable-ssl
# make all
# make install-plugin
```

#### 2.2 check_nrpe语法：
```ruby
通过NRPE监控远程Linux主机要使用chech_nrpe插件进行，其语法格式如下：
check_nrpe -H <host> [-n] [-u] [-p <port>] [-t <timeout>] [-c <command>] [-a <arglist...>]
【说明】# ls /usr/local/nagios/libexec/会多出来一个check_nrpe插件

```

##### 使用示例1：
&emsp;&emsp;定义监控远程Linux主机swap资源的命令：
```ruby
	define command
	{
		command_name check_swap_nrpe
		command_line $USER1$/check_nrpe –H "$HOSTADDRESS$" -c "check_swap"
	}
【说明】command_line  -c "check_swap"是在被监控端的nrpe.cfg中定义的

command[check_load]=/usr/local/nagios/libexec/check_load -w 30,25,10 -c 55,40,20
command[check_men]=/usr/local/nagios/libexec/check_memory.pl -f -w 10 -c 3
command[check_Rootdisk]=/usr/local/nagios/libexec/check_disk -w %5 -c %1 -p /
command[check_Partiondisk]=/usr/local/nagios/libexec/check_disk -w %5 -c %1 -p /mnt/datadisk1
command[check_swap]=/usr/local/nagios/libexec/check_swap -w 20 -c 10
command[check_iostat]=/usr/local/nagios/libexec/check_iostat -d xvde -w 2000 -c 4000 
command[check_cpu]=/usr/local/nagios/libexec/check_cpu.sh  -w 70 -c 90

定义远程Linux主机的swap资源：
	define service
	{
		use generic-service
		host_name linuxserver1,linuxserver2
		hostgroup_name linux-servers
		service_description SWAP
		check_command check_swap_nrpe
		normal_check_interval 30
	}
	
```

##### 使用示例2：
&emsp;&emsp;如果希望上面的command定义更具有通用性，那么上面的定义也可以修改为如下：

```ruby
定义监控远程Linux主机的命令：
	define command
	{
		command_name check_nrpe
		command_line $USER1$/check_nrpe –H "$HOSTADDRESS$" -c $ARG1$
	}

定义远程Linux主机的swap资源：
	define service
	{
		use generic-service
		host_name linuxserver1,linuxserver2
		hostgroup_name linux-servers
		service_description SWAP
		check_command check_nrpe!check_swap
		normal_check_interval 30
	}
// 每个！后面代表传递的参数，此例子中定义主机中传递的参数使用的是 check_swap，同样check_swap也是在nrpe.cfg中定义的
// 因为command中使用-c定义了宏（变量），所以也可以这些写：check_command check_nrpe!check_users
// -c后面接的是命令，也就是nrpe.cfg中定义的command后面的命令名称

```

##### 使用示例3：
&emsp;&emsp;如果还希望在监控远程Linux主机时还能向其传递参数，则可以使用类似如下方式进行：
```ruby
定义监控远程Linux主机disk资源的命令：
	define command
	{
		command_name check_swap_nrpe
		command_line $USER1$/check_nrpe –H "$HOSTADDRESS$" -c "check_swap" -a $ARG1$ $ARG2$
	}

定义远程Linux主机的swap资源：
	define service
	{
		use generic-service
		host_name linuxserver1,linuxserver2
		hostgroup_name linux-servers
		service_description SWAP
		check_command check_swap_nrpe!20!10
		normal_check_interval 30
	}
// 示例2、示例3都是接收传递参数的形式，示例3中-c 明确使用check_swap命令，而后面的参数是check_swap可以接收的参数
// 需要在被监控端的nrpe.cfg中使用check_swap的参数命令形式：
// command[check_swap]=/usr/local/nagios/libexec/check_disk -w $ARG1$ -c $ARG2$
// ,$s/windows-server/linux-server/g 将全文的windows-server替换为linux-server


# vim /usr/local/nagios/etc/objects/linux.cfg
define host{
        use                     linux-server
        host_name               Export-seller-WEB
        alias                   Export-seller-WEB
        address         192.168.1.105
        }

define service {
        use                     generic-service,svr-pnp
        host_name               Export-seller-WEB
        service_description     SWAP
        check_command           check_swap_nrpe!20!10
        normal_check_interval   30
}

# vim /usr/local/nagios/etc/nagios.cfg
cfg_file=/usr/local/nagios/etc/objects/linux.cfg

```

##### 使用示例4：
```ruby
# vim commands.cfg
define command{
	   command_name check_nrpe
	   command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
}
【说明】定义命令-c使用参数，表示-c后面可以接被监控端nrpe.cfg中定义的任何的command的名称
# vim linux.cfg 
define service {
        use                     generic-service,svr-pnp
        host_name               Export-seller-WEB
        service_description     check users
        check_command           check_nrpe!check_users
}

在定义服务中，check_command中使用check_nrpe!后面接了一个check_users的命令名称
检查配置：
# /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg

```

### 3、配置被监控端
```ruby

# 安装相关软件包：
# yum -y groupinstall "Development Tools" "Development Libraries" openssl*
```
#### 3.1 添加nagios用户：
```ruby
# useradd -s /sbin/nologin nagios
```

#### 3.2 NRPE依赖于nagios-plugins：
```ruby
# tar zxf nagios-plugins-1.4.15.tar.gz 
# cd nagios-plugins-1.4.15
# ./configure --with-nagios-user=nagios --with-nagios-group=nagios
# make all
# make instal
```

#### 3.3 安装NRPE：

```ruby
# tar -zxvf nrpe-2.15.tar.gz
# cd nrpe-2.15.tar.gz
# ./configure --with-nrpe-user=nagios \
     --with-nrpe-group=nagios \
     --with-nagios-user=nagios \
     --with-nagios-group=nagios \
     --enable-command-args \
     --enable-ssl
# make all
# make install-plugin
# make install-daemon
# make install-daemon-config

```

#### 3.4  配置NRPE：

```ruby
# vim /usr/local/nagios/etc/nrpe.cfg

log_facility=daemon
pid_file=/var/run/nrpe.pid
server_address=172.16.100.11         #server_address=127.0.0.1
server_port=5666
nrpe_user=nagios
nrpe_group=nagios
allowed_hosts=192.168.1.111         #Nagios Server端-监控端
command_timeout=60
connection_timeout=300
debug=0

【说明】allowed_hosts指令用于定义本机所允许的监控端的IP地址。
```

#### 3.5 启动NRPE：
```ruby
# /usr/local/nagios/bin/nrpe -c /usr/local/nagios/etc/nrpe.cfg -d
【说明】这种启动方式不利于开机自动启的设置，为了便于NRPE服务的启动，可以将如下内容定义为/etc/init.d/nrped脚本：
#!/bin/bash
# chkconfig: 2345 88 12
# description: NRPE DAEMON

NRPE=/usr/local/nagios/bin/nrpe
NRPECONF=/usr/local/nagios/etc/nrpe.cfg

case "$1" in
	start)
		echo -n "Starting NRPE daemon..."
		$NRPE -c $NRPECONF -d
		echo " done."
		;;
	stop)
		echo -n "Stopping NRPE daemon..."
		pkill -u nagios nrpe
		echo " done."
	;;
	restart)
		$0 stop
		sleep 2
		$0 start
		;;
	*)
		echo "Usage: $0 start|stop|restart"
		;;
	esac
exit 0

# chmod +x /etc/init.d/nrped
【说明】或者也可以在/etc/xinetd.d目录中创建nrpe文件，使其成为一个基于非独立守护进程的服务，文件内容如下：
service nrpe
{
	flags = REUSE
	socket_type = stream
	wait = no
	user = nagios
	group = nagios
	server = /usr/local/nagios/bin/nrpe
	server_args = -c /etc/nagios/nrpe.cfg -i
	log_on_failure += USERID
	disable = no
}
```
#### 3.6 配置允许远程主机监控的对象：
```ruby
在被监控端，可以通过NRPE监控的服务或资源需要通过nrpe.cfg文件使用命令进行定义，
定义命令的语法格式为：command[<command_name>]=<command_to_execute>。比如：

command[check_rootdisk]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /
command[check_swap]=/usr/local/nagios/libexec/check_disk -w 40% -c 20%
command[check_sensors]=/usr/local/nagios/libexec/check_sensors
command[check_users]=/usr/local/nagios/libexec/check_users -w 10 -c 20
command[check_load]=/usr/local/nagios/libexec/check_load -w 10,8,5 -c 20,18,15
command[check_zombies]=/usr/local/nagios/libexec/check_procs -w 5 -c 10 -s Z
command[check_all_procs]=/usr/local/nagios/libexec/check_procs -w 150 -c 200

```
### 4、配置被监控端监控项
#### 4.1 监控硬盘I/O：
```ruby

【说明】Params-Validate-0.91、Class-Accessor-0.31、Config-Tiny-2.14、Math-Calc-Units-1.07、Nagios-Plugin-0.37、Regexp-Common-2013031301都是iostat所需要的软件

监控端配置：
# cd Params-Validate-0.91
# perl Makefile.PL
# make
# make install

# cd Class-Accessor-0.31
# perl Makefile.PL
# make
# make install

# cd Config-Tiny-2.14
# perl Makefile.PL
# make
# make install

# cd Math-Calc-Units-1.07
# perl Makefile.PL
# make
# make install

# cd Regexp-Common-2013031301
# perl Makefile.PL
# make
# make install

# cd Nagios-Plugin-0.37
# perl Makefile.PL
# make
# make install


被监控端配置：
# cd Params-Validate-0.91
# perl Makefile.PL
# make
# make install

# cd Class-Accessor-0.31
# perl Makefile.PL
# make
# make install

# cd Config-Tiny-2.14
# perl Makefile.PL
# make
# make install

# cd Math-Calc-Units-1.07
# perl Makefile.PL
# make
# make install

# cd Regexp-Common-2013031301
# perl Makefile.PL
# make
# make install

# cd Nagios-Plugin-0.37
# perl Makefile.PL
# make
# make install

# yum install sysstat -y

重新启动nrpe 
cp nnrped.sh /etc/init.d/nagios_client
chmod 755 /etc/init.d/nrped.sh
/etc/init.d/nrped start
chkconfig nrped on

[root@localhost ~]# /usr/local/nagios/libexec/check_iostat -d sda1 -w 1000 -c 2000
OK - I/O stats tps=0.00 KB_read/s=0.03 KB_written/s=0.00 | 'tps'=0.00; 'KB_read/s'=0.03; 'KB_written/s'=0.00;

```
#### 4.2 监控主机存活状态：
```ruby

Ping的检测：
-c 次数
-w 执行时间，也就是整个ping过程的执行时间 ，单位是秒
【说明】ping 172.30.105.112 -c 20 -w 1 执行时间是1秒，那么时间到了，20包没发送完也就结束了
-i ping包之间的时间间隔

check_ping -H <host_address> -w <wrta>,<wpl>% -c <crta>,<cpl>%   [-p packets] [-t timeout] [-4|-6]
[root@localhost ~]# /usr/local/nagios/libexec/check_ping 172.30.105.115 -w 3000.0,80% -c 5000.0,100% -p 5
PING OK - Packet loss = 0%, RTA = 1.06 ms|rta=1.059000ms;3000.000000;5000.000000;0.000000 pl=0%;80;100;0
【说明】The check must result in a 100% packet loss or 5 second (5000ms) round trip


使用NRPE的使用示例2方式
command[check-host-alive]=/usr/local/nagios/libexec/check_ping -w 3000.0,80% -c 5000.0,100% -p 5

```
#### 4.3 check_load检测：
```ruby
例如check_load -w 15,10,5 -c 30,25,20这个命令的意义如下
当1分钟多于15个进程等待,5分钟多于10个,15分钟多于5个则为warning状态
当1分钟多于30个进程等待,5分钟多于25个,15分钟多于20个则为critical状态

check_load表示检查负载，是通过系统命令top显示，check_load并不是cpu 的负载，也不是IO的负载。
check_load是检查系统正在运行的任务数+等待的任务数。/proc/loadavg是这里表示的负载。
cat /proc/loadavg

command[check_load]=/usr/local/nagios/libexec/check_load -w 15,10,5 -c 30,25,20
command[check_men]=/usr/local/nagios/libexec/check_memory.pl -f -w 10 -c 3
command[check_disk]=/usr/local/nagios/libexec/check_disk -w %5 -c %1 -p /
command[check_swap]=/usr/local/nagios/libexec/check_swap -w 20 -c 10
command[check_iostat]=/usr/local/nagios/libexec/check_iostat -d sda -w 1000 -c 2000 

```
#### 4.4 监控CPU使用情况：
```ruby

[root@localhost ~]# /usr/local/nagios/libexec/check_cpu.sh                        
OK: CPU=3.92 | used=3.92;;;; system=2.05;;;; user=1.09;;;; nice=0;;;; iowait=.38;;;; irq=.04;;;; softirq=.34;;;;

[root@localhost ~]# /usr/local/nagios/libexec/check_cpu.sh -w 0 -c 0
CRITICAL: CPU=2.70 | used=2.70;0;0;; system=1.01;;;; user=.90;;;; nice=0;;;; iowait=.56;;;; irq=0;;;; softirq=.22;;;;
```

#### 4.5 监控内存使用情况：
```ruby
# ./check_memory.pl -h            
usage:
 check_mem.pl -<f|u> -w <warnlevel> -c <critlevel>
options:
 -f           Check FREE memory
 -u           Check USED memory
 -C           Count OS caches as FREE memory
 -w PERCENT   Percent free/used when to warn
 -c PERCENT   Percent free/used when critical
 
[root@localhost ~]# /usr/local/nagios/libexec/check_memory.pl -u -w 50 -c 90
CRITICAL - 91.2% (915680 kB) used!|TOTAL=1004412KB;;;; USED=915680KB;;;; FREE=88732KB;;;; CACHES=185056KB;;;;

```

#### 4.6 监控网络流量：
```ruby
监控端和被监控端都需要安装 # yum install net-snmp* bc

被监控端配置：
# vim /etc/snmp/snmpd.conf
com2sec notConfigUser  default       public
group   notConfigGroup v1           notConfigUser
group   notConfigGroup v2c           notConfigUser
view    systemview    included   .1.3.6.1.2.1.1
view    systemview    included   .1.3.6.1.2.1.25.1.1
access  notConfigGroup ""      any       noauth    exact  all none none
view all    included  .1                               80
syslocation Unknown (edit /etc/snmp/snmpd.conf)
syscontact Root <root@localhost> (configure /etc/snmp/snmp.local.conf)
dontLogTCPWrappersConnects yes

监控端配置同被监控端：
com2sec notConfigUser   172.30.105.115         public

[root@localhost ~]# /usr/local/nagios/libexec/check_traffic.sh -V 2c -C public -H 172.30.105.115 -L
List Interface for host 172.30.105.115.
Interface index 1 orresponding to  lo
Interface index 2 orresponding to  eth0

[root@localhost ~]# /usr/local/nagios/libexec/check_traffic.sh -V 2c -C public -H 172.30.105.115 -I 2 -w 200,300 -c 400,500 -K -B
OK - It's the first time for this plugins run. We'll get the data from the next time.
【说明】将“default”改为你想哪台机器可以看到你的snmp信息（172.30,105.112）就改成这个IP，不改表示所有机器充许

【说明】I后面的数字是上面命令中出现的网卡标识（Interface index 2）；-V SNMP的版本； -C 团体名 ；-w warning ； -c critical；定义in和out值分别超过200K、300K警告，超过400K,500k严重警告
【说明】第一次运行没有输出，30s后第二次运行才有输出
[root@localhost ~]# /usr/local/nagios/libexec/check_traffic.sh -V 2c -C public -H 172.30.105.115 -I 2 -w 200,300 -c 400,500 -K -B
OK - The Traffic In is 0.0KB, Out is 0.0KB, Total is 0.0KB. The Check Interval is 217s |In=0.0KB;200;400;0;0 Out=0.0KB;300;500;0;0 Total=0.0KB;500;900;0;0 Interval=217s;1200;1800;0;0
【说明】In是0.0KB，Out是0.0KB，总共是0.0KB；In是0.0KB，warning是200KB，critical是400KB；Out是0.0KB，warning是300KB，Critical是500KB

监控端配置：
增加nagios的command.cfg
# 'check_traffic' command definition
define command{
command_name    check_eth0_traffic
command_line    $USER1$/check_traffic.sh -V 2c -C 团体名称 -H $HOSTADDRESS$ -I $ARG1$ -w $ARG2$ -c $ARG3$ -K -B
}
【说明】-I 第几个网卡  -K –B设置的报警参数数值是KB，不是MB

在监控端localhost.cfg中定义主机
vim /usr/local/nagios/etc/objects/localhost.cfg
define host {
host_name      db_31
alias               centos-107
address         192.168.10.107
notification_options    d,u,r
check_interval  1
max_check_attempts      2
contact_groups  admins
notification_interval   10
notification_period     24x7
}

在监控端localhost.cfg中定义服务
vim /usr/local/nagios/etc/objects/localhost.cfg
define service {
use                                        generic-service      
host_name                           db_31
service_description              check_eth0_traffic
check_period                        24x7
normal_check_interval         2
retry_check_interval             1
max_check_attempts           5
notification_period                 24x7
notification_options               w,u,c,r
check_command                  check_nrpe!check_traffic!2!4000,5000!6000,7000;该处设置的是kB，不是MB      //与command文件中的名字一样

检查Nagios配置文件
/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
重启Nagios服务
service nagios restart
【说明】首先保证系统防火墙开放5666端口号
```
#### 4.7 监控MySQL（官网自带的check_mysql和check_mysql_health）：
```ruby

监控端IP：172.30.105.112
被监控端IP：172.30.105.115（111）

check_mysql 插件使用：
[root@localhost ~]# /usr/local/nagios/libexec/check_mysql -H 172.30.105.115 -u root -d nagios -p wangzongyu
Uptime: 88029  Threads: 1  Questions: 119  Slow queries: 0  Opens: 89  Flush tables: 1  Open tables: 24  Queries per second avg: 0.1|Connections=13c;;; Open_files=48;;; Open_tables=24;;; Qcache_free_memory=0;;; Qcache_hits=0c;;; Qcache_inserts=0c;;; Qcache_lowmem_prunes=0c;;; Qcache_not_cached=0c;;; Qcache_queries_in_cache=0;;; Queries=119c;;; Questions=119c;;; Table_locks_waited=0c;;; Threads_connected=1;;; Threads_running=1;;; Uptime=88029c;;;


check_mysql_health插件使用：
【说明】check_mysql_health插件比起官方的check_mysql插件功能更为强大，check_mysql_health不但能监控MySQL是否正常运行，还能监控MySQL主从、MySQL连接数情况、MySQL慢查询等多种监控指标

在监控端配置：
# yum install -y perl-DBD-MySQL perl-DBD-Pg && echo ok
# cd /nagios_soft/
# tar xvf check_mysql_health-2.2.2.tar.gz 
# cd check_mysql_health-2.2.2
# ./configure --prefix=/usr/local/nagios && echo ok
# make && make install && echo ok
# cd /usr/local/nagios/libexec/
# chmod 755 check_mysql_health 
# chown nagios.nagios check_mysql_health 

[root@localhost ~]# /usr/local/nagios/libexec/check_mysql_health -H 172.30.105.115 -u root -d nagios -mode threads-connected
CRITICAL - cannot connect to information_schema. Please specify hostname, username and password or a .cnf file
--hostname　定义被监控主机的IP或机器名
--port　　　  定义被监控主机上MySQL的运行端口
--username　定义被监控主机上MySQL的用户名
--password　 定义被监控主机上MySQL的密码
--mode　　　定义被监控主机上MySQL的监控指标

被监控端配置(建议被监控端不要让监控端使用root连接，最好新建一个mysql用户)：
mysql> update mysql.user set Password = PASSWORD('nagios') where user='nagios';
mysql> grant all on *.* to nagios@172.20.105.112 identified by "nagios";
mysql> grant all on *.* to nagios@127.0.0.1 identified by "nagios";
mysql> flush privileges;
监控端的配置：
vi /usr/local/nagios/etc/objects/commands.cfg
define command{
        command_name check_mysql_health
        command_line $USER1$/check_mysql_health --hostname $ARG1$ --port $ARG2$ --username $ARG3$ --password $ARG4$ --mode $ARG5$
        }

#监控MySQL连接时间
define service{
use generic-service ; Name of service template to use
host_name mysql-master-1
service_description check_mysql_connection_time
check_command check_mysql_health!192.168.163.130!3306!nagios!nagios!connection-time
notifications_enabled 1
}

#监控MySQL连接数
define service{
use generic-service ; Name of service template to use
host_name mysql-master-1
service_description check_mysql_threads_connected
check_command check_mysql_health!192.168.163.130!3306!nagios!nagios!threads-connected
notifications_enabled 1
}

#监控MySQL慢查询情况
define service{
use generic-service ; Name of service template to use
host_name mysql-master-1
service_description check_mysql_slow_queries
check_command check_mysql_health!192.168.163.130!3306!nagios!nagios!slow-queries
notifications_enabled 1
}

#监控MySQL锁表情况
define service{
use generic-service ; Name of service template to use
host_name mysql-master-1
service_description check_mysql_table_lock_contention
check_command check_mysql_health!192.168.163.130!3306!nagios!nagios!table-lock-contention
notifications_enabled 1
}

check_mysql_health 参数说明：
# ./check_mysql_health 

  Usage:
    check_mysql_health [-v] [-t <timeout>] [[--hostname <hostname>] 
        [--port <port> | --socket <socket>]
        --username <username> --password <password>] --mode <mode>
        [--method mysql]
    check_mysql_health [-h | --help]
    check_mysql_health [-V | --version]

  Options:
    --hostname
       the database server's hostname
    --port
       the database's port. (default: 3306)
    --socket
       the database's unix socket.
    --username
       the mysql db user
    --password
       the mysql db user's password
    --database
       the database's name. (default: information_schema)
    --replication-user
       the database's replication user name (default: replication)
    --warning
       the warning range
    --critical
       the critical range
    --mode
       the mode of the plugin. select one of the following keywords:
      connection-time         // 连接到服务器上的时间
      uptime         // 服务器运行的时间
      threads-connected             // 当前连接到数据库的连接数
      threadcache-hitrate         // 线程缓存命中率
      slave-lag slave        // 落后master多少时间
      slave-io-running slave            // 复制是否正常
      slave-sql-running slave        // 复制是否正常
      qcache-hitrate 查询缓存命中率   //这个命中率越高，表明服务器的select查询性能越好。
      qcache-lowmem-prunes             // 增大query_cache_size的值， 以减小lowmem，增加缓存命中率。
      keycache-hitrate (MyISAM key cache hitrate ：key缓存命中率)        // 如果命中率过低，则调大key_buffer_size
      bufferpool-hitrate (InnoDB buffer pool hitrate：innodb缓冲池命中率)
      bufferpool-wait-free (InnoDB buffer pool waits for clean page available：innodb的缓存池等等清理页。)
      log-waits (InnoDB log waits because of a too small log buffer：因为太小log缓冲区导致innodb log等待。)
      tablecache-hitrate         // 表缓存命中率
      table-lock-contention         // 表锁率
      table_locks_waited         // 不能立即获得的表的锁表次数。
      table_locak_waited        // 立即获得的表的锁表次数。小于1%较优，如果1%需要引起注意，大于3%性能有问题。
      index-usage         // 索引使用情况
      tmp-disk-tables (Percent of temp tables created on disk：建立的临时表)
      slow-queries         // 慢查询
      long-running-procs         // 长时间运行的进程
      cluster-ndbd-running ndb    // 节点运行状况
      sql     // 返回一个数字的任何SQL语句

    --name
       the name of something that needs to be further specified,
       currently only used for sql statements
    --name2
       if name is a sql statement, this statement would appear in
       the output and the performance data. This can be ugly, so 
       name2 can be used to appear instead.
    --regexp
       if this parameter is used, name will be interpreted as a 
       regular expression.
    --units
       one of %, KB, MB, GB. This is used for a better output of mode=sql
       and for specifying thresholds for mode=tablespace-free
    --labelformat
       one of pnp4nagios (which is the default) or groundwork.
       It is used to shorten performance data labels to 19 characters.
       
[root@localhost ~]# /usr/local/nagios/libexec/check_mysql_health -H 172.30.105.115 -u root -d nagios -p wangzongyu -mode threads-connected
OK - 1 client connection threads | threads_connected=1;10;20
[root@localhost ~]# /usr/local/nagios/libexec/check_mysql_health -H 172.30.105.115 -u root -d nagios -p wangzongyu -mode slow-queries     
OK - 0 slow queries in 1493268930 seconds (0.00/sec) | slow_queries_rate=0.00%;0.1;1
[root@localhost ~]# /usr/local/nagios/libexec/check_mysql_health -H 172.30.105.115 -u root -d nagios -p wangzongyu -mode table-lock-contention
OK - table lock contention 0.00% | tablelock_contention=0.00%;1;2 tablelock_contention_now=0.00%
[root@localhost ~]# /usr/local/nagios/libexec/check_mysql_health -H 172.30.105.115 -u root -d nagios -p wangzongyu -mode connection-time      
OK - 0.08 seconds to connect as root | connection_time=0.0842s;1;5

```

#### 4.8 日志监控：
```ruby

# yum install ntp
# ntpdate cn.pool.ntp.org
# tar xvf check_logfiles-3.8.0.2.tar.gz 

# ./configure --prefix=/usr/local/nagios --with-nagios-user=nagios --with-nagios-group=nagios --with-perl=/usr/bin/perl --with-gzip=/usr/bin/gzip --with-trusted-path=/sbin:/usr/sbin:/usr/local/sbin:/bin:/usr/bin:/usr/local/nagios/libexec --with-seekfiles-dir=/tmp --with-protocols-dir=/tmp

# wget http://ftp.gnu.org/gnu/autoconf/autoconf-2.69.tar.gz 
# tar xvf autoconf-2.69.tar.gz
# cd autoconf-2.69
# ./configure --prefix=/usr/
# make && make install
【注意】一定要安装在/usr下，否则编译check_logfile时不会使用新版的autoconf

# make && make install    //编译logfile
【说明】安装完成后check_logfile插件将安装到/usr/local/nagios/libexec/下
# chown nagios.nagios /usr/local/nagios/libexec/check_logfiles 


# /usr/local/nagios/libexec/check_logfiles -h
This Nagios Plugin comes with absolutely NO WARRANTY. You may use
it on your own risk!
Copyright by ConSol Software GmbH, Gerhard Lausser.

This plugin looks for patterns in logfiles, even in those who were rotated
since the last run of this plugin.

You can find the complete documentation at 
http://labs.consol.de/nagios/check_logfiles/

Usage: check_logfiles [-t timeout] -f <configfile>

The configfile looks like this:

$seekfilesdir = '/opt/nagios/var/tmp';    //  写状态信息的目录，这里面记录已经检查过的日志内容，相当于历史记录  
# where the state information will be saved.

$protocolsdir = '/opt/nagios/var/tmp';   //  写协议信息的目录，这里面记录日志检查的匹配信息
# where protocols with found patterns will be stored.

$scriptpath = '/opt/nagios/var/tmp';    // 可调用的脚本或程序 
# where scripts will be searched for.

$MACROS = { CL_DISK01 => "/dev/dsk/c0d1", CL_DISK02 => "/dev/dsk/c0d2" };          // 定义宏，我们可以调用的变量  

@searches = (    // 此处为配置文件的内容，我们可以通过配置文件来执行程序，也可以通过在命令行中直接定义。通过配置文件更方便  
  {
    tag => 'temperature',       // tag可以理解为一个自定义的标志，它将在生成状态信息或协议信息中作为名字中的一部分使用，并没有实际的意义  
    logfile => '/var/adm/syslog/syslog.log',    // logfile为所要监控的日志文件  
    rotation => 'bmwhpux',     // rotation如果有截断日志的话用来定义如何匹配截断日志  
    criticalpatterns => ['OVERTEMP_EMERG', 'Power supply failed'],              // 严重错误，可以匹配一个或多个正则表达式 
    warningpatterns => ['OVERTEMP_CRIT', 'Corrected ECC Error'],                // 警告错误，可以匹配一个或多个正则表达式  
    options => 'script,protocol,nocount',             // 选项列表，我们可以选择启动脚本，写协议，不计数等操作  
    script => 'sendnsca_cmd'          //脚本的名字  
  },
  {
    tag => 'scsi',
    logfile => '/var/adm/messages',
    rotation => 'solaris',
    criticalpatterns => 'Sense Key: Not Ready',
    criticalexceptions => 'Sense Key: Not Ready /dev/testdisk',
    options => 'noprotocol'
  },
  {
    tag => 'logins',
    logfile => '/var/adm/messages',
    rotation => 'solaris',
    criticalpatterns => ['illegal key', 'read error.*$CL_DISK01$'],
    criticalthreshold => 4
    warningpatterns => ['read error.*$CL_DISK02$'],
  }
);

两种调用方式：
方式一：
[root@localhost ~]# /usr/local/nagios/libexec/check_logfiles 
Usage: check_logfiles [-t timeout] -f <configfile> [--searches=tag1,tag2,...]
       check_logfiles [-t timeout] --logfile=<logfile> --tag=<tag> --rotation=<rotation>
                      --criticalpattern=<regexp> --warningpattern=<regexp>
                                         
方式二：
在被监控端配置：
# vim /usr/local/nagios/var/log.cfg
@searches = (  
    {  
        tag => 'web_monitor',  
        logfile => '/var/log/web_monitor.log',  
        criticalpatterns => ['nginx has restart','nginx is down'],  
        warningpatterns => ['500','302','502']  
        #options => 'noprotocol'  
    }  
);  
【说明】定义了一个标志web_monitor,检查的日志文件为/var/log/web_monitor.log,当日志信息中匹配ciritical pattern中的内容时会报严重错误，当匹配warning pattern中的内容时会报警告错误；状态信息和协议信息会写入到/usr/local/nagios/var/tmp中，如log._var_log_web_monitor.log.web_monitor，其中web_monitor就是我们配置中的tag
# cat log._var_log_web_monitor.log.web_monitor   
$state = {  
           'runcount' => 17,  
           'serviceoutput' => '',  
           'logoffset' => 642985,  
           'runtime' => 1431504819,  
           'devino' => '64768:1178440',  
           'privatestate' => {  
                               'runcount' => 17,  
                               'lastruntime' => 1431504220,  
                               'logfile' => '/var/log/web_monitor.log'  
                             },  
           'logtime' => 1431504602,  
           'servicestateid' => 0,  
           'tag' => 'web_monitor'  
         };  
  
  
1;  
【说明】上面是监控之后生成的文件

被监控端的check_logfiles配置好了后，我们还需在nrpe.cfg中添加命令
command[check_logfile]=/usr/local/nagios/libexec/check_logfiles -f /usr/local/nagios/var/log.cfg

监控端配置：
define service{
    use                     nrpe-service         
    host_name               test
    service_description     web_monitor
    check_command           check_nrpe!check_logfile
    check_interval          10
    notifications_enabled   1
    service_groups          logfile_check
    contact_groups          test
    }

重启Nagios就可以看到监控项
```

#### 4.9 监控Tomcat服务：
```ruby

在tomcat的webapps目录下，新建一个目录jiankong（这个目录随便建），然后在其下面放一个asp文件。然后修改commands.cfg ，在里面添加
#tomcat1 set
define command{
command_name check_tomcat_8080
command_line /usr/local/nagios/libexec/check_http -I $HOSTADDRESS$ -p 8080 -u /jiankong/test.jsp -e 200
}
如果有多个端口，可以建立多个，只需要修改端口号，上面这个是8080端口，然后在servers.cfg中添加服务就好了。

[root@tomcat1 ~]# /usr/local/nagios/libexec/check_http     
check_http: Could not parse arguments
Usage:
 check_http -H <vhost> | -I <IP-address> [-u <uri>] [-p <port>]
       [-J <client certificate file>] [-K <private key>]
       [-w <warn time>] [-c <critical time>] [-t <timeout>] [-L] [-E] [-a auth]
       [-b proxy_auth] [-f <ok|warning|critcal|follow|sticky|stickyport>]
       [-e <expect>] [-d string] [-s string] [-l] [-r <regex> | -R <case-insensitive regex>]
       [-P string] [-m <min_pg_size>:<max_pg_size>] [-4|-6] [-N] [-M <age>]
       [-A string] [-k string] [-S <version>] [--sni] [-C <warn_age>[,<crit_age>]]
       [-T <content-type>] [-j method]


<servlet>
    <servlet-name>HelloWorld</servlet-name>
    <servlet-class>test.HelloWorld</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>HelloWorld</servlet-name>
    <url-pattern>/HelloWorld</url-pattern>
</servlet-mapping>

[root@localhost ~]# /usr/local/nagios/libexec/check_http -p 80 baidu.com -e 200
HTTP OK: Status line output matched "200" - 381 bytes in 5.083 second response time |time=5.083088s;;;0.000000 size=381B;;;0

```
