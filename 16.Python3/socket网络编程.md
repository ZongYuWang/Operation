## Socket网络编程

### 使用Socket模拟接打电话
- 电话发送与接收(固定发一句，收一句)
socket_client代码：
```ruby
import socket

client = socket.socket() # 声明socket类型，同时生成socket连接对象
client.connect(('localhost',6969))

client.send(b"Hello World") # python2中可以发送字符串或者是字节，但是在python3中只能发送字节
data = client.recv(1024) # 客户端接收1024字节数据=1kb
print("recv:",data)

client.close()
```
socket_server代码：
```ruby
import socket
server = socket.socket()
server.bind(('localhost',6969))
server.listen(5)                   # 表示最多可以5个客户端挂起的连接(也就是最大的排队数)

print("我要开始等待电话了")
conn,addr = server.accept()
print(conn,addr)
print("电话来了！")

data =server.recv(1024)
print("recv:",data)
server.send(data.upper())

server.close()


# 报错如下：
我要开始等待电话了
<socket.socket fd=344, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 6969), raddr=('127.0.0.1', 56153)> ('127.0.0.1', 56153)
电话来了！
Traceback (most recent call last):
  File "E:/PycharmProjects/untitled/study/day7/socket_server.py", line 15, in <module>
    data =server.recv(1024)
OSError: [WinError 10057] 由于套接字没有连接并且(当使用一个 sendto 调用发送数据报套接字时)没有提供地址，发送或接收数据的请求没有被接受。
```
`通过上面得知conn就是客户端连过来而在服务器端为其生成的一个连接实例 `

socket_server修改代码如下：
```ruby
import socket
server = socket.socket()
server.bind(('localhost',6969))
server.listen(5)

print("我要开始等待电话了")
conn,addr = server.accept()
print(conn,addr)
print("电话来了！")

data =conn.recv(1024)
print("recv:",data)
conn.send(data.upper())

server.close()

# 服务端输出：
我要开始等待电话了
<socket.socket fd=344, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 6969), raddr=('127.0.0.1', 59516)> ('127.0.0.1', 59516)
电话来了！
recv: b'Hello World'


# 客户端输出：
recv: b'HELLO WORLD'

```
- 传输中文-socket_client代码修改：
```ruby
import socket

client = socket.socket()
client.connect(('localhost',6969))

client.send("世界你好".encode("utf-8"))
data = client.recv(1024)
print("recv:",data.decode())

client.close()

# 服务端输出：
我要开始等待电话了
<socket.socket fd=344, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 6969), raddr=('127.0.0.1', 59795)> ('127.0.0.1', 59795)
电话来了！
recv: b'\xe4\xb8\x96\xe7\x95\x8c\xe4\xbd\xa0\xe5\xa5\xbd'

# 客户端输出：
recv: 世界你好
```
- 用户持续输入：
socket_client代码：
```ruby
import socket
client = socket.socket()
client.connect(('localhost',6969))

while True:
    msg = input(">>:").strip()

    client.send(msg.encode("utf-8"))
    data = client.recv(1024)
    print("recv:",data.decode())

client.close()
```
socket_server代码：
```ruby
import socket
server = socket.socket()
server.bind(('localhost',6969))
server.listen(5)

print("我要开始等待电话了")
while True:
    conn,addr = server.accept()
    print(conn,addr)
    print("电话来了！")
    data =conn.recv(1024)
    print("recv:",data)
    conn.send(data.upper())

server.close()

# 服务端输出：
我要开始等待电话了
<socket.socket fd=344, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 6969), raddr=('127.0.0.1', 60574)> ('127.0.0.1', 60574)
电话来了！
recv: b'\xe6\xb1\xaa'

客户端输出：
>>:汪
recv: 汪
>>:宗     # 卡住了

```
`
可以再开启一个客户端，但是也只能发送一条数据，第二条就卡主了，实现了多客户端会话，但是每个客户端只能发送一句话`   

继续修改代码：
socket_server代码修改：
```ruby
import socket
server = socket.socket()
server.bind(('localhost',6969))
server.listen(5)

print("我要开始等待电话了")
conn,addr = server.accept()
print(conn,addr)
print("电话来了！")
while True:
    data =conn.recv(1024)
    print("recv:",data)
    conn.send(data.upper())

server.close()

# 服务端输出：
我要开始等待电话了
<socket.socket fd=344, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 6969), raddr=('127.0.0.1', 60747)> ('127.0.0.1', 60747)
电话来了！
recv: b'\xe5\x93\x88\xe5\x93\x88'
recv: b'\xe5\x91\xb5\xe5\x91\xb5'
recv: b'\xe5\x97\xaf\xe5\x97\xaf'

# 客户端输出：
>>:哈哈
recv: 哈哈
>>:呵呵
recv: 呵呵
>>:嗯嗯
recv: 嗯嗯
>>:

```
`但是连接的第二个客户端还是不能实现联系输入,将socket-client断开之后，服务器也就结束了`
```ruby
我要开始等待电话了
<socket.socket fd=344, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 6969), raddr=('127.0.0.1', 60747)> ('127.0.0.1', 60747)
电话来了！
recv: b'\xe5\x93\x88\xe5\x93\x88'
recv: b'\xe5\x91\xb5\xe5\x91\xb5'
recv: b'\xe5\x97\xaf\xe5\x97\xaf'
Traceback (most recent call last):
  File "E:/PycharmProjects/untitled/study/day7/socket_server.py", line 15, in <module>
    data =conn.recv(1024)
ConnectionResetError: [WinError 10054] 远程主机强迫关闭了一个现有的连接。

```
`上面相同的代码在windows上可能有问题，在linux上测试`

