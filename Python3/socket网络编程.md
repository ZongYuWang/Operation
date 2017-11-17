## Socket网络编程

- socket_client：
```ruby
import socket

client = socket.socket() # 声明socket类型，同时生成socket连接对象
client.connect(('localhost',6969))

client.send(b"Hello World") # python2中可以发送字符串或者是字节，但是在python3中只能发送字节
data = client.recv(1024) # 客户端接收1024字节数据=1kb
print("recv:",data)

client.close()
```
- socket_server：
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