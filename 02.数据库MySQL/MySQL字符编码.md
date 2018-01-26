## MySQL字符编码

不乱码的思想：建议中英文环境选择UTF8
`连接MySQL服务器的软件，如Xshell或者SecureCRT软件使用的编码`

### 1、设置Linux系统的字符集：
```ruby
[root@mysql ~]# cat /etc/sysconfig/i18n 
LANG="en_US.UTF-8"
SYSFONT="latarcyrheb-sun16"
```
### 2、设置MySQL客户端的字符集：
#### 2.1 临时修改(退出mysql客户端再重新登录则修改失效)：

这种方法可以确保插入后不出险乱码，但是对插入之前的乱码无效

- ##### 方法一：
```ruby
MariaDB [(none)]> set names utf8;

影响变量：
MariaDB [(none)]> show variables like 'character_set%';
+--------------------------+----------------------------------+
| Variable_name            | Value                            |
+--------------------------+----------------------------------+
| character_set_client     | utf8                             |
| character_set_connection | utf8                             |
| character_set_results    | utf8                             |
+--------------------------+----------------------------------+
```
- ##### 方法二：
```ruby
[root@mysql ~]# mysql -u system -pwangzongyu --default-character-set=utf8

应用场景：
[root@mysql ~]# mysql -usystem -pwangzongyu --default-character-set=utf8 < /tmp/BABYDB.sql

// 如果在建库建表时使用了gbk，并且生成了备份文件.sql，那么通过--default-character-set=utf8指定后，也不会改变原来库和表的编码
```

#### 2.2 永久修改：
更改my.cnf客户端模块的参数，修改完是不需要重启数据库的，并且永久生效
```ruby
[root@mysql ~]# vim /etc/my.cnf 
[client]
port            = 3306
socket          = /tmp/mysql.sock
default-character-set=utf8

```

### 3、设置MySQL服务端的字符集：
设置生效后，创建数据库和表默认都是这个设置的字符集，MySQL5.1和MySQL5.5的服务端字符集参数不同
```ruby
[root@mysql ~]# vim /etc/my.cnf 
[mysqld]
port            = 3306
socket          = /tmp/mysql.sock
#default-character-set=gbk  // 适合5.1及以前的版本
character-set-server=gbk    // 适合5.5

[root@mysql ~]# service mysqld restart

影响变量：
MariaDB [(none)]> show variables like 'character_set%';
+--------------------------+----------------------------------+
| Variable_name            | Value                            |
+--------------------------+----------------------------------+
| character_set_database   | utf8                             |
| character_set_server     | utf8                             |
+--------------------------+----------------------------------+

```

### 4、设置MySQL库的字符集：
```ruby
MariaDB [(none)]> create database if not exists BabyDB DEFAULT CHARSET utf8 COLLATE utf8_general_ci; 

MariaDB [(none)]> show create database BabyDB;
+----------+-----------------------------------------------------------------+
| Database | Create Database                                                 |
+----------+-----------------------------------------------------------------+
| BabyDB   | CREATE DATABASE `BabyDB` /*!40100 DEFAULT CHARACTER SET utf8 */ |
+----------+-----------------------------------------------------------------+
1 row in set (0.00 sec)

```

### 5、设置MySQL表的字符集：
```ruby

create table `student`(
`id` int(4) not null,
`name` char(20) not null,
`age` tinyint(2) not null default '0',
`dept` varchar(16) default null
) ENGINE=InnoDB DEFAULT CHARSET=utf8;


MariaDB [(none)]> show create table babydb.student\G;
*************************** 1. row ***************************
       Table: student
Create Table: CREATE TABLE `student` (
  `Snum` int(10) NOT NULL COMMENT '学号',
  `Sname` varchar(16) NOT NULL COMMENT '姓名',
  `Ssex` char(2) NOT NULL COMMENT '性别',
  `Sage` tinyint(2) NOT NULL DEFAULT '0' COMMENT '学生年龄',
  `Sdept` varchar(16) DEFAULT NULL COMMENT '学生所在系别',
  PRIMARY KEY (`Snum`),
  KEY `index_Sname` (`Sname`),
  KEY `Sdept_index` (`Sdept`) USING HASH,
  KEY `a_index` (`Sage`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.00 sec)

```


`注意：插入中文最好在文本文件中提前输入好，然后复制进MySQL客户端，否则敲的时候可能会乱码；`


### 6、查看MySQL字符集(必须都一致才不会乱码):
```ruby
MariaDB [(none)]> show variables like 'character_set%';
+--------------------------+----------------------------------------------------+
| Variable_name            | Value                                              |
+--------------------------+----------------------------------------------------+
| character_set_client     | utf8    #客户端字符集                               |
| character_set_connection | utf8    #连接字符集                                 |
| character_set_database   | utf8    #数据库字符集，配置文件指定或建库建表指定       |
| character_set_filesystem | binary                                             |
| character_set_results    | utf8    #返回结果字符集                             |
| character_set_server     | utf8    #服务器字符集，配置文件指定或建库建表指定      |                 
| character_set_system     | utf8                                               |
| character_sets_dir       | /usr/local/mysql/share/charsets/                   | 
+--------------------------+----------------------------------------------------+
8 rows in set (0.04 sec)

```

