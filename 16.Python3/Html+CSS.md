## HTML
`HTML相当于一个赤裸裸的人站在那，CSS相当于给人穿上华丽的衣服，JS相当于让这个人动起来`

### 1、开发后台程序：
- 写html文件(充当模板的作用)
- 数据库获取数据，然后替换到html文件的指定位置(web框架)

### 2、编写HTML文件：
```html
放在html页顶端，表示使用标准的html语法对应关系(当然也有其他的语法对应关系)
<!DOCTYPE html>  

注释：
 <!--  注释的内容  -->

html标签，如<html>babyshen</html>(只能有一个)：标签内部可以写属性

html页固定格式：
<!DOCTYPE html>
<html lang="en">
<head>
  ......
</head>
<body>
  .......
</body>
</html>

lang="en"，标签内部的属性,可以写也可以不写：
<html lang="en">
```

### 3、标签分类：
#### 3.1 自闭合标签:
```html
<meta charset="UTF-8">
```

#### 3.2 主动闭合标签:
```html
 <title>babyshen</title>
```
### 4、head标签：
#### 4.1 meta标签:
```html
定义编码：
<meta charset="UTF-8">  
```
```html
10秒就跳转到后面的网址:
<meta http-equiv="Refresh" content="10;Url=http://www.baidu.com"> 
```
```html
每页每隔10秒自动刷新:
<meta http-equiv="Refresh" content="10">  
```

```html
设置搜索关键字
<meta name="keywords" content="汽车,汽车之家,汽车网,汽车报价,汽车图片,新闻,评测,社区,俱乐部"> 

```
```html
IE兼容(支持IE8、IE7)
<meta http-equiv="X-UA-COMPATIBLE" content="IE=IE8;IE=IE7;"> 
```
#### 4.2 title标签：
```html
 <title>babyshen</title>
```
#### 4.3 link标签：
```html
图标：
<link rel="shortcut icon" href="image/favicon.ico">
```

### 5、body标签：
所有的标签分为：      
块级标签(占据一整行)：div标签(白板)、h系列(加大加粗)、p标签(段落和段落之间有间距)；        
行内标签(内联标签)：span标签(白板)       
`标签也是可以实现嵌套的,如下，div里面嵌套div或者嵌套span`
```html
<body>
    <div>
        <div></div>
        <span></span>
    </div>
</body>
```

