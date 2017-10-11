**安装Netcat：**
```py
[root@localhost netcat]# tar xvf netcat-0.7.1.tar.gz 
[root@localhost netcat]# tar xvf netcat-0.7.1.tar.gz -C /usr/local
[root@localhost netcat]# cd /usr/local/
[root@localhost local]# mv netcat-0.7.1/ netcat
[root@localhost local]# cd /usr/local/netcat/
[root@localhost netcat]# ./configure && make && make install

```
**配置Netcat环境变量：**
```py
[root@localhost ~]# vim /etc/profile
    # set  netcat path
    export NETCAT_HOME=/usr/local/netcat
    export PATH=$PATH:$NETCAT_HOME/bin
[root@localhost ~]# source /etc/profile

```
**测试Netcat-nc命令：**
```py
[root@localhost ~]# nc --help

```

#### 使用Netcat进行Banner获取： ####
服务Banner标识正在运行的服务，通常也是版本号。Banner抓取是一种在开放端口上检索有关特定服务的信息的技术，可用于进行漏洞评估的渗透测试。当使用Netcat进行Banner抓取时，您实际上将与指定端口上的指定主机进行原始连接。当Banner可用时，它会打印到控制台

```py
[root@localhost ~]# nc -z -v -n 127.0.0.1 21-25  
127.0.0.1 22 (ssh) open
127.0.0.1 25 (smtp) open

可以运行在TCP或者UDP模式，默认是TCP，-u参数调整为udp.
-z 参数告诉netcat使用0 IO,连接成功后立即关闭连接，不进行数据交换
-v 参数指使用冗余选项，即详细输出
-n 参数告诉netcat不使用DNS解析，即仅仅是一串IP数字，一般如果后面是跟IP数字的话，就带上-n参数；跟着是域名的话，就不带-n参数。

[root@localhost ~]# nc -z -v -n localhost 21-25  #加了-n参数，就不会解析localhost，导致报错
Error: Couldn't resolve host "localhost"

得到开放端口的banner信息：
[root@localhost ~]# nc -v 127.0.0.1 22
localhost [127.0.0.1] 22 (ssh) open
SSH-2.0-OpenSSH_5.3

```