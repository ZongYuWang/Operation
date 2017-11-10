## python文件操作

文件操作：
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