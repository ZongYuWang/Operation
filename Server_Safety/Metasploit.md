#### 安装Metasploit：

**安装依赖包：**
```py
[root@localhost ~]# yum update
[root@localhost ~]# yum upgrade


[root@localhost ~]# yum groupinstall 'Development Tools'
[root@localhost ~]# yum install sqlite-devel libxslt-devel libxml2-devel java-1.7.0-openjdk libpcap-devel nano openssl-devel zlib-devel libffi-devel gdbm-devel readline-devel nano wget
```
**安装Ruby 1.9.3：**
```py
[root@localhost ~]# cd /usr/src/
[root@localhost src]# wget http://pyyaml.org/download/libyaml/yaml-0.1.4.tar.gz
[root@localhost src]# tar xvf yaml-0.1.4.tar.gz 
[root@localhost src]# cd yaml-0.1.4
[root@localhost yaml-0.1.4]# ./configure --prefix=/usr/local/
[root@localhost yaml-0.1.4]# make && make install 


[root@localhost yaml-0.1.4]# cd /usr/src/
[root@localhost src]# wget http://ftp.ruby-lang.org/pub/ruby/1.9/ruby-1.9.3-p374.tar.gz
[root@localhost src]# tar xvf ruby-1.9.3-p374.tar.gz 
[root@localhost src]# cd ruby-1.9.3-p374
[root@localhost ruby-1.9.3-p374]# ./configure --prefix=/usr/local --with-opt-dir=/usr/local/lib64/

make会报错，需要用下面的方式解决：
[root@localhost ruby-1.9.3-p374]# wget https://bugs.ruby-lang.org/attachments/download/3707/out.patch
# find . -name ossl_pkey_ec.c
./ext/openssl/ossl_pkey_ec.c
[root@localhost ruby-1.9.3-p374]# patch ./ext/openssl/ossl_pkey_ec.c < out.patch
patching file ./ext/openssl/ossl_pkey_ec.c
Hunk #1 succeeded at 757 (offset -5 lines).
Hunk #2 succeeded at 814 (offset -5 lines).
patching file ./ext/openssl/ossl_pkey_ec.c
Hunk #1 FAILED at 7.
1 out of 1 hunk FAILED -- saving rejects to file ./ext/openssl/ossl_pkey_ec.c.rej


[root@localhost ruby-1.9.3-p374]# make clean all
[root@localhost ruby-1.9.3-p374]# make && make install

```

**安装Nmap：**
```py
[root@localhost ruby-1.9.3-p374]# cd /usr/src/
[root@localhost src]# svn co https://svn.nmap.org/nmap
[root@localhost src]# cd nmap/
[root@localhost nmap]# ./configure 
[root@localhost nmap]# make && make install && make clean
```
**Configuring Postgre SQL Server:**
```py
[root@localhost nmap]# cd /usr/src/
[root@localhost src]# wget http://yum.postgresql.org/9.2/redhat/rhel-6-x86_64/pgdg-centos92-9.2-8.noarch.rpm
[root@localhost src]# rpm -ivh pgdg-centos92-9.2-8.noarch.rpm

[root@localhost yum.repos.d]# sed -i 's/https/http/g' /etc/yum.repos.d/pgdg-92-centos.repo 
[root@localhost src]# yum update
[root@localhost yum.repos.d]# yum install postgresql92-server postgresql92-devel postgresql92


[root@localhost ~]# service postgresql-9.2 initdb
Initializing database:                                     [  OK  ]
[root@localhost ~]# service postgresql-9.2 start
Starting postgresql-9.2 service:                           [  OK  ]
[root@localhost ~]# chkconfig postgresql-9.2 on


[root@localhost ~]# echo export PATH=/usr/pgsql-9.2/bin:\$PATH >> /etc/bashrc
[root@localhost ~]# source ~/.bashrc


[root@localhost ~]# su - postgres
-bash-4.1$ createuser msf -P -S -R -D
Enter password for new role: wangzongyu
Enter it again: wangzongyu
-bash-4.1$ createdb -O msf msf
-bash-4.1$ exit
logout


[root@localhost ~]# vim /var/lib/pgsql/9.2/data/pg_hba.conf
local   msf msf md5
host  msf   msf 127.0.0.1/8 md5
host  msf   msf ::1/128 md5
[root@localhost ~]# service postgresql-9.2 start
Starting postgresql-9.2 service:                           [  OK  ]
```

** 安装Metasploit Framework：**
```py
[root@localhost ~]# gem install wirble pg sqlite3 msgpack activerecord redcarpet rspec simplecov yard bundler

[root@localhost ~]# cd /opt/
[root@localhost opt]# git clone https://github.com/rapid7/metasploit-framework.git  
Initialized empty Git repository in /opt/metasploit-framework/.git/
remote: Counting objects: 423789, done.
remote: Compressing objects: 100% (37/37), done.
remote: Total 423789 (delta 20), reused 22 (delta 8), pack-reused 423744
Receiving objects: 100% (423789/423789), 319.30 MiB | 117 KiB/s, done.
Resolving deltas: 100% (309353/309353), done.
[root@localhost opt]# cd metasploit-framework/

[root@localhost metasploit-framework]# bash -c 'for MSF in $(ls msf*); do ln -s /opt/metasploit-framework/$MSF /usr/local/bin/$MSF;done'
[root@localhost metasploit-framework]# ln -s /opt/metasploit-framework/armitage /usr/local/bin/armitage

[root@localhost metasploit-framework]# useradd wangzy
[root@localhost metasploit-framework]# passwd wangzy
New password: wangzy

[root@localhost metasploit-framework]# gem update --system
[root@localhost metasploit-framework]# chmod 775 -R /opt/metasploit-framework/

[root@localhost metasploit-framework]# vim /opt/metasploit-framework/Gemfile
source 'https://rubygems.org'
修改为：
source 'http://rubygems.org'

[root@localhost metasploit-framework]# su - wangzy
[wangzy@localhost ~]$ cd /opt/metasploit-framework/
[wangzy@localhost metasploit-framework]$ bundle install --path vendor/bundle

```
`会出现这个错误:`
```py
Using rack-test 0.6.3
Gem::InstallError: public_suffix requires Ruby version >= 2.1.
An error occurred while installing public_suffix (3.0.0), and Bundler cannot continue.
Make sure that `gem install public_suffix -v '3.0.0'` succeeds before bundling.

In Gemfile:
  metasploit-framework was resolved to 4.16.12, which depends on
    octokit was resolved to 4.7.0, which depends on
      sawyer was resolved to 0.8.1, which depends on
        addressable was resolved to 2.5.2, which depends on
          public_suffix

```
`解决办法：`
```py
[root@localhost metasploit-framework]# chmod 777 -R /usr/local/lib/ruby/gems/1.9.1


[wangzy@localhost metasploit-framework]$ bundle install 

还是报错，未能解决，待后续解决
```

nano /opt/metasploit-framework/database.yml
参考链接：http://darkoperator.squarespace.com/msf-centosrhel/