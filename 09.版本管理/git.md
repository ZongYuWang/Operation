## Git的部署(Windows环境下)、Git结合Github：

### 1、Git的主要功能：版本控制
在我们平时使用的软件中，在软件升级之后，你用的就是新版本的软件。你应该见过这样的版本号：`v2.0` 或者 `1712`（表示发布时为17年12月）   
那么如果你修改并保存了一个文件，从版本管理的角度来说，你得到的是这个文件的新版本。
可是很多情况下，这种修改是不可逆的。你修改完之后，无法回到你修改前的样子。为了避免这种情况，有的人会把新版本的内容保存到一个新的文件里面
由于 Git 更多地用于代码管理，举个程序员的例子。比如以下是计算机专业学生的作业：


图片
这样存储多个文件夹，可能会造成混乱。你可能想保存以前写的代码，因为它们可能在以后会用到。但是更多的时候是，你不知道各个文件夹都做了什么修改。
这时候你需要一款软件帮你管理版本，它就是Git。

### 2、个人本地使用：

|行为|命令|备注
|------|-------|------- 
|初始化	|init|	在本地的当前目录里初始化git仓库
|       |clone 地址|	从网络上某个地址拷贝仓库(repository)到本地
|查看当前状态  |	status	|查看当前仓库的状态。碰到问题不知道怎么办的时候，可以通过看它给出的提示来解决问题
|查看不同	|diff	|查看当前状态和最新的commit之间不同的地方
|           |diff 版本号1 版本号2	|查看两个指定的版本之间不同的地方。这里的版本号指的是commit的hash值
|添加文件|	add -A	|这算是相当通用的了。在commit之前要先add
|撤回stage的东西	|checkout -- .	|这里用小数点表示撤回所有修改，在--的前后都有空格
|提交|	commit -m "提交信息"|	提交信息最好能体现更改了什么
|删除未tracked|	clean -xf	|删除当前目录下所有没有track过的文件。不管它是否是.gitignore文件里面指定的文件夹和文件
|查看提交记录	|log|	查看当前版本及之前的commit记录
|	|reflog	|HEAD的变更记录
|版本回退|	reset --hard 版本号	|回退到指定版本号的版本，该版本之后的修改都被删除。同时也是通过这个命令回到最新版本。需要reflog配合



### 4、安装：
https://git-for-windows.github.io/
`Git for Windows从2.8.0版本开始，默认添加环境变量，所以环境变量部分就不用再手动配置了`

### 5、运行 git init 来初始化仓库
```ruby

Administrator@wanzy-PC MINGW64 /d (master)
$ cd D:/Git_Project

Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git init
Initialized empty Git repository in D:/Git_Project/.git/

```
图

### 6、文件的添加和提交
在这个文件夹里面创建了一个 demo1.txt 的文件(里面的内容是:世界，你好！)
使用 git status 来查看有什么变化：
```ruby
Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        demo1.txt

nothing added to commit but untracked files present (use "git add" to track)

# 说明：有一个还未追踪的文件，并提示我可以使用 git add <file>... 把它加进去
```
使用git add -A命令：
```ruby
Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git add -A


Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

        new file:   demo1.txt

# 说明：说明add成功。再看看它的提示 Changes to be committed ，也就是说现在可以执行commit了
```
执行git commit -m "说明信息"将文件提交到repository里。提交信息用英文的双引号括起来：
```ruby

Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git commit -m "upload20171123V1"
[master (root-commit) c9edd8a] upload20171123V1
 1 file changed, 1 insertion(+)
 create mode 100644 demo1.txt

```
运行git log就可以看到提交的记录了:
```ruby
Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git log
commit c9edd8ae002768b199333b974139b2b5977c57a7 (HEAD -> master)
Author: WangZongYu <774503166@qq.com>
Date:   Thu Nov 23 15:34:45 2017 +0800

    upload20171123V1

```
`为什么要有一个add，直接commit不就行了？这是因为stage有很多用处，具体可以去查找相关资料`
### 7、文件的修改：
接着我修改文件内容,内容修改为（世界，大家好！）,使用git status看看有什么变化：

