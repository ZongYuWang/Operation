## Shell脚本编程-gawk
`报告生成器，格式化文本输出`
```ruby
[root@localhost ~]# which awk
/bin/awk
[root@localhost ~]# ls -l /bin/awk
lrwxrwxrwx. 1 root root 4 Apr 29 13:41 /bin/awk -> gawk
```
### 1、语法：
```ruby
gawk [options] 'program' FILE ...
   options：
      -F：指明输入时用到的字段分隔符；
      -v：var=value，自定义变量；
```
```ruby
[root@localhost ~]# tail -4 /etc/fstab | awk '{print $2,$4}'
/dev/shm defaults
/dev/pts gid=5,mode=620
/sys defaults
/proc defaults

// cut只能识别单个空白或者固定的空白字数
```
### 2、print命令
```ruby
print item1,item2, ...

// 逗号分隔符；
// 输出的各items可以字符串，也可以是数值，当前记录的字段、变量或awk的表达式
// 如果省略item，相当于print $0
```
```ruby
[root@localhost ~]# tail -4 /etc/fstab | awk '{print "hello:$1"}'
hello:$1
hello:$1
hello:$1
hello:$1
[root@localhost ~]# tail -4 /etc/fstab | awk '{print "hello:"$1}'
hello:tmpfs
hello:devpts
hello:sysfs
hello:proc

```
### 3、变量
#### 3.1 内建变量
&emsp;&emsp;FS：(input field seperator) 默认为空白字符
&emsp;&emsp;OFS：(output field seperator) 默认为空白字符
```ruby
[root@localhost ~]# awk -v FS=':' '{print $1}' /etc/passwd
[root@localhost ~]# awk -F':' '{print $1}' /etc/passwd

```
```ruby
[root@localhost ~]# awk -v FS=':' '{print $1,$3,$7}' /etc/passwd  
// 默认的输出使用空格
root 0 /bin/bash
bin 1 /sbin/nologin
daemon 2 /sbin/nologin
adm 3 /sbin/nologin
lp 4 /sbin/nologin
sync 5 /bin/sync

[root@localhost ~]# awk -v FS=':' -v OFS=':' '{print $1,$3,$7}' /etc/passwd  
// 指定输出使用冒号分割
root:0:/bin/bash
bin:1:/sbin/nologin
daemon:2:/sbin/nologin
adm:3:/sbin/nologin
lp:4:/sbin/nologin
sync:5:/bin/sync

```
&emsp;&emsp;RS：(input record seperator) 输入时的换行符
&emsp;&emsp;ORS：(output record seperator) 输出时的换行符
```ruby
[root@localhost ~]# awk -v RS=' ' '{print}' /etc/passwd
// {print $0} $0可以省略，空格为换行符
// 输入换行，就是输入时有空格就也换行，输出的原本是换行还是换行
```
&emsp;&emsp;NF：(number of field) 字段数量
```ruby
[root@localhost ~]# cat /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Sun Apr 29 13:40:31 2018


[root@localhost ~]# awk '{print NF}' /etc/fstab 
// NF前没有$
0
1
2
10

```
&emsp;&emsp;NR：(number of record) 行数
&emsp;&emsp;FNR：各文件分别计数，行数
```ruby
[root@localhost ~]# awk '{print NR}' /etc/fstab 
1
2
3
......
14
15
```
```ruby
[root@localhost ~]# awk '{print FNR}' /etc/fstab /etc/issue
1
2
3
......
14
15
1
2
3

```
&emsp;&emsp;FILENAME：当前文件名
&emsp;&emsp;ARGC：命令行参数的个数
&emsp;&emsp;ARGV：数组，保存的是命令行中所给定的各参数
```ruby
[root@localhost ~]# awk 'BEGIN{print ARGC}' /etc/fstab 
2
[root@localhost ~]# awk 'BEGIN{print ARGV[0]}' /etc/fstab 
awk
[root@localhost ~]# awk 'BEGIN{print ARGV[1]}' /etc/fstab 
/etc/fstab

// awk当做一个参数，后面跟的文件也都当做一个参数
```
#### 3.2 自定义变量
- `-v var=value 变量名区分字符大小写`

```ruby
[root@localhost ~]# awk -v test='hello gawk' 'BEGIN{print test}'
hello gawk
// test已经是一个变量，所以print test中的test不需要加$

[root@localhost ~]# awk 'BEGIN{test="hello gawk";print test}'
hello gawk

```

- 在program中直接定义

