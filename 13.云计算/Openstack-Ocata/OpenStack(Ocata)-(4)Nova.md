## OpenStack(Ocata)-在controller节点安装Nova


### 1、创建nova数据库：
```ruby
[root@controller ~]# mysql -u root -p
MariaDB [(none)]> CREATE DATABASE nova_api;
MariaDB [(none)]> CREATE DATABASE nova;
MariaDB [(none)]> CREATE DATABASE nova_cell0;

```
### 2、创建数据库用户并赋予权限:
```ruby
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'wangzy';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'wangzy';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'wangzy';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'wangzy';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' IDENTIFIED BY 'wangzy';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY 'wangzy';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO 'root'@'controller' IDENTIFIED BY 'wangzy';
MariaDB [(none)]> FLUSH PRIVILEGES;
```
```ruby
// 查看授权表信息:
SELECT DISTINCT CONCAT('User: ''',user,'''@''',host,''';') AS query FROM mysql.user;

// 取消之前某个授权:
REVOKE ALTER ON *.* TO 'root'@'controller' IDENTIFIED BY 'wangzy';
```

### 3、创建nova用户及赋予admin权限：
```ruby
[root@controller ~]# . admin-openrc
[root@controller ~]# openstack user create --domain default nova --password wangzy
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 3695b93b1efa4fa1a3457b70aca88903 |
| name                | nova                             |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

[root@controller ~]# openstack role add --project service --user nova admin

```
### 4、创建computer服务:
```ruby
[root@controller ~]# openstack service create --name nova --description "OpenStack Compute" compute
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Compute                |
| enabled     | True                             |
| id          | c2bb8447d31943a9ab2b9c8125efce01 |
| name        | nova                             |
| type        | compute                          |
+-------------+----------------------------------+

```
### 5、创建nova的endpoint:
```ruby
[root@controller ~]# openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1/%\(tenant_id\)s
+--------------+-------------------------------------------+
| Field        | Value                                     |
+--------------+-------------------------------------------+
| enabled      | True                                      |
| id           | 789aee9de4814613b0bf45fb01898ce3          |
| interface    | public                                    |
| region       | RegionOne                                 |
| region_id    | RegionOne                                 |
| service_id   | c2bb8447d31943a9ab2b9c8125efce01          |
| service_name | nova                                      |
| service_type | compute                                   |
| url          | http://controller:8774/v2.1/%(tenant_id)s |
+--------------+-------------------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1/%\(tenant_id\)s
+--------------+-------------------------------------------+
| Field        | Value                                     |
+--------------+-------------------------------------------+
| enabled      | True                                      |
| id           | d9262941630342f589e419670d394999          |
| interface    | internal                                  |
| region       | RegionOne                                 |
| region_id    | RegionOne                                 |
| service_id   | c2bb8447d31943a9ab2b9c8125efce01          |
| service_name | nova                                      |
| service_type | compute                                   |
| url          | http://controller:8774/v2.1/%(tenant_id)s |
+--------------+-------------------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1/%\(tenant_id\)s
+--------------+-------------------------------------------+
| Field        | Value                                     |
+--------------+-------------------------------------------+
| enabled      | True                                      |
| id           | 867dfa18fe334fae948e7de177dca3b4          |
| interface    | admin                                     |
| region       | RegionOne                                 |
| region_id    | RegionOne                                 |
| service_id   | c2bb8447d31943a9ab2b9c8125efce01          |
| service_name | nova                                      |
| service_type | compute                                   |
| url          | http://controller:8774/v2.1/%(tenant_id)s |
+--------------+-------------------------------------------+

```
### 6、安装nova相关软件：
```ruby
[root@controller ~]# yum install -y openstack-nova-api openstack-nova-conductor openstack-nova-cert openstack-nova-console openstack-nova-novncproxy openstack-nova-scheduler
```

