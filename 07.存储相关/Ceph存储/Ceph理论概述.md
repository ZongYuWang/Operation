## Ceph理论概述

### 1、简介
Ceph是一个高性能、可扩容的分布式存储系统，它提供三大功能：
- 对象存储：提供RESTful接口，也提供多种编程语言绑定。兼容S3、Swift
- 块存储：由RBD提供，可以直接作为磁盘挂载，内置了容灾机制
- 文件系统：提供POSIX兼容的网络文件系统CephFS，专注于高性能、大容量存储

### 2、架构

![](https://github.com/ZongYuWang/image/blob/master/Ceph/Ceph_architecture3.png)

<br>

![](https://github.com/ZongYuWang/image/blob/master/Ceph/Ceph_architecture1.png)

<br>

![](https://github.com/ZongYuWang/image/blob/master/Ceph/Ceph_architecture2.png)

### 3、基础术语和组件说明
#### 3.1 术语：

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

#### 3.2 组件：

**3.2.1 mon：**       
&emsp;&emsp;监视器维护集群状态的多种映射—— 包monmap、OSD map、PG map、CRUSH map、MDS map，同时提供认证和日志记录服务。Ceph会记录Monitor、OSD、PG的每次状态变更历史（此历史称作epoch）。客户端连到单个监视器并获取当前映射就能确定所有监视器、 OSD 和元数据服务器的位置。依赖于CRUSH算法和当前集群状态映射，客户端就能计算出任何对象的位置，直连OSD读写数据。

&emsp;&emsp; Ceph客户端、其它守护进程通过配置文件发现mon，但是mon之间的相互发现却依赖于monmap的本地副本。所有mon会基于分布式一致性算法Paxos，确保各自本地的monmap是一致的，当新增一个mon后，所有现有mon的monmap都自动更新为最新版本

##### 监视器同步：     
使用多个mon时，每个mon都会检查其它mon是否具有更新的集群状态映射版本 —— 存在一个或多个epoch大于当前mon的最高epoch。太过落后的mon可能会离开quorum，同步后再加入quorum。执行同步时，mon分为三类角色：
- Leader：具有最新版本状态映射的mon
- Provider：同上，但是它的最新状态是从Leader同步获得
- Requester：落后于Leader，必须获取最新集群状态映射才能重回quorum

##### 时钟偏移：  
如果mon的时钟不同步，可能会导致：    
- 守护进程忽略收到的消息（时间戳过时）
- 消息未及时收到时，超时触发得太快或太晚

**3.2.2 OSD：**
##### OSD日志：
OSD使用日志的原因有两个：
- 速度： 日志使得 OSD 可以快速地提交小块数据的写入， Ceph 把小片、随机 IO 依次写入日志，这样，后端文件系统就有可能归并写入动作，并最终提升并发承载力。因此，使用 OSD 日志能展现出优秀的突发写性能，实际上数据还没有写入 OSD ，因为文件系统把它们捕捉到了日志
- 一致性：OSD需要一个能保证原子化复合操作的文件系统接口。 OSD 把一个操作的描述写入日志，并把操作应用到文件系统。这确保了对象（例如归置组元数据）的原子更新。每隔一段时间（由filestore max sync interval 和 filestore min sync interval控制 ）， OSD 会停止写入，把日志同步到文件系统，这样允许 OSD 修整日志里的操作并重用空间。若失败， OSD 从上个同步点开始重放日志。日志的原子性表现在，它不使用操作系统的文件缓存（基于内存），避免断电丢数据的问题        

`OSD进程在往数据盘上刷日志数据的过程中，是停止写操作的。 `

通常使用独立SSD来存储日志，原因是：
- 避免针对单块磁盘的双重写入 —— 先写日志，再写filestore
- SSD性能好，可以降低延迟提升IOPS

##### OSD状态矩阵：
| - | IN | OUT |
| :- | :- | :- | 
|UP|正常状态，OSD位于集群中，且接收数据|OSD虽然在运行，但是被踢出集群 —— CRUSH不会再分配归置组给它 |
|DOWN|这种状态不正常，集群处于非健康状态|正常状态|


**3.2.3 Bluestore:**   

&emsp;&emsp; 在Luminous中，Bluestore已经代替Filestore作为默认的存储引擎。Bluestore直接管理裸设备，不使用OS提供的文件系统接口，因此它不会收到OS缓存影响       
&emsp;&emsp; 使用Bluestore时，你不需要配备SSD作为独立的日志存储，Bluestore不存在双重写入问题，它直接把数据落盘到块上，然后在RockDB中更新元数据（指定数据块的位置）     

一个基于Bluestore的OSD最多可以利用到三块磁盘，例如下面的最优化性能组合：
- 使用HDD作为数据盘
- 使用SSD作为RockDB元数据盘
- 使用NVRAM作为RockDB WAL

**3.2.4 PG:**
- Acting Set：牵涉到PG副本的OSD集合
- Up Set：指Acting Set中排除掉Down掉的OSD的子集

&emsp;&emsp; Ceph依赖于Up Set来处理客户端请求。如果 Up Set 和 Acting Set 不一致，这可能表明集群内部在重均衡或者有潜在问题。

&emsp;&emsp; 写入数据前，归置组必须处于 active 、而且应该是 clean 状态。假设一存储池的归置组有 3 个副本，为让 Ceph 确定归置组的当前状态，一归置组的主 OSD （即 acting set 内的第一个 OSD ）会与第二和第三 OSD 建立连接，并就归置组的当前状态达成一致意见

由于以下原因，集群状态可能显示为HEALTH WARN：
- 刚刚创建了一个存储池，归置组还没互联好
- 归置组正在恢复
- 刚刚增加或删除了一个 OSD
- 刚刚修改了 CRUSH 图，并且归置组正在迁移
- 某一归置组的副本间的数据不一致
- Ceph 正在洗刷一个归置组的副本
- Ceph 没有足够空余容量来完成回填操作  
 
这些情况下，集群会自行恢复，并返回 HEALTH OK 状态，归置组全部变为active+clean

##### 归置组状态表:
| 状态 | 说明 | 
| :- | :- | 
|Creating|在你创建存储池时，Ceph会创建指定数量的PG，对应此状态 <br> 创建PG完毕后，Acting Set中的OSD将进行互联，互联完毕后，PG变为Active+Clean状态，PG可以接受数据写入|
|Peering|Acting Set中的OSD正在进行互联，它们需要就PG中对象、元数据的状态达成一致。互联完成后，所有OSD达成一致意见，但是不代表所有副本的内容都是最新的|
|Active|互联完成后归置组状态会变为Active|
|Clean|主OSD和副本OSD已成功互联，并且没有偏离的归置组。 Ceph 已把归置组中的对象复制了规定次数|
|Degraded|当客户端向主 OSD 写入数据时，由主 OSD 负责把数据副本写入其余副本 OSD 。主 OSD 把对象写入存储器后，在副本 OSD 创建完对象副本并报告给主 OSD 之前，主 OSD 会一直停留在 degraded 状态 <br> 如果OSD挂了， Ceph 会把分配到此 OSD 的归置组都标记为 degraded。只要它归置组仍然处于active 状态，客户端仍可以degraded归置组写入新对象 <br> 如果OSD挂了（down）长期（ mon osd down out interval ，默认300秒）不恢复，Ceph会将其标记为out，并将其上的PG重新映射到其它OSD|
|Recovering|当挂掉的OSD重启（up）后，其内的PG中的对象副本可能是落后的，副本更新期间OSD处于此状态|
|Backfilling|新 OSD 加入集群时， CRUSH 会把现有集群内的部分归置组重分配给它。强制新 OSD 立即接受重分配的归置组会使之过载，用归置组回填可使这个过程在后台开始 <br> 回填执行期间，你可能看到以下状态之一：<br> ① backfill_wait，等待时机，回填尚未开始 <br> ② backfill_too_full，需要进行回填，但是因存储空间不足而不能完成|
|Remapped|负责某个PG的Acting Set发生变更时，数据需要从久集合迁移到新集合。此期间老的主OSD仍然需要提供服务，直到数据迁移完成|
|Stale|默认情况下，OSD每0.5秒会一次报告其归置组、出流量、引导和失败统计状态，此频率高于心跳 <br> 如果：<br> ① 归置组的主 OSD 所在的 Acting Set 没能向MON报告 <br> ② 或者其它MON已经报告，说主 OSD 已 down了 <br> 则MONs就会把此归置组标记为 stale <br> 集群运行期间，出现此状态，所有PG的主OSD挂了| 
|Inactive|归置组不能处理读写请求，因为它们在等着一个持有最新数据的 OSD 回到 up 状态|
|Unclean|归置组里有些对象的副本数未达到期望次数，它们应该在恢复中|
|Down|归置组的权威副本OSD宕机，必须等待其开机，或者被标记为lost才能继续|


**3.2.5 CRUSH:**     
&emsp;&emsp; CRUSH 算法通过计算数据存储位置来确定如何存储和检索。 CRUSH授权Ceph 客户端直接连接 OSD ，而非通过一个中央服务器或代理。数据存储、检索算法的使用，使 Ceph 避免了单点故障、性能瓶颈、和伸缩的物理限制。

&emsp;&emsp; CRUSH 需要一张集群的 Map，利用该Map中的信息，将数据伪随机地、尽量平均地分布到整个集群的 OSD 里。此Map中包含：   
- OSD 列表
- 把设备汇聚为物理位置的“桶”（Bucket，也叫失败域，Failure Domain）列表
- 指示 CRUSH 如何复制存储池中的数据的规则列表

&emsp;&emsp; 通过CRUSH map来建模存储设备的物理位置，Ceph能够避免潜在的关联性故障 —— 例如一个机柜中的设备可能共享电源、网络供应，它们更加可能因为断电而同时出现故障，Ceph会刻意的避免把数据副本放在同一机柜。

&emsp;&emsp; 新部署的OSD自动被放置到CRUSH map中，位于一个host节点（OSD所在主机名）。在默认的CRUSH失败域（Failure Domain） 设置中，副本/EC分片会自动分配在不同的host节点上，避免单主机的单点故障。在大型集群中，管理员需要更加仔细的考虑失败域设置，将副本分散到不同的Rack、Row

**3.2.6 RBD:**
##### 缓存：
&emsp;&emsp; 用户空间的Ceph块设备实现（librbd）不能使用Linux的页面缓存，因此它自己实现了一套基于内存的LRU缓存——RBD Cacheing。

&emsp;&emsp; 此缓存的行为类似于页面缓存，当OS发送屏障/Flush请求时，内存中的脏数据被刷出到OSD。


**3.2.7 CephFS:**      

&emsp;&emsp;这是一个POSIX兼容的文件系统，它使用Ceph的存储集群来保存数据。       
&emsp;&emsp;一个Ceph集群可以有0-N个CephFS文件系统，每个CephFS具有可读名称和一个集群文件系统ID（FSCID）。每个CephFS可以指定多个处于standby状态的MDS进程。        
&emsp;&emsp;每个CephFS包含若干Rank，默认是1个。Rank可以看作是元数据分片。CephFS的每个守护进程（ceph-mds）默认情况下无Rank启动，Mon会自动为其分配Rank。每个守护进程最多持有一个Rank。       
&emsp;&emsp;如果Rank没有关联到ceph-mds，则其状态为failed，否则其状态为up。       
&emsp;&emsp;每个ceph-mds都有一个关联的名称，典型情况下设置为所在的节点的主机名。每当ceph-mds启动时，会获得一个GID，在进程生命周期中，它都使用此GID。       
&emsp;&emsp;如果MDS进程超过 mds_beacon_grace seconds没有和MON联系，则它被标记为laggy       

**3.2.8 RGW:**       
&emsp;&emsp;Ceph对象存储网关是基于librados构建的一套RESTful服务，提供对Ceph存储集群的访问。此服务提供两套接口：
- S3兼容接口：Amazon S3的子集
- Swift兼容接口： OpenStack Swift的子集          

这两套接口可以混合使用。      
&emsp;&emsp;对象存储网关由守护程序radosgw负责，它作为客户端和Ceph存储集群之间的媒介。radosgw具有自己的用户管理系统。       
&emsp;&emsp;从firefly版本开始，对象存储网关在Civetweb上运行，Civetweb内嵌在ceph-radosw这个Daemon中。在老版本中，对象网关基于Apache+FastCGI