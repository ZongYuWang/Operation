## PNP4Nagios的安装

&emsp;&emsp;PHP4Nagios有三种工作模式，分别是Synchronous Mode、Bulk Mode和Bulk Mode with NPCD；     
本实验使用Bulk Mode方式

### 1、安装依赖包：
```ruby
# yum install -y rrdtool perl-Time-HiRes perl-devel perl-CPAN
```
#### 2、安装pnp4nagios：
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
#### 3、配置pnp4nagios：
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
#### 4、修改pnp4nagios使用Bulk Mode的方式：
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
#### 4.1 修改commands.cfg、templates.cfg、localhost.cfg（主机配置文件）：
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
#### 4.2 修改RRD文件路径的配置文件：
```ruby
# vim /usr/local/pnp4nagios/etc/config.php 
$conf['rrdbase'] = "/usr/local/pnp4nagios/var/perfdata/";

# vim /usr/local/pnp4nagios/etc/process_perfdata.cfg
RRDPATH = /usr/local/pnp4nagios/var/perfdata
```
#### 4.3 检查pnp4nagios是否安装正确：
```ruby
http://192.168.1.111/pnp4nagios/
# mv /usr/local/pnp4nagios/share/install.php /usr/local/pnp4nagios/share/install.php.bak
```
### 5、添加Nagios监控图像：
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

![](https://github.com/ZongYuWang/image/tree/master/Nagios/Nagios-pnp4nagios1.png)

```ruby
cp /nagios_soft/pnp4nagios-0.6.6/contrib/ssi/status-header.ssi /usr/local/nagios/share/ssi/
【注意】status-header.ssi必须没有执行权限

```
### 6、修改Nagios的模板文件：
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
### 7、检查配置文件并启动
#### 7.1 检查配置文件是否正确：
```ruby
# /usr/local/nagios/bin/nrpe -c /usr/local/nagios/etc/nrpe.cfg -d
```

#### 7.2 重启npcd服务：
```ruby
# /etc/init.d/npcd restart 
```
#### 7.3 重启nagios服务：
```ruby
# service nagios restart
【注意】监控内存FREE的单位错误，单位应该是M，此处体现是K，但是数据正确
```