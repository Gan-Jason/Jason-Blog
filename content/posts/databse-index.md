---
title: "MySQL主键、索引相关"
date: 2019-09-24T07:51:37+08:00
draft: false
categories: ["数据库相关"]
tags: ["MySQL","数据库基础","SQL","查询优化"]
author: "Jason·Gan"
---
* 最近在公司的一个项目中，遇到了一个因为基础不牢而引发的小问题，就是数据库主键、索引等问题，由于公司的一套封装好的开发工具中查询数据库是必须得基于主键查询上的，我把主键查询语句给删了导致查询不到数据，浪费了时间。此文记录一下数据库主键、索引等知识。   
## 概念  
* 主键(primary key) 能够唯一标识表中某一行的属性或属性组。一个表只能有一个主键，但可以有多个候选索引。主键常常与外键构成参照完整性约束，防止出现数据不一致。主键可以保证记录的唯一和主键域非空,数据库管理系统对于主键自动生成唯一索引，所以主键也是一个特殊的索引。

* 外键（foreign key） 是用于建立和加强两个表数据之间的链接的一列或多列。外键约束主要用来维护两个表之间数据的一致性。简言之，表的外键就是另一表的主键，外键将两表联系起来。一般情况下，要删除一张表中的主键必须首先要确保其它表中的没有相同外键（即该表中的主键没有一个外键和它相关联）。

* 索引(index) 是用来快速地寻找那些具有特定值的记录。主要是为了检索的方便，是为了加快访问速度， 按一定的规则创建的，一般起到排序作用。所谓唯一性索引，这种索引和前面的“普通索引”基本相同，但有一个区别：索引列的所有值都只能出现一次，即必须唯一。 
* 主键一定是唯一性索引，唯一性索引并不一定就是主键。
* 一个表中可以有多个唯一性索引，但只能有一个主键。
* 主键列不允许空值，而唯一性索引列允许空值。
* 主键可以被其他字段作外键引用，而索引不能作为外键引用。
* KEY是数据库的物理结构，一般理解为键，包含两层含义，一是约束，偏重于约束和规范数据库的结构完整性，二是索引，辅助查询。约束作用一般有约束唯一性和引用完整性。
* INDEX也是数据库的物理结构，一般翻译为索引，但只辅助查询，会在创建时占用另外的空间。索引分为前缀索引、全文索引等。索引只是索引，不会去约束索引字段的行为。
## 具体介绍  
* 主键：
    * 主键是数据表的唯一索引，比如学生表里有学号和姓名，姓名可能有重名的，但学号确是唯一的，你要从学生表中搜索一条纪录如查找一个人，就只能根据学号去查找，这才能找出唯一的一个，这就是主键;如：idint(10) not null primary key auto_increment ；自增长的类型 ；  
* 外键：  
    * 比如有一张学生表(students)存储了学生的基本信息如学号、姓名、性别、父母信息、电话等，另一张表存储了学生的成绩(grade)，也存储了学生的姓名、学号、各课成绩等，这两张表中都存储了学生的学号，且这个学号是唯一性的，也就是说成绩表的录入是根据学生表的学号姓名等已存在的信息去录入该学生的成绩,创建表：  
    ```

    create table students(
        ...
        id smallint not null;
        ...
    );
    create table grade(
        ...
        id smallint not null;
    );

    ```  
### 设置索引  
* 若要设置外键，在参照表(referencing table，即grade表) 和被参照表 (referencedtable，即students表) 中，相对应的两个字段必须都设置索引(index)。对Parts表：```ALTER TABLE grade ADD INDEX idx_id(id);```这句话的意思是，为grade表增加一个索引，索引建立在id字段上，给这个索引起个名字叫idx_id。对student表也类似：```ALTER TABLE student ADD INDEX idx_id(id);```事实上这两个索引可以在创建表的时候就设置。这里只是为了突出其必要性。  
### 定义外键  
* 下面为两张表之间建立前面所述的那种“约束”。因为grade的学号id必须参照students表中的相应学号，所以我们将grade表的id字段设置为“外键”(FOREIGNKEY)，即这个键的参照值来自于其他表。 
```

ALTER TABLE grade ADD CONSTRAINT fk_id
FOREIGN KEY (id)
REFERENCES students(id);
```  
* 第一行是说要为grade表设置外键，给这个外键起一个名字叫做fk_id；第二行是说将本表的id字段设置为外键；第三行是说这个外键受到的约束来自于students表的id字段。这样，我们的外键就可以了。如果我们试着录入一个学生成绩，它的学号在students表中是不存在的，那么MySQL会禁止录入这条成绩。
* 级联操作
考虑以下这种情况：一天老师发现之前students表中录入的学生信息有失误，学号和姓名都搞错了，需要重新修改，那此时成绩表grade也会受影响，设想我们想在修改了students的学号后，grade表也会随之更新。
可以在定义外键的时候，在最后加入这样的关键字：```ON UPDATE CASCADE;```即在主表更新时，子表（们）产生连锁更新动作，似乎有些人喜欢把这个叫“级联”操作。  
```

ALTER TABLE grade ADD CONSTRAINT fk_id
FOREIGN KEY (id)
REFERENCES students(id)
ON UPDATE CASCADE;
```  

