## heartbeat

### 1、Heartbeat工作原理：
- 通过修改heartbeat软件的配置文件，可以指定哪一台heartbeat服务器作为主服务器，则另外一台自动成为热备服务器，然后在热备服务器上配置heartbeat守护进程来监听来自主服务器的心跳信息，如果热备服务器在指定时间内未监听到来自主服务器的心跳，就会启动故障转移程序，并取得主服务器上的相关资源服务的所有权，接替主服务器继续不间断的提供服务，从而达到资源及服务高可用的目的；
- 以上描述的事heartbeat的主备模式，heartbeat还支持主主模式，即两台服务器互为主备，这时它们之间会相互发送报文来告诉对象自己当前的状态，如果在指定的时间内未接受到对方发来的心跳报文，那么，一方就会认为对象失效或者宕机了，这时每个运行正常的主机都会启动自身的资源接管模块来接管运行在对方主机上的资源或者服务，继续为用户提供服务。一般情况下，可以较好的实现一台主机故障后，企业业务能都不间断的持续运行，所谓的业务不间断，在故障转移期间也是需要切换时间的，heartbeat的切换时间一般是在5-20秒左右；

- 另外：heartbeat和keepalived服务一样，heartbeat高可用是服务器级别的，不是服务级别的，如果是MySQL或者是Apache服务宕了，那么不会启动切换，必须是Server宕机才可以，可以通过服务故障把heartbeat服务停掉；

##### 切换的条件：
- 服务器宕机；
- Heartbeat服务本身故障；
- 心跳线连接故障

#### 1.1 Heartbeat脑裂：
由于两台高可用服务器对之间在指定时间内，无法相互检测到对方心跳而各自启动故障转移功能，取得了资源及服务的所有权，而此时的两台高可用服务器对都活着在正常运行，这样就会导致同一个IP或服务在两端同时启动而发生冲突的严重问题，最严重的是两台主机占用通一个VIP地址，当用户写入数据时可能会分别写入到两端，这样可能会导致服务器两端的数据不一致或造成数据丢失，这种情况就被成为脑裂；
##### 导致脑裂发生的多种原因：
- 高可用服务器对之间心跳线链路故障，导致无法正常通信；
- 高可用服务器对上开启了防火墙阻挡了心跳消息传输；
- 高可用服务器对上心跳网卡地址等信息配置不正确，导致发送心跳失败；
- 其他原因如心跳方式不同，心跳广博冲突、软件BUG等；

##### 实际生产环境中，我们可以从以下几个方面来防止脑裂的发生：
- 同时使用串行电缆和以外网电缆连接，同时用两条心跳线路；
- 检测到脑裂时强行关闭一个心跳节点(这个功能需特殊设备支持)，相当于程序上备节点发现心跳线故障，发送关机命令到主节点；
- 做好对脑裂的监控报警，在问题发生时人为第一时间介入仲裁，降低损失；
- 启用磁盘锁，正在服务一方锁住共享磁盘，脑裂发生时，让对方完全抢不走共享磁盘资源，但是使用磁盘锁也会有问题，如果占用共享磁盘的一方不主动解锁，另一方就永远得不到共享磁盘；
- 报警后，不直接自动服务器接管，而是由人为人员控制接管；
- 报警报在服务器接管之前，给人员处理留足够时间，如1分钟内报警了，但是服务器此时没有接管，而是5分钟接管，接管的时间较长，数据不会丢失，导致用户无法写数据；

### 2、Heartbeat消息类型：
#### 2.1 心跳消息：
心跳消息为约150字节的数据包，可能为单播、广播或多播的方式，控制心跳频率及出现故障要等待多久进行故障转换；
#### 2.2 集群转换消息：
ip-request和ip-request-resp
当主服务器恢复在线状态时，通过ip-request消息要求备机释放主服务器失败备服务器取得的资源，然后备份服务器关闭释放主服务器失败时取得的资源及服务；    
备服务器释放主服务器失败时取得的资源及服务后，就会通过ip-request-resp消息通知主服务器它不再拥有该资源及服务，主服务器收到来自备节点的ip-request-resp消息通知后，启动失败时释放的资源及服务，并开始提供正常的访问服务；

#### 2.3 Heartbeat IP地址接管和故障转移：
Heartbeat是通过IP地址接管和ARP广播进行故障转移的；
ARP广播：在主服务器故障时，备用节点接管资源后，会立即强制更新所有客户端本地的ARP表(即清除客户端本地缓存的失败服务器的VIP地址和MAC地址的解析记录)确保客户端和新的主服务器对话；      
`客户端：是和heartbeat高可用服务器对在同一网络中的客户机，不是最终的互联网用户`    

### 3、配置Heartbeat：
#### 3.1 配置VIP(这个步骤不用配置)：
```ruby
手动添加VIP：
[root@localhost ~]# ip addr add 172.30.105.200/24 broadcast 172.30.105.0 dev ensxx (辅助IP)

// keepalive和Heartbeat3采用上面的命令添加VIP

手动删除VIP：
[root@localhost ~]# ip addr del 172.30.105.200/24 broadcast 172.30.105.0 dev ensxx (辅助IP)

// heartbeat版本起，不在使用别名，而是使用辅助IP提供服务

```
#### 3.2 Heartbeat的配置文件：
heartbeat的默认配置文件目录为/etc/ha.d，heartbeat常用的配置文件有三个，分别为ha.cf、authkey、haresource
`/etc/ha.d/resource.d/，如果以后自己开发程序，就放在这个地方，然后在haresource文件里直接调用`

