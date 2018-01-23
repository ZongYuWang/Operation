## MySQL安全设置

### 1、强制Linux不记录敏感历史命令
```ruby
[root@mysql ~]# export HISTCONTROL=ignorespace
[root@mysql ~]#  mysql -u root -p'wangzongyu'  // 命令前加一个空格
[root@mysql ~]# history  //将不会记录上一条历史记录，以防密码泄露
```

### 2、隐藏的mysql_history文件内容
```ruby
记录mysql操作记录：
[root@mysql ~]# cat /root/.mysql_history 

系统历史记录中也会有mysql的操作日志：
[root@mysql ~]# cat /root/.bash_history
```

### 3、删除系统多余账号
安装mysql数据库之后，默认的管理员root密码为空，这样不安全，因此需要为root账号设置密码，删除无用的mysql库内的用户账号并且删除默认存在的test数据库，当然，出了上面的方法，针对mysql数据库的用户处理，更安全的措施例如删除root，添加新的管理员用户
```ruby
增加system并提升为超级管理员，即和root等价的用户，只是名字不同
MariaDB [(none)]> grant all privileges on *.* to system@'localhost' identified by 'wangzongyu' with grant option;

```
```ruby
MariaDB [(none)]> select user,host,password from mysql.user;
+--------+-----------+-------------------------------------------+
| user   | host      | password                                  |
+--------+-----------+-------------------------------------------+
| root   | localhost | *9676FBC3B6BA6FB6672DA4AD25939828D969EA19 |
| root   | mysql     |                                           |
| root   | 127.0.0.1 |                                           |
| root   | ::1       |                                           |
| system | localhost | *9676FBC3B6BA6FB6672DA4AD25939828D969EA19 |
+--------+-----------+-------------------------------------------+
5 rows in set (0.00 sec)

可以一条一条的删除：
MariaDB [(none)]> delete from mysql.user where user='root' and host='::1';

也可以直接将root账号删除：
MariaDB [(none)]> delete from mysql.user where user='root';

MariaDB [(none)]> flush privileges;

```

### 4、防止误操作MySQL数据库
假如对数据库做一个update操作，但是忘记加where条件，那么就会悲剧发生了，所有的记录将全部被修改
```ruby
[root@mysql ~]# mysql --help | grep dummy
  -U, --i-am-a-dummy  Synonym for option --safe-updates, -U.
i-am-a-dummy                      FALSE

// 在mysql命令加上选项-U后，当发出没有WHERE或LIMIT关键字的UPDATE或DELETE时，mysql程序就会拒绝执行
```
```ruby
[root@mysql ~]# mysql -usystem -pwang123 -U
MariaDB [(none)]> delete from babydb.student;
ERROR 1175 (HY000): You are using safe update mode and you tried to update a table without a WHERE that uses a KEY column
// 不加条件无法删除，目的达到
```
做成别名防止误操作：
```ruby
[root@mysql ~]# alias mysql='mysql -U'
[root@mysql ~]# mysql -usystem -pwang123

[root@mysql ~]# echo "alias mysql='mysql -U'" >> /etc/profile
[root@mysql ~]# source /etc/profile
[root@mysql ~]# tail -1 /etc/profile
alias mysql='mysql -U'

```