* 除了 CASCADE 外，还有 RESTRICT(禁止主表变更)、SET NULL(子表相应字段设置为空)等操作
### 索引：
* 索引用来快速地寻找那些具有特定值的记录，所有MySQL索引都以B-树的形式保存。如果没有索引，执行查询时MySQL必须从第一个记录开始扫描整个表的所有记录，直至找到符合要求的记录。表里面的记录数量越多，这个操作的代价就越高。如果作为搜索条件的列上已经创建了索引，MySQL无需扫描任何记录即可迅速得到目标记录所在的位置。如果表有1000个记录，通过索引查找记录至少要比顺序扫描记录快100倍。  
* 创建一个people的表：  
```CREATE TABLE people ( peopleid SMALLINT NOT NULL, name CHAR(50) NOT NULL );```  
* 然后，我们完全随机把1000个不同name值插入到people表。在数据文件中name列没有任何明确的次序。如果我们创建了name列的索引，MySQL将在索引中排序name列。对于索引中的每一项，MySQL在内部为它保存一个数据文件中实际记录所在位置的“指针”。因此，如果我们要查找name等于“Mike”记录的 peopleid（SQL命令为“SELECTpeopleid FROM people WHEREname='Mike';”），MySQL能够在name的索引中查找“Mike”值，然后直接转到数据文件中相应的行，准确地返回该行的 peopleid（999）。在这个过程中，MySQL只需处理一个行就可以返回结果。如果没有“name”列的索引，MySQL要扫描数据文件中的所有 记录，即1000个记录！显然，需要MySQL处理的记录数量越少，则它完成任务的速度就越快。  
### 索引的类型  
* 普通索引  
* 这是最基本的索引类型，而且它没有唯一性之类的限制。普通索引可以通过以下几种方式创建：
```

创建索引，例如:
       CREATE INDEX <索引的名字> ON tablename (列的列表);
修改表，例如:
       ALTER TABLE tablename ADD INDEX [索引的名字] (列的列表); 
创建表的时候指定索引，例如:
       CREATE TABLE tablename ( [...], INDEX [索引的名字] (列的列表) );  
```
* 唯一性索引  
* 这种索引和前面的“普通索引”基本相同，但有一个区别：索引列的所有值都只能出现一次，即必须唯一。唯一性索引可以用以下几种方式创建：
```

创建索引，例如:
      CREATE UNIQUE INDEX <索引的名字> ON tablename (列的列表); 
修改表，例如:
      ALTER TABLE tablename ADD UNIQUE [索引的名字] (列的列表); 
创建表的时候指定索引，例如:
      CREATE TABLE tablename ( [...], UNIQUE [索引的名字] (列的列表) );
```  

### 单列索引和多列索引  
* 索引可以是单列索引，也可以是多列索引。下面我们通过具体的例子来说明这两种索引的区别。假设有这样一个people表：  
```

CREATE TABLE people 
( peopleid SMALLINT NOT NULL AUTO_INCREMENT, 
  firstname CHAR(50) NOT NULL, 
  lastname CHAR(50) NOT NULL, 
  age SMALLINT NOT NULL, 
  townid SMALLINT NOT NULL,
  PRIMARY KEY (peopleid) );
```  

* 首先，我们可以考虑在单个列上创建索引，比如firstname、lastname或者age列。如果我们创建firstname列的索引```ALTERTABLE people ADD INDEX firstname(firstname);```,MySQL将通过这个索引迅速把搜索范围限制到那些firstname='Mike'的记录，然后再在这 个“中间结果集”上进行其他条件的搜索：它首先排除那些lastname不等于“Sullivan”的记录，然后排除那些age不等于17的记录。当记录满足所有搜索条件之后，MySQL就返回最终的搜索结果。
* 由于建立了firstname列的索引，与执行表的完全扫描相比，MySQL的效率提高了很多，但我们要求MySQL扫描的记录数量仍旧远远超过了实际所需要的。虽然我们可以删除firstname列上的索引，再创建lastname或者age列的索引，但总地看来，不论在哪个列上创建索引搜索效率仍旧相似。  
* 为了提高搜索效率，我们需要考虑运用多列索引。如果为firstname、lastname和age这三个列创建一个多列索引，MySQL只需一次检索就能够找出正确的结果！下面是创建这个多列索引的SQL命令：  
```  

ALTER TABLE people ADD INDEX fname_lname_age (firstname,lastname,age);
```  
* 由于索引文件以B-树格式保存，MySQL能够立即转到合适的firstname，然后再转到合适的lastname，最后转到合适的age。在没有扫描数据文件任何一个记录的情况下，MySQL就正确地找出了搜索的目标记录.
* 如果在firstname、lastname、age这三个列上分别创建单列索引，效果是否和创建一个firstname、lastname、 age的多列索引一样呢？答案是否定的，两者完全不同。当我们执行查询的时候，MySQL只能使用一个索引。如果你有三个单列的索引，MySQL会试图选择一个限制最严格的索引。但是,即使是限制最严格的单列索引，它的限制能力也肯定远远低于firstname、lastname、age这三个列上的多列索引。