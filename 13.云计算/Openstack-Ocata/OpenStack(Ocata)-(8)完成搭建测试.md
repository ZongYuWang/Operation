## OpenStack(Ocata)-最终测试

### 1、创建配额的命令：
```ruby
[root@controller ~]# openstack flavor create m1.tiny --id 1 --ram 512 --disk 1 --vcpus 1
[root@controller ~]# openstack flavor create m1.small --id 2 --ram 2048 --disk 20 --vcpus 1
[root@controller ~]# openstack flavor create m1.medium --id 3 --ram 4096 --disk 40 --vcpus 2
[root@controller ~]# openstack flavor create m1.large --id 4 --ram 8192 --disk 80 --vcpus 4
[root@controller ~]# openstack flavor create m1.xlarge --id 5 --ram 16384 --disk 160 --vcpus 8
[root@controller ~]# openstack flavor list
```

### 1、命令行上传镜像
#### 1.1 把原生iso镜像上传到controller节点：

图片
#### 1.2 转换原生ISO镜像格式为qcow2：
```ruby
[root@controller ~]# openstack image create --disk-format qcow2 --container-format bare --public --file /root/CentOS-7-x86_64-Minimal-1708.iso  CentOS-7-x86_64
+------------------+------------------------------------------------------+
| Field            | Value                                                |
+------------------+------------------------------------------------------+
| checksum         | 5848f2fd31c7acf3811ad88eaca6f4aa                     |
| container_format | bare                                                 |
| created_at       | 2018-06-05T08:02:02Z                                 |
| disk_format      | qcow2                                                |
| file             | /v2/images/16eda428-7889-4af2-a780-3c9cd844c156/file |
| id               | 16eda428-7889-4af2-a780-3c9cd844c156                 |
| min_disk         | 0                                                    |
| min_ram          | 0                                                    |
| name             | CentOS-7-x86_64                                      |
| owner            | 249fc0e345b5410b80b2a4fbb97be709                     |
| protected        | False                                                |
| schema           | /v2/schemas/image                                    |
| size             | 830472192                                            |
| status           | active                                               |
| tags             |                                                      |
| updated_at       | 2018-06-05T08:02:05Z                                 |
| virtual_size     | None                                                 |
| visibility       | public                                               |
+------------------+------------------------------------------------------+

```

```ruby

[root@controller ~]#  openstack image create "CentOS7-50G" --file /root/CentOS-7-x86_64-GenericCloud-1611.raw --disk-format raw --container-format bare --public
+------------------+-------------------------------------------------------------------------------------------------------------+
| Field            | Value                                                                                                       |
+------------------+-------------------------------------------------------------------------------------------------------------+
| checksum         | 2b83b84e06b09332318427a834caccf4                                                                            |
| container_format | bare                                                                                                        |
| created_at       | 2018-06-25T07:10:07Z                                                                                        |
| disk_format      | raw                                                                                                         |
| file             | /v2/images/ecacb29a-a2ac-44c4-b903-c74e285d6cc3/file                                                        |
| id               | ecacb29a-a2ac-44c4-b903-c74e285d6cc3                                                                        |
| min_disk         | 0                                                                                                           |
| min_ram          | 0                                                                                                           |
| name             | CentOS7-50G                                                                                                 |
| owner            | 064e23cfc70e4cb4929b9344d32d3fed                                                                            |
| properties       | direct_url='rbd://23d0ad61-de3b-40a5-ac55-69c40e0d1369/images/ecacb29a-a2ac-44c4-b903-c74e285d6cc3/snap',   |
|                  | locations='[{u'url': u'rbd://23d0ad61-de3b-40a5-ac55-69c40e0d1369/images/ecacb29a-a2ac-                     |
|                  | 44c4-b903-c74e285d6cc3/snap', u'metadata': {}}]'                                                            |
| protected        | False                                                                                                       |
| schema           | /v2/schemas/image                                                                                           |
| size             | 8589934592                                                                                                  |
| status           | active                                                                                                      |
| tags             |                                                                                                             |
| updated_at       | 2018-06-25T07:16:35Z                                                                                        |
| virtual_size     | None                                                                                                        |
| visibility       | public                                                                                                      |
+------------------+-------------------------------------------------------------------------------------------------------------+


```










