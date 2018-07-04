## OpenStack(Ocata)-在controller节点安装Cinder

### 1、创建数据库并授权：
```ruby
MariaDB [(none)]> CREATE DATABASE cinder;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' IDENTIFIED BY 'wangzy';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' IDENTIFIED BY 'wangzy';
```

#### 1.2 创建cinder用户并赋予admin权限：
```ruby
[root@controller ~]# . admin-openrc 
[root@controller ~]# openstack user create --domain default cinder --password wangzy
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 92b4e7ece8a74ad287b39a7cd46c0978 |
| name                | cinder                           |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

[root@controller ~]# openstack role add --project service --user cinder admin

```

### 3、创建volume服务:
```ruby
[root@controller ~]# openstack service create --name cinder --description "OpenStack Block Storage" volume
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Block Storage          |
| enabled     | True                             |
| id          | fb59ad62ddd74b50b944b307568ed895 |
| name        | cinder                           |
| type        | volume                           |
+-------------+----------------------------------+

[root@controller ~]# openstack service create --name cinderv2 --description "OpenStack Block Storage" volumev2
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Block Storage          |
| enabled     | True                             |
| id          | 96f3ccccb80e4b66a749e225032656dc |
| name        | cinderv2                         |
| type        | volumev2                         |
+-------------+----------------------------------+
```
### 4、创建endpoint:
```ruby
[root@controller ~]# openstack endpoint create --region RegionOne volume public http://controller:8776/v1/%\(tenant_id\)s
+--------------+-----------------------------------------+
| Field        | Value                                   |
+--------------+-----------------------------------------+
| enabled      | True                                    |
| id           | 4b841bbb69344bb3999b8e5f6ad0e8e0        |
| interface    | public                                  |
| region       | RegionOne                               |
| region_id    | RegionOne                               |
| service_id   | fb59ad62ddd74b50b944b307568ed895        |
| service_name | cinder                                  |
| service_type | volume                                  |
| url          | http://controller:8776/v1/%(tenant_id)s |
+--------------+-----------------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne volume internal http://controller:8776/v1/%\(tenant_id\)s
+--------------+-----------------------------------------+
| Field        | Value                                   |
+--------------+-----------------------------------------+
| enabled      | True                                    |
| id           | 65ec43a0aec74471809a877cb2cf53dc        |
| interface    | internal                                |
| region       | RegionOne                               |
| region_id    | RegionOne                               |
| service_id   | fb59ad62ddd74b50b944b307568ed895        |
| service_name | cinder                                  |
| service_type | volume                                  |
| url          | http://controller:8776/v1/%(tenant_id)s |
+--------------+-----------------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne volume admin http://controller:8776/v1/%\(tenant_id\)s
+--------------+-----------------------------------------+
| Field        | Value                                   |
+--------------+-----------------------------------------+
| enabled      | True                                    |
| id           | 965fdd6d75c04885a8fe829d7891ba25        |
| interface    | admin                                   |
| region       | RegionOne                               |
| region_id    | RegionOne                               |
| service_id   | fb59ad62ddd74b50b944b307568ed895        |
| service_name | cinder                                  |
| service_type | volume                                  |
| url          | http://controller:8776/v1/%(tenant_id)s |
+--------------+-----------------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne volumev2 public http://controller:8776/v2/%\(tenant_id\)s
+--------------+-----------------------------------------+
| Field        | Value                                   |
+--------------+-----------------------------------------+
| enabled      | True                                    |
| id           | d9dfc2101d254c78b5de8cd55c6b5238        |
| interface    | public                                  |
| region       | RegionOne                               |
| region_id    | RegionOne                               |
| service_id   | 96f3ccccb80e4b66a749e225032656dc        |
| service_name | cinderv2                                |
| service_type | volumev2                                |
| url          | http://controller:8776/v2/%(tenant_id)s |
+--------------+-----------------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne volumev2 internal http://controller:8776/v2/%\(tenant_id\)s
+--------------+-----------------------------------------+
| Field        | Value                                   |
+--------------+-----------------------------------------+
| enabled      | True                                    |
| id           | 8622974987ae43e69891408fb0119fa9        |
| interface    | internal                                |
| region       | RegionOne                               |
| region_id    | RegionOne                               |
| service_id   | 96f3ccccb80e4b66a749e225032656dc        |
| service_name | cinderv2                                |
| service_type | volumev2                                |
| url          | http://controller:8776/v2/%(tenant_id)s |
+--------------+-----------------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne volumev2 admin http://controller:8776/v2/%\(tenant_id\)s
+--------------+-----------------------------------------+
| Field        | Value                                   |
+--------------+-----------------------------------------+
| enabled      | True                                    |
| id           | e218d07919fe411ea587ae1b4804fcf1        |
| interface    | admin                                   |
| region       | RegionOne                               |
| region_id    | RegionOne                               |
| service_id   | 96f3ccccb80e4b66a749e225032656dc        |
| service_name | cinderv2                                |
| service_type | volumev2                                |
| url          | http://controller:8776/v2/%(tenant_id)s |
+--------------+-----------------------------------------+

```

