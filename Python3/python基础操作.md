## python基础操作：

### python的基本数据类型以及简单操作：

- 整型：
```ruby
python2:
>>> type(2**16)
<type 'int'>
>>> type(2**32)
<type 'int'>
>>> type(2**100)
<type 'long'>

python3:
>>> type(2**16)
<class 'int'>
>>> type(2**32)
<class 'int'>
>>> type(2**100)  # 在python3.x中已经没有了long int类型
<class 'int'>
```
- 十六进制：
```ruby
>>> 52.3E4
523000.0
>>> 52.3 * 10**4
523000.0
```
- 布尔型(1为真，0为假)
```ruby
a=0时，不会输出a:

a = 0
if a:
    print("a")
    
a=1时，会输出a:
a = 1
if a:
    print("a")
    
```
- 三元运算：
```ruby
a,b,c = 1,3,5

d = a if a > b else c
print(d)  # 输出5

【说明】意思就是a>b如果成立，那么d=a，如果a>b不成立，那么d=c
```

### 切片：
```ruby
names = "wangzy,baby,shen"
names = ["wangzy","baby","shen","hehe"]
print(names)
print(names[0],names[2])
print(names[1:3])
print(names[3])
print(names[-2])
print(names[-1])

# 输出：
['wangzy', 'baby', 'shen', 'hehe']
wangzy shen
['baby', 'shen']
hehe
shen
hehe

```
` 起始位置内容会包括，结束位置内容不包括 （顾头不顾尾）`
- 要求：输出最后面两个值：   

`【注意】如果按照下面的方式写，会输出一个空值`
```ruby
print(names[-1:-3])
```
因为切片是从左至右的，最后一个值的标号是-1，后数第二个是-2，但是取值的顺序是自左至右取值；
正确的代码形式：
```ruby
print(names[-3:-1])   # 输出：['baby', 'shen']
```
`【说明】同样是顾头不顾尾，-1位置的值不会被取到`
因为取值是顾头不顾尾，-1的那项不会被取到，所以如果真想取最后那个值，可以这样：
```ruby
names = "wangzy,baby,shen"
names = ["wangzy","baby","shen","hehe"]

print(names[-3:])
print(names[:3])

#【说明】0是可以忽略不写的
```

### append(添加):
`追加到文件内容最后`
```ruby
names = ["wangzy","baby","shen","hehe"]

names.append("trade")
print(names)

#输出：
['wangzy', 'baby', 'shen', 'hehe', 'trade']
```
### insert:
` 按照列表位置插入`
```ruby
names = ["wangzy","baby","shen","hehe"]

names.insert(1,"trade")  # 1就是baby的位置
print(names)
```
### 修改：
```ruby
names = ["wangzy","baby","shen","hehe"]

names[2] = "world"
print(names)

#输出:
['wangzy', 'baby', 'world', 'hehe']
```
### 删除：
- 方式1：remove-直接指定列表数值
```ruby
names = ["wangzy","baby","shen","hehe"]

names.remove("hehe")
print(names)
```
- 方式2：del-指定列表的编号
```ruby
names = ["wangzy","baby","shen","hehe"]

del names[3]
print(names)
```
- 方式3：pop-如果不写列表的角标编号，默认删除最后一项
```ruby
names = ["wangzy","baby","shen","hehe"]

names.pop()
print(names)
names.pop()
print(names)

pop源码解释：
def pop(self, index=None): # real signature unknown; restored from __doc__
        """
        L.pop([index]) -> item -- remove and return item at index (default last).
        Raises IndexError if list is empty or index is out of range.
        """
        pass

```
```ruby
names = ["wangzy","baby","shen","hehe"]

names.pop(2)
print(names)

# 输出：
['wangzy', 'baby', 'hehe']
```
### index
` 查找列表中某项数据的位置(角标编号)`
```ruby
names = ["wangzy","baby","shen","hehe"]

print(names.index("baby"))

# 输出：
1
```
根据列表中角标值打印列表数据：
```ruby
names = ["wangzy","baby","shen","hehe"]

print(names[names.index("baby")])

# 输出：
baby
```

