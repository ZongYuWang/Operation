## MySQL基础概念(小知识点)

### 1、MySQL启动基本原理说明：
/etc/init.d/mysqld(mysql.server)是一个shell启动脚本(/etc/rc.d/init.d/mysqld是部署中由mysql.server重命名而来)，mysql.server程序主要是会用到两个程序和一个函数，分别是my_print_defaults、myslqd_safe和parse_server_arguments     
- my_print_defaults:读取my.cnf配置文件，输出参数传递给parse_server_arguments，该程序只读my.cnf中[mysqld]中的参数；      
- parse_server_arguments：该函数处理my_print_defaults传递过来的参数赋值给--basedir、--datadir、--pid-file、--server-startup-timeout；
- myslqd_safe:mysqld_safe程序调用mysqld程序来启动mysql服务；

```ruby
# mysql.server内容：

$bindir/mysqld_safe --datadir="$datadir" --pid-file="$mysqld_pid_file_path" $other_args &
      wait_for_ready; return_value=$?

```
```ruby
#parse_server_arguments内容：

parse_server_arguments() {
  for arg do
    case "$arg" in
      --basedir=*)  basedir=`echo "$arg" | sed -e 's/^[^=]*=//'`
                    bindir="$basedir/bin"
                    if test -z "$datadir_set"; then
                      datadir="$basedir/data"
                    fi
                    sbindir="$basedir/sbin"
                    if test -f "$basedir/bin/mysqld"
                    then
                      libexecdir="$basedir/bin"
                    else
                      libexecdir="$basedir/libexec"
                    fi
                    libexecdir="$basedir/libexec"
        ;;
      --datadir=*)  datadir=`echo "$arg" | sed -e 's/^[^=]*=//'`
                    datadir_set=1
        ;;
      --log-basename=*|--hostname=*|--loose-log-basename=*)
        mysqld_pid_file_path=`echo "$arg.pid" | sed -e 's/^[^=]*=//'`
        ;;
      --pid-file=*) mysqld_pid_file_path=`echo "$arg" | sed -e 's/^[^=]*=//'` ;;
      --service-startup-timeout=*) service_startup_timeout=`echo "$arg" | sed -e 's/^[^=]*=//'` ;;
    esac
  done
}

```

```ruby

#pare_arguments函数内容：
#该函数是mysqld_safe程序中用来处理参数的一个函数,从下面的代码中可以了解到mysqld_safe主要处理哪些参数。

for arg do
    # the parameter after "=", or the whole $arg if no match
    val=`echo "$arg" | sed -e 's;^--[^=]*=;;'`
    # what's before "=", or the whole $arg if no match
    optname=`echo "$arg" | sed -e 's/^\(--[^=]*\)=.*$/\1/'`
    # replace "_" by "-" ; mysqld_safe must accept "_" like mysqld does.
    optname_subst=`echo "$optname" | sed 's/_/-/g'`
    arg=`echo $arg | sed "s/^$optname/$optname_subst/"`
    case "$arg" in
      # these get passed explicitly to mysqld
      --basedir=*) MY_BASEDIR_VERSION="$val" ;;
      --datadir=*) DATADIR="$val" ;;
      --pid-file=*) pid_file="$val" ;;
      --plugin-dir=*) PLUGIN_DIR="$val" ;;
      --user=*) user="$val"; SET_USER=1 ;;

      # these might have been set in a [mysqld_safe] section of my.cnf
      # they are added to mysqld command line to override settings from my.cnf
      --log-error=*) err_log="$val" ;;
      --port=*) mysql_tcp_port="$val" ;;
      --socket=*) mysql_unix_port="$val" ;;

      # mysqld_safe-specific options - must be set in my.cnf ([mysqld_safe])!
      --core-file-size=*) core_file_size="$val" ;;
      --ledir=*) ledir="$val" ;;
      --malloc-lib=*) set_malloc_lib "$val" ;;
      --mysqld=*) MYSQLD="$val" ;;
      --mysqld-version=*)
        if test -n "$val"
        then
          MYSQLD="mysqld-$val"
          PLUGIN_VARIANT="/$val"
        else
          MYSQLD="mysqld"
        fi
        ;;
      --nice=*) niceness="$val" ;;
      --open-files-limit=*) open_files="$val" ;;
      --open_files_limit=*) open_files="$val" ;;
      --skip-kill-mysqld*) KILL_MYSQLD=0 ;;
      --syslog) want_syslog=1 ;;
      --skip-syslog) want_syslog=0 ;;
      --syslog-tag=*) syslog_tag="$val" ;;
      --timezone=*) TZ="$val"; export TZ; ;;

      --help) usage ;;

      *)

```

