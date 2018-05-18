## 分布式监控架构部署
由于数据中心不在同一个机房，或者是用一台nagios无法满足监控的需求，或者是两台以才能满足你的监控需求，这时就需要用nagios的分布式监控
分布式主要是分布式检测，集中式展现，集中式报警（当然也可以分布式报警）等

- 监控中心服务器：通过NSCA获取分布式监控服务器的相关状态，呈现相关服务器状态和发出报警等;
- 分布式服务器：通过对被监控服务器状态采集并且把被监控服务器的状态通过NSCA_send发送给监控中心服务器。
- 被监控服务器：被监控服务器就是生产环境服务器。
![](https://github.com/ZongYuWang/image/blob/master/Nagios/Nagios-distributed1.png)
![](https://github.com/ZongYuWang/image/blob/master/Nagios/Nagios-distributed2.png)

【说明】
- 对次架构说明，分布式服务器和被监控服务器之间使用NRPE插件获取监控信息，分布式服务器主动获取被监控主机的监控信息，但是分布式服务器和监控中心之间使用的被动监控模式，被动监控模式使用了NSCA插件完成
- 如果分布式服务器和被监控主机之间也使用被动监控模式，个人认为被监控主机上也需要安装NSCA（使用nsca_send将监控信息发送给分布式服务器），然后分布式服务器上再额外需要安装nsca（作为被监控主机监控信息的接收端），也就是分布式服务器上同时需要使用nsca、nsca_send，nsca作为被监控服务器的接收端、nsca_send作为给监控主机发送告警的发送端（实验未测试）

### 一、安装被监控服务器：

#### 1.1 设置防火墙
```ruby
[root@localhost ~]# iptables -I INPUT -p tcp -m tcp --dport 5666 -j ACCEPT
```

#### 1.2安装相关软件包
```ruby
[root@localhost ~]# yum -y install vim  gcc glibc glibc-common gd gd-devel  openssl-devel

```
#### 1.3 设置登陆账号
```ruby
# groupadd nagcmd
# useradd -G nagcmd nagios
# passwd nagios
# chown nagios.nagios /usr/local/nagios/
```

#### 1.4 安装nagios-plugins
```ruby
# cd /nagios_soft/
# tar xvf nagios-plugins-2.1.4.tar.gz
# cd nagios-plugins-2.1.4
# ./configure && echo ok
# make && make install && echo ok
```
#### 1.5 安装nrpe
```ruby
# cd /nagios_soft/
# tar xvf nrpe-2.15.tar.gz 
# cd nrpe-2.15
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
#### 1.6 配置并启动nrpe
```ruby
# vim /usr/local/nagios/etc/nrpe.cfg
allowed_hosts=192.168.5.128
【说明】填写分布式服务器的IP地址,如果本地也想使用NRPE监控，则allowed_hosts也需要写上127.0.0.1
server_address=192.168.1.105
【说明】此处填写服务器的网卡地址，如果是云主机，填写内网IP地址或者外网IP地址都可以，但是在服务上使用ifconfig能看到的IP地址

# /usr/local/nagios/bin/nrpe -c /usr/local/nagios/etc/nrpe.cfg -d
# netstat -tlnp | grep 5666
```




### 二、安装分布式服务器

#### 2.1 安装相关软件包
```ruby
[root@localhost ~]# yum -y install vim  gcc glibc glibc-common gd gd-devel  openssl-devel

```
#### 2.2 设置登陆账号
```ruby
# useradd nagios
# passwd nagios
# groupadd nagcmd
# usermod -G nagcmd nagios
```

#### 2.3 安装nagios-plugins
```ruby
# tar xvf nagios-plugins-2.1.4.tar.gz 
# cd nagios-plugins-2.1.4
# ./configure --with-nagios-user=nagios --with-nagios-group=nagcmd 
# make && make install 
```
#### 2.4 安装nagios
```ruby
# mkdir /nagios_soft
# cd /nagios_soft/
# tar xvf nagios-3.5.0.tar.gz 
# cd nagios
# ./configure --with-command-group=nagcmd 
# make all 
# make install
# make install-init 
# make install-config 
# make install-commandmode 

# chkconfig --add nagios
# chkconfig nagios on
```
#### 2.5 安装nrpe
```ruby
# tar xvf nrpe-2.15.tar.gz 
# cd nrpe-2.15
# ./configure --with-nrpe-user=nagios \
     --with-nrpe-group=nagios \
     --with-nagios-user=nagios \
     --with-nagios-group=nagios \
     --enable-command-args \
     --enable-ssl
# make all
# make install-plugin
# /usr/local/nagios/libexec/check_nrpe -H 192.168.5.134      
【说明】测试被监控服务器是否能连通，正常情况才会返回被监控端的NRPE版本

# vim /usr/local/nagios/etc/objects/commands.cfg 
#check nrpe
define command{
       command_name check_nrpe
       command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
       }
【说明】上面是添加NRPE的外部检测命令，监控主机一般是通过check_nrpe插件进行远程或者本地主机的监控，check_nrpe插件会连接到远程的nrpe daemon，所用的方式是SSL
```
#### 2.6 安装nsca
```ruby
# tar xvf nsca-2.7.2.tar.gz 
# cd nsca-2.7.2
# ./configure 
# make all 
# cp sample-config/send_nsca.cfg /usr/local/nagios/etc/
# cp src/send_nsca /usr/local/nagios/bin/
# cd /usr/local/nagios/etc/
# chown nagios.nagios send_nsca.cfg 
# cd /usr/local/nagios/bin/
# chown nagios.nagios send_nsca 
```
#### 2.7 修改配置文件
```ruby
修改send_nsca.cfg配置文件：
# vim /usr/local/nagios/etc/send_nsca.cfg
password=wangzongyu
【说明】此处要和监控中心服务器中的密码一致
encryption_method=1

修改nagios主配置文件：
# vim /usr/local/nagios/etc/nagios.cfg 
enable_notifications=0 
// 监控中心设置不发送告警信息，此项设置的是当监控的对象的状态发生变化的时候，是否启动通知机制，当值等于1时，表示通知 ；当值等于0时，表示不通知；默认情况下是启动事件处理通知                                                               
obsess_over_services=1                                                          
// 此项设置的是决定nagios 是否被服务检测并运行之后定义的ocsp_command 命令，此项只在执行分布式检测是才启用，否则不要轻易启用该项，当然默认是是不启用的，当值为1是是启用，值为0时是不启用，也就是被监控客户端发送给分布式服务器监控信息之后，分布式服务器需要对数据进行处理通过开启ocsp_command定义的脚本命令，$1是主机名、$2是服务描述、$3是定义告警级别、$4是输出信息
ocsp_command=submit_check_result  
// 此命令是自己定义的命令，且该命令是在nagios的家目录下面的libexec 子目录下面； 该项是由nagios处理的，为每个服务检测而运行的命令，该命令仅当obsess_over_service选项的值设定为1时，此项执行有效；默认情况下是为空                               
obsess_over_hosts=1                                                                 
// 开启主机的被动监控
obsess_over_services=1

# mkdir /usr/local/nagios/libexec/eventhandlers/
# vim /usr/local/nagios/libexec/eventhandlers/submit_check_result 
#!/bin/sh
        # Arguments:
        #  $1 = host_name (Short name of host that the service is
        #       associated with)
        #  $2 = svc_description (Description of the service)
        #  $3 = state_string (A string representing the status of
        #       the given service - "OK", "WARNING", "CRITICAL"
        #       or "UNKNOWN")
        #  $4 = plugin_output (A text string that should be used
        #       as the plugin output for the service checks)
        # Convert the state string to the corresponding return code
        return_code=-1

        case "$3" in
                    OK)
                    return_code=0
                        ;;
                WARNING)
                    return_code=1
                        ;;
                CRITICAL)
                    return_code=2
                        ;;
                UNKNOWN)
                    return_code=-1
                        ;;
        esac
        # pipe the service check info into the send_nsca program, which
        # in turn transmits the data to the nsca daemon on the central
        # monitoring server

        /bin/printf "%s\t%s\t%s\t%s\n" "$1" "$2" "$return_code" "$4" | /usr/local/nagios/bin/send_nsca 192.168.5.131 -c  /usr/local/nagios/etc/send_nsca.cfg
