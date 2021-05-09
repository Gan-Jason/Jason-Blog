---
title: "C++连接SQLite3，进行增删改查"
date: 2019-07-22T18:20:51+08:00
draft: false
tags: ["C++","VS","SQLite3","数据库"]
categories: ["C++相关"]
author: "Jason·Gan"
---  
* 前言：这两天在公司要开发一些自动化工具，其中涉及到SQLite的访问，SQLite的文件是后缀名为.db的文件，一开始我还不了解这个数据库，直接把这些文件导入到MySQL里进行操作；后来同事提醒我这是SQLite，如果转为MySQL的话在生产环境中可能没配有MySQL的环境就回有点麻烦，我赶紧把它改了过来，不然就白费力气啦。  

## SQLite3  
* SQLite是一个进程内的库，实现了自给自足的、无服务器的、零配置的、事务性的 SQL 数据库引擎。它是一个零配置的数据库，这意味着与其他数据库一样，不需要在系统中配置。就像其他数据库，SQLite 引擎不是一个独立的进程，可以按应用程序需求进行静态或动态连接。SQLite 直接访问其存储文件（.db）。  
*  C++连接SQLite要有静态库，动态库，以及sqlite3.h头文件，这一套直接下载就行，**[下载地址](https://download.csdn.net/download/binuogan_c/11399374)**；下载完后直接添加到工程里，在cpp文件中包含一下头文件就行，静态库的链接也很简单，我直接写过一篇关于VS链接外部库文件的[文章](http://jasongan.xyz/post/2/)  

### SQLite3的C++接口  
1.sqlite3_open(const char *filename, sqlite3 **ppDb)  

> This routine opens a connection to an SQLite database file and returns a database connection object to be used by other SQLite routines.  
If the filename argument is NULL or ‘:memory:’, sqlite3_open() will create an in-memory database in RAM that lasts only for the duration of the session.  
If filename is not NULL, sqlite3_open() attempts to open the database file by using its value. If no file by that name exists, sqlite3_open() will open a new database file by that name.  

2.sqlite3_exec(sqlite3*, const char *sql, sqlite_callback, void *data, char **errmsg) 

> This routine provides a quick, easy way to execute SQL commands provided by sql argument which can consist of more than one SQL command.  
Here, first argument sqlite3 is open database object, sqlite_callback is a call back for which data is the 1st argument and errmsg will be return to capture any error raised by the routine.  

3.sqlite3_close(sqlite3*)  

> This routine closes a database connection previously opened by a call to sqlite3_open(). All prepared statements associated with the connection should be finalized prior to closing the connection.  
If any queries remain that have not been finalized, sqlite3_close() will return SQLITE_BUSY with the error message Unable to close due to unfinalized statements.  

* 以上这三个方法分别是创建数据库的连接、执行SQL语句、关闭数据库连接  

## 增删改查  
**注：因为增删改查中，增删改都没什么复杂度，直接调用接口就行了，但是查有点特殊，他的设计是采用回调函数进行查找的，如果要保存查找的结果，就需要改写一下callback函数，这是本文重点。网上大多数都是直接cout数据的**
### Insert  （增删改只写一个增，其他可参照来写，或者参考[官方文档](https://www.tutorialspoint.com/sqlite/sqlite_c_cpp.htm)）
```  
#include <stdio.h>
#include <stdlib.h>
#include <sqlite3.h> 

static int callback(void *NotUsed, int argc, char **argv, char **azColName) {
   int i;
   for(i = 0; i<argc; i++) {
      printf("%s = %s\n", azColName[i], argv[i] ? argv[i] : "NULL");
   }
   printf("\n");
   return 0;
}

int main(int argc, char* argv[]) {
   sqlite3 *db;
   char *zErrMsg = 0;
   int rc;
   char *sql;

   /* Open database */
   rc = sqlite3_open("test.db", &db);
   
   if( rc ) {
      fprintf(stderr, "Can't open database: %s\n", sqlite3_errmsg(db));
      return(0);
   } else {
      fprintf(stderr, "Opened database successfully\n");
   }

   /* Create SQL statement,这里随便写一句，实际要根据你的数据表来写 */
   sql = "INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY) "  \
         "VALUES (1, 'Paul', 32, 'California', 20000.00 ); " 


   /* Execute SQL statement */
   rc = sqlite3_exec(db, sql, callback, 0, &zErrMsg);
   
   if( rc != SQLITE_OK ){
      fprintf(stderr, "SQL error: %s\n", zErrMsg);
      sqlite3_free(zErrMsg);
   } else {
      fprintf(stdout, "Records created successfully\n");
   }
   sqlite3_close(db);
   return 0;
}  
```  
### select  

```  
#include <stdio.h>
#include <stdlib.h>
#include <sqlite3.h> 

static int callback(void *data, int argc, char **argv, char **azColName){
   int i;
   //fprintf(stderr, "%s: ", (const char*)data);    //注意此处！
   vector<string>* result=(vector<string>*)data;
   for(i = 0; i<argc; i++){
      //网上的版本：printf("%s = %s\n", azColName[i], argv[i] ? argv[i] : "NULL");  
      result.push_back(string(argv[i]));
   }
   
   printf("\n");
   return 0;
}

int main(int argc, char* argv[]) {
   sqlite3 *db;
   char *zErrMsg = 0;
   int rc;
   char *sql;
   vector<string> result;

   /* Open database */
   rc = sqlite3_open("test.db", &db);
   
   if( rc ) {
      fprintf(stderr, "Can't open database: %s\n", sqlite3_errmsg(db));
      return(0);
   } else {
      fprintf(stderr, "Opened database successfully\n");
   }

   /* Create SQL statement */
   sql = "SELECT * from COMPANY";

   /* Execute SQL statement */
   rc = sqlite3_exec(db, sql, callback, &result, &zErrMsg); //注意此处：第四个参数是存放结果的
   
   if( rc != SQLITE_OK ) {
      fprintf(stderr, "SQL error: %s\n", zErrMsg);
      sqlite3_free(zErrMsg);
   } else {
      fprintf(stdout, "Operation done successfully\n");
   }
   sqlite3_close(db);
   return 0;
}  
```  


* 如果我一开始能看到我自己写的这篇文章的话，我就省了不少时间了！羡慕此时阅读此文的你。  




