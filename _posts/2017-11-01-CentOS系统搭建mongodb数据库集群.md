---
layout:     post
title:      CentOS下搭建mongodb集群
subtitle:   CentOS7系统虚拟机两台
date:       2017-11-01
author:     QC
header-img: img/timg6.jpg
catalog: true
tags:
    - mongodb
---

### CentOS系统下搭建mongodb数据库集群
> 前一段时间搭建了windows下的mongodb数据库集群小试牛刀，最终项目的服务器的系统还是CentOS，所以还是要搭建CentOS系统下的mongodb数据库集群。
> 设备：两台不同机器上的CentOS7系统虚拟机两台
> mongodb版本：3.4

#### 1.注意事项

##### 1.1 mongodb3.4要求必须有多个配置服务器，一定要注意，如果没有多个配置服务器则会报以下错误：
```
BadValue: configdb supports only replica set connection string
try 'mongos --help' for more information
```
##### 1.2 内部测试时可以关闭防火墙，部署环境时可以关闭相关端口，这两种方法二选一

###### 1.2.1 关闭firewall
```
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
firewall-cmd --state #查看默认防火墙状态（关闭后显示notrunning，开启后显示running）
```
###### 1.2.2 关闭相关端口

```
firewall-cmd --zone=public --add-port=27016/tcp --permanent
firewall-cmd --zone=public --add-port=27017/tcp –permanent
firewall-cmd –reload
```
> 我采用的方法是关闭防火墙。

