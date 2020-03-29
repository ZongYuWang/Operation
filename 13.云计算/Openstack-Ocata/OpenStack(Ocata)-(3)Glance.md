## OpenStack(Ocata)-在controller节点安装Glance



### 1、创建并授权glance数据库：
```ruby
[root@controller ~]# mysql -u root -p
MariaDB [(none)]> CREATE DATABASE glance;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'wangzy';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%'  IDENTIFIED BY 'wangzy';

```
### 2、创建glance用户及赋予admin权限:
```ruby
[root@controller ~]# source /root/admin-openrc
[root@controller ~]# openstack user create --domain default glance --password wangzy
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 45d59bef73ef4e2486c411a0d7e1921e |
| name                | glance                           |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
[root@controller ~]# openstack role add --project service --user glance admin
```

### 3、创建image服务:
```ruby
[root@controller ~]# openstack service create --name glance --description "OpenStack Image service" image
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Image service          |
| enabled     | True                             |
| id          | 0de32ce543e34b72b425c0278fa7df10 |
| name        | glance                           |
| type        | image                            |
+-------------+----------------------------------+
```
### 4、创建glance的endpoint:
```ruby
[root@controller ~]# openstack endpoint create --region RegionOne image public http://controller:9292 
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 55fa86db2d19402d89ae08612e4de620 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 0de32ce543e34b72b425c0278fa7df10 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne image internal http://controller:9292 
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 8cbcbda84c934d84ac31917265aa2f3d |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 0de32ce543e34b72b425c0278fa7df10 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne image admin http://controller:9292
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 54d197c1378445c882bee5280e6d6419 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 0de32ce543e34b72b425c0278fa7df10 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+

```

### 5、安装安装glance相关软件包:
```ruby
[root@controller ~]# yum install openstack-glance -y

```
### 6、修改glance配置文件/etc/glance/glance-api.conf:
```ruby
# cp /etc/glance/glance-api.conf /etc/glance/glance-api.conf.bak
# >/etc/glance/glance-api.conf
# openstack-config --set /etc/glance/glance-api.conf DEFAULT transport_url rabbit://openstack:wangzy@controller
# openstack-config --set /etc/glance/glance-api.conf database connection mysql+pymysql://glance:wangzy@controller/glance 
# openstack-config --set /etc/glance/glance-api.conf keystone_authtoken auth_uri http://controller:5000 
# openstack-config --set /etc/glance/glance-api.conf keystone_authtoken auth_url http://controller:35357 
# openstack-config --set /etc/glance/glance-api.conf keystone_authtoken memcached_servers controller:11211 
# openstack-config --set /etc/glance/glance-api.conf keystone_authtoken auth_type password 
# openstack-config --set /etc/glance/glance-api.conf keystone_authtoken project_domain_name default 
# openstack-config --set /etc/glance/glance-api.conf keystone_authtoken user_domain_name default 
# openstack-config --set /etc/glance/glance-api.conf keystone_authtoken username glance 
# openstack-config --set /etc/glance/glance-api.conf keystone_authtoken password wangzy
# openstack-config --set /etc/glance/glance-api.conf keystone_authtoken project_name service
# openstack-config --set /etc/glance/glance-api.conf paste_deploy flavor keystone 
# openstack-config --set /etc/glance/glance-api.conf glance_store stores file,http 
# openstack-config --set /etc/glance/glance-api.conf glance_store default_store file 
# openstack-config --set /etc/glance/glance-api.conf glance_store filesystem_store_datadir /var/lib/glance/images/

```

### 7、修改glance配置文件/etc/glance/glance-registry.conf：
```ruby
# cp /etc/glance/glance-registry.conf /etc/glance/glance-registry.conf.bak
# >/etc/glance/glance-registry.conf
# openstack-config --set /etc/glance/glance-registry.conf DEFAULT transport_url rabbit://openstack:wangzy@controller
# openstack-config --set /etc/glance/glance-registry.conf database connection mysql+pymysql://glance:wangzy@controller/glance 
# openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken auth_uri http://controller:5000 
# openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken auth_url http://controller:35357 
# openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken memcached_servers controller:11211 
# openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken auth_type password 
# openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken project_domain_name default 
# openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken user_domain_name default 
# openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken project_name service 
# openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken username glance 
# openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken password wangzy
# openstack-config --set /etc/glance/glance-registry.conf paste_deploy flavor keystone
```

