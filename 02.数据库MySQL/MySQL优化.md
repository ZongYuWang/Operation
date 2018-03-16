## MySQL优化

### 1、硬件优化：
- CPU：一台机器8-16颗CPU
- Mem：96G-128G，3-4个实例；32G-64G，运行2个实例
- Disk：数量越多越好，性能：SSD(高并发) > SAS(普通线上业务) > SATA(线下)    
`RAID 4块盘：RAID0 > RAID10 > RAID5 > RAID1`
- 网卡：多块网卡bond，以及buffer、tcp优化

### 2、my.cnf里参数的优化：
`my.cnf里参数优化的幅度很小，大部分是架构以及SQL语句优化`
```ruby

[mysql]
no-auto-rehash

[mysqld]
user			= mysql
port            = 3306
socket          = /tmp/mysql.sock

basedir = /usr/local/mysql
datadir = /data/mysql
open_files_limit = 10240
back_log = 600

max_connections = 3000
max_connect_errors = 6000
table_cache = 614 

external-locking = FALSE
max_allowed_packet = 32M
sort_buffer_size = 2M
join_buffer_size = 2M
thread_cache_size = 300
thread_concurrency = 8
query_cache_size = 64M
query_cache_limit = 4M
query_cache_min_res_unit = 2k
default-storage-engine = InnoDB

thread_stack = 192k
transaction_isolation = READ-COMMITTED
tmp_table_size = 256M
max_heap_table_size = 256M
long_query_time = 2
log_long_format
log-error = /log/mysql/mysql_error.log

log-slow-queries = /log/mysql/slow-log.log
pid-file = /data/mysql/mysql.pid
log-bin = /log/mysql/mysql-bin
relay-log = /log/mysql/relay-bin
relay-log-info-file = /log/mysql/relay-log.info

binlog_cache_size = 4M
max_binlog_cache_size = 8M
max_binlog_size = 512M
expire_logs_days = 7
key_buffer_size = 32M
read_buffer_size = 1M
read_rnd_buffer_size = 16M
bulk_insert_buffer_size = 64M


myisam_sort_buffer_size = 128M
myisam_max_sort_file_size = 10G
myisam_max_extra_file_size = 10G
myisam_repair_threads = 1
myisam_recover
lower_case_table_names = 1

skip-name-resolve
slave-skip-errors = 1032,1062
replicate-ignore-db=mysql

server-id = 1

innodb_additional_mem_pool_size = 16M
innodb_buffer_pool_size = 2048M
innodb_data_file_path = ibdata1:1024M:autoextend
innodb_file_io_threads = 4
innodb_thread_concurrency = 8
innodb_flush_log_at_trx_commit = 2
innodb_log_buffer_size = 16M
innodb_log_file_size = 128M
innodb_log_files_in_group = 3
innodb_max_dirty_pages_pct = 90
innodb_lock_wait_timeout  = 120
innodb_file_per_table = 0

```

调优工具：mysqlreport


















### 3、SQL语句的优化：
#### 3.1 索引优化：
##### 白名单机制
&emsp;&emsp;抓出慢SQL，配置my.cnf
```ruby
long_query_time = 2
log-slow-queries = /log/mysql/slow-log.log
```

##### 慢查询日志分析工具-mysqlsla
&emsp;&emsp;mysqldumpslow、mysqllsla、myprofi、mysql-explain-slow-log、mysqllogfilter比较

### 3.2 拆分复杂SQL语句： 
&emsp;&emsp;大的复杂的SQL语句拆分成多个小的SQL语句

### 3.3 不要让数据库参与计算 
&emsp;&emsp;数据库是存储数据的地方，不是计算数据的地方，所以计算数据不要交给数据库处理，对于数据计算，应用类处理，都要拿到前端应用解决，禁止在数据库上处理；

### 3.4 搜索功能：
&emsp;&emsp;像"like '%鞋%'",一般不要用MySQL数据库


————————————————————————