// 此处的IP为监控中心的Server IP地址
// $1是主机名、$2是服务描述、$3是定义告警级别、$4是输出信息，也就是定义了一个命令的脚本，通过引用这个ocsp_command命令的脚本将告警信息发送给监控中心，所以上面的IP地址要写监控中心的IP地址
# chmod +x /usr/local/nagios/libexec/eventhandlers/submit_check_result 
# chown nagios.nagios /usr/local/nagios/libexec/eventhandlers/submit_check_result 

# vim /usr/local/nagios/etc/objects/commands.cfg 
define command{
                command_name submit_check_result
                command_line /usr/local/nagios/libexec/eventhandlers/submit_check_result $HOSTNAME$ '$SERVICEDESC$' $SERVICESTATE$ '$SERVICEOUTPUT$'
              }

添加被监控主机和服务：
# mkdir /usr/local/nagios/etc/objects/monitor/
# vim 172.30.105.114.cfg
define host {
        use                     linux-server
        host_name               172.30.105.114
        address                 172.30.105.114
        }

define service{
        use                     generic-service
        host_name               172.30.105.114
        service_description     Root Partiton
        check_command           check_disk!99%!96%!/
        check_period            24x7
        max_check_attempts      3
        normal_check_interval     1
        retry_check_interval       1
        }

