## 安装Nagios ##

### 1、Nagios基本安装： ###


### 2、PNP4Nagios的安装:
PHP4Nagios有三种工作模式，分别是Synchronous Mode、Bulk Mode和Bulk Mode with NPCD；
本实验使用Bulk Mode方式

#### 安装依赖包：
```py
# yum install -y rrdtool perl-Time-HiRes perl-devel perl-CPAN
```
#### 安装pnp4nagios：
```py
# cd /nagios_soft/
# tar xvf pnp4nagios-0.6.6.tar.gz
# cd pnp4nagios-0.6.6
# make all
# make install
# make install-webconf 
# make install-config
# make install-init
```
#### 配置pnp4nagios：
```py
# cd /usr/local/pnp4nagios/etc
# mv misccommands.cfg-sample misccommands.cfg        
# mv nagios.cfg-sample nagios.cfg        
# mv npcd.cfg-sample npcd.cfg
# mv process_perfdata.cfg-sample process_perfdata.cfg        
# mv rra.cfg-sample rra.cfg 

# cd /usr/local/pnp4nagios/etc/pages
# mv web_traffic.cfg-sample web_traffic.cfg

# cd /usr/local/pnp4nagios/etc/check_commands
# mv check_all_local_disks.cfg-sample check_all_local_disks.cfg        
# mv check_nrpe.cfg-sample check_nrpe.cfg        
# mv check_nwstat.cfg-sample check_nwstat.cfg 

# cp /usr/local/pnp4nagios/libexec/process_perfdata.pl /usr/local/nagios/libexec/
# chown -R nagios.nagios /usr/local/nagios/libexec/*
# /etc/init.d/npcd restart
```
#### 修改pnp4nagios使用Bulk Mode的方式：
```
# vim /usr/local/nagios/etc/nagios.cfg
enable_environment_macros=1
process_performance_data=1

service_perfdata_file=/usr/local/pnp4nagios/var/service-perfdata
service_perfdata_file_template=DATATYPE::SERVICEPERFDATA\tTIMET::$TIMET$\tHOSTNAME::$HOSTNAME$\tSERVICEDESC::$SERVICEDESC$\tSERVICEPERFDATA::$SERVICEPERFDATA$\tSERVICECHECKCOMMAND::$SERVICECHECKCOMMAND$\tHOSTSTATE::$HOSTSTATE$\tHOSTSTATETYPE::$HOSTSTATETYPE$\tSERVICESTATE::$SERVICESTATE$\tSERVICESTATETYPE::$SERVICESTATETYPE$
service_perfdata_file_mode=a
service_perfdata_file_processing_interval=15
service_perfdata_file_processing_command=process-service-perfdata-file

host_perfdata_file=/usr/local/pnp4nagios/var/host-perfdata
host_perfdata_file_template=DATATYPE::HOSTPERFDATA\tTIMET::$TIMET$\tHOSTNAME::$HOSTNAME$\tHOSTPERFDATA::$HOSTPERFDATA$\tHOSTCHECKCOMMAND::$HOSTCHECKCOMMAND$\tHOSTSTATE::$HOSTSTATE$\tHOSTSTATETYPE::$HOSTSTATETYPE$
host_perfdata_file_mode=a
host_perfdata_file_processing_interval=15
host_perfdata_file_processing_command=process-host-perfdata-file
【说明】host_perfdata_file_processing_command后面的名称要和后面配置的commands.cfg中的  command_name  匹配

host_perfdata_command=process-host-perfdata-file
service_perfdata_command=process-service-perfdata-file
【说明】修改process-service-perfdata-file名称的匹配

```
修改commands.cfg、templates.cfg、localhost.cfg（主机配置文件）：
```py
# vim /usr/local/nagios/etc/objects/commands.cfg
define command{
       command_name    process-service-perfdata-file
       command_line    /usr/local/pnp4nagios/libexec/process_perfdata.pl --bulk=/usr/local/pnp4nagios/var/service-perfdata
}

define command{
       command_name    process-host-perfdata-file
       command_line    /usr/local/pnp4nagios/libexec/process_perfdata.pl --bulk=/usr/local/pnp4nagios/var/host-perfdata
}

# vim /usr/local/nagios/etc/objects/localhost.cfg
define host{
        use                     linux-server,host-pnp
        host_name               localhost
        alias                   localhost
        address                 127.0.0.1
        }

define service{
        use                             local-service,service-pnp         
        host_name                       localhost
        service_description             PING
        check_command                   check_ping!100.0,20%!500.0,60%
        }

```
修改RRD文件路径的配置文件：
```py
# vim /usr/local/pnp4nagios/etc/config.php 
$conf['rrdbase'] = "/usr/local/pnp4nagios/var/perfdata/";

# vim /usr/local/pnp4nagios/etc/process_perfdata.cfg
RRDPATH = /usr/local/pnp4nagios/var/perfdata
```
检查pnp4nagios是否安装正确：
```py
http://192.168.1.111/pnp4nagios/
# mv /usr/local/pnp4nagios/share/install.php /usr/local/pnp4nagios/share/install.php.bak
```
#### 添加Nagios监控图像：
```py
将/etc/httpd/conf.d/pnp4nagios.conf 中的所有内容全部添加到apache的httpd.conf文件最后
Alias /pnp4nagios "/usr/local/pnp4nagios/share"
<Directory "/usr/local/pnp4nagios/share">
        AllowOverride None
        Order allow,deny
        Allow from all
        #
        # Use the same value as defined in nagios.conf
        #
        AuthName "Nagios Access"
        AuthType Basic
        AuthUserFile /usr/local/nagios/etc/htpasswd.users
        Require valid-user
        <IfModule mod_rewrite.c>
                # Turn on URL rewriting
                RewriteEngine On
                Options FollowSymLinks
                # Installation directory
                RewriteBase /pnp4nagios/
                # Protect application and system files from being viewed
                RewriteRule ^(application|modules|system) - [F,L]
                # Allow any files or directories that exist to be displayed directly
                RewriteCond %{REQUEST_FILENAME} !-f
                RewriteCond %{REQUEST_FILENAME} !-d
                # Rewrite all other URLs to index.php/URL
                RewriteRule .* index.php/$0 [PT,L]
        </IfModule>
</Directory>
```
弹窗方式显示性能图表：
效果图：   

