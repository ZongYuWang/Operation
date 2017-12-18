## HTML+CSS
`HTML相当于一个赤裸裸的人站在那，CSS相当于给人穿上华丽的衣服，JS相当于让这个人动起来`

#### 开发后台程序：
- 写html文件(充当模板的作用)
- 数据库获取数据，然后替换到html文件的指定位置(web框架)

#### HTML基础概念：
```html
<!DOCTYPE html>  

#放在html页顶端，表示使用标准的html语法对应关系(当然也有其他的语法对应关系)
```
- html页固定格式：
```html
<!DOCTYPE html>
<html lang="en">
<head>
  ......
</head>
<body>
  .......
</body>
</html>
```
```html
# 这种格式叫做html标签（只能有一个）：
<html>babyshen</html>

# lang="en"，标签内部的属性,可以写也可以不写：
<html lang="en">
```
- 注释代码：
```html
<!--  注释的内容 -->
```

#### 标签分类：
- 自闭合标签
```html
<meta charset="UTF-8">
```

- 主动闭合标签
```html
 <title>babyshen</title>
```
#### head标签：
```html
<meta charset="UTF-8">  # 定义编码
<meta http-equiv="Refresh" content="10">  # 每页每隔10秒自动刷新
<meta http-equiv="Refresh" content="10;Url=http://www.baidu.com"> # 10秒就跳转到后面的网址
<meta name="keywords" content="汽车,汽车之家,汽车网,汽车报价,汽车图片,新闻,评测,社区,俱乐部">  #设置搜索关键字
<meta http-equiv="X-UA-COMPATIBLE" content="IE=IE8;IE=IE7;"> # IE兼容问题，支持IE8、IE7
```
link标签：
```html
 <link rel="shortcut icon" href="image/favicon.ico">
```

### body标签：
#### a标签：
- 跳转
```html
跳转到百度产生一个新页，不会在原网页的基础上打开百度
<body>
    <a href="http://www.baidu.com" target="_blank">百度</a>
</body>
```
- 锚
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

```html
 <a href="http://www.baidu.com">百&nbsp;&nbsp;&nbsp;&nbsp;&lt;a&gt;度</a>  #一个&nbsp表示一个空格
 
<p>wang<br />wangzongyu</p>  #<br就是换行>
<p>zong</p>                #<p>是段落
<p>yu</p>

<h1>wang</h1>
<h6>yu</h6>

<span>hello</span>
<span>hello</span>
<span>hello</span>

<div>wang</div>
<div>zong</div>
<div>yu</div>
```
所有的标签分为：      
块级标签(占据一整行)：div标签(白板)、h系列(加大加粗)、p标签(段落和段落之间有间距)；        
行内标签(内联标签)：span标签(白板)       
`标签也是可以实现嵌套的`
```html
<body>
    <div>
        <div></div>
        <span></span>
    </div>
</body>
```

网页特殊符号HTML代码大全:https://www.cnblogs.com/web-d/archive/2010/04/16/1713298.html

#### input系列：
```ruby
input type='text'      -name属性
input type='password'  -name属性
input type='submit'    -value='提交' 提交按钮，表单
input type='button'    -value='登录' 按钮
```
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

#### 可以多行输入：
```html
<body>
    <textarea name="meno">babyshen</textarea>
</body>
```
`<textarea>默认值</textarea> 有name属性`

#### 搜索：
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
同时也可以嵌套，但是div里面的值不会提交到后台，只是input里面的值可以提交到后台
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

#### 设置选择项:
- 单选框
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

- 复选框
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
#### 默认值设置：
```html
男：<input type="radio" name="gender" value="1" checked="checked">

checked="checked"表示该项是默认值
```
#### 上传文件：
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

#### 重置：
```html
<input type="reset" value="重置" />
```

#### 下拉框：
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

#### img：
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
```html
图片位置或者图片找不到的时候，该处显示"流程图"
<body>
    <a href="http://www.baidu.com">
       <img src="123.png" style="height: 200px;width: 200px;" alt="流程图">
    </a>
</body>
```
```html
鼠标放在图片上会显示"python图片"字样
<body>
    <a href="http://www.baidu.com">
       <img src="image.png" title="python图片" style="height: 200px;width: 200px;" alt="流程图">
    </a>
</body>
```
#### 列表：
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
#### 表格：
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

#### label
`用于点击文件，使得关联的标签获得光标`
```html
点"用户名"或者点击输入框都是进入输入模式

<body>
    <label for="username">用户名：</label>
    <input id="username" type="text" name="user">
</body>
```
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