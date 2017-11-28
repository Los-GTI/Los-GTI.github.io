---
layout:     post
title:  Eclipse使用Maven创建SSM项目升级jre和web.xml版本
subtitle:   Eclipse，Maven，SSM
date:       2017-11-28
author:     QC
header-img: img/home-bg-geek.jpg
catalog: true
tags:
    - SSM
---
#### Eclipse创建Maven web项目解决jre及web.xml版本问题

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/星爷.jpg)

##### 1.前言

使用Eclipse通过Maven创建SSM（SpringMVC、Spring|、Mybatis）框架项目时，默认的web.xml的版本为2.3，jre的版本为1.5。因此在创建完成后我们需要对版本进行升级，升级过程中遇到一些问题，下面我写一个教程来升级web.xml及jre的版本。

##### 2.解决方法

###### 2.1方法一

将web.xml修改为以下的新版本：

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns="http://java.sun.com/xml/ns/javaee"
xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">
</web-app>
```

到工作区中项目的` .setting`文件夹中，找到`org.eclipse.wst.common.project.facet.core.xml`文件，修改其中的version到3.0，然后再选择 `Maven` -> `Update Project...`

```
<installed facet="jst.web" version="3.0"/>
```

然后修改jre库为当前工作区jre，这是第一种方法，下面我主要着重介绍第二种方法，这种方法不需要修改这个文件，更加简单快捷。

###### 2.2方法二

###### 2.2.1

首先新建一个Maven Project，这里不再赘述，主要的截图如下：

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/newMavenPro.png)

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/newMavenPro2.png)

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/newMavenPro3.png)

新建的Maven项目的结构如下

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/项目结构.png)

可以看到出现了一个HttpServlet错误，我们先解决一下这个错误。

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/错误1.png)

项目右击 `Properties` 选择`Java Build Path` 选择 `Add Library`添加 `Server Runtime` 选择 `Apache Tomcat7` 服务器（Tomcat8 需要 web版本3.1）

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/配置runserver.png)

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/错误解决.png)

HttpServlet错误解决，项目目录也有变化，自动补全` src/main/java`  和` src/test/java`  目录。

此时maven项目的web.xml版本为2.3,jre版本为1.5，需要升级。

###### 2.2.2

网上的方法多而且杂，自己实践才是关键，下面是我总结的升级web.xml和jre版本的方法，请大家参考！

修改jre版本，不能再` Java Build Path`  中修改，因为项目一旦` updata project`  ，jre版本又会回到1.5，所以我们在pom.xml文件中声明：

> pom.xml

```
 <build>
    <finalName>ssmTest</finalName>
     <plugins>
            <!-- 修改maven默认的JRE编译版本，1.8代表JRE编译的版本，根据自己的安装版本选择1.7或1.8 -->
            <plugin>
         <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.7</source>
                    <target>1.7</target>
                </configuration>
            </plugin>
        </plugins>
  </build>
```

项目右击选择Maven `Update Project`项目的`Java Build Path`为:

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/修改jre版本.png)

jre版本修改已经完成，下面修改web.xml版本：

这是默认生成的`web.xml`,可以看到是2.3版本的。

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/webXML23.png)

项目右击`Properties` 选择`Project Facets`（项目模板），如下图，可以看到`Dynamic Web Module`版本为 2.3

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/webXML232.png)

直接更改为3.0？，但是并不能更改

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/不能直接修改webXML.png)

这里可以先把`Dynamic Web Module`**勾选去掉**，之后Apply。

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/勾去掉.png)

**再接着勾选**`Dynamic Web Module`，注意下方出现`Further configuration available...`选项；接着更改为想要的版本 3.0，点击`Further configuration available...`

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/webXML修改1.png)

之后，修改 `Content director`为`src/main/webapp`,勾选`Generate web.xml deployment descriptor`OK，保存退出`Project Facets`

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/webXML修改2.png)

之后修改`src/main/webapp/WEB-INF/web.xml`文件,头信息版本修改为 3.0 版本的。

> web.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
</web-app>
```

之后，项目右击选择Maven `Update Project`。项目结构为：

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/最终项目结构.png)

到这就大功告成了，jre和web.xml版本的升级完成，接下来你就开开心心撸代码吧！！！

> 摁钉 author by chongqin
