## OpenStack(ocata版)-部署前环境准备

`本文中涉及的密码均为：wangzy`

http://blog.51cto.com/liuleis/2094190
https://www.unixhot.com/sort_type-new__day-0__is_recommend-0__page-2

[ceph官网记录了openstack如何跟ceph结合使用](http://docs.ceph.com/docs/master/rbd/rbd-openstack/) 

<table>
    <tr>
        <td>环境说明</td> 
        <td>主机名</td> 
        <td>外部网络IP地址</td>
		<td>管理网络IP地址</td>
		<td>隧道网络IP地址</td>
		<td>存储网络IP地址</td>
    </tr>
    <tr>
        <td rowspan="3">OpenStack环境</td>    
        <td >controller</td> 
		<td >ens192:172.25.101.104</td>
		<td >ens224:172.25.253.104</td>
		<td >ens256:172.25.200.104</td>
		<td >集成CEPH使用</td>
    </tr>
    <tr>
        <td >compute</td> 
		<td >ens192:172.25.101.105</td>
		<td >ens224:172.25.253.105</td>
		<td >ens256:172.25.200.105</td> 
		<td >集成CEPH使用</td>

    </tr>
    <tr>
        <td >cinder</td>  
		<td >ens192:172.25.101.106</td>
		<td >ens224:172.25.253.106</td>
		<td >ens256:172.25.200.106</td>
		<td >集成CEPH使用</td>
    </tr>

    <tr>
        <td rowspan="3">CEPH存储环境</td>    
        <td >ceph-0</td>
		<td >bond0:172.25.1.139</td>
		<td > - </td>
		<td > -  </td>
		<td > -  </td>
    </tr>
    <tr>
        <td >ceph-1</td> 
		<td >bond0:172.25.1.140</td>
		<td > - </td>
		<td > - </td>
		<td > - </td>
    </tr>
    <tr>
        <td >ceph-2</td>
		<td >bond0:172.25.1.141</td>
		<td > - </td>
		<td > - </td>
		<td > - </td>
    </tr>
</table>
 

** &emsp;&emsp;网络名词解释：**
&emsp;&emsp;外网（external） : 这个网络是链接外网的，也就是说openstack环境里的虚拟机要让用户访问，那必须有个网段是连外网的，用户通过这个网络能访问到虚拟机。如果是搭建的公有云，这个IP段一般是公网的（不是公网，你让用户怎么访问你的虚拟机？）

&emsp;&emsp;管理网络（admin mgt）：这个网段是用来做管理网络的。管理网络，顾名思义，你的openstack环境里面各个模块之间需要交互，连接数据库，连接Message Queue都是需要一个网络去支撑的，那么这个网段就是这个作用。最简单的理解，openstack自己本身用的IP段。

&emsp;&emsp;隧道网络（tunnel）:openstack里面使用gre或者vxlan模式，需要有隧道网络；隧道网络采用了点到点通信协议代替了交换连接，在openstack里，这个tunnel就是虚拟机走网络数据流量用的。

&emsp;&emsp;三种网络在生产环境里是必须分开的，有的生产环境还有分布式存储，所以还得额外给存储再添加一网络，storage段。网络分开的好处就是数据分流、安全、不相互干扰

### 1、配置主机名(在每台主机上配置):
```ruby
# hostnamectl set-hostname controller / compute / cinder

```

```ruby
172.25.253.104 controller
172.25.253.105 compute
172.25.253.106 cinder

```

### 2、时钟同步(在每台主机上配置)：
```ruby
[root@controller ~]# yum install chrony -y 
[root@controller ~]# vim /etc/chrony.conf
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst
server controller iburst  
// 所有节点向controller节点同步时间
allow 172.25.16/24   // 设置时间同步网段

```
```ruby
[root@controller ~]# systemctl enable chronyd.service
[root@controller ~]# systemctl start chronyd.service
[root@controller ~]# systemctl status chronyd.service
```
```ruby
[root@compute ~]# yum install chrony -y
[root@compute ~]# vim /etc/chrony.conf
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst
server controller iburst
```
```ruby
[root@compute ~]# systemctl enable chronyd.service
[root@compute ~]# systemctl start chronyd.service

```
```ruby
[root@controller ~]# chronyc sources
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^? controller                    0  10     0     -     +0ns[   +0ns] +/-    0ns

// MS列中带* 为NTP服务当前同步的服务器。
```
```ruby
[root@compute ~]# chronyc sources
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^? controller                    0  10     0     -     +0ns[   +0ns] +/-    0ns

```
`日常运维中经常遇见时钟飘逸问题，导致集群服务脑裂`

### 3、配置OpenStack的YUM源(在每台主机上配置):

`不要yum update升级到7.3, Ocata版在7.3下依然有虚拟机启动出现iPXE启动问题`
```ruby
[root@controller ~]# yum install centos-release-openstack-ocata -y 
=====================================================================================
 Package                 架构                版本                           源                   大小
=====================================================================================
正在安装:
 centos-release-openstack-ocata         noarch          1-2.el7      extras
```
```ruby
# mv CentOS-QEMU-EV.repo CentOS-QEMU-EV.repo.backup
# mv CentOS-Ceph-Jewel.repo CentOS-Ceph-Jewel.repo.back
# yum upgrade
[root@controller ~]# vim /etc/yum.repos.d/CentOS-OpenStack-ocata.repo
[centos-openstack-ocata-source]
name=CentOS-7 - OpenStack ocata - Source
baseurl=http://vault.centos.org/centos/7/cloud/Source/openstack-ocata/
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-Cloud
exclude=sip,PyQt4

```
#### 3.1 安装工具软件包(在每台主机上配置)：
```ruby
# yum install net-tools wget vim ntpdate bash-completion openstack-utils -y

```


### 4、关闭防火墙和SElinux(在每台主机上配置)：
```ruby
# systemctl stop firewalld.service
# systemctl disable firewalld.service

# setenforce 0
# vim /etc/selinux/config
...
SELINUX=disabled

```

## OpenStack(Ocata)-Controller节点安装必备应用服务

### 1、搭建MariaDB

#### 1.1 安装mariadb数据库：
```ruby
[root@controller ~]# yum install mariadb mariadb-server python2-PyMySQL

```
#### 1.2 配置mariadb：
```ruby
[root@controller ~]# vim /etc/my.cnf.d/mariadb-openstack.cnf  // 自己创建该文件

[mysqld]
bind-address = 172.25.253.104
// bind-address使用controller节点的管理IP
default-storage-engine = innodb
// 设置默认的存储引擎
innodb_file_per_table = on
// 使用独享表空间
max_connections = 4096
// 设置MySQL的最大连接数，生产请根据实际情况设置
collation-server = utf8_general_ci
// 服务器的默认校对规则
character-set-server = utf8
// 服务器安装时指定的默认字符集设定
init-connect = 'SET NAMES utf8'

```
#### 1.3 启动数据库及设置mariadb开机启动：
```ruby
[root@controller ~]# systemctl enable mariadb.service
[root@controller ~]# systemctl restart mariadb.service
[root@controller ~]# systemctl status mariadb.service
[root@controller ~]# systemctl list-unit-files |grep mariadb.service

```
#### 1.4 配置mariadb，给mariadb设置密码
```ruby
[root@controller ~]# mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] 
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] 
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] 
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] 
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] 
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!

```
### 2、安装RabbitMQ

#### 2.1 安装erlang（此步在每台节点上配置）：
```ruby
# yum install -y erlang
```
#### 2.2 安装RabbitMQ（此步在每台节点上配置）:
```ruby
# yum install -y rabbitmq-server
```
#### 2.3 启动Rabbitmq及设置开机启动(此步在每台节点上配置)：
```ruby
# systemctl enable rabbitmq-server.service 
# systemctl restart rabbitmq-server.service 
# systemctl status rabbitmq-server.service
# systemctl list-unit-files |grep rabbitmq-server.service
	rabbitmq-server.service                       enabled
```

#### 2.4 创建Rabbit用户openstack(下面只需要在controller节点配置)：
```ruby
[root@controller ~]# rabbitmqctl add_user openstack wangzy
Creating user "openstack" ...
...done.
```
#### 2.5 将openstack用户赋予权限:
```ruby
[root@controller ~]# rabbitmqctl set_permissions openstack ".*" ".*" ".*"
Setting permissions for user "openstack" in vhost "/" ...
...done.

[root@controller ~]# rabbitmqctl set_user_tags openstack administrator
Setting tags for user "openstack" to [administrator] ...
...done.

[root@controller ~]# rabbitmqctl list_users
Listing users ...
guest	[administrator]
openstack	[administrator]
...done.

```
#### 2.6 看下监听端口 rabbitmq用的是5672端口:
```ruby
[root@controller ~]# ss -tnlp | grep 5672
```

#### 2.7 查看RabbitMQ插件：
```ruby
[root@controller ~]# /usr/lib/rabbitmq/bin/rabbitmq-plugins list
[ ] amqp_client                       3.3.5
[ ] cowboy                            0.5.0-rmq3.3.5-git4b93c2d
[ ] eldap                             3.3.5-gite309de4
[ ] mochiweb                          2.7.0-rmq3.3.5-git680dba8
[ ] rabbitmq_amqp1_0                  3.3.5
[ ] rabbitmq_auth_backend_ldap        3.3.5
[ ] rabbitmq_auth_mechanism_ssl       3.3.5
[ ] rabbitmq_consistent_hash_exchange 3.3.5
[ ] rabbitmq_federation               3.3.5
[ ] rabbitmq_federation_management    3.3.5
[ ] rabbitmq_management               3.3.5
[ ] rabbitmq_management_agent         3.3.5
[ ] rabbitmq_management_visualiser    3.3.5
[ ] rabbitmq_mqtt                     3.3.5
[ ] rabbitmq_shovel                   3.3.5
[ ] rabbitmq_shovel_management        3.3.5
[ ] rabbitmq_stomp                    3.3.5
[ ] rabbitmq_test                     3.3.5
[ ] rabbitmq_tracing                  3.3.5
[ ] rabbitmq_web_dispatch             3.3.5
[ ] rabbitmq_web_stomp                3.3.5
[ ] rabbitmq_web_stomp_examples       3.3.5
[ ] sockjs                            0.3.4-rmq3.3.5-git3132eb9
[ ] webmachine                        1.10.3-rmq3.3.5-gite9359c7
```
#### 2.8 打开RabbitMQ相关插件:
```ruby
/usr/lib/rabbitmq/bin/rabbitmq-plugins enable rabbitmq_management mochiweb webmachine rabbitmq_web_dispatch amqp_client rabbitmq_management_agent
The following plugins have been enabled:
  webmachine
  rabbitmq_web_dispatch
  rabbitmq_management_agent
  rabbitmq_management
  mochiweb
  amqp_client
Plugin configuration has changed. Restart RabbitMQ for changes to take effect.
```
#### 2.9 重启Rabbitmq并在浏览器中查看状态：
```ruby
# systemctl restart rabbitmq-server
```

浏览器输入： http://172.25.253.104:15672（默认的用户名和密码：guest/guest）
`可以使用上面创建的用户名和密码：openstack/wangzy`

![](https://github.com/ZongYuWang/image/blob/master/OpenStack/OpenStack-RabbitMQ1.png)


