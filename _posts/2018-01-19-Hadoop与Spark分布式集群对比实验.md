---
layout:     post
title:      Hadoop与Spark分布式集群对比实验
subtitle:   附实例
date:       2018-01-19
author:     QC
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Hadoop
---
### Hadoop分布式集群与Spark分布式集群比较实验
> 附实例

#### 1、实验环境

```
设备：虚拟机三台
系统：Ubuntu14.04
虚拟机环境：VMware Workstation
Hadoop版本：2.7.5，Spark版本：2.2.1
JDK版本：1.8.0，scala版本：2.12.4
IP：Master:192.168.10.123
       node1:192.168.10.124
       node2:192.168.10.125
```

#### 2.Hadoop分布式集群的搭建以及程序设计
##### 2.1 Hadoop分布式集群搭建步骤
首先搭建Hadoop分布式集群环境需要安装java运行环境，我这里使用的是jdk1.8,使用’java -version’命令检查机器上是否有java运行环境，没有的话三台机器都需要安装，安装的过程在这里就不再赘述。

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop1.png)

安装vim工具，命令如下：’sudo apt-get install vim’。
下一步我们要创建用户组，然后将三台主机加入同一用户组，我创建的用户组是Hadoop组。

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop2.png)

然后将当前用户qin加入hadoop组，然后执行同样的操作将其他两台机器加入hadoop组。

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop3.png)

下一步我们下载Hadoop安装包，进行Hadoop的安装与配置，因为要搭建Hadoop集群环境，所以三台电脑都要进行Hadoop的下载与配置，这里的下载以及Hadoop的配置文件的配置以master主机为例，另外两台从节点如没有特殊说明则下载配置过程与此相同：
下载Hadoop2.7.5压缩包：’hadoop-2.7.5.tar.gz’，将其放到当前用户目录
/home/qin下，执行解压缩命令将其解压到当前目录：

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop4.png)

Hadoop解压后的文件目录为/home/qin/hadoop-2.7.5。解压完成后进行配置文件的配置工作，需要配置的文件都在解压后的hadoop2.7.5目录即/home/qin/hadoop2.7.5下的/etc/hadoop目录下。
配置hadoop-env.sh:

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop5.png)

添加java安装路径，如下图：

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop6.png)

配置core-site.xml:

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop7.png)

<name>fs.default.name</name>：此参数设置NameNode的URI，此处设置master主机为NameNode。
<name>hadoop.tmp.dir</name>：此参数设置Hadoop的一个临时目录，用来存放每次运行的作业jpb的信息，这里指定的tmp目录需要自己创建。
配置hdfs-site.xml:

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop8.png)

<name>dfs.name.dir</name>：dfs.name.dir存储永久性的元数据的目录列表。这个目录会创建在master主机上。
<name>dfs.data.dir</name>：dfs.data.dir存放数据块的目录列表，这个目录在node1和node2上创建。
<name>dfs.replication</name>：dfs.replication设置文件副本数,此处有两个从机，设置副本数为2。
配置mapred-site.xml：
因为目录中没有mapred-site.xml文件，只有mapred-site.xml.template临时文件，我们需要复制一下mapred-site.xml.template临时文件，然后将其命名为mapred-site.xml

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop9.png)

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop10.png)

配置slaves：

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop11.png)

有几个从节点就添加几个，多添加会无法运行。
配置环境变量：

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop12.png)

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop13.png)

使用’source /etc/profile’命令使配置文件生效，完成以上步骤单机上的Hadoop就算安装好了，输入’hadoop version’命令检查Hadoop环境变量有没有配置成功，出现如下图所示的情况就是配置成功了。
![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop14.png)

到这里master主机的Hadoop安装以及配置就算完成了，下面使用VMvare Workstation的克隆功能克隆出来两台从机，这样我们就不需要反复配置了，我们只需要修改一下主机名就可以了，将主机名分别修改为node1和node2。
![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop15.png)

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop16.png)

node2主机也执行如上图所示的操作将主机名改为node2即可。然后修改/etc/hosts文件，添加以下内容，注意这里也是每台主机都要修改：

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop17.png)

然后要将三台机器的网络的连接方式设置为桥接模式然后重启虚拟机，之后分别测试三台机器能不能联通网络以及能不能相互通信，这里我们要成功搭建Hadoop集群环境需要我们的三台机器都有网络而且能相互ping通，所以如果出现网络问题抓紧解决解决完之后再执行下面的步骤，这里我就不再多说。
Hadoop配置文件的配置到这里我们就配置完了，下面要做一个重要的工作就是让master通过SSH免密登录其它两台从机，首先我们需要在每台机器上安装ssh，命令如下：

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop18.png)

然后通过以下命令来检查有没有安装成功ssh，如果缺少openssh-server则需要重新安装：’sudo apt-get install openssh-server’。

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop99.png)

