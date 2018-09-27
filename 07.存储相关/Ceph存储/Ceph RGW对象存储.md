## Ceph RGW(RADOS Gateway)-Ceph网关对象存储
&emsp;&emsp;Ceph RGW(即RADOS Gateway)是Ceph对象存储网关服务，是一个基于librados构建的对象存储接口，为应用程序提供Ceph存储集群的RESTful网关。 对象存储适用于图片、视频等各类文件的上传下载，可以设置相应的访问权限。目前Ceph RGW对象存储支持两个接口：
- S3兼容：提供对象存储功能，其接口与Amazon S3 RESTful API的大部分兼容
- Swift兼容：提供对象存储功能，其接口与OpenStack Swift API的大部分兼容。       


&emsp;&emsp;Ceph对象存储使用Ceph对象网关守护进程（radosgw），它是一个用于与Ceph存储集群交互的HTTP服务器。 由于它提供了与OpenStack Swift和Amazon S3兼容的接口，因此Ceph对象网关具有自己的用户管理。

![](https://github.com/ZongYuWang/image/blob/master/Ceph/ceph-rgw1.png)

安装一台CentOS7的主机，并且安装Ceph到这台机器,作为rgw-node的节点,此处,我们使用cnode1节点充当，由于已经安装好Ceph软件,所以省略

### 1、安装ceph-radosgw和Nginx

```ruby
[root@cnode1 ~]# yum install ceph-radosgw
```
```ruby
[root@cnode1 ~]# vim /etc/yum.repos.d/nginx.repo
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/x86_64/
gpgcheck=0
enabled=1

[root@cnode1 ~]# yum install -y nginx
[root@cnode1 ~]# systemctl start nginx
[root@cnode1 ~]# systemctl enable nginx
```

### 2、创建RGW用户和keyring
#### 2.1 在Ceph集群的管理节点上创建对象网关密钥并赋予权限

```ruby
[root@cnode1 ~]# ceph-authtool --create-keyring /etc/ceph/ceph.client.radosgw.keyring
creating /etc/ceph/ceph.client.radosgw.keyring
[root@cnode1 ~]# chmod +r /etc/ceph/ceph.client.radosgw.keyring

```
`rgw实例名是gateway`
`这个实例名可以修改,ceph的对象网关支持多网关部署：即负载均衡+多网关`


#### 2.2 为对象网关rgw实例生成网关用户和密码:
```ruby
[root@cnode1 ~]# ceph-authtool /etc/ceph/ceph.client.radosgw.keyring -n client.radosgw.gateway --gen-key

```
#### 2.3 为用户添加访问权限：
```ruby
[root@cnode1 ~]# ceph-authtool -n client.radosgw.gateway --cap osd 'allow rwx' --cap mon 'allow rwx' /etc/ceph/ceph.client.radosgw.keyring

```

#### 2.4 导入keyring到集群中：
```ruby
root@cnode1 ~]# sudo ceph -k /etc/ceph/ceph.client.admin.keyring auth add client.radosgw.gateway -i /etc/ceph/ceph.client.radosgw.keyring
added key for client.radosgw.gateway

```

##### FAQ (如果前面添加过再次添加就会失败)：
```ruby
[root@cnode1 ceph]# ceph -k /etc/ceph/ceph.client.admin.keyring auth add client.radosgw.gateway -i /etc/ceph/ceph.client.radosgw.keyring
Error EEXIST: entity client.radosgw.gateway exists but key does not match

[root@cnode1 ceph]# ceph auth del client.radosgw.gateway
updated
[root@cnode1 ceph]# ceph -k /etc/ceph/ceph.client.admin.keyring auth add client.radosgw.gateway -i /etc/ceph/ceph.client.radosgw.keyring
added key for client.radosgw.gateway

```


#### 2.5 把集群配置文件推送到集群主机：
```ruby
[root@cnode1 ~]# scp /etc/ceph/ceph.client.radosgw.keyring cnode2:/etc/ceph/
[root@cnode1 ~]# chmod 777 /etc/ceph/ceph.client.radosgw.keyring 
```



### 3、创建资源池：
```ruby
[root@cnode1 ~]# ceph osd pool create .rgw.root 8 8
[root@cnode1 ~]# ceph osd pool create default.rgw.control 8 8
[root@cnode1 ~]# ceph osd pool create default.rgw.meta 8 8
[root@cnode1 ~]# ceph osd pool create default.rgw.log 8 8
[root@cnode1 ~]# ceph osd pool create default.rgw.buckets.index 8 8
[root@cnode1 ~]# ceph osd pool create default.rgw.buckets.data 8 8

```
#### 3.1 列出pool信息确认全部成功创建:
```ruby
[root@cnode1 ~]# rados lspools
.rgw.root
default.rgw.control
default.rgw.meta
default.rgw.log
default.rgw.buckets.index
default.rgw.buckets.data

// {zone-name}.pool-name
```

##### FAQ(删除资源池失败)：
```ruby
[root@cnode1 ~]# ceph osd pool delete newtv_pool newtv_pool  --yes-i-really-really-mean-it
Error EPERM: pool deletion is disabled; you must first set the mon_allow_pool_delete config option to true before you can destroy a pool
[root@cnode1 ~]# ceph -n mon.0 --show-config | grep mon_allow_pool_delete
mon_allow_pool_delete = true

[root@cnode1 ~]# ceph tell mon.\* injectargs '--mon-allow-pool-delete=true'
// 不用重启
```

### 4、RGW配置
#### 4.1 在rgw-node1中的ceph.conf中添加如下内容：
```ruby
[root@client ~]# vim /etc/ceph/ceph.conf

......
[client.radosgw.gateway]
rgw frontends = civetweb port=8080
host = cnode1
keyring = /etc/ceph/ceph.client.radosgw.keyring
log file = /var/log/radosgw/client.radosgw.gateway.log
rgw print continue=false
rgw content length_compat=true

// 上面安装了nginx，80端口已经被占用
```

#### 4.2 启动rgw：
```ruby
[root@client ~]# mkdir /var/log/radosgw
[root@client ~]# chown ceph:ceph /var/log/radosgw
[root@client ~]# cp /usr/lib/systemd/system/ceph-radosgw@.service /usr/lib/systemd/system/ceph-radosgw@radosgw.gateway.service 
[root@client ~]# systemctl start ceph-radosgw@radosgw.gateway
[root@client ~]# systemctl status ceph-radosgw@radosgw.gateway
● ceph-radosgw@radosgw.gateway.service - Ceph rados gateway
   Loaded: loaded (/usr/lib/systemd/system/ceph-radosgw@radosgw.gateway.service; disabled; vendor preset: disabled)
   Active: active (running) since Tue 2018-08-14 23:39:26 EDT; 1s ago
 Main PID: 8554 (radosgw)
   CGroup: /system.slice/system-ceph\x2dradosgw.slice/ceph-radosgw@radosgw.gateway.service
           └─8554 /usr/bin/radosgw -f --cluster ceph --name client.radosgw.gateway --setuser ceph --setgroup ceph

Aug 14 23:39:26 client systemd[1]: ceph-radosgw@radosgw.gateway.service holdoff time over, scheduling restart.
Aug 14 23:39:26 client systemd[1]: Started Ceph rados gateway.
Aug 14 23:39:26 client systemd[1]: Starting Ceph rados gateway...

# tail -f /var/log/radosgw/client.radosgw.gateway.log
```

#### 4.4 测试rgw：

```ruby
# curl cnode1:8080
<?xml version="1.0" encoding="UTF-8"?><ListAllMyBucketsResult xmlns="http://s3.amazonaws.com/doc/2006-03-01/"><Owner><ID>anonymous</ID><DisplayName></DisplayName></Owner><Buckets></Buckets></ListAllMyBucketsResult>

// 输出这个表示rgw就正常了
```
#### 4.5 集群状态观察：
```ruby
[root@cnode1 ~]# ceph -s
  cluster:
    id:     a0b2ee0c-562c-43ca-86a1-d64fb84b8ce0
    health: HEALTH_WARN
            application not enabled on 3 pool(s)
            clock skew detected on mon.cnode1
 
  services:
    mon: 3 daemons, quorum cnode2,cnode3,cnode1
    mgr: newtv-109(active), standbys: newtv-108, newtv-112
    mds: cephfs-1/1/1 up  {0=0=up:active}
    osd: 6 osds: 6 up, 6 in
    rgw: 1 daemon active

......
```

### 5、使用radosgw-admin管理RGW
#### 5.1 为S3连接创建一个RADOSGW用户
radosgw-admin是RGW服务的命令行管理工具。我们已经知道了RGW兼容绝大部分Amazon S3 API，下面我们先使用radosgw-admin来创建一个S3用户：
```ruby
[root@cnode1 ~]# radosgw-admin user create --uid="testuser" --display-name="First User"
{
    "user_id": "testuser",
    "display_name": "First User",
    "email": "",
    "suspended": 0,
    "max_buckets": 1000,
    "auid": 0,
    "subusers": [],
    "keys": [
        {
            "user": "testuser",
            "access_key": "CA567HO4YMWXO8BE01BP",
            "secret_key": "tRq25kvxEP27TCsErxY66h6Bdb9bzoPBVNhvmr9C"
        }
    ],
    "swift_keys": [],
    "caps": [],
    "op_mask": "read, write, delete",
    "default_placement": "",
    "placement_tags": [],
    "bucket_quota": {
        "enabled": false,
        "check_on_raw": false,
        "max_size": -1,
        "max_size_kb": 0,
        "max_objects": -1
    },
    "user_quota": {
        "enabled": false,
        "check_on_raw": false,
        "max_size": -1,
        "max_size_kb": 0,
        "max_objects": -1
    },
    "temp_url_keys": [],
    "type": "rgw"
}

```
`注意上面的命令输出中的access_key和secret_key`

```ruby
"user": "testuser",
"access_key": "CA567HO4YMWXO8BE01BP",
"secret_key": "tRq25kvxEP27TCsErxY66h6Bdb9bzoPBVNhvmr9C"

```
使用S3 API需要使用access_key和secret_key。access_key用于标识客户端身份；     
secret_key作为私钥保存在客户端服务器，不会在网络中传输，通常用于作为计算请求签名的密钥;     
使用access_key进行身份识别，使用secret_key进行签名，完成客户端的接入、认证和授权

删除用户：
```ruby
[root@cnode1 ~]# radosgw-admin  user rm --uid="testuser"
```

### 6、使用Admin Ops REST接口管理RGW:
Admin OPERATIONS是RGW提供的一套REST接口，可以用来管理S3用户、Bucket、配额等信息

#### 6.1 允许admin读写users信息：

```ruby
[root@cnode1 ~]# radosgw-admin caps add --uid="testuser" --caps="users=*"
```

#### 6.2 允许admin读写所有的usage信息：    

```ruby
[root@cnode1 ~]# radosgw-admin caps add --uid="testuser" --caps="usage=read,write"

```