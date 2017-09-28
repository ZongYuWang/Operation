## Snort ##

- Snort作为一种开源IDS(Intrusion Detection Systems)系统,不但可以主动发现网络内遭受攻击，还可以作为防火墙的补充，虽然不能阻止网络入侵行为，但是能够帮助系统对网络攻击进行报警和分析。


**主要安装包及功能：**
- libpcap:从网卡抓包
- libdnet:通用的网络开发包接口，提供网络地址转换ARP缓存和路由表查询等功能
- DAQ：数据采集库
- libnetfilter_queue：一个函数库，主要为用户空间接收处理内核队列里缓冲的数据包
- flex：一个语法分析器
- Bison：语法分析器经常和flex一起搭配使用
- Zliblg-dev：压缩函数库
- ADODB：一个接口
- GD：绘制图像的GD库
- BASE：分析入侵数据库的前端(Basic Analysis and Security Engine)
- Snort

![](https://github.com/ZongYuWang/Operation/blob/master/image/Snort1.png)

**相关软件包版本：**
- adodb519.tar.gz 
- base-1.4.5.tar.gz
- libdnet-1.12.tgz  
- snort-2.9.7.0.tar.gz
- barnyard2-1.9.tar.gz 
- daq-2.0.4.tar.gz 
- libpcap-1.0.0.tar.gz 
- snortrules-snapshot-2970.tar.gz


**1、准备工作：**
- 使用CentOS-6.5-x86_64-minimal.iso，（用CentOS7后面配置base会报错），需要设置系统的IP和DNS让其可以联网

```py
查看主机名：
[root@localhost ~]# hostname
localhost.localdomain

查看网卡配置：
[root@localhost ~]# vim /etc/sysconfig/network-scripts/ifcfg-eth0

DEVICE=eth0
TYPE=Ethernet
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=none
HWADDR=00:0C:29:7b:8c:25
IPADDR=172.30.105.115
PREFIX=24
GATEWAY=172.30.105.254
DNS1=8.8.8.8
DEFROUTE=yes
IPV4_FAILURE_FATAL=yes
IPV6INIT=no
NAME="System eth0"

```
###### 1.1安装依赖包
```py
[root@localhost ~]# yum install -y gcc flex bison zlib libpcap tcpdump gcc-c++ pcre* zlib* 

```

###### 1.2 更换yum源
```py

[root@localhost ~]# yum install -y wget

[root@localhost ~]# mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
[root@localhost ~]# wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
[root@localhost ~]# yum clean all
[root@localhost ~]# yum makecache
[root@localhost ~]# yum -y update

安装epel源：
[root@localhost ~]# yum install -y epel-release

```

###### 1.3 创建用户和用户组：
```py
[root@localhost ~]# groupadd -g 40000 snort
[root@localhost ~]# useradd snort -u 40000 -d /var/log/snort -s /sbin/nologin -c SNORT_IDS -g snort
```


**2、安装配置LAMP环境：**
###### 2.1安装LAMP：
```py
[root@localhost ~]# yum install -y httpd mysql-server php php-mysql php-mbstring php-mcrypt mysql-devel php-gd

修改php.ini：
[root@localhost ~]# vim /etc/php.ini
第513行：
error_reporting = E_ALL & ~E_NOTICE

设置html目录权限：
[root@localhost ~]# chown -R apache:apache /var/www/html

```

###### 2.2安装PHP插件：
```py
[root@localhost ~]# yum install -y mcrypt libmcrypt libmcrypt-devel

```

###### 2.3安装pear插件：
```py
[root@localhost ~]# yum install -y php-pear
[root@localhost ~]# pear upgrade pear
[root@localhost ~]# pear channel-update pear.php.net
[root@localhost ~]# pear install mail
[root@localhost ~]# pear install Image_Graph-alpha Image_Canvas-alpha Image_Color Numbers_Roman
[root@localhost ~]# pear install  mail_mime

```

**3、安装adodb：**

```py
[root@localhost snort]# tar zxvf adodb519.tar.gz -C /var/www/html
[root@localhost snort]# mv /var/www/html/adodb5 /var/www/html/adodb

设置adodb权限：
[root@localhost ~]# chmod 755 /var/www/html/adodb/

```

**4、安装base：**
```py

[root@localhost snort]# tar zxvf base-1.4.5.tar.gz -C /var/www/html
[root@localhost snort]# mv /var/www/html/base-1.4.5 /var/www/html/base

```
###### 4.1 配置base
```py
[root@localhost ~]# service mysqld start                                         
[root@localhost ~]# service httpd start
[root@localhost ~]# service iptables stop

浏览器打开：
http://172.30.105.115/base/setup/index.php

```
![](https://github.com/ZongYuWang/Operation/blob/master/image/snort.png)
![](https://github.com/ZongYuWang/Operation/blob/master/image/Snort2.png)
![](https://github.com/ZongYuWang/Operation/blob/master/image/Snort3.png)
![](https://github.com/ZongYuWang/Operation/blob/master/image/Snort4.png)
![](https://github.com/ZongYuWang/Operation/blob/master/image/Snort5.png)
![](https://github.com/ZongYuWang/Operation/blob/master/image/Snort6.png)
![](https://github.com/ZongYuWang/Operation/blob/master/image/Snort7.png)
![](https://github.com/ZongYuWang/Operation/blob/master/image/Snort8.png)


**5、安装libdnet：（必须使用这个版本）**
```py
[root@localhost ~]# cd /snort/
[root@localhost snort]# tar xvf libdnet-1.12.tgz
[root@localhost snort]# cd libdnet-1.12
[root@localhost libdnet-1.12]# ./configure && make && make install

```

**6、安装libpcap：**
```py
[root@localhost ~]# cd /snort/
[root@localhost snort]# tar zxvf libpcap-1.0.0.tar.gz
[root@localhost snort]# cd libpcap-1.0.0
[root@localhost libpcap-1.0.0]# ./configure && make && make install
```

**7、安装DAQ：**
```py
[root@localhost ~]# cd /snort/
[root@localhost snort]# tar zxvf daq-2.0.4.tar.gz 
[root@localhost snort]# cd daq-2.0.4
[root@localhost daq-2.0.4]# ./configure && make && make install

```

**8、安装Snort：**
```py
[root@localhost ~]# cd /snort/
[root@localhost snort]# tar zxvf snort-2.9.7.0.tar.gz 
[root@localhost snort]# cd snort-2.9.7.0
[root@localhost snort-2.9.7.0]# ./configure && make && make install
```
###### 8.1 配置Snort：
```py

[root@localhost ~]# mkdir /etc/snort
[root@localhost ~]# chown -R snort:snort /etc/snort
[root@localhost ~]# mkdir /var/log/snort
[root@localhost ~]# chown snort.snort -R /var/log/snort/*
[root@localhost ~]# mkdir /usr/local/lib/snort_dynamicrules
[root@localhost ~]# chown -R snort:snort /usr/local/lib/snort_dynamicrules
[root@localhost ~]# chown -R 755 /usr/local/lib/snort_dynamicrules
[root@localhost ~]# mkdir /etc/snort/rules
[root@localhost ~]# touch /etc/snort/rules/white_list.rules /etc/snort/rules/black_list.rules

[root@localhost ~]# cd /snort/snort-2.9.7.0/etc/
[root@localhost etc]# cp gen-msg.map threshold.conf classification.config reference.config unicode.map snort.conf /etc/snort/

[root@localhost etc]# ls /etc/snort/
classification.config  gen-msg.map  reference.config  rules  snort.conf  threshold.conf  unicode.map


[root@localhost ~]# vim /etc/snort/snort.conf（修改为如下内容）
#第104行
var RULE_PATH /etc/snort/rules
#第105行
var SO_RULE_PATH /etc/snort/so_rules
#第106行
var PREPROC_RULE_PATH /etc/snort/preproc_rules
#第117行
var WHITE_LIST_PATH /etc/snort/rules
#第118行
var BLACK_LIST_PATH /etc/snort/rules
#第193行
config logdir:/var/log/snort
#第528行
output unified2:filename snort.log,limit 128

```
**9、配置snortrules默认规则：**
```py
[root@localhost ~]# cd /snort/
[root@localhost snort]# tar zxvf snortrules-snapshot-2970.tar.gz -C /etc/snort/

[root@localhost snort]# cp /etc/snort/etc/sid-msg.map /etc/snort/

```

**10、配置开机自启动Snort程序：**
```py

[root@localhost ~]# cd /snort/snort-2.9.7.0/rpm/
[root@localhost ~]# cp snortd /etc/init.d/snortd
[root@localhost ~]# cp /snort/snort-2.9.7.0/rpm/snort.sysconfig /etc/sysconfig/snort
[root@localhost ~]# chkconfig --add /etc/init.d/snortd 
[root@localhost ~]# chkconfig snortd on
[root@localhost ~]# cd /usr/sbin/
[root@localhost ~]# ln -s /usr/local/bin/snort snort

```

**11、测试Snort：**
```py
[root@localhost ~]# snort -i eth0 -u snort -g snort -c /etc/snort/snort.conf -l /var/log/snort/
 
【参数解释】：
　-T   指定启动模式：测试(加上这个不会生成数据)
  -i   指定网络接口
  -c   指定配置文件

 Running in Test mode

        --== Initializing Snort ==--
Initializing Output Plugins!
Initializing Preprocessors!
Initializing Plug-ins!
Parsing Rules file "/etc/snort/snort.conf"
PortVar 'HTTP_PORTS' defined :  [ 80:81 311 383 591 593 901 1220 1414 1741 1830 2301 2381 2809 3037 3128 3702 4343 4848 5250 6988 7000:7001 7144:7145 7510 7777 7779 8000 8008 8014 8028 8080 8085 8088 8090 8118 8123 8180:8181 8243 8280 8300 8800 8888 8899 9000 9060 9080 9090:9091 9443 9999 11371 34443:34444 41080 50002 55555 ]
PortVar 'SHELLCODE_PORTS' defined :  [ 0:79 81:65535 ]
PortVar 'ORACLE_PORTS' defined :  [ 1024:65535 ]
PortVar 'SSH_PORTS' defined :  [ 22 ]
PortVar 'FTP_PORTS' defined :  [ 21 2100 3535 ]
PortVar 'SIP_PORTS' defined :  [ 5060:5061 5600 ]
PortVar 'FILE_DATA_PORTS' defined :  [ 80:81 110 143 311 383 591 593 901 1220 1414 1741 1830 2301 2381 2809 3037 3128 3702 4343 4848 5250 6988 7000:7001 7144:7145 7510 7777 7779 8000 8008 8014 8028 8080 8085 8088 8090 8118 8123 8180:8181 8243 8280 8300 8800 8888 8899 9000 9060 9080 9090:9091 9443 9999 11371 34443:34444 41080 50002 55555 ]
PortVar 'GTP_PORTS' defined :  [ 2123 2152 3386 ]
Detection:
   Search-Method = AC-Full-Q
    Split Any/Any group = enabled
    Search-Method-Optimizations = enabled
    Maximum pattern length = 20
Tagged Packet Limit: 256
Loading dynamic engine /usr/local/lib/snort_dynamicengine/libsf_engine.so... done
Loading all dynamic detection libs from /usr/local/lib/snort_dynamicrules...
WARNING: No dynamic libraries found in directory /usr/local/lib/snort_dynamicrules.
  Finished Loading all dynamic detection libs from /usr/local/lib/snort_dynamicrules
Loading all dynamic preprocessor libs from /usr/local/lib/snort_dynamicpreprocessor/...
  Loading dynamic preprocessor library /usr/local/lib/snort_dynamicpreprocessor//libsf_modbus_preproc.so... done
  Loading dynamic preprocessor library /usr/local/lib/snort_dynamicpreprocessor//libsf_imap_preproc.so... done
  Loading dynamic preprocessor library /usr/local/lib/snort_dynamicpreprocessor//libsf_sdf_preproc.so... done
  Loading dynamic preprocessor library /usr/local/lib/snort_dynamicpreprocessor//libsf_ftptelnet_preproc.so... done
  Loading dynamic preprocessor library /usr/local/lib/snort_dynamicpreprocessor//libsf_reputation_preproc.so... done
  Loading dynamic preprocessor library /usr/local/lib/snort_dynamicpreprocessor//libsf_dns_preproc.so... done
  Loading dynamic preprocessor library /usr/local/lib/snort_dynamicpreprocessor//libsf_ssl_preproc.so... done
  Loading dynamic preprocessor library /usr/local/lib/snort_dynamicpreprocessor//libsf_gtp_preproc.so... done
  Loading dynamic preprocessor library /usr/local/lib/snort_dynamicpreprocessor//libsf_smtp_preproc.so... done
  Loading dynamic preprocessor library /usr/local/lib/snort_dynamicpreprocessor//libsf_pop_preproc.so... done
  Loading dynamic preprocessor library /usr/local/lib/snort_dynamicpreprocessor//libsf_dce2_preproc.so... done
  Loading dynamic preprocessor library /usr/local/lib/snort_dynamicpreprocessor//libsf_dnp3_preproc.so... done
  Loading dynamic preprocessor library /usr/local/lib/snort_dynamicpreprocessor//libsf_ssh_preproc.so... done
  Loading dynamic preprocessor library /usr/local/lib/snort_dynamicpreprocessor//libsf_sip_preproc.so... done
  Finished Loading all dynamic preprocessor libs from /usr/local/lib/snort_dynamicpreprocessor/
Log directory = /var/log/snort
WARNING: ip4 normalizations disabled because not inline.
WARNING: tcp normalizations disabled because not inline.
WARNING: icmp4 normalizations disabled because not inline.
WARNING: ip6 normalizations disabled because not inline.
WARNING: icmp6 normalizations disabled because not inline.
Frag3 global config:
    Max frags: 65536
    Fragment memory cap: 4194304 bytes
Frag3 engine config:
    Bound Address: default
    Target-based policy: WINDOWS
    Fragment timeout: 180 seconds
    Fragment min_ttl:   1
    Fragment Anomalies: Alert
    Overlap Limit:     10
    Min fragment Length:     100
      Max Expected Streams: 768
Stream global config:
    Track TCP sessions: ACTIVE
    Max TCP sessions: 262144
    TCP cache pruning timeout: 30 seconds
    TCP cache nominal timeout: 3600 seconds
    Memcap (for reassembly packet storage): 8388608
    Track UDP sessions: ACTIVE
    Max UDP sessions: 131072
    UDP cache pruning timeout: 30 seconds
    UDP cache nominal timeout: 180 seconds
    Track ICMP sessions: INACTIVE
    Track IP sessions: INACTIVE
    Log info if session memory consumption exceeds 1048576
    Send up to 2 active responses
    Wait at least 5 seconds between responses
    Protocol Aware Flushing: ACTIVE
        Maximum Flush Point: 16000
Stream TCP Policy config:
    Bound Address: default
    Reassembly Policy: WINDOWS
    Timeout: 180 seconds
    Limit on TCP Overlaps: 10
    Maximum number of bytes to queue per session: 1048576
    Maximum number of segs to queue per session: 2621
    Options:
        Require 3-Way Handshake: YES
        3-Way Handshake Timeout: 180
        Detect Anomalies: YES
    Reassembly Ports:
      21 client (Footprint) 
      22 client (Footprint) 
      23 client (Footprint) 
      25 client (Footprint) 
      42 client (Footprint) 
      53 client (Footprint) 
      79 client (Footprint) 
      80 client (Footprint) server (Footprint)
      81 client (Footprint) server (Footprint)
      109 client (Footprint) 
      110 client (Footprint) 
      111 client (Footprint) 
      113 client (Footprint) 
      119 client (Footprint) 
      135 client (Footprint) 
      136 client (Footprint) 
      137 client (Footprint) 
      139 client (Footprint) 
      143 client (Footprint) 
      161 client (Footprint) 
      additional ports configured but not printed.
Stream UDP Policy config:
    Timeout: 180 seconds
HttpInspect Config:
    GLOBAL CONFIG
      Detect Proxy Usage:       NO
      IIS Unicode Map Filename: /etc/snort/unicode.map
      IIS Unicode Map Codepage: 1252
      Memcap used for logging URI and Hostname: 150994944
      Max Gzip Memory: 838860
      Max Gzip Sessions: 1807
      Gzip Compress Depth: 65535
      Gzip Decompress Depth: 65535
    DEFAULT SERVER CONFIG:
      Server profile: All
      Ports (PAF): 80 81 311 383 591 593 901 1220 1414 1741 1830 2301 2381 2809 3037 3128 3702 4343 4848 5250 6988 7000 7001 7144 7145 7510 7777 7779 8000 8008 8014 8028 8080 8085 8088 8090 8118 8123 8180 8181 8243 8280 8300 8800 8888 8899 9000 9060 9080 9090 9091 9443 9999 11371 34443 34444 41080 50002 55555 
      Server Flow Depth: 0
      Client Flow Depth: 0
      Max Chunk Length: 500000
      Small Chunk Length Evasion: chunk size <= 10, threshold >= 5 times
      Max Header Field Length: 750
      Max Number Header Fields: 100
      Max Number of WhiteSpaces allowed with header folding: 200
      Inspect Pipeline Requests: YES
      URI Discovery Strict Mode: NO
      Allow Proxy Usage: NO
      Disable Alerting: NO
      Oversize Dir Length: 500
      Only inspect URI: NO
      Normalize HTTP Headers: NO
      Inspect HTTP Cookies: YES
      Inspect HTTP Responses: YES
      Extract Gzip from responses: YES
      Decompress response files:   
      Unlimited decompression of gzip data from responses: YES
      Normalize Javascripts in HTTP Responses: YES
      Max Number of WhiteSpaces allowed with Javascript Obfuscation in HTTP responses: 200
      Normalize HTTP Cookies: NO
      Enable XFF and True Client IP: NO
      Log HTTP URI data: NO
      Log HTTP Hostname data: NO
      Extended ASCII code support in URI: NO
      Ascii: YES alert: NO
      Double Decoding: YES alert: NO
      %U Encoding: YES alert: YES
      Bare Byte: YES alert: NO
      UTF 8: YES alert: NO
      IIS Unicode: YES alert: NO
      Multiple Slash: YES alert: NO
      IIS Backslash: YES alert: NO
      Directory Traversal: YES alert: NO
      Web Root Traversal: YES alert: NO
      Apache WhiteSpace: YES alert: NO
      IIS Delimiter: YES alert: NO
      IIS Unicode Map: GLOBAL IIS UNICODE MAP CONFIG
      Non-RFC Compliant Characters: 0x00 0x01 0x02 0x03 0x04 0x05 0x06 0x07 
      Whitespace Characters: 0x09 0x0b 0x0c 0x0d 
rpc_decode arguments:
    Ports to decode RPC on: 111 32770 32771 32772 32773 32774 32775 32776 32777 32778 32779 
    alert_fragments: INACTIVE
    alert_large_fragments: INACTIVE
    alert_incomplete: INACTIVE
    alert_multiple_requests: INACTIVE
FTPTelnet Config:
    GLOBAL CONFIG
      Inspection Type: stateful
      Check for Encrypted Traffic: YES alert: NO
      Continue to check encrypted data: YES
    TELNET CONFIG:
      Ports: 23 
      Are You There Threshold: 20
      Normalize: YES
      Detect Anomalies: YES
    FTP CONFIG:
      FTP Server: default
        Ports (PAF): 21 2100 3535 
        Check for Telnet Cmds: YES alert: YES
        Ignore Telnet Cmd Operations: YES alert: YES
        Ignore open data channels: NO
      FTP Client: default
        Check for Bounce Attacks: YES alert: YES
        Check for Telnet Cmds: YES alert: YES
        Ignore Telnet Cmd Operations: YES alert: YES
        Max Response Length: 256
SMTP Config:
    Ports: 25 465 587 691 
    Inspection Type: Stateful
    Normalize: ATRN AUTH BDAT DATA DEBUG EHLO EMAL ESAM ESND ESOM ETRN EVFY EXPN HELO HELP IDENT MAIL NOOP ONEX QUEU QUIT RCPT RSET SAML SEND STARTTLS SOML TICK TIME TURN TURNME VERB VRFY X-EXPS XADR XAUTH XCIR XEXCH50 XGEN XLICENSE X-LINK2STATE XQUE XSTA XTRN XUSR CHUNKING X-ADAT X-DRCP X-ERCP X-EXCH50 
    Ignore Data: No
    Ignore TLS Data: No
    Ignore SMTP Alerts: No
    Max Command Line Length: 512
    Max Specific Command Line Length: 
       ATRN:255 AUTH:246 BDAT:255 DATA:246 DEBUG:255 
       EHLO:500 EMAL:255 ESAM:255 ESND:255 ESOM:255 
       ETRN:246 EVFY:255 EXPN:255 HELO:500 HELP:500 
       IDENT:255 MAIL:260 NOOP:255 ONEX:246 QUEU:246 
       QUIT:246 RCPT:300 RSET:246 SAML:246 SEND:246 
       SIZE:255 STARTTLS:246 SOML:246 TICK:246 TIME:246 
       TURN:246 TURNME:246 VERB:246 VRFY:255 X-EXPS:246 
       XADR:246 XAUTH:246 XCIR:246 XEXCH50:246 XGEN:246 
       XLICENSE:246 X-LINK2STATE:246 XQUE:246 XSTA:246 XTRN:246 
       XUSR:246 
    Max Header Line Length: 1000
    Max Response Line Length: 512
    X-Link2State Alert: Yes
    Drop on X-Link2State Alert: No
    Alert on commands: None
    Alert on unknown commands: No
    SMTP Memcap: 838860
    MIME Max Mem: 838860
    Base64 Decoding: Enabled
    Base64 Decoding Depth: Unlimited
    Quoted-Printable Decoding: Enabled
    Quoted-Printable Decoding Depth: Unlimited
    Unix-to-Unix Decoding: Enabled
    Unix-to-Unix Decoding Depth: Unlimited
    Non-Encoded MIME attachment Extraction: Enabled
    Non-Encoded MIME attachment Extraction Depth: Unlimited
    Log Attachment filename: Enabled
    Log MAIL FROM Address: Enabled
    Log RCPT TO Addresses: Enabled
    Log Email Headers: Enabled
    Email Hdrs Log Depth: 1464
SSH config: 
    Autodetection: ENABLED
    Challenge-Response Overflow Alert: ENABLED
    SSH1 CRC32 Alert: ENABLED
    Server Version String Overflow Alert: ENABLED
    Protocol Mismatch Alert: ENABLED
    Bad Message Direction Alert: DISABLED
    Bad Payload Size Alert: DISABLED
    Unrecognized Version Alert: DISABLED
    Max Encrypted Packets: 20  
    Max Server Version String Length: 100  
    MaxClientBytes: 19600 (Default) 
    Ports:
	22
DCE/RPC 2 Preprocessor Configuration
  Global Configuration
    DCE/RPC Defragmentation: Enabled
    Memcap: 102400 KB
    Events: co 
    SMB Fingerprint policy: Disabled
  Server Default Configuration
    Policy: WinXP
    Detect ports (PAF)
      SMB: 139 445 
      TCP: 135 
      UDP: 135 
      RPC over HTTP server: 593 
      RPC over HTTP proxy: None
    Autodetect ports (PAF)
      SMB: None
      TCP: 1025-65535 
      UDP: 1025-65535 
      RPC over HTTP server: 1025-65535 
      RPC over HTTP proxy: None
    Invalid SMB shares: C$ D$ ADMIN$ 
    Maximum SMB command chaining: 3 commands
    SMB file inspection: Disabled
DNS config: 
    DNS Client rdata txt Overflow Alert: ACTIVE
    Obsolete DNS RR Types Alert: INACTIVE
    Experimental DNS RR Types Alert: INACTIVE
    Ports: 53
SSLPP config:
    Encrypted packets: not inspected
    Ports:
      443      465      563      636      989
      992      993      994      995     7801
     7802     7900     7901     7902     7903
     7904     7905     7906     7907     7908
     7909     7910     7911     7912     7913
     7914     7915     7916     7917     7918
     7919     7920
    Server side data is trusted
    Maximum SSL Heartbeat length: 0
Sensitive Data preprocessor config: 
    Global Alert Threshold: 25
    Masked Output: DISABLED
SIP config: 
    Max number of sessions: 40000  
    Max number of dialogs in a session: 4 (Default) 
    Status: ENABLED
    Ignore media channel: DISABLED
    Max URI length: 512  
    Max Call ID length: 80  
    Max Request name length: 20 (Default) 
    Max From length: 256 (Default) 
    Max To length: 256 (Default) 
    Max Via length: 1024 (Default) 
    Max Contact length: 512  
    Max Content length: 2048  
    Ports:
	5060	5061	5600
    Methods:
	  invite cancel ack bye register options refer subscribe update join info message notify benotify do qauth sprack publish service unsubscribe prack
IMAP Config:
    Ports: 143 
    IMAP Memcap: 838860
    MIME Max Mem: 838860
    Base64 Decoding: Enabled
    Base64 Decoding Depth: Unlimited
    Quoted-Printable Decoding: Enabled
    Quoted-Printable Decoding Depth: Unlimited
    Unix-to-Unix Decoding: Enabled
    Unix-to-Unix Decoding Depth: Unlimited
    Non-Encoded MIME attachment Extraction: Enabled
    Non-Encoded MIME attachment Extraction Depth: Unlimited
POP Config:
    Ports: 110 
    POP Memcap: 838860
    MIME Max Mem: 838860
    Base64 Decoding: Enabled
    Base64 Decoding Depth: Unlimited
    Quoted-Printable Decoding: Enabled
    Quoted-Printable Decoding Depth: Unlimited
    Unix-to-Unix Decoding: Enabled
    Unix-to-Unix Decoding Depth: Unlimited
    Non-Encoded MIME attachment Extraction: Enabled
    Non-Encoded MIME attachment Extraction Depth: Unlimited
Modbus config: 
    Ports:
	502
DNP3 config: 
    Memcap: 262144
    Check Link-Layer CRCs: ENABLED
    Ports:
	20000
Reputation config: 
WARNING: Can't find any whitelist/blacklist entries. Reputation Preprocessor disabled.

+++++++++++++++++++++++++++++++++++++++++++++++++++
Initializing rule chains...
5376 Snort rules read
    5376 detection rules
    0 decoder rules
    0 preprocessor rules
5376 Option Chains linked into 207 Chain Headers
0 Dynamic rules
+++++++++++++++++++++++++++++++++++++++++++++++++++

+-------------------[Rule Port Counts]---------------------------------------
|             tcp     udp    icmp      ip
|     src    1764       9       0       0
|     dst    2806     665       0       0
|     any     130       1       4       0
|      nc       3       0       1       0
|     s+d       2       2       0       0
+----------------------------------------------------------------------------

+-----------------------[detection-filter-config]------------------------------
| memory-cap : 1048576 bytes
+-----------------------[detection-filter-rules]-------------------------------
-------------------------------------------------------------------------------

+-----------------------[rate-filter-config]-----------------------------------
| memory-cap : 1048576 bytes
+-----------------------[rate-filter-rules]------------------------------------
| none
-------------------------------------------------------------------------------

+-----------------------[event-filter-config]----------------------------------
| memory-cap : 1048576 bytes
+-----------------------[event-filter-global]----------------------------------
+-----------------------[event-filter-local]-----------------------------------
| none
+-----------------------[suppression]------------------------------------------
| none
-------------------------------------------------------------------------------
Rule application order: activation->dynamic->pass->drop->sdrop->reject->alert->log
Verifying Preprocessor Configurations!
WARNING: flowbits key 'tlsv1.2_handshake' is set but not ever checked.
WARNING: flowbits key 'file.rmf' is set but not ever checked.
WARNING: flowbits key 'cve.2008-4265' is set but not ever checked.
WARNING: flowbits key 'file.fpx' is set but not ever checked.
WARNING: flowbits key 'tlsv1.0_handshake' is set but not ever checked.
WARNING: flowbits key 'file.msi' is set but not ever checked.
WARNING: flowbits key 'ssl_handshake' is set but not ever checked.
WARNING: flowbits key 'file.htc' is set but not ever checked.
WARNING: flowbits key 'file.torrent' is set but not ever checked.
WARNING: flowbits key 'file.ram' is checked but not ever set.
WARNING: flowbits key 'ms.packager' is set but not ever checked.
WARNING: flowbits key 'file.lanman' is set but not ever checked.
WARNING: flowbits key 'file.hhk' is set but not ever checked.
WARNING: flowbits key 'kit.blackhole' is set but not ever checked.
WARNING: flowbits key 'file.dmg' is checked but not ever set.
WARNING: flowbits key 'spyrat_bd' is set but not ever checked.
WARNING: flowbits key 'tlsv1.1_handshake' is set but not ever checked.
WARNING: flowbits key 'hornet.2' is set but not ever checked.
WARNING: flowbits key 'acunetix-scan' is set but not ever checked.
WARNING: flowbits key 'file.zip.winrar.spoof' is set but not ever checked.
WARNING: flowbits key 'imap.cram_md5' is set but not ever checked.
WARNING: flowbits key 'file.wri' is set but not ever checked.
WARNING: flowbits key 'file.file.jpeg' is set but not ever checked.
WARNING: flowbits key 'file.xbm' is set but not ever checked.
WARNING: flowbits key 'file.vwr' is set but not ever checked.
WARNING: flowbits key 'file.xfdl' is set but not ever checked.
138 out of 1024 flowbits in use.

[ Port Based Pattern Matching Memory ]
+- [ Aho-Corasick Summary ] -------------------------------------
| Storage Format    : Full-Q 
| Finite Automaton  : DFA
| Alphabet Size     : 256 Chars
| Sizeof State      : Variable (1,2,4 bytes)
| Instances         : 171
|     1 byte states : 161
|     2 byte states : 10
|     4 byte states : 0
| Characters        : 99807
| States            : 77197
| Transitions       : 8265567
| State Density     : 41.8%
| Patterns          : 5417
| Match States      : 6047
| Memory (MB)       : 39.74
|   Patterns        : 0.60
|   Match Lists     : 1.32
|   DFA
|     1 byte states : 1.09
|     2 byte states : 36.42
|     4 byte states : 0.00
+----------------------------------------------------------------
[ Number of patterns truncated to 20 bytes: 323 ]
pcap DAQ configured to passive.
Acquiring network traffic from "eht0".
Set gid to 40000
Set uid to 40000

        --== Initialization Complete ==--

   ,,_     -*> Snort! <*-
  o"  )~   Version 2.9.7.0 GRE (Build 149) 
   ''''    By Martin Roesch & The Snort Team: http://www.snort.org/contact#team
           Copyright (C) 2014 Cisco and/or its affiliates. All rights reserved.
           Copyright (C) 1998-2013 Sourcefire, Inc., et al.
           Using libpcap version 1.0.0
           Using PCRE version: 7.8 2008-09-05
           Using ZLIB version: 1.2.3

           Rules Engine: SF_SNORT_DETECTION_ENGINE  Version 2.4  <Build 1>
           Preprocessor Object: SF_SIP  Version 1.1  <Build 1>
           Preprocessor Object: SF_SSH  Version 1.1  <Build 3>
           Preprocessor Object: SF_DNP3  Version 1.1  <Build 1>
           Preprocessor Object: SF_DCERPC2  Version 1.0  <Build 3>
           Preprocessor Object: SF_POP  Version 1.0  <Build 1>
           Preprocessor Object: SF_SMTP  Version 1.1  <Build 9>
           Preprocessor Object: SF_GTP  Version 1.1  <Build 1>
           Preprocessor Object: SF_SSLPP  Version 1.1  <Build 4>
           Preprocessor Object: SF_DNS  Version 1.1  <Build 4>
           Preprocessor Object: SF_REPUTATION  Version 1.1  <Build 1>
           Preprocessor Object: SF_FTPTELNET  Version 1.2  <Build 13>
           Preprocessor Object: SF_SDF  Version 1.1  <Build 1>
           Preprocessor Object: SF_IMAP  Version 1.0  <Build 1>
           Preprocessor Object: SF_MODBUS  Version 1.1  <Build 1>

Snort successfully validated the configuration!
Snort exiting


【说明】如果出现“success”的字样说明配置好了

```


**12、添加规则：**
```py

[root@localhost ~]# vim /etc/snort/rules/local.rules

文件最后添加一条检查ping包的规则：
alert tcp any any -> $HOME_NET 22 (msg:"Scanning Port 22";sid:1000002;rev:1;)
alert icmp any any -> 172.30.105.115 any (msg: "IcmP Packet detected";sid:1000001;)

其他规则：
drop icmp any any -> any any (itype:0;msg:"Chan Ping";sid:1000002;)
alert icmp any any -> $HOME_NET 81 (msg:"Scanning Port 81";sid:1000001;rev:1;)
alert tcp any any -> $HOME_NET 22 (msg:"Scanning Port 22";sid:1000002;rev:1;)
alert icmp any any -> any any (msg:"UDP Tesing Rule";sid:1000006;rev:1;)
alert tcp any any -> $HOME_NET 80(msg:"HTTP Test!!!"; classtype:not-suspicious; sid:1000005;  rev:1;)

【说明】
规则注解：
alert                触发规则后做出的动作
icmp                 协议类型
第一个any             源IP（网段），any表示任意
第二个any            源端口，any表示任意
->      　　　　　　  表示方向
第三个any             目标IP（网段），any表示任意
第四个any             目标端口，any表示任意
Msg字符               告警名称
Sid                   id号，个人编写的规则使用1,000,000以上
ID：　　　　　　　　    报警序号
特征：　　　　　　      报警名称 对应Msg字段


[root@localhost ~]# tail -f /var/log/snort/alert 
我本机测试地址就是172.30.105.19
07/05-02:54:22.589329  [**] [1:1000002:1] Scanning Port 22 [**] [Priority: 0] {TCP} 172.30.105.19:60761 -> 172.30.105.115:22
07/05-02:54:22.617650  [**] [1:1000002:1] Scanning Port 22 [**] [Priority: 0] {TCP} 172.30.105.19:60761 -> 172.30.105.115:22
07/05-02:54:22.826095  [**] [1:1000002:1] Scanning Port 22 [**] [Priority: 0] {TCP} 172.30.105.19:60761 -> 172.30.105.115:22
07/05-02:54:23.143549  [**] [1:1000002:1] Scanning Port 22 [**] [Priority: 0] {TCP} 172.30.105.19:60761 -> 172.30.105.115:22
07/05-02:54:23.202371  [**] [1:1000001:0] IcmP Packet detected [**] [Priority: 0] {ICMP} 172.30.105.19 -> 172.30.105.115
07/05-02:54:23.202387  [**] [1:1000001:0] IcmP Packet detected [**] [Priority: 0] {ICMP} 172.30.105.115 -> 172.30.105.19
07/05-02:54:23.343173  [**] [1:1000002:1] Scanning Port 22 [**] [Priority: 0] {TCP} 172.30.105.19:60761 -> 172.30.105.115:22
07/05-02:54:23.374751  [**] [1:1000002:1] Scanning Port 22 [**] [Priority: 0] {TCP} 172.30.105.19:60761 -> 172.30.105.115:22
07/05-02:54:23.510720  [**] [1:1000002:1] Scanning Port 22 [**] [Priority: 0] {TCP} 172.30.105.19:60761 -> 172.30.105.115:22
07/05-02:54:23.545026  [**] [1:1000002:1] Scanning Port 22 [**] [Priority: 0] {TCP} 172.30.105.19:60761 -> 172.30.105.115:22

```
- 【注意】对snort的性能影响最大的是snort的配置设定以及规则集设置。内部瓶颈则主要出现在包解码阶段，要snort检查包的容，那么它比一般的规则都要更加耗费系统资源。启用的检查包内容的规则越多，snort的运行就需要越多的系统资源。如果要激活预处理程序中的某些设置选项，就会需要消耗额外的系统资源。最明显的例子就是启用在frag2预处理程序和stream4预处理程序中的“最大存储容量(memcap)”选项。如果您打算激活大量耗费资源的预处理程序选项，最好确定有足够的硬件资源的支持


**13、配置MySQL：**
```py

启动MySQL：
[root@localhost ~]# service mysqld start

为root用户设置密码：
[root@localhost ~]# mysql
mysql> SET PASSWORD FOR 'root'@'localhost'=PASSWORD('wangzongyu');

以root登陆mysql，并创建snort库：
[root@localhost ~]# mysql -u root -p
mysql> create database snort;

创建名为snort、密码为snort的数据库用户并赋予名为snort数据库权限：
mysql> grant create,select,update,insert,delete on snort.* to snort@localhost identified by 'snort';

创建数据库表：
[root@localhost ~]# mysql -usnort -p -Dsnort < /snort/barnyard2-1.9/schemas/create_mysql 
Enter password: snort

```

**14、安装barnyard2：**
- Snort配置文件中自身含有插件允许将Snort报警记录到Mysql中，但这样以来，系统会形成瓶颈 ，当IDS系统检测到攻击行为，就会用到INSERT语句向数据库里写入数据，导致到UPDATE时非常慢。所以直接将Snort输出到数据库，这种方案的效率并不高。这里就使用外部代理将报警输出到Barnyard2。言而言之Barnyard的作用是读取snort产生的二进制事件文件并存储到MySQL

```py

[root@localhost snort]# tar xvf barnyard2-1.9.tar.gz 
[root@localhost snort]# cd /snort/barnyard2-1.9
[root@localhost barnyard2-1.9]# ./configure --with-mysql --with-mysql-libraries=/usr/lib64/mysql/
[root@localhost barnyard2-1.9]# make && make install 

```
###### 14.1 配置barnyard2
```py

[root@localhost ~]# mkdir /var/log/barnyard2
[root@localhost ~]# chown snort.snort /var/log/barnyard2
[root@localhost ~]# touch /var/log/snort/barnyard2.waldo
[root@localhost ~]# chown snort.snort /var/log/snort/barnyard2.waldo
[root@localhost ~]# cp /snort/barnyard2-1.9/etc/barnyard2.conf /etc/snort/

[root@localhost ~]# vim /etc/snort/barnyard2.conf 
第44行：
config logdir:/var/log/barnyard2

第56、57行：
#   Typical options would be:
#     config hostname:  thor  默认的
#     config interface: eth0  默认的
#     config alert_with_interface_name
config hostname:localhost
config interface:eth0

第136行：
#config waldo_file: /tmp/waldo 默认的
config waldo_file:/var/log/snort/barnyard2.waldo

#第170行：
input unified2

文件最后一行：
output database:log,mysql,user=snort password=snort dbname=snort host=localhost

```

###### 14.2 测试barnyard2
```py
[root@localhost ~]# barnyard2 -c /etc/snort/barnyard2.conf -f /var/log/snort/snort.log -w /var/log/snort/barnyard2.waldo -u snort -g snort -d /var/log/snort/ -v


【参数解释】：
　　-c      指定配置文件
　　-d      指定log目录
　　-f      指定log文件
　　-w      指定waldo文件

Running in Continuous mode

        --== Initializing Barnyard2 ==--
Initializing Input Plugins!
Initializing Output Plugins!
Parsing config file "/etc/snort/barnyard2.conf"
Log directory = /var/log/barnyard2
database: inconsistent cid information for sid=1
          Recovering by rolling forward the cid=4191
database: compiled support for (mysql)
database: configured to use mysql
database: schema version = 107
database:           host = localhost
database:           user = snort
database:  database name = snort
database:    sensor name = localhost:eth0
database:      sensor id = 1
database:     sensor cid = 4192
database:  data encoding = hex
database:   detail level = full
database:     ignore_bpf = no
database: using the "log" facility

        --== Initialization Complete ==--

  ______   -*> Barnyard2 <*-
 / ,,_  \  Version 2.1.9 (Build 263)
 |o"  )~|  By the SecurixLive.com Team: http://www.securixlive.com/about.php
 + '''' +  (C) Copyright 2008-2010 SecurixLive.

           Snort by Martin Roesch & The Snort Team: http://www.snort.org/team.html
           (C) Copyright 1998-2007 Sourcefire Inc., et al.

Using waldo file '/var/log/snort/barnyard2.waldo':
    spool directory = /var/log/snort/
    spool filebase  = snort.log
    time_stamp      = 1499117242
    record_idx      = 8382
Waiting for new spool file

【说明】出现“Waiting for new spool file”表示barnyard2配置成功
【说明】ctrl+c终止测试

^C===============================================================================
Record Totals:
   Records:            0
    Events:            0 (0.000%)
   Packets:            0 (0.000%)
===============================================================================
Packet breakdown by protocol (includes rebuilt packets):
      ETH: 0          (0.000%)
  ETHdisc: 0          (0.000%)
     VLAN: 0          (0.000%)
     IPV6: 0          (0.000%)
  IP6 EXT: 0          (0.000%)
  IP6opts: 0          (0.000%)
  IP6disc: 0          (0.000%)
      IP4: 0          (0.000%)
  IP4disc: 0          (0.000%)
    TCP 6: 0          (0.000%)
    UDP 6: 0          (0.000%)
    ICMP6: 0          (0.000%)
  ICMP-IP: 0          (0.000%)
      TCP: 0          (0.000%)
      UDP: 0          (0.000%)
     ICMP: 0          (0.000%)
  TCPdisc: 0          (0.000%)
  UDPdisc: 0          (0.000%)
  ICMPdis: 0          (0.000%)
     FRAG: 0          (0.000%)
   FRAG 6: 0          (0.000%)
      ARP: 0          (0.000%)
    EAPOL: 0          (0.000%)
  ETHLOOP: 0          (0.000%)
      IPX: 0          (0.000%)
    OTHER: 0          (0.000%)
  DISCARD: 0          (0.000%)
InvChkSum: 0          (0.000%)
   S5 G 1: 0          (0.000%)
   S5 G 2: 0          (0.000%)
    Total: 0         
===============================================================================

```


**15、测试Snort和Barnyard2：**
```py

[root@localhost ~]# snort -q -u snort -g snort -c /etc/snort/snort.conf -i eth0 -D
Spawning daemon child...
My daemon child 45188 lives...
Daemon parent exiting (0)

[root@localhost ~]# barnyard2 -c /etc/snort/barnyard2.conf -d /var/log/snort -f snort.log -w /var/log/snort/barnyard2.waldo -g snort -u snort

查看是否已经存入数据库：
[root@localhost ~]# mysql -u snort -p -D snort -e "select count(*) from event"
Enter password: 
+----------+
| count(*) |
+----------+
|     3546 |
+----------+

mysql> select * from sensor;
+-----+----------------+-----------+--------+--------+----------+----------+
| sid | hostname       | interface | filter | detail | encoding | last_cid |
+-----+----------------+-----------+--------+--------+----------+----------+
|   1 | localhost:eth0 | eth0      | NULL   |      1 |        0 |        0 |
+-----+----------------+-----------+--------+--------+----------+----------+
1 row in set (0.00 sec)


```



**16、测试IDS：**
```py

向IDS的IP发送ping包，base的页面会出现ICMP告警

C:\Users\Administrator>ping 172.30.105.115

正在 Ping 172.30.105.115 具有 32 字节的数据:
来自 172.30.105.115 的回复: 字节=32 时间=1ms TTL=64
来自 172.30.105.115 的回复: 字节=32 时间=1ms TTL=64
来自 172.30.105.115 的回复: 字节=32 时间=1ms TTL=64
来自 172.30.105.115 的回复: 字节=32 时间=1ms TTL=64

```
###### 会变成下面这样
![](https://github.com/ZongYuWang/Operation/blob/master/image/Snort9.png)