运行socket-client:
```ruby
[root@localhost ~]# python socket_client.py 
>>:哈哈
recv: 哈哈
>>:呵呵
recv: 呵呵
>>:嗯嗯
recv: 嗯嗯
>>:^CTraceback (most recent call last):  # ctrl+c 结束运行
  File "socket_client.py", line 6, in <module>
    msg = input(">>:").strip()
KeyboardInterrupt

```

运行socket-server:
```ruby
[root@localhost ~]# python socket_server.py 
我要开始等待电话了
<socket.socket fd=4, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 6969), raddr=('127.0.0.1', 43616)> ('127.0.0.1', 43616)
电话来了！
recv: b'\xe5\x93\x88\xe5\x93\x88'
recv: b'\xe5\x91\xb5\xe5\x91\xb5'
recv: b'\xe5\x97\xaf\xe5\x97\xaf'
recv: b''
recv: b''
recv: b''
recv: b''
recv: b''    # 进入了死循环
recv: b''
recv: b''
recv: b''
recv: b''
recv: b''
^Crecv: b''
Traceback (most recent call last):
  File "socket_server.py", line 12, in <module>
    print("recv:",data)
KeyboardInterrupt

```
添加一个计数功能看一下什么时候开始断开：

socket-server代码修改：
```ruby

import socket
server = socket.socket()
server.bind(('localhost',6969))
server.listen(5)

print("我要开始等待电话了")
conn,addr = server.accept()
print(conn,addr)
print("电话来了！")

count = 0
while True:
    data =conn.recv(1024)
    print("recv:",data)
    conn.send(data.upper())
    count += 1
    if count > 10:break
server.close()

```
socket-client运行：
```ruby
[root@localhost ~]# python socket_client.py 
>>:haha
recv: HAHA
>>:hehe
recv: HEHE
>>:enen
recv: ENEN
>>:MY
recv: MY
>>:NAME
recv: NAME
>>:Is
recv: IS
>>:^CTraceback (most recent call last):
  File "socket_client.py", line 6, in <module>
    msg = input(">>:").strip()
KeyboardInterrupt

```
socket-server运行：
```ruby
[root@localhost ~]# python socket_server.py 
我要开始等待电话了
<socket.socket fd=4, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 6969), raddr=('127.0.0.1', 43618)> ('127.0.0.1', 43618)
电话来了！
recv: b'haha'
recv: b'hehe'
recv: b'enen'
recv: b'MY'
recv: b'NAME'
recv: b'Is'
recv: b''
recv: b''
recv: b''
recv: b''
recv: b''

```
`当客户端断开，服务端不能进入死循环`
```ruby
import socket
server = socket.socket()
server.bind(('localhost',6969))
server.listen(5)

print("我要开始等待电话了")
conn,addr = server.accept()
print(conn,addr)
print("电话来了！")

# count = 0
while True:
    data =conn.recv(1024)
    print("recv:",data)
    if not data:
        print("client has lost....")
        break
    conn.send(data.upper())
#    count += 1
#    if count > 10:break
server.close()

```
socket-server运行：
```ruby
[root@localhost ~]# python socket_server.py 
我要开始等待电话了
<socket.socket fd=4, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 6969), raddr=('127.0.0.1', 43620)> ('127.0.0.1', 43620)
电话来了！
recv: b'haha'
recv: b'hehe'
recv: b'enen'
recv: b''
client has lost....

```
socket-client运行:
```ruby
[root@localhost ~]# python socket_client.py 
>>:haha
recv: HAHA
>>:hehe
recv: HEHE
>>:enen
recv: ENEN
>>:^CTraceback (most recent call last):
  File "socket_client.py", line 6, in <module>
    msg = input(">>:").strip()
KeyboardInterrupt

```
`服务端要不断的监听，不能因为一个客户端的断开就影响服务端停止其他客户端的监听`
```ruby

```
socket-server运行：
```ruby
[root@localhost ~]# python socket_server.py 
我要开始等待电话了
<socket.socket fd=4, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 6969), raddr=('127.0.0.1', 43626)> ('127.0.0.1', 43626)
电话来了！
recv: b'haha'
recv: b'enen'
recv: b'hehe'
recv: b''
client has lost....  # client1断开，client2开始接入
<socket.socket fd=5, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 6969), raddr=('127.0.0.1', 43628)> ('127.0.0.1', 43628)
电话来了！
recv: b'hello'

```
socket-client1运行：
```ruby
[root@localhost ~]# python socket_client.py 
>>:haha
recv: HAHA
>>:enen
recv: ENEN
>>:hehe
recv: HEHE
>>:^CTraceback (most recent call last):  # ctrl+c终止
  File "socket_client.py", line 6, in <module>
    msg = input(">>:").strip()
KeyboardInterrupt

```
socket-client2运行：
```ruby
[root@localhost ~]# python socket_client.py 
>>:hello

```
`还存在一个问题，客户端输入内容直接回车，也就是直接sent一个空值会卡主`
socket-client代码修改：
```ruby
import socket
client = socket.socket()
client.connect(('localhost',6969))

while True:
    msg = input(">>:").strip()
    if len(msg) == 0:continue
    client.send(msg.encode("utf-8"))
    data = client.recv(1024)
    print("recv:",data.decode())

client.close()

```
### 使用Socket模拟SSH客户端

