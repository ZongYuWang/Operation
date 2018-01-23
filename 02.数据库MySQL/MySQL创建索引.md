## MySQL索引类型
### 1、B+Tree索引：
- BTREE索引就是一种将索引值按一定的算法，存入一个树形的数据结构中，BTREE在MyISAM里的形式和Innodb稍有不同；
- InnoDB：一种是Cluster形式的主键索引（Primary Key），另外一种则是和其他存储引擎（如MyISAM 存储引擎）存放形式基本相同的普通 B-Tree索引，这种索引在Innodb存储引擎中被称为Secondary Index

```ruby
MariaDB [babydb]> show index from student\G;
*************************** 1. row ***************************
        Table: student
   Non_unique: 0
     Key_name: PRIMARY
 Seq_in_index: 1
  Column_name: Snum
    Collation: A
  Cardinality: 6
     Sub_part: NULL
       Packed: NULL
         Null: 
   Index_type: BTREE  //InnoDB默认使用BTree索引
      Comment: 
Index_comment: 

```

Primary Key方式：
![](https://github.com/ZongYuWang/image/blob/master/MySQL1.png)


上述查找过程详解：

每个节点占用一个盘块的磁盘空间，一个节点上有两个升序排序(17到35)的关键字和三个指向子树根节点的指针；
两个关键词划分成的三个范围域对应三个指针指向的子树的数据的范围域；
以根节点为例，关键字为17和35，P1指针指向的子树的数据范围为小于17，P2指针指向的子树的数据范围为17~35，P3指针指向的子树的数据范围为大于35；
如果正好查找where id=17，那么17下面的data正好被取出来；

然后针对上图模拟下 where id=29的具体过程：（首先mysql读取数据是以块（page）为单位的）。
首先根据根节点找到磁盘块1，读入内存; // 磁盘I/O操作第1次
比较关键字29在区间（17,35），找到磁盘块1的指针P2。
根据P2指针找到磁盘块3，读入内存;  // 磁盘I/O操作第2次
比较关键字29在区间（26,30），找到磁盘块3的指针P2。
根据P2指针找到磁盘块8，读入内存  // 磁盘I/O操作第3次

在磁盘块8中的关键字列表中找到关键字29。

// 分析上面过程，发现需要3次磁盘I/O操作，和3次内存查找操作。由于内存中的关键字是一个有序表结构，可以利用二分法查找提高效率。而3次磁盘I/O操作是影响整个B-Tree查找效率的决定因素。



- MyISAM:同Innodb中的Secondary Index方式
![](https://github.com/ZongYuWang/image/blob/master/MySQL2.png)


在 Innodb 中如果通过主键来访问数据效率是非常高的，而如果是通过Secondary Index来访问数据的话，Innodb 首先通过Secondary Index的相关信息，通过相应的索引键检索到 Leaf Node之后，需要再通过Leaf Node 中存放的主键值再通过主键索引来获取相应的数据行。MyISAM存储引擎的主键索引和非主键索引差别很小，只不过是主键索引的索引键是一个唯一且非空的键而已。而且 MyISAM 存储引擎的索引和Innodb 的 Secondary Index 的存储结构也基本相同，主要的区别只是 MyISAM 存储引擎在 Leaf Nodes上面除了存放索引键信息之外，还存放能直接定位到MyISAM数据文件中相应的数据行的信息（如 Row Number），但并不会存放主键的键值信息


#### 1.1 适用场景：
- ##### 全值匹配：精确某个值，“JinJiao King”；
- ##### 匹配最左前缀：只精确匹配起头部分 “Jin%”；
- ##### 匹配范围值：
- ##### 精确匹配某一列并范围匹配另一列：例如查找名字是“jerry”，并且年龄大于20的
- ##### 只访问索引的查询
   
####  1.2 不适合使用B-Tree索引的场景：
- ##### 如果不从最左列开始，索引无效；
- ##### 不能跳过索引中的列；
- ##### 如果查询中某个列是为范围查询，那么其右侧的列都无法再使用索引优化查询；             
   
### 2、Hash索引：
- Hash索引：基于哈希表实现，特别适用于精确匹配索引中的所有列，只有Memory存储引擎支持显式hash索引；

#### 2.1 适用场景：
- ##### 只支持等值比较查询，包括=，IN()，<=>，不能使用范围查询
由于 Hash 索引比较的是进行 Hash 运算之后的 Hash值，所以它只能用于等值的过滤，不能用于基于范围的过滤，因为经过相应的 Hash算法处理之后的 Hash 值的大小关系，并不能保证和Hash运算前完全一样；
- ##### Hash 索引无法被用来避免数据的排序操作
由于 Hash 索引中存放的是经过 Hash 计算之后的 Hash值，而且Hash值的大小关系并不一定和 Hash运算前的键值完全一样，所以数据库无法利用索引的数据来避免任何排序运算；
- ##### Hash索引不能利用部分索引键查询
对于组合索引，Hash 索引在计算 Hash 值的时候是组合索引键合并后再一起计算 Hash 值，而不是单独计算 Hash值，所以不能使用单独的某一个或某几个索引键进行查询，Hash 索引也无法被利用；
- ##### Hash索引在任何时候都不能避免表扫描
Hash 索引是将索引键通过 Hash 运算之后，将 Hash运算结果的 Hash值和所对应的行指针信息存放于一个 Hash 表中，由于不同索引键存在相同 Hash 值，所以即使取满足某个 Hash 键值的数据的记录条数，也无法从 Hash索引中直接完成查询，还是要通过访问表中的实际数据进行相应的比较，并得到相应的结果；
- ##### Hash索引遇到大量Hash值相等的情况后性能并不一定就会比B-Tree索引高
对于选择性比较低的索引键，如果创建 Hash 索引，那么将会存在大量记录指针信息存于同一个 Hash 值相关联。这样要定位某一条记录时就会非常麻烦，会浪费多次表数据的访问，而造成整体性能低下


如当我们为某一列或某几列建立hash索引时（目前就只有MEMORY引擎显式地支持这种索引），会在硬盘上生成类似如下的文件：

| hash值 | 存储地址 |
| ------------- |:---------:| 
| 1db54bc745a1 | 77#45b5 | 
| 4bca452157d4 | 76#4556,77#45cc… | 

......

hash值即为通过特定算法由指定列数据计算出来，磁盘地址即为所在数据行存储在硬盘上的地址（也有可能是其他存储地址，其实MEMORY会将hash表导入内存）；
这样，当我们进行WHERE age = 18 时，会将18通过相同的算法计算出一个hash值->在hash表中找到对应的储存地址->根据存储地址取得数据；
所以，每次查询时都要遍历hash表，直到找到对应的hash值，如上面的解释中`“Hash索引在任何时候都不能避免表扫描”`，数据量大了之后，hash表也会变得庞大起来，性能下降，遍历耗时增加，如上面的解释中`“Hash索引遇到大量Hash值相等的情况后性能并不一定就会比B-Tree索引高”`

#### 2.2 不适合使用hash索引的场景：
- ##### 存储的不是值的顺序，因此，不适用于顺序查询；
- ##### 不支持模糊匹配；
   
### 3、空间索引（R-Tree）：
MyISAM支持空间索引；

### 4、全文索引（FULLTEXT）：



## MySQL创建索引方法
索引就像书的目录一样，如果在字段上建立了索引，那么以索引列为查询条件时可以加快查询数据的速度，这是MySQL优化的重要内容；

查询数据库，按主键查询时最快的，每个表只能有一个主键列，但是可以有多个普通索引列，主键列要求列的所有内容必须唯一，而索引列不要求内容必须唯一；主键就类似学生在学校的学号一样，班级内是唯一的，主键值在表内都是唯一的；无论建立主键索引还是普通索引，都要在表的对应列上创建，可以对单列创建索引，也可以对多列创建索引；
### 1、在建表时创建索引

`在唯一值多的列上建立索引查询效率高`
```ruby
create table student_index(
id int(4) not null AUTO_INCREMENT,
name char(20) not null,
age tinyint(2) not null default '0',
dept varchar(16) default null,
primary key(id),
key index_name (name)
);

// primary key(id)主键索引，设置为自增的字段必须设置为主键；
// key index_name(name) name字段普通索引;

MariaDB [babydb]> desc student_index;
+-------+-------------+------+-----+---------+----------------+
| Field | Type        | Null | Key | Default | Extra          |
+-------+-------------+------+-----+---------+----------------+
| id    | int(4)      | NO   | PRI | NULL    | auto_increment |
| name  | char(20)    | NO   | MUL | NULL    |                |
| age   | tinyint(2)  | NO   |     | 0       |                |
| dept  | varchar(16) | YES  |     | NULL    |                |
+-------+-------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)

```
`设为自增的字段必须是主键,如果再将另外一个字段设置为主键则会报错`

### 2、建表后可以通过alter命令创建/修改/删除索引
```ruby
原始表：
create table student(
id int(4) not null,
name char(20) not null,
age tinyint(2) not null default '0',
dept varchar(16) default null
);

MariaDB [babydb]> desc student;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(4)      | NO   |     | NULL    |       |
| name  | char(20)    | NO   |     | NULL    |       |
| age   | tinyint(2)  | NO   |     | 0       |       |
| dept  | varchar(16) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
4 rows in set (0.00 sec)

```
#### 2.1 添加主键(PRIMARY KEY)索引：
```ruby
语法：
ALTER TABLE tb_name ADD [CONSTRAINT [symbol]] PRIMARY KEY
        [index_type] (index_col_name,...) [index_option] ...
        
通过添加方式：
MariaDB [babydb]> alter table student add primary key(id);
```
#### 2.2 添加唯一(UNIQUE)索引(非主键)：
```ruby
ALTER TABLE tb_name ADD [CONSTRAINT [symbol]]
        UNIQUE [INDEX|KEY] [index_name]
        [index_type] (index_col_name,...) [index_option] ...

MariaDB [babydb]> alter table student add unique(age);
MariaDB [babydb]> desc student;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(4)      | NO   |     | NULL    |       |
| name  | char(20)    | NO   |     | NULL    |       |
| age   | tinyint(2)  | NO   | PRI | 0       |       |
| dept  | varchar(16) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
4 rows in set (0.00 sec)

此时Key为PRI，如果再将id设置为主键，alter table student add primary key(id);，那么age将会变化：

MariaDB [babydb]> desc student;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(4)      | NO   | PRI | NULL    |       |
| name  | char(20)    | NO   |     | NULL    |       |
| age   | tinyint(2)  | NO   | UNI | 0       |       |
| dept  | varchar(16) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+

```

#### 2.3 添加普通(INDEX)索引：
```ruby
语法：
ALTER TABLE tbl_name ADD {INDEX|KEY} [index_name]
        [index_type] (index_col_name,...) [index_option] ...

MariaDB [babydb]> alter table student add index index_name(name);

MariaDB [babydb]> desc student;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(4)      | NO   |     | NULL    |       |
| name  | char(20)    | NO   | MUL | NULL    |       |
| age   | tinyint(2)  | NO   |     | 0       |       |
| dept  | varchar(16) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
4 rows in set (0.00 sec)

```

#### 2.4 添加全文(FULLTEXT)索引:
```ruby
MyISAM引擎支持全文索引
```
#### 2.5 添加联合索引:
`按条件查询数据时，联合索引是有前缀生效特性的，index(a,b,c)仅a,ab,abc三个查询条件列可以走索引；b,bc,ac,c等无法使用索引`
```ruby
MariaDB [babydb]> alter table student add index index_name(id,name,age);

MariaDB [babydb]> desc student;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(4)      | NO   | MUL | NULL    |       |
| name  | char(20)    | NO   |     | NULL    |       |
| age   | tinyint(2)  | NO   |     | 0       |       |
| dept  | varchar(16) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
4 rows in set (0.00 sec)

```
#### 2.6 修改索引：
```ruby
语法：
ALTER TABLE tb_name CHANGE [COLUMN] old_col_name new_col_name column_definition [FIRST|AFTER col_name]

MariaDB [babydb]> alter table student change id id int primary key auto_increment;


MariaDB [babydb]> desc student;
+-------+-------------+------+-----+---------+----------------+
| Field | Type        | Null | Key | Default | Extra          |
+-------+-------------+------+-----+---------+----------------+
| id    | int(11)     | NO   | PRI | NULL    | auto_increment |
| name  | char(20)    | NO   |     | NULL    |                |
| age   | tinyint(2)  | NO   |     | 0       |                |
| dept  | varchar(16) | YES  |     | NULL    |                |
+-------+-------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)
```

#### 2.7 删除索引:
```ruby
主键列是不能重复创建索引的，如果已经存在索引，必须先删除(不推荐此种操作):

删除主键索引：
MariaDB [babydb]> alter table student drop primary key;

删除唯一键索引：
MariaDB [babydb]> alter table student drop index age;

```
#### 2.8 查看索引：
```ruby
MariaDB [babydb]> show index from table_name;
MariaDB [babydb]> show keys from table_name;

```

### 3、通过create创建索引
#### 3.1 对字段的前n个字符创建普通索引：
```ruby
当遇到表中比较大的列时，列内容的前n个字符在所有内容中已经接近唯一时，
这时可以对列的前n个字符创建索引，而无需对整个列建立索引，
这样可以节省创建索引占用的系统空间，以及降低读取和更新维护索引消耗的系统资源

MariaDB [babydb]> create index index_name on student(name(8));

// index_name 是自己命名的，可以任意命名
```
#### 3.2 创建唯一索引(非主键)：
```ruby
MariaDB [babydb]> create unique index uni_index_name on student(name);

```

#### 3.3 为表的多个字段创建爱你联合索引(同使用alter命令)：
```ruby
如果查询数据的条件是多列的，那么可以为多个查询的列创建联合索引，甚至可以为多列的前n个字符列创建联合索引

MariaDB [babydb]> create index name_dept on student(name,dept);
MariaDB [babydb]> desc student;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(4)      | NO   |     | NULL    |       |
| name  | char(20)    | NO   | MUL | NULL    |       |
| age   | tinyint(2)  | NO   |     | 0       |       |
| dept  | varchar(16) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
4 rows in set (0.00 sec)

MariaDB [babydb]> create index name_dept on student(name(8),dept(10));
//name列的前8个字符，dept列的前10个字符
```

### 4、索引列的创建及生效条件：
`既然索引可以加快查询速度，那么就给所有的列创建索引吧？`
索引不但占用系统空间，更新数据库时还需要维护索引数据的，因此，索引是一把双刃剑，并不是越多越好，例如：数十到几百行的小表上无需建立索引，更新频繁，读取比较少的业务要少建立索引。

`到底在哪些列上创建索引呢？`
select user,host from mysql.user where host=...，索引一定要创建在条件列，而不是select后的选择数据的列，另外要尽量选择在唯一值多的大表上建立索引。

### 5、使用explain查询select查询语句执行计划
```ruby
create table student(
Snum int(10) NOT NULL COMMENT '学号',
Sname varchar(16) NOT NULL COMMENT '姓名',
Ssex char(2) NOT NULL COMMENT '性别',
Sage tinyint(2) NOT NULL default '0' COMMENT '学生年龄',
Sdept varchar(16) default NULL COMMENT '学生所在系别',
PRIMARY KEY (Snum),
key index_Sname(Sname)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;


MariaDB [babydb]> create index index_name on student(Sname);
MariaDB [babydb]> explain select * from student where Sname='神童'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: student
         type: ref
possible_keys: index_Sname,index_name
          key: index_Sname
      key_len: 50
          ref: const
         rows: 1
        Extra: Using index condition
1 row in set (0.03 sec)

```
`输出详解：`
```ruby

id： 当前查询语句中，每个SELECT语句的编号；
select_type: 简单子查询、用于FROM中的子查询、联合查询(UNION)； //UNION查询的分析结果会出现一个额外匿名临时表
      简单查询为SIMPLE
      复杂查询：
         SUBQUERY：简单子查询；
         DERIVED：用于FROM中的子查询；
         UNION：UNION语句的第一个之后的SELECT语句；
         UNION RESULT：匿名临时表；
        
table：SELECT语句关联到的表；
type：关联类型，或访问类型，即MySQL决定的如何去查询表中的行的方式：
      ALL：全表扫描；
      index：根据索引的次序进行全表扫描；如果在Extra列出现“Using index”表示使用了覆盖索引，而非全表扫描；
      range：有范围限制的根据索引实现范围扫描，扫描位置始于索引中的某一点，结束于另一点；
      ref：根据索引返回表中匹配某单个值的所有行；
      eq_ref：仅返回一个行，但需要额外 与某个参考值做比较（就是会多一步比较的操作，不好实验演示）；
      const，system：直接返回单个行；
      // const > eq_ref > ref > fulltext > range > index > All
      // const代表一次就命中,ALL代表扫描了全表才确定结果。一般来说,得保证查询至少达到range级别,最好能达到ref

possible_keys：指出MySQL能使用哪个索引在该表中找到行,如果是空的,表示没有相关的索引，这时要提高性能,可通过检验WHERE子句看是否引用某些字段,或者检查字段不是适合索引；
key：实际使用到的索引。如果为NULL,则没有使用索引，如果为primary的话,表示使用了主键；
key_len：在索引使用的字节数(是在创建表的时候指定的上面key输出的列的大小)；如果键是NULL,长度就是NULL。在不损失精确性的情况下长度越短越好；
ref：显示哪个字段或常数与key一起被使用；
rows：MySQL估计为找所有的目标行而需要读取的行数，是一个估计值不是准确值，（我MySQL为了查找目标行 估计总共需要查找10行，MySQL的估计值）；
Extra：额外信息
      Using index：MySQL将会使用覆盖索引，为避免访问表；
      Using where：MySQL服务器将在存储引擎层 索引出来数据以后，再进行一次过滤；
      Using temporary：MySQL对结果排序时会使用临时表；
      Using filesort：对结果使用一个外部索引排序；
```

` select_type是SUBQUERY举例：`
```ruby
MariaDB [babydb]> explain select Sname,Sage from student where Sage>(select avg(Sage) from student);
+------+-------------+---------+------+---------------+------+---------+------+------+-------------+
| id   | select_type | table   | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+------+-------------+---------+------+---------------+------+---------+------+------+-------------+
|    1 | PRIMARY     | student | ALL  | NULL          | NULL | NULL    | NULL |    6 | Using where |
|    2 | SUBQUERY    | student | ALL  | NULL          | NULL | NULL    | NULL |    6 |             |
+------+-------------+---------+------+---------------+------+---------+------+------+-------------+
2 rows in set (0.02 sec)

```

` select_type是UNION/UNION RESULT的举例：`
```ruby
MariaDB [babydb]> explain select Sname from student UNION select Cname from course;
+------+--------------+------------+-------+---------------+-------------+---------+------+------+-------------+
| id   | select_type  | table      | type  | possible_keys | key         | key_len | ref  | rows | Extra       |
+------+--------------+------------+-------+---------------+-------------+---------+------+------+-------------+
|    1 | PRIMARY      | student    | index | NULL          | index_Sname | 50      | NULL |    6 | Using index |
|    2 | UNION        | course     | ALL   | NULL          | NULL        | NULL    | NULL |    5 |             |
| NULL | UNION RESULT | <union1,2> | ALL   | NULL          | NULL        | NULL    | NULL | NULL |             |
+------+--------------+------------+-------+---------------+-------------+---------+------+------+-------------+
3 rows in set (0.00 sec)

```

`type类型是const的举例(where后面的条件时主键)：`
```ruby
MariaDB [babydb]> explain select Sname from student where Snum=4;
+------+-------------+---------+-------+---------------+---------+---------+-------+------+-------+
| id   | select_type | table   | type  | possible_keys | key     | key_len | ref   | rows | Extra |
+------+-------------+---------+-------+---------------+---------+---------+-------+------+-------+
|    1 | SIMPLE      | student | const | PRIMARY       | PRIMARY | 4       | const |    1 |       |
+------+-------------+---------+-------+---------------+---------+---------+-------+------+-------+
1 row in set (0.00 sec)

```


### 5、高性能索引策略：
- 独立使用列，尽量避免其参与运算；（SELECT * FROM students WHERE Age+20>50;）
- 左前缀索引：索引构建于字段的左侧的多少个字符（可以通过左侧1个字符、也可以通过左侧5个字符构建），要通过索引选择性来评估；
- 索引选择性：不重复的索引值和数据表的记录总数的比值；（假如有N行，那么这个值是1/N到1之间，要保证较低的重复率）
- 如果是char类型而且定义了30个字符，那么可以把这30个字符都定义成索引，就不用再定义左前缀索引了
- 多列索引：AND操作时更适合使用多列索引；
- 选择合适的索引列次序：将选择性最高的放左侧；

### 6、总结：
- 要在表的列上创建索引；
- 索引会加快查询速度，但是会影响更新的速度，因为要维护索引；
- 索引不是越多越好，要在频繁查询的where后的条件列上创建索引；
- 小表或唯一值极少的列上不建索引，要在大表以及不同内容多的列上创建索引；
