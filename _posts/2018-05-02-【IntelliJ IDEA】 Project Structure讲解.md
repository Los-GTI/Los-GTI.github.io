---
layout:     post
title: 【IntelliJ IDEA】 Project Structure讲解
subtitle: Project Structure讲解
date:       2018-05-02
author:     Los-GTI
header-img: img/ms1.jpg
catalog: true
tags:
    - IDEA
---


#### 项目配置的理解

IDEA中最重要的各种设置项，就是这个Project Structre了，关乎你的项目运行，缺胳膊少腿都不行。最近正好从Eclipse转战IDEA，为了更深入理解和使用，就找各种资料研究一下，最后整理出来这篇博客！

**项目的左侧**

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/IDEA/idea1.png)

**Project**

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/IDEA/idea2.png)

- Project name：定义项目的名称；
- Project SDK：设置该项目使用的JDK，也可以在此处新添加其他版本的JDK；
- Project language level：这个和JDK的类似，区别在于，假如你设置了JDK1.8，却只用到1.6的特性，那么这里可以设置语言等级为1.6，这个是限定项目编译检查时最低要求的JDK特性；
- Project compiler output：项目中的默认编译输出总目录，实际上每个模块可以自己设置特殊的输出目录（Modules - (project) - Paths - Use module compile output path），所以这个设置有点鸡肋。
- 另外一点需要注意的是输出目录下idea一般会创建三个子目录：
	- artifacts：存放war包解压以后的标准Web结构的代码，里面子文件的名字一般为**（项目名）_war_exploded**；
	- production：存放Java源代码src目录下编译以后的字节码文件和Web项目的配置文件；
	- test：存放Java源代码test目录下编译以后的字节码文件，即测试代码的字节码文件；

**Modules**

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/IDEA/idea3.png)

**Modules --> Sources面板**

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/IDEA/idea4.png)

**Modules -->Paths面板 **

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/IDEA/idea5.png)

**Modules --> Dependencies**

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/IDEA/idea6.png)

依赖的Scope属性有必要说一下，类似Maven的编译范围，有四种状态，默认为Compile；
- Compile：对项目类和测试类来说，编译和运行都有效；
- Test：仅对测试类来说，编译和运行都有效；
- Runtime：对项目类和测试类来说，仅运行时有效；
- Provided：对项目类来说，仅编译时有效，对测试类来说构建和运行时都有效；

**每个项目之下都可以定义它所使用的框架，这里重点说明一下Web部分的设置。**

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/IDEA/idea7.png)

**Libraries模块：**

- 这里可以显示所添加的jar包，同时也可以添加jar包，并且可以把多个jar放在一个组里面，类似于jar包整理。
- 这个模块其实就是将我们依赖的jar包进行分类，一般选择Java，而不选择From Maven，这里添加jar包和上面modules面板添加jar包是一样的，不过这里可以起别名，便于自己对jar包的管理；

**Facets模块：**

官方的解释是：
When you select a framework (a facet) in the element selector pane, the settings for the framework are shown in the right-hand part of the dialog.
（当你在左边选择面板点击某个技术框架，右边将会显示这个框架的一些设置）

说实话，并没有感觉到有什么作用。

**Artifacts模块：**

项目的打包部署设置，这个是项目配置里面比较关键的地方，重点说一下。

即编译后的Java类，Web资源等的整合，用以测试、部署等工作。再白话一点，就是说某个module要如何打包，例如war exploded、war、jar、ear等等这种打包形式。某个module有了 Artifacts 就可以部署到应用服务器中了。

- jar：Java ARchive,通常用于聚合大量的Java类文件、相关的元数据和资源（文本、图片等）文件到一个文件，以便分发Java平台应用软件或库；
- war：Web Application ARchive,一种JAR文件，其中包含用来分发的JSP、Java Servlet、Java类、XML文件、标签库、静态网页（HTML和相关文件），以及构成Web应用程序的其他资源；
- exploded：在这里你可以理解为展开，不压缩的意思。也就是war、jar等产出物没压缩前的目录结构。建议在开发的时候使用这种模式，便于修改了文件的效果立刻显现出来。

默认情况下，IDEA的 Modules 和 Artifacts 的 output目录已经设置好了，不需要更改，打成war包的时候会自动在 WEB-INF目录下生成classes，然后把编译后的文件放进去。

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/IDEA/idea8.png)

你可能对这里的输出目录不太理解，之前不是配置过了文件编译的输出目录了吗？为什么这里还有一个整合这些资源的目录呢？它又做了哪些事呢？ 

其实，实际上，当你点击运行tomcat时，默认就开始做以下事情：

- 编译，IDEA在保存/自动保存后不会做编译，不像Eclipse的保存即编译，因此在运行server前会做一次编译。编译后class文件存放在指定的项目编译输出目录下（即模块的输出目录下，在modules-->paths下设置）；
- 根据artifact中的设定对目录结构进行创建；
- 拷贝web资源的根目录下的所有文件到artifacts的目录下；
- 拷贝编译输出目录下的classes目录到artifacts下的WEB-INF下；
- 拷贝lib目录下所需的jar包到artifacts下的WEB_INF下；
- 运行server，运行成功后，如有需要，会自动打开浏览器访问指定url。

在这里还要注意的是，配置完成的artifact，需要在tomcat中进行添加。