### count：
` 统计同名的数据的个数`
```ruby
names = ["wangzy","baby","shen","wangzy","wangzy"]

print(names.count("wangzy"))

# 输出：
3
```

### 清空列表：
```ruby
names = ["wangzy","baby","shen","wangzy","wangzy"]

names.clear()
print(names)

# 输出：
[]
```

### 顺序反转：
```ruby
names = ["wangzy","baby","shen","hehe"]

names.reverse()
print(names)

# 输出：
['hehe', 'shen', 'baby', 'wangzy']
```

### 排序：
```ruby
names = ["wangzy","baby","shen","hehe"]

names.sort()
print(names)

# 输出：
['baby', 'hehe', 'shen', 'wangzy']

【说明】排序是按照ASCII顺序排序的
```

### 合并：
```ruby
names = ["wangzy","baby","shen","hehe"]
names_new = ["1","2","3","4"]

names.extend(names_new)
print(names,names_new)

# 输出：
['wangzy', 'baby', 'shen', 'hehe', '1', '2', '3', '4'] ['1', '2', '3', '4']

【说明】合并之后列表names_new还是会存在，需要手动删除：
del names_new
```

### copy：浅复制
- 浅复制-方式1：
```ruby
将names复制一份给names_new

names = ["wangzy","baby","shen","hehe"]
names_new = names.copy()

print(names)
print(names_new)

修改names的某个值：
names = ["wangzy","baby","shen","hehe"]
names_new = names.copy()

print(names)
print(names_new)

names[1] = "BABY"
print(names)
print(names_new)

# 输出：
['wangzy', 'baby', 'shen', 'hehe']
['wangzy', 'baby', 'shen', 'hehe']
['wangzy', 'BABY', 'shen', 'hehe']
['wangzy', 'baby', 'shen', 'hehe']
```

列表中包含列表：
```ruby
names = ["wangzy","baby",["hello","world"],"shen","hehe"]
names_new = names.copy()

print(names)
print(names_new)

names[2][0] = "HELLO"
print(names)
print(names_new)

# 输出：
['wangzy', 'baby', ['hello', 'world'], 'shen', 'hehe']
['wangzy', 'baby', ['hello', 'world'], 'shen', 'hehe']
['wangzy', 'baby', ['HELLO', 'world'], 'shen', 'hehe']
['wangzy', 'baby', ['HELLO', 'world'], 'shen', 'hehe']

# 修改子列表中的一项值，复制的文件的内容也被修改了
# 大列表中的2号角标中的子列表的1号角标数据修改为“HELLO”
```
- 浅复制-方式2：
```ruby
import copy
names = ["wangzy","baby",["hello","world"],"shen","hehe"]
names_new = copy.copy(names)

print(names)
print(names_new)
```
- 浅复制-方式3：
```ruby
names = ["wangzy","baby",["hello","world"],"shen","hehe"]
names_new = names[:]

print(names)
print(names_new)
```
- 浅复制-方式4：
```ruby
names = ["wangzy","baby",["hello","world"],"shen","hehe"]
names_new = list(names)

print(names)
print(names_new)
```

### copy：深度复制
```ruby
import copy
names = ["wangzy","baby",["hello","world"],"shen","hehe"]
names_new = copy.deepcopy(names)

print(names)
print(names_new)

```

### index:
`在列表中，index是打印下角标的号码，在字符串中是输出字符的位置 `
```ruby
name = "my name is wangzy"
print(name.index("i"))  # 输出：8

names = ["wangzy","baby","shen","hehe"]
print(names.index("shen"))  # 输出：2
```

### format、format_map使用：

```ruby
name = "my \tname is {name} and i am {year} old"

print(name.format(name="wangzy",year=23))
print(name.format_map( {'name':'wangzy','year':23} ))

# 输出：
my 	name is wangzy and i am 23 old
my 	name is wangzy and i am 23 old
```
### isalnum、isalpha使用：
```ruby
print("ab23".isalnum())   # True
print("#ab23".isalnum())  # False

print("abA".isalpha())    # True
print("ab23".isalpha())   # False

#isalnum是不能有符号的，否则输出false；isalpha是必须纯字母的
```

