####  安装Netcat：
```ruby
[root@localhost netcat]# tar xvf netcat-0.7.1.tar.gz -C /usr/local
[root@localhost netcat]# cd /usr/local/
[root@localhost local]# mv netcat-0.7.1/ netcat
[root@localhost local]# cd /usr/local/netcat/
[root@localhost netcat]# ./configure && make && make install

```
#### 配置Netcat环境变量：
```ruby
[root@localhost ~]# vim /etc/profile
    # set  netcat path
    export NETCAT_HOME=/usr/local/netcat
    export PATH=$PATH:$NETCAT_HOME/bin
[root@localhost ~]# source /etc/profile

```
#### Netcat命令基本使用（端口扫描）：

**1、测试Netcat-nc命令：**
```ruby
[root@localhost ~]# nc --help  # 获取nc的参数

```

#### 2、使用Netcat进行Banner获取： ####
服务Banner标识正在运行的服务，通常也是版本号。Banner抓取是一种在开放端口上检索有关特定服务的信息的技术，可用于进行漏洞评估的渗透测试。当使用Netcat进行Banner抓取时，您实际上将与指定端口上的指定主机进行原始连接。当Banner可用时，它会打印到控制台

```ruby
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

远程主机：
[root@localhost ~]# nc -v 172.30.105.116 22
172.30.105.116 22 (ssh) open
SSH-2.0-OpenSSH_6.6.1

```

#### 将文件从一个系统复制到另外一个系统：
```ruby
server1:172.30.105.115
server2:172.30.105.116

将server1中的ncfile.tar.gz复制到server2中：
server2：
[root@localhost ~]# nc -lp 1234 > ncfile.tar.gz
【说明】1234是未占用的端口，可以随意设置未占用的端口号，server2将等待端口1234上的文件ncfile.tar.gz

server1：
[root@localhost ~]# nc -w 1 172.30.105.116 1234 < ncfile.tar.gz 
```


#### 克隆硬盘和分区：

操作与上面的拷贝是雷同的，只需要由dd获得硬盘或分区的数据，然后传输即可。
克隆硬盘或分区的操作，不应在已经mount的的系统上进行。当然，要克隆的分区必须在目标系统上卸载，所以需要使用安装光盘引导后，进入拯救模式（或使用Knoppix工具光盘）启动系统后，
在server2(172.30.105.116)上进行类似的监听动作：
```ruby
[root@localhost ~]# nc -l -p 1234 | dd of=/dev/sda

server1(172.30.105.115)上执行传输，即可完成从server1克隆sda硬盘到server2的任务：
[root@localhost ~]# dd if=/dev/sda | nc 172.30.105.116 1234

```

#### 服务网页：
可以使用netcat作为Web服务器
```ruby
[root@localhost ~]# vim somepage.html 
123

[root@localhost ~]# while true; do nc -l -p 80 < somepage.html; done
【说明】用浏览器请求somepage页面，在nc服务器端会输出下面信息：
GET /somepage.html HTTP/1.1
Host: 172.30.105.115
Connection: keep-alive
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.90 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.8

按ctrl+c会输出：
^CGET /favicon.ico HTTP/1.1
Host: 172.30.105.115
Connection: keep-alive
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.90 Safari/537.36
Accept: image/webp,image/apng,image/*,*/*;q=0.8
Referer: http://172.30.105.115/somepage.html
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.8

同时浏览器会输出：
http://172.30.105.115/somepage.html
123
```

#### 聊天：
甚至可以在命令行中使用netcat从一个系统到另一个系统进行聊天
```ruby
在server2(172.30.105.116)上执行：
[root@localhost ~]# nc -lp 1234  #  server2会等到server1在端口1234上连接

在server1(172.30.105.115)上执行：
[root@localhost ~]# nc 172.30.105.116 1234

现在可以在两个系统上键入消息，然后按ENTER键 ，它们将显示在另一个系统上。 要关闭聊天，请在两个系统上按CTRL + C。

```
#### 欺骗HTTP头：
```ruby
其实就是自己编写一些模拟客户端的的信息(也就是HTTP请求头部)：
GET / HTTP/1.1
Host: nctest
Referrer: mypage.com
User-Agent: my-nc

[root@localhost ~]# nc www.linuxfly.org 80
输入上面的模拟http头信息，然后按两次Enter，就会输出下面的内容(模拟假的HTTP头部，正常请求了网页)：
HTTP/1.1 200 OK
Date: Fri, 13 Oct 2017 03:36:38 GMT
Server: Apache/2.2.15 (CentOS)
X-Powered-By: PHP/5.3.3
Expires: Mon, 26 Jul 1997 05:00:00 GMT
Last-Modified: Fri, 13 Oct 2017 03:36:41 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Connection: close
Transfer-Encoding: chunked
Content-Type: text/html; charset=utf-8
.....
