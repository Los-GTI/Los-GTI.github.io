---
layout:     post
title:      Linux系统安装MySQL数据库
subtitle:   Ubuntu14.04系统
date:       2017-10-20
author:     QC
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - mysql
---
### linux系统下安装MySQL数据库
> 博主是ubuntu14.04系统

#### 1.前言
MySQL是Oracle公司的一种开放源代码的关系型数据库系统，被广泛应用于各种中小网站，是一种跨平台的数据库管理系统，现在介绍一下如何在Ubuntu14.04上安装和配置MySQL。

#### 2.安装过程

##### 2.1 更新源列表

打开“终端窗口”，输入“sudo apt-get update” --> "回车" --> "输入root用户的密码" -->"回车"，就可以了。如果不运行该命令直接安装mysql可能会导致mysql的一些包无法下载，导致安装失败。

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/installMysql1.png)

##### 2.2 安装mysql

打开“终端窗口”，输入“sudo apt-get install mysql-server mysql-client” --> "回车" -->"输入Y" --> "回车" --> "在对话框中输入mysql中“root”用户的密码" --> "回车" -->"确认密码" -->”回车“ --> 安装完成

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/installMysql2.png) 
![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/installMysql3.png) 
![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/installMysql4.png) 
![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/installMysql5.png) 

##### 2.3 判断mysql是否安装成功

打开”终端窗口“，输入”sudo service mysql restart“ --> "回车" --> 如果mysql启动成功处于运行状态则说明安装成功。

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/installMysql6.png) 

##### 2.4 让Apache支持mysql

打开”终端窗口“，输入”sudo apt-get install libapache2-mod-auth-mysql“ -->"回车"-->安装成功，安装这个模块后，Apache才能支持mysql。

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/installMysql7.png) 

##### 2.5 登录mysql

打开”终端窗口“，输入”mysql -u root -p“ --> "回车"  --> 输入安装mysql时设置的root用户的密码 --> "回车" -- > 登录成功。

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/installMysql8.png) 

> 摁钉：大功告成。 author:chong qin
