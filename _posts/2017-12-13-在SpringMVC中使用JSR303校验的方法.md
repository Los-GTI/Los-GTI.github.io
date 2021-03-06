---
layout:     post
title:      在SpringMVC中使用JSR303校验的方法
subtitle:   JSR303校验
date:       2017-12-13
author:     Los-GTI
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Spring
---

#### 1.前言

参数校验是我们程序开发中必不可少的环节。用户在前端页面上填写表单时，前端js程序会校验参数的合法性，但是只有前端校验并不能完全保证数据的合法性，当数据到了后端，为了防止一些别有心机的人的恶意操作，保持程序的健壮性，我们在后端同样要对重要数据进行校验。后端参数校验最简单的做法是直接在业务方法里面进行判断，当判断成功之后再继续往下执行。但这样带给我们的是代码的耦合，冗余。当我们多个地方需要校验时，我们就需要在每一个地方调用校验程序,导致代码很冗余，且不美观。

> 那么如何优雅的对参数进行校验呢？JSR303就是为了解决这个问题出现的，本篇文章主要是介绍 JSR303，Hibernate Validator 等校验工具的使用，以及自定义校验注解的使用。

#### 2.校验框架介绍

JSR303 是一套JavaBean参数校验的标准，它定义了很多常用的校验注解，我们可以直接将这些注解加在我们JavaBean的属性上面，就可以在需要校验的时候进行校验了。注解如下：

```
 @NotNull          注解元素必须是非空
 @Null             注解元素必须是空
 @Digits           验证数字构成是否合法
 @Future           验证是否在当前系统时间之后
 @Past             验证是否在当前系统时间之前
 @Max              验证值是否小于等于最大指定整数值
 @Min              验证值是否大于等于最小指定整数值
 @Pattern          验证字符串是否匹配指定的正则表达式
 @Size             验证元素大小是否在指定范围内
 @DecimalMax       验证值是否小于等于最大指定小数值
 @DecimalMin       验证值是否大于等于最小指定小数值
 @AssertTrue       被注释的元素必须为true
 @AssertFalse       被注释的元素必须为false
Hibernate validator  在JSR303的基础上对校验注解进行了扩展，扩展注解如下：
 @Email             被注释的元素必须是电子邮箱地址
 @Length            被注释的字符串的大小必须在指定的范围内
 @NotEmpty          被注释的字符串的必须非空
 @Range             被注释的元素必须在合适的范围内

```

#### 3.代码实现

> 这里我给出的实例是基于SpringMVC的使用JSR303校验的代码实现，当然还有其他方法，这个实例是用户新增的表单校验。

##### 3.1 添加jar包依赖

在pom.xml中添加hibetnate-validator包

```
        <!-- hibernate-validator后端校验 -->
		<dependency>
			<groupId>org.hibernate.validator</groupId>
			<artifactId>hibernate-validator</artifactId>
			<version>6.0.5.Final</version>
		</dependency>
```
##### 3.2在bean中添加校验注解

![](https://i.imgur.com/artbofk.png)

> 这里我是对empName和email进行校验，我使用的是正则表达式的方式。

##### 3.3 controller中的校验方法

![](https://i.imgur.com/8aFpp6f.png)

> 注意：这里一个@Valid的参数后必须紧挨着一个BindingResult 参数，否则spring会在校验不通过时直接抛出异常。

这里我将校验不通过产生的错误信息存到map中返回到前台页面。

##### 3.4前台jsp页面将校验不通过的相关信息展示出来。

![](https://i.imgur.com/Mk8OwTB.png)

> 备注：这里有一些我已经封装好的方法，有些人可能看不太懂，我这里讲一下大体思路，就是使用ajax请求，将存在map中返回到浏览器的校验错误的信息使用result接收，然后再将相关的错误信息展示出来，当然方法很多，大家自己发挥。

##### 3.4 效果展示

![](https://i.imgur.com/XyMmgxk.png)

备注：运行之前我已经把相关的前端的js校验的代码注释掉了，这里展示的是我们的后端校验，我们在控制台中看一下打印出来的提示信息也可以看出来是我们的后端校验。

![](https://i.imgur.com/V8uvdgO.png)

#### 4. 总结

这样的话我们就比较容易在前后端都进行参数的校验，这样的话就避免了一些别有用心的人跳过你的前端校验输入一些不法字符造成一些问题，对于一些比较重要的数据我们前后端都要进行校验，这样才会比较安全，我们就是要把校验进行到底！当然今天写的这点东西只是基于SpringMVC使用JSR303进行校验，还有其他实现方法，等我下次再用到的话再来写一下吧。

> 摁钉 author by Los-GTI
