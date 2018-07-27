## Jenkins分布式部署

&emsp;&emsp;根据你是用的源代码管理工具Git或者SVN来安装对于的工具，还有需要安装可能会构建的项目所需的环境，比如.NET Core 项目就需要安装 .NET Core SDK，JAVA项目就需要安装JAVA环境，从节点无需安装Jenkins

![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins.png)

### 1、安装JDK
```ruby
# rpm -ivh jdk-8u171-linux-x64.rpm
```
```ruby
# java -version
java version "1.8.0_171"
Java(TM) SE Runtime Environment (build 1.8.0_171-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)
```

### 2、安装部署Maven

```ruby
# tar xvf apache-maven-3.5.4-bin.tar.gz -C /bin/
# ln -sv /bin/apache-maven-3.5.4/ /bin/maven
‘/bin/maven’ -> ‘/bin/apache-maven-3.5.4/’
```
```ruby
# vim /etc/profile
export M2_HOME=/bin/maven
export MAVEN_HOME=/bin/maven
PATH=$M2_HOME/bin:$PATH
```
```ruby
# source /etc/profile
# mvn -v
Apache Maven 3.5.4 (1edded0938998edf8bf061f1ceb3cfdeccf443fe; 2018-06-17T14:33:14-04:00)
Maven home: /usr/local/maven3
Java version: 1.8.0_171, vendor: Oracle Corporation, runtime: /usr/java/jdk1.8.0_171-amd64/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-693.el7.x86_64", arch: "amd64", family: "unix"
```

#### 2.1 添加Maven私服：
```ruby
# vim /bin/maven/conf/settings.xml
<mirrors>
      <mirror>
        <id>localCentralRepo</id>
        <mirrorOf>central</mirrorOf>
        <name>local central</name>
        <url>http://172.25.253.87:8090/nexus/content/groups/public/</url>
    </mirror>

    <mirror>
        <id>localystRepo</id>
        <mirrorOf>ystpub</mirrorOf>
        <name>local ystpub</name>
        <url>http://172.25.253.87:8090/nexus/content/repositories/ystpub/</url>
    </mirror>

    <!-- mirror
     | Specifies a repository mirror site to use instead of a given repository. The repository that
     | this mirror serves has an ID that matches the mirrorOf element of this mirror. IDs are used
     | for inheritance and direct lookup purposes, and must be unique across the set of mirrors.
     |
    <mirror>
      <id>mirrorId</id>
      <mirrorOf>repositoryId</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://my.repository.com/repo/path</url>
    </mirror>
     -->
  </mirrors>

```

### 3、安装Git