#### 1.3 查看制作的镜像信息：
```ruby
[root@controller ~]# openstack image list
+--------------------------------------+-----------------+--------+
| ID                                   | Name            | Status |
+--------------------------------------+-----------------+--------+
| 16eda428-7889-4af2-a780-3c9cd844c156 | CentOS-7-x86_64 | active |
| 0200a22c-ff8a-4bbc-9c8e-1c290130a36c | cirros          | active |
+--------------------------------------+-----------------+--------+

```

### 2、创建虚拟机流程
#### 2.1 创建网络：
```ruby
[root@controller ~]# . admin-openrc 
[root@controller ~]# openstack network create  --share --internal  --provider-physical-network provider  --provider-network-type flat netnewtv
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2018-06-22T07:14:40Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | c3c1b228-1163-4ea8-b453-2b7be25784a3 |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | None                                 |
| mtu                       | 1500                                 |
| name                      | netnewtv                             |
| port_security_enabled     | True                                 |
| project_id                | 064e23cfc70e4cb4929b9344d32d3fed     |
| provider:network_type     | flat                                 |
| provider:physical_network | provider                             |
| provider:segmentation_id  | None                                 |
| qos_policy_id             | None                                 |
| revision_number           | 3                                    |
| router:external           | Internal                             |
| segments                  | None                                 |
| shared                    | True                                 |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| updated_at                | 2018-06-22T07:14:40Z                 |
+---------------------------+--------------------------------------+

// --share 允许所有项目使用虚拟网络
// --external 定义外接虚拟网络 如果需要创建外网使用 --internal
// --provider-physical-network netnewtv && --provider-network-type flat 连接flat 虚拟网络
```
#### 2.2 创建子网：
```ruby
openstack subnet create \
  --network netnewtv  \
  --allocation-pool start=10.0.0.10,end=10.0.0.100 \
  --dns-nameserver 114.114.114.114 \
  --gateway 10.0.0.1  \
  --subnet-range 10.0.0.0/24 \
  subnewtv
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| allocation_pools  | 10.0.0.10-10.0.0.100                 |
| cidr              | 10.0.0.0/24                          |
| created_at        | 2018-06-22T07:15:22Z                 |
| description       |                                      |
| dns_nameservers   | 114.114.114.114                      |
| enable_dhcp       | True                                 |
| gateway_ip        | 10.0.0.1                             |
| host_routes       |                                      |
| id                | bbfa4552-bc2f-421c-b69d-db685c41f1e5 |
| ip_version        | 4                                    |
| ipv6_address_mode | None                                 |
| ipv6_ra_mode      | None                                 |
| name              | subnewtv                             |
| network_id        | c3c1b228-1163-4ea8-b453-2b7be25784a3 |
| project_id        | 064e23cfc70e4cb4929b9344d32d3fed     |
| revision_number   | 2                                    |
| segment_id        | None                                 |
| service_types     |                                      |
| subnetpool_id     | None                                 |
| updated_at        | 2018-06-22T07:15:22Z                 |
+-------------------+--------------------------------------+

```
```ruby
// 列出网络

[root@controller ~]# openstack network list
+--------------------------------------+----------+--------------------------------------+
| ID                                   | Name     | Subnets                              |
+--------------------------------------+----------+--------------------------------------+
| 32ef8664-b8df-4beb-a97a-e5a2f5dbb725 | nettv    | 96cb116a-ae88-4f08-8d9e-b47bc338471d |
| c3c1b228-1163-4ea8-b453-2b7be25784a3 | netnewtv | bbfa4552-bc2f-421c-b69d-db685c41f1e5 |
+--------------------------------------+----------+--------------------------------------+

```

