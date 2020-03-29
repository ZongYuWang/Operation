## PHP安装与配置

[PHP软件包下载](http://php.net/get/php-7.2.5.tar.gz/from/a/mirror)

### 1、安装PHP7.2
```ruby
[root@localhost php]# yum install -y libxml2-devel bzip2-devel curl-devel libjpeg-devel libpng libpng-devel freetype-devel
```

```ruby
[root@localhost php]# tar xvf php-7.2.5.tar.gz
[root@localhost php]# cd php-7.2.5
```
```ruby
[root@localhost php]# ./configure \
--prefix=/usr/local/php \
--with-config-file-path=/usr/local/php/etc \
--with-apxs2=/usr/local/apache/bin/apxs \
--with-bz2 \
--with-curl \
--enable-ftp \
--enable-sockets \
--disable-ipv6 \
--with-gd \
--with-jpeg-dir=/usr/local \
--with-png-dir=/usr/local \
--with-freetype-dir=/usr/local \
--enable-gd-native-ttf \
--with-iconv-dir=/usr/local \
--enable-mbstring \
--enable-calendar \
--with-gettext \
--with-libxml-dir=/usr/local \
--with-zlib \
--with-pdo-mysql=mysqlnd \
--with-mysqli=mysqlnd \
--with-mysql=mysqlnd \
--enable-dom  \
--enable-xml \
--enable-fpm \
--with-libdir=lib64 \
--enable-bcmath

[root@localhost php]# make && make install 
```
```ruby
[root@localhost ~]# cp php/php.ini-production /usr/local/php/etc/php.ini
[root@localhost ~]# cd /usr/local/php/etc/
[root@localhost etc]# mv php-fpm.conf.default php-fpm.conf
[root@localhost ~]# ln -s /usr/local/php/sbin/* /usr/sbin/

```
```ruby
[root@localhost ~]# vim /usr/local/php/etc/php.ini 
post_max_size = 16M
max_execution_time = 300
max_input_time = 300
mbstring.func_overload = 0

```

FAQ:
```ruby
[root@localhost etc]# /usr/local/php/sbin/php-fpm 
[01-May-2018 13:40:42] WARNING: Nothing matches the include pattern '/usr/local/php/etc/php-fpm.d/*.conf' from /usr/local/php/etc/php-fpm.conf at line 125.
[01-May-2018 13:40:42] ERROR: No pool defined. at least one pool section must be specified in config file
[01-May-2018 13:40:42] ERROR: failed to post process the configuration
[01-May-2018 13:40:42] ERROR: FPM initialization failed

解决办法:
[root@localhost php-fpm.d]# cp www.conf.default www.conf

```

### 启动PHP：
```ruby
[root@localhost ~]# /usr/local/php/sbin/php-fpm 
[root@localhost ~]# ps -ef | grep php
root      85835      1  0 13:43 ?        00:00:00 php-fpm: master process (/usr/local/php/etc/php-fpm.conf)
nobody    85836  85835  0 13:43 ?        00:00:00 php-fpm: pool www          
nobody    85837  85835  0 13:43 ?        00:00:00 php-fpm: pool www          
root      85839   1276  0 13:43 pts/0    00:00:00 grep php

```
```ruby
[root@localhost ~]# netstat -ntlp | grep 9000
tcp        0      0 127.0.0.1:9000              0.0.0.0:*                   LISTEN      85835/php-fpm
```

### 配置Apache：
```ruby
[root@localhost ~]# vim /usr/local/apache/conf/httpd.conf
<IfModule dir_module>
    DirectoryIndex index.html index.php
</IfModule>

AddType application/x-httpd-php .php .html .htm
```