### 7、配置nova的配置文件/etc/nova/nova.conf:
```ruby
# cp /etc/nova/nova.conf /etc/nova/nova.conf.bak
# >/etc/nova/nova.conf
# openstack-config --set /etc/nova/nova.conf DEFAULT enabled_apis osapi_compute,metadata
# openstack-config --set /etc/nova/nova.conf DEFAULT auth_strategy keystone
# openstack-config --set /etc/nova/nova.conf DEFAULT my_ip 172.25.253.104
# openstack-config --set /etc/nova/nova.conf DEFAULT use_neutron True
# openstack-config --set /etc/nova/nova.conf DEFAULT firewall_driver nova.virt.firewall.NoopFirewallDriver
# openstack-config --set /etc/nova/nova.conf DEFAULT transport_url rabbit://openstack:wangzy@controller
# openstack-config --set /etc/nova/nova.conf database connection mysql+pymysql://nova:wangzy@controller/nova
# openstack-config --set /etc/nova/nova.conf api_database connection mysql+pymysql://nova:wangzy@controller/nova_api
# openstack-config --set /etc/nova/nova.conf scheduler discover_hosts_in_cells_interval -1
# openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_uri http://controller:5000
# openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_url http://controller:35357
# openstack-config --set /etc/nova/nova.conf keystone_authtoken memcached_servers controller:11211
# openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_type password
# openstack-config --set /etc/nova/nova.conf keystone_authtoken project_domain_name default
# openstack-config --set /etc/nova/nova.conf keystone_authtoken user_domain_name default
# openstack-config --set /etc/nova/nova.conf keystone_authtoken project_name service
# openstack-config --set /etc/nova/nova.conf keystone_authtoken username nova
# openstack-config --set /etc/nova/nova.conf keystone_authtoken password wangzy
# openstack-config --set /etc/nova/nova.conf keystone_authtoken service_token_roles_required True
# openstack-config --set /etc/nova/nova.conf vnc vncserver_listen 172.25.253.104
# openstack-config --set /etc/nova/nova.conf vnc vncserver_proxyclient_address 172.25.253.104
# openstack-config --set /etc/nova/nova.conf glance api_servers http://controller:9292
# openstack-config --set /etc/nova/nova.conf oslo_concurrency lock_path /var/lib/nova/tmp
```

### 8、设置cell（单元格）：

&emsp;&emsp;OpenStack 在控制平面上的性能瓶颈主要在 Message Queue 和 Database 。尤其是 Message Queue , 随着计算节点的增加，性能变的越来越差，因为openstack里每个资源和接口都是通过消息队列来通信的，有测试表明，当集群规模到了200，一个消息可能要在十几秒后才会响应；为了应对这种情况，引入Cells功能以解决OpenStack集群的扩展性。

#### 8.1 同步下nova数据库：
```ruby
[root@controller ~]# su -s /bin/sh -c "nova-manage api_db sync" nova
[root@controller ~]# su -s /bin/sh -c "nova-manage db sync" nova
```
#### 8.2 设置cell_v2关联上创建好的数据库nova_cell0:
```ruby
[root@controller ~]# nova-manage cell_v2 map_cell0 \
> --database_connection mysql+pymysql://root:wangzy@controller/nova_cell0

```
#### 8.3 创建一个常规cell(cell1)，这个单元格里面将会包含计算节点:
```ruby
[root@controller ~]# nova-manage cell_v2 create_cell \
> --verbose \
> --name cell1 \
> --database_connection mysql+pymysql://root:wangzy@controller/nova_cell0 \
> --transport-url rabbit://openstack:wangzy@controller:5672/
886f26d8-4b52-47f2-b48b-729ca41ed9f3
```
#### 8.4 检查部署是否正常:
```ruby
[root@controller ~]# nova-status upgrade check
```
#### 8.5 创建和映射cell0，并将现有计算主机和实例映射到单元格中:
```ruby
[root@controller ~]# nova-manage cell_v2 simple_cell_setup
```
#### 8.6 查看已经创建好的单元格列表：
```ruby
[root@controller ~]# nova-manage cell_v2 list_cells --verbose
```
`如果有新添加的计算节点，需要运行下面命令来发现，并且添加到单元格中`
```ruby
[root@controller ~] nova-manage cell_v2 discover_hosts

// 也可以在控制节点的nova.conf文件里[scheduler]模块下添加 discover_hosts_in_cells_interval=-1 这个设置来自动发现
```
#### 8.7 FAQ：
```ruby
// 创建虚拟机后出现错误提示
Host 'compute' is not mapped to any cell

只在控制节点执行就行：
[root@controller ~]# nova-manage cell_v2 map_cell_and_hosts
[root@controller ~]# nova-manage cell_v2 discover_hosts

```

