## Linux内存命令free详解

### CentOS6
```ruby
[root@CentOS6 ~]# uname -a
Linux CentOS6.5 2.6.32-431.el6.x86_64 #1 SMP Fri Nov 22 03:15:09 UTC 2013 x86_64 x86_64 x86_64 GNU/Linux
```

查看free帮助手册：
```ruby
[root@CentOS6 ~]# man free
DESCRIPTION
       free  displays the total amount of free and used physical and swap memory in the system, as well as the buffers used by the kernel.  The shared memory column should
       be ignored; it is obsolete.

```

#### 概念解释：
```ruby
[root@CentOS6 ~]# free -m  
             total       used       free     shared    buffers     cached
Mem:          1861        104       1757          0       0          5
-/+ buffers/cache:         98       1762
Swap:         2047          0       2047

Mem行：
这一行的输出是从操作系统(OS)的角度来看的，也就是说从OS角度来看，服务器一共有：
· 1861M的物理内存，这些物理内存中有104M被使用了，还有1757M剩余可用
· 等式：total = used + free
· shared表示被几个进程共享的内存，现在已经obsolete(废弃),其值总是0
· buffers表示内OS buffer住的内存，cached表示被OS cache的内存(buffers和cached下面有详解)


-/+ buffers/cache行:
这一行的输出是从一个应用程序的角度看系统内存的使用情况：
· used：表示一个应用程序认为系统被用掉多少内存
· free：表示一个应用程序认为系统还有多少内存  # 其实这个才是系统真正剩余的可用内存
· 因为被系统cache和buffer占用的内存可以被快速回收，所以通常free(-/+ buffers/cache行)的值 > free(Mem)的值
· 等式： free(buffers/cache) = free(Mem) + buffers(Mem) + cached(Mem)
	    used(buffers/cache) = used(Mem) + buffers(Mem) + cached(Mem)
	    total(Mem) = free(buffers/cache) +  used(buffers/cache)
```
- cache(把数据从硬盘读到内存中)：

cache 就是缓存的意思。当系统读文件的时候，都是把数据从硬盘读到内存里，因为硬盘比内存慢很多，所以这个过程会很耗时。为了提高效率，linux 会把读进来的文件在内存中缓存下来（因为读取相近部分的内容是程序很常见的情况），即使程序结束，cache 也不会被自动释放。所以呢，如果有程序进行大量的读文件操作，你会发现内存使用率就上去了。

不过也不用担心，如果其他程序使用要使用内存的时候，linux 也会把这些没人使用的 cache 释放掉，给其他运行的程序使用。当然你也可以手动去释放掉这部分内存：
```ruby
[root@CentOS6 ~]# echo 1 > /proc/sys/vm/drop_caches
```
- buffer(把数据从内存写到硬盘中):

buffer 的意思和 cache 相近，不过稍有区别。考虑内存写文件到硬盘的过程，因为硬盘太慢了，如果内存要等待数据写完之后才继续后面的操作，实在是效率很低的事情，也会影响程序的运行速度。所以就有了 buffer，写到硬盘的数据会放到 buffer 里面，内存很快把数据写到 buffer，可以继续其他的工作，而硬盘可以在后台慢慢读出 buffer 中的数据，保存起来。这样就提高了读写的效率！

讲一个大家会经常遇到的例子，当我们把电脑里中的文件拷贝到 U 盘的时候，如果文件特别大，大家会遇到这种情况：明明看到文件已经拷贝完了，但系统还是会提示 U 盘正在使用中。这就是 buffer 的原因，拷贝程序把东西放到 buffer 之后，但是 U 盘还没有写完。

同样的，可以手动来 flush buffer 的内容，使用的命令是 sync。

- swap

swap 是实现虚拟内存的重要概念。如果系统的负载太大，内存被用完，可能会出现严重的问题。swap 就是把硬盘上一部分空间当做内存使用，正在运行程序会使用物理内存，把没有正在使用的内存放到硬盘，这叫做 swap out；而把硬盘 swap 部分的内存重新放到物理内存中，叫做swap in。

swap 可以再逻辑上扩大内存空间，但是会造成系统变慢，因为硬盘读写速度很慢。linux 系统比较智能，会把那些不怎么频繁使用的内存放到 swap。
```ruby
[root@CentOS6 ~]# swapoff -a  # 关闭swap
[root@CentOS6 ~]# swapon -a   # 开启swap

```


#### 实验验证：
```ruby
先清理一下cache：
[root@CentOS6 ~]# echo 1 > /proc/sys/vm/drop_caches

```
最开始的时候，先查看一下内存信息`(使用-m参数)`

