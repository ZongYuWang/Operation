## Shell脚本编程-数组

`变量：存储单个元素的内存空间`
`数组：存储多个元素的连续的内存空间`

### 1、 数组操作：
#### 1.1 索引：
&emsp;&emsp;编号从0开始；`索引也可支持使用自定义的格式，而不仅仅是数值格式`
&emsp;&emsp;引用数组中的元素：`${ARRAY_NAME[INDEX]}` 省略[INDEX]表示引用下标为0的元素

### 1.2 声明数组：
```ruby
declare -a ARRAY_NAME
declare -A ARRAY_NAME(关联数组)
```

### 1.3 数组元素的赋值：
- 一次只赋值一个元素：

```ruby 
ARRAY_NAME[INDEX]=VALUE
```
- 一次赋值全部元素：

```ruby
ARRAY_NAME[INDEX]=（"VAL1""VAL2""VAL3"...）
```
- 只赋值特定元素：

```ruby
ARRAY_NAME=（[0]="VAL1" [3]="VAL2"...）
```
- read -a ARRAY：

#### 1.4 数组的长度(数组中元素的个数)：
```ruby
${#ARRAY_NAME[*]}，${#ARRAY_NAME[@]
```

#### 1.5 引用数组中的元素：
```ruby
所有元素：${ARRAY[@]}，${ARRAY[*]}
${ARRAY[@]:offset:number}
    offset:要跳过的元素个数
    number：要取出的元素个数;偏移量之后的所有元素${ARRAY[@]:offset}
```

#### 1.6 向数组中追加元素:
```ruby
ARRAY[${#ARRAY[*]}]
```

#### 1.7 删除数组中的某元素：
```ruby
unset ARRAY[INDEX]
```

#### 1.8 关联数组：
```ruby
declare -A ARRAY_NAME
ARRAY_NAME=([index_name1]='val1' [index_name2]='val2' ...)
```

### 2、实验：
&emsp;&emsp;实验1：生成10个随机数保存于数组中，并找出其最大值和最小值
```ruby
[root@localhost ~]# echo $RANDOM  // $RANDOM是获取随机数
18001

[root@localhost ~]# vim A1.sh
#!/bin/bash
#
declare -a rand
declare -i max=0

for i in {0..9};do
    rand[$i]=$RANDOM
    echo ${rand[$i]}
    [ ${rand[$i]} -gt $max ] && max=${rand[$i]}
done
echo "Max:$max"

[root@localhost ~]# bash -n A1.sh 
[root@localhost ~]# bash A1.sh 
28569
14081
21937
30212
276
18455
416
23309
28569
21880
Max:30212
```
&emsp;&emsp;实验2：定义一个数组，数组中的元素是/var/log目录下所有以.log结尾的文件；要统计其下标为偶数的文件中的行数之和；
```ruby
[root@localhost ~]# vim array2.sh
#!/bin/bash
#
declare -a files
files=(/var/log/*.log)
declare -i lines=0

for i in $(seq 0 $[${#files[*]}-1]); do
    if [ $[$i%2] -eq 0 ];then
        let lines+=$(wc -l ${files[$i]} | cut -d' ' -f1)
    fi
done
echo "Lines:$lines."

[root@localhost ~]# bash array2.sh 
Lines:2283.
```
&emsp;&emsp;实验3：生成10个随机数，升序排序或降序排序
```ruby
略
```

## Shell脚本编程-字符串处理工具

### 1、字符串处理：

#### 1.1 字符串切片： 

&emsp;&emsp;`${var:offset:number}`
&emsp;&emsp;`${var: -lengh}` 取字符串的最右侧几个字符;`冒号后面必须有一个空白字符`

```ruby
[root@localhost ~]# name="wangzy"
[root@localhost ~]# echo ${name:1:3}
ang
```
```ruby
[root@localhost ~]# echo ${name: -3}
gzy
```
#### 1.2 基于模式取子串：