下面的操作以master主机为例，请注意区分，通过以下命令生成master主机的一对公钥和私钥，成功后如下图所示：

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop100.png)

进入.ssh目录查看公钥和私钥，id_rsa和id_rsa.pub,将公钥加入到已认证的key中

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop101.png)

这样就可以登录本机，如果出现The authenticity of host 'localhost (127.0.0.1)' can't be established.，输入yes即可。

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop102.png)

以上是在主机上的操作，然后在从机上进行以下操作，两个从机上的操作相同，这里以在node1上的操作为例，将master主机上的id_rsa.pub复制到node1从机上，命名为node1_rs.pub，然后将密钥加入到认证中即可：

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop19.png)

最后检查能否在master主机上通过ssh node1或者ssh node2命令免密登录两个从机，登录成功之后如下图所示（第一次登录的时候可能需要输入一次密码，后来再登录就不需要输入）：

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop20.png)

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop21.png)

到这一步Hadoop集群所有的搭建工作就基本完成了，下面开启集群，首先格式化nameNode,
以下操作均在主机上运行，出现如下图所示情况表示格式化成功：
![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop22.png)

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop23.png)

‘start-all’命令开启集群，开启之后输入jps命令检查集群有没有开启成功，也可以在web页面查看集群有没有开启成功，开启成功的话如下图所示，主机上会开启NameNode、ResourceManager等进程，从机上会开启DataNode，DataNodeManager等进程，开启的过程以及结果如下图所示：

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop24.png)

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop25.png)

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop26.png)

##### 2.2 Hadoop集群上程序设计
数据集介绍：数据集是实验室爬虫项目中爬下来的部分网页的源码，先去除除中文以外的其它字符，然后去掉停用词，比如介词等，最后结巴分词形成此次程序设计的数据集，数据集大小为209M，文件格式为txt。
程序功能介绍：是python语言写的一个统计词频的程序，运行方法直接在master主机上通过运行脚本来运行，运行步骤及结果如下：
首先在Hadoop的hdfs文件系统上创建存放数据集的文件夹

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop104.png)

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop103.png)

然后将数据集上传到文件夹中

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop105.png)

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop106.png)

之后将程序的主要代码上传到master主机上，我这边主要是两个py文件reducer.py和mapper.py，源代码如下：
reducer.py
```
#!/usr/bin/env python
from operator import itemgetter
import sys
current_word = None
current_count = 0
word = None
# input comes from STDIN
for line in sys.stdin:
    # remove leading and trailing whitespace
line = line.strip()

    # parse the input we got from mapper.py
    word, count = line.split('\t', 1)

    # convert count (currently a string) to int
    try:
        count = int(count)
    except ValueError:
        # count was not a number, so silently
        # ignore/discard this line
        continue
    # this IF-switch only works because Hadoop sorts map output
    # by key (here: word) before it is passed to the reducer
    if current_word == word:
        current_count += count
    else:
        if current_word:
            # write result to STDOUT
            print '%s\t%s' % (current_word, current_count)
        current_count = count
        current_word = word
# do not forget to output the last word if needed!
```
mapper.py
```
#!/usr/bin/env python

import sys
# input comes from STDIN (standard input)
for line in sys.stdin:
    # remove leading and trailing whitespace
    line = line.strip()
    # split the line into words
    words = line.split()
    # increase counters
    for word in words:
        # write the results to STDOUT (standard output);
        # what we output here will be the input for the
        # Reduce step, i.e. the input for reducer.py
        #
        # tab-delimited; the trivial word count is 1
        print '%s\t%s' % (word, 1)           
```
然后通过配置一个run.sh脚本来实现运行此程序，脚本如下：

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop27.png)

配置完之后用’source run.sh’命令更新脚本，然后根据脚本里面配置的执行语句执行程序，执行成功以后如下图所示：

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop28.png)

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop29.png)

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop30.png)

整个程序跑完用时3分57秒，统计词频的情况如下图所示，因为输出的文件较大这里只显示末尾10条数据。

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop31.png)

#### 3.Spark分布式集群以及程序设计
##### 3.1 Spark分布式集群搭建过程
Spark分布式集群搭建需要在Hadoop分布式集群的基础上进行，你在搭建Spark分布式集群的时候一定要确定你的Hadoop分布式集群搭建成功而且各个进程可以成功运行，下面开始Spark分布式集群的搭建过程。
搭建Spark之前需要配置scala环境，下载scala安装包’scala-2.12.4.tgz’,我把它放在了当前用户的家目录下，然后用’tar -zxvf scala-2.12.4.tgz’命令将其解压到当前目录下，然后配置scala环境，输入’sudo vim /etc/profile’命令添加如下内容：

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop32.png)

添加完之后保存退出，然后执行’source /etc/profile’命令使当前配置立即生效，之后检查scala环境有没有配置成功，配置成功之后如下图所示：

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop33.png)

