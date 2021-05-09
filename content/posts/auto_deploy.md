---
title: "Webhooks自动部署网站"
date: 2019-07-09T00:09:58+08:00
draft: false
tags: ["ssh","Webhook","自动部署","GitHub","Golang","Node.js","js","shell"]
categories: ["中间件相关"]
author: "Jason·Gan"
---

* 前言：自动部署的工具有很多，Webhooks只是其中一个，它是一个API的概念，就是一种web的回调（callback）或者说用户定义的HTTP回调接口，是向APP或其他web应用提供实时信息的一种方式。简而言之，就是我从本地仓库push到GitHub上面，Webhooks就会给我的网站服务器发送信息，让服务器自动pull变动到站点根目录下，网站就自动更新了。

## 优点  
* 每次更新网站的东西，都要经过几个重复又繁琐的步骤：本地编辑->本地push到GitHub->ssh连接站点服务器->服务器git pull远程仓库的变动，看着没什么，但是每次都这样重复甚至过程中还要输入账号密码就有点难受，用Webhooks自动部署就省略了后面的pull,懒人是绝对忍受不了这样的重复无聊的动作的，所以无论如何我都要搞定这个自动部署。  
* 学习新知识  

## 准备工作
* 需要一个服务器外网能够访问它，准备好你的网站
* 你的web服务器要自己配置好，例如Ngixn/Apache
* GitHub账号和仓库，其他代码仓库或者主机都大同小异，以此类推
* 响应GitHub上post过来的请求的服务器Webhook后端（本文用第三方库 github-webhook-handler）
* ssh连接你的服务器  

## 开始动手  
### SSH:public key and private key  

* 在你的服务器上，安装配置好git
* 创建一个新用户，不能用root，此处用户名为www-data
* 为web服务器生成SSH目录 `sudo mkdir /var/www/.ssh`  
* 修改文件夹权限 `sudo chown -R www-data:www-data /var/www/.ssh/`  
* 为刚才新建的用户生成SSH密钥，用与连接远程仓库，SSH连接可以避免每次都要输入密码的尴尬，输入`sudo -Hu www-data ssh-keygen -t rsa`,接下来会提示你填写一些信息，把目录填上面生成的.ssh路径就行，如/var/www/.ssh，其他参数随意
* 一旦生成密钥成功，可以到密钥所在目录里查看一下，或者直接敲`sudo cat /var/www/.ssh/id_rsa.pub`,并把公钥内容复制粘贴到GitHub的设置里，找到SSH，粘贴。如图  
![](/image/auto1.jpg) 
* 再回到你的服务器上，进入网站目录，`cd /var/www`
* 修改一下你的网站的文件夹权限，`sudo chown -R www-data:www-data /var/www/[site_dir]`
* 切换一下用户`sudo su www-data`
* SSH连接GitHub关键一点，把私钥添加到你的SSH客户机上，否则待会请求远程仓库时会报“Permission denied (publickey).”的错误。敲命令，`ssh-add /var/www/.ssh/id_rsa`。注：就是把你刚才生成密钥的私钥路径填进去，私钥是没有.pub后缀的那个文件。如果报错“Could not open a connection to your authentication agent”的话就是还没有启动ssh客户端，手动启动一下：eval \`ssh-agent -s`  
* 如果你的系统每一次都得重启ssh-agent的话，说明得手动配置一下，加一个shell脚本，让它每次自启动ssh-agent，修改你的文件：```.bash_profile```  
```  
SSH_ENV="$HOME/.ssh/environment"

function start_agent {
    echo "Initialising new SSH agent..."
    /usr/bin/ssh-agent | sed 's/^echo/#echo/' > "${SSH_ENV}"
    echo succeeded
    chmod 600 "${SSH_ENV}"
    . "${SSH_ENV}" > /dev/null
    /usr/bin/ssh-add;
}

# Source SSH settings, if applicable

if [ -f "${SSH_ENV}" ]; then
    . "${SSH_ENV}" > /dev/null
    #ps ${SSH_AGENT_PID} doesn't work under cywgin
    ps -ef | grep ${SSH_AGENT_PID} | grep ssh-agent$ > /dev/null || {
        start_agent;
    }
else
    start_agent;
fi
```  
* 最后重载一下这个文件就行:``` source ~/.bash_profile```  