### 5、安装cinder相关服务：
```ruby
[root@controller ~]# yum install -y openstack-cinder

```
### 6、配置cinder配置文件：
```ruby
# cp /etc/cinder/cinder.conf /etc/cinder/cinder.conf.bak
# >/etc/cinder/cinder.conf
# openstack-config --set /etc/cinder/cinder.conf DEFAULT my_ip 172.25.253.104
# openstack-config --set /etc/cinder/cinder.conf DEFAULT auth_strategy keystone
# openstack-config --set /etc/cinder/cinder.conf DEFAULT transport_url rabbit://openstack:wangzy@controller
# openstack-config --set /etc/cinder/cinder.conf database connection mysql+pymysql://cinder:wangzy@controller/cinder
# openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_uri http://controller:5000
# openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_url http://controller:35357
# openstack-config --set /etc/cinder/cinder.conf keystone_authtoken memcached_servers controller:11211
# openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_type password
# openstack-config --set /etc/cinder/cinder.conf keystone_authtoken project_domain_name default
# openstack-config --set /etc/cinder/cinder.conf keystone_authtoken user_domain_name default
# openstack-config --set /etc/cinder/cinder.conf keystone_authtoken project_name service
# openstack-config --set /etc/cinder/cinder.conf keystone_authtoken username cinder
# openstack-config --set /etc/cinder/cinder.conf keystone_authtoken password wangzy
# openstack-config --set /etc/cinder/cinder.conf oslo_concurrency lock_path /var/lib/cinder/tmp
```
### 7、同步数据库:
````ruby
[root@controller ~]# su -s /bin/sh -c "cinder-manage db sync" cinder
Option "logdir" from group "DEFAULT" is deprecated. Use option "log-dir" from group "DEFAULT".

```
### 8、在controller上启动cinder服务，并设置开机启动
```ruby
[root@controller ~]# systemctl enable openstack-cinder-api.service openstack-cinder-scheduler.service 
[root@controller ~]# systemctl restart openstack-cinder-api.service openstack-cinder-scheduler.service 
[root@controller ~]# systemctl status openstack-cinder-api.service openstack-cinder-scheduler.service
```

## OpenStack(Ocata)-在Cinder节点安装Cinder-LVM
`也可以安装在controller节点上`

### 1、安装软件包：
```ruby
[root@cinder ~]# yum install lvm2 -y
```
### 2、启动服务并设置为开机自启：
```ruby
[root@cinder ~]# systemctl enable lvm2-lvmetad.service
[root@cinder ~]# systemctl start lvm2-lvmetad.service
[root@cinder ~]# systemctl status lvm2-lvmetad.service
```
### 3、创建lvm:
`/dev/sdb就是额外添加的硬盘`
```ruby
[root@cinder ~]# fdisk -l
[root@cinder ~]# pvcreate /dev/sdb
[root@cinder ~]# vgcreate cinder-volumes /dev/sdb
```
### 4、编辑存储节点lvm.conf文件:
```ruby
[root@cinder ~]# vim /etc/lvm/lvm.conf

