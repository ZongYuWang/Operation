## Ceph管理块设备-客户端连接Ceph块设备


### 1、客户端环境部署：

#### 1.1 客户端安装Ceph：
```ruby
[root@client ~]# yum install centos-release-ceph-luminous.noarch -y
[root@client ~]# yum install ceph ceph-radosgw -y

```
#### 1.2 集群节点和Client节点的无秘钥登录：
```ruby
[root@client ~]# ssh-keygen -t rsa -P ''

// 配置看集群部署部分
```

#### 1.3 集群的配置文件拷贝至Client：
```ruby
[root@cnode1 ~]# scp /etc/ceph/* client:/etc/ceph/
ceph.client.admin.keyring                                                                                                  100%  161    11.8KB/s   00
ceph.conf                                                                                                                  100%  583   111.1KB/s   00
rbdmap  
```

### 2、存储池(pool)管理:

#### 2.1 创建pool：
```ruby
[root@client ~]# sudo ceph osd pool create newtv_pool 128 128 replicated
pool 'newtv_pool' created

```
##### 重命名pool：
```ruby
ceph osd pool rename {current-pool-name} {new-pool-name}
```

#### 2.2 初始化存储池：
```ruby
[root@client ~]# sudo rbd pool init newtv_pool
```

#### 2.3 查看pool的详细信息：
```ruby
[root@client ~]# ceph osd pool ls detail
pool 1 'newtv_pool' replicated size 3 min_size 2 crush_rule 0 object_hash rjenkins pg_num 128 pgp_num 128 last_change 34 flags hashpspool stripe_width 0 application rbd
	removed_snaps [1~5]

```
#### 2.4 设置并获取副本数:
```ruby
// 设置副本数
[root@client ~]# ceph osd pool set pool-name size num

```
```ruby
// 查看副本数
[root@client ~]# ceph osd pool get newtv_pool size
size: 3
// 默认是3个
```

#### 2.5 查看存储池列表:

```ruby
[root@client ~]# ceph osd lspools
1 newtv_pool,

```

### 3、创建块设备镜像:

```ruby
[root@client ~]# rbd create --size 10240 newtv_image -p newtv_pool
// 10240 表示10G

[root@client ~]# rdb rm newtv_pool/newtv_image
```

```ruby
[root@client ~]# ceph status
...... 
  data:
    pools:   1 pools, 128 pgs
    objects: 5 objects, 709 bytes
    usage:   6787 MB used, 113 GB / 119 GB avail
    pgs:     128 active+clean

```
#### 3.1 查看块设备镜像
```ruby
[root@client ~]# rbd info newtv_pool/newtv_image
rbd image 'newtv_image':
	size 10240 MB in 2560 objects
	order 22 (4096 kB objects)
	block_name_prefix: rbd_data.ac58643c9869
	format: 2
	features: layering, exclusive-lock, object-map, fast-diff, deep-flatten
	flags: 
	create_timestamp: Sat Aug 11 11:52:07 2018

```

#### 3.2 创建快照：
```ruby
[root@client ~]# rbd snap create newtv_pool/newtv_image@mysnap1

```
#### 3.3 删除快照：
```ruby
[root@client ~]# rbd snap rm newtv_pool/newtv_image@mysnap1
Removing snap: 100% complete...done.
```

### 4、将块设备映射到系统内核
```ruby
[root@client ~]# rbd map newtv_pool/newtv_image
rbd: sysfs write failed
RBD image feature set mismatch. Try disabling features unsupported by the kernel with "rbd feature disable".
In some cases useful info is found in syslog - try "dmesg | tail".
rbd: map failed: (6) No such device or address

// 表示当前系统不支持feature，禁用当前系统内核不支持的feature
```
```ruby
[root@client ~]# rbd feature disable newtv_pool/newtv_image exclusive-lock, object-map, fast-diff, deep-flatten
```
重新映射：
```ruby
[root@client ~]# rbd map newtv_pool/newtv_image
/dev/rbd0
```

### 5、格式化块设备镜像
```ruby
[root@client ~]# mkfs.ext4 /dev/rbd/newtv_pool/newtv_image
mke2fs 1.42.9 (28-Dec-2013)
Discarding device blocks: done                            
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=1024 blocks, Stripe width=1024 blocks
655360 inodes, 2621440 blocks
131072 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=2151677952
80 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done 

```
### 6、挂载文件系统
```ruby
[root@client ~]# mkdir /mnt/ceph-block-device
[root@client ~]# chmod 777  /mnt/ceph-block-device
[root@client ~]# mount /dev/rbd/newtv_pool/newtv_image /mnt/ceph-block-device
[root@client ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root   47G  1.3G   46G   3% /
devtmpfs                 486M     0  486M   0% /dev
tmpfs                    497M     0  497M   0% /dev/shm
tmpfs                    497M  6.6M  490M   2% /run
tmpfs                    497M     0  497M   0% /sys/fs/cgroup
/dev/vda1               1014M  125M  890M  13% /boot
tmpfs                    100M     0  100M   0% /run/user/0
/dev/rbd0                9.8G   37M  9.2G   1% /mnt/ceph-block-device
```

### 7、测试
`块存储两个客户端是不能共享的`
```ruby
[root@client ceph-block-device]# touch block_test
[root@client ceph-block-device]# ls
block_test  lost+found
[root@client ceph-block-device]# vim block_test 
this is a block test file
[root@client ceph-block-device]# cat block_test 
this is a block test file

```

### 8、查看：
```ruby
# for i in `ceph osd pool ls`;do echo -n $i" ";ceph osd pool get  $i pg_num;done;
newtv_pool pg_num: 128

```
```ruby
# ceph pg ls-by-pool vms   -o pool_pg_stat.log
```