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

### 判断十进制：
```ruby
print("1A".isdigit())  # 输出：False

```
### 判断是不是整数：
```ruby
print("1A".isdigit())  # 输出：False
print("123".isdigit()) # 输出：True
```

### 判断是不是合法的标识符：
```ruby
print("1A".isidentifier())  # 输出：False
print("123".isidentifier()) # 输出：False
```

### join(把列表转换成字符串):
```ruby
print("+".join(["1","2","3"]))   # 输出：1+2+3
```
### ljuest(原字符在左侧输出，右侧补齐字符):
```ruby
name = "My name is wangzy"
print(name.ljust(50,"*"))

# 输出：
My name is wangzy*********************************

#首先保证这句话长50个字符，如果不够的话，用星号补上
```

### rjust(原字符在右侧，左侧补齐字符)
```ruby
name = "My name is wangzy"
print(name.rjust(50,"+"))
```

### lower/upper(字符大小写转换)：
```ruby
print("Wangzy".lower())  # 输出：wangzy
print("wangzy".upper())  # 输出：WANGZY
```

### lstrip/rstrip/strip(自动去掉空格和换行)
```ruby
lstrip()
rstrip()
strip() :左边和右边的空格和换行都去掉

print("\n      Wangzy".lstrip())
print("-------------")

print("\nwangzy\n       ".rstrip())
print("-------------")

# 输出：
Wangzy
-------------

wangzy
-------------

```

### maketrans(字符替换)：
`a替换1 b替换2 c替换3 d替换4 e替换5 f替换6`
```ruby
p = str.maketrans("abcdef","123456")
print("wang zong yu".translate(p))

# 输出：
w1ng zong yu
# 根据上面的匹配原则，wang中的w、n、g都没有匹配项，a匹配1，所以会输出a1ng zong yu
```


### replace(将某个字符替换成另外一个字符):
```ruby
print("wang zong yu".replace("n","N"))  # 输出：waNg zoNg yu

# 只替换一个相关字符
print("wang zong yu".replace("n","N",1)) # 输出：waNg zong yu
```

### rfind(顺序是自左至右找，但是如果有多个重复的，选取最右侧的，返回下角标值):
```ruby
print("wang zong yu".rfind("u"))  # 输出：11

print("wang zong yu".rfind("n"))  # 输出：7
```

### split(将字符串按照空格分成列表)：
```ruby
print("wang zong yu".split())

#按照n进行分割：
print("wang zong yu".split("n"))

# 输出：
['wang', 'zong', 'yu']
['wa', 'g zo', 'g yu']
# 被当成分隔符的“n”会没了
```

###  splitlines：
```ruby
print("1+2\n3+4".splitlines())

# 输出：
['1+2', '3+4']
```

### startswith/endswitch(判断以什么字符开始/判断以什么字符结束)：
```ruby
print("12345".startswith("1"))      # 输出：True
print("wangzy".startswith("y"))     # 输出：False

print("wangzy".endswith("zy"))      # 输出：True
print("wangzy".endswith("wangzy"))  # 输出：True
```

### swapcase(大写字符转换成小写字符，小写字符转换成大写字符)：
```ruby
print("wANG ZONG yU".swapcase())   # 输出：Wang zong Yu
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

### 字典：
` 字典是无序的，是因为没有下标`
```ruby
info = {
    "person1":"wangzy",
    "person2":"baby",
    "person3":"shen"
}

print(info)
print(info["person1"])

info["person3"] = "hello"
print(info)

# 输出：
{'person2': 'baby', 'person1': 'wangzy', 'person3': 'shen'}
wangzy
{'person2': 'baby', 'person1': 'wangzy', 'person3': 'hello'}
```
首先得确定字典中一定有那个key值，如果不确定是否有那个key值，则使用下面的办法：
```ruby
info = {
    "person1":"wangzy",
    "person2":"baby",
    "person3":"shen"
}

print(info.get("person2"))  # 输出：baby
print(info.get("person4"))  # 输出：None

# 在python2.x中使用语法 info.has_key(“person1”)
```

如果没有对应的key就是在字典中添加一项，如果有对应的key就是在字典中修改key对应的值
```ruby

info = {
    "person1":"wangzy",
    "person2":"baby",
    "person3":"shen"
}

info["person3"] = "hello"
info["person4"] = "world"

print(info)