优化的起因：
- 网站出问题，网站访问慢
```ruby
mysql> show full processlist;
+----+-----------+----------------------+---------+---------+------+-------+-----------------------+
| Id | User      | Host                 | db      | Command | Time | State | Info                  |
+----+-----------+----------------------+---------+---------+------+-------+-----------------------+
| 40 | userproxy | 172.30.105.106:35080 | myproxy | Sleep   | 4466 |       | NULL                  |
| 41 | userproxy | 172.30.105.106:35082 | myproxy | Sleep   | 7334 |       | NULL                  |
| 43 | root      | localhost            | NULL    | Query   |    0 | NULL  | show full processlist |
+----+-----------+----------------------+---------+---------+------+-------+-----------------------+

```
- 慢查询语句(日志文件)：
```ruby
long_query_time = 1
log-slow-queries = /mysqlbinlogs/slow.log
```

开启慢查询日志：
```ruby
mysql> show global variables like 'slow_query_log';
+----------------+-------+
| Variable_name  | Value |
+----------------+-------+
| slow_query_log | OFF   |
+----------------+-------+

```
```ruby
mysql> set global slow_query_log=1;

```
```ruby
mysql> show global variables like 'slow_query_log';
+----------------+-------+
| Variable_name  | Value |
+----------------+-------+
| slow_query_log | ON    |
+----------------+-------+

```


### XX服务器SQL优化：
```ruby
[root@master ~]# uptime
 15:29:15 up 5 days, 15:49,  1 user,  load average: 6.44, 5.76, 5.38

```





使用show processlist可以看到大量线程等待：
```ruby
[root@CentOS-67-x-4 ~]# mysql -u root -pTianjin_sunvsoft.2016! -e "show processlist;"
+--------+-------------+----------------------+---------------+-------------+----------+-----------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------+
| Id     | User        | Host                 | db            | Command     | Time     | State                                                                       | Info                                                                                                 |
+--------+-------------+----------------------+---------------+-------------+----------+-----------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------+
| 128943 | bjyzf_zh    | 185.15.173.245:35962 | NULL          | Binlog Dump | 11728670 | Master has sent all binlog to slave; waiting for binlog to be updated       | NULL                                                                                                 |
| 236300 | system user |                      | NULL          | Connect     |   695288 | Waiting for master to send event                                            | NULL                                                                                                 |
| 236301 | system user |                      | NULL          | Connect     |     -109 | Slave has read all relay log; waiting for the slave I/O thread to update it | NULL                                                    |                                             
| 243311 | root        | 117.78.42.40:23510   | tradeeaseexit | Query       |       56 |                                                                             | select id from ad_babyshen_detail where ader='ibm_mui_vedio' and dataline='2018-02-10';              |                                                                                   
| 243312 | root        | 117.78.42.40:23509   | tradeeaseexit | Query       |       26 |                                                                             | update ad_babyshen_detail set views=views+1 where id 24513;                                          |                                                       
| 243313 | root        | 117.78.42.40:23519   | tradeeaseexit | Query       |       56 |                                                                             | update ad_babyshen_detail set views=views+1 where id 24498;                                          |                                                       
| 243314 | root        | 117.78.42.40:23517   | tradeeaseexit | Query       |       56 |                                                                             | select id from ad_babyshen_detail where ader='ibm_esd_jazz_flash' and dataline='2018-03-23';         |                                                                                       
| 243315 | root        | 117.78.42.40:23521   | tradeeaseexit | Query       |        1 |                                                                             | select id from ad_babyshen_detail where ader='ibm_esd_jazz_flash' and dataline='2018-03-20';         |                                                                                       
| 243316 | root        | 192.168.2.102:45673  | tradeeaseexit | Query       |       25 |                                                                             | select id from ad_babyshen_detail where ader='ibm_esd_jazz_flash' and dataline='2018-03-14';         |                                                                                        
| 243323 | root        | 117.78.42.40:23535   | tradeeaseexit | Query       |       27 |                                                                             | update ad_babyshen_detail set views=views+1 where id 24496;                                          |                                                     
| 243324 | root        | 117.78.42.40:23537   | tradeeaseexit | Sleep       |       12 |                                                                             | NULL                                                                                                 |
| 243325 | root        | 192.168.2.102:45681  | tradeeaseexit | Query       |        1 | Sending data                                                                | SELECT COUNT(*) from es_store s left join es_member_collect m on s.store_id=m.store_id where s.store |
| 243327 | root        | 192.168.2.102:45680  | tradeeaseexit | Sleep       |        0 |                                                                             | NULL                                                                                                 |
| 243326 | root        | 192.168.2.102:45682  | tradeeaseexit | Query       |        9 | Sending data                                                                | SELECT COUNT(*) from es_store s left join es_member_collect m on s.store_id=m.store_id where s.store |
| 243328 | root        | 192.168.2.102:45685  | tradeeaseexit | Sleep       |       10 |                                                                             | NULL                                                                                                 |
| 243329 | root        | 192.168.2.102:45686  | tradeeaseexit | Sleep       |        1 |                                                                             | NULL                                                                                                 |
| 243334 | root        | localhost            | NULL          | Query       |        0 | NULL                                                                        | show processlist                                                                                     |
+--------+-------------+----------------------+---------------+-------------+----------+-----------------------------------------------------------------------------+-------------------------
```

