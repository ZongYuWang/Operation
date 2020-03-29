## OpenStack(Ocata)-在controller节点安装Horizon

### 1、安装软件包:
```ruby
[root@controller ~]# yum install openstack-dashboard -y

```
### 2、修改配置文件/etc/openstack-dashboard/local_settings：
```ruby
[root@controller ~]# vim /etc/openstack-dashboard/local_settings

```
[openstack-dashboard_local_settings配置文件下载地址](https://github.com/ZongYuWang/image/blob/master/File/openstack-dashboard_local_settings)

### 3、启动dashboard服务并设置开机启动:
```ruby
[root@controller ~]# systemctl restart httpd.service memcached.service
[root@controller ~]# systemctl status httpd.service memcached.service
```
#### 1.7 在WEB中访问：
`在浏览器输入http://172.25.253.104/dashboard.,访问openstack的web页面`
```ruby
admin
wangzy
```