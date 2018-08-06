## 配置OpenLDAP使用LAM

[LAM软件包下载地址](https://www.ldap-account-manager.org/lamcms/)

### 1、安装Apache：
```ruby
# yum install -y httpd 
```

```ruby
# systemctl start httpd
```


### 2、安装LAM
```ruby
# wget https://jaist.dl.sourceforge.net/project/lam/LAM/6.4/ldap-account-manager-6.4.tar.bz2
// 不要在windows上下载，很慢
# mv ldap-account-manager-6.4.tar.bz2 /var/www/html/
```
```ruby
# yum install -y bzip2
# bzip2 -d ldap-account-manager-6.4.tar.bz2 
# tar xvf ldap-account-manager-6.4.tar
# mv ldap-account-manager-6.4 lam
# cd ldap/config
# cp config.cfg.sample config.cfg
# chown -R apache.apache /var/www/html/lam
```

```ruby
# cp unix.conf.sample unix.conf
# vim unix.conf
// 也就是把your-domain都改为自己的域名

ServerURL: ldap://localhost:389
Admins: cn=Manager,dc=newtvldap,dc=com
Passwd: lam
treesuffix: dc=newtvldap,dc=com
defaultLanguage: zh_CN.utf8
......
modules: posixGroup_pwdHash: SSHA
modules: posixAccount_pwdHash: SSHA
activeTypes: user,group
types: suffix_user: ou=People,dc=newtvldap,dc=com
types: modules_user: inetOrgPerson,posixAccount,shadowAccount
types: suffix_group: ou=group,dc=newtvldap,dc=com
types: modules_group: posixGroup
lamProMailSubject: Your password was reset
lamProMailText: Dear @@givenName@@ @@sn@@,+::++::+your password was reset to: @@newPassword@@+::++::++::+Best regards+::++::+deskside support+::+
......
loginSearchSuffix: dc=yourdomain,dc=org

// 实施完成这个文件的内容还会变化
```

#### 2.1 登录浏览器WEB访问：
```ruby
http://172.25.101.110/lam

// web页面出现如下问题
Your PHP has no LDAP support!
Please install the LDAP extension for PHP.

Your PHP has no imagick support.
Please install the imagick extension for PHP.

```

#### 2.2、安装PHP7.0支持LDAP扩展：
`不要使用较低的PHP版本(5.6也是比较低的版本)，会报错`

```ruby
# yum install yum-utils
# yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
```
```ruby
# yum-config-manager --enable remi-php70   // 安装php7.0
# yum-config-manager --enable remi-php71   // 安装php7.1
# yum-config-manager --enable remi-php72   // 安装php7.2
```
```ruby
# yum install php php-mcrypt php-cli php-gd php-curl php-mysql php-ldap php-zip php-fileinfo
# php -v
PHP 7.0.31 (cli) (built: Jul 17 2018 15:30:29) ( NTS )
Copyright (c) 1997-2017 The PHP Group
Zend Engine v3.0.0, Copyright (c) 1998-2017 Zend Technologies

```

#### 2.3、安装PHP7.0支持ImageMagick扩展：
```ruby
# yum groupinstall "Development Tools" -y
# yum install ImageMagick ImageMagick-devel -y
# yum install php-pear
# yum install php-devel
# pecl install Imagick

# vim /etc/php.ini
extension=imagick.so
# php -i | grep Imagick
imagick classes => Imagick, ImagickDraw, ImagickPixel, ImagickPixelIterator
Imagick compiled with ImageMagick version => ImageMagick 6.7.8-9 2016-06-16 Q16 http://www.imagemagick.org
Imagick using ImageMagick library version => ImageMagick 6.7.8-9 2016-06-16 Q16 http://www.imagemagick.org
```

### 3、LAM-WEB端配置

![](https://github.com/ZongYuWang/image/blob/master/LDAP-LAM/LAM1.png)

</br>

![](https://github.com/ZongYuWang/image/blob/master/LDAP-LAM/LAM4.png)


</br>
![](https://github.com/ZongYuWang/image/blob/master/LDAP-LAM/LAM9.png)

</br>
![](https://github.com/ZongYuWang/image/blob/master/LDAP-LAM/LAM3.png)

</br>
![](https://github.com/ZongYuWang/image/blob/master/LDAP-LAM/LAM7.png)

</br>
![](https://github.com/ZongYuWang/image/blob/master/LDAP-LAM/LAM8.png)

</br>
![](https://github.com/ZongYuWang/image/blob/master/LDAP-LAM/LAM2.png)

</br>
![](https://github.com/ZongYuWang/image/blob/master/LDAP-LAM/LAM5.png)

</br>
![](https://github.com/ZongYuWang/image/blob/master/LDAP-LAM/LAM6.png)