### 4、printf命令
#### 4.1 格式符：
```ruby
格式化输出：printf FORMAT，item1，item2，...

// FORMAT必须给出;
// 不会自动换行，需要显式给出换行控制符，\n；
// FORMAT中需要分别为后面的每个item指定一个格式化符号；

格式符：
	%c:显示字符的ASCII码；
	%d,%i:显示十进制整数；
	%e,%E：科学计数法数值显示；
	%f：显示为浮点数；
	%g，%G：以科学计数法或浮点形式显示数值；
	%s：显示字符串；
	%u:无符号整数；
	%%：显示%自身；

```
```ruby
[root@localhost ~]# awk -F: '{printf "%s",$1}' /etc/passwd
rootbindaemonadmlpsyncshutdownhaltmailuucpoperatorgamesgopherftpnobodyvcsasaslauthpostfixsshdzabbixdockeruser1user2user3user4user5user6user7user8user9user10myusermyuser1myuser2myuser3myuser4myuser5myuser6myuser7myuser8myuser9myuser10
```
```ruby
[root@localhost ~]# awk -F: '{printf "%s\n",$1}' /etc/passwd
root
bin
daemon
adm

```
```ruby
[root@localhost ~]# awk -F: '{printf "Username:%s\n",$1}' /etc/passwd
Username:root
Username:bin
Username:daemon
Username:adm

```
```ruby
[root@localhost ~]# awk -F: '{printf "Username:%s,UID:%d\n",$1,$3}' /etc/passwd
Username:root,UID:0
Username:bin,UID:1
Username:daemon,UID:2
Username:adm,UID:3

```

#### 4.2 修饰符：
```ruby
修饰符：
	#[.#]:第一个数字控制显示的宽度，第二个#表示小数点后的精度（%3.1f）；
	-:左对齐；
	+：显示数值的符号；
```

```ruby
[root@localhost ~]# awk -F: '{printf "Username:%15s,UID:%d\n",$1,$3}' /etc/passwd
Username:           root,UID:0
Username:            bin,UID:1
Username:         daemon,UID:2
Username:            adm,UID:3
```
```ruby
[root@localhost ~]# awk -F: '{printf "Username:%-15s,UID:%d\n",$1,$3}' /etc/passwd
Username:root           ,UID:0
Username:bin            ,UID:1
Username:daemon         ,UID:2
Username:adm            ,UID:3
Username:lp             ,UID:4

```
### 5、操作符
#### 5.1 算术操作符：
```ruby
x+y,x-y,x*y,x/y,x^y,x%y
-x
+x:转换为数值
```
#### 5.2 字符串操作符：
```ruby
没有符号的操作符，字符串链接
```

