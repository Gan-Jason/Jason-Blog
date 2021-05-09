---
title: "MySQL常用的一些命令"
date: 2019-05-24T07:51:37+08:00
draft: false
categories: ["数据库相关"]
tags: ["MySQL","数据库基础","SQL"]
author: "Jason·Gan"
---

## 常见的MySQL命令

---

## 一.用户验证  
### 1.连接MySQL  

* 语法  
    <table><tr><td bgcolor=pink>    
    mysql -h xx.xx.xx(主机ip,或为localhost) -u xxx(用户名) -p xxx(密码)
    </td></tr></table>

***注：用户名和主机IP前有无空格均可，但是密码前不能前不能有空格***

---
### 2.修改密码
* 语法
    <table><tr><td bgcolor=pink>
    mysqladmin -u用户名 -p旧密码 password 新密码
    </td></tr></table>
* 给用户(root)加个密码
    <table><tr><td bgcolor=pink>
    mysqladmin -u用户名 -p旧密码 password 新密码
    </td></tr></table>
***注：因为开始时root没有密码，所以-p旧密码一项就可以省略了。 如果进入了mysql后想修改密码，参照下条*** 

* 初始化密码
   <table><tr><td bgcolor=pink>
     set PASSWORD=PASSWORD("123");注意mysql语句以分号结束
   </td></tr></table>
***注：以上均是对MySQL服务的操作，不属于SQL语句，句尾不需要分号，和以下区分***

---
## 二.数据库操作
### 3.添加新用户
* 语法
   <table><tr><td bgcolor=pink>
     grant select on 数据库.* to 用户名@登录主机 identified by “密码”
   </td></tr></table>
* 增加一个用户test1密码为abc，让他可以在任何主机上登录，并对所有数据库有查询、插入、修改、删除的权限。首先用root用户连入MYSQL，然后键入以下命令：

* 但增加的用户是十分危险的，你想如某个人知道test1的密码，那么他就可以在internet上的任何一台电脑上登录你的mysql数据库并对你的数据可以为所欲为了，解决办法见👇。
   <table><tr><td bgcolor=pink>
    grant select,insert,update,delete on mydb.* to [email=test2@localhost]test2@localhost[/email] identified by “”;
   </td></tr></table>
   
* 增加一个用户test2密码为abc,让他只可以在localhost上登录，并可以对数据库mydb进行查询、插入、修改、删除的操作(localhost指本地主机，即MYSQL数据库所在的那台主机),这样用户即使用知道test2的密码，他也无法从internet上直接访问数据库，只能通过MYSQL主机上的web页来访问了。

### 4.数据库操作
* 创建数据库
   <table><tr><td bgcolor=pink>
    create database database_name;
   </td></tr></table>
* 显示所有数据库
    <table><tr><td bgcolor=pink>
     show databases;
    </td></tr></table> 
* 删除数据库
   <table><tr><td bgcolor=pink>
    drop database <数据库名>;
   </td></tr></table>
* 选择数据库
   <table><tr><td bgcolor=pink>
    use <数据库名>;
   </td></tr></table>
### 5.数据表操作
* 显示所有数据表
   <table><tr><td bgcolor=pink>
    show tables;
   </td></tr></table>
* 显示table_name表的字段信息
   <table><tr><td bgcolor=pink>
    desc <数据表名>;
   </td></tr></table>

* 创建表
   <table><tr><td bgcolor=pink>
    create table  <数据表名> (name char(100), path char(100), count int(10), firstName char(100), firstMD5 char(100), secondName char(100), secondMD5 char(100), thirdName char(100), thirdMD5 char(100))engine=innodb,charset='utf8';
   </td></tr></table>
* 列出创建表时的创建语句，包含注释
   <table><tr><td bgcolor=pink>
    show create table <表名> \G
   </td></tr></table>
    
***注：创建表的结构、默认值、索引、递增递减自己选***

* 修改表名
   <table><tr><td bgcolor=pink>
    rename table <表1> to <表2>;
   </td></tr></table>
