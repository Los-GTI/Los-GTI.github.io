---
layout:     post
title: centOS7系统安装mysql并开启远程访问权限
subtitle: centOS7，mysql
date:       2017-12-30
author:     QC
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - mysql
---
### CentOS7系统mysql安装及开启远程访问权限

##### 1.前言
因为centOS7上安装mysql数据库和centOS6上有比较大的不同，我刚开始的时候没有意识到这一点，看了好多网上的错误的资料和解决办法，导致我走了一些弯路，所以我写下这篇安装过程一是为了记录我的学习经历，二是希望可以帮助到小白不走弯路！好了废话不多说，下面就开始安装mysql吧，再次强调本文是centOS7系统，centOS6系统会有较大不同！
>注： 使用yum安装
##### 2.准备工作
centOS7和centOS7.1系统中，默认安装的是它的分支mariadb数据库，以前我们在centOS6中安装mysql的许多操作都是无效的。首先我们先把centOS7中默认安装的mariadb数据库卸载(用以下命令)：
`yum remove mariadb*`
卸载完成后如下图：
![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/卸载mariadb.png)

##### 3.mysql数据库安装
###### 3.1 下载mysql的repo源
CentOS7的yum源中默认是没有mysql的。为了解决这个问题，我们首先要先下载mysql的repo源：`wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm`

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/添加mysqlrepo源.png)
###### 3.2 安装mysql-community-release-el7-5.noarch.rpm包
命令：`sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm`
![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/安装mysqlrpm包.png)

安装这个包后，会获得两个mysql的yum repo源：/etc/yum.repos.d/mysql-community.repo，/etc/yum.repos.d/mysql-community-source.repo（如下图）。
![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/多两个mysqlrepo源.png)
###### 3.3 安装mysql
命令：`sudo yum install mysql-server`
根据步骤安装就可以了，不过安装完成后，没有密码，需要重置密码。
![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/mysql安装成功.png)
###### 3.4 重置密码
首先启动mysql：`service mysql start`
重置密码前首先要登录：`mysql -u root -p`
这时候登录是没有密码的，所以我们要修改一下密码：
![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/mysql无密码登录.png)
```
mysqladmin -u root password 'root'
```
> 密码重置成功，下次登录就需要这个密码了！

##### 4.开启远程访问权限
以上部分已经安装好mysql并将密码重置，下面我们想通过Navicat工具来远程操作mysql，如果你做什么操作直接连接的话，毫无疑问是连接不上的，我们要通过配置防火墙的一些配置并且开放3306端口来实现远程连接：
> 这里注意，网上一些配置什么iptables的方法并不适用于centos7，如果你是centOS7的话请不要去相信网上的什么通过vim配置iptables的一些方法，下面我来说一下centos7下怎么去开启远程访问权限。

CentOS7这个版本的防火墙默认使用的是firewall，与之前的版本使用iptables不一样。按如下方便配置防火墙：
- 1、关闭默认防火墙：`sudo systemctl stop firewalld.service`
- 2、关闭开机启动：`sudo systemctl disable firewalld.service`
- 3、安装iptables防火墙：`sudo yum install iptables-services`
- 4、修改防火墙配置文件：`vi /etc/sysconfig/iptables `
- 5、加入端口配置，开放3306端口：
`-A INPUT -p tcp -m state --state NEW -m tcp --dport 3306 -j ACCEPT`
![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/新增3306端口.png)
- 6、重新加载规则：`service iptables restart  `
> 这样配置完成后发现还是远程连接不了，然后继续配置！

- 7、 给root用户授权：`grant all privileges on *.* to root@'%'identified by 'root';`
- 8、刷新配置：`flush privileges; `
这样就大功告成了！！！

> 摁钉：author by chongqin
