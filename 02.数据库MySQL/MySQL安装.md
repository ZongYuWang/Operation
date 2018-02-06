## 一、采用cmake+LVM方式安装MariaDB:
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

-g --gid GROUP  name or ID of the primary group of the new account
-r --system   create a system account
-s --shell SHELL  login shell of the new account
-M --no-create-home  do not create the user's home directory
-d --home-dir HOME_DIR    home directory of the new account

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

##### 编译选项：
```ruby
指定安装文件的安装路径时常用的选项：
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql
-DMYSQL_DATADIR=/data/mysql
-DSYSCONFDIR=/etc

默认编译的存储引擎包括：csv、myisam、myisammrg和heap。若要安装其它存储引擎，可以使用类似如下编译选项：
-DWITH_INNOBASE_STORAGE_ENGINE=1
-DWITH_ARCHIVE_STORAGE_ENGINE=1
-DWITH_BLACKHOLE_STORAGE_ENGINE=1
-DWITH_FEDERATED_STORAGE_ENGINE=1

若要明确指定不编译某存储引擎，可以使用类似如下的选项：
-DWITHOUT_<ENGINE>_STORAGE_ENGINE=1
比如：
-DWITHOUT_EXAMPLE_STORAGE_ENGINE=1
-DWITHOUT_FEDERATED_STORAGE_ENGINE=1
-DWITHOUT_PARTITION_STORAGE_ENGINE=1

如若要编译进其它功能，如SSL等，则可使用类似如下选项来实现编译时使用某库或不使用某库：
-DWITH_READLINE=1
-DWITH_SSL=system
-DWITH_ZLIB=system
-DWITH_LIBWRAP=0

其它常用的选项：
-DMYSQL_TCP_PORT=3306
-DMYSQL_UNIX_ADDR=/tmp/mysql.sock
-DENABLED_LOCAL_INFILE=1
-DEXTRA_CHARSETS=all
-DDEFAULT_CHARSET=utf8
-DDEFAULT_COLLATION=utf8_general_ci
-DWITH_DEBUG=0
-DENABLE_PROFILING=1
```
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

// 如果想清理此前的编译所生成的文件，则需要使用如下命令：
make clean
rm CMakeCache.txt
```
#### 4.3 初始化配置
```ruby
配置格式：类ini格式，为各程序均通过单个配置文件提供配置信息
配置文件查找次序：/etc/my.cnf --> /etc/mysql/my.cnf -->$MYSQL_HOME/my.cnf --> --default-extra-file=/path/to/somedir/my.cnf --> ~/.my.cnf
// 按照上面的顺序查找mysql的配置文件，上一个若没有，会依次向下查找

配置文件：集中式的配置，能够为mysql的各应用程序提供配置信息
[mysqld]
[mysqld_safe]
[mysqld_muti]
[server]
[mysql]
[mysqldump]
[client]
// 在配置文件中配置文件项可以使用中横线-也可以使用下划线_，但是最好统一一致
```

```ruby
[root@mysql ~]# chown -R mysql.mysql /usr/local/mysql/*
[root@mysql ~]# cp /usr/local/mysql/support-files/my-large.cnf /etc/my.cnf 
cp: overwrite `/etc/my.cnf'? y

[root@mysql ~]# vim /etc/my.cnf 
datadir = /mydata/data
innodb_file_per_table = on
skip_name_resolve = on
log-bin=/ourdata/binlogs/mysql-bin  //默认配置文件中有log-bin的配置
```
```ruby
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
```
```ruby
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

## 二、采用cmake方式安装MySQL
由于MySQL5.5.xx-5.6.xx产品系列特殊性，所以编译方式也和早期的产品安装方式不同，采用cmake活gmake方式编译安装
### 1、安装依赖包
```ruby
[root@mysql ~]# yum install -y gcc gcc-c++ ncurses-devel
```
```ruby
编译安装cmake：

[root@mysql ~]# tar xf cmake-3.3.2.tar.gz 
[root@mysql ~]# cd cmake-3.3.2
[root@mysql cmake-3.3.2]# ./configure && make && make install

[root@mysql ~]# which cmake
/usr/local/bin/cmake

```
```ruby
编译安装bison

[root@mysql ~]# tar xf bison-2.5.tar.gz 
[root@mysql ~]# cd bison-2.5
[root@mysql bison-2.5]# ./configure && make && make install

[root@mysql ~]# which bison
/usr/local/bin/bison

```

### 2、创建MySQL用户并创建MySQL数据目录：
```ruby
[root@mysql ~]# mkdir -p /mydata/data
[root@mysql ~]# mkdir -p /usr/local/mysql

[root@mysql ~]# groupadd -r mysql
[root@mysql ~]# useradd -g mysql -r -d /mydata/data/ mysql

[root@mysql ~]# chown mysql.mysql -R /mydata/data/

```

### 3、编译安装MySQL：
```ruby
[root@mysql ~]# tar xf mysql-5.5.20.tar.gz 
[root@mysql ~]# cd mysql-5.5.20
[root@mysql mysql-5.5.20]#  cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
>  -DMYSQL_UNIX_ADDR=/tmp/mysql.sock \
>  -DDEFAULT_CHARSET=utf8 \
>  -DDEFAULT_COLLATION=utf8_general_ci \
>  -DWITH_EXTRA_CHARSETS=all \
>  -DWITH_MYISAM_STORAGE_ENGINE=1 \
>  -DWITH_MYISAM_STORAGE_ENGINE=1 \
>  -DWITH_MEMORY_STORAGE_ENGINE=1 \
>  -DWITH_READLINE=1 \
>  -DENABLED_LOCAL_INFILE=1 \
>  -DMYSQL_DATADIR=/mydata/data \
>  -DMYSQL_USER=mysql

