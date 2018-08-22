## 私有CA服务器的搭建

&emsp;&emsp;数字证书就是互联网通讯中标志通讯各方身份信息,数字证书不是通信双方的身份证书,而是认证机构在数字证书上扣得一个章,确认你的身份证书有效,它是由权威的CA机构颁发用于表示互联网上通信双方的身份。个人搭建的自由CA服务器在互联网中是不被认可的,但是可以在自己的内部网络中使用


### 1、架构图

&emsp;&emsp;首先在根CA进行签署自证证书,然后子CA向根CA申请证书,根CA签署证书后子CA就可以向其他申请者发放证书。此时的子CA服务器相对于根服务器来说是申请者,相对于web服务器申请者是签署者,所以子CA是两个身份,既是申请者又是签署者。      
`三者之间的关系一定要搞清楚,否则在搭建的时候容易出现混乱`

| 主机名 | IP地址 | 角色 | 目录结构 | 
| - | :- | :- | :- | 
| ca.newtv.com | 172.25.101.104 | 根CA服务器 |  /etc/pki/CA |
| subca.newtv.top | 172.25.101.105 | 子CA服务器 | /etc/pki/CA && /etc/pki/tls | 
| www.newtv.com | 172.25.101.106 | 证书申请者  | /etc/pki/tls | 


