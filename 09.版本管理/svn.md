## CentOS7搭建SVN服务器

SVN服务端软件下载
SVN客户端软件下载

### 1.安装SVN服务端：
```ruby
[root@CentOS7 ~]# yum install subversion -y

```

#### 1.2.新建一个目录用于存储SVN所有文件：
```ruby
[root@CentOS7 ~]# mkdir /svn

```

#### 1.3.新建一个资源仓库：
```ruby
[root@CentOS7 ~]# svnadmin create /svn/project

[root@CentOS7 ~]# ls /svn/project/
conf  db  format  hooks  locks  README.txt

hooks目录：放置hook脚本文件的目录
locks目录：用来放置subversion的db锁文件和db_logs锁文件的目录，用来追踪存取文件库的客户端
format文件：是一个文本文件，里面只放了一个整数，表示当前文件库配置的版本号
conf目录：是这个仓库的配置文件（仓库的用户访问账号、权限等）

```
#### 1.4.配置svn服务的配置文件:
```ruby
[root@CentOS7 ~]# vim /svn/project/conf/svnserve.conf
[general]
anon-access = none
auth-access = write
password-db = /svn/project/conf/passwd
authz-db = /svn/project/conf/authz
realm = My Test Repository         #这是个提示信息

```
#### 1.5.添加两个访问用户及口令:
```ruby
[root@CentOS7 ~]# vim /svn/project/conf/passwd 
[users]
wangzy = 123qwe
user1 = 123qwe
user2 = 123qwe

# 对用户配置文件的修改立即生效，不必重启svn服务
```
#### 1.6.配置新用户的授权文件
```ruby
[root@CentOS7 ~]# vim /svn/project/conf/authz 
[groups]
admin = wangzy,user1
user = user2

[/]

@admin = rw
@user =r

# / 表示对根目录（即/svn/project目录）下的所有子目录范围设置权限
# [/abc] 表示对资料库中abc项目设置权限
# 空权限表示禁止访问本目录
```
#### 1.7.启动svn服务
```ruby
[root@CentOS7 ~]# svnserve -d -r /svn/project/

[root@CentOS7 ~]# ps -ef | grep svn
root      32416      1  0 00:07 ?        00:00:00 svnserve -d -r /svn/project/
root      32418  32279  0 00:07 pts/0    00:00:00 grep --color=auto svn


[root@CentOS7 ~]# netstat -tlnp | grep svn
tcp        0      0 0.0.0.0:3690            0.0.0.0:*               LISTEN      32416/svnserve 

```

### 2.使用客户端连接：
打开TortoiseSVN Repository Browser工具
在URL中输入：
svn://192.168.11.229回车，提示输入用户名和口令