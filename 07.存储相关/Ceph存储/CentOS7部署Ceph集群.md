## CentOS7部署Ceph集群

### 1、安装RHEL7操作系统
#### 1.1 查看系统版本号：
```ruby
# cat /etc/redhat-release 
CentOS Linux release 7.4.1708 (Core) 
```
#### 1.2 查看磁盘分区：
```ruby
# fdisk -l

Disk /dev/sdb: 21.5 GB, 21474836480 bytes
Disk /dev/sdc: 21.5 GB, 21474836480 bytes
```

### 2、配置操作系统
#### 2.1 关闭防火墙：
```ruby
# systemctl stop firewalld.service
# systemctl disable firewalld.service
```
#### 2.2 关闭SElinux:
##### 临时禁用：
```ruby
# setenforce 0
```
##### 永久禁用：
```ruby
# vim /etc/selinux/config

SELINUX=disabled
SELINUXTYPE=targeted
```
#### 2.3 配置机器名:
```ruby
# hostnamectl set-hostname cnode1
# hostnamectl set-hostname cnode2
# hostnamectl set-hostname cnode3
# hostnamectl set-hostname cephclient001
```

#### 2.4 配置操作系统的主机表:
```ruby
cat>>/etc/hosts<<EOF
172.25.101.112 cnode1
172.25.101.108 cnode2
172.25.101.109 cnode3
EOF
```
### 3、配置节点间无密码SSH
```ruby
# ssh-keygen -t rsa -P ''  // 在每个节点都要执行
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Created directory '/root/.ssh'.
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:vnGC2H/2GwnksjdhCzrbz0BgsNItbjIxOJUUID1QR6Y root@ceph-node001
The key's randomart image is:
+---[RSA 2048]----+
|+*=+=            |
|.o+= +           |
|o E.+ +   .      |
| . = o . o       |
|  o o   S =      |
|   + o = = + .   |
|    . = * = o    |
|       = Oo. .   |
|      . ++o.o.   |
+----[SHA256]-----+

[root@ceph-node001 ~]# cat .ssh/id_rsa.pub >> .ssh/authorized_keys_cnode1
[root@ceph-node002 ~]# cat .ssh/id_rsa.pub >> .ssh/authorized_keys_cnode2
[root@ceph-node003 ~]# cat .ssh/id_rsa.pub >> .ssh/authorized_keys_cnode3

// 将这四个authorized_keys***文件全部拷贝到任何一个服务器上
# cat authorized_keys_cnode1 >> authorized_keys
# cat authorized_keys_cnode2 >> authorized_keys
# cat authorized_keys_cnode3 >> authorized_keys

// 再将authorized_keys文件全部拷贝至所有节点，并删除各个节点的authorized_keys***
```

### 4、配置NTP
#### 4.1 安装NTP软件：
```ruby
# yum install ntp -y
```
#### 4.2 配置NTP服务端：
`以ceph-node001节点作为NTP服务端，其余节点为NTP客户端`
```ruby
# vim /etc/ntp.conf

# Hosts on local network are less restricted.
# 允许内网其他机器同步时间
restrict 192.168.101.0 mask 255.255.255.0 nomodify notrap
// 其他的restrict项全部注销

server 202.112.10.36             # 1.cn.pool.ntp.org
// 将所有的server项去掉

# allow update time by the upper server 
# 允许上层时间服务器主动修改本机时间
restrict 202.112.10.36 nomodify notrap noquery

# 外部时间服务器不可用时，以本地时间作为时间服务
server  127.127.1.0     # local clock
fudge   127.127.1.0 stratum 10

# systemctl restart ntpd.service
# systemctl enable ntpd.service

# ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
 gus.buptnet.edu 10.3.8.150       5 u   20   64    1   29.221    6.437   0.001
*LOCAL(0)        .LOCL.          10 l   19   64    1    0.000    0.000   0.001

```

#### 4.3 配置NTP客户端：
```ruby
# vim /etc/ntp.conf
server 192.168.101.112  //指向内网时间服务器节点ceph-node001的IP地址
# ntpdate -u cnode1
 8 Aug 02:06:56 ntpdate[22154]: adjust time server 172.25.101.112 offset 0.008667 sec

//时间同步后可以将时间写入硬件寄存器
# clock -w
# systemctl restart ntpd.service
# systemctl enable ntpd.service
```
```ruby
# crontab -e
*/5 * * * * ntpdate -u cnode1 >/dev/null 2>&1
```

### 5、Ceph安装配置

