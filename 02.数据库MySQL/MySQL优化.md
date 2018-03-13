## MySQL优化

### 1、硬件优化：
- CPU：一台机器8-16颗CPU
- Mem：96G-128G，3-4个实例；32G-64G，运行2个实例
- Disk：数量越多越好，性能：SSD(高并发) > SAS(普通线上业务) > SATA(线下)    
`RAID 4块盘：RAID0 > RAID10 > RAID5 > RAID1`
- 网卡：多块网卡bond，以及buffer、tcp优化

### 2、my.cnf里参数的优化：
`my.cnf里参数优化的幅度很小，大部分是架构以及SQL语句优化`


### 3、SQL语句的优化：
#### 3.1 索引优化：
##### 白名单机制
`在`  