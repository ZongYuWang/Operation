## MySQL全量增量备份

```ruby
[root@mysql ~]# mysqldump -usystem -pwangzongyu -B babydb > /opt/student.sql
[root@mysql ~]# grep -E -v "#|\/|^$|--" /opt/student.sql

错误的备份命令：
[root@mysql ~]# mysqldump -usystem -pwangzongyu -A -B babydb > /opt/student.sql
//-A是备份所有的库，后面不能再指定库，再指定某个库babydb就引起了冲突
```

mysqldump命令使用-F参数后，会刷新binlog
`由于切换Binlog导致show master status位置变化无影响`
```

```