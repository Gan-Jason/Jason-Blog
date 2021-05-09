---
title: "在服务器上用nginx部署Hugo"
date: 2019-07-08T00:56:46+08:00
draft: false
categories: ["中间件相关"]
tags: ["Python","nginx","https","TSL","go","ssh"]
author: "Jason·Gan"
---

* **前言：Hugo是由Go语言实现的静态网站生成器。简单、易用、高效、易扩展、快速部署。
用Hugo来构建个人博客是简单快捷的，今天我们把他通过nginx部署在服务器上。**

## 1.准备工作
* 服务器  
我用的是GCP（Google cloud platform），个人博客的话随便租一个小型的就行，要有一个静态的IP；性能要求不高，毕竟又不是大流量小鲜肉的博客。
* 域名  
我的域名是.xyz的，条件宽裕的可以租一个好看一点的比如.com，租好域名后，在域名提供商那自己配置好DNS服务器，就是把租来的域名和租来的服务器通过DNS服务商绑定在一起，都是租来的，只有知识是自己的了。
* 静态文件  
Hugo生成静态网站，也就是通过命令<hugo --theme=主题名  --baseUrl="你的网站域名">生成一个public文件夹，请保管好你这个文件夹宝贝。
* Nginx  
Nginx (engine x) 是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务。Nginx支持热部署，高并发，内存消耗低，具有很高的可靠性。先安装好Nginx，举例centos:yum install nginx,然后配置Nginx。  
## 2.开始配置
* TSL  
TSL是SSL的继承者，TLS提供了更强大和效率更高的HTTPS（网络应用层协议),且它包含了SSL所没有的改进例如正向加密，兼容现代openSSL密码套件和HSTS。在配置Nginx前，得先申请一个你的网站的TSL证书和密钥
* 生成证书  
这个证书如果是用于你私人或内部网站，可以自己设计；如果是商业站点，就得去正规的证书颁发机构去申请。我这里介绍的是个人使用的不经过机构认证的证书生成方法。

>1.新建存储证书文件夹：`mkdir /root/certs && cd /root/certs`    注：切换为root用户  
>2.生成证书：`openssl req -new -newkey rsa:4096 -x509 -sha256 -days 365 -nodes -out MyCertificate.crt -keyout MyKey.key`注：参考图1  
>3.修改文件夹的权限:`chmod 400 /root/certs/MyKey.key`

![图1](/image/certs.jpg)<p align="center">图1</p>  

* 配置Nginx.conf  
cd到Nginx目录，vim修改Nginx.conf配置文件，我就直接上配置了  

![](/image/nginx_conf.jpg)<p align="center">图2</p>
* 重新加载Nginx服务  
配置文件中，root参数是指向你的public文件的，而这个public文件可以托管在Github上面，然后gitclone下来（本文这个不是重点，自行搜索）最后重新加载Nginx`nginx -s reload`，请在浏览器输入你的域名体验逛自己网站的乐趣吧~  

#### 注：Hugo配置文件中有一点需要注意，在config.toml中（具体看你自己的主题）需要配置Url，这里可以写http协议头，也可以写https协议头，但是在类似上面的Nginx配置中就要相应的设置了，不然会出现连接错误，有什么问题就在评论区留言吧。

