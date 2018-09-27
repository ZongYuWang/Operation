## CentOS7部署Ceph-MGR

[官方文档](http://docs.ceph.com/docs/master/mgr/)

### 1、手动部署MGR服务

#### 1.1 新建用户：
创建用户newtv用于MGR监控:

```ruby
[root@cnode1 ~]# ceph auth get-or-create mgr.newtv-112 mon 'allow *' osd 'allow *' mds 'allow *'
[mgr.newtv-112]
	key = AQA+gWxbiD/4GRAApJbCZ2fQhjjO3PyPi3JAUA==

// newtv-112是用户名，112是cnode1的IP地址最后一位，便于区分
```

##### 修改用户权限：
假如用户已经存在, 可用下面方法进行权限修改：
```ruby
[root@cnode1 ~]# ceph auth caps mgr.newtv-112 mon 'allow *' osd 'allow *' mds 'allow *'

```

##### 删除用户：
```ruby
[root@cnode1 ~]# ceph auth del mgr.newtv
updated
[root@cnode1 ~]# systemctl restart ceph-mgr@newtv
// 不重启使用ceph-s还是会显示active状态
```

#### 1.2 导出秘钥
```ruby
[root@cnode1 ~]# mkdir /var/lib/ceph/mgr/ceph-newtv-112
[root@cnode1 ~]# ceph auth get mgr.newtv-112 -o  /var/lib/ceph/mgr/ceph-newtv-112/keyring
exported keyring for mgr.newtv-112
```

#### 1.3 启动mgr:
```ruby
[root@cnode1 ~]# ceph-mgr -i newtv-112

```
```ruby
[root@cnode1 ~]# ss  -tunlp|grep ceph-mgr|grep LISTEN
tcp    LISTEN     0      128    172.25.101.112:6808                  *:*                   users:(("ceph-mgr",pid=25910,fd=25))
tcp    LISTEN     0      128    172.25.101.112:6809                  *:*                   users:(("ceph-mgr",pid=26106,fd=25))

```
#### 1.4 查看监控状态：

```ruby
......
  services:
    mon: 3 daemons, quorum cnode2,cnode3,cnode1
    mgr: newtv-112(active)
    osd: 6 osds: 6 up, 6 in
......
```
### 2、在另外两个节点上部署Ceph-MGR
```ruby
[root@cnode2 ~]# ceph auth get-or-create mgr.newtv-108 mon 'allow *' osd 'allow *' mds 'allow *'
[mgr.newtv-108]
	key = AQDMhmxbryidNhAADNQjXXAM1M2WDQW0yAsSMQ==

[root@cnode2 ~]# mkdir /var/lib/ceph/mgr/ceph-newtv-108

[root@cnode2 ~]# ceph auth get mgr.newtv-108 -o  /var/lib/ceph/mgr/ceph-newtv-108/keyring
exported keyring for mgr.newtv-108

[root@cnode2 ~]# ceph-mgr -i newtv-108

```
#### 2.1 查看总状态：
```ruby
[root@cnode1 ~]# ceph -s
  cluster:
    id:     a0b2ee0c-562c-43ca-86a1-d64fb84b8ce0
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum cnode2,cnode3,cnode1
    mgr: newtv-112(active), standbys: newtv-108, newtv-109
    osd: 6 osds: 6 up, 6 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 bytes
    usage:   6786 MB used, 113 GB / 119 GB avail
    pgs:     

```

### 3、MGR高可用

- 默认情况下，创建的第一个mgr由监视器激活(active状态)，其他的mgr将是standby状态，在mgr进程中不需要仲裁产生active状态的mgr；
- 如果active的mgr在规定时间内未向监视器发送信号(默认30秒)，则它将被standby的mgr代替；
- 手动故障转移，可以使用ceph mgr fail <mgr name>将ceph-mgr守护程序标记为失败；

### 4、使用模块(dashboard)

#### 4.1 查看哪个模块可用，哪个模块当前是开启的：
```ruby
[root@cnode1 ~]# ceph mgr module ls 
{
    "enabled_modules": [
        "balancer",
        "restful",
        "status"
    ],
    "disabled_modules": [
        "dashboard",
        "influx",
        "localpool",
        "prometheus",
        "selftest",
        "zabbix"
    ]
}

```
#### 4.2 开启和关闭模块：
```ruby
[root@cnode1 ~]# ceph mgr module enable dashboard

// 语法：ceph mgr module enable <module> 
        ceph mgr module disable <module>
```
`dashboard模块只会在active状态的mgr上显示`
#### 4.3 手动配置：

```ruby
[root@cnode1 ~]# ceph config-key set mgr/dashboard/openstack/server_addr 172.25.101.112
set mgr/dashboard/openstack/server_addr

[root@cnode1 ~]# ceph config-key set mgr/dashboard/openstack/server_port 7000
set mgr/dashboard/openstack/server_port


// 执行完上述命令之后不会在相应的目录下创建目录或文件
// 这个端口设置成其他端口号不生效，最后还是访问7000
```
```ruby
// 验证
[root@cnode1 ~]# ceph config-key dump
{
    "mgr/dashboard/openstack/server_addr": "172.25.101.112",
    "mgr/dashboard/openstack/server_port": "7000"
}

```
```ruby
// 删除
[root@cnode1 ~]# ceph config-key del mgr/dashboard/openstack/server_addr
[root@cnode1 ~]# ceph config-key del mgr/dashboard/openstack/server_port
```
#### 4.4 修改配置文件：
```ruby
[root@cnode1 ~]# vim /etc/ceph/ceph.conf
[mgr]
mgr modules = dashboard
```

#### 4.5 服务管理：
```ruby
[root@cnode1 ~]# ll /usr/lib/systemd/system/ | grep ceph-mgr
-rw-r--r--. 1 root root  543 Apr 23 12:18 ceph-mgr@.service
-rw-r--r--. 1 root root  181 Apr 23 12:18 ceph-mgr.target

```
```ruby
[root@cnode1 ~]# cp /usr/lib/systemd/system/ceph-mgr@.service  /usr/lib/systemd/system/ceph-mgr@newtv-112.service
```
`这里 mgr 服务命名需要跟 ceph -s 中 mgr 定义的命名匹配`
#### 4.6 启动服务：
```ruby
[root@cnode1 ~]# systemctl status  ceph-mgr@newtv-112.service
● ceph-mgr@newtv-112.service - Ceph cluster manager daemon
   Loaded: loaded (/usr/lib/systemd/system/ceph-mgr@newtv-112.service; disabled; vendor preset: disabled)
   Active: inactive (dead)

[root@cnode1 ~]# systemctl restart ceph-mgr@newtv-112.service
[root@cnode1 ~]# systemctl status  ceph-mgr@newtv-112.service
● ceph-mgr@newtv-112.service - Ceph cluster manager daemon
   Loaded: loaded (/usr/lib/systemd/system/ceph-mgr@newtv-112.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2018-08-09 15:02:55 EDT; 2s ago
 Main PID: 27844 (ceph-mgr)
   CGroup: /system.slice/system-ceph\x2dmgr.slice/ceph-mgr@newtv-112.service
           └─27844 /usr/bin/ceph-mgr -f --cluster ceph --id newtv-112 --setuser ceph --setgroup ceph

Aug 09 15:02:55 cnode1 systemd[1]: Started Ceph cluster manager daemon.
Aug 09 15:02:55 cnode1 systemd[1]: Starting Ceph cluster manager daemon...
Aug 09 15:02:55 cnode1 ceph-mgr[27844]: [09/Aug/2018:15:02:55] ENGINE Bus STARTING
Aug 09 15:02:55 cnode1 ceph-mgr[27844]: [09/Aug/2018:15:02:55] ENGINE Started monitor thread '_TimeoutMonitor'.
Aug 09 15:02:55 cnode1 ceph-mgr[27844]: [09/Aug/2018:15:02:55] ENGINE Serving on :::7000
Aug 09 15:02:55 cnode1 ceph-mgr[27844]: [09/Aug/2018:15:02:55] ENGINE Bus STARTED

```
```ruby
[root@cnode1 ~]# ss -tunlp|grep ceph-mgr|grep LISTEN
tcp    LISTEN     0      128    172.25.101.112:6808                  *:*                   users:(("ceph-mgr",pid=25910,fd=25))
tcp    LISTEN     0      5        :::7000                 :::*                   users:(("ceph-mgr",pid=33297,fd=24))

```
#### 4.7 访问dashboard：
```ruby
http://172.25.101.108:7000
```

![](https://github.com/ZongYuWang/image/blob/master/Ceph/Ceph-mgr1.png)