### 9、安装placement：
`从Ocata开始，需要安装配置placement参与nova调度了，不然虚拟机将无法创建`
```ruby
[root@controller ~]# yum install -y openstack-nova-placement-api

```
#### 9.1 创建placement用户和placement服务:
```ruby
[root@controller ~]# openstack user create --domain default placement --password wangzy
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 3ce68e71f7a742ee8c273bb8fe228ca1 |
| name                | placement                        |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
[root@controller ~]# openstack role add --project service --user placement admin

[root@controller ~]# openstack service create --name placement --description "OpenStack Placement" placement
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Placement              |
| enabled     | True                             |
| id          | 0d57c060f754475c8e167f276c5c6f0f |
| name        | placement                        |
| type        | placement                        |
+-------------+----------------------------------+
```
#### 9.2 创建placement endpoint:
```ruby
[root@controller ~]# openstack endpoint create --region RegionOne placement public http://controller:8778
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 615f659af90a4d5baeec80d5291874eb |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 0d57c060f754475c8e167f276c5c6f0f |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+
[root@controller ~]# openstack endpoint create --region RegionOne placement admin http://controller:8778
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | a262d30bff6a40808d7f222e2db09ab0 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 0d57c060f754475c8e167f276c5c6f0f |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+
[root@controller ~]# openstack endpoint create --region RegionOne placement internal http://controller:8778
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | e2d3f211ee22460cba6f32eb59126c78 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 0d57c060f754475c8e167f276c5c6f0f |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+
```
#### 9.3 把placement 整合到nova.conf里:
```ruby
# openstack-config --set /etc/nova/nova.conf placement auth_url http://controller:35357
# openstack-config --set /etc/nova/nova.conf placement memcached_servers controller:11211
# openstack-config --set /etc/nova/nova.conf placement auth_type password
# openstack-config --set /etc/nova/nova.conf placement project_domain_name default
# openstack-config --set /etc/nova/nova.conf placement user_domain_name default
# openstack-config --set /etc/nova/nova.conf placement project_name service
# openstack-config --set /etc/nova/nova.conf placement username nova
# openstack-config --set /etc/nova/nova.conf placement password wangzy
# openstack-config --set /etc/nova/nova.conf placement os_region_name RegionOne
```
#### 9.4 配置修改00-nova-placement-api.conf文件:
`这步没做创建虚拟机的时候会出现禁止访问资源的问题`
```ruby
[root@controller ~]# cd /etc/httpd/conf.d/
[root@controller ~]# cp 00-nova-placement-api.conf 00-nova-placement-api.conf.bak
[root@controller ~]# >00-nova-placement-api.conf
[root@controller ~]# vim 00-nova-placement-api.conf
添加以下内容：
Listen 8778

<VirtualHost *:8778>
WSGIProcessGroup nova-placement-api
WSGIApplicationGroup %{GLOBAL}
WSGIPassAuthorization On
WSGIDaemonProcess nova-placement-api processes=3 threads=1 user=nova group=nova
WSGIScriptAlias / /usr/bin/nova-placement-api
<Directory "/">
Order allow,deny
Allow from all
Require all granted
</Directory>
<IfVersion >= 2.4>
ErrorLogFormat "%M"
</IfVersion>
ErrorLog /var/log/nova/nova-placement-api.log
</VirtualHost>

Alias /nova-placement-api /usr/bin/nova-placement-api
<Location /nova-placement-api>
SetHandler wsgi-script
Options +ExecCGI
WSGIProcessGroup nova-placement-api
WSGIApplicationGroup %{GLOBAL}
WSGIPassAuthorization On
</Location>
```
#### 9.5 重启httpd服务并检查配置：
```ruby
[root@controller ~]# systemctl restart httpd
[root@controller ~]# nova-status upgrade check
```

