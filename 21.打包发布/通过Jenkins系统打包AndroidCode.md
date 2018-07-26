## Linux系统本地打包AndroidCode(APK)


### 1、在jenkins安装必备的插件：
```ruby
Gradle Plugin
```

### 2、在Jenkins任务服务端安装Gradle：	
[gradle软件包下载](https://gradle.org/releases/)

```ruby
# unzip gradle-4.1-bin.zip
# mv gradle-4.1 /bin/
# ln -sv /bin/gradle-4.1/ /bin/gradle
‘/bin/gradle’ -> ‘/bin/gradle-4.1/’

# vim /etc/profile
GRADLE_HOME=/bin/gradle
export PATH=${GRADLE_HOME}/bin:${PATH}
# source /etc/profile

```
```ruby
# gradle -v

------------------------------------------------------------
Gradle 4.1
------------------------------------------------------------

Build time:   2017-08-07 14:38:48 UTC
Revision:     941559e020f6c357ebb08d5c67acdb858a3defc2

Groovy:       2.4.11
Ant:          Apache Ant(TM) version 1.9.6 compiled on June 29 2015
JVM:          1.8.0_171 (Oracle Corporation 25.171-b11)
OS:           Linux 3.10.0-693.5.2.el7.x86_64 amd64
```
### 3、根据项目对应的SDK版本安装相应的SDK：

```ruby
# yum install bind-utils
// 系统默认没有dig命令
```

```ruby
# dig dl.google.com   //在接下来的安装SDK的过程中本机不能连接dl.google.com

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> dl.google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 31713
;; flags: qr rd ra; QUERY: 1, ANSWER: 12, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;dl.google.com.                 IN      A

;; ANSWER SECTION:
dl.google.com.          604629  IN      CNAME   dl.l.google.com.
dl.l.google.com.        129     IN      A       203.208.41.39
dl.l.google.com.        129     IN      A       203.208.41.38
dl.l.google.com.        129     IN      A       203.208.41.35
dl.l.google.com.        129     IN      A       203.208.41.36
dl.l.google.com.        129     IN      A       203.208.41.46
dl.l.google.com.        129     IN      A       203.208.41.33
dl.l.google.com.        129     IN      A       203.208.41.37
dl.l.google.com.        129     IN      A       203.208.41.41
dl.l.google.com.        129     IN      A       203.208.41.40
dl.l.google.com.        129     IN      A       203.208.41.32
dl.l.google.com.        129     IN      A       203.208.41.34

;; Query time: 29 msec
;; SERVER: 114.114.114.114#53(114.114.114.114)
;; WHEN: Mon Jul 23 02:11:58 UTC 2018
;; MSG SIZE  rcvd: 237
```

```ruby
# vim /etc/hosts
203.208.41.70 dl-ssl.google.com  // 从上面任选一个就行
```

```ruby
# wget http://dl.google.com/android/android-sdk_r24.2-linux.tgz
# tar xvf android-sdk_r24.2-linux.tgz 

# vim /etc/profile
export ANDROID_HOME="/newdev/android-sdk-linux"
export PATH="$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools:$PATH"
# source /etc/profile

// 此处使用的24.2 为了下面使用26.0.2的buildTool，不要使用最新版，最新版中没有26.0.2的tools集成
```

```ruby
# android update sdk --no-ui
```

```ruby
# android list sdk --all
......
Packages available for installation or update: 207
   1- Android SDK Tools, revision 25.2.5
   2- Android SDK Platform-tools, revision 28
   3- Android SDK Build-tools, revision 28.0.1
   4- Android SDK Build-tools, revision 28
   5- Android SDK Build-tools, revision 27.0.3
   6- Android SDK Build-tools, revision 27.0.2
   7- Android SDK Build-tools, revision 27.0.1
   8- Android SDK Build-tools, revision 27
   9- Android SDK Build-tools, revision 26.0.3
  10- Android SDK Build-tools, revision 26.0.2
  11- Android SDK Build-tools, revision 26.0.1
  12- Android SDK Build-tools, revision 26
  13- Android SDK Build-tools, revision 25.0.3
  14- Android SDK Build-tools, revision 25.0.2
  15- Android SDK Build-tools, revision 25.0.1
  16- Android SDK Build-tools, revision 25
  17- Android SDK Build-tools, revision 24.0.3
  18- Android SDK Build-tools, revision 24.0.2
  19- Android SDK Build-tools, revision 24.0.1
  20- Android SDK Build-tools, revision 24
  21- Android SDK Build-tools, revision 23.0.3
......
 44- SDK Platform Android 9, API 28, revision 4
  45- SDK Platform Android 8.1.0, API 27, revision 3
  46- SDK Platform Android 8.0.0, API 26, revision 2
  47- SDK Platform Android 7.1.1, API 25, revision 3
  48- SDK Platform Android 7.0, API 24, revision 2
......
 181- Sources for Android SDK, API 27, revision 1
 182- Sources for Android SDK, API 26, revision 1
 183- Sources for Android SDK, API 25, revision 1
 184- Sources for Android SDK, API 24, revision 1
......
```

```ruby
# android update sdk -u -a -t 10
// 10是上面list的编号，也就是按照26.0.2版本的SDK

# android update sdk -u -a -t 46,182
```


```ruby
# cat /mnt/android_back/app/build.gradle    //这个是项目的配置文件

android {

    signingConfigs {
        config {
            keyAlias 'androiddebugkey'
            keyPassword 'android'
            storeFile file('debug.keystore')
            storePassword 'android'
        }
    }
    compileSdkVersion 26
    buildToolsVersion '26.0.2'   // 要求使用的26.0.2
    defaultConfig {
        applicationId "tv.newtv.tvlauncher"
        minSdkVersion 21
        targetSdkVersion 21
        versionCode 9
        versionName '1.0.9'
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
        javaCompileOptions {
            annotationProcessorOptions {
                includeCompileClasspath = true
            }
        }
        multiDexEnabled = true
        signingConfig signingConfigs.config
        flavorDimensions "versionCode"
    }
```

### 4、本地测试生成APK包：

#### 4.1 使用gradle打包生成Debug和Release的软件包

```ruby
# ./gradlew assembleDebug   // 生成Debug的包
......
BUILD SUCCESSFUL in 5m 52s
50 actionable tasks: 3 executed, 47 up-to-date
```
```ruby
# ./gradlew assembleRelease   // 生成Release的包
......
BUILD SUCCESSFUL in 21s
51 actionable tasks: 5 executed, 46 up-to-date
```
`这俩命令可以通用：gradle  assembleRelease`

#### 4.2 最终生成的apk包路径：
```ruby
# ls /mnt/android_back/app/build/outputs/apk/
Panda  VSoon
```

### 5、Linux系统下实现APK的签名：

#### 5.1 证书的创建：
```ruby
# keytool -genkey -v alias KeyName -keyalg RSA -keysize 2048 -validity 10000 -keystore KeyFileName.keystore  

KeyName：表示证书的别名
KeyFileName.keystore： 证书保存的文件名
10000： 表示证书的有效期，单位（天）
RSA：证书的加密类型，一般默认为RSA
```
#### 5.2 证书生成后查看：
```ruby
# keytool -list -alias KeyName -keystore KeyFileName.keystore  
```
#### 5.3 对APK进行签名:
```ruby
jarsigner -verbose -keystore KeyFileName.keystore sign_apk_file.apk KeyName  

KeyFileName.keystore 已经生成号的证书
sign_apk_file.apk 需要签名的apk文件
KeyName 证书的别名

// 待签名的apk文件根根目录下如果有文件夹“META-INFO”，请先删除（重新签名就需要这样做）
// 如果不想创建过程输出太多信息，可以删除“-verbose”
// 上述签名会直接覆盖原来的文件，如果不想被覆盖而签名为另外的新文件 signed.akp
只需将signed.apk 改为 -signedjar signed.apk sign_old.akp  
```

### 6、查看apk的签名：

```ruby
# cat /mnt/android_back/app/build.gradle  // 项目配置文件

productFlavors {
        Panda { manifestPlaceholders.put("channelId", "10002") }
        VSoon { manifestPlaceholders.put("channelId", "10007") }
    }
```
```ruby
[root@host-192-168-1-8 android_back]# ./gradlew assemblePandaDebug
......
BUILD SUCCESSFUL in 1s
26 actionable tasks: 2 executed, 24 up-to-date
```


```ruby
# jarsigner -verify -verbose -certs android_back_v1.0.9_Panda_20180723.apk
......
jar verified.

Warning: 
This jar contains entries whose certificate chain is not validated.
This jar contains signatures that does not include a timestamp. Without a timestamp, users may not be able to validate this jar after the signer certificate's expiration date (2045-07-16) or after any future revocation date.
```


## 通过Jenkins系统打包Android-APK

![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins-android1.png)

Jenkins系统中任务的存放路径：
```ruby
/root/.jenkins/jobs
```

### 1、系统管理中配置SDK环境变量
`Jenkins_Master和Jenkins_Slave所有的环境变量路径要一致`

![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins_Android1.png)

- 设置的值也就是Linux系统环境变量(/etc/profile)的设置

### 2、项目配置

#### 2.1 项目General选项配置：
![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins_Android2.png)
![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins_Android2-1.png)

#### 2.2 源码管理-Git:
![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins_Android3.png)

#### 2.3 构建触发器：
![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins_Android4.png)

- 作用就是当gitlab中有代码更新迭代时立即触发打包操作

#### 2.4 构建：
![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins_Android5.png)


- 此处使用Invoke Grade选择gradle4.1之后，编译会出错，所以选择Use Gradle Wrapper

#### 2.5 构建后的操作(也就是把打包好的apk包发送到远程主机上)：
![](https://github.com/ZongYuWang/image/blob/master/Jenkins/Jenkins_Android6.png)

- SSH Server Name：不是在此处自己手动填写，是在前面(系统管理-系统设置-Publish over SSH)自己设置好的，这里是下拉框选择
- Source files：这是打包服务器相对于上面的图中“使用自定义的工作空间”的相对路径
- Remove prefix:移除上面设置的目录，意思就是只留下包，不然会在远程主机上创建一个路径
- Remote directory：远程主机路径，此处是相对于系统管理-系统设置-Publish over SSH中设置的远程主机的Remote Directory的相对路径
- Exec command：在接收端服务器也就是远程主机上执行的命令