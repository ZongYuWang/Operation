## CentOS6.5编译安装TFS分布式文件系统：

目前TFS的编译环境基于gcc 4.1.2版本，高版本的gcc在编译时可能会出现一些错误，建议4.1.2版本上编译
`但是先不要安装gcc 4.1.2版本的，先要安装依赖软件，最后再对gcc降级，否则会出现大量的错误`

##### 1、安装依赖包：
```py
[root@localhost ~]# yum -y install  automake libtool readline readline-devel libuuid-devel zlib-devel mysql-devel gcc-c++

安装jemalloc(编译tfs源码会用到)
[root@localhost ~]# mkdir tfs
[root@localhost ~]# cd tfs
[root@localhost tfs]# wget http://www.canonware.com/download/jemalloc/jemalloc-3.5.0.tar.bz2
[root@localhost tfs]# tar jxvf jemalloc-4.4.0.tar.bz2 
[root@localhost jemalloc-4.4.0]# ./configure --prefix=/usr/local/
[root@localhost jemalloc-4.4.0]# make && make install

```
##### 2、安装tbsys和tbnet(TFS依赖底层开发包tbnet)：
```py
[root@localhost jemalloc-4.4.0]# yum install subversion
[root@localhost jemalloc-4.4.0]# cd /usr/local/src/
[root@localhost src]# svn checkout -r 18 http://code.taobao.org/svn/tb-common-utils/trunk/ tb-common-utils

[root@localhost src]# cd tb-common-utils/
[root@localhost tb-common-utils]# mkdir -p /opt/tblib

[root@localhost tb-common-utils]# vim /etc/profile
export TBLIB_ROOT=/opt/tblib/
[root@localhost tb-common-utils]# source /etc/profile
[root@localhost tb-common-utils]# sh build.sh

```

##### 3、对GCC降级：
`CentOS默认是gcc版本是4.4.7`
```py
[root@localhost tb-common-utils]# cd /root/tfs/
[root@localhost tfs]# wget http://www.mirrorservice.org/sites/sources.redhat.com/pub/gcc/releases/gcc-4.1.2/gcc-4.1.2.tar.bz2

[root@localhost tfs]# tar jxvf gcc-4.1.2.tar.bz2 
[root@localhost tfs]# cd gcc-4.1.2
[root@localhost gcc-4.1.2]# mkdir -p /opt/gcc-4.1.2/
[root@localhost gcc-4.1.2]# ./configure --prefix=/opt/gcc-4.1.2/

[root@localhost gcc-4.1.2]# yum install texinfo
[root@localhost gcc-4.1.2]# which makeinfo
/usr/bin/makeinfo

#修改当前目录下的Makefile文件：
[root@localhost gcc-4.1.2]# vim Makefile
#MAKEINFO = /root/tfs/gcc-4.1.2/missing makeinfo
MAKEINFO = /usr/bin/makeinfo

#安装依赖：
[root@localhost gcc-4.1.2]# yum -y install glibc-devel.i686 glibc-devel

#安装gcc 4.1.2
[root@localhost gcc-4.1.2]# make && make install
【说明】时间会特别长，多核处理器可以在make的时候加上并发处理命名，如make -j 8

#替换原来系统的gcc版本：
[root@localhost gcc-4.1.2]# mv /usr/bin/gcc /usr/bin/gccold
[root@localhost gcc-4.1.2]# ln -s /opt/gcc-4.1.2/bin/gcc /usr/bin/gcc
[root@localhost gcc-4.1.2]# mv /usr/bin/g++ /usr/bin/g++old
[root@localhost gcc-4.1.2]# ln -s /opt/gcc-4.1.2/bin/g++ /usr/bin/g++
```
`查看目前gcc的版本是gcc 4.1.2`
```py
[root@localhost gcc-4.1.2]# gcc -v
Using built-in specs.
Target: x86_64-unknown-linux-gnu
Configured with: ./configure --prefix=/opt/gcc-4.1.2/
Thread model: posix
gcc version 4.1.2

```

##### 4、编译安装TFS：
```py
[root@localhost gcc-4.1.2]# cd /root/tfs/
[root@localhost tfs]# svn co http://code.taobao.org/svn/tfs/tags/release-2.2.16
【说明】release-2.6.6版本有点问题，报EASY_ROOT is not work! expect EASY_ROOT/include/easy and EASY_ROOT/lib64 directory错误
[root@localhost tfs]# cd release-2.2.16/
[root@localhost release-2.2.16]# mkdir /opt/tfs-2.2.16
[root@localhost release-2.2.16]# sh build.sh init
[root@localhost release-2.2.16]# ./configure --prefix=/opt/tfs-2.2.16/  --with-release --without-tcmalloc
[root@localhost release-2.2.16]# make && make install

```
##### 5、TFS的配置：
`参考`[TFS的配置部署](http://code.taobao.org/p/tfs/wiki/deploy/ )

##### 6、TFS的启动运行：
`参考`[TFS的启动运行](http://code.taobao.org/p/tfs/wiki/start/)

##### 7、TFS的挂载说明：
` TFS不一定非得挂载一整个磁盘，可以单独挂载一个文件夹为disk目录`
```py
#编译安装之后需要从源码目录/conf下copy配置文件到安装目录
[root@localhost release-2.2.16]# cp /root/tfs/release-2.2.16/conf/ds.conf /opt/tfs-2.2.16/conf/
[root@localhost release-2.2.16]# mkdir -p /home/disk
[root@localhost release-2.2.16]# vim /opt/tfs-2.2.16/conf/ds.conf 

#执行tfs 安装目录下的format命令
[root@localhost release-2.2.16]# cd /opt/tfs-2.2.16/scripts/
[root@localhost scripts]# ./stfs format 1
【说明】执行完之后会自动在/home下创建一个disk1的目录，可以看到这个目录下面已经新增了很多文件，这些文件就是TFS的Block存储单元

```

##### 8、FAQ：
` 64为的系统，如果启动ds的时候提示error while loading shared libraries: libjemalloc.so.1: cannot open shared object file: No such file or directory 
做一下软链接就可以了 `
```py
[root@localhost ~]# ln -s /usr/local/lib/libjemalloc.so.1 /usr/lib64/libjemalloc.so.1
```