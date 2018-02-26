## MySQL-DRBD
&emsp;&emsp;DRBD(Distributed Replicated Block Device)是基于块设备在不同的高可用服务器之间同步镜像数据的软件，通过它可以实现在网络中的两台服务器之间基于块设备级别之间的实时或异步镜像或同步复制，其实就类似于rsync+inotify这样的架构项目软件。只不过drbd是基于文件系统底层的，即block层级同步，而rsync+inotify是在文件系统之上的实际物理文件的同步，因此drbd的效率更高，效果更好；        
`上文提到的块设备可以是磁盘分区、LVM逻辑卷、整块磁盘等`    

### 1、DRBD工作原理介绍：
&emsp;&emsp;DRBD软件工作位置是在文件系统层级以下，比文件系统更加靠近操作系统内核及IO栈。在基于DRBD的高可用（HA）两台服务器主机中，当我们将数据写入到本地磁盘系统时，数据还会被实时的发送到网络中的另外一台主机上，并以相同的形式记录在另一个磁盘系统中，使得本地（主节点）与远程主机（备节点）的数据保持实时同步。     
&emsp;&emsp;这时，如果本地系统(主节点)发生故障，那么远程主机(备节点)上还会保留一份和主节点相同的数据备份可以继续使用，不但数据不会丢失，还会提升访问数据的用户访问体验，DRBD服务的作用类似于磁盘阵列里的RAID1功能，就相当于把网络中的两台服务器做成了类似磁盘阵列里的RAID1一样；      


### 2、DRBD两种工作模式：
#### 2.1 实时同步模式：

&emsp;&emsp;当数据写到本地磁盘和远端服务器磁盘都成功后才会返回成功写入。DRBD服务协议C级别就是这种模式，可以防止本地和远端数据丢失和不一致，此种模式是生产环境中最常用的模式；      

#### 2.2 异步同步模式：

&emsp;&emsp;当数据写入到本地服务器成功后就返回成功写入，不管远端服务器是否写入成功；       
&emsp;&emsp;还可能是数据写入到本地服务器或远端服的buffer成功后，返回成功，就是DRBD服务协议的A，B级别；     
`在nfs网络文件系统的时候也有类似的参数和功能,如：nfs服务参数sync和async，mount挂载参数也有sync和async`   

### 3、DRBD的同步协议：
&emsp;&emsp;DRBD的复制功能就是将应用程序提交的数据一份保存在本地节点，一份复制传输保存在另一个节点上。但是DRBD需要对传输的数据进行确认以便保证另一个节点的写操作完成，就需要用到DRBD的同步协议，DRBD同步协议有三种：
- 协议A：数据在本地完成写操作且数据已经发送到TCP/IP协议栈的队列中，则认为写操作完成。如果本地节点的写操作完成，此时本地节点发生故障，而数据还处在TCP/IP队列中，则数据不会发送到对端节点上。因此，两个节点的数据将不会保持一致。这种协议虽然高效，但是并不能保证数据的可靠性；    

- 协议B：数据在本地完成写操作且数据已到达对端节点则认为写操作完成。如果两个节点同时发生故障，即使数据到达对端节点，这种方式同样也会导致在对端节点和本地节点的数据不一致现象，也不具有可靠性；     

- 协议C：只有当本地节点的磁盘和对端节点的磁盘都完成了写操作，才认为写操作完成。这是集群流行的一种方式，应用也是最多的，这种方式虽然不高效，但是最可靠；    

### 4、DRBD生产应用模式：
- 单主模式：即主备模式，为典型的高可用性集群方案。
- 复制模式：需要采用共享Cluster文件系统，如GFS和OCFS2。需要从2个节点并发访问数据的场合，需要特别配置。

生产场景中DRBD常用与基于高可用服务器之间的数据同步解决方案,      
如：heartbeat+drbd+nfs/mfs/gfs,heartbeat+drbd+mysql/oracle等，实际上DRBD可以配合任意需要数据同步的所有服务的应用场景

