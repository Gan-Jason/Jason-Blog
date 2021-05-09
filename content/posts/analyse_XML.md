---
title: "ElementTree解析XML文件"
date: 2019-07-18T00:16:21+08:00
draft: false
categories: ["Python相关"]
tags: ["Python","ElementTree","XML"]
author: "Jason·Gan"
---
* 前言：Python对于解析数据有着很完善的各种库和社区，比如处理JSON、CSV、HTML、XML、Excel等等格式的数据，今天用ElementTree修改了XML，故记录一下。   
* 先放样例XML  
```
<props>
        <prop class="CODEditProperties" id="50"/>
        <prop class="CODEditProperties" id="40" ccontain="1"/>
        <prop class="CODFillProperties" id="500" clr="0">
            <matchanimatestyle/>
            <matchanimateflash/>
        </prop>
</props>
    <children>
        <child class="CODRectComponent" pts="0,0;86,0;86,229;0,229;" type="Rectangle" name="Rectangle751" id="182308005" rect="0,0,87,76">
            <xform m00="1.011628" m11="0.331878"/>
            <props>
                <prop class="CODFillProperties" id="500" clr="0">
                    <matchanimatestyle/>
                    <matchanimateflash/>
                </prop>
                <prop class="CODLineProperties" id="580">
                    <matchanimatestyle/>
                    <matchanimateflash/>
                </prop>
                <prop class="CODEditProperties" id="40"/>
            </props>
            <children/>
            <spvars/>
            <cuslists/>
            <matchanimate8/>
            <matchanimate11/>
            <matchanimate12/>
        </child>
    </children>
      
``` 

## XML  
* Python解析XML一般有四种方法:
> 1.SAX (simple API for XML )  
> 2.DOM(Document Object Model)  
> 3.ElementTree  
> 4.BeautifulSoup  
* 这里只介绍一下ElementTree  
## ElementTree  
* ElementTree 有两种方式进行读取XML文件，一种是从文件读取，一种是从字符串读入：  
1.
```  
import xml.etree.ElementTree as ET  
tree = ET.parse('sample.xml')  
root = tree.getroot()  
```
2.
```
root = ET.fromstring(sample_as_string)  
```
* element就是一个节点对象，先找到根节点，再往下进行遍历  
* 查找孩子节点有两种方式：一种是通过Element.findall()查找所有的指定节点名的孩子节点，一种是Element.find()查找第一个找到的符合条件的节点  
* 查看Element的值用Element.text,查看节点属性的键值，用Element.attrib(name)查找属性名为name的值  
* 要修改属性值的话，可以用Element.set()修改，这样修改的话是写入到内存中的解析器对象树中的，所以最后通过解析器对象进行保存修改即可  
* 也可以新增和删除节点：  

> 新增节点：Element.append()  
> 删除节点：Element.remove()  

* 最后保存修改好的XML文件：Element.write(),其中有几个参数是常用的，比如encoding,xml_declaration,即doc.write("efault.xml", encoding="UTF-8", xml_declaration=True)  
## Element+Xpath  
* 在Element中可以通过Xpath定位节点  
```  
root=ET.parse("default.xml")  
root.findall(".//children//prop//@class)  

...  
```

