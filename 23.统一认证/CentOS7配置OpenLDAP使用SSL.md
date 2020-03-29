## CentOS7配置OpenLDAP使用SSL

### 1、准备工作
```ruby
172.25.101.110  server.newtvldap.com
```
### 2、创建LDAP认证-自定义CA签名证书
也可以使用： [私有CA服务器的搭建](https://github.com/ZongYuWang/Operation/blob/master/23.%E7%BB%9F%E4%B8%80%E8%AE%A4%E8%AF%81/%E7%A7%81%E6%9C%89CA%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%9A%84%E6%90%AD%E5%BB%BA.md)
#### 2.1 创建根认证(newtvldaprootCA.key)：
```ruby
[root@newtvldap ~]# cd /etc/openldap/certs/
[root@newtvldap certs]# openssl genrsa -out newtvldaprootCA.key 2048
Generating RSA private key, 2048 bit long modulus
....+++
...........+++
e is 65537 (0x10001)
```
```ruby
[root@newtvldap certs]# openssl req -x509 -new -nodes -key newtvldaprootCA.key -sha256 -days 1024 -out newtvldaprootCA.pem
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:TianJin
Locality Name (eg, city) [Default City]:TianJin
Organization Name (eg, company) [Default Company Ltd]:newtv
Organizational Unit Name (eg, section) []:newtv
Common Name (eg, your name or your server's hostname) []:newtvldap
Email Address []:
```
#### 2.2 为LDAP服务创建一个私钥（newtvldap.key）：
```ruby
[root@newtvldap certs]# openssl genrsa -out newtvldap.key 2048
Generating RSA private key, 2048 bit long modulus
........+++
......................+++
e is 65537 (0x10001)
```

```ruby
[root@newtvldap certs]# openssl req -new -key newtvldap.key -out newtvldap.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:TianJin
Locality Name (eg, city) [Default City]:TianJin
Organization Name (eg, company) [Default Company Ltd]:newtv
Organizational Unit Name (eg, section) []:newtv
Common Name (eg, your name or your server's hostname) []:newtvldap
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:newtv123
An optional company name []:newtv123

```
#### 2.3 使用自定义根CA签署证书签名请求：
```ruby
[root@newtvldap certs]# openssl \
x509 \
-req \
-in newtvldap.csr \
-CA newtvldaprootCA.pem \
-CAkey newtvldaprootCA.key \
-CAcreateserial \
-out newtvldap.crt \
-days 1460 \
-sha256

Signature ok
subject=/C=CN/ST=TianJin/L=TianJin/O=newtv/OU=newtv/CN=newtvldap
Getting CA Private Key
```
#### 2.4 设置文件权限：
```ruby
# chown -R ldap:ldap /etc/openldap/certs/newtvldap*

```
```ruby
#  ll /etc/openldap/certs/newtvldap*
-rw-r--r--. 1 ldap ldap 1196 Sep 21 05:06 /etc/openldap/certs/newtvldap.crt
-rw-r--r--. 1 ldap ldap 1070 Sep 21 05:03 /etc/openldap/certs/newtvldap.csr
-rw-r--r--. 1 ldap ldap 1679 Sep 21 05:02 /etc/openldap/certs/newtvldap.key
-rw-r--r--. 1 ldap ldap 1675 Sep 21 05:00 /etc/openldap/certs/newtvldaprootCA.key
-rw-r--r--. 1 ldap ldap 1314 Sep 21 05:01 /etc/openldap/certs/newtvldaprootCA.pem
-rw-r--r--. 1 ldap ldap   17 Sep 21 05:06 /etc/openldap/certs/newtvldaprootCA.srl
```

#### 2.5 创建certs.ldif文件以将LDAP配置为使用自签名证书进行安全通信：
```ruby
[root@newtvldap slapd.d]# pwd
/etc/openldap/slapd.d
[root@newtvldap slapd.d]# vim certs.ldif

dn: cn=config
changetype: modify
replace: olcTLSCACertificateFile
olcTLSCACertificateFile: /etc/openldap/certs/newtvldaprootCA.pem

dn: cn=config
changetype: modify
replace: olcTLSCertificateFile
olcTLSCertificateFile: /etc/openldap/certs/newtvldap.crt

dn: cn=config
changetype: modify
replace: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/openldap/certs/newtvldap.key

```
#### 2.6 将认证文件导入LDAP服务：
```ruby
# ldapmodify -Y EXTERNAL  -H ldapi:/// -f certs.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "cn=config"
ldap_modify: Other (e.g., implementation specific) error (80)

```
```ruby
# slaptest -u
config file testing succeeded
```

### 3、配置OpenLDAP通过SSL监听
```ruby
# vim /etc/sysconfig/slapd

SLAPD_URLS="ldapi:/// ldap:/// ldaps:///"

```

```ruby
# systemctl restart slapd
# ss -antup | grep -i 636
tcp    LISTEN     0      128       *:636                   *:*                   users:(("slapd",pid=30740,fd=10))
tcp    LISTEN     0      128      :::636                  :::*                   users:(("slapd",pid=30740,fd=11))

```
### 4、LDAP使用SSL连接测试
```ruby
[root@newtvldap ~]# ldapsearch -x -h ldaps://172.25.101.110 -p 636 -D "cn=user1,ou=people,dc=newtvldap,dc=com" -x -W
Enter LDAP Password: 
# extended LDIF
#
# LDAPv3
# base <> (default) with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# search result
search: 2
result: 32 No such object

# numResponses: 1
```
```ruby
[root@newtvldap ~]# ldapsearch -x -W -h ldaps://172.25.101.110 -p 636 -D "cn=user1,ou=people,dc=newtvldap,dc=com" -b "" -s base -LLL
Enter LDAP Password: 
dn:
objectClass: top
objectClass: OpenLDAProotDSE

```