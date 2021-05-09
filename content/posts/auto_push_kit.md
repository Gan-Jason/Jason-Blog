---
title: "自动化push代码到GitHub工具"
date: 2019-07-24T19:22:27+08:00
draft: false
tags: ["Python","git","auto"]
categories: ["Python相关"]
author: "JasonDian·Gan"
---
* 前言：因为每次写完代码，要push到GitHub上时都要进行繁琐的几个操作，无外乎```git add .```,```git pull```,```git commit```,```git push```,等这些重复工作，但是作为[一个拥有者懒惰这一优点的伟大的程序员](http://threevirtues.com/)，是坚决不能忍受这个问题的。  
## Python开发一个自动push脚本  
* 首先拿我的个人博客来作为需求来开发，我的博客是基于Hugo框架的，所以只要简单的几个操作就可以了，环境条件是git-bash或者CMD命令都可以，所以我得调用windows的命令行工具  
* Python中执行windows命令可以调用os库，用os库中的system函数可以直接执行命令，如：```os.system('ipconfig')```，就可以执行了，下图是控制台输出结果。也可以用另一个方法：```os.popen('xxx')```这个方法会返回命令执行结果，请看下面的代码中有介绍。  

<span align="center">![](/image/kit/1.jpg/)</span>

* 好，开始动手：  
```  
import os
# 首先先新建一个markdown文档
def new_md():
    md_name = input('Please input your markdown-file name:')
    os.chdir('e:/Hugo/JasonBlog')                       #chdir()是设置当前路径的方法，就不用去cd了
    os.popen('hugo new post/{}.md'.format(md_name))     #hugo新建文档的方式
    os.chdir('e:/Hugo/JasonBlog/public')
    build = os.popen('git pull')                        #在提交更改之前，先抓取远程仓库并且合并，用poen()方法执行命令
    print(build.read())                                 #read()就是读返回的结果

def hugo_build_push():
    commit_string = input('Please input commit message:')
    os.chdir('e:/Hugo/JasonBlog')
    os.system('hugo --baseUrl="http://jasongan.xyz"')   #hugo渲染生成静态网页
    os.chdir('e:/Hugo/JasonBlog/public')
    os.system('git add .')                              #开始push

    os.system('git commit -m"{}"'.format(commit_string))
    os.system('git push')
    os.system('pause')  

if __name__ == '__main__':
    new_md()
    hugo_build_push()
```  

* 这个脚本不复杂，把你要执行的命令按条例列出来即可，本来我想要实现多进程调用命令行的，就是同时进行两个cmd输入，但是写了多进程的程序并执行后，电脑就会死机！不知道是不是我把它编译成可执行文件的原因，在pycharm执行倒是相安无事，有空再优化一下吧，这样已经省事多了，写博客只需要打开我这个exe，然后编辑文本内容即可。