```ruby
[root@CentOS6 ~]# free -m     # 等同于:# /usr/bin/free 
             total       used       free     shared    buffers     cached
Mem:          1861        104       1757          0       0          5
-/+ buffers/cache:         98       1762
Swap:         2047          0       2047

从上可得知：
服务器的内存是：1861M
其中使用104M，98M是真正使用的，还有5M是cached
```

使用dd命令创建一个50M的文件试试：
```ruby
[root@CentOS6 ~]# dd if=/dev/zero of=test.log bs=1M count=50

[root@CentOS6 ~]# ls -lh | grep test.log
-rw-r--r--. 1 root root  50M Nov 10 18:39 test.log
```

再看一下内存信息：
```
[root@CentOS6 ~]# free -m   
             total       used       free     shared    buffers     cached
Mem:          1861        156       1705          0       0          55
-/+ buffers/cache:        100       1761
Swap:         2047          0       2047

可以看到 cached 的内容变成了 55M（多出来的 50M 就是我们刚刚创建出来的文件大小），其他的信息都没有变化
```
现在这部分文件都在内存里，我们读取的话就会很快
```ruby
[root@CentOS6 ~]# time cat test.log >/dev/null

real	0m0.119s
user	0m0.001s
sys	    0m0.026s
```
手动清理一下cached：
```ruby
[root@CentOS6 ~]# echo 1 > /proc/sys/vm/drop_caches
```
再读取一下刚才创建的文件：
```ruby
[root@CentOS6 ~]# time cat test.log >/dev/null

real	0m1.068s
user	0m0.003s
sys	    0m0.114s

时间已经从0.119s变成了1.068s，速度慢了很多
```
在看一下内存，发现内存又恢复到了原样
```ruby
[root@CentOS6 ~]# free -m     # 等同于:# /usr/bin/free 
             total       used       free     shared    buffers     cached
Mem:          1861        104       1757          0       0          5
-/+ buffers/cache:         98       1762
Swap:         2047          0       2047

ceched可能会有变比5M小点，可能其他的cache内容被刚才的命令也一同清理了
```

#### 问题：如果我们创建的文件大小超过了物理内存，情况会怎么样呢？

做同样的实验，因为内存只有1861M，那就创建一个2000M的文件：
```ruby
[root@CentOS6 ~]# dd if=/dev/zero of=test.log bs=1M count=2000
```
然后来看一下内存：
```ruby
[root@CentOS6 ~]# free -m     # 等同于:# /usr/bin/free 
             total       used       free     shared    buffers     cached
Mem:          1861       1778        83          0       0          1631
-/+ buffers/cache:        146       1714
Swap:         2047          0       2047

cached变成了1631M，并没有出现异常，这是因为系统会自动清理没有用的cache来给后面的应用；   
【注意】不管怎样，cache并不会把内存占满

```

当然，并不是所有cache都可以释放，也包含不能释放的page cache，比如shared memory segments、tmpfs、ramfs，而available才真正表明系统目前可以提供给应用程序使用的内存。


### CentOS7

```ruby
[root@CentOS7 ~]# uname -a
Linux CentOS7 3.10.0-514.el7.x86_64 #1 SMP Tue Nov 22 16:42:41 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
```
```ruby
[root@CentOS7 ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           1823         118        1528         8         176         1524
Swap:          2047           0        2047

```
查看free帮助手册：
```ruby
[root@CentOS7 ~]# man free

DESCRIPTION
       free  displays the total amount of free and used physical and swap memory in the system, as well as the buffers and caches used by the kernel. The information is gathered by pars‐
       ing /proc/meminfo. The displayed columns are:

       total  Total installed memory (MemTotal and SwapTotal in /proc/meminfo)

       used   Used memory (calculated as total - free - buffers - cache)

       free   Unused memory (MemFree and SwapFree in /proc/meminfo)

       shared Memory used (mostly) by tmpfs (Shmem in /proc/meminfo, available on kernels 2.6.32, displayed as zero if not available)

       buffers
              Memory used by kernel buffers (Buffers in /proc/meminfo)

       cache  Memory used by the page cache and slabs (Cached and Slab in /proc/meminfo)

       buff/cache
              Sum of buffers and cache

       available
              Estimation of how much memory is available for starting new applications, without swapping. Unlike the data provided by the cache or free  fields,  this  field  takes  into
              account  page  cache  and  also that not all reclaimable memory slabs will be reclaimed due to items being in use (MemAvailable in /proc/meminfo, available on kernels 3.14,
              emulated on kernels 2.6.27+, otherwise the same as free)

```

