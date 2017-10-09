## Nmap ##

**安装Nmap**
```py
[root@localhost ~]# rpm -ivh nmap-7.60-1.x86_64.rpm 
Preparing...                ########################################### [100%]
   1:nmap                   ########################################### [100%]

```
- **用主机名和IP地址扫描系统**
```py
[root@localhost ~]# /usr/bin/nmap test-server2
[root@localhost ~]# /usr/bin/nmap 172.30.105.116

Starting Nmap 7.60 ( https://nmap.org ) at 2017-10-09 23:11 CST
Nmap scan report for test-server2 (172.30.105.116)
Host is up (0.00032s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 00:0C:29:CF:B0:6A (VMware)

Nmap done: 1 IP address (1 host up) scanned in 5.80 seconds

```

- **扫描使用-v选项**
```py
[root@localhost ~]# /usr/bin/nmap -v test-server2

Starting Nmap 7.60 ( https://nmap.org ) at 2017-10-09 23:14 CST
Initiating ARP Ping Scan at 23:14
Scanning test-server2 (172.30.105.116) [1 port]
Completed ARP Ping Scan at 23:14, 0.00s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 23:14
Scanning test-server2 (172.30.105.116) [1000 ports]
Discovered open port 22/tcp on 172.30.105.116
Completed SYN Stealth Scan at 23:14, 5.06s elapsed (1000 total ports)
Nmap scan report for test-server2 (172.30.105.116)
Host is up (0.00017s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 00:0C:29:CF:B0:6A (VMware)

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 5.11 seconds
           Raw packets sent: 1991 (87.588KB) | Rcvd: 12 (792B)

```

- **扫描多台主机**  

可以简单的在Nmap命令后加上多个IP地址或主机名来扫描多台主机。
```py

[root@localhost ~]# /usr/bin/nmap test-server2 test-server3

Starting Nmap 7.60 ( https://nmap.org ) at 2017-10-09 23:20 CST
Nmap scan report for test-server2 (172.30.105.116)
Host is up (0.00017s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 00:0C:29:CF:B0:6A (VMware)

Nmap scan report for test-server3 (172.30.105.111)
Host is up (0.0014s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 00:0C:29:FA:19:86 (VMware)

Nmap done: 2 IP addresses (2 hosts up) scanned in 7.33 seconds

```

- **扫描整个子网**  

