## LoadRunner


### 1、安装LoadRunner11版本：

### 2、安装LoadRunner12.53版本（未能破解）：

#### 2.1 安装KB2999226补丁包：
`根据系统要求下载响应的版本，我下载Windows6.1-KB2999226-x64.msu放在了E:\LoadRunner\下`

```ruby
C:\Users\Administrator>expand -F:* E:\LoadRunner\Windows6.1-KB2999226-x64.msu E:
\LoadRunner\
Microsoft (R) 文件扩展实用程序版本 6.1.7600.16385
版权所有 (c) Microsoft Corporation。保留所有权利。

正在将 E:\LoadRunner\WSUSSCAN.cab 添加到提取队列
正在将 E:\LoadRunner\Windows6.1-KB2999226-x64.cab 添加到提取队列
正在将 E:\LoadRunner\Windows6.1-KB2999226-x64-pkgProperties.txt 添加到提取队列
正在将 E:\LoadRunner\Windows6.1-KB2999226-x64.xml 添加到提取队列

正在展开文件 ....

完成展开文件 ...
总共 4 个文件。
```
```ruby
C:\Users\Administrator>dism.exe /online /Add-Package /PackagePath:E:\LoadRunner\
Windows6.1-KB2999226-x64.cab

部署映像服务和管理工具
版本: 6.1.7600.16385

映像版本: 6.1.7600.16385

正在处理 1 (共 1) - 正在添加程序包 Package_for_KB2999226~31bf3856ad364e35~amd64~
~6.1.1.7
[==========================100.0%==========================]
操作成功完成。
```

#### 2.2 安装LoadRunner 12.53
① 运行HP LoadRunner 12.53 Community Edition.exe     

② HP LoadRunner安装程序-HP 身份验证设置：      
若有LoadRunner代理证书则默认勾选并添加CA证书，若没有证书必须取消勾选否则安装不能继续     

③ 安装成功之后，桌面上会出现三个应用程序：     
Analysis    
Controller     
Virtual User Generator     
