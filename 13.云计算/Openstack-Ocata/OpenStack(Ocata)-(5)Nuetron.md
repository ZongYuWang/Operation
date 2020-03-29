## OpenStack(Ocata)-在controller节点安装Nuetron


### 1、创建nuetron数据库和授权:
```ruby
[root@controller ~]# mysql -u root -p
MariaDB [(none)]> CREATE DATABASE neutron;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'wangzy';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'wangzy';
```
### 2、创建neutron用户及赋予admin权限:
```ruby
[root@controller ~]# . admin-openrc 
[root@controller ~]#  openstack user create --domain default neutron --password wnagzy
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 8295d2feea684205aebb50e79071d22a |
| name                | neutron                          |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

[root@controller ~]# openstack role add --project service --user neutron admin
```

### 3、创建neutron服务:
```ruby
[root@controller ~]# openstack service create --name neutron --description "OpenStack Networking" network
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Networking             |
| enabled     | True                             |
| id          | e2025bffcdde46fe8410a36abfac107d |
| name        | neutron                          |
| type        | network                          |
+-------------+----------------------------------+
```
### 4、创建endpoint：
```ruby
[root@controller ~]# openstack endpoint create --region RegionOne network public http://controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | e0121c607e0144c8a307ffaa212f4fdb |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | e2025bffcdde46fe8410a36abfac107d |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne network internal http://controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 2626db9a6bce49e89319de15486ec754 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | e2025bffcdde46fe8410a36abfac107d |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+

[root@controller ~]# openstack endpoint create --region RegionOne network admin http://controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 48fadc4396a24f8db999ed74d9241110 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | e2025bffcdde46fe8410a36abfac107d |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+

```
### 5、安装neutron相关软件
```ruby
[root@controller ~]# yum install openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables -y

```

### 6、配置neutron配置文件/etc/neutron/neutron.conf：
```ruby
# cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.bak
# >/etc/neutron/neutron.conf
# openstack-config --set /etc/neutron/neutron.conf DEFAULT core_plugin ml2
# openstack-config --set /etc/neutron/neutron.conf DEFAULT service_plugins router
# openstack-config --set /etc/neutron/neutron.conf DEFAULT allow_overlapping_ips True
# openstack-config --set /etc/neutron/neutron.conf DEFAULT auth_strategy keystone
# openstack-config --set /etc/neutron/neutron.conf DEFAULT transport_url rabbit://openstack:wangzy@controller
# openstack-config --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_status_changes True
# openstack-config --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_data_changes True
# openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_uri http://controller:5000
# openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_url http://controller:35357 
# openstack-config --set /etc/neutron/neutron.conf keystone_authtoken memcached_servers controller:11211
# openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_type password
# openstack-config --set /etc/neutron/neutron.conf keystone_authtoken project_domain_name default
# openstack-config --set /etc/neutron/neutron.conf keystone_authtoken user_domain_name default
# openstack-config --set /etc/neutron/neutron.conf keystone_authtoken project_name service
# openstack-config --set /etc/neutron/neutron.conf keystone_authtoken username neutron
# openstack-config --set /etc/neutron/neutron.conf keystone_authtoken password devops
# openstack-config --set /etc/neutron/neutron.conf database connection mysql+pymysql://neutron:wangzy@controller/neutron
# openstack-config --set /etc/neutron/neutron.conf nova auth_url http://controller:35357
# openstack-config --set /etc/neutron/neutron.conf nova auth_type password
# openstack-config --set /etc/neutron/neutron.conf nova project_domain_name default
# openstack-config --set /etc/neutron/neutron.conf nova user_domain_name default
# openstack-config --set /etc/neutron/neutron.conf nova region_name RegionOne
# openstack-config --set /etc/neutron/neutron.conf nova project_name service
# openstack-config --set /etc/neutron/neutron.conf nova username nova
# openstack-config --set /etc/neutron/neutron.conf nova password wangzy
# openstack-config --set /etc/neutron/neutron.conf oslo_concurrency lock_path /var/lib/neutron/tmp
```