### 配置服务器上Webhook后端    

* clone一下远程仓库的东西过来，本地已经有了的话直接复制过来就行。`git clone git@github.com:[githubuser]/[gitrepo].git /var/www/[site_dir]`,这个方法clone 就是建立SSH连接    
* github-webhook-handler,这是一个基于Node.js的第三方库，相当于一个处理器或者叫做中间件，可以帮你处理各种从GitHub传过来的Webhook请求，小型的后台服务。
* 这个自动部署的主要原理就是写一个脚本，把git pull写进去，当GitHub传来请求时，自动触发这个脚本，就是自动部署了，所以先把这个关键又简单的shell脚本auto_deploy.sh写出来:(因为我的是静态网页，所以没什么别的操作，就一个git pull)  

```  
#! /bin/bash
SITE_PATH='/var/www/Gan-Blog'
cd $SITE_PATH
git clean -f
git pull
```  

* 接下来就是实现服务端，在auto_deploy.sh脚本同目录下，初始化一个package.json:  
``` npm init -f```  
* 在同目录下安装第三方库：github-webhook-handler.  
``` npm i -S github-webhook-handler```  
* 同目录下，新建一个js脚本名为index.js并写入以下内容：  

```  
var http = require('http');
var spawn = require('child_process').spawn;
var createHandler = require('github-webhook-handler');

// 下面填写的screct跟github webhooks配置一样，后面会说这个的重要性；path是我们访问的路径
var handler = createHandler({ path: '/var/www', secret: 'gan' });
//
 http.createServer(function (req, res) {
   handler(req, res, function (err) {
       res.statusCode = 404;
       res.end('came in this point');
             })
             }).listen(6666);           //监听端口你随意，只要不和服务器上的端口冲突就行

             handler.on('error', function (err) {
               console.error('Error:', err.message)
               });

               // 监听到push事件的时候执行我们的自动化脚本
            handler.on('push', function (event) {
                console.log('Received a push event for %s to %s',
                event.payload.repository.name,
                event.payload.ref);

            runCommand('sh', ['./auto_build.sh'], function( txt ){
                    console.log(txt);
                });
            });

function runCommand( cmd, args, callback ){
    var child = spawn( cmd, args );
    var resp = '';
    child.stdout.on('data', function( buffer ){ resp += buffer.toString(); });
    child.stdout.on('end', function(){ callback( resp ) });
}
```  
**注1：secret参数一定要填写，不填写的话GitHub上post过来的请求是不带有X-Hub-Signature这个参数的，但是github-webhook-handler这个库又必须检测这个参数，所以就会报错：{"error":"No X-Hub-Signature found on request"}。这个问题我也被困了大半天，最后才发现要设置secret才带这个参数。**  

**注2：这个js脚本一开始是我从网上改的别人写的，那真的是bug连篇，函数名、变量名都写错，真的是恶心的不行，以后看网上的资料一定要仔细点看，不然就是坑死自己。**  

* 利用Node管理工具pm2让这个服务跑起来，先安装pm2:  ```npm install pm2 -g```  

* 启动服务:```pm2 start index.js``` 

* 这时候服务跑起来了，会出现下图情况:  

![](/image/pm2.jpg)  

* 接下来就是配置nginx的反向代理：  

```  
    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  jasongan.xyz www.jasongan.xyz ;
        root         /var/www/Gan-Blog;
        # Load configuration files for the default server block.

         error_page 500 502 503 504 /50x.html;
            location = /50x.html {
       }
            //这里定位到你的index.js所在目录          
            location /var/www {             
                proxy_pass http://127.0.0.1:6666;
       }

```  
### Github Webhook  
* 配置GitHub上的Webhook，在你仓库的setting里面，有一个Webhook选项，添加一个Webhook，把你index.js路径接上你的域名填到Payload URL里：  
![](/image/webhook.jpg)  

* 其他参数默认即可，记得勾上activated。
* 随心所欲push吧