### 5、安装准备：
#### 5.1 主机名配置与IP地址规划：
##### master服务器:
```ruby
[root@localhost ~]# hostnamectl set-hostname master
```
```ruby
[root@master ~]# ip a | grep ens
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    inet 172.30.105.101/24 brd 172.30.105.255 scope global ens33
3: ens38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    inet 192.168.1.101/24 brd 192.168.1.255 scope global ens38
4: ens39: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    inet 10.0.0.101/24 brd 10.0.0.255 scope global ens39

```
##### backup服务器:
```ruby
[root@localhost ~]# hostnamectl set-hostname backup
```
```ruby
[root@backup ~]# ip add | grep ens
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    inet 172.30.105.102/24 brd 172.30.105.255 scope global ens33
3: ens38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    inet 192.168.1.102/24 brd 192.168.1.255 scope global ens38
4: ens39: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    inet 10.0.0.102/24 brd 10.0.0.255 scope global ens39
```
#### 5.3 配置hosts文件：
##### master/slave服务器:
```ruby
# tail -2 /etc/hosts
172.30.105.101 master
172.30.105.102 backup

```

#### 5.4 防火墙设置：
##### master/slave服务器:
```ruby
# setenforce 0
# sed -i.bak "s/SELINUX=enforcing/SELINUX=permissive/g" /etc/selinux/config

# systemctl disable firewalld.service
# systemctl stop firewalld.service
# iptables --flush

```

#### 5.5 磁盘分区：
##### master服务器:
`Disk /dev/sdb: 10.7 GB`
```ruby
[root@master ~]# parted /dev/sdb mklabel gpt
Information: You may need to update /etc/fstab.

[root@master ~]# parted /dev/sdb mkpart primary 0 7000
Warning: The resulting partition is not properly aligned for best performance.
Ignore/Cancel? Ignore                                                     
Information: You may need to update /etc/fstab.
// 4800约等于6.5G

[root@master ~]# parted /dev/sdb mkpart primary 7001 100%
Information: You may need to update /etc/fstab.

[root@master ~]# parted /dev/sdb p 
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 10.7GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name     Flags
 1      17.4kB  7000MB  7000MB               primary
 2      7001MB  10.7GB  3735MB               primary

// 先不要格式化
```

##### backup服务器:
`分区大小和master服务器大小不一样，为方便后面的测试使用`
```ruby
[root@backup ~]# parted /dev/sdb mklabel gpt                              
Information: You may need to update /etc/fstab.

[root@backup ~]# parted /dev/sdb mkpart primary 0 9000 Ignore            
Warning: The resulting partition is not properly aligned for best performance.
Information: You may need to update /etc/fstab.


[root@backup ~]# parted /dev/sdb mkpart primary 9001 100%
Information: You may need to update /etc/fstab.

[root@backup ~]# parted /dev/sdb p                                        
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 10.7GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name     Flags
 1      17.4kB  9000MB  9000MB               primary
 2      9001MB  10.7GB  1735MB               primary


// 先不要格式化
```

### 6、安装部署DRBD：
#### 6.1 安装drbd软件：
##### master/slave服务器:
```ruby
# rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
# rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm

# yum install drbd84-utils kmod-drbd84
# rpm -qa | grep drbd
drbd84-utils-9.1.0-1.el7.elrepo.x86_64
kmod-drbd84-8.4.10-1_2.el7_4.elrepo.x86_64

```
#### 6.2 内核加载drbd模块：
##### master/slave服务器:
```ruby
# lsmod | grep -i drbd
// 检查drbd内核模块是否已经加载，如果没有自动加载，可以使用下面的命令进行加载 

[root@master ~]# modprobe drbd
[root@master ~]# lsmod | grep -i drbd
drbd                  524709  0 
libcrc32c              12644  4 xfs,drbd,nf_nat,nf_conntrack

```

### 7、配置文件介绍：

