## Apache安装与配置

### 1、编译安装Apache2.4
```ruby
[root@localhost ~]# yum groupinstall development tools

```

```ruby
[root@localhost ~]# tar xvf apr-1.5.2.tar.gz
[root@localhost ~]# cd apr-1.5.2
[root@localhost apr-1.5.2]# ./configure --prefix=/usr/local/apr/
[root@localhost apr-1.5.2]# make && make install 
```
```ruby
[root@localhost ~]# tar xf apr-util-1.5.4.tar.gz 
[root@localhost ~]# cd apr-util-1.5.4
[root@localhost apr-util-1.5.4]# ./configure --prefix=/usr/local/apr-util/ --with-apr=/usr/local/apr/
[root@localhost apr-util-1.5.4]# make && make install 
```
```ruby
[root@localhost ~]# unzip -o pcre-8.38.zip
[root@localhost ~]# cd pcre-8.38
[root@localhost pcre-8.38]# ./configure --prefix=/usr/local/pcre
[root@localhost pcre-8.38]# make && make install
```
```ruby
[root@localhost ~]# tar xf httpd-2.4.18.tar.gz 
[root@localhost ~]# cd httpd-2.4.18
[root@localhost httpd-2.4.18]# ./configure  \
--prefix=/usr/local/apache  \
--with-apr=/usr/local/apr  \
--with-apr-util=/usr/local/apr-util/  \
--with-pcre=/usr/local/pcre
[root@localhost httpd-2.4.18]# make && make install
```
### 2、启动Apache：
```ruby
[root@localhost ~]# /usr/local/apache/bin/httpd 
httpd (pid 43823) already running
```
```ruby
[root@localhost ~]# ps -ef | grep http
root      43823      1  0 12:27 ?        00:00:00 /usr/local/apache/bin/httpd --help
daemon    43824  43823  0 12:27 ?        00:00:00 /usr/local/apache/bin/httpd --help
daemon    43825  43823  0 12:27 ?        00:00:00 /usr/local/apache/bin/httpd --help
daemon    43826  43823  0 12:27 ?        00:00:00 /usr/local/apache/bin/httpd --help
root      43919   1276  0 12:30 pts/0    00:00:00 grep http
```