Scala环境配置成功以后下载Spark压缩包spark-2.2.1-bin-hadoop2.7.tgz，我放在了当前用户的家目录下，然后使用’tar -zxvf spark-2.2.1-bin-hadoop2.7.tgz’命令将其解压到当前目录，然后修改配置文件’sudo vim /etc/profile’配置Spark环境变量：

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop34.png)

同样配置完之后执行’source /etc/profile’命令使之立即生效。下面进入解压后的Spark的配置文件目录，在spark目录下的conf目录中，修改一些配置文件（因为有些配置文件只有临时文件，所以我们需要先复制然后再修改）：
复制spark-env.sh.template成spark-env.sh

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop35.png)

修改spark-env.sh

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop36.png)

复制slaves.template成slaves

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop37.png)

修改slaves，有几个从节点就写几个
![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop38.png)

到这一步Spark的配置就基本完成了，下面将我们在master主机上配置过的同样在两台从机上配置一遍，当然你也可以直接复制过去。
最后开启Spark分布式集群，用如下命令开启：
![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop39.png)
开启成功之后jps查看进程，你会发现master主机上多了Master、Worker两个进程，而从机上多了Worker进程，你也可以去web网页检查Spark有没有开启成功。

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop40.png)

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop41.png)

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop42.png)

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop43.png)

##### 3.2 Spark集群上程序设计
数据集介绍：数据集和在Hadoop上运行的数据集相同，数据集是实验室爬虫项目中爬下来的部分网页的源码，先去除除中文以外的其它字符，然后去掉停用词，比如介词等，最后结巴分词形成此次程序设计的数据集，数据集大小为209M，文件格式为txt。
程序功能：为了要和在Hadoop上的运行结果做一个比较，我们这边也是写了一个python程序来统计数据集中的词频，运行方式是在master主机上直接运行wordcount.py程序，实现的代码如下：
wordcount.py
```
# -*-coding:utf-8-*-
import logging
from operator import add
from pyspark import SparkContext
logging.basicConfig(format='%(message)s', level=logging.INFO)

#import local file  
test_file_name = "hdfs:///user6/input/wangye.txt"
out_file_name = "hdfs:///user6/sparkWangyeOut"

sc = SparkContext("local","wordcount app")

# text_file rdd object  
text_file = sc.textFile(test_file_name)

# counts  
counts = text_file.flatMap(lambda line: line.split(" ")).map(lambda word: (word, 1)).reduceByKey(lambda a, b: a + b)
counts.saveAsTextFile(out_file_name)
```
执行spark-submit wordcount.py运行程序

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop44.png)

运行结果如下图所示：

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop45.png)

同样也是选择输出文件中末尾十行来展示，因为程序输出的是Unicode编码，所以我把它转换为utf-8编码之后如下图所示：

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop46.png)

![](https://raw.githubusercontent.com/Los-GTI/Los-GTI.github.io/master/img/Hadoop47.png)

全部程序跑完用时51.213518s,统计词频的情况如上图所示，因为输出文件记录数较多，这里只展示最后10条记录。
#### 4.实验结果对比分析以及优化建议
上文已经给出了Hadoop和Spark分布式集群上的程序设计过程以及实验结果，很明显完成同样的程序设计功能，跑同样大小的数据集，在Hadoop分布式集群上需要3分57秒，而在Spark分布式集群上仅仅需要51s，这就体现出了Spark分布式集群在性能上的巨大优势，处理时间仅仅是Hadoop平台上的1/4，如果数据再多一点我相信优势会更加明显，而且经过计算运行Spark程序占用的计算机资源也仅仅只有Hadoop的1/4，所以说Spark在我今天的实验中毫无疑问是最合适的，而且占了巨大的优势。之所以Hadoop的表现没有Spark好，原因可能是MapReduce的工作机制问题，在Hadoop分布式集群上一个Job只分为Map和Reduce阶段，代码的时候也要分开来写，最关键的是在完成一个Job时Reduce Task只有等待Map Task完成之后才能开始工作，所以在处理速度上和性能上表现的较差，而Spark可以在内存中缓存数据，一个Job可以包含RDD的多个转换操作，在调度时可以生成多个Stage，并且如果多个map操作的RDD分区不变还可以放在同一个Task中进行，Spark的这种工作机制就解释了在实验中为什么Spark的处理性能更佳。当然本次实验还有很多可以优化的地方，首先局限于硬件设备资源不够，本次实验的实验环境是在三台虚拟机上搭建的，如果在三台真机上搭建我相信结果会更好。还有就是在代码编写的层次上还有可优化的地方，经过代码的优化我相信在处理速度和性能上也会有较大的提升。
> 这篇博客的内容是我一门叫做并行分布式计算的课的作业，我完成的过程中也踩了不少坑，这里分享给大家！


> 摁钉 author by chongqin
