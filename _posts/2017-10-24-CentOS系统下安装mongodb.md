---
layout:     post
title:      CentOS系统利用yum源安装mongdb数据库
subtitle: 系统版本CentOS7
date:       2017-10-24
author:     QC
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - CentOS
---

### CentOS系统下利用yum安装mongodb

##### 1.准备工作

打开终端输入：cd  /etc/yum.repos.d/，查看自己的机器中的yum源中有没有包含mongodb的相关资源，我机器中的yum源信息如下图，没有mongodb的相关资源。

![是否mongodb相关资源](F:\typora_md\CentOS系统安装mongodb\是否mongodb相关资源.png)

所以要在使用yum命令安装mongodb前增加yum源，也就是在 /etc/yum.repos.d/目录中增加 *.repo yum源配置文件，以下是针对centos 64位系统的MongoDB yum 源配置内容(32位系统略有不同)：  

我们这里就将该文件命名为：/etc/yum.repos.d/mongodb-3.4.repo 

For 64-bit yum源配置：  

vi /etc/yum.repos.d/mongodb-3.4.repo

```
[mongodb-org-3.4]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.4/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc
```

  ![yum源配置](F:\typora_md\CentOS系统安装mongodb\yum源配置.png)

做好yum源的配置后，下面就可以执行相关命令安装mongodb及相关服务了。

##### 2.安装mongodb及相关服务

###### 2.1 安装mongodb

运行命令：yum -y install mongodb-org

![install1](F:\typora_md\CentOS系统安装mongodb\install1.png)

![install2](F:\typora_md\CentOS系统安装mongodb\install2.png)

![install3](F:\typora_md\CentOS系统安装mongodb\install3.png)

> 出现complete说明安装成功。

###### 2.2 mongodb相关配置

运行命令：vi /etc/mongod.conf 查看修改相关配置

![相关配置](F:\typora_md\CentOS系统安装mongodb\相关配置.png)

fork=true   允许程序在后台运行

logpath=/var/lib/mongod.log logappend=true 写日志的模式：设置为true为追加。默认是覆盖

 dbpath=/var/lib/mongo数据存放目录

pidfilepath=/var/run/mongodb/mongod.pid 进程ID，没有指定则启动时候就没有PID文件。默认缺省。
port=27017，默认端口

> 存放日志和数据存放目录和端口等信息可以修改，我这边没有修改。

##### 3.运行mongodb

启动mongod ：systemctl start mongod.service

>注意：外网访问需要关闭防火墙：
>
>CentOS 7.0默认使用的是firewall作为防火墙，这里改为iptables防火墙。
>
>关闭firewall：systemctl stop firewalld.service #停止firewall
>
>systemctl disable firewalld.service #禁止firewall开机启动

使用mongodb ： mongo 127.0.0.1:27017

![进入mongo服务](F:\typora_md\CentOS系统安装mongodb\进入mongo服务.png)

![测试](F:\typora_md\CentOS系统安装mongodb\测试.png)

停止mongod ：systemctl stop mongod,service

> 摁钉： author ：chongqin







