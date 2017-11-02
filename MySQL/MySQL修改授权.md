## MySQL修改授权

##### 改表法：
```ruby
mysql> use mysql;
Database changed
mysql>update user set host = '%' where user = 'root';
mysql>select host, user from user;
```
##### 授权法：
```ruby
mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;
【说明】*.*第一个*可以改为库名，表示授权可以连接哪个库
mysql>FLUSH   PRIVILEGES;
```