在devices 下面添加 filter = [ "a/sda/", "a/sdb/", "r/.*/"]  // 130行
```
### 5、lvm2服务：
```ruby
[root@cinder ~]# systemctl restart lvm2-lvmetad.service
[root@cinder ~]# systemctl status lvm2-lvmetad.service
```

### 6、安装openstack-cinder、targetcli:
```ruby
[root@cinder ~]# yum install openstack-cinder openstack-utils targetcli python-keystone ntpdate -y

```
### 7、配置cinder配置文件：

```ruby
# cp /etc/cinder/cinder.conf /etc/cinder/cinder.conf.bak
# >/etc/cinder/cinder.conf 
# openstack-config --set /etc/cinder/cinder.conf DEFAULT debug False
# openstack-config --set /etc/cinder/cinder.conf DEFAULT verbose True
# openstack-config --set /etc/cinder/cinder.conf DEFAULT auth_strategy keystone
# openstack-config --set /etc/cinder/cinder.conf DEFAULT my_ip 172.25.253.106
# openstack-config --set /etc/cinder/cinder.conf DEFAULT enabled_backends lvm
# openstack-config --set /etc/cinder/cinder.conf DEFAULT glance_api_servers http://controller:9292
# openstack-config --set /etc/cinder/cinder.conf DEFAULT glance_api_version 2
# openstack-config --set /etc/cinder/cinder.conf DEFAULT enable_v1_api True
# openstack-config --set /etc/cinder/cinder.conf DEFAULT enable_v2_api True
# openstack-config --set /etc/cinder/cinder.conf DEFAULT enable_v3_api True
# openstack-config --set /etc/cinder/cinder.conf DEFAULT storage_availability_zone nova
# openstack-config --set /etc/cinder/cinder.conf DEFAULT default_availability_zone nova
# openstack-config --set /etc/cinder/cinder.conf DEFAULT os_region_name RegionOne
# openstack-config --set /etc/cinder/cinder.conf DEFAULT api_paste_config /etc/cinder/api-paste.ini
# openstack-config --set /etc/cinder/cinder.conf DEFAULT transport_url rabbit://openstack:wangzy@controller
# openstack-config --set /etc/cinder/cinder.conf database connection mysql+pymysql://cinder:wangzy@controller/cinder
# openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_uri http://controller:5000
# openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_url http://controller:35357
# openstack-config --set /etc/cinder/cinder.conf keystone_authtoken memcached_servers controller:11211
# openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_type password
# openstack-config --set /etc/cinder/cinder.conf keystone_authtoken project_domain_name default
# openstack-config --set /etc/cinder/cinder.conf keystone_authtoken user_domain_name default
# openstack-config --set /etc/cinder/cinder.conf keystone_authtoken project_name service
# openstack-config --set /etc/cinder/cinder.conf keystone_authtoken username cinder
# openstack-config --set /etc/cinder/cinder.conf keystone_authtoken password wangzy
# openstack-config --set /etc/cinder/cinder.conf lvm volume_driver cinder.volume.drivers.lvm.LVMVolumeDriver
# openstack-config --set /etc/cinder/cinder.conf lvm volume_group cinder-volumes
# openstack-config --set /etc/cinder/cinder.conf lvm iscsi_protocol iscsi
# openstack-config --set /etc/cinder/cinder.conf lvm iscsi_helper lioadm
# openstack-config --set /etc/cinder/cinder.conf oslo_concurrency lock_path /var/lib/cinder/tmp