可以使用*通配符来扫描整个子网或某个范围的IP地址
```py
[root@localhost ~]# /usr/bin/nmap  172.30.105.*

Starting Nmap 7.60 ( https://nmap.org ) at 2017-10-09 23:22 CST
Nmap scan report for 172.30.105.1
Host is up (0.0029s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
23/tcp  open  telnet
80/tcp  open  http
443/tcp open  https
MAC Address: D0:57:4C:B2:15:42 (Cisco Systems)

Nmap scan report for 172.30.105.2
Host is up (0.010s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
23/tcp  open  telnet
80/tcp  open  http
443/tcp open  https
MAC Address: A4:56:30:AD:2F:42 (Cisco Systems)

Nmap scan report for 172.30.105.3
Host is up (0.014s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
23/tcp  open  telnet
80/tcp  open  http
443/tcp open  https
MAC Address: 3C:CE:73:43:70:C2 (Cisco Systems)

Nmap scan report for 172.30.105.5
Host is up (0.017s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
23/tcp  open  telnet
80/tcp  open  http
443/tcp open  https
MAC Address: 00:23:EA:FF:80:42 (Cisco Systems)

Nmap scan report for 172.30.105.6
Host is up (0.0094s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
23/tcp  open  telnet
80/tcp  open  http
443/tcp open  https
MAC Address: A8:B1:D4:3A:9A:42 (Cisco Systems)

Nmap scan report for 172.30.105.7
Host is up (0.0024s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
23/tcp  open  telnet
80/tcp  open  http
443/tcp open  https
MAC Address: 00:26:98:D2:C6:C2 (Cisco Systems)

Nmap scan report for 172.30.105.17
Host is up (0.00074s latency).
Not shown: 999 filtered ports
PORT     STATE SERVICE
3306/tcp open  mysql
MAC Address: 50:7B:9D:C0:8B:05 (Lcfc(hefei) Electronics Technology)

Nmap scan report for 172.30.105.27
Host is up (0.0010s latency).
Not shown: 992 closed ports
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
49152/tcp open  unknown
49156/tcp open  unknown
49157/tcp open  unknown
49163/tcp open  unknown
49176/tcp open  unknown
MAC Address: C8:5B:76:1F:1C:05 (Lcfc(hefei) Electronics Technology)

Nmap scan report for 172.30.105.28
Host is up (0.000034s latency).
Not shown: 987 closed ports
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
443/tcp   open  https
445/tcp   open  microsoft-ds
902/tcp   open  iss-realsecure
912/tcp   open  apex-mesh
3389/tcp  open  ms-wbt-server
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49156/tcp open  unknown
49157/tcp open  unknown
MAC Address: 34:97:F6:7F:02:56 (Asustek Computer)

Nmap scan report for 172.30.105.29
Host is up (0.00039s latency).
All 1000 scanned ports on 172.30.105.29 are filtered
MAC Address: F8:32:E4:A8:94:0C (Asustek Computer)

Nmap scan report for 172.30.105.30
Host is up (0.0027s latency).
Not shown: 994 filtered ports
PORT     STATE SERVICE
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
5357/tcp open  wsdapi
6000/tcp open  X11
8081/tcp open  blackice-icecap
MAC Address: 3C:97:0E:AE:FE:26 (Wistron InfoComm(Kunshan)Co.)

Nmap scan report for 172.30.105.31
Host is up (0.00094s latency).
Not shown: 987 closed ports
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
443/tcp   open  https
445/tcp   open  microsoft-ds
902/tcp   open  iss-realsecure
912/tcp   open  apex-mesh
8000/tcp  open  http-alt
8443/tcp  open  https-alt
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49160/tcp open  unknown
49161/tcp open  unknown
MAC Address: 50:7B:9D:C0:99:4B (Lcfc(hefei) Electronics Technology)

Nmap scan report for 172.30.105.32
Host is up (0.00096s latency).
Not shown: 989 closed ports
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
843/tcp   open  unknown
2869/tcp  open  icslap
7000/tcp  open  afs3-fileserver
8000/tcp  open  http-alt
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
MAC Address: 50:7B:9D:02:24:51 (Lcfc(hefei) Electronics Technology)

Nmap scan report for 172.30.105.33
Host is up (0.00095s latency).
Not shown: 982 closed ports
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
443/tcp   open  https
445/tcp   open  microsoft-ds
554/tcp   open  rtsp
902/tcp   open  iss-realsecure
912/tcp   open  apex-mesh
1025/tcp  open  NFS-or-IIS
1026/tcp  open  LSA-or-nterm
1029/tcp  open  ms-lsa
1031/tcp  open  iad2
1035/tcp  open  multidropper
1046/tcp  open  wfremotertm
2869/tcp  open  icslap
3306/tcp  open  mysql
5357/tcp  open  wsdapi
8080/tcp  open  http-proxy
10243/tcp open  unknown
MAC Address: 28:D2:44:37:92:96 (Lcfc(hefei) Electronics Technology)

Nmap scan report for 172.30.105.35
Host is up (0.000098s latency).
Not shown: 974 closed ports
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
808/tcp   open  ccproxy-http
1433/tcp  open  ms-sql-s
1801/tcp  open  msmq
2103/tcp  open  zephyr-clt
2105/tcp  open  eklogin
2107/tcp  open  msmq-mgmt
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
3389/tcp  open  ms-wbt-server
10000/tcp open  snet-sensor-mgmt
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49156/tcp open  unknown
49157/tcp open  unknown
49158/tcp open  unknown
MAC Address: 00:0C:29:27:5B:2C (VMware)

Nmap scan report for 172.30.105.50
Host is up (0.00096s latency).
Not shown: 990 closed ports
PORT     STATE SERVICE
21/tcp   open  ftp
80/tcp   open  http
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
1801/tcp open  msmq
2103/tcp open  zephyr-clt
2105/tcp open  eklogin
2107/tcp open  msmq-mgmt
3306/tcp open  mysql
MAC Address: 68:F7:28:69:D2:11 (Lcfc(hefei) Electronics Technology)

Nmap scan report for 172.30.105.68
Host is up (0.00059s latency).
All 1000 scanned ports on 172.30.105.68 are filtered
MAC Address: 20:6B:E7:76:69:7B (Tp-link Technologies)

Nmap scan report for test-server3 (172.30.105.111)
Host is up (0.0015s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 00:0C:29:FA:19:86 (VMware)

Nmap scan report for test-server2 (172.30.105.116)
Host is up (0.00015s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 00:0C:29:CF:B0:6A (VMware)

Nmap scan report for 172.30.105.254
Host is up (0.0030s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 64:12:25:4D:AA:61 (Cisco Systems)

Nmap scan report for 172.30.105.115
Host is up (0.0000080s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE
22/tcp open  ssh

Nmap done: 256 IP addresses (21 hosts up) scanned in 29.41 seconds

```

