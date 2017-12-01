## Socket网络编程

### 使用Socket实现聊天机器人
`聊天机器人发送与接收(固定发一句，收一句)`
- socket客户端代码：
```ruby
import socket

c1 = socket.socket() # 创建一个对象

c1.connect(("127.0.0.1",9999))  # 表示连接那个服务端，写服务端的IP地址和服务器允许连接的端口
#c1.recv(1024) # 表示客户端“最多”接收1024字节，超过1024接收不了，剩下的就下次接收
result_bytes = c1.recv(1024)  # 发送的是字节类型，那么收到的也是字节类型
# recv也会阻塞,client1连接服务端，按理说服务端应该立即返还给我，但是如果服务端没返还，recv将一直等待
result_str = str(result_bytes,encoding="utf-8") # 将字节转换成字符串
print(result_str)
while True:
    inp = input("请输入要发送的内容>>>：")
    if inp == "q": # 如果输入了q，就只发送就行了，不用接收了
        c1.sendall(bytes(inp,encoding="utf-8")) # python2中可以发送字符串或者是字节，但是在python3中只能发送字节，而且只能字符串使用encoding转码，bytes不能转码
        break
    else: # 如果输入的不是q，那么就既要发送，又要接收
        c1.sendall(bytes(inp,encoding="utf-8"))
        result = str(c1.recv(1024),encoding="utf-8")
        print(result)
c1.close() # 连接完就关闭

```
- socket服务端代码：
```ruby
import socket

s1 = socket.socket()  # 创建一个对象
s1.bind(("127.0.0.1",9999,))  # 绑定自己的IP和端口，IP和端口是一个元组
s1.listen(5) # 表示最多允许5个客户端连接(也就是最大的排队数)

print("before")
#sk.accept() # 接收客户端的请求，accept阻塞
print("after")

# ------------- 代码写到这，before会执行，但是after不会执行，accept会等待客户端连接------------------

while True: # 不能让服务端连接一次就断开，需要让服务端不断接收连接，所以加一个while循环
    conn,address = s1.accept() # conn相当于其中一个client1和我之间连接的线，address就相当于client1的IP地址
    conn.sendall(bytes("欢迎致电babyshen",encoding="utf-8")) #通过conn发送消息，在python3中不能直接发送字符串，只能发送字节,这发送相应的客户端就有一个接收recv
    while True: # 针对一个客户端循环接收
        result_bytes = conn.recv(1024)
        result_str = str(result_bytes,encoding="utf-8")
        if result_str == "q": # 这块是为了客户端退出，当收到一个q，就break跳出当前客户端的循环
            break
        conn.sendall(bytes(result_str+"好",encoding="utf-8"))
    print(conn,address)
    
```
`通过上面得知conn就是客户端连过来而在服务器端为其生成的一个连接实例 `
`运行:`
```ruby
# 运行客户端
......

# 运行客户端
欢迎致电babyshen
请输入要发送的内容>>>：我
我好
请输入要发送的内容>>>：你
你好
请输入要发送的内容>>>：他
他好
请输入要发送的内容>>>：
```






- 电话发送与接收(固定发一句，收一句)
socket_client代码：
```ruby
import socket

client = socket.socket() # 声明socket类型，同时生成socket连接对象
client.connect(('localhost',6969))

client.send(b"Hello World") 
data = client.recv(1024) # 客户端接收1024字节数据=1kb
print("recv:",data)

client.close()
```
socket_server代码：
```ruby
import socket
server = socket.socket()
server.bind(('localhost',6969))
server.listen(5)                   

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

#### 实现简单的SSH工具

-socket-ssh-server：
```ruby
import socket
import os
server = socket.socket()
server.bind(('localhost',9999)) #如果是windows连接linux-server，需要把localhost修改为0.0.0.0
server.listen(5)

while True:
    conn,addr = server.accept()
    print("new conn:",addr)
    while True:
        print("等待指令")
        data = conn.recv(1024)
        if not data:
            print("客户端已经断开")
            break
        print("执行指令：",data)
        cmd_res = os.popen(data.decode()).read()  # 接收字符串，执行结果也是字符串
        print("before send",len(cmd_res))
        if len(cmd_res) == 0:
            cmd_res = "cmd has no output..."

        conn.send(str(len(cmd_res.encode())).encode("utf-8"))
        # 先发送数据的大小给客户端，因为设置了1024，客户端不可能一下全接收，设置成8192也行，但是万一数据超过8192呢，让客户端循环遍历整个大小
        conn.send(cmd_res.encode("utf-8"))
        print("send done")