```

### 8、启动openstack-cinder-volume和target并设置开机启动:
```ruby
[root@cinder ~]# systemctl enable openstack-cinder-volume.service target.service 
[root@cinder ~]# systemctl restart openstack-cinder-volume.service target.service 
[root@cinder ~]# systemctl status openstack-cinder-volume.service target.service
```

### 9、验证cinder服务是否正常：
```ruby
[root@controller ~]# source /root/admin-openrc
[root@controller ~]# cinder service-list
+------------------+------------+------+---------+-------+----------------------------+-----------------+
| Binary           | Host       | Zone | Status  | State | Updated_at                 | Disabled Reason |
+------------------+------------+------+---------+-------+----------------------------+-----------------+
| cinder-scheduler | controller | nova | enabled | up    | 2018-07-02T07:24:39.000000 | -               |
| cinder-volume    | cinder@lvm | nova | enabled | up    | 2018-07-02T07:24:33.000000 | -               |
+------------------+------------+------+---------+-------+----------------------------+-----------------+

```

## OpenStack(Ocata)-在存储节点安装Cinder-Ceph
`Ceph存储和OpenStack不在同一个环境中`

### 1、必备条件配置
#### 1.1 验证安装的CEPH：
```ruby
[root@cephm-0 ~]# ceph -s
  cluster:
    id:     23d0ad61-de3b-40a5-ac55-69c40e0d1369
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum cephm-0,cephm-1,cephm-2
    mgr: openstack1(active)
    osd: 12 osds: 12 up, 12 in
 
  data:
    pools:   1 pools, 512 pgs
    objects: 47860 objects, 136 GB
    usage:   304 GB used, 10854 GB / 11159 GB avail
    pgs:     512 active+clean

[root@cephm-0 ~]# ceph health
HEALTH_OK

[root@cephm-0 ~]# ceph -v
ceph version 12.2.5 (cad919881333ac92274171586c827e01f554a70a) luminous (stable)

[root@cephm-0 ~]# ceph osd tree
ID CLASS WEIGHT   TYPE NAME            STATUS REWEIGHT PRI-AFF 
-1       12.00000 root default                                 
-3       12.00000     rack rack01                              
-2        4.00000         host cephm-0                         
 0   hdd  1.00000             osd.0        up  1.00000 1.00000 
 1   hdd  1.00000             osd.1        up  1.00000 1.00000 
 2   hdd  1.00000             osd.2        up  1.00000 1.00000 
 3   hdd  1.00000             osd.3        up  1.00000 1.00000 
-7        4.00000         host cephm-1                         
 4   hdd  1.00000             osd.4        up  1.00000 1.00000 
 5   hdd  1.00000             osd.5        up  1.00000 1.00000 
 6   hdd  1.00000             osd.6        up  1.00000 1.00000 
 7   hdd  1.00000             osd.7        up  1.00000 1.00000 
-9        4.00000         host cephm-2                         
 8   hdd  1.00000             osd.8        up  1.00000 1.00000 
 9   hdd  1.00000             osd.9        up  1.00000 1.00000 
10   hdd  1.00000             osd.10       up  1.00000 1.00000 
11   hdd  1.00000             osd.11       up  1.00000 1.00000

[root@cephm-0 ~]# ceph df 
GLOBAL:
    SIZE       AVAIL      RAW USED     %RAW USED 
    11159G     10854G         304G          2.73 
POOLS:
    NAME        ID     USED     %USED     MAX AVAIL     OBJECTS 
    newtv       1      136G      3.88         3379G       47860 

```
#### 1.2 安装软件包：
##### 在openstack节点中安装ceph客户端软件包：
```ruby
# yum search ceph
# yum install centos-release-ceph-luminous.noarch -y
# yum install python-ceph

```

### 2、Ceph存储卷：
#### 2.1 创建存储卷：
`在 ceph 中创建三个 pool 分别给 Cinder，Glance 和 nova 使用`
```ruby
[root@cephm-0 ~]# ceph osd pool create volumes 64
pool 'volumes' created
[root@cephm-0 ~]# ceph osd pool create images 64
pool 'images' created
[root@cephm-0 ~]# ceph osd pool create vms 64
pool 'vms' created

```
#### 2.2 删除存储卷(当上面创建的存储卷有问题的时候才执行此步)：
```ruby
[root@cephm-0 ~]# ceph osd pool delete volumes  volumes --yes-i-really-really-mean-it
Error EPERM: pool deletion is disabled; you must first set the mon_allow_pool_delete config option to true before you can destroy a pool
// 必须指定两次pool name