### 列表：
```ruby
列表循环：
names = ["wangzy","baby",["hello","world"],"shen","hehe"]
for i in  names:
    print(i)
    
#输出：
wangzy
baby
['hello', 'world']
shen
hehe


列表切片：
names = ["wangzy","baby",["hello","world"],"shen","hehe"]
print(names[0:-1:2])  # 0是可以省略的
print(names[:-1:2])

#输出：
['wangzy', ['hello', 'world']]


获取列表下角标：
names = ["wangzy","baby","shen","hehe"]

enumerate(names)
print(enumerate(names))

for i in enumerate(names):
    print(i)
    
# 输出：
<enumerate object at 0x021D66E8>
(0, 'wangzy')
(1, 'baby')
(2, 'shen')
(3, 'hehe')

```

### 元组：
元组其实跟列表差不多，也是存一组数，只不过是它一旦创建，便不能再修改，所以又叫只读列表
```ruby
person = ("wangzy","baby","shen","hehe")
```
它只有2个方法：count、index


### 字符串：
` 字符串是不能被修改的，小写变大写只是把原来的值覆盖了` 
```ruby

首字母大写：
name = "wangzy"
print(name.capitalize())  # 输出：Wangzy

统计字符的个数：
name = "my name is wangzy"
print(name.count("a"))

打印50个字符，将name内容放中间：
name = "my name is wangzy"
print(name.center(50,"-"))  # 输出：----------------my name is wangzy-----------------

判断以什么结尾：
name = "my name is wangzy"
print(name.endswith("zy")) # True

print(name.endswith("my"))  # False


将tab键转成多少个空格：
name = "my \tname is wangzy"

print(name)
print(name.expandtabs(tabsize=30))
输出：
my 	name is wangzy
my                            name is wangzy


字符串切片：
name = "my name is wangzy"

name.find("wangzy")
print(name.find("wangzy"))  # 11

print( name[name.find("wangzy"):])  # wangzy

```

### range介绍：
```ruby
for count in range(1,5):
    print(count)

# 输出：
1
2
3
4
【说明】（1,5）是从1取值，4结束
```
```ruby
for count in range(1,10,2):
    print(count)
    
# 输出：
1
3
5
7
9
【说明】（1,10,2）是每隔2个取一个值
```

- pyc解释：
【说明】这个路径下E:\PycharmProjects\untitled\study\day01\__pycache__会有.pyc文件
python到底是什么？
其实Python和Java/C#一样，也是一门基于虚拟机的语言，我们先来从表面上简单地了解一下Python程序的运行过程吧。
当我们在命令行中输入python hello.py时，其实是激活了Python的“解释器”，告诉“解释器”：你要开始工作了。可是在“解释”之前，其实执行的第一项工作和Java一样，是编译。
熟悉Java的同学可以想一下我们在命令行中如何执行一个Java的程序：
javac hello.java
java hello
只是我们在用Eclipse之类的IDE时，将这两部给融合成了一部而已。其实Python也一样，当我们执行python hello.py时，他也一样执行了这么一个过程，所以我们应该这样来描述Python，Python是一门先编译后解释的语言。

- 简述Python的运行过程：
在说这个问题之前，我们先来说两个概念，PyCodeObject和pyc文件。
我们在硬盘上看到的pyc自然不必多说，而其实PyCodeObject则是Python编译器真正编译成的结果。我们先简单知道就可以了，继续向下看。
当python程序运行时，编译的结果则是保存在位于内存中的PyCodeObject中，当Python程序运行结束时，Python解释器则将PyCodeObject写回到pyc文件中。
当python程序第二次运行时，首先程序会在硬盘中寻找pyc文件，如果找到，则直接载入，否则就重复上面的过程。
所以我们应该这样来定位PyCodeObject和pyc文件，我们说pyc文件其实是PyCodeObject的一种持久化保存方式。
【说明】但是如果改动了源代码，第二次查找再去找.pyc肯定就会出问题(跟原来的代码不匹配了)，那么怎么办呢？python是通过改动时间来判断的，如果源代码的时间比.pyc的时间要新的话，那么就需要重新对源代码重新编译