#### 5.1 安装ceph软件包(12.2.5-luminous版)
```ruby
# yum search ceph
......
centos-release-ceph-hammer.noarch : Ceph Hammer packages from the CentOS Storage SIG repository
centos-release-ceph-jewel.noarch : Ceph Jewel packages from the CentOS Storage SIG repository
centos-release-ceph-luminous.noarch : Ceph Luminous packages from the CentOS Storage SIG repository
ceph-common.x86_64 : Ceph Common

# yum install centos-release-ceph-luminous.noarch -y
# yum install ceph ceph-radosgw -y

# ceph -v
ceph version 12.2.5 (cad919881333ac92274171586c827e01f554a70a) luminous (stable)

```
#### 5.2 登录到初始监视器节点cnode1：
确保保存 Ceph 配置文件的目录存在， Ceph 默认使用 /etc/ceph 。安装 ceph 软件时，安装器也会自动创建 /etc/ceph/ 目录  

```ruby
[root@cnode1 ~]# ls /etc/ceph/
rbdmap

// 部署工具在清除集群时可能删除此目录
```

- 执行uuidgen命令，给集群分配惟一ID :

```ruby
[root@cnode1 ~]# uuidgen
a0b2ee0c-562c-43ca-86a1-d64fb84b8ce0
```

- 创建ceph配置文件，Ceph默认使用ceph.conf，其中的ceph是集群名字：

```ruby
[root@cnode1 ~]# vim /etc/ceph/ceph.conf 

[global]
fsid = a0b2ee0c-562c-43ca-86a1-d64fb84b8ce0  # fsid = {UUID}
mon initial members = cnode1    # mon initial members = {hostname}[,{hostname}]
mon host = 172.25.101.112
public network = 172.25.101.0/24
auth cluster required = cephx
auth service required = cephx
auth client required = cephx
osd journal size = 1024
osd pool default size = 3
osd pool default min size = 2
osd pool default pg num = 333
osd pool default pgp num = 333
osd crush chooseleaf type = 1
```

- 为此集群创建密钥环、并生成监视器密钥:

```ruby
[root@cnode1 ~]# ceph-authtool --create-keyring /tmp/ceph.mon.keyring --gen-key -n mon. --cap mon 'allow *'
creating /tmp/ceph.mon.keyring
```
- 生成管理员密钥环，生成 client.admin 用户并加入密钥环:

```ruby
[root@ceph-node001 ~]# sudo ceph-authtool \
--create-keyring /etc/ceph/ceph.client.admin.keyring \
--gen-key \
-n client.admin \
--set-uid=0 \
--cap mon 'allow *' \
--cap osd 'allow *' \
--cap mds 'allow *' \
--cap mgr 'allow *'
creating /etc/ceph/ceph.client.admin.keyring

```
- 生成bootstrap-osd密钥环，生成client.bootstrap-osd用户并加入密钥环:

```ruby
[root@cnode1 ~]# sudo ceph-authtool \
--create-keyring /var/lib/ceph/bootstrap-osd/ceph.keyring \
--gen-key \
-n client.bootstrap-osd \
--cap mon 'profile bootstrap-osd'
creating /var/lib/ceph/bootstrap-osd/ceph.keyring

```

- 把密钥加入 ceph.mon.keyring:

```ruby
[root@cnode1 ~]# sudo ceph-authtool /tmp/ceph.mon.keyring \
--import-keyring /etc/ceph/ceph.client.admin.keyring
importing contents of /etc/ceph/ceph.client.admin.keyring into /tmp/ceph.mon.keyring

```
```ruby
[root@cnode1 ~]# sudo ceph-authtool /tmp/ceph.mon.keyring \
--import-keyring /var/lib/ceph/bootstrap-osd/ceph.keyring
importing contents of /var/lib/ceph/bootstrap-osd/ceph.keyring into /tmp/ceph.mon.keyring

```

- 用规划好的主机名、对应 IP 地址、和 FSID 生成一个监视器图，并保存为 /tmp/monmap：

```ruby
[root@cnode1 ~]# monmaptool \
--create \
--add cnode1 172.25.101.112 \
--fsid a0b2ee0c-562c-43ca-86a1-d64fb84b8ce0 \
/tmp/monmap

monmaptool: monmap file /tmp/monmap
monmaptool: set fsid to a0b2ee0c-562c-43ca-86a1-d64fb84b8ce0
monmaptool: writing epoch 0 to /tmp/monmap (1 monitors)

// 语法：monmaptool --create --add {hostname} {ip-address} --fsid {uuid} /tmp/monmap
```

- 在监视器主机上分别创建数据目录：

```ruby
[root@cnode1 ~]# sudo -u ceph mkdir /var/lib/ceph/mon/ceph-cnode1

// 语法：sudo mkdir /var/lib/ceph/mon/{cluster-name}-{hostname}
```

- 监视器图和密钥环组装守护进程所需的初始数据：

