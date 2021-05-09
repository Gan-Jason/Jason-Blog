---
title: "关于win10上安装docker一些坑"
date: 2019-06-30T22:44:53+08:00
draft: false
tags: ["Python","这位同学很帅什么也没留下","docker","win10","splash"]
categories: ["中间件相关"]

author: "Jason·Gan"
---
* 前言：前些天学习到python爬虫遇到有一些网站是动态加载的，也就是ajax异步请求内容，所以需要用到一些渲染的功能，故学习了一波splash，但是splash是只安装在容器中的，因此有了此文--安装docker  


## Docker安装
**win10下docker的安装有两种方式：**


* Hyper-V + docker镜像  
这种方式需要打开Hyper-V功能，也就是windows下的虚拟机服务，但是只有专业版的win10才支持这个功能，家庭版的并不支持（我的就是家庭版...），这个方式的安装应该不存在什么大问题，我也还没尝试过，略。 
也有一个方法，家庭版打开Hyper-V,需要伪装成专业版，修改一下注册表，这个方法我试过，但是后面还是报错，伪装不成功
* docker toolbox  
docker toolbox是一个工具集，包括了：  
1.Docker CLI 客户端，用来运行docker引擎创建镜像和容器  
2.Docker Machine. 可以让你在windows的命令行中运行docker引擎命令  
3.Docker Compose. 用来运行docker-compose命令  
4.Kitematic. 这是Docker的GUI版本  
5.Docker QuickStart shell. 这是一个已经配置好Docker的命令行环境  
6.oracle VM Virtualbox. 虚拟机  
由于Docker引擎的守护进程使用的是Linux的内核，所以我们不能够直接在windows中运行docker引擎。而是需要运行Docker Machine命令 docker-machine， 在你的机器上创建和获得一个Linux虚拟机，用这个虚拟机才可以在你的windows系统上运行Docker引擎。

**开始安装**  

* 先打开任务管理器->性能 查看是否开启了虚拟化，若没开启，则打开浏览器，用谷歌搜索一下如何开启（我的默认开的，先溜）  
* 打开<https://download.docker.com/win/stable/DockerToolbox.exe>,下载toolbox
* 双击安装
* 一路next
* 桌面出现三个小东西   
 ![](/image/docker.jpg)  
* 其中有个问题，安装好后，需要手动把安装目录下的一个boot2docker.iso镜像文件放到此目录下：C:/User/xxx(用户名)/.docker/machine/machines/default 里，否则Oracle虚机启动时报错找不到镜像文件，如果该路径已存在镜像文件则不需要
* 还有一点，toolbox的虚拟机需要开一个网络适配器，虚拟一个ip，一般都是：192.168.99.100，当报错说你的ip地址不存在或者有问题时，检查网络适配器，类似下图  
![](/image/docker_net.jpg)  是否启动且正常，如果你的本机ip分配的动态ip且网段不是192.xx...的，会报错，因为你只能给虚拟机分配同网段的ip，就不是docker默认的192.168.99.100了，我在公司的时候试过，公司的网段是10.开头，解决办法：开手机热点
* 此外用powershell 也可以连docker的虚拟环境，不一定要开Docker Quickstart Terminal
* 用命令docker-machine start default启动默认docker虚机，命令docker-machine env 可查看你的虚拟机ip，检查是否为192.168.99.100
* powershell连接docker，敲命令ssh docker@192.168.99.100
* Docker Quickstart Terminal连接docker虚拟机，敲命令docker-machine ssh <docker machine name>
