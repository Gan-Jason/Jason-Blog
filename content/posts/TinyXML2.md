---
title: "用TinyXML2解析XML"
date: 2019-07-19T23:49:29+08:00
draft: false
tags: ["C++","TinyXML","C++11","XML"]
categories: ["C++相关"]
author: "Jason·Gan"
---

* 前言：上一篇有讲到用Python的一个库ElementTree解析XML文件，方便是方便，一开始也是贪图其方便所以采用这个方案的，但是写完之后出现一个问题，原XML文件的节点属性里嵌有一个VB脚本，VB脚本语法是由严格缩进规约的，解析完后发现VB脚本格式全乱，导致VB失效，这样这个XML文件就相当于坏了，因为只能另寻他法，但是发现这个Python很难解决节点属性格式的问题，因此只能考虑二进制读写文件。最后采用了TinyXML2这个C++的库。  
## TinyXML2  
* 这个C++库是比较成熟的解析XML文件的库，他的实现是可以二进制读写文件的，所以极为满足我的需求  
* 这个解析库的模型通过解析XML文件，然后在内存中生成DOM模型，从而让我们很方便的遍历这棵XML树。 DOM模型即文档对象模型，是将整个文档分成多个元素（如书、章、节、段等），并利用树型结构表示这些元素之间的顺序关系以及嵌套包含关系。   
* 在GitHub上clone这个库到本地仓库上来，地址为：<https://github.com/leethomason/tinyxml2.git>  
* 在下载好的文件夹根路径里可以看到这些文件： 
![](/image/tiny.jpg)  
* 只需要把里面的tinyxml2.h和tinyxml2.cpp这两个文件复制到你的工程目录里面，如果用的是VS的话，还得在项目属性里面添加外部头文件的依赖路径。  
* 在工程里引用tinyxml2.h这个头文件，并加上using namespace tinyxml2这个命名空间，否则一些数据类型会出问题。  
* 这里介绍一些常用的类： 

> FirstChildElement(const char* value=0):获取第一个值为value的子节点，value默认值为空，则返回第一个子节点。  
> RootElement():获取根节点，相当于FirstChildElement的空参数版本；  
> const XMLAttribute* FirstAttribute() const:获取第一个属性值；  
> XMLHandle NextSiblingElement( const char* _value=0 ):获得下一个节点。  
> Attribute(const char* name):获取节点指定属性的属性值  
> SaveFile(const char* path):写入文件  
> etAttribute(const char* name,const char* value):修改某节点属性值  
> element->Name():节点的名字  
> element->Value():节点的值  

* TinyXML2里所有的字符串变量都是const char* 常字符指针型，所以使用时要注意，可以使用string类方便点，通过c_str()方法将string转化为const char*  


