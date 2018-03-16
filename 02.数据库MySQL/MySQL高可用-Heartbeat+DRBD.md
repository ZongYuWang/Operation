
## CentOS7:MySQL高可用-Heartbeat+DRBD(二)

`CentOS7的部署中有点问题，见最后的FAQ，目前还没有解决`

[Heartbeat安装文档](https://github.com/ZongYuWang/Operation/blob/master/04.%E4%BB%A3%E7%90%86%E8%B4%9F%E8%BD%BD/Heartbeat/Heartbeat.md)
[DRBD安装文档](https://github.com/ZongYuWang/Operation/blob/master/02.%E6%95%B0%E6%8D%AE%E5%BA%93MySQL/DRBD.md)

### 1、环境配置：

#### 1.1 设置主机名：
##### master/backup服务器：
```ruby
[root@localhost ~]# hostnamectl set-hostname master
[root@localhost ~]# hostnamectl set-hostname backup

```

#### 1.2 网络设置：
##### master/backup服务器：
```ruby
[root@master ~]# cd /etc/sysconfig/network-scripts/
[root@master network-scripts]# cp ifcfg-ens33 ifcfg-ens38
[root@master network-scripts]# cp ifcfg-ens33 ifcfg-ens39

```
##### master服务器：
```ruby
[root@master ~]# ip add | grep ens
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    inet 172.30.105.101/24 brd 172.30.105.255 scope global ens33
3: ens38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    inet 192.168.1.101/24 brd 192.168.1.255 scope global ens38
4: ens39: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    inet 10.0.0.101/24 brd 10.0.0.255 scope global ens39

```
服务器角色 | 网卡名称 | IP地址 | 备注 | 
|:-: | :-: | :-: | :- | 
master | ens33| 172.30.105.101 | 管理IP，用于WAN数据转发|
| | ens38| 192.168.1.101 | DRBD|
| | ens39| 10.0.0.101 | 用于服务期间心跳连接(直连)|
VIP地址 | | 172.30.105.201 | |
##### backup服务器：
```ruby
[root@backup ~]# ip add | grep ens
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    inet 172.30.105.102/24 brd 172.30.105.255 scope global ens33
3: ens38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    inet 192.168.1.102/24 brd 192.168.1.255 scope global ens38
4: ens39: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    inet 10.0.0.102/24 brd 10.0.0.255 scope global ens39

```

服务器角色 | 网卡名称 | IP地址 | 备注 | 
|:-: | :-: | :-: | :- | 
master | ens33| 172.30.105.102 | 管理IP，用于WAN数据转发|
| | ens38| 192.168.1.102 | DRBD|
| | ens39| 10.0.0.102 | 用于服务期间心跳连接(直连)|

### 2、安装部署DRBD
[安装DRBD-安装部署DRBD部分](https://github.com/ZongYuWang/Operation/blob/master/02.%E6%95%B0%E6%8D%AE%E5%BA%93MySQL/DRBD.md)


<table class="table table-bordered table-striped table-condensed">
    <tr>
        <th align="left">主机名称</th>
    <td>master</td>
    <td>backup</td>
    </tr>
    <tr>
        <th align="left">管理IP  </th>
    <td>ens33:172.30.105.101</td>
    <td>ens33:172.30.105.102</td>
    </tr>
     <tr>
        <th align="left">DRBD管理名称</th>
    <td>mysql0</td>
    <td>mysql0</td>
    </tr>
     <tr>
        <th align="left">DRBD挂载目录</th>
    <td>/data</td>
    <td>/data</td>
    </tr>
     <tr>
        <th align="left">DRBD逻辑设备</th>
    <td>/dev/drbd0</td>
    <td>/dev/drbd0</td>
    </tr>
     <tr>
        <th align="left">DRBD对接IP</th>
    <td>ens38:192.168.1.101/24</td>
    <td>ens38:192.168.1.102/24</td>
    </tr>
      <tr>
        <th align="left">DRBD存储设备</th>
    <td>/dev/sdb1</td>
    <td>/dev/sdb1</td>
    </tr>
      <tr>
        <th align="left">DRBD Meta设备</th>
    <td>internal</td>
    <td>internal</td>
    </tr>
      <tr>
        <th align="left">NFS导出目录</th>
    <td>/data(暂时没有使用)</td>
    <td>/data(暂时没有使用)</td>
    </tr>
    <tr>
        <th align="left">NFS虚拟IP</th>
    <td>ens33:172.30.105.201</td>
    <td>ens33:172.30.105.201</td>
    </tr>
</table>


#### 2.1 创建drbd磁盘：
```ruby
[root@master ~]# parted /dev/sdb p
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  21.5GB  21.5GB  primary


```
```ruby
[root@backup ~]# parted /dev/sdb p
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 10.7GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  10.7GB  10.7GB  primary


```

```ruby

[root@master ~]# dd if=/dev/zero of=/dev/sdb1 bs=1M count=100
100+0 records in
100+0 records out
104857600 bytes (105 MB) copied, 0.889133 s, 118 MB/s

[root@master ~]# drbdadm create-md mysql0
initializing activity log
initializing bitmap (640 KB) to all zero
Writing meta data...
New drbd meta data block successfully created.
success


```

#### 2.2 启动drbd并查看状态：
##### master服务器：
```ruby
[root@master ~]# cat /proc/drbd
version: 8.4.10-1 (api:1/proto:86-101)
GIT-hash: a4d5de01fffd7e4cde48a080e2c686f9e8cebf4c build by mockbuild@, 2017-09-15 14:23:22
 0: cs:Connected ro:Secondary/Secondary ds:Inconsistent/Inconsistent C r-----
    ns:0 nr:0 dw:0 dr:0 al:16 bm:0 lo:0 pe:0 ua:0 ap:0 ep:1 wo:f oos:10484380

// 注意着的ds状态是Inconsistent/Inconsistent，如果不使用上面的dd命令，那么这地是Diskless/Diskless状态
```

```ruby
[root@master ~]# drbdadm -- --overwrite-data-of-peer primary all
[root@master ~]# mkfs.ext4 /dev/drbd0
[root@master ~]# mkdir -p /mysql/data
[root@master ~]# mount /dev/drbd0 /mysql/data/

```
```ruby
[root@master ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root   17G  1.1G   16G   7% /
devtmpfs                 446M     0  446M   0% /dev
tmpfs                    456M     0  456M   0% /dev/shm
tmpfs                    456M  6.3M  450M   2% /run
tmpfs                    456M     0  456M   0% /sys/fs/cgroup
/dev/sda1               1014M  125M  890M  13% /boot
tmpfs                     92M     0   92M   0% /run/user/0
/dev/drbd0               9.8G   37M  9.2G   1% /mysql/data
[root@master ~]# 
[root@master ~]# cat /proc/drbd
version: 8.4.10-1 (api:1/proto:86-101)
GIT-hash: a4d5de01fffd7e4cde48a080e2c686f9e8cebf4c build by mockbuild@, 2017-09-15 14:23:22
 0: cs:SyncSource ro:Primary/Secondary ds:UpToDate/Inconsistent C r-----
    ns:4160860 nr:0 dw:299360 dr:3865813 al:89 bm:0 lo:0 pe:0 ua:0 ap:0 ep:1 wo:f oos:6360524
	[======>.............] sync'ed: 39.4% (6208/10236)M
	finish: 0:02:57 speed: 35,840 (29,244) K/sec

```
`同步完成后的状态：`
```ruby
[root@master ~]# cat /proc/drbd
version: 8.4.10-1 (api:1/proto:86-101)
GIT-hash: a4d5de01fffd7e4cde48a080e2c686f9e8cebf4c build by mockbuild@, 2017-09-15 14:23:22
 0: cs:Connected ro:Primary/Secondary ds:UpToDate/UpToDate C r-----
    ns:10521384 nr:0 dw:299360 dr:10226337 al:89 bm:0 lo:0 pe:0 ua:0 ap:0 ep:1 wo:f oos:0

```
##### backup服务器：
```ruby
[root@backup ~]# mkdir -p /mysql/data

```

### 3、简化安装数据库：
#### 3.1 编译安装方式：
```ruby
[root@master ~]# tar xf mariadb-5.5.56-linux-x86_64.tar.gz -C /usr/local/
[root@master ~]# cd /usr/local/
[root@master local]# ln -s mariadb-5.5.56-linux-x86_64/ mysql

[root@master local]# useradd -r -g mysql mysql
[root@master local]# cd mysql

[root@master mysql]# chown -R mysql .
[root@master mysql]# chgrp -R mysql .
[root@master mysql]# mkdir -p /mysql/data/mydata/
[root@master mysql]# chown -R mysql.mysql /mysql/data/mydata/

[root@master mysql]# /usr/local/mysql/scripts/mysql_install_db --user=mysql --datadir=/mysql/data/mydata/ --basedir=/usr/local/mysql 
[root@master mysql]# chown -R root .
         
[root@master mysql]# cp support-files/my-medium.cnf /etc/my.cnf
cp: overwrite ‘/etc/my.cnf’? y

[root@master mysql]# cp support-files/mysql.server /etc/init.d/mysqld
[root@master mysql]# 
[root@master mysql]# chmod 755 /etc/init.d/mysqld 
[root@master mysql]# egrep 'datadir|basedir' /etc/my.cnf 

[root@master mysql]# vim /etc/my.cnf
// 添加上如下内容：

[mysqld]
......
datadir=/mysql/data/mydata
basedir=/usr/local/mysql
......
```


#### 3.2 YUM安装方式：
##### master/slave服务器：
```ruby
# yum install -y mariadb-server
# systemctl start mariadb.service

```
##### slave服务器：
`slave服务器的mysql不要启动，交给heartbeat启动`
```ruby
[root@backup ~]# mkdir -p /mydata/mysql/

// master服务器的数据目录也是/mydata/mysql/

```

##### 变更数据目录：
```ruby
# yum install -y rsync
```
```ruby
# systemctl start mariadb.service
# mysql
MariaDB [(none)]> select @@datadir;
+-----------------+
| @@datadir       |
+-----------------+
| /var/lib/mysql/ |
+-----------------+
1 row in set (0.00 sec)

# systemctl stop mariadb.service

```
```ruby
# ll /var/lib/mysql/
total 28700
-rw-rw----. 1 mysql mysql    16384 Feb 26 21:20 aria_log.00000001
-rw-rw----. 1 mysql mysql       52 Feb 26 21:20 aria_log_control
-rw-rw----. 1 mysql mysql 18874368 Feb 26 21:20 ibdata1
-rw-rw----. 1 mysql mysql  5242880 Feb 26 21:20 ib_logfile0
-rw-rw----. 1 mysql mysql  5242880 Feb 26 21:00 ib_logfile1
drwx------. 2 mysql mysql     4096 Feb 26 21:00 mysql
drwx------. 2 mysql mysql     4096 Feb 26 21:00 performance_schema
drwx------. 2 mysql mysql        6 Feb 26 21:00 test

# rsync -av /var/lib/mysql/ /mydata/mysql/
# ll /mydata/mysql/
total 28708
-rw-rw----. 1 mysql mysql    16384 Feb 26 21:20 aria_log.00000001
-rw-rw----. 1 mysql mysql       52 Feb 26 21:20 aria_log_control
-rw-rw----. 1 mysql mysql 18874368 Feb 26 21:20 ibdata1
-rw-rw----. 1 mysql mysql  5242880 Feb 26 21:20 ib_logfile0
-rw-rw----. 1 mysql mysql  5242880 Feb 26 21:00 ib_logfile1
drwx------. 2 mysql mysql     8192 Feb 26 21:00 mysql
drwx------. 2 mysql mysql     4096 Feb 26 21:00 performance_schema
drwx------. 2 mysql mysql        6 Feb 26 21:00 test

# mv /var/lib/mysql /var/lib/mysql.bak
```

```ruby
# vim /etc/my.cnf
.......
[mysqld]
datadir=/mydata/mysql/mysql
socket=/mydata/mysql/mysql/mysql.sock
......

[client]
port=3306
socket=/mydata/mysql/mysql/mysql.sock

```
```ruby
# systemctl start mariadb.service
# systemctl status mariadb.service
● mariadb.service - MariaDB database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; disabled; vendor preset: disabled)
   Active: active (running) since Mon 2018-02-26 21:30:21 EST; 7s ago
  Process: 15077 ExecStartPost=/usr/libexec/mariadb-wait-ready $MAINPID (code=exited, status=0/SUCCESS)
  Process: 15046 ExecStartPre=/usr/libexec/mariadb-prepare-db-dir %n (code=exited, status=0/SUCCESS)
 Main PID: 15076 (mysqld_safe)
   CGroup: /system.slice/mariadb.service
           ├─15076 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
           └─15239 /usr/libexec/mysqld --basedir=/usr --datadir=/mydata/mysql/mysql --plugin-dir=/usr/lib64/mysql...

Feb 26 21:30:19 master systemd[1]: Starting MariaDB database server...
Feb 26 21:30:19 master mariadb-prepare-db-dir[15046]: Database MariaDB is probably initialized in /mydata/mys...one.
Feb 26 21:30:19 master mysqld_safe[15076]: 180226 21:30:19 mysqld_safe Logging to '/var/log/mariadb/mariadb.log'.
Feb 26 21:30:19 master mysqld_safe[15076]: 180226 21:30:19 mysqld_safe Starting mysqld daemon with databases...mysql
Feb 26 21:30:21 master systemd[1]: Started MariaDB database server.
Hint: Some lines were ellipsized, use -l to show in full.

MariaDB [(none)]> select @@datadir;
+----------------------+
| @@datadir            |
+----------------------+
| /mydata/mysql/mysql/ |
+----------------------+
1 row in set (0.00 sec)

```

```ruby
# systemctl disable heartbeat.service
# systemctl disable drbd.service

```
`有时候得重新启动一下服务器`


### 4、测试DRBD的切换：

#### 4.1 手动测试切换：
##### master服务器：
```ruby
[root@master ~]# umount /mysql/data/
[root@master ~]# drbdadm secondary all

```
##### backup服务器：
```ruby
[root@backup ~]# drbdadm  primary all 
[root@backup ~]# mount /dev/drbd0 /mysql/data/
[root@backup ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root   17G  2.3G   15G  14% /
devtmpfs                 869M     0  869M   0% /dev
tmpfs                    880M     0  880M   0% /dev/shm
tmpfs                    880M  8.7M  871M   1% /run
tmpfs                    880M     0  880M   0% /sys/fs/cgroup
/dev/sda1               1014M  143M  872M  15% /boot
tmpfs                    176M     0  176M   0% /run/user/0
/dev/drbd0               9.8G   38M  9.2G   1% /mysql/data

[root@backup ~]# cat /proc/drbd
version: 8.4.10-1 (api:1/proto:86-101)
GIT-hash: a4d5de01fffd7e4cde48a080e2c686f9e8cebf4c build by mockbuild@, 2017-09-15 14:23:22
 0: cs:Connected ro:Primary/Secondary ds:UpToDate/UpToDate C r-----
    ns:4 nr:10524468 dw:10524472 dr:3201 al:9 bm:0 lo:0 pe:0 ua:0 ap:0 ep:1 wo:f oos:0
```

`master服务器数据库目录里面就为空了：`
```ruby
[root@master ha.d]# cd /mysql/data/
[root@master data]# ls
[root@master data]# 
```

### 5、安装Heartbeat：

[CentOS7安装heartbeat](https://github.com/ZongYuWang/Operation/blob/master/04.%E4%BB%A3%E7%90%86%E8%B4%9F%E8%BD%BD/Heartbeat/Heartbeat.md)

[root@master heartbeat]# tar jxf Cluster_Glue_1.0.12.bz2 

// 注意下这个解压命令，heartbeat中需要修改


[root@master ha.d]# chkconfig mysqld off
[root@master ha.d]# systemctl disable heartbeat.service
[root@master ha.d]# systemctl disable drbd.service



### 6、FAQ：



```ruby
2018/02/28_03:12:44 ERROR: Setup problem: couldn't find command: fuser

```
```ruby
2018/02/28_03:12:44 ERROR: Return code 5 from /usr/local/heartbeat/etc/ha.d/resource.d/Filesystem

```
```ruby
ResourceManager(default)[57922]:	2018/02/28_03:12:45 ERROR: Cannot locate resource script drbddisk
```





## CentOS6:MySQL高可用-Heartbeat+DRBD

### 1、环境设置：
#### 1.1 修改hostname：
##### master/backup服务器：
```ruby
[root@localhost ~]# hostname master
[root@localhost ~]# hostname backup
```

#### 1.2 网络设置：
##### master服务器：
```ruby
[root@master ~]# ip a | grep eth
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    link/ether 00:0c:29:54:7a:ef brd ff:ff:ff:ff:ff:ff
    inet 172.30.105.121/24 brd 172.30.105.255 scope global eth0
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    link/ether 00:0c:29:54:7a:f9 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.121/24 brd 192.168.0.255 scope global eth1
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    link/ether 00:0c:29:54:7a:03 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.121/24 brd 10.0.0.255 scope global eth2
```

服务器角色 | 网卡名称 | IP地址 | 备注 | 
|:-: | :-: | :-: | :- | 
master | eth0| 172.30.105.121 | 管理IP，用于WAN数据转发|
| | eth1| 192.168.0.121 | DRBD|
| | eth2| 10.0.0.121 | 用于服务期间心跳连接(直连)|
VIP地址 | | 172.30.105.201 | |

##### backup服务器：
```ruby
[root@backup ~]# ip a | grep eth
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    link/ether 00:0c:29:da:65:76 brd ff:ff:ff:ff:ff:ff
    inet 172.30.105.122/24 brd 172.30.105.255 scope global eth0
3: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    link/ether 00:0c:29:da:65:80 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.122/24 brd 10.0.0.255 scope global eth2
4: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    link/ether 00:0c:29:da:65:8a brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.122/24 brd 192.168.0.255 scope global eth1

```
服务器角色 | 网卡名称 | IP地址 | 备注 | 
|:-: | :-: | :-: | :- | 
master | eth0| 172.30.105.122 | 管理IP，用于WAN数据转发|
| | eth1| 192.168.0.122 | DRBD|
| | eth2| 10.0.0.122 | 用于服务期间心跳连接(直连)|

#### 1.3 安装依赖软件包：
##### master/backup服务器：
```ruby
[root@master ~]# yum install -y kernel kernel-devel kernel-headers  flex 

// 需要重新启动服务器
```
### 2、安装DRBD并配置DRBD：

#### 2.1 创建DRBD磁盘：
```ruby
[root@master ~]# parted /dev/sdb p
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos

Number  Start   End     Size    Type     File system  Flags
 1      32.3kB  21.5GB  21.5GB  primary

```

```ruby
[root@backup ~]# parted /dev/sdb p
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 10.7GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos

Number  Start   End     Size    Type     File system  Flags
 1      32.3kB  10.7GB  10.7GB  primary

```
```ruby
[root@master ~]# dd if=/dev/zero of=/dev/sdb1 bs=1M count=100

```

#### 2.2 安装DRBD软件：
##### master/backup服务器：
```ruby
[root@master ~]# wget http://www.elrepo.org/elrepo-release-6-8.el6.elrepo.noarch.rpm
[root@master ~]# yum -y install elrepo-release-6-8.el6.elrepo.noarch.rpm
[root@master ~]# sed -i 's#keepcache=0#keepcache=1#g' /etc/yum.conf 

[root@master ~]# yum install kernel-devel kernel-headers flex drbd84-utils kmod-drbd84 drbd

[root@master ~]# rpm -qa | grep drbd
drbd84-utils-8.9.8-1.el6.elrepo.x86_64
kmod-drbd84-8.4.9-1.el6.elrepo.x86_64

[root@master ~]# export LC_ALL=C
[root@master ~]# lsmod | grep drbd

[root@master ~]# modprobe drbd
[root@master ~]# lsmod | grep drbd
drbd                  374888  0 
libcrc32c               1246  1 drbd
[root@master ~]# echo "modprobe drbd >/dev/null 2>&1" > /etc/sysconfig/modules/drbd.modules
```
#### 2.3 配置drbd：
```ruby
[root@master ~]# vim /etc/drbd.d/global_common.conf
```
```ruby
global { usage-count no; }
common { syncer { rate 20M; } } 
resource mysql0 {                    
        protocol C;                
        startup {
        }
        disk {
                on-io-error detach;
        }
        net {
        }
        on master {    
                device /dev/drbd0; 
                disk /dev/sdb1;
                address 192.168.0.121:7888; 
                meta-disk internal;
        }
        on backup {
                device /dev/drbd0;
                disk /dev/sdb1;
                address 192.168.0.122:7888;
                meta-disk internal;  
       }
}

```

#### 2.3 启动drbd并查看状态：
```ruby
[root@master ~]# drbdadm create-md mysql0
initializing activity log
NOT initializing bitmap
Writing meta data...
New drbd meta data block successfully created.
```

```ruby
[root@master ~]# drbdadm -- --overwrite-data-of-peer primary mysql0

[root@master ~]# cat /proc/drbd
version: 8.4.9-1 (api:1/proto:86-101)
GIT-hash: 9976da086367a2476503ef7f6b13d4567327a280 build by mockbuild@Build64R6, 2016-12-13 18:38:15
 0: cs:SyncSource ro:Primary/Secondary ds:UpToDate/Inconsistent C r-----
    ns:37180 nr:0 dw:0 dr:38888 al:16 bm:0 lo:0 pe:2 ua:5 ap:0 ep:1 wo:f oos:10444864
	[>....................] sync'ed:  0.4% (10200/10236)M
	finish: 0:18:43 speed: 9,288 (9,288) K/sec

```
`状态更新完成：`
```ruby
[root@master ~]# cat /proc/drbd
version: 8.4.9-1 (api:1/proto:86-101)
GIT-hash: 9976da086367a2476503ef7f6b13d4567327a280 build by mockbuild@Build64R6, 2016-12-13 18:38:15
 0: cs:Connected ro:Primary/Secondary ds:UpToDate/UpToDate C r-----
    ns:10482024 nr:0 dw:0 dr:10482688 al:16 bm:0 lo:0 pe:0 ua:0 ap:0 ep:1 wo:f oos:0

```
##### master服务器：
```ruby
[root@master ~]# /etc/init.d/drbd status
drbd driver loaded OK; device status:
version: 8.4.9-1 (api:1/proto:86-101)
GIT-hash: 9976da086367a2476503ef7f6b13d4567327a280 build by mockbuild@Build64R6, 2016-12-13 18:38:15
m:res     cs         ro                 ds                 p  mounted  fstype
0:mysql0  Connected  Primary/Secondary  UpToDate/UpToDate  C

```
##### backup服务器：
```ruby
[root@backup ~]# /etc/init.d/drbd status
drbd driver loaded OK; device status:
version: 8.4.9-1 (api:1/proto:86-101)
GIT-hash: 9976da086367a2476503ef7f6b13d4567327a280 build by mockbuild@Build64R6, 2016-12-13 18:38:15
m:res     cs         ro                 ds                 p  mounted  fstype
0:mysql0  Connected  Secondary/Primary  UpToDate/UpToDate  C

```
```ruby
[root@master ~]# service iptables stop
[root@master ~]# setenforce 0
[root@master ~]# chkconfig iptables off

```
#### 2.4 挂载DRBD分区：
##### master服务器：
```ruby
[root@master ~]# mkfs.ext4 /dev/drbd0
[root@master ~]# mkdir /data
[root@master ~]# 
[root@master ~]# mount /dev/drbd0 /data/

[root@master ~]# df -h
Filesystem                     Size  Used Avail Use% Mounted on
/dev/mapper/vg_mysql1-lv_root   26G 1019M   24G   5% /
tmpfs                          733M     0  733M   0% /dev/shm
/dev/sda1                      477M   51M  401M  12% /boot
/dev/drbd0                     9.8G   23M  9.2G   1% /data
```
##### backup服务器：
```ruby
[root@backup ~]# mkdir /data

```

### 3、安装Heartbeat软件：(缺配置文件)

#### 3.1 安装Heartbeat：
##### master/backup服务器：
```ruby
[root@master ~]# rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
[root@master ~]# yum install heartbeat -y

[root@master ~]# cd /etc/ha.d/
[root@master ~]# chmod 600 /etc/ha.d/authkeys

```
```ruby
[root@master ~]# cat /etc/rc.local 
#!/bin/sh
#
# This script will be executed *after* all the other init scripts.
# You can put your own initialization stuff in here if you don't
# want to do the full Sys V style init stuff.

touch /var/lock/subsys/local

modprobe drbd              
/etc/init.d/drbd start
/etc/init.d/heartbeat start

```

#### 3.2 启动heartbeat服务：
```
[root@master ~]# /etc/init.d/heartbeat start
Starting High-Availability services: INFO:  Resource is stopped
Done.
```

```ruby
[root@master ~]# ip a | grep eth
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    link/ether 00:0c:29:54:7a:ef brd ff:ff:ff:ff:ff:ff
    inet 172.30.105.121/24 brd 172.30.105.255 scope global eth0
    inet 172.30.105.201/24 brd 172.30.105.255 scope global secondary eth0
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    link/ether 00:0c:29:54:7a:f9 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.121/24 brd 192.168.0.255 scope global eth1
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    link/ether 00:0c:29:54:7a:03 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.121/24 brd 10.0.0.255 scope global eth2

```

```ruby
[root@master ~]# tail -n 20 /var/log/ha-log 
Mar 01 23:20:09 master heartbeat: [3036]: info: Resources being acquired from backup.
harc(default)[3047]:	2018/03/01_23:20:09 info: Running /etc/ha.d//rc.d/status status
mach_down(default)[3087]:	2018/03/01_23:20:10 info: /usr/share/heartbeat/mach_down: nice_failback: foreign resources acquired
mach_down(default)[3087]:	2018/03/01_23:20:10 info: mach_down takeover complete for node backup.
Mar 01 23:20:10 master heartbeat: [3036]: info: Initial resource acquisition complete (T_RESOURCES(us))
Mar 01 23:20:10 master heartbeat: [3036]: info: mach_down takeover complete.
/usr/lib/ocf/resource.d//heartbeat/IPaddr(IPaddr_172.30.105.201)[3111]:	2018/03/01_23:20:10 INFO:  Resource is stopped
Mar 01 23:20:10 master heartbeat: [3048]: info: Local Resource acquisition completed.
harc(default)[3220]:	2018/03/01_23:20:10 info: Running /etc/ha.d//rc.d/ip-request-resp ip-request-resp
ip-request-resp(default)[3220]:	2018/03/01_23:20:10 received ip-request-resp IPaddr::172.30.105.201/24/eth0 OK yes
ResourceManager(default)[3243]:	2018/03/01_23:20:10 info: Acquiring resource group: master IPaddr::172.30.105.201/24/eth0 drbddisk::mysql0 Filesystem::/dev/drbd0::/data::ext4 mysqld
/usr/lib/ocf/resource.d//heartbeat/IPaddr(IPaddr_172.30.105.201)[3271]:	2018/03/01_23:20:10 INFO:  Resource is stopped
ResourceManager(default)[3243]:	2018/03/01_23:20:10 info: Running /etc/ha.d/resource.d/IPaddr 172.30.105.201/24/eth0 start
IPaddr(IPaddr_172.30.105.201)[3396]:	2018/03/01_23:20:11 INFO: Adding inet address 172.30.105.201/24 with broadcast address 172.30.105.255 to device eth0
IPaddr(IPaddr_172.30.105.201)[3396]:	2018/03/01_23:20:11 INFO: Bringing device eth0 up
IPaddr(IPaddr_172.30.105.201)[3396]:	2018/03/01_23:20:11 INFO: /usr/libexec/heartbeat/send_arp -i 200 -r 5 -p /var/run/resource-agents/send_arp-172.30.105.201 eth0 172.30.105.201 auto not_used not_used
/usr/lib/ocf/resource.d//heartbeat/IPaddr(IPaddr_172.30.105.201)[3370]:	2018/03/01_23:20:11 INFO:  Success
/usr/lib/ocf/resource.d//heartbeat/Filesystem(Filesystem_/dev/drbd0)[3502]:	2018/03/01_23:20:11 INFO:  Running OK
Mar 01 23:20:20 master heartbeat: [3036]: info: Local Resource acquisition completed. (none)
Mar 01 23:20:20 master heartbeat: [3036]: info: local resource transition completed.
```

```ruby
[root@master ~]# chkconfig mysqld off
[root@master ~]# chkconfig heartbeat off
[root@master ~]# chkconfig drbd off 

```


```ruby
[root@master ~]# ip a | grep eth
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    link/ether 00:0c:29:54:7a:ef brd ff:ff:ff:ff:ff:ff
    inet 172.30.105.121/24 brd 172.30.105.255 scope global eth0
    inet 172.30.105.201/24 brd 172.30.105.255 scope global secondary eth0
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    link/ether 00:0c:29:54:7a:f9 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.121/24 brd 192.168.0.255 scope global eth1
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    link/ether 00:0c:29:54:7a:03 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.121/24 brd 10.0.0.255 scope global eth2

```
```ruby
[root@master ~]# cat /proc/drbd
version: 8.4.9-1 (api:1/proto:86-101)
GIT-hash: 9976da086367a2476503ef7f6b13d4567327a280 build by mockbuild@Build64R6, 2016-12-13 18:38:15
 0: cs:Connected ro:Primary/Secondary ds:UpToDate/UpToDate C r-----
    ns:10482024 nr:0 dw:340300 dr:20965505 al:111 bm:0 lo:0 pe:0 ua:0 ap:0 ep:1 wo:f oos:0

```
```ruby
[root@master ~]# df -h
Filesystem                     Size  Used Avail Use% Mounted on
/dev/mapper/vg_mysql1-lv_root   26G  2.6G   23G  11% /
tmpfs                          733M     0  733M   0% /dev/shm
/dev/sda1                      477M   51M  401M  12% /boot
/dev/drbd0                     9.8G   53M  9.2G   1% /data

```
```ruby
[root@master ~]# service mysqld status
 SUCCESS! MySQL running (2665)

[root@backup ~]# service mysqld status
 ERROR! MySQL is not running, but lock file (/var/lock/subsys/mysql) exists
```




### 4、FAQ:
#### 4.1 双方互认对方为Unknown状态：
```ruby
[root@master ~]# cat /proc/drbd
version: 8.4.9-1 (api:1/proto:86-101)
GIT-hash: 9976da086367a2476503ef7f6b13d4567327a280 build by mockbuild@Build64R6, 2016-12-13 18:38:15
 0: cs:StandAlone ro:Primary/Unknown ds:UpToDate/DUnknown   r-----
    ns:10822324 nr:0 dw:340300 dr:10483481 al:111 bm:0 lo:0 pe:0 ua:0 ap:0 ep:1 wo:f oos:0


[root@backup ~]# cat /proc/drbd
version: 8.4.9-1 (api:1/proto:86-101)
GIT-hash: 9976da086367a2476503ef7f6b13d4567327a280 build by mockbuild@Build64R6, 2016-12-13 18:38:15
 0: cs:StandAlone ro:Secondary/Unknown ds:UpToDate/DUnknown   r-----
    ns:0 nr:10822324 dw:10822572 dr:3186 al:11 bm:0 lo:0 pe:0 ua:0 ap:0 ep:1 wo:f oos:116

```

```ruby
[root@backup ~]# /etc/init.d/drbd stop
Stopping all DRBD resources: .


[root@backup ~]# drbdadm create-md mysql0
......

Do you want to proceed?
[need to type 'yes' to confirm] yes

initializing activity log
NOT initializing bitmap
Writing meta data...
New drbd meta data block successfully created.


[root@backup ~]# /etc/init.d/drbd start
Starting DRBD resources: [
     create res: mysql0
   prepare disk: mysql0
    adjust disk: mysql0
     adjust net: mysql0
]
..........


[root@backup ~]# cat /proc/drbd
version: 8.4.9-1 (api:1/proto:86-101)
GIT-hash: 9976da086367a2476503ef7f6b13d4567327a280 build by mockbuild@Build64R6, 2016-12-13 18:38:15
 0: cs:WFConnection ro:Secondary/Unknown ds:Inconsistent/DUnknown C r----s
    ns:0 nr:0 dw:0 dr:0 al:8 bm:0 lo:0 pe:0 ua:0 ap:0 ep:1 wo:f oos:10482024

```

```ruby
[root@master ~]# service drbd reload  
Reloading DRBD configuration: .

[root@master ~]# cat /proc/drbd
version: 8.4.9-1 (api:1/proto:86-101)
GIT-hash: 9976da086367a2476503ef7f6b13d4567327a280 build by mockbuild@Build64R6, 2016-12-13 18:38:15
 0: cs:SyncSource ro:Primary/Secondary ds:UpToDate/Inconsistent C r-----
    ns:2764 nr:0 dw:340300 dr:10487777 al:111 bm:0 lo:0 pe:1 ua:12 ap:0 ep:1 wo:f oos:10479920
	[>....................] sync'ed:  0.1% (10232/10236)M
	finish: 2:25:33 speed: 1,052 (1,052) K/sec

```



切换报错：

```ruby
ResourceManager(default)[16525]:	2018/03/05_13:41:27 info: Running /etc/init.d/mysqld  start
ResourceManager(default)[16525]:	2018/03/05_13:41:28 ERROR: Return code 1 from /etc/init.d/mysqld
ResourceManager(default)[16525]:	2018/03/05_13:41:28 CRIT: Giving up resources due to failure of mysqld
ResourceManager(default)[16525]:	2018/03/05_13:41:28 info: Releasing resource group: master IPaddr::172.30.105.201/24/eth0 drbddisk::mysql0 Filesystem::/dev/drbd0::/data::ext4 mysqld
ResourceManager(default)[16525]:	2018/03/05_13:41:28 info: Running /etc/init.d/mysqld  stop
ResourceManager(default)[16525]:	2018/03/05_13:41:28 info: Running /etc/ha.d/resource.d/Filesystem /dev/drbd0 /data ext4 stop
Filesystem(Filesystem_/dev/drbd0)[17436]:	2018/03/05_13:41:28 INFO: Running stop for /dev/drbd0 on /data
Filesystem(Filesystem_/dev/drbd0)[17436]:	2018/03/05_13:41:28 INFO: Trying to unmount /data
Filesystem(Filesystem_/dev/drbd0)[17436]:	2018/03/05_13:41:28 INFO: unmounted /data successfully

```