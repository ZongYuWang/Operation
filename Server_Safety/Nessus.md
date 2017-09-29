
**安装Nessus：**
```py

[root@localhost ~]# mkdir /nessus
[root@localhost ~]# cd /nessus/

[root@localhost nessus]# rpm -ivh Nessus-6.11.1-es6.x86_64.rpm 
warning: Nessus-6.11.1-es6.x86_64.rpm: Header V4 RSA/SHA1 Signature, key ID 1c0c4a5d: NOKEY
Preparing...                ########################################### [100%]
   1:Nessus                 ########################################### [100%]
Unpacking Nessus Core Components...
nessusd (Nessus) 6.11.1 [build M20101] for Linux
Copyright (C) 1998 - 2017 Tenable Network Security, Inc

Processing the Nessus plugins...
[##################################################]

All plugins loaded (1sec)
 - You can start Nessus by typing /sbin/service nessusd start
 - Then go to https://localhost.localdomain:8834/ to configure your scanner

启动nessus：
[root@localhost nessus]# service nessusd start


```
连接nessus-WEB界面：（必须要使用https）
https://172.30.105.115:8834/
![](https://github.com/ZongYuWang/Operation/blob/master/image/Nessus1.png)
![](https://github.com/ZongYuWang/Operation/blob/master/image/Nessus2.png)
![](https://github.com/ZongYuWang/Operation/blob/master/image/Nessus3.png)
![](https://github.com/ZongYuWang/Operation/blob/master/image/Nessus4.png)

**激活Nessus：**
```py

http://www.tenable.com/products/nessus-home

填写信息：
First Name：trade
Last Name：ease
Eamil：*********@qq.com
勾选：Check to receive updates from Tenable

邮箱中会收到一个激活码：
Your activation code for the Nessus Home is 
A244-2600-BD0F-BDA9-****

将激活码粘贴在上图中的Activation Code处

```

**更新插件：**
家庭版不支持在线更新的方法，需要离线更新：
```py
关闭nessus程序：
[root@localhost ~]# service nessusd stop

获取challenge code：
[root@localhost ~]# /opt/nessus/sbin/nessuscli fetch --challenge

Challenge code: 4db420f1b47710741399b2067b3aec875b148392
You can copy the challenge code above and paste it alongside your
Activation Code at:
https://plugins.nessus.org/v2/offline.php

获取Active code，需要重新登陆网站获取(同时需要新的用户名和邮箱)：
https://plugins.nessus.org/offline.php
Your activation code for the Nessus Home is 
D045-2403-4D1C-FB85-7E88

下载all-2.0.tar.gz与nessus-fetch.rc文件：

下载完成执行：（两条命令不分先后）
[root@localhost nessus]# /opt/nessus/sbin/nessuscli update /nessus/all-2.0.tar.gz 
[root@localhost nessus]# /opt/nessus/sbin/nessuscli fecth --register-offline nessus.license
【说明】Challenge code和Active code后，没有nessus.license文件下载，而是nessus-fetch.rc。这是由于没有关闭nessus程序导致的，再申请只有nessus-fetch.rc

重启nessus服务：
[root@localhost nessus]# service nessusd restart

手动更新：
登陆Nessus → Setting → Software Update → 右上角 Manual Software Update → Upload your own plugin archive

更新插件成功如下所示：
```
![](https://github.com/ZongYuWang/Operation/blob/master/image/Nessus5.png)


**Nessus的使用操作：**
##### 个性化扫描说明操作：
- 选择Scans → New Scan，这时可以看到所有的扫描模板(All Templates)。Scanner模板是程序带有的扫描模板，User模板我们自己配置的策略模板；
- 选择Advanced Scan，有Settings、Credentials、Compliance、Plugins子项，在Settings设置基本配置，Credentials添加目标权限，Compliance添加目标服务配置文件，Plugins插件激活与关闭设置；
- Settings中BASIC有三个子项：General、Schedule、Notification。在General配置基本信息，包括项目命名，描述，归档，目标地址（家庭版一次项目只支持16个IP）。Schedule是定时扫描设置。Notification是扫描结束后邮件通知设置，添加项目名，扫描IP地址；
- Discovery有三个子项：Host Discovery、Port Scanning、Service Discovery。这里保存默认设置即可；
- ASSESSMENT有五子项：General、Brute Force、Web Applications、Windows、Malware。在这里可以设置Web、Windows、恶意软件扫描，还可以在Brute Force设置暴力破解。看需求添加扫描类型；
- REPORT报告设置，保存默认设置即可；
- ADVANCED高级选项设置，配置同时扫描的主机数和主机扫描线程数，保存默认设置即可；
- 最后单击save保存，在主界面单击扫描键（Launch ）执行；