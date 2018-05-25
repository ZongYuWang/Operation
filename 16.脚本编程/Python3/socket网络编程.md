## Socket网络编程

### 使用Socket实现聊天机器人
`聊天机器人发送与接收(固定发一句，收一句)`     

![](https://github.com/ZongYuWang/image/blob/master/python-socket1.png)

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
    if len(inp) == 0:continue
    elif inp == "q": # 如果输入了q，就只发送就行了，不用接收了
        c1.sendall(bytes(inp,encoding="utf-8")) # python2中可以发送字符串或者是字节，但是在python3中只能发送字节，而且只能字符串使用encoding转码，bytes不能转码
        break
    else: # 如果输入的不是q，那么就既要发送，又要接收
        c1.sendall(bytes(inp,encoding="utf-8"))
        result = str(c1.recv(1024),encoding="utf-8") # encoding="utf-8"必须是字符串才可用
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
    print("conn_address内容：",conn,address)
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
# 运行服务端
conn_address内容： <socket.socket fd=308, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 9999), raddr=('127.0.0.1', 59365)> ('127.0.0.1', 59365)

#通过输出可知：
conn的内容是： <socket.socket fd=308, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 9999), raddr=('127.0.0.1', 59365)>
address的内容是：('127.0.0.1', 59365)
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


#### 使用Socket实现FTP-上传功能(MD5)

MD5实现
```ruby
import hashlib

h = hashlib.md5()
h.update("wangzy".encode("utf-8")) # 生成加密串
print(h.hexdigest())   # 获取加密串
2ea763badcd027b0087306a3ef3ef95f
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
获取文件大小：
```ruby
>>> import os
>>> os.getcwd()
'E:\\PycharmProjects\\untitled\\study'
>>> os.chdir('D:\\Git_Project')
>>> os.getcwd()
'D:\\Git_Project'
>>> os.stat('demo1.txt')
os.stat_result(st_mode=33206, st_ino=2251799813754222, st_dev=2797671063, st_nlink=1, st_uid=0, st_gid=0, st_size=14, st_atime=1511424758, st_mtime=1511424758, st_ctime=1511422197)

# st_size也就是文件大小
```

- socket客户端代码：
```ruby
import socket
import os
import hashlib

c1 = socket.socket()

c1.connect(("127.0.0.1",9999))
result_bytes = c1.recv(1024)
result_str = str(result_bytes,encoding="utf-8")  # encoding="utf-8"必须是字符串才可用
print(result_str)

file_size = os.stat('f.png').st_size  # 获取文件大小,单位字节
c1.sendall(bytes(str(file_size),encoding="utf-8"))  # 先发送文件大小

# 解决粘包问题(上面发送了一个文件大小，等待服务端给回复一个确认信息之后，再发送文件内容)
c1.recv(1024)

# 发送文件
h = hashlib.md5()
with open('f.png','rb') as f:# 直接以rb的方式打开，如果以字符串的形式打开还要转换成bytes形式
    for line in f:  # 一行一行的读取
        h.update(line)
        c1.sendall(line)
    print("file MD5(before send):",h.hexdigest())

c1.sendall(bytes(h.hexdigest().encode()))
after_md5 = str(c1.recv(1024),encoding="utf-8")
print(after_md5)

c1.close()


```
- socket服务端代码：
```ruby
import socket
import hashlib

s1 = socket.socket()
s1.bind(("127.0.0.1",9999,))
s1.listen(5)
# print("before")
# print("after")
while True:
    conn,address = s1.accept()
    conn.sendall(bytes("欢迎登陆FTP系统",encoding="utf-8"))
     # 先接收文件大小，然后再开始接收
    size = str(conn.recv(1024),encoding="utf-8") # 先拿到文件的大小
    print(size)

    # 解决粘包问题（客户端发送了一个文件大小的数据包，客户端现在等着服务端给回复一个确认信息之后才会发送文件数据包）
    conn.sendall(bytes("已经接收到文件大小数据包，往下继续吧",encoding="utf-8"))

    total_size = int(size) # 文件总大小
    print(total_size)
    has_recv = 0 # 默认接收到了0字节
    f = open('new.png','wb') # 在服务端新打开一个文件，将客户端一行一行上传来的数据最后写入到这个文件中
    h = hashlib.md5()

    # 接收文件内容，直到获取完毕
    while True:  # 一直接收客户端上传来的数据
        if total_size == has_recv:
            conn.recv(1024)
            new_file_md5 = h.hexdigest()
            conn.sendall(bytes("file MD5(after send):"+new_file_md5,encoding="utf-8"))
            break
        data = conn.recv(1024) # 最多接收1024自己
        h.update(data)  # MD5放在最后一行输出，也是前面总的行数的MD5值
        f.write(data)
        # 但是怎么判断全部接收完呢？客户端源源不断的发，服务端不知道什么时候接收终止应该提前发送一个文件的大小
        has_recv += len(data) # 已经接收到的 要加上每次循环已经收到的
        # 什么时候接收终止呢？直到total_size = has_recv

    f.close()

