## 项目目录：

[一、三级菜单项目](#一)  
[二、购物卡项目](#二)   
[三、账号登陆超出次数加入黑名单项目](#三)    
[四、实现简单的shell sed替换功能](#四)    
[五、修改haproxy配置文件](#五)   
[六、模拟实现一个ATM + 购物商城程序](#六)   
[七、开发一个简单的python计算器](#七)  
[八、选课系统](#八) 


<h3 id="一">一、三级菜单项目</h3>

#### 代码如下：
```py
# Author：wangzongyu
# -*- coding:utf-8 -*-
data = {
    '北京':{
        "昌平":{
            "沙河":["oldboy","test"],
            "天通苑":["链家地产","我爱我家"]
        },
        "朝阳":{
            "望京":["奔驰","陌陌"],
            "国贸":{"CICC","HP"},
            "东直门":{"Advent","飞信"},
        },
        "海淀":{},
    },
    '山东':{
        "德州":{},
        "青岛":{},
        "济南":{}
    },
    '广东':{
        "东莞":{},
        "常熟":{},
        "佛山":{},
    },
}

exit_flag = False

while not exit_flag:
    for i in data:
        print(i)
    choice = input("选择进入1>>:")
    if choice == "q":
        exit_flag = True
    while not exit_flag:
        if choice in data:
            for i2 in data[choice]:
                print("\t",i2)
            choice2 = input("选择进入2>>:")
            if choice2 == "b":
                break
            elif choice2 == "q":
                exit_flag = True
            while not exit_flag:
                if choice2 in data[choice]:
                    for i3 in data[choice][choice2]:
                        print("\t\t", i3)
                    choice3 = input("选择进入3>>:")
                    if choice3 == "b":
                        break
                    elif choice3 == "q":
                        exit_flag = True
                    while not exit_flag:
                        if choice3 in data[choice][choice2]:
                            for i4 in data[choice][choice2][choice3]:
                                print("\t\t",i4)
                            choice4 = input("最后一层，按b返回>>:")
                            if choice4 == "b":
                                break
                            elif choice4 == "q":
                                exit_flag = True

````

```py

data = {
    '北京':{
        "昌平":{
            "沙河":["oldboy","test"],
            "天通苑":["链家地产","我爱我家"]
        },
        "朝阳":{
            "望京":["奔驰","陌陌"],
            "国贸":{"CICC","HP"},
            "东直门":{"Advent","飞信"},
        },
        "海淀":{},
    },
    '山东':{
        "德州":{},
        "青岛":{},
        "济南":{}
    },
    '广东':{
        "东莞":{},
        "常熟":{},
        "佛山":{},
    },
}
exit_flag = False

while not exit_flag:
    for i in data:
        print(i)
    choice = input("选择进入1>>:")
    if choice in data:
        while not exit_flag:
            for i2 in data[choice]:
                print("\t",i2)
            choice2 = input("选择进入2>>:")
            if choice2 in data[choice]:
                while not exit_flag:
                    for i3 in data[choice][choice2]:
                        print("\t\t", i3)
                    choice3 = input("选择进入3>>:")
                    if choice3 in data[choice][choice2]:
                        for i4 in data[choice][choice2][choice3]:
                            print("\t\t",i4)
                        choice4 = input("最后一层，按b返回>>:")
                        if choice4 == "b":
                            pass
                        elif choice4 == "q":
                            exit_flag = True
                    if choice3 == "b":
                        break
                    elif choice3 == "q":
                        exit_flag = True
            if choice2 == "b":
                break
            elif choice2 == "q":
                exit_flag = True

````

<h3 id="二">二、购物卡项目</h3>

#### 代码如下：
```py

# Author：wangzongyu
# -*- coding:utf-8 -*-

product_list = [
    ('Iphone',5800),
    ('Mac Pro',9800),
    ('Bike',800),
    ('Watch',10600),
    ('Coffee',31),
    ('Book Python',120),
]

shopping_cart = []

card_money = 20000
print("您当前可用余额为:",card_money)
print("------------------Product Info---------------------------")

while True:
    for index,list in enumerate(product_list):
        print(index,list)
    while True:
        user_choice = input("请输入要购买的商品编号>>>: ")
        if user_choice.isdigit():
            user_choice = int(user_choice)
            if user_choice >= 0 and user_choice <= len(product_list):
                product_item = product_list[user_choice]
                if product_item[1] <= card_money:
                    shopping_cart.append(product_item)
                    card_money = card_money - product_item[1]
                    print("Added %s into shopping cart,your current balance is \033[31;1m%s\033[0m" %(product_item,card_money))
                    break
                else:
                    print("\033[41;1m你的余额只剩[%s]啦，还买个毛线\033[0m",card_money)
                    break
            else:
                print("物品不存在！！！！")
                break

        elif user_choice == "q":
            print("----------Shopping Card List---------------")
            for list_new in shopping_cart:
                print(list_new)
            print("\033[41;1m卡内余额还有：\033[0m" ,card_money)
            exit()
        else: #输入任何特殊字符，都打印商品列表
            break

```

<h3 id="三">三、账号登陆超出次数加入黑名单项目</h3>  

#### 代码如下：
```py
# Author：wangzongyu
# -*- coding:utf-8 -*-

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
###### black_username文件内容：
```py

wangzy
hello
wangzongyu

```

<h3 id="四">四、实现简单的shellsed替换功能</h3>     

```ruby
import sys

file = open('test1','r')
file_new = open('test2','w')

find_str = sys.argv[1]
replace_str = sys.argv[2]

for line in file.readlines():
    line = line.replace(find_str,replace_str)
    print(line)   # 将所有的内容全部打印
    file_new.write(line)

file.close()
file_new.close()

运行：
E:\PycharmProjects\untitled\study\ATM\test>python main.py myname yourname

```

<h3 id="五">五、修改haproxy配置文件</h3>    

```ruby

# Author：wangzongyu
# -*- coding:utf-8 -*-

########################  search begin ########################
back_domain = input("请输入要查询的域名(eg:www.baidu.com)>>>:")

def search(back_domain):   # back_domain = "www.wangzy.com"
    flag = "green"
    result = []  #最终通过return将返回值存入result
    with open("haproxy.conf",'r',encoding="utf-8") as f:
        for line in f:
            if line.strip() == "backend %s" %back_domain:
               flag = "red"      #读到想查询的那条记录了，先停住，将flag设置为red，跳出这个循环，执行一些动作，green的时候会一直读取，不做任何动作
               continue          # 跳出本次循环，继续向下执行；

            if flag == "red":
                '''
                >>> print(line)
                        server 100.1.7.9 100.1.7.9 weight 20 maxconn 3000
                backend www.oldboy.com
                        server 100.1.7.19 100.1.7.19 weight 20 maxconn 3000
                '''
                if line.strip().startswith("backend"):  # 读取到某条是以backend开头的数据，就不再存入到result中了，下面设置为green，让程序自己再去向下读取吧
                    flag = "green"
                    break # 找到相应的backend之后，下面如果还有很多backend，就不再读取了
                else:   #如果不是backend，就存到result中，也就是匹配到的backend和下一个backend之间的值要存取
                    if line.strip(): # 空值是假，line.strip()是非空值，是真，如果不写这行，backend下面有一个空行会显示出来
                        result.append(line.strip())

    return result

print( search(back_domain) )

########################  search end  ########################

########################  add begin  ########################
def add():

    import json

    domain = input("请输入backend的域名(eg:www.baidu.com)>>>:")
    ipaddr = input("请输入backend的服务器IP地址(eg:192.168.1.100)>>>:")
    weight = input("请输入backend相应的权重值(eg:20)>>>:")
    maxconn = input("请输入backend的最大链接数(eg:50)>>>:")

    # json的标准格式：要求必须 只能使用双引号作为键 或者 值的边界符号，不能使用单引号，而且“键”必须使用边界符（双引号）
    arg = ''' {
         "backend":"%s",
         "record":{
             "server":"%s",
             "weight":%s,
             "maxconn":%s
          }
    }'''  % (domain,ipaddr,weight,maxconn)

    # print(arg)   # 输出用户输入的字符串

    dict_domain = json.loads(arg) # 将用户输入的格式转换成字典格式
    back_domain = dict_domain["backend"] # 获取backend后的域名(www.test.com)
    # print(back_domain)
    back_record = "server %s %s weight %d maxconn %d" %(dict_domain["record"]["server"],
                                                        dict_domain["record"]["server"],
                                                        dict_domain["record"]["weight"],
                                                        dict_domain["record"]["maxconn"])
   # 获取backend域名之下的服务器信息，如：server 100.1.7.9 100.1.7.9 weight 20 maxconn 3000

    record_list = search(back_domain) # 通过上面的search函数，获取域名下的所有server数据
    record_list.append(back_record)  # 将新的server信息使用append加入到原来的record_list列表中
    # print(record_list)

    flag = "green"  #定义开始一路绿灯
    with open("haproxy.conf",'r',encoding="utf-8") as f_old,open("haproxy.conf_new",'w',encoding="utf-8") as f_new:

        for line in f_old:
            if line.strip() == "backend %s" %back_domain:  #当遇到输入的域名(这个域名需要提前在配置文件中存在)

                user_confirm = input("%s already exists,You will continue to add? " %back_domain)
                if user_confirm == "yes":
                    flag = "red"  # 先把灯置为红灯，接下来后面都是红灯了，红灯之后，先把backend+域名写入到new文件中
                    f_new.write(line)

                    for new_line in record_list:  #接下来也要一行行的遍历对应backend+域名下的server信息，一行行的写入到新的文件中
                        f_new.write(" "*8 + new_line + "\n")
                    continue
                else:  # user_confirm == "no"
                    pass #调用删除或者修改函数

            if flag == "green":
                f_new.write(line)

            else:  # 也就是还是红灯的时候，也不能都不写入，响应backend后面的backend也要继续写入到新的文件
                flag = "green"
                f_new.write(line)

        #back_domain不在配置文件中，也就是backend后面的域名不是提前存在配置文件中（line.strip() ！= "backend %s" %back_domain）
        f_new.write("backend %s" %back_domain + "\n")
        f_new.write(" "*8 + back_record)

add()

########################  delete begin  ########################

def delete():

    back_domain = input("请输入需要删除的backend域名(eg:www.baidu.com)>>>:")
    record_list = search(back_domain)
    flag = "green"
    with open('haproxy.conf','r',encoding="utf-8") as f_old,open('haproxy.conf_new1','w',encoding="utf-8") as f_new:
        for line in f_old:
            if line.strip() != "backend %s" %back_domain and flag=="green":
                f_new.write(line)

            else:  # line.strip() == "backend %s" %back_domain
                flag = "red"

                if line.strip().startswith("backend") and line.strip() != "backend %s" %back_domain:
                    f_new.write(line)
                    flag = "green"

delete()
########################  delete end  ########################
```

<h3 id="六">六、模拟实现一个ATM + 购物商城程序</h3>    

要求：
- 额度 15000或自定义
- 实现购物商城，买东西加入 购物车，调用信用卡接口结账
- 可以提现，手续费5%
- 每月22号出账单，每月10号为还款日，过期未还，按欠款总额 万分之5 每日计息
- 支持多账户登录
- 支持账户间转账
- 记录每月日常消费流水
- 提供还款接口
- ATM记录操作日志 
- 提供管理接口，包括添加账户、用户额度，冻结账户等
- 用户认证用装饰器


<h3 id="七">七、开发一个简单的python计算器</h3>    

- 实现加减乘除及拓号优先级解析
- 用户输入 1 - 2 * ( (60-30 +(-40/5) * (9-2*5/3 + 7 /3*99/4*2998 +10 * 568/14 )) - (-4*3)/ (16-3*2) )等类似公式后，必须自己解析里面的(),+,-,*,/符号和公式(不能调用eval等类似功能偷懒实现)，运算后得出结果，结果必须与真实的计算器所得出的结果一致


<h3 id="八">八、选课系统</h3>    

角色:学校、学员、课程、讲师
要求:
- 创建北京、上海 2 所学校
- 创建linux , python , go 3个课程 ， linux\py 在北京开， go 在上海开
- 课程包含，周期，价格，通过学校创建课程 
- 通过学校创建班级， 班级关联课程、讲师
- 创建学员时，选择学校，关联班级
- 创建讲师角色时要关联学校， 
- 提供两个角色接口
  - 学员视图， 可以注册， 交学费， 选择班级，
  - 讲师视图， 讲师可管理自己的班级， 上课时选择班级， 查看班级学员列表 ， 修改所管理的学员的成绩 
  - 管理视图，创建讲师， 创建班级，创建课程
- 上面的操作产生的数据都通过pickle序列化保存到文件里