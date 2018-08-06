## CentOS7部署OpenLDAP(2.4.44)

### 1、系统环境配置：

#### 1.1 关闭防火墙和SElinux：
```ruby
# systemctl stop firewall
# systemctl disable firewalld

# setenforce 0
# getenforce 
Permissive
```

#### 1.2 配置时钟同步：
```ruby
# yum install -y ntp
# ntpdate time.windows.com
```
```ruby
# crontab -e
*/5 * * * * ntpdate time.windows.com >/dev/null 2>&1
# crontab -l
*/5 * * * * ntpdate time.windows.com >/dev/null 2>&1
```
### 2、安装启动OpenLDAP：

#### 2.1 安装ldap相关软件包：
```ruby
# yum install -y openldap openldap-* compat-openldap migrationtools
# rpm -qa | grep ldap
openldap-2.4.44-15.el7_5.x86_64
openldap-clients-2.4.44-15.el7_5.x86_64
openldap-servers-sql-2.4.44-15.el7_5.x86_64
compat-openldap-2.3.43-5.el7.x86_64
openldap-servers-2.4.44-15.el7_5.x86_64
openldap-devel-2.4.44-15.el7_5.x86_64
```
#### 2.2 启动ldap：
```ruby
[root@newtvldap ~]# systemctl start slapd
[root@newtvldap ~]# ss -tlnp | grep slapd
LISTEN     0      128          *:389                      *:*                   users:(("slapd",pid=11799,fd=8))
LISTEN     0      128         :::389                     :::*                   users:(("slapd",pid=11799,fd=9))
```
```ruby
[root@newtvldap ldap]# slapd -VV
@(#) $OpenLDAP: slapd 2.4.44 (May 16 2018 09:55:53) $
	mockbuild@c1bm.rdu2.centos.org:/builddir/build/BUILD/openldap-2.4.44/openldap-2.4.44/servers/slapd
```
```ruby
[root@newtvldap ldap]# systemctl status slapd
● slapd.service - OpenLDAP Server Daemon
   Loaded: loaded (/usr/lib/systemd/system/slapd.service; disabled; vendor preset: disabled)
   Active: active (running) since Sun 2018-08-05 22:01:48 EDT; 26min ago
     Docs: man:slapd
           man:slapd-config
           man:slapd-hdb
           man:slapd-mdb
           file:///usr/share/doc/openldap-servers/guide.html
 Main PID: 11799 (slapd)
   CGroup: /system.slice/slapd.service
           └─11799 /usr/sbin/slapd -u ldap -h ldapi:/// ldap:///

Aug 05 22:18:24 newtvldap slapd[11799]: conn=1001 op=0 BIND authcid="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" auth...cn=auth"
Aug 05 22:18:24 newtvldap slapd[11799]: conn=1001 op=0 BIND dn="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" mech=EXTE...0 ssf=71
Aug 05 22:18:24 newtvldap slapd[11799]: conn=1001 op=0 RESULT tag=97 err=0 text=
Aug 05 22:18:24 newtvldap slapd[11799]: conn=1001 op=1 MOD dn="cn=config"
Aug 05 22:18:24 newtvldap slapd[11799]: conn=1001 op=1 MOD attr=olcLogFile
Aug 05 22:18:24 newtvldap slapd[11799]: conn=1001 op=1 RESULT tag=103 err=0 text=
Aug 05 22:18:24 newtvldap slapd[11799]: conn=1001 op=2 MOD dn="cn=config"
Aug 05 22:18:24 newtvldap slapd[11799]: conn=1001 op=2 MOD attr=olcLogLevel
Aug 05 22:18:24 newtvldap slapd[11799]: slapd: line 0: unknown attr "shadowLastChange" in to clause
Aug 05 22:18:24 newtvldap slapd[11799]: <access clause> ::= access to <what> [ by <who> [ <access> ] [ <control> ] ]+ 
                                        <what> ::= * | dn[.<dnstyle>=<DN>] [filter=<filter>] [attrs=<attrspec>]
                                        <attrspec> ::= <attrname> [val[/<matchingRule>][.<attrstyle>]=<value>] | <attrlist>...
Hint: Some lines were ellipsized, use -l to show in full.

```
### 3、配置OpenLDAP：

#### 3.1 配置ldap的配置文件：
```ruby
# cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
# chown ldap:ldap -R /var/lib/ldap/
# chmod 700 -R /var/lib/ldap/
# ll /var/lib/ldap/
total 348
-rwx------. 1 ldap ldap     2048 Aug  5 22:01 alock
-rwx------. 1 ldap ldap   286720 Aug  5 22:01 __db.001
-rwx------. 1 ldap ldap    32768 Aug  5 22:01 __db.002
-rwx------. 1 ldap ldap    49152 Aug  5 22:01 __db.003
-rwx------. 1 ldap ldap      845 Aug  5 22:30 DB_CONFIG
-rwx------. 1 ldap ldap     8192 Aug  5 22:01 dn2id.bdb
-rwx------. 1 ldap ldap    32768 Aug  5 22:01 id2entry.bdb
-rwx------. 1 ldap ldap 10485760 Aug  5 22:01 log.0000000001
```
#### 3.2 生成秘钥串：
```ruby
# slappasswd
New password: wangzy
Re-enter new password: wangzy
{SSHA}w8GmzYcBoWzzAz1qMfJKm1f+OoZWi2ed
```

