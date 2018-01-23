## MySQL备份

```ruby
[root@mysql ~]# mysqldump -usystem -pwangzongyu -B babydb > /opt/student.sql
[root@mysql ~]# grep -E -v "#|\/|^$|--" /opt/student.sql

错误的备份命令：
[root@mysql ~]# mysqldump -usystem -pwangzongyu -A -B babydb > /opt/student.sql
//-A是备份所有的库，后面不能再指定库，再指定某个库babydb就引起了冲突
```