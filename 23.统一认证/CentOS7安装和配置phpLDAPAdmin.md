## CentOS7安装和配置phpLDAPAdmin

### 1、准备环境
#### 1.1 安装YUM源：
```ruby
# rpm -ivh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
# yum -y install epel-release
```

### 2、安装phpLDAPAdmin:
```ruby
# yum install -y phpldapadmin
```

### 3、配置Apache虚拟主机：
```ruby
[root@newtvldap ~]# vim /etc/httpd/conf.d/phpldapadmin.conf

Alias /phpldapadmin /usr/share/phpldapadmin/htdocs
Alias /ldapadmin /usr/share/phpldapadmin/htdocs

<Directory /usr/share/phpldapadmin/htdocs>
  <IfModule mod_authz_core.c>
    # Apache 2.4
    #Require local
    Require all granted
  </IfModule>
  <IfModule !mod_authz_core.c>
    # Apache 2.2
    Order Deny,Allow
    Deny from all
    Allow from 127.0.0.1
    Allow from ::1
  </IfModule>
</Directory>

```
```ruby
# systemctl restart httpd.service
```

### 4、配置phpLDAPAdmin:
```ruby
# vim /etc/phpldapadmin/config.php

$servers->setValue('server','name','newtv Local LDAP Server');
$servers->setValue('server','host','127.0.0.1');
$servers->setValue('server','port',389);
$servers->setValue('server','base',array('dc=newtvldap,dc=com'));

$servers->setValue('login','attr','dn');
// $servers->setValue('login','attr','uid');

```

```ruby
$servers->setValue('login','bind_id','cn=Manager,dc=newtvldap,dc=com');

// 配置了这项在输入框中会默认填写
```
### 5、连接phpLDAPAdmin：
![](https://github.com/ZongYuWang/image/blob/master/LDAP-phpLDAPAdmin/phpLDAPadmin1.png)

</br>

![](https://github.com/ZongYuWang/image/blob/master/LDAP-phpLDAPAdmin/phpLDAPadmin2.png)

</br>

![](https://github.com/ZongYuWang/image/blob/master/LDAP-phpLDAPAdmin/phpLDAPadmin3.png)