[git下载地址](https://mirrors.edge.kernel.org/pub/software/scm/git/)

#### 3.1 安装git依赖包：
```ruby
# yum groupinstall "Development Tools"
# yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker openssh-clients

```
#### 3.2 编译安装git：
```ruby
# tar xvf git-2.18.0.tar.gz
# cd git-2.18.0
# make prefix=/usr/local/git all 
# make prefix=/usr/local/git install
# git --version
git version 1.8.3.1   // 此处的版本不是上面安装的版本

```
```
# whereis git 
git: /usr/bin/git /usr/local/git /usr/local/git/bin/git /usr/share/man/man1/git.1.gz
// 问题所在，居然新安装的在最后了

# vim /etc/profile
export PATH=/usr/local/git/bin:$PATH  // 一定要把新安装的写在系统变量前面
```

### 4、部署Tomcat
```ruby
# tar xvf apache-tomcat-8.5.9.tar.gz -C /usr/local/
```

### 5、部署Jenkins
```ruby
# wget http://mirrors.jenkins.io/war/latest/jenkins.war
# cp ~/jenkins_soft/jenkins.war /usr/local/apache-tomcat-8.5.9/webapps/
```
#### 5.1 访问Jenkins：
```ruby
http://172.25.253.111:8080/jenkins
[root@jenkins ~]# cat /root/.jenkins/secrets/initialAdminPassword 
11b9ddfa2653403f9d62a809c0b25c0e
```

#### 5.2 在jenkins安装必备的插件：
```ruby
Gitlab Authentication
GitLab
// 安装之后才可以在系统配置中指定gitlab的IP地址
Git Plugin 
Git Client Plugin 
// 用于jenkins在gitlab中拉取源码
Publish Over SSH 
// 用于通过ssh部署应用
Maven Integration
// 用于新建maven项目
```

### 6、 秘钥配置

- 确保Master和Slave之间能相互通信
- master执行以下shell命令，确保master能通过ssh登录slave

```ruby
# vim /etc/hosts
172.25.101.207 jmaster
172.25.101.212 jslave2
172.25.101.220 jslave1
```

#### 6.1 在jenkins的master节点生成秘钥：
```ruby
// 生成私钥
# ssh-keygen -t rsa 
# cat /root/.ssh/id_rsa
-----BEGIN RSA PRIVATE KEY-----
.....
.....
-----END RSA PRIVATE KEY-----

```
#### 6.2 将master的秘钥拷贝至slave节点：
```ruby
# ssh-copy-id root@jslave
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host 'jslave (172.25.101.212)' can't be established.
ECDSA key fingerprint is SHA256:qqkQW6QLKNqq3WIKrtdKDXK1CmTeZRipcpiwT44Hhe0.
ECDSA key fingerprint is MD5:85:58:ff:87:c3:c0:26:d6:89:e5:7b:c2:1f:72:db:60.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@jslave's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@jslave'"
and check to make sure that only the key(s) you wanted were added.
```
#### 6.3 在slave节点上查看秘钥：
```ruby
[root@host-192-168-1-11 .ssh]# cat authorized_keys 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQD6QoJQewQ1PLENPddHEawdeBTjq9ZoS0czs1bVuDawEoTeFZvv5j2+m35gyfvRQ65qEwCsLT9dHKJmshmkapz8gFi2GZ7NCDFMmtpbb5TPopABYy+ay5RQdkv5XWZtMl+R2z5mSmw9zHuAi36Xsg+ukdeGqat4xWos5vlnxD+9r1RI63Z3vCaSZdj3zce8AcR4JpXxwW6XSz4TcKzYReLcpWyOvh4zdTeQkx8GAwxY6AQ4vBJrfKqjAlRxpeuRcgawi74vaZJfQPFmpL49IsARmjOIZk0RXGJn28JCc7i3V3VqfdgzeHPCxLq0kG9HYvY7j1b4xH95B1nFsn58b9zB root@jmaster
```


![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins_Credentials1.png)
- 所有系统使用到的凭据都可以在凭据——系统中设置
![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins_Credentials2.png)
- Private Key:是连接端的私钥地址，这个秘钥是设置Master连接Slave的，所以此处设置的是Master的私钥
- 描述：描述信息一定要写上，不然最后秘钥不容易区分


### 7、Jenkins系统环境基本配置

![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins1.png)
![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins2.png)
![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins3.png)
![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins3-1.png)
![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins4.png)


### 8、主从连接的三种方式：
系统管理——节点管理——新建节点

#### 8.1 通过Java Web启动代理：
`如果Slave节点是Windows系统建议使用此种方式`

![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins_slave_web.png)

#### 8.2 Launch slave agents via SSH:
![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins_slave_ssh.png)

- 首先提前配置好主可以无秘钥登录到从节点

#### 8.3 可以使用JNLP的方式主从连接：
`这种连接方式是针对第一种方式的执行命令在Linux系统中执行`
```ruby
[root@jenkinsslave1 ~]# java -jar agent.jar -jnlpUrl http://172.25.101.111:8080/jenkins/computer/openstack_slave/slave-agent.jnlp -secret bf3b48019606f36771f001d857b2ee4c2649ebdc76fc7532520dd15aba8e3ced -workDir "/mnt/jenkins"
Jul 17, 2018 3:54:26 AM org.jenkinsci.remoting.engine.WorkDirManager initializeWorkDir
INFO: Using /mnt/jenkins/remoting as a remoting work directory
Both error and output logs will be printed to /mnt/jenkins/remoting
Jul 17, 2018 3:54:36 AM hudson.remoting.jnlp.Main createEngine
INFO: Setting up agent: openstack_slave
Jul 17, 2018 3:54:36 AM hudson.remoting.jnlp.Main$CuiListener <init>
INFO: Jenkins agent is running in headless mode.
Jul 17, 2018 3:54:36 AM hudson.remoting.Engine startEngine
INFO: Using Remoting version: 3.20
Jul 17, 2018 3:54:36 AM org.jenkinsci.remoting.engine.WorkDirManager initializeWorkDir
INFO: Using /mnt/jenkins/remoting as a remoting work directory
Jul 17, 2018 3:54:40 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Locating server among [http://172.25.253.111:8080/jenkins/, http://172.25.101.111:8080/jenkins/]
Jul 17, 2018 3:55:10 AM org.jenkinsci.remoting.engine.JnlpAgentEndpointResolver resolve
INFO: Remoting server accepts the following protocols: [JNLP4-connect, Ping]
Jul 17, 2018 3:55:10 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Agent discovery successful
  Agent address: 172.25.101.111
  Agent port:    38348
  Identity:      97:3e:7d:71:bc:d7:24:37:2d:b5:5b:f2:45:3e:bb:c1
Jul 17, 2018 3:55:11 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Handshaking
Jul 17, 2018 3:55:11 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Connecting to 172.25.101.111:38348
Jul 17, 2018 3:55:11 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Trying protocol: JNLP4-connect
Jul 17, 2018 3:55:14 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Remote identity confirmed: 97:3e:7d:71:bc:d7:24:37:2d:b5:5b:f2:45:3e:bb:c1
Jul 17, 2018 3:55:46 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Connected
```


## Jenkins打包完成的文件发送到远程主机

### 1、在jenkins安装必备的插件
```ruby
Publish Over SSH 插件
```
![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins_publish_over_ssh.png)

- Passphrase:jenkins是发送端（分布式任务不会在Master端，发送端可能是任何一个Slave节点）生成私钥的密码，可能没有设置；
- Path to key:是jenkins发送端的私钥的路径；
- Key:jenkins是发送端的私钥；
- Disable exec:此选项如果√上，那么在后面的配置中将不能使用命令；
- SSH Servers：Remote Directory是接收端的路径，记住后面如果使用接收端的路径时，都是针对此路径的相对路径；

![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins_Android6.png)

## Jenkins配置文件的备份：
&emsp;&emsp;以防因为Master节点损坏，可以重新搭建一个环境导入配置文件即可    
`最好单独找一个磁盘专门存放备份文件`
### 1、在jenkins安装必备的插件
```ruby
系统管理——插件管理——可选插件——thinbackup
```
![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins_ThinBackup.png)

- 在其他主机上恢复完成的时候，要重启jenkins才能使恢复生效
