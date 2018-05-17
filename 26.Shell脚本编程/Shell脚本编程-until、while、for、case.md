## Shell脚本编程-if语句
### 1、语法：


## Shell脚本编程-while语句
### 1、语法：
```ruby
while CONDITION;do
	循环体
done

// CONDITION:循环控制条件，进入循环之前，先做一次判断，每一次循环之后再次做判断；
// 条件为"true",则执行一次循环,直到条件状态为"false"终止循环；
// CONDITION一般应该有循环控制变量，而此变量的值会在循环体不断的被修正；
```
### 2、创建死循环
```ruby
while true；do
    循环体
done
```
### 3、while循环的特殊用法：
`遍历文件的每一行`
```ruby
while read line;do
    循环体
done < /PATH/FROM/SOMEFILE
// 依次读取/PATH/FROM/SOMEFILE文件中的每一行，且将行赋值给变量line；
```
### 4、练习：
&emsp;&emsp;练习1：找出其ID号为偶数的所有用户，显示其用户名和ID号
```ruby
[root@node1 ~]# vim evenid.sh
#!/bin/bash
#
while read line;do
    if [ $[`echo $line | cut -d: -f3` % 2] -eq 0 ];then
        echo -e -n "username:`echo $line | cut -d: -f1`\t"
        echo "uid:`echo $line | cut -d: -f3`"
    fi
done < /etc/passwd

[root@node1 ~]# bash evenid.sh 
username:root	uid:0
username:daemon	uid:2
username:lp	uid:4
username:shutdown	uid:6
username:mail	uid:8
username:uucp	uid:10
username:games	uid:12
username:ftp	uid:14
username:sshd	uid:74
username:zabbix	uid:498
username:docker	uid:500
```
&emsp;&emsp;练习2：写一个脚本，完成如下任务
- 显示一个如下的菜单：
  cpu) show cpu information;
  mem) show memory information;
  disk) show disk information;
  quit) quit
- 提示用户选择选项：
- 显示用户选择的内容：
- 用户选择，并显示完成后不退出脚本，而是提示用户继续选择显示其他内容，直到使用quit退出；

```ruby
#!/bin/bash
#

cat << EOF
cpu) show cpu information
mem) show memory information
disk) show disk information
quit) quit
==================================
EOF

read -p "Enter a option:" option
while [ "$option" != "cpu" -a "$option" != "mem" -a "$option" != "disk" -a "$option" != "quit" ];do
    read -p "Wrong option,Enter again:" option
done

if [ "$option" == "cpu" ];then
    lscpu
elif [ "$option" == "mem" ];then
    cat /proc/meminfo
elif [ "$option" == "disk" ];then
    fdisk -l
else
    echo "quit"
    exit 0
fi

[root@node1 ~]# bash -n sysinfo.sh 
[root@node1 ~]# bash sysinfo.sh 
cpu) show cpu information
mem) show memory information
disk) show disk information
quit) quit
==================================
Enter a option:a
Wrong option,Enter again:b
Wrong option,Enter again:cpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                1
On-line CPU(s) list:   0
Thread(s) per core:    1

```
&emsp;&emsp;练习3：添加10个用户：
```ruby
[root@node1 ~]# vim useradd.sh
#!/bin/bash
#

declare -i i=1
declare -i users=0

while [ $i -le 10 ];do
    if ! id user$i &> /dev/null;then
        useradd user$i
        echo "Add user:user$i."
        let users++
    fi
    let i++
done
echo "Add $users Users."

[root@node1 ~]# bash -n useradd.sh 
[root@node1 ~]# bash useradd.sh 
Add user:user1.
Add user:user2.
Add user:user3.
Add user:user4.
Add user:user5.
Add user:user6.
Add user:user7.
Add user:user8.
Add user:user9.
Add user:user10.
Add 10 Users.

```
&emsp;&emsp;练习4：打印九九乘法表：
```ruby
[root@node1 ~]# vim mul2.sh
#!/bin/bash
#
declare -i i=1
declare -i j=1

while [ $j -le 9 ];do
    while [ $i -le $j ];do
        echo -e -n "${i}X${j}=$[$i*$j]\t"
        let i++
    done
    echo 
    let i=1
    let j++
done

[root@node1 ~]# bash -n mul2.sh 
[root@node1 ~]# bash mul.sh
```

## Shell脚本编程-for语句
### 1、语法：
```ruby
for NAME in LIST; do
    循环体
done
```
#### 1.1 LIST生成方式：
- 整数列表
  {start..end}
  $(seq start [[step]end])
- glob
  /etc/rc.d/rc3.d.K*
