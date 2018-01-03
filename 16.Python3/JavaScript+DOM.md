## Dom(Document Object Model)

### 1、查找元素
#### 1.1 直接查找：
`想让页面'动'起来，需要先找到标签，然后再操作标签`
```ruby
document.getElementById             根据ID获取一个标签
document.getElementsByName          根据name属性获取标签集合
document.getElementsByClassName     根据class属性获取标签集合
document.getElementsByTagName       根据标签名获取标签集合
```
```js
html文件内容如下：
<body>
    <div id="i1">BabyShen</div>
    <a>wang</a>
    <a>zong</a>
    <a>yu</a>
</body>


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
#### 1.2 间接查找：
`先找到某个标签，再通过这个标签去找其他的标签`
```js
parentNode           父节点
childNodes           所有子节点
firstChild           第一个子节点
lastChild            最后一个子节点
nextSibling          下一个兄弟节点
previousSibling      上一个兄弟节点
 
parentElement            父节点标签元素
children                 所有子标签
firstElementChild        第一个子标签元素
lastElementChild         最后一个子标签元素
nextElementtSibling      下一个兄弟标签元素
previousElementSibling   上一个兄弟标签元素
```
```js
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
### 2、操作：
#### 2.1 class操作：
```js
className                获取所有类名
classList.remove(cls)    删除指定类
classList.add(cls)       添加类
```
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
<div id=​"2" class=​"c3">​zong​</div>


obj.style.fontSize = '16px';
//相当于对下面的语句加上一个style
<div class='c1' style='font-size:16px;'></div>​​
```
`<div onclick="func();"></div> 表示一点击，就执行func函数`

#### 2.2 内容文本操作:
```js
innerText   文本
outerText
innerHTML   HTML内容
innerHTML  
value       值
```
```html
<body>
    <div id="i1">
        BabyShen
        <a>Wang</a>
    </div>
</body>
```
innerText 仅文本:
```html
obj = document.getElementById('i1');
<div id=​"i1">​…​</div>​
obj.innerText
"BabyShen Wang"

赋值：
obj.innerText = "ZONG"
"ZONG"    // 这样BabyShen Wang 都会替换成ZONG

obj.innerText = "<a href='http://www.dubai.com'>BabyShen</a>"
"<a href='http://www.dubai.com'>BabyShen</a>"
```
innerHTML 全内容:
```html
obj = document.getElementById('i1');
<div id=​"i1">​…​</div>​
obj.innerHTML
"
        BabyShen
        <a>Wang</a>
    "

赋值：
obj.innerHTML = 'ZONG';
"ZONG"

obj.innerHTML = "<a href='http://www.dubai.com'>BabyShen</a>"
"<a href='http://www.dubai.com'>BabyShen</a>"
```
input value获取当前标签中的值：
```html
<body>
    <input type="text" id="i2">
</body>


obj = document.getElementById('i2')
<input type=​"text" id=​"i2">​
obj.value
""
<!--在搜索框中输入值-->
obj.value
"babyshen"
```
`value select(selectedIndex) 获取选中的value值:`
```html
<body>
    <input type="text" id="i2" />
    <select id="i3">
        <option value="11">北京</option>
        <option value="12">天津</option>
        <option value="13">上海</option>

    </select>
</body>


obj = document.getElementById('i3');
<select id=​"i3">​…​</select>​
obj.value
"11"
obj.value
"13"


obj.selectedIndex
2
obj.selectedIndex
0
obj.selectedIndex = 1
1
```

```html
<body>
    <textarea id="i4"></textarea>
</body>

obj = document.getElementById('i4');
<textarea id=​"i4">​</textarea>​
obj.value
""
<!--在框内输入值-->
obj.value
"Babyshen"
```
- 实验：输入框中默认有提示语言(当鼠标点进去输入框提示语消失，鼠标离开框，提示语出现)
```js
实验雏形：

<body>
   <div style="width: 600px;margin: 0 auto">
       <input onfocus="Focus();" onblur="Blur();" type="text" value="请输入关键字">
   </div>

    <script>
        function Focus(){
            console.log(1);
        }
        function Blur(){
            console.log(2);
        }
    </script>
</body>

// 鼠标点击到框里浏览器console中输出1，点击外面就输出2


