## python文件操作

### 文件操作：
对文件操作流程
    ① 打开文件，得到文件句柄并赋值给一个变量
    ② 通过句柄对文件进行操作
    ③ 关闭文件
`python3.x 打开文件都是使用open，而在python2.x中可以使用file命令打开`

打开yesterday文件：
```ruby
open("yesterday")
```
打开文件之后可以read()直接读取文件：
```ruby

open("yesterday").read()
# 最好加上编码设置：
F = open("yesterday",encoding='utf-8').read()
print(F)
```

文件的内存对象：
读取2遍文件内容：
```ruby
f = open("yesterday",'r',encoding='utf-8')  # 要写上‘r’读模式，不写‘r’默认也是读模式
data = f.read()
data2 = f.read()
print(data)
print("----------data2-----%s--" %data2)

# 输出：
Somehow, it seems the love I knew was always the most destructive kind
Yesterday when I was young
The taste of life was sweet
As rain upon my tongue
I teased at life as if it were a foolish game
should be at the begging of the second line
----------data2-------
# 【说明】data读取完数据之后，读取数据的“指针”已经指到了文件的最低端，所以data2读取的时候，是在文件的最低端，没什么内容了
```

- 写文件：

r属性：
```ruby
f = open("yesterday",'r',encoding='utf-8')
f.write("我爱北京天安门")

#报错：
f.write("我爱北京天安门")
io.UnsupportedOperation: not writable
```
w属性：
```ruby
# 改为'w':
f = open("yesterday",'w',encoding='utf-8')
f.write("我爱北京天安门")
```
`【注意】单纯把文件设置为w模式，其实是会创建一个文件，如果同名的源文件有内容，那么源文件的内容会丢失`
可以创建一个其他名称的文件，如yesterday2，就能看出单个w会新创建一个文件的效果了

```ruby
f = open("yesterday2",'w',encoding='utf-8')
f.write("我爱北京天安门")

# 直接往yesterday2文件中写两段内容：
f = open("yesterday2",'w',encoding='utf-8')
f.write("我爱北京天安门,\n")
f.write("天安门上太阳升")
```
a属性：
a属性是将内容追加到源文件的最后，不会覆盖掉源文件里面的内容（ a=append）
```ruby
f = open("yesterday2",'a',encoding='utf-8')
```

` 打开文件，最后都要关闭文件：`
```ruby
f.close()
```
`【注意】a(append)也是不能读取文件的`
```ruby
f = open("yesterday",'a',encoding='utf-8')
f.write("我爱北京天安门,\n")

data = f.read()
print("---read",data)
```

r+属性：
下面的代码按逻辑，应该是先读取3行数据，然后写入1行内容“My name is wangzy”，然后再打印1行，但是事实不是这样，“My name is wangzy”这行内容是加到了整个文件的最后
```ruby
f = open("yesterday",'r+',encoding='utf-8')

print(f.readline())
print(f.readline())
print(f.readline())

f.write("-----My name is wangzy-----")

print(f.readline())

# 输出：
Somehow, it seems the love I knew was always the most destructive kind

不知为何，我经历的爱情总是最具毁灭性的的那种

Yesterday when I was young

昨日当我年少轻狂

# yesterday文件尾部内容：
终于到了付出代价的时间 为了昨日
When I was young
当我年少轻狂-----My name is wangzy-----
```
r+属性：
```ruby
# 读写：
f = open("yesterday",'r+',encoding='utf-8')

f = open("yesterday",'r+',encoding="utf-8")

f.write("------My name is babashen1-------\n")
f.write("------My name is babashen2-------\n")
f.write("------My name is babyshen3-------\n")
f.write("------My name is babyshen4-------\n")
f.write("------My name is babyshen5-------\n")

print(f.tell())
f.seek(10)
print(f.tell())
print(f.readline())

f.write("should be at the begging of the second line")
f.close()

# 输出：
175
10
ame is babashen1

# 开始写入的内容“My name is babyshenx”正好总共175个字符
f = open("yesterday",'r+',encoding="utf-8")
f.seek(175)
print(f.readline())
# 将yesterday源文件输出175个字符之后的内容是：it were a foolish game
执行完程序的yesterday文件内容：

开始文件的指针在文件的开始处(seed(0))，插入的文件内容直接按照字符个数匹配覆盖掉原来的内容，后来“指针”指在文件内容的中间位置，再次插入的文件内容(should be at the begging of the second line)不能在文件内容中间插入，而是直接插入到文件底部

```
w+属性：
```ruby
# 先创建一个文件，再向文件中写内容，该功能作用不是很大
f = open("yesterday",'w+',encoding='utf-8')
```
a+属性：
```ruby
# 追加读写：
f = open("yesterday",'a+',encoding='utf-8')
```
rb属性:
```ruby
# 二进制文件：
f = open("yesterday",'rb',encoding='utf-8')

f = open("yesterday",'rb')

print(f.readline())
print(f.readline())
print(f.readline())
# 二进制不需要写encoding=‘utf-8’

f = open("yesterday",'rb')

print(f.readline())
print(f.readline())
print(f.readline())

# 网络传输只能用二进制形式

```

