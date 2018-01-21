## nginx

```ruby
[root@nginx ~]# yum groupinstall "Development tools"
[root@nginx ~]# yum install pcre-devel openssl-devel gd-devel
// 后面的编译安装中需要使用到的软件包

```

```ruby
[root@nginx ~]# groupadd nginx
[root@nginx ~]# useradd -g nginx -r -s /sbin/nologin -M nginx

```
```ruby

[root@nginx ~]# mkdir -p  /data/logs/nginx  
// 创建一个存放日志的目录，建议是单独一块磁盘

[root@nginx ~]# tar xf nginx-1.12.2.tar.gz 
```

```ruby
[root@nginx nginx-1.12.2]# ./configure --prefix=/usr/local/nginx \
--user=nginx \
--group=nginx \
--with-http_ssl_module \
--with-http_realip_module \
--with-http_flv_module \
--with-http_image_filter_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_stub_status_module \
--http-log-path=/data/logs/nginx/access.log \
--error-log-path=/data/logs/nginx/error.log

[root@nginx nginx-1.12.2]# make && make install
```

```ruby
启动:
[root@nginx nginx-1.12.2]# /usr/local/nginx/sbin/nginx 
```