* 删除表
   <table><tr><td bgcolor=pink>
    drop table <表名>;   
   </td></tr></table>
* 插入数据
   <table><tr><td bgcolor=pink>
    insert into <表名> (name, path, count, firstName, firstMD5, secondName, secondMD5, thirdName, thirdMD5) VALUES ('test', 'test', 1, 'name1', 'md1', 'name2', 'md2', 'name3', 'md3');   
   </td></tr></table>
* 查询数据库中的重复数据 
   <table><tr><td bgcolor=pink>
    select firstMD5, count(*) as count from MIFit_Image group by firstMD5 having count > 1;  
   </td></tr></table>
* 内联接查询数据
   <table><tr><td bgcolor=pink>
    select <table1>.id,<table1>.name,<table2>.id,<table2>.name from <table1> inner join <table2> on <table1>.id=<table2>.id; --组合两个表中的记录，返回关联字段相符的记录，也就是返回两个表的交集
   </td></tr></table>
* 左联接查询数据
   <table><tr><td bgcolor=pink>
    select <table1>.id,<table1>.name,<table2>.id,<table2>.name from <table1> left join <table2> on <table1>.id=<table2>.id; --left join 是left outer join的简写，它的全称是左外连接，是外连接中的一种。左(外)连接，左表(a_table)的记录将会全部表示出来，而右表(b_table)只会显示符合搜索条件的记录。右表记录不足的地方均为NULL。也就是以左表数据为准，把左表数据全部列出来，右表没有相关数据的补NULL；
   </td></tr></table>
* 更新数据
   <table><tr><td bgcolor=pink>
    update <表名> set folderName ='Mary' where id=1;
   </td></tr></table>
* 删除数据
   <table><tr><td bgcolor=pink>
    delete from <表名> where folderName = 'test';
   </td></tr></table>
    
### 6.数据表段操作
* 语法
* 添加字段
   <table><tr><td bgcolor=pink>
    alter table <表名> add id int auto_increment not null primary key;
   </td></tr></table>
* 修改字段的顺序 (id 放置最前)
   <table><tr><td bgcolor=pink>
    ALTER TABLE <表名> MODIFY 字段名1 数据类型 FIRST ｜ AFTER 字段名2;
    其中：

    字段名1：表示需要修改位置的字段的名称。
    数据类型：表示“字段名1”的数据类型。
    FIRST：指定位置为表的第一个位置。
    AFTER 字段名2：指定“字段名1”插入在“字段名2”之后。

    alter table <表名> modify id int first;

    alter table <表名> modify id int after name;
   </td></tr></table>
* 移除id主键标志
   <table><tr><td bgcolor=pink>
    alter table <表名> modify id int, drop primary key;
   </td></tr></table>
* 修改字段名name为folderName
    <table><tr><td bgcolor=pink>
    alter table <表名> change name folderName char(100);
    </td></tr></table>
* 删除字段
    <table><tr><td bgcolor=pink>
    alter table <表名> drop folderName;
    </td></tr></table>
* 加索引
    <table><tr><td bgcolor=pink>
    alter table <表名> add index indexName (folderName);
    </td></tr></table>
* 删除索引
    <table><tr><td bgcolor=pink>
    alter table <表名> drop index indexName;
    </td></tr></table>
    
* 修改字段的默认值(例如修改时间字段默认值为当前时间戳)
    <table><tr><td bgcolor=pink>
    alter table <表名> modify <columnName> timestamp not null default current_timestamp;
    </td></tr></table>
    
    
  
## MySQL数据类型  
* MySQL支持所有标准SQL数值数据类型。这些类型包括严格数值数据类型(INTEGER、SMALLINT、DECIMAL和NUMERIC)，以及近似数值数据类型(FLOAT、REAL和DOUBLE PRECISION)。关键字INT是INTEGER的同义词，关键字DEC是DECIMAL的同义词。BIT数据类型保存位字段值，并且支持MyISAM、MEMORY、InnoDB和BDB表。作为SQL标准的扩展，MySQL也支持整数类型TINYINT、MEDIUMINT和BIGINT。下面的表显示了需要的每个整数类型的存储和范围。  
* 1、整型

