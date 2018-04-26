---
layout:     post
title: 混口饭吃专题之Spring详解之四Spring命名空间介绍
subtitle: Spring系列之四
date:       2018-04-25
author:     Los-GTI
header-img: img/ms1.jpg
catalog: true
tags:
    - 混口饭吃
---

### Spring详解（四） ------ Spring命名空间介绍

什么是Spring命名空间？这个就要从XML说了，Spring的配置管理可以利用XML方式进行配置，而XML里面就有命名空间这个概念。实际上就和标签的意思有点像，你给一个命名空间以后，这个XML文件里面就可以用那个命名空间上下文里面的标签了。

Spring 配置文件是用于指导Spring工厂进行Bean生产、依赖关系注入（装配）及Bean实例分发的"图纸"。Java EE程序员必须学会并灵活应用这份"图纸"准确地表达自己的"生产意图"。Spring配置文件是一个或多个标准的XML文 档，applicationContext.xml是Spring的默认配置文件，当容器启动时找不到指定的配置文档时，将会尝试加载这个默认的配置文件。

**Spring头文件的格式如下：**

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xmlns:context="http://www.springframework.org/schema/context"
 xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd">
</beans>
```

- xmlns:content 是为引用Spring的模块功能指定命名空间。其中的content是"http://www.springframework.org/schema/content"这个命名空间的简称，"http://www.springframework.org/schema/content"是这个命名空间的全称。必须在xsi中为命名空间指定对应的schema文件。

- xsi:scemaLocation是为每个命名空间指定了对应的Schema文档，其定义的语法为：`xsi:schemaLocation = "全称命名空间1 全称命名空间1对应的Schema文件空格"`

- 首先 `xmlns="http://www.springframework.org/schema/beans" xmlns:xsi=http://www.w3.org/2001/XMLSchema-instance `是必须有的。
- xsi:schemaLocation：为指定了用于解析和校验xml的定义文件（xsd）的位置。

**Spring中所有的命名空间：**

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/spring/Spring4_1.png)

**Spring常用命名空间以及schema地址：**

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/spring/Spring2_2.png)
