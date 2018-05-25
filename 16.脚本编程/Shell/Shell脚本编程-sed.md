## Shell脚本编程-sed
`sed:Stream EDitor(行编辑器)，不对源文件进行处理，而且一行一行的先读取到模式空间中再操作`

### 1、sed用法：
```ruby
sed [option] ... 'script' inputfile....
``` 
#### 1.1 常用选项：
```ruby
-n :不输出模式中的内容至屏幕；
-e :多点编辑；
-f /PATH/TO/SCRIPT_FILE :从指定文件中读取编辑脚本；
-r ：支持使用扩展正则表达式；
-i :原处编辑(也就是修改原文件)

```
#### 1.2 地址定界：
- 不给地址：对全文进行处理；
- 单地址：

```ruby
# :指定的行；
/pattern/：被此处模式所能够匹配到的每一行；
```
- 地址范围：

```ruby
#,# ：第几行到第几行；
#,+# ：从第几行再往下进行几行；
/pat1/,/pat2/ :第一次匹配到的行到第几次匹配到的行；
#，/pat1/ :第几行到第一次被匹配到的行；
```

- ~ :步进

```ruby
[root@localhost ~]# cat test.txt 
1
2
3
4
5
6
[root@localhost ~]# sed -n '1~2p' test.txt 
1
3
5

[root@localhost ~]# sed -n '2~2p' test.txt 
2
4
6

```

#### 1.3 编辑命令：
```ruby
d :删除
[root@localhost ~]# sed '/^UUID/d' /etc/fstab  // 将UUID开头的都删除
[root@localhost ~]# sed '/^#/d' /etc/fstab  // #好开头的行都删除
[root@localhost ~]# sed '/^$/d' /etc/fstab  // 空白行都删除
[root@localhost ~]# sed '1,4d' /etc/fstab  // 删除第一行到第四行
```
```ruby
p ：打印(显示模式空间中的内容)
[root@localhost ~]# sed '/^UUID/p' /etc/fstab 
// 效果就是会显示两行UUID开头的
// 默认就会显示一次/etc/fstab的内容，又指定了UUID开头的条件，所以显示两次

[root@localhost ~]# sed -n '/^UUID/p' /etc/fstab 
// 不输出模式中的内容至屏幕
```
```ruby
a \'test' :在行后面追加文本，支持使用\n实现多行追加
[root@localhost ~]# sed '/^UUID/a \#hello sed.' /etc/fstab
[root@localhost ~]# sed '/^UUID/a \#hello sed. \nwelcome' /etc/fstab
```
```ruby
i \'test' :在行前面追加文本，支持使用\n实现多行插入
[root@localhost ~]# sed '/^UUID/i \#hello sed. \nwelcome' /etc/fstab
```
```ruby
c \'test' :替换行为单行或多行文本
[root@localhost ~]# sed '/^UUID/c \#hello sed. \nwelcome' /etc/fstab 
```
```ruby
w /path/to/somefile :保存模式空间匹配到的行至指定文件中
[root@localhost ~]# sed '/^UUID/w /tmp/fstab.txt' /etc/fstab
[root@localhost ~]# cat /tmp/fstab.txt 
UUID=23dbdffc-4590-47a5-b5d6-f06d83f2a297 /boot                   ext4    defaults        1 2

```
```ruby
r /path/to/somefile :读取指定文件的文本流至模式空间中匹配到的行的行后

[root@localhost ~]# cat /etc/issue
CentOS release 6.5 (Final)
Kernel \r on an \m

[root@localhost ~]# sed '6r /etc/issue' /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Sun Apr 29 13:40:31 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
CentOS release 6.5 (Final)
Kernel \r on an \m

# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for mor
```

```ruby
= :为模式空间中的行打印行号
[root@localhost ~]# sed '/^UUID/=' /etc/fstab
/dev/mapper/vg_node1-lv_root /                       ext4    defaults        1 1
10

```
```ruby
[root@localhost ~]# sed '/^UUID/!d' /etc/fstab 
UUID=23dbdffc-4590-47a5-b5d6-f06d83f2a297 /boot                   ext4    defaults        1 2
```
```ruby
s/// :支持使用其分隔符，s@@@,s###
替换标记：
	g ：行内全局替换
	p :显示替换成功的行
	w /path/to/somefile :将替换成功的结果保存至指定文件中

[root@localhost ~]# sed 's/^UUID/uuid/' /etc/fstab //没有加g，每行只修改第一次出现的

[root@localhost ~]# sed 's/r..t/&er/' /etc/passwd
rooter:x:0:0:root:/root:/bin/bash

[root@localhost ~]# sed -n 's/r..t/&er/p' /etc/passwd
rooter:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/rooter:/sbin/nologin
ftp:x:14:50:FTP User:/var/fterp:/sbin/nologin

```
### 2、练习
&emsp;&emsp;练习1：删除/boot/grub/grub.conf文件中所有以空白开头的行行首的空白字符
```ruby
[root@localhost ~]# sed 's/^[[:space:]]\+//' /boot/grub/grub.conf 
// [[:space:]]\+表示空白至少出现1次
```
&emsp;&emsp;练习2：删除/etc/fstab文件中所有以#开头，后面至少跟一个空白字符的行的行首的#和空白字符
```ruby
[root@localhost ~]# sed 's/^#[[:space:]]\+//' /etc/fstab
// 有的#后面没有跟空白字符
```

&emsp;&emsp;练习3：echo一个绝对路径给sed命令，取出其目录名
```ruby
[root@localhost ~]# echo "/etc/sysconfig" | sed 's@[^/]\+$@@'
/etc/
[root@localhost ~]# echo "/etc/sysconfig/" | sed 's@[^/]\+$@@'
/etc/sysconfig/
[root@localhost ~]# echo "/etc/sysconfig/" | sed 's@[^/]\+/\?$@@'
/etc/

// "/\? 表示不确定末尾的\，/表示转义"

```

### 3、高级编辑命令：
```ruby
h :把模式空间中的内容覆盖至保存空间中
H :把模式空间中的内容追加至保存空间中
```
```ruby
g :从保持空间中取出数据至覆盖至模式空间
G :从保持空间取出内容追加至模式空间

[root@localhost ~]# cat test.txt 
1
2
3
4
5
6
[root@localhost ~]# sed '1!G;h;$!d' test.txt 
6
5
4
3
2
1

// 1! 表示不是第一行，1!G不是第一行做G操作
// $! 表示不是最后一行，$!d不是最后一行就删除

[root@localhost ~]# sed 'G' test.txt 
1

2

3

4

5

6

// 因为保持空间是空白，所以每一行后面都加一个空白行
```
```ruby
x :把模式空间中的内容与保持空间中的内容进行互换
```
```ruby
n :读取匹配到的行的下一行至模式空间
N :追加匹配到的行的下一行至模式空间

[root@localhost ~]# cat test.txt 
1
2
3
4
5
6
[root@localhost ~]# sed -n 'n;p' test.txt 
2
4
6

[root@localhost ~]# sed 'n;d' test.txt 
1
3
5


// 多个命令之间要使用分号隔开
```

```ruby
d :删除模式空间中的行
D :删除多行模式空间中的所有行

[root@localhost ~]# sed '$!d' test.txt   //取文件的最后一行(不是最后一行就删除)
6

[root@localhost ~]# sed '$!N;$!D' test.txt  //取出文件后两行
5
6

```