|MySQL数据类型|含义（有符号）|
|----|----|
|tinyint(m) |1个字节  范围(-128~127)|
|smallint(m)	|2个字节  范围(-32768~32767)|
|mediumint(m)	|3个字节  范围(-8388608~8388607)|
|int(m)	        |4个字节  范围(-2147483648~2147483647)|
|bigint(m)	    |8个字节  范围(+-9.22*10的18次方)|   


* 取值范围如果加了unsigned，则最大值翻倍，如tinyint unsigned的取值范围为(0~256)。
int(m)里的m是表示SELECT查询结果集中的显示宽度，涉及MySQL中一个fillzero机制，也就是补零，对于整型，如果不满m位则在前面补零，例：int(4),select结果为18时，显示结果是0018，如果超出了这个m范围，则显示实际的查询结果，不影响；而对于字符串类型的则是存储的数据长度，如果超出了m，则超出的被丢失。  
* 2、浮点型(float和double) 

|MySQL数据类型|含义|
|----|----|
|float(m,d)	    |单精度浮点型    8位精度(4字节)     m总个数，d小数位|
|double(m,d)    |双精度浮点型    16位精度(8字节)    m总个数，d小数位|  

* 设一个字段定义为float(5,3)，如果插入一个数123.45678,实际数据库里存的是123.457，但总个数还以实际为准，即6位。  
* 3、定点数  
* 浮点型在数据库中存放的是近似值，而定点类型在数据库中存放的是精确值。decimal(m,d) 参数m<65 是总个数，d<30且 d<m 是小数位。  
* 4、字符串(char,varchar,_text)  

|MySQL数据类型	|含义|
|----|----|
|char(n)	|固定长度，最多255个字节|
|varchar(n)  |可变长度，最多65535个字节|
|tinytext    |短文本字符串，最多255个字节|
|text    |长文本数据，最多65535个字节|
|mediumtext  |中文本数据，最多2的24次方-1个字节|
|longtext    |极长文本数据，最多2的32次方-1个字节|   

* char和varchar：
1.char(n) 若存入字符数小于n，则以空格补于其后，查询之时再将空格去掉。所以char类型存储的字符串末尾不能有空格，varchar不限于此。
2.char(n) 固定长度，char(4)不管是存入几个字符，都将占用4个字节，varchar是存入的实际字符数+1个字节（n<=255）或2个字节(n>255)，所以varchar(4),存入3个字符将占用4个字节。
3.char类型的字符串检索速度要比varchar类型的快。
varchar和text：
1.varchar可指定n，text不能指定，内部存储varchar是存入的实际字符数+1个字节（n<=255）或2个字节(n>255)，text是实际字符数+2个字节。
2.text类型不能有默认值。
3.varchar可直接创建索引，text创建索引要指定前多少个字符。varchar查询速度快于text。  

* BLOB二进制数据  
* BLOB 是一个二进制大对象，可以容纳可变数量的数据。有 4 种 BLOB 类型：TINYBLOB、BLOB、MEDIUMBLOB 和 LONGBLOB。它们区别在于可容纳存储范围不同。可以存储音乐、图片、视频等媒体信息。  

|MySQL数据类型|含义|
|----|----|
|TINYBLOB   |0-255字节	不超过 255 个字符的二进制字符串|
|BLOB    |0-65 535字节	二进制形式的长文本数据|
|MEDIUMBLOB  |0-16 777 215字节	二进制形式的中等长度文本数据|
|LONGBLOB    |0-4 294 967 295字节	二进制形式的极大文本数据|

* 6.日期时间类型

|MySQL数据类型|含义|
|----|----|
|date   |日期 '2008-12-2'|
|time    |时间 '12:25:36'|
|datetime    |日期时间 '2008-12-2 22:06:44'|
|timestamp	|自动存储记录修改时间|

* 若定义一个字段为timestamp，这个字段里的时间数据会随其他字段修改的时候自动刷新，所以这个数据类型的字段可以存放这条记录最后被修改的时间。  


