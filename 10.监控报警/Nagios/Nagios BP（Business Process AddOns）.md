## Nagios BP（Business Process AddOns）     

### 1、安装Nagios BP
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
### 2、运行测试结果：
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
internetconnection = internetconnection;Provider 1 | internetconnection;Provider 2    
// 网络连接，只有有一个运营商存活即可；
display 0;internetconnection;Internet Connection
		
loadbalancers = loadbalancer1;System Health | loadbalancer2;System Health      
// 两个主机的系统状况，这是负载均衡器，只要有一个存活即可
display 0;loadbalancers;Loadbalancer Cluster
		
dns = dns1;DNS | dns2;DNS | dns3;DNS    
// 三个DNS服务器，只要有一个存活即可
display 0;dns;DNS Cluster
		
website_webserver1 = webserver1;HTTP & webserver1;HTTPD Slots   
// website_webserver1主机的HTTP服务和HTTPD接口都正常
website_webserver2 = webserver2;HTTP & webserver2;HTTPD Slots
website_webservers = website_webserver1 | website_webserver2       
// 表示这个网站只要有一个网站服务器存活就行
website = internetconnection & loadbalancers & dns & website_webservers  
// 表示整个网站需要服务商&其中一台主机&其中一台DNS服务器&其中一个HTTP服务正常

display <x>;<bp_name>;<long_name> 		
display 0;website_webserver1;WebServer 1
display 0;website_webserver2;WebServer 2
display 0;website_webservers;WebServer Cluster
display 1;website;WebSite
// long_name是显示进程时使用的名称或说明，用户从未在GUI中看到<bp_name>，始终为<long_name>
// 0表示此过程不显示在顶级视图中，比如一些前提条件可以不用显示，最后要的结果可以显示出来，可以看下面的例子

external_info <bp_name>;<script>
external_info website;echo '<b>Please note:</b> Today maintainance on WebServer1,<br>Production only on WebServer2'
或者：external_info website;/path/to/your/script.sh
// 对于未用显示0（display 0）定义的每个业务流程，可以使用external_info后面可以加一行的脚本，但是该脚本必须打印一行到stdout显示提示信息，或者直接跟上脚本的路径也行
info_url  website;/more_info/website.html
或者：info_url website;http://some.other.site.com/more_info/website.html
// 对于未用显示0（display 0）定义的每个业务流程，也可以使用info_url，后面可以跟自己写的html页面也可以跟上外部的html页面

<bp_name> = <host>;<service> [& <host>;<service>]+
// 只要有一个服务是CRITIAL的，那么整个过程都是CRITIAL的（&）
```
检查nagios-bp.conf配置文件是否正确：
```ruby
# /usr/local/nagiosbp/bin/nagios-bp-consistency-check.pl
```