![](https://github.com/ZongYuWang/image/blob/master/SSL/SSL1.png)

### 2、配置文件
&emsp;&emsp;配置文件 /etc/pki/tls/openssl.cnf省略了一部分配置文件只保留了有关CA的配置。如果服务器为证书签署者的身份那么就会用到此配置文件,此配置文件对于证书申请者是无作用的
```ruby
# yum install openssl
# vim /etc/pki/tls/openssl.cnf
```

```ruby
####################################################################
[ ca ]
default_ca      = CA_default    # 默认的CA配置;CA_default指向下面配置块

####################################################################
[ CA_default ]

dir             = /etc/pki/CA           # CA的默认工作目录 
certs           = $dir/certs            # 认证证书的目录 
crl_dir         = $dir/crl              # 证书吊销列表的路径
database        = $dir/index.txt        # 数据库的索引文件

new_certs_dir   = $dir/newcerts         # 新颁发证书的默认路径
certificate     = $dir/cacert.pem       # 此服务认证证书,如果此服务器为根CA那么这里为自颁发证书
serial          = $dir/serial           # 下一个证书的证书编号
crlnumber       = $dir/crlnumber        # 下一个吊销的证书编号
                                       
crl             = $dir/crl.pem          # The current CRL
private_key     = $dir/private/cakey.pem# CA的私钥
RANDFILE        = $dir/private/.rand    # 随机数文件 

x509_extensions = usr_cert              # The extentions to add to the cert

name_opt        = ca_default            # 命名方式,以ca_default定义为准
cert_opt        = ca_default            # 证书参数,以ca_default定义为准

default_days    = 365                   # 证书默认有效期 
default_crl_days= 30                    # CRl的有效期
default_md      = sha256                # 加密算法
preserve        = no                    # keep passed DN ordering

policy          = policy_match          #policy_match策略生效

# For the CA policy
[ policy_match ]
countryName             = match         # 国家;match表示申请者的申请信息必须与此一致
stateOrProvinceName     = match         # 州、省 
organizationName        = match         # 组织名、公司名      
organizationalUnitName  = optional      # 部门名称;optional表示申请者可以的信息与此可以不一致 
commonName              = supplied
emailAddress            = optional

[ policy_anything ]                     # 由于定义了policy_match策略生效,所以此策略暂未生效
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

```

### 3、目录结构图 
`在搭建之前要把最重要两个目录结构搞清楚,否则搭建失败`

- 根CA服务器:因为只有CA服务器的角色,所以用到的目录只有 /etc/pki/CA 
- 子CA服务器:因为既充当证书签署者,又充当证书申请者的角色。所以两个目录都有用到。子CA服务器是最容易出错的地方。所以要时刻保持头脑清醒 
- web服务器:只是证书申请者的角色,所以用到的目录只有 /etc/pki/tls 

![](https://github.com/ZongYuWang/image/blob/master/SSL/certs1.png)


### 4、步骤
- 根CA生成自己的私钥,生成自签名证书 
- 子CA生成自己的私钥、证书申请文件,将证书申请文件发送给根CA请求根CA签署证书。(此时子CA的身份为申请者) 子CA拿到根CA签署的证书文件,修改配置文件后,可以向其他申请者签署证书(此时子CA为签署者) 
- web服务器,生成自己的私钥、证书申请文件,将证书申请根文件发送给子CA请求子CA签署证书。 

### 5、实例过程

#### 5.1 根CA搭建：

##### 创建所需要的文件：
```ruby
[root@ca ~]# touch /etc/pki/CA/index.txt  # 生成证书索引数据库文件
[root@ca ~]# echo 01 > /etc/pki/CA/serial # 指定第一个颁发证书的序列号 
```
##### 在根CA服务器上创建密钥：
密钥的位置必须为 /etc/pki/CA/private/cakey.pem,这个是openssl.cnf中指定的路径,只要与配置文件中指定的匹配即可

```ruby
[root@ca ~]# (umask 066; openssl genrsa -out private/cakey.pem 2048)
Generating RSA private key, 2048 bit long modulus
...............+++
............+++
e is 65537 (0x10001)

```
##### 根CA自签名证书：

根CA是最顶级的认证机构,没有人能够认证他,所以只能自己认证自己生成自签名证书
```ruby
[root@ca ~]# openssl req -new -x509 -key /etc/pki/CA/private/cakey.pem -days 7300 -out /etc/pki/CA/cacert.pem -days 7300 
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
Organization Name (eg, company) [Default Company Ltd]:CA
Organizational Unit Name (eg, section) []:newtv
Common Name (eg, your name or your server's hostname) []:ca.newtv.com
Email Address []:

# -new: 生成新证书签署请求 
# -x509: 专用于CA生成自签证书 
# -key: 生成请求时用到的私钥文件 
# -days n:证书的有效期限 
# -out /PATH/TO/SOMECERTFILE: 证书的保存路径
```
##### 安装证书文件：
etc/pki/CA/cacert.pem就是生成的自签名证书文件，传到windows机器中。然后双击安装此证书到受信任的根证书颁发机构
`在windows上需要修改一下cacert.pem的格式为cacert.cer才可以双击安装`


![](https://github.com/ZongYuWang/image/blob/master/SSL/ca1.png)

</br>

![](https://github.com/ZongYuWang/image/blob/master/SSL/ca2.png)
![](https://github.com/ZongYuWang/image/blob/master/SSL/ca3.png)


#### 5.2 子CA搭建：
##### 创建所需要的文件：
```ruby
[root@subca ~]# (umask 066; openssl genrsa -out /etc/pki/tls/private/subca.newtv.top.key) 
Generating RSA private key, 2048 bit long modulus
....+++
..........................................................................................................+++
e is 65537 (0x10001)
```

##### 子CA生成证书申请文件：
```ruby
[root@subca ~]# openssl req -new -key /etc/pki/tls/private/subca.newtv.top.key -out /etc/pki/tls/subca.newtv.top.csr 
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
Organization Name (eg, company) [Default Company Ltd]:CA
Organizational Unit Name (eg, section) []:newtv
Common Name (eg, your name or your server's hostname) []:subca.newtv.top
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:newtv123
An optional company name []:newtv


```
##### 将申请文件复制到根CA目录下,请求根CA签署证书:
```ruby
[root@subca ~]# scp /etc/pki/tls/subca.newtv.top.csr 172.25.101.104:/etc/pki/CA/certs/

```

##### 在根CA服务器下,签署子CA的证书：
```ruby
[root@ca ~]# openssl ca -in subca.newtv.top.csr -out subca.newtv.top.crt -days 3650
Using configuration from /etc/pki/tls/openssl.cnf
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number: 1 (0x1)
        Validity
            Not Before: Aug 22 06:14:00 2018 GMT
            Not After : Aug 19 06:14:00 2028 GMT
        Subject:
            countryName               = CN
            stateOrProvinceName       = TianJin
            organizationName          = CA
            organizationalUnitName    = newtv
            commonName                = subca.newtv.top
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            Netscape Comment: 
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier: 
                2D:5B:B0:31:FD:3E:80:FF:76:8C:66:95:35:1B:18:95:DD:99:34:85
            X509v3 Authority Key Identifier: 
                keyid:65:C1:87:85:52:54:21:80:CE:66:A9:BF:10:C5:BF:58:82:45:3F:86

Certificate is to be certified until Aug 19 06:14:00 2028 GMT (3650 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated

```
##### 将签署好的证书发回给子CA服务器,并改名为cacert.pem：
`/etc/pki/CA/cacert.pem 为子CA服务器中openssl.cnf配置文件中的certificate指定的路径`
```ruby
[root@ca ~]# scp subca.newtv.top.crt 172.25.101.105:/etc/pki/CA/cacert.pem
```

![](https://github.com/ZongYuWang/image/blob/master/SSL/subca1.png)
![](https://github.com/ZongYuWang/image/blob/master/SSL/subca2.png)


&emsp;&emsp;子CA要能给别人签署证书,那么还要生成自己的CA私钥存放在/etc/pki/CAprivate/cakey.pem,这个文件路径也是在openssl.cnf中指定的
当然也可以使用之前生成的私钥复制一份改名后放到此路径即可
```ruby
[root@subca ~]# cp /etc/pki/tls/private/subca.newtv.top.key /etc/pki/CA/private/cakey.pem
```

##### 创建所需要的文件,子CA搭建完成,子CA可以给其他人签署证书了
```ruby
[root@subca ~]# touch /etc/pki/CA/index.txt  # 生成证书索引数据库文件
[root@subca ~]# echo 01 > /etc/pki/CA/serial # 指定第一个颁发证书的序列号

```

#### 5.3 web服务器向子CA申请签署证书:

##### 生成密钥:
```ruby
[root@www ~]# (umask 066; openssl genrsa -out /etc/pki/tls/private/www.newtv.com.key 2048)
Generating RSA private key, 2048 bit long modulus
.............................+++
..................................+++
e is 65537 (0x10001)

```
##### 生成申请文件:
```ruby
[root@www ~]# openssl req -new -key /etc/pki/tls/private/www.newtv.com.key -days 365 -out /etc/pki/tls/www.newtv.com.csr
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
Organization Name (eg, company) [Default Company Ltd]:CA
Organizational Unit Name (eg, section) []:newtv
Common Name (eg, your name or your server's hostname) []:www.newtv.com
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:newtv123
An optional company name []:newtv123
```
##### 上传申请文件,请求子CA签署：
```ruby
[root@www ~]# scp www.newtv.com.csr 172.25.101.105:/etc/pki/CA/private/
```

##### 子CA确认信息后,两次y确认签署证书，然后将证书返回给www服务器即可
```ruby
[root@subca ~]# openssl ca -in /etc/pki/CA/private/www.newtv.com.csr -out /etc/pki/CA/certs/www.newtv.com.crt -days 365
Using configuration from /etc/pki/tls/openssl.cnf
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number: 1 (0x1)
        Validity
            Not Before: Aug 22 06:46:31 2018 GMT
            Not After : Aug 22 06:46:31 2019 GMT
        Subject:
            countryName               = CN
            stateOrProvinceName       = TianJin
            organizationName          = CA
            organizationalUnitName    = newtv
            commonName                = www.newtv.com
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            Netscape Comment: 
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier: 
                45:7D:F3:04:A1:F7:47:93:CB:74:57:E5:23:06:33:4E:72:34:75:13
            X509v3 Authority Key Identifier: 
                keyid:2D:5B:B0:31:FD:3E:80:FF:76:8C:66:95:35:1B:18:95:DD:99:34:85

Certificate is to be certified until Aug 22 06:46:31 2019 GMT (365 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated

```
##### 将签署好的证书发回给www服务器,并改名为cacert.pem：
```ruby
[root@subca ~]# scp www.newtv.com.crt 172.25.101.106:/etc/pki/CA/cacert.pem
```

![](https://github.com/ZongYuWang/image/blob/master/SSL/wwwca1.png)