```
`运行：`
```ruby
欢迎登陆FTP系统
file MD5(before send): 8369c9f8daa15463f41332c1a5827b95
8369c9f8daa15463f41332c1a5827b95
```   

![](https://github.com/ZongYuWang/image/blob/master/python-socket2.png)   
`粘包是由于两个send同时发送数据，上面发送的数据和文件每行内容发送都存储在缓冲区中，而在发送中，一起发送了出去，为解决这个问题，先在之前发送一个文件大小，再等待回复之后在发送文件内容`

#### 使用socket实现FTP-下载功能(增加MD5验证功能)：

① 读取文件名     
② 检测文件是否存在       
③ 打开文件       
④ 检测文件大小       
⑤ 发送文件大小给客户端       
⑥ 等客户确认       
⑦ 开始边读边发数据       
⑧ 发送MD5      

- socket服务端代码:
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

- socket客户端代码：
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
#### 使用socket实现简单的SSH工具

- socket服务端代码：
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
- socket客户端代码:
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

### socketserver：
socketserver是对socket的再封装，使socket更加的简单      
并发处理多个客户端请求       
使用：        
    ① 创建类，必须继承        
    ② handle方法        
    ③ server_forever       
    
```ruby
import socketserver

class MyServer(socketserver.BaseRequestHandler)
# 类中必须继承socketserver.BaseRequestHandler
```

- socketserver服务端代码：
```ruby
import socketserver

class MyServer(socketserver.BaseRequestHandler):

    def handle(self):
            print(self.request,self.client_address,self.server)
            conn = self.request
            conn.sendall(bytes("欢迎致电10086",encoding="utf-8"))
            while True:
                ret_bytes = conn.recv(1024)
                ret_str = str(ret_bytes,encoding="utf-8")
                if ret_str == 'q':
                    break
                conn.sendall(bytes(ret_str+"好",encoding="utf-8"))

'''
# BaseRequestHandler源码
def __init__(self, server_address, RequestHandlerClass, bind_and_activate=True):
        """Constructor.  May be extended, do not override."""
        BaseServer.__init__(self, server_address, RequestHandlerClass)
        self.socket = socket.socket(self.address_family,
                                    self.socket_type)
'''


if __name__ == '__main__':
    server = socketserver.ThreadingTCPServer(('127.0.0.1',9999),MyServer)
    server.serve_forever()  # 相当于while循环，不断的等待用户连接


# print(self.request,self.client_address,self.server)信息如下：
<socket.socket fd=324, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 9999), raddr=('127.0.0.1', 55702)> ('127.0.0.1', 55702) <socketserver.ThreadingTCPServer object at 0x01EEF870>
```
- socketserver客户端代码(支持多客户端连接)：
```ruby
import socket

ip_port = ('127.0.0.1',9999)
sk = socket.socket()
sk.connect(ip_port)
sk.settimeout(5)

while True:
    data_bytes = sk.recv(1024)
    data_str = str(data_bytes,encoding="utf-8")
    print(data_str)
    inp = input('please input:')
    sk.sendall(bytes(inp,encoding="utf-8"))

    if inp == "q":
        break

sk.close()

```


### IO多路复用
可以监听多个文件描述符(文件句柄)，一旦文件句柄发生变化，即可感知
文件描述符(socket对象)

- socket服务端代码：
```ruby
import socket
import select

sk1 = socket.socket()
sk1.bind(('127.0.0.1',8001))
sk1.listen(5)

sk2 = socket.socket()
sk2.bind(('127.0.0.1',8002))
sk2.listen(5)