#### 5.3 赋值操作符：
```ruby
=,+=,-=,*=,/=,%=,^=
++,--
```
#### 5.4 比较操作符：
```ruby
>,>=,<,<=,!=,==
```
#### 5.5 模式匹配符：
```ruby
~ ：是否匹配
！~：是否不匹配
```
#### 5.6 逻辑操作符：
```ruby
&&
||
!
```
#### 5.7 条件表达式：
```ruby
selector?if-true-expression:if-false-expression
```
```ruby
[root@localhost ~]# awk -F: '{$3>=1000?usertype="Common User":usertype="Sysadmin or SysUser";printf "%15s:%-s\n",$1,usertype}' /etc/passwd
           root:Sysadmin or SysUser
            bin:Sysadmin or SysUser
         daemon:Sysadmin or SysUser
            adm:Sysadmin or SysUser

```
### 6、PATTERN
```ruby
empty：空模式，匹配每一行
/regular expression/:仅处理能够被此处的模式匹配到的行
relational expression：关系表达式，结果有"真"有"假"，结果为"真"才会被处理
		真：结果为非0值、非空字符串
地址定界：startline，endline
BEGIN/END模式：
		BEGIN{}:仅在开始处理文件中的文本之前执行一次
		END{}:仅在文本处理完成之后执行一次
```
```ruby
[root@localhost ~]# awk '/^UUID/{print $1}' /etc/fstab
UUID=23dbdffc-4590-47a5-b5d6-f06d83f2a297
[root@localhost ~]# 
[root@localhost ~]# awk '!/^UUID/{print $1}' /etc/fstab

```
```ruby
[root@localhost ~]# awk -F: '$3>=100 {print $1,$3}' /etc/passwd
saslauth 499
zabbix 498

// 这就是关系表达式，只有$3>=100为真了才会处理
```
```ruby
[root@localhost ~]# awk -F: '$NF=="/bin/bash"{print $1,$NF}' /etc/passwd
root /bin/bash
docker /bin/bash
user1 /bin/bash
user2 /bin/bash

```
```ruby
// 地址定界：
[root@localhost ~]# awk -F: '/^root/,/^myuser/{print $1}' /etc/passwd
// 使用行号定界效果无效

[root@localhost ~]# awk -F: '(NR>=10&&NR<=20){print $1}' /etc/passwd
uucp
operator
games
gopher
ftp
nobody
vcsa
saslauth
postfix
sshd
zabbix
```
```ruby
[root@localhost ~]# awk -F: 'BEGIN{print "   username      uid    \n========================"}{print $1,$3}' /etc/passwd
   username      uid    
========================
root 0
bin 1
daemon 2
adm 3


[root@localhost ~]# awk -F: '{print "   username      uid    \n========================";print $1,$3}' /etc/passwd

  username      uid    
========================
mail 8
   username      uid    
========================
uucp 10
   username      uid    
========================


[root@localhost ~]# awk -F: 'BEGIN{print "   username      uid    \n========================"}{print $1,$3}END{print "================\n  end          "}' /etc/passwd
   username      uid    
========================
root 0
bin 1
......
myuser9 520
myuser10 521
================
  end          

```
### 7、控制语句
#### 7.1 if-else:
`应用场景：对awk取得的整行或某个字段做条件判断`
```ruby
if(condition) statement [else statement]
```
```ruby
[root@localhost ~]# awk -F: '{if($3>=500)print $1,$3}' /etc/passwd
docker 500
user1 501
user2 502

```
```ruby
[root@localhost ~]# awk -F: '{if($3>=500){printf "Common user:%s\n",$1} else {printf "root or Sysuser:%s\n",$1}}' /etc/passwd
root or Sysuser:root
root or Sysuser:bin
root or Sysuser:daemon

```
```ruby
[root@localhost ~]# awk -F: '{if($NF=="/bin/bash")print $1}' /etc/passwd
root
docker
user1
user2
```
```ruby
[root@localhost ~]# awk '{if(NF>5) print $0}' /etc/fstab
```
```ruby
[root@localhost ~]# df -h | awk -F[%] '/^\/dev/{print $1}' | awk '{if($NF>=5) print $1,$NF"%"}'
/dev/sda1 7%

// awk -F[%]：以%作为分割；
// /^\/dev/：取以/dev/开头的，磁盘占用大于5%的
```
#### 7.2 while循环：
`使用场景：对一行内的多个字段逐一类似处理时使用；对数组中的各元素逐一处理时使用`

```ruby
while(cindition) statement
	条件"真"，进入循环：条件"假"，退出循环

```
```ruby
[root@localhost ~]# awk '/^[[:space:]]*kernel/{i=1;while(i<=NF) {print $i,length($i);i++}}' /etc/grub.conf
kernel 6
/vmlinuz-2.6.32-431.el6.x86_64 30
ro 2
root=/dev/mapper/vg_node1-lv_root 33
rd_NO_LUKS 10
LANG=en_US.UTF-8 16
rd_LVM_LV=vg_node1/lv_swap 26
rd_NO_MD 8
SYSFONT=latarcyrheb-sun16 25
crashkernel=auto 16
rd_LVM_LV=vg_node1/lv_root 26
KEYBOARDTYPE=pc 15
KEYTABLE=us 11
rd_NO_DM 8
rhgb 4
quiet 5

[root@localhost ~]# awk '/^[[:space:]]*kernel/{i=1;while(i<=NF) {if(length($i)>=7) {print $i,length($i)};i++}}' /etc/grub.conf

```
#### 7.3 do-while循环：
`至少执行一次循环体`
```ruby
do statement while(condition)
```
#### 7.4 for循环：
```ruby
for(expr1;expr2;expr3)statement
for(variable assignment;condition;iteration process){for-body}
```
```ruby
[root@localhost ~]# awk '/^[[:space:]]*kernel/{for(i=1;i<=NF;i++) {print $i,length($i)}}' /etc/grub.conf
kernel 6
/vmlinuz-2.6.32-431.el6.x86_64 30
ro 2
```
##### 特殊用法：
- 能够遍历数组中的元素：