```ruby
[root@CentOS-67-x-4 ~]# mysql -u root -pTianjin_sunvsoft.2016! -e "show full processlist;"

```
`语句会一直处于执行状态，执行两次之后，语句还处于执行状态，这样就是查询比较慢了`

待优化的语句：
```ruby
mysql> SELECT COUNT(*) from es_store s left join es_member_collect m on s.store_id=m.store_id where s.store;
```
查看利用索引情况：
```ruby
mysql> explain SELECT COUNT(*) from es_store as s left join es_member_collect as m on s.store_id=m.store_id where s.store;
```
```ruby
mysql> SELECT SQL_NO_CACHE COUNT(*) from es_store s left join es_member_collect m on s.store_id=m.store_id where s.store; 
```
查看表结构：
```ruby
mysql> show create table es_store\G
*************************** 1. row ***************************
       Table: es_store
Create Table: CREATE TABLE `es_store` (
  `store_id` int(8) NOT NULL AUTO_INCREMENT,
  `store_name` varchar(200) DEFAULT NULL,
  `store_type` int(8) DEFAULT NULL,
  `store_provinceid` int(8) DEFAULT NULL,
  `store_cityid` int(8) DEFAULT NULL,
  `store_regionid` int(8) DEFAULT NULL,
  `attr` varchar(255) DEFAULT NULL,
  `zip` varchar(20) DEFAULT NULL,
  `tel` varchar(20) DEFAULT NULL,
  `store_level` int(10) DEFAULT NULL,
  `member_id` int(10) DEFAULT NULL,
  `member_name` varchar(50) DEFAULT NULL,
  `id_number` varchar(20) DEFAULT NULL,
  `id_img` varchar(255) DEFAULT NULL,
  `license_img` varchar(255) DEFAULT NULL,
  `disabled` int(10) DEFAULT NULL,
  `create_time` bigint(20) DEFAULT NULL,
  `end_time` bigint(20) DEFAULT NULL,
  `store_logo` varchar(255) DEFAULT NULL,
  `description` text,
  `store_recommend` int(10) DEFAULT NULL,
  `store_theme` int(10) DEFAULT NULL,
  `store_credit` int(10) DEFAULT NULL,
  `praise_rate` decimal(20,2) DEFAULT NULL,
  `store_desccredit` decimal(20,2) DEFAULT NULL,
  `store_servicecredit` decimal(20,2) DEFAULT NULL,
  `store_deliverycredit` decimal(20,2) DEFAULT NULL,
  `store_collect` int(10) DEFAULT NULL,
  `store_auth` int(10) DEFAULT NULL,
  `name_auth` int(10) DEFAULT NULL,
  `store_province` varchar(50) DEFAULT NULL,
  `store_city` varchar(50) DEFAULT NULL,
  `store_region` varchar(50) DEFAULT NULL,
  `goods_num` int(10) DEFAULT NULL,
  `qq` varchar(20) DEFAULT NULL,
  `store_banner` varchar(100) DEFAULT NULL,
  `commission` decimal(20,2) DEFAULT NULL,
  `bank_account_name` varchar(50) DEFAULT NULL,
  `bank_account_number` varchar(50) DEFAULT NULL,
  `bank_name` varchar(50) DEFAULT NULL,
  `bank_code` varchar(50) DEFAULT NULL,
  `bank_provinceid` int(10) DEFAULT NULL,
  `bank_cityid` int(10) DEFAULT NULL,
  `bank_regionid` int(10) DEFAULT NULL,
  `bank_province` varchar(20) DEFAULT NULL,
  `bank_city` varchar(20) DEFAULT NULL,
  `bank_region` varchar(200) DEFAULT NULL,
  `company_name` varchar(200) DEFAULT NULL,
  `enterprise_legal` varchar(200) DEFAULT NULL,
  `registration_number` varchar(200) DEFAULT NULL,
  `company_create_time` bigint(20) DEFAULT NULL,
  `company_size` int(8) DEFAULT NULL,
  `main_products` varchar(255) DEFAULT NULL,
  `email` varchar(200) DEFAULT NULL,
  `fax_number` varchar(200) DEFAULT NULL,
  `company_url` varchar(200) DEFAULT NULL,
  `company_description` varchar(255) DEFAULT NULL,
  `check_description` varchar(255) DEFAULT NULL,
  `check_time` bigint(20) DEFAULT NULL,
  `account` decimal(20,2) DEFAULT NULL,
  `credit_account` decimal(20,2) DEFAULT NULL,
  `credit_account_status` int(11) DEFAULT NULL,
  `prove_name` varchar(80) DEFAULT NULL,
  `prove_type` int(11) DEFAULT NULL,
  `prove_mobile` varchar(40) DEFAULT NULL,
  `bank_address` varchar(255) DEFAULT NULL,
  `prove_img` varchar(255) DEFAULT NULL,
  `reverse_id_img` varchar(255) DEFAULT NULL,
  `bcard_img` varchar(255) DEFAULT NULL,
  `parent_store` int(11) DEFAULT NULL,
  `prove_name2` varchar(80) DEFAULT NULL,
  `prove_mobile2` varchar(40) DEFAULT NULL,
  `prove_name3` varchar(80) DEFAULT NULL,
  `prove_mobile3` varchar(40) DEFAULT NULL,
  `store_location` varchar(255) DEFAULT NULL,
  `store_country` varchar(255) DEFAULT 'CHN',
  `store_market` varchar(255) DEFAULT 'RUS',
  `store_initiallist` int(11) DEFAULT '1',
  `account_manager` varchar(255) DEFAULT NULL,
  `account_area` int(8) DEFAULT '0',
  `modify_persion` varchar(255) DEFAULT NULL,
  `init_commission1` decimal(20,2) DEFAULT '0.00',
  `init_pic` varchar(255) DEFAULT NULL,
  `store_manager` varchar(255) DEFAULT NULL,
  `store_script` varchar(255) DEFAULT NULL,
  `store_num` varchar(255) DEFAULT NULL,
  `store_address` varchar(255) DEFAULT NULL,
  `store_tel` varchar(255) DEFAULT NULL,
  `store_phone` varchar(255) DEFAULT NULL,
  `parent_id` int(11) DEFAULT NULL,
  `store_ownership` int(11) DEFAULT NULL,
  `pay_id` int(11) DEFAULT '0',
  `merchant_name` varchar(255) DEFAULT NULL,
  `delivery_rule` int(11) DEFAULT '0',
  `b2bpay_id` int(20) DEFAULT '0',
  `init_limit` decimal(20,2) DEFAULT '0.00',
  `settle_type` int(8) DEFAULT NULL,
  `foreign_store_code` varchar(80) DEFAULT NULL,
  `init_limitdeposit` decimal(20,2) DEFAULT '0.00',
  `swift_code` varchar(80) DEFAULT NULL,
  `payee_address` varchar(255) DEFAULT NULL,
  `settlement_currency` int(11) DEFAULT NULL,
  `init_pic2` varchar(255) DEFAULT NULL,
  `payee` varchar(80) DEFAULT NULL,
  `passport_img` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`store_id`),
  KEY `index1` (`store_id`),
  KEY `index2` (`store_market`),
  KEY `index3` (`store_id`,`store_market`),
  KEY `index4` (`store_name`)
) ENGINE=InnoDB AUTO_INCREMENT=871 DEFAULT CHARSET=utf8

```