- **扫描一个IP地址范围** 

可以在nmap执行扫描时指定IP范围
```py
[root@localhost ~]# /usr/bin/nmap 172.30.105.100-120

Starting Nmap 7.60 ( https://nmap.org ) at 2017-10-09 23:39 CST
Nmap scan report for test-server3 (172.30.105.111)
Host is up (0.0014s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 00:0C:29:FA:19:86 (VMware)

Nmap scan report for test-server2 (172.30.105.116)
Host is up (0.00016s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 00:0C:29:CF:B0:6A (VMware)

Nmap scan report for 172.30.105.115
Host is up (0.0000030s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE
22/tcp open  ssh

Nmap done: 21 IP addresses (3 hosts up) scanned in 7.84 seconds

```


- **使用IP地址的最后一个字节扫描多台服务器**  

可以简单的指定IP地址的最后一个字节来对多个IP地址进行扫描。例如，我在下面执行中扫描了IP地址172.30.105.116和172.30.105.111
```py
[root@localhost ~]# /usr/bin/nmap 172.30.105.116,111

Starting Nmap 7.60 ( https://nmap.org ) at 2017-10-09 23:27 CST
Nmap scan report for test-server3 (172.30.105.111)
Host is up (0.0014s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 00:0C:29:FA:19:86 (VMware)

Nmap scan report for test-server2 (172.30.105.116)
Host is up (0.00018s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 00:0C:29:CF:B0:6A (VMware)

Nmap done: 2 IP addresses (2 hosts up) scanned in 7.33 seconds

```

- **从一个文件中扫描主机列表**  

如果有多台主机需要扫描且所有主机信息都写在一个文件中，那么你可以直接让nmap读取该文件来执行扫描，如：创建一个名为“nmaptest.txt ”的文本文件，并定义所有你想要扫描的服务器IP地址或主机名
```py
[root@localhost ~]# cat nmaptest.txt 
172.30.105.116
test-server3

```
接下来运行带“iL” 选项的nmap命令来扫描文件中列出的所有IP地址
```py
[root@localhost ~]# nmap -iL nmaptest.txt 

Starting Nmap 7.60 ( https://nmap.org ) at 2017-10-09 23:32 CST
Nmap scan report for test-server2 (172.30.105.116)
Host is up (0.00016s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 00:0C:29:CF:B0:6A (VMware)

Nmap scan report for test-server3 (172.30.105.111)
Host is up (0.0014s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 00:0C:29:FA:19:86 (VMware)

Nmap done: 2 IP addresses (2 hosts up) scanned in 7.32 seconds

```

- **排除一些远程主机后再扫描** 
```py
[root@localhost ~]# /usr/bin/nmap 172.30.105.* --exclude 172.30.105.116

```
- **扫描操作系统信息和路由跟踪**  

