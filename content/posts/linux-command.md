---
title: "常用的Linux基础命令"
date: 2019-11-01T10:51:37+08:00
draft: false
categories: ["Linux相关"]
tags: ["Linux","基础命令","shell","操作系统"]
author: "Jason·Gan"
---


## 命令的基本格式  

* [root@localhost~]
* root为用户名
* ～表示当前所在位置
* localhost主机名
* '#'超级用户
* '$' 普通用户  

**命令基本格式：命令 [选项] [参数]**  

## 查询目录的内容  
**命令格式：ls [选项] [文件或者目录]**  
* 选项：
  * -a 所有文件
  * -l 查看详情
  * -d查看目录属性
  * -h显示文件大小

## 文件处理命令  
### 建立目录  
* mkdir -p [目录名],(注：-p表述递归建文件夹)  
### 切换目录
* cd [目录]  
* cd ~ 进入当前用户目录
* cd - 上次目录
* cd .. 进入上一级目录
* pwd 查看当前目录所在位置  
### 删除目录  
* rmdir [目录] 
### 删除目录下所有文件 
* rm -rf [目录]  
### 复制目录  
* cp[选项][原文件目录][目标目录]
* 选项：
  * -r复制目录
  * -p连文件属性一起复制
  * -a 相当于-pdr  
### 剪切、改名  
* mv[原文件目录][目标文件目录]
### 常见目录的含义  
* / 根目录
* /bin 命令保存目录
* /boot 启动目录
* /dev 设备文件命令
* /etc 配置文件保存目录
* /home 家目录
* /lib 系统库保存命令
* /mnt 系统挂载目录  
## 文件搜索命令  
* locate [文件名]  
### 命令搜索  
* which [选项][命令名]
* 选项：
  * -b 只查找可执行文件
  * -m 只查找帮助文件  
### 文件搜索  
* find [搜索范围][选项][条件]  
* 举例子：   
  * 在根目录下查找名为install.log文件:find  /  -name install.log  
  * 忽略大小写查找文件:find /root  -inname install.log
  * find /var/log -mtime +10
    其中-mtime 文件修改时间
    -atime 文件访问时间
    -ctime 改变文件属性时间
    +10 10天前
    10  10天
    -10 10天内
  * 查找文件大于20M的文件:find /etc -size +20M  
## 压缩与解压缩命令  
* 常见压缩格式：(.zip),(.gz),(.bz2),(.tar.gz),(.tar.bz2)  
### zip格式  
* 压缩文件：zip [压缩文件名][原文件]  
* 压缩目录：zip -r  [压缩文件名][原文件]  
* 举例子：  
  * touch jp/cangls
  * touch jp/longls
  * zip -r jp.zip jp  
* 解压：unzip [压缩文件名]  
### gz格式  
* 压缩为.gz格式，源文件不保留：gzip [文件名]  
* 压缩为.gz格式，源文件保留：gzip -c [原文件] > [压缩文件]  
* 压缩目录：gzip -r  [目录]  
* 直接解压：guzip [文件]  
* 解压为目录：guzip -r [目录]  
### tar格式  
* 压缩：tar -cvf [打包文件名] [原文件]  
* 解压：tar -xvf [文件名]  
### tar.gz格式  
* 压缩：tar -zcvf [打包文件名] [原文件]  
* 解压：tar -zxvf  [压缩包名.tar.gz] 
## 关机和重启  
### 关机  
* shutdown [选项][时间]

* 选项：
  *  -c 取消前一个关机命令
  *  -h 关机
  *  -r 重启  
* 重启：init 6  
* 关机：init 0  
* 退出登陆：logout  
## 查看用户信息  
* w  
* who  
* last  
* lastlog
* 都敲一遍就知道是什么格式的命令了  
## shell基础  
* shell是命令行解释器，逻辑上是一层壳，是用户与操作系统内核进行交互的接口。  
* 输出：echo[选项][输出内容]  
* 选项：
  * -e:支持转义  
### 创建一个sh脚本  
```
# 进入编辑(如果文件不存在，则新建文件并进入编辑该文件)：
vim hello.sh

#!/bin/bash
# the first program

echo "hello world!" 
```  
### 执行脚本  
```
chmod 755 hello.sh

./hello.sh
```
* 或者  
```
bash hello.sh
```
## 历史命令  
* 查看历史命令：history  
* history [选项][历史保存文件]  
* 选项：  
  * -c 清除历史  
  * -w 把缓存中的历史写进历史命令保存文件  
## 输出重定向  
* 命令>文件   以覆盖的方式，把正确的命令输出到指定文件
* 命令>>文件  以追加的方式，把正确的命令输出到指定文件
* 错误命令 2> 文件  以覆盖的方式把错误的命令覆盖到指定文件
* 错误命令 2>> 文件  以追击的方式把错误的命令覆盖到指定文件
* 命令>>文件 2>&1  同时追加正确命令和错误命令到 指定文件
* 命令 >>文件1 2>>文件2   把正确的命令输出到文件1错误的文件2  
* 举个例子：  
```
#/bin/bash
while command
do
curl -d "userid=123343,action=getuserinfo" https://app.testasss.com.cn/ >test.txt
time=$(date)
cat=$(cat test.txt |cut -d \: -f 3|cut -d \" -f 2)
echo "$time $cat" >>test.log
sleep 15s
done  
```  
## 网络相关  
* 查看ip：ifconfig  














