---
layout:     post
title: Spring Boot系列之一快速入门
subtitle: 快速构建一个Spring Boot项目
date:       2018-05-12
author:     Los-GTI
header-img: img/ms1.jpg
catalog: true
tags:
    - SpringBoot
---


#### 1.Spring Boot简介

Spring Boot是由Pivotal团队提供的全新框架，其设计目的是用来简化新Spring应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。用我的话来理解，就是spring boot其实不是什么新的框架，它默认配置了很多框架的使用方式，就像maven整合了所有的jar包，spring boot整合了所有的框架

#### 2.Spring Boot的优势

其实就是简单、快速、方便！平时如果我们需要搭建一个spring web项目的时候需要怎么做呢？

1）配置web.xml，加载spring和spring mvc
2）配置数据库连接、配置spring事务
3）配置加载配置文件的读取，开启注解
4）配置日志文件
…
配置完成之后部署tomcat 调试
…

现在非常流行微服务，如果我这个项目仅仅只是需要发送一个邮件，如果我的项目仅仅是生产一个积分；我都需要这样折腾一遍!

但是如果使用spring boot呢？
很简单，我仅仅只需要非常少的几个配置就可以迅速方便的搭建起来一套web项目或者是构建一个微服务！

**总结一下Spring Boot的主要优点其实是以下几个：**
- 为所有Spring开发者更快的入门；
- 开箱即用，提供各种默认配置来简化项目配置；
- 内嵌式容器简化Web项目；
- 没有冗余的代码生成和XML配置的要求；

#### 3.快速入门

本章的目标是快速完成Spring Boot基础项目的搭建，并且实现一个简单的Http请求处理，通过这个例子让你对Spring Boot有个初步的了解，并体验其结构简单，快速开发的特性，下面开始吧！

**版本要求：**
- Java7及以上；
- Spring Framework4.1.5及以上；

> 例子采用Java 1.8，Spring Boot2.0.2调试通过

##### 3.1 使用Maven构建项目

**使用SPRING INITIALIZR工具产生基础项目：**

- 访问网址：`http://start.spring.io/`
- 选择构建工具`Maven Project`、Spring Boot版本2.0.2。根据自己的需要修改包名，还可以添加一些依赖，我这里使用默认的包名，也没有添加任何依赖：

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/SpringBoot/springboot1.png)

- 点击`Generate Project`下载项目压缩包

**解压项目包并用IDEA将项目导入：**

- `File -> New -> Project from Existing Sources`,选择解压后的项目文件夹，点击`OK`
- 点击`Import project from external model`并选择Maven，点击`Next`到底；
- 注意一下，我使用的是JDK1.8，注意选择JDK1.7及以上版本；


##### 3.2 项目结构解析

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/SpringBoot/springboot2.png)

通过上面步骤完成了基础项目的创建，如上图所示，Spring Boot的基础结构共三个文件（具体路径根据用户生成项目时填写的Group所有差异）：

- `src/main/java`下的程序入口：DemoApplication
- `src/main/resources`下的配置文件：application.properties
- `src/test/`下的测试入口：DemoApplicationTests

生成的DemoApplication和DemoApplicationTests类都可以直接运行来启动当前创建的项目，由于目前该项目未配合任何数据访问或Web模块，程序会在加载完Spring之后结束运行。

##### 3.3 引入Web模块

当前的`pom.xml`仅仅引入了两个模块：

- spring-boot-starter：核心模块，包括自动配置支持、日志和YAML；
- spring-boot-starter-test：测试模块，包括JUnit、Hamcrest、Mockito；

```
		<!--核心模块，包括自动配置，日志和YAML-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>
        <!--测试模块，包括JUnit，Hamcrest、Mockito-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
```

需要引入Web模块，添加`spring-boot-starter-web`模块：

```
		<!--引入web模块-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
```

##### 3.4 编写HelloWorld服务

在`DemoApplication`同package下创建`HelloController`类，内容如下：

```
@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String index(){
        return "Hello World";
    }
}
```

> 友情提示：如果在写HelloControlller的时候没有@RestController注解提示，需要手动下载添加的web模块依赖；

然后启动主程序，打开浏览器访问`http://localhost:8080/hello`,就可以看到页面输出`Hello World`

至此我们就快速的构建了一个空白的Spring Boot项目，再通过引入web模块实现了一个简单的请求处理；