![](https://github.com/ZongYuWang/image/blob/master/Nagios-pnp4nagios1.png)

```py
cp /nagios_soft/pnp4nagios-0.6.6/contrib/ssi/status-header.ssi /usr/local/nagios/share/ssi/
【注意】status-header.ssi必须没有执行权限

```
#### 修改Nagios的模板文件：
```py
# vim /usr/local/nagios/etc/objects/templates.cfg
define host {
        name       host-pnp
        action_url /pnp4nagios/index.php/graph?host=$HOSTNAME$&srv=_HOST_' class='tips' rel='/pnp4nagios/index.php/popup?host=$HOSTNAME$&srv=_HOST_
         register  0
}

define service {
         name       service-pnp
        action_url /pnp4nagios/index.php/graph?host=$HOSTNAME$&srv=$SERVICEDESC$' class='tips' rel='/pnp4nagios/index.php/popup?host=$HOSTNAME$&srv=$SERVICEDESC$
        register  0
}
```
检查配置文件是否正确：
```py
# /usr/local/nagios/bin/nrpe -c /usr/local/nagios/etc/nrpe.cfg -d
```

重启npcd服务：
```py
# /etc/init.d/npcd restart 
```
重启nagios服务：
```py
# service nagios restart
【注意】监控内存FREE的单位错误，单位应该是M，此处体现是K，但是数据正确
```

### 3、使用NDOUtils将Nagios监控信息存入MySQL:
![](https://github.com/ZongYuWang/image/blob/master/Nagios-NDOUtils1.png)

#### 安装配置MySQL：
```py
# yum install mysql mysql-server mysql-devel perl-DBD-MySQL
mysql> USE mysql;
mysql> update user set Password=password('newpassword') where User='root';
mysql> flush privileges;
mysql> create database nagios;
mysql> quit
```

#### 安装ndoitils：
```py
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
```py
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
#### 配置ndoitils：
```py
[root@localhost ~]# cp config/{ndo2db.cfg-sample,ndomod.cfg-sample} /usr/local/nagios/etc  
[root@localhost ~]# mv /usr/local/nagios/etc/ndo2db.cfg-sample /usr/local/nagios/etc/ndo2db.cfg
[root@localhost ~]# mv /usr/local/nagios/etc/ndomod.cfg-sample /usr/local/nagios/etc/ndomod.cfg 
[root@localhost ~]# cp src/ndomod-3x.o /usr/local/nagios/bin/
[root@localhost ~]# cp src/ndo2db-3x /usr/local/nagios/bin/

[root@localhost ~]# chmod 644 /usr/local/nagios/etc/ndo*  
[root@localhost ~]# chown nagios:nagios /usr/local/nagios/etc/*
[root@localhost ~]# chown nagios:nagios /usr/local/nagios/bin/*
```
```py
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
```py
[root@localhost db]# ./installdb -u root -p wangzongyu -h localhost -d nagios
DBI connect('database=nagios;host=localhost','root',...) failed: Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2) at ./installdb line 41

解决办法：
[root@localhost ~]# find / -name mysql.sock
/tmp/mysql.sock
[root@localhost ~]# mkdir /var/lib/mysql/
[root@localhost ~]# ln -s /tmp/mysql.sock /var/lib/mysql/mysql.sock
```

#### 设置ndo2db开机自启动：
```py

[root@localhost ndoutils-2.0.0]# cd /nagios_soft/ndoutils-2.0.0
[root@localhost ndoutils-2.0.0]# cp ./daemon-init /etc/init.d/ndo2db
[root@localhost ndoutils-2.0.0]# vim /etc/init.d/ndo2db 
    Ndo2dbBin=/usr/local/nagios/bin/ndo2db-3x
[root@localhost ndoutils-2.0.0]# chmod +x /etc/init.d/ndo2db 
[root@localhost ndoutils-2.0.0]# service ndo2db start
```
`可能报如下错误：`
```py
[root@localhost ndoutils-2.0.0]# service ndo2db start
Starting ndo2db:/usr/local/nagios/bin/ndo2db: error while loading shared libraries: libmysqlclient.so.18: cannot open shared object file: No such file or directory
done.

解决办法：
[root@localhost ~]# ln -s /usr/local/mysql/lib/* /usr/lib64
```
检查开启的端口：
```py
[root@localhost ~]# netstat -antup | grep 5668
tcp        0      0 0.0.0.0:5668                0.0.0.0:*                   LISTEN      121725/ndo2db-3x 
```
```py
[root@localhost ~]# service ndo2db stop
Stopping ndo2db: head: cannot open `/usr/local/nagios/var/ndo2db.lock' for reading: No such file or directory
done.
【说明】关闭ndo2db会存在问题
```
[ndo2pnp.pl下载地址]( https://github.com/ZongYuWang/File/tree/master/File )
```py

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


查看存入数据库的数据：
```py
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

查看连接数据库的日志：
```py
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

### 3、Nagios告警通知设置： 
```py
# yum install mailx*
# vim /etc/mail.rc
set from=13662097373@163.com
set smtp=smtp.163.com
set smtp-auth-user=13662097373@163.com
set smtp-auth-password=29755396Wzy             #授权码
set smtp-auth=login

# vim /usr/local/nagios/etc/objects/templates.cfg
define contact{
        name                            generic-contact        
        service_notification_period     24x7                   
        host_notification_period        24x7                  
        service_notification_options    w,u,c,r,f,s           
        host_notification_options       d,u,r,f,s              
        service_notification_commands   notify-service-by-email
        host_notification_commands      notify-host-by-email   
        register                        0                      
        }


define host{
        name                            generic-host    
        notifications_enabled           1              
        event_handler_enabled           1              
        flap_detection_enabled          1              
        failure_prediction_enabled      1             
        process_perf_data               1             
        retain_status_information       1              
        retain_nonstatus_information    1             
        notification_period             24x7         
        register                        0              
        }

define host{
        name                            linux-server   
        use                             generic-host   
        check_period                    24x7           
        check_interval                  5              
        retry_interval                  1              
        max_check_attempts              10          
        check_command                   check-host-alive
        notification_period             workhours      
                                                    
        notification_interval           120            
        notification_options            d,u,r  
        contact_groups                  admins        
        register                        0             
        }
【说明】max_check_attempts定义为1，检测到问题后立即报警，不重试。
notification_interval：重复发送提醒信息的最短间隔时间。默认间隔时间是60分钟。如果这个值设置为0，将不会发送重复提醒。


# vim /usr/local/nagios/etc/objects/commands.cfg
# 'notify-host-by-email' command definition
define command{
   command_name   notify-host-by-email
   command_line   /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\nHost: $HOSTNAME$\nState: $HOSTSTATE$\nAddress: $HOSTADDRSS$\nInfo: $HOSTOUTPUT$\n\nDate/Time: $LONGDATETIME$\n" | /bin/mailx -s "** $NOTIFICATIONTYPE$ Host Alert: $HOSTNAME$ is $HOSTSTATE$ **" $CONTACTEMAIL$
   }

# 'notify-service-by-email' command definition
 define command{
   command_name       notify-service-by-email
   command_line      /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\n\nService: $SERVICEDESC$\nHost: $HOSTALIAS$\nAddress: $HOSTADDRESS$\nState: $SERVICESTATE$\n\nDate/Time: $LONGDATETIME$\n\nAdditional Info:\n\n$SERVICEOUTPUT$\n" | /bin/mailx -s "** $NOTIFICATIONTYPE$ Service Alert: $HOSTALIAS$/$SERVICEDESC$ is $SERVICESTATE$ **" $CONTACTEMAIL$
   }

# vim /usr/local/nagios/etc/objects/contacts.cfg
define contact{
        contact_name                    nagiosadmin           
        use                             generic-contact        
        alias                           Nagios Admin        
        host_notifications_enabled      1
        service_notifications_enabled   1
        email                           479414941@qq.com      
        }


define contactgroup{
        contactgroup_name       admins
        alias                   Nagios Administrators
        members                 nagiosadmin
        }

# vim /usr/local/nagios/etc/objects/linux.cfg
define host{
        use             linux-server,host-pnp
        host_name       mysql1
        alias           My Linux Server
        address         192.168.1.112
        }

```
`测试：`
```py
# echo "hello word" | mailx -s "mail title" 479414941@qq.com
设置延时：
# vim /usr/local/nagios/etc/nagios.cfg
notification_timeout=300
参数：
# vim /usr/local/nagios/etc/objects/templates.cfg
host_notification_options：
d = notify on DOWN host states,
u = notify on UNREACHABLE host states
r = notify on host recoveries (UP states)
f = notify when the host starts and stops flapping
s = send notifications when host or service scheduled downtime starts and ends
n (none) as an option, the contact will not receive any type of host notifications.
service_notification_options:
w = notify on WARNING service states
u = notify on UNKNOWN service states
c = notify on CRITICAL service states
r = notify on service recoveries (OK states)
f = notify when the service starts and stops flapping
n (none) as an option, the contact will not receive any type of service notifications.

常用的设置
host_notification_options：d,u,r
service_notification_options:w,u,c,r
```
#### 设置告警次数:
```py
vi /usr/local/nagios/etc/objects/escalations.cfg
define serviceescalation{
host_name                    192.168.1.1      ;被监控主机名称，多个用逗号隔开与Hosts.cfg中一致
service_description        SSH                ;被监控服务名称，多个用逗号隔开 与services.cfg中一致
first_notification           4                         ; 第4条信息起，改变频率间隔
last_notification            0                        ; 第n条信息起，恢复频率间隔
notification_interval        30                    ; 通知间隔(单位：分)
contact_groups          admins
}

define serviceescalation{
host_name                  192.168.1.1
service_description        SSH
first_notification           10
last_notification            0
notification_interval        30
contact_groups            boss
}

最后，编辑nagios.cfg文件
#vi /usr/local/nagios/etc/nagios.cfg
添加：
cfg_file=/usr/local/nagios/etc/objects/escalations.cfg

检查nagios配置文件是否正确：
/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg

重启nagios服务：
service nagios restart
【说明】报警从第4次起之后都是每隔30分钟发一次报警 ，发给admins 组，到第10次之后 admins组和boss 组都能收到报警 时间一各自配置文件为准，本例为30分钟
```


### 4、Nagvis安装： 
`【说明】需要前提先安装NDOUtils，将Nagios监控信息存入MySQL`

```py
# yum install php-mbstring php-pdo graphviz rsync

# cd /nagios_soft/
# tar xvf nagvis-1.8.5.tar.gz
# mv nagvis-1.8.5 nagvis
# cd nagvis
# ./install.sh
[root@tomcat1 nagvis]# ./install.sh 
+------------------------------------------------------------------------------+
| Welcome to NagVis Installer 1.8.5                                            |
+------------------------------------------------------------------------------+
| This script is built to facilitate the NagVis installation and update        |
| procedure for you. The installer has been tested on the following systems:   |
| - Debian, since Etch (4.0)                                                   |
| - Ubuntu, since Hardy (8.04)                                                 |
| - SuSE Linux Enterprise Server 10 and 11                                     |
|                                                                              |
| Similar distributions to the ones mentioned above should work as well.       |
| That (hopefully) includes RedHat, Fedora, CentOS, OpenSuSE                   |
|                                                                              |
| If you experience any problems using these or other distributions, please    |
| report that to the NagVis team.                                              |
+------------------------------------------------------------------------------+
| Do you want to proceed? [y]: y
+------------------------------------------------------------------------------+
| Starting installation of NagVis 1.8.5                                        |
+------------------------------------------------------------------------------+
| OS  : CentOS release 6.5 (Final)                                             |
|                                                                              |
+--- Checking for tools -------------------------------------------------------+
| Using packet manager /bin/rpm                                          found |
|                                                                              |
+--- Checking paths -----------------------------------------------------------+
| Please enter the path to the nagios base directory [/usr/local/nagios]: 
|   nagios path /usr/local/nagios                                        found |
| Please enter the path to NagVis base [/usr/local/nagvis]: 
|                                                                              |
+--- Checking prerequisites ---------------------------------------------------+
| PHP 5.3                                                                found |
|   PHP Module: gd php                                                   found |
|   PHP Module: mbstring php                                             found |
|   PHP Module: gettext compiled_in                                      found |
|   PHP Module: session compiled_in                                      found |
|   PHP Module: xml php                                                  found |
|   PHP Module: pdo php                                                  found |
|   Apache mod_php                                                       found |
| Checking Backends. (Available: mklivestatus,ndo2db,ido2db)                   |
| Do you want to use backend mklivestatus? [y]: n
| Do you want to use backend ndo2db? [n]: y
| Do you want to use backend ido2db? [n]: n
|   /usr/local/nagios/bin/ndo2db-3x (ndo2db)                             found |
|   PHP Module: mysql php                                                found |
| WARNING: The Graphviz package was not found.                                 |
|          This may not be a problem if you installed it from source           |
|   Graphviz Module dot                                                MISSING |
|   Graphviz Module neato                                              MISSING |
|   Graphviz Module twopi                                              MISSING |
|   Graphviz Module circo                                              MISSING |
|   Graphviz Module fdp                                                MISSING |
| SQLite 3.6                                                             found |
|                                                                              |
+--- Trying to detect Apache settings -----------------------------------------+
| Please enter the web path to NagVis [/nagvis]: 
| Please enter the name of the web-server user [apache]: 
| Please enter the name of the web-server group [apache]: 
| create Apache config file [y]: y
|                                                                              |
+--- Checking for existing NagVis ---------------------------------------------+
|                                                                              |
+------------------------------------------------------------------------------+
| Summary                                                                      |
+------------------------------------------------------------------------------+
| NagVis home will be:           /usr/local/nagvis                             |
| Owner of NagVis files will be: apache                                        |
| Group of NagVis files will be: apache                                        |
| Path to Apache config dir is:  /etc/httpd/conf.d                             |
| Apache config will be created: yes                                           |
|                                                                              |
| Installation mode:             install                                       |
|                                                                              |
| Do you really want to continue? [y]: y
+------------------------------------------------------------------------------+
| Starting installation                                                        |
+------------------------------------------------------------------------------+
| Creating directory /usr/local/nagvis...                                done  |
| Creating directory /usr/local/nagvis/var...                            done  |
| Creating directory /usr/local/nagvis/var/tmpl/cache...                 done  |
| Creating directory /usr/local/nagvis/var/tmpl/compile...               done  |
| Creating directory /usr/local/nagvis/share/var...                      done  |
| Copying files to /usr/local/nagvis...                                  done  |
| Creating directory /usr/local/nagvis/etc/profiles...                   done  |
| Creating main configuration file...                                    done  |
| setting backend to ndomy_1                                             done  |
|   Adding webserver group to file_group...                              done  |
| Creating web configuration file...                                     done  |
| Setting permissions for web configuration file...                      done  |
|                                                                              |
|                                                                              |
|                                                                              |
+--- Setting permissions... ---------------------------------------------------+
| /usr/local/nagvis/etc/nagvis.ini.php-sample                            done  |
| /usr/local/nagvis/etc                                                  done  |
| /usr/local/nagvis/etc/maps                                             done  |
| /usr/local/nagvis/etc/maps/*                                           done  |
| /usr/local/nagvis/etc/geomap                                           done  |
| /usr/local/nagvis/etc/geomap/*                                         done  |
| /usr/local/nagvis/etc/profiles                                         done  |
| /usr/local/nagvis/share/userfiles/images/maps                          done  |
| /usr/local/nagvis/share/userfiles/images/maps/*                        done  |
| /usr/local/nagvis/share/userfiles/images/shapes                        done  |
| /usr/local/nagvis/share/userfiles/images/shapes/*                      done  |
| /usr/local/nagvis/var                                                  done  |
| /usr/local/nagvis/var/*                                                done  |
| /usr/local/nagvis/var/tmpl                                             done  |
| /usr/local/nagvis/var/tmpl/cache                                       done  |
| /usr/local/nagvis/var/tmpl/compile                                     done  |
| /usr/local/nagvis/share/var                                            done  |
|                                                                              |
+------------------------------------------------------------------------------+
| Installation complete                                                        |
|                                                                              |
| You can safely remove this source directory.                                 |
|                                                                              |
| For later update/upgrade you may use this command to have a faster update:   |
| ./install.sh -n /usr/local/nagios -p /usr/local/nagvis -b ndo2db -u apache -g apache -w /etc/httpd/conf.d -a y
|                                                                              |
| What to do next?                                                             |
| - Read the documentation                                                     |
| - Maybe you want to edit the main configuration file?                        |
|   Its location is: /usr/local/nagvis/etc/nagvis.ini.php                      |
| - Configure NagVis via browser                                               |
|   <http://localhost/nagvis/config.php>                                       |
| - Initial admin credentials:                                                 |
|     Username: admin                                                          |
|     Password: admin                                                          |
+------------------------------------------------------------------------------+
```
```py
# vim /usr/local/nagvis/etc/nagvis.ini.php
ndo2db MySQL backend (ndomy)
The ndo2db MySQL backend, in short ndomy backend, is used to fetch Nagios information like status and configuration data via a MySQL database. The Nagios addon called ndoutils stores all information which are present in a running Nagios in a MySQL database. This database is being queried by the NagVis ndomy backend.
You can use the following parameters to configure an ndomy backend:

dbhost="localhost"
dbport=3306
dbname="nagios"
dbuser="root"
dbpass="wangzongyu"
dbprefix="nagios_"
dbinstancename="default"
maxtimewithoutupdate=180
htmlcgi="/nagios/cgi-bin"

# vim /etc/httpd/conf/httpd.conf
添加：ServerName localhost:80
# service httpd restart
```
#### 添加一个MAP：
`Options`  -> `Manage Maps` -> `Create Map中填写ID(tradeease)`
`Open` -> `选择上面创建的Map`

添加主机、添加相关服务：
`都是在上面的MAP下操作`
`Edit Map` -> `Add Icon` ->

如果要对主机或者服务图标操作，需要先Unlock和lock，右键图标操作即可

`图片格式都是png`
`Logo`:/usr/local/nagvis/share/frontend/nagvis-js/images/internal
`背景图片路径`：/usr/local/nagvis/share/userfiles/images/maps
`服务器图片路径`：/usr/local/nagvis/share/userfiles/images/iconsets

`Edit Map` -> `Map Options`
![](https://github.com/ZongYuWang/image/blob/master/Nagios-Nagvis1.png)