## Mysql系列相关面试题（汇总版）

### 一、数据库基础知识

**1、为什么要使用数据库**

这个问题其实就是反着来回答，意思是不使用数据库的缺点。

**（1）数据保存在内存**

优点：存取速度快

缺点：数据不能永久保存

**（2）数据保存在文件**

优点：数据永久保存

缺点：1）速度比内存操作慢，频繁的IO操作。2）查询数据不方便

**（3）数据保存在数据库**

1）数据永久保存

2）使用SQL语句，查询方便效率高。

3）管理数据方便

**2、什么是SQL？什么是MySQL?**

结构化查询语言(Structured Query Language)简称SQL，是一种数据库查询语言。

MySQL是一个关系型数据库管理系统，由瑞典MySQL AB 公司开发，属于 Oracle 旗下产品。MySQL 是最流行的关系型数据库管理系统之一，在 WEB 应用方面，MySQL是最好的 RDBMS (Relational Database Management System，关系数据库管理系统) 应用软件之一。在Java企业级开发中非常常用，因为 MySQL 是开源免费的，并且方便扩展。

**3、数据库三大范式是什么？**

第一范式：每个列都不可以再拆分，原子数据项。 系中包括系名 系主任可以分割

第二范式：在第一范式的基础上，非主键列完全依赖于主键，而不能是依赖于主键的一部分。消除部分依赖，

第三范式：在第二范式的基础上，非主键列只依赖于主键，不依赖于其他非主键。消除传递依赖

**4、mysql有关权限的表都有哪几个**

MySQL服务器通过权限表来控制用户对数据库的访问，权限表存放在mysql数据库里，由mysql_install_db脚本初始化。这些权限表分别user，db，table_priv，columns_priv和host。下面分别介绍一下这些表的结构和内容：

（1）user权限表：记录允许连接到服务器的用户帐号信息，里面的权限是全局级的。

（2）db权限表：记录各个帐号在各个数据库上的操作权限。

（3）table_priv权限表：记录数据表级的操作权限。

（4）columns_priv权限表：记录数据列级的操作权限。

（5）host权限表：配合db权限表对给定主机上数据库级操作权限作更细致的控制。这个权限表不受GRANT和REVOKE语句的影响。

**5、MySQL的binlog有有几种录入格式？分别有什么区别？**

有三种格式，statement，row和mixed。

（1）statement模式下，**每一条会修改数据的sql**都会记录在binlog中。不需要记录每一行的变化，减少了binlog日志量，节约了IO，提高性能。由于sql的执行是有上下文的，因此在保存的时候需要保存相关的信息，同时还有一些使用了函数之类的语句无法被记录复制。

（2）row级别下，不记录sql语句上下文相关信息，**仅保存哪条记录被修改**，修改成了什么样子了。记录单元为每一行的改动，基本是可以全部记下来但是由于很多操作，会导致大量行的改动(比如alter table)，因此这种模式的文件保存的信息太多，日志量太大。

（3）mixed，一种折中的方案，普通操作使用statement记录，当无法使用statement的时候使用row。

此外，新版的MySQL中对row级别也做了一些优化，当表结构发生变化的时候，会记录语句而不是逐行记录。

### 二、存储引擎（重点）

**1、MySQL存储引擎MyISAM与InnoDB区别？**

存储引擎Storage engine：MySQL中的数据、索引以及其他对象是如何存储的，是一套文件系统的实现。

常用的存储引擎有以下：

（1）Innodb引擎：Innodb引擎提供了对数据库ACID事务的支持。并且还提供了行级锁和外键的约束。它的设计的目标就是处理大数据容量的数据库系统。

（2）MyIASM引擎(原本Mysql的默认引擎)：不提供事务的支持，也不支持行级锁和外键。

（3）MEMORY引擎：所有的数据都在内存中，数据的处理速度快，但是安全性不高。

MyISAM与InnoDB区别

![img](https://mmbiz.qpic.cn/mmbiz_png/N8scgexEBuLYOiaUBNfLEyvwYunlXrUzAjUCy3EicD8g4l3bBAlNx1ibAtjWd3RpquicZ9MByCooRtCmvJZRa7Idqg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**2、MyISAM索引与InnoDB索引的区别？**

（1）InnoDB索引是聚簇索引，MyISAM索引是非聚簇索引。

（2）InnoDB的主键索引的叶子节点存储着行数据，因此主键索引非常高效。

（3）MyISAM索引的叶子节点存储的是行数据地址，需要再寻址一次才能得到数据。

（4）InnoDB非主键索引的叶子节点存储的是主键和其他带索引的列数据，因此查询时做到覆盖索引会非常高效。

**3、InnoDB引擎的4大特性**

（1）插入缓冲（insert buffer)

（2）二次写(double write)

（3）自适应哈希索引(ahi)

（4）预读(read ahead)

**4、如何选择存储引擎**

如果没有特别的需求，使用默认的`Innodb`即可。

MyISAM：以读写插入为主的应用程序，比如博客系统、新闻门户网站。

Innodb：更新（删除）操作频率也高，或者要保证数据的完整性；并发量高，支持事务和外键。比如OA自动化办公系统。

### 三、索引

**1、什么是索引？**

索引是一种特殊的文件(InnoDB数据表上的索引是表空间的一个组成部分)，它们包含着对数据表里所有记录的引用指针。

索引是一种数据结构。数据库索引，是数据库管理系统中一个排序的数据结构，以协助快速查询、更新数据库表中数据。索引的实现通常使用B树及其变种B+树。

更通俗的说，索引就相当于目录。为了方便查找书中的内容，通过对内容建立索引形成目录。索引是一个文件，它是要占据物理空间的。

**2、索引有哪些优缺点？**

（1）索引的优点

可以大大加快数据的检索速度，这也是创建索引的最主要的原因。
通过使用索引，可以在查询的过程中，使用优化隐藏器，提高系统的性能。

（2）索引的缺点

时间方面：创建索引和维护索引要耗费时间，具体地，当对表中的数据进行增加、删除和修改的时候，索引也要动态的维护，会降低增/改/删的执行效率；

空间方面：索引需要占物理空间。

**3、索引使用场景（重点）**

**（1）where**

