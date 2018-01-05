## MySQL (MariaDB) 安装:
`建议：将数据目录和二进制目录使用不同的目录，将数据单独存放`

### 1、LVM创建新的磁盘分区(模拟两块磁盘)
```ruby
/dev/sdb磁盘(总共10G)：

[root@mysql ~]# fdisk /dev/sdb

Command (m for help): p

Disk /dev/sdb: 10.7 GB, 10737418240 bytes
255 heads, 63 sectors/track, 1305 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xb2ddf76e

   Device Boot      Start         End      Blocks   Id  System

Command (m for help): n
Command action
   e   extended
   p   primary partition (1-4)
p
Partition number (1-4): 1
First cylinder (1-1305, default 1): 
Using default value 1
Last cylinder, +cylinders or +size{K,M,G} (1-1305, default 1305): +6G

Command (m for help): t
Selected partition 1
Hex code (type L to list codes): 8e
Changed system type of partition 1 to 8e (Linux LVM)

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.


/dev/sdc磁盘(总共10G)：
[root@mysql ~]# fdisk /dev/sdc

Command (m for help): p

Disk /dev/sdc: 10.7 GB, 10737418240 bytes
255 heads, 63 sectors/track, 1305 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x94d82ace

   Device Boot      Start         End      Blocks   Id  System

Command (m for help): n
Command action
   e   extended
   p   primary partition (1-4)
p
Partition number (1-4): 1
First cylinder (1-1305, default 1): 
Using default value 1
Last cylinder, +cylinders or +size{K,M,G} (1-1305, default 1305): +6G

Command (m for help): t
Selected partition 1
Hex code (type L to list codes): 8e
Changed system type of partition 1 to 8e (Linux LVM)

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```
### 2、创建逻辑卷并挂载磁盘

```ruby
/dev/sdb:

[root@mysql ~]# partx -a /dev/sdb1 /dev/sdb  //partx -a /dev/sdb可能因为虚拟机的原因，这样方式会报错
[root@mysql ~]# pvcreate /dev/sdb1
  Physical volume "/dev/sdb1" successfully created
[root@mysql ~]# vgcreate myvg /dev/sdb1 
  Volume group "myvg" successfully created
[root@mysql ~]# lvcreate -L 3G -n mydata myvg
  Logical volume "mydata" created
[root@mysql ~]# mke2fs -t ext4 /dev/myvg/mydata 
# 如果想使用xfs格式化，需要yum install xfsprogs -y
#[root@mysql ~]# modprobe xfs // 内核是支持xfs的
#[root@mysql ~]# modinfo xfs
#[root@mysql ~]# mkfs.xfs /dev/myvg/mydata
[root@mysql ~]# mkdir -pv /mydata
mkdir: created directory `/mydata'

```

```ruby
/dev/sdc:

[root@mysql ~]# partx -a /dev/sdc1 /dev/sdc 
[root@mysql ~]# pvcreate /dev/sdc1
  Physical volume "/dev/sdc1" successfully created
[root@mysql ~]# vgcreate ourvg /dev/sdc1 
  Volume group "ourvg" successfully created
[root@mysql ~]# lvcreate -L 3G -n ourdata ourvg
  Logical volume "ourdata" created