```ruby
Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   demo1.txt

no changes added to commit (use "git add" and/or "git commit -a")

# 说明：比较一下就会看到，之前的是添加新文件，当时文件还没被追踪（untracked），而这次是更改已经追踪（tracked）的文件
```
通过git看看文件做了哪些变化，执行 git diff ：
```ruby
Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git diff
diff --git a/demo1.txt b/demo1.txt
index e95eda7..e08adec 100644
--- a/demo1.txt
+++ b/demo1.txt
@@ -1 +1 @@
-<CA><C0><BD>磬<C4><E3><BA>ã<A1>
\ No newline at end of file
+<CA><C0><BD>磬<B4><F3><BC>Һã<A1>
\ No newline at end of file

# 它默认跟最新的一个commit进行比较。
# 红色（前面有减号-）表示删除，绿色（前面有加号+）表示添加。
# 因此，在git看来，我们是删除了原来那一行，并添加了新的两行。这在文件内容特别多的时候效果比较明显
# 如果你用 windows 创建 txt 文件，并用自带文本编辑器来编辑文本，得到的编码是 GBK 。而 Git 读取文件时，使用 UTF-8 无 ROM 编码。因此会出现中文无法正常显示的情况
```
假如我现在想撤销这些更改，执行git checkout -- .：
```ruby
Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git checkout -- .

```
再执行git status看看：
```ruby
Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git status
On branch master
nothing to commit, working tree clean

# 在看文件内容，果然恢复到以前的状态
```
再次修改一下，使用git add -A和git commit -m "将[你]改为[大家]"
```ruby
Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git add -A

Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git commit -m "将[你]改为[大家]"
[master 1a71f27] 将[你]改为[大家]
 1 file changed, 1 insertion(+), 1 deletion(-)

Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git log
commit 1a71f27512971c7ac47045918bad1857c34c3f04 (HEAD -> master)
Author: WangZongYu <774503166@qq.com>
Date:   Thu Nov 23 15:59:44 2017 +0800

    将[你]改为[大家]

commit c9edd8ae002768b199333b974139b2b5977c57a7
Author: WangZongYu <774503166@qq.com>
Date:   Thu Nov 23 15:34:45 2017 +0800

    upload20171123V1

```
### 8、版本回退:
还是以上面提交的文件为例(里面的内容是:世界，大家好！)现在试着将文件回退到第一个commit时的状态

从刚才的 git log查看：
```ruby
Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git log
commit 1a71f27512971c7ac47045918bad1857c34c3f04 (HEAD -> master)
Author: WangZongYu <774503166@qq.com>
Date:   Thu Nov 23 15:59:44 2017 +0800

    将[你]改为[大家]

commit c9edd8ae002768b199333b974139b2b5977c57a7
Author: WangZongYu <774503166@qq.com>
Date:   Thu Nov 23 15:34:45 2017 +0800

    upload20171123V1

```
`看到以 commit 开头的，后面接着一串字符。这一串字符是16进制的数，是一串哈希值。叫它版本号就行了`
开始回退，执行git reset --hard c9edd8a（取版本号前7位就可以了）：

```ruby
Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git reset --hard c9edd8a
HEAD is now at c9edd8a upload20171123V1

# 再看文件内容，变为"世界，你好"变为了第一种状态
```
- 文件的时间问题
由于文件的修改日期是由windows修改的，因为它检测到这个文件被修改了。而我们刚才从最新版本回退到现在这个版本，就像是我们手动修改了文件内容一样，事实上是由git来完成的

再看一下git log:
```ruby
Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git log
commit c9edd8ae002768b199333b974139b2b5977c57a7 (HEAD -> master)
Author: WangZongYu <774503166@qq.com>
Date:   Thu Nov 23 15:34:45 2017 +0800

    upload20171123V1

```
再次回到之前的最新版：
```ruby
Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git reflog
c9edd8a (HEAD -> master) HEAD@{0}: reset: moving to c9edd8a
1a71f27 HEAD@{1}: commit: 将[你]改为[大家]
c9edd8a (HEAD -> master) HEAD@{2}: commit (initial): upload20171123V1

```
可以看到HEAD的变化情况。
第一行表示当前HEAD所在的版本号是c9edd8a ，而之所以在这个版本号，是由于我们执行了reset命令。
看第二行，它告诉我们，这个HEAD所在的版本号是1a71f27 ，这个版本号是在执行commit之后形成的。

此时我再用一次reset，将HEAD指向 1a71f27 ， 同时查看log ：
```ruby
Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git reset --hard 1a71f27
HEAD is now at 1a71f27 将[你]改为[大家]

```
再看一下git log信息：
```ruby
Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git log
commit 1a71f27512971c7ac47045918bad1857c34c3f04 (HEAD -> master)
Author: WangZongYu <774503166@qq.com>
Date:   Thu Nov 23 15:59:44 2017 +0800

    将[你]改为[大家]

commit c9edd8ae002768b199333b974139b2b5977c57a7
Author: WangZongYu <774503166@qq.com>
Date:   Thu Nov 23 15:34:45 2017 +0800

    upload20171123V1

```

### 9、清除未追踪的文件
首先创建一个文件：test1
```ruby
Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)

        test1.txt

nothing added to commit but untracked files present (use "git add" to track)

```
使用 git clean -xf ：
```ruby
Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git clean -xf
Removing test1.txt

```
这个命令的杀伤力比较大，它删除当前目录下所有没有track过的文件。不管它是否是.gitignore文件里面指定的文件夹和文件。当然，也有杀伤力比较小的，但这里就不介绍了



