## python变量和字符编码：

### Python3和Python2的比较：   
Python3比Python2的一个很大的改进就是支持了 unicode，也就是可以直接支持中文了，默认python2是不支持中文的；

#### 第一个python程序:
` 如果要把这段代码的文件变成一个可执行文件，必须前提声明解释下面代码的“解释器”；python有两种说明解释器的方法`
```py
#！/usr/bin/env python # 相当于去整个系统找环境变量，python的环境变量
#! /usr/bin/python  # 相当于直接写死了python的环境变量的路径

print("Hello World!")
```

### 变量：
```ruby
name = "wangzy"
print("Hello",name)
print("------------------------------------------------------")

name = "wangzy"
name2 = name
print("My name is" ,name,name2)
print("------------------------------------------------------")

name = "wangzy"
name2 = name
print("My name is" ,name,name2)

name = "babyshen"
print("My name is",name,name2)
print("------------------------------------------------------")
```
*`输出：`*
```ruby
Hello wangzy
------------------------------------------------------
My name is wangzy wangzy
------------------------------------------------------
My name is wangzy wangzy
My name is babyshen wangzy
------------------------------------------------------
```
#### 变量定义的规则：
- 变量名只能字母、数字或下划线的任意组合
- 变量名的第一个字符不能是数字
- 以下关键字不能声明为变量名
['and', 'as', 'assert', 'break', 'class', 'continue', 'def', 'del', 'elif', 'else', 'except', 'exec', 'finally', 'for', 'from', 'global', 'if', 'import', 'in', 'is', 'lambda', 'not', 'or', 'pass', 'print', 'raise', 'return', 'try', 'while', 'with', 'yield']  


`【说明】如果想声明一个常量，需要将变量名设置为全大写(如：PIE = 常量值)，但是其实也是可以修改的，只是编写者全大写之后，警示这是一个常量`

### 字符编码：
- ###### ASCII编码：
python解释器在加载 .py 文件中的代码时，会对内容进行编码（默认ASCII）
ASCII（American Standard Code for Information Interchange，美国标准信息交换代码）是基于拉丁字母的一套电脑编码系统，主要用于显示现代英语和其他西欧语言，其最多只能用 8 位来表示（一个字节），即：2**8 = 256-1，所以，ASCII码最多只能表示 255 个符号。     
`【说明】计算机是西方的产物，所以前128位用于表示英语和其他西欧语言字符，后127位留作其他国家使用`

- ###### GB2312编码：
为了处理汉字，程序员设计了用于简体中文的GB2312和用于繁体中文的big5。
GB2312(1980年)一共收录了7445个字符，包括6763个汉字和682个其它符号，占用的码位是72*94=6768。

- ###### GBK1.0编码：
GB2312 支持的汉字太少。1995年的汉字扩展规范GBK1.0收录了21886个符号，它分为汉字区和图形符号区。汉字区包括21003个字符。

- ###### GB18030编码：
2000年的 GB18030是取代GBK1.0的正式国家标准。该标准收录了27484个汉字，同时还收录了藏文、蒙文、维吾尔文等主要的少数民族文字。现在的PC平台必须支持GB18030，对嵌入式产品暂不作要求。所以手机、MP3一般只支持GB2312。

`【说明】从ASCII、GB2312、GBK 到GB18030，这些编码方法是向下兼容的，即同一个字符在这些方案中总是有相同的编码，后面的标准支持更多的字符。在这些编码中，英文和中文可以统一地处理。区分中文编码的方法是高字节的最高位不为0。按照程序员的称呼，GB2312、GBK到GB18030都属于双字节字符集 (DBCS)。`

有的中文Windows的缺省内码还是GBK，可以通过GB18030升级包升级到GB18030。不过GB18030相对GBK增加的字符，普通人是很难用到的，通常我们还是用GBK指代中文Windows内码。

显然ASCII码无法将世界上的各种文字和符号全部表示，所以，就需要新出一种可以代表所有字符和符号的编码，即：Unicode

- ######  Unicode编码：
Unicode（统一码、万国码、单一码）是一种在计算机上使用的字符编码。Unicode 是为了解决传统的字符编码方案的局限而产生的，它为每种语言中的每个字符设定了统一并且唯一的二进制编码，规定虽有的字符和符号最少由 16 位来表示（2个字节），即：2 **16 = 65536   
`【注意】此处说的的是最少2个字节，可能更多，这样西欧国家和ASCII编码的不满意，每个字符占据的空间变大，本来他们一个字符占用1个字节，现在出现Unicode之后，却占用了2个字节，所以出现了UTF-8`