### 10 设置nova相关服务开机启动
```ruby
[root@controller ~]# systemctl enable openstack-nova-api.service \
openstack-nova-cert.service \
openstack-nova-consoleauth.service  \
openstack-nova-scheduler.service \
openstack-nova-conductor.service \
openstack-nova-novncproxy.service

[root@controller ~]# systemctl restart openstack-nova-api.service \
openstack-nova-cert.service \
openstack-nova-consoleauth.service  \
openstack-nova-scheduler.service \
openstack-nova-conductor.service \
openstack-nova-novncproxy.service

[root@controller ~]# systemctl status openstack-nova-api.service \
openstack-nova-cert.service  \
openstack-nova-consoleauth.service  \
openstack-nova-scheduler.service \
openstack-nova-conductor.service \
openstack-nova-novncproxy.service

[root@controller ~]# systemctl list-unit-files |grep openstack-nova-*
```

### 11、验证nova服务：
```ruby
[root@controller ~]# unset OS_TOKEN OS_URL
[root@controller ~]# source /root/admin-openrc
[root@controller ~]# nova service-list 
+----+------------------+------------+----------+---------+-------+----------------------------+-----------------+
| Id | Binary           | Host       | Zone     | Status  | State | Updated_at                 | Disabled Reason |
+----+------------------+------------+----------+---------+-------+----------------------------+-----------------+
| 1  | nova-scheduler   | controller | internal | enabled | up    | 2018-07-02T08:07:59.000000 | -               |
| 2  | nova-consoleauth | controller | internal | enabled | up    | 2018-07-02T08:08:00.000000 | -               |
| 3  | nova-conductor   | controller | internal | enabled | up    | 2018-07-02T08:08:01.000000 | -               |
| 4  | nova-cert        | controller | internal | enabled | up    | 2018-07-02T08:07:57.000000 | -               |
| 7  | nova-compute     | compute    | nova     | enabled | up    | 2018-07-02T08:07:58.000000 | -               |
+----+------------------+------------+----------+---------+-------+----------------------------+-----------------+

[root@controller ~]# openstack endpoint list
+------------------------------+-----------+--------------+--------------+---------+-----------+-------------------------------+
| ID                           | Region    | Service Name | Service Type | Enabled | Interface | URL                           |
+------------------------------+-----------+--------------+--------------+---------+-----------+-------------------------------+
| 2626db9a6bce49e89319de15486e | RegionOne | neutron      | network      | True    | internal  | http://controller:9696        |
| c754                         |           |              |              |         |           |                               |
| 48fadc4396a24f8db999ed74d924 | RegionOne | neutron      | network      | True    | admin     | http://controller:9696        |
| 1110                         |           |              |              |         |           |                               |
| 4b841bbb69344bb3999b8e5f6ad0 | RegionOne | cinder       | volume       | True    | public    | http://controller:8776/v1/%(t |
| e8e0                         |           |              |              |         |           | enant_id)s                    |
| 54d197c1378445c882bee5280e6d | RegionOne | glance       | image        | True    | admin     | http://controller:9292        |
| 6419                         |           |              |              |         |           |                               |
| 55fa86db2d19402d89ae08612e4d | RegionOne | glance       | image        | True    | public    | http://controller:9292        |
| e620                         |           |              |              |         |           |                               |
| 5744259d5671450a9d2e09f7cb86 | RegionOne | keystone     | identity     | True    | internal  | http://controller:35357/v3    |
| 2c7b                         |           |              |              |         |           |                               |
| 615f659af90a4d5baeec80d52918 | RegionOne | placement    | placement    | True    | public    | http://controller:8778        |
| 74eb                         |           |              |              |         |           |                               |
| 65ec43a0aec74471809a877cb2cf | RegionOne | cinder       | volume       | True    | internal  | http://controller:8776/v1/%(t |
| 53dc                         |           |              |              |         |           | enant_id)s                    |
| 6fb57977be3f47d694028f3b2be4 | RegionOne | keystone     | identity     | True    | admin     | http://controller:35357/v3    |
| cb57                         |           |              |              |         |           |                               |
| 789aee9de4814613b0bf45fb0189 | RegionOne | nova         | compute      | True    | public    | http://controller:8774/v2.1/% |
| 8ce3                         |           |              |              |         |           | (tenant_id)s                  |
| 8622974987ae43e69891408fb011 | RegionOne | cinderv2     | volumev2     | True    | internal  | http://controller:8776/v2/%(t |
| 9fa9                         |           |              |              |         |           | enant_id)s                    |
| 867dfa18fe334fae948e7de177dc | RegionOne | nova         | compute      | True    | admin     | http://controller:8774/v2.1/% |
| a3b4                         |           |              |              |         |           | (tenant_id)s                  |
| 8cbcbda84c934d84ac31917265aa | RegionOne | glance       | image        | True    | internal  | http://controller:9292        |
| 2f3d                         |           |              |              |         |           |                               |
| 965fdd6d75c04885a8fe829d7891 | RegionOne | cinder       | volume       | True    | admin     | http://controller:8776/v1/%(t |
| ba25                         |           |              |              |         |           | enant_id)s                    |
| a262d30bff6a40808d7f222e2db0 | RegionOne | placement    | placement    | True    | admin     | http://controller:8778        |
| 9ab0                         |           |              |              |         |           |                               |
| bf5958417249441e83643ecb892d | RegionOne | keystone     | identity     | True    | public    | http://controller:5000/v3     |
| 4eae                         |           |              |              |         |           |                               |
| d9262941630342f589e419670d39 | RegionOne | nova         | compute      | True    | internal  | http://controller:8774/v2.1/% |
| 4999                         |           |              |              |         |           | (tenant_id)s                  |
| d9dfc2101d254c78b5de8cd55c6b | RegionOne | cinderv2     | volumev2     | True    | public    | http://controller:8776/v2/%(t |
| 5238                         |           |              |              |         |           | enant_id)s                    |
| e0121c607e0144c8a307ffaa212f | RegionOne | neutron      | network      | True    | public    | http://controller:9696        |
| 4fdb                         |           |              |              |         |           |                               |
| e218d07919fe411ea587ae1b4804 | RegionOne | cinderv2     | volumev2     | True    | admin     | http://controller:8776/v2/%(t |
| fcf1                         |           |              |              |         |           | enant_id)s                    |
| e2d3f211ee22460cba6f32eb5912 | RegionOne | placement    | placement    | True    | internal  | http://controller:8778        |
| 6c78                         |           |              |              |         |           |                               |
+------------------------------+-----------+--------------+--------------+---------+-----------+-------------------------------+
```


