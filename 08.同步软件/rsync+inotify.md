## rsync+inotify实现实时同步

![](https://github.com/ZongYuWang/image/blob/master/rsync1.png)
### 1、防火墙设置：
```ruby

# vim /etc/sysconfig/iptables
#注意规则，不要添加在reject之后

-A INPUT -p tcp -m state --state NEW -m tcp --dport 873 -j ACCEPT 
     
[root@localhost ~]# service iptables restart     
```
### 2、rsync安装：
`rsync有断点续传功能`
#### 2.1 在客户端（接收端）安装rsync软件包：
```ruby

[root@localhost ~]# yum install -y rsync gcc vim 
[root@localhost ~]# useradd rsync -s /sbin/nologin  
#最好就使用root账号，有些目录是对属主和属组有要求的，改为rsync之后，可能影响其他的服务正常运行
[root@localhost ~]# grep rsync /etc/passwd
    rsync:x:500:500::/home/rsync:/sbin/nologin    
[root@localhost ~]# mkdir /inotify_rsync_data  
# 测试创建一个接收文件的目录，可以设置一个真正需要接收到的目录
[root@localhost ~]# chown rsync.rsync /inotify_rsync_data/
[root@localhost ~]# ll -d /inotify_rsync_data/
    drwxr-xr-x. 2 rsync rsync 4096 Apr  1 17:46 /inotify_rsync_data/
```
#### 2.2 配置rsync配置文件：
```ruby
[root@localhost ~]# vim /etc/rsyncd.conf  # 需要自己建立
uid = root(rsync)  #工作中指定用户(需要指定用户)
gid = root(rsync)
use chroot = no #相当于黑洞.出错定位
max connections = 200  #有多少个客户端同时传文件
timeout = 300  #超时时间
pid file = /var/run/rsyncd.pid   #进程号文件
lock file = /var/run/rsync.lock  #日志文件
log file = /var/log/rsyncd.log  #日志文件

[inotify_rsync_data]  
#模块名称随便起名（后面推送端需要用到这个名称），如果该服务器需要同步多个目录，则需要另写多个模块
path = /inotify_rsync_data  #需要同步的目录
ignore errors  #表示出现错误忽略错误
read only = false  #表示网络权限可写(本地控制真正可写)

# 117.78.51.112(例)
[jkshengc]
path = /trade/jkshengc/webapps/ROOT/
ignore errors
read only = false

#这里设置IP或让不让同步
list = false
hosts allow = 172.30.105.0/24 #指定允许的网段
hosts deny = 0.0.0.0/32  #拒绝链接的地址，这个表示没有拒绝的链接
auth users = wangzongyu   #虚拟用户
secrets file = /etc/rsync112_165.password    #虚拟用户的密码文件                   

```
`不同的机器同步修改 /etc/rsync.password的名称 （ /etc/rsync112.password）`

#### 2.3 配置rsync密码文件：
```ruby
[root@localhost ~]# echo "wangzongyu:tradeease" >/etc/rsync112_165.password
#这个名称要跟上面的配置文件中的配置文件名称一致 （/etc/rsync112_165.password）
[root@localhost ~]# cat /etc/rsync112_165.password
#wangzongyu为虚拟用户，tradeease是虚拟用户的密码
[root@localhost ~]# chmod 600 /etc/rsync112_165.password
```
#### 2.4 启动rsync服务：
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
[root@localhost ~]# echo "tradeease" >/etc/rsync112_165.password  # 这里只要写密码即可
# cat /etc/rsync112_165.password
# 不同的机器同步修改 /etc/rsync.password的文件名称

[root@localhost ~]# chmod 600 /etc/rsync112_165.password
[root@localhost ~]# mkdir /inotify_rsync_data
[root@localhost ~]# cd /inotify_rsync_data/
[root@localhost inotify_rsync_data]# echo "hello">test.txt
[root@localhost inotify_rsync_data]# rsync -avz test.txt wangzongyu@172.30.105.112::inotify_rsync_data --password-file=/etc/rsync112_165.password
# inotify_rsync_data是接收端的/etc/rsyncd.conf的配置文件中的[inotify_rsync_data]

[root@localhost ~]# ll /proc/sys/fs/inotify/  #有下面的三个文件表示支持inotify
total 0
-rw-r--r-- 1 root root 0 Aug 27 04:25 max_queued_events
-rw-r--r-- 1 root root 0 Aug 27 04:25 max_user_instances
-rw-r--r-- 1 root root 0 Aug 27 04:25 max_user_watches

```
### 3、inotify在服务端（发送端）安装：
` inotify不需要在客户端(接收端)安装，inotify作用是实时发送变更的文件，文件内容变更也会实时同步到接收端`

#### 3.1 安装inotify软件包：
```ruby
[root@localhost ~]# mkdir /inotify_rsync_soft
[root@localhost ~]# cd /inotify_rsync_soft/
[root@localhost inotify_rsync_soft]# wget http://github.com/downloads/rvoicilas/inotify-tools/inotify-tools-3.14.tar.gz
[root@localhost inotify_rsync_soft]# tar xvf inotify-tools-3.14.tar.gz 
[root@localhost inotify_rsync_soft]# cd inotify-tools-3.14
[root@localhost inotify-tools-3.14]# ./configure --prefix=/usr/local/inotify-3.14 
[root@localhost inotify-tools-3.14]# make && make install && echo ok
```

#### 3.2 配置inotify的启动脚本：
`这个脚本实现了多客户端接收，发送端多路径监控`
```ruby
[root@localhost inotify-tools-3.14]# cd /inotify_rsync_soft/
[root@localhost inotify_rsync_soft]# vim inotify.sh
#!/bin/bash

host="49.4.12.165 IP2 IP3" # 接收端的多个IP地址之间用空格隔开
src=/tradeease/apache-tomcat-7.0.64/webapps/ROOT/  # 本地监控目录
des=jkshengc # 接收端rsync中设置的模块名
user=tradeease #rsync服务的虚拟用户
rsync_passwordfile=/etc/rsync112_165.password   # 本地调用rsync服务的密码文件
inotify_home=/tradeease/inotify/inotify  # inotify的安装目录(这个目录下面有bin目录)
rsync_exclude='/tradeease/inotify/rsync_exclude.list'  # 这个文件中的这些文件不会同步(除了这里面的都会同步，是相对于上面的src下的相对文件)
rsync_log='/tradeease/inotify/rsync.log'


${inotify_home}/bin/inotifywait -mrq --timefmt '%d/%m/%y %H:%M' --format '%T %w%f%e' -e modify,delete,create,attrib $src/ | while read files
do
for ip in $host
do
/tradeease/rsync/rsync/bin/rsync -vzrtopg --exclude-from=${rsync_exclude}  --progress --password-file=${rsync_passwordfile} $src $user@$ip::$des
echo "${files} was rsynced" >>${rsync_log} 2>&1
done
done

#要修改脚本中的rsync_passfile=/etc/rsync.password中的rsync.password的文件名称名称

```

- rsync_exclude.list文件内容：
```ruby
# cat rsync_exclude.list 
admin/
adminthemes/
b2b2c/
cms/
commons/
config/
core/
docs/
editor/
excel/
font/
html/
install/
license/
logs/
META-INF/
products/
shop/
test/
themes/
WEB-INF/
favicon.ico
robots.txt
WebContent.zip

```

#### 3.3 启动inotify：
```ruby
[root@localhost ~]# nohup sh inotify.sh &
```

```ruby
[root@whj-centos63-64 inotify]# ps -ef | grep inotify

root      74015  74014 32 14:39 pts/2    00:00:02 /tradeease/inotify/inotify/bin/inotifywait -mrq --timefmt %d/%m/%y %H:%M --format %T %w%f%e -e modify,delete,create,attrib /tradeease/apache-tomcat-7.0.64/webapps/ROOT//
root      74016  74014  0 14:39 pts/2    00:00:00 sh rsync_inotify.sh

root      74019  74016 14 14:39 pts/2    00:00:00 /tradeease/rsync/rsync/bin/rsync -vzrtopg --exclude-from=/tradeease/inotify/rsync_exclude.list --progress --password-file=/etc/rsync112_165.password /tradeease/apache-tomcat-7.0.64/webapps/ROOT/ tradeease@49.4.12.165::jkshengc

root      74022  68301  0 14:39 pts/2    00:00:00 grep inotify

```


- inotify的inotifywait命令常用参数详解：
```ruby
# cd /usr/local/inotify-3.14/
# ./bin/inotifywait --help
```
| 参数  |  英文解释  | 中文解释  |
|-------|-----------|----------|
|-r|--recursive  | Watch directories recursively| 递归查询目录
|-q|--quiet  |    Print less (only print events)|打印监控事件的信息
|-m|--monitor |  Keep listening for events forever |Without this option, inotifywait will exit after one  event is received |始终保持事件监听状态
|--excludei  | Like --exclude but case insensitive |排除文件或目录时，不区分大小写。
|--timefmt |strftime-compatible format string for use with %T in --format string |指定时间输出的格式
|--format  | Print using a specified printf-like format string; read the man page for more details |打印使用指定的输出类似格式字符串
|-e|--event [ -e|--event ... ] |Listen for specific event(s).  If omitted, all events are  listened for |通过此参数可以指定需要监控的事件

|Events|中文解释|英文解释|
|------|-------|-------|
|access    |       file or directory contents were read      |文件或目录被读取
|modify     |      file or directory contents were written    |文件或目录内容被修改
|attrib    |       file or directory attributes changed      |文件或目录属性被改变
|close            file or directory closed, regardless of read/write mode    |文件或目录封闭，无论读/写模式
|open       |     file or directory opened                    |文件或目录被打开
|moved_to   |     file or directory moved to watched directory    |文件或目录被移动至另外一个目录
|move       |     file or directory moved to or from watched directory    |文件或目录被移动另一个目录或从另一个目录移动至当前目录
|create     |      file or directory created within watched directory     |文件或目录被创建在当前目录
|delete     |      file or directory deleted within watched directory     |文件或目录被删除
|unmount    |     file system containing file or directory unmounted  |文件系统被卸载
