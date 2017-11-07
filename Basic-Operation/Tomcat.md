## Tomcat

### 一、Tomcat服务器的配置：
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
/trade/tomcat/webapps/ROOT，这是因为server.xml文件中定义的（vim /trade/tomcat/conf/server.xml ）
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