配置文件 | 作用 | 备注 |  
|- | :- | :- | 
ha.cf | heartbeat参数配置文件| 在这里配置heartbeat的一些基本参数 | 
authkey | heartbeat认证文件 | 高可用服务器对之间根据对端的authkey，对端进行认证|
haresource | heartbeat资源配置文件 | 如配置IP资源及脚本程序等|

### 3、CentOS7搭建并配置Heartbeat的双主模式：
##### Master1服务器规划：
```ruby
[root@localhost ~]# hostnamectl set-hostname master1
```
服务器角色 | 网卡名称 | IP地址 | 备注 | 
|:-: | :-: | :-: | :- | 
Master | ens38| 172.30.105.12 | 管理IP，用于WAN数据转发|
| | ens33| 172.30.105.101 | 用于服务期间心跳连接(直连)|
VIP地址 | | 172.30.105.201 | 用于提供应用程序A挂载服务|

```ruby
[root@master1 ~]# ip a
......
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    link/ether 00:0c:29:7a:7b:da brd ff:ff:ff:ff:ff:ff
    inet 172.30.105.101/24 brd 172.30.105.255 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::1196:a820:405c:5e41/64 scope link 
       valid_lft forever preferred_lft forever
3: ens38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    link/ether 00:0c:29:7a:7b:e4 brd ff:ff:ff:ff:ff:ff
    inet 172.30.105.12/24 brd 172.30.105.255 scope global dynamic ens38
       valid_lft 81744sec preferred_lft 81744sec
    inet6 fe80::55d7:ee39:a697:cdbe/64 scope link 
       valid_lft forever preferred_lft forever

```

##### Master2服务器规划：
```ruby
[root@localhost ~]# hostnamectl set-hostname master2

```
服务器角色 | 网卡名称 | IP地址 | 备注 | 
|:-: | :-: | :-: | :- | 
Master | ens38| 172.30.105.13 | 管理IP，用于WAN数据转发|
| | ens33| 172.30.105.102 | 用于服务期间心跳连接(直连)|
VIP地址 | | 172.30.105.202 | 用于提供应用程序A挂载服务|

```ruby
[root@master2 ~]# ip a
......
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    link/ether 00:0c:29:14:93:9e brd ff:ff:ff:ff:ff:ff
    inet 172.30.105.102/24 brd 172.30.105.255 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::1984:3e61:a482:3f3c/64 scope link 
       valid_lft forever preferred_lft forever
3: ens38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    link/ether 00:0c:29:14:93:a8 brd ff:ff:ff:ff:ff:ff
    inet 172.30.105.13/24 brd 172.30.105.255 scope global dynamic ens38
       valid_lft 82963sec preferred_lft 82963sec
    inet6 fe80::25c4:cd69:e680:432e/64 scope link 
       valid_lft forever preferred_lft forever

```

#### 3.1 配置主机hosts以及服务器防火墙:
```ruby
172.30.105.12 master1
172.30.105.13 master2

// 有时候hostname获取的主机名和uname -n不一样，heartbeat必须和uname -n一样
[root@master1 ~]# uname -n
master1
```
```ruby
master1：
[root@master1 ~]# setenforce 0
[root@master1 ~]# getenforce sestatus -ve
Permissive
master2：
[root@master2 ~]# setenforce 0
[root@master2 ~]# getenforce sestatus -v
Permissive

master1：
[root@master1 ~]# systemctl stop firewalld.service
master2：
[root@master2 ~]# systemctl stop firewalld.service
```

#### 3.2 指定心跳信息从指定的网卡连通：
##### Master1：
```ruby
[root@master1 ~]# ip route add 172.30.105.102 dev ens33
[root@master1 ~]# ip route show

```
##### Master2：
```ruby
[root@master2 ~]# ip route add 172.30.105.101 dev ens33
[root@master2 ~]# ip route show
```

#### 3.3 安装heartbeat3.0软件：
##### CentOS6安装方式：
```ruby
[root@master1 ~]# wget http://mirrors.ustc.edu.cn/epel/7/x86_64/Packages/e/epel-release-7-11.noarch.rpm
[root@master1 ~]# rpm -ivh epel-release-7-11.noarch.rpm
[root@master1 ~]# rpm -qa | grep epel
epel-release-7-11.noarch
```
```ruby
[root@master1 ~]# yum install -y heartbeat -y
```

