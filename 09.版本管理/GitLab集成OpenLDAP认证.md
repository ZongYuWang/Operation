## GitLab集成OpenLDAP认证

### 1、Gitlab的默认登录界面

![](https://github.com/ZongYuWang/image/blob/master/GitLab/LDAP_GitLab/Gitlab_page.png)      

### 2、编辑Gitlab配置文件并验证LDAP

#### 2.1 检查gitlab的信息：
```ruby
# gitlab-rake gitlab:env:info

System information
System:		
Current User:	git
Using RVM:	no
Ruby Version:	2.4.4p296
Gem Version:	2.7.6
Bundler Version:1.16.2
Rake Version:	12.3.1
Redis Version:	3.2.11
Git Version:	2.17.1
Sidekiq Version:5.1.3
Go Version:	unknown

GitLab information
Version:	11.0.2
Revision:	d9540ee
Directory:	/opt/gitlab/embedded/service/gitlab-rails
DB Adapter:	postgresql
URL:		http://172.25.101.182
HTTP Clone URL:	http://172.25.101.182/some-group/some-project.git
SSH Clone URL:	git@172.25.101.182:some-group/some-project.git
Using LDAP:	yes
Using Omniauth:	no

GitLab Shell
Version:	7.1.4
Repository storage paths:
- default: 	/var/opt/gitlab/git-data/repositories
Hooks:		/opt/gitlab/embedded/service/gitlab-shell/hooks
Git:		/opt/gitlab/embedded/bin/git

```

#### 2.2 编辑gitlab配置文件：

```ruby
# vim /etc/gitlab/gitlab.rb

gitlab_rails['ldap_enabled'] = true

gitlab_rails['ldap_servers'] = YAML.load <<-'EOS'
   main: # 'main' is the GitLab 'provider ID' of this LDAP server
     label: 'LDAP'
     host: '172.25.101.110'
     port: 389
     uid: 'uid'
     bind_dn: 'cn=Manager,dc=newtvldap,dc=com'
     password: 'newtv123'
     encryption: 'plain'
     verify_certificates: true
     active_directory: false
     allow_username_or_email_login: true
     lowercase_usernames: false
     block_auto_created_users: true
     base: 'ou=people,dc=newtvldap,dc=com'
     user_filter: ''
#     ## EE only
#     group_base: ''
#     admin_group: ''
#     sync_ssh_keys: false
#
#   secondary: # 'secondary' is the GitLab 'provider ID' of second LDAP server
#     label: 'LDAP'
#     host: '_your_ldap_server'
#     port: 389
#     uid: 'sAMAccountName'
#     bind_dn: '_the_full_dn_of_the_user_you_will_bind_with'
#     password: '_the_password_of_the_bind_user'
#     encryption: 'plain' # "start_tls" or "simple_tls" or "plain"
#     verify_certificates: true
#     active_directory: true
#     allow_username_or_email_login: false
#     lowercase_usernames: false
#     block_auto_created_users: false
#     base: ''
#     user_filter: ''
#     ## EE only
#     group_base: ''
#     admin_group: ''
#     sync_ssh_keys: false
EOS

```
```ruby
# gitlab-ctl reconfigure   // 重新刷新配置
# gitlab-ctl restart       // 重启服务
```

#### 2.3 验证gitlab与ldap认证：
```ruby
# gitlab-rake gitlab:ldap:check gives:
Checking LDAP ...

Server: ldapmain
LDAP authentication... Success
LDAP users with access to your GitLab server (only showing the first 100 results)
	DN: cn=user1,ou=people,dc=newtvldap,dc=com	 uid: user1
	DN: cn=user3,ou=people,dc=newtvldap,dc=com	 uid: user3
	DN: cn=test2,ou=group3,cn=group2,ou=group,dc=newtvldap,dc=com	 uid: test2
	DN: cn=user10,ou=people,dc=newtvldap,dc=com	 uid: user10
	DN: cn=user11,ou=people,dc=newtvldap,dc=com	 uid: user11

Checking LDAP ... Finished

```

#### 2.4 使用ldap账号登录gitlab：
![](https://github.com/ZongYuWang/image/blob/master/GitLab/LDAP_GitLab/LDAP_Gitlab1.png)       


![](https://github.com/ZongYuWang/image/blob/master/GitLab/LDAP_GitLab/LDAP_Gitlab.png)    


### 3、FAQ:

登录界面输入用户名和密码之后提示错误：           
Your account has been blocked. Please contact your GitLab administrator if you think this is an error.

** ldap中的账号必须设置邮箱才可以在gitlab中登录 **

![](https://github.com/ZongYuWang/image/blob/master/GitLab/LDAP_GitLab/LDAP_Mail.png)   

```ruby
# gitlab-rails console
-------------------------------------------------------------------------------------
 GitLab:       11.0.2 (d9540ee)
 GitLab Shell: 7.1.4
 postgresql:   9.6.8
-------------------------------------------------------------------------------------
Loading production environment (Rails 4.2.10)
irb(main):001:0> user = User.find_by_email("691793599@qq.com")
=> #<User id:2 @user1>
irb(main):002:0> user.state = "active"
=> "active"
irb(main):003:0> user.save
=> true
irb(main):004:0> exit

```
`如果有多个用户，每个用户都需要这样设置`
```ruby
# gitlab-rails console
-------------------------------------------------------------------------------------
 GitLab:       11.0.2 (d9540ee)
 GitLab Shell: 7.1.4
 postgresql:   9.6.8
-------------------------------------------------------------------------------------
Loading production environment (Rails 4.2.10)
irb(main):001:0> user = User.find_by_email("user3@newtvldap.com")
=> #<User id:3 @user3>
irb(main):002:0> user.state = "active"
=> "active"
irb(main):003:0> user.save
=> true
irb(main):004:0> exit

```