查看条件字段列的唯一性：
```ruby
mysql> select count(store_num) from es_store;
+------------------+
| count(store_num) |
+------------------+
|                7 |
+------------------+

mysql> select count(member_name) from es_store;
+--------------------+
| count(member_name) |
+--------------------+
|                491 |
+--------------------+

mysql> select count(*) from es_store;
+----------+
| count(*) |
+----------+
|      498 |
+----------+

// 选择唯一的列上查找结果比较少、重复比较低的列上创建索引
```
根据以上及咨询DBA其他语句的情况下，创建如下索引：

```ruby
mysql> create index s_n on es_store(store_num);

// 在生产环境中如果访问频繁的大表，创建索引会很消耗时间，也许需要几分钟，应该在业务流量低俗时创建索引；
```
```ruby
mysql> explain SELECT COUNT(*) from es_store s left join es_member_collect m on s.store_id=m.store_id where store_num=205;
```

优化之后的负载：
```ruby
[root@whj-centos63-64 ~]# uptime
 10:58:32 up 164 days,  5:51,  1 user,  load average: 1.11, 1.11, 0.98
```


案例二：
Memcached服务应用优化案例(网站打开慢，数据库负载高)：

```ruby
[root@whj-centos63-64 ~]# uptime
 10:58:32 up 164 days,  5:51,  1 user,  load average: 20, 15, 10
```
```ruby
[root@whj-centos63-64 ~]# mysql -u root -pTianjin_sunvsoft.2016! -e "show full processlist" | grep -vi sleep
381517	root	192.168.1.5:50296	tradeeaseipt	Query	0	query end	SELECT COUNT(*) from es_store like '%鞋%' 
381599	root	localhost	NULL	Query	0	NULL	show full processlist

```
&emsp;&emsp;数据库中像"like '%鞋%'", 这样的语句特别多，导致数据库负载很高，"like '%鞋%'"这样的语句对于MySQL数据库没有太大优化的余地，打开网站首页看了一下应该是首页的搜索框搜索带来的结果。   