socket-server代码：
```ruby
import os
import socket

server = socket.socket()
server.bind(('localhost',6969))
server.listen(5)

#print("我要开始等待电话了")

while True:
    conn,addr = server.accept()
    while True:
        data =conn.recv(1024)
        res = os.popen("{}".format(data.decode())).read()
        conn.send(res.encode('utf-8'))
server.close()

```
socket-client代码：
```ruby
import socket
client = socket.socket()
client.connect(('localhost',6969))

while True:
    msg = input(">>:").strip()
    if len(msg) == 0:continue
    client.send(msg.encode("utf-8"))
    data = client.recv(1024)       # 表示只能传输1024字节
    print(data.decode())

client.close()

```
运行：
```ruby
# 服务端：
[root@localhost ~]# python socket_server.py 

# 客户端：
[root@localhost ~]# python socket_client.py 
>>:df
Filesystem           1K-blocks    Used Available Use% Mounted on
/dev/mapper/VolGroup-lv_root
                      18003272 3701812  13380272  22% /
tmpfs                   502056       0    502056   0% /dev/shm
/dev/sda1               487652   49502    412550  11% /boot


>>:ls
anaconda-ks.cfg
install.log
install.log.syslog
Python-3.6.2
Python-3.6.2.tgz
socket_client.py
socket_server.py

# 注意：不能指定top之类的命令
>>:top  # 卡主了

```
`修改一下上面top显示不全的问题`
```ruby
data = client.recv(1024000)   # 修改这块值 = 10M,但是这个值也不是随意设置，有每次接收的最大限度

```
socket-client运行：
```ruby
>>:top -bn 1
top - 00:17:34 up  1:27,  2 users,  load average: 0.39, 0.29, 0.11
Tasks:  89 total,   1 running,  88 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.9%us,  1.3%sy,  0.0%ni, 97.1%id,  0.6%wa,  0.0%hi,  0.1%si,  0.0%st
Mem:   1004112k total,   263468k used,   740644k free,    54000k buffers
Swap:  2031612k total,        0k used,  2031612k free,    58468k cached

   PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND                                                                              
     1 root      20   0 21404 1540 1244 S  0.0  0.2   0:01.08 init                                                                                 
     2 root      20   0     0    0    0 S  0.0  0.0   0:00.00 kthreadd                                                                             
     3 root      RT   0     0    0    0 S  0.0  0.0   0:00.00 migration/0                                                                          
     4 root      20   0     0    0    0 S  0.0  0.0   0:00.00 ksoftirqd

```
#### 总结：
- 服务端：
```ruby
server = socket.socket(AF.INET,sock.SOCK_STREAM)
server.bind(localhost,9999)
server.listen()
while True:
    conn,addr = server.accept()  # 阻塞
    while True:
        print("new conn",addr)
        data = conn.recv(1024) #8192 recv默认是阻塞的
        if not data:
            break  # 客户端已断开，conn.recv收到的就都是空数据
        print(data)
        conn.sned(data.upper())
```
- 客户端：
```ruby
client = socket.socket()
client.connect(serverip,9999)
client.send(data)
client.sned(data)
client.recv(date)
```


### 断言
```ruby
choice = input("PLZ inpuet>>: ")
assert type(choice) is str
print("True")

# 运行：
PLZ inpuet>>: 123  # 键盘输入就是str类型
True

choice = input("PLZ inpuet>>: ")
assert type(choice) is int
print("True")

# 运行：
PLZ inpuet>>: 123   # 只有断言对了，才会向下进行
Traceback (most recent call last):
  File "E:/PycharmProjects/untitled/study/day7/动态加载.py", line 14, in <module>
    assert type(choice) is int
AssertionError
# 断言错了就不再继续往下执行
```