##### CentOS7安装方式：
```ruby
[root@master1 ~]# yum install -y bzip2 autoconf automake libtool glib2-devel libxml2-devel bzip2-devel libtool-ltdl-devel asciidoc libuuid-devel
```
[软件包下载地址](http://linux-ha.org/wiki/Download)
```ruby
[root@master1 ~]# tar xf Cluster_Glue_1.0.12.bz2 
[root@master1 ~]# cd Reusable-Cluster-Components-glue--0a7add1d9996/
[root@master1 Reusable-Cluster-Components-glue--0a7add1d9996]# ./autogen.sh
[root@master1 Reusable-Cluster-Components-glue--0a7add1d9996]# ./configure --prefix=/usr/local/heartbeat/
[root@master1 Reusable-Cluster-Components-glue--0a7add1d9996]# make && make install
```

```ruby
[root@master1 ~]# tar xf resource-agents-3.9.6.tar.gz
[root@master1 ~]# cd resource-agents-3.9.6

[root@master1 resource-agents-3.9.6]# ./autogen.sh 
[root@master1 resource-agents-3.9.6]# export CFLAGS="$CFLAGS -I/usr/local/heartbeat/include -L/usr/local/heartbeat/lib"
// CFLAGS表示用于C编译器(gcc)的选项 -L表示搜索库目录-l或者-I表示包含头文件目录;
[root@master1 resource-agents-3.9.6]# ./configure --prefix=/usr/local/heartbeat/ 

[root@master1 resource-agents-3.9.6]# vim /etc/ld.so.conf.d/heartbeat.conf
/usr/local/heartbeat/lib
[root@master1 resource-agents-3.9.6]# ldconfig
[root@master1 resource-agents-3.9.6]# make && make install  
```

```ruby
[root@master1 ~]# tar xf Heartbeat_3.0.6.bz2 
[root@master1 ~]# cd Heartbeat-3-0-958e11be8686/
[root@master1 Heartbeat-3-0-958e11be8686]# ./bootstrap
[root@master1 Heartbeat-3-0-958e11be8686]# export CFLAGS="$CFLAGS -I/usr/local/heartbeat/include -L/usr/local/heartbeat/lib"
[root@master1 Heartbeat-3-0-958e11be8686]# ./configure --prefix=/usr/local/heartbeat/ 
[root@master1 Heartbeat-3-0-958e11be8686]# vim /usr/local/heartbeat/include/heartbeat/glue_config.h 
/*#define HA_HBCONF_DIR "/usr/local/heartbeat/etc/ha.d/" */
// 此行注释掉，否则下面的make会报错
[root@master2 Heartbeat-3-0-958e11be8686]# make && make install 
```
```ruby
[root@master1 ~]# cp  /usr/local/heartbeat/share/doc/heartbeat/ha.cf  /usr/local/heartbeat/etc/ha.d
[root@master1 ~]# cp /usr/local/heartbeat/share/doc/heartbeat/authkeys /usr/local/heartbeat/etc/ha.d
[root@master1 ~]# cp  /usr/local/heartbeat/share/doc/heartbeat/haresources /usr/local/heartbeat/etc/ha.d

```
```ruby
[root@master1 ha.d]# pwd
/usr/local/heartbeat/etc/ha.d
[root@master1 ha.d]# ll
total 40
-rw-r--r--. 1 root root   645 Feb  7 22:00 authkeys
-rw-r--r--. 1 root root 10502 Feb  7 22:00 ha.cf
-rwxr-xr-x. 1 root root   745 Feb  7 21:59 harc
-rw-r--r--. 1 root root  5905 Feb  7 22:01 haresources
drwxr-xr-x. 2 root root   101 Feb  7 21:59 rc.d
-rw-r--r--. 1 root root   692 Feb  7 21:59 README.config
drwxr-xr-x. 2 root root  4096 Feb  7 21:59 resource.d
-rw-r--r--. 1 root root  2112 Feb  7 21:49 shellfuncs
```
#### 3.4 Heartbeat配置文件详解：
##### ha.cf配置文件：
```ruby
debugfile /var/log/ha-debug
logfile /var/log/ha-log
logfacility     local1
// 以上三行为日志的配置

keepalive 2
deadtime 30
warntime 10
initdead 60
// 以上四行为一些基础参数

udpport 694
ucast ens33 172.30.105.102
// IP地址是对端的IP地址
// ucast是使用单播发送heartbeat心跳信息
// mcast是组播的形式，格式为： [dev][mcast group][port][ttl][loop]

auto_failback on

node    master1
node    master2
ping 172.30.105.254
respawn hacluster /usr/local/heartbeat/libexec/heartbeat/ipfail

```
##### ha.cf文件详细说明：
|参数 | 说明 |  
| - | :- | 
|debugfile /var/log/ha-debug | heartbeat的调试日志存放位置|
|logfile /var/log/ha-log | heartbeat的日志存放位置 |
|logfacility local1| 在syslog服务中配置通过local1设置接收日志|
|keepalive 2 | 指定心跳间隔时间为2秒(即每3秒在eth1上发一次广播) |
|deadtime 30 | 指定若备用节点在30秒内没有收到主节点的心跳信号，则立即接管主节点的服务资源|
|warntime 10 | 指定心跳延迟的时间为10秒，当10秒钟内备份节点不能接收到主节点的心跳信号时，就会往日志写入一个警告日志，但此时不会切换服务 |
|initdead 60 | 指定在heartbeat首次运行后，需要等待120秒才启动主服务器的任何资源，该选项用于解决这种情况产生的时间间隔，取值至少为deadtime的两倍，单机启动时会遇到VIP绑定很慢，为正常现象，是因为该值设置的时间上 |
|udpport 694 | 设置通信使用的端口，694为默认使用的端口号 |
|ucast ens33 172.30.105.102 | 使用单播，IP地址是对端的IP地址 |
|auto_failback on | 用来定义当主节点恢复后，是否将服务自动切回 |
|node  master1 | 主节点的主机名，可以通过uname -n 查看|
|node  master2 | 备用节点主机名，可以通过uname -n 查看 |
|ping 172.30.105.254 | IP地址是网关地址 |
|respawn hacluster /usr/local/heartbeat/libexec/heartbeat/ipfail | 该进程用于检测和处理网络故障，需要配合ping语句指定的ping node来检测网络连接。如果你的系统是64bit，请注意该文件的路径 |

##### authkeys配置文件 ：
```ruby
[root@master1 ha.d]# chmod 600 authkeys
[root@master1 ha.d]# vim authkeys
auth 2
2 sha1 UMXyx2of32qUezRFevhLsYUW0u6Hz2QnIu8z00ta

// 上面的码是随机生成的，自己随意输入
// Authentication file.  Must be mode 600
// Available methods: crc sha1, md5.  Crc doesn't need/want a key
// sha1 is believed to be the "best", md5 next best(sha1是最好的选择，md5是其次的选择)
// 默认的配置使用的crc方法，这是不加密的，不够安全
```

##### haresource配置文件：
```ruby
master1 IPaddr::172.30.105.201/24/ens38 nginx
master2 IPaddr::172.30.105.202/24/ens38


// 全面的写法：master1 IPaddr::172.30.105.201/24/ens38 drbddisk::data Filesystem::/dev/drbd0::/data::ext3 nginx
// master1/2 为主机名，IPaddr为heartbeat配置IP的默认脚本，其后的IP等都是脚本的参数；
// 172.30.105.201/24/ens38为集群对外服务的VIP，24是子网掩码，ens38为IP绑定的实际物理网卡，为heartbeat提供对外服务的通信接口；
// 这里相当于执行/usr/local/heartbeat/etc/ha.d/resource.d/IPaddr 172.30.105.202/24/ens38 stop/start

// drbddisk::data 启动drbd data资源，这里相当于执行/usr/local/heartbeat/etc/ha.d/resource.d/drbddisk data stop/start
// Filesystem::/dev/drbd0::/data::ext3  drbd分区挂载到/data目录，这里相当于执行/usr/local/heartbeat/etc/ha.d/resource.d/Filesystem /dev/drbd0 /data ext3 stop/start
// nginx 表示会通过heartbeat启动nginx服务，nginx的安装见后续文档，启动脚本见后面

```
`VIP其实是由IPaddr脚本自动启动的，和heartbeat没什么关系`
```ruby
配置完成后可以通过ha-log查看:（截取选段）
2018/02/09_01:01:28 info: Running /usr/local/heartbeat/etc/ha.d/resource.d/IPaddr 172.30.105.202/24/ens38 start
```
##### /resource.d/目录：
`/usr/local/heartbeat/etc/ha.d/resource.d/为heartbeat的默认脚本文件目录`
- 脚本路径要放入/etc/init.d/或/usr/local/heartbeat/etc/ha.d/resource.d/里；
- 脚本执行需要以/etc/init.d/xxx stop/start方式；
- 脚本具有可执行权限
- /etc/init.d/name名字和master1 IPaddr::172.30.105.201/24/ens38 name名字一致；
- 可以自己编写脚本放到该目录下；

如果脚本文件在/etc/init.d/和/usr/local/heartbeat/etc/ha.d/resource.d/里都存在，那么会以后者目录中存放的脚本优先启动
```ruby
[root@master1 ~]# tail -f /var/log/ha-log
2018/02/09_02:05:59 info: Running /usr/local/heartbeat/etc/ha.d/resource.d/nginx  start

```

#### 3.5 将配置文件拷贝至master2服务器：
```ruby
[root@master1 ~]# scp /usr/local/heartbeat/etc/ha.d/ha.cf master2:/usr/local/heartbeat/etc/ha.d/
[root@master1 ~]# scp /usr/local/heartbeat/etc/ha.d/authkeys master2:/usr/local/heartbeat/etc/ha.d/
[root@master1 ~]# scp /usr/local/heartbeat/etc/ha.d/haresources master2:/usr/local/heartbeat/etc/ha.d/

```
##### master2需要修改的配置文件：
```ruby
检查拷贝过去的文件的权限：
[root@master2 ha.d]# chmod 600 /usr/local/heartbeat/etc/ha.d/authkeys
```
```ruby
ha.cf配置文件:
ucast ens33 172.30.105.101
// 互相指向对端的IP地址

```

```ruby
haresources配置文件:
master1 IPaddr::172.30.105.201/24/ens38
master2 IPaddr::172.30.105.202/24/ens38 nginx
```

### 4、启动heartbeat并查看状态：
##### Master1：
```ruby
[root@master1 ha.d]# systemctl status  heartbeat
● heartbeat.service - Heartbeat High Availability Cluster Communication and Membership
   Loaded: loaded (/usr/lib/systemd/system/heartbeat.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2018-02-08 03:19:48 EST; 2s ago
  Process: 38534 ExecStop=/usr/local/heartbeat/libexec/heartbeat/heartbeat -k (code=exited, status=0/SUCCESS)
 Main PID: 38569 (heartbeat)
   CGroup: /system.slice/heartbeat.service
           ├─38569 heartbeat: master control process                  
           ├─38571 heartbeat: FIFO reader                             
           ├─38572 heartbeat: write: ucast ens33                      
           ├─38573 heartbeat: read: ucast ens33                       
           ├─38574 heartbeat: write: ping 172.30.105.254              
           └─38575 heartbeat: read: ping 172.30.105.254               

Feb 08 03:19:48 master1 heartbeat[38569]: [38569]: info: glib: ucast: set SO_REUSEPORT(w)
Feb 08 03:19:48 master1 heartbeat[38569]: [38569]: info: glib: ucast: set SO_REUSEADDR
Feb 08 03:19:48 master1 heartbeat[38569]: [38569]: info: glib: ucast: bound receive socket to device: ens33
Feb 08 03:19:48 master1 heartbeat[38569]: [38569]: info: glib: ucast: set SO_REUSEPORT
Feb 08 03:19:48 master1 heartbeat[38569]: [38569]: info: glib: ucast: started on port 694 interface ens33 to 172.30.105.101
Feb 08 03:19:48 master1 heartbeat[38569]: [38569]: info: glib: ping heartbeat started.
Feb 08 03:19:48 master1 heartbeat[38569]: [38569]: info: Local status now set to: 'up'
Feb 08 03:19:48 master1 heartbeat[38569]: [38569]: info: Link master1:ens33 up.
Feb 08 03:19:49 master1 heartbeat[38569]: [38569]: info: Link 172.30.105.254:172.30.105.254 up.
Feb 08 03:19:49 master1 heartbeat[38569]: [38569]: info: Status update for node 172.30.105.254: status ping

```
##### Master2：
```ruby
[root@master2 ha.d]# systemctl start heartbeat
[root@master2 ha.d]# systemctl status heartbeat
● heartbeat.service - Heartbeat High Availability Cluster Communication and Membership
   Loaded: loaded (/usr/lib/systemd/system/heartbeat.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2018-02-08 03:08:10 EST; 3s ago
  Process: 39895 ExecStop=/usr/local/heartbeat/libexec/heartbeat/heartbeat -k (code=exited, status=0/SUCCESS)
 Main PID: 40345 (heartbeat)
   CGroup: /system.slice/heartbeat.service
           ├─40345 heartbeat: master control process
           ├─40347 heartbeat: FIFO reader
           ├─40348 heartbeat: write: ucast ens33
           ├─40349 heartbeat: read: ucast ens33
           ├─40350 heartbeat: write: ping 172.30.105.254
           └─40351 heartbeat: read: ping 172.30.105.254

Feb 08 03:08:10 master2 heartbeat[40345]: [40345]: info: glib: ucast: set SO_REUSEPORT(w)
Feb 08 03:08:10 master2 heartbeat[40345]: [40345]: info: glib: ucast: set SO_REUSEADDR
Feb 08 03:08:10 master2 heartbeat[40345]: [40345]: info: glib: ucast: bound receive socket to device: ens33
Feb 08 03:08:10 master2 heartbeat[40345]: [40345]: info: glib: ucast: set SO_REUSEPORT
Feb 08 03:08:10 master2 heartbeat[40345]: [40345]: info: glib: ucast: started on port 694 interface ens33 to 172.30.105.102
Feb 08 03:08:10 master2 heartbeat[40345]: [40345]: info: glib: ping heartbeat started.
Feb 08 03:08:10 master2 heartbeat[40345]: [40345]: info: Local status now set to: 'up'
Feb 08 03:08:10 master2 heartbeat[40345]: [40345]: info: Link 172.30.105.254:172.30.105.254 up.
Feb 08 03:08:10 master2 heartbeat[40345]: [40345]: info: Status update for node 172.30.105.254: status ping
Feb 08 03:08:10 master2 heartbeat[40345]: [40345]: info: Link master2:ens33 up.

```

```ruby
[root@master2 ~]# ps -ef | grep heartbeat
root      40345      1  0 03:08 ?        00:00:01 heartbeat: master control process
root      40347  40345  0 03:08 ?        00:00:00 heartbeat: FIFO reader
root      40348  40345  0 03:08 ?        00:00:00 heartbeat: write: ucast ens33
root      40349  40345  0 03:08 ?        00:00:00 heartbeat: read: ucast ens33
root      40350  40345  0 03:08 ?        00:00:00 heartbeat: write: ping 172.30.105.254
root      40351  40345  0 03:08 ?        00:00:00 heartbeat: read: ping 172.30.105.254
haclust+  40403  40345  0 03:09 ?        00:00:00 /usr/local/heartbeat/libexec/heartbeat/ipfail
root      42110  38393  0 03:30 pts/2    00:00:00 grep --color=auto heartbeat

```

### 5、Heartbeat+Nginx服务：
#### 5.1 安装Nginx服务：
为了结合后面的nginx启动脚本，nginx需要编译安装，指定安装路径为--prefix=/usr/local/nginx
```ruby
[root@master1 ~]# echo master1 > /usr/local/nginx/html/index.html
[root@master2 ~]# echo master2 > /usr/local/nginx/html/index.html

```
```ruby
开机不能自启动Nginx,交由heartbeat启动：
[root@master1 ~]# systemctl disable httpd.service
[root@master2 ~]# systemctl disable httpd.service
```

#### 5.2 haresources配置文件中添加服务名称：
```ruby
/usr/local/heartbeat/etc/ha.d/haresources：

master1：
master1 IPaddr::172.30.105.201/24/ens38 nginx
master2 IPaddr::172.30.105.202/24/ens38

master2：
master1 IPaddr::172.30.105.201/24/ens38 
master2 IPaddr::172.30.105.202/24/ens38 nginx

// 采用的双主模式，当master1宕机了，可以飘到master2上，当master2宕机了可以飘到master1上
// 在haresources配置文件中也可以不指定服务(Nginx),这样就只是VIP的展示及VIP的飘动
// 如果加上了服务名称(Nginx)则服务不需要提前启动，heartbeat会自行启动服务(根据/etc/init.d/里面的名称)
```
`haresources配置文件最后的服务名称一定要和/etc/init.d/nginx的名称相对应`

#### 5.3 Nginx的启动脚本：
```ruby
vim /etc/init.d/nginx

#!/bin/sh
# chkconfig: 2345 80 90  
# description: Start and Stop nginx


#PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

DESC="nginx daemon"
NAME=nginx
DAEMON=/usr/local/nginx/sbin/$NAME
CONFIGFILE=/usr/local/nginx/conf/$NAME.conf
PIDFILE=/usr/local/nginx/logs/$NAME.pid
SCRIPTNAME=/etc/init.d/$NAME

set -e
[ -x "$DAEMON" ] || exit 0

do_start() {
$DAEMON -c $CONFIGFILE || echo -n "nginx already running"
}

do_stop() {
kill -INT `cat $PIDFILE` || echo -n "nginx not running"
}


do_reload() {
kill -HUP `cat $PIDFILE` || echo -n "nginx can't reload"
}

case "$1" in
start)
echo -n "Starting $DESC: $NAME"
do_start
echo "."
;;
stop)
echo -n "Stopping $DESC: $NAME"
do_stop
echo "."
;;
reload|graceful)
echo -n "Reloading $DESC configuration..."
do_reload
echo "."
;;
restart)
echo -n "Restarting $DESC: $NAME"
do_stop
do_start
echo "."
;;
*)
echo "Usage: $SCRIPTNAME {start|stop|reload|restart}" >&2
exit 3
;;
esac

exit 0
```
```ruby
注册系统服务：
[root@master1 ~]# chkconfig --add nginx

设置权限：
[root@master1 ~]# chmod a+wrx /etc/init.d/nginx
```
#### 5.4 heartbeat日志说明：
`heartbeat结合nginx的启动日志`
##### Master1：
```ruby
[root@master1 ~]# tail -f /var/log/ha-debug

[root@master1 ~]# tail -f /var/log/ha-log
Feb 09 01:05:18 master1 heartbeat: [65190]: info: glib: ucast: bound send socket to device: ens33
Feb 09 01:05:18 master1 heartbeat: [65190]: info: glib: ucast: set SO_REUSEPORT(w)
Feb 09 01:05:18 master1 heartbeat: [65190]: info: glib: ucast: set SO_REUSEADDR
Feb 09 01:05:18 master1 heartbeat: [65190]: info: glib: ucast: bound receive socket to device: ens33
Feb 09 01:05:18 master1 heartbeat: [65190]: info: glib: ucast: set SO_REUSEPORT
Feb 09 01:05:18 master1 heartbeat: [65190]: info: glib: ucast: started on port 694 interface ens33 to 172.30.105.102
Feb 09 01:05:18 master1 heartbeat: [65190]: info: glib: ping heartbeat started.
Feb 09 01:05:18 master1 heartbeat: [65190]: info: Local status now set to: 'up'
Feb 09 01:05:18 master1 heartbeat: [65190]: info: Link 172.30.105.254:172.30.105.254 up.
Feb 09 01:05:18 master1 heartbeat: [65190]: info: Status update for node 172.30.105.254: status ping



Feb 09 01:05:31 master1 heartbeat: [65190]: info: Link master2:ens33 up.
Feb 09 01:05:31 master1 heartbeat: [65190]: info: Status update for node master2: status up
harc(default)[65199]:	2018/02/09_01:05:32 info: Running /usr/local/heartbeat/etc/ha.d/rc.d/status status
Feb 09 01:05:32 master1 heartbeat: [65190]: info: Comm_now_up(): updating status to active
Feb 09 01:05:32 master1 heartbeat: [65190]: info: Local status now set to: 'active'
Feb 09 01:05:32 master1 heartbeat: [65190]: info: Starting child client "/usr/local/heartbeat/libexec/heartbeat/ipfail  " (1000,1000)
Feb 09 01:05:32 master1 heartbeat: [65217]: info: Starting "/usr/local/heartbeat/libexec/heartbeat/ipfail  " as uid 1000  gid 1000 (pid 65217)
Feb 09 01:05:32 master1 heartbeat: [65190]: info: Status update for node master2: status active
harc(default)[65220]:	2018/02/09_01:05:33 info: Running /usr/local/heartbeat/etc/ha.d/rc.d/status status
Feb 09 01:05:35 master1 ipfail: [65217]: info: Status update: Node master2 now has status active
Feb 09 01:05:36 master1 ipfail: [65217]: info: Asking other side for ping node count.
Feb 09 01:05:38 master1 ipfail: [65217]: info: No giveup timer to abort.
Feb 09 01:05:43 master1 heartbeat: [65190]: info: remote resource transition completed.
Feb 09 01:05:43 master1 heartbeat: [65190]: info: remote resource transition completed.
Feb 09 01:05:43 master1 heartbeat: [65190]: info: Initial resource acquisition complete (T_RESOURCES(us))
/usr/lib/ocf/resource.d//heartbeat/IPaddr(IPaddr_172.30.105.201)[65273]:	2018/02/09_01:05:43 INFO:  Resource is stopped
Feb 09 01:05:43 master1 heartbeat: [65237]: info: Local Resource acquisition completed.
harc(default)[65325]:	2018/02/09_01:05:43 info: Running /usr/local/heartbeat/etc/ha.d/rc.d/ip-request-resp ip-request-resp
ip-request-resp(default)[65325]:	2018/02/09_01:05:43 received ip-request-resp IPaddr::172.30.105.201/24/ens38 OK yes
ResourceManager(default)[65348]:	2018/02/09_01:05:43 info: Acquiring resource group: master1 IPaddr::172.30.105.201/24/ens38 nginx
/usr/lib/ocf/resource.d//heartbeat/IPaddr(IPaddr_172.30.105.201)[65376]:	2018/02/09_01:05:43 INFO:  Resource is stopped
ResourceManager(default)[65348]:	2018/02/09_01:05:43 info: Running /usr/local/heartbeat/etc/ha.d/resource.d/IPaddr 172.30.105.201/24/ens38 start
IPaddr(IPaddr_172.30.105.201)[65470]:	2018/02/09_01:05:43 INFO: Using calculated netmask for 172.30.105.201: 255.255.255.0
IPaddr(IPaddr_172.30.105.201)[65470]:	2018/02/09_01:05:43 INFO: eval ifconfig ens38:0 172.30.105.201 netmask 255.255.255.0 broadcast 172.30.105.255
/usr/lib/ocf/resource.d//heartbeat/IPaddr(IPaddr_172.30.105.201)[65444]:	2018/02/09_01:05:43 INFO:  Success
ResourceManager(default)[65348]:	2018/02/09_01:05:43 info: Running /etc/rc.d/init.d/nginx  start

```
#### 5.5 网卡信息：
##### Master1：
```ruby
[root@master1 ~]# ip a
......
3: ens38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    link/ether 00:0c:29:7a:7b:e4 brd ff:ff:ff:ff:ff:ff
    inet 172.30.105.12/24 brd 172.30.105.255 scope global dynamic ens38
       valid_lft 71678sec preferred_lft 71678sec
    inet 172.30.105.201/24 brd 172.30.105.255 scope global secondary ens38:0
       valid_lft forever preferred_lft forever
    inet6 fe80::55d7:ee39:a697:cdbe/64 scope link 
       valid_lft forever preferred_lft forever

```
##### Master2：
```ruby
[root@master2 ~]# ip a
......
3: ens38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    link/ether 00:0c:29:14:93:a8 brd ff:ff:ff:ff:ff:ff
    inet 172.30.105.13/24 brd 172.30.105.255 scope global dynamic ens38
       valid_lft 72775sec preferred_lft 72775sec
    inet 172.30.105.202/24 brd 172.30.105.255 scope global secondary ens38:0
       valid_lft forever preferred_lft forever
    inet6 fe80::25c4:cd69:e680:432e/64 scope link 
       valid_lft forever preferred_lft forever

```
### 6、有关heartbeat调用资源在生产场景的应用：
#### 6.1 在实际工作中有两种常见的办法实现高可用：
- heartbeat可以仅控制VIP资源的飘移，不负责服务资源的启动及停止(适合WEB服务)；
- heartbeat即控制VIP资源的飘移，同时又控制服务资源启动及停止，如果VIP正常，服务宕了，这种情况下heartbeat不会做高可用切换，可以写一个脚本判断服务运行状态，如果有问题，则停止heartbeat(服务和IP要切换就都切换)；
#### 6.2 HA高可用Nginx案例结论：
- 日志的重要性(ha-log、ha-debug);
- Nginx服务的启动可以提前启动正常，即不交给heartbeat启动也可以；
- 这个模型在生产环境中用的较少，如数据库使用此模型就会出现问题，heartbeat+drbd+mysql可实现数据库高可用配置；

#### 6.3 heartbeat服务生产环境下维护要点：
- 方法1：在修改前执行systemctl stop heartbeat或/usr/local/heartbeat/share/heartbeat/hb_standby(此命令优先选择)，把本机业务推到备节点上，当确认备节点工作正常后，开始修改本机的配置，修改好之后可以再执行systemctl start heartbeat把资源服务接管回来，接管回来之后可以检查一下所有的服务及VIP是否运行正常（所有配置可以放到SVN上，更改后提交至SVN，然后和修改前的对比，正常之后推送到正式环境）；
- 方法2：先在需要更改配置的Server上手动配置VIP，然后再去修改配置文件，到服务器业务量底的时候，把手动配置的VIP去掉，然后再重启hearteat(尽量使用stop再start，少用restart)；


### 7、将本机自己变为standby状态、接管其他主机的两个脚本：

#### 7.1 执行hb_standby脚本会将本机变为standby状态：
```ruby
[root@master2 ~]# /usr/local/heartbeat/share/heartbeat/hb_standby 
Going standby [all].

```
```ruby
[root@master2 ~]# tail -f /var/log/ha-log 
ResourceManager(default)[87140]:	2018/02/09_01:33:47 info: Running /etc/rc.d/init.d/nginx  stop
ResourceManager(default)[87140]:	2018/02/09_01:33:47 info: Running /usr/local/heartbeat/etc/ha.d/resource.d/IPaddr 172.30.105.202/24/ens38 stop
IPaddr(IPaddr_172.30.105.202)[87219]:	2018/02/09_01:33:47 INFO: ifconfig ens38:0 down
/usr/lib/ocf/resource.d//heartbeat/IPaddr(IPaddr_172.30.105.202)[87193]:	2018/02/09_01:33:47 INFO:  Success
Feb 09 01:33:47 master2 heartbeat: [87038]: info: all HA resource release completed (standby).
Feb 09 01:33:47 master2 heartbeat: [86615]: info: Local standby process completed [all].
Feb 09 01:33:50 master2 heartbeat: [86615]: WARN: 1 lost packet(s) for [master1] [873:875]
Feb 09 01:33:50 master2 heartbeat: [86615]: info: remote resource transition completed.
Feb 09 01:33:50 master2 heartbeat: [86615]: info: No pkts missing from master1!
Feb 09 01:33:50 master2 heartbeat: [86615]: info: Other node completed standby takeover of all resources.

```

#### 7.2 执行hb_takeover脚本会接管另外一个主机：
```ruby
[root@master2 ~]# /usr/local/heartbeat/share/heartbeat/hb_takeover 
```
```ruby
[root@master2 ~]# ip a
......
3: ens38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    link/ether 00:0c:29:14:93:a8 brd ff:ff:ff:ff:ff:ff
    inet 172.30.105.13/24 brd 172.30.105.255 scope global dynamic ens38
       valid_lft 71076sec preferred_lft 71076sec
    inet 172.30.105.202/24 brd 172.30.105.255 scope global secondary ens38:0
       valid_lft forever preferred_lft forever
    inet 172.30.105.201/24 brd 172.30.105.255 scope global secondary ens38:1
       valid_lft forever preferred_lft forever
    inet6 fe80::25c4:cd69:e680:432e/64 scope link 
       valid_lft forever preferred_lft forever

// 把172.30.105.201从master1接管过来了
```
```ruby
[root@master2 ~]# tail -f /var/log/ha-log 
ResourceManager(default)[88444]:	2018/02/09_01:40:00 info: Running /usr/local/heartbeat/etc/ha.d/resource.d/IPaddr 172.30.105.201/24/ens38 start
IPaddr(IPaddr_172.30.105.201)[88569]:	2018/02/09_01:40:01 INFO: Using calculated netmask for 172.30.105.201: 255.255.255.0
IPaddr(IPaddr_172.30.105.201)[88569]:	2018/02/09_01:40:01 INFO: eval ifconfig ens38:1 172.30.105.201 netmask 255.255.255.0 broadcast 172.30.105.255
/usr/lib/ocf/resource.d//heartbeat/IPaddr(IPaddr_172.30.105.201)[88543]:	2018/02/09_01:40:01 INFO:  Success
ResourceManager(default)[88680]:	2018/02/09_01:40:01 info: Acquiring resource group: master2 IPaddr::172.30.105.202/24/ens38 nginx
/usr/lib/ocf/resource.d//heartbeat/IPaddr(IPaddr_172.30.105.202)[88708]:	2018/02/09_01:40:01 INFO:  Running OK
ResourceManager(default)[88680]:	2018/02/09_01:40:01 info: Running /etc/rc.d/init.d/nginx  start
Feb 09 01:40:03 master2 heartbeat: [88431]: info: all HA resource acquisition completed (standby).
Feb 09 01:40:03 master2 heartbeat: [88010]: info: Standby resource acquisition done [all].
Feb 09 01:40:03 master2 heartbeat: [88010]: info: remote resource transition completed.

```

### 8、FAQ:
```ruby
 ./configure的时候会报如下错误：
 
configure: error: in `/root/Heartbeat-3-0-958e11be8686':
configure: error: Core development headers were not found
See `config.log' for more details   

// 所以要加上export CFLAGS="$CFLAGS -I/usr/local/heartbeat/include -L/usr/local/heartbeat/lib"
```

```ruby
Feb 08 03:15:40 master1 heartbeat[38533]: heartbeat: udpport setting must precede media statementsFeb 08 03:15:40 master1 heartbeat: [38533]: ERROR: Illegal directive [ucast] in /usr/local/heartbeat/etc/ha.d/ha.cf
Feb 08 03:15:40 master1 heartbeat[38533]: Feb 08 03:15:40 master1 heartbeat: [38533]: ERROR: Illegal directive [ping] in /usr/local/heartbeat/etc/ha.d/ha.cf

解决办法：
[root@master1 ha.d]# ln -svf /usr/local/heartbeat/lib64/heartbeat/plugins/RAExec/* /usr/local/heartbeat/lib/heartbeat/plugins/RAExec/
[root@master1 ha.d]# ln -svf /usr/local/heartbeat/lib64/heartbeat/plugins/* /usr/local/heartbeat/lib/heartbeat/plugins/
```

```ruby
[root@master2 ha.d]# tail -f /var/log/ha-debug 
Feb 08 02:59:36 master2 heartbeat: [39893]: info: One or the other must be specified.
Feb 08 02:59:36 master2 heartbeat: [39893]: ERROR: Invalid group name [haclient]
Feb 08 02:59:36 master2 heartbeat: [39893]: ERROR: Bad gid list [haclient]
Feb 08 02:59:36 master2 heartbeat: [39893]: ERROR: Invalid apiauth directive [anon uid=root gid=haclient]

解决办法：
[root@master2 ~]# groupadd haclient  
[root@master2 ~]# useradd hacluster -g haclient -G haclient,root
[root@master2 ~]# chmod g+rwx /

```

```ruby
Feb 08 09:44:09 master1 ipfail: [56403]: ERROR: Cannot chdir to [/usr/local/heartbeat/var/lib/heartbeat/cores/hacluster]: Permission denied
IPaddr(IPaddr_172.30.105.201)[56914]:	2018/02/08_09:44:09 ERROR: Setup problem: couldn't find command: ifconfig
/usr/lib/ocf/resource.d//heartbeat/IPaddr(IPaddr_172.30.105.201)[56888]

解决办法：
[root@master1 ~]# yum install net-tools
[root@master1 ~]# chmod 755 /usr/local/heartbeat/var/lib/heartbeat/cores/hacluster
```