#### 3.3 编辑并导入ldif格式文件：
##### 导入chrootpw.ldif文件：
```ruby
# vim chrootpw.ldif
# specify the password generated above for "olcRootPW" section
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}w8GmzYcBoWzzAz1qMfJKm1f+OoZWi2ed
```
```
# ldapadd -Y EXTERNAL -H ldapi:/// -f chrootpw.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "olcDatabase={0}config,cn=config"
```
##### 导入cosine.ldif、nis.ldif、inetorgperson.ldif文件：
```ruby
# vim ldapaddBaseSchema.sh
#!/bin/bash
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif

# chmod 755 ldapaddBaseSchema.sh 
```
```ruby
# sh -x ldapaddBaseSchema.sh 
+ ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=cosine,cn=schema,cn=config"

+ ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=nis,cn=schema,cn=config"

+ ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=inetorgperson,cn=schema,cn=config"

```

##### 导入chdomain.ldif文件：

```ruby
# vim chdomain.ldif 
# replace to your own domain name for "dc=***,dc=***" section
# specify the password generated above for "olcRootPW" section
dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth"
  read by dn.base="cn=Manager,dc=newtvldap,dc=com" read by * none

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=newtvldap,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=Manager,dc=newtvldap,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootPW
olcRootPW: {SSHA}w8GmzYcBoWzzAz1qMfJKm1f+OoZWi2ed

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to attrs=userPassword,shadowLastChange by
  dn="cn=Manager,dc=newtvldap,dc=com" write by anonymous auth by self write by * none
olcAccess: {1}to dn.base="" by * read
olcAccess: {2}to * by dn="cn=Manager,dc=newtvldap,dc=com" write by * read

```

```ruby
# ldapmodify -Y EXTERNAL -H ldapi:/// -f chdomain.ldif

// 报错如下：
ldap_modify: Inappropriate matching (18)
	additional info: modify/add: olcRootPW: no equality matching rule

// 将chdomain.ldif文件中的"add"全部替换成"replace"，然后重新执行上面命令即可
```
```ruby
# ldapmodify -Y EXTERNAL -H ldapi:/// -f chdomain.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "olcDatabase={1}monitor,cn=config"

modifying entry "olcDatabase={2}hdb,cn=config"

modifying entry "olcDatabase={2}hdb,cn=config"

modifying entry "olcDatabase={2}hdb,cn=config"

modifying entry "olcDatabase={2}hdb,cn=config"

```

##### 导入base.ldif文件：
```ruby
# vim base.ldif

dn: dc=newtvldap,dc=com
dc: newtvldap
objectClass: top
objectClass: domain

dn: cn=Manager ,dc=newtvldap,dc=com
objectClass: organizationalRole
cn: Manager
description: LDAP Manager

dn: ou=People,dc=newtvldap,dc=com
objectClass: organizationalUnit
ou: People

dn: ou=Group,dc=newtvldap,dc=com
objectClass: organizationalUnit
ou: Group
```

```ruby
[root@gitlab ldap]# ldapadd -x -W -D "cn=Manager,dc=newtvldap,dc=com" -f base.ldif
Enter LDAP Password: wangzy
adding new entry "dc=newtvldap,dc=com"

adding new entry "cn=Manager ,dc=newtvldap,dc=com"

adding new entry "ou=People,dc=newtvldap,dc=com"

adding new entry "ou=Group,dc=newtvldap,dc=com"

```

#### 3.4 开启日志配置：
查看OpenLDAP的日志级别，日志主要用于对OpenLDAP排查
```ruby
# slapd -d ?
Installed log subsystems:

	Any                            (-1, 0xffffffff)
	Trace                          (1, 0x1)
	Packets                        (2, 0x2)
	Args                           (4, 0x4)
	Conns                          (8, 0x8)
	BER                            (16, 0x10)
	Filter                         (32, 0x20)
	Config                         (64, 0x40)
	ACL                            (128, 0x80)
	Stats                          (256, 0x100)
	Stats2                         (512, 0x200)
	Shell                          (1024, 0x400)
	Parse                          (2048, 0x800)
	Sync                           (16384, 0x4000)
	None                           (32768, 0x8000)

NOTE: custom log subsystems may be later installed by specific code
```
##### 导入logLevel.ldif文件：
```ruby
# vim logLevel.ldif 
dn: cn=config
changetype: modify
replace: olcLogLevel
olcLogLevel: stats
```
```ruby
# ldapmodify -Y EXTERNAL -H ldapi:/// -f logLevel.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "cn=config"
```
```ruby
# touch /var/log/slapd.log
# vim /etc/rsyslog.conf +73    // "+73"表示指定位到文件73行
.......
local4.*                                               /var/log/slapd.log
```
```ruby
# systemctl restart rsyslog
# systemctl restart slapd
# systemctl status slapd
```

#### 3.5 测试配置文件：
```ruby
# slaptest -u
config file testing succeeded
```


- 很多文章是修改olcDatabase={2}hdb.ldif或olcDatabase={1}monitor.ldif这些文件，其实按照上面的配置完毕，这些文件也就自动配置好了







查询用户：
```ruby
# ldapsearch -x cn=newtv_user1 -b dc=newtvldap,dc=com
# extended LDIF
#
# LDAPv3
# base <dc=newtvldap,dc=com> with scope subtree
# filter: cn=newtv_user1
# requesting: ALL
#

# newtv_user1, People, newtvldap.com
dn: cn=newtv_user1,ou=People,dc=newtvldap,dc=com
objectClass: posixAccount
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
loginShell: /bin/bash
homeDirectory: /home/newtvuser1
uid: newtvuser1
cn: newtv_user1
uidNumber: 10000
gidNumber: 10000
sn: newtv_user1

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1

```