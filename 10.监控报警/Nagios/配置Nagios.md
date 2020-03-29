## 配置Nagios

### 1、Nagios的主配置文件
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

### 2、resource_file和宏定义
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

### 3、定义命令对象
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
### 4、定义主机对象
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
### 5、定义服务对象
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

### 6、定义“时段”对象
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
### 7、定义联系人对象
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
### 8、模板及对象继承
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
### 9、依赖关系
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

### 10、FAQ：
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