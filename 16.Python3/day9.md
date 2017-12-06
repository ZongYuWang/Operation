
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