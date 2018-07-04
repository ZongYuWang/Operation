## OpenStack(Ocata)-Controller节点安装keystone

### 1、创建keystone数据库并授权：
```ruby
[root@controller ~]# mysql -u root -p
MariaDB [(none)]> CREATE DATABASE keystone;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'wangzy';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'wangzy';

```
### 2、安装keystone和memcached：
```ruby
[root@controller ~]# yum -y install openstack-keystone httpd mod_wsgi python-openstackclient memcached python-memcached 

```
### 3、启动memcache服务并设置开机自启动：
```ruby
[root@controller ~]# systemctl enable memcached.service
[root@controller ~]# systemctl restart memcached.service
[root@controller ~]# systemctl status memcached.service
```

### 4、配置/etc/keystone/keystone.conf文件：
```ruby
# cp /etc/keystone/keystone.conf /etc/keystone/keystone.conf.bak
# >/etc/keystone/keystone.conf
# openstack-config --set /etc/keystone/keystone.conf DEFAULT transport_url rabbit://openstack:wangzy@controller
# openstack-config --set /etc/keystone/keystone.conf database connection mysql://keystone:wangzy@controller/keystone
# openstack-config --set /etc/keystone/keystone.conf cache backend oslo_cache.memcache_pool
# openstack-config --set /etc/keystone/keystone.conf cache enabled true
# openstack-config --set /etc/keystone/keystone.conf cache memcache_servers controller:11211
# openstack-config --set /etc/keystone/keystone.conf memcache servers controller:11211
# openstack-config --set /etc/keystone/keystone.conf token expiration 3600
# openstack-config --set /etc/keystone/keystone.conf token provider fernet
```
### 5、配置httpd.conf文件和memcached文件：
```ruby
[root@controller ~]# sed -i "s/#ServerName www.example.com:80/ServerName controller/" /etc/httpd/conf/httpd.conf
[root@controller ~]# sed -i 's/OPTIONS*.*/OPTIONS="-l 127.0.0.1,::1,172.25.253.104"/' /etc/sysconfig/memcached
```
### 6、配置keystone与httpd结合：
```ruby
[root@controller ~]# ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
```

### 7、同步keystone数据库：
```ruby
[root@controller ~]# su -s /bin/sh -c "keystone-manage db_sync" keystone

```
### 8、初始化fernet:
```ruby
[root@controller ~]# keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
[root@controller ~]# keystone-manage credential_setup --keystone-user keystone --keystone-group keystone

```
### 9、启动HTTP并设置开机自启动：
```ruby
[root@controller ~]# systemctl enable httpd.service 
[root@controller ~]# systemctl restart httpd.service
[root@controller ~]# systemctl status httpd.service
[root@controller ~]# systemctl list-unit-files |grep httpd.service
```

### 10、创建admin用户角色:
```ruby
[root@controller ~]# keystone-manage bootstrap \
--bootstrap-password wangzy \
--bootstrap-username admin \
--bootstrap-project-name admin \
--bootstrap-role-name admin \
--bootstrap-service-name keystone \
--bootstrap-region-id RegionOne \
--bootstrap-admin-url http://controller:35357/v3 \
--bootstrap-internal-url http://controller:35357/v3 \
--bootstrap-public-url http://controller:5000/v3
```

#### 10.1 验证：
```ruby
[root@controller ~]# [root@controller ~]# openstack project list \
--os-username admin \
--os-project-name admin \
--os-user-domain-id default \
--os-project-domain-id default \
--os-identity-api-version 3 \
--os-auth-url http://controller:5000 \
--os-password wangzy
+----------------------------------+-------+
| ID                               | Name  |
+----------------------------------+-------+
| 4f57db84d18049ee9ba2f107196f7a8d | admin |
+----------------------------------+-------+
```

