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

### 5、流程制度安全管控：
`任何一次人为数据库记录的更新，都要走一个流程`
- 人的流程：开发->核心开发->运维或者DBA；
- 测试流程：内网测试->IDC测试->线上测试；
- 客户端管理：PHPMyadmin；

### 6、数据库客户端访问控制：
- 更改默认mysql client端口，如phpmyadmin管理端口位999，其他客户端也是一样；
- 数据库web client端统一部署在1-2台不对外服务的Server上，限制IP及999端口只能从办公区内网访问；
- 不做公网域名解析，用host实现访问(限制任何IP直接访问)或者用内部IP访问；
- PHPMyadmin站点目录独立于所有其它站点根目录外，只能由指定的域名或IP地址访问；
- 限制使用web连接的账号管理数据库，根据开发人员用户角色设置指定账号访问；
- 按开发及相关人员根据职位角色分配适合的管理账号；
- 设置指定账号访问权限层次，WEB层使用apache/nginx账号验证，数据库层使用mysql用户登录验证；
- 统一所有数据库账号登陆入口地址，禁止所有开发人员私自上传PHPMyadmin等数据库管理的程序；
- 开通VPN、跳板机，只能通过局域网内部IP管理数据库；

### 7、系统层控制：
- 限制或禁止开发人员SSH ROOT管理，通过sudo细化权限，使用日志审计；
- 对phpmyadmin端config等配置文件进行读写权限控制；
- 取消非指定服务器的所有phpmyadmin WEB连接端；
- 禁止非管理人员管理有数据库web client端的服务器的权限；

### 8、MySQL数据库安全权限管理控制思想：

#### 8.1 制度与流程控制：
##### 项目开发制度流程：
办公开发环境——>办公测试环境——>IDC测试环境——>IDC正式环境，通过这种较完善的项目开发制度及流程控制，尽可能的防止潜在的问题隐患发生

##### 数据库更新流程：
开发人员提交需求——>开发主管审核——>部门领导审核——>DBA(运维)审核——>DBA(运维)执行项目开发制度及流程控制的数据库更新步骤(每个步骤都要测试)，最后在IDC正式环境执行；

### 9、账户权限控制：
#### 9.1 内部开发等人员权限分配：
- 权限申请流程要设置规范、合理、让需求不明确者知难而退；
- 办公开发和测试环境可以开放权限，IDC测试和正式环境要严格控制数据库写权限，并且读权限和对外业务服务分离；
- 开发人员正式环境数据库权限分配规则：给单独的不对外服务的正式从库只读权限，不能分配线上正式主库写权限；
- 特殊人员(如领导),需要权限时，我们要问清楚他做什么，如邮件回复，注明用户名、密码、权限范围，多提醒操作注意事项，如果有可能由DBA人员代替其操作；
- 特权账号(all privileges)由DBA控制，禁止在任何客户端上执行特权账号操作(如只能localhost或其他策略)

#### 9.2 WEB账号权限分配制度：
- 写库账号默认权限为select、insert、update、delete，不要给建表改表(create、alter)等权限，更不能是all权限；
- 读库账号默认权限为select(配合mysql read-only参数使用)，确保从库对所有非super权限是只读的；
- 最好专库专账号，不要一个账号管理多个库，碎库特别多的小公司根据情况特殊对待处理；
- 如果是LAMP、LNMP一体在一台服务器的环境，DB权限主机要设置为localhost，避免用root用户作为web连接使用；