wb属性：
```ruby
f = open("yesterday",'wb')
f.write("hello binary\n")
f.close()

# 报错信息：
TypeError: 'str' does not support the buffer interface

# 不让写入字符串，需要字符串和二进制的转换
f = open("yesterday",'wb')
f.write("hello binary\n".encode())
f.close()
```

### readline
```ruby
输出两行内容，就是写两次 print (f.readline())
f = open("yesterday",'r',encoding='utf-8')

print(f.readline())
print(f.readline())

如果多写几行可以使用for循环：
f = open("yesterday",'r',encoding='utf-8')

for i in range(5):
    print(f.readline())
```
### readlines() 
readlines() 将文件的全部内容打印成一个“列表”的形式
```ruby
f = open("yesterday",'r',encoding='utf-8')

print(f.readlines())

# line.strip() 是将空格和换行都去掉
for line in f.readlines():
    print(line.strip())
```

### startswith
去掉空格和换行之后，判断行是不是以dis开头
```ruby
f = open("tarfile1",'r',encoding="utf-8")

for line in f.readlines():
    if line.strip().startswith("dis"):
        print("yes")
    else:
        print("no")
```

实验一：不想打印第10行数据，其余全部打印出来
思路：直接取下角标，下角标为9就是第10行，不需要取出来(index，enumerate是打印下角标)
```ruby
f = open("yesterday",'r',encoding='utf-8')

for index,line in enumerate(f.readlines()):
    if index == 9:
        print("---------this is tenth line----------")
        continue

    print(index,line.strip())

# 输出：
0 Somehow, it seems the love I knew was always the most destructive kind
1 不知为何，我经历的爱情总是最具毁灭性的的那种
2 Yesterday when I was young
3 昨日当我年少轻狂
4 The taste of life was sweet
5 生命的滋味是甜的
6 As rain upon my tongue
7 就如舌尖上的雨露
8 I teased at life as if it were a foolish game
---------this is tenth line----------
10 The way the evening breeze
11 就如夜晚的微风
```
【注意】f.readline只适合读小文件，因为f.readline是将文件的全部内容读取到内存中，如果是一个大文件，那么就会把内存撑爆了
【说明】如果处理大文件，可以一行一行的读取，读取到内存中，处理一行释放一行，这样内存中只会保存一行数据
```ruby
f = open("yesterday",'r',encoding='utf-8')

for line in f:
    print(line)

# 这样处理已经不是一个列表了，这样就是从文件中一行一行的读取
```
用这种方式实现实验一的需求(不打印第10行，其余全打印)
```ruby
f = open("yesterday",'r',encoding='utf-8')

count = 0
for line in f:
    if count == 9:
        print("------------ this is tenth lines--------------")
        count += 1
        continue
    print(line)
    count += 1
```