[root@mysql ~]# mke2fs -t ext4 /dev/ourvg/ourdata 
[root@mysql ~]# mkdir -pv /ourdata
mkdir: created directory `/ourdata'   
```
```ruby
[root@mysql ~]# vim /etc/fstab
	/dev/myvg/mydata      /mydata                ext4    defaults        0 0
	/dev/ourvg/ourdata    /ourdata               ext4    defaults        0 0

[root@mysql ~]# mount -a
[root@mysql ~]# lvs
  LV      VG        Attr       LSize  Pool Origin Data%  Move Log Cpy%Sync Convert
  mydata  myvg      -wi-ao----  3.00g                                             
  ourdata ourvg     -wi-ao----  3.00g                                             
  lv_root vg_mysql1 -wi-ao---- 26.51g                                             
  lv_swap vg_mysql1 -wi-ao----  3.00g  

```
### 3、创建MySQL的数据目录和binlog目录
`其实数据存放和binlog存放都应该是单独的一块磁盘`
```ruby
[root@mysql ~]# df -h
Filesystem                     Size  Used Avail Use% Mounted on
/dev/mapper/vg_mysql1-lv_root   27G  1.7G   24G   7% /
tmpfs                          939M     0  939M   0% /dev/shm
/dev/sda1                      485M   32M  428M   7% /boot
/dev/mapper/myvg-mydata        3.0G   69M  2.8G   3% /mydata
/dev/mapper/ourvg-ourdata      3.0G   69M  2.8G   3% /ourdata


[root@mysql ~]# chown -R mysql.mysql /mydata/*   // data目录存放路径
[root@mysql ~]# chown -R mysql.mysql /ourdata/*  // binlogs文件存放路径

[root@mysql ~]# groupadd -r mysql
[root@mysql ~]# useradd -g mysql -r -s /sbin/nologin -M -d /mydata/data mysql
```
### 4、编译安装MariaDB：
#### 4.1 安装cmake：
```ruby
方式一： yum安装
[root@mysql ~]# yum info cmake
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.tuna.tsinghua.edu.cn
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
Installed Packages
Name        : cmake
Arch        : x86_64
Version     : 2.8.12.2
......

[root@mysql ~]# yum install cmake -y
[root@mysql ~]# which cmake
/usr/bin/cmake

方式二：编译安装：
[root@mysql ~]# yum groupinstall "Development Tools" "Server Platform Development" -y
[root@mysql ~]# tar xf cmake-2.8.8.tar.gz 
[root@mysql ~]# cd cmake-2.8.8
[root@mysql cmake-2.8.8]# ./bootstrap
[root@mysql cmake-2.8.8]# gmake
[root@mysql cmake-2.8.8]# gmake install

[root@mysql cmake-2.8.8]# which cmake
/usr/local/bin/cmake

```
#### 4.2 编译MariaDB：
```ruby
[root@mysql ~]# tar xf mariadb-5.5.56.tar.gz
[root@mysql mariadb-5.5.56]# cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DMYSQL_DATADIR=/mydata/data \
-DSYSCONFDIR=/etc \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITH_READLINE=1 \
-DWITH_SSL=system \
-DWITH_ZLIB=system \
-DWITH_LIBWRAP=0 \
-DMYSQL_UNIX_ADDR=/tmp/mysql.sock \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci
# 上面编译安装的路径是/usr/local/mysql，而不是/usr/local/mariadb-5.5.56，所以不需要创建软连接；
# 如果上面的安装路径是/usr/local/mariadb-5.5.56，则就需要创建软连接:ln -sv mariadb-5.5.56/ mysql

[root@mysql mariadb-5.5.56]# make && make install 
```
#### 4.3 初始化配置
```ruby
[root@mysql ~]# chown -R mysql.mysql /usr/local/mysql/*
[root@mysql ~]# cp /usr/local/mysql/support-files/my-large.cnf /etc/my.cnf 
cp: overwrite `/etc/my.cnf'? y

[root@mysql ~]# vim /etc/my.cnf 
datadir = /mydata/data
innodb_file_per_table = on
skip_name_resolve = on
log-bin=/ourdata/binlogs/mysql-bin  //默认配置文件中有log-bin的配置

[root@mysql mysql]# scripts/mysql_install_db --user=mysql --datadir=/mydata/data/
WARNING: The host 'mysql' could not be looked up with resolveip.
This probably means that your libc libraries are not 100 % compatible
with this binary MariaDB version. The MariaDB daemon, mysqld, should work
normally with the exception that host name resolving will not work.
This means that you should use IP addresses instead of hostnames
when specifying MariaDB privileges !
Installing MariaDB/MySQL system tables in '/mydata/data/' ...
180105 22:49:43 [Warning] 'THREAD_CONCURRENCY' is deprecated and will be removed in a future release.
180105 22:49:43 [Note] ./bin/mysqld (mysqld 5.5.56-MariaDB) starting as process 16641 ...
OK
Filling help tables...
180105 22:49:43 [Warning] 'THREAD_CONCURRENCY' is deprecated and will be removed in a future release.
180105 22:49:43 [Note] ./bin/mysqld (mysqld 5.5.56-MariaDB) starting as process 16650 ...
OK
......

