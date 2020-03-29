## 管理CephFS设备

[官网-添加MDS](http://docs.ceph.com/docs/master/install/manual-deployment/#adding-osds)      
[官网-创建文件系统](http://docs.ceph.com/docs/master/cephfs/createfs/)      
[官网-内核驱动挂载](http://docs.ceph.com/docs/master/cephfs/kernel/)      
[官网-FUSE挂载](http://docs.ceph.com/docs/master/cephfs/fuse/)      

### 1、添加MDS
#### 1.1 创建mds数据目录：
```ruby
[root@cnode1 ~]# mkdir -p /var/lib/ceph/mds/ceph-0
```
#### 1.2 创建一个秘钥keyring：
```ruby
[root@cnode1 ~]# ceph-authtool --create-keyring /var/lib/ceph/mds/ceph-0/keyring --gen-key -n mds.0
creating /var/lib/ceph/mds/ceph-0/keyring
```
#### 1.3 导入秘钥环并设置权限：
```ruby
[root@cnode1 ~]# ceph auth add mds.0 osd "allow rwx" mds "allow" mon "allow profile mds" -i /var/lib/ceph/mds/ceph-0/keyring
added key for mds.0
```
#### 1.4 添加到配置文件中：
```ruby
[root@cnode1 ~]# vim /etc/ceph/ceph.conf 
[mds.0]
host = 172.25.101.112
```
#### 1.5 启动一个服务：
```ruby
[root@cnode1 ~]# ceph-mds --cluster ceph -i 0 -m 172.25.101.112:6789
2018-08-12 07:35:44.235858 7f925bbb6200 -1 deprecation warning: MDS id 'mds.0' is invalid and will be forbidden in a future version.  MDS names may not start with a numeric digit.
starting mds.0 at -
```

```ruby
[root@cnode1 ~]# ceph mds stat
, 1 up:standby
```


### 2、创建CEPH文件系统

#### 2.1 创建池：
Ceph文件系统至少需要两个RADOS池，一个用于数据，一个用于元数据
- 对元数据池使用更高的复制级别，因为此池中的任何数据丢失都可能导致整个文件系统无法访问
- 使用较低延迟的存储（如SSD）作为元数据池，因为这将直接影响客户端上文件系统操作的观察延迟

```ruby
[root@cnode1 ~]# ceph osd pool create cephfs_data 128
pool 'cephfs_data' created
// 语法：ceph osd pool create cephfs_data <pg_num>

[root@cnode1 ~]# ceph osd pool create cephfs_metadata 128
pool 'cephfs_metadata' created
// 语法：ceph osd pool create cephfs_metadata <pg_num>
```
#### 2.2 创建文件系统：
```ruby
[root@cnode1 ~]# ceph fs new cephfs cephfs_metadata cephfs_data
new fs with metadata pool 3 and data pool 2
// 语法：ceph fs new <fs_name> <metadata> <data>

[root@cnode1 ~]# ceph fs ls
name: cephfs, metadata pool: cephfs_metadata, data pools: [cephfs_data ]
```

##### 创建文件系统后，MDS将能够进入活动状态:
```ruby
[root@cnode1 ~]# ceph mds stat
cephfs-1/1/1 up  {0=0=up:active}

```
#### 2.3 状态查看：
```ruby
[root@cnode1 ~]# ceph mds dump
dumped fsmap epoch 5
fs_name	cephfs
epoch	5
flags	c
created	2018-08-12 08:18:11.139818
modified	2018-08-12 08:18:11.139819
tableserver	0
root	0
session_timeout	60
session_autoclose	300
max_file_size	1099511627776
last_failure	0
last_failure_osd_epoch	0
compat	compat={},rocompat={},incompat={1=base v0.20,2=client writeable ranges,3=default file layouts on dirs,4=dir inode in separate object,5=mds uses versioned encoding,6=dirfrag is stored in omap,8=no anchor table,9=file layout v2}
max_mds	1
in	0
up	{0=44243}
failed	
damaged	
stopped	
data_pools	[2]
metadata_pool	3
inline_data	disabled
balancer	
standby_count_wanted	0
44243:	172.25.101.112:6809/503880056 '0' mds.0.4 up:active seq 639

```

```ruby
[root@cnode1 ~]# ceph -s
  cluster:
    id:     a0b2ee0c-562c-43ca-86a1-d64fb84b8ce0
    health: HEALTH_WARN
            clock skew detected on mon.cnode1
 
  services:
    mon: 3 daemons, quorum cnode2,cnode3,cnode1
    mgr: newtv-108(active), standbys: newtv-109, newtv-112
    mds: cephfs-1/1/1 up  {0=0=up:active}
    osd: 6 osds: 6 up, 6 in
 
  data:
    pools:   3 pools, 384 pgs
    objects: 74 objects, 136 MB
    usage:   7186 MB used, 112 GB / 119 GB avail
    pgs:     384 active+clean

```

### 3、客户端映射CephFS：

#### 3.1 内核驱动挂载(三种方式)：
```ruby
方式1：
[root@client ~]# cat ceph.client.admin.keyring 
[client.admin]
	key = AQBXi2pbktb3JxAAH04sFhnoAQLpDRfY/E/QIQ==
	auid = 0
	caps mds = "allow *"
	caps mgr = "allow *"
	caps mon = "allow *"
	caps osd = "allow *"

[root@client ~]# mount -t ceph 172.25.101.112:6789:/ /mnt/cephfs -o name=admin,secret=AQBXi2pbktb3JxAAH04sFhnoAQLpDRfY/E/QIQ==
```

```ruby
方式2：
[root@client ceph]# mount -t cephfs 172.25.101.112:6789:/ /mnt/cephfs -o name=admin,secretfile=/etc/ceph/ceph.client.admin.keyring 
secret is not valid base64: Invalid argument.
adding ceph secret key to kernel failed: Invalid argument.
failed to parse ceph_options

// 不好用
```
```ruby
方式3：
[root@client ~]# mkdir /mnt/cephfs
[root@client ~]# mount -t ceph 172.25.101.112:6789:/ /mnt/cephfs
mount error 22 = Invalid argument
// 不好用
```

#### 3.2 查看挂载：
```ruby
[root@client ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
......
172.25.101.112:6789:/    120G  7.1G  113G   6% /mnt/cephfs

```

### 4、FUSE(file system in user space)挂载：
```ruby
[root@client ~]# yum install -y ceph-fuse

```
#### 4.1 从monitor主机拷贝配置文件到客户端主机到/etc/ceph目录下
```ruby
[root@client ~]# mkdir -p /etc/ceph
[root@client ~]# scp {user}@{server-machine}:/etc/ceph/ceph.conf /etc/ceph/ceph.conf

// 将mon节点的ceph配置文件拷贝到客户端
```

#### 4.2 从monitor拷贝秘钥环文件到客户端主机/etc/ceph目录下
```ruby
sudo scp {user}@{server-machine}:/etc/ceph/ceph.keyring /etc/ceph/ceph.keyring
```
`确保客户端这些文件的权限是644`

#### 4.3 启动服务：
```ruby
[root@client ~]# mkdir /home/cephfs
[root@client ~]# ceph-fuse -m 172.25.101.112:6789 /home/cephfs/
ceph-fuse[7394]: starting ceph client
2018-08-14 01:52:04.642152 7f8b0d0fd0c0 -1 init, newargv = 0x561e23bbacc0 newargc=9
ceph-fuse[7394]: starting fuse

```

```ruby
[root@client ~]# ps -ef | grep ceph-fuse
root        7394       1  0 01:52 pts/0    00:00:00 ceph-fuse -m 172.25.101.112:6789 /home/cephfs/
root        7544    7182  0 02:28 pts/0    00:00:00 grep --color=auto ceph-fuse
```
#### 4.4 查看挂载：
```ruby
[root@client ~]# df -hT
......
ceph-fuse               fuse.ceph-fuse   36G     0   36G   0% /home/cephfs
```

### 5、CephFS共享测试
#### 5.1 客户端1：
```ruby
[root@client1 ~]# mkdir -p /mnt/cephfs
[root@client1 ~]# mount -t ceph 172.25.101.112:6789:/ /mnt/cephfs -o name=admin,secret=AQBXi2pbktb3JxAAH04sFhnoAQLpDRfY/E/QIQ==

[root@client1 ~]# cd /mnt/cephfs/
[root@client1 cephfs]# vim wangzy0927
[root@client1 cephfs]# cat wangzy0927 
this is a test file by wangzy in 0927

```

#### 5.2 客户端2：
```ruby
[root@client2 ~]# mkdir /mnt/cephfs2
[root@client2 ~]# mount -t ceph 172.25.101.112:6789:/ /mnt/cephfs2 -o name=admin,secret=AQBXi2pbktb3JxAAH04sFhnoAQLpDRfY/E/QIQ==

[root@client2 ~]# cd /mnt/cephfs2/
[root@client2 cephfs2]# ls
cephfs  wangzy0927
[root@client2 cephfs2]# cat wangzy0927 
this is a test file by wangzy in 0927

```