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
`图片在IE浏览器中打开会默认加上了蓝色边框，字体也会变为蓝色，设置一个border 0即可`
```html
默认img标签，有一个1px的边框

<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        img{
            border: 0;
        }
    </style>
</head>
<body>
    <a href="http://www.baidu.com"></a>
    <a href="http://www.baidu.com">
        <img src="1.JPG" style="width: 500px;height: 200px">
    </a>
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

css重用：
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .c{
            共有
        }
        .c1{
            独有
        }
        .c2{
            独有
        }
    </style>
</head>
<body>
    <div class="c c1"></div>
    <div class="c c2"></div>
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
`防止页面随着浏览器大小变化页面变形：`
`在外面设置一个绝对的宽度  在里面内容使用百分比  就不会随着页面的变化 排版有变化了`
```html
<body>
    <div class="pg-header">
        <div style="width: 980px">
            头部数据
        </div>
    </div>

    <div class="pg-body">
        <div style="width: 980px">
            中间内容
        </div>
    </div>

    <div class="pg-footer">
        <div style="width: 980px">
            底部菜单
        </div>
    </div>
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

![](https://github.com/ZongYuWang/image/blob/master/python-margin-padding1.png)

#### 8.1 margin(外边距)
```ruby
margin-top:30px 距上30像素

margin(0,auto)
margin-top:0px;
margin-right:auto;
margin-bottom:0px;
margin-left:auto;

```
```html
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .pg-header{
            height: 38px;
            background-color: #dddddd;
            line-height: 38px;  
        }
    </style>
</head>
<body style="margin: 0 auto;">
    <div class="pg-header">
        <div style="width: 980px;margin: 0 auto">
            <div style="float: left">收藏本站</div>
            <div style="float: right">
            <a>登陆</a>
            <a>注册</a>
        </div>
        <div style="clear: both"></div>   
        </div>
    </div>
</body>
```
#### 8.2 padding(内边距)
`如果使用了padding就表示内容相对于外面边框的上下左右距离`

### 9、position
#### 9.1 fiexd(固定在页面的某个位置)
返回顶部按钮：
```html
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>

        .icon{
            width: 65px;
            height: 50px;
            background-color: black;
            color: white;
            position: fixed;
            bottom: 20px;
            right: 20px;
        }
    </style>
</head>
<body>

    <div onclick="GoTop();" class="icon">返回顶部</div>
    <div style="height:5000px;background-color: #dddddd"></div>

    <script>
        function GoTop(){
            document.body.scrollTop = 0;
        }
    </script>

</body>
```
顶部标题栏：
```html
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .pg-header{
            height: 48px;
            background-color: black;
            color: white;
            position: fixed;
            top: 0;
            right: 0;
            left: 0;
        }

        .pg-body{
            background-color: #dddddd;
            height: 5000px;
            margin-top: 48px;  #为了标题栏不会把正文的文字覆盖，与顶部留出标题栏的大小
        }
    </style>
</head>
<body>
    <div class="pg-header">头部</div>
    <div class="pg-body">内容</div>
</body>
```

#### 9.2 relative+absolute
```html
<div style='position:relative;'>
     <div style='position:absolute;top:0;left:0;'></div>
</div>
```
在框中写一个按钮：
```html
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .pg-body{
            width: 500px;
            height: 200px;
            border: 1px solid red;
            margin: 0 auto;
        }
        .pg-button{
            width: 50px;
            height: 50px;
            background-color: black;
        }
    </style>
</head>
<body>
    <div style="position: relative" class="pg-body">
        <div style="position: absolute;left: 0;bottom: 0" class="pg-button" ></div>
    </div>
</body>
```
### 10、opcity(透明度)、z-index(层级顺序)
`z-index:9这个值谁的大，哪个页面就在上面；也就是层级顺序`
`opcity:0.5 是透明度`
```html
display: none 默认页面是隐藏的，可以通过设置这个让页面显现和隐藏

<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .content-page{
            height: 5000px;
            background-color: white;
        }

        .second-page{
            /*display:none;*/
            z-index:9;
            position: fixed;
            background-color: #999999;
            top:0;
            bottom: 0;
            right: 0;
            left: 0;
            opacity: 0.5;
        }

        .pop-page{
            /*display: none;*/
            z-index: 10px;
            position: fixed;
            top: 50%;
            left: 50%;
            margin-left: -250px;
            margin-top: -200px;
            background-color: #2459a2;
            height: 400px;
            width: 500px;
        }
    </style>
</head>
<body>
    <div class="pop-page">
        <input type="text" />
        <input type="text" />
        <input type="text" />
    </div>
    <div class="second-page"></div>
    <div class="content-page">
        BabyShen
    </div>
</body>
```
### 11、overflow
#### 11.1 auto
`图片的大小超过了设定的值，就会自动生成下拉框`
```html
<body>
    <div style="height: 200px;width: 300px;overflow: auto">
        <img src="1.JPG">
    </div>
</body>
```
#### 11.2 hidden
`图片的大小超过了设定的值，超出部分就会全部隐藏掉`
```html
<body>
    <div style="height: 200px;width: 300px;overflow: hidden">
        <img src="1.JPG">
    </div>
</body>
```

### 12、hover(鼠标移到到此就会变色)
```html
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .pg-header{
            height: 44px;
            background-color: #2459a2;
            color: white;
            position: fixed;
            top: 0;
            right: 0;
            left: 0;
        }
        .pg-body{
            margin-top: 44px;
        }

        .w{
            width: 980px;
            margin: 0 auto; # 整个这个980px的框会在中间，是框，不是框里面的文字
        }

        .pg-header .menu{
            display: inline-block;
            padding: 10px;  # 上下左右全留10像素
            color: white;
        }
        .pg-header .menu:hover{
            background-color: blue;
        }
    </style>
</head>
<body>
    <div class="pg-header">
        <div class="w">
            <a>LOGO</a>
            <a class="menu">全部</a>
            <a class="menu">42区</a>
            <a class="menu">段子</a>
            <a class="menu">1024</a>
        </div>
    </div>
    <div class="pg-body">
        <div class="w">BabyShen</div>
    </div>
</body>
```

### 13、background
#### 13.1 background-image
```html
默认background会根据设置的大小复制图片

<body>
    <div style="background-image: url(icon_18_118.png);height: 500px;width: 500px"></div>
</body>
```
#### 13.2 background-repeat: no-repeat
```html
background-repeat: no-repeat 不会复制图片

<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .img{
            height: 500px;
            width: 1190px;
            background-repeat: no-repeat;
        }
    </style>
</head>
<body>
    <div class="img" style="background-image: url(bg.png);"></div>
</body>
```
#### 13.3  background-repeat: repeat-x/y
```html
background-repeat: repeat-x 图片横向复制
background-repeat: repeat-y 图片纵向复制

<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .img{
            height: 5000px;
            width: 1190px;
            background-repeat: repeat-y;
        }
    </style>
</head>
<body>
    <div class="img" style="background-image: url(bg.png);"></div>
</body>
```

#### 13.4 截取图片中的某一个图标
```html
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .img{
            background-repeat: no-repeat;
            height: 80px;
            width: 20px;
            border: 1px solid red;
        }
    </style>
</head>
<body>
    <div class="img" style="background-image: url(icon_18_118.png);"></div>
</body>

这时候的再显示图片的其他位置，不能移动div，div的位置是固定的，只能移动图片位置
```

#### 13.5 background-position-x/y
```html
background-position-x/y分别表示x轴和y轴
background-position：10px 10px；也可以直接这么写

<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .img{
            background-repeat: no-repeat;
            height: 20px;
            width: 20px;
            border: 1px solid red;
            background-position-x: 0px;
            background-position-y: 0px;
        }
    </style>
</head>
<body>
    <div class="img" style="background-image: url(icon_18_118.png);"></div>
</body>
```

- 用户输入框添加图片
```html
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .user-button{
            height: 48px;
            width: 980px;
            margin: 0 auto;
            line-height: 48px;
            padding-left: 500px;

        }
        .user-button .input{
            height: 30px;
            width: 400px;
        }

        .user-button .img{
            height: 16px;
            width: 16px;
            right: 500px;
            top: 18px;
            display: inline-block;

        }
    </style>

</head>
<body>
    <div class="user-button" style="position: relative" >
        <span>*用户名</span>
        <input class="input" type="text" style="padding-right:30px ">
        <span style="background-image: url(i_name.jpg);position: absolute" class="img" ></span>
    </div>

</body>

```

## JavaScript:
### 1. JavaScript代码存在形式：
```ruby

JavaScript代码存在形式：
        - Head中
                <script>
                    //javascript代码
                    alert(123);
                </script>
                
                <script type="text/javascript">
                    //javascript代码
                    alert(123);
                </script>
        - 文件
            <script src='js文件路径'> </script>
            
        PS: JS代码需要放置在 <body>标签内部的最下方
```
```js
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        /**css代码*/
    </style>
    
    <script>
//        javascript代码
    </script>
</head>
```
```html
默认就是type="text/javascript",不要修改text成其他的

    <script type="text/javascript">
//        javascript代码
    </script>
```
### 2、注释：
```ruby
当行注释  //
多行注释  /*     */
```

### 3、变量：
```ruby
name = 'alex'     # 全局变量
var name = 'eric' # 局部变量
```
```js
<head>
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
    <h1>BabyShen</h1>
    <script>
        name = "Wang";
        function func(){
            var name = "Zong"
        }
    </script>
</body>
```
### 4、基本数据类型：
#### 4.1 数字
```ruby
a = 18;
```
#### 4.2 字符串
```ruby
a = "alex"
a.chartAt(索引位置)
a.substring(起始位置，结束位置)
a.lenght    获取当前字符串长度
...
```
```js
a = "BabyShen"
"BabyShen"
a.charAt(0)
"B"
a.charAt(1)
"a"
a.charAt(2)
"b"

a.substring(1,3)
"ab"
a.substring(1,4)
"aby"

a.length
8

字符串拼接：
a = "Baby"
"Baby"
a.concat("Shen")
"BabyShen"

查找子序列的位置：
a = "BabyShen"
"BabyShen"
a.indexOf("Sh")
4

切片分隔：
a = "wangzongyu"
"wangzongyu"
a.split("n")
(3) ["wa", "gzo", "gyu"]
```
#### 4.3 列表(在JS中叫数组)
```js
a = [11,22,33]

a = [11,22,33,44]
(4) [11, 22, 33, 44]
a.length
4
a.splice(1,1,99)  //第一个1表示起始位置，第二个1表示删除1个值，插入99
[22]
a
(4) [11, 99, 33, 44]
```

#### 4.4 字典
```js
a = {'k1':'v1','k2':'v2'}
a["k1"]
"v1"
```

#### 4.5 布尔类型
```js
a = true
true
```

### 5、函数：
```js
<body>

    <script>
        function f1(){
            console.log(1)
        }
//        setInterval("alert(WangZY);",5000);
          setInterval("f1()",5000);
    </script>
</body>
```
- 字符串的拼接+流动演示实验：
```html
<body>
    <div id="id1">汪宗宇你好</div>
</body>
```
```js
document.getElementById('id1')
<div id=​"id1">​汪宗宇你好​</div>​
tag = document.getElementById('id1')
<div id=​"id1">​汪宗宇你好​</div>​
tag.innerText
"汪宗宇你好"
content = tag.innerText
"汪宗宇你好"
content
"汪宗宇你好"
f= content.charAt(0)
"汪"
l = content.substring(1,content.length)
"宗宇你好"
new_content = l + f
"宗宇你好汪"
```
`代码整合`
```js
<body>
    <div id="id1">汪宗宇你好</div>

    <script>
        function func(){
            // 根据ID获取指定标签的内容，定义局部变量接收
            var tag = document.getElementById('id1')
            // 获取标签内部的内容
            var content = tag.innerText

            var f = content.charAt(0);
            var l = content.substring(1,content.length);

            var new_content = l + f;  //字符串拼接
            tag.innerText = new_content  //将改造后的值再重新赋值给tag.innertext
        }
        setInterval('func()',1000)
    </script>
</body>
```

### 6、for循环(JS有两种循环方式)
```js
方式一：

循环数组：

a = [11,22,33,44]
        for(var item in a){
            console.log(item);
        }

// item就是a里面的每个元素的索引(坐标)

 a = [11,22,33,44]
        for(var item in a){
            console.log(a[item]);
        }
11
22
33
44
// 循环时，循环的元素就是索引


循环字典：
 a = {'k1':'v1','k2':'v2'}
            for(var item in a){
                console.log(item);
        }
k1
k2

a = {'k1':'v1','k2':'v2'}
            for(var item in a){
                console.log(a[item]);
        }
v1
v2


方式二：

循环数组：
a = [11,22,33,44]
       for(var i=0;i<a.length;i++){   // 方式二不能循环字典

        }

```

### 7、条件语句：
```js
 if(条件){
        
        }else if(条件){
        
        }else if(条件){
        
        }else{
            
         }
         
== 		值相等
===  	值和类型都要相等
&&  	和
|| 		或


if(1==1){
        
        }
        
if(1!=1){
        
        }
        
if(1===1){
        
        }
        
if(1!==1){
        
        }
        
if(1==1 || 2==2)
```

### 8、Dom(其实就是Document)
#### 8.1 Dom直接选择器：
`想让页面'动'起来，需要先找到标签，然后再操作标签`
```html
html文件内容如下：

<body>
    <div id="i1">BabyShen</div>
    <a>wang</a>
    <a>zong</a>
    <a>yu</a>
</body>
```
```js
在浏览器中打开上面的html文件，并在开发者工具的Console中输入以下内容，查看浏览器显示内容的变化值

document.getElementById('i1')  // 获取单个元素
<div id=​"i1">​BabyShen​</div>​
document.getElementById('i1').innerText  //获取标签中的文本内容
"BabyShen"
document.getElementById('i1').innerText = "new_BabyShen"  //对标签内部文本进行重新赋值
"new_BabyShen"   


document.getElementsByTagName('a')[0]  // 获取多个元素(列表)
<a>​wang​</a>​
document.getElementsByTagName('a')[1]
<a>​zong​</a>​
document.getElementsByTagName('a')[2]
<a>​yu​</a>​
document.getElementsByTagName('a')[1].innerText = 'ZONG';
"ZONG"
tags = document.getElementsByTagName('a');
for(var i=0;i<tags.length;i++){
	tags[i].innerText='SHEN'
	}

// document.getElementsByClassName 根据class属性获取标签集合
// document.getElementsByName  根据name属性获取标签集合
```
#### 8.2 Dom间接选择器：
`先找到某个标签，再通过这个标签去找其他的标签`
```html
html内容如下：

<body>
    <div>
        <div>brother:Wang</div>
        <div>wang</div>
    </div>
    <div>
        <div>brother:Zong</div>
        <div id="2">zong</div>
    </div>
    <div>
        <div>brother:Yu</div>
        <div>yu</div>
    </div>
</body>
```
```js
tag = document.getElementById('2')
<div id=​"2">​zong​</div>

tag
<div id=​"2">​zong​</div>​
​
tag.parentElement   // 找到父亲标签
<div>​
	<div>​brother:Zong​</div>
	​<div id=​"2">​zong​</div>
</div>​
tag.parentElement.children  // 找到父亲标签的孩子标签

tag.parentElement.previousElementSibling  // 通过父亲标签，找到父亲的兄弟标签
<div>​
	<div>​brother:Wang​</div>​
	<div>​wang​</div>​
</div>​
```
- 简单操作：
```js
tag.className = 'c1';
"c1"
tag
<div id=​"2" class=​"c1">​zong​</div>​

tag.className = 'c2';
"c2"
tag
<div id=​"2" class=​"c2">​zong​</div>​


tag.classList
["c2"]  // 列表形式
tag.classList.add('c3')  //增加一个c3，就多一个c3
undefined
tag
<div id=​"2" class=​"c2 c3">​zong​</div>

tag.classList.remove('c2')
tag
<div id=​"2" class=​"c3">​zong​</div>​​
```
`<div onclick="func();"></div> 表示一点击，就执行func函数`


- 实验：点击框(添加、全选、取消、反选) 

`效果图:`      
![](https://github.com/ZongYuWang/image/blob/master/python-js1.png)    
```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .hide{   /* 默认遮罩层和弹出框不能显示出来，所以后面调用这个修改这个display值就行*/
            display: none;
        }
        .c1{   /*遮罩层的class*/
            position: fixed;
            left: 0;
            top: 0;
            right: 0;
            bottom: 0;
            background-color: black;
            opacity: 0.6;
            z-index: 9;
        }
        .c2{   /*弹出框的class*/
            width: 500px;
            height: 400px;
            background-color: white;
            position: fixed;
            left: 50%;
            top: 50%;  /* 当前这种情况是框的"左上角"在中间了*/
            margin-left: -250px;
            margin-top: -200px;
            z-index: 10;
        }
    </style>
</head>
<body style="margin: 0;">  <!-- body默认两遍有边距，去掉边距-->
    <div>
        <input type="button" value="添加" onclick="ShowModel();" />
        <!--点击"添加"按钮，应该弹出遮罩层和弹出框，所以设置一个onclick调用后面的函数-->
        <input type="button" value="全选" onclick="ChooseAll();" />
        <input type="button" value="取消" onclick="cancelAll();" />
        <input type="button" value="反选" onclick="ReverseAll();" />

        <table>
            <thead>
                <tr>
                    <th>选择</th>
                    <th>主机名</th>
                    <th>端口</th>
                </tr>
            </thead>
            <tbody id="tb">  <!--需要通过tbody这个标签找到所有的孩子；全选、取消、反选都需要用这个tbody-->
                <tr>
                    <td>
                        <input type="checkbox" />
                    </td>
                    <td>1.1.1.1</td>
                    <td>190</td>
                </tr>
                <tr>
                    <td><input type="checkbox"f id="test" /></td>
                    <td>1.1.1.2</td>
                    <td>192</td>
                </tr>
                <tr>
                    <td><input type="checkbox" /></td>
                    <td>1.1.1.3</td>
                    <td>193</td>
                </tr>
            </tbody>
        </table>
    </div>

    <!-- 遮罩层开始 -->
   <div id="i1" class="c1 hide"></div>   <!-- 都应用hide样式；定义id为了下面js的使用，要修改hide里面的值-->
    <!-- 遮罩层结束 -->

    <!-- 弹出框开始 -->
    <div id="i2" class="c2 hide">  <!-- 都应用hide样式-->
        <p><input type="text" /></p>
        <p><input type="text" /></p>
        <p>
            <input type="button" value="取消" onclick="HideModel();"/>  <!-- 点击弹出框中的取消就应该消失，所以再写一个HideModel方法 -->
            <input type="button" value="确定"/>

        </p>
    </div>
     <!-- 弹出框结束 -->

    <script>
        function ShowModel(){
            document.getElementById('i1').classList.remove('hide');
            document.getElementById('i2').classList.remove('hide');
        }
        function HideModel(){
            document.getElementById('i1').classList.add('hide');
            document.getElementById('i2').classList.add('hide');
        }

        function ChooseAll(){
            var tbody = document.getElementById('tb');
            // 获取所有的tr
            var tr_list = tbody.children;
            for(var i=0;i<tr_list.length;i++){
                // 循环所有的tr，current_tr
                var current_tr = tr_list[i];   //当前循环的tr
                var checkbox = current_tr.children[0].children[0]; //current_tr.children取所有的td，但是需要取第一个td；然后再取td下的第一个孩子
                // 取完的值就是checkbox了，对checkbox打对勾
                checkbox.checked = true;   // 设置为true，打上对勾

            }
        }

        function cancelAll(){
            var tbody = document.getElementById('tb');
            // 获取所有的tr
            var tr_list = tbody.children;
            for(var i=0;i<tr_list.length;i++){
                // 循环所有的tr，current_tr
                var current_tr = tr_list[i];
                var checkbox = current_tr.children[0].children[0];
                checkbox.checked = false;  // 取消和全选的唯一区别就是这地方设置为false就行

            }
        }

        function ReverseAll(){
            var tbody = document.getElementById('tb');
            // 获取所有的tr
            var tr_list = tbody.children;
            for(var i=0;i<tr_list.length;i++){
                // 循环所有的tr，current_tr
                var current_tr = tr_list[i];
                var checkbox = current_tr.children[0].children[0];
                if(checkbox.checked){
                    checkbox.checked = false;
                }else{checkbox.checked = true;
                }
            }
        }
        // 反选就是判断现在假如是true，就设置为false，如果是false，就设置为true
    </script
</body>
</html>


checkbox.checked = true 通过这个值设置勾选与不勾选
document.getElementById('test')
<input type=​"checkbox" f id=​"test">​
document.getElementById('test').checked
false
document.getElementById('test').checked = true;
true
```
- 实验：左侧菜单： 

`效果图:`    
![](https://github.com/ZongYuWang/image/blob/master/python-js2.png)    
```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .hide{
            display: none;
        }
        .item .header{
            height: 35px;
            background-color: #2459a2;
            color: white;
            line-height: 35px;
        }
    </style>
</head>
<body>
    <div style="height: 48px"></div>

    <div style="width: 300px">

        <div class="item">
            <div id='i1' class="header" onclick="ChangeMenu('i1');">菜单1</div>
            <div class="content">
                <div>内容1</div>
                <div>内容1</div>
                <div>内容1</div>
            </div>
        </div>
        <div class="item">
            <div id='i2' class="header" onclick="ChangeMenu('i2');">菜单2</div>
            <div class="content hide">
                <div>内容2</div>
                <div>内容2</div>
                <div>内容2</div>
            </div>
        </div>
        <div class="item">
            <div id='i3' class="header" onclick="ChangeMenu('i3');">菜单3</div>
            <div class="content hide">
                <div>内容3</div>
                <div>内容3</div>
                <div>内容3</div>
            </div>
        </div>
        <div class="item">
            <div id='i4' class="header" onclick="ChangeMenu('i4');">菜单4</div>
            <div class="content hide">
                <div>内容4</div>
                <div>内容4</div>
                <div>内容4</div>
            </div>
        </div>
    </div>

    <script>
        function ChangeMenu(nid){
            var current_header = document.getElementById(nid);  // 接收传过来的值nid，上面的ChangeMenu('i#')中都传递了形参

            var item_list = current_header.parentElement.parentElement.children;

            for(var i=0;i<item_list.length;i++){
                var current_item = item_list[i];
                current_item.children[1].classList.add('hide');  //先都添加上hide，都隐藏
            }

            current_header.nextElementSibling.classList.remove('hide');  // 然后再去掉当前标签的hide
        }
    </script>
</body>
</html>
```