[root@mysql ~]# ls /mydata/data/
aria_log.00000001  aria_log_control  mysql  performance_schema  test
[root@mysql ~]# 
[root@mysql ~]# ls /ourdata/binlog/
mysql-bin.000001  mysql-bin.000002  mysql-bin.index

[root@mysql mysql]# cp support-files/mysql.server /etc/rc.d/init.d/mysqld
[root@mysql ~]# chkconfig --add mysqld
[root@mysql ~]# chkconfig --list mysqld
[root@mysql ~]# chmod 777 -R /var/log/
[root@mysql ~]# vim /etc/profile
PATH=$PATH:/usr/local/mysql/bin/
[root@mysql ~]# source /etc/profile
```
#### 4.4 启动MySQL：
```ruby
[root@mysql ~]# service mysqld start
Starting MySQL.180105 01:10:30 mysqld_safe Logging to '/var/log/mysqld.log'.
180105 01:10:30 mysqld_safe Starting mysqld daemon with databases from /mydata/data/
. SUCCESS! 
[root@mysql ~]# ps -ef | grep mysql
root      1926     1  0 01:10 pts/1    00:00:00 /bin/sh /usr/local/mysql/bin/mysqld_safe --datadir=/mydata/data/ --pid-file=/mydata/data//mysql.pid
mysql     2328  1926  1 01:10 pts/1    00:00:00 /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/mydata/data/ --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=/var/log/mysqld.log --pid-file=/mydata/data//mysql.pid --socket=/tmp/mysql.sock --port=3306
root      2360  1310  0 01:10 pts/1    00:00:00 grep mysql

# mysql服务器真正运行的不是mysqld，而是mysqld_safe（线程安全的mysql）
```

### 5、对编译好的MariaDB包直接使用：
```ruby
[root@mysql ~]# tar xf mariadb-5.5.56-linux-x86_64.tar.gz -C /usr/local/
[root@mysql ~]# ln -sv /usr/local/mariadb-5.5.56-linux-x86_64/ /usr/local/mysql
`/usr/local/mysql' -> `/usr/local/mariadb-5.5.56-linux-x86_64/'
[root@mysql ~]# chown -R root:mysql /usr/local/mysql/*

[root@mysql mysql]# scripts/mysql_install_db --datadir=/mydata/data/ --user=mysql
WARNING: The host 'mysql' could not be looked up with resolveip.
This probably means that your libc libraries are not 100 % compatible
with this binary MariaDB version. The MariaDB daemon, mysqld, should work
normally with the exception that host name resolving will not work.
This means that you should use IP addresses instead of hostnames
when specifying MariaDB privileges !
Installing MariaDB/MySQL system tables in '/mydata/data/' ...
180105  0:56:10 [Note] ./bin/mysqld (mysqld 5.5.56-MariaDB) starting as process 1867 ...
OK
Filling help tables...
180105  0:56:11 [Note] ./bin/mysqld (mysqld 5.5.56-MariaDB) starting as process 1876 ...
OK
......

# 查看帮助信息：scripts/mysql_install_db --help

```
```ruby
[root@mysql mysql]# cp support-files/mysql.server /etc/rc.d/init.d/mysqld
[root@mysql mysql]# chkconfig --add mysqld
[root@mysql mysql]# chkconfig --list mysqld
mysqld         	0:off	1:off	2:on	3:on	4:on	5:on	6:off

[root@mysql mysql]# ls /mydata/data/
aria_log.00000001  aria_log_control  mysql  performance_schema  test

[root@mysql mysql]# mkdir /etc/mysql
[root@mysql mysql]# 
[root@mysql mysql]# cd /usr/local/mysql/          
[root@mysql mysql]# cp support-files/my-large.cnf /etc/mysql/my.cnf
[root@mysql mysql]# vim /etc/mysql/my.cnf 
datadir = /mydata/data/
innodb_file_per_table = on
skip_name_resolve = on 

[root@mysql mysql]# chmod 777 -R /var/log/
```


