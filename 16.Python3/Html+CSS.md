### HTML+CSS

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

#### body标签：
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

https://www.cnblogs.com/web-d/archive/2010/04/16/1713298.html