![](https://i.imgur.com/zkjZD6Y.png)

#### 2.下载安装mongodb数据库
> 参加官方文档 `https://docs.mongodb.com/manual/tutorial/install-mongodb-enterprise-on-red-hat/`

##### 2.1 创建yum库
```
cd /etc/yum.repos.d/
vim mongodb-enterprise.repo
```
![](https://i.imgur.com/Dm0AkJL.png)

##### 2.2 将下面文字复制到repo文件中
```
[mongodb-enterprise]
name=MongoDB Enterprise Repository
baseurl=https://repo.mongodb.com/yum/redhat/$releasever/mongodb-enterprise/3.4/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc
```
![](https://i.imgur.com/uwZB7jy.png)

> 创建库成功

![](https://i.imgur.com/WcsuIKq.png)


##### 2.3 mongodb数据库包
命令：`sudo yum install -y mongodb-enterprise`

![](https://i.imgur.com/Jk82wJJ.png)

> 安装成功

![](https://i.imgur.com/rxeO88o.png)

###### 2.4 在另外一台虚拟机安装mongodb

过程一样，直接上图

![](https://i.imgur.com/195eqlB.png)

#### 3.搭建mongodb数据库集群

##### 3.1 Xshell连接虚拟机终端

为了方便操作，这边我在我的主机上连接我电脑上的一台虚拟机的终端和另外一台电脑上的虚拟机的终端，这样所有的操作我都可以在我自己电脑上完成。

![](https://i.imgur.com/hm1jV2s.png)

![](https://i.imgur.com/BwgFckC.png)

##### 3.2 测试环境

操作系统：VMVare+CentOS7+静态IP
mongodb版本：mongodb-3.4

##### 3.3 布局预览

![](https://i.imgur.com/o1PKiAP.png)
![](https://i.imgur.com/skoH2WN.png)

##### 3.4 资源分配

> 因为机器有限，我今天搭建过程用了两台虚拟机，每台虚拟机上开了三个分片。

IP分配：
机器1：172.16.134.58
机器2:172.16.134.50

>当然你也可以有机器3，机器4，机器5等等

端口分配：

mongos:172.16.134.58:27017
config:172.16.134.58:27001
config:172.16.134.50:27001
shard1:172.16.134.58:27002
shard2:172.16.134.58:27003
shard3:172.16.134.58:27004
shard4:172.16.134.50:27002
shard5:172.16.134.50:27003
shard6:172.16.134.50:27004

##### 3.5 创建相关目录
两台主机分别创建如下目录：
/home/XXXX/data/mongodb/config/data
/home/XXXX/data/mongodb/config/log/
/home/XXXX/data/mongodb/mongos/log

> XXXX为你主机的名字，每个主机不同，我两台机器创建的如下图：

![](https://i.imgur.com/o02ru1H.png)

![](https://i.imgur.com/eyvZofA.png)

两台主机分别创建分片目录：

/home/ubuntu/data/mongodb/shard0X/data
/home/ubuntu/data/mongodb/shard0X/log

> 我这里每台主机创建3个分片，目录如下图：

![](https://i.imgur.com/YTYea6e.png)

![](https://i.imgur.com/eFCBoON.png)

##### 3.6 启动配置服务器

用如下命令：
```
mongod --configsvr --replSet configReplSet --port 27001 --dbpath  /home/xxxx/data/mongodb/config/data/ --logpath /home/xxxx/data/mongodb/config/log/mongo.log --fork
```
![](https://i.imgur.com/ERtHZ0X.png)

![](https://i.imgur.com/ju3VSvQ.png)

##### 3.7 登录初始化配置服务器

用如下命令：
`mongo 172.16.134.58:27001`
```
 rs.initiate( {
		_id: "configReplSet",
		configsvr: true,
		members: [
		{ _id: 0, host: "172.16.134.58:27001" },
		{ _id: 1, host: "172.16.134.50:27001" },
		]
		} )
```
![](https://i.imgur.com/YKTVwzD.png)

![](https://i.imgur.com/uRU8Lyv.png)

##### 3.7 启动分片服务器

用如下命令：
```
mongod --shardsvr --dbpath /home/XXXX/data/mongodb/shard0X/data/ --port 27002 --logpath /home/XXXX/data/mongodb/shard0X/log/mongo.log --fork
```
> 我这边一共6个分片就要开启6个分片服务器，你有几个片就开几个。

![](https://i.imgur.com/QJY5hjA.png)

![](https://i.imgur.com/6nBB7jc.png)
![](https://i.imgur.com/UAw0iQE.png)
![](https://i.imgur.com/voduGNG.png)

##### 3.8 启动mongos

用如下命令：
```
mongos --configdb configReplSet/172.16.134.58:27001,172.16.134.50:27001
```
![](https://i.imgur.com/Zrk3e0p.png)

##### 3.9 连接mongos

用如下命令：
```
mongo 172.16.134.58：27017
```
![](https://i.imgur.com/4srMjCW.png)

##### 3.10 设置分片

用如下命令：
```
        use admin
        db.runCommand({ addshard:"172.16.134.58:27002" })
        db.runCommand({ addshard:"172.16.134.58:27003" })
        db.runCommand({ addshard:"172.16.134.58:27004" })
	    db.runCommand({ addshard:"172.16.134.50:27002" })
	    db.runCommand({ addshard:"172.16.134.50:27003" })
        db.runCommand({ addshard:"172.16.134.50:27004" })
        sh.enableSharding("testdb1")	
	    sh.shardCollection("testdb1.tab", { "_id": "hashed" })
```

> 数据库为testdb1，collection为tab

![](https://i.imgur.com/4EwKjyc.png)

![](https://i.imgur.com/OsJY5sX.png)

##### 3.11 插入数据测试

我这边往tab里插入1000条数据看分片情况，看看有没有分片成功

![](https://i.imgur.com/eiK6D90.png)

查询分片情况：
总数：1000条数据

![](https://i.imgur.com/f4FFW9Y.png)

下面是各个分片的情况：
shrad00:172条
![](https://i.imgur.com/qaOh0Kg.png)

shard01:165条
![](https://i.imgur.com/U5prmDi.png)

shard02:166条
![](https://i.imgur.com/7ScDBdU.png)

shard03:174条
![](https://i.imgur.com/oXNRnyi.png)

sharf04:163条
![](https://i.imgur.com/Z0snfEf.png)

shard05:160条
![](https://i.imgur.com/UHzuKmb.png)

由上面的分片结果可以看出我们分片成功！喜大普奔！！！

> 摁钉 author by chongqin