## OpenStack(Ocata)-在Compute节点安装Nova

### 1、安装依赖软件包：
```ruby
[root@compute ~]# yum install openstack-selinux python-openstackclient yum-plugin-priorities openstack-nova-compute openstack-utils ntpdate -y
```
### 2、配置nova.conf：
```ruby
# cp /etc/nova/nova.conf /etc/nova/nova.conf.bak
# >/etc/nova/nova.conf
# openstack-config --set /etc/nova/nova.conf DEFAULT auth_strategy keystone
# openstack-config --set /etc/nova/nova.conf DEFAULT my_ip 172.25.253.105
# openstack-config --set /etc/nova/nova.conf DEFAULT use_neutron True
# openstack-config --set /etc/nova/nova.conf DEFAULT firewall_driver nova.virt.firewall.NoopFirewallDriver
# openstack-config --set /etc/nova/nova.conf DEFAULT transport_url rabbit://openstack:wangzy@controller
# openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_uri http://controller:5000
# openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_url http://controller:35357
# openstack-config --set /etc/nova/nova.conf keystone_authtoken memcached_servers controller:11211
# openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_type password
# openstack-config --set /etc/nova/nova.conf keystone_authtoken project_domain_name default
# openstack-config --set /etc/nova/nova.conf keystone_authtoken user_domain_name default
# openstack-config --set /etc/nova/nova.conf keystone_authtoken project_name service
# openstack-config --set /etc/nova/nova.conf keystone_authtoken username nova
# openstack-config --set /etc/nova/nova.conf keystone_authtoken password wangzy
# openstack-config --set /etc/nova/nova.conf placement auth_uri http://controller:5000
# openstack-config --set /etc/nova/nova.conf placement auth_url http://controller:35357
# openstack-config --set /etc/nova/nova.conf placement memcached_servers controller:11211
# openstack-config --set /etc/nova/nova.conf placement auth_type password 
# openstack-config --set /etc/nova/nova.conf placement project_domain_name default
# openstack-config --set /etc/nova/nova.conf placement user_domain_name default
# openstack-config --set /etc/nova/nova.conf placement project_name service
# openstack-config --set /etc/nova/nova.conf placement username nova
# openstack-config --set /etc/nova/nova.conf placement password wangzy
# openstack-config --set /etc/nova/nova.conf placement os_region_name RegionOne
# openstack-config --set /etc/nova/nova.conf vnc enabled True
# openstack-config --set /etc/nova/nova.conf vnc keymap en-us
# openstack-config --set /etc/nova/nova.conf vnc vncserver_listen 0.0.0.0
# openstack-config --set /etc/nova/nova.conf vnc vncserver_proxyclient_address 10.1.1.151
# openstack-config --set /etc/nova/nova.conf vnc novncproxy_base_url http://172.25.101.104:6080/vnc_auto.html
# openstack-config --set /etc/nova/nova.conf glance api_servers http://controller:9292
# openstack-config --set /etc/nova/nova.conf oslo_concurrency lock_path /var/lib/nova/tmp
# openstack-config --set /etc/nova/nova.conf libvirt virt_type qemu
```