#### 2.3 创建flavor：
```ruby
[root@controller ~]# openstack flavor create --id 2 --vcpus 4 --ram 4112 --disk 40 NewTV01
+----------------------------+---------+
| Field                      | Value   |
+----------------------------+---------+
| OS-FLV-DISABLED:disabled   | False   |
| OS-FLV-EXT-DATA:ephemeral  | 0       |
| disk                       | 40      |
| id                         | 2       |
| name                       | NewTV01 |
| os-flavor-access:is_public | True    |
| properties                 |         |
| ram                        | 4112    |
| rxtx_factor                | 1.0     |
| swap                       |         |
| vcpus                      | 4       |
+----------------------------+---------+


// --ram 4112 单位是M
// --disk 40  单位是G
```
```ruby
// 列出flavor

[root@controller ~]# openstack flavor list
+----+---------+------+------+-----------+-------+-----------+
| ID | Name    |  RAM | Disk | Ephemeral | VCPUs | Is Public |
+----+---------+------+------+-----------+-------+-----------+
| 1  | m1.nano |  128 |    1 |         0 |     4 | True      |
| 2  | NewTV01 | 4112 |   40 |         0 |     4 | True      |
+----+---------+------+------+-----------+-------+-----------+

```

#### 2.4 控制节点生成秘钥对
`在启动实例之前，需要将公钥添加到Compute服务`
```ruby
[root@controller ~]# . admin-openrc 
[root@controller ~]# ssh-keygen -q -N ""
Enter file in which to save the key (/root/.ssh/id_rsa): 
[root@controller ~]# openstack keypair create --public-key ~/.ssh/id_rsa.pub newtvkey
+-------------+-------------------------------------------------+
| Field       | Value                                           |
+-------------+-------------------------------------------------+
| fingerprint | bd:14:bf:3a:82:5a:f5:1c:2b:b3:55:27:13:a8:74:2e |
| name        | newtvkey                                        |
| user_id     | 5924e1ef912e40d09bd062a3f8486915                |
+-------------+-------------------------------------------------+
[root@controller ~]# openstack keypair list
+----------+-------------------------------------------------+
| Name     | Fingerprint                                     |
+----------+-------------------------------------------------+
| newtvkey | bd:14:bf:3a:82:5a:f5:1c:2b:b3:55:27:13:a8:74:2e |
+----------+-------------------------------------------------+

```
#### 2.5 添加安全组
`允许ICMP（ping）和安全shell（SSH）`
```ruby
[root@controller ~]# openstack security group rule create --proto icmp default
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| created_at        | 2018-06-22T07:20:11Z                 |
| description       |                                      |
| direction         | ingress                              |
| ether_type        | IPv4                                 |
| id                | 9a5437c1-773c-4d38-8bf1-68c119039d74 |
| name              | None                                 |
| port_range_max    | None                                 |
| port_range_min    | None                                 |
| project_id        | 064e23cfc70e4cb4929b9344d32d3fed     |
| protocol          | icmp                                 |
| remote_group_id   | None                                 |
| remote_ip_prefix  | 0.0.0.0/0                            |
| revision_number   | 1                                    |
| security_group_id | 4b7c0f34-5038-45d6-8821-2ee4390f86b0 |
| updated_at        | 2018-06-22T07:20:11Z                 |
+-------------------+--------------------------------------+

```
```ruby
[root@controller ~]# openstack security group rule create --proto tcp --dst-port 22 default
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| created_at        | 2018-06-22T07:20:39Z                 |
| description       |                                      |
| direction         | ingress                              |
| ether_type        | IPv4                                 |
| id                | 3b03c217-d342-43cd-a555-50017fec1d3f |
| name              | None                                 |
| port_range_max    | 22                                   |
| port_range_min    | 22                                   |
| project_id        | 064e23cfc70e4cb4929b9344d32d3fed     |
| protocol          | tcp                                  |
| remote_group_id   | None                                 |
| remote_ip_prefix  | 0.0.0.0/0                            |
| revision_number   | 1                                    |
| security_group_id | 4b7c0f34-5038-45d6-8821-2ee4390f86b0 |
| updated_at        | 2018-06-22T07:20:39Z                 |
+-------------------+--------------------------------------+

```
```ruby
// 列出安全组

[root@controller ~]# openstack security group list
+--------------------------------------+---------+-------------+----------------------------------+
| ID                                   | Name    | Description | Project                          |
+--------------------------------------+---------+-------------+----------------------------------+
| 4b7c0f34-5038-45d6-8821-2ee4390f86b0 | default | 缺省安全组  | 064e23cfc70e4cb4929b9344d32d3fed |
+--------------------------------------+---------+-------------+----------------------------------+

```
#### 2.6 创建虚拟机：
```ruby
[root@controller ~]# openstack server create \
--flavor NewTV01 \
--image CentOS-7-x86_64 \
--nic net-id=c3c1b228-1163-4ea8-b453-2b7be25784a3 \
--security-group default \
--key-name newtvkey \
newtvvm01

net-id就是创建网络时候的ID
+-------------------------------------+------------------------------------------------+
| Field                               | Value                                          |
+-------------------------------------+------------------------------------------------+
| OS-DCF:diskConfig                   | MANUAL                                         |
| OS-EXT-AZ:availability_zone         |                                                |
| OS-EXT-SRV-ATTR:host                | None                                           |
| OS-EXT-SRV-ATTR:hypervisor_hostname | None                                           |
| OS-EXT-SRV-ATTR:instance_name       |                                                |
| OS-EXT-STS:power_state              | NOSTATE                                        |
| OS-EXT-STS:task_state               | scheduling                                     |
| OS-EXT-STS:vm_state                 | building                                       |
| OS-SRV-USG:launched_at              | None                                           |
| OS-SRV-USG:terminated_at            | None                                           |
| accessIPv4                          |                                                |
| accessIPv6                          |                                                |
| addresses                           |                                                |
| adminPass                           | SFDDQH7AVPyf                                   |
| config_drive                        |                                                |
| created                             | 2018-06-22T07:25:21Z                           |
| flavor                              | NewTV01 (2)                                    |
| hostId                              |                                                |
| id                                  | ad5a36ef-8b74-435a-8af0-afc832174be5           |
| image                               | CentOS7 (781e48fc-71bb-48ff-8ea2-699cceed96f7) |
| key_name                            | newtvkey                                       |
| name                                | newtvvm01                                      |
| progress                            | 0                                              |
| project_id                          | 064e23cfc70e4cb4929b9344d32d3fed               |
| properties                          |                                                |
| security_groups                     | name='default'                                 |
| status                              | BUILD                                          |
| updated                             | 2018-06-22T07:25:21Z                           |
| user_id                             | 5924e1ef912e40d09bd062a3f8486915               |
| volumes_attached                    |                                                |
+-------------------------------------+------------------------------------------------+

```