### 7、配置/etc/neutron/plugins/ml2/ml2_conf.ini:
```ruby
# openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 type_drivers flat,vlan,vxlan 
# openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 mechanism_drivers linuxbridge,l2population 
# openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 extension_drivers port_security 
# openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 tenant_network_types vxlan 
# openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 path_mtu 1500
# openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2_type_flat flat_networks provider
# openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2_type_vxlan vni_ranges 1:1000 
# openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini securitygroup enable_ipset True
```
### 8、配置/etc/neutron/plugins/ml2/linuxbridge_agent.ini:
```ruby
# openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini DEFAULT debug false
# openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini linux_bridge physical_interface_mappings provider:ens192
# openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan enable_vxlan True
# openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan local_ip 172.25.200.104
# openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan l2_population True 
# openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini agent prevent_arp_spoofing True
# openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup enable_security_group True 
# openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup firewall_driver neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```
`ens192是连接外网的网卡，一般这里写的网卡名都是能访问外网的，如果不是外网网卡，那么VM就会与外界网络隔离`
`local_ip 定义的是隧道网络，vxLan下vm-linuxbridge->vxlan ------tun-----vxlan->linuxbridge-vm`

### 9、配置 /etc/neutron/l3_agent.ini：
```ruby
# openstack-config --set /etc/neutron/l3_agent.ini DEFAULT interface_driver neutron.agent.linux.interface.BridgeInterfaceDriver 
# openstack-config --set /etc/neutron/l3_agent.ini DEFAULT external_network_bridge
# openstack-config --set /etc/neutron/l3_agent.ini DEFAULT debug false
```
### 10、配置/etc/neutron/dhcp_agent.ini:
```ruby
# openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT interface_driver neutron.agent.linux.interface.BridgeInterfaceDriver
# openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT dhcp_driver neutron.agent.linux.dhcp.Dnsmasq
# openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT enable_isolated_metadata True
# openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT verbose True
# openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT debug false
```
### 11、重新配置/etc/nova/nova.conf：
`配置这步的目的是让compute节点能使用上neutron网络`
```ruby
# openstack-config --set /etc/nova/nova.conf neutron url http://controller:9696 
# openstack-config --set /etc/nova/nova.conf neutron auth_url http://controller:35357 
# openstack-config --set /etc/nova/nova.conf neutron auth_plugin password 
# openstack-config --set /etc/nova/nova.conf neutron project_domain_id default 
# openstack-config --set /etc/nova/nova.conf neutron user_domain_id default 
# openstack-config --set /etc/nova/nova.conf neutron region_name RegionOne
# openstack-config --set /etc/nova/nova.conf neutron project_name service 
# openstack-config --set /etc/nova/nova.conf neutron username neutron 
# openstack-config --set /etc/nova/nova.conf neutron password wangzy
# openstack-config --set /etc/nova/nova.conf neutron service_metadata_proxy True 
# openstack-config --set /etc/nova/nova.conf neutron metadata_proxy_shared_secret wangzy
```
### 12、将dhcp-option-force=26,1450写入/etc/neutron/dnsmasq-neutron.conf：
```ruby
# echo "dhcp-option-force=26,1450" >/etc/neutron/dnsmasq-neutron.conf
```
### 13、配置/etc/neutron/metadata_agent.ini：
```ruby
# openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT nova_metadata_ip controller
# openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT metadata_proxy_shared_secret devops
# openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT metadata_workers 4
# openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT verbose True
# openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT debug false
# openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT nova_metadata_protocol http
```