功能完善：
<body>
   <div style="width: 600px;margin: 0 auto">
       <input id="i1" onfocus="Focus();" onblur="Blur();" type="text" value="请输入关键字">
   </div>

    <script>
        function Focus(){
            var tag = document.getElementById('i1');
            var val = tag.value;
            if(val == "请输入关键字"){
                tag.value = ''
            }
        }
        function Blur(){
            var tag = document.getElementById('i1');
            var val = tag.value;
            if(val.length == 0){
                tag.value = "请输入关键字";
            }
        }
    </script>
</body>


当然也可以使用下面的语法实现：
<body>
    <input type="text" placeholder="请输入关键字" />
</body>
```

#### 2.3 属性操作：
```js
attributes                获取所有标签属性
setAttribute(key,value)   设置标签属性
getAttribute(key)         获取指定标签属性
removeAttribute
```
	
```html
obj = document.getElementById('i1')
<input id=​"i1" onfocus=​"Focus()​;​" onblur=​"Blur()​;​" type=​"text" value=​"请输入关键字">​
obj.setAttribute('xxx','Babyshen');
undefined
obj
<input id=​"i1" onfocus=​"Focus()​;​" onblur=​"Blur()​;​" type=​"text" value=​"请输入关键字" xxx=​"Babyshen">​
obj.removeAttribute('value');
undefined
obj
<input id=​"i1" onfocus=​"Focus()​;​" onblur=​"Blur()​;​" type=​"text" xxx=​"Babyshen">​
obj.attributes
NamedNodeMap {0: id, 1: onfocus, 2: onblur, 3: type, 4: xxx, id: id, onfocus: onfocus, onblur: onblur, type: type, xxx: xxx, …}
```

#### 2.4 标签操作：
##### 2.4.1 创建标签：
```js
方式一：
var tag = document.createElement('input');
tag.innerText = "BabyShen";
tag.className = 'c1';
tag.style.color = 'red';

方式二：
var tag = "<a class='c1' href='http://www.baidu.com'>BabyShen</a>"
```
##### 2.4.2 操作标签：
```js
方式一：
var tag = "<p></p><input type='text' /></p>";  // p标签包起来，这样添加就一行一行添加
document.getElementById('i1').insertAdjacentHTML("beforeEnd",tag);
// 注意：第一个参数只能是'beforeBegin'、‘afterBegin’、'beforeEnd'、'afterEnd'

方式二：
var p = document.createElement('p');
p.appendChild(tag);
document.getElementById('i1').appendChild(p);

```
- 示例应用：
```js

<body>
    <input type="button" onclick="addEle();" value="addEle"/>
    <input type="button" onclick="new_addEle();" value="new_addEle"/>
    <div id='i1'>
        <p><input type="text"></p>
        <hr />  <!-- hr是分割线-->
    </div>
    // 可以添加一个孩子位置在最上面，也可以添加一个孩子位置在最后；也可以添加到上面的兄弟位置，也可以添加到下方的兄弟位置；
    // 'beforeBegin'、‘afterBegin’、'beforeEnd'、'afterEnd'；
    <script>
        function addEle(){
            // 创建一个标签
            // 将标签添加到i1里面
            var tag = "<p></p><input type='text' /></p>";  // p标签包起来，这样添加就一行一行添加
            document.getElementById('i1').insertAdjacentHTML("beforeEnd",tag);
            // 注意：第一个参数只能是'beforeBegin'、‘afterBegin’、'beforeEnd'、'afterEnd'
        }

        function new_addEle(){
            var tag = document.createElement('input');
            tag.setAttribute('type','text');
            tag.style.fontSize = '16px';
            tag.style.color = 'red';

            // 假如也想跟上面一样将input添加到p标签里
            var p = document.createElement('p');
            p.appendChild(tag);
            document.getElementById('i1').appendChild(p);
        }
    </script>
</body>


使用a标签(任何标签通过DOM都可提交表单)：
<body>
    <form id="f1" action="http://www.baidu.com">
        <a onclick="submitForm();">提交</a>
    </form>

    <script>
        function submitForm(){
            document.getElementById('f1').submit()
        }
    </script>
</body>
```
#### 2.5 样式操作：
```js
var obj = document.getElementById('i1')
obj.style.fontSize = "32px";
obj.style.backgroundColor = "red";
```
#### 2.6 提交表单：
```js
document.getElementById('f1').submit()
```
#### 2.7 其他操作：
##### URL和刷新：
```js
location.reload()  //页面刷新
location.href  = location.reload()
```
```html

