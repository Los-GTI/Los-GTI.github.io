---
layout:     post
title: python脚本运行
subtitle: IndentationError错误解决办法
date:       2017-10-29
author:     QC
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - python
---
### Python脚本运行出现语法错误：IndentationError: unindent does not match any outer indentation level

> 解决问题流程

![](https://i.imgur.com/aPd36Pe.png)

#### 1. 问题

一个好好地python脚本，我看明明都对齐了，语法格式什么的都没什么问题，但是一运行就提示我有错误，这我就很纳闷了，我就百度搜了一下相关的问题，后来发现可能出现的原因如下：

#### 2.可能原因

对于此错误，最常见的原因是：的确可能是没对齐，但是我根据错误提示的行去看了一下，没什么问题啊，都对齐了呀，根本看不出来有什么问题，可他就是报错，好吧，我也很无奈。只能另寻他法。
以为是前面的注释的内容影响后面的语句的语法了，所以把前面的注释也删除了。 结果还是此语法错误。 
后来我就开始用最原始的方法，将代码从头开始只要需要空格的地方全都使用Tab键，从头到尾下来再使用就没什么问题了，后来发现是空格和Tab键混用了，可能我明眼看没什么问题，可是的确混用了。

#### 3.总结

遇到IdentationError,应该立即想到应该是Tab键和空格键混用的问题，大部分人应该都是这个问题，遇到问题后逐一排查，最后解决问题。

> 摁钉 author by qinchong