查看mysql进程信息可以看到通过mysql.server启动首先会对参数--datedir和--pid-file赋值，这两个参数是从my.cnf文件[mysqld]部分中读取来的，而且这两个参数的值不会受到mysqld_safe程序中的参数赋值给覆盖。但是在my.cnf中其它的参数值如果[mysqld]和[mysqld_safe]相同的话就以mysqld_safe为主；
```ruby
# 查看mysql进程：

[root@mysql ~]# ps -ef | grep mysql
root     16676     1  0 Jan05 ?        00:00:00 /bin/sh /usr/local/mysql/bin/mysqld_safe --datadir=/mydata/data --pid-file=/mydata/data/mysql.pid
mysql    17006 16676  0 Jan05 ?        00:00:07 /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/mydata/data --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=/mydata/data/mysql.err --pid-file=/mydata/data/mysql.pid --socket=/tmp/mysql.sock --port=3306
root     17184 17146  0 05:31 pts/0    00:00:00 grep mysql

```
在现在的新版本中不建议在[mysqld_safe]中进行参数的配置，对应多实例的服务器在启动的时候可以通过mysqld_safe来指定不同实例的路径和配置文件进行启动，需要用到----defaults-file、--ledir两个参数进行启动,从启动的代码可以看出mysql的启动要用到的两个关键参数--datadir --pid-file,所以为什么经常会在启动和关闭mysql的时候提示找不到pid了

```ruby
#mysqld

/usr/local/mysql/bin/mysqld
直接运行mysqld程序也是可以启动mysql服务，mysqld会使用默认的配置进行启动，对于多实例的mysql使用这种方法就不好实现

```

### 2、MySQL的关闭方式：
#### 2.1 mysqladmin方式：
```ruby
[root@mysql ~]# mysqladmin -u root -p'wangzongyu' shutdown
```
#### 2.2 自带的脚本关闭方式：
```ruby
[root@mysql ~]# /etc/init.d/mysqld stop

```
#### 2.3 使用kill信号方式：
```ruby
[root@mysql ~]# kill pid

```

### 3、MySQL的编码字符集问题:
`字符集的不一致是数据库中文内容乱码的罪魁祸首`
`Server version: 5.5.56`
```ruby
MariaDB [(none)]> create database babyshen;

MariaDB [(none)]> show create database babyshen\G;
*************************** 1. row ***************************
       Database: babyshen
Create Database: CREATE DATABASE `babyshen` /*!40100 DEFAULT CHARACTER SET utf8 */
// 因为在编译安装mariadb时指定了编码类型，所以创建数据库使用的编码是utf8

如果编译的时候指定了特定的字符集，那么以后创建相应的数据就不需要再指定字符集了
>  -DDEFAULT_CHARSET=utf8 \
>  -DDEFAULT_COLLATION=utf8_general_ci \

```
#### 3.1 建立一个名为babyshen_gbk的GBK字符集数据库
```ruby
MariaDB [(none)]> create database babyshen_gbk DEFAULT CHARACTER SET gbk COLLATE gbk_chinese_ci;

MariaDB [(none)]> show create database babyshen_gbk;
+--------------+----------------------------------------------------------------------+
| Database     | Create Database                                                      |
+--------------+----------------------------------------------------------------------+
| babyshen_gbk | CREATE DATABASE `babyshen_gbk` /*!40100 DEFAULT CHARACTER SET gbk */ |
+--------------+----------------------------------------------------------------------+
1 row in set (0.00 sec)
```
#### 3.2 建立一个名为babyshen_utf8的UTF8字符集数据库
```ruby

MariaDB [(none)]> create database babyshen_utf8 DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;

MariaDB [(none)]> show create database babyshen_utf8;
+---------------+------------------------------------------------------------------------+
| Database      | Create Database                                                        |
+---------------+------------------------------------------------------------------------+
| babyshen_utf8 | CREATE DATABASE `babyshen_utf8` /*!40100 DEFAULT CHARACTER SET utf8 */ |
+---------------+------------------------------------------------------------------------+
1 row in set (0.00 sec)

```

http://blog.51cto.com/oldboy/1431161
http://blog.51cto.com/oldboy/1431172
https://www.cnblogs.com/peida/archive/2012/12/20/2825837.html
http://blog.51cto.com/oldboy/909696