### 14、创建软链接：
```ruby
[root@controller ~]# ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```
### 15、同步数据库：
```ruby
[root@controller ~]# su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```
### 16、重启nova服务：
```ruby
[root@controller ~]# systemctl restart openstack-nova-api.service
[root@controller ~]# systemctl status openstack-nova-api.service
```
### 17、重启neutron服务并设置开机启动：
```ruby
[root@controller ~]# systemctl enable neutron-server.service \
neutron-linuxbridge-agent.service \
neutron-dhcp-agent.service  \
neutron-metadata-agent.service

[root@controller ~]# systemctl restart neutron-server.service \
neutron-linuxbridge-agent.service  \
neutron-dhcp-agent.service  \
neutron-metadata-agent.service

[root@controller ~]# systemctl status neutron-server.service \
neutron-linuxbridge-agent.service \
neutron-dhcp-agent.service \
neutron-metadata-agent.service

```
### 18、启动neutron-l3-agent.service并设置开机启动:
```ruby
[root@controller ~]# systemctl enable neutron-l3-agent.service 
[root@controller ~]# systemctl restart neutron-l3-agent.service
[root@controller ~]# systemctl status neutron-l3-agent.service
```
### 19、执行验证：
```ruby
[root@controller ~]# source /root/admin-openrc
[root@controller ~]# neutron ext-list
[root@controller ~]# neutron agent-list
```
### 20、创建vxLan模式网络，让虚拟机能外出：
#### 20.1 创建flat模式的public网络:
`这个public是外出网络，必须是flat模式的`
```ruby
[root@controller ~]# source /root/admin-openrc
[root@controller ~]# neutron --debug net-create --shared provider --router:external True --provider:network_type flat --provider:physical_network provider
```
`执行完这步，在界面里进行操作，把public网络设置为共享和外部网络`

#### 20.2 创建public网络子网，名为public-sub:
```ruby
[root@controller ~]# neutron subnet-create provider 172.25.101.0/24 --name public-sub --allocation-pool start=172.25.101.110,end=172.25.101.150 --dns-nameserver 114.114.114.114 --gateway 172.25.101.1
neutron CLI is deprecated and will be removed in the future. Use openstack CLI instead.
Created a new subnet:
+-------------------+------------------------------------------------------+
| Field             | Value                                                |
+-------------------+------------------------------------------------------+
| allocation_pools  | {"start": "172.25.101.110", "end": "172.25.101.150"} |
| cidr              | 172.25.101.0/24                                      |
| created_at        | 2018-06-28T05:32:14Z                                 |
| description       |                                                      |
| dns_nameservers   | 114.114.114.114                                      |
| enable_dhcp       | True                                                 |
| gateway_ip        | 172.25.101.1                                         |
| host_routes       |                                                      |
| id                | 52787d44-afd4-4bcc-a8ba-0a419366c4e9                 |
| ip_version        | 4                                                    |
| ipv6_address_mode |                                                      |
| ipv6_ra_mode      |                                                      |
| name              | public-sub                                        |
| network_id        | 0f158974-6239-4d41-8d4e-5b088f13e594                 |
| project_id        | 4f57db84d18049ee9ba2f107196f7a8d                     |
| revision_number   | 2                                                    |
| service_types     |                                                      |
| subnetpool_id     |                                                      |
| tags              |                                                      |
| tenant_id         | 4f57db84d18049ee9ba2f107196f7a8d                     |
| updated_at        | 2018-06-28T05:32:14Z                                 |
+-------------------+------------------------------------------------------+
```

