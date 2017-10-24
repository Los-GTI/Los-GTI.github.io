---
layout:     post
title:      Ubuntu系统配置Java环境变量
subtitle:   Ubuntu系统版本:14.0.1
date:       2017-10-24
author:     QC
header-img: img/timg11.jpg
catalog: true
tags:
    - JAVA_HOME
---
#### Ubuntu系统下配置jdk和Java环境变量![](/home/chongqin/mdImg/java环境测试2.png) 
> 系统：Ubuntu14.0.4
##### 1.安装jdk配置环境变量
先从Oracle官网下载JDK。先选择同意按钮，然后根据自己的系统下载相应版本。我的系统是Ubuntu14.04 64位的，所以我下载
![](/home/chongqin/mdImg/jdk下载.png) 
> 下载的最新版本

把下载好的文件放在你想放的目录下，我的放在了我平常放下载文件的地方。
然后根据你文件的路径解压文件
![](/home/chongqin/mdImg/解压文件命令.png) 
![](/home/chongqin/mdImg/解压成功.png) 
> 解压成功

然后我把我解压后的文件夹放到了我新创建的文件夹jvm中。
进入jvm文件夹。
![](/home/chongqin/mdImg/进入jvm文件夹.png) 
![](/home/chongqin/mdImg/jdk.png) 

先进入vi编辑器（第一幅图），然后输入以下内容（第二幅图）。

![](/home/chongqin/mdImg/vi.png) 

![](/home/chongqin/mdImg/编辑环境变量.png) 
> :wq 保存退出

##### 2.测试
再执行以下命令然后测试

![](/home/chongqin/mdImg/sourcebashrc.png) 

测试
出现下图所示结果说明配置安装成功，你就可以尽情书写Java程序了。
![](/home/chongqin/mdImg/测试1.png) 
![](/home/chongqin/mdImg/java环境测试2.png) 

> 摁钉： author chongqin
