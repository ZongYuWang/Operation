## CentOS6.5编译安装TFS分布式文件系统：

目前TFS的编译环境基于gcc 4.1.2版本，高版本的gcc在编译时可能会出现一些错误，建议4.1.2版本上编译
##### 1、安装gcc 4.1.2：
```py
[root@localhost tfs]# tar xvf gcc-4.1.2.tar.bz2 
[root@localhost tfs]# cd gcc-4.1.2
[root@localhost gcc-4.1.2]# ./configure --prefix=/usr/local/gcc-4.1.2
[root@localhost gcc-4.1.2]# make

```
报错了，报错信息如下：
```py

WARNING: `makeinfo' is missing on your system.  You should only need it if
         you modified a `.texi' or `.texinfo' file, or any other file
         indirectly affecting the aspect of the manual.  The spurious
         call might also be the consequence of using a buggy `make' (AIX,
         DU, IRIX).  You might want to install the `Texinfo' package or
         the `GNU make' package.  Grab either from any GNU archive site.
make[3]: *** [fastjar.info] Error 1
make[3]: Leaving directory `/root/tfs/gcc-4.1.2/host-x86_64-unknown-linux-gnu/fastjar'
make[2]: *** [all] Error 2
make[2]: Leaving directory `/root/tfs/gcc-4.1.2/host-x86_64-unknown-linux-gnu/fastjar'
make[1]: *** [all-fastjar] Error 2
make[1]: Leaving directory `/root/tfs/gcc-4.1.2'
make: *** [all] Error 2

解决办法：
[root@localhost tfs]# yum install texinfo #安装的是4.13版本
[root@localhost gcc-4.1.2]# yum install -y ncurses
[root@localhost gcc-4.1.2]# makeinfo --version

出现此错误的原因也在于configure文件中texinfo对该版本不支持，可以在解压gcc4.1.1文件夹中的configure文件里找到
[root@localhost gcc-4.1.2]# vim configure

 if ${MAKEINFO} --version \
       | egrep 'texinfo[^0-9]*([1-3][0-9]|4\.[2-9]|[5-9])' >/dev/null 2>&1; then
      :
    else
      MAKEINFO="$MISSING makeinfo"
    fi

修改为：
 if ${MAKEINFO} --version \
       | egrep 'texinfo[^0-9]*([1-3][0-9]|4\.[1-9][2-9]|[5-9])' >/dev/null 2>&1; then
      :
    else
      MAKEINFO="$MISSING makeinfo"
    fi
    
```

重新make：
```py
[root@localhost gcc-4.1.2]# make
```

又报错了:
```py

/usr/include/gnu/stubs.h:7:27: error: gnu/stubs-32.h: No such file or directory
make[4]: *** [32/crtbegin.o] Error 1
make[4]: Leaving directory `/root/tfs/gcc-4.1.2/host-x86_64-unknown-linux-gnu/gcc'
make[3]: *** [extra32] Error 2
make[3]: Leaving directory `/root/tfs/gcc-4.1.2/host-x86_64-unknown-linux-gnu/gcc'
make[2]: *** [stmp-multilib] Error 2
make[2]: Leaving directory `/root/tfs/gcc-4.1.2/host-x86_64-unknown-linux-gnu/gcc'
make[1]: *** [all-gcc] Error 2
make[1]: Leaving directory `/root/tfs/gcc-4.1.2'
make: *** [all] Error 2

解决办法：
[root@localhost gcc-4.1.2]# yum install glibc-devel.i686 -y

```

重新编译：
```py
[root@localhost gcc-4.1.2]# ./configure --prefix=/usr/local/gcc-4.1.2 --enable-languages=c,c++ --disable-multilib && echo ok
[root@localhost gcc-4.1.2]# make
[root@localhost gcc-4.1.2]# make install

[root@localhost gcc-4.1.2]# gcc -v  #还是比较新的版本4.4.7
gcc version 4.4.7 20120313 (Red Hat 4.4.7-18) (GCC) 

[root@localhost gcc-4.1.2]# yum remove gcc

[root@localhost gcc-4.1.2]# ln -s /usr/local/gcc-4.1.2/bin/gcc /usr/bin/gcc
[root@localhost gcc-4.1.2]# ln -s /usr/local/gcc-4.1.2/bin/g++ /usr/bin/g++


```
##### 2、依赖包安装：
```py
安装libuuid-devel zlib-devel mysql-devel：
[root@localhost ~]# yum install -y libuuid-devel zlib-devel mysql-devel

安装autoconf、automake，也可以使用yum安装：
[root@localhost tfs]# tar xvf autoconf-2.69.tar.gz 
[root@localhost tfs]# cd autoconf-2.69
[root@localhost autoconf-2.69]# ./configure && make && make install

[root@localhost tfs]# cd automake-1.15
[root@localhost automake-1.15]# ./configure && make && make install 
[root@localhost tfs]# automake --version

安装libtool：
[root@localhost tfs]# yum install -y libtool

安装tbnet和tbsys：
[root@localhost tfs]# yum install -y svn
[root@localhost tfs]# svn checkout -r 18 http://code.taobao.org/svn/tb-common-utils/trunk/ tb-common-utils

需要设置TBLIB_ROOT路径
[root@localhost ~]# vim /etc/profile
export TBLIB_ROOT="/root/tfs/lib"
[root@localhost ~]# source /etc/profile

[root@localhost ~]# cd /root/tfs/tb-common-utils/
[root@localhost tb-common-utils]# sh build.sh 
```
报错了:
```py

config.status: error: cannot find input file: `src/Makefile.in'
Making all in src
make[1]: Entering directory `/root/tfs/tb-common-utils/tbnet/src'
make[1]: *** No rule to make target `all'.  Stop.
make[1]: Leaving directory `/root/tfs/tb-common-utils/tbnet/src'
make: *** [all-recursive] Error 1
Making install in src
make[1]: Entering directory `/root/tfs/tb-common-utils/tbnet/src'
make[1]: *** No rule to make target `install'.  Stop.
make[1]: Leaving directory `/root/tfs/tb-common-utils/tbnet/src'
make: *** [install-recursive] Error 1

解决办法：
[root@localhost tb-common-utils]# CPLUS_INCLUDE_PATH=$CPLUS_INCLUDE_PATH:/root/tfs/tb-common-utils/tbsys/src:/root/tfs/tb-common-utils/tbnet/src
[root@localhost tb-common-utils]# export CPLUS_INCLUDE_PATH

```

##### 3、编译安装TFS：
```py
[root@localhost tfs]# tar xvf tfs-1.4.tar.gz 
[root@localhost tfs]# cd tfs-1.4
[root@localhost tfs-1.4]# sh build.sh init
```