<body>
    <script>
        var c = confirm('真的要删除吗？');
        console.log(c);
    </script>
</body>

// 选择确认删除就会返回true

location.href = 'http://baidu.com'; //会直接跳转百度
```

##### 定时器
```js
setInterval    定时器一直执行(多次定时器)
clearInterval  清除多次定时器

setTimeout     单次定时器
clearTimeout   清除单次定时器
```


```js
var o1 = setInterval(function(){},5000)
clearInterval(o1)
```
```js
var o2 = setTimeout(function(){},5000);
clearInterval(o2)
```
```html
<body>
    <script>
        var obj = setInterval(function(){
            console.log(1);
            clearInterval(obj);
        },1000)
    </script>
</body>
```
- 示例：停止定时器功能作用：
```js
<body>
    <div id="status"></div>
    <input type="button" value="删除" onclick="deleteEle();" />

    <script>
        function deleteEle(){
            document.getElementById('status').innerText='已删除';
            setTimeout(function(){
                document.getElementById('status').innerText='';
            },5000);
        }
    </script>
</body>
```

- 综合实验1：点击框(添加、全选、取消、反选) 

`效果图:`      
![](https://github.com/ZongYuWang/image/blob/master/python-js1.png)    
```ruby
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
- 综合实验2：左侧菜单： 

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


### 3、事件操作：
绑定事件的两种方式：
- 直接标签绑定 onclick='xxx()'  onfocus
- 先获取Dom对象，然后进行绑定
	- document.getElementById('xx').onclick
	- document.getElementById('xx').onfocus
- this：当前触发事件的标签(谁调用这个函数，this就指向谁)
	  
#### 3.1 第一种绑定方式（直接写在标签中）：
```js
<input id="i1" type="button" onclick="clickOn(this)">

function clickOn(self){
	// self 当前点击的标签
            
     }
     
     
左侧菜单实验使用this语法改版：
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
            <div class="header" onclick="ChangeMenu(this);">菜单1</div>
            <div class="content">
                <div>内容1</div>
                <div>内容1</div>
                <div>内容1</div>
            </div>
        </div>
        <div class="item">
            <div class="header" onclick="ChangeMenu(this);">菜单2</div>
            <div class="content hide">
                <div>内容2</div>
                <div>内容2</div>
                <div>内容2</div>
            </div>
        </div>
        <div class="item">
            <div class="header" onclick="ChangeMenu(this);">菜单3</div>
            <div class="content hide">
                <div>内容3</div>
                <div>内容3</div>
                <div>内容3</div>
            </div>
        </div>
        <div class="item">
            <div class="header" onclick="ChangeMenu(this);">菜单4</div>
            <div class="content hide">
                <div>内容4</div>
                <div>内容4</div>
                <div>内容4</div>
            </div>
        </div>
    </div>

    <script>
        function ChangeMenu(ths){
            current_header = ths;
            var item_list = current_header.parentElement.parentElement.children;

            for(var i=0;i<item_list.length;i++){
                var current_item = item_list[i];
                current_item.children[1].classList.add('hide');  //先都添加上hide，都隐藏
            }

            current_header.nextElementSibling.classList.remove('hide');  // 然后再去掉当前标签的hide
        }
    </script>
</body>
```

#### 3.2 第二种绑定方式（单独写一个匿名函数）：
```js
<input id="i1" type="button" >
document.getElementById('i1').onclick = function(){
		// this 代指当前点击的标签
            
        }
```
```js
鼠标点进表格中就变色，鼠标移除表格颜色消失

<body>
    <table border="1" width="300px">
        <tr><td>1</td><td>2</td><td>3</td></tr>
        <tr><td>1</td><td>2</td><td>3</td></tr>
        <tr><td>1</td><td>2</td><td>3</td></tr>
    </table>
    <script>

        var myTrs = document.getElementsByTagName("tr");
        var len = myTrs.length;
        for(var i=0;i<len;i++){
            myTrs[i].onmouseover = function(){
                this.style.backgroundColor = "red";
            }

            myTrs[i].onmouseout = function(){
                this.style.backgroundColor = "";
            }
        }
    </script>
</body>
```


#### 3.3 第三种绑定方式