#### 20.3 创建名为private的私有网络, 网络模式为vxlan:
```ruby
[root@controller ~]# neutron net-create private --provider:network_type vxlan --router:external False --shared
neutron CLI is deprecated and will be removed in the future. Use openstack CLI instead.
Created a new network:
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | True                                 |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2018-06-28T05:32:42Z                 |
| description               |                                      |
| id                        | 039bb5ed-3ae8-4ea3-8bc4-bc854771230e |
| ipv4_address_scope        |                                      |
| ipv6_address_scope        |                                      |
| mtu                       | 1450                                 |
| name                      | private                              |
| port_security_enabled     | True                                 |
| project_id                | 4f57db84d18049ee9ba2f107196f7a8d     |
| provider:network_type     | vxlan                                |
| provider:physical_network |                                      |
| provider:segmentation_id  | 76                                   |
| revision_number           | 3                                    |
| router:external           | False                                |
| shared                    | True                                 |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tags                      |                                      |
| tenant_id                 | 4f57db84d18049ee9ba2f107196f7a8d     |
| updated_at                | 2018-06-28T05:32:42Z                 |
+---------------------------+--------------------------------------+
```
#### 20.4 创建名为private-subnet的私有网络子网:
`网段为192.168.1.0, 这个网段就是虚拟机获取的私有的IP地址`
```ruby
[root@controller ~]# neutron subnet-create private --name private-subnet --gateway 192.168.1.1 192.168.1.0/24
neutron CLI is deprecated and will be removed in the future. Use openstack CLI instead.
Created a new subnet:
+-------------------+--------------------------------------------------+
| Field             | Value                                            |
+-------------------+--------------------------------------------------+
| allocation_pools  | {"start": "192.168.1.2", "end": "192.168.1.254"} |
| cidr              | 192.168.1.0/24                                   |
| created_at        | 2018-06-28T05:33:12Z                             |
| description       |                                                  |
| dns_nameservers   |                                                  |
| enable_dhcp       | True                                             |
| gateway_ip        | 192.168.1.1                                      |
| host_routes       |                                                  |
| id                | 29906ef9-2efa-4c64-a8eb-df413c847b9b             |
| ip_version        | 4                                                |
| ipv6_address_mode |                                                  |
| ipv6_ra_mode      |                                                  |
| name              | private-subnet                                   |
| network_id        | 039bb5ed-3ae8-4ea3-8bc4-bc854771230e             |
| project_id        | 4f57db84d18049ee9ba2f107196f7a8d                 |
| revision_number   | 2                                                |
| service_types     |                                                  |
| subnetpool_id     |                                                  |
| tags              |                                                  |
| tenant_id         | 4f57db84d18049ee9ba2f107196f7a8d                 |
| updated_at        | 2018-06-28T05:33:12Z                             |
+-------------------+--------------------------------------------------+

```
##### 根据业务不同，创建不同的网络：
`假如你们公司的私有云环境是用于不同的业务，比如行政、销售、技术等，那么你可以创建3个不同名称的私有网络`
```ruby
# neutron net-create private-office --provider:network_type vxlan --router:external False --shared
# neutron subnet-create private-office --name office-net --gateway 192.168.2.1 192.168.2.0/24

# neutron net-create private-sale --provider:network_type vxlan --router:external False --shared
# neutron subnet-create private-sale --name sale-net --gateway 192.168.3.1 192.168.3.0/24

# neutron net-create private-technology --provider:network_type vxlan --router:external False --shared
# neutron subnet-create private-technology --name technology-net --gateway 192.168.4.1 192.168.4.0/24
```

