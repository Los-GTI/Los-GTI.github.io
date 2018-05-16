---
layout:     post
title: Spring Boot系列之二
subtitle: 使用Intellij中的Spring Initializr来快速构建Spring Boot/Cloud工程
date:       2018-05-16
author:     Los-GTI
header-img: img/ms1.jpg
catalog: true
tags:
    - SpringBoot
---


> Spring Boot 工程的创建方式多种多样，我们可以通过Maven来手工构建或者是通过脚手架等方式快速搭建，也可以通过我们上一篇博客中提到的`SPRING INITIALIZR`页面工具来创建，大家肯定也有自己喜欢的或者是习惯的构建方式。

**本文主要介绍嵌入在Intellij中的Spring Initializr工具，它同Web提供的创建功能一样，可以帮助我们快速的构建出一个基础的Spring Boot/Cloud工程。**

下面具体说一下构建步骤：

- 菜单栏中选择`File -> New -> Project..`，我们可以看到如下图所示的创建功能窗口。选择`Spring Initializr`,然后我们可以看到`Initial Service Url`的默认地址就是Spring官方提供的Spring Initializr工具地址，所以这里创建的工程实际上也是基于它的Web工具来实现的。

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/SpringBoot/springboot3.png)

- 点击`Next`，等待片刻后，我们可以看到如下图所示的工程信息窗口，在这里我们可以编辑我们想要创建的工程信息。其中，`Type`可以改变我们要构建的工程类型，比如：Maven、Gradle；`Language`可以选择：Java、Groovy、Kotlin。

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/SpringBoot/springboot4.png)

- 点击`Next`，进入选择`Spring Boot`版本和依赖管理的窗口。在这里值的我们关注的是，它不仅包含了`Spring Boot Starter POMs`中的各个依赖，还包含了`Spring Cloud`的各种依赖。

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/SpringBoot/springboot5.png)

- 点击`Next`，进入最后关于工程物理存储的一些细节。最后，点击`Finish`就能完成工程的构建了。

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/SpringBoot/springboot6.png)

`Intellij`中的`Spring Initializr`虽然还是基于官方Web实现，但是通过工具来进行调用并直接将结果构建到我们的本地文件系统中，让整个构建流程变得更加顺畅，还没有体验过此功能的`Spring Boot/Cloud`爱好者们不妨可以尝试一下这种不同的构建方式。