### 3、个人使用远程仓库：
|行为	|命令	|备注
|------|-------|-------
|设置用户名	|config --global user.name "你的用户名"	
|设置邮箱	|config --global user.email "你的邮箱"	
|生成ssh |key	ssh-keygen -t rsa -C "你的邮箱"	|这条命令前面不用加git
|添加远程仓库	|remote add origin 你复制的地址	|设置origin
|上传并指定默认	|push -u origin master	|指定origin为默认主机，以后push默认上传到origin上
|提交到远程仓库	|push	|将当前分支增加的commit提交到远程仓库
|从远程仓库同步	|pull	|在本地版本低于远程仓库版本的时候，获取远程仓库的commit


图片

- workspace 即工作区，逻辑上是本地计算机，还没添加到repository的状态；
- staging 即版本库中的stage，是暂存区。修改已经添加进repository，但还没有作为commit提交，类似于缓存；
- Local repository 即版本库中master那个地方。到这一步才算是成功生成一个新版本；
- Remote repository 则是远程仓库。用来将本地仓库上传到网络，可以用于备份、共享、合作。本文将使用Github作为远程仓库的例子。


- 本地配置用户名和邮箱（如果已经设置好，跳过该步）：
```ruby

Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git config --global user.name "wangzy711"

Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git config --global user.email "691793599@qq.com"

```
或者你直接在config文件里改，位置在 C:\Users\你的用户名\.gitconfig ,如下图所示，添加相应信息：
```ruby
[filter "lfs"]
	clean = git-lfs clean -- %f
	smudge = git-lfs smudge -- %f
	process = git-lfs filter-process
	required = true
[user]
	name = wangzy711
	email = 691793599@qq.com
[core]
	quotepath = false
```
- 生成ssh key：
运行 ssh-keygen -t rsa -C "你的邮箱" ，它会有三次等待你输入，直接回车即可
```ruby
Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ ssh-keygen -t rsa -C "691793599@qq.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/Administrator/.ssh/id_rsa):
Created directory '/c/Users/Administrator/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/Administrator/.ssh/id_rsa.
Your public key has been saved in /c/Users/Administrator/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:Ztd99sLngtzr7+CLj1UOZdBHqgMbbEvhM1hksrolLuY 691793599@qq.com
The key's randomart image is:
+---[RSA 2048]----+
|        ..=   .o.|
|         O .   oo|
|        o X   . +|
|       . o O o o |
|      o S + + o +|
|     . * .   o *.|
|    o o    . o= +|
|   o .      o=o= |
|    E       oo**+|
+----[SHA256]-----+

```
- 到上图提示的路径里去打开文件id_rsa.pub并复制生成的ssh key复制到剪贴板
- 打开Github，进入Settings，点击左边的 SSH and GPG keys ，点击new SSH key，将ssh key粘贴到右边的Key里面。Title随便命名即可，最后点击下面的 Add SSH key 就添加成功了

测试：
```ruby
Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ ssh -T git@github.com
The authenticity of host 'github.com (192.30.255.112)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'github.com,192.30.255.112' (RSA) to the list of known hosts.
Hi wangzy711! You've successfully authenticated, but GitHub does not provide shell access.

# 对于 oschina 的 “码云” ，执行 ssh -T git@git.oschina.net
# 对于 coding 的 “码市” ，执行 ssh -T git@git.coding.net
```

创建远程仓库并与本地关联：
创建远程仓库：
首先是在右上角+号，选择New repository创建界面  -> 接着输入远程仓库名 —> 点击 Create repository 就创建好了（其他选项可以暂时不管）

将远程仓库和本地仓库关联起来

先到Github上复制远程仓库的SSH地址：
```ruby
git@github.com:wangzy711/gitlearning.git

# 有两种方式可以关联，一种是SSH，一种是HTTPS。由于HTTPS比较慢，所以推荐使用SSH。
# 注意:SSH的地址格式是这样开头的： git@github.com
```
运行 git remote add origin 你复制的地址 ：
```ruby
Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git remote add origin git@github.com:wangzy711/gitlearning.git

```
如果你在创建 repository 的时候，加入了 README.md 或者 LICENSE ，那么 github 会拒绝你的 push ,你需要先执行 git pull origin master。

执行 git push -u origin master 将本地仓库上传至Github的仓库并进行关联：
```ruby
Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git push -u origin master
Counting objects: 6, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (6/6), 481 bytes | 160.00 KiB/s, done.
Total 6 (delta 0), reused 0 (delta 0)
To github.com:wangzy711/gitlearning.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.

```
关联已经完成！

以后想在commit后同步到Github上，只要直接执行 git push 就可以
```ruby
Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git add -A

Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git commit -m "添加file：gittest"
[master ea64663] 添加file：gittest
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 gittest.txt

Administrator@wanzy-PC MINGW64 /d/Git_Project (master)
$ git push
Warning: Permanently added the RSA host key for IP address '192.30.255.113' to the list of known hosts.
Counting objects: 3, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 292 bytes | 97.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To github.com:wangzy711/gitlearning.git
   1a71f27..ea64663  master -> master

```