### 11、创建admin用户环境变量：
创建/root/admin-openrc 文件并写入如下内容
```ruby
[root@controller ~]# vim /root/admin-openrc
添加以下内容：
export OS_USER_DOMAIN_ID=default
export OS_PROJECT_DOMAIN_ID=default
export OS_USERNAME=admin
export OS_PROJECT_NAME=admin
export OS_PASSWORD=wangzy
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
export OS_AUTH_URL=http://controller:35357/v3
```
### 12、创建service项目：
```ruby
[root@controller ~]# source /root/admin-openrc 
[root@controller ~]# openstack project create --domain default --description "Service Project" service
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Service Project                  |
| domain_id   | default                          |
| enabled     | True                             |
| id          | 5588ac29f6ee410ebdb8e18d7cd7f3b0 |
| is_domain   | False                            |
| name        | service                          |
| parent_id   | default                          |
+-------------+----------------------------------+
```
### 13、创建demo项目：
```ruby
[root@controller ~]# openstack project create --domain default --description "Demo Project" demo
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Demo Project                     |
| domain_id   | default                          |
| enabled     | True                             |
| id          | f613ac3d38d04ef8aa4f2876141b9e1d |
| is_domain   | False                            |
| name        | demo                             |
| parent_id   | default                          |
+-------------+----------------------------------+

```
### 14、创建demo用户：
```ruby
[root@controller ~]# openstack user create --domain default demo --password wangzy
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 2d63eb26744c4179b294e804cdaceade |
| name                | demo                             |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```
#### 14.1 创建user角色将demo用户赋予user角色：
```ruby
[root@controller ~]# openstack role create user
+-----------+----------------------------------+
| Field     | Value                            |
+-----------+----------------------------------+
| domain_id | None                             |
| id        | 8f9740d80bd14b5bb6ddc50cf273e413 |
| name      | user                             |
+-----------+----------------------------------+
[root@controller ~]# openstack role add --project demo --user demo user
```
### 15、验证keystone：
```ruby
[root@controller ~]# unset OS_TOKEN OS_URL
[root@controller ~]# [root@controller ~]# openstack \
--os-auth-url http://controller:35357/v3 \
--os-project-domain-name default \
--os-user-domain-name default \
--os-project-name admin \
--os-username admin token issue \
--os-password wangzy
+------------+------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                        |
+------------+------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2018-06-28T03:51:05+0000                                                                                                     |
| id         | gAAAAABbNE0ZqelDOGnDUtDxB3kPj9eJVwwYnaGo9CnTkjTHdSxyzN6Zv7ZM63diKyUamiT-                                                     |
|            | m08Krsoos9cXA_cg_RHLHCOm_ZStFPtcxuwpqRdilUn8HzSEK8LCcn_l4-kl_3aIludCYEfApxMiuvfsb1NgNvQBoORZJdQDfhydPhkhkx7uwaw              |
| project_id | 4f57db84d18049ee9ba2f107196f7a8d                                                                                             |
| user_id    | f56699018e26416ab0482d1faa7100ee                                                                                             |
+------------+------------------------------------------------------------------------------------------------------------------------------+

[root@controller ~]# [root@controller ~]# openstack \
--os-auth-url http://controller:5000/v3 \
--os-project-domain-name default \
--os-user-domain-name default \
--os-project-name demo \
--os-username demo token issue \
--os-password wangzy
+------------+------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                        |
+------------+------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2018-06-28T03:51:23+0000                                                                                                     |
| id         | gAAAAABbNE0rXVa0baggWg92QK1NjcwyKr-                                                                                          |
|            | mg89sVqcSpA7EzROFt_RSetGfiouI127tTFMgqxNXvM3m_PpWzkEVqi6GFnalBDwA9fdk5p0xmmPfJfPvrkTs2wZM38hqS-                              |
|            | 2MH1KBQaPffqBDXsZDXMnH6HbQV21y1qPheMphpSMjrPcszTXRvI0                                                                        |
| project_id | f613ac3d38d04ef8aa4f2876141b9e1d                                                                                             |
| user_id    | 2d63eb26744c4179b294e804cdaceade                                                                                             |
+------------+------------------------------------------------------------------------------------------------------------------------------+
```