## ceph的高级使用：
#### 1、作为OSD的日志盘，不涉及到CRUSH的修改:  
![](https://github.com/ZongYuWang/image/blob/master/ceph1.png)
 Ceph使用日志来提高性能及保证数据一致性。使用快速的SSD作为OSD的日志盘来提高集群性能是SSD应用于Ceph环境中最常见的使用场景。由于OSD日志读写的特点，在选择SSD盘时，除了关注IOPS性能外，要重点注意以下3个方面：
- 写密集场景
OSD日志是大量小数据块、随机IO写操作。选购SSD作为日志盘，需要重点关注随机、小块数据、写操作的吞吐量及IOPS；当一块SSD作为多个OSD的日志盘时，因为涉及到多个OSD同时往SSD盘写数据，要重点关注顺序写的带宽。
- 分区对齐
在对SSD分区时，使用Parted进行分区并保证4KB分区对齐，避免分区不当带来的性能下降。
- O_DIRECT和O_DSYNC写模式及写缓存
由Ceph源码可知(ceph/src/os/FileJournal.cc)，OSD在写日志文件时，使用的flags是：
flags |= O_DIRECT | O_DSYNC
O_DIRECT表示不使用Linux内核Page Cache; O_DSYNC表示数据在写入到磁盘后才返回。
由于磁盘控制器也同样存在缓存，而Linux操作系统不负责管理设备缓存，O_DSYNC在到达磁盘控制器缓存之后会立即返回给调用者，并无法保证数据真正写入到磁盘中，Ceph致力于数据的安全性，对用来作为日志盘的设备，应禁用其写缓存。(# hdparm -W 0 /dev/hda 0)
使用工具测试SSD性能时，应添加对应的flag：dd … oflag=direct,dsync; fio … —direct=1, —sync=1…

#### 2、 与性能较低的SATA、SAS硬盘混用，但独立组成全SSD的Pool。需要修改CRUSH MAP，可以引出后面第四种场景:
![](https://github.com/ZongYuWang/image/blob/master/ceph2.png)
- 基本思路是编辑CRUSH MAP，先标示出散落在各存储服务器的SSD OSD以及硬盘OSD(host元素)，再把这两种类型的OSD聚合起来形成两种不同的数据根(root元素)，然后针对两种不同的数据源分别编写数据存取规则(rule元素)，最后，创建SSD Pool，并指定其数据存取操作都在SSD OSD上进行。

- 在该场景下，同一个Ceph集群里存在传统机械盘组成的存储池，以及SSD组成的快速存储池，可把对读写性能要求高的数据存放在SSD池，而把备份数据存放在普通存储池。
- 对应于Ceph作为OpenStack里统一存储后端，各组件所使用的四个存储池：Glance Pool存放镜像及虚拟机快照、Nova Pool存放虚拟机系统盘、Cinder Volume Pool存放云硬盘及云硬盘快照、Cinder Backup Pool存放云硬盘备份，可以判断出，Nova及Cinder Volume存储池对IO性能有相对较高的要求，并且大部分都是热数据，可存放在SSD Pool；而Glance和Cinder Backup存储池作为备份冷数据池，对性能要求相对较低，可存放在普通存储池。
- 这种使用场景，SSD Pool里的主备数据都是在SSD里，但正常情况下，Ceph客户端直接读写的只有主数据，这对相对昂贵的SSD来说存在一定程度上的浪费。这就引出了下一个使用场景—配置CRUSH数据读写规则，使主备数据中的主数据落在SSD的OSD上。

```py
Ceph里的命令操作不详细叙述，简单步骤示例如下：  
(1)标示各服务器上的SSD与硬盘OSD
# SAS HDD OSD
host ceph-server1-sas {
  id -2
  alg straw
  hash 0
  item osd.0 weight 1.000
  item osd.1 weight 1.000
  …
}
# SSD OSD
host ceph-server1-ssd {
  id -2
  alg straw
  hash 0
  item osd.2 weight 1.000
  …
}

(2)聚合OSD，创建数据根
root sas {
  id -1
  alg straw
  hash 0
  item ceph-server1-sas
  …
  item ceph-servern-sas
}
root ssd {
  id -1
  alg straw
  hash 0
  item ceph-server1-ssd
  …
  item ceph-servern-ssd
}

(3)创建存取规则
rule sas {
  ruleset 0
  type replicated
  …
  step take sas
  step chooseleaf firstn 0 type host
  step emit
}
rule ssd {
  ruleset 1
  type replicated
  …
  step take ssd
  step chooseleaf firstn 0 type host
  step emit
}

(4)编译及使用新的CRUSH MAP
# crushtool -c ssd_sas_map.txt -o ssd_sas_map
# ceph osd setcrushmap -i ssd_sas_map

(5)创建Pool并指定存取规则
# ceph osd pool create ssd 4096 4096
# ceph osd pool set ssd crush_ruleset 1
```

#### 3、配置CRUSH数据读写规则，使主备数据中的主数据落在SSD的OSD上，涉及CRUSH MAP的修改:  
![](https://github.com/ZongYuWang/image/blob/master/ceph3.png)
该场景基本思路和第二种类似，SATA/SAS机械盘和SSD混用，但SSD的OSD节点不用来组成独立的存储池，而是配置CURSH读取规则，让所有数据的主备份落在SSD OSD上。Ceph集群内部的数据备份从SSD的主OSD往非SSD的副OSD写数据。
这样，所有的Ceph客户端直接读写的都是SSD OSD 节点，既提高了性能又节约了对OSD容量的要求。
配置重点是CRUSH读写规则的设置，关键点如下：
```py

rule ssd-primary {
  ruleset 1
  …
  step take ssd
  step chooseleaf firstn 1 type host #从SSD根节点下取1个OSD存主数据
  step emit
  step take sas
  step chooseleaf firstn -1 type host #从SAS根节点下取其它OSD节点存副本数据
  step emit
}
```
#### 4、作为Ceph Cache Tiering技术中的Cache层，涉及CRUSH MAP的修改:
![](https://github.com/ZongYuWang/image/blob/master/ceph4.png)
- Cache Tiering的基本思想是冷热数据分离，用相对快速/昂贵的存储设备如SSD盘，组成一个Pool来作为Cache层，后端用相对慢速/廉价的设备来组建冷数据存储池。
- Ceph Cache Tiering Agent处理缓存层和存储层的数据的自动迁移，对客户端透明操作透明。Cahe层有两种典型使用模式：
  (1) Writeback模式
Ceph客户端直接往Cache层写数据，写完立即返回，Agent再及时把数据迁移到冷数据池。当客户端取不在Cache层的冷数据时，Agent负责把冷数据迁移到Cache层。也就是说，Ceph客户端直接在Cache层上进行IO读写操作，不会与相对慢速的冷数据池进行数据交换。
这种模式适用于可变数据的操作，如照片/视频编辑、电商交易数据等等。
  (2) 只读模式
Ceph客户端在写操作时往后端冷数据池直接写，读数据时，Ceph把数据从后端读取到Cache层。如果cachepool上有过时的数据，就把数据清除，再返回给客户端，这种模式不能保证两个层的一致性，读的时候很可能从cachepool读到过时的数据，所以不适合用于数据时常变化的场景
这种模式适用于不可变数据，如微博/微信上的照片/视频、DNA数据、X射线影像等。
 
- 第二种和第四种用法中，热数据（cache pool）用多副本模式，性能较低的冷数据（data pool）用纠删码模式。进一步降低成本。
以writeback模式把cache pool作为data pool的tier，这样对data pool的读写实际上使用的是cache pool，并且能用到cache pool的灵活性和高性能。
- 由于纠删码模式不支持部分写，故在data pooll中无法创建RBD image；设置了data pool的tier分层cache pool后，就可以创建在data pool创建RBD image了。（没有添加cache tier时，无法用rbd创建镜像）


`纠删码和Cache分层是两个紧密联系的功能。这2个功能是redhat收购Ceph后一直重视的功能。
纠删码提高了存储容量，却降低了速度；而Cache分层刚好解决了这个问题；`
原理架构如下：  
![](https://github.com/ZongYuWang/image/blob/master/ceph5.png)  
##### Cache分层的分层部署不只解决纠删码速度慢的问题:
CRUSH算法是Ceph集群的核心，在深刻理解CRUSH算法的基础上，利用SSD的高性能，可利用较少的成本增加，满足企业关键应用对存储性能的高要求。
 
Ceph作为openstack的后端存储时，可以考虑多副本的ssd pool用writeback，作为纠删码模式的冷数据存储层的cache tier，用来存放实例和云硬盘，这样用iops限制控制各个租户的性能（这也是openstack中iops限制和存储类型的正确用法）。Glance的镜像可以直接用冷数据的pool来存放。
如果用ceph作为surdoc的存储的话，也可以用只读模式的cache tier。冷数据存储还是用纠删码模式，这个结构就和我们之前有一个单独的ssd 组成的glusterfs volume类似。但是比gluster那种，更节省硬盘。
对于内蒙项目来说，由于之前zfs存储用iscsi挂载到虚拟机中，会出现掉盘的故障，可能是网络的原因。抛开故障原因不谈，如果用ceph来做存储的话，也可以用ssd cache pool来实现高性能和低成本（可能成本会略高于目前的实际成本），而且可以用openstack来统一管理，使用上也会更方便。
另外，修改ceph的crush map，也许也能做到我们一直说的就近读写，但是我还没太想明白。希望能和大家一起讨论。
 
以上就是我对ceph进一步的了解。至于具体的操作和实验，由于环境和时间所限，并没有完成。以后再补上具体的操作过程。