server.close()
```
- socket-ssh-client:
```ruby
import socket

client = socket.socket()
client.connect(('localhost',9999)) # 如果是windows连接linux-server，localhost修改为windows本机的IP地址

while True:
    cmd = input(">>：").strip()
    if len(cmd) == 0: continue
    client.send(cmd.encode("utf-8"))

    cmd_res_size = client.recv(1024) # 接收命令结果的长度
    print("命令结果大小：",cmd_res_size)
    received_size = 0
    received_data = b''
    while received_size < int(cmd_res_size.decode()):
        data = client.recv(1024)
        received_size += len(data) # 每次收到的有可能小于1024，不一定每次都是1024，所以必须用len判断
        #print(data.decode())
        received_data += data
    else:
        print("cmd res receive done....",received_size)
        print(received_data.decode())
        # cmd_res = client.recv(1024)
        # print(cmd_res.decode())

client.close()
```
`问题来了：上述代码会存在socket粘包问题,为什么会出现粘包问题呢，是因为两次的send同时进行了`

方式1(这种方式太low了)：
```ruby
#socket-ssh-server修改代码：

import time
......
conn.send(str(len(cmd_res.encode())).encode("utf-8"))
        # 先发送数据的大小给客户端，因为设置了1024，客户端不可能一下全接收，设置成8192也行，但是万一数据超过8192呢，让客户端循环遍历整个大小
        time.sleep(0.5)  # 两次send中间加了一个0.5秒的时间间隔
        conn.send(cmd_res.encode("utf-8"))
......
```
方式2：
```ruby
#socket-ssh-server修改代码：

......
conn.send(str(len(cmd_res.encode())).encode("utf-8"))
        # 先发送数据的大小给客户端，因为设置了1024，客户端不可能一下全接收，设置成8192也行，但是万一数据超过8192呢，让客户端循环遍历整个大小
        client_ack = conn.recv(1024) # wait client to confirm
        print("ack from client:",client_ack)
        conn.send(cmd_res.encode("utf-8"))
        print("send done")
......


#socket-ssh-client修改代码：
......
 cmd_res_size = client.recv(1024) # 接收命令结果的长度
    print("命令结果大小：",cmd_res_size)
    client.send("准备好接受了，loser可以发了".encode("utf-8"))
    received_size = 0
    received_data = b''
......
```

中文的长度问题：
```ruby
Python 3.4.3 (v3.4.3:9b73f1c3e601, Feb 24 2015, 22:43:06) [MSC v.1600 32 bit (Intel)] on win32
>>> a = "神"
>>> len(a)
1
>>> len(a.encode())  # 转码之后变为3
3
```

#### FTP使用：
- socket-ftp-server:
```ruby
import socket,os,time
import hashlib

server = socket.socket()
server.bind(('0.0.0.0',9999))
server.listen(5)

while True:
    conn,addr = server.accept()
    print("new conn:",addr)
    while True:
        print("等待指令")
        data = conn.recv(1024)
        if not data:
            print("客户端已经断开")
            break

        cmd,filename = data.decode().split()
        print(filename)

        if os.path.isfile(filename):
            f = open(filename,"rb")
            m = hashlib.md5()
            file_size = os.stat(filename).st_size
            conn.send( str(file_size).encode() ) # sedn file size
            conn.recv(1024) # wait for ack

            for line in f:
                m.update(line)
                conn.send(line)
            #print("file md5",m.hexdigest())
            f.close()
        print("send done")
server.close()

```
- socket-ftp-client:
```ruby
import socket

client = socket.socket()
client.connect(('localhost',9999))