![](https://github.com/ZongYuWang/image/blob/master/MySQL/MySQL1.png)

优化方案思路：

&emsp;&emsp;从业务上实现用户登陆后再搜索，这样减少搜索次数，从而减轻数据库服务的压力；
&emsp;&emsp;如果有大量频繁的搜索，一般是由爬虫在爬网站，分析WEB日志IP封掉；
&emsp;&emsp;配置多个主从同步，程序上实现读写分离(最好让"like '%鞋%'"这样的查询去从库查询)，减轻主库的读写压力；
&emsp;&emsp;在数据库前端加上memcached缓存服务器；
&emsp;&emsp;"like '%鞋%'"这样的语句，一般在MySQL里很难优化，可以通过搜索服务Sphinx实现搜索；
&emsp;&emsp;利用C、Ruby开发程序，实现每日读库计算搜索索引，保存在服务器上提供搜索，然后，每5分钟在从库做一次增量(这是大公司针对站内搜索采取的比较好的方案)；


### 架构上的优化：
- 业务拆分：搜索功能，like '%老男孩%',一般不要用MySQL数据库；
- 业务拆分：某些业务应用使用NoSQL持久化存储，如memcachedb、redis、ttserver
- 数据库前端必须要加cache，如memcached，用户登录、商品查询；
- 动态的数据静态化，整个文件静态化，页面片段静态化；
- 数据库集群读写分离，一主多从，通过程序或者dbproxy进行集群读写分离；
- 单表超过2000万，拆库拆表，人工拆表拆库(分布式自动伸缩架构)；



### 索引优化：

- 索引列一般为where子句中的列或连接子句中的列；
```ruby
mysql> select * from amtb where name="babyshen";

```
- 尽量不要对基数小的列做索引，如性别列；
- 尽可能使用短索引，如果对字符列索引尽量指定最小长度；
`Short keys are better，Integer best`
```ruby
mysql> create index name_index on name(name(3));

```
- 复合索引前缀特性，索引的顺序很重要；
`创建复合索引时应将最常用作限制条件列放在最左边，依次递减`
```ruby
key(a,b)....where b=5 will not use index
```
```ruby
key(a,b,c)联合索引：
可以走索引的组合：key(a),key(a,b),key(a,b,c)
不可以走索引的组合：key(b),key(b,c),key(a,c)

```
- 避免出现无用的索引(很少使用或从未被调用)；
- InnoDB:尽量指定主键，最常用较短数据类型唯一列做主键；

### 避免过度使用索引：
- 索引的建立对提高索引能力很有用，但是数据库维护它很费资源；
`尽量使用定长字符类型char，而不用varchar`
- 对性别列索引(被称为过度索引)；
`只有两个值，建立索引不仅没优势，还会影响到插入、更新速度`
- 索引会占用磁盘空间，降低写操作，执行计划要考虑各个索引；
- 索引不是越多越好；


