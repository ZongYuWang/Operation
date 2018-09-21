## Linux上部署Gitlab

[Gitlab-ce镜像源站](https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/)

### 1、安装gitlab软件包
```ruby
# yum install -y curl policycoreutils-python openssh-server

```
```ruby
# yum install postfix
# systemctl enable postfix
# systemctl start postfix
```
```ruby
# curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash

```
### 2、配置gitlab

```ruby
# rpm -ivh gitlab-ce-11.0.2-ce.0.el7.x86_64.rpm
# vim /etc/gitlab/gitlab.rb
external_url 'http://172.25.253.112/'

# vim /var/opt/gitlab/gitlab-rails/etc/unicorn.rb
listen "172.25.253.112:8080", :tcp_nopush => true

```

```ruby
# gitlab-ctl reconfigure

......
Running handlers:
Running handlers complete
Chef Client finished, 427/611 resources updated in 05 minutes 17 seconds
gitlab Reconfigured!

```
### 3、启动gitlab
```ruby
[root@gitlab gitlab_soft]# gitlab-ctl restart
ok: run: alertmanager: (pid 22163) 1s
ok: run: gitaly: (pid 22171) 0s
ok: run: gitlab-monitor: (pid 22182) 1s
ok: run: gitlab-workhorse: (pid 22195) 0s
ok: run: logrotate: (pid 22205) 0s
ok: run: nginx: (pid 22212) 0s
ok: run: node-exporter: (pid 22218) 0s
ok: run: postgres-exporter: (pid 22224) 1s
ok: run: postgresql: (pid 22307) 0s
ok: run: prometheus: (pid 22309) 1s
ok: run: redis: (pid 22324) 0s
ok: run: redis-exporter: (pid 22441) 1s
ok: run: sidekiq: (pid 22449) 0s
ok: run: unicorn: (pid 22461) 0s

[root@gitlab gitlab_soft]# gitlab-ctl stop
```

### 4、访问gitlab
```ruby
http://172.25.253.112:8080

// 登陆后会修改密码
root newtv123.com
```

