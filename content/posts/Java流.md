---
title: "Java Stream相关"
date: 2018-12-28T19:00:50+08:00
draft: false
categories: ["Java相关"]
tags: ["Java","基础语法","面向对象","I/O流"]
author: "Jason·Gan"
---
* 一个流理解为一个数据序列，输入流表示为从一个源读取数据，输出流表示输出数据到一个源。因此平时我们写entity的时候应该养成记得实现可序列化接口的习惯(Serializable)。Java.io 包几乎包含了所有操作输入、输出需要的类。所有这些流类代表了输入源和输出目标，包中的流支持很多种格式，比如：基本类型、对象、本地化字符集等等。  
* 下图是Java中输出和输入流的一个类关系图：  
![](/image/stream/1.png)  

* 本文挑其中几个较为常用的流来讨论。  

## InputStream、OutputStream  
* 处理字节流的抽象类，面对的对象是字节流  
* InputStream 是字节输入流的所有类的超类,一般我们使用它的子类,如FileInputStream等。  
* OutputStream是字节输出流的所有类的超类,一般我们使用它的子类,如FileOutputStream等。  
* 举个例子：  
``` 

...

public class Test_InputStream {

    /**
     * 获取字节流
     * @param url
     * @return
     */
    private String getStream(String url){
        //获取字节流
        InputStream in = null;
        String result = "";
        try {
            in = new URL(url).openStream();
            int tmp;
            while((tmp = in.read()) != -1){
                result += (char)tmp;
            }
        } catch (MalformedURLException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        //输出字节流
        return result;
    }

    public static void main(String[] args){

        String URL = "http://www.baidu.com";
        Test_InputStream test = new Test_InputStream();
        System.out.println(test.getStream(URL));

    }
}


```
* 通过URL连接获取了InputStream流连接，然后通过read方法来一个字节一个字节的读取字节流并组合在一起（read方法返回-1则数据读取结束），最后以reasults返回。获取到的字节流如：60 33 68 79 67 84 89 80 69 32 104 116 109 108 62 60 33 45 45 83 84 65 84 ……每个数字都是一个字节（Byte，8位），所以如果读取英文的话，用字节流，然后用(char)强制转化一下就行了，但如果有中文等双字节语言或者说需要指定字符编码集的情况，就必须用到InputStreamReader将字节流转化为字符流了。  

