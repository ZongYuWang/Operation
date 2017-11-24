## KVM
目录：
1.安装KVM虚拟机

### 1.KVM虚拟机桥接网卡配置：
```ruby
[root@localhost ~]# cd /etc/sysconfig/network-scripts/
[root@localhost network-scripts]# cp ifcfg-eth0 ifcfg-br0

[root@localhost network-scripts]# vim ifcfg-eth0 
DEVICE=eth0
TYPE=Ethernet
ONBOOT=yes
BRIDGE=br0

[root@localhost network-scripts]# vim ifcfg-br0 
DEVICE=br0
TYPE=Bridge
ONBOOT=yes
BOOTPROTO=static
IPADDR=172.30.105.107
PREFIX=24
GATEWAY=172.30.105.254

[root@localhost network-scripts]# service network restart

[root@localhost ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 50:7b:9d:ff:53:e1 brd ff:ff:ff:ff:ff:ff
    inet6 fe80::527b:9dff:feff:53e1/64 scope link 
       valid_lft forever preferred_lft forever
3: wlan0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN qlen 1000
    link/ether a0:d3:7a:16:b3:22 brd ff:ff:ff:ff:ff:ff
4: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN 
    link/ether 50:7b:9d:ff:53:e1 brd ff:ff:ff:ff:ff:ff
    inet 172.30.105.107/24 brd 172.30.105.255 scope global br0
    inet6 fe80::527b:9dff:feff:53e1/64 scope link 
       valid_lft forever preferred_lft forever
```
### 1.安装KVM虚拟机：
```ruby
[root@localhost ~]# yum install libvirt qemu qemu-kvm kvm python-virtinst yum upgrade device-mapper-libs -y

```
创建qcow2的img：
```ruby
[root@localhost ~]# mkdir /vms
[root@localhost ~]# qemu-img create -f qcow2 /vms/vm1.qcow2 50G
Formatting '/vms/vm1.qcow2', fmt=qcow2 size=53687091200 encryption=off cluster_size=65536 
[root@localhost ~]# chown qemu:qemu /vms/vm1.qcow2 
[root@localhost ~]# chmod 777 /vms/vm1.qcow2
```
启动kvm虚拟机：
```ruby
[root@localhost ~]# service libvirtd start
Starting libvirtd daemon:                                  [  OK  ]

```
kvm虚拟机安装：
```ruby
[root@localhost ~]# virt-install --name vm1 --ram=2048 --vcpus=2 --disk path=/vms/vm1.qcow2,size=50,bus=virtio,format=qcow2 --accelerate --cdrom /iso/CentOS-6.5-x86_64-minimal.iso --vnc --vncport=6121 --vnclisten=0.0.0.0 --network bridge=br0,model=virtio --noautoconsole
WARNING  KVM acceleration not available, using 'qemu'

Starting install...
Creating domain...                                                                                                                               |    0 B     00:00     
Domain installation still in progress. You can reconnect to 
the console to complete the installation process.

```
查看虚拟机状态并查看虚拟机配置文件：
```ruby
[root@localhost ~]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 1     vm1                            running


[root@localhost ~]# vim /etc/libvirt/qemu/vm1.xml
<domain type='qemu'>
  <name>vm1</name>
  <uuid>30c873f2-bd3d-4027-cd20-719d75409d7a</uuid>
  <memory unit='KiB'>2097152</memory>
  <currentMemory unit='KiB'>2097152</currentMemory>
  <vcpu placement='static'>2</vcpu>
  <os>
    <type arch='x86_64' machine='rhel6.6.0'>hvm</type>
    <boot dev='hd'/>
  </os>
  .......
  
```

kvm虚拟机的关闭和删除：
```ruby
# 关闭kvm虚拟机：
[root@localhost ~]# virsh destroy vm1
Domain vm1 destroyed

# 消除kvm虚拟机：
[root@localhost ~]# virsh undefine vm1
Domain vm1 has been undefined

```