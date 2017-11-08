## rsync+inotify实现实时同步

![](https://github.com/ZongYuWang/image/blob/master/rsync1.png)
### 一、防火墙设置：
```ruby
# vim /etc/sysconfig/iptables

192.168.1.106：
      -A INPUT -p tcp -m state --state NEW -m tcp --dport 873 -j ACCEPT
      -A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
192.168.1.57：
      -A INPUT -p tcp -m state --state NEW -m tcp --dport 873 -j ACCEPT
      -A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
192.168.1.139：
      -A INPUT -p tcp -m state --state NEW -m tcp --dport 873 -j ACCEPT
      -A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
```
### 二、rsync安装：
#### 2.1、rsync在客户端（接收端）安装：
```ruby

[root@localhost ~]# yum install -y rsync gcc vim 
[root@localhost ~]# useradd rsync -s /sbin/nologin 
[root@localhost ~]# grep rsync /etc/passwd
    rsync:x:500:500::/home/rsync:/sbin/nologin    
[root@localhost ~]# mkdir /inotify_rsync_data
[root@localhost ~]# chown rsync.rsync /inotify_rsync_data/
[root@localhost ~]# ll -d /inotify_rsync_data/
    drwxr-xr-x. 2 rsync rsync 4096 Apr  1 17:46 /inotify_rsync_data/
    
[root@localhost ~]# vim /etc/rsyncd.conf

uid = rsync  #工作中指定用户(需要指定用户)
gid = rsync
use chroot = no #相当于黑洞.出错定位
max connections = 200  #有多少个客户端同时传文件
timeout = 300  #超时时间
pid file = /var/run/rsyncd.pid   #进程号文件
lock file = /var/run/rsync.lock  #日志文件
log file = /var/log/rsyncd.log  #日志文件

[inotify_rsync_data]  #模块名称随便起名，后面推送端需要用到这个名称
path = /inotify_rsync_data  #需要同步的目录
ignore errors  #表示出现错误忽略错误
read only = false  #表示网络权限可写(本地控制真正可写)

#这里设置IP或让不让同步
list = false
hosts allow = 172.30.105.0/24 #指定允许的网段
hosts deny = 0.0.0.0/32  #拒绝链接的地址，一下表示没有拒绝的链接
auth users = wangzongyu   #虚拟用户
secrets file = /etc/rsync.password    #虚拟用户的密码文件                   

```
`【说明】不同的机器同步修改 /etc/rsync.password的名称 （ /etc/rsync112.password）`

```ruby
[root@localhost ~]# echo "wangzongyu:tradeease" >/etc/rsync.password
#【说明】这个名称要跟上面的配置文件中的配置文件名称一致 （/etc/rsync112.password）
[root@localhost ~]# cat /etc/rsync.password
#【说明】wangzongyu为虚拟用户，tradeease是虚拟用户的密码
[root@localhost ~]# chmod 600 /etc/rsync.password 
```
- 启动rsync服务
```ruby
[root@localhost ~]# rsync --daemon  

[root@localhost ~]# ps -ef | grep rsync
root     15390 15227  0 06:12 pts/0    00:00:00 grep rsync
root     25432     1  0 Sep05 ?        00:00:09 rsync --daemon

[root@localhost ~]# netstat -tlnp | grep rsync
tcp        0      0 0.0.0.0:873                 0.0.0.0:*                   LISTEN      25432/rsync         
tcp        0      0 :::873                      :::*                        LISTEN      25432/rsync
```

#### 2.2、rsync在服务端（发送端）安装：
- 发送端测试发送文件
```ruby
[root@localhost ~]# yum install -y rsync gcc vim 
[root@localhost ~]# echo "tradeease" >/etc/rsync.password  # 这里只要写密码即可
# cat /etc/rsync.password
【说明】不同的机器同步修改 /etc/rsync.password的名称 （ /etc/rsync112.password）

[root@localhost ~]# chmod 600 /etc/rsync.password
[root@localhost ~]# mkdir /inotify_rsync_data
[root@localhost ~]# cd /inotify_rsync_data/
[root@localhost inotify_rsync_data]# echo "hello">test.txt
[root@localhost inotify_rsync_data]# rsync -avz test.txt wangzongyu@172.30.105.112::inotify_rsync_data --password-file=/etc/rsync.password
【说明】:inotify_rsync_data是接收端的/etc/rsyncd.conf的配置文件中的[inotify_rsync_data]

[root@localhost ~]# ll /proc/sys/fs/inotify/  #有下面的三个文件表示支持inotify
total 0
-rw-r--r-- 1 root root 0 Aug 27 04:25 max_queued_events
-rw-r--r-- 1 root root 0 Aug 27 04:25 max_user_instances
-rw-r--r-- 1 root root 0 Aug 27 04:25 max_user_watches

```
### 三、inotify在服务端（发送端）安装：
` inotify不需要在客户端(接收端)安装，inotify作用是实时发送变更的文件，文件内容变更也会实时同步到接收端`
```ruby
[root@localhost ~]# mkdir /inotify_rsync_soft
[root@localhost ~]# cd /inotify_rsync_soft/
[root@localhost inotify_rsync_soft]# wget http://github.com/downloads/rvoicilas/inotify-tools/inotify-tools-3.14.tar.gz
[root@localhost inotify_rsync_soft]# tar xvf inotify-tools-3.14.tar.gz 
[root@localhost inotify_rsync_soft]# cd inotify-tools-3.14
[root@localhost inotify-tools-3.14]# ./configure --prefix=/usr/local/inotify-3.14 
[root@localhost inotify-tools-3.14]# make && make install && echo ok

[root@localhost inotify-tools-3.14]# cd /inotify_rsync_soft/
[root@localhost inotify_rsync_soft]# vim inotify.sh
#!/bin/bash
#para
host01=172.30.105.112  #inotify-slave的ip地址
src=/inotify_rsync_data        #本地监控的目录
dst=inotify_rsync_data         #inotify-slave的rsync服务的模块名
user=wangzongyu      #inotify-slave的rsync服务的虚拟用户
rsync_passfile=/etc/rsync.password   #本地调用rsync服务的密码文件
inotify_home=/usr/local/inotify-3.14    #inotify的安装目录
#judge
if [ ! -e "$src" ] \
|| [ ! -e "${rsync_passfile}" ] \
|| [ ! -e "${inotify_home}/bin/inotifywait" ] \
|| [ ! -e "/usr/bin/rsync" ];
then
echo "Check File and Folder"
exit 9
fi
${inotify_home}/bin/inotifywait -mrq --timefmt '%d/%m/%y %H:%M' --format '%T %w%f' -e close_write,delete,create,attrib $src \
| while read file
do
#  rsync -avzP --delete --timeout=100 --password-file=${rsync_passfile} $src $user@$host01::$dst >/dev/null 2>&1
cd $src && rsync -aruz -R --delete ./  --timeout=100 $user@$host01::$dst --password-file=${rsync_passfile} >/dev/null 2>&1
done
exit 0

【说明】要修改脚本中的rsync_passfile=/etc/rsync.password中的rsync.password的名称（rsync112.password）

172.30.105.112的配置文件展示：
[root@localhost etc]# ls | grep rsync
rsync112.password
rsync114.password
rsyncd.conf

```
- 启动inotify
```ruby
[root@localhost ~]# nohup sh inotify.sh &
```

- inotify的inotifywait命令常用参数详解：
```ruby
# cd /usr/local/inotify-3.14/
# ./bin/inotifywait --help

-r|--recursive   Watch directories recursively. #递归查询目录
-q|--quiet      Print less (only print events). #打印监控事件的信息
-m|--monitor   Keep listening for events forever.  Without this option, inotifywait will exit after one  event is received.        #始终保持事件监听状态
--excludei   Like --exclude but case insensitive.    #排除文件或目录时，不区分大小写。
--timefmt strftime-compatible format string for use with %T in --format string. #指定时间输出的格式
--format   Print using a specified printf-like format string; read the man page for more details.#打印使用指定的输出类似格式字符串
-e|--event [ -e|--event ... ] Listen for specific event(s).  If omitted, all events are  listened for.   
#通过此参数可以指定需要监控的事件，如下所示:
Events：
access           file or directory contents were read       #文件或目录被读取。
modify           file or directory contents were written    #文件或目录内容被修改。
attrib            file or directory attributes changed      #文件或目录属性被改变。
close            file or directory closed, regardless of read/write mode    #文件或目录封闭，无论读/写模式。
open            file or directory opened                    #文件或目录被打开。
moved_to        file or directory moved to watched directory    #文件或目录被移动至另外一个目录。
move            file or directory moved to or from watched directory    #文件或目录被移动另一个目录或从另一个目录移动至当前目录。
create           file or directory created within watched directory     #文件或目录被创建在当前目录
delete           file or directory deleted within watched directory     #文件或目录被删除
unmount         file system containing file or directory unmounted  #文件系统被卸载
```