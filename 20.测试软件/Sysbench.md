## sysbench测试
`sysbench0.5之后的版本不支持oltp选项,可以使用sysbench-0.4.8版本`
sysbench目前可以进行如下几个方面的性能测试： 
- fileio - File I/O test  // 磁盘IO性能 
- cpu - CPU performance test   // CPU性能 
- memory - Memory functions speed test  // 内存性能 
- threads - Threads subsystem performance test // POSIX线程性能 
- mutex - Mutex performance test //调度程序性能 
- oltp - OLTP test  //数据库性能(OLTP基准测试)


### 1、数据库测试(oltp)：

#### 1.1 安装sysbench：
```ruby
[root@localhost ~]# tar xf sysbench-0.4.8.tar.gz
[root@localhost ~]# cd sysbench-0.4.8

[root@localhost sysbench-0.4.8]# ./configure --prefix=/usr/local/sysbench \
--with-mysql=/usr/local/mysql/ \
--with-mysql-includes=/usr/local/mysql/include/ \
--with-mysql-libs=/usr/local/mysql/lib/ 

[root@localhost sysbench-0.4.8]#  make && make install

```
#### 1.2 数据库测试指定：
```ruby
mysql> create database sbtest;
// 默认的库和默认的表都是sbtest
```
```ruby
/usr/local/bin/sysbench --test=oltp \
--oltp-table-size=1000000 \
--oltp-read-only=off \
--init-rng=on \
--num-threads=16 \
--max-requests=0 \
--oltp-dist-type=uniform \
--max-time=1800 \
--mysql-user=root 
--mysql-socket=/tmp/mysql.sock \
--db-driver=mysql \
--mysql-table-engine=innodb \
--oltp-test-mode=complex prepare

sysbench v0.4.8:  multi-threaded system evaluation benchmark

Creating table 'sbtest'...
Creating 1000000 records in table 'sbtest'...
 
```
#### 1.3 参数说明：
```ruby
[root@localhost ~]# /usr/local/sysbench/bin/sysbench --help=oltp help
Usage:
  sysbench [general-options]... --test=<test-name> [test-options]... command

```

##### general-options参数：
```
General options:
  --num-threads=N            number of threads to use [1]
  --max-requests=N           limit for total number of requests [10000]
  --max-time=N               limit for total execution time in seconds [0]
  --thread-stack-size=SIZE   size of stack per thread [32K]
  --init-rng=[on|off]        initialize random number generator [off]
  --test=STRING              test to run
  --debug=[on|off]           print more debugging info [off]
  --validate=[on|off]        perform validation checks where possible [off]
  --help=[on|off]            print help and exit
```
##### test-name参数：
```ruby
Compiled-in tests:
  fileio - File I/O test
  cpu - CPU performance test
  memory - Memory functions speed test
  threads - Threads subsystem performance test
  mutex - Mutex performance test
  oltp - OLTP test

```
##### test-options参数：
```ruby 
–oltp-test-mode=STRING 执行模式{simple,complex(advanced transactional),nontrx(non-transactional),sp}。默认是complex 
–oltp-reconnect-mode=STRING 重新连接模式{session(不使用重新连接。每个线程断开只在测试结束),transaction(在每次事务结束后重新连接),query(在每个SQL语句执行完重新连接),random(对于每个事务随机选择以上重新连接模式)}。默认是session 
–oltp-sp-name=STRING 存储过程的名称。默认为空 
–oltp-read-only=[on|off] 只读模式。Update，delete，insert语句不可执行。默认是off 
–oltp-skip-trx=[on|off] 省略begin/commit语句。默认是off 
–oltp-range-size=N 查询范围。默认是100 
–oltp-point-selects=N number of point selects [10] 
–oltp-simple-ranges=N number of simple ranges [1] 
–oltp-sum-ranges=N number of sum ranges [1] 
–oltp-order-ranges=N number of ordered ranges [1] 
–oltp-distinct-ranges=N number of distinct ranges [1] 
–oltp-index-updates=N number of index update [1] 
–oltp-non-index-updates=N number of non-index updates [1] 
–oltp-nontrx-mode=STRING 查询类型对于非事务执行模式{select, update_key, update_nokey, insert, delete} [select] 
–oltp-auto-inc=[on|off] AUTO_INCREMENT是否开启。默认是on 
–oltp-connect-delay=N 在多少微秒后连接数据库。默认是10000 
–oltp-user-delay-min=N 每个请求最短等待时间。单位是ms。默认是0 
–oltp-user-delay-max=N 每个请求最长等待时间。单位是ms。默认是0 
–oltp-table-name=STRING 测试时使用到的表名。默认是sbtest 
–oltp-table-size=N 测试表的记录数。默认是10000 
–oltp-dist-type=STRING 分布的随机数{uniform(均匀分布),Gaussian(高斯分布),special(空间分布)}。默认是special 
–oltp-dist-iter=N 产生数的迭代次数。默认是12 
–oltp-dist-pct=N 值的百分比被视为’special’ (for special distribution)。默认是1 
–oltp-dist-res=N ‘special’的百分比值。默认是75
```
```ruby
–db-driver=STRING specifies database driver to use (‘help’ to get list of available drivers) #指定需求测试的数据库类型，默认是mysql 
#mysql链接选项 
–mysql-host=[LIST,…] MySQL server host [localhost] #mysql主机地址 
–mysql-port= mysql端口 
–mysql-socket= mysql socket文件位置，指定这个之后 其他的链接选项均可以不指定 
–mysql-user=用来测试的mysql用户名,默认sbtest
–mysql-password=密码 
–mysql-db=测试数据库名 默认sbtest
```
##### command控制命令:参数
```ruby
prepare #准备测试，主要是生成测试数据 
run #执行测试，根据选项限制来执行测试 
cleanup #清除准备阶段生成的测试数据 
help #获取帮助文档 
version #获取版本信息
```

```ruby
[root@localhost ~]# sysbench --test=oltp \
--oltp-table-size=1000000 \
--oltp-read-only=off  \
--num-threads=16 \
--max-requests=0 \
--oltp-dist-type=uniform \
--max-time=180 \
--mysql-user=system \
--mysql-password=wangzongyu \
--mysql-socket=/tmp/mysql.sock \
--db-driver=mysql 
--mysql-table-engine=innodb 
--oltp-test-mode=complex run 
```