```ruby
for(var in array) {for-body}
```
#### 7.5 switch语句：
```ruby
switch(expression) {case VALUE1 or /REGEXP/:statement;case VALUE2 or /REGEXP2/:statement;...;default:statement}
```
#### 7.6 break和continue
```ruby
break [n]
continue
```
#### 7.7 next
`提前结束对本行的处理而直接进入下一行`
```ruby
// 显示ID号为偶数的用户
[root@localhost ~]# awk -F: '{if($3%2!=0) next;print $1,$3}' /etc/passwd
root 0
daemon 2
lp 4
shutdown 6
mail 8

```
### 8、数组

```ruby
[root@localhost ~]# awk 'BEGIN{weekdays["mon"]="Monday";weekdays["tue"]="Tuesday";print weekdays["mon"]}'
Monday

```
- 若要遍历数组中的每个元素，要使用for循环：

```ruby
for(var in array){for-body}
```
```ruby
[root@localhost ~]# awk 'BEGIN{weekdays["mon"]="Monday";weekdays["tue"]="Tuesday";for(i in weekdays) {print weekdays[i]}}'
Monday
Tuesday

// var会遍历arrary的每个索引
```
```ruby
[root@localhost ~]# netstat -tan | awk '/^tcp\>/{state[$NF]++}END{for(i in state) {print i,state[i]}}'
ESTABLISHED 1
LISTEN 4

// state["LISTEN"]++
// state["ESTABLISHED"]++
// ss -tan
```
```ruby
// 统计每个IP访问Apache网站的次数

[root@localhost ~]# tail /var/log/httpd/access_log 
192.168.67.220 - - [12/May/2018:10:12:49 +0800] "GET / HTTP/1.1" 403 4961 "-" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.139 Safari/537.36"
192.168.67.220 - - [12/May/2018:10:12:49 +0800] "GET /icons/apache_pb.gif HTTP/1.1" 200 2326 "http://192.168.67.102/" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.139 Safari/537.36"
192.168.67.220 - - [12/May/2018:10:12:49 +0800] "GET /icons/poweredby.png HTTP/1.1" 200 3956 "http://192.168.67.102/" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.139 Safari/537.36"
192.168.67.220 - - [12/May/2018:10:12:49 +0800] "GET /favicon.ico HTTP/1.1" 404 289 "http://192.168.67.102/" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.139 Safari/537.36"

[root@localhost ~]# awk '{ip[$1]++}END{for(i in ip){print i,ip[i]}}' /var/log/httpd/access_log 
192.168.67.220 4

```
#### 8.1 实验：
&emsp;&emsp; 实验1：统计/etc/fstab文件中每个文件系统类型出现的次数:
```ruby
[root@localhost ~]# awk '/^UUID/{fs[$4]++}END{for(i in fs){print i,fs[i]}}' /etc/fstab 
defaults 1

```
&emsp;&emsp; 实验2：统计指定文件中每个单词出现的次数：
```ruby
[root@localhost ~]# awk '{for(i=1;i<=NF;i++){count[$i]++}}END{for(i in count){print i,count[i]}}' /etc/fstab
```
### 9、函数
#### 9.1 内置函数：
##### 数值处理：
- rand()：返回0和1之间一个随机数

```ruby
[root@localhost ~]# awk 'BEGIN{print rand()}'
0.237788
```
##### 字符串处理：
- length([s]):返回指定字符串的长度；
- sub(r,s,[t]):以r表示的模式来查找t所表示的字符中的匹配的内容，并将其第一次出现替换为s所表示的内容
- gsub(r,s,[t]):以r表示的模式来查找t所表示的字符中的匹配的内容，并将其所有出现均替换为s所表示的内容
- split(s,a,[,r]):以r为分隔符切割字符s,并将切割后的结果保存至a所表示的数组中

```ruby
[root@localhost ~]# netstat -tan | awk '/^tcp\>/{print $0}'
tcp        0      0 0.0.0.0:22                  0.0.0.0:*                   LISTEN      
tcp        0      0 127.0.0.1:25                0.0.0.0:*                   LISTEN      
tcp        0      0 192.168.67.102:22           192.168.67.220:50683        ESTABLISHED 
tcp        0      0 192.168.67.102:22           192.168.67.220:49498        ESTABLISHED 
tcp        0      0 :::80                       :::*                        LISTEN      
tcp        0      0 :::22                       :::*                        LISTEN      
tcp        0      0 ::1:25 
```
```ruby
// 统计每个客户端的链接次数：

[root@localhost ~]# netstat -tan | awk '/^tcp\>/{split($5,ip,":");count[ip[1]]++}END{for (i in count){print i,count[i]}}'
```

#### 9.2 自定义函数