## SQL语句的分类

### 1、数据查询语言(DQL)：
DQL全程Data Query Language，用以从表中获得数据，确定数据怎样再应用程序给出，SELECT是DQL用得最多的，其他的DQL有WHERE，ORDER BY，GROUP BY和HAVING，这些DQL保留字常与其他的SQL语句一起使用
```ruby
MariaDB [(none)]> select user,host,password from mysql.user order by user;

```
- 查看当前使用的数据库
```ruby
MariaDB [(none)]> select database();
+------------+
| database() |
+------------+
| NULL       |
+------------+
1 row in set (0.01 sec)

MariaDB [(none)]> use babyshen;
Database changed
MariaDB [babyshen]> select database();
+------------+
| database() |
+------------+
| babyshen   |
+------------+
1 row in set (0.00 sec)
```


### 2、数据操作语言(DML):
DML全程Data Manipulation Language，其语句包括INSERT、UPDATE、DELETE，他们分别用于添加、修改、删除表中的行(数据)。

```ruby
MariaDB [(none)]> delete from mysql.user where user='';
```

### 3、事务处理语言(TPL)：
它的语句能确保被DML语句影响的表的所有行及时得以更新，TPL语句包括BEGIN、TRANSACTION、COMMIT和ROLLBACK

### 4、数据控制语言(DCL)：
DCL全程Data Control Language，它的语句通过GRANT或REVOKE获得许可；

### 5、数据定义语言(DDL)：
DDL全程Data Definition Language，其语句包括CREATE和DROP，在数据库中创建新表或删除表，为表加入索引等；