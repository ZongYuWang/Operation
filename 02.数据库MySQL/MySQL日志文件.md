## MySQL日志文件

### 7、mysqlbinlog参数：
#### 7.1 -d参数(拆库):
```ruby
[root@MySQlL1-Master ~]# /usr/local/mysql/bin/mysqlbinlog -d babyshen mysql-bin.000005 > babyshen.sql

```
```ruby
[root@master ~]# mysql -uroot -p < all.sql

// 这样可能会报错，因为不分库的恢复，可能其他库正常，然后all.sql再向正常的库里面插入东西可能就报错

```
#### 7.2 -r参数
```ruby
-r参数类似于重定向
```
#### 7.3 --start-position/--stop-position参数：
`按照位置点截取`
```ruby
[root@MySQlL1-Master ~]# /usr/local/mysql/bin/mysqlbinlog /mysqlbinlogs/mysql-bin.000010 --start-position=2310 --stop-position=2417 -r pos.sql
```
```ruby
[root@MySQlL1-Master ~]# grep insert pos.sql
/*!40019 SET @@session.max_insert_delayed_threads=0*/;
insert into mytb values('10','myproxy10')

```

#### 7.4 --start-position/--stop-position参数：
`按照时间截取`
```ruby
[root@MySQlL1-Master ~]# /usr/local/mysql/bin/mysqlbinlog /mysqlbinlogs/mysql-bin.000010 --start-datetime='2018-02-17 14:10:30' --stop-datetime='2018-02-19 17:24:16'  -r time.sql

// 时间可能不是特别准确，因为同一时间可能有多条语句
```
```ruby
[root@master ~]# grep insert time.sql 
/*!40019 SET @@session.max_insert_delayed_threads=0*/;
insert into mytb values('10','myproxy10')

```