sk3 = socket.socket()
sk3.bind(('127.0.0.1',8003))
sk3.listen(5)

inputs = [sk1,sk2,sk3]

while True:
    # [sk1,sk2]select内部自动监听sk1,sk2，sk3三个对象，一旦某个句柄发生变化
    # 如果有人连接sk1；
    # r_list = [sk1,sk2,sk3]
    r_list,w_list,e_list = select.select(inputs,inputs,inputs,1)
    # 这行后面的1就是等待1秒，如果一直没人来连接，inputs就是空，不会向下进行，因为一直while True，如果在0.5秒的时候来连接了，就会向下进行执行print
    # 第三个inputs表示哪个发生错误(sk1,sk2,sk3)就会放到e_list中
    # w_list对应的第二个inputs，假如第二个inputs传递的是[sk1,sk2],那么w_list永远是sk1,sk2
    print(r_list)

    for sk in r_list:
        # 每一个连接对象
        conn,address = sk.accept()
        conn.sendall(bytes('hello',encoding="utf-8"))
        conn.close()

```
- socket客户端1代码：
```ruby
import socket

obj = socket.socket()
obj.connect(('127.0.0.1',8001))

content =  str(obj.recv(1024),encoding="utf-8")
print(content)

obj.close()

```
- socket客户端2代码：
```ruby
import socket

obj = socket.socket()
obj.connect(('127.0.0.1',8002))

content =  str(obj.recv(1024),encoding="utf-8")
print(content)

obj.close()
```
`运行:`
```ruby
[]
[<socket.socket fd=272, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 8001)>]
[]
[]
[]
[<socket.socket fd=280, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 8002)>]
[]

```

#### 使用(IO多路复用)select实现多客户端连接
`实现了多客户端的接收，但是还是一个个的处理，实际上是虚假的多客户端连接`

select中第一个参数(r_list)详解：

- socket服务端代码：
```ruby
import socket
import select

sk1 = socket.socket()
sk1.bind(('127.0.0.1',8001))
sk1.listen(5)

inputs = [sk1]
print("还没有用户连接inputs的内容：",inputs)
while True:
    r_list,w_list,e_list = select.select(inputs,[],inputs,1)
    # 最开始没人来连接的时候，r_list = [sk1]
    # client1来连接之后，inputs的内容就是[sk1,client1]
    print("正在监听的socket对象%d" %len(inputs))
    #inputs = ["sk1","client1"]
    #print(len(inputs))
    print(r_list)
    for sk1_or_conn in r_list:
        # 每一个连接对象
        if sk1_or_conn == sk1:   # 表示有新的用户来连接
            conn,address = sk1_or_conn.accept()
            inputs.append(conn)  # 这个时候inputs的内容是[sk1,c1]
            print("新客户端连接之后-inputs内容：",inputs)
        else:  # 有老用户发消息了
            data_bytes = sk1_or_conn.recv(1024)
            data_str = str(data_bytes,encoding="utf-8")
            sk1_or_conn.sendall(bytes(data_str+"好",encoding="utf-8"))
            
```
- socket客户端代码：
```ruby
import socket

obj = socket.socket()
obj.connect(('127.0.0.1',8001))

while True:
    inp = input("请输入要发送的内容>>>：")
    obj.sendall(bytes(inp,encoding="utf-8"))
    content =  str(obj.recv(1024),encoding="utf-8")
    print(content)

obj.close()

```
`服务端运行：`
```ruby
还没有用户连接inputs的内容： [<socket.socket fd=272, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 8001)>]
正在监听的socket对象1
[]
正在监听的socket对象1
[]
正在监听的socket对象1
[<socket.socket fd=272, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 8001)>]
新客户端连接之后-inputs内容： [<socket.socket fd=272, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 8001)>, <socket.socket fd=280, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 8001), raddr=('127.0.0.1', 59998)>]
正在监听的socket对象2
[]
正在监听的socket对象2
[]
正在监听的socket对象2
[<socket.socket fd=280, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 8001), raddr=('127.0.0.1', 59998)>]
# 上面是客户端发送了一条信息
正在监听的socket对象2
[]
正在监听的socket对象2
[]
正在监听的socket对象2
[<socket.socket fd=280, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 8001), raddr=('127.0.0.1', 59998)>]
Traceback (most recent call last):
  File "E:/PycharmProjects/untitled/study/day8/IO多路复用/s1_加强版.py", line 31, in <module>
    data_bytes = sk1_or_conn.recv(1024)