#### 20.5 创建路由:
```ruby
点击项目——>网络——>路由——>新建路由
```
![](https://github.com/ZongYuWang/image/blob/master/OpenStack/OpenStack-Router1.png)
![](https://github.com/ZongYuWang/image/blob/master/OpenStack/OpenStack-Router1.png)
![](https://github.com/ZongYuWang/image/blob/master/OpenStack/OpenStack-interface1.png)

```ruby
[root@controller ~]# openstack router create router
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| admin_state_up          | UP                                   |
| availability_zone_hints |                                      |
| availability_zones      |                                      |
| created_at              | 2018-06-30T06:02:48Z                 |
| description             |                                      |
| distributed             | False                                |
| external_gateway_info   | None                                 |
| flavor_id               | None                                 |
| ha                      | False                                |
| id                      | 4194c269-47a9-4b31-92f4-92f8988455d7 |
| name                    | router                               |
| project_id              | 4f57db84d18049ee9ba2f107196f7a8d     |
| revision_number         | None                                 |
| routes                  |                                      |
| status                  | ACTIVE                               |
| updated_at              | 2018-06-30T06:02:48Z                 |
+-------------------------+--------------------------------------+

[root@controller ~]# openstack router set --external-gateway provider router
[root@controller ~]# openstack router list
+--------------------------------------+--------+--------+-------+-------------+-------+----------------------------------+
| ID                                   | Name   | Status | State | Distributed | HA    | Project                          |
+--------------------------------------+--------+--------+-------+-------------+-------+----------------------------------+
| 4194c269-47a9-4b31-92f4-92f8988455d7 | router | ACTIVE | UP    | False       | False | 4f57db84d18049ee9ba2f107196f7a8d |
+--------------------------------------+--------+--------+-------+-------------+-------+----------------------------------+
[root@controller ~]# openstack router show router
+-------------------------+----------------------------------------------------------------------------------------------------+
| Field                   | Value                                                                                              |
+-------------------------+----------------------------------------------------------------------------------------------------+
| admin_state_up          | UP                                                                                                 |
| availability_zone_hints |                                                                                                    |
| availability_zones      | nova                                                                                               |
| created_at              | 2018-06-30T06:02:48Z                                                                               |
| description             |                                                                                                    |
| distributed             | False                                                                                              |
| external_gateway_info   | {"network_id": "0f158974-6239-4d41-8d4e-5b088f13e594", "enable_snat": true, "external_fixed_ips":  |
|                         | [{"subnet_id": "52787d44-afd4-4bcc-a8ba-0a419366c4e9", "ip_address": "172.25.101.103"}]}           |
| flavor_id               | None                                                                                               |
| ha                      | False                                                                                              |
| id                      | 4194c269-47a9-4b31-92f4-92f8988455d7                                                               |
| name                    | router                                                                                             |
| project_id              | 4f57db84d18049ee9ba2f107196f7a8d                                                                   |
| revision_number         | 6                                                                                                  |
| routes                  |                                                                                                    |
| status                  | ACTIVE                                                                                             |
| updated_at              | 2018-06-30T06:07:23Z                                                                               |
+-------------------------+----------------------------------------------------------------------------------------------------+
```

#### 20.6 检查网络服务：
```ruby
[root@controller ~]# neutron agent-list
neutron CLI is deprecated and will be removed in the future. Use openstack CLI instead.
+---------------------+--------------------+------------+-------------------+-------+----------------+----------------------+
| id                  | agent_type         | host       | availability_zone | alive | admin_state_up | binary               |
+---------------------+--------------------+------------+-------------------+-------+----------------+----------------------+
| 27a1466e-8bfd-4371- | Metadata agent     | controller |                   | :-)   | True           | neutron-metadata-    |
| 9207-c9badafec655   |                    |            |                   |       |                | agent                |
| 67344bb1-e507-4bfb- | Linux bridge agent | compute    |                   | :-)   | True           | neutron-linuxbridge- |
| b63b-086943f36dd9   |                    |            |                   |       |                | agent                |
| ad1f1a2b-4d0e-4b58- | DHCP agent         | controller | nova              | :-)   | True           | neutron-dhcp-agent   |
| ae91-fd7d8156c730   |                    |            |                   |       |                |                      |
| d947e88c-45dd-4fca- | L3 agent           | controller | nova              | :-)   | True           | neutron-l3-agent     |
| 9f90-7e799b24fe44   |                    |            |                   |       |                |                      |
| fac1e677-222f-43ba- | Linux bridge agent | controller |                   | :-)   | True           | neutron-linuxbridge- |
| a558-5fcaa582b8f0   |                    |            |                   |       |                | agent                |
+---------------------+--------------------+------------+-------------------+-------+----------------+----------------------+
```


## OpenStack(Ocata)-在Compute节点安装Nuetron

### 1、安装相关软件包：
```ruby
[root@compute ~]# yum install openstack-neutron-linuxbridge ebtables ipset -y

```
### 2、配置neutron.conf:

```ruby
# cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.bak
# >/etc/neutron/neutron.conf
# openstack-config --set /etc/neutron/neutron.conf DEFAULT auth_strategy keystone
# openstack-config --set /etc/neutron/neutron.conf DEFAULT advertise_mtu True
# openstack-config --set /etc/neutron/neutron.conf DEFAULT dhcp_agents_per_network 2
# openstack-config --set /etc/neutron/neutron.conf DEFAULT control_exchange neutron
# openstack-config --set /etc/neutron/neutron.conf DEFAULT nova_url http://controller:8774/v2
# openstack-config --set /etc/neutron/neutron.conf DEFAULT transport_url rabbit://openstack:wangzy@controller
# openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_uri http://controller:5000
# openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_url http://controller:35357
# openstack-config --set /etc/neutron/neutron.conf keystone_authtoken memcached_servers controller:11211
# openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_type password
# openstack-config --set /etc/neutron/neutron.conf keystone_authtoken project_domain_name default
# openstack-config --set /etc/neutron/neutron.conf keystone_authtoken user_domain_name default
# openstack-config --set /etc/neutron/neutron.conf keystone_authtoken project_name service
# openstack-config --set /etc/neutron/neutron.conf keystone_authtoken username neutron
# openstack-config --set /etc/neutron/neutron.conf keystone_authtoken password wangzy
# openstack-config --set /etc/neutron/neutron.conf oslo_concurrency lock_path /var/lib/neutron/tmp
```

### 3、配置/etc/neutron/plugins/ml2/linuxbridge_agent.ini：
```ruby
# openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini linux_bridge physical_interface_mappings provider:ens256
# openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan enable_vxlan True
# openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan local_ip 172.25.200.105
# openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan l2_population True
# openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup enable_security_group True
# openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup firewall_driver neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```

`provider后面那个网卡名是隧道网络IP地址名称，我这里就是10.2.2.x网段网卡的名称`


### 4、配置nova.conf：
```ruby
# openstack-config --set /etc/nova/nova.conf neutron url http://controller:9696
# openstack-config --set /etc/nova/nova.conf neutron auth_url http://controller:35357
# openstack-config --set /etc/nova/nova.conf neutron auth_type password
# openstack-config --set /etc/nova/nova.conf neutron project_domain_name default
# openstack-config --set /etc/nova/nova.conf neutron user_domain_name default
# openstack-config --set /etc/nova/nova.conf neutron region_name RegionOne
# openstack-config --set /etc/nova/nova.conf neutron project_name service
# openstack-config --set /etc/nova/nova.conf neutron username neutron
# openstack-config --set /etc/nova/nova.conf neutron password wangzy
```

### 5、重启和enable相关服务：
```ruby
[root@compute ~]# systemctl restart libvirtd.service openstack-nova-compute.service 
[root@compute ~]# systemctl enable neutron-linuxbridge-agent.service 
[root@compute ~]# systemctl restart neutron-linuxbridge-agent.service
[root@compute ~]# systemctl status libvirtd.service openstack-nova-compute.service neutron-linuxbridge-agent.service
```


### 6、计算节点结合Cinder：
#### 6.1 计算节点要是想用cinder,那么需要配置nova配置文件：
```ruby
[root@compute ~]# openstack-config --set /etc/nova/nova.conf cinder os_region_name RegionOne
[root@compute ~]# systemctl restart openstack-nova-compute.service
```
#### 6.2 在controller上重启nova服务:
```ruby
[root@compute ~]# systemctl restart openstack-nova-api.service
[root@compute ~]# systemctl status openstack-nova-api.service
```

### 7、在controler上执行验证：
```ruby
[root@controller ~]# source /root/admin-openrc
[root@controller ~]# neutron agent-list
[root@controller ~]# nova-manage cell_v2 discover_hosts

// 运行nova host-list可以查看新加入的compute节点
// 如果需要再添加另外一个compute节点，只要重复下第二大部即可
```