define service{
        use                             local-service
        host_name                       172.30.105.114
        service_description             PING
        check_command                   check_ping!100.0,20%!500.0,60%
        }

# vim /usr/local/nagios/etc/nagios.cfg 
cfg_dir=/usr/local/nagios/etc/objects/monitor
# /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
# service nagios start
```

### 三、安装监控中心服务器：

#### 3.1 设置防火墙：
```ruby
[root@localhost ~]# iptables -I INPUT -p tcp -m tcp --dport 5667 -j ACCEPT
[root@localhost ~]# iptables -I INPUT -p tcp -m tcp --dport 80 -j ACCEPT
同时需要关闭SElinux，否则会报如下错误：
```

#### 3.2 安装相关软件包
```ruby
# yum -y install httpd gcc glibc glibc-common gd gd-devel php php-mysql mysql mysql-devel mysql-server vim perl-devel perl-CPAN -y
```

#### 3.3 设置登陆账号
```ruby
# useradd nagios
# passwd nagios
# groupadd nagcmd
# usermod -G nagcmd nagios
# usermod -G nagcmd apache  
```

#### 3.4 安装nagios-plugins
```ruby
# tar xvf nagios-plugins-2.1.4.tar.gz 
# cd nagios-plugins-2.1.4
# ./configure --with-nagios-user=nagios --with-nagios-group=nagcmd --enable-perl-modules
# make && make install 

// 在Nagios监控平台的服务器上如果没有安装nagios-plugins插件，nagios展示平台也就不能获取本机的监控信息
// 监控平台也必须要安装nagios-plugins插件，否则/usr/local/nagios/libexec下没有任何插件

```
![](https://github.com/ZongYuWang/image/blob/master/Nagios/Nagios-distributed3.png)

#### 3.5 安装nagios
```ruby
#mkdir /nagios_soft
# cd /nagios_soft/
# tar xvf nagios-3.5.0.tar.gz 
# cd nagios
# ./configure --with-command-group=nagcmd && echo ok
# make all 
# make install 
# make install-init
# make install-config 
# make install-commandmode 
# make install-webconf 
# chkconfig --add nagios
# chkconfig nagios on
```

#### 3.6 设置nagiosadmin的登陆密码
```ruby
# htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
```

#### 3.7 安装nsca
```ruby
# tar xvf nsca-2.7.2.tar.gz 
# cd nsca-2.7.2
# ./configure 
# make all 
# cp /nagios_soft/nsca-2.7.2/src/nsca /usr/local/nagios/bin/
# cp /nagios_soft/nsca-2.7.2/sample-config/nsca.cfg /usr/local/nagios/etc/
# chown nagios.nagios /usr/local/nagios/etc/nsca.cfg 
# chown nagios.nagios /usr/local/nagios/bin/nsca 
# cp init-script /etc/init.d/nsca
# chmod a+x /etc/init.d/nsca
 
 修改nsca的配置文件：
# vim /usr/local/nagios/etc/nsca.cfg 
password=wangzongyu
// 此处要和分布式监控服务器中的密码一致
server_port=5667
server_address=192.168.1.1
encryption_method=1