ConnectionResetError: [WinError 10054] 远程主机强迫关闭了一个现有的连接。

# 这是python3新的功能，当客户端终止连接之后，服务端最后报错，所以使用try一下
```

- socket服务端代码修改(看输出信息)：
```ruby
import socket
import select

sk1 = socket.socket()
sk1.bind(('127.0.0.1',8001))
sk1.listen(5)

inputs = [sk1]
print("还没有用户连接inputs的内容：",inputs)
while True:
    r_list,w_list,e_list = select.select(inputs,[],inputs,1)
    # 最开始没人来连接的时候，r_list = [sk1]
    # client1来连接之后，inputs的内容就是[sk1,client1]
    print("正在监听的socket对象%d" %len(inputs))
    #inputs = ["sk1","client1"]
    #print(len(inputs))
    print(r_list)
    for sk1_or_conn in r_list:
        # 每一个连接对象
        if sk1_or_conn == sk1:
            # 表示有新的用户来连接
            conn,address = sk1_or_conn.accept()
            inputs.append(conn)  # 这个时候inputs的内容是[sk1,c1]
            print("新客户端连接之后-inputs内容：",inputs)
            print("sk1_or_conn内容：",sk1_or_conn)
        else:
            # 有老用户发消息了
            try:
                data_bytes = sk1_or_conn.recv(1024)
                data_str = str(data_bytes,encoding="utf-8")
                sk1_or_conn.sendall(bytes(data_str+"好",encoding="utf-8"))

            except Exception as ex:
                # 用户终止连接
                inputs.remove(sk1_or_conn)
                print("客户端断开之后sk1_or_conn内容：",sk1_or_conn)
```
`服务端输出：`
```ruby
还没有用户连接inputs的内容： [<socket.socket fd=280, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 8001)>]
正在监听的socket对象1
[]
正在监听的socket对象1
[]
正在监听的socket对象1
[<socket.socket fd=280, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 8001)>]
新客户端连接之后-inputs内容： [<socket.socket fd=280, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 8001)>, <socket.socket fd=308, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 8001), raddr=('127.0.0.1', 60714)>]
sk1_or_conn内容： <socket.socket fd=280, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 8001)>
正在监听的socket对象2
[]
正在监听的socket对象2
[]
正在监听的socket对象2
[<socket.socket fd=308, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 8001), raddr=('127.0.0.1', 60714)>]
正在监听的socket对象2
[]
正在监听的socket对象2
[]
正在监听的socket对象2
[<socket.socket fd=308, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 8001), raddr=('127.0.0.1', 60714)>]
客户端断开之后sk1_or_conn内容： <socket.socket fd=308, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 8001), raddr=('127.0.0.1', 60714)>
正在监听的socket对象1
[]
正在监听的socket对象1
[]
正在监听的socket对象1
[]
正在监听的socket对象1
```

socket中第二个参数(w_list)详解（目的是实现读写分离）：

- socket服务端代码：

```ruby
import socket
import select

sk1 = socket.socket()
sk1.bind(('127.0.0.1',8001))
sk1.listen(5)

inputs = [sk1]
print("还没有用户连接inputs的内容：",inputs)

outputs = []


while True:
    r_list,w_list,e_list = select.select(inputs,outputs,inputs,1)
    # 最开始没人来连接的时候，r_list = [sk1]
    # client1来连接之后，inputs的内容就是[sk1,client1]
    print("正在监听的socket对象%d" %len(inputs))
    #inputs = ["sk1","client1"]
    #print(len(inputs))
    print(r_list)
    for sk1_or_conn in r_list:
        # 每一个连接对象
        if sk1_or_conn == sk1:
            # 表示有新的用户来连接
            conn,address = sk1_or_conn.accept()
            inputs.append(conn)  # 这个时候inputs的内容是[sk1,c1]
            print("新客户端连接之后-inputs内容：",inputs)
            print("sk1_or_conn内容：",sk1_or_conn)
        else:
            # 有老用户发消息了
            try:
                data_bytes = sk1_or_conn.recv(1024)

            except Exception as ex:
                inputs.remove(sk1_or_conn) # 中断之后就要移出掉，以后不再跟这个联系
                print("客户端断开之后sk1_or_conn内容：",sk1_or_conn)

            else:
                # 用户正常发送数据
                #  data_str = str(data_bytes,encoding="utf-8")
                #  sk1_or_conn.sendall(bytes(data_str+"好",encoding="utf-8"))
                outputs.append(sk1_or_conn)  # 也就是谁给我发送过消息，我就存在w_list中
                # w_list中仅仅保存了谁给我发过消息,但是还没给客户回呢，下面给是回复
                print("outputs内容：",outputs)

    for conn in w_list:
        conn.sendall(bytes('hello',encoding="utf-8"))
        outputs.remove(conn)

    for sk in e_list:
        inputs.remove(sk)