#### 5.1 图标:
[网页特殊符号HTML代码大全](https://www.cnblogs.com/web-d/archive/2010/04/16/1713298.html)
```html
一个&nbsp表示一个空格
<a href="http://www.baidu.com">百&nbsp;&nbsp;&nbsp;&nbsp;&lt;a&gt;度</a> 
```

#### 5.2 p标签:
```html
<p>是段落
<p>wang<br />wangzongyu</p> 
<p>zong</p>                
<p>yu</p>
```
#### 5.3 br标签:
```html
换行
<br>就是换行
```
#### 5.4 h系列：
```html
<h1>wang</h1>
<h6>yu</h6>
```
#### 5.5 span标签：
```html
<span>hello</span>
<span>hello</span>
<span>hello</span>
```
#### 5.6 div标签
```html
<div>wang</div>
<div>zong</div>
<div>yu</div>
```
#### 5.7 a标签：
##### 跳转:
```html
跳转到百度产生一个新页，不会在原网页的基础上打开百度
<body>
    <a href="http://www.baidu.com" target="_blank">百度</a>
</body>
```
##### 锚:
```html
<body>
    <a>第一章</a>
    <a>第二章</a>
    <a href="#i3">第三章</a>
    <a>第四章</a>
    <div style="height: 600px;">第一章内容</div>
    <div style="height: 600px;">第二章内容</div>
    <div id="i3" style="height: 600px;">第三章内容</div>
    <div style="height: 600px;">第四章内容</div>
</body>

href='#某个标签的ID'，标签的ID不允许重复
```

#### 5.8 input系列：
```ruby
input type='text'      可加name属性
input type='password'  可加name属性
input type='submit'    -value='提交' 提交按钮，表单
input type='button'    -value='登录' 按钮

input type='radio'    - 单选框 value，checked="checked"，name属性（name相同则互斥）
input type='checkbox' - 复选框 value, checked="checked"，name属性（批量获取数据）
input type='file'     - 依赖form表单的一个属性 enctype="multipart/form-data"
input type='rest'     - 重置
    
<textarea >默认值</textarea>  可加name属性
select标签            可加name,内部option value, 提交到后台，size，multiple
```
- 实验演练：
安装tornado：
```ruby
C:\Users\Administrator>pip install tornado
Collecting tornado
  Downloading tornado-4.5.2.tar.gz (483kB)
```
服务端代码(tornado-web框架)：
```ruby
import tornado.ioloop
import tornado.web

# pip3 install tornado
class MainHandler(tornado.web.RequestHandler):
    def get(self):
        print(111)
        u = self.get_argument('user')
        e = self.get_argument('email')
        p = self.get_argument('pwd')
        if u == 'alex' and p == '123' and e == 'alex@126.com':
            self.write("OK")
        else:
            self.write("滚")

    def post(self, *args, **kwargs):
        u = self.get_argument('user', None)
        e = self.get_argument('email', None)
        p = self.get_argument('pwd', None)
        print(u, e, p)
        self.write('POST')

application = tornado.web.Application([
    (r"/index", MainHandler),
])
if __name__ == "__main__":
    application.listen(8888)
    tornado.ioloop.IOLoop.instance().start()
    
# 客户端请求之后，服务端输出信息：
111
wangzy wangzy@163.com 123456
```

客户端代码(WEB)：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
    <form action="http://localhost:8888/index" method="get" >
        <!-- form只要一点击，就是提交当前form里面的表单 -->
        <input type="text" name="user" />
        <input type="text" name="email" />
        <input type="password" name="pwd" />
        <input type="button" value="登陆1" />
        <input type="submit" value="登陆2" />
        <!-- 其实用户数据就是打包发送到后台，按照下面的格式 -->
        <!-- {'user':'用户输入的用户名'，'email':'xxxx','pwd':'xxxx'} -->
    </form>
    <br />
    <form action="http://localhost:8888/index" method="post" >
        <!-- form只要一点击，就是提交当前form里面的表单 -->
        <input type="text" name="user" />
        <input type="text" name="email" />
        <input type="password" name="pwd" />
        <input type="button" value="登陆1" />
        <input type="submit" value="登陆2" />
    </form>
</body>
</html>
```
使用get方法点击登录2之后浏览器返回值：
```ruby
# <form action="http://localhost:8888/index" method="get" >
http://localhost:8888/index?user=wangzy&email=wangzy%40163.com&pwd=123456
```
使用post方法点击登录2之后浏览器返回值：
```ruby
# <form action="http://localhost:8888/index" method="post" >
http://localhost:8888/index
```
- GET和POST区别：
提交的时候内容不一样，一个是放在body里面发过去了，一个是在url里(把值都放在url中提交过去了)，服务端怎么发过来的，怎么取

##### 设置可以多行输入功能(<textarea>)：
```html
<body>
    <textarea name="meno">babyshen</textarea>
</body>
```
`<textarea>默认值</textarea> 有name属性`

##### 设置搜索功能(type="submit" value="搜索")：
```html
<body>
   <form action="https://www.sogou.com/web">
       <input type="text" name="query" />  
       <input type="submit" value="搜索" />
   </form>
</body>

type="text"也可以设置value值，表示默认值

<body>
   <form action="https://www.sogou.com/web">
       <input type="text" name="query" value="wangzy" />
       <input type="submit" value="搜索" />
   </form>
</body>
```
`同时也可以嵌套，但是div里面的值不会提交到后台，只是input里面的值可以提交到后台`
```html
<body>
   <form action="https://www.sogou.com/web">
       <div>
           <p>hello p</p>
           <br>
           <span>hello span</span>
       </div>
       <input type="text" name="query" value="wangzy" />
       <input type="submit" value="搜索" />
   </form>
</body>
```

##### 设置单选框(input type="radio"):
`input type="radio"  单选框`
```html
<body>
    <form>
        <div>
            <p>请选择性别：</p>
            男：<input type="radio" name="gender" value="1">
            女：<input type="radio" name="gender" value="2">
        </div>
        <input type="submit" value="提交" />
    </form>
</body>
```
`男和女设置的name值一样，为了在选择时只能选择一个，如果不设置name值，那么男女可以多选，name相同则互斥`

##### 设置复选框(input type="checkbox")
```html
<body>
    <form>
        <div>
            <p>请选择性别：</p>
            男：<input type="radio" name="gender" value="1">
            女：<input type="radio" name="gender" value="2">

            <p>爱好</p>
            看书：<input type="checkbox" name="favor" value="1">
            看电影：<input type="checkbox" name="favor" value="2">
            跑步：<input type="checkbox" name="favor" value="3">

            <p>技能</p>
            写代码：<input type="checkbox" name="skill">
            玩：<input type="checkbox" name="skill">
        </div>
        <input type="submit" value="提交" />
    </form>
</body>

浏览器返回值：
http://localhost:63342/study/day14/choice.html?favor=1&favor=2&favor=3
```
##### 默认值设置：
```html
男：<input type="radio" name="gender" value="1" checked="checked">

checked="checked"表示该项是默认值
```
##### 上传文件：
`input type='file' 依赖form表单的一个属性 enctype="multipart/form-data"`
```html
<p>上传文件</p>
<input type="file" name="fname">

这样写默认不能上传文件，需要在form表单中写enctype="multipart/form-data"
<body>
    <form enctype="multipart/form-data">
         <p>上传文件</p>
            <input type="file" name="fname">
    </form>
</body>
```

##### 重置(input type="reset")：
```html
<input type="reset" value="重置" />
```

##### 下拉框(select标签)：
```ruby
<body>
   <select name="city">
       <option value="1">北京</option>
       <option value="2">天津</option>
       <option value="3">上海</option>
   </select>
</body>

浏览器会有返回值如：city=1就是将北京传递给后台程序

设置天津是默认值：
<option value="2" selected="selected">天津</option>

```
`多选：`
```ruby
  <select name="city" size="10" multiple="multiple" >
```

#### 5.9 img标签：
##### src：
```html
<body>
    <img src="image.png" style="height: 200px;width: 200px;">
</body>
```
```html
点击图片可以跳转到百度：
<body>
    <a href="http://www.baidu.com">
       <img src="image.png" style="height: 200px;width: 200px;">
    </a>
</body>
```
##### alt：
```html
图片位置或者图片找不到的时候，该处显示"流程图"
<body>
    <a href="http://www.baidu.com">
       <img src="123.png" style="height: 200px;width: 200px;" alt="流程图">
    </a>
</body>
```
##### title：
```html
鼠标放在图片上会显示"python图片"字样
<body>
    <a href="http://www.baidu.com">
       <img src="image.png" title="python图片" style="height: 200px;width: 200px;" alt="流程图">
    </a>
</body>
```
#### 5.10 列表：
##### ul：
```html
每行前面显示黑点的列表
<body>
    <ul>
        <li>wa</li>
        <li>ng</li>
        <li>zo</li>
        <li>yu</li>
    </ul>
</body>
```
##### ol：
```html
每行前面显示数字列表
<body>
    <ol>
        <li>wa</li>
        <li>ng</li>
        <li>zo</li>
        <li>yu</li>
    </ol>
</body>

```
##### dl：
```html
<body>
    <dl>
        <dt>wang</dt>
        <dd>zong</dd>
        <dd>yu</dd>
        <dt>wang</dt>
        <dd>zong</dd>
        <dd>yu</dd>
    </dl>
</body>
```
#### 5.11 表格：
```html
<body>
    <table border="1">
        <thead>
        <tr>
            <th>表头1</th>
            <th>表头1</th>
            <th>表头1</th>
            <th>表头1</th>
        </tr>
        </thead>
        <tbody>
        <tr>
            <td>1</td>
            <td>1</td>
            <td>1</td>
            <td>1</td>
        </tr>
        </tbody>
    </table>
</body>
```
```html
合并单元格(左右合并)

 <tr>
	<td>1</td>
	<td colspan="2">1</td>
	<td>1</td>
</tr>
```
```html
合并单元格(上下合并)

<body>
    <table border="1">
        <thead>
        <tr>
            <th>表头1</th>
            <th>表头1</th>
            <th>表头1</th>
            <th>表头1</th>
        </tr>
        </thead>
        <tbody>
        <tr>
            <td>1</td>
            <td>1</td>
            <td>1</td>
            <td>1</td>
        </tr>
         <tr>
            <td rowspan="2">1</td>
            <td>1</td>
            <td>1</td>
             <td>1</td>
        </tr>
         <tr>
            <td>1</td>
            <td>1</td>
            <td>1</td>
        </tr>
        </tbody>
    </table>
</body>
```

#### 5.12 label标签：
`用于点击文件，使得关联的标签获得光标`
```html
点"用户名"或者点击输入框都是进入输入模式

<body>
    <label for="username">用户名：</label>
    <input id="username" type="text" name="user">
</body>
```
#### 5.13 fieldset标签：
```html
<body>
    <fieldset>
        <legend>登陆</legend>
        <label for="username">用户名：</label>
        <input id="username" type="text" name="user" />
        <br />
        <label for="pwd">密码：</label>
        <input id="pwd" type="text" name="user" />
    </fieldset>
</body>
```

## CSS
[RGB颜色查询对照表](http://www.114la.com/other/rgb.htm)  `也可以通过某个网站的代码查看使用的颜色值`
`CSS的注释使用 /*  */`

### 1、在标签上设置style属性：
```html
<body>
    <div style="background-color: #2459a2;height: 44px;">wang</div>
</body>
```
### 2、写在head里面(style标签)：
#### 2.1 id选择器
```html
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        #i1{
            background-color: #2459a2;
            height: 44px
        }
    </style>