使用Nmap，你可以检测远程主机上运行的操作系统和版本。为了启用操作系统和版本检测，脚本扫描和路由跟踪功能，我们可以使用NMAP的“-A“选项。
```py
[root@localhost ~]# /usr/bin/nmap -A 172.30.105.116

Starting Nmap 7.60 ( https://nmap.org ) at 2017-10-09 23:47 CST
Nmap scan report for test-server2 (172.30.105.116)
Host is up (0.00022s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1 (protocol 2.0)
| ssh-hostkey: 
|   2048 f3:f2:8f:67:7b:d1:4c:1b:f5:4a:37:f0:04:0f:19:77 (RSA)
|   256 8a:58:8e:2e:a3:d7:f8:5e:29:c3:5a:9b:73:ae:59:0f (ECDSA)
|_  256 f3:d1:75:dc:d6:6e:0d:f9:60:89:45:44:95:f2:87:d3 (EdDSA)
MAC Address: 00:0C:29:CF:B0:6A (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.10 - 4.8, Linux 3.2 - 4.8
Network Distance: 1 hop

TRACEROUTE
HOP RTT     ADDRESS
1   0.22 ms test-server2 (172.30.105.116)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.78 seconds

```

- **启用Nmap的操作系统探测功能** 

使用选项“-O”和“-osscan-guess”也帮助探测操作系统信息。
```py
[root@localhost ~]# /usr/bin/nmap -O 172.30.105.116

Starting Nmap 7.60 ( https://nmap.org ) at 2017-10-09 23:50 CST
Nmap scan report for test-server2 (172.30.105.116)
Host is up (0.00019s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 00:0C:29:CF:B0:6A (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.10 - 4.8, Linux 3.2 - 4.8
Network Distance: 1 hop

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.22 seconds

```
- **扫描主机侦测防火墙** 
下面的命令将扫描远程主机以探测该主机是否使用了包过滤器或防火墙
```py
[root@localhost ~]# /usr/bin/nmap -sA 172.30.105.116

Starting Nmap 7.60 ( https://nmap.org ) at 2017-10-09 23:52 CST
Nmap scan report for test-server2 (172.30.105.116)
Host is up (0.00020s latency).
Not shown: 999 filtered ports
PORT   STATE      SERVICE
22/tcp unfiltered ssh
MAC Address: 00:0C:29:CF:B0:6A (VMware)

Nmap done: 1 IP address (1 host up) scanned in 5.11 seconds

```

- **扫描主机检测是否有防火墙保护**
扫描主机检测其是否受到数据包过滤软件或防火墙的保护
```py
[root@localhost ~]# /usr/bin/nmap -PN 172.30.105.116

Starting Nmap 7.60 ( https://nmap.org ) at 2017-10-09 23:58 CST
Nmap scan report for test-server2 (172.30.105.116)
Host is up (0.00018s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 00:0C:29:CF:B0:6A (VMware)

Nmap done: 1 IP address (1 host up) scanned in 5.12 seconds

```
- **找出网络中的在线主机**
使用“-sP”选项，我们可以简单的检测网络中有哪些在线主机，该选项会跳过端口扫描和其他一些检测
```py
[root@localhost ~]# /usr/bin/nmap -sP 172.30.105.*

Starting Nmap 7.60 ( https://nmap.org ) at 2017-10-10 00:00 CST
Nmap scan report for 172.30.105.1
Host is up (0.0017s latency).
MAC Address: D0:57:4C:B2:15:42 (Cisco Systems)
Nmap scan report for 172.30.105.2
Host is up (0.0022s latency).
MAC Address: A4:56:30:AD:2F:42 (Cisco Systems)
Nmap scan report for 172.30.105.3
Host is up (0.0027s latency).
MAC Address: 3C:CE:73:43:70:C2 (Cisco Systems)
Nmap scan report for 172.30.105.5
Host is up (0.0021s latency).
MAC Address: 00:23:EA:FF:80:42 (Cisco Systems)
Nmap scan report for 172.30.105.6
Host is up (0.0021s latency).
MAC Address: A8:B1:D4:3A:9A:42 (Cisco Systems)
Nmap scan report for 172.30.105.7
Host is up (0.0021s latency).
MAC Address: 00:26:98:D2:C6:C2 (Cisco Systems)
Nmap scan report for 172.30.105.17
Host is up (0.00054s latency).
MAC Address: 50:7B:9D:C0:8B:05 (Lcfc(hefei) Electronics Technology)
Nmap scan report for 172.30.105.27
Host is up (0.00067s latency).
MAC Address: C8:5B:76:1F:1C:05 (Lcfc(hefei) Electronics Technology)
Nmap scan report for 172.30.105.28
Host is up (0.000028s latency).
MAC Address: 34:97:F6:7F:02:56 (Asustek Computer)
Nmap scan report for 172.30.105.29
Host is up (0.00057s latency).
MAC Address: F8:32:E4:A8:94:0C (Asustek Computer)
Nmap scan report for 172.30.105.30
Host is up (0.00066s latency).
MAC Address: 3C:97:0E:AE:FE:26 (Wistron InfoComm(Kunshan)Co.)
Nmap scan report for 172.30.105.31
Host is up (0.0014s latency).
MAC Address: 50:7B:9D:C0:99:4B (Lcfc(hefei) Electronics Technology)
Nmap scan report for 172.30.105.32
Host is up (0.0013s latency).
MAC Address: 50:7B:9D:02:24:51 (Lcfc(hefei) Electronics Technology)
Nmap scan report for 172.30.105.33
Host is up (0.00064s latency).
MAC Address: 28:D2:44:37:92:96 (Lcfc(hefei) Electronics Technology)
Nmap scan report for 172.30.105.35
Host is up (0.000068s latency).
MAC Address: 00:0C:29:27:5B:2C (VMware)
Nmap scan report for 172.30.105.50
Host is up (0.00096s latency).
MAC Address: 68:F7:28:69:D2:11 (Lcfc(hefei) Electronics Technology)
Nmap scan report for 172.30.105.68
Host is up (0.00041s latency).
MAC Address: 20:6B:E7:76:69:7B (Tp-link Technologies)
Nmap scan report for test-server3 (172.30.105.111)
Host is up (0.0012s latency).
MAC Address: 00:0C:29:FA:19:86 (VMware)
Nmap scan report for test-server2 (172.30.105.116)
Host is up (0.000063s latency).
MAC Address: 00:0C:29:CF:B0:6A (VMware)
Nmap scan report for 172.30.105.254
Host is up (0.0012s latency).
MAC Address: 64:12:25:4D:AA:61 (Cisco Systems)
Nmap scan report for 172.30.105.115
Host is up.
Nmap done: 256 IP addresses (21 hosts up) scanned in 14.43 seconds

```