- 命令

###2、for循环的特殊格式：
```ruby
for ((控制变量初始化；条件判断表达式；控制变量的修正表达式))；do
    循环体
done

// 控制变量初始化：仅在运行到循环代码段时执行一次
// 控制变量的修正表达式：每轮循环结束会先进行控制变量修正运算，而后再做条件判断
```
&emsp;&emsp;练习1：求100以内所有正整数之和

```ruby
[root@node1 ~]# vim sum.sh
#!/bin/bash
#
declare -i sum=0

for ((i=1;i<=100;i++));do
    let sum+=$i
done
echo "Sum:$sum"

[root@node1 ~]# bash -n sum.sh 
[root@node1 ~]# bash sum.sh 
Sum:5050

```
&emsp;&emsp;练习2；打印九九乘法表：
```ruby
[root@node1 ~]# vim muilti.sh
#!/bin/bash
#

for ((j=1;j<=9;j++));do
    for((i=1;i<=j;i++));do
        echo -e -n "${i}X${j}=$[$i*$j]\t"
    done
    echo 
done

[root@node1 ~]# bash -n multi.sh 
[root@node1 ~]# bash multi.sh 
1x1=1	
1x2=2	2x2=4	
1x3=3	2x3=6	3x3=9	
1x4=4	2x4=8	3x4=12	4x4=16	
1x5=5	2x5=10	3x5=15	4x5=20	5x5=25	
1x6=6	2x6=12	3x6=18	4x6=24	5x6=30	6x6=36	
1x7=7	2x7=14	3x7=21	4x7=28	5x7=35	6x7=42	7x7=49	
1x8=8	2x8=16	3x8=24	4x8=32	5x8=40	6x8=48	7x8=56	8x8=64	
1x9=9	2x9=18	3x9=27	4x9=36	5x9=45	6x9=54	7x9=63	8x9=72	9x9=81
```
```ruby
#!/bin/bash
#
for j in {1..9};do
    for i in $(seq 1 $j);do
        echo -e -n "${i}X${j}=$[$i*$j]\t "
    done
    echo 
done

```
&emsp;&emsp;练习3:ping一个网段里面的主机，并计算有多少个主机在线/不在线：
```ruby
[root@node1 ~]# vim ping.sh
#!/bin/bash
#
net='192.168.67'
uphosts=0
downhosts=0

for i in {1..20};do
    ping -c 1 -w 1${net}.${i} &> /dev/null
    if [ $? -eq 0 ];then
        echo "${net}.${i} is up."
        let uphosts++
    else
        echo "${net}.${i} is down."
        let downhosts++
    fi
done
echo "Up hosts:$uphosts."
echo "Down hosts:$downhosts."

[root@node1 ~]# bash -n ping.sh 
[root@node1 ~]# bash ping.sh 
192.168.67.1 is down.
192.168.67.2 is down.
192.168.67.3 is down.
......
192.168.67.20 is down.
Up hosts:0.
Down hosts:20.
```


## Shell脚本编程-until语句

### 1、语法
```ruby
until CONDITION; do
	循环体
done

进入条件：false
退出条件：true
```
### 2、创建死循环语法：
```ruby
until false;do
    循环体
done
```
### 3、练习
&emsp;&emsp;练习1：求100以内所有正整数之和
```ruby
[root@node1 ~]# vim summary.sh

#!/bin/bash

declare -i i=1
declare -i sum=0
until [ $i -gt 100 ]; do   // gt是大于
    let sum+=$i
    let i++
done
echo "Sum:$sum"

[root@node1 ~]# bash -n summary.sh
[root@node1 ~]# bash summary.sh 
Sum:5050
```
&emsp;&emsp;练习2：打印九九乘法表
```ruby
[root@node1 ~]# vim multi.sh
#!/bin/bash
#
declare -i i=1
declare -i j=1

until [ $j -gt 9 ]; do
    until [ $i -gt $j ]; do
        echo -n -e "${i}x${j}=$[$i*$j]\t"
        let i++
    done
    echo
    let i=1
    let j++
done

[root@node1 ~]# bash -n multi.sh 
[root@node1 ~]# bash multi.sh
1x1=1	
1x2=2	2x2=4	
1x3=3	2x3=6	3x3=9	
1x4=4	2x4=8	3x4=12	4x4=16	
1x5=5	2x5=10	3x5=15	4x5=20	5x5=25	
1x6=6	2x6=12	3x6=18	4x6=24	5x6=30	6x6=36	
1x7=7	2x7=14	3x7=21	4x7=28	5x7=35	6x7=42	7x7=49	
1x8=8	2x8=16	3x8=24	4x8=32	5x8=40	6x8=48	7x8=56	8x8=64	
1x9=9	2x9=18	3x9=27	4x9=36	5x9=45	6x9=54	7x9=63	8x9=72	9x9=81
```
## Shell脚本编程-continue语句
`用于循环体中，continue [N]提前结束第几层的本轮循环，而直接进入下一轮判断`
```ruby
while CONDITION1; do
    CMD1
    ...
    if CONDITION2; then
        continue
    fi
    CMDn
    ...
done
```

## Shell脚本编程-break语句
`用于循环体中,break [N]提前结束循环`
```ruby
while CONDITION1; do
    CMD1
    ...
    if CONDITION2; then
        break
    fi
    CMDn
    ...
done
```
### 1、练习
&emsp;&emsp;练习1：求100以内所有偶数之和，要求循环遍历100以内的所有正整数
```ruby
[root@node1 ~]# vim even.sh

#!/bin/bash
#
declare -i i=0
declare -i sum=0

until [ $i -gt 100 ];do
    let i++                   // i++之后先让i=1
    if [ $[$i%2] -eq 1 ];then   
        // i=1,1和2取模等于1，那么执行continue，不再向下执行，跳出本次循环，现在i=1，1 gt 100
        continue
    fi
    let sum+=$i
done
echo "Even sum:$sum"

[root@node1 ~]# bash -n even.sh 
[root@node1 ~]# bash even.sh 
Even sum:2550

```
&emsp;&emsp;练习2：每隔3秒钟到系统上获取已经登录的用户的信息，如果docker用户登录了，则记录于日志中，并退出；否则就每隔3秒钟获取一次
```ruby
// 提前先创建一个用户docker：
[root@node1 ~]# useradd docker
[root@node1 ~]# echo docker | passwd --stdin docker
Changing password for user docker.
passwd: all authentication tokens updated successfully.

```
```ruby
[root@node1 ~]# vim user.sh
#!/bin/bash
#
read -p "Enter a user name: " username

while true;do
    if who | grep "^$username" &> /dev/null;then
        break
    fi
    sleep 3
done

echo "$username logged on." >> /tmp/user.log

[root@node1 ~]# bash -n user.sh 
[root@node1 ~]# bash -x user.sh
[root@node1 ~]# bash user.sh  // 使用docker用户登录
Enter a user name: docker
[root@node1 ~]# cat /tmp/user.log 
docker logged on.
```
```ruby
另外一种写法1：
#!/bin/bash
#
read -p "Enter a user name: " username

while ! who | grep "^$username" &> /dev/null;do
    sleep 3
done
echo "$username logged on." >> /tmp/user.log
```
```ruby
另外一种写法2：
// while ! = until
#!/bin/bash
#
read -p "Enter a user name: " username

until who | grep "^$username" &> /dev/null;do
    sleep 3
done
echo "$username logged on." >> /tmp/user.log

```

## Shell脚本编程-case语句
### 1、语法:
```ruby
case 变量引用 in
PAT1）
   分支1
   ;;   // ;;也可以和分支1写到同一行
PAT2）
   分支2
   ;;
*)
   默认分支
   ;;   // 最后的;;可以不写
esac
```
### 2、练习
&emsp;&emsp;练习1：写一个脚本，完成如下任务
- 显示一个如下的菜单：
  cpu) show cpu information;
  mem) show memory information;
  disk) show disk information;
  quit) quit
- 提示用户选择选项：
- 显示用户选择的内容：
- 用户选择，并显示完成后不退出脚本，而是提示用户继续选择显示其他内容，直到使用quit退出；

```ruby
#!/bin/bash
#

cat << EOF
cpu) show cpu information
mem) show memory information
disk) show disk information
quit) quit
==================================
EOF

read -p "Enter a option:" option
while [ "$option" != "cpu" -a "$option" != "mem" -a "$option" != "disk" -a "$option" != "quit" ];do
    read -p "Wrong option,Enter again:" option
done

case "$option" in
cpu)
    lscpu
    ;;

mem)
    cat /proc/meminfo
    ;;

disk)
    fdisk -l
    ;;

*)
    echo "quit"
    exit 0
    ;;
esac

[root@node1 ~]# bash -n sysinfo2.sh 
[root@node1 ~]# bash sysinfo2.sh 
cpu) show cpu information
mem) show memory information
disk) show disk information
quit) quit
==================================
Enter a option:mem
MemTotal:        1483640 kB
MemFree:         1282940 kB
Buffers:           33948 kB
Cached:            43404 kB
SwapCached:            0 kB
Active:            59536 kB
```



