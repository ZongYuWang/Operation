## Jenkins使用OpenLDAP统一认证

LDAP配置参考：
![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins_LDAP/LDAP.png)

### 1、Jenkins安装LDAP插件：

#### 1.1 后台插件管理里直接安装:

![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins_LDAP/LDAP_Plugin.png)

`我已经安装过了LDAP插件，所有这里搜索不到LDAP插件，只有LDAP Email插件`

#### 1.2 官网下载安装文件后台上传:
[插件下载地址](https://updates.jenkins-ci.org/download/plugins/)

![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins_LDAP/Plugin_Manager.png)


### 2、配置LDAP认证
系统管理 ——> 全局安全配置

![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins_LDAP/Jenkins_LDAP.png)

</br>

![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins_LDAP/Jenkins_LDAP_user.png)

- Server:服务器地址,可以直接填写LDAP服务器的主机名或IP,例如ldap://172.25.101.110(默认端口号是389)，如果使用了SSL，可以填写ldaps://172.25.101.110(默认端口号是636)
- root DN：这里的root DN只是指搜索的根，并非LDAP服务器的root DN，理论上，我们如果从子节点（而不是根节点）开始搜索，因为缩小了搜索范围那么就可以获得更高的性能。这里的root DN指的就是这个子节点的DN，当然也可以不填，表示从LDAP的根节点开始搜索
- User search base：这是一个相对的值，相对于上边的root DN，例如你上边的root DN填写的是dc=newtvldap,dc=com，那么user search base这里填写了ou=People，那么登陆用户去LDAP搜索时就只会搜索ou=People,dc=newtvldap,dc=com下的用户了
- User search filter:这个配置定义登陆的“用户名”对应LDAP中的哪个字段，如果你想用LDAP中的uid作为用户名来登录，那么这里可以配置为uid={0}（{0}会自动的替换为用户提交的用户名），如果你想用LDAP中的mail作为用户名来登录，那么这里就需要改为mail={0}
- Group search filter：如果不填写有一个默认值：`(& (cn={0}) (| (objectclass=groupOfNames) (objectclass=groupOfUniqueNames) (objectclass=posixGroup)))`
- Group membership:使用objectclass减小搜索范围