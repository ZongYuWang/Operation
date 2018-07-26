## 通过Jenkins系统打包JavaCode


### 1、在Jenkins任务服务端安装Mavne：	

#### 1.1 编译安装Maven：
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

#### 1.2 添加Maven私服：
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

### 2、源码管理-SVN

#### 2.1 服务器上测试SVN连接：
```ruby
# svn co svn://172.25.254.40/ysten-code/TMS/tms
```
![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins_svn1.png)


待完善...