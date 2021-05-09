---
title: "idea配置自定义类模板、方法模板及其快捷键"
date: 2019-09-22T18:33:08+08:00
draft: false
tags: ["Java","idea","JDK11","Template"]
categories: ["Java相关"]
author: "Jason·Gan"
---

* 前言:最近看源码，发现别人的注释都写的很好看，很规范，自己写的注释就是没那么工整。于是想着弄一个类模板。  

## 类模板  
* 打开idea，在左上角，File->Setting->Editor->Live Templates里，如下图：  
![](/image/template/1.jpg)  
在上图中的右边的加号那里，新建一个Template Group，我的名为userDefine，你的随意。在这个新建的组里，新建一个Live Template，我的名为*，Abbreviation是你的快捷键设置，我的就写*，于是我添加模板的时候，按```/**```+```Tab```就行了，Tab是默认的，你可以右下角改你想要的键。  
![](/image/template/2.jpg)  
* 在Template text的文本编辑框里，写入你的注释模板：  
```
**
 * @ClassName $classname$
 * @Author Jason
 * @Description //TODO $end$
 * @Date $time$ $date$
 * @Version 1.0
 */
```  
上面的$$是加入脚本的地方，这个脚本不用自己写，它自带有，你只需要写你想要的注释内容，如前面的@ClassName等这些。  
* 在上图红圈的上面的Edit variables里选择你注释里脚本表达式，可以参考我选的：  
![](/image/template/4.jpg)  
基本就可以了，之后你就在类的上面，按```/*```加你定义模板缩写加快捷键，就可以快速生成了，我也看到网上有说类模板可以在File and Code Templates 里设置，但是我不知道那里面的快捷键是什么，所以就用这个方法了。
* 注意的一点是，你添加类模板后，有可能会有一个tag warning类似的错误，就是注释会闪着深深的荧光绿，解决办法是鼠标点着改荧光绿，按alt+enter键，添加一个自定义tag点就行了，下次就不会出现这种情况了  

## 方法模板  
 * 方法模板是一样的设置方法，就是Template text里模板的内容有所不同，以下是我的写法:  
 ```
 **
 * @Author Jason
 * @Description //TODO $end$
 * @Date $time$ $date$
 * @Param $param$
 * @return $return$
 */
 ```  
 * 再选一下上面所说的脚本表达式即可。  
 * 至于格式问题，自己多试试就可以了。