## InputStreamReader,OutputStreamWriter  
* 两个都是处理字符流的抽象类  
* InputStreamReader 是字节流通向字符流的桥梁,它将字节流转换为字符流。
* OutputStreamWriter是字符流通向字节流的桥梁，它将字符流转换为字节流。  
* 举个例子：  
```  

public class Test_InputStreamReader {

    /*
     * 得到字符流前需先有字节流
     */
    private String getStream(String url){
        try {
            //得到字节流
            InputStream in = new URL(url).openStream();
            //将字节流转化成字符流，并指定字符集
            InputStreamReader isr = new InputStreamReader(in,"UTF-8");
            String results = "";
            int tmp;
            while((tmp = isr.read()) != -1){
                results += (char)tmp;
            }
            return results;

        } catch (MalformedURLException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        return null;
    }

    /**
     * @param args
     */
    public static void main(String[] args) {
        // TODO Auto-generated method stub
        String URL = "http://www.baidu.com";
        Test_InputStreamReader test = new Test_InputStreamReader();
        System.out.println(test.getStream(URL));
    }

}

```  
* 先获取字节流，然后创建InputStreamReader将字节流转化成字符流，并指定其字符集为UTF-8，然后使用强制转化将read到的int字节转化为char型，此时已可以输出中文字符，并且可速度上看出，输出字符流比输出字节流要快。下面是上述程序执行结果，可见中文：  
```

<!DOCTYPE html>
<!--STATUS OK--><html> <head><meta http-equiv=content-type content=text/html;charset=utf-8><meta http-equiv=X-UA-Compatible content=IE=Edge><meta content=always name=referrer><link rel=stylesheet type=text/css href=http://s1.bdstatic.com/r/www/cache/bdorz/baidu.min.css><title>百度一下，你就知道</title></head> <body link=#0000cc> <div id=wrapper> <div id=head> <div class=head_wrapper> <div class=s_form> <div class=s_form_wrapper> <div id=lg> <img hidefocus=true src=//www.baidu.com/img/bd_logo1.png width=270 height=129> </div> <form id=form name=f action=//www.baidu.com/s class=fm> <input type=hidden name=bdorz_come value=1> <input type=hidden name=ie value=utf-8> <input type=hidden name=f value=8> <input type=hidden name=rsv_bp value=1> <input type=hidden
name=rsv_idx value=1> <input type=hidden name=tn value=baidu><span class="bg s_ipt_wr"><input id=kw name=wd class=s_ipt value maxlength=255 autocomplete=off autofocus></span><span class="bg s_btn_wr"><input type=submit id=su value=百度一下 class="bg s_btn"></span> </form> </div> </div> <div id=u1> <a href=http://news.baidu.com name=tj_trnews class=mnav>新闻</a> <a href=http://www.hao123.com name=tj_trhao123 class=mnav>hao123</a> <a href=http://map.baidu.com name=tj_trmap class=mnav>地图</a> <a href=http://v.baidu.com name=tj_trvideo class=mnav>视频</a> <a href=http://tieba.baidu.com name=tj_trtieba class=mnav>贴吧</a> <noscript> <a href=http://www.baidu.com/bdorz/login.gif?login&amp;tpl=mn&amp;u=http%3A%2F%2Fwww.baidu.com%2f%3fbdorz_come%3d1 name=tj_login class=lb>登录</a> </noscript> <script>document.write('<a href="http://www.baidu.com/bdorz/login.gif?login&tpl=mn&u='+ encodeURIComponent(window.location.href+ (window.location.search === "" ? "?" : "&")+ "bdorz_come=1")+ '" name="tj_login" class="lb">登录</a>');</script> <a href=//www.baidu.com/more/ name=tj_briicon class=bri style="display: block;">更多产品</a> </div> </div> </div> <div id=ftCon> <div id=ftConw> <p id=lh> <a href=http://home.baidu.com>
关于百度</a> <a href=http://ir.baidu.com>About Baidu</a> </p> <p id=cp>&copy;2017&nbsp;Baidu&nbsp;<a href=http://www.baidu.com/duty/>使用百度前必读</a>&nbsp; <a href=http://jianyi.baidu.com/ class=cp-feedback>意见反馈</a>&nbsp;京ICP证030173号&nbsp; <img src=//www.baidu.com/img/gs.gif> </p> </div> </div> </div> </body> </html>
```

* BufferedReader,BufferedReader则是比InputStreamReader更高级,它封裝了StreamReader类,一次读取取一行的字符。  
```

public class Test_BufferedReader {

    /*
     * 字节流——字符流——缓存输出的字符流
     */
    private String getStream(String url){
        try {
            //得到字节流
            InputStream in = new URL(url).openStream();
            //将字节流转化成字符流，并指定字符集
            InputStreamReader isr = new InputStreamReader(in,"UTF-8");
            //将字符流以缓存的形式一行一行输出
            BufferedReader bf = new BufferedReader(isr); 
            String results = "";
            String newLine = "";
            while((newLine = bf.readLine()) != null){
                results += newLine+"\n";
            }
            return results;

        } catch (MalformedURLException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        return null;
    }

    /**
     * @param args
     */
    public static void main(String[] args) {
        // TODO Auto-generated method stub
        String URL = "http://www.baidu.com";
        Test_BufferedReader test = new Test_BufferedReader();
        System.out.println(test.getStream(URL));
    }

}
```  
* 获取字符流后，可直接缓存，然后从缓存区取出，这时的速度比InputStreamReader又将快上很多。输出结果同上。  

* 如果是音频文件、图片、歌曲，就用字节流好点，如果是关系到中文（文本）的，用字符流好点，所有文件的储存是都是字节（byte）的储存，在磁盘上保留的并不是文件的字符而是先把字符编码成字节，再储存这些字节到磁盘。
* 在读取文件（特别是文本文件）时，也是一个字节一个字节地读取以形成字节序列，字节流可用于任何类型的对象，包括二进制对象，而字符流只能处理字符或者字符串；

* Reader、Writer：为字符流（一个字符占两个字节）,主要用来处理字符或字符串。
* 字符流处理的单元为2个字节的Unicode字符，分别操作字符、字符数组或字符串，而字节流处理单元为1个字节，操作字节和字节数组。
* 所以字符流是由Java虚拟机将字节转化为2个字节的Unicode字符为单位的字符而成的，所以它对多国语言支持性比较好。  
* 其他字节、字符相关概念，见[Java 字节、字符、二进制等关系](/posts/byte-stream/)