while True:
    cmd = input(">>：").strip()
    if len(cmd) == 0: continue
    if cmd.startswith("get"):
        client.send(cmd.encode())
        server_response = client.recv(1024)
        print("server response:",server_response)
        client.send(b"ready to recv file")
        file_total_size = int(server_response.decode())
        received_size = 0
        filename = cmd.split()[1]
        f = open(filename + ".new","wb")
        while received_size < file_total_size:
            data = client.recv(2014)
            received_size += len(data)
            f.write(data)
            #print(file_total_size,received_size)
        else:
            print("file recv done",received_size,file_total_size)
            f.close()


client.close()

```


运行：
```ruby
# socket-ftp-server：
[root@localhost ~]# python socket_ftp_server.py 
new conn: ('127.0.0.1', 46858)
等待指令
filename.txt
send done
等待指令

# socket-ftp-client：
[root@localhost ~]# python socket_ftp_client.py 
>>：get filename.txt  # 客户端上传文件
server response: b'24305842'
file recv done 24305842 24305842
>>：

```

增加MD5验证功能：
- socket-ftp-server:
```ruby
import hashlib
import socket,os,time
import hashlib

server = socket.socket()
server.bind(('0.0.0.0',9999))
server.listen(5)

while True:
    conn,addr = server.accept()
    print("new conn:",addr)
    while True:
        print("等待指令")
        data = conn.recv(1024)
        if not data:
            print("客户端已经断开")
            break

        cmd,filename = data.decode().split()
        print(filename)

        if os.path.isfile(filename):
            f = open(filename,"rb")
            m = hashlib.md5()
            file_size = os.stat(filename).st_size
            conn.send( str(file_size).encode() ) # sedn file size
            conn.recv(1024) # wait for ack

            for line in f:
                m.update(line)
                conn.send(line)
            print("file md5",m.hexdigest())
            f.close()
            conn.send(m.hexdigest().encode()) # send md5
                
```
- socket-ftp-client:
```ruby
import socket
import hashlib

client = socket.socket()
client.connect(('localhost',9999))

while True:
    cmd = input(">>：").strip()
    if len(cmd) == 0: continue
    if cmd.startswith("get"):
        client.send(cmd.encode())
        server_response = client.recv(1024)
        print("server response:",server_response)
        client.send(b"ready to recv file")
        file_total_size = int(server_response.decode())
        received_size = 0
        filename = cmd.split()[1]
        f = open(filename + ".new","wb")
        m = hashlib.md5()
        while received_size < file_total_size:
            data = client.recv(1024)
            received_size += len(data)
            m.update(data)
            f.write(data)
            #print(file_total_size,received_size)
        else:
            new_file_md5 = m.hexdigest()
            print("file recv done",received_size,file_total_size)
            f.close()
        server_file
```
但是这么做会有粘包问题，运行一下
```ruby
# socket-ftp-client
[root@localhost ~]# python socket_ftp_client.py 
>>：get socket_server.py
server response: b'321'
file recv done 353 321
# 客户端卡在这了，应该输出MD5验证，出现了粘包问题

# socket-ftp-server
[root@localhost ~]# python socket_ftp_server.py 
new conn: ('127.0.0.1', 46874)
等待指令
socket_server.py
file md5 23388392e227016bc916f882553aebf7
send done
等待指令
```
`解决粘包问题 socket-ftp-client代码：`
```ruby
import socket
import hashlib

client = socket.socket()

client.connect(('localhost', 9999))

while True:
    cmd = input(">>:").strip()
    if len(cmd) == 0: continue
    if cmd.startswith("get"):
        client.send(cmd.encode())
        server_response = client.recv(1024)
        print("servr response:", server_response)
        client.send(b"ready to recv file")
        file_total_size = int(server_response.decode())
        received_size = 0
        filename = cmd.split()[1]
        f = open(filename + ".new", "wb")
        m = hashlib.md5()

        while received_size < file_total_size:
            if file_total_size - received_size > 1024:  # 要收不止一次
                size = 1024
            else:  # 最后一次了，剩多少收多少
                size = file_total_size - received_size
                print("last receive:", size)

            data = client.recv(size)
            received_size += len(data)
            m.update(data)
            f.write(data)
            # print(file_total_size,received_size)
        else:
            new_file_md5 = m.hexdigest()
            print("file recv done", received_size, file_total_size)
            f.close()
        server_file_md5 = client.recv(1024)
        print("server file md5:", server_file_md5)
        print("client file md5:", new_file_md5)

client.close()

```