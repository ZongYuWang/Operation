## Shell脚本编程-function函数

### 1、语法：
#### 1.1 语法1：
```ruby
function f_name{
    ... 函数体...
}
```
#### 1.2 语法2：
```ruby
f_name() {
    ...函数体...
}

// 函数只有被调用才会执行
```
### 2、实验
```ruby
[root@node1 ~]# vim f1.sh
#!/bin/bash
#
username="myuser"
function adduser {
    if id $username &> /dev/null;then
        echo "$username exits."
    else
        useradd $username
        [ $? -eq 0 ] && echo "Add $username finished."
    fi
}
adduser

[root@node1 ~]# bash -n f1.sh 
[root@node1 ~]# bash f1.sh 
Add myuser finished.

```
```ruby
#!/bin/bash
#
username="myuser"
function adduser {
    if id $username &> /dev/null;then
        echo "$username exits."
        return 1
    else
        useradd $username
        [ $? -eq 0 ] && echo "Add $username finished." && return 0
    fi
    echo "hello there."
}
adduser
echo $?

[root@node1 ~]# bash f1.sh 
Add myuser finished.
0         // 用户不存在的时候，会return 0
[root@node1 ~]# bash f1.sh 
myuser exits.
1     // 用户存在的时候，会返回return1

// 最后的echo "hello there."就没有执行，所以return一定要慎用，确定代码执行完毕了，才可以用return
```
```ruby
#!/bin/bash
#
username="myuser"
function adduser {
    if id $username &> /dev/null;then
        echo "$username exits."
        return 1
    else
        useradd $username
        [ $? -eq 0 ] && echo "Add $username finished." && return 0
    fi
    echo "hello there."
}

for i in {1..10};do
    username=myuser$i
    adduser
done

```

&emsp;&emsp;综合练习：写一个脚本，完成如下要求：
- 脚本可以接受参数：start、stop、restart、status；
- 如果参数非此四者之一，提示使用格式后报错退出；
- 如果是start：则创建/var/lock/subsys/SCRIPT_NAME，并显示"启动成功"；
   考虑：如果事先已经启动过一次，该如何处理？
- 如果是stop：则删除/var/lock/subsys/SCRIPT_NAME，并显示"停止完成"；
   考虑：如果事先已经停止过了，该如何处理？
- 如果是restart，则先stop，再start；
   考虑：如果本来没有start，如何处理？
- 如果是status，则
   如果/var/lock/subsys/SCRIPT_NAME文件存在，则显示"SCRIPT_NAME is running..."
   如果/var/lock/subsys/SCRIPT_NAME文不件存在，则显示"SCRIPT_NAME is stopped..."
- 其中，SCRIPT_NAME为当前脚本名；

```ruby
#!/bin/bash
#
# chkconfig: 2345 67 34
#
srvName=$(basename $0)

lockFile=/var/lock/subsys/$srvName

start() {
    if [ -f $lockFile ];then
        echo "$srvName is already running."
        return 1
    else
        touch $lockFile
        [ $? -eq 0 ] && echo "Starting $srvName OK."
        return 0
     fi
}

stop() {
    if [ -f $lockFile ];then
        rm -f $lockFile &> /dev/null
        [ $? -eq 0 ] && echo "Stop $srvName OK" && return 0
    else
        echo "$srvName is not started."
        return 1
    fi
}

status() {
    if [ -f $lockFile ]; then
        echo "$srvName is running."
    else
        echo "$srvName is stopped."
    fi
    return 0
usage() {
     echo "Usage: $srvName {start|stop|restart|status}"
     return 0
}

case $1 in
start)
        start
        ;;
stop)
        stop ;;
restart)
        stop
        start ;;
status)
        status ;;
*)
        usage
        exit 1 ;;
esac

}


```
```ruby
[root@node1 ~]# bash -n testsrv.sh
[root@node1 ~]# chmod +x testsrv.sh       
[root@node1 ~]# cp testsrv.sh /etc/rc.d/init.d/testsrv
[root@node1 ~]# ll /etc/rc.d/init.d/testsrv 
-rwxr-xr-x. 1 root root 793 May  7 16:52 /etc/rc.d/init.d/testsrv
[root@node1 ~]# chkconfig --add testsrv
[root@node1 ~]# chkconfig --list testsrv
testsrv        	0:off	1:off	2:on	3:on	4:on	5:on	6:off
 
[root@node1 ~]# service testsrv status
testsrv is stopped.
[root@node1 ~]# service testsrv start
Starting testsrv OK.
[root@node1 ~]# ls /var/lock/subsys/
testsrv
[root@node1 ~]# service testsrv stop
Stop testsrv OK
[root@node1 ~]# service testsrv restart
testsrv is not started.
Starting testsrv OK.

```
### 3、函数返回值
#### 3.1 函数的执行结果返回值：
- 使用echo或print命令进行输出
- 函数体中调用命令的执行结果

#### 3.2 函数的退出状态码：
- 默认取决于函数体中执行的最后一条命令的退出状态码
- 自定义退出状态码 

### 4、变量作用域
- 本地变量：当前shell进行，为了执行脚本会启动专用的shell进行，因此，本地变量的作用范围是当前shell脚本程序；
- 局部变量：函数的生命周期，函数结束时变量被自动销毁；
`如果函数中有局部变量，其名称同本地变量`

```ruby
[root@node1 ~]# vim f2.sh
#!/bin/bash
#
declare -i i=6

f1() {
    let i++
    echo "Function:$i"
}
f1
echo "main:$i"

[root@node1 ~]# bash f2.sh
Function:7
main:7

```
#### 4.1 在函数中定义局部变量的方法：
```ruby
local NAME=VALUE
```
```ruby
[root@node1 ~]# vim f2.sh
#!/bin/bash
#
declare -i i=6

f1() {
    local i=10
    let i++
    echo "Function:$i"
}
f1
echo "main:$i"

[root@node1 ~]# bash f2.sh
Function:11
main:6

```
### 5、函数递归：
&emsp;&emsp;函数直接或间接调用自身
```ruby
[root@node1 ~]# vim f3.sh 
#!/bin/bash
#
recursion() {
    if [ $1 -eq 0 -o $1 -eq 1 ]; then
        echo 1
    else
        echo $[$1*$(recursion $[$1-1])]
    fi
}
recursion 5

```