## Ceph简单命令操作手册

### pg操作：

```ruby
[root@cnode1 ~]# ceph pg dump_stuck stale
ok

```

### 查看集群利用率：
```ruby
[root@cnode1 ~]# rados df
POOL_NAME       USED  OBJECTS CLONES COPIES MISSING_ON_PRIMARY UNFOUND DEGRADED RD_OPS RD    WR_OPS WR    
cephfs_data        20       1      0      3                  0       0        0      0     0      4  9216 
cephfs_metadata 55731      22      0     66                  0       0        0      4  4096     84 96256 
newtv_pool       136M      53      0    159                  0       0        0   1426 1286k    236  132M 

total_objects    76
total_used       7193M
total_avail      112G
total_space      119G

```


### 查询pool列表：
```ruby
[root@cnode1 ~]# ceph osd pool ls
newtv_pool
cephfs_data
cephfs_metadata
.rgw.root
default.rgw.control
```

### 查询pool的pg数量
```ruby
[root@cnode1 ~]# ceph osd pool get newtv_pool pg_num
pg_num: 128

```
### 修改pg数：
```ruby
[root@cnode1 ~]# ceph osd pool set newtv_pool pg_num 32
Error EEXIST: specified pg_num 32 <= current 128

pg_num只能增加, 不能缩小
```

### OSD的空间查看：
```
[root@cnode1 ~]# ceph osd df
ID CLASS WEIGHT  REWEIGHT SIZE   USE   AVAIL  %USE VAR  PGS 
 0   hdd 0.01949  1.00000 20469M 1220M 19248M 5.96 1.02 199 
 3   hdd 0.01949  1.00000 20469M 1176M 19292M 5.75 0.98 201 
 1   hdd 0.01949  1.00000 20469M 1192M 19276M 5.82 0.99 200 
 4   hdd 0.01949  1.00000 20469M 1208M 19260M 5.90 1.01 200 
 2   hdd 0.01949  1.00000 20469M 1192M 19276M 5.82 0.99 198 
 5   hdd 0.01949  1.00000 20469M 1204M 19264M 5.88 1.00 202 
                    TOTAL   119G 7193M   112G 5.86          
MIN/MAX VAR: 0.98/1.02  STDDEV: 0.07

// ID 0和3都在cnode1节点上，两个OSD共计使用了400个PG
```

### ceph-deploy安装
```ruby
[ceph]
name=ceph
baseurl=http://mirrors.163.com/ceph/rpm-luminous/el7/x86_64/
gpgcheck=0
[ceph-noarch]
name=cephnoarch
baseurl=http://mirrors.163.com/ceph/rpm-luminous/el7/noarch/
gpgcheck=0

# yum install ceph-deploy
```

### FAQ（application not enabled on 4 pool(s)）:
```ruby
[root@cnode1 ~]# ceph -s
  cluster:
    id:     a0b2ee0c-562c-43ca-86a1-d64fb84b8ce0
    health: HEALTH_WARN
            application not enabled on 4 pool(s)

[root@cnode1 ~]# ceph health detail
HEALTH_WARN application not enabled on 4 pool(s)
POOL_APP_NOT_ENABLED application not enabled on 4 pool(s)
    application not enabled on pool '.rgw.root'
    application not enabled on pool 'default.rgw.control'
    application not enabled on pool 'default.rgw.meta'
    application not enabled on pool 'default.rgw.log'
    use 'ceph osd pool application enable <pool-name> <app-name>', where <app-name> is 'cephfs', 'rbd', 'rgw', or freeform for custom applications.


[root@cnode1 ~]#  ceph osd pool application enable .rgw.root cephfs
[root@cnode1 ~]#  ceph osd pool application enable default.rgw.control cephfs
[root@cnode1 ~]#  ceph osd pool application enable default.rgw.meta cephfs
[root@cnode1 ~]#  ceph osd pool application enable default.rgw.log cephfs

```