![img](https://mmbiz.qpic.cn/mmbiz_jpg/N8scgexEBuLYOiaUBNfLEyvwYunlXrUzAc9smPytR2vsdnn9qibucwxicgXCeEaNHhe9GcbMpicT9AUdNobVmXcnPg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





上图中，根据id查询记录，因为id字段仅建立了主键索引，因此此SQL执行可选的索引只有主键索引，如果有多个，最终会选一个较优的作为检索的依据。

```
-- 增加一个没有建立索引的字段
alter table innodb1 add sex char(1);
-- 按sex检索时可选的索引为null
EXPLAIN SELECT * from innodb1 where sex='男';
```

**（1）order by**

当我们使用order by将查询结果按照某个字段排序时，如果该字段没有建立索引，那么执行计划会将查询出的所有数据使用外部排序（将数据从硬盘分批读取到内存使用内部排序，最后合并排序结果），这个操作是很影响性能的，因为需要将查询涉及到的所有数据从磁盘中读到内存（如果单条数据过大或者数据量过多都会降低效率），更无论读到内存之后的排序了。

但是如果我们对该字段建立索引alter table 表名 add index(字段名)，那么由于索引本身是有序的，因此直接按照索引的顺序和映射关系逐条取出数据即可。而且如果分页的，那么只用取出索引表某个范围内的索引对应的数据，而不用像上述那取出所有数据进行排序再返回某个范围内的数据。（从磁盘取数据是最影响性能的）

**（3）join**

对join语句匹配关系（on）涉及的字段建立索引能够提高效率

**（4）索引覆盖**

如果要查询的字段都建立过索引，那么引擎会直接在索引表中查询而不会访问原始数据（否则只要有一个字段没有建立索引就会做全表扫描），这叫索引覆盖。因此我们需要尽可能的在select后只写必要的查询字段，以增加索引覆盖的几率。

这里值得注意的是不要想着为每个字段建立索引，因为优先使用索引的优势就在于其体积小。

**4、索引有哪几种类型？**

（1）主键索引: 数据列不允许重复，不允许为NULL，一个表只能有一个主键。

（2）唯一索引: 数据列不允许重复，允许为NULL值，一个表允许多个列创建唯一索引。

可以通过 ALTER TABLE table_name ADD UNIQUE (column); 创建唯一索引

可以通过 ALTER TABLE table_name ADD UNIQUE (column1,column2); 创建唯一组合索引

（3）普通索引: 基本的索引类型，没有唯一性的限制，允许为NULL值。

可以通过ALTER TABLE table_name ADD INDEX index_name (column);创建普通索引

可以通过ALTER TABLE table_name ADD INDEX index_name(column1, column2, column3);创建组合索引

（4）全文索引：是目前搜索引擎使用的一种关键技术。

可以通过ALTER TABLE table_name ADD FULLTEXT (column);创建全文索引

**5、索引的数据结构是什么？（b树，hash）**

在MySQL中使用较多的索引有**Hash索引**，**B+树索引**等，而我们经常使用的InnoDB存储引擎的默认索引实现为：B+树索引。对于哈希索引来说，底层的数据结构就是哈希表，因此在绝大多数需求为单条记录查询的时候，可以选择哈希索引，查询性能最快；其余大部分场景，建议选择BTree索引。

**6、索引的基本原理是什么？**

索引用来快速地寻找那些具有特定值的记录。如果没有索引，一般来说执行查询时遍历整张表。索引的原理很简单，就是把无序的数据变成有序的查询：

（1）把创建了索引的列的内容进行排序

（2）对排序结果生成倒排表

（3）在倒排表内容上拼上数据地址链

（4）在查询的时候，先拿到倒排表内容，再取出数据地址链，从而拿到具体数据

**7、索引算法有哪些？**

索引算法有 BTree算法和Hash算法

（1）BTree算法

BTree是最常用的mysql数据库索引算法，也是mysql默认的算法。因为它不仅可以被用在=,>,>=,<,<=和between这些比较操作符上，而且还可以用于like操作符，只要它的查询条件是一个不以通配符开头的常量， 例如：

```
-- 只要它的查询条件是一个不以通配符开头的常量
select * from user where name like 'jack%'; 
-- 如果一通配符开头，或者没有使用常量，则不会使用索引，例如： 
select * from user where name like '%jack'; 
```

（2）Hash算法

Hash Hash索引只能用于对等比较，例如=,<=>（相当于=）操作符。由于是一次定位数据，不像BTree索引需要从根节点到枝节点，最后才能访问到页节点这样多次IO访问，所以检索效率远高于BTree索引。

**8、索引设计的原则？**

（1）适合索引的列是出现在where子句中的列，或者连接子句中指定的列

（2）基数较小的类，索引效果较差，没有必要在此列建立索引

（3）使用短索引，如果对长字符串列进行索引，应该指定一个前缀长度，这样能够节省大量索引空间

（4）不要过度索引。索引需要额外的磁盘空间，并降低写操作的性能。在修改表内容的时候，索引会进行更新甚至重构，索引列越多，这个时间就会越长。所以只保持需要的索引有利于查询即可。

**9、创建索引的原则（重中之重）**

索引虽好，但也不是无限制的使用，最好符合一下几个原则

1） 最左前缀匹配原则，组合索引非常重要的原则，mysql会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配，比如a = 1 and b = 2 and c > 3 and d = 4 如果建立(a,b,c,d)顺序的索引，d是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整。

2）较频繁作为查询条件的字段才去创建索引

3）更新频繁字段不适合创建索引

4）若是不能有效区分数据的列不适合做索引列(如性别，男女未知，最多也就三种，区分度实在太低)

5）尽量的扩展索引，不要新建索引。比如表中已经有a的索引，现在要加(a,b)的索引，那么只需要修改原来的索引即可。

6）定义有外键的数据列一定要建立索引。

7）对于那些查询中很少涉及的列，重复值比较多的列不要建立索引。

8）对于定义为text、image和bit的数据类型的列不要建立索引。

**10、如何创建索引，如何删除索引？**

（1）创建索引

第一种方式：在执行CREATE TABLE时创建索引

```
CREATE TABLE user_index2 (
    id INT auto_increment PRIMARY KEY,
    first_name VARCHAR (16),
    last_name VARCHAR (16),
    id_card VARCHAR (18),
    information text,
    KEY name (first_name, last_name),
    FULLTEXT KEY (information),
    UNIQUE KEY (id_card)
);
```

第二种方式：使用ALTER TABLE命令去增加索引

```
ALTER TABLE table_name ADD INDEX index_name (column_list);
```

第三种方式：使用CREATE INDEX命令创建

```
CREATE INDEX index_name ON table_name (column_list);
```

（2）删除索引

第一种方式：根据索引名删除普通索引、唯一索引、全文索引：alter table 表名 drop KEY 索引名

```
alter table user_index drop KEY name;
alter table user_index drop KEY id_card;
alter table user_index drop KEY information;
```

第二种方式：删除主键索引：alter table 表名 drop primary key（因为主键只有一个）。这里值得注意的是，如果主键自增长，那么不能直接执行此操作（自增长依赖于主键索引），需要取消自增长再进行删除。

**12、创建索引时需要注意什么？**

（1）非空字段：应该指定列为NOT NULL，除非你想存储NULL。在mysql中，含有空值的列很难进行查询优化，因为它们使得索引、索引的统计信息以及比较运算更加复杂。你应该用0、一个特殊的值或者一个空串代替空值；

（2）取值离散大的字段：（变量各个取值之间的差异程度）的列放到联合索引的前面，可以通过count()函数查看字段的差异值，返回值越大说明字段的唯一值越多字段的离散程度高；

（3）索引字段越小越好：数据库的数据存储以页为单位一页存储的数据越多一次IO操作获取的数据越大效率越高。

**13、使用索引查询一定能提高查询的性能吗？为什么**

通常，通过索引查询数据比全表扫描要快。但我们也要注意到它的代价。

（1）索引需要空间来存储，也需要定期维护， 每当有记录在表中增减或索引列被修改时，索引本身也会被修改。这意味着每条记录的INSERT，DELETE，UPDATE将为此多付出4，5 次的磁盘I/O。因为索引需要额外的存储空间和处理，那些不必要的索引反而会使查询反应时间变慢。

（2）使用索引查询不一定能提高查询性能，索引范围查询(INDEX RANGE SCAN)适用于两种情况:

基于一个范围的检索，一般查询返回结果集小于表中记录数的30%

基于非唯一性索引的检索

**14、百万级别或以上的数据如何删除**

我们删除数据库百万级别数据的时候，查询MySQL官方手册得知删除数据的速度和创建的索引数量是成正比的。

（1）可以先删除索引（此时大概耗时三分多钟）

（2）然后删除其中无用数据（此过程需要不到两分钟）

（3）删除完成后重新创建索引(此时数据较少了)创建索引也非常快，约十分钟左右。

与之前的直接删除绝对是要快速很多，更别说万一删除中断,一切删除会回滚。那更是坑了。

**15、什么是前缀索引？如何建立？**

语法：index(field(10))，使用字段值的前10个字符建立索引，默认是使用字段的全部内容建立索引。

前提：前缀的标识度高。比如密码就适合建立前缀索引，因为密码几乎各不相同。

实操的难度：在于前缀截取的长度。

我们可以利用select count(*)/count(distinct left(password,prefixLen));，通过从调整prefixLen的值（从1自增）查看不同前缀长度的一个平均匹配度，接近1时就可以了（表示一个密码的前prefixLen个字符几乎能确定唯一一条记录）

**16、什么是最左前缀原则？什么是最左匹配原则**

顾名思义，就是最左优先，在创建多列索引时，要根据业务需求，where子句中使用最频繁的一列放在最左边。

最左前缀匹配原则，非常重要的原则，mysql会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配，比如a = 1 and b = 2 and c > 3 and d = 4 如果建立(a,b,c,d)顺序的索引，d是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整。
=和in可以乱序，比如a = 1 and b = 2 and c = 3 建立(a,b,c)索引可以任意顺序，mysql的查询优化器会帮你优化成索引可以识别的形式

**17、B树和B+树的区别**

（1）在B树中，你可以将键和值存放在内部节点和叶子节点；但在B+树中，内部节点都是键，没有值，叶子节点同时存放键和值。

（2）B+树的叶子节点有一条链相连，而B树的叶子节点各自独立。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/N8scgexEBuLYOiaUBNfLEyvwYunlXrUzAA9Nnib3dzEUUqqqibmKsdCoLJp5ZKK70sVgUYdXqBHFfI2anjXFLUukw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**18、Hash索引和B+树所有有什么区别或者说优劣呢?**

首先要知道Hash索引和B+树索引的底层实现原理：

hash索引底层就是hash表，进行查找时，调用一次hash函数就可以获取到相应的键值，之后进行回表查询获得实际数据。B+树底层实现是多路平衡查找树。对于每一次的查询都是从根节点出发，查找到叶子节点方可以获得所查键值，然后根据查询判断是否需要回表查询数据。

那么可以看出他们有以下的不同：

（1）hash索引进行等值查询更快(一般情况下)，但是却无法进行范围查询。因为在hash索引中经过hash函数建立索引之后，索引的顺序与原顺序无法保持一致，不能支持范围查询。而B+树的的所有节点皆遵循(左节点小于父节点，右节点大于父节点，多叉树也类似)，天然支持范围。

（2）hash索引不支持使用索引进行排序，原理同上。

（3）hash索引不支持模糊查询以及多列索引的最左前缀匹配。原理也是因为hash函数的不可预测。AAAA和AAAAB的索引没有相关性。

（4）hash索引任何时候都避免不了回表查询数据，而B+树在符合某些条件(聚簇索引，覆盖索引等)的时候可以只通过索引完成查询。

（5）hash索引虽然在等值查询上较快，但是不稳定。性能不可预测，当某个键值存在大量重复的时候，发生hash碰撞，此时效率可能极差。而B+树的查询效率比较稳定，对于所有的查询都是从根节点到叶子节点，且树的高度较低。

因此，在大多数情况下，直接选择B+树索引可以获得稳定且较好的查询速度。而不需要使用hash索引。

**19、数据库为什么使用B+树而不是B树**

（1）B树只适合随机检索，而B+树同时支持随机检索和顺序检索；

（2）B+树空间利用率更高，可减少I/O次数，磁盘读写代价更低。一般来说，索引本身也很大，不可能全部存储在内存中，因此索引往往以索引文件的形式存储的磁盘上。这样的话，索引查找过程中就要产生磁盘I/O消耗。B+树的内部结点并没有指向关键字具体信息的指针，只是作为索引使用，其内部结点比B树小，盘块能容纳的结点中关键字数量更多，一次性读入内存中可以查找的关键字也就越多，相对的，IO读写次数也就降低了。而IO读写次数是影响索引检索效率的最大因素；

（3）B+树的查询效率更加稳定。B树搜索有可能会在非叶子结点结束，越靠近根节点的记录查找时间越短，只要找到关键字即可确定记录的存在，其性能等价于在关键字全集内做一次二分查找。而在B+树中，顺序检索比较明显，随机检索时，任何关键字的查找都必须走一条从根节点到叶节点的路，所有关键字的查找路径长度相同，导致每一个关键字的查询效率相当。

（4）B树在提高了磁盘IO性能的同时并没有解决元素遍历的效率低下的问题。B+树的叶子节点使用指针顺序连接在一起，只要遍历叶子节点就可以实现整棵树的遍历。而且在数据库中基于范围的查询是非常频繁的，而B树不支持这样的操作。

（5）增删文件（节点）时，效率更高。因为B+树的叶子节点包含所有关键字，并以有序的链表结构存储，这样可很好提高增删效率。

**20、什么是聚簇索引？何时使用聚簇索引与非聚簇索引**

（1）聚簇索引：将数据存储与索引放到了一块，找到索引也就找到了数据

（2）非聚簇索引：将数据存储于索引分开结构，索引结构的叶子节点指向了数据的对应行，myisam通过key_buffer把索引先缓存到内存中，当需要访问数据时（通过索引访问数据），在内存中直接搜索索引，然后通过索引找到磁盘相应数据，这也就是为什么索引不在key buffer命中时，速度慢的原因

何时使用聚簇索引与非聚簇索引：

![img](https://mmbiz.qpic.cn/mmbiz_jpg/N8scgexEBuLYOiaUBNfLEyvwYunlXrUzAMEWH7ZDF8Dmk8YV4XMwHfpMu1ianlnNZdRxjpb2X8ycqHhjWYYGzZ0g/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**21、B+树在满足聚簇索引和覆盖索引的时候为什么不需要回表查询数据？**

在B+树的索引中，叶子节点可能存储了当前的key值，也可能存储了当前的key值以及整行的数据，这就是聚簇索引和非聚簇索引。在InnoDB中，只有主键索引是聚簇索引，如果没有主键，则挑选一个唯一键建立聚簇索引。如果没有唯一键，则隐式的生成一个键来建立聚簇索引。

**当查询使用聚簇索引时，在对应的叶子节点，可以获取到整行数据，因此不用再次进行回表查询。**

**22、非聚簇索引一定会回表查询吗？**

不一定，这涉及到查询语句所要求的字段是否全部命中了索引，如果全部命中了索引，那么就不必再进行回表查询。

举个简单的例子，假设我们在员工表的年龄上建立了索引，那么当进行select age from employee where age < 20的查询时，在索引的叶子节点上，已经包含了age信息，不会再次进行回表查询。

**23、联合索引是什么？为什么需要注意联合索引中的顺序？**

MySQL可以使用多个字段同时建立一个索引，叫做联合索引。在联合索引中，如果想要命中索引，需要按照建立索引时的字段顺序挨个使用，否则无法命中索引。

具体原因为:

MySQL使用索引时需要索引有序，假设现在建立了"name，age，school"的联合索引，那么索引的排序为: 先按照name排序，如果name相同，则按照age排序，如果age的值也相等，则按照school进行排序。

当进行查询时，此时索引仅仅按照name严格有序，因此必须首先使用name字段进行等值查询，之后对于匹配到的列而言，其按照age字段严格有序，此时可以使用age字段用做索引查找，以此类推。因此在建立联合索引的时候应该注意索引列的顺序，一般情况下，将查询需求频繁或者字段选择性高的列放在前面。此外可以根据特例的查询或者表结构进行单独的调整。

**24、什么是事务的隔离级别？MySQL的默认隔离级别是什么？**

SQL 标准定义了四个隔离级别：

（1）READ-UNCOMMITTED(读取未提交)：最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读。

（2）READ-COMMITTED(读取已提交)：允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生。

（3）REPEATABLE-READ(可重复读)：对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。

（4）SERIALIZABLE(可串行化)：最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。

这里需要注意的是：Mysql 默认采用的 REPEATABLE_READ隔离级别 Oracle 默认采用的 READ_COMMITTED隔离级别

### 四、锁机制

**1、mysql中为什么要使用到锁？**

当数据库有并发事务的时候，可能会产生数据的不一致，这时候需要一些机制来保证访问的次序，锁机制就是这样的一个机制。

就像酒店的房间，如果大家随意进出，就会出现多人抢夺同一个房间的情况，而在房间上装上锁，申请到钥匙的人才可以入住并且将房间锁起来，其他人只有等他使用完毕才可以再次使用。

**2、隔离级别与锁的关系？**

（1）在Read Uncommitted级别下，读取数据不需要加共享锁，这样就不会跟被修改的数据上的排他锁冲突

（2）在Read Committed级别下，读操作需要加共享锁，但是在语句执行完以后释放共享锁；

（3）在Repeatable Read级别下，读操作需要加共享锁，但是在事务提交之前并不释放共享锁，也就是必须等待事务执行完毕以后才释放共享锁。

（4）SERIALIZABLE 是限制性最强的隔离级别，因为该级别锁定整个范围的键，并一直持有锁，直到事务完成。

**3、按照锁的粒度分数据库锁有哪些？**

在关系型数据库中，可以按照锁的粒度把数据库锁分为行级锁(INNODB引擎)、表级锁(MYISAM引擎)和页级锁(BDB引擎 )。

**MyISAM和InnoDB存储引擎使用的锁：**

（1）MyISAM采用表级锁(table-level locking)。

（2）InnoDB支持行级锁(row-level locking)和表级锁，默认为行级锁

**行级锁，表级锁和页级锁对比**

（1）行级锁：行级锁是Mysql中锁定粒度最细的一种锁，表示只针对当前操作的行进行加锁。行级锁能大大减少数据库操作的冲突。其加锁粒度最小，但加锁的开销也最大。行级锁分为共享锁 和 排他锁。

特点：开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。

（2）表级锁 表级锁是MySQL中锁定粒度最大的一种锁，表示对当前操作的整张表加锁，它实现简单，资源消耗较少，被大部分MySQL引擎支持。最常使用的MYISAM与INNODB都支持表级锁定。表级锁定分为表共享读锁（共享锁）与表独占写锁（排他锁）。

特点：开销小，加锁快；不会出现死锁；锁定粒度大，发出锁冲突的概率最高，并发度最低。

（3）页级锁 页级锁是MySQL中锁定粒度介于行级锁和表级锁中间的一种锁。表级锁速度快，但冲突多，行级冲突少，但速度慢。所以取了折衷的页级，一次锁定相邻的一组记录。

特点：开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般

**4、从锁的类别上分MySQL都有哪些锁呢？像上面那样子进行锁定岂不是有点阻碍并发效率了**

从锁的类别上来讲，有共享锁和排他锁。

（1）共享锁: 又叫做读锁。当用户要进行数据的读取时，对数据加上共享锁。共享锁可以同时加上多个。

（2）排他锁: 又叫做写锁。当用户要进行数据的写入时，对数据加上排他锁。排他锁只可以加一个，他和其他的排他锁，共享锁都相斥。

用上面的例子来说就是用户的行为有两种，一种是来看房，多个用户一起看房是可以接受的。一种是真正的入住一晚，在这期间，无论是想入住的还是想看房的都不可以。

锁的粒度取决于具体的存储引擎，InnoDB实现了行级锁，页级锁，表级锁。

他们的加锁开销从大到小，并发能力也是从大到小。

**5、MySQL中InnoDB引擎的行锁是怎么实现的？**

答：InnoDB是基于索引来完成行锁

例: select * from tab_with_index where id = 1 for update;

for update 可以根据条件来完成行锁锁定，并且 id 是有索引键的列，如果 id 不是索引键那么InnoDB将完成表锁，并发将无从谈起

**6、InnoDB存储引擎的锁的算法有什么？**

（1）Record lock：单个行记录上的锁

（2）Gap lock：间隙锁，锁定一个范围，不包括记录本身

（3）Next-key lock：record+gap 锁定一个范围，包含记录本身

相关知识点：

（1）innodb对于行的查询使用next-key lock

（2）Next-locking keying为了解决Phantom Problem幻读问题

（3）当查询的索引含有唯一属性时，将next-key lock降级为record key

（4）Gap锁设计的目的是为了阻止多个事务将记录插入到同一范围内，而这会导致幻读问题的产生

（5）有两种方式显式关闭gap锁：（除了外键约束和唯一性检查外，其余情况仅使用record lock） A. 将事务隔离级别设置为RC B. 将参数innodb_locks_unsafe_for_binlog设置为1

**7、mysql如何解决死锁？**

常见的解决死锁的方法

（1）如果不同程序会并发存取多个表，尽量约定以相同的顺序访问表，可以大大降低死锁机会。

（2）在同一个事务中，尽可能做到一次锁定所需要的所有资源，减少死锁产生概率；

（3）对于非常容易产生死锁的业务部分，可以尝试使用升级锁定颗粒度，通过表级锁定来减少死锁产生的概率；

如果业务处理不好可以用分布式事务锁或者使用乐观锁

**8、数据库的乐观锁和悲观锁是什么？怎么实现的？**

数据库管理系统（DBMS）中的并发控制的任务是确保在多个事务同时存取数据库中同一数据时不破坏事务的隔离性和统一性以及数据库的统一性。乐观并发控制（乐观锁）和悲观并发控制（悲观锁）是并发控制主要采用的技术手段。

（1）悲观锁：假定会发生并发冲突，屏蔽一切可能违反数据完整性的操作。在查询完数据的时候就把事务锁起来，直到提交事务。实现方式：使用数据库中的锁机制

（2）乐观锁：假设不会发生并发冲突，只在提交操作时检查是否违反数据完整性。在修改数据的时候把事务锁起来，通过version的方式来进行锁定。实现方式：乐一般会使用版本号机制或CAS算法实现。

两种锁的使用场景：

从上面对两种锁的介绍，我们知道两种锁各有优缺点，不可认为一种好于另一种，像乐观锁适用于写比较少的情况下（多读场景），即冲突真的很少发生的时候，这样可以省去了锁的开销，加大了系统的整个吞吐量。

但如果是多写的情况，一般会经常产生冲突，这就会导致上层应用会不断的进行retry，这样反倒是降低了性能，所以一般多写的场景下用悲观锁就比较合适。

### 五、mysql高级属性

**1、为什么要使用视图？什么是视图？**

所谓视图，本质上是一种虚拟表，在物理上是不存在的，其内容与真实的表相似，包含一系列带有名称的列和行数据。

视图使开发者只关心感兴趣的某些特定数据和所负责的特定任务，只能看到视图中所定义的数据，而不是视图所引用表中的数据，从而提高了数据库中数据的安全性。

举个例子，平时学校每个学生都有一个固定的班级，这个班级就是表，但是突然某一天学校要组织一部分学生参加一项活动，这时候从各个班级抽出几个组成了一个临时班级，这个班级就是视图。另外这个临时班级解散了，对每一个学生的原班级没什么影响。

**2、视图的使用场景有哪些？**

视图根本用途：简化sql查询，提高开发效率。如果说还有另外一个用途那就是兼容老的表结构。

下面是视图的常见使用场景：

（1）重用SQL语句；

（2）简化复杂的SQL操作。在编写查询后，可以方便的重用它而不必知道它的基本查询细节；

（3）使用表的组成部分而不是整个表；

（4）保护数据。可以给用户授予表的特定部分的访问权限而不是整个表的访问权限；

（5）更改数据格式和表示。视图可返回与底层表的表示和格式不同的数据。

**3、什么是游标？**

游标是系统为用户开设的一个数据缓冲区，存放SQL语句的执行结果，每个游标区都有一个名字。用户可以通过游标逐一获取记录并赋给主变量，交由主语言进一步处理。

**4、什么是存储过程？有哪些优缺点？**

存储过程是一个预编译的SQL语句，优点是允许模块化的设计，就是说只需要创建一次，以后在该程序中就可以调用多次。如果某次操作需要执行多次SQL，使用存储过程比单纯SQL语句执行要快。

优点

（1）存储过程是预编译过的，执行效率高。

（2）存储过程的代码直接存放于数据库中，通过存储过程名直接调用，减少网络通讯。

（3）安全性高，执行存储过程需要有一定权限的用户。

（4）存储过程可以重复使用，减少数据库开发人员的工作量。

缺点

（1）调试麻烦，但是用 PL/SQL Developer 调试很方便！弥补这个缺点。

（2）移植问题，数据库端代码当然是与数据库相关的。但是如果是做工程型项目，基本不存在移植问题。

（3）重新编译问题，因为后端代码是运行前编译的，如果带有引用关系的对象发生改变时，受影响的存储过程、包将需要重新编译（不过也可以设置成运行时刻自动编译）。

（4）如果在一个程序系统中大量的使用存储过程，到程序交付使用的时候随着用户需求的增加会导致数据结构的变化，接着就是系统的相关问题了，最后如果用户想维护该系统可以说是很难很难、而且代价是空前的，维护起来更麻烦。

**5、什么是触发器？触发器的使用场景有哪些？**

触发器是用户定义在关系表上的一类由事件驱动的特殊的存储过程。触发器是指一段代码，当触发某个事件时，自动执行这些代码。

使用场景

（1）可以通过数据库中的相关表实现级联更改。

（2）实时监控某张表中的某个字段的更改而需要做出相应的处理。例如可以生成某些业务的编号。

（3）注意不要滥用，否则会造成数据库及应用程序的维护困难。

大家需要牢记以上基础知识点，重点是理解数据类型CHAR和VARCHAR的差异，表存储引擎InnoDB和MyISAM的区别。

**6、MySQL中都有哪些触发器？**

在MySQL数据库中有如下六种触发器：

- Before Insert
- After Insert
- Before Update
- After Update
- Before Delete
- After Delete

**7、SQL语句主要分为哪几类**

（1）数据定义语言DDL（Data Ddefinition Language）CREATE，DROP，ALTER

（2）数据查询语言DQL（Data Query Language）SELECT

这个较为好理解 即查询操作，以select关键字。各种简单查询，连接查询等 都属于DQL。

（3）数据操纵语言DML（Data Manipulation Language）INSERT，UPDATE，DELETE

主要为以上操作 即对数据进行操作的，对应上面所说的查询操作 DQL与DML共同构建了多数初级程序员常用的增删改查操作。而查询是较为特殊的一种 被划分到DQL中。

（4）数据控制功能DCL（Data Control Language）GRANT，REVOKE，COMMIT，ROLLBACK

主要为以上操作 即对数据库安全性完整性等有操作的，可以简单的理解为权限控制等。

**8、超键、候选键、主键、外键分别是什么？**

（1）超键：在关系中能唯一标识元组的属性集称为关系模式的超键。一个属性可以为作为一个超键，多个属性组合在一起也可以作为一个超键。超键包含候选键和主键。

（2）候选键：是最小超键，即没有冗余元素的超键。

（3）主键：数据库表中对储存数据对象予以唯一和完整标识的数据列或属性的组合。一个数据列只能有一个主键，且主键的取值不能缺失，即不能为空值（Null）。

（4）外键：在一个表中存在的另一个表的主键称此表的外键。

**9、SQL 约束有哪几种？**

（1）NOT NULL: 用于控制字段的内容一定不能为空（NULL）。

（2）UNIQUE: 控件字段内容不能重复，一个表允许有多个 Unique 约束。

（3）PRIMARY KEY: 也是用于控件字段内容不能重复，但它在一个表只允许出现一个。

（4）FOREIGN KEY: 用于预防破坏表之间连接的动作，也能防止非法数据插入外键列，因为它必须是它指向的那个表中的值之一。

（5）CHECK: 用于控制字段的值范围。

**10、关联查询有哪几种？**

（1）交叉连接（CROSS JOIN）

```
SELECT * FROM A,B(,C)
或者
SELECT * FROM A CROSS JOIN B (CROSS JOIN C)
#没有任何关联条件，结果是笛卡尔积，结果集会很大，没有意义，很少使用
SELECT * FROM A,B WHERE A.id=B.id
或者
SELECT * FROM A INNER JOIN B ON A.id=B.id
#多表中同时符合某种条件的数据记录的集合，INNER JOIN可以缩写为JOIN
```

（2）内连接分为三类

等值连接：ON A.id=B.id

不等值连接：ON A.id > B.id

自连接：SELECT * FROM A T1 INNER JOIN A T2 ON T1.id=T2.pid

（3）外连接（LEFT JOIN/RIGHT JOIN）

左外连接：LEFT OUTER JOIN, 以左表为主，先查询出左表，按照ON后的关联条件匹配右表，没有匹配到的用NULL填充，可以简写成LEFT JOIN

右外连接：RIGHT OUTER JOIN, 以右表为主，先查询出右表，按照ON后的关联条件匹配左表，没有匹配到的用NULL填充，可以简写成RIGHT JOIN

（4）联合查询（UNION与UNION ALL）

SELECT * FROM A UNION SELECT * FROM B UNION …

就是把多个结果集集中在一起，UNION前的结果为基准，需要注意的是联合查询的列数要相等，相同的记录行会合并

如果使用UNION ALL，不会合并重复的记录行

效率 UNION 高于 UNION ALL

（5）全连接（FULL JOIN）

MySQL不支持全连接

可以使用LEFT JOIN 和UNION和RIGHT JOIN联合使用

SELECT * FROM A LEFT JOIN B ON A.id=B.id UNIONSELECT * FROM A RIGHT JOIN B ON A.id=B.id

表连接面试题

有2张表，1张R、1张S，R表有ABC三列，S表有CD两列，表中各有三条记录。

R表

A B C
a1 b1 c1
a2 b2 c2
a3 b3 c3

S表

C D
c1 d1
c2 d2
c4 d3

交叉连接(笛卡尔积):

select r.*,s.* from r,s

A B C C D
a1 b1 c1 c1 d1
a2 b2 c2 c1 d1
a3 b3 c3 c1 d1
a1 b1 c1 c2 d2
a2 b2 c2 c2 d2
a3 b3 c3 c2 d2
a1 b1 c1 c4 d3
a2 b2 c2 c4 d3
a3 b3 c3 c4 d3

内连接结果：

select r.*,s.* from r inner join s on r.c=s.c

A B C C D
a1 b1 c1 c1 d1
a2 b2 c2 c2 d2

左连接结果：

select r.*,s.* from r left join s on r.c=s.c

A B C C D
a1 b1 c1 c1 d1
a2 b2 c2 c2 d2
a3 b3 c3

右连接结果：

select r.*,s.* from r right join s on r.c=s.c

A B C C D
a1 b1 c1 c1 d1
a2 b2 c2 c2 d2
c4 d3

全表连接的结果（MySql不支持，Oracle支持）：

select r.*,s.* from r full join s on r.c=s.c

A B C C D
a1 b1 c1 c1 d1
a2 b2 c2 c2 d2
a3 b3 c3
c4 d3

### 六、子查询（主要考察的是sql语句）

**1、子查询有哪几种情况？**

（1）子查询是单行单列的情况：结果集是一个值，父查询使用：=、 <、 > 等运算符

```
-- 查询工资最高的员工是谁？ 
select  * from employee where salary=(select max(salary) from employee);   
```

（2）子查询是多行单列的情况：结果集类似于一个数组，父查询使用：in 运算符

```
-- 查询工资最高的员工是谁？ 
select  * from employee where salary=(select max(salary) from employee);    
```

（3）子查询是多行多列的情况：结果集类似于一张虚拟表，不能用于where条件，用于select子句中做为子表

```
-- 1) 查询出2011年以后入职的员工信息
-- 2) 查询所有的部门信息，与上面的虚拟表中的信息比对，找出所有部门ID相等的员工。
select * from dept d,  (select * from employee where join_date > '2011-1-1') e where e.dept_id =  d.id;    

-- 使用表连接：
select d.*, e.* from  dept d inner join employee e on d.id = e.dept_id where e.join_date >  '2011-1-1'  
```

**2、mysql中 in 和 exists 区别**

mysql中的in语句是把外表和内表作hash 连接，而exists语句是对外表作loop循环，每次loop循环再对内表进行查询。一直大家都认为exists比in语句的效率要高，这种说法其实是不准确的。这个是要区分环境的。

（1）如果查询的两个表大小相当，那么用in和exists差别不大。

（2）如果两个表中一个较小，一个是大表，则子查询表大的用exists，子查询表小的用in。

（3）not in 和not exists：如果查询语句使用了not in，那么内外表都进行全表扫描，没有用到索引；而not extsts的子查询依然能用到表上的索引。所以无论那个表大，用not exists都比not in要快。

**3、varchar与char的区别**

**char的特点**

（1）char表示定长字符串，长度是固定的；

（2）如果插入数据的长度小于char的固定长度时，则用空格填充；

（3）因为长度固定，所以存取速度要比varchar快很多，甚至能快50%，但正因为其长度固定，所以会占据多余的空间，是空间换时间的做法；

（4）对于char来说，最多能存放的字符个数为255，和编码无关

**varchar的特点**

（1）varchar表示可变长字符串，长度是可变的；

（2）插入的数据是多长，就按照多长来存储；

（3）varchar在存取方面与char相反，它存取慢，因为长度不固定，但正因如此，不占据多余的空间，是时间换空间的做法；

（4）对于varchar来说，最多能存放的字符个数为65532

总之，结合性能角度（char更快）和节省磁盘空间角度（varchar更小），具体情况还需具体来设计数据库才是妥当的做法。

**4、varchar(50)中50的涵义**

最多存放50个字符，varchar(50)和(200)存储hello所占空间一样，但后者在排序时会消耗更多内存，因为order by col采用fixed_length计算col长度(memory引擎也一样)。在早期 MySQL 版本中， 50 代表字节数，现在代表字符数。

**5、int(20)中20的涵义**

是指显示字符的长度。20表示最大显示宽度为20，但仍占4字节存储，存储范围不变；

不影响内部存储，只是影响带 zerofill 定义的 int 时，前面补多少个 0，易于报表展示

**6、 FLOAT和DOUBLE的区别是什么？**

- FLOAT类型数据可以存储至多8位十进制数，并在内存中占4字节。
- DOUBLE类型数据可以存储至多18位十进制数，并在内存中占8字节。

**7、drop、delete与truncate的区别**

![img](https://mmbiz.qpic.cn/mmbiz_png/N8scgexEBuLYOiaUBNfLEyvwYunlXrUzAEHL7BzQZdibMwCKOKgVm6hrr1lSYLiauswFtyh6gPGnh70Fer62GfxwQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





**8、UNION与UNION ALL的区别？**

- 如果使用UNION ALL，不会合并重复的记录行
- 效率 UNION 高于 UNION ALL

### 七、SQL优化（非常重要）

**1、如何定位及优化SQL语句的性能问题？创建的索引有没有被使用到?或者说怎么才可以知道这条语句运行很慢的原因？**

对于低性能的SQL语句的定位，最重要也是最有效的方法就是使用执行计划，MySQL提供了explain命令来查看语句的执行计划。我们知道，不管是哪种数据库，或者是哪种数据库引擎，在对一条SQL语句进行执行的过程中都会做很多相关的优化，对于查询语句，最重要的优化方式就是使用索引。而执行计划，就是显示数据库引擎对于SQL语句的执行的详细情况，其中包含了是否使用索引，使用什么索引，使用的索引的相关信息等。

**2、SQL的生命周期？**

（1）应用服务器与数据库服务器建立一个连接

（2）数据库进程拿到请求sql

（3）解析并生成执行计划，执行

（4）读取数据到内存并进行逻辑处理

（5）通过步骤一的连接，发送结果到客户端

（6）关掉连接，释放资源

![img](https://mmbiz.qpic.cn/mmbiz_png/N8scgexEBuLYOiaUBNfLEyvwYunlXrUzAWqwJmXyIrqibnksRQpd7w4RKe3OD3e4da4mn0ibTRicHn8xaMXzoQibpTg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**3、大表数据查询，怎么优化？**

（1）优化shema、sql语句+索引；

（2）第二加缓存，memcached, redis；

（3）主从复制，读写分离；

（4）垂直拆分，根据你模块的耦合度，将一个大的系统分为多个小的系统，也就是分布式系统；

（5）水平切分，针对数据量大的表，这一步最麻烦，最能考验技术水平，要选择一个合理的sharding key, 为了有好的查询效率，表结构也要改动，做一定的冗余，应用也要改，sql中尽量带sharding key，将数据定位到限定的表上去查，而不是扫描全部的表；

**4、超大分页怎么处理？**

超大的分页一般从两个方向上来解决.

（1）数据库层面,这也是我们主要集中关注的(虽然收效没那么大),类似于select * from table where age > 20 limit 1000000,10这种查询其实也是有可以优化的余地的. 这条语句需要load1000000数据然后基本上全部丢弃,只取10条当然比较慢. 当时我们可以修改为select * from table where id in (select id from table where age > 20 limit 1000000,10).这样虽然也load了一百万的数据,但是由于索引覆盖,要查询的所有字段都在索引中,所以速度会很快. 同时如果ID连续的好,我们还可以select * from table where id > 1000000 limit 10,效率也是不错的,优化的可能性有许多种,但是核心思想都一样,就是减少load的数据.

（2）从需求的角度减少这种请求…主要是不做类似的需求(直接跳转到几百万页之后的具体某一页.只允许逐页查看或者按照给定的路线走,这样可预测,可缓存)以及防止ID泄漏且连续被人恶意攻击.

解决超大分页,其实主要是靠缓存,可预测性的提前查到内容,缓存至redis等k-V数据库中,直接返回即可.

在阿里巴巴《Java开发手册》中,对超大分页的解决办法是类似于上面提到的第一种：

```
# 【推荐】利用延迟关联或者子查询优化超多分页场景。 
# 说明：MySQL并不是跳过offset行，而是取offset+N行，然后返回放弃前offset行，返回N行，那当offset特别大的时候，效率就非常的低下，要么控制返回的总页数，要么对超过特定阈值的页数进行SQL改写。 
# 正例：先快速定位需要获取的id段，然后再关联： 
SELECT a.* FROM 表1 a, (select id from 表1 where 条件 LIMIT 100000,20 ) b where a.id=b.id
```

**5、mysql1如何分页？**

LIMIT 子句可以被用于强制 SELECT 语句返回指定的记录数。LIMIT 接受一个或两个数字参数。参数必须是一个整数常量。如果给定两个参数，第一个参数指定第一个返回记录行的偏移量，第二个参数指定返回记录行的最大数目。初始记录行的偏移量是 0(而不是 1)

```
mysql> SELECT * FROM table LIMIT 5,10; // 检索记录行 6-15 
#为了检索从某一个偏移量到记录集的结束所有的记录行，可以指定第二个参数为 -1：
mysql> SELECT * FROM table LIMIT 95,-1; // 检索记录行 96-last. 
#如果只给定一个参数，它表示返回最大的记录行数目：
mysql> SELECT * FROM table LIMIT 5; //检索前 5 个记录行 
#换句话说，LIMIT n 等价于 LIMIT 0,n。
```

**6、说一下慢查询日志？**

开启慢查询日志

配置项：slow_query_log

可以使用show variables like ‘slov_query_log’查看是否开启，如果状态值为OFF，可以使用set GLOBAL slow_query_log = on来开启，它会在datadir下产生一个xxx-slow.log的文件。

设置临界时间

配置项：long_query_time

查看：show VARIABLES like 'long_query_time'，单位秒

设置：set long_query_time=0.5

实操时应该从长时间设置到短的时间，即将最慢的SQL优化掉

查看日志，一旦SQL超过了我们设置的临界时间就会被记录到xxx-slow.log中

**7、关心过业务系统里面的sql耗时吗？统计过慢查询吗？对慢查询都怎么优化过？**

在业务系统中，除了使用主键进行的查询，其他的我都会在测试库上测试其耗时，慢查询的统计主要由运维在做，会定期将业务中的慢查询反馈给我们。

慢查询的优化首先要搞明白慢的原因是什么？是查询条件没有命中索引？是load了不需要的数据列？还是数据量太大？

所以优化也是针对这三个方向来的，

（1）首先分析语句，看看是否load了额外的数据，可能是查询了多余的行并且抛弃掉了，可能是加载了许多结果中并不需要的列，对语句进行分析以及重写。

（2）分析语句的执行计划，然后获得其使用索引的情况，之后修改语句或者修改索引，使得语句可以尽可能的命中索引。

（3）如果对语句的优化已经无法进行，可以考虑表中的数据量是否太大，如果是的话可以进行横向或者纵向的分表。

**8、主键使用自增ID还是UUID？**

推荐使用自增ID，不要使用UUID。

因为在InnoDB存储引擎中，主键索引是作为聚簇索引存在的，也就是说，主键索引的B+树叶子节点上存储了主键索引以及全部的数据(按照顺序)，如果主键索引是自增ID，那么只需要不断向后排列即可，如果是UUID，由于到来的ID与原来的大小不确定，会造成非常多的数据插入，数据移动，然后导致产生很多的内存碎片，进而造成插入性能的下降。

总之，在数据量大一些的情况下，用自增主键性能会好一些。

关于主键是聚簇索引，如果没有主键，InnoDB会选择一个唯一键来作为聚簇索引，如果没有唯一键，会生成一个隐式的主键。

**9、字段为什么要求定义为not null？**

null值会占用更多的字节，且会在程序中造成很多与预期不符的情况。

**10、如果要存储用户的密码散列，应该使用什么字段进行存储？**

密码散列，盐，用户身份证号等固定长度的字符串应该使用char而不是varchar来存储，这样可以节省空间且提高检索效率。

**11、优化查询过程中的数据访问**

（1）访问数据太多导致查询性能下降

（2）确定应用程序是否在检索大量超过需要的数据，可能是太多行或列

（3）确认MySQL服务器是否在分析大量不必要的数据行

（4）避免犯如下SQL语句错误

（5）查询不需要的数据。解决办法：使用limit解决

（6）多表关联返回全部列。解决办法：指定列名

（7）总是返回全部列。解决办法：避免使用SELECT *

（8）重复查询相同的数据。解决办法：可以缓存数据，下次直接读取缓存

（9）是否在扫描额外的记录。解决办法：

（10）使用explain进行分析，如果发现查询需要扫描大量的数据，但只返回少数的行，可以通过如下技巧去优化：

（11）使用索引覆盖扫描，把所有的列都放到索引中，这样存储引擎不需要回表获取对应行就可以返回结果。

（12）改变数据库和表的结构，修改数据表范式

（13）重写SQL语句，让优化器可以以更优的方式执行查询。

**12、优化长难的查询语句**

一个复杂查询还是多个简单查询
MySQL内部每秒能扫描内存中上百万行数据，相比之下，响应数据给客户端就要慢得多
使用尽可能小的查询是好的，但是有时将一个大的查询分解为多个小的查询是很有必要的。
切分查询
将一个大的查询分为多个小的相同的查询
一次性删除1000万的数据要比一次删除1万，暂停一会的方案更加损耗服务器开销。
分解关联查询，让缓存的效率更高。
执行单个查询可以减少锁的竞争。
在应用层做关联更容易对数据库进行拆分。
查询效率会有大幅提升。
较少冗余记录的查询。

**13、优化特定类型的查询语句**

count(*)会忽略所有的列，直接统计所有列数，不要使用count(列名) MyISAM中，没有任何where条件的count(*)非常快。
当有where条件时，MyISAM的count统计不一定比其它引擎快。
可以使用explain查询近似值，用近似值替代count(*)
增加汇总表
使用缓存

**14、优化关联查询**

- 确定ON或者USING子句中是否有索引。
- 确保GROUP BY和ORDER BY只有一个表中的列，这样MySQL才有可能使用索引。

**15、优化子查询**

用关联查询替代
优化GROUP BY和DISTINCT
这两种查询据可以使用索引来优化，是最有效的优化方法
关联查询中，使用标识列分组的效率更高
如果不需要ORDER BY，进行GROUP BY时加ORDER BY NULL，MySQL不会再进行文件排序。
WITH ROLLUP超级聚合，可以挪到应用程序处理

**16、优化LIMIT分页**

- LIMIT偏移量大的时候，查询效率较低
- 可以记录上次查询的最大ID，下次查询时直接根据该ID来查询

**17、优化WHERE子句**

对于此类考题，先说明如何定位低效SQL语句，然后根据SQL语句可能低效的原因做排查，先从索引着手，如果索引没有问题，考虑以上几个方面，数据访问的问题，长难查询句的问题还是一些特定类型优化的问题，逐一回答。

SQL语句优化的一些方法？

1.对查询进行优化，应尽量避免全表扫描，首先应考虑在 where 及 order by 涉及的列上建立索引。
2.应尽量避免在 where 子句中对字段进行 null 值判断，否则将导致引擎放弃使用索引而进行全表扫描，如：

```
select id from t where num is null
-- 可以在num上设置默认值0，确保表中num列没有null值，然后这样查询：
select id from t where num=
```

- 3.应尽量避免在 where 子句中使用!=或<>操作符，否则引擎将放弃使用索引而进行全表扫描。
- 4.应尽量避免在 where 子句中使用or 来连接条件，否则将导致引擎放弃使用索引而进行全表扫描，如：

```
select id from t where num=10 or num=20
-- 可以这样查询：
select id from t where num=10 union all select id from t where num=20
```

5 in 和 not in 也要慎用，否则会导致全表扫描，如：

```
select id from t where num in(1,2,3) 
-- 对于连续的数值，能用 between 就不要用 in 了：
select id from t where num between 1 and 3
```

6.下面的查询也将导致全表扫描：select id from t where name like ‘%李%’若要提高效率，可以考虑全文检索。
7.如果在 where 子句中使用参数，也会导致全表扫描。因为SQL只有在运行时才会解析局部变量，但优化程序不能将访问计划的选择推迟到运行时；它必须在编译时进行选择。然 而，如果在编译时建立访问计划，变量的值还是未知的，因而无法作为索引选择的输入项。如下面语句将进行全表扫描：

```
select id from t where num=@num
-- 可以改为强制查询使用索引：
select id from t with(index(索引名)) where num=@num
```

8.应尽量避免在 where 子句中对字段进行表达式操作，这将导致引擎放弃使用索引而进行全表扫描。如：

```
select id from t where num/2=100
-- 应改为:
select id from t where num=100*2
```

9.应尽量避免在where子句中对字段进行函数操作，这将导致引擎放弃使用索引而进行全表扫描。如：

```
select id from t where substring(name,1,3)=’abc’
-- name以abc开头的id应改为:
select id from t where name like ‘abc%’
```

10、10.不要在 where 子句中的“=”左边进行函数、算术运算或其他表达式运算，否则系统将可能无法正确使用索引。

### 八、数据库设计上的优化

**1、为什么要优化**

- 系统的吞吐量瓶颈往往出现在数据库的访问速度上
- 随着应用程序的运行，数据库的中的数据会越来越多，处理时间会相应变慢
- **数据是存放在磁盘上的，读写速度无法和内存相比**

**2、数据库结构优化**

一个好的数据库设计方案对于数据库的性能往往会起到事半功倍的效果。

需要考虑数据冗余、查询和更新的速度、字段的数据类型是否合理等多方面的内容。

（1）将字段很多的表分解成多个表

对于字段较多的表，如果有些字段的使用频率很低，可以将这些字段分离出来形成新表。

因为当一个表的数据量很大时，会由于使用频率低的字段的存在而变慢。

（2）增加中间表

对于需要经常联合查询的表，可以建立中间表以提高查询效率。

通过建立中间表，将需要通过联合查询的数据插入到中间表中，然后将原来的联合查询改为对中间表的查询。

（3）增加冗余字段

设计数据表时应尽量遵循范式理论的规约，尽可能的减少冗余字段，让数据库设计看起来精致、优雅。但是，合理的加入冗余字段可以提高查询速度。

表的规范化程度越高，表和表之间的关系越多，需要连接查询的情况也就越多，性能也就越差。

注意：

冗余字段的值在一个表中修改了，就要想办法在其他表中更新，否则就会导致数据不一致的问题。

**3、MySQL数据库cpu飙升到500%的话他怎么处理？**

当 cpu 飙升到 500%时，先用操作系统命令 top 命令观察是不是 mysqld 占用导致的，如果不是，找出占用高的进程，并进行相关处理。

如果是 mysqld 造成的， show processlist，看看里面跑的 session 情况，是不是有消耗资源的 sql 在运行。找出消耗高的 sql，看看执行计划是否准确， index 是否缺失，或者实在是数据量太大造成。

一般来说，肯定要 kill 掉这些线程(同时观察 cpu 使用率是否下降)，等进行相应的调整(比如说加索引、改 sql、改内存参数)之后，再重新跑这些 SQL。

也有可能是每个 sql 消耗资源并不多，但是突然之间，有大量的 session 连进来导致 cpu 飙升，这种情况就需要跟应用一起来分析为何连接数会激增，再做出相应的调整，比如说限制连接数等

**4、大表怎么优化？某个表有近千万数据，CRUD比较慢，如何优化？分库分表了是怎么做的？分表分库了有什么问题？有用到中间件么？他们的原理知道么？**

当MySQL单表记录数过大时，数据库的CRUD性能会明显下降，一些常见的优化措施如下：

（1）限定数据的范围：务必禁止不带任何限制数据范围条件的查询语句。比如：我们当用户在查询订单历史的时候，我们可以控制在一个月的范围内。；

（2）读/写分离：经典的数据库拆分方案，主库负责写，从库负责读；
缓存：使用MySQL的缓存，另外对重量级、更新少的数据可以考虑使用应用级别的缓存；

（3）还有就是通过分库分表的方式进行优化，主要有垂直分表和水平分表

**垂直分区：**

**根据数据库里面数据表的相关性进行拆分。** 例如，用户表中既有用户的登录信息又有用户的基本信息，可以将用户表拆分成两个单独的表，甚至放到单独的库做分库。

**简单来说垂直拆分是指数据表列的拆分，把一张列比较多的表拆分为多张表。** 如下图所示，这样来说大家应该就更容易理解了。

**垂直拆分的优点：** 可以使得行数据变小，在查询时减少读取的Block数，减少I/O次数。此外，垂直分区可以简化表的结构，易于维护。

**垂直拆分的缺点：** 主键会出现冗余，需要管理冗余列，并会引起Join操作，可以通过在应用层进行Join来解决。此外，垂直分区会让事务变得更加复杂；

#### 垂直分表

把主键和一些列放在一个表，然后把主键和另外的列放在另一个表中

![img](https://mmbiz.qpic.cn/mmbiz_jpg/N8scgexEBuLYOiaUBNfLEyvwYunlXrUzADch8bO0QzWIxf4xKNSHQsxPwePz1nrBdiccWAqxicAcupXAmpPYqr6LQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



适用场景

1、如果一个表中某些列常用，另外一些列不常用

2、可以使数据行变小，一个数据页能存储更多数据，查询时减少I/O次数

缺点

（1）有些分表的策略基于应用层的逻辑算法，一旦逻辑算法改变，整个分表逻辑都会改变，扩展性较差

（2）对于应用层来说，逻辑算法增加开发成本

（3）管理冗余列，查询所有数据需要join操作

**水平分区：**

保持数据表结构不变，通过某种策略存储数据分片。这样每一片数据分散到不同的表或者库中，达到了分布式的目的。水平拆分可以支撑非常大的数据量。

水平拆分是指数据表行的拆分，表的行数超过200万行时，就会变慢，这时可以把一张的表的数据拆成多张表来存放。举个例子：我们可以将用户信息表拆分成多个用户信息表，这样就可以避免单一表数据量过大对性能造成影响。

水品拆分可以支持非常大的数据量。需要注意的一点是:分表仅仅是解决了单一表数据过大的问题，但由于表的数据还是在同一台机器上，其实对于提升MySQL并发能力没有什么意义，所以 水平拆分最好分库 。

水平拆分能够 支持非常大的数据量存储，应用端改造也少，但 分片事务难以解决 ，跨界点Join性能较差，逻辑复杂。

《Java工程师修炼之道》的作者推荐 尽量不要对数据进行分片，因为拆分会带来逻辑、部署、运维的各种复杂度 ，一般的数据表在优化得当的情况下支撑千万以下的数据量是没有太大问题的。如果实在要分片，尽量选择客户端分片架构，这样可以减少一次和中间件的网络I/O。

#### 水平分表：

表很大，分割后可以降低在查询时需要读的数据和索引的页数，同时也降低了索引的层数，提高查询次数

![img](https://mmbiz.qpic.cn/mmbiz_jpg/N8scgexEBuLYOiaUBNfLEyvwYunlXrUzAC8lQvU0txXcjHN4bUQo7OVUx91PsMFWBdogus92C9vFAMLIIMyO7Ug/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**适用场景**

（1）表中的数据本身就有独立性，例如表中分表记录各个地区的数据或者不同时期的数据，特别是有些数据常用，有些不常用。

（2）需要把数据存放在多个介质上。

水平切分的缺点

（1）给应用增加复杂度，通常查询时需要多个表名，查询所有数据都需UNION操作

（2）在许多数据库应用中，这种复杂度会超过它带来的优点，查询时会增加读一个索引层的磁盘次数

**下面补充一下数据库分片的两种常见方案：**

（1）客户端代理：分片逻辑在应用端，封装在jar包中，通过修改或者封装JDBC层来实现。当当网的 Sharding-JDBC 、阿里的TDDL是两种比较常用的实现。

（2）中间件代理：在应用和数据中间加了一个代理层。分片逻辑统一维护在中间件服务中。我们现在谈的 Mycat 、360的Atlas、网易的DDB等等都是这种架构的实现。

**5、分库分表后面临的问题？**

（1）事务支持 分库分表后，就成了分布式事务了。如果依赖数据库本身的分布式事务管理功能去执行事务，将付出高昂的性能代价；如果由应用程序去协助控制，形成程序逻辑上的事务，又会造成编程方面的负担。

（2）跨库join

只要是进行切分，跨节点Join的问题是不可避免的。但是良好的设计和切分却可以减少此类情况的发生。解决这一问题的普遍做法是分两次查询实现。在第一次查询的结果集中找出关联数据的id,根据这些id发起第二次请求得到关联数据。分库分表方案产品

（3）跨节点的count,order by,group by以及聚合函数问题 这些是一类问题，因为它们都需要基于全部数据集合进行计算。多数的代理都不会自动处理合并工作。解决方案：与解决跨节点join问题的类似，分别在各个节点上得到结果后在应用程序端进行合并。和join不同的是每个结点的查询可以并行执行，因此很多时候它的速度要比单一大表快很多。但如果结果集很大，对应用程序内存的消耗是一个问题。

（4）数据迁移，容量规划，扩容等问题 来自淘宝综合业务平台团队，它利用对2的倍数取余具有向前兼容的特性（如对4取余得1的数对2取余也是1）来分配数据，避免了行级别的数据迁移，但是依然需要进行表级别的迁移，同时对扩容规模和分表数量都有限制。总得来说，这些方案都不是十分的理想，多多少少都存在一些缺点，这也从一个侧面反映出了Sharding扩容的难度。

（5）ID问题，一旦数据库被切分到多个物理结点上，我们将不能再依赖数据库自身的主键生成机制。一方面，某个分区数据库自生成的ID无法保证在全局上是唯一的；另一方面，应用程序在插入数据之前需要先获得ID,以便进行SQL路由. 一些常见的主键生成策略

（6）跨分片的排序分页

般来讲，分页时需要按照指定字段进行排序。当排序字段就是分片字段的时候，我们通过分片规则可以比较容易定位到指定的分片，而当排序字段非分片字段的时候，情况就会变得比较复杂了。为了最终结果的准确性，我们需要在不同的分片节点中将数据进行排序并返回，并将不同分片返回的结果集进行汇总和再次排序，最后再返回给用户。如下图所示：

![img](https://mmbiz.qpic.cn/mmbiz_png/N8scgexEBuLYOiaUBNfLEyvwYunlXrUzAkIhMCjcW7KJNWbEymRD6erEibjTMKZObrPsibWaF9ia5tibb6tVtgnlv7Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**6、MySQL的复制原理以及流程**

主从复制：将主数据库中的DDL和DML操作通过二进制日志（BINLOG）传输到从数据库上，然后将这些日志重新执行（重做）；从而使得从数据库的数据与主数据库保持一致。

**主从复制的作用**

（1）主数据库出现问题，可以切换到从数据库。

（2）可以进行数据库层面的读写分离。

（3）可以在从数据库上进行日常备份。

**MySQL主从复制解决的问题**

（1）数据分布：随意开始或停止复制，并在不同地理位置分布数据备份

（2）负载均衡：降低单个服务器的压力

（3）高可用和故障切换：帮助应用程序避免单点失败

（4）升级测试：可以用更高版本的MySQL作为从库

**MySQL主从复制工作原理**

（1）在主库上把数据更高记录到二进制日志

（2）从库将主库的日志复制到自己的中继日志

（3）从库读取中继日志的事件，将其重放到从库数据中

基本原理流程，3个线程以及之间的关联

主：binlog线程——记录下所有改变了数据库数据的语句，放进master上的binlog中；

从：io线程——在使用start slave 之后，负责从master上拉取 binlog 内容，放进自己的relay log中；

从：sql执行线程——执行relay log中的语句；

![img](https://mmbiz.qpic.cn/mmbiz_jpg/N8scgexEBuLYOiaUBNfLEyvwYunlXrUzACfmHVG6uvLxKLucCQxd9CEgANNPWVUB3zlL6DKOph176PUPyyDq1sw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**7、读写分离有哪些解决方案？**

读写分离是依赖于主从复制，而主从复制又是为读写分离服务的。因为主从复制要求slave不能写只能读（如果对slave执行写操作，那么show slave status将会呈现Slave_SQL_Running=NO，此时你需要按照前面提到的手动同步一下slave）。

**方案一：使用mysql-proxy代理**

优点：直接实现读写分离和负载均衡，不用修改代码，master和slave用一样的帐号，mysql官方不建议实际生产中使用

缺点：降低性能， 不支持事务

方案二

使用AbstractRoutingDataSource+aop+annotation在dao层决定数据源。
如果采用了mybatis， 可以将读写分离放在ORM层，比如mybatis可以通过mybatis plugin拦截sql语句，所有的insert/update/delete都访问master库，所有的select 都访问salve库，这样对于dao层都是透明。plugin实现时可以通过注解或者分析语句是读写方法来选定主从库。不过这样依然有一个问题， 也就是不支持事务， 所以我们还需要重写一下DataSourceTransactionManager， 将read-only的事务扔进读库， 其余的有读有写的扔进写库。

方案三

使用AbstractRoutingDataSource+aop+annotation在service层决定数据源，可以支持事务.

缺点：类内部方法通过this.xx()方式相互调用时，aop不会进行拦截，需进行特殊处理。

**8、备份计划，mysqldump以及xtranbackup的实现原理**

(1)备份计划

视库的大小来定，一般来说 100G 内的库，可以考虑使用 mysqldump 来做，因为 mysqldump更加轻巧灵活，备份时间选在业务低峰期，可以每天进行都进行全量备份(mysqldump 备份出来的文件比较小，压缩之后更小)。

100G 以上的库，可以考虑用 xtranbackup 来做，备份速度明显要比 mysqldump 要快。一般是选择一周一个全备，其余每天进行增量备份，备份时间为业务低峰期。

(2)备份恢复时间

物理备份恢复快，逻辑备份恢复慢

这里跟机器，尤其是硬盘的速率有关系，以下列举几个仅供参考

20G的2分钟（mysqldump）

80G的30分钟(mysqldump)

111G的30分钟（mysqldump)

288G的3小时（xtra)

3T的4小时（xtra)

逻辑导入时间一般是备份时间的5倍以上

(3)备份恢复失败如何处理

首先在恢复之前就应该做足准备工作，避免恢复的时候出错。比如说备份之后的有效性检查、权限检查、空间检查等。如果万一报错，再根据报错的提示来进行相应的调整。

(4)mysqldump和xtrabackup实现原理

mysqldump

mysqldump 属于逻辑备份。加入–single-transaction 选项可以进行一致性备份。后台进程会先设置 session 的事务隔离级别为 RR(SET SESSION TRANSACTION ISOLATION LEVELREPEATABLE READ)，之后显式开启一个事务(START TRANSACTION /*!40100 WITH CONSISTENTSNAPSHOT */)，这样就保证了该事务里读到的数据都是事务事务时候的快照。之后再把表的数据读取出来。如果加上–master-data=1 的话，在刚开始的时候还会加一个数据库的读锁(FLUSH TABLES WITH READ LOCK),等开启事务后，再记录下数据库此时 binlog 的位置(showmaster status)，马上解锁，再读取表的数据。等所有的数据都已经导完，就可以结束事务

Xtrabackup:

xtrabackup 属于物理备份，直接拷贝表空间文件，同时不断扫描产生的 redo 日志并保存下来。最后完成 innodb 的备份后，会做一个 flush engine logs 的操作(老版本在有 bug，在5.6 上不做此操作会丢数据)，确保所有的 redo log 都已经落盘(涉及到事务的两阶段提交

概念，因为 xtrabackup 并不拷贝 binlog，所以必须保证所有的 redo log 都落盘，否则可能会丢最后一组提交事务的数据)。这个时间点就是 innodb 完成备份的时间点，数据文件虽然不是一致性的，但是有这段时间的 redo 就可以让数据文件达到一致性(恢复的时候做的事

情)。然后还需要 flush tables with read lock，把 myisam 等其他引擎的表给备份出来，备份完后解锁。这样就做到了完美的热备。

**9、数据表损坏的修复方式有哪些？**

使用 myisamchk 来修复，具体步骤：

1）修复前将mysql服务停止。

2）打开命令行方式，然后进入到mysql的/bin目录。

3）执行myisamchk –recover 数据库所在路径/*.MYI

使用repair table 或者 OPTIMIZE table命令来修复，REPAIR TABLE table_name 修复表 OPTIMIZE TABLE table_name 优化表 REPAIR TABLE 用于修复被破坏的表。OPTIMIZE TABLE 用于回收闲置的数据库空间，当表上的数据行被删除时，所占据的磁盘空间并没有立即被回收，使用了OPTIMIZE TABLE命令后这些空间将被回收，并且对磁盘上的数据行进行重排（注意：是磁盘上，而非数据库）