#### 7.1 主配置文件：  
##### master/slave服务器:
`/etc/drbd.conf`
```ruby
# more /etc/drbd.conf 
# You can find an example in  /usr/share/doc/drbd.../drbd.conf.example

include "drbd.d/global_common.conf";
include "drbd.d/*.res";
```
#### 7.2 全局配置文件：
##### master/slave服务器:
`/etc/drbd.d/global_common.conf`
```ruby
# ll /etc/drbd.d/
total 4
-rw-r--r--. 1 root root 2563 Sep 14 17:52 global_common.conf

```
```ruby
global {
	usage-count no;
	// 是否参加DRBD使用统计，默认为yes,官方统计drbd的装机量;
}
common {
        protocol C;
        // 使用DRBD的同步协议;
	handlers {
		pri-on-incon-degr "/usr/lib/drbd/notify-pri-on-incon-degr.sh; /usr/lib/drbd/notify-emergency-reboot.sh; echo b > /proc/sysrq-trigger ; reboot -f";
		pri-lost-after-sb "/usr/lib/drbd/notify-pri-lost-after-sb.sh; /usr/lib/drbd/notify-emergency-reboot.sh; echo b > /proc/sysrq-trigger ; reboot -f";
		local-io-error "/usr/lib/drbd/notify-io-error.sh; /usr/lib/drbd/notify-emergency-shutdown.sh; echo o > /proc/sysrq-trigger ; halt -f";
	}
	startup {
	}
	options {
	}
	disk {
                on-io-error detach;
                // 配置I/O错误处理策略为分离；
	}
	net {
	}
        syncer {
                rate 1024M;
                // 设置主备节点同步时的网络速率;
        }
}

```
```ruby
 on-io-error 策略可能为以下选项之一 
detach 分离：这是默认和推荐的选项，如果在节点上发生底层的硬盘I/O错误，它会将设备运行在Diskless无盘模式下 
pass_on：DRBD会将I/O错误报告到上层，在主节点上，它会将其报告给挂载的文件系统，但是在此节点上就往往忽略（因此此节点上没有可以报告的上层） 
-local-in-error：调用本地磁盘I/O处理程序定义的命令；这需要有相应的local-io-error调用的资源处理程序处理错误的命令；这就给管理员有足够自由的权力命令命令或是脚本调用local-io-error处理I/O错误 
```
#### 7.3 定义一个资源：
##### master/slave服务器:
```ruby
# vim /etc/drbd.d/mysql.res

resource mysql {
// 资源名称；
protocol C;
// 使用协议；
device /dev/drbd0;
// drbd设备名称；
meta-disk /dev/sdb2[0];

syncer {
verify-alg sha1;
// 加密算法；
}

net {
allow-two-primaries;

}

on master {
disk /dev/sdb1;
address 192.168.1.101:7789;
// drbd监控地址与端口；
}
    
on backup {
disk /dev/sdb1;
address 192.168.1.102:7789;
}
}

```
`可以直接将master配置好的配置文件拷贝至backup节点上`

```ruby
[root@master ~]# drbdadm create-md drbd0
initializing activity log
initializing bitmap (144 KB) to all zero
Writing meta data...
New drbd meta data block successfully created.

```


### 8、启动drbd并查看状态：
#### 8.1 master服务器启动drbd并查看状态：
##### master服务器:
```ruby
[root@master ~]# drbdadm create-md mysql
initializing activity log
initializing bitmap (112 KB) to all zero
Writing meta data...
New drbd meta data block successfully created.

[root@master ~]# drbdadm up mysql
```