#### 2.7 查看实列状态:
```ruby
[root@controller ~]# openstack server list
+--------------------------------------+-------------------+--------+----------+-----------------+
| ID                                   | Name              | Status | Networks | Image Name      |
+--------------------------------------+-------------------+--------+----------+-----------------+
| 61cc7e4d-f99a-44b8-a7db-477ee31c9671 | demo-instance     | ERROR  |          | cirros          |
| 25b31376-0139-49a5-9454-c8311f94312e | newtvvm01         | ERROR  |          | CentOS-7-x86_64 |
| 496a11f0-ac05-4124-aa98-0e252ac74979 | provider-instance | ERROR  |          | CentOS-7-x86_64 |
+--------------------------------------+-------------------+--------+----------+-----------------+


```

FAQ：
```ruby
2018-06-22 15:57:44.442 21814 ERROR nova.compute.manager [req-e1a565f7-6fb7-4aea-9ec2-f365b9317882 - - - - -] 更新节点 compute资源时发生错误。
2018-06-22 15:57:44.442 21814 ERROR nova.compute.manager Traceback (most recent call last):
2018-06-22 15:57:44.442 21814 ERROR nova.compute.manager   File "/usr/lib/python2.7/site-packages/nova/compute/manager.py", line 6606, in update_available_resource_for_node
2018-06-22 15:57:44.442 21814 ERROR nova.compute.manager     rt.update_available_resource(context, nodename)
2018-06-22 15:57:44.442 21814 ERROR nova.compute.manager   File "/usr/lib/python2.7/site-packages/nova/compute/resource_tracker.py", line 535, in update_available_resource
2018-06-22 15:57:44.442 21814 ERROR nova.compute.manager     resources = self.driver.get_available_resource(nodename)
2018-06-22 15:57:44.442 21814 ERROR nova.compute.manager   File "/usr/lib/python2.7/site-packages/nova/virt/libvirt/driver.py", line 5748, in get_available_resource
2018-06-22 15:57:44.442 21814 ERROR nova.compute.manager     disk_info_dict = self._get_local_gb_info()
2018-06-22 15:57:44.442 21814 ERROR nova.compute.manager   File "/usr/lib/python2.7/site-packages/nova/virt/libvirt/driver.py", line 5359, in _get_local_gb_info
2018-06-22 15:57:44.442 21814 ERROR nova.compute.manager     info = LibvirtDriver._get_rbd_driver().get_pool_info()
2018-06-22 15:57:44.442 21814 ERROR nova.compute.manager   File "/usr/lib/python2.7/site-packages/nova/virt/libvirt/storage/rbd_utils.py", line 372, in get_pool_info
2018-06-22 15:57:44.442 21814 ERROR nova.compute.manager     with RADOSClient(self) as client:
2018-06-22 15:57:44.442 21814 ERROR nova.compute.manager   File "/usr/lib/python2.7/site-packages/nova/virt/libvirt/storage/rbd_utils.py", line 105, in __init__
2018-06-22 15:57:44.442 21814 ERROR nova.compute.manager     self.cluster, self.ioctx = driver._connect_to_rados(pool)
2018-06-22 15:57:44.442 21814 ERROR nova.compute.manager   File "/usr/lib/python2.7/site-packages/nova/virt/libvirt/storage/rbd_utils.py", line 136, in _connect_to_rados
2018-06-22 15:57:44.442 21814 ERROR nova.compute.manager     client.connect()
2018-06-22 15:57:44.442 21814 ERROR nova.compute.manager   File "/usr/lib/python2.7/site-packages/rados.py", line 429, in connect
2018-06-22 15:57:44.442 21814 ERROR nova.compute.manager     raise make_ex(ret, "error connecting to the cluster")
2018-06-22 15:57:44.442 21814 ERROR nova.compute.manager ObjectNotFound: error connecting to the cluster

```

