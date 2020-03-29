## Nagios基本安装

[Nagios的相关软件包下载1](https://sourceforge.net/projects/nagios/files/)  
[Nagios的相关软件包下载2](https://www.nagios.org/downloads/nagios-core-addons/)   
[Nagios的相关软件包下载3](http://nagios-plugins.org/download/)    
[Nagios的插件下载](https://labs.consol.de/nagios/)     

### 1、解决安装Nagios的依赖关系：
`Nagios基本组件的运行依赖于httpd、gcc和gd。可以通过以下命令来检查nagios所依赖的rpm包是否已经完全安装：`
```js
# yum -y install httpd gcc glibc glibc-common gd gd-devel php php-mysql mysql mysql-devel mysql-server vim
【说明】以上软件包您也可以通过编译源代码的方式安装，只是后面许多要用到的相关文件的路径等需要按照您的源代码安装时的配置逐一修改。此外，您还得按需启动必要的服务，如httpd等。
```
### 2、添加nagios运行所需要的用户和组：
```ruby
# groupadd  nagcmd
# useradd -G nagcmd nagios
# passwd nagios

把apache加入到nagcmd组，以便于在通过web Interface操作nagios时能够具有足够的权限：
# usermod -a -G nagcmd apache

```

### 3、编译安装nagios：
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
// 修改为 email                           479414941@qq.com

创建一个登录nagios web程序的用户，这个用户帐号在以后通过web登录nagios认证时所用：
// 增加nagios登陆认证文件，一定要用默认的nagiosadmin作为用户，否则需要修改其他文件
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

### 4\编译、安装nagios-plugins
```ruby
nagios的所有监控工作都是通过插件完成的，因此，在启动nagios之前还需要为其安装官方提供的插件。
# tar zxf nagios-plugins-2.1.4.tar.gz 
# cdnagios-plugins-2.1.4
# ./configure --with-nagios-user=nagios --with-nagios-group=nagios
# make
# make install
```

### 5、配置并启动Nagios

#### 5.1 把nagios添加为系统服务并将之加入到自动启动服务队列：
```
# chkconfig --add nagios
# chkconfig nagios on
```
#### 5.2 检查其主配置文件的语法是否正确：
```js
# /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg

```

#### 5.3 如果上面的语法检查没有问题，接下来就可以正式启动nagios服务了：
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