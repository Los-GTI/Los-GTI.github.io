---
layout:     post
title:  向Tomcat服务器发送PUT和DELETE请求一直报403错误的问题
subtitle: PUT和DELETE请求
date:       2017-12-16
author:     Los-GTI
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Java
---

#### 向Tomcat服务器发送PUT和DELETE请求一直报403错误的解决方法

> 前言：我在做修改功能的时候向tomact服务器发送了PUT请求，一直报403拒绝访问错误。搞了半天不知道怎么回事，后来才知道tomact默认就不支持PUT和DELETE请求，下面我提供一下解决这个问题的方法，让tomcat服务器支持PUT和DELETE请求。

#### 解决办法

说来也简单，找到你tomcat文件夹下conf文件夹下的web.xml文件，在配置org.apache.catalina.servlets.DefaultServlet的时候加上这几行代码：
```
<init-param> 
   <param-name>readonly</param-name> 
   <param-value>false</param-value> 
</init-param> 
```
![](https://i.imgur.com/F9wMv3M.png)

readonly参数默认是true，即不允许delete和put操作，所以通过XMLHttpRequest对象的put或者delete方法访问就会报告http 403错误。为REST服务起见，应该设置该属性为false。

完美解决！！！

摁钉：author by Los-GTI
