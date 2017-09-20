# Markdown语法 #


### 标题 ###

**两种方式设置标题：**

方式1 ：
# H1 

## Welcome to MarkdownPad 2 ##

###### H6
【注意】末尾可以加上也可以不加#

方式2：

This is an H1
=============

This is an H2
-------------

### 强调 ###


**MarkdownPad** is a full-featured `Markdown` editor for Windows.


*斜体1*	斜体1
_斜体2_	斜体2
**粗体1**	粗体1
__粗体2__	粗体2
这是一个 ~~删除线~~	这是一个 删除线
***斜粗体1***	斜粗体1
___斜粗体2___	斜粗体2
***~~斜粗体删除线1~~***	斜粗体删除线1
~~***斜粗体删除线2***~~	斜粗体删除线2
说明：斜体、粗体、删除线可混合使用


### 列表 ###

Give them a try:

- **Bold** (`Ctrl+B`) and *Italic* (`Ctrl+I`)
- Quotes (`Ctrl+Q`)
- Code blocks (`Ctrl+K`)
- Headings 1, 2, 3 (`Ctrl+1`, `Ctrl+2`, `Ctrl+3`)
- Lists (`Ctrl+U` and `Ctrl+Shift+O`)

+   Red
+   Green
+   Blue

*   Red
*   Green
*   Blue

*   Lorem ipsum dolor sit amet, consectetuer adipiscing elit.
Aliquam hendrerit mi posuere lectus. Vestibulum enim wisi,
viverra nec, fringilla in, laoreet vitae, risus.

1.  Bird
1.  McHale
1.  Parish

### 连接 ###

Markdown编辑工具：[Markdown编辑工具](http://www.williamlong.infoarchives/4319.html) 
Markdown语法：[Markdown语法](http://www.appinn.com/markdown/)

Don't guess if your [hyperlink syntax](http://markdownpad.com) is correct; LivePreview will show you exactly what your document looks like every time you press a key.


### 引用 ###

说明：>> 这个是引用符号

>> MarkdownPad supports multiple Markdown processing engines, including standard Markdown, Markdown Extra (with Table support) and GitHub Flavored Markdown.

>>> With a tabbed document interface, PDF export, a built-in image uploader, session management, spell check, auto-save, syntax highlighting and a built-in CSS management interface

如果要在列表项目内放进引用，那 > 就需要缩进：
* there's no limit to what you can do with MarkdownPad
    > This is a blockquote
    > inside a list item.



### 分割线 ###
***
* * *

***

*****

- - -

---------------------------------------

### 代码  ###
在Tool ——> Options ——> Markdown ——> GitHub Flavored Markdown 选择GitHub风格
破解：
Soar360@live.com
授权秘钥：
GBPduHjWfJU1mZqcPM3BikjYKF6xKhlKIys3i1MU2eJHqWGImDHzWdD6xhMNLGVpbP2M5SN6bnxn2kSE8qHqNY5QaaRxmO3YSMHxlv2EYpjdwLcPwfeTG7kUdnhKE0vVy4RidP6Y2wZ0q74f47fzsZo45JE2hfQBFi2O9Jldjp1mW8HUpTtLA2a5/sQytXJUQl/QKO0jUQY4pa5CCx20sV1ClOTZtAGngSOJtIOFXK599sBr5aIEFyH0K7H4BoNMiiDMnxt1rD8Vb/ikJdhGMMQr0R4B+L3nWU97eaVPTRKfWGDE8/eAgKzpGwrQQoDh+nzX1xoVQ8NAuH+s4UcSeQ==

Markdown连接github：
在Tool ——> Options ——> Markdown 设置连接github的账号

```
real_username = 'liangyx'
real_password = '123456'

count = 0
while count < 3:
    username = input("请输入用户名>>>：")
    userpassword = input("请输入密码>>>: ")
    f = open('black_username','r')
    for lock_user in f:
        if lock_user.strip() == username:  # 不加strip时候，每读取一行后面有一个换行符
            print("你早就已经被锁定了！")
            count = 4
    f.close()   # 跟open文件要对其
    if real_username == username and real_password == userpassword:
        print('登陆成功！！')
        count = 4
    else:
        print("登陆失败")
        count += 1
        if count == 3:
            f = open('black_username','a')
            f.write("\n%s"%username)
            f.close()

```


### 图片 ###
Markdown 使用一种和链接很相似的语法来标记图片，同样也允许两种样式： 行内式和参考式。

行内式的图片语法看起来像是：
![Alt text](/path/to/img.jpg)

![Alt text](/path/to/img.jpg "Optional title")

详细叙述如下：

* 一个惊叹号 !
* 接着一个方括号，里面放上图片的替代文字
* 接着一个普通括号，里面放上图片的网址，最后还可以用引号包住并加上 选择性的 'title' 文字。

参考式的图片语法则长得像这样：
![Alt text][id]

「id」是图片参考的名称，图片参考的定义方式则和连结参考一样：
[id]: url/to/image  "Optional title attribute"

URL即图片的url地址，如果引用本仓库中的图片，直接使用相对路径就可了，如果引用其他github仓库中的图片要注意格式，即：仓库地址/raw/分支名/图片路径，如：

https://github.com/guodongxiaren/ImageCache/raw/master/Logo/foryou.gif
语法：![baidu](http://www.baidu.com/img/bdlogo.gif "百度logo")

![](https://github.com/ZongYuWang/Python3.x/blob/master/image/coding.png)

### 自动链接 ###

Markdown 支持以比较简短的自动链接形式来处理网址和电子邮件信箱，只要是用方括号包起来， Markdown 就会自动把它转成链接。一般网址的链接文字就和链接地址一样
<http://example.com/>


### 反斜杠 ###
Markdown 支持以下这些符号前面加上反斜杠来帮助插入普通的符号：
 反斜线 \
 反引号 `
 星号  *
 底线 _
 花括号 {}
 方括号 []
 括弧  （）
 井字号  #
 加号  +
 减号 -
 英文句点 .
 惊叹号  ！

### 表格 ### 
|每天 |主食 |价格 |
|--------|---------|-------|
|周一 |面<br>食 |$6 |
|周二 |鸡 |$8 |

注意：表格中的“面<br>食”，如果想指定某个位置换行的话，需要使用“<br>”标签，直接使用<Enter>回车键是不行的。

