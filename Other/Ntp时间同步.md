## NTP-时钟同步

### 修改时区：
```ruby
[root@localhost ~]# yum install -y ntp*
[root@localhost ~]# cp /etc/localtime /etc/localtime.bak
[root@localhost ~]# ln -svf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
[root@localhost ~]# cat /etc/sysconfig/clock
[root@localhost ~]# ntpdate 0.centos.pool.ntp.org

```