[root@cephm-0 ~]# ceph -n mon.0 --show-config | grep mon_allow_pool_delete
mon_allow_pool_delete = false

[root@cephm-0 ~]# ceph -n mon.1 --show-config | grep mon_allow_pool_delete
mon_allow_pool_delete = false

[root@cephm-0 ~]# ceph -n mon.2 --show-config | grep mon_allow_pool_delete
mon_allow_pool_delete = false

[root@cephm-0 ~]# cat /etc/ceph/ceph.conf
[global]
fsid = 23d0ad61-de3b-40a5-ac55-69c40e0d1369
mon_initial_members = cephm-0, cephm-1, cephm-2
mon_host = 172.25.5.19,172.25.5.20,172.25.5.21
auth_cluster_required = none 
auth_service_required = none 
auth_client_required = none 

osd pool default size = 3
public network = 172.25.5.0/24
private network = 172.25.5.0/24

[mgr]
mgr modules = dashboard

[mon]
mon initial members = cephm-0,cephm-1,cephm-2
mon data = /data/ceph/data/$name
log file = /data/ceph/logs/$name.log
mon allow pool delete = true
```
### 3、配置Ceph文件：
`将ceph的配置文件传到ceph client节点（glance-api, cinder-volume, nova-compute andcinder-backup）上`
```ruby
[root@compute ~]# mkdir /etc/ceph
[root@controller ~]# mkdir /etc/ceph

[root@cephm-0 ~]# ssh controller sudo tee /etc/ceph/ceph.conf < /etc/ceph/ceph.conf
[root@cephm-0 ~]# ssh compute sudo tee /etc/ceph/ceph.conf < /etc/ceph/ceph.conf
```
### 4、配置cinder和glance用户访问ceph的权限：
`nova-compute主机不需要密钥对`
```ruby
[root@cephm-0 ~]# ceph auth get-or-create client.cinder mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=volumes, allow rwx pool=vms, allow rx pool=images'
[client.cinder]
	key = AQDaECtb0jZABxAAjjxIDhFN2x9/O6/j2g++RA==
// cinder 用户会被 cinder 和 nova 使用，需要访问三个pool

[root@cephm-0 ~]# ceph auth get-or-create client.glance mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=images'
[client.glance]
	key = AQDtECtbY+T+LxAAfc+QuV7W4eJl+50wczAw6Q==
// glance 用户只会被 Glance 使用，只需要访问 images 这个 pool


[root@cephm-0 ~]# ceph auth get-or-create client.cinder-backup mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=backups'
[client.cinder-backup]
	key = AQD5ECtbFP4tLBAAwzoCFmCqwt8PWCnWBSD3qw==
```

### 5、配置glance-api(controller)
```ruby
[root@cephm-0 ~]# ceph auth get-or-create client.glance | ssh controller tee /etc/ceph/ceph.client.glance.keyring
root@controller's password: 
[client.glance]
	key = AQDtECtbY+T+LxAAfc+QuV7W4eJl+50wczAw6Q==

[root@cephm-0 ~]# ssh controller chown glance:glance /etc/ceph/ceph.client.glance.keyring

```

### 6、配置cinder-volume(cinder)

```ruby
[root@cephm-0 ~]# ceph auth get-or-create client.cinder | ssh controller tee /etc/ceph/ceph.client.cinder.keyring
root@controller's password: 
[client.cinder]
	key = AQDaECtb0jZABxAAjjxIDhFN2x9/O6/j2g++RA==

[root@cephm-0 ~]# ssh controller chown cinder:cinder /etc/ceph/ceph.client.cinder.keyring

```

```ruby
[root@cephm-0 ~]# ceph auth get-or-create client.cinder-backup
[client.cinder-backup]
	key = AQD5ECtbFP4tLBAAwzoCFmCqwt8PWCnWBSD3qw==

