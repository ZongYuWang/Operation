## Linux下phpMyAdmin的安装

安装的软件包 | 软件包版本 |  备注 | 
|:- | :-: | :-| 
Apache | 2.4.6 | yum安装方式 |
PHP| 7.0.27| yum安装方式|
MySQL| 5.5.20| 编译安装方式|
phpMyAdmin |4.7.9 | 需要PHP5.5-7.2和MySQL5.5或更新版本 |


### 1、安装并配置Apache：
#### 1.1 安装Apache：
```ruby
[root@phpmyadmin ~]# yum install httpd

```
#### 1.2 配置phpmyadmin的项目路径：
```ruby

[root@phpmyadmin ~]# mkdir /var/www/html/phpmyadmin
[root@phpmyadmin phpmyadmin]# vim index.php 
<?php
    phpinfo()
?>


[root@phpmyadmin ~]# vim /etc/httpd/conf/vhost.conf
<VirtualHost 172.30.105.106:80>
    ServerName www.mytest.com
    DocumentRoot "/var/www/html/phpmyadmin"
   <Directory "/var/www/html/phpmyadmin">
      Options FollowSymLinks
      require all Granted
   </Directory>
</VirtualHost>

```
#### 1.3 启动Apache：
```ruby
[root@phpmyadmin ~]# systemctl start httpd.service
[root@phpmyadmin ~]# systemctl enable httpd.service 
```


### 2、安装PHP：
```ruby
[root@phpmyadmin ~]# yum install -y http://dl.iuscommunity.org/pub/ius/stable/CentOS/7/x86_64/ius-release-1.0-14.ius.centos7.noarch.rpm
[root@phpmyadmin ~]# yum update -y
[root@phpmyadmin ~]# yum -y install php70u php70u-pdo php70u-mysqlnd php70u-opcache php70u-xml php70u-mcrypt php70u-gd php70u-devel php70u-mysql php70u-intl php70u-mbstring php70u-bcmath php70u-json php70u-iconv php70u-soap

```
```ruby
[root@phpmyadmin ~]# php -v
PHP 7.0.27 (cli) (built: Jan  4 2018 13:39:07) ( NTS )
Copyright (c) 1997-2017 The PHP Group
Zend Engine v3.0.0, Copyright (c) 1998-2017 Zend Technologies
    with Zend OPcache v7.0.27, Copyright (c) 1999-2017, by Zend Technologies

```

### 3、安装并配置phpMyAdmin：
#### 3.1 安装phpMyAdmin：
```ruby
[root@phpmyadmin ~]# tar xvf phpMyAdmin-4.7.9-all-languages.tar.gz 
[root@phpmyadmin ~]# mv /var/www/html/phpmyadmin/phpMyAdmin-4.7.9-all-languages /var/www/html/phpmyadmin/phpMyAdmin

````
#### 3.2 配置phpMyAdmin允许使用服务器的IP地址链接：
```ruby
[root@phpmyadmin phpMyAdmin]# cd /var/www/html/phpmyadmin/phpMyAdmin/libraries
[root@phpmyadmin libraries]# vim config.default.php 
$cfg['AllowArbitraryServer'] = true;

```

### 4、设置MySQL的授权账号：
```ruby

mysql> grant all on *.* to 'phpmyadmin'@'172.30.105.%' identified by 'phpmyadmin';
mysql> flush privileges;

```