- ######  UTF-8编码：
UTF-8，是对Unicode编码的压缩和优化，他不再使用最少使用2个字节，而是将所有的字符和符号进行分类：ASCII码中的内容用1个字节保存、欧洲的字符用2个字节保存，东亚的字符用3个字节保存，也就是存英文字符或西欧语言字符按照ASCII存储，存其他字符(如汉字)就按照UTF-8存放-占 3B=24bite
![](https://github.com/ZongYuWang/image/blob/master/python-ascii1.png)

### 代码注释：
` 如果是多行注释，需要使用'''引用，多行赋值给一个变量时，也可以使用使用'''` 
```ruby
msg = '''
name2 = name
print("My name is",name,name2)
name = "wangzy"
'''
print(msg)
```
` 如果是单行，可以直接使用双引号/单引号，引用之后可以赋值给一个变量` 
```ruby
msg = "name2 = name"
```
python中的单引号和双引号没什么区别(但是在后面讲解的json中有区别，json有严格的格式)，只是在如 msg = " I'm wangzy "中有区别，要么就是双引号套单引号，要么就是单引号套双引号

### 接收用户输入：
```ruby
username = input("username: ")
password = input("password: ")
print(username,password)
```

### 字符串拼接：    
- 方式1：    ` 不要用+拼接方式，会浪费内存块`

```ruby
name = input("name: ")
age = input("age: ")
job = input("job: ")
salary = input("salary: ")

info = '''  # 注意引号之间的对应关系
-------------------- Info of ''' +name+ ''' --------------------
name:'''+name+'''
age:'''+age+'''
job:'''+job+'''
salary:'''+salary   # 最后一个+salary后面没有+号

print(info)
```
- 方式二：   

```ruby
name = input("name: ")
age = input("age: ")
job = input("job: ")
salary = input("salary: ")

info = '''
-------------------- Info of %s --------------------  # %s = %string
name: %s  
age: %s
job: %s
salary:%s
''' %(name,name,age,job,salary)  # 一共5个%s，所以最后要给出5个变量，数量必须匹配

print(info)

```
将上面的age:%s修改为age:%d   
```ruby
name = input("name: ")
age = input("age: ")
job = input("job: ")
salary = input("salary: ")

info = '''
-------------------- Info of %s --------------------
name: %s
age: %d
job: %s
salary:%s
''' %(name,name,age,job,salary)

print(info)
```
*`输出:`*
```ruby
name: wangzy
age: 20
job: It
salary: 2000
Traceback (most recent call last):
  File "E:/PycharmProjects/untitled/study/day01/123456789.py", line 15, in <module>
    ''' %(name,name,age,job,salary)
TypeError: %d format: a number is required, not str
```
键盘输入默认的是字符串，虽然眼睛看输入的是20(整型)数据，但是实际上输入的是字符串，下面的实验验证：
```ruby
age = input("age: ")
print(type(age))
age = int(input("age: "))
print(type(age))

输出：
age: 20
<class 'str'>
age: 20
<class 'int'>
```
- 方式3：

```ruby
name = input("name: ")
age = input("age: ")
job = input("job: ")
salary = input("salary: ")

info = '''
-------------------- Info of {_name} --------------------
name: {_name}
age: {_age}
job: {_job}
salary: {_salary}
''' .format(_name=name,
            _age=age,
            _job=job,
            _salary=salary)

print(info)
```
_name、_age、_job、_salary是自己定义的其他变量，目的是和上面的name、age、job、salary区分开,输出部分有两个_name，format中不用写两次_name=name，定义一次将name值赋值给_name即可

- 方式4：

```ruby
name = input("name: ")
age = input("age: ")
job = input("job: ")
salary = input("salary: ")

info = '''
-------------------- Info of {0} --------------------
name: {0}
age: {1}
job: {2}
salary: {3}
''' .format(name,age,job,salary)

print(info)
```
### 隐藏用户输入密码：

```ruby
import getpass
username = input("username: ")
password = getpass.getpass("password: ")

print(username,password)
```

*`执行/输出：`*
```ruby
E:\PycharmProjects\untitled\study\day01>py 123456789.py
username: wangzy
password:
wangzy 123456

```

### if语句：    
- 实验一：根据输入的用户名，如果输入正确，就提示“Welcome user username login...”，如果输入错误就提示“Invalid username or password！”

```ruby
_username = "wangzy"
_password = "wangzy"

username = input("username: ")
password = input("password: ")

if _username == username and _password == password:
    print("Welcome to {name} login..." .format(name=username))
else:
    print("Invalid username or password!")

【说明】因为python没有结束符，shell中的if 最后有一个fi结束，因为python中没有结束符，所以就要强制缩进
【说明】如果不是在“”双引号之内引用变量的情况，可以直接这么写： print ("Welcome to login..",username)


if _username == username and _password == password:
    print("Welcome to {name} login..." .format(name=username))
 
else:
    print("Invalid username or password!")
【说明】if和else各是一块子程序代码
```
- 实验二：猜测一个人的年龄，如果给定的年龄大了，就提示往小猜，给的年龄小了就往大了猜，如果猜对了，就提示猜测正确    

```ruby
age_of_wangzy = 30

guess_age = int(input("Plz guess wangzy's age: "))

if age_of_wangzy == guess_age:
    print(" yes,you got it! ")
         
elif age_of_wangzy > guess_age:
    print(" think bigger! ")
else:
    print(" think smaller! ")
    
```
### while语句：
`python有 while-else的语法（while可以结合else使用）`

```ruby
# 对上面的实验改进：输入完3次之后，提示是否还要继续输入，只要不输入n/N就可以继续尝试猜测输入

age_of_wangzy = 30
count = 0

while count < 3:
    guess_age = int(input("Plz guess wangzy's age: "))
    if age_of_wangzy == guess_age:
        print(" yes,you got it! ")
        break
    elif age_of_wangzy > guess_age:
        print(" think bigger! ")
    else:
        print(" think smaller! ")
    count += 1

if count == 3:
    countine_confirm = input("do you want to keep guessing? ")
    if countine_confirm != "n" or "N":
        count == 0
```
### for语句：
` for循环也有for...else语句 `

```ruby
age_of_wangzy = 30

for count in range(3):
    guess_age = int(input("Plz guess wangzy's age: "))
    if age_of_wangzy == guess_age:
        print(" yes,you got it! ")
        break
    elif age_of_wangzy > guess_age:
        print(" think bigger! ")
    else:
        print(" think smaller! ")
    
else:
    print("you have tried too many times,fuck off...")
    
【说明】上面的循环都正常走完了，才执行最后的else，如果if中的break执行了，那么最后一步的esle就不会被执行了
```

### break和continue区别：
- break

```ruby
for i in range(0,10):
   print(i)
   if i < 3:
       print("loop",i)
   else:
       break
   print("hehe...")

# 输出：
0
loop 0
hehe...
1
loop 1
hehe...
2
loop 2
hehe...
3
【说明】遇到break，直接跳出整个循环体
```

- continue    

```ruby
for i in range(0,10):
   print(i)
   if i < 3:
       print("loop",i)
   else:
       continue
   print("hehe...")
   
# 输出
0
loop 0
hehe...
1
loop 1
hehe...
2
loop 2
hehe...
3
4
5
6
7
8
9

【说明】当使用continue的时候，假如当i=2时，会执行print("loop",i)，也会执行最后的print("hehe..")      
当i=3时，会执行else，遇到continue，不会向下继续循环体内的语句，本例中就不会执行print("hehe")了，而是去执行上面的for循环 赋值 i=4
```
`print写在循环体之外：`
```ruby
for i in range(0,10):
   print(i)
   if i < 3:
       print("loop",i)
   else:
       continue
print("hehe...")

# 输出
0
loop 0
1
loop 1
2
loop 2
3
4
5
6
7
8
9
hehe...
```

```ruby
for i in range(3):
   print("----------",i)
   for j in range(5):
       print(j)
       
# 输出：
---------- 0
0
1
2
3
4
---------- 1
0
1
2
3
4
---------- 2
0
1
2
3
4
```

```ruby
for i in range(10):
   print("----------",i)
   for j in range(10):
       print(j)
       if j > 5:
           break
           
# 输出：
---------- 0
0
1
2
3
4
5
6
---------- 1
0
1
2
3
4
5
6
---------- 2
0
1
2
3
4
5
6
---------- 3
0
1
2
3
4
5
6
---------- 4
0
1
2
3
4
5
6
---------- 5
0
1
2
3
4
5
6
---------- 6
0
1
2
3
4
5
6
---------- 7
0
1
2
3
4
5
6
---------- 8
0
1
2
3
4
5
6
---------- 9
0
1
2
3
4
5
6

【说明】break结束的是j的循环，而不是结束的i的for循环
```