```ruby

[root@master ~]# cat /proc/drbd 
version: 8.4.10-1 (api:1/proto:86-101)
GIT-hash: a4d5de01fffd7e4cde48a080e2c686f9e8cebf4c build by mockbuild@, 2017-09-15 14:23:22
 0: cs:WFConnection ro:Secondary/Unknown ds:Inconsistent/DUnknown C r----s
    ns:0 nr:0 dw:0 dr:0 al:8 bm:0 lo:0 pe:0 ua:0 ap:0 ep:1 wo:f oos:6835924

// 对端还没有启动，所以ro:Secondary/Unknown对端显示Unknown

```
`将master服务器强制置为primary`
```ruby
[root@master ~]# drbdadm -- --force primary mysql

[root@master ~]# cat /proc/drbd 
version: 8.4.10-1 (api:1/proto:86-101)
GIT-hash: a4d5de01fffd7e4cde48a080e2c686f9e8cebf4c build by mockbuild@, 2017-09-15 14:23:22
 0: cs:SyncSource ro:Primary/Secondary ds:UpToDate/Inconsistent C r-----
    ns:3757056 nr:0 dw:0 dr:3759178 al:8 bm:0 lo:0 pe:1 ua:0 ap:0 ep:1 wo:f oos:3079892
	[==========>.........] sync'ed: 55.1% (3004/6672)M
	finish: 0:05:53 speed: 8,696 (9,180) K/sec
```
```ruby
[root@master ~]# cat /proc/drbd 
version: 8.4.10-1 (api:1/proto:86-101)
GIT-hash: a4d5de01fffd7e4cde48a080e2c686f9e8cebf4c build by mockbuild@, 2017-09-15 14:23:22
 0: cs:Connected ro:Primary/Secondary ds:UpToDate/UpToDate C r-----
    ns:6835921 nr:0 dw:0 dr:6838043 al:8 bm:0 lo:0 pe:0 ua:0 ap:0 ep:1 wo:f oos:0

```
#### 8.2 slave服务器启动drbd并查看状态：
##### slave服务器:
```ruby
[root@backup drbd.d]# drbdadm create-md mysql
initializing activity log
initializing bitmap (52 KB) to all zero
Writing meta data...
New drbd meta data block successfully created.

```
```ruby
[root@backup ~]# drbdadm up mysql
[root@backup ~]# cat /proc/drbd
version: 8.4.10-1 (api:1/proto:86-101)
GIT-hash: a4d5de01fffd7e4cde48a080e2c686f9e8cebf4c build by mockbuild@, 2017-09-15 14:23:22
 0: cs:Connected ro:Secondary/Secondary ds:Inconsistent/Inconsistent C r-----
    ns:0 nr:0 dw:0 dr:0 al:16 bm:0 lo:0 pe:0 ua:0 ap:0 ep:1 wo:f oos:6835924

```

### 9、相关配置操作：
#### 9.1 资源连接状态：
```ruby
# drbdadm cstate mysql
Connected

```
```ruby
资源的连接状态：
StandAlone 独立的：网络配置不可用；资源还没有被连接或是被管理断开（使用 drbdadm disconnect 命令），或是由于出现认证失败或是脑裂的情况 
Disconnecting 断开：断开只是临时状态，下一个状态是StandAlone独立的 
Unconnected 悬空：是尝试连接前的临时状态，可能下一个状态为WFconnection和WFReportParams 
Timeout 超时：与对等节点连接超时，也是临时状态，下一个状态为Unconected悬空 
BrokerPipe：与对等节点连接丢失，也是临时状态，下一个状态为Unconected悬空 
NetworkFailure：与对等节点推动连接后的临时状态，下一个状态为Unconected悬空 
ProtocolError：与对等节点推动连接后的临时状态，下一个状态为Unconected悬空 
TearDown 拆解：临时状态，对等节点关闭，下一个状态为Unconected悬空 
WFConnection：等待和对等节点建立网络连接 
WFReportParams：已经建立TCP连接，本节点等待从对等节点传来的第一个网络包 
Connected 连接：DRBD已经建立连接，数据镜像现在可用，节点处于正常状态 
StartingSyncS：完全同步，有管理员发起的刚刚开始同步，未来可能的状态为SyncSource或PausedSyncS 
StartingSyncT：完全同步，有管理员发起的刚刚开始同步，下一状态为WFSyncUUID 
WFBitMapS：部分同步刚刚开始，下一步可能的状态为SyncSource或PausedSyncS 
WFBitMapT：部分同步刚刚开始，下一步可能的状态为WFSyncUUID 
WFSyncUUID：同步即将开始，下一步可能的状态为SyncTarget或PausedSyncT 
SyncSource：以本节点为同步源的同步正在进行 
SyncTarget：以本节点为同步目标的同步正在进行 
PausedSyncS：以本地节点是一个持续同步的源，但是目前同步已经暂停，可能是因为另外一个同步正在进行或是使用命令(drbdadm pause-sync)暂停了同步 
PausedSyncT：以本地节点为持续同步的目标，但是目前同步已经暂停，这可以是因为另外一个同步正在进行或是使用命令(drbdadm pause-sync)暂停了同步 
VerifyS：以本地节点为验证源的线上设备验证正在执行 
VerifyT：以本地节点为验证目标的线上设备验证正在执行 
```

