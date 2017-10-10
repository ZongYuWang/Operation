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