```ruby
[root@cnode1 ~]# chown -R ceph.ceph /tmp/ceph.mon.keyring
[root@cnode1 ~]# sudo -u ceph ceph-mon --mkfs -i cnode1 --monmap /tmp/monmap --keyring /tmp/ceph.mon.keyring

// 语法：sudo -u ceph ceph-mon [--cluster {cluster-name}] --mkfs -i {hostname} --monmap /tmp/monmap --keyring /tmp/ceph.mon.keyring
```

- 建一个空文件 done ，表示监视器已创建、可以启动了：

```ruby
[root@cnode1 ~]# sudo touch /var/lib/ceph/mon/ceph-cnode1/done

```
- 启动监视器：

```ruby
[root@cnode1 ~]# systemctl start ceph-mon@cnode1
[root@cnode1 ~]# systemctl status ceph-mon@cnode1
● ceph-mon@cnode1.service - Ceph cluster monitor daemon
   Loaded: loaded (/usr/lib/systemd/system/ceph-mon@.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2018-08-08 04:25:58 EDT; 4s ago
 Main PID: 23399 (ceph-mon)
   CGroup: /system.slice/system-ceph\x2dmon.slice/ceph-mon@cnode1.service
           └─23399 /usr/bin/ceph-mon -f --cluster ceph --id cnode1 --setuser ceph --setgroup ceph

Aug 08 04:25:58 cnode1 systemd[1]: Started Ceph cluster monitor daemon.
Aug 08 04:25:58 cnode1 systemd[1]: Starting Ceph cluster monitor daemon...
[root@cnode1 ~]# systemctl enable ceph-mon@cnode1

```
```ruby
[root@cnode1 ~]# ss -tnlp | grep 6789
LISTEN     0      128    172.25.101.112:6789                     *:*                   users:(("ceph-mon",pid=23399,fd=21))

```

- 确认下集群在运行：

```ruby
[root@cnode1 ~]# ceph -s
  cluster:
    id:     a0b2ee0c-562c-43ca-86a1-d64fb84b8ce0
    health: HEALTH_OK
 
  services:
    mon: 1 daemons, quorum cnode1
    mgr: no daemons active
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 bytes
    usage:   0 kB used, 0 kB / 0 kB avail
    pgs:   
```

### 6、新增mon节点(cnode2)

- 把cnode1上生成的配置文件和密钥文件拷贝到cnode2:

```ruby
[root@cnode1 ~]# scp /etc/ceph/* root@cnode2:/etc/ceph
ceph.client.admin.keyring                                                                                                    
ceph.conf                                                                                                                    
rbdmap  

[root@cnode1 ~]# scp /var/lib/ceph/bootstrap-osd/ceph.keyring root@cnode2:/var/lib/ceph/bootstrap-osd/
ceph.keyring 

[root@cnode1 ~]# scp /tmp/ceph.mon.keyring root@cnode2:/tmp/ceph.mon.keyring
ceph.mon.keyring 
```
- 在cnode2上创建一个默认的数据目录:

```ruby
[root@cnode2 ~]# sudo -u ceph mkdir /var/lib/ceph/mon/ceph-cnode2

```
- 在cnode2上修改ceph.mon.keyring属主和属组为ceph:

```ruby
[root@cnode2 ~]# chown ceph.ceph /tmp/ceph.mon.keyring

```
- 获取密钥和monmap信息：

```ruby
[root@cnode2 ~]# ceph auth get mon. -o /tmp/ceph.mon.keyring
exported keyring for mon.
[root@cnode2 ~]# ceph mon getmap -o /tmp/ceph.mon.map
got monmap epoch 1

```
- 初始化mon：

```ruby
[root@cnode2 ~]# sudo -u ceph ceph-mon \
--mkfs \
-i cnode2 \
--monmap /tmp/ceph.mon.map \
--keyring /tmp/ceph.mon.keyring

```
- 为了防止重新被安装创建一个空的done文件:

```ruby
[root@cnode2 ~]# sudo touch /var/lib/ceph/mon/ceph-cnode2/done

```
- 将新的mon节点添加至ceph集群的mon列表:

```ruby
[root@cnode2 ~]# ceph mon add cnode2 172.25.101.108:6789
adding mon.cnode2 at 172.25.101.108:6789/0

```
- 启动新添加的mon:

```ruby
[root@cnode2 ~]# systemctl start ceph-mon@cnode2
[root@cnode2 ~]# systemctl status ceph-mon@cnode2
● ceph-mon@cnode2.service - Ceph cluster monitor daemon
   Loaded: loaded (/usr/lib/systemd/system/ceph-mon@.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2018-08-08 05:38:34 EDT; 7s ago
 Main PID: 23927 (ceph-mon)
   CGroup: /system.slice/system-ceph\x2dmon.slice/ceph-mon@cnode2.service
           └─23927 /usr/bin/ceph-mon -f --cluster ceph --id cnode2 --setuser ceph --setgroup ceph

Aug 08 05:38:34 cnode2 systemd[1]: Started Ceph cluster monitor daemon.
Aug 08 05:38:34 cnode2 systemd[1]: Starting Ceph cluster monitor daemon...
[root@cnode2 ~]# systemctl enable ceph-mon@cnode2

```
- 查看目前的集群状态