### 8、同步镜像服务数据库:
```ruby
[root@controller ~]# su -s /bin/sh -c "glance-manage db_sync" glance
Option "verbose" from group "DEFAULT" is deprecated for removal.  Its value may be silently ignored in the future.
/usr/lib/python2.7/site-packages/oslo_db/sqlalchemy/enginefacade.py:1241: OsloDBDeprecationWarning: EngineFacade is deprecated; please use oslo_db.sqlalchemy.enginefacade
  expire_on_commit=expire_on_commit, _conf=conf)
INFO  [alembic.runtime.migration] Context impl MySQLImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> liberty, liberty initial
INFO  [alembic.runtime.migration] Running upgrade liberty -> mitaka01, add index on created_at and updated_at columns of 'images' table
INFO  [alembic.runtime.migration] Running upgrade mitaka01 -> mitaka02, update metadef os_nova_server
INFO  [alembic.runtime.migration] Running upgrade mitaka02 -> ocata01, add visibility to and remove is_public from images
INFO  [alembic.runtime.migration] Context impl MySQLImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
Upgraded database to: ocata01, current revision(s): ocata01

// 忽略输出信息
```
### 9、启动glance及设置开机启动:
```ruby
[root@controller ~]# systemctl enable openstack-glance-api.service openstack-glance-registry.service 
[root@controller ~]# systemctl restart openstack-glance-api.service openstack-glance-registry.service
[root@controller ~]# systemctl status openstack-glance-api.service openstack-glance-registry.service
```
### 10、上传镜像到glance：
使用CirrOS验证Image服务的操作，这是一个小型Linux映像，可帮助您测试OpenStack部署。     
[OpenStack虚拟机映像指南](https://docs.openstack.org/image-guide/)     
[OpenStack最终用户指南](https://docs.openstack.org/queens/user/)

```ruby
[root@controller ~]# . admin-openrc 
[root@controller ~]# wget http://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img

```
#### 10.1 上传镜像:
&emsp;&emsp;使用QCOW2磁盘格式，裸容器格式和公开可见性将图像上传到Image服务，以便所有项目都可以访问它：
```ruby
[root@controller ~]# glance image-create \
--name "cirros-0.3.4-x86_64" \
--file cirros-0.3.4-x86_64-disk.img \
--disk-format qcow2 \
--container-format bare \
--visibility public \
--progress
[=============================>] 100%
+------------------+--------------------------------------+
| Property         | Value                                |
+------------------+--------------------------------------+
| checksum         | f8ab98ff5e73ebab884d80c9dc9c7290     |
| container_format | bare                                 |
| created_at       | 2018-06-28T03:11:33Z                 |
| disk_format      | qcow2                                |
| id               | 170a75ec-7fed-4084-bdd2-384c52bbaf60 |
| min_disk         | 0                                    |
| min_ram          | 0                                    |
| name             | cirros-0.3.4-x86_64                  |
| owner            | 4f57db84d18049ee9ba2f107196f7a8d     |
| protected        | False                                |
| size             | 13267968                             |
| status           | active                               |
| tags             | []                                   |
| updated_at       | 2018-06-28T03:11:33Z                 |
| virtual_size     | None                                 |
| visibility       | public                               |
+------------------+--------------------------------------+

```
#### 10.2 查看镜像列表:
```ruby
[root@controller ~]# glance image-list
+--------------------------------------+---------------------+
| ID                                   | Name                |
+--------------------------------------+---------------------+
| 170a75ec-7fed-4084-bdd2-384c52bbaf60 | cirros-0.3.5-x86_64 |
+--------------------------------------+---------------------+

// OpenStack动态生成ID，因此可能看到不同的值
```

[glance具体配置选项](https://docs.openstack.org/glance/queens/configuration/index.html)