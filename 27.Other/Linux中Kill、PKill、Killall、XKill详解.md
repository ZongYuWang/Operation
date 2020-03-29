## Linux中Kill、PKill、Killall、XKill详解

终止一个进程或终止一个正在运行的程式，一般是通过 kill 、killall、pkill、xkill 等进行。
比如一个程式已死掉，但又不能退出，这时就应该考虑应用这些工具。
另外应用的场合就是在服务器管理中，在不涉及数据库服务器程式的父进程的停止运行，也能用这些工具来终止。为什么数据库服务器的父进程不能用这些工具杀死呢？原因非常简单，这些工具在强行终止数据库服务器时，会让数据库产生更多的文件碎片，当碎片达到一定程度的时候，数据库就有崩溃的危险。
比如mysql服务器最佳是按其正常的程式关闭，而不是用pkill mysqld 或killall mysqld这样危险的动作；当然对于占用资源过多的数据库子进程，我们应该用kill 来杀掉

### 1、kill
kill的应用是和ps 或pgrep 命令结合在一起使用的；
kill 的用法：
```ruby
kill ［信号代码］   进程ID

// 信号代码能省略；我们常用的信号代码是 -9 ，表示强制终止；
```
- 实验举例：
```ruby
[root@localhost ~]# ps -auxf | grep httpd
vivian   27005  0.0  0.0  4928  680 pts/0    S    09:42   0:00  |           \_ grep httpd
root      1742  0.0  0.0 19588  560 ?        S    Nov24   0:00 /var/email/apache/bin/httpd -k start
nobody    1744  0.0  0.0 11304  540 ?        S    Nov24   0:03  \_ /var/email/apache/bin/httpd -k start
nobody   23055  0.0  2.3 327748 24232 ?      S    Nov26   1:43  \_ /var/email/apache/bin/httpd -k start
nobody   23087  0.0  2.4 328252 24832 ?      S    Nov26   1:38  \_ /var/email/apache/bin/httpd -k start
nobody   10607  0.0  2.3 327144 24064 ?      S    Nov27   1:12  \_ /var/email/apache/bin/httpd -k start

```
我们看上面例子中的第二列，就是进程PID的列，其中1742是httpd服务器的父进程，剩下列出的进程都是1742的子进程；如果我们杀掉父进程1742的话，其下的子进程也会跟着死掉
```ruby
[root@localhost ~]# kill 1742

```
对于僵尸进程，能用kill -9 来强制终止退出；
比如一个程式已完全死掉，如果kill 不加信号强度是没有办法退出，最佳的办法就是加信号强度 -9 


### killall
killall 通过程式的名字，直接杀死所有进程,killall 也和ps或pgrep 结合使用，比较方便；通过ps或pgrep 来查看哪些程式在运行；
killall用法:
```ruby
killall 正在运行的程式名
```
- 实验举例：
```ruby
[root@localhost ~]# pgrep -l  httpd
1742 httpd
1744 httpd
23055 httpd
23087 httpd
10607 httpd
[root@localhost ~]# killall httpd
```

### pkill
pkill 和killall 应用方法差不多，也是直接杀死运行中的程式；
如果杀掉单个进程，请用kill来杀掉。
pkill用法：
```ruby
pkill  正在运行的程式名
```

### xkill
xkill 是在桌面用的杀死图像界面的程式。比如当firefox 出现崩溃不能退出时，点鼠标就能杀死firefox 。当xkill运行时出来和个人脑骨的图标，哪个图像程式崩溃一点就OK了。如果你想终止xkill ，就按右键取消；
xkill用法：
```ruby
[root@localhost ~]# xkill

```