```ruby
[root@cnode1 ~]# ceph -s
  cluster:
    id:     a0b2ee0c-562c-43ca-86a1-d64fb84b8ce0
    health: HEALTH_WARN
            clock skew detected on mon.cnode1
 
  services:
    mon: 2 daemons, quorum cnode2,cnode1
    mgr: no daemons active
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 bytes
    usage:   0 kB used, 0 kB / 0 kB avail
    pgs:     

```

### 7、新增mon节点(cnode3)

- 把cnode1上生成的配置文件和密钥文件拷贝到cnode3:

```ruby
[root@cnode1 ~]# scp /etc/ceph/* root@cnode3:/etc/ceph/ 
ceph.client.admin.keyring                                                                                                     
ceph.conf                                                                                                                  
rbdmap  

[root@cnode1 ~]# scp /var/lib/ceph/bootstrap-osd/ceph.keyring root@cnode3:/var/lib/ceph/bootstrap-osd/
ceph.keyring 

[root@cnode1 ~]# scp /tmp/ceph.mon.keyring root@cnode3:/tmp/ceph.mon.keyring
ceph.mon.keyring 
```
- 在cnode3上创建一个默认的数据目录:

```ruby
[root@cnode3 ~]# sudo -u ceph mkdir /var/lib/ceph/mon/ceph-node3

```
- 在cnode3上修改ceph.mon.keyring属主和属组为ceph:

```ruby
[root@cnode3 ~]# chown ceph.ceph /tmp/ceph.mon.keyring

```

- 获取密钥和monmap信息：

```ruby
[root@cnode3 ~]# ceph auth get mon. -o /tmp/ceph.mon.keyring
exported keyring for mon.
[root@cnode3 ~]# ceph mon getmap -o /tmp/ceph.mon.map
got monmap epoch 2

```

- 初始化mon：

```ruby
[root@cnode3 ~]# sudo -u ceph ceph-mon \
--mkfs \
-i cnode3 \
--monmap /tmp/ceph.mon.map \
--keyring /tmp/ceph.mon.keyring

```
- 为了防止重新被安装创建一个空的done文件:

```ruby
[root@cnode3 ~]# sudo touch /var/lib/ceph/mon/ceph-cnode3/done

```
- 将新的mon节点添加至ceph集群的mon列表:

```ruby
[root@cnode3 ~]# ceph mon add cnode3 172.25.101.109:6789
adding mon.cnode3 at 172.25.101.109:6789/0

```
- 启动新添加的mon:

```ruby
[root@cnode3 ~]# systemctl start ceph-mon@cnode3
[root@cnode3 ~]# systemctl status ceph-mon@cnode3
● ceph-mon@cnode3.service - Ceph cluster monitor daemon
   Loaded: loaded (/usr/lib/systemd/system/ceph-mon@.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2018-08-08 07:52:59 EDT; 13s ago
 Main PID: 23853 (ceph-mon)
   CGroup: /system.slice/system-ceph\x2dmon.slice/ceph-mon@cnode3.service
           └─23853 /usr/bin/ceph-mon -f --cluster ceph --id cnode3 --setuser ceph --setgroup ceph

Aug 08 07:52:59 cnode3 systemd[1]: Started Ceph cluster monitor daemon.
Aug 08 07:52:59 cnode3 systemd[1]: Starting Ceph cluster monitor daemon...
[root@cnode3 ~]# systemctl enable ceph-mon@cnode3

```

- 查看目前的集群状态：

```ruby
[root@cnode1 ~]# ceph -s
  cluster:
    id:     a0b2ee0c-562c-43ca-86a1-d64fb84b8ce0
    health: HEALTH_WARN
            clock skew detected on mon.cnode1
 
  services:
    mon: 3 daemons, quorum cnode2,cnode3,cnode1
    mgr: no daemons active
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 bytes
    usage:   0 kB used, 0 kB / 0 kB avail
    pgs:     

```

```ruby
[root@cnode1 ~]# ceph df
GLOBAL:
    SIZE     AVAIL     RAW USED     %RAW USED 
    119G      113G        6786M          5.53 
POOLS:
    NAME     ID     USED     %USED     MAX AVAIL     OBJECTS 
```
```
[root@cnode1 ~]# ceph health detail
HEALTH_WARN clock skew detected on mon.cnode1
MON_CLOCK_SKEW clock skew detected on mon.cnode1
    mon.cnode1 addr 172.25.101.112:6789/0 clock skew 2.1273s > max 0.05s (latency 0.00107841s)
```