# 输出：
{'person1': 'wangzy', 'person3': 'hello', 'person2': 'baby', 'person4': 'world'}
```
- 删除字典数据：
```ruby
# 方式1：
del info["person3"]

方式2：
info.pop("person3")

方式3：popitem是随机删除一个
info.popitem()
```

### 多级字典嵌套及操作：
```ruby
av_catalog = {
    "欧美":{
        "www.youporn.com": ["很多免费的,世界最大的","质量一般"],
        "www.pornhub.com": ["很多免费的,也很大","质量比yourporn高点"],
        "letmedothistoyou.com": ["多是自拍,高质量图片很多","资源不多,更新慢"],
        "x-art.com":["质量很高,真的很高","全部收费,屌比请绕过"]
    },
    "日韩":{
        "tokyo-hot":["质量怎样不清楚,个人已经不喜欢日韩范了","听说是收费的"]
    },
    "大陆":{
        "1024":["全部免费,真好,好人一生平安","服务器在国外,慢"]
    }
}

修改：
av_catalog ["大陆"]["1024"][1] = "可以做镜像下载"
print(av_catalog ["大陆"]["1024"])
# 【注意】key尽量不要使用中文

# 输出：
['全部免费,真好,好人一生平安', '可以做镜像下载']
```

### setdefault(先去字典取值，如果没有对应的值，就创建一个，如果有也不会对原来的做修改操作)
```ruby
av_catalog.setdefault("大陆",{"www.baidu.com":["哈哈","还可以"]})
print(av_catalog["大陆"])

这样就新增了一条“taiwan”
把key=“taiwan”设置成key=“大陆”，原来的key=“大陆”那条也不会被修改，还会是原来的

# 输出：
{'1024': ['全部免费,真好,好人一生平安', '服务器在国外,慢']}
```

### update(两个字典合并，有交叉的就覆盖，没有交叉的就合并了)
```ruby

info = {
    "person1":"wangzy",
    "person2":"baby",
    "person3":"shen"
}

info_new = {
   "person1":"babyshen",
   "man1":"nihao",
   "man2":"hehe"
}

info.update(info_new)
print(info)
print(info_new)

# 输出：
{'man1': 'nihao', 'person1': 'babyshen', 'person3': 'shen', 'person2': 'baby', 'man2': 'hehe'}
{'person1': 'babyshen', 'man2': 'hehe', 'man1': 'nihao'}
```

### items(将字典转成列表): 
```ruby
info = {
    "person1":"wangzy",
    "person2":"baby",
    "person3":"shen"
}

print(info.items())

# 输出：
dict_items([('person2', 'baby'), ('person1', 'wangzy'), ('person3', 'shen')])
```

- 通过一个列表生成默认的字典
` 这是一个坑，少用`
```ruby
c = dict.fromkeys([1,2,3],'test')
print(c)
print(c.items())

# 输出：
{1: 'test', 2: 'test', 3: 'test'}
dict_items([(1, 'test'), (2, 'test'), (3, 'test')])


c = dict.fromkeys([1,2,3])
print(c)
# 输出：
{1: None, 2: None, 3: None}

# 列表中如果后面没有值，转化成字典的时候，value值都会显示none


做一下修改：
c = dict.fromkeys([6,7,8],[1,{"name":"wangzy"},444])
print(c)

c[7][1]["name"] = "baby"
print(c)

# 输出：
{8: [1, {'name': 'wangzy'}, 444], 6: [1, {'name': 'wangzy'}, 444], 7: [1, {'name': 'wangzy'}, 444]}
{8: [1, {'name': 'baby'}, 444], 6: [1, {'name': 'baby'}, 444], 7: [1, {'name': 'baby'}, 444]}

# 【注意】大坑，居然将值全改了
```

### 字典打印 键/值：
```ruby

info = {
    "person1":"wangzy",
    "person2":"baby",
    "person3":"shen"
}

# 打印键：
print(info.keys())

# 输出：
dict_keys(['person1', 'person2', 'person3'])

# 打印值：
print(info.values())

# 输出：
dict_values(['shen', 'baby', 'wangzy'])

```

### 字典循环：

```ruby
info = {
    "person1":"wangzy",
    "person2":"baby",
    "person3":"shen"
}

# 打印key
for i in info:
    print(i)
    
# 输出：
person3
person2
person1


# 打印value：

for i in info:
    print(info[i])
    
# 输出：
shen
baby
wangzy

# 打印key：value
# 方式1：这种方式比方式2效率高
for i in info:
    print(i,info[i])
    