```ruby
错误： 实例 "vm" 执行所请求操作失败，实例处于错误状态。: 请稍后再试 [错误: 找不到有效主机，原因是 没有足够的主机可用。。].

[root@controller ~]# tail -f /var/log/nova/nova-conductor.log 
2018-06-25 17:46:48.044 35977 ERROR nova.conductor.manager [instance: 038ba0e1-0d82-4ec9-9fc0-f17ea84283d0]   File "/usr/lib/python2.7/site-packages/nova/network/neutronv2/api.py", line 1193, in deallocate_for_instance
2018-06-25 17:46:48.044 35977 ERROR nova.conductor.manager [instance: 038ba0e1-0d82-4ec9-9fc0-f17ea84283d0]     self._unbind_ports(context, ports_to_skip, neutron)
2018-06-25 17:46:48.044 35977 ERROR nova.conductor.manager [instance: 038ba0e1-0d82-4ec9-9fc0-f17ea84283d0]   File "/usr/lib/python2.7/site-packages/nova/network/neutronv2/api.py", line 502, in _unbind_ports
2018-06-25 17:46:48.044 35977 ERROR nova.conductor.manager [instance: 038ba0e1-0d82-4ec9-9fc0-f17ea84283d0]     get_client(context, admin=True))
2018-06-25 17:46:48.044 35977 ERROR nova.conductor.manager [instance: 038ba0e1-0d82-4ec9-9fc0-f17ea84283d0]   File "/usr/lib/python2.7/site-packages/nova/network/neutronv2/api.py", line 151, in get_client
2018-06-25 17:46:48.044 35977 ERROR nova.conductor.manager [instance: 038ba0e1-0d82-4ec9-9fc0-f17ea84283d0]     _ADMIN_AUTH = _load_auth_plugin(CONF)
2018-06-25 17:46:48.044 35977 ERROR nova.conductor.manager [instance: 038ba0e1-0d82-4ec9-9fc0-f17ea84283d0]   File "/usr/lib/python2.7/site-packages/nova/network/neutronv2/api.py", line 74, in _load_auth_plugin
2018-06-25 17:46:48.044 35977 ERROR nova.conductor.manager [instance: 038ba0e1-0d82-4ec9-9fc0-f17ea84283d0]     raise neutron_client_exc.Unauthorized(message=err_msg)
2018-06-25 17:46:48.044 35977 ERROR nova.conductor.manager [instance: 038ba0e1-0d82-4ec9-9fc0-f17ea84283d0] Unauthorized: Unknown auth type: None
2018-06-25 17:46:48.044 35977 ERROR nova.conductor.manager [instance: 038ba0e1-0d82-4ec9-9fc0-f17ea84283d0]


[root@controller neutron]# vim /usr/lib/python2.7/site-packages/nova/network/neutronv2/api.py
def _load_auth_plugin(conf):
    auth_plugin = ks_loading.load_auth_from_conf_options(conf,
                                    nova.conf.neutron.NEUTRON_GROUP)

    if auth_plugin:
        return auth_plugin

    err_msg = _('Unknown auth type: %s') % conf.neutron.auth_type
    raise neutron_client_exc.Unauthorized(message=err_msg)

```
```ruby
2018-06-27 13:34:11.302 2554 ERROR nova.scheduler.utils [req-1046c6c4-9503-483c-a72b-5d2844caa70d 5924e1ef912e40d09bd062a3f8486915 064e23cfc70e4cb4929b9344d32d3fed - - -] [instance: 7c37d70e-74dc-466d-8eb2-2bf8175655b1] Error from last host: compute (node compute): [u'Traceback (most recent call last):\n', u'  File "/usr/lib/python2.7/site-packages/nova/compute/manager.py", line 1787, in _do_build_and_run_instance\n    filter_properties)\n', u'  File "/usr/lib/python2.7/site-packages/nova/compute/manager.py", line 2025, in _build_and_run_instance\n    instance_uuid=instance.uuid, reason=six.text_type(e))\n', u"RescheduledException: Build of instance 7c37d70e-74dc-466d-8eb2-2bf8175655b1 was re-scheduled: internal error: qemu unexpectedly closed the monitor: 2018-06-27T05:29:18.001612Z qemu-kvm: -drive file=rbd:volumes/volume-c00af220-53d2-4585-b6b8-d2eec224e9c3:id=cinder:auth_supported=cephx\\;none:mon_host=172.25.5.139\\:6789\\;172.25.5.140\\:6789\\;172.25.5.141\\:6789,file.password-secret=virtio-disk0-secret0,format=raw,if=none,id=drive-virtio-disk0,serial=c00af220-53d2-4585-b6b8-d2eec224e9c3,cache=none: 'serial' is deprecated, please use the corresponding option of '-device' instead\n2018-06-27T05:34:18.053233Z qemu-kvm: -drive file=rbd:volumes/volume-c00af220-53d2-4585-b6b8-d2eec224e9c3:id=cinder:auth_supported=cephx\\;none:mon_host=172.25.5.139\\:6789\\;172.25.5.140\\:6789\\;172.25.5.141\\:6789,file.password-secret=virtio-disk0-secret0,format=raw,if=none,id=drive-virtio-disk0,serial=c00af220-53d2-4585-b6b8-d2eec224e9c3,cache=none: error connecting: Connection timed out\n"]
2018-06-27 13:34:11.901 2554 WARNING nova.scheduler.utils [req-1046c6c4-9503-483c-a72b-5d2844caa70d 5924e1ef912e40d09bd062a3f8486915 064e23cfc70e4cb4929b9344d32d3fed - - -] Failed to compute_task_build_instances: No valid host was found. There are not enough hosts available.
Traceback (most recent call last):

  File "/usr/lib/python2.7/site-packages/oslo_messaging/rpc/server.py", line 218, in inner
    return func(*args, **kwargs)

  File "/usr/lib/python2.7/site-packages/nova/scheduler/manager.py", line 98, in select_destinations
    dests = self.driver.select_destinations(ctxt, spec_obj)

  File "/usr/lib/python2.7/site-packages/nova/scheduler/filter_scheduler.py", line 79, in select_destinations
    raise exception.NoValidHost(reason=reason)

NoValidHost: No valid host was found. There are not enough hosts available.

2018-06-27 13:34:11.903 2554 WARNING nova.scheduler.utils [req-1046c6c4-9503-483c-a72b-5d2844caa70d 5924e1ef912e40d09bd062a3f8486915 064e23cfc70e4cb4929b9344d32d3fed - - -] [instance: 7c37d70e-74dc-466d-8eb2-2bf8175655b1] Setting instance to ERROR state.
```