&emsp;&emsp;`${var#*word}`:其中word可以是指定的任意字符:
&emsp;&emsp;&emsp;&emsp;功能：自左而右，查找var变量所存储的字符串中，第一次出现的word，删除字符串开头至第一次出现word字符之间的所有字符
&emsp;&emsp;`${var##*word}`:其中word可以是指定的任意字符:
&emsp;&emsp;&emsp;&emsp;同上，只不过删除的是字符串开头至最后一次由word指定的字符串质检的所有内容

```ruby
[root@localhost ~]# file='/var/log/messages'
[root@localhost ~]# echo ${file#*/}
var/log/messages
[root@localhost ~]# file='var/log/messages'
[root@localhost ~]# echo ${file#*/}
log/messages

```
```ruby
[root@localhost ~]# file='/var/log/messages'
[root@localhost ~]# echo ${file##*/}   
messages

```
&emsp;&emsp;`${var%word*}`:其中word可以是指定的任意字符:
&emsp;&emsp;&emsp;&emsp;功能：自右而左，查找var变量所存储的字符串中，第一次出现的word，删除字符串最后一个字符向左至第一次出现word字符之间的所有字符

&emsp;&emsp;`${var%%word*}`:同上，只不过删除字符串最右侧的字符向左至最后一次出现word字符之间的所有字符
```ruby
[root@localhost ~]# file='/var/log/messages'
[root@localhost ~]# echo ${file%/*}
/var/log
[root@localhost ~]# echo ${file%%/*}

[root@localhost ~]# 
```
```ruby
[root@localhost ~]# url=http://www.baidu.com:80
[root@localhost ~]# echo ${url##*:}
80
[root@localhost ~]# echo ${url%%:*}
http
```

#### 1.3 查找替换：
&emsp;&emsp;`${var/pattern/substi}`:查找var所表示的字符串中，第一次被pattern所匹配到的字符串，以substi替换之
&emsp;&emsp;`${var//pattern/substi}`:查找var所表示的字符串中，所有能被pattern所匹配到的字符串，以substi替换之        

```ruby
[root@localhost ~]# user=$(head -1 /etc/passwd)
[root@localhost ~]# echo $user
root:x:0:0:root:/root:/bin/bash
[root@localhost ~]# echo ${user/root/ROOT}
ROOT:x:0:0:root:/root:/bin/bash
[root@localhost ~]# echo ${user//root/ROOT}
ROOT:x:0:0:ROOT:/ROOT:/bin/bash

```
&emsp;&emsp;`${var/#pattern/substi}`:查找var所表示的字符串中，行首被pattern所匹配到的字符串，以substi替换之 
&emsp;&emsp;`${var/%pattern/substi}`:查找var所表示的字符串中，行尾被pattern所匹配到的字符串，以substi替换之 

```ruby
[root@localhost ~]# user=$(head -1 /etc/passwd)
[root@localhost ~]# echo $user
root:x:0:0:root:/root:/bin/bash
[root@localhost ~]# echo ${user/#root/ROOT}
ROOT:x:0:0:root:/root:/bin/bash
[root@localhost ~]# user="admin:$user:root"  // 有意改造一下root不在行首
[root@localhost ~]# echo $user
admin:root:x:0:0:root:/root:/bin/bash:root
[root@localhost ~]# echo ${user/#root/ROOT}
admin:root:x:0:0:root:/root:/bin/bash:root
```
#### 1.4 查找并删除：
&emsp;&emsp;`${var/pattern}`:查找var所表示的字符串中，删除第一次被pattern所匹配到的字符串
&emsp;&emsp;`${var//pattern}`:查找var所表示的字符串中，删除所有被pattern所匹配到的字符串
&emsp;&emsp;`${var/#pattern}`:查找var所表示的字符串中，删除行首被pattern所匹配到的字符串
&emsp;&emsp;`${var/%pattern}`:查找var所表示的字符串中，删除行尾被pattern所匹配到的字符串
```ruby
[root@localhost ~]# user=$(head -1 /etc/passwd)
[root@localhost ~]# echo ${user/root}
:x:0:0:root:/root:/bin/bash
[root@localhost ~]# echo ${user//root}
:x:0:0::/:/bin/bash
[root@localhost ~]# echo ${user/#root}
:x:0:0:root:/root:/bin/bash
```