- **执行快速扫描**
可以使用“-F”选项执行一次快速扫描，仅扫描列在nmap-services文件中的端口而避开所有其它的端口
```py
[root@localhost ~]# /usr/bin/nmap -F 172.30.105.116

Starting Nmap 7.60 ( https://nmap.org ) at 2017-10-10 00:04 CST
Nmap scan report for test-server2 (172.30.105.116)
Host is up (0.00018s latency).
Not shown: 99 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 00:0C:29:CF:B0:6A (VMware)

Nmap done: 1 IP address (1 host up) scanned in 2.27 seconds

```
- **顺序扫描端口**
使用“–r”选项表示不会随机的选择端口扫描
```py
[root@localhost ~]# /usr/bin/nmap -r 172.30.105.116

Starting Nmap 7.60 ( https://nmap.org ) at 2017-10-10 00:13 CST
Nmap scan report for test-server2 (172.30.105.116)
Host is up (0.00017s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 00:0C:29:CF:B0:6A (VMware)

Nmap done: 1 IP address (1 host up) scanned in 5.53 seconds

```

- **打印主机接口和路由**
可以使用nmap的“–iflist”选项检测主机接口和路由信息
```py
[root@localhost ~]# /usr/bin/nmap --iflist

Starting Nmap 7.60 ( https://nmap.org ) at 2017-10-10 00:19 CST
************************INTERFACES************************
DEV  (SHORT) IP/MASK                     TYPE     UP MTU   MAC
lo   (lo)    127.0.0.1/8                 loopback up 16436
lo   (lo)    ::1/128                     loopback up 16436
eth0 (eth0)  172.30.105.115/24           ethernet up 1500  00:0C:29:7B:8C:25
eth0 (eth0)  fe80::20c:29ff:fe7b:8c25/64 ethernet up 1500  00:0C:29:7B:8C:25

**************************ROUTES**************************
DST/MASK                     DEV  METRIC GATEWAY
172.30.105.0/24              eth0 0
169.254.0.0/16               eth0 1002
0.0.0.0/0                    eth0 0      172.30.105.254
::1/128                      lo   0
fe80::20c:29ff:fe7b:8c25/128 lo   0
fe80::/64                    eth0 256
ff00::/8                     eth0 256

```
- **扫描特定的端口**
使用Nmap扫描远程机器的端口有各种选项，你可以使用“-P”选项指定你想要扫描的端口，默认情况下nmap只扫描TCP端口
```py
[root@localhost ~]# /usr/bin/nmap -p 22 test-server2

Starting Nmap 7.60 ( https://nmap.org ) at 2017-10-10 00:20 CST
Nmap scan report for test-server2 (172.30.105.116)
Host is up (0.00018s latency).

PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 00:0C:29:CF:B0:6A (VMware)

Nmap done: 1 IP address (1 host up) scanned in 0.05 seconds

```