[root@mysql mysql-5.5.20]# make && make install
```

### 4、配置MySQL：
```ruby

[root@mysql ~]# cd /usr/local/mysql/
[root@mysql mysql]# scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/mydata/data

[root@mysql mysql]# cp support-files/my-large.cnf /etc/my.cnf 
[root@mysql mysql]# cp support-files/mysql.server /etc/init.d/mysqld
[root@mysql mysql]# chmod 755 /etc/init.d/mysqld
```

`设置环境变量:`
```ruby
[root@mysql ~]# vim /etc/profile
export PATH=/usr/local/mysql/bin:$PATH
[root@mysql ~]# source /etc/profile

```
### 5、启动MySQL：
```ruby
[root@mysql ~]# chkconfig --add mysqld
[root@mysql ~]# /etc/init.d/mysqld start
Starting MySQL..... SUCCESS!
```
`自动化安装脚本：`
```ruby
#!/bin/sh

# 参数设置
INSTALL_PATH=/usr/local/mysql # 指定安装目录
DATA_PATH=/mnt/datadisk1/soft/mysql # 指定数据目录

if [ -s /etc/my.cnf ];then
rm -rf /etc/my.cnf
fi

echo "----------------------------------start install mysql -----------------------------"
yum install -y gcc gcc-c++ gcc-g77 autoconf automake zlib* fiex* libxml* ncurses-devel libmcrypt* libtool-ltdl-devel* cmake
mkdir -p $DATA_PATH
if [ 'grep "mysql" /etc/passwd | wc -l' ]; then
echo "adding user mysql"
groupadd mysql
useradd -s /sbin/nologin -M -g mysql mysql
else
echo "mysql user exists"
fi

echo "------------------------------unpackaging mysql -----------------------------------"
tar -xvf mysql-5.5.20.tar.gz
cd mysql-5.5.20

echo "-------------------------configuring mysql,please wait-----------------"
cmake -DCMAKE_INSTALL_PREFIX=$INSTALL_PATH \
-DMYSQL_UNIX_ADDR=/tmp/mysql.sock \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DWITH_EXTRA_CHARSETS:STRING=utf8,gbk \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_MEMORY_STORAGE_ENGINE=1 \
-DWITH_READLINE=1 \
-DENABLED_LOCAL_INFILE=1 \
-DMYSQL_DATADIR=$DATA_PATH \
-DMYSQL_USER=mysql

if [ $? -ne 0 ];then
echo "configure failed ,please check it out!"
exit 1
fi

echo "make mysql, please wait for 10 minutes"
make
if [ $? -ne 0 ];then
echo "make failed ,please check it out!"
exit 1
fi

make install

chmod +w $INSTALL_PATH
chown -R mysql:mysql $INSTALL_PATH
ln -s $INSTALL_PATH/lib/libmysqlclient.so.16 /usr/lib/libmysqlclient.so.16
cd support-files/
cp my-large.cnf /etc/my.cnf
cp mysql.server /etc/init.d/mysqld
sed -i 's#^thread_concurrency = 8#& \ndatadir = '$DATA_PATH' \nbasedir = '$INSTALL_PATH' \nlog-error = '$INSTALL_PATH'/mysql_error.log \npid-file = '$INSTALL_PATH'/data/mysql.pid \ndefault-storage-engine=MyISAM \nuser = mysql#g' /etc/my.cnf
$INSTALL_PATH/scripts/mysql_install_db --user=mysql --basedir=$INSTALL_PATH --datadir=$DATA_PATH
chmod +x /etc/init.d/mysqld

sed -i "s#^basedir=#basedir="$INSTALL_PATH"#g" /etc/init.d/mysqld
sed -i "s#^datadir=#datadir="$DATA_PATH"#g" /etc/init.d/mysqld

chkconfig --add mysqld
chkconfig --level 345 mysqld on

export PATH=/usr/local/mysql/bin:$PATH

ln -s $INSTALL_PATH/bin/mysql /usr/bin

echo "------------------------------------------------------------------------------------------"
echo "------------------------mysql install successful,congratulations!-------------------------"
echo "------------------------------------------------------------------------------------------"

```



## 三、采用YUM/RPM方式安装MySQL
[下载最新版MySQL源](http://dev.mysql.com/downloads/repo/yum/)     
yum/rpm安装适合对数据库要求不太高的场合

### 1、安装MySQL：
```ruby
[root@mysql ~]# rpm -Uvh mysql57-community-release-el6-11.noarch.rpm
[root@mysql ~]# yum install mysql-server

[root@mysql ~]# service mysqld start
Initializing MySQL database:                               [FAILED]
[root@mysql ~]# service mysqld start
Starting mysqld:                                           [  OK  ]

```
### 2、修改MySQL的root账号密码：
```ruby
[root@mysql ~]# /etc/init.d/mysqld stop
[root@mysql ~]# mysqld_safe --user=mysql --skip-grant-tables --skip-networking &
[root@mysql ~]# mysql -u root mysql
mysql> use mysql;
mysql> update mysql.user set authentication_string=password('root') where user='root';
// 新版本的MySQL已经没有password这个字段了
mysql> flush privileges;

```
### 3、登陆MySQL：
```ruby
[root@mysql ~]# /etc/init.d/mysqld start
[root@mysql ~]# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 5.7.20
```


## 四、对编译好的MariaDB包直接使用：
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