# 方式2：
for k,v in info.items():
    print(k,v)
```


### 集合：
```ruby
两种生成集合的方式：

#方式1：
list_1 = [1,4,5,7,3,6,7,9]
list_1 = set(list_1)

print(list_1,type(list_1))
# 输出：
{1, 3, 4, 5, 6, 7, 9} <class 'set'>

方式2：
list_2 = set([2,6,0,66,22,8,4])

print(list_2)
# 输出：
{0, 2, 66, 4, 6, 8, 22}

# 集合中是没有重复项的，集合也是无序的
```

- 集合的交接（也可以使用符号：list_1 & list_2）：
```ruby

list_1 = [1,4,5,7,3,6,7,9]
list_1 = set(list_1)

list_2 = set([2,6,0,66,22,8,4])

print(list_1.intersection(list_2)) # 输出：{4, 6}
```

- 集合的并集（也可是使用符号：list_1 | list_2）：
```ruby
print(list_1.union(list_2))
```
- 差集：（也可以使用符号：list_1 - list_2）(in list_1 but not in list_2)
```ruby
print(list_1.difference(list_2))  # 输出:{1, 9, 3, 5, 7}
print(list_2.difference(list_1))  # 输出:{0, 8, 2, 66, 22}
```

- 判断子集:
```ruby
print(list_1,list_2)
print(list_1.issubset(list_2))

# 输出:
{1, 3, 4, 5, 6, 7, 9} {0, 2, 66, 4, 6, 8, 22}
False
```
- 判断父集:
```ruby
list_1 = [1,4,5,7,3,6,7,9]
list_1 = set(list_1)
list_3 = set([1,3,7])
print(list_3.issubset(list_1))   # 输出True
print(list_1.issuperset(list_3)) # 输出True
```
- 对称差集：（也可以使用符号：list_1 ^ list_2）(在list_1或list_2中，但不会同时出现在二者中)
```ruby
list_1 = [1,4,5,7,3,6,7,9]
list_1 = set(list_1)
list_2 = set([2,6,0,66,22,8,4])
list_3 = set([1,3,7])

print(list_1.symmetric_difference(list_2))

# 输出：
{0, 1, 2, 66, 3, 5, 7, 8, 9, 22}
```

- isdisjoint
```ruby
list_3 = set([1,3,7])
list_4 = set([5,6,8])

# Return True if two sets have a null intersection
print(list_3.isdisjoint(list_4))

# return True

# 如果list_4的集合是{5,6,7,3,1}，那么就会返回false
```

- 集合的添加：
```ruby
list_1.add(999)
print(list_1)

# 输出：
{1, 3, 4, 5, 6, 7, 999, 9}


# 在list_2中添加多项：
list_2 = set([2,6,0,66,22,8,4])
list_2.update([10,37,42])

print(list_2)  # 输出：{0, 2, 66, 4, 37, 6, 8, 10, 42, 22}

```

- 集合的删除：
```ruby
remove：指定删除一项：

list_1 = set([1,4,5,7,3,6,7,9])
list_1.remove(9)
print(list_1)  # 输出：{1, 3, 4, 5, 6, 7}
#如果出现两个9，会删除第几个9呢？答：集合不会出现两个9，因为集合中是没有重复项的
#【注意】remove如果删除一个不存在的数据会报错

pop：随机删除一项：
list_1 = set([1,4,5,7,3,6,7,9])
print(list_1.pop())  # 输出：1
print(list_1)        # 输出：{3, 4, 5, 6, 7, 9}

discard：指定删除一项：
list_1 = set([1,4,5,7,3,6,7,9])
print(list_1.discard(6))  # 输出：None
print(list_1)             # 输出：{1, 3, 4, 5, 7, 9}
# discard无论删除的数据是否存在集合中，都会提示none
# 不用print，删就删除了，不会有提示，其实print不用写了

```

- len:set的长度：
```ruby
list_1 = set([1,4,5,7,3,6,7,9])
length = len(list_1)
print(length)  # 输出7
```
- in：判断x是否为 list_1的成员
```ruby
list_1 = set([1,4,5,7,3,6,7,9])
print(5 in list_1)    # 输出：True
print(100 in list_1)  # 输出：False
```
- not in：判断x是否不是list_1的成员
```ruby
list_1 = set([1,4,5,7,3,6,7,9])
print(5 not in list_1)
print(100 not in list_1)
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