### 7、查看当前MySQL系统支持的字符集：
`第三列是校对规则`
```ruby

[root@mysql ~]# mysql -usystem -p"wangzongyu" -e "SHOW CHARACTER SET;"
+----------+-----------------------------+---------------------+--------+
| Charset  | Description                 | Default collation   | Maxlen |
+----------+-----------------------------+---------------------+--------+
| big5     | Big5 Traditional Chinese    | big5_chinese_ci     |      2 |
| dec8     | DEC West European           | dec8_swedish_ci     |      1 |
| cp850    | DOS West European           | cp850_general_ci    |      1 |
| hp8      | HP West European            | hp8_english_ci      |      1 |
| koi8r    | KOI8-R Relcom Russian       | koi8r_general_ci    |      1 |
| latin1   | cp1252 West European        | latin1_swedish_ci   |      1 |
| latin2   | ISO 8859-2 Central European | latin2_general_ci   |      1 |
| swe7     | 7bit Swedish                | swe7_swedish_ci     |      1 |
| ascii    | US ASCII                    | ascii_general_ci    |      1 |
| ujis     | EUC-JP Japanese             | ujis_japanese_ci    |      3 |
| sjis     | Shift-JIS Japanese          | sjis_japanese_ci    |      2 |
| hebrew   | ISO 8859-8 Hebrew           | hebrew_general_ci   |      1 |
| tis620   | TIS620 Thai                 | tis620_thai_ci      |      1 |
| euckr    | EUC-KR Korean               | euckr_korean_ci     |      2 |
| koi8u    | KOI8-U Ukrainian            | koi8u_general_ci    |      1 |
| gb2312   | GB2312 Simplified Chinese   | gb2312_chinese_ci   |      2 |
| greek    | ISO 8859-7 Greek            | greek_general_ci    |      1 |
| cp1250   | Windows Central European    | cp1250_general_ci   |      1 |
| gbk      | GBK Simplified Chinese      | gbk_chinese_ci      |      2 |
| latin5   | ISO 8859-9 Turkish          | latin5_turkish_ci   |      1 |
| armscii8 | ARMSCII-8 Armenian          | armscii8_general_ci |      1 |
| utf8     | UTF-8 Unicode               | utf8_general_ci     |      3 |
| ucs2     | UCS-2 Unicode               | ucs2_general_ci     |      2 |
| cp866    | DOS Russian                 | cp866_general_ci    |      1 |
| keybcs2  | DOS Kamenicky Czech-Slovak  | keybcs2_general_ci  |      1 |
| macce    | Mac Central European        | macce_general_ci    |      1 |
| macroman | Mac West European           | macroman_general_ci |      1 |
| cp852    | DOS Central European        | cp852_general_ci    |      1 |
| latin7   | ISO 8859-13 Baltic          | latin7_general_ci   |      1 |
| utf8mb4  | UTF-8 Unicode               | utf8mb4_general_ci  |      4 |
| cp1251   | Windows Cyrillic            | cp1251_general_ci   |      1 |
| utf16    | UTF-16 Unicode              | utf16_general_ci    |      4 |
| cp1256   | Windows Arabic              | cp1256_general_ci   |      1 |
| cp1257   | Windows Baltic              | cp1257_general_ci   |      1 |
| utf32    | UTF-32 Unicode              | utf32_general_ci    |      4 |
| binary   | Binary pseudo charset       | binary              |      1 |
| geostd8  | GEOSTD8 Georgian            | geostd8_general_ci  |      1 |
| cp932    | SJIS for Windows Japanese   | cp932_japanese_ci   |      2 |
| eucjpms  | UJIS for Windows Japanese   | eucjpms_japanese_ci |      3 |
+----------+-----------------------------+---------------------+--------+

```

### 8、如何更改生产MySQL数据库库表的字符集：

对于已有的数据库想修改字符集不能直接通过"alter database character set"或"alter table tablename character set"，这两个命令都没有更改已有记录的字符集，而只是对新创建的表或者记录生效。老的数据字符集还是保持原来的不变。     

下面模拟将GBK字符集的数据库修改成UTF8字符集的实际过程：

#### 8.1 导出表结构：
```ruby
[root@mysql ~]# mysqldump -usystem -p --default-character-set=utf8 -d BABYDB >BABY.sql

```
#### 8.2 修改备份的表结构的字符编码：
```ruby
[root@mysql ~]# sed -i s#gbk#utf8#g BABY.sql
DROP TABLE IF EXISTS `student`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `student` (
  `id` int(4) NOT NULL,
  `name` char(20) NOT NULL,
  `age` tinyint(2) NOT NULL DEFAULT '0',
  `dept` varchar(16) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

```
#### 8.3 确保数据库不再更新，导出所有数据：
```ruby
[root@mysql ~]# mysqldump -usystem -p --quick --no-create-info --extended-insert --default-character-set=gbk BABYDB > alldata.sql

```
#### 8.4 修改备份的表数据的字符编码：
```ruby
[root@mysql ~]# sed -i s#gbk#utf8#g alldata.sql
```
#### 8.5 创建utf8库：
`把原来的库连带表一起都删除，导入新的库和表`
```ruby
MariaDB [(none)]> create database BABYDB_utf8 default charset utf8;
```
#### 8.6 创建表：
`将上面备份的数据sql、并且已经修改好的字符编码文件导入`
```ruby
[root@mysql ~]# mysql -usystem -p BABYDB_utf8 < BABY.sql
```
#### 8.7 导入数据：
```ruby
[root@mysql ~]# mysql -usystem -p BABYDB_utf8 < alldata.sql
```