```

### 7、配置nova-compute(compute)
```ruby
[root@cephm-0 ~]# ceph auth get-or-create client.cinder | ssh compute tee /etc/ceph/ceph.client.cinder.keyring
root@compute's password: 
[client.cinder]
	key = AQDaECtb0jZABxAAjjxIDhFN2x9/O6/j2g++RA==

[root@cephm-0 ~]# ceph auth get-key client.cinder | ssh compute tee client.cinder.key
root@compute's password: 
AQDaECtb0jZABxAAjjxIDhFN2x9/O6/j2g++RA==
```

```ruby
[root@compute ~]# uuidgen
aeeacf5a-5a98-40f0-8e87-79f9f44fdc36

[root@compute ~]# cat > secret.xml <<EOF
<secret ephemeral='no' private='no'>
   <usage type='ceph'>
     <name>client.cinder secret</name>
   </usage>
</secret>
EOF
 
[root@compute ~]# virsh secret-define --file secret.xml 
生成 secret 89f915a8-f062-46f9-a211-7cdee0ae04b0

[root@compute ~]# virsh secret-set-value \
--secret bb01323f-684a-41a0-9cc4-db4f61b610d5 \
--base64 AQDaECtb0jZABxAAjjxIDhFN2x9/O6/j2g++RA && rm client.cinder.key secret.xml



rm：是否删除普通文件 "client.cinder.key"？
rm：是否删除普通文件 "secret.xml"？


[root@compute ceph]# virsh secret-set-value --secret bb01323f-684a-41a0-9cc4-db4f61b610d5 --base64 AQDaECtb0jZABxAAjjxIDhFN2x9/O6/j2g++RA==
secret 值设定

[root@compute ceph]# virsh secret-list
 UUID                                  用量
--------------------------------------------------------------------------------
 bb01323f-684a-41a0-9cc4-db4f61b610d5  ceph client.cinder secret



```
`在有多个计算节点时，在compute节点上执行相同的操作`

FAQ:
```ruby
[root@compute ~]# vim secret.xml 
[root@compute ~]# virsh secret-define --file secret.xml
错误：使用 secret.xml 设定属性失败
错误：internal error: 已将 UUID 为89f915a8-f062-46f9-a211-7cdee0ae04b0 的 secret 定义为与 client.cinder secret 一同使用

[root@compute ~]# virsh secret-list
 UUID                                  用量
--------------------------------------------------------------------------------
 89f915a8-f062-46f9-a211-7cdee0ae04b0  ceph client.cinder secret

[root@compute ~]# virsh secret-undefine 89f915a8-f062-46f9-a211-7cdee0ae04b0
已删除 secret 89f915a8-f062-46f9-a211-7cdee0ae04b0

```



```ruby
[root@compute ~]# cp /etc/ceph/ceph.conf /etc/ceph/ceph.conf.back
[root@compute ~]# > /etc/ceph/ceph.conf
[root@compute ~]# vim /etc/ceph/ceph.conf
[client]
rbd cache = true
rbd cache writethrough until flush = true
admin socket = /var/run/ceph/guests/$cluster-$type.$id.$pid.$cctid.asok
log file = /var/log/qemu/qemu-guest-$pid.log
rbd concurrent management ops = 20

```

```ruby
[root@compute ~]# mkdir -p /var/run/ceph/guests/ /var/log/qemu/
[root@compute ~]# groupadd libvirtd
[root@compute ~]# chown qemu:libvirtd /var/run/ceph/guests /var/log/qemu/

