## Nagios监控部署 

## 目录：
[一、Nagios基本安装](#一)     
[二、配置Nagios](#二)     
[三、PNP4Nagios的安装](#三)      
[四、使用NDOUtils将Nagios监控信息存入MySQL](#四)     
[五、Nagios告警通知设置](#五)     
[六、Nagvis安装](#六)       
[七、配置被监控端](#七)       
[八、Nagios BPI（Business Process Intelligence）](#八)      
[九、Nagios BP（Business Process AddOns）](#九)       
[十、附：项目监控配置](#十)       

<h3 id="一">一、Nagios基本安装</h3>

[Nagios的相关软件包下载1](https://sourceforge.net/projects/nagios/files/)  
[Nagios的相关软件包下载2](https://www.nagios.org/downloads/nagios-core-addons/)   
[Nagios的相关软件包下载3](http://nagios-plugins.org/download/)    
[Nagios的插件下载](https://labs.consol.de/nagios/)     

#### 1.1 解决安装Nagios的依赖关系：
`Nagios基本组件的运行依赖于httpd、gcc和gd。可以通过以下命令来检查nagios所依赖的rpm包是否已经完全安装：`
```js
# yum -y install httpd gcc glibc glibc-common gd gd-devel php php-mysql mysql mysql-devel mysql-server vim
【说明】以上软件包您也可以通过编译源代码的方式安装，只是后面许多要用到的相关文件的路径等需要按照您的源代码安装时的配置逐一修改。此外，您还得按需启动必要的服务，如httpd等。
```
#### 1.2 添加nagios运行所需要的用户和组：
```ruby
# groupadd  nagcmd
# useradd -G nagcmd nagios
# passwd nagios

把apache加入到nagcmd组，以便于在通过web Interface操作nagios时能够具有足够的权限：
# usermod -a -G nagcmd apache

```

#### 1.3 编译安装nagios：
```ruby
解决Perl软件编译问题:
# echo 'export LC_ALL=C' >> /etc/profile
# source /etc/profile
# echo $LC_ALL 
C 

# mkdir /nagios_soft
# cd /nagios_soft/
# tar zxf nagios-3.5.0.tar.gz
# cd nagios
# ./configure --with-command-group=nagcmd --enable-event-broker 
# make all
# make install
# make install-init
# make install-commandmode
# make install-config
在httpd的配置文件目录(conf.d)中创建Nagios的Web程序配置文件：
# make install-webconf

为email指定您想用来接收nagios警告信息的邮件地址，默认是本机的nagios用户:
# vi /usr/local/nagios/etc/objects/contacts.cfg 
email        nagios@localhost       #这个是默认设置
#【说明】修改为 email                           479414941@qq.com

创建一个登录nagios web程序的用户，这个用户帐号在以后通过web登录nagios认证时所用：
#【说明】增加nagios登陆认证文件，一定要用默认的nagiosadmin作为用户，否则需要修改其他文件
[root@nagios etc]# cd /usr/local/nagios/etc
[root@nagios etc]# sed -i s@nagiosadmin@nagiosadmin\,admin@g cgi.cfg
[root@nagios etc]# sed -i s@\#default_user_name=guest@default_user_name=admin@g cgi.cfg
# htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
New password: wangzongyu
Re-type new password: wangzongyu
Adding password for user nagiosadmin

以上过程配置结束以后需要重新启动httpd：
# service httpd restart
```

#### 1.4 编译、安装nagios-plugins
```ruby
nagios的所有监控工作都是通过插件完成的，因此，在启动nagios之前还需要为其安装官方提供的插件。
# tar zxf nagios-plugins-2.1.4.tar.gz 
# cdnagios-plugins-2.1.4
# ./configure --with-nagios-user=nagios --with-nagios-group=nagios
# make
# make install
```

#### 1.5 配置并启动Nagios

###### 1.5.1 把nagios添加为系统服务并将之加入到自动启动服务队列：
```
# chkconfig --add nagios
# chkconfig nagios on
```
###### 1.5.2 检查其主配置文件的语法是否正确：
```js
# /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg

```

###### 1.5.3 如果上面的语法检查没有问题，接下来就可以正式启动nagios服务了：
```ruby
# service nagios start

配置selinux
如果您的系统开启了selinux服务，则默认为拒绝nagios web cgi程序的运行。您可以通过下面的命令来检查您的系统是否开启了selinux：
#getenforce

如果上面命令的结果显示开启了selinux服务，您可以通过下面的命令暂时性的将其关闭：
#setenforce 0

如果您想在以后完全关闭selinux，可以通过编辑/etc/sysconfig/selinux文件，将其中的selinux后面的值“force”修改为“disable”即可。

当然，您也可以通过以下方式将nagios的CGI程序运行于SELinux/targeted模式而不用关闭selinux：
# chcon -R -t httpd_sys_content_t /usr/local/nagios/sbin
# chcon -R -t httpd_sys_content_t /usr/local/nagios/share

```

<h3 id="二">二、配置Nagios</h3>  

#### 2.1 Nagios的主配置文件
Nagios的主配置文件为nagios.cfg，参数的设置格式为<parameter>=<value>；其中，有些参数是可以重复出现的。其中常用的参数说明如下：
```ruby
log_file: 设定Nagios的日志文件；
cfg_file: Nagios对象定义的相关文件，此参数可重复使用多次以指定多个文件；
cfg_dir:  设定Nagios对象定义的相关文件所在的目录，此目录中的所有文件都会被作为对象定义的文件；此参数可重复使用多次以指定多个目录；
resource_file: 设定Nagios附加的宏定义的相关文件；
status_file: 设定Nagios存储所有主机和服务当前状态信息的文件；
status_update_interval: 设定status_file指定的文件中状态信息的更新频率；
service_check_timeout: 设定服务检测的超时时间，默认为60秒；
host_check_timeout: 设定主机检测的超时时间，默认为30秒；
notification_timeout: 设定通知信息发送尝试的超时时间，默认为30秒；
```

#### 2.2 resource_file和宏定义
```ruby
# less /usr/local/nagios/etc/resource.cfg 
在主配置文件中，参数resource_file用于定义所有用户变量(即“宏”)的存储文件，它用于存储对象定义中的可以访问的额外信息，如访问某服务的密码等；因此，这些信息通常都是些敏感数据，一般不允许通过Web接口来访问。此文件中可以定义的宏可多达32个，它们分别为$USER1$,$USER2$...$USER32，这些宏一般在check命令中引用。通常情况下$USER1$用于引用Nagios插件所在目录这个路径信息，因此，一般不建议修改其值。

# Nagios supports up to 32 $USERx$ macros ($USER1$ through $USER32$)
# Sets $USER1$ to be the path to the plugins
$USER1$=/usr/local/nagios/libexec

【说明】Nagios事先定义了许多宏，它们的值通常依赖于其上下文。如下：
HOSTNAME: 用于引用host_name指定所定义的主机的主机名；每个主机的主机名都是唯一的；
HOSTADDRESS: 用于引用host对象中的address指令的值，它通常可以为IP地址或主机名；
HOSTDISPLAYNAME: 用于引用host对象中alias指令的值，用以描述当前主机，即主机的显示名称；
HOSTSTATE：某主机的当前状态，为UP,DOWN,UNREACHABLE三者之一；
HOSTGROUPNAMES: 用于引用某主机所属的所有主机组的简名，主机组名称之间以逗号分隔；
LASTHOSTCHECK：用于引用某主机上次检测的时间和日期，Unix时间戳格式；
LISTHOSTSTATE：用于引用某主机前一次检测时的状态，为UP,DOWN或UNREACHABLE三者之一；
SERVICEDESC: 用于引用对应service对象中的desccription指令的值；
SERVICESTATE: 用于引用某服务的当前状态，为OK,WARNING,UNKOWN或CRITICAL四者之一；
SERVICEGROUPNAMES: 用于引用某服务所属的所有服务组的简名，服务组名称之间以逗号分隔；
CONTACTNAME: 用于引用某contact对象中contact_name指令的值；
CONTACTALIAS: 用于引用某contact对象中alias指令的值；
CONTACTEMAIL: 用于引用某contact对象中email指令的值；
CONTACTGROUPNAMES: 用于引用某contact所属的所有contact组的简名，contact组名称之间以逗号分隔；

Nagios 3还支持自定义宏，只是它的定义和使用方式比较独特。管理员可以在某类型对象的定义中使用额外的指令，并能够在命令中使用特别格式的宏来引用此指令的值。其引用方式根据对象类型的不同也有所不同，具体如下：

	$_HOST<variable>$ – 引用在主机对象中定义的指令的值；
	$_SERVICE<variable>$ – 引用在服务对象中定义的指令的值；
	$_CONTACT<variable>$ – 引用在联系人对象中定义的指令的值；

一个简单的例子如下：

如某主机定义为：
	define host
  {
    host_name somemachine
    address 10.0.0.1
    _MAC 12:34:56:78:90:ab
    check_command check-host-by-mac
  }

对应的检测命令则可以定义为：
  define command
  {
    command_name check-host-by-mac
    command_line $USER1$/check_hostmac -H $HOSTADDRESS$ -m $_HOSTMAC$
  }

```

#### 2.3 定义命令对象
```ruby
“命令”用于描述如何对主机或服务进行状态检测。服务对象的定义包含两个指令：名字(command_name)和命令行(command_line)；名字用于标识此命令对象，命令行则是执行检测时真正要执行的命令。

当命令对象用于检测其它对象时，其通常需要用到额外的参数以标识要检测的某特定对象，此时，命令对象需要以command_name[!arg1][!arg2][...]的语法格式进行引用。因此，命令对象的定义中，命令行指令中通常会用到宏$ARG1$, $ARG2$...，对应用于接收[!arg1][!arg2][...]传递而来的参数。

如下命令对象的定义：
	define command
	{
		command_name	check_local_swap
		command_line	$USER1$/check_swap -w $ARG1$ -c $ARG2$
	}

如下的服务中使用上面定义的命令对象来检测服务对象：

	define service
	{
    host_name  localhost
    service_description  Swap Usage
		check_command	 check_local_swap!20!10
  }

```
#### 2.4 定义主机对象
```ruby
“主机”指的是被监控的机器，可是物理主机，也可以是虚拟设备。一个主机对象的定义至少应该包含一个简名(short name)、一个别名、一个IP地址和用到的检测命令。此外，很多时候，其定义中还应该包含监控时段、联系人及要通知的相关问题、检测的频率、重试检测的方式、发送通知的频率等。具体的各指令及说明请参见官方文档：http://nagios.sourceforge.net/docs/3_0/objectdefinitions.html#host。

一个主机定义的例子：
	define host
	{
		host_name webserver1
		hostgroups webservers
		alias www.magedu.com
		address 172.16.100.11
		check_command check-host-alive
		check_interval 5
		retry_interval 1
		max_check_attempts 5
		check_period 24x7
		contact_groups linux-admins
		notification_interval 30
		notification_period 24x7
		notification_options d,u,r
	}

其中的notification_options用于指定当主机处于什么状态时应该发送通知。其各状态及其表示符如下：
		d —— DOWN
		u —— UNREACHABLE
		r —— UP(host recovery)
		f —— flapping
		s —— 调试宕机时间开始或结束
		
主机可以被划分成组，这些组即主机组。每一个主机组对象一般包含一个全局唯一的简名、一个描述名以及属于这个组的成员。此外，一个主机组的成员也可以是其它主机组。主机组的定义例子如下：

	define hostgroup
	{
		hostgroup_name webservers
		alias Linux web servers
		members webserver1
	}

```
#### 2.5 定义服务对象
```ruby
“服务”即某“主机”所提供的功能或资源对象，如HTTP服务、存储空间资源或CPU负载等。服务附属于主机，每一个服务使用服务名来标识，此服务名要求在特定的主机上具有唯一性。每一个服务对象还通常定义一个检测命令及如何进行问题通知等。

	define service
	{
		host_name webserver1
		service_description www
		check_command check_http
		check_interval 10
		check_period 24x7
		retry_interval 3
		max_check_attempts 3
		notification_interval 30
		notification_period 24x7
		notification_options w,c,u,r
		contact_groups linux-admins
	}

其中的notification_options用于指定当服务处于什么状态时应该发送通知。其各状态及其表示符如下：
		w —— WARNING
		u —— UNKNOWN
		c —— CRITICAL
		r —— OK(recovery)
		f —— flapping
		s —— 调试宕机时间开始或结束
		
与主机对象有所不同的是，有时个，多个主机可能会提供同样的服务，比如多台服务器同时提供Web等。因此，在定义服务对象时，其host_name可以为逗号隔开的多个主机。

服务可以被划分成组，这些组即服务组。每一个服务组对象一般包含一个全局唯一的简名、一个描述名以及属于这个组的成员。此外，一个服务组的成员通常是某主机上的某服务，其指定时使用<host>,<service>的格式，多个服务也使用逗号分隔。服务组的定义例子如下：

	define servicegroup
	{
		servicegroup_name webservices
		alias All services related to web
		members webserver1,www,webserver2,www
	}

```

#### 2.6 定义“时段”对象
```ruby
“时段”用于定义某“操作”可以执行或不能执行的日期和时间跨度，如工作日内的每天8:00-18:00等，其可以在多个不同的操作中重复引用。一个时段对象的定义包含一个全局唯一的名称标识及一个或多个时间跨度。例如：

	define timeperiod
	{
		timeperiod_name workinghours
		alias Working Hours, from Monday to Friday
		monday 09:00-17:00
		tuesday 09:00-17:00
		wednesday 09:00-17:00
		thursday 09:00-17:00
		friday 09:00-17:00
	}

其中，时间的指定格式有许多方式：
	日历时间：格式为YYYY-MM-DD，如2012-04-21；
	日期：如 April 21；
	每月的某一天：如 day 21，指每月的21号；
	每月的第几个周几：如 saturday 1，指每月的第一个星期六；
	星期几：如monday, tuesday等；
```
#### 2.7 定义联系人对象
```ruby
“联系人”对象用于定义某主机设备的拥有者或某问题出现时接受通知者。联系人对象的定义包含一个全局唯一的标识名称、一个描述名及一个或多个邮件地址等。此外，其通常还应该包括对相应的主机或服务出现故障时所用到的通知命令。例如：

	define contact
	{
		contact_name mageedu
		alias Mage Education
		email linuxedu@magedu.com
		host_notification_period  workinghours
		service_notification_period  workinghours
		host_notification_options  d,u,r
		service_notification_options  w,u,c,r
		host_notification_commands     host-notify-by-email
		service_notification_commands   notify-by-email
	}

联系人也可划分为组，即联系人组。一个联系人组对象包含一个全局惟一的标识名称，一个描述名称和属于此联系人组的联系人成员(members)或其人联系人组成员(contactgroup_members)。例如：

	define contactgroup
	{
		contactgroup_name linux-admins
		alias Linux Administrators
		members magedu, mageedu
	}

在主机或服务对象的定义中，既可以指定联系人，也可以指定联系人组。当然，某主机的问题联系人与其上运行的服务的联系人也可以不同。

```
#### 2.8 模板及对象继承
```ruby
Nagios通过功能强大的继承引擎来实现基于模板的对象继承。这就意味着可以定义将某类型的对象的通用属性组织起来定义为对象模板，并在定义其类型中的对象时直接从此模板继承其相关属性的定义。定义对象模板的方法很简单，通常只需要在定义某类型对象时使用register指令并将其值设定为0即可。对象模板的名称通常使用name指令定义，这与某特定类型对象使用的指令也有所不同。而定义此种类型的对象时，只需要使用use指令并将其值设定为对应模板的名称即可。例如：

	define host
	{
		name generic-server
		check_command check-host-alive
		check_interval 5
		retry_interval 1
		max_check_attempts 5
		check_period 24x7
		notification_interval 30
		notification_period 24x7
		notification_options d,u,r
		register 0
	}

	define host
	{
		use generic-server
		name webserver1
		alias Web Server 01
		address 172.16.100.11
		contact_groups linux-admins
	}

一个对象在定义时也以同时继承多个模板，此时只需要为use指令指定以逗号分隔的多个模板名称即可。同时，Nagios也支持模板的多级继承。

```
#### 2.9 依赖关系
```ruby
为了描述Nagios对象间的依赖关系，这里要用到两个术语：master（被依赖的主机或服务）和dependent（依赖关系中的依赖于master的Nagios对象）。Nagios可以定义对象间的彼此依赖性，也可以为某对象定义其父对象，甚至也可以指定此依赖关系生效的时段。下面是一个关于依赖关系定义的例子：

	define hostdependency
	{
		dependent_host_name backuphost
		host_name vpnserver1
		dependency_period maintenancewindows
	}

其中host_name用于定义master主机，dependent_host_name定义dependent主机。而在依赖关系的定义中，通常还会用到execution_failure_criteria定义master主机为何种状态时不再对依赖于此master的主机进行检测，notification_failure_criteria用于定义master处于何种状态时不会发送dependent相关的主机问题通知到联系人。

服务间依赖关系的定义类似于主机间的依赖关系，例如：

	define servicedependency
	{
		host_name mysqlserver
		service_description mysql
		dependent_hostgroup_name apacheservers
		dependent_service_description webservice
		execution_failure_criteria c,u
		notification_failure_criteria c,u,w
	}


```

#### 2.10 安装出现问题集合：
- 在首次配置了nagios监控端后，在浏览器输入地址后连接不上   
```ruby
可能是防火墙屏蔽了80端口，此时打开防火墙的80端口即可：
firewall-cmd --add-service=http （即时打开）
firewall-cmd --permanent --add-service=http（写入配置文件）
firewall-cmd --reload （重启防火墙）
如果出现如下错误：
You don't have permission to access /nagios/ on this server.nagios
此时要安装php
yum install php –y
然后重启httpd
```
- 启动nrpe后却不能互相通信   
```ruby
首先启动nrpe进程
systemctl restart nrped.service
此时可以检查nrpe绑定的5666端口是否被防火墙屏蔽了：
netstat -tnpl （观察是否有下面的两个服务之一）
如果5666端口没有打开就打开防火墙的5666端口：
firewall-cmd --zone=public --add-port=5666/tcp --permanent （添加5666端口）
firewall-cmd --reload （重启防火墙）
或者直接关闭防火墙
[root@localhost ~]# systemctl stop firewalld.service
```

- 安装pnp4nagios后出现The requested URL /pnp4nagios/graph was not found on this server.   
```ruby
当你在pnp4nagios安装的时候执行了make install-webconf，注意它生成了一个apache的配置文件。
你把这个文件：/etc/httpd/conf.d/pnp4nagios.conf 中的所有内容全部添加到apache的httpd.conf文件最后，再重新启动nagios和apache就应该可以

```

- 出现“CHECK_NRPE: Error - Could not complete SSL handshake.”的错误   
```ruby
yum install openssl openssl-devel
检查nagios监控端的允许地址和目标端的nrpe允许地址配置正确。比如被监控端的配置（命令：vi  /usr/local/nagios/etc/nrpe.cfg）：
allowed_hosts=127.0.0.1,192.168.1.112 （两个地址之间只有一个逗号，不能有空格）

执行 ./configure时报错：configure error cannot find ssl headers
yum -y install openssl-devel

```

- 解压./configure 后，在nagios-4.0.8进行make all报错   
```ruby
cd ./base && make
make[1]:Entering directory '/tmp/nagios/base'
make[1]:*** No rule to make target '/include/locations.h', needed by 'broker.o'. Stop.
make[1]:Leaving directory '/tmp/nagios/base'
make:***[all]Error 2
安装好perl就不出这个问题了！命令如下：
yum -y install perl
注意，install perl之后需要重新./configure一下，要不然还是提示这个错误
安装nrpe时执行.configure出错
在监控主机上安装check-nrpe插件时（实际上就是nrpe的整个安装）

```
- 安装nrpe时执行.configure出错   
```ruby
./configure 提示报错：
checking for SSL headers... configure: error: Cannot find ssl headers
如果这时运行命令 make all，则会报错：make: *** 没有规则可以创建目标“all”。停止。
解决办法：
yum -y install openssl-devel
记得：装完openssl-devel之后，要执行 ./configure
然后再make all
make install-plugin
```

- 错误：perfdata directory "/usr/local/pnp4nagios/var/perfdata/" is empty   
```ruby
# vim /usr/local/pnp4nagios/etc/config.php 
$conf['rrdbase'] = "/usr/local/pnp4nagios/var/";

# vim /usr/local/pnp4nagios/etc/process_perfdata.cfg 
RRDPATH = /usr/local/pnp4nagios/var
【注意】单独修改process_perfdata.cfg 这个配置文件的目录没有生效

```

<h3 id="三">三、PNP4Nagios的安装:</h3>    

PHP4Nagios有三种工作模式，分别是Synchronous Mode、Bulk Mode和Bulk Mode with NPCD；     
本实验使用Bulk Mode方式

#### 3.1 安装依赖包：
```ruby
# yum install -y rrdtool perl-Time-HiRes perl-devel perl-CPAN
```
#### 3.2 安装pnp4nagios：
```ruby
# cd /nagios_soft/
# tar xvf pnp4nagios-0.6.6.tar.gz
# cd pnp4nagios-0.6.6
# make all
# make install
# make install-webconf 
# make install-config
# make install-init
```
#### 3.3 配置pnp4nagios：
```ruby
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
#### 3.4 修改pnp4nagios使用Bulk Mode的方式：
```ruby
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
```ruby
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
```ruby
# vim /usr/local/pnp4nagios/etc/config.php 
$conf['rrdbase'] = "/usr/local/pnp4nagios/var/perfdata/";

# vim /usr/local/pnp4nagios/etc/process_perfdata.cfg
RRDPATH = /usr/local/pnp4nagios/var/perfdata
```
检查pnp4nagios是否安装正确：
```ruby
http://192.168.1.111/pnp4nagios/
# mv /usr/local/pnp4nagios/share/install.php /usr/local/pnp4nagios/share/install.php.bak
```
#### 3.5 添加Nagios监控图像：
```ruby
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

```ruby
cp /nagios_soft/pnp4nagios-0.6.6/contrib/ssi/status-header.ssi /usr/local/nagios/share/ssi/
【注意】status-header.ssi必须没有执行权限

```
#### 3.6 修改Nagios的模板文件：
```ruby
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
```ruby
# /usr/local/nagios/bin/nrpe -c /usr/local/nagios/etc/nrpe.cfg -d
```

重启npcd服务：
```ruby
# /etc/init.d/npcd restart 
```
重启nagios服务：
```ruby
# service nagios restart
【注意】监控内存FREE的单位错误，单位应该是M，此处体现是K，但是数据正确
```

<h3 id="四">四、使用NDOUtils将Nagios监控信息存入MySQL</h3>    

![](https://github.com/ZongYuWang/image/blob/master/Nagios-NDOUtils1.png)

#### 4.1 安装配置MySQL：
```ruby
# yum install mysql mysql-server mysql-devel perl-DBD-MySQL
mysql> USE mysql;
mysql> update user set Password=password('newpassword') where User='root';
mysql> flush privileges;
mysql> create database nagios;
mysql> quit
```

#### 4.2 安装ndoitils：
```ruby
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
```ruby
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
#### 4.3 配置ndoitils：
```ruby
[root@localhost ~]# cp config/{ndo2db.cfg-sample,ndomod.cfg-sample} /usr/local/nagios/etc  
[root@localhost ~]# mv /usr/local/nagios/etc/ndo2db.cfg-sample /usr/local/nagios/etc/ndo2db.cfg
[root@localhost ~]# mv /usr/local/nagios/etc/ndomod.cfg-sample /usr/local/nagios/etc/ndomod.cfg 
[root@localhost ~]# cp src/ndomod-3x.o /usr/local/nagios/bin/
[root@localhost ~]# cp src/ndo2db-3x /usr/local/nagios/bin/

[root@localhost ~]# chmod 644 /usr/local/nagios/etc/ndo*  
[root@localhost ~]# chown nagios:nagios /usr/local/nagios/etc/*
[root@localhost ~]# chown nagios:nagios /usr/local/nagios/bin/*
```
```ruby
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
```ruby
[root@localhost db]# ./installdb -u root -p wangzongyu -h localhost -d nagios
DBI connect('database=nagios;host=localhost','root',...) failed: Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2) at ./installdb line 41

解决办法：
[root@localhost ~]# find / -name mysql.sock
/tmp/mysql.sock
[root@localhost ~]# mkdir /var/lib/mysql/
[root@localhost ~]# ln -s /tmp/mysql.sock /var/lib/mysql/mysql.sock
```

#### 4.4 设置ndo2db开机自启动：
```ruby

[root@localhost ndoutils-2.0.0]# cd /nagios_soft/ndoutils-2.0.0
[root@localhost ndoutils-2.0.0]# cp ./daemon-init /etc/init.d/ndo2db
[root@localhost ndoutils-2.0.0]# vim /etc/init.d/ndo2db 
    Ndo2dbBin=/usr/local/nagios/bin/ndo2db-3x
[root@localhost ndoutils-2.0.0]# chmod +x /etc/init.d/ndo2db 
[root@localhost ndoutils-2.0.0]# service ndo2db start
```
`可能报如下错误：`
```ruby
[root@localhost ndoutils-2.0.0]# service ndo2db start
Starting ndo2db:/usr/local/nagios/bin/ndo2db: error while loading shared libraries: libmysqlclient.so.18: cannot open shared object file: No such file or directory
done.

解决办法：
[root@localhost ~]# ln -s /usr/local/mysql/lib/* /usr/lib64
```
检查开启的端口：
```ruby
[root@localhost ~]# netstat -antup | grep 5668
tcp        0      0 0.0.0.0:5668                0.0.0.0:*                   LISTEN      121725/ndo2db-3x 
```
```ruby
[root@localhost ~]# service ndo2db stop
Stopping ndo2db: head: cannot open `/usr/local/nagios/var/ndo2db.lock' for reading: No such file or directory
done.
【说明】关闭ndo2db会存在问题
```
[ndo2pnp.pl下载地址]( https://github.com/ZongYuWang/File/tree/master/File )
```ruby

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
```ruby
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
```ruby
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

<h3 id="五">五、Nagios告警通知设置</h3>   

```ruby
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
```ruby
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
#### 5.1 设置告警次数:
```ruby
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


<h3 id="六">六、Nagvis安装</h3>   

`【说明】需要前提先安装NDOUtils，将Nagios监控信息存入MySQL`

```ruby
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
```ruby
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
#### 6.1 添加一个MAP：
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


<h3 id="七">七、配置被监控端(NRPE)</h3>    

#### 7.1 NRPE简介：

- Nagios监控远程主机的方法有多种，其方式包括SNMP、NRPE、SSH和NCSA等。这里介绍其通过NRPE监控远程Linux主机的方式。
- NRPE（Nagios Remote Plugin Executor）是用于在远端服务器上运行检测命令的守护进程，它用于让Nagios监控端基于安装的方式触发远端主机上的检测命令，并将检测结果输出至监控端。而其执行的开销远低于基于SSH的检测方式，而且检测过程并不需要远程主机上的系统帐号等信息，其安全性也高于SSH的检测方式。

![](https://github.com/ZongYuWang/image/blob/master/Nagios-NRPE1.png)

#### 7.2 配置监控端

###### 7.2.1 安装NRPE
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

###### 7.2.2 check_nrpe语法：
```ruby
通过NRPE监控远程Linux主机要使用chech_nrpe插件进行，其语法格式如下：
check_nrpe -H <host> [-n] [-u] [-p <port>] [-t <timeout>] [-c <command>] [-a <arglist...>]
【说明】# ls /usr/local/nagios/libexec/会多出来一个check_nrpe插件

```

- 使用示例1：
定义监控远程Linux主机swap资源的命令：
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

- 使用示例2：
如果希望上面的command定义更具有通用性，那么上面的定义也可以修改为如下：

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
【说明】每个！后面代表传递的参数，此例子中定义主机中传递的参数使用的是 check_swap，同样check_swap也是在nrpe.cfg中定义的
【说明】因为command中使用-c定义了宏（变量），所以也可以这些写：check_command check_nrpe!check_users
【注意】-c后面接的是命令，也就是nrpe.cfg中定义的command后面的命令名称

```

- 使用示例3：
如果还希望在监控远程Linux主机时还能向其传递参数，则可以使用类似如下方式进行：
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
【说明】示例2、示例3都是接收传递参数的形式，示例3中-c 明确使用check_swap命令，而后面的参数是check_swap可以接收的参数
需要在被监控端的nrpe.cfg中使用check_swap的参数命令形式：
command[check_swap]=/usr/local/nagios/libexec/check_disk -w $ARG1$ -c $ARG2$
【说明】: ,$s/windows-server/linux-server/g 将全文的windows-server替换为linux-server


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

- 使用示例4：
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

#### 7.3 配置被监控端
```ruby

# 安装相关软件包：
# yum -y groupinstall "Development Tools" "Development Libraries" openssl*
```
###### 7.3.1 添加nagios用户
```ruby
# useradd -s /sbin/nologin nagios
```

###### 7.3.2 NRPE依赖于nagios-plugins，因此，需要先安装之
```ruby
# tar zxf nagios-plugins-1.4.15.tar.gz 
# cd nagios-plugins-1.4.15
# ./configure --with-nagios-user=nagios --with-nagios-group=nagios
# make all
# make instal
```

###### 7.3.3 安装NRPE

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

###### 7.3.4 配置NRPE

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

###### 7.3.5 启动NRPE
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
###### 7.3.6 配置允许远程主机监控的对象
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
#### 7.4 配置被监控端监控项
###### 7.4.1 监控硬盘I/O
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
###### 7.4.2 监控主机存活状态
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
###### 7.4.3 check_load检测：
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
###### 7.4.4 监控CPU使用情况：
```ruby

[root@localhost ~]# /usr/local/nagios/libexec/check_cpu.sh                        
OK: CPU=3.92 | used=3.92;;;; system=2.05;;;; user=1.09;;;; nice=0;;;; iowait=.38;;;; irq=.04;;;; softirq=.34;;;;

[root@localhost ~]# /usr/local/nagios/libexec/check_cpu.sh -w 0 -c 0
CRITICAL: CPU=2.70 | used=2.70;0;0;; system=1.01;;;; user=.90;;;; nice=0;;;; iowait=.56;;;; irq=0;;;; softirq=.22;;;;
```

###### 7.4.5 监控内存使用情况：
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

###### 7.4.6 监控网络流量：
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
###### 7.4.7 监控MySQL（官网自带的check_mysql和check_mysql_health）：
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

###### 7.4.8 日志监控：
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

###### 7.4.9 监控Tomcat服务：
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

<h3 id="八">八、Nagios BPI（Business Process Intelligence）</h3>     

`Nagios Business Process Intelligence is an advanced grouping tool that allows you to set more complex dependencies to determine groups states. Nagios BPI provides an interface to effectively view the ‘real’ state of the network. Rules for group states can be determined by the user, and parent-child relationships are easily identified when you need to ‘drill down’ on a problem. This tool can also be used in conjunction with a check plugin to allow for notifications through Nagios.  This document describes how to fully utilize the Nagios Business Process Intelligence (or BPI) add-on and incorporate checks into Nagios.`  
【说明】Nagios Business Process Intelligence （BPI）是一种高级的分组工具，允许你设置更复杂的依赖关系来确定组状态。 Nagios BPI提供了一个界面来有效地查看网络的“真实”状态。 组状态的规则可以由用户确定。此工具也可以与检查插件结合使用，以通过Nagios进行通知。 本文档介绍如何充分利用Nagios业务流程智能（或BPI）附件，并将检查纳入Nagios。

```ruby
# cd /tmp
# wget https://github.com/NagiosEnterprises/nagiosbpi/archive/master.zip
# unzip master.zip 
# mv /tmp/nagiosbpi-master/nagiosbpi /usr/local/nagios/share/
# cd /usr/local/nagios/share/nagiosbpi
# mkdir tmp
# chmod +x set_bpi_perms.sh
# ./set_bpi_perms.sh 
## chown -R apache:nagios /usr/local/nagios/share/nagiosbpi/
# vim constants.conf
STATUSFILE=/usr/local/nagios/var/status.dat
OBJECTSFILE=/usr/local/nagios/var/objects.cache
CONFIGFILE=/usr/local/nagios/share/nagiosbpi/bpi.conf
CONFIGBACKUP=/usr/local/nagios/share/nagiosbpi/bpi.conf.backup
XMLOUTPUT=/usr/local/nagios/share/nagiosbpi/tmp/bpi.xml
```
http://192.168.5.113/nagios/nagiosbpi/
![](https://github.com/ZongYuWang/image/blob/master/Nagios-BPI1.png)
```js
define localServices1 {
                title=Local Services
                desc=Example BPI Group
                primary=1
                info=http://localhost
                members=localhost;Current Load;&, localhost;Current Users;&, localhost;HTTP;&, localhost;PING;|,
                warning_threshold=1
                critical_threshold=2
                priority=1

}

##################################
define localServices2 {
                title=More Local Services
                desc=Demo Group 2
                primary=1
                info=http://localhost/nagios
                members=$localServices1;&, localhost;Root Partition;|, localhost;SSH;&, localhost;Swap Usage;&, localhost;Total Processes;|,
                warning_threshold=3
                critical_threshold=4
                priority=2

}
【说明】每个members成员服务使用这种格式：localhost;Current Users;&,     主机名;检测的服务;&，   &表示非必要成员，|表示必要成员，使用这个符号服务名后面会显示**，各个成员服务名之间使用,分割

```
- A Basic BPI Group  
`This is a basic group with 5 members.The group has no thresholds set, and there are no essential members. Since there are still some members in an 'Ok' state, the group state is listed as 'Ok.'`  
【说明】一个基本的BPI组，这个基本的BPI组有5个成员，这个组没有阈值的设置，也没有“必要”的成员，只要这个组成员中有OK状态的，这个组的状态就是OK的   
![](https://github.com/ZongYuWang/image/blob/master/Nagios-BPI2.png) 

- A Group Using Thresholds   
`This  group has no essential members, but it has a warning threshold set at 3 problems, and a critical threshold set at 6 problems. Since the problem count of the group's members exceeds the warning threshold, the group state is 'Warning.'`  
【说明】一个使用了阈值的组，这个组没有“必要”的成员，但是设置上了超过3个问题就会有“warning”的阈值，超过6个问题就会有“critical”的阈值，
这个组的成员出现的问题数超过了“warning”设置的阈值（3个），所以这个组的状态就是“warning”  
![](https://github.com/ZongYuWang/image/blob/master/Nagios-BPI3.png) 

- A Group Using Essential Members   
`This group has 2 essential members defined, which are denoted with a '**' next to their state. If an essential member has a problem, the entire group will be in a problem state, even though the thresholds have not been exceeded, and there is only one problem.`  
【说明】这个组使用了“必要的”成员，这个组有2个必要的成员的定义，后面标记**的为必要的成员，如果一个必要的成员出现了问题，那么整个组将是问题状态，尽管没有超出阈值，也会存在一个问题


- Complex BPI Groups   
`The BPI groups determine state by looking down only one level. The BPI group will essentially look for the worst state trigger in the group, so if the warning threshold is exceeded for a group, but an essential member is “critical”, the group will still be “critical”. There is no limit to the number of sub groups that can be created, you can define as many levels in your dependency tree as you want.`  
【说明】这个BPI组只有一个水平的状态，这个BPI组将会找一个组内最糟糕的状态触发，所以如果组内的“warning”超出了阈值，但是一个必要的成员状态是“critical”（critical > warning），那么这个组的状态也是“critical”，子组的创建时没有限制的，你可以创建多个水平在你的依赖树状中  
![](https://github.com/ZongYuWang/image/blob/master/Nagios-BPI4.png) 

- Primary Groups   
`“Primary” BPI groups are seen from the top level of BPI page, while a non-primary group must have a visible parent group in order to be seen on the display. If a non-primary group is defined but never assigned as a member somewhere else, it will not be visible on the display.`  
【说明】“Primary”BPI组会在BPI的页面的最顶端看到，然而没有主组的必要要有一个依赖的“父组”，为了是能在页面中显示，如果一个“non-primary”组被定义了，但是没有委派任何的成员，那么这个组将不会再页面中显示


<h3 id="九">九、Nagios BP（Business Process AddOns）</h3>      

```ruby
# wget http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
# rpm -ivh epel-release-6-8.noarch.rpm
# yum install --enablerepo=epel perl-JSON-XS perl-CGI-Simple
# perl -MCPAN -eshell
  cpan> install Bundle::LWP


# tar xvf nagios-business-process-addon-0.9.6.tar.gz
# cd nagios-business-process-addon-0.9.6
# ./configure --prefix=/usr/local/nagiosbp --sysconfdir=/usr/local/nagiosbp/etc/nagiosbp --with-nagetc=/usr/local/nagios/etc 
# make install 
# cd /usr/local/nagiosbp/etc/nagiosbp
# cp nagios-bp.conf-sample nagios-bp.conf
# cp ndo.cfg-sample ndo.cfg

# cd  /usr/local/nagiosbp/etc/nagiosbp
# vim ndo.cfg
ndo=db
#ndo_livestatus_socket=/usr/local/nagios/var/rw/ 
【说明】注销此行
ndodb_host=localhost
ndodb_port=3306
ndodb_database=nagios
ndodb_username=root
ndodb_password=wangzongyu

```
```ruby
[root@localhost ~]# /usr/local/nagiosbp/bin/nagios-bp-check-ndo-connection.pl 

Report of actual status information in NDO
------------------------------------------

Backend is db (NDO Database)
which got it's last update at 2017-04-27 14:59:42

[192.168.67.112;Hoststatus] [CRITICAL] CRITICAL - 192.168.67.112: rta nan, lost 100%
      [192.168.67.112;PING] [CRITICAL] CRITICAL - 192.168.67.112: rta nan, lost 100%
   [localhost;Current Load] [OK      ] OK - load average: 0.00, 0.00, 0.00
  [localhost;Current Users] [OK      ] USERS OK - 3 users currently logged in
           [localhost;HTTP] [WARNING ] HTTP WARNING: HTTP/1.1 403 Forbidden - 5159 bytes in 0.003 second response time
     [localhost;Hoststatus] [OK      ] PING OK - Packet loss = 0%, RTA = 0.05 ms
           [localhost;PING] [OK      ] PING OK - Packet loss = 0%, RTA = 0.05 ms
 [localhost;Root Partition] [OK      ] DISK OK - free space: / 43017 MB (94% inode=98%):
            [localhost;SSH] [OK      ] SSH OK - OpenSSH_5.3 (protocol 2.0)
[localhost;Total Processes] [OK      ] PROCS OK: 69 processes with STATE = RSZDT
        [mysql1;Hoststatus] [CRITICAL] (Host Check Timed Out)
       [mysql1;check users] [CRITICAL] CHECK_NRPE: Socket timeout after 10 seconds.

```
```ruby
# vim /usr/local/nagiosbp/etc/nagios-bp.conf
<hostname>;<servicename>

例子：
internetconnection = internetconnection;Provider 1 | internetconnection;Provider 2    #网络连接，只有有一个运营商存活即可；
display 0;internetconnection;Internet Connection
		
loadbalancers = loadbalancer1;System Health | loadbalancer2;System Health      #两个主机的系统状况，这是负载均衡器，只要有一个存活即可
display 0;loadbalancers;Loadbalancer Cluster
		
dns = dns1;DNS | dns2;DNS | dns3;DNS     #三个DNS服务器，只要有一个存活即可
display 0;dns;DNS Cluster
		
website_webserver1 = webserver1;HTTP & webserver1;HTTPD Slots   # website_webserver1主机的HTTP服务和HTTPD接口都正常
website_webserver2 = webserver2;HTTP & webserver2;HTTPD Slots
website_webservers = website_webserver1 | website_webserver2       #表示这个网站只要有一个网站服务器存活就行
website = internetconnection & loadbalancers & dns & website_webservers   #表示整个网站需要服务商&其中一台主机&其中一台DNS服务器&其中一个HTTP服务正常

display <x>;<bp_name>;<long_name> 		
display 0;website_webserver1;WebServer 1
display 0;website_webserver2;WebServer 2
display 0;website_webservers;WebServer Cluster
display 1;website;WebSite
【说明】long_name是显示进程时使用的名称或说明，用户从未在GUI中看到<bp_name>，始终为<long_name>
【说明】0表示此过程不显示在顶级视图中，比如一些前提条件可以不用显示，最后要的结果可以显示出来，可以看下面的例子

external_info <bp_name>;<script>
external_info website;echo '<b>Please note:</b> Today maintainance on WebServer1,<br>Production only on WebServer2'
或者：external_info website;/path/to/your/script.sh
【说明】对于未用显示0（display 0）定义的每个业务流程，可以使用external_info后面可以加一行的脚本，但是该脚本必须打印一行到stdout显示提示信息，或者直接跟上脚本的路径也行
info_url  website;/more_info/website.html
或者：info_url website;http://some.other.site.com/more_info/website.html
【说明】对于未用显示0（display 0）定义的每个业务流程，也可以使用info_url，后面可以跟自己写的html页面也可以跟上外部的html页面

<bp_name> = <host>;<service> [& <host>;<service>]+
【说明】只要有一个服务是CRITIAL的，那么整个过程都是CRITIAL的（&）
```
检查nagios-bp.conf配置文件是否正确：
```ruby
# /usr/local/nagiosbp/bin/nagios-bp-consistency-check.pl
```

<h3 id="十">十、附：项目监控配置</h3>     

`监控端和被监控端都需要安装NRPE（有些插件如check_memory.pl是NRPE格式编写，所以必须结合nrpe插件使用）`

- 启动NRPE：/usr/local/nagios/bin/nrpe -c /usr/local/nagios/etc/nrpe.cfg -d
```ruby
# chmod 755 check_memory.pl
# /usr/local/nagios/libexec/check_memory.pl -f -w 40 -c 20    
  // -f是free memory  -u是used memory  ，剩余40%以下是警告，剩余20%以下是严重

# yum install -y bc
# chmod 755 check_cpu.sh 
# /usr/local/nagios/libexec/check_cpu.sh  -w 60 -c 90    
  // 超过60%是警告，超过90%是严重

# /usr/local/nagios/libexec/check_disk -w 60% -c 20% -p /
  // 剩余空间剩余60%以下会警告，剩余20%以下会严重 ，-p指定分区

# /usr/local/nagios/libexec/check_http -I 172.30.105.114 -p 80 -u /test.html -e 200
  // 在http或者tomcat的项目根目录下写了一个测试页面

# yum install -y perl-devel perl-CPAN perl-Time-HiRes sysstat
# /usr/local/nagios/libexec/check_iostat -d sda1 -w 1000 -c 2000

# /usr/local/nagios/libexec/check_load -w 15,10,5 -c 30,25,20
# /usr/local/nagios/libexec/check_ping 172.30.105.113 -w 3000,80% -c 5000,100% -p 5
  // 3000毫秒的响应=30秒，丢包80%是警告，总共发送5个包

被监控端（nrpe.cfg）
command[check_memory]=/usr/local/nagios/libexec/check_memory.pl -f -w 40 -c 20
command[check_cpu]=/usr/local/nagios/libexec/check_cpu.sh  -w 60 -c 90 
command[check_disk]=/usr/local/nagios/libexec/check_disk -w 60% -c 20% -p /
command[check_http]=/usr/local/nagios/libexec/check_http -I 172.30.105.114 -p 80 -u /test.html -e 200
command[check_iostat]=/usr/local/nagios/libexec/check_iostat -d sda -w 1000 -c 2000
【说明】check_iostat -d sda1不要写分区，直接写硬盘名称就行check_iostat -d sda
command[check_load]=/usr/local/nagios/libexec/check_load -w 15,10,5 -c 30,25,20
```
- 监控CPU：  
```ruby

在监控端配置：
define command{
        command_name    check_local_cpu
        command_line    $USER1$/check_cpu.sh -w $ARG1$ -c $ARG2$
        }
```
- 监控内存：  
```ruby

在监控端配置：
 define command {
			command_name		check_nrpe
			command_line		$USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
}

# vim nrpe.cfg（监控端和被监控端都需要添加下面的command）
command[check_memory]=/usr/local/nagios/libexec/check_memory.pl -f -w 40 -c 20
在监控端配置：
define service {
        name                            check_mem
        use                             local-service
        host_name                       172.30.105.114
        service_description             MemInfo
        check_command                   check_nrpe!check_memory    
        }
【说明】不需要在objects/commands.cfg文件中再定义define command的memory，因为在objects/commands.cfg中已经定义了check_nrpe -c后面就可以接检测插件
```
- 监控硬盘：   
```ruby
【注意】在编译NRPE的时候必须指明--enable-command-args参数，而且nrpe.cfg中需要设置dont_blame_nrpe=1

在监控端配置：
define command{
        command_name    check_local_disk
        command_line    $USER1$/check_disk -w $ARG1$ -c $ARG2$ -p $ARG3$
        }

define service{
        use                             local-service
        host_name                       172.30.105.114
        service_description             Root Partition
        check_command                   check_local_disk!99%!96%!/
        }
【注意】实验证实，这样配置监控出来的数据是监控端的数据，而不是被监控端的数据，而且check_disk不支持 -H hostname的参数，所以只能借用check_nrpe插件


# vim nrpe.cfg（监控端和被监控端都需要添加下面的command）
command[check_disk]=/usr/local/nagios/libexec/check_disk -w $ARG1$ -c $ARG2$ -p $ARG3$

在监控端配置：
# vim /usr/local/nagios/etc/objects/commands.cfg
define command{
        command_name    check_diskk
        command_line    $USER1$/check_nrpe -H $HOSTADDRESS$ -c check_disk -a $ARG1$  $ARG2$  $ARG3$
        }
【说明】这样定义一个command是为了在监控定义服务的时候，不使用check_nrpe!check_disk形式，可以直接在define service中使用check_disk!99%!96%!/

在监控端配置：
define service {
    use                                 local-service
    host_name                           172.30.105.114
    service_description                 check_disk_114
    check_command                       check_diskk!99%!96%!/
        }
【注意】99%和96%数值是测试告警使用
```