### 3、确认主机是否支持硬件加速：
```ruby
[root@compute ~]# egrep -c '(vmx|svm)' /proc/cpuinfo
16

// 如果返回一个空值，那么你的计算节点不支持硬件加速
// 必须配置/etc/nova/nova.conf这个配置文件的libvirt选项使用QEMU来代替KVM

openstack-config --set /etc/nova/nova.conf libvirt virt_type qemu
```
### 4、设置服务开机自启动:
```ruby
[root@compute ~]# systemctl enable libvirtd.service openstack-nova-compute.service
[root@compute ~]# systemctl restart libvirtd.service openstack-nova-compute.service
[root@compute ~]# systemctl status libvirtd.service openstack-nova-compute.service

```
### 5、到controller上执行验证：
```ruby
[root@compute ~]# source /root/admin-openrc
[root@compute ~]# openstack compute service list
[root@controller ~]# openstack compute service list
+----+------------------+------------+----------+---------+-------+----------------------------+
| ID | Binary           | Host       | Zone     | Status  | State | Updated At                 |
+----+------------------+------------+----------+---------+-------+----------------------------+
|  1 | nova-scheduler   | controller | internal | enabled | up    | 2018-07-02T08:16:09.000000 |
|  2 | nova-consoleauth | controller | internal | enabled | up    | 2018-07-02T08:16:10.000000 |
|  3 | nova-conductor   | controller | internal | enabled | up    | 2018-07-02T08:16:11.000000 |
|  4 | nova-cert        | controller | internal | enabled | up    | 2018-07-02T08:16:07.000000 |
|  7 | nova-compute     | compute    | nova     | enabled | up    | 2018-07-02T08:16:08.000000 |
+----+------------------+------------+----------+---------+-------+----------------------------+

```


## OpenStack(Ocata)-在controller节点上进行验证性操作

### 1、添加compute节点到cell数据库:

```ruby
[root@controller ~]# . admin-openrc 
[root@controller ~]# openstack hypervisor list
+----+---------------------+-----------------+---------------+-------+
| ID | Hypervisor Hostname | Hypervisor Type | Host IP       | State |
+----+---------------------+-----------------+---------------+-------+
|  1 | compute             | QEMU            | 111.32.138.22 | up    |
+----+---------------------+-----------------+---------------+-------+

```
```ruby
[root@controller ~]# openstack compute service list --service nova-compute
+----+--------------+---------+------+---------+-------+----------------------------+
| ID | Binary       | Host    | Zone | Status  | State | Updated At                 |
+----+--------------+---------+------+---------+-------+----------------------------+
| 10 | nova-compute | compute | nova | enabled | up    | 2018-06-19T02:29:36.000000 |
+----+--------------+---------+------+---------+-------+----------------------------+

```
### 2、发现计算节点：
```ruby
[root@controller ~]# nova-manage cell_v2 discover_hosts 

// 当添加新的计算节点的时候，你必须执行nova-manage cell_v2 discover_hosts在controller节点来发现新的计算节点
// 另外，可以在/etc/nova/nova.conf配置文件中设置时间间隔

[scheduler]
discover_hosts_in_cells_interval = 300
```

### 3、在controller节点验证计算服务操作
#### 3.1 列出服务组件：
```ruby
[root@controller ~]# . admin-openrc 
[root@controller ~]# openstack compute service list
+----+------------------+------------+----------+---------+-------+----------------------------+
| ID | Binary           | Host       | Zone     | Status  | State | Updated At                 |
+----+------------------+------------+----------+---------+-------+----------------------------+
|  1 | nova-consoleauth | controller | internal | enabled | up    | 2018-06-19T02:31:21.000000 |
|  2 | nova-scheduler   | controller | internal | enabled | up    | 2018-06-19T02:31:20.000000 |
|  3 | nova-conductor   | controller | internal | enabled | up    | 2018-06-19T02:31:20.000000 |
| 10 | nova-compute     | compute    | nova     | enabled | up    | 2018-06-19T02:31:17.000000 |
+----+------------------+------------+----------+---------+-------+----------------------------+

// 输出信息应该是在控制节点上启用三个服务组件，在计算节点上启动一个服务组件
```
#### 2.2 列出身份服务中的API端点以验证与身份服务的连接：
```ruby
[root@controller ~]# openstack catalog list
+-----------+-----------+-----------------------------------------+
| Name      | Type      | Endpoints                               |
+-----------+-----------+-----------------------------------------+
| keystone  | identity  | RegionOne                               |
|           |           |   internal: http://controller:5000/v3/  |
|           |           | RegionOne                               |
|           |           |   admin: http://controller:35357/v3/    |
|           |           | RegionOne                               |
|           |           |   public: http://controller:5000/v3/    |
|           |           |                                         |
| glance    | image     | RegionOne                               |
|           |           |   public: http://controller:9292        |
|           |           | RegionOne                               |
|           |           |   internal: http://controller:9292      |
|           |           | RegionOne                               |
|           |           |   admin: http://controller:9292         |
|           |           |                                         |
| placement | placement | RegionOne                               |
|           |           |   admin: http://controller:8778         |
|           |           | RegionOne                               |
|           |           |   public: http://controller:8778        |
|           |           | RegionOne                               |
|           |           |   internal: http://controller:8778      |
|           |           |                                         |
| nova      | compute   | RegionOne                               |
|           |           |   public: http://controller:8774/v2.1   |
|           |           | RegionOne                               |
|           |           |   admin: http://controller:8774/v2.1    |
|           |           | RegionOne                               |
|           |           |   internal: http://controller:8774/v2.1 |
|           |           |                                         |
+-----------+-----------+-----------------------------------------+

```
#### 2.3 列出镜像:

```ruby
[root@controller ~]# openstack image list
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| 10e41719-99ba-4a3d-86b9-eef167a740e6 | cirros | active |
+--------------------------------------+--------+--------+
```
#### 2.4 检查cells和placement API是否正常:
```ruby
[root@controller ~]# nova-status upgrade check
+---------------------------+
| Upgrade Check Results     |
+---------------------------+
| Check: Cells v2           |
| Result: 成功              |
| Details: None             |
+---------------------------+
| Check: Placement API      |
| Result: 成功              |
| Details: None             |
+---------------------------+
| Check: Resource Providers |
| Result: 成功              |
| Details: None             |
+---------------------------+

```