### tell：打印当前指向的位置
- 文件“指针”
【说明】开始时的指向位置是0
```ruby
f = open("yesterday",'r',encoding='utf-8')

print(f.tell())
print(f.readline())

print(f.tell())

# 输出：
0
Somehow, it seems the love I knew was always the most destructive kind

72

# 读一行内容之后就变成了72，说明“指针”是按照字符数计算的
```
读取5个字符，“指针”就指到了第5个字符位置
```ruby
f = open("yesterday",'r',encoding='utf-8')

print(f.tell())
print(f.read(5))

print(f.tell())

# 输出：
0
Someh
5
```

### seek：将指针的指向回到原点
```ruby
f = open("yesterday",'r',encoding='utf-8')

print(f.tell())
print(f.readline())
print(f.tell())

f.seek(0)
print(f.tell())
```
- 打印文件的编码：
```ruby
f = open("yesterday",'r',encoding='utf-8')

print(f.encoding)
```

### 内存数据强制刷新到硬盘：
```ruby
f = open("test_flush.txt",'w')
f.write("hello\n")
# 执行完write之后，查看test_flush.txt文件，并没有把hello写入到test_flush.txt中
f.flush()
# 再查看test_flush.txt文件，出现了写入的hello
```
- 制作进度条功能：
```ruby
import sys,time

for i in range(20):
    sys.stdout.write("#")
    sys.stdout.flush()
    time.sleep(0.1)
```
### truncate
`truncate()什么都不指定，就是直接把文件的内容清空 ，文件指针需要指回文件最初点`
```ruby
f = open("yesterday",'a',encoding='utf-8')
f.truncate()

# truncate(5)
f = open("yesterday",'a',encoding='utf-8')
f.truncate(5)

# 输出：
Someh

f = open("yesterday",'a',encoding='utf-8')
f.seek(0)
print(f.tell())   # 输出0
f.seek(10)
print(f.tell())   # 输出10
f.truncate(20)

# 按照上面的程序设计，seek(10)，文件指针应该先指向到第10个字符，然后再保留20个字符，然后再删除剩余的全部内容，但是实际的输出却不是这样，即seek(10)不好用
# truncate(20)就是从文件的最开头（seek(0)）处，开始取20个字符
```

实验二：修改大文件中的某一项内容
文件内容如下：
```ruby
So many lovely songs were waiting to be sung
有那么多甜美的曲儿等我歌唱
So many wild pleasures lay in store for me
有那么多肆意的快乐等我享受  # 将“我”改为“汪宗宇”
And so much pain my eyes refused to see
还有那么多痛苦 我的双眼却视而不见
```
思路：大文件不能直接在内存中打开，可以一行一行打开，然后一行一行的匹配寻找修改
新创建一个新的文件(yesterday_new)，将改好的数据写入到新的文件中
```ruby

f = open("yesterday",'r',encoding='utf-8')
new_f = open("yesterday_new",'w',encoding='utf-8')

for line in f:
    if "有那么多肆意的快乐等我享受" in line:
        line = line.replace('有那么多肆意的快乐等我享受','有那么多肆意的快乐等汪宗宇享受')
    new_f.write(line)

f.close()
new_f.close()

# 其他写法：
 if "有那么多肆意的快乐等我享受" in line:
        line = line.replace('有那么多肆意的快乐等我享受','有那么多肆意的快乐等汪宗宇享受')
        new_f.write(line)
    else:
        new_f.write(line)
```

### with语句：
为了避免打开文件后忘记关闭，可以通过管理上下文，即
```ruby
with open('log','r') as f:
    pass

# 当with代码块执行完毕时，内部会自动关闭并释放文件资源
```
`在python2.7之后，with又支持同时对多个文件的上下文进行管理`
```ruby
with open('log1','r') as obj1,open('log2') as obj2:
    pass
    

with open("yesterday",'r',encoding="utf-8") as f:
    print(f.readline())
    

python开发者建议每行最多不能超过80个字符，所以打开多个文件，超出字符要换行：
with open("yesterday",'r',encoding="utf-8") as f:\
    open("yesterday",'r',encoding='utf-8') as f2
```