```
`但是上面的代码存在一个问题，无论客户端发什么都回复一个hello，没获取到客户端发送的内容`

- socket服务端代码修改：
```ruby
# 通过建立一个空字典，将客户端发送的内容存在字典中

import socket
import select

sk1 = socket.socket()
sk1.bind(('127.0.0.1',8001))
sk1.listen(5)

inputs = [sk1]
print("还没有用户连接inputs的内容：",inputs)

outputs = []
message_dict = {} # 定义一个空字典，也就是将用户发送的信息存在字典里

while True:
    r_list,w_list,e_list = select.select(inputs,outputs,inputs,1)
    # 最开始没人来连接的时候，r_list = [sk1]
    # client1来连接之后，inputs的内容就是[sk1,client1]
    print("正在监听的socket对象%d" %len(inputs))
    #inputs = ["sk1","client1"]
    #print(len(inputs))
    print(r_list)
    for sk1_or_conn in r_list:
        # 每一个连接对象
        if sk1_or_conn == sk1:
            # 表示有新的用户来连接
            conn,address = sk1_or_conn.accept()
            inputs.append(conn)  # 这个时候inputs的内容是[sk1,c1]
            print("新客户端连接之后-inputs内容：",inputs)
            print("sk1_or_conn内容：",sk1_or_conn)
            message_dict[conn] = [] # 字典的key是conn，也就是其中的某个客户端，value是空列表
            # 第二个客户端来连接{"client1":[],"client2":[]}
        else:
            # 有老用户发消息了
            try:
                data_bytes = sk1_or_conn.recv(1024)

            except Exception as ex:
                inputs.remove(sk1_or_conn) # 中断之后就要移出掉，以后不再跟这个联系
                print("客户端断开之后sk1_or_conn内容：",sk1_or_conn)

            else:
                # 用户正常发送数据
                data_str = str(data_bytes,encoding="utf-8")
                message_dict[sk1_or_conn].append(data_str) # 将发的消息存在了字典中{"client1":[data_str],"client2":[]}
                #  sk1_or_conn.sendall(bytes(data_str+"好",encoding="utf-8"))
                outputs.append(sk1_or_conn)  # 也就是谁给我发送过消息，我就存在w_list中
                # w_list中仅仅保存了谁给我发过消息,但是还没给客户回呢，下面给是回复
                print("outputs内容：",outputs)

    for conn in w_list:
        recv_str = message_dict[conn][0]
        # 拿完数据之后应该再删除掉之前的，等待下次再接收信息{"client1":[你]}，把这个"你"需要删除了，再接收"他"，要不然就成{"client1":[你][他]}
        conn.sendall(bytes(recv_str+'好',encoding="utf-8"))
        outputs.remove(conn)

    for sk in e_list:
        inputs.remove(sk)
```



### poll和epoll
```ruby
r_list,w_list,e_list = select.select(inputs,[sk1,sk2],[],1)
# select后面的这个[sk1,sk2]这个是有个数限制的，最多是1024个
后来有了poll，不再使用select，只是对个数没限制了，底层也是使用for循环
epoll革新了poll，底层不再使用for循环，底层使用了异步的方式，谁有变化谁告诉我，不再一个一个的去问

for sk in r_list:
     # 每一个连接对象
     conn,address = sk.accept()
     conn.sendall(bytes('hello',encoding="utf-8"))
     conn.close()
```



### send和sendall区别：
```ruby
conn.sendall()
conn.send("this is a test send")
# 发送这一串(this is a test send)的时候，可能都会发过去，也可能只发送一部分，sendall就是循环的send，全部发送出去
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