#### 9.2 资源角色：
```ruby
# drbdadm role mysql
Primary/Secondary

[root@master ~]# cat /proc/drbd 
version: 8.4.10-1 (api:1/proto:86-101)
GIT-hash: a4d5de01fffd7e4cde48a080e2c686f9e8cebf4c build by mockbuild@, 2017-09-15 14:23:22
 0: cs:Connected ro:Primary/Secondary ds:UpToDate/UpToDate C r-----
    ns:6850687 nr:2050 dw:16829 dr:6845699 al:15 bm:0 lo:0 pe:0 ua:0 ap:0 ep:1 wo:f oos:0

```
```ruby
Parimary 主：资源目前为主，并且可能正在被读取或写入，如果不是双主只会出现在两个节点中的其中一个节点上 
Secondary 次：资源目前为次，正常接收对等节点的更新 
Unknown 未知：资源角色目前未知，本地的资源不会出现这种状态 
```

#### 9.3 硬盘状态：
```ruby
# drbdadm dstate mysql
UpToDate/UpToDate

```
```ruby
Diskless 无盘：本地没有块设备分配给DRBD使用，这表示没有可用的设备，或者使用drbdadm命令手工分离或是底层的I/O错误导致自动分离 
Attaching：读取无数据时候的瞬间状态 
Failed 失败：本地块设备报告I/O错误的下一个状态，其下一个状态为Diskless无盘 
Negotiating：在已经连接的DRBD设置进行Attach读取无数据前的瞬间状态 
Inconsistent：数据是不一致的，在两个节点上（初始的完全同步前）这种状态出现后立即创建一个新的资源。此外，在同步期间（同步目标）在一个节点上出现这种状态 
Outdated：数据资源是一致的，但是已经过时 
DUnknown：当对等节点网络连接不可用时出现这种状态 
Consistent：一个没有连接的节点数据一致，当建立连接时，它决定数据是UpToDate或是Outdated 
UpToDate：一致的最新的数据状态，这个状态为正常状态 
```
#### 9.4 启用和禁用资源：
```ruby
手动启用资源
drbdadm up <resource>
手动禁用资源
drbdadm down <resource>

// resource：为资源名称；当然也可以使用all表示[停用|启用]所有资源 
```

#### 9.5 升级和降级资源：
```ruby
升级资源
drbdadm primary <resource>
降级资源
drbdadm secondary <resource>
```