- **扫描TCP端口**
可以指定具体的端口类型和端口号来让nmap扫描
```py
[root@localhost ~]# /usr/bin/nmap -p -T:8888,80 test-server2

```

- **扫描UDP端口**
```py
[root@localhost ~]# /usr/bin/nmap -sU 53 test-server2

```
- **扫描多个端口**
可以使用选项“-P”来扫描多个端口
```py
[root@localhost ~]# /usr/bin/nmap -p 80,443 test-server2

```
- **扫描指定范围内的端口**
```py
[root@localhost ~]# /usr/bin/nmap -p 80-160 test-server2

```

- **查找主机服务版本号**
可以使用“-sV”选项找出远程主机上运行的服务版本
```py
[root@localhost ~]# /usr/bin/nmap -sV test-server2

Starting Nmap 7.60 ( https://nmap.org ) at 2017-10-10 00:39 CST
Nmap scan report for test-server2 (172.30.105.116)
Host is up (0.00018s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1 (protocol 2.0)
MAC Address: 00:0C:29:CF:B0:6A (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 5.82 seconds

```
- **使用TCP ACK (PA)和TCP Syn (PS)扫描远程主机**
有时候包过滤防火墙会阻断标准的ICMP ping请求，在这种情况下，我们可以使用TCP ACK和TCP Syn方法来扫描远程主机
```py
[root@localhost ~]# /usr/bin/nmap -PS test-server3

Starting Nmap 7.60 ( https://nmap.org ) at 2017-10-10 00:40 CST
Nmap scan report for test-server3 (172.30.105.111)
Host is up (0.0014s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 00:0C:29:FA:19:86 (VMware)

Nmap done: 1 IP address (1 host up) scanned in 5.11 seconds

```

- **使用TCP ACK扫描远程主机上特定的端口**
```py
[root@localhost ~]# /usr/bin/nmap -PA -p 22,80 test-server3

Starting Nmap 7.60 ( https://nmap.org ) at 2017-10-10 00:41 CST
Nmap scan report for test-server3 (172.30.105.111)
Host is up (0.0011s latency).

PORT   STATE    SERVICE
22/tcp open     ssh
80/tcp filtered http
MAC Address: 00:0C:29:FA:19:86 (VMware)

Nmap done: 1 IP address (1 host up) scanned in 0.05 seconds

```
- **使用TCP Syn扫描远程主机上特定的端口**
```py
[root@localhost ~]# /usr/bin/nmap -PS -p 22,80 test-server3

Starting Nmap 7.60 ( https://nmap.org ) at 2017-10-10 00:42 CST
Nmap scan report for test-server3 (172.30.105.111)
Host is up (0.0012s latency).

PORT   STATE    SERVICE
22/tcp open     ssh
80/tcp filtered http
MAC Address: 00:0C:29:FA:19:86 (VMware)

Nmap done: 1 IP address (1 host up) scanned in 0.04 seconds

```

- **执行一次隐蔽的扫描**
```py
[root@localhost ~]# /usr/bin/nmap -sS test-server3

Starting Nmap 7.60 ( https://nmap.org ) at 2017-10-10 00:43 CST
Nmap scan report for test-server3 (172.30.105.111)
Host is up (0.0014s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 00:0C:29:FA:19:86 (VMware)

Nmap done: 1 IP address (1 host up) scanned in 5.10 seconds
```

- **使用TCP Syn扫描最常用的端口**
```py
[root@localhost ~]# /usr/bin/nmap -sT test-server3
```
- **执行TCP空扫描以骗过防火墙**
```py

```