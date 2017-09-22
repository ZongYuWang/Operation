* ## 三级菜单项目
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


* ## 购物卡项目
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

* ## 账号登陆超出次数加入黑名单项目
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