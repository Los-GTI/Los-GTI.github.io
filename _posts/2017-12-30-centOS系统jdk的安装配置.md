---
layout:     post
title:  centOS7系统jdk的安装与配置
subtitle:  centOS7,jdk
date:       2017-12-30
author:     Los-GTI
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - jdk
---

### centOS7系统jdk的配置安装
##### 1.安装jdk

###### 1.1 添加卸载程序

- windows控制面板，添加/卸载程序，进行程序的安装、更新、卸载、查看
- linux：rpm命令，相当于windows的添加/卸载程序，进行程序的安装更新、卸载、查看
  - 本地程序安装：`rpm -ivh 程序名`
  - 本地程序查看：`rpm -qa`
  - 本地程序卸载：`rpm -e --nodeps 程序名`
- linux：yum命令，相当于可以联网的rpm命令，相当于先联网下载程序安装包、程序的更新包然后自动执行rpm命令。

###### 1.2 安装下载依赖

因为jdk，tomact，mysql的安装过程中需要从网上下载部分支持包才可以继续，所以要求提前下载安装好依赖。

```
yum install glibc.i686
yum -y install libaio.so.1 libgcc_s.so.1 libstdc++.so.6
yum update libstdc++-4.4.7-4.e16.x86_64
yum install qcc-c++
```
![](https://i.imgur.com/CLmNBYy.png)

> 依次根据命令安装以上基本依赖，过程不再赘述。

###### 1.3 卸载机器上自带的jdk

检查本机上是否携带jdk，发现本机上携带openjdk，我们在windows上安装的jdk1.7啊jdk1.8啊都属于Oracle jdk，这两个jdk只有版权的区别，基本一样。
你如果想直接使用自带的jdk也是没有问题的.
> 因为我需要手动的部署一下Java环境的安装，所以我们在装jdk之前将原有版本的jdk删掉。

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/openjdk.png)

上图可以看到我们机器上自带了openjdk，下面我们把他删掉然后重新安装jdk。删除之后效果图如下：

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/卸载jdk.png)

###### 1.4  安装jdk

因为我下载的jdk文件是压缩包，首先将其解压，我把它解压到/usr/local/java文件夹下，如下图（解压之前先创建/usr/local/java文件夹）：
新建文件夹命令：`mkdir -p /usr/local/java`

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/jdk解压成功.png)

下面查看一下解压后的Java文件夹中的目录，如下图可以看到和我们在windows上的目录没什么区别：

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/解压后目录.png)

可以看到我们已经解压成功了，下面去配置环境变量，centos系统的环境变量配置在/etc/profile,我们使用`vim /etc/profile`命令配置环境变量

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/设置环境变量.png)

然后用`source /etc/profile`重新加载配置文件，然后我们检验一下Java环境有没有安装成功。命令`java -version`

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/javaVersion.png)

如上图我们输入`java -version`命令可以看到我们的java的版本号，说明我们的jdk安装配置成功了。

> 摁钉：author by Los-GTI