# vim /usr/local/nagios/etc/nagios.cfg 
check_external_commands=1
// 设定是否检测外部命令。默认是不启用的，但是由于需要配合Apache工作，在Web界面下进行Nagiso的控制和管理的话，必须要将此项打开
acept_passive_service_checks=1
// 此项是设定nagios在启动或者是重启时，是否将接受被动服务检测的结果。值1表示接受被动检测，值0表示拒绝被动检测。默认时是启动被动服务检测功能
accept_passive_host_checks=1
// 此项设定nagios在启动或者是重启是是否会接受被动主机检测的结果。值是1时表示接受，值是0时表示拒绝

# cd /usr/local/nagios/etc/
# mkdir monitor
# cd monitor/
```

#### 3.8 添加被监控客户端：
  
&emsp;&emsp;中心服务器和分布式服务器的时间一定要调整一致   
&emsp;&emsp;中心服务器和分布式服务器都需要添加监控的主机和服务，分布式服务器监控客户端主机可以用任何方式，主动/被动都可以（此实验分布式服务器是通过主动方式NRPE监控被监控端）   
&emsp;&emsp;中心服务器主机定义的host_name值需要和分布式服务器主机定义的host_name值一致；   
&emsp;&emsp;中心服务器服务定义的service_description值需要和分布式服务器服务定义的service_description值一致；   
&emsp;&emsp;分布式服务器上定义的服务检测命令（check_command）是真正的检测服务的命令 ，中心服务器上定义的服务检测命令   （check_command）是当中心服务器由被动检测变为主动刷新检测时执行的命令（也就是当分布式主机不发送检测命令或超时发送告警时中心服务器执行的命令），正常情况下不执行这个命令。   
&emsp;&emsp;分布服务器通常上面只安装有Nagios，它不需要安装Web接口   

##### 分布式服务器
     分布式服务器的配置增加和主动监控一样，先添加host，然后添加service即可
##### 中心服务器
     根据以上提到的注意事项，原则是中心服务器和分布式服务器的host_name和 service_descriptio一
     样，这样数据才可以在主的nagios中显示；  
##### 添加passive模式的主机和服务模板：  
```ruby
# vim /usr/local/nagios/etc/objects/templates.cfg 
  define host{
        name                            passive-host
        use                             generic-host
        check_period                    24x7
        check_interval                  5
        retry_interval                  1
        max_check_attempts              10
        check_command                   check-host-alive
        notification_period             24x7
        notification_interval           60
        notification_options            d,u,r
        #contact_groups                  sysmaint
        register                        0
        check_freshness                 1  ;定义强制刷新检测
        freshness_threshold             600  ;指定服务检测结果应该在何时间内刷新，单位是s
        passive_checks_enabled          1  ;打开被动检测
        active_checks_enabled           0  ;关闭主动监测
}
【说明】监控中心关闭主动监测，不会去主动监测被监控客户端，但是需要开启强制刷新监测，也就是超时没收到分布式服务器发来的监控信息，那么监控中心直接触发脚本，判定该服务为critical状态，监控中心不会真正的通过插件去监控被监控客户端，这样监控中心属于“越权”
【说明】freshness_threshold定义了600s，也就是说当分布式服务器超过了100s还没发送过来数据，中心服务器就会变被动为主动强制刷新监测，用check_command定义的命令执行服务检测，而中心服务器直接判定服务为critical

```
##### 添加强制刷新监测命令  
```ruby

# vim /usr/local/nagios/libexec/staleservice.sh
#!/bin/sh 
/bin/echo "CRITICAL: Service results are stale!" 
exit 2
# chmod +x /usr/local/nagios/libexec/staleservice.sh 
     添加命令：
    # vim /usr/local/nagios/etc/objects/commands.cfg 
define command{
        command_name    service-is-stale
        command_line    $USER1$/staleservice.sh
}

```
##### 添加主机和服务   
```ruby

复制分布式服务器中的hosts和services到nagios中心服务器中，修改定义主机中的use模板为定义的passive-host，之前默认一般为linux-server，修改定义服务器中的use模板为定义的passive-service，取消check_command值的定义，类似如下：
define host {
        use                     passive-host
        host_name               172.30.105.114
        address                 172.30.105.114
}

define service{
        use                    passive-service
        host_name              172.30.105.114
        service_description     Root Partiton 
}

define service{
        use                             passive-service
        host_name                       172.30.105.114
        service_description             PING
        }

