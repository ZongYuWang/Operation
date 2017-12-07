
### Paramiko模块安装和使用

常见的解决方法都会需要对远程服务器必要的配置，如果远程服务器只有一两台还好说，如果有N台，还需要逐台进行配置，或者需要使用代码进行以上操作时，上面的办法就不太方便了。

使用paramiko可以很好的解决以上问题，比起前面的方法，它仅需要在本地上安装相应的软件（python以及PyCrypto），对远程服务器没有配置要求，对于连接多台服务器，进行复杂的连接操作特别有帮助。


pip下载：
https://pypi.python.org/pypi/pip#downloads
```ruby
C:\Users\Administrator>d:
D:\>cd python34\pip-9.0.1
D:\python34\pip-9.0.1>python setup.py install
......

Installing pip.exe.manifest script to D:\python34\Scripts

Installed d:\python34\lib\site-packages\pip-9.0.1-py3.4.egg
Processing dependencies for pip==9.0.1
Finished processing dependencies for pip==9.0.1

# 设置环境变量：
;D:\python34\Scripts;
C:\Users\Administrator>pip list
```

```ruby
D:\python34\paramiko-2.4.0>pip install --upgrade setuptools
参考链接：https://wiki.python.org/moin/WindowsCompilers
```

安装paramiko：
```ruby
C:\Users\Administrator>d:
D:\>cd python34\paramiko-2.4.0
D:\python34\paramiko-2.4.0>python setup.py install
running install
......
Installed d:\python34\lib\site-packages\pycparser-2.18-py3.4.egg
Searching for pyasn1==0.4.2
Best match: pyasn1 0.4.2
Processing pyasn1-0.4.2-py3.4.egg
pyasn1 0.4.2 is already the active version in easy-install.pth

Using d:\python34\lib\site-packages\pyasn1-0.4.2-py3.4.egg
Finished processing dependencies for paramiko==2.4.0

```
在pycharm中测试是否安装成功：
```ruby
import paramiko

# 运行
```

### SSHClient:

- 基于用户名密码连接：
```ruby
import paramiko

# 创建SSH对象
ssh = paramiko.SSHClient()
# 允许连接不在know_hosts文件中的主机
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
# 连接服务器
ssh.connect(hostname='172.30.105.115', port=22, username='root', password='wangzongyu')

# 执行命令
stdin, stdout, stderr = ssh.exec_command('df')
# 获取命令结果
result = stdout.read()
print(result.decode())

# 关闭连接
ssh.close()
```


- 基于公钥密钥连接：

`SSH秘钥认证：`
```ruby
# 将本机(172.30.105.115)的公钥(私钥不能传给别人)发给需要连接的主机(172.30.105.112)：

[root@localhost ~]# ssh-copy-id "-p2222 baby@172.30.105.112"
The authenticity of host '[172.30.105.112]:2222 ([172.30.105.112]:2222)' can't be established.
RSA key fingerprint is 79:48:d2:8c:ff:19:f9:e5:a2:f5:d6:84:07:17:4c:bd.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[172.30.105.112]:2222' (RSA) to the list of known hosts.
baby@172.30.105.112's password: 
Now try logging into the machine, with "ssh '-p2222 baby@172.30.105.112'", and check in:

  .ssh/authorized_keys

to make sure we haven't added extra keys that you weren't expecting.

# 使用ssh-copy-id这个命令是为了手动拷贝不能保证一行，这样拷贝能保证公钥在远程主机的这个文件中.ssh/authorized_keys保证是一行

[root@localhost ~]# ssh -p2222 baby@172.30.105.112

```

```ruby
import paramiko

private_key = paramiko.RSAKey.from_private_key_file('id_rsa_105-115')

# 创建SSH对象
ssh = paramiko.SSHClient()
# 允许连接不在know_hosts文件中的主机
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
# 连接服务器
ssh.connect(hostname='172.30.105.112', port=2222, username='baby', pkey=private_key)

# 执行命令
stdin, stdout, stderr = ssh.exec_command('df')
# 获取命令结果
result = stdout.read()
print(result.decode())

# 关闭连接
ssh.close()
```

### SFTPClient:

- 基于用户名密码上传下载:

```ruby
import paramiko

transport = paramiko.Transport(('172.30.105.112',2222))
transport.connect(username='baby',password='wangzongyu')

sftp = paramiko.SFTPClient.from_transport(transport)
# 将location.py 上传至服务器 /tmp/test.py
sftp.put('D:\Python27\LICENSE.txt', '/tmp/license_test.py')

#将remove_path 下载到本地 local_path
sftp.get('/tmp/snmpd.conf.bak', 'E:\result')

transport.close()
```


- 基于公钥密钥上传下载:
```ruby
import paramiko
 
private_key = paramiko.RSAKey.from_private_key_file('id_rsa_105-115')
 
transport = paramiko.Transport(('172.30.105.112', 2222))
transport.connect(username='baby', pkey=private_key )
 
sftp = paramiko.SFTPClient.from_transport(transport)
# 将location.py 上传至服务器 /tmp/test.py
sftp.put('/tmp/location.py', '/tmp/test.py')
# 将remove_path 下载到本地 local_path
sftp.get('remove_path', 'local_path')
 
transport.close()
```