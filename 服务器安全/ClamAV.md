## 服务器查杀病毒 ##

### Clamav ###

**官网地址**
http://www.clamav.net/downloads#collapsePreviousReleases官网

```ruby
[root@localhost ~]# mkdir clamav
[root@localhost ~]# cd /root/clamav/

# 编译安装zlib
[root@localhost clamav]# tar xvf zlib-1.2.7.tar.gz
[root@localhost clamav]# cd zlib-1.2.7
[root@localhost zlib-1.2.7]# ./configure && make && make install && echo ok

# 编译安装clamav
[root@localhost clamav-0.99.2]# yum -y install openssl-devel 
[root@localhost clamav]# tar xvf clamav-0.99.2.tar.gz
[root@localhost clamav]# cd clamav-0.99.2
[root@localhost clamav-0.99.2]# ./configure --prefix=/usr/local/clamav && make && make install && echo ok

```

```ruby
# 新建日志存放目录
[root@localhost clamav-0.99.2]# mkdir /usr/local/clamav/logs

# 新建clamav病毒库目录
[root@localhost clamav-0.99.2]# mkdir /usr/local/clamav/updata

```

```ruby
# 修改配置文件
[root@localhost clamav-0.99.2]# cp /usr/local/clamav/etc/clamd.conf.sample /usr/local/clamav/etc/clamd.conf

第8行注释掉：
#Example

第14行：
LogFile /usr/local/clamav/logs/clamd.log

#第66行
PidFile /usr/local/clamav/updata/clamd.pid

# 第74行：
DatabaseDirectory /usr/local/clamav/updata

```

```ruby
# 修改配置文件：
[root@localhost clamav-0.99.2]# cp /usr/local/clamav/etc/freshclam.conf.sample /usr/local/clamav/etc/freshclam.conf

[root@localhost clamav-0.99.2]# vim /usr/local/clamav/etc/freshclam.conf

第8行注释：
# Example

第13行：
DatabaseDirectory /usr/local/clamav/updata

第17行：
UpdateLogFile /usr/local/clamav/logs/freshclam.log

第51行：
PidFile /usr/local/clamav/updata/freshclam.pid

```

```ruby
# 创建clamav组
[root@localhost ~]# groupadd clamav

# 创建clamav用户
[root@localhost ~]# useradd -g clamav clamav

```

```ruby
创建日志文件：
[root@localhost ~]# touch /usr/local/clamav/logs/freshclam.log
[root@localhost ~]# chown clamav:clamav /usr/local/clamav/logs/freshclam.log
[root@localhost ~]# touch /usr/local/clamav/logs/clamd.log
[root@localhost ~]# chown clamav:clamav /usr/local/clamav/logs/clamd.log
[root@localhost ~]# chown clamav:clamav /usr/local/clamav/updata

```

升级病毒库：
```ruby
# 前提能保证服务器能上网

[root@localhost ~]# /usr/local/clamav/bin/freshclam
```
*`打印输出`*
```ruby
[root@localhost ~]# /usr/local/clamav/bin/freshclam 
ClamAV update process started at Wed Aug 16 20:19:23 2017
Downloading main.cvd [100%]
main.cvd updated (version: 58, sigs: 4566249, f-level: 60, builder: sigmgr)
Downloading daily.cvd [100%]
daily.cvd updated (version: 23855, sigs: 1743078, f-level: 63, builder: neo)
Downloading bytecode.cvd [100%]
bytecode.cvd updated (version: 312, sigs: 74, f-level: 63, builder: neo)
[LibClamAV] ******************************************************
[LibClamAV] ***      Virus database timestamp in the future!   ***
[LibClamAV] ***  Please check the timezone and clock settings  ***
[LibClamAV] ******************************************************
Database updated (6309401 signatures) from database.clamav.net (IP: 72.21.91.8)

```

查杀病毒：
```ruby
# 查杀在哪个目录下就在哪个目录下执行，如查杀/tmp目录
[root@localhost tmp]# /usr/local/clamav/bin/clamscan --remove

```

*`打印输出`*
```ruby
[root@localhost tmp]# /usr/local/clamav/bin/clamscan --remove
LibClamAV Warning: ******************************************************
LibClamAV Warning: ***      Virus database timestamp in the future!   ***
LibClamAV Warning: ***  Please check the timezone and clock settings  ***
LibClamAV Warning: ******************************************************
/tmp/tradeease.net: OK
/tmp/tmpfZocmC: Empty file
/tmp/yum.log: Empty file
/tmp/tradeease.ru: OK
/tmp/tradeease.sh: OK
/tmp/tradeease.cn: OK

----------- SCAN SUMMARY -----------
Known viruses: 6303077
Engine version: 0.99.2
Scanned directories: 1
Scanned files: 4
Infected files: 0
Data scanned: 0.40 MB
Data read: 0.20 MB (ratio 2.00:1)
Time: 13.378 sec (0 m 13 s)

```

生产环境应用：
一般使用计划任务，让服务器每天晚上定时跟新和定时杀毒。保存杀毒日志，我的crontab文件如下：

```ruby
1  3  * * *          /usr/local/clamav/bin/freshclam
20 3  * * *          /usr/local/clamav/bin/clamscan  -r /home  --remove -l /var/log/clamscan.log
```

```ruby
# 扫描root下的病毒
[root@localhost ~]# /usr/local/clamav/bin/clamscan -r --bell -i /root

/root/clamav/clamav-0.99.2/test/clam.newc.cpio: Clamav.Test.File-6 FOUND
/root/clamav/clamav-0.99.2/test/clam.zip: Clamav.Test.File-6 FOUND
/root/clamav/clamav-0.99.2/test/clam.iso: Clamav.Test.File-6 FOUND
/root/clamav/clamav-0.99.2/test/clam.ea05.exe: Clamav.Test.File-6 FOUND
/root/clamav/clamav-0.99.2/test/clam.ea06.exe: Clamav.Test.File-6 FOUND
/root/clamav/clamav-0.99.2/test/clam.tar.gz: Clamav.Test.File-6 FOUND
【说明】这些事病毒文件，需要删除

----------- SCAN SUMMARY -----------
Known viruses: 6303077
Engine version: 0.99.2
Scanned directories: 292
Scanned files: 5650
Infected files: 50
Data scanned: 279.85 MB
Data read: 178.38 MB (ratio 1.57:1)
Time: 97.259 sec (1 m 37 s)

```

```ruby
# root目录下的排除之后

----------- SCAN SUMMARY -----------
Known viruses: 6303077
Engine version: 0.99.2
Scanned directories: 292
Scanned files: 5650
Infected files: 50
Data scanned: 279.85 MB
Data read: 178.38 MB (ratio 1.57:1)
Time: 97.259 sec (1 m 37 s)

```