`addEventListener中的第三个参数设置为true表示捕捉，设置为false或者不填写表示冒泡`
![](https://github.com/ZongYuWang/image/blob/master/python-event1.png)    
`红色是捕捉，绿色是冒泡`

```ruby
<head>
    <meta charset="UTF-8">
    <title></title>

    <style>
        #main{
            background-color: red;
            width: 300px;
            height: 400px;
        }

        #content{
            background-color: pink;
            width: 150px;
            height: 200px;
        }
    </style>
</head>
<body>
    <div id="main">
        <div id="content"></div>
    </div>
    <script>
        var mymain = document.getElementById("main");
        var mycontent = document.getElementById("content");
        mymain.addEventListener("click",function(){console.log("main")},true);
        mycontent.addEventListener("click",function(){console.log("content")},true);
    </script>

</body>
```


### 4、词法分析
`词法分析不涉及执行阶段`
```js
 <script>
        function t1(age){
            console.log(age);
            var age = 27;
            console.log(age);
            function age(){}
            console.log(age);
        }

        t1(3);
    </script>

// 输出：
ƒ age(){}
27
27

active object ——> AO
a.形式参数
	AO.age = undefined
	AO.age = 3;
b.函数内局部变量(如果已经有值不改变，没有值设置为undefined)
	AO.age = undefined
c.函数声明表达式
	AO.age = function()
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
#### 4.3 数组
`JavaScript中的数组类似于Python中的列表`
```js
obj.length          数组的大小
 
obj.push(ele)       尾部追加元素
obj.pop()           尾部获取一个元素
obj.unshift(ele)    头部插入元素
obj.shift()         头部移除元素
obj.splice(start, deleteCount, value, ...)  插入、删除或替换数组的元素
                    obj.splice(n,0,val) 指定位置插入元素
                    obj.splice(n,1,val) 指定位置替换元素
                    obj.splice(n,1)     指定位置删除元素
obj.slice( )        切片
obj.reverse( )      反转
obj.join(sep)       将数组元素连接起来以构建一个字符串
obj.concat(val,..)  连接数组
obj.sort( )         对数组元素进行排序
```
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
`布尔类型仅包含真假，与Python不同的是其首字母小写`
```js
==       比较值相等
!=       不等于
===      比较值和类型相等
!===     不等于
||       或
&&       且

a = true
true
```

### 5、函数：
#### 5.1 普通函数：
```js
function func(){
            
        }
```
#### 5.2 匿名函数：
```js
普通函数写法：
 function func(arg){
            return arg+1
        }
        setInterval("func()",5000);
        

匿名函数写法(没有函数名的函数叫匿名函数)：
setInterval(function(){
            console.log(123);
        },5000)
```
#### 5.3 自执行函数：
```js
自执行函数就是创建函数并且自动执行

 (function(arg){
           console.log(arg);
       })(1)
```

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
- 示例：字符串的拼接+流动演示实验：
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

`使用带返回值的函数：`
```html
<body>
    <script>
        function func(arg){
            return arg+1
        }
        var result = func(1)
        console.log(result);
    </script>
</body>
```


#### 5.4 面向对象：
```js
this代指对象(也就是python中的self)
创建对象时使用 new 函数名()

function foo(){
            this.name = "n"
        }
        
        var obj = new foo('Baby')；
        obj.name  // 此时的obj.name 就是Baby了
```

```js
function Foo(n){
    this.name = n;
    this.sayName = function(){
        console.log(this.name);
    }
}

var obj1 = new Foo('Baby');
obj1.name
obj1.sayName()

var obj2 = new Foo('Shen')
obj2.name
obj2.sayName()

// 上面的代码会存在"方法复用"的问题，obj1拿到name='Baby'，然后会调用sayName function(){this.name;}；
// 同时obj2拿到name='Shen'，同样也会调用sayName function(){this.name;}；


Python中定义类和对象的方式：
class Foo:
    def __init__(self,name):
        self.name = name
    
    def sayName(self):
        print(self.name)

obj1 = Foo('Baby')
obj2 = Foo('Shen')

//python中就会把ayName(self):print(self.name)单独存放，后面的多个对象只需要单独调用就可以了
```
#### 5.5 原型:
```js
function Foo(n){
            this.name = n;
        }

        Foo.prototype = {
            'sayName':function(){
                console.log(this.name)
            }
        }

        obj1 = new Foo('Baby')
        obj1.sayName()

        obj2 = new Foo('Shen')
        obj2.sayName()

// 通过obj1先找到Foo这个类，然后去类的原型中照sayName，如果有就执行;
原型可以理解成字典，key是方法名，value就是值;
```



### 6、语句
#### 6.1 for循环(JS有两种循环方式)
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
            continue;
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
       break；

        }

```

#### 6.2 while循环：
```html
 while(条件){
    
    }
```

#### 6.3 switch语句：
```html
switch(name){
        case '1':
            age = 123;
            break;
        case '2':
            age = 456;
            break;
        default:
            age = 777;
    }
```

#### 6.4 条件语句：
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


### 7、作用域：
#### 7.1 其他语言：以代码块作为作用域
```ruby
  public void Func(){
            if (1==1){      //一个大括号之间就是一个代码块；
                string name = 'Java';
            }
            console.writeline(name);  //这name不会有值，因为name=Java的作用域不在这
        }
  
  Func()  // 报错
```
#### 7.2 Python：以函数作为作用域
```ruby
情况一：

def func():
    if 1==1:
        name = "Baby"
    print(name)

func()


情况二：

def func():
    if 1==1:
        name = "Baby"
    print(name)

func()
print(name)   # 这个name不会被拿到，这句会报错
```

#### 7.3 JavaScript：以函数作为作用域(不考虑let关键字)
```js

 function func(){
            if(1==1){
                var name = "Baby";
            }
            console.log(name);
        }
        
        func()
```

#### 7.4 函数的作用域在函数未被调用之前，已经创建
```js
 function func(){
            if(1==1){
                var name = "Baby";
            }
            console.log(name);
        }

// 这个函数放在这，还没有调用之前，当解释器去解释的时候，这个作用域其实就已经创建了
```

#### 7.5 函数的作用域存在"作用域链"，并且也是在被调用之前创建
- 实例一
```js
 name = "Wang";
        function func(){
           var name = "Baby";
            function innser(){
                var name = "Shen";
                console.log(name)
            }
            innser()
        }
        func()

// 这个输出Shen，先找当前函数console.log(name)有没有name，如果没有就向上级函数查找，再没有就去找全局的name变量，最后再没有就报错
```
- 实例二
```js
  name = "Wang";
        function func(){
           var name = "Baby";
            function innser(){
                console.log(name)
            }
            return innser;
        }
        
        var ret = func()
        ret()

// 这个输出Baby,假如没有下面的var ret = func()和ret()，上面的会提前生成作用域链，因为innser函数没被调用；
//当执行func函数的时候，返回innser这个函数变量，所以ret就是innser；
//最后对ret加上括号，就是执行innser函数，没有name变量，就去上面的函数再找
```
- 实例三
```js
 name = "Wang";
        function func(){
           var name = "Baby";
            function innser(){
                console.log(name)
            }
            var name = "Shen";   // 其实这个Shen会覆盖上面的Baby变量
            return innser;
        }
        var ret = func()
        ret()

// 这个输出Shen
```


#### 7.6 函数内部变量提前声明
`在函数还没执行函数的时候，程序会找到里面所有的局部变量，如var name都找出来，但是不会带着值`
```js
// var name 会提前把这个name局部变量找出来
 function func(){
            console.log(name);
            var name = "BabyShen"
        }
        
        func()
// 会输出undefined，因为程序自上而下执行，console.log(name)中的name没有值，所以输出undefined
```

### 8、其他
#### 8.1 序列化：
##### JSON.stringify() :将对象转换为字符串
```js
BabyShen = [1,2,3,4,5]
(5) [1, 2, 3, 4, 5]
JSON.stringify(BabyShen)
"[1,2,3,4,5]"
Wang = JSON.stringify(BabyShen)
"[1,2,3,4,5]"
Wang
"[1,2,3,4,5]"
```
##### JSON.parse():将字符串转换为对象(数组)类型
```js
newWang = JSON.parse(Wang)
(5) [1, 2, 3, 4, 5]
newWang
(5) [1, 2, 3, 4, 5]
```

#### 8.2 转义：
`客户端(cookie) => 服务器端`
`需要转义之后保存到硬盘上，如果有特殊字符则必须转义，cookie就是将登陆数据保存在客户端硬盘上，将数据经过转义之后，保存在cookie`

```js
decodeURI( )                   URl中未转义的字符
decodeURIComponent( )          URI组件中的未转义字符
encodeURI( )                   URI中的转义字符
encodeURIComponent( )          转义URI组件中的字符
escape( )                      对字符串转义
unescape( )                    给转义字符串解码
URIError                       由URl的编码和解码方法抛出
```

```js
decodeURI()	URL中未定义的字符

newurl = encodeURI(url);
"https://www.sogou.com/web?query=%E7%A5%9E%E7%AB%A5"
decodeURI(newurl)
"https://www.sogou.com/web?query=神童"
```

```js
decodeURIComponent()	URL组件中的来转义字符
encodeURI()	URL中的转义字符

url = "https://www.sogou.com/web?query=神童";
"https://www.sogou.com/web?query=神童"
url
"https://www.sogou.com/web?query=神童"
encodeURI(url)
"https://www.sogou.com/web?query=%E7%A5%9E%E7%AB%A5"
```

```js
encodeURIComponent()	转义URL组件中的字符

newurl = encodeURIComponent(url)
"https%3A%2F%2Fwww.sogou.com%2Fweb%3Fquery%3D%E7%A5%9E%E7%AB%A5"
```

- 示例：爬虫-给抽屉网站自动点赞
```ruby
C:\Users\Administrator>pip3 install requests
```
```js
首先登陆任何页面(http://dig.chouti.com/)，获取cookie
 
i1 = requests.get(url= "http://dig.chouti.com/help/service")
 
用户登陆，携带上一次的cookie，后台对cookie中的 gpsd 进行授权
i2 = requests.post(
    url= "http://dig.chouti.com/login",
    data= {
        'phone': "86手机号",
        'password': "密码",
        'oneMonth': ""
    },
    cookies = i1.cookies.get_dict()
)
 
点赞（只需要携带已经被授权的gpsd即可）
gpsd = i1.cookies.get_dict()['gpsd']
i3 = requests.post(
    url="http://dig.chouti.com/link/vote?linksId=8589523",
    cookies={'gpsd': gpsd}
)
print(i3.text)
```

#### 8.3 evel:

```ruby
python中的evel：

val = eval(表达式)
	  exec(执行代码)
# python中的exec是没有返回值的
```
```js
JavaScript中的evel:

JavaScript中的evel是Python中eval+exec的合并
```

#### 8.4 时间处理：

`Data类：在python中通过类创建一个对象d = Data()就是直接类后面加括号`
在JavaScript中创建对象需要加一个new，`var d = new Data()`
```html
>var d = new Date
<undefined
>d
<Fri Dec 29 2017 15:10:12 GMT+0800 (中国标准时间)

>d.getMinutes()
<10
>d
<Fri Dec 29 2017 15:10:12 GMT+0800 (中国标准时间)

>n = d.getMinutes() + 4
<14
>d.setMinutes(n)
<1514531652102
>d
<Fri Dec 29 2017 15:14:12 GMT+0800 (中国标准时间)

d.getxxx ：获取
d.setxxx  :设置
```


### 9、后台管理布局
```ruby
position:
	fixed:永远固定在窗口的某个位置；
	absolute：第一次定位，可以指定位置，滑动滚动条时，就不在了；
	relative：单独使用无意义；
```

#### 9.1 方式一：
```html
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        body{
            margin: 0;
        }

        .left{
            float: left;
        }

        .right{
            float: right;
        }
        .pg-header{
            height: 48px;
            background-color: #2459a2;
            color: white;
        }

        .pg-content .menu{
            width: 20%;
            min-width: 200px;
            /*定义最小宽度，如果浏览器不断缩小，20%之后还大于200，那么就按照20%执行，如果20%之后小于了200，那么最小只能是200px*/
            height: 500px;
            background-color: #999999;
        }

        .pg-content .content{
            width: 80%;
            background-color: aqua;
        }
    </style>
</head>
<body>
    <div class="pg-header"></div>
    <div class="pg-content">
        <div class="menu left">Baby</div>
        <div class="content left">
            <p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>
            <p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>
            <p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>
            <p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>
        </div>
    </div>
    <div class="pg-footer"></div>
</body>
</html>
```
#### 9.2 方式二：
```html
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        body{
            margin: 0;
        }

        .left{
            float: left;
        }

        .right{
            float: right;
        }
        .pg-header{
            height: 48px;
            background-color: #2459a2;
            color: white;
        }

        .pg-content .menu{
           position: fixed;
            top: 48px;
            left: 0;
            bottom: 0;
            width: 200px;
            background-color: #999999;
        }

        .pg-content .content{
            position: fixed;
            top: 48px;
            right: 0;
            bottom: 0;
            left: 200px;
            background-color: aquamarine;
            overflow: auto;
            /*出现滚动条*/
        }
    </style>
</head>
<body>
    <div class="pg-header"></div>
    <div class="pg-content">
        <div class="menu left">Baby</div>
        <div class="content left">
            <p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>
            <p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>
            <p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>
            <p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>
        </div>
    </div>
    <div class="pg-footer"></div>
</body>
```
#### 9.3 方式三：
```html
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        body{
            margin: 0;
        }

        .left{
            float: left;
        }

        .right{
            float: right;
        }
        .pg-header{
            height: 48px;
            background-color: #2459a2;
            color: white;
        }

        .pg-content .menu{
            position: absolute;
            top: 48px;
            left: 0;
            bottom: 0;
            width: 200px;
            background-color: #999999;
        }

        .pg-content .content{
            position: absolute;
            top: 48px;
            right: 0;
            bottom: 0;
            left: 200px;
            background-color: aquamarine;
            overflow: auto;
            /*出现滚动条,只需要切换这个，就可以完成两种布局*/
        }
    </style>
</head>
<body>
    <div class="pg-header"></div>
    <div class="pg-content">
        <div class="menu left">Baby</div>
        <div class="content left" >
            <!--<div style="background-color: aquamarine">-->
                <p style="margin: 0">Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>  /* p标签默认有边距 */
                <p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>
                <p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>
                <p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>
                <p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>
                <p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>
            </div>

        </div>
    </div>
    <div class="pg-footer"></div>
</body>
```

#### 9.4 方式四：
```html
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        body{
            margin: 0;
        }

        .left{
            float: left;
        }

        .right{
            float: right;
        }
        .pg-header{
            height: 48px;
            background-color: #2459a2;
            color: white;
            line-height: 48px;
        }

        .pg-header .logo{
            width: 200px;
            background-color: #AA12AA;
            text-align: center;
            /*让文字左右居中*/
        }

        .pg-header .user{
            /*width: 160px;*/
            /*background-color: crimson;*/
            height: 48px;
            padding: 0 20px;
            margin-right: 60px;
        }

        .pg-header .user:hover{
            background-color: #204982;
        }

        .pg-header .user a{
            display: block;
        }

        .pg-header .user .a img{
            height: 40px;
            width: 40px;
            margin-top: 4px;
            border-radius: 50%;   /* 可以调节这个值让用户图片出现各种形状，50%是圆形 */

        }

        .pg-header .user .b{       /*下拉框的*/
            z-index:20;
            position: absolute;
            width:160px;
            top: 48px;
            right: 44px;
            background-color: white;
            color: black;
            display: none
        }

        .pg-header .user:hover .b{
            display: block;
        }

        .pg-content .menu{
            position: absolute;
            top: 48px;
            left: 0;
            bottom: 0;
            width: 200px;
            background-color: #999999;
        }

        .pg-content .content{
            position: absolute;
            top: 48px;
            right: 0;
            bottom: 0;
            left: 200px;
            background-color: aquamarine;
            overflow: auto;
            /*出现滚动条,只需要切换这个，就可以完成两种布局*/
            z-index: 10;
        }
    </style>
</head>
<body>
    <div class="pg-header">
        <div class="logo left">
            BabyShen
        </div>

        <div class="user right" style="position: relative">
            <a class="a" href="#">
                <img src="1.png">
            </a>
            <div class="b">
                <a>我的资料</a>
                <a>注销</a>
            </div>
        </div>

    </div>
    <div class="pg-content">
        <div class="menu left">Baby</div>
        <div class="content left" >
            <!--<div style="background-color: aquamarine">-->
                <p style="margin: 0">Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>
                <p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>
                <p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>
                <p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>
                <p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>
                <p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>
            </div>

        </div>
    </div>
    <div class="pg-footer"></div>
</body>
```

`页面的美观优化功能知识点补充：`
```html
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .item{
            background-color: #999999;
        }

        .item:hover{
            color: red;
        }

        /*.item .b:hover{*/
            /*color: green;*/
        /*}*/
        /*这种方式当鼠标从上之下移动时，还会是全红色字体；当鼠标从下至上移动时，字体就会达到下面的是绿色，上面的是红色*/

        .item:hover .b{
            color: green;
        }
    </style>
</head>
<body>
    <div class="item">
        <div class="a">Baby</div>
        <div class="b">Shen</div>
    </div>
</body>
```
#### 9.5 用户名下方添加下拉框：
```html
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .item{
            background-color: #999999;
        }

        .item:hover{
            color: red;
        }

        /*.item .b:hover{*/
            /*color: green;*/
        /*}*/
        /*这种方式当鼠标从上之下移动时，还会是全红色字体；当鼠标从下至上移动时，字体就会达到下面的是绿色，上面的是红色*/

        .item:hover .b{
            color: green;
        }
    </style>
</head>
<body>
    <div class="item">
        <div class="a">Baby</div>
        <div class="b">Shen</div>
    </div>
</body>
```


#### 9.5 小图标的使用

[下载模板插件-需要翻墙](http://fontawesome.io/)     
格式如下：     
![](https://github.com/ZongYuWang/image/blob/master/python-fontawesome.png) 

```html
<head>
    <meta charset="UTF-8">
    <title></title>
    <link rel="stylesheet" href="font-awesome-4.7.0/css/font-awesome.min.css">
     <!--加载模板-->
    <style>
        body{
            margin: 0;
        }

        .left{
            float: left;
        }

        .right{
            float: right;
        }
        .pg-header{
            height: 48px;
            background-color: #2459a2;
            color: white;
            line-height: 48px;
        }

        .pg-header .logo{
            width: 200px;
            background-color: #AA12AA;
            text-align: center;
            /*让文字左右居中*/
        }

        .pg-header .icons{
            padding: 0 20px;
        }

        .pg-header .style{
            border-radius: 50%;
            display: inline-block;
            padding: 10px 7px;
            line-height: 1px;
            background-color: red;
            font-size: 12px;
        }

        .pg-header .user{
            /*width: 160px;*/
            /*background-color: crimson;*/
            height: 48px;
            padding: 0 20px;
            margin-right: 60px;
        }

        .pg-header .icons:hover{
            background-color: #204982;
        }


        .pg-header .user:hover{
            background-color: #204982;
        }

        .pg-header .user a{
            display: block;
        }

        .pg-header .user .a img{
            height: 40px;
            width: 40px;
            margin-top: 4px;
            border-radius: 50%;

        }

        .pg-header .user .b{       /*下拉框的*/
            z-index:20;
            position: absolute;
            width:160px;
            top: 48px;
            right: 44px;
            background-color: white;
            color: black;
            display: none
        }

        .pg-header .user:hover .b{
            display: block;
        }

        .pg-content .menu{
            position: absolute;
            top: 48px;
            left: 0;
            bottom: 0;
            width: 200px;
            background-color: #999999;
        }

        .pg-content .content{
            position: absolute;
            top: 48px;
            right: 0;
            bottom: 0;
            left: 200px;
            background-color: aquamarine;
            overflow: auto;
            /*出现滚动条,只需要切换这个，就可以完成两种布局*/
            z-index: 10;
        }
    </style>
</head>
<body>
    <div class="pg-header">
        <div class="logo left">
            BabyShen
        </div>


        <div class="user right" style="position: relative">
            <a class="a" href="#">
                <img src="../day16/1.png">
            </a>
            <div class="b">
                <a>我的资料</a>
                <a>注销</a>
            </div>
        </div>

        <div class="icons right">
            <i class="fa fa-bell-o" aria-hidden="true"></i>
            <!--打开图标的网站，找到相应的图标，浏览器打开即可看到语法-->
        </div>
        <div class="icons right">
            <i class="fa fa-commenting" aria-hidden="true"></i>
            <span class="style">5</span>
        </div>


    </div>
    <div class="pg-content">
        <div class="menu left">Baby</div>
        <div class="content left" >
            <!--<div style="background-color: aquamarine">-->
            <p style="margin: 0">Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>
            <p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>
            <p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>
            <p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>
            <p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>
            <p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p><p>Shen</p>
        </div>

    </div>

    <div class="pg-footer"></div>
</body>
```