#  /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
# /usr/local/nagios/bin/nsca -d -c /usr/local/nagios/nsca.cfg
# service httpd start
# service nagios restart
```
`为什么分布式服务器和监控中心配置的被监控服务和被监控主机一样？`
`答：因为一旦被监控主机宕机，分布式服务器过了超时时间没发送告警信息给监控中心，监控中心会强制刷新，为了使监控中心不“越权”，超时的监控将被监控中心直接判定为critical，分布式服务器上定义的监控主机与服务，在中心服务器上也要定义，保证主机名（host_name）和服务描述（service_description）一致即可，在监控中心不需要定义监控的命令，一旦监控中心超时没收到告警信息，那么监控中心直接触发脚本staleservice.sh，此脚本直接返回critical，意思也就是监控中心直接定义超时的监控对象为严重告警`


#### 3.9 查看输出日志
&emsp;&emsp;监控中心查看：
![](https://github.com/ZongYuWang/image/blob/master/Nagios/Nagios-distributed4.png)

&emsp;&emsp;监控中心开始强制刷新：
![](https://github.com/ZongYuWang/image/blob/master/Nagios/Nagios-distributed5.png)

&emsp;&emsp;分布式服务器发送到监控中心
```ruby
# echo "172.30.105.114;wangzy service;0;testOK"| /usr/local/nagios/bin/send_nsca 172.30.105.112 -d ";" -c /usr/local/nagios/etc/send_nsca.cfg 
```
![](https://github.com/ZongYuWang/image/blob/master/Nagios/Nagios-distributed6.png)

【说明】手动测试发送，通过一个管道将要发送的数据传给send_nsca插件，然后send_nsca再将数据发送到监控中心的nsca服务，其中172.30.105.114是被监控的客户端，wangzy service是nagios中定义的被动监控检测服务的名称，0是表示正常，testOK是输出信息，管道符之前的也就是分布式服务器中定义的submit_check_result 脚本中，$1、$2、$3、$4定义的内容；
172.30.105.112是监控中心地址，-d ";" 是分隔符，-c /usr/local/nagios/etc/send_nsca.cfg 是send_nsca这个发送程序的配置文件

&emsp;&emsp;分布式服务器显示信息：
![](https://github.com/ZongYuWang/image/blob/master/Nagios/Nagios-distributed7.png)

#### 3.10 监控中心展示效果说明：
![](https://github.com/ZongYuWang/image/blob/master/Nagios/Nagios-distributed8.png)
![](https://github.com/ZongYuWang/image/blob/master/Nagios/Nagios-distributed9.png)
![](https://github.com/ZongYuWang/image/blob/master/Nagios/Nagios-distributed10.png)
【说明】主动监控模式已经关闭，使用的被动监控模式
![](https://github.com/ZongYuWang/image/blob/master/Nagios/Nagios-distributed11.png)

## 被动监控架构部署
![](https://github.com/ZongYuWang/image/blob/master/Nagios/Nagios-distributed12.png)  
### 1、被动模式工作原理：
相比与主动模式中服务器主动去被监控机上轮询获取监控数据的方式，被动模式则是在被监控机上面通过插件或脚本获取监控数据，然后将数据通过send_nsca发往监控机，最后监控机通过nsca接收并解析数据，并传递给Nagios。这样做的一个很大的优势就是将除去处理数据的其他工作都放在了被监控机上面（包括了数据的传输），这样就避免了被监控机数量大时，一次轮询时间过长而导致监控反应延迟，这也是被动模式能承担更大监控量的关键

### 2、NSCA由两个部分组成：
nsca （安装在MonitorServer上，用来接收并解析MonitorClient发来的监控数据，传递给nagios）  
send_nsca（安装在MonitorClient上，用来发送监控数据。）

### 3、被动监控的过程：
MonitorClient上面，使用nagios-plugins提供的插件，得出监控数据，然后将数据存为一个文件，利用输入重定向，通过send_nsca将数据发往MonitorServer，MonitorServer上面运行一个nsca的daemon（默认开启5667端口），用来接收这些数据，然后做一个简单的处理（会和nagios的service文件进行对应，将多余的监控数据排除），然后将数据进行格式的转换，发给nagios的“外部命令文件”（默认配置为“/usr/local/nagios/var/rw/nagios.cmd”在nagios.cfg中定义的），该文件是一个管道文件，也是nagios主程序的一个接口（用来接收监控数据），使用cat查看该文件时候，会出来经nsca处理后的数据格式。然后nagios主程序对数据进行处理（前台展示，警报）