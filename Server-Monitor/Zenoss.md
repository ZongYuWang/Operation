## zenoss3的安装配置、升级和数据迁移

### 一、安装Zenoss
- zenoss3的安装有三种形式，源码包形式（太恶心人直接略过），rpm包形式，虚拟镜像形式。
- 虚拟镜像形式是最简单的，只要你电脑上有VMware或者virtualbox，选择带有vmware后缀的那些zip文件，在VMware里面导入就可以直接用了。下面讲一下rpm包安装形式。

安装前确认几件事:
- 你要确定你机器的型号和系统的版本（uname -a 查看），不同版本对应不同的安装包，这里默认是 centos5.X 的64位系统，选择 这个 rpm包。
- 要保持联网状态
- yum源正常可用  

#### 1.1 安装zenoss依赖的软件包
```py
yum install -y mysql-server
yum install -y net-snmp net-snmp-utils net-snmp-devel
yum install -y liberation-fonts libgcj libgomp
不要死板按照上面的命令，如果上面这些都安装了，依然提示缺少其他依赖，那就yum安装剩下的依赖。
然后安装zenoss包
```
#### 1.2 安装zenoss
```py
rpm -ivh zenoss-3.2.1.el5.x86_64.rpm
有warning不怕，无视掉，进行下一步配置
```
#### 1.3 启动zenoss
```py
service mysqld start
/etc/init.d/zenoss start
到此为止，zenoss安装完毕。

```
#### 1.4 配置防火墙
```py
service iptables start
chkconfig iptables on
iptables -A INPUT -p tcp --dport 8080 -j ACCEPT
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 8080
/sbin/service iptables save

```
### 二、升级
#### 2.1 替换如下几个目录：
- Products目录
- Extentions目录
- ZenPacks目录
- 以及一个文件 /etc/my.cnf，更换这个文件后记得设置权限并重启mysql。
```py
# chmod 644 /etc/my.cnf
# service mysqld restart
然后进入bin目录,执行
# rm -rf zenperfsql zenperfwmi
 
# ln -s /opt/zenoss/ZenPacks/ZenPacks.community.SQLDataSource-2.2.egg/ZenPacks/community/SQLDataSource/daemons/zenperfsql
 
# ln -s /opt/zenoss/ZenPacks/ZenPacks.community.WMIDataSource-2.11.egg/ZenPacks/community/WMIDataSource/daemons/zenperfwmi
 
# su - zenoss zenoss restart
然后要注意，zenoss文件夹里的所有文件，其所有者都要改为zenoss。
```
以新的products目录为例：
```py
chown -R zenoss:zenoss Products/
这时候重启zenoss，会报一个zenactions.py的错误。
把附件中的文件替换掉 Products/ZenEvents/ 下的同名文件就好了
然后就剩插件了，删除ZenPacks目录里的五个文件
首先去web页面的设备管理页面，左侧的目录树，手工更改，改成和云悦一样的目录结构。
然后删除/opt/zenoss/ZenPacks目录里的五个文件
ZenPacks.community.OpenSolaris-2.1-py2.6.egg
ZenPacks.teamsun.HPUX-1.9-py2.6.egg
ZenPacks.zenoss.LinuxMonitor-1.3-py2.6.egg
ZenPacks.its.AIX-1.3-py2.6.egg
ZenPacks.its.checkping-1.0-py2.6.egg
重启zenoss，然后去web页面这个网址
yourip:/zport/dmd/ZenPackManager/viewZenPacks
这里有个小齿轮，点击，选择 install ZenPack
把附件中的五个插件依次安装(最好是每安装一个，就以zenoss用户重启一次zopectl服务，然后刷新页面)
zopectl restart
```
如果不需要数据迁移的话，就到此为止了。


### 三、数据迁移
数据主要是在三个地方：
- perf目录 性能数据
- var目录 配置数据
- mysql 告警数据
- perf目录  

3.1 直接整个perf目录替换掉。 其中这里两个子目录 Daemons是采集进程的相关配置数据，Devices目录是采集到的所有数据。
3.2 里面都是rrd文件，要注意的是，如果旧机器和新机器的位数不同，比如一个32位一个64位的，那这些rrd文件都是无效的，切记切记。一旦遇到这种情况，就放弃旧数据吧。删掉这两个子目录，然后重启zenoss，会自动生成新的。  
3.3 var目录
这里真正有用的，只有Data.fs开头的那几个文件。其他的那些pid文件可以删掉。 然后把var目录整个替换掉。
3.4 mysql数据
这里都是些旧的告警数据，如果不需要旧的告警，这一步可以省略。
```py
mysqldump -uzenoss -pzenoss events > events.sql  #从旧的导出
mysql -uzenoss -pzenoss events < events.sql   #往新的导入
```
其他配置
zenoss数据迁移后，admin的密码应该依照旧机器。如果登不上，可以自行修改密码
zenpass 回车输入新密码
再次强调，zenoss目录下的所有文件的所有者都应该是zenoss。
每次做完更改之后，可以重启zenoss服务，看看进展情况。
计划任务配置如下。
复制如下文件到相应的位置：
```py
/etc/cron.weekly/
/home/zenoss/ 目录下的所有脚本。
之后 crontab -e 打开计划任务，将命令清单9粘贴到crontab
命令清单9:
20 2 * * * /etc/cron.weekly/zenoss
16 */2 * * * su - zenoss -c "/home/zenoss/summary"
*/10 * * * *  su - zenoss -c '/home/zenoss/dobeat'
*/5 * * * *  su - zenoss -c '/home/zenoss/check.sh >> /usr/local/zenoss/log/check_stat.log 2>&1 '
```
然后重启zenoss服务。
到此为止，不出意外的话，数据迁移完毕。


[Zenoss相关文件下载](https://pan.baidu.com/disk/home#list/vmode=list&path=%2F%E5%90%8C%E6%AD%A5%E4%B8%BA%E7%9F%A5%E7%AC%94%E8%AE%B0%2FZenoss)