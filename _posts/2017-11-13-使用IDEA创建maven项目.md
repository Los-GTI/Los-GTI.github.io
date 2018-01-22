---
layout:     post
title:      使用InetlliJ IDEA创建maven项目
subtitle:   IDEA-maven
date:       2017-11-13
author:     Los-GTI
header-img: img/home-bg-art.jpg
catalog: true
tags:
    - IDEA
---
### 使用IntelliJ IDEA创建maven项目

#### 1.前言

作为一个刚刚从eclipse转战到IDEA的菜鸟，IDEA用起来可谓是真不顺手，正好最近要做一个maven项目，就想着这次用IDEA来开发，不用eclipse了，首先要做一个创建maven项目的例子供大家参考。

#### 2.创建过程

##### 2.1 首先你的电脑里要安装maven，maven只需安装一次，无论以前你是用哪个开发工具，只要你安装过maven就无需再安装，还有就是配置环境变量，这两项工作我已经做过，这里就不再做重复工作，maven的版本和环境变量的path放上来。

![](https://i.imgur.com/LW9rhxa.png)

![](https://i.imgur.com/WcLhQn5.png)

![](https://i.imgur.com/HJ9O2hy.png)

maven安装成功了，maven的环境变量也配置好了，下面我们开始我们使用IDEA的第一个maven项目吧！

##### 2.2 创建maven项目详细过程

file——>new project——>选择maven，勾选Create from archetype——>选择maven-archetype-webapp——>next

![](https://i.imgur.com/MQMI0zt.png)

输入groupid和artifactId——>next

![](https://i.imgur.com/knmoZbz.png)

选择maven版本，这里我选择的是本地的maven——>next

![](https://i.imgur.com/fDMg01l.png)

输入项目location，和工程名称——>finsh,效果如下

![](https://i.imgur.com/soI9YVh.png)

open Modules settings——> 检查下面两个圈出来的部分，第一个部分的路径是不是项目的web.xml的路径，第二个部分的路径是不是webapp的路径
不是的话修改

![](https://i.imgur.com/ehLCI2k.png)

选择source——>新建Java文件夹，将其设置为source，设置完之后文件夹变蓝

![](https://i.imgur.com/6UfKMqI.png)

最后配置tomact服务器，打开edit configuration——>点击加号——>选择tomact——>local——>配置tomact服务——>点击deployment——>添加项目的artifacts——>确定

![](https://i.imgur.com/kVRamRs.png)

![](https://i.imgur.com/azKiWBb.png)

![](https://i.imgur.com/wYBdlKM.png)

##### 2.3 运行项目

run

![](https://i.imgur.com/2DHRsgF.png)

![](https://i.imgur.com/42dcU0o.png)

> 熟悉的hello world
> 摁钉 author by Los-GTI
