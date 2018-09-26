## Ceph理论概述

### 1、简介
Ceph是一个高性能、可扩容的分布式存储系统，它提供三大功能：
- 对象存储：提供RESTful接口，也提供多种编程语言绑定。兼容S3、Swift
- 块存储：由RBD提供，可以直接作为磁盘挂载，内置了容灾机制
- 文件系统：提供POSIX兼容的网络文件系统CephFS，专注于高性能、大容量存储

### 2、架构

图




#### 术语：

| 术语 | 说明 | 
| :- | :- | 
| RADOS（Reliable-Autonomic Distributed Object Store） | 可靠的、自动化的分布式对象存储是Ceph的核心之一<br>librados是RADOS提供的库，上层的RBD、RGW和CephFS都是通过librados访问RADOS的 | 
|RGW（RADOS Gateway）|Ceph的对象存储API或者RGW守护进程|
|RBD（RADOS Block Device）|指Ceph提供的基于复制性的分布式的块设备。类似于LVM中的逻辑卷，RBD只能属于一个Pool
|MDS|即Ceph元数据服务器，是CephFS服务依赖的元数据服务|
|CephFS（Ceph File System）|Ceph对外提供的文件系统服务|
|Pool|存储池是Ceph中一些对象的逻辑分组。它不是一个连续的分区，而是一个逻辑概念，类似LVM中的卷组（Volume Group）<br> 存储池分为两个类型：<br> ① Replicated 复制型，对象具有多份拷贝，确保部分OSD丢失时数据不丢失，需要更多的磁盘空间。复制份数可以动态调整，可以置为1 <br> ② Erasure-coded 纠错码型，节约空间，但是速度慢，不支持所有对象操作（例如局部写）|
|PG（Placement Group）|PG是Pool组织对象的方式，便于更好的分配数据和定位数据，Pool由若干PG组成 <br> PG 的数量会影响Ceph集群的行为和数据的持久性。集群扩容后可以增大PG数量：5个以下OSD设置为128即可 <br> PG的特点：同一个PG中所有的对象，在相同一组OSDs上被复制。复制型Pool中PG可以有一个作为主（Primary）OSD，其它作为从OSD。一个对象仅仅属于一个PG，也就是说对象存储在固定的一组OSDs上|
|CRUSH（Controlled Replication Under Scalable Hashing）|CRUSH即基于可扩容哈希的受控复制，是一种数据分发算法，类似于哈希和一致性哈希。哈希的问题在于数据增长时不能动态添加Bucket，一致性哈希的问题在于添加Bucket时数据迁移量比较大，其他数据分发算法依赖中心的Metadata服务器来存储元数据因而效率较低，CRUSH则是通过计算、接受多维参数的来解决动态数据分发的场景<br> CRUSH与一致性哈希最大的区别在于接受的参数多了Cluster map和Placement rules，这样就可以根据目前Cluster的状态动态调整数据位置，同时通过算法得到一致的结果,基于此算法，Ceph存储集群能够动态的扩容、再平衡、恢复|
|Object|Ceph最底层的存储单元是Object，每个Object包含元数据和原始数据 <br> 一个RBD会包含很多个Object|
|OSD（Object Storage Daemon）|对象存储守护进程，负责响应客户端请求返回具体数据的进程。Ceph集群中有大量OSD <br> 一个节点上通常只运行一个OSD守护进程，此守护进程在一个存储驱动器上只运行一个 filestore|
|EC（Erasure Code）|即纠删码，是一种前向错误纠正技术（Forward Error Correction，FEC），主要应用在网络传输中避免包的丢失， 存储系统利用它来提高可靠性。相比多副本复制而言， 纠删码能够以更小的数据冗余度获得更高数据可靠性， 但编码方式较复杂，需要大量计算 。纠删码只能容忍数据丢失，无法容忍数据篡改，纠删码正是得名与此|

#### 组件：

**mon：**       
&emsp;&emsp;监视器维护集群状态的多种映射—— 包monmap、OSD map、PG map、CRUSH map、MDS map，同时提供认证和日志记录服务。Ceph会记录Monitor、OSD、PG的每次状态变更历史（此历史称作epoch）。客户端连到单个监视器并获取当前映射就能确定所有监视器、 OSD 和元数据服务器的位置。依赖于CRUSH算法和当前集群状态映射，客户端就能计算出任何对象的位置，直连OSD读写数据。

&emsp;&emsp; Ceph客户端、其它守护进程通过配置文件发现mon，但是mon之间的相互发现却依赖于monmap的本地副本。所有mon会基于分布式一致性算法Paxos，确保各自本地的monmap是一致的，当新增一个mon后，所有现有mon的monmap都自动更新为最新版本

##### 监视器同步： 

&emsp;&emsp; 使用多个mon时，每个mon都会检查其它mon是否具有更新的集群状态映射版本 —— 存在一个或多个epoch大于当前mon的最高epoch。太过落后的mon可能会离开quorum，同步后再加入quorum。执行同步时，mon分为三类角色：
- Leader：具有最新版本状态映射的mon
- Provider：同上，但是它的最新状态是从Leader同步获得
- Requester：落后于Leader，必须获取最新集群状态映射才能重回quorum