```

```ruby
openstack-config --set /etc/nova/nova.conf libvirt images_type rbd
openstack-config --set /etc/nova/nova.conf libvirt images_rbd_pool vms
openstack-config --set /etc/nova/nova.conf libvirt images_rbd_ceph_conf /etc/ceph/ceph.conf
openstack-config --set /etc/nova/nova.conf libvirt rbd_user cinder
openstack-config --set /etc/nova/nova.conf libvirt rbd_secret_uuid 89f915a8-f062-46f9-a211-7cdee0ae04b0
openstack-config --set /etc/nova/nova.conf libvirt disk_cachemodes "network=writeback"
openstack-config --set /etc/nova/nova.conf libvirt inject_password false
openstack-config --set /etc/nova/nova.conf libvirt inject_key false
openstack-config --set /etc/nova/nova.conf libvirt inject_partition -2
openstack-config --set /etc/nova/nova.conf libvirt hw_disk_discard unmap
```


### 8、配置glance-api（controller)：
```ruby
openstack-config --set /etc/glance/glance-api.conf glance_store stores rbd
openstack-config --set /etc/glance/glance-api.conf glance_store default_store rbd
openstack-config --set /etc/glance/glance-api.conf glance_store rbd_store_pool images
openstack-config --set /etc/glance/glance-api.conf glance_store rbd_store_user glance
openstack-config --set /etc/glance/glance-api.conf glance_store rbd_store_ceph_conf /etc/ceph/ceph.conf
openstack-config --set /etc/glance/glance-api.conf glance_store rbd_store_chunk_size 8
openstack-config --set /etc/glance/glance-api.conf DEFAULT show_image_direct_url True
openstack-config --set /etc/glance/glance-api.conf DEFAULT show_multiple_locations True
openstack-config --set /etc/glance/glance-api.conf DEFAULT show_image_direct_url True
openstack-config --set /etc/glance/glance-api.conf paste_deploy flavor keystone
```

### 9、配置cinder-volume（cinder）:
```ruby
openstack-config --set /etc/cinder/cinder.conf DEFAULT enabled_backends ceph
openstack-config --set /etc/cinder/cinder.conf DEFAULT volume_driver cinder.volume.drivers.rbd.RBDDriver
openstack-config --set /etc/cinder/cinder.conf DEFAULT rbd_pool volumes
openstack-config --set /etc/cinder/cinder.conf DEFAULT rbd_ceph_conf /etc/ceph/ceph.conf
openstack-config --set /etc/cinder/cinder.conf DEFAULT rbd_flatten_volume_from_snapshot false
openstack-config --set /etc/cinder/cinder.conf DEFAULT rbd_max_clone_depth 5
openstack-config --set /etc/cinder/cinder.conf DEFAULT rbd_store_chunk_size 4
openstack-config --set /etc/cinder/cinder.conf DEFAULT rados_connect_timeout -1
openstack-config --set /etc/cinder/cinder.conf DEFAULT glance_api_version 2
openstack-config --set /etc/cinder/cinder.conf DEFAULT rbd_user cinder
openstack-config --set /etc/cinder/cinder.conf DEFAULT rbd_secret_uuid 89f915a8-f062-46f9-a211-7cdee0ae04b0
```
### 10、在各个相关节点重启以下服务：
```ruby
[root@controller ~]# systemctl restart openstack-glance-api
[root@compute ~]# systemctl restart openstack-nova-compute
[root@controller ~]# systemctl restart openstack-cinder-volume
```
### 11、验证操作
#### 11.1 上传镜像并验证：
```ruby
[root@controller ~]# openstack image list
+--------------------------------------+---------+--------+
| ID                                   | Name    | Status |
+--------------------------------------+---------+--------+
| 781e48fc-71bb-48ff-8ea2-699cceed96f7 | CentOS7 | active |
| 10e41719-99ba-4a3d-86b9-eef167a740e6 | cirros  | active |
+--------------------------------------+---------+--------+
[root@controller ~]# rbd ls images
781e48fc-71bb-48ff-8ea2-699cceed96f7

```

#### 11.2 创建虚拟机并验证:
```ruby

```
#### 11.3 创建云盘并验证:
```ruby
[root@controller cinder]# rbd ls volumes
volume-9ce5c22c-1ca8-4e8b-b0e2-2eeea4db7aac
```


[root@controller ceph]# cinder type-create ceph
+--------------------------------------+------+-------------+-----------+
| ID                                   | Name | Description | Is_Public |
+--------------------------------------+------+-------------+-----------+
| d7dc85d9-7806-497a-8e90-e5ce39afbef0 | ceph | -           | True      |
+--------------------------------------+------+-------------+-----------+
[root@controller ~]# cinder type-key ceph set volume_backend_name=ceph