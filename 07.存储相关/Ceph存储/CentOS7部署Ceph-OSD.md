## CentOS7部署Ceph-OSD

[官网文档](http://docs.ceph.com/docs/master/install/manual-deployment/#adding-osds)

### 1、手动添加OSD:

#### 1.1 为OSD生成UUID：
```ruby
[root@cnode1 ~]# uuidgen
071ac88a-6428-4bac-9c6c-aee4ca7d910d

// UUID=$(uuidgen)
```
#### 1.2 为OSD生成cephx key：
```ruby
[root@cnode1 ~]# ceph-authtool --gen-print-key
AQCa3GpbzwXlJhAAgthmcmYzu/P8sLOYJFYjlw==

// OSD_SECRET=$(ceph-authtool --gen-print-key)
```

#### 1.3 生成OSD号：
```ruby
[root@cnode1 ~]# echo "{\"cephx_secret\": \"AQCa3GpbzwXlJhAAgthmcmYzu/P8sLOYJFYjlw==\"}" | ceph osd new 071ac88a-6428-4bac-9c6c-aee4ca7d910d -i - -n client.bootstrap-osd -k /var/lib/ceph/bootstrap-osd/ceph.keyring
0

// ID=$(echo "{\"cephx_secret\": \"$OSD_SECRET\"}" | \
   ceph osd new $UUID -i - \
   -n client.bootstrap-osd -k /var/lib/ceph/bootstrap-osd/ceph.keyring)
```
#### 1.4 格式化并mount磁盘:
```ruby
[root@cnode1 ~]# mkdir /var/lib/ceph/osd/ceph-0
[root@cnode1 ~]# mkfs.xfs /dev/sdb1
[root@cnode1 ~]# mount /dev/sdb1 /var/lib/ceph/osd/ceph-0/

// mount /dev/sdb1 /var/lib/ceph/osd/ceph-$ID
```
#### 1.5 为osd创建keyring:
```ruby
[root@cnode1 ~]# ceph-authtool --create-keyring /var/lib/ceph/osd/ceph-0/keyring --name osd.0 --add-key AQCa3GpbzwXlJhAAgthmcmYzu/P8sLOYJFYjlw==
creating /var/lib/ceph/osd/ceph-0/keyring
added entity osd.0 auth auth(auid = 18446744073709551615 key=AQCa3GpbzwXlJhAAgthmcmYzu/P8sLOYJFYjlw== with 0 caps)

// ceph-authtool --create-keyring /var/lib/ceph/osd/ceph-$ID/keyring \
    --name osd.$ID --add-key $OSD_SECRET
```

#### 1.6 初始化OSD:

```ruby
[root@cnode1 ~]# ceph-osd --cluster ceph -i 0 --mkfs --osd-uuid 071ac88a-6428-4bac-9c6c-aee4ca7d910d
2018-08-08 08:14:27.316144 7fdcbe8e4d40 -1 journal FileJournal::_open: disabling aio for non-block journal.  Use journal_force_aio to force use of aio anyway
2018-08-08 08:14:27.331367 7fdcbe8e4d40 -1 journal FileJournal::_open: disabling aio for non-block journal.  Use journal_force_aio to force use of aio anyway
2018-08-08 08:14:27.331614 7fdcbe8e4d40 -1 journal do_read_entry(4096): bad header magic
2018-08-08 08:14:27.331639 7fdcbe8e4d40 -1 journal do_read_entry(4096): bad header magic
2018-08-08 08:14:27.332331 7fdcbe8e4d40 -1 read_settings error reading settings: (2) No such file or directory
2018-08-08 08:14:27.341380 7fdcbe8e4d40 -1 created object store /var/lib/ceph/osd/ceph-0 for osd.0 fsid a0b2ee0c-562c-43ca-86a1-d64fb84b8ce0

// ceph-osd -i $ID --mkfs --osd-uuid $UUID

[root@cnode1 ~]# ls /var/lib/ceph/osd/ceph-0
ceph_fsid  current  fsid  journal  keyring  magic  ready  store_version  superblock  type  whoami
```
#### 1.7 设置权限并启动OSD：
```ruby
[root@cnode1 ~]# chown -R ceph:ceph /var/lib/ceph/osd/ceph-0
[root@cnode1 ~]# systemctl start ceph-osd@0
[root@cnode1 ~]# systemctl status ceph-osd@0
● ceph-osd@0.service - Ceph object storage daemon osd.0
   Loaded: loaded (/usr/lib/systemd/system/ceph-osd@.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2018-08-08 08:15:29 EDT; 1s ago
  Process: 24720 ExecStartPre=/usr/lib/ceph/ceph-osd-prestart.sh --cluster ${CLUSTER} --id %i (code=exited, status=0/SUCCESS)
 Main PID: 24724 (ceph-osd)
   CGroup: /system.slice/system-ceph\x2dosd.slice/ceph-osd@0.service
           └─24724 /usr/bin/ceph-osd -f --cluster ceph --id 0 --setuser ceph --setgroup ceph

Aug 08 08:15:29 cnode1 systemd[1]: Starting Ceph object storage daemon osd.0...
Aug 08 08:15:29 cnode1 systemd[1]: Started Ceph object storage daemon osd.0.
Aug 08 08:15:29 cnode1 ceph-osd[24724]: 2018-08-08 08:15:29.372729 7f6e5a2d1d40 -1 Public network was set, but cluster network was not set
Aug 08 08:15:29 cnode1 ceph-osd[24724]: 2018-08-08 08:15:29.372747 7f6e5a2d1d40 -1     Using public network also for cluster network
Aug 08 08:15:29 cnode1 ceph-osd[24724]: starting osd.0 at - osd_data /var/lib/ceph/osd/ceph-0 /var/lib/ceph/osd/ceph-0/journal
Aug 08 08:15:29 cnode1 ceph-osd[24724]: 2018-08-08 08:15:29.405244 7f6e5a2d1d40 -1 journal FileJournal::_open: disabling aio for non-block...o anyway
Aug 08 08:15:29 cnode1 ceph-osd[24724]: 2018-08-08 08:15:29.405505 7f6e5a2d1d40 -1 journal do_read_entry(8192): bad header magic
Aug 08 08:15:29 cnode1 ceph-osd[24724]: 2018-08-08 08:15:29.405530 7f6e5a2d1d40 -1 journal do_read_entry(8192): bad header magic
Aug 08 08:15:29 cnode1 ceph-osd[24724]: 2018-08-08 08:15:29.421762 7f6e5a2d1d40 -1 osd.0 0 log_to_monitors {default=true}
Aug 08 08:15:30 cnode1 ceph-osd[24724]: 2018-08-08 08:15:30.532396 7f6e3d049700 -1 osd.0 0 waiting for initial osdmap
Hint: Some lines were ellipsized, use -l to show in full.

[root@cnode1 ~]# systemctl enable ceph-osd@0
```

### 2、添加完一个OSD的状态
```ruby
[root@cnode1 ~]# ceph osd tree
ID CLASS WEIGHT  TYPE NAME       STATUS REWEIGHT PRI-AFF 
-1       0.01949 root default                            
-3       0.01949     host cnode1                         
 0   hdd 0.01949         osd.0       up  1.00000 1.00000 
```

### 3、按照上述步骤手动添加另外一个OSD

#### 3.1 为OSD生成UUID：
```ruby
[root@cnode2 ~]# uuidgen
ae2b7aba-7e6d-485d-9f68-ef901a194eb1
```
#### 3.2 为OSD生成cephx key：
```ruby
[root@cnode2 ~]# ceph-authtool --gen-print-key
AQAv32pb8DNXHBAANzInMia/w+6aTyjUquTPFA==
```
#### 3.3 生成OSD号：
```
[root@cnode2 ~]# echo "{\"cephx_secret\": \"AQAv32pb8DNXHBAANzInMia/w+6aTyjUquTPFA==\"}" | ceph osd new ae2b7aba-7e6d-485d-9f68-ef901a194eb1 -i - -n client.bootstrap-osd -k /var/lib/ceph/bootstrap-osd/ceph.keyring
1

// 注意UUID和key号换成新生成的
```
#### 3.4 格式化并mount磁盘:
```ruby
[root@cnode2 ~]# mkdir /var/lib/ceph/osd/ceph-1
[root@cnode2 ~]# chown -R ceph:ceph /var/lib/ceph/osd/ceph-1
[root@cnode2 ~]# mkfs.xfs /dev/sdb1
meta-data=/dev/sdb1              isize=512    agcount=4, agsize=1310656 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=5242624, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

[root@cnode2 ~]# mount /dev/sdb1 /var/lib/ceph/osd/ceph-1/
// ceph-1中的1是OSD号
```
#### 3.5 为osd创建keyring:
```ruby
[root@cnode2 ~]# ceph-authtool --create-keyring /var/lib/ceph/osd/ceph-1/keyring --name osd.1 --add-key AQAv32pb8DNXHBAANzInMia/w+6aTyjUquTPFA==
creating /var/lib/ceph/osd/ceph-1/keyring
added entity osd.1 auth auth(auid = 18446744073709551615 key=AQAv32pb8DNXHBAANzInMia/w+6aTyjUquTPFA== with 0 caps)

// 注意替换OSD号，此命令有两处需要替换OSD号
```
#### 3.6 初始化OSD:
```ruby
[root@cnode2 ~]# ceph-osd --cluster ceph -i 1 --mkfs --osd-uuid ae2b7aba-7e6d-485d-9f68-ef901a194eb1
2018-08-08 08:19:55.355671 7f415036dd40 -1 journal FileJournal::_open: disabling aio for non-block journal.  Use journal_force_aio to force use of aio anyway
2018-08-08 08:19:55.369111 7f415036dd40 -1 journal FileJournal::_open: disabling aio for non-block journal.  Use journal_force_aio to force use of aio anyway
2018-08-08 08:19:55.369467 7f415036dd40 -1 journal do_read_entry(4096): bad header magic
2018-08-08 08:19:55.369492 7f415036dd40 -1 journal do_read_entry(4096): bad header magic
2018-08-08 08:19:55.370130 7f415036dd40 -1 read_settings error reading settings: (2) No such file or directory
2018-08-08 08:19:55.378771 7f415036dd40 -1 created object store /var/lib/ceph/osd/ceph-1 for osd.1 fsid a0b2ee0c-562c-43ca-86a1-d64fb84b8ce0

[root@cnode2 ~]# ls /var/lib/ceph/osd/ceph-1
ceph_fsid  current  fsid  journal  keyring  magic  ready  store_version  superblock  type  whoami

// -i 后面是OSD号
```
#### 3.7 设置权限并启动OSD：
```ruby
[root@cnode2 ~]# chown -R ceph:ceph /var/lib/ceph/osd/ceph-1

[root@cnode2 ~]# systemctl start ceph-osd@1
[root@cnode2 ~]# systemctl status ceph-osd@1
● ceph-osd@1.service - Ceph object storage daemon osd.1
   Loaded: loaded (/usr/lib/systemd/system/ceph-osd@.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2018-08-08 08:21:24 EDT; 1s ago
  Process: 24572 ExecStartPre=/usr/lib/ceph/ceph-osd-prestart.sh --cluster ${CLUSTER} --id %i (code=exited, status=0/SUCCESS)
 Main PID: 24577 (ceph-osd)
   CGroup: /system.slice/system-ceph\x2dosd.slice/ceph-osd@1.service
           └─24577 /usr/bin/ceph-osd -f --cluster ceph --id 1 --setuser ceph --setgroup ceph

Aug 08 08:21:24 cnode2 systemd[1]: Starting Ceph object storage daemon osd.1...
Aug 08 08:21:24 cnode2 systemd[1]: Started Ceph object storage daemon osd.1.
Aug 08 08:21:24 cnode2 ceph-osd[24577]: 2018-08-08 08:21:24.332637 7f7a93612d40 -1 Public network was set, but cluster network was not set
Aug 08 08:21:24 cnode2 ceph-osd[24577]: 2018-08-08 08:21:24.332654 7f7a93612d40 -1     Using public network also for cluster network
Aug 08 08:21:24 cnode2 ceph-osd[24577]: starting osd.1 at - osd_data /var/lib/ceph/osd/ceph-1 /var/lib/ceph/osd/ceph-1/journal
Aug 08 08:21:24 cnode2 ceph-osd[24577]: 2018-08-08 08:21:24.368470 7f7a93612d40 -1 journal FileJournal::_open: disabling aio for non-block journa...io anyway
Aug 08 08:21:24 cnode2 ceph-osd[24577]: 2018-08-08 08:21:24.368775 7f7a93612d40 -1 journal do_read_entry(8192): bad header magic
Aug 08 08:21:24 cnode2 ceph-osd[24577]: 2018-08-08 08:21:24.368798 7f7a93612d40 -1 journal do_read_entry(8192): bad header magic
Aug 08 08:21:24 cnode2 ceph-osd[24577]: 2018-08-08 08:21:24.385985 7f7a93612d40 -1 osd.1 0 log_to_monitors {default=true}
Aug 08 08:21:25 cnode2 ceph-osd[24577]: 2018-08-08 08:21:25.595465 7f7a76387700 -1 osd.1 0 waiting for initial osdmap
Hint: Some lines were ellipsized, use -l to show in full.

[root@cnode2 ~]# systemctl enable ceph-osd@1

```

### 4、删除OSD
```ruby
[root@cnode1 osd]# ceph osd crush remove osd.4
removed item id 4 name 'osd.4' from crush map

[root@cnode1 osd]# ceph auth del osd.4
updated

[root@cnode1 osd]# ceph osd down osd.4; ceph osd rm "$_"
marked down osd.4. 
removed osd.4

```

### 5、添加完全部的OSD
`注意要轮询添加`

```ruby
[root@cnode1 ~]# ceph osd tree
ID CLASS WEIGHT  TYPE NAME       STATUS REWEIGHT PRI-AFF 
-1       0.11691 root default                            
-3       0.03897     host cnode1                         
 0   hdd 0.01949         osd.0       up  1.00000 1.00000 
 3   hdd 0.01949         osd.3       up  1.00000 1.00000 
-5       0.03897     host cnode2                         
 1   hdd 0.01949         osd.1       up  1.00000 1.00000 
 4   hdd 0.01949         osd.4       up  1.00000 1.00000 
-7       0.03897     host cnode3                         
 2   hdd 0.01949         osd.2       up  1.00000 1.00000 
 5   hdd 0.01949         osd.5       up  1.00000 1.00000 

```