#### 9.6 初始化设备同步:
&emsp;&emsp;选择一个初始同步源；如果是新初始化的或是空盘，这个选择可以是任意的，但是如果其中的一个节点已经在使用并包含有用的数据，那么选择同步源是至关重要的；如果选错了初始化同步方向，就会造成数据丢失；      
&emsp;&emsp;启动初始化完全同步，这一步只能在初始化资源配置的一个节点上进行，并作为同步源选择的节点上；
```ruby
[root@master ~]# drbdadm -- --overwrite-data-of-peer primary mysql

[root@master ~]# cat /proc/drbd 
version: 8.4.10-1 (api:1/proto:86-101)
GIT-hash: a4d5de01fffd7e4cde48a080e2c686f9e8cebf4c build by mockbuild@, 2017-09-15 14:23:22
 0: cs:SyncSource ro:Primary/Secondary ds:UpToDate/Inconsistent C r-----
    ns:3757056 nr:0 dw:0 dr:3759178 al:8 bm:0 lo:0 pe:1 ua:0 ap:0 ep:1 wo:f oos:3079892
	[==========>.........] sync'ed: 55.1% (3004/6672)M
	finish: 0:05:53 speed: 8,696 (9,180) K/sec

```
`查看同步进度也可使用以下命令:`
```ruby
[root@master ~]# drbd-overview
NOTE: drbd-overview will be deprecated soon.
Please consider using drbdtop.

 0:mysql/0  Connected Primary/Secondary UpToDate/UpToDate /mydata/mysql xfs 6.6G 33M 6.5G 1% 
```
```ruby
Primary：当前节点为主；在前面为当前节点 
Secondary：备用节点为次 
```


### 10、挂载文件系统：
&emsp;&emsp;文件系统只能挂载在主(Primary)节点上，因此在设置好主节点后才可以对DRBD设备进行格式化操作 
#### 10.1 master服务器格式化文件系统：
```ruby
[root@master ~]# mkdir -p /mydata/mysql

```
```ruby
[root@master ~]# mkfs.xfs /dev/drbd0
meta-data=/dev/drbd0             isize=512    agcount=4, agsize=427245 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=1708980, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

```
#### 10.2 master服务器挂载文件系统：
```ruby
[root@master ~]# mount /dev/drbd0 /mydata/mysql/

```

### 11、切换主备节点：
#### 11.1 master主节点降级：
```ruby
[root@master ~]# cd /mydata/mysql/
[root@master mysql]# touch demo1 demo2 demo3 demo4 demo5

```
```ruby
[root@master ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root   17G  1.7G   16G  10% /
devtmpfs                 869M     0  869M   0% /dev
tmpfs                    880M     0  880M   0% /dev/shm
tmpfs                    880M  8.6M  871M   1% /run
tmpfs                    880M     0  880M   0% /sys/fs/cgroup
/dev/sda1               1014M  143M  872M  15% /boot
tmpfs                    176M     0  176M   0% /run/user/0
/dev/drbd0               6.6G   33M  6.5G   1% /mydata/mysql

[root@master ~]# umount /mydata/mysql/
[root@master ~]# drbdadm secondary mysql
[root@master ~]# cat /proc/drbd 
version: 8.4.10-1 (api:1/proto:86-101)
GIT-hash: a4d5de01fffd7e4cde48a080e2c686f9e8cebf4c build by mockbuild@, 2017-09-15 14:23:22
 0: cs:Connected ro:Secondary/Secondary ds:UpToDate/UpToDate C r-----
    ns:6848639 nr:0 dw:12731 dr:6842452 al:15 bm:0 lo:0 pe:0 ua:0 ap:0 ep:1 wo:f oos:0

```
#### 11.2 backup节点升级：
```ruby
[root@backup ~]# drbdadm primary mysql

[root@backup ~]# cat /proc/drbd 
version: 8.4.10-1 (api:1/proto:86-101)
GIT-hash: a4d5de01fffd7e4cde48a080e2c686f9e8cebf4c build by mockbuild@, 2017-09-15 14:23:22
 0: cs:Connected ro:Primary/Secondary ds:UpToDate/UpToDate C r-----
    ns:0 nr:60 dw:60 dr:2122 al:0 bm:0 lo:0 pe:0 ua:0 ap:0 ep:1 wo:f oos:0
```
挂载设备并验证文件是否存在:
```
[root@backup ~]# mount /dev/drbd0 /mnt/
[root@backup ~]# ls /mnt/
demo1  demo2  demo3  demo4  demo5

```
### 12、DRBD脑裂的模拟及修复：
`模拟master主节点发生了故障，然后将backup节点升级为primary节点`
#### 12.1 断开主(primary)节点：
&emsp;&emsp;关机、断开网络或重新配置其他的IP都可以,这里选择的是断开网络    


```ruby
[root@master ~]# ifdown ens38
Device 'ens38' successfully disconnected.
```
#### 12.2 查看两个节点的状态：
##### master服务器：
```ruby
[root@master ~]# drbd-overview
NOTE: drbd-overview will be deprecated soon.
Please consider using drbdtop.

 0:mysql/0  StandAlone Primary/Unknown UpToDate/DUnknown /mydata/mysql xfs 6.6G 33M 6.5G 1% 
```
##### backup服务器：
```ruby
[root@backup ~]# drbd-overview
NOTE: drbd-overview will be deprecated soon.
Please consider using drbdtop.

 0:mysql/0  WFConnection Secondary/Unknown UpToDate/DUnknown 
```

#### 12.3 将backup节点升级为主(primary)节点并挂载资源：
```ruby
[root@backup ~]# drbdadm primary mysql
[root@backup ~]# drbd-overview
NOTE: drbd-overview will be deprecated soon.
Please consider using drbdtop.

 0:mysql/0  WFConnection Primary/Unknown UpToDate/DUnknown 


[root@backup ~]# mount /dev/drbd0 /mnt

```
#### 12.4 原来的主(primary)节点修复好重新上线了（这时出现了脑裂情况）
#### 12.5 再次查看两节点的状态：
##### master服务器：
```ruby
[root@master ~]# drbdadm role mysql
Primary/Unknown

[root@master ~]# drbd-overview
 0:mysql/0  StandAlone Primary/Unknown UpToDate/DUnknown /mydata/mysql xfs 6.6G 33M 6.5G 1% 

// 状态为StandAlone时，主备节点是不会通信的 
```
##### backup服务器：
```ruby
[root@backup ~]# drbdadm role mysql
Primary/Unknown


[root@backup ~]# drbd-overview
 0:mysql/0  WFConnection Primary/Unknown UpToDate/DUnknown /mnt xfs 6.6G 33M 6.5G 1% 
```

#### 12.6 backup服务器上处理办法：
```ruby
[root@backup ~]# umount /mnt

[root@backup ~]# drbdadm disconnect mysql
[root@backup ~]# drbdadm secondary mysql
[root@backup ~]# drbd-overview
 0:mysql/0  StandAlone Secondary/Unknown UpToDate/DUnknown 

// 发现还是不可用状态,需要再重新连接一下
```
```ruby
root@backup ~]# drbdadm -- --discard-my-data connect mysql

```
在master服务器上，通过cat /proc/drbd查看状态，如果不是WFConnection状态，需要再手动连接：
```ruby
[root@master ~]# drbdadm connect mysql

```

### 13、FAQ:
```ruby
[root@backup ~]# umount /mnt
umount: /mnt: target is busy.
        (In some cases useful info about processes that use
         the device is found by lsof(8) or fuser(1))

[root@backup ~]# lsof | grep /mnt
bash       1487         root  cwd       DIR               8,17         6         64 /mnt
lsof      12653         root  cwd       DIR               8,17         6         64 /mnt
grep      12654         root  cwd       DIR               8,17         6         64 /mnt
lsof      12655         root  cwd       DIR               8,17         6         64 /mnt
[root@backup ~]# ps -ef | grep 1487
root       1487   1483  0 20:46 pts/0    00:00:00 -bash
root      12656   1487  0 22:35 pts/0    00:00:00 ps -ef
root      12657   1487  0 22:35 pts/0    00:00:00 grep --color=auto 1487
[root@backup ~]# kill -9 1487

[root@backup ~]# umount /mnt
[root@backup ~]# drbdadm up mysql
```