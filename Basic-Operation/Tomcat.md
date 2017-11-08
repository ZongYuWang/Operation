## Tomcat

目录：   
[一、Tomcat服务器的配置](#一、Tomcat服务器的配置)    
[二、一个Tomcat部署多个项目](#二、一个Tomcat部署多个项目)      
[三、Tomcat性能优化](#三、Tomcat性能优化)     

### 一、Tomcat服务器的配置
#### 1.1 安装JDK环境：
```ruby
[root@cxy-65 ~]# ls /trade/
apache-tomcat-7.0.64.tar.gz  jdk-7u79-linux-x64.rpm  lost+found

[root@cxy-65 trade]# rpm -ivh jdk-7u79-linux-x64.rpm --prefix=/usr/local/
Preparing...                ########################################### [100%]
   1:jdk                    ########################################### [100%]
Unpacking JAR files...
	rt.jar...
	jsse.jar...
	charsets.jar...
	tools.jar...
	localedata.jar...
	jfxrt.jar...
ln: creating symbolic link `/usr/java/jdk1.7.0_79': No such file or directory


[root@cxy-65 ~]# cd /usr/local/
[root@cxy-65 local]# mv jdk1.7.0_79/ jdk

[root@cxy-65 local]# vim /etc/profile
JAVA_HOME=/usr/local/jdk
JAVA_BIN=/usr/local/jdk/bin
PATH=$PATH:$JAVA_BIN
CLASSPATH=$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export JAVA_HOME JAVA_BIN PATH CLASSPATH

[root@cxy-65 local]# source /etc/profile
[root@cxy-65 local]# java -version
java version "1.7.0_45"
OpenJDK Runtime Environment (rhel-2.4.3.3.el6-x86_64 u45-b15)
OpenJDK 64-Bit Server VM (build 24.45-b08, mixed mode)

```
#### 1.2 安装配置Tomcat环境：
```ruby
[root@cxy-65 ~]# cd /trade/
[root@cxy-65 trade]# tar xvf apache-tomcat-7.0.64.tar.gz 
[root@cxy-65 trade]# mv apache-tomcat-7.0.64 tomcat7
【说明】如果想使用其他目录可以使用软连接：
ln -s /usr/local/apache-tomcat-8.0.33/  /usr/local/tomcat

[root@cxy-65 trade]# vim /trade/tomcat7/bin/catalina.sh
添加：
CATALINA_HOME=/trade/tomcat7/apache-tomcat-8.0.33
```
启动tomcat：
```ruby
[root@cxy-65 bin]# /trade/tomcat7/bin/catalina.sh start

```
项目路径需要在：   
/trade/tomcat/webapps/ROOT，这是因为server.xml文件中定义的`（vim /trade/tomcat/conf/server.xml ）`
```ruby
 <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">

```

#### 1.3 连接后端MySQL服务器：
```ruby
[root@cxy-65 ~]# tar xvf mysql-connector-java-5.1.39.tar.gz 
[root@cxy-65 ~]# cd mysql-connector-java-5.1.39
[root@cxy-65 ~]# cp mysql-connector-java-5.1.39-bin.jar /trade/tomcat/lib/
[root@cxy-65 ~]# vim /usr/local/tomcat/webapps/ROOT/testmysql.jsp

<%@ page language="java" %>
<%@ page import="com.mysql.jdbc.Driver" %>
<%@ page import="java.sql.*" %>
<%
String driverName="com.mysql.jdbc.Driver";
String userName="root";
String userPasswd="wangzongyu";
String dbName="test";
String url="jdbc:mysql://192.168.1.252/"+dbName+"?user="+userName+"&password="+userPasswd;     #IP地址填写数据库服务器的IP地址
Class.forName("com.mysql.jdbc.Driver").newInstance();
try
{
        Connection connection=DriverManager.getConnection(url);
        out.println(" O K !");
        connection.close();
}
catch( Exception e )
{
        out.println( "connent mysql error:" + e );
}
%>
```

### 二、一个Tomcat部署多个项目
#### 2.1、使用不同的端口：
`使用不同的端口的情况下，每个项目会对应一个进程`
每个Tomcat项目，需要修改的配置如下：


#### 2.2、使用相同的端口：
` 使用相同的端口，多个项目会共用一个tomcat，只需要将项目文件放在appBase指定的路径中即可 `



- tomcat连接数据库的配置文件说明：


### 三、Tomcat性能优化
#### 3.1、开启远程调试端口：
作用：本地电脑监控远程的tomcat端口，可以把服务器上运行的tomcat程序的断点打到本地电脑上面调试，也就是本地可以管理服务器的程序的运行（需要在服务器端配置监听的端口）
- 方法1：    
Linux系统，在catalina.sh里：
CATALINA_OPTS="-server -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8899"

- 方法2：     
startup.sh 中的最后一行 exec "$PRGDIR"/"$EXEXUTABLE" start "$@"中的start修改成jpda start ；      
默认的调试端口是8000 ，可以在catalina.sh 文件中设置JPDA_APPDESS=8000
使用startup.sh 或者catalina.sh jpda start 启动tomcat

#### 3.2、JVM GC日志和内存DUMP参数配置：
GC产生背景：     
1.程序员忘记去释放内存     
2.应用程序访问已经释放的内存     
产生的后果很严重，常见的如内存泄露、数据内容乱码，而且大部分时候，程序的行为会变得怪异而不可预测，还有Access Violation等。
.NET、Java等给出的解决方案，就是通过自动垃圾回收机制GC进行内存管理。这样，问题1自然得到解决，问题2也没有存在的基础。     

总结：无法自动化的内存管理方式极容易产生bug，影响系统稳定性，尤其是线上多服务器的集群环境，程序出现执行时bug必须定位到某台服务器然后dump内存再分析bug所在，极其打击开发人员编程积极性，而且源源不断的类似bug让人厌恶。

不管是YGC还是Full GC,GC过程中都会对导致程序运行中中断,正确的选择不同的GC策略,调整JVM、GC的参数，可以极大的减少由于GC工作，而导致的程序运行中断方面的问题，进而适当的提高Java程序的工作效率。但是调整GC是以个极为复杂的过程，由于各个程序具备不同的特点，如：web和GUI程序就有很大区别（Web可以适当的停顿，但GUI停顿是客户无法接受的），而且由于跑在各个机器上的配置不同（主要cup个数，内存不同），所以使用的GC种类也会不同，所以通过调整 JVM、GC的一些重要参数的设置来提高系统的性能。    
```ruby
JAVA_OPTS="$JAVA_OPTS -server -Xms2G -Xmx2G -Xss256k -XX:PermSize=128m -XX:MaxPermSize=128m -XX:+UseConcMarkSweepGC -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/soft/apache-tomcat-7.0.76/logs/dump_tomcat.hprof  -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:/soft/apache-tomcat-7.0.76/logs/gc_tomcat.log -XX:NewSize=1G -XX:MaxNewSize=1G

```

#### 3.3、Tomcat配置添加对JMX的支持：
JMX作用：JMX要管理的对象是企业中的的各种应用软件和平台，举例来说，一个公司内部可能有许多应用服务器、若干Web服务器、一台至多台的数据库服务器及文件服务器等等，那么，如果我们想监视数据库服务器的内存使用情况，或者我们想更改应用服务器上JDBC最大连接池的数目，但我们又不想重启数据库和应用服务器，这就是典型意义上的资源管理，即对我们的资源进行监视(Monitoring，查看)和管理(Management，更改)，这种监视和更改不妨碍当前资源的正常运行。对资源进行适当的监测和管理，可以让我们的IT资源尽可能的平稳运行，可以为我们的客户提供真正意思上的24×7服务。在资源耗尽或者在硬件出故障之前，我们就可以通过管理工具监测到，并通过管理工具进行热调整和插拔。
```ruby
vim /usr/local/tomcat/bin/catalina.sh

CATALINA_OPTS=" \
-Dcom.sun.management.jmxremote=true \
-Dcom.sun.management.jmxremote.port=9012 \
-Dcom.sun.management.jmxremote.ssl=false \
-Dcom.sun.management.jmxremote.authenticate=false \
-Djava.rmi.server.hostname=192.168.1.111"
```
- 启动Tomcat并测试(查看9012端口是否启动)

![](https://github.com/ZongYuWang/image/blob/master/tomcat1.png)