#### 1.5 字符大小写转换：
&emsp;&emsp; `${var^^}`：把var中的所有小写字母转换成大写
&emsp;&emsp; `${var,,}`：把var中的所有大写字母转换成小写

```ruby
[root@localhost ~]# user=$(head -1 /etc/passwd)
[root@localhost ~]# echo $user
root:x:0:0:root:/root:/bin/bash
[root@localhost ~]# echo ${user^^}
ROOT:X:0:0:ROOT:/ROOT:/BIN/BASH
[root@localhost ~]# echo ${user,,}
root:x:0:0:root:/root:/bin/bash
```
#### 1.6 变量赋值：
&emsp;&emsp;`${var:-value}`：如果var为空或未设置，那么返回value；否则，则返回var的值；
&emsp;&emsp;`${var:=value}`：如果var为空或未设置，那么返回value，并将value赋值给var；否则，则返回var的值；
&emsp;&emsp;`${var:+value}`：如果var不空，则返回value；
&emsp;&emsp;`${var:?error_info}`：如果var为空或未设置，那么返回error_info，否则，则返回var的值；

```ruby
[root@localhost ~]# weekday='monday'
[root@localhost ~]# echo ${weekday:-sunday}
monday
[root@localhost ~]# unset weekday
[root@localhost ~]# echo ${weekday:-sunday}
sunday
[root@localhost ~]# echo $weekday

[root@localhost ~]# 

```
```ruby
[root@localhost ~]# echo ${weekday:=sunday}
sunday
[root@localhost ~]# echo $weekday
sunday

```
#### 1.7 为脚本程序使用配置文件：
- 定义文本文件，每行定义"name=value"
- 在脚本中source此文件即可

```ruby
#!/bin/bash
#
[ -r /tmp/hostname ] && source /tmp/hostname
HOSTNAME=${HOSTNAME:-www.wangzy.com}
hostname $HOSTNAME

```

### 2、mktemp命令：
#### 2.1 语法：
```ruby
mktemp [OPTION] ...[TEMPLATE]
    TEMPLATE:filename.XXX
        XXX至少要出现三个
    OPTION：
        -d:创建临时目录
        --tmpdir=/PATH/TO/SOMEDIR：指定临时文件目录位置
```
#### 2.2 实验：
```ruby
[root@localhost ~]# mktemp /tmp/test.XXX
/tmp/test.QvN
[root@localhost ~]# mktemp /tmp/test.XXX
/tmp/test.Bkz
[root@localhost ~]# mktemp /tmp/test.XXX
/tmp/test.vju
[root@localhost ~]# mktemp /tmp/test.XXXXXX
/tmp/test.UmY6i1

```
```ruby
// 引用这个文件：

[root@localhost ~]# tmpfile=$(mktemp /tmp/test.XXX)
[root@localhost ~]# echo $tmpfile
/tmp/test.Nx4
[root@localhost ~]# echo $tmpfile
/tmp/test.Nx4

```
```ruby
// 创建一个目录：

[root@localhost ~]# mktemp -d /tmp/test.XXXX
/tmp/test.lLmF
[root@localhost ~]# ll /tmp
drwx------. 2 root root 4096 May 10 14:30 test.lLmF

```
### 3、install命令：
#### 3.1 语法：
```ruby
install [OPTION]... [-T] SOURCE DEST
install [OPTION]... SOURCE... DIRECTORY
install [OPTION]... -t DIRECTORY SOURCE...
install [OPTION]... -d DIRECTORY...
      OPTION:
          -m MODE
          -o OWNER
          -g GROUP
          
```
#### 3.2 实验：
```ruby
[root@localhost ~]# install -m 700 -d testdir
[root@localhost ~]# ll
drwx------. 2 root root     4096 May 10 14:41 testdir
```