</head>
<body>
    <div id="i1">wang</div>
</body>

id不可以重复
```
#### 2.2 class选择器
```html
class可以重复

<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .c1{
            background-color: #2459a2;
            height: 44px
        }
    </style>
</head>
<body>
    <div class="c1">wang</div>
    <span class="c1">zong</span>
    <div class="c1">yu</div>
</body>
```

#### 2.3 标签选择器(所有的div设置上此样式)：
```html
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        div{
            background-color: blue;
            color: white;
        }
    </style>
</head>
<body>
    <div class="c1">wang</div>
    <span class="c1">zong
        <div>yu</div>
    </span>
</body>
```

#### 2.4 层级选择器(如：只应用于span里面的div)(用空格分隔)
```html
.c1 .c2 div{

}
```
```html
方式1：

<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .c2{
            background-color: blue;
            color: white;
        }
    </style>
</head>
<body>
    <div class="c1">wang</div>
    <span class="c1">zong
        <div class="c2">yu</div>
    ZONG</span>
    <div class="c1">WANGZONGYU</div>
</body>

方式2：
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        span div{
            background-color: blue;
            color: white;
        }
    </style>
</head>
<body>
    <div class="c1">wang</div>
    <span class="c1">zong
        <div class="c2">yu</div>
    ZONG</span>
    <div class="c1">WANGZONGYU</div>
</body>
```

#### 2.5 组合选择器(用逗号分隔)：

```html
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        #i1,#i2,#i3{
            background-color: blue;
            color: white;
        }
    </style>
</head>
<body>
    <div id="i1">wang</div>
    <div id="i2">zong</div>
    <div id="i3">yu</div>
</body>
```
```html
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .i1,.i2,.i3{
            background-color: blue;
            color: white;
        }
    </style>
</head>
<body>
    <div class="i1">wang</div>
    <div class="i2">zong</div>
    <div class="i3">yu</div>
</body>
```

#### 2.6 属性选择器(根据关键字匹配)：

```html
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        input[type='text']{width: 100px;height: 200px}
    </style>
</head>
<body>
    <input type="text">
    <input type="password">
</body>

head标签中设置了type='text',就会匹配body标签中的type='text'行
```
```html
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        input[name='wangzy']{width: 100px;height: 200px}
    </style>
</head>
<body>
    <input type="text" name="wangzy">
    <input type="password">
</body>
```

```html
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .c1[name='babyshen']{width: 100px;height: 200px}
    </style>
</head>
<body>
    <div class="c1">wang</div>
    <div class="c1">zong</div>
    <div class="c1">yu</div>
    <input class="c1" type="text" name="babyshen"> 只有这段会被设置
    <input class="c1" type="password">
</body>

```

- 优先级：也就是越靠下越优先
```html
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .c1{
            background-color: red;
            color: white;
        }
        .c2{
            font-size: 28px;
            color: black;
        }
    </style>
</head>
<body>
    <div class="c1 c2" style="color: pink;">BabyShen</div>
</body>
```

### 3、css样式也可以写在单独的文件中：
```html
.c1{
    background-color: red;
    color: white;
    }
.c2{
    font-size: 28px;
    color: black;
    }
```
```html
<head>
    <meta charset="UTF-8">
    <title></title>
    <link rel="stylesheet" href="commons.css">
</head>
<body>
    <div class="c1 c2" style="color: pink">BabyShen</div>
</body>
```
`添加路径:`
```html
<link rel="stylesheet" href="css/commons.css">
```

### 4、边框：
```html
实体的：

<body>
    <div style="border: 1px solid red;">BabyShen</div>
</body>
```
```html
虚体的：

<body>
    <div style="border: 1px dotted red;">BabyShen</div>
</body>
```
### 5、style属性：
```ruby
height        		高度 百分比
width          		宽度 像素，百分比
text-align:ceter		水平方向居中
line-height			垂直方向根据标签高度
color     			字体颜色
font-size 			字体大小
font-weight 			字体加粗
```
```html
<body>
    <div style="border: 1px dotted red;">BabyShen</div>
    <div style="height: 48px;width: 200px;border: 1px solid red;">WangZY</div>
    <div style="height: 48px;width: 80%;border: 1px solid red;">WangZY</div>
</body>

高度永远没有完，所以高度不能设置80%  宽度可以设置
```

```html
<body>
    <div style="height: 48px;
        width: 80%;
        border: 1px solid red;
        font-size: 16px;
        text-align: center;
        line-height: 48px;">WangZY</div>
</body>
```

### 6、float:
```ruby
让标签飘起来，块级标签也可以堆叠
父样式管不住：
<div style="clear: both;"></div>
```
- 例：左侧占20%  右侧占80%
```html
<body>
    <div style="width: 20%;background-color: red;float: left;">Baby</div>
    <div style="width: 80%;background-color: black;float: left">Shen</div>
</body>
```

```html
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .pg-header{
            height: 38px;
            background-color: #dddddd;
            line-height: 38px;   /* 为了使字体在框的中间*/
        }
    </style>
</head>
<body style="margin: 0 auto;">
    <div class="pg-header">
        <div style="float: left">收藏本站</div>
        <div style="float: right">
            <a>登陆</a>
            <a>注册</a>
        </div>
    </div>
    <div style="width: 300px;border: 1px solid red;">
        <div style="width: 96px;height: 30px;border: 1px solid green;float: left;"></div>
        <div style="width: 96px;height: 30px;border: 1px solid green;float: left;"></div>
        <div style="clear: both;"></div>    /* 为了使红框四周都圈住 */
    </div>
</body>
```

### 7、display(行内和块的转换)
```html
display: none; -- 让标签消失
display: inline;
display: block;
display: inline-block;
         具有inline,默认自己有多少占多少
         具有block，可以设置无法设置高度，宽度，padding  margin
```
```html
通过display可实现转化

<body>
    <div style="background-color: red;display:inline;">Baby</div>
    <span style="background-color: red;display: block;">Shen</span>
</body>
```

`行内标签：无法设置高度，宽度，padding  margin`
`块级标签：设置高度，宽度，padding  margin`

```html

<body>
    <span style="background-color: red;height: 50px;width: 70px;">Baby</span>
    <a style="background-color: red;">Shen</a>
</body>
```

```html
<body>
    <span style="display:inline-block;background-color: red;height: 50px;width: 70px;">Baby</span>
</body>
```
### 8、padding  margin(0,auto)