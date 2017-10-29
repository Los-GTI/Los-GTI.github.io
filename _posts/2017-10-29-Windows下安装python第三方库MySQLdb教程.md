### Windows下安装python第三方库MySQLdb教程

> 做的项目中需要用到python的MySQLdb库，使用pip安装和下载下来之后使用`python setup.py install`命令安装此库都出现错误，遂才有了这篇记录安装教程的博客。

![](https://i.imgur.com/4MV2CQk.png)

![](https://i.imgur.com/avB58yX.png)

> 无法正常安装，然后我去网上搜了搜相关问题的解决方法逐一排查，看看最后能不能成功安装。

#### 解决过程

##### 1.首先检查你有没有安装mysql数据库，使用python访问mysql需要安装mysql数据库。

我这边检查了一下，我本机没有安装mysql数据库，现在我去安装一下mysql数据库，看看安装完mysql数据库之后能不能成功安装mysqldb库。下面安装mysql数据库。

MySQL针对不同的用户提供了2中不同的版本：
MySQL Community Server：社区版。由MySQL开源社区开发者和爱好者提供技术支持，对开发者开放源代码并提供免费下载。
MySQL Enterprise Server：企业版。包括最全面的高级功能和管理工具，不过对用户收费。
下面讲到的MySQL安装都是以免费开源的社区版为基础。打开MySQL数据库官网的下载地址h`ttp://dev.mysql.com/downloads/mysql`，上面提供了两种安装文件，一种是直接安装的MSI安装文件，另一种是需要解压并配置的压缩包文件。我这里使用压缩包安装。根据自己的操作系统下载对应的32位或64位的压缩包。按如下步骤操作： 
第一步:解压缩
将压缩包解压到C:\Program Files\MySQL路径下，也可以自定义路径。我的路径为E:\Softwares\mysql5.7\mysql-5.7.20-winx64，如下图：
![](https://i.imgur.com/1YpJrGL.png) 

第二步：配置环境变量
右键点击“计算机”，选择“属性”，依次打开“高级系统设置”->“环境变量”，在系统变量中选择“Path”项，并点击编辑，如下图： 
![](https://i.imgur.com/eqj81h0.png)
保持原有值不变，追加E:\Softwares\mysql5.7\mysql-5.7.20-winx64\bin，将MySQL安装路径下的bin目录配置到Path变量中，使在命令行的任何目录下可以执行MySQL命令。

第三步： 修改配置 
在网上教程中都是要打开MySQL安装目录下面的my-default.ini文件，但是我的安装包解压后没有这个文件，我就手动添加一个my.ini文件，加上如下配置：
[mysqld] 
basedir = E:\Softwares\mysql5.7\mysql-5.7.20-winx64
datadir = E:\Softwares\mysql5.7\mysql-5.7.20-winx64\data 
分别表示MySQL的安装目录和数据目录。如果在第一步中解压缩到其它的文件夹则修改对应的值。 

第四步： 安装 
以管理员身份运行cmd，进入到MySQL的bin目录，执行初始化命令：
`mysqld --initialize --user=mysql --console`
![](https://i.imgur.com/9W3dJn2.png)
该命令用来初始化数据，在5.7以前的版本是不需要执行该命令的。初始化完成后会提供一个临时的root密码，记下该密码。 
再执行如下命令进行MySQL服务安装：
`mysqld –install mysql`
mysql为默认的服务名，可不写，若安装成功则有如下提示： 
![](https://i.imgur.com/Qp913DX.png)
> 需要注意的是一定要以管理员身份运行cmd，否则会出现错误

第五步： 启动服务 
在管理员cmd窗口中执行如下命令来启动MySQL服务：
` net start mysql`
这一步不是必要操作，如果计算机提示你“net不是内部或外部命令，也不是可执行的程序”，可以直接跳过这一步到第六步。

第六步： 登录 
执行如下命令：`mysql -u root -p`
提示输入密码，输入第四步中记录下的密码，按回车后出现如下页面表示登录成功，并进入了MySQL命令行模式。 
![](https://i.imgur.com/3hwjKRc.png)
如果出现如下ERROR2003错误：
![](https://i.imgur.com/pH8FWJd.png)
进行如下设置：右击计算机——》管理——》服务与应用程序——》将mysql服务手动启动即可

第七步： 修改密码 
在MySQL命令行执行如下命令：
`ALTER USER ‘root’@’localhost’  IDENTIFIED BY ‘new_password’ `
大家改成自己的密码，如下图所示表示修改成功： 
![](https://i.imgur.com/TdB30CZ.png)
> 执行完以上步骤mysql服务就安装成功，下面我们可以到图形化界面去连接一下mysql

靠，措手不及啊，又出现了错误，啊啊啊！！！出现了1045错误！！
![](https://i.imgur.com/yYDn6wr.png)
然后我发现用命令行也是这个错误，完了又得找办法解决，经过半个小时的探索和实验终于找到了适合我的方法。
1.在my.ini文件中加入skip-grant-tables这句话，免输入密码进入mysql，然后关闭mysql服务再重启，这时候就可以不用输入密码进入mysql服务了；
![](https://i.imgur.com/Z8iuOgD.png)
2.下面输入下面几行命令修改密码：
```use mysql;
   update user set authentication_string=password("123456") where user="root";//其中大部分人的authentication_string这个字段应该是paaword，但是我后来通过图形界面登录的时候发现我的user表里面没有password这个字段，然后网上有一个哥们说这个字段可以取代paaword字段，我就拿来试试看看行不行
   flush privileges;
   quit
```
![](https://i.imgur.com/rzJa4KG.png)
3.将my.ini里面添加的那句话去掉，然后重启mysql服务。然后再登录看看成不成功。
![](https://i.imgur.com/fwKfue4.png)
![](https://i.imgur.com/86whNV7.png)
> 靠，泪牛满面，终于成功了，不过现在干的都不是正事，妈个鸡我要装的是python的mysqldb库，不是来装mysql的，下面我试试安装完mysql之后能不能直接安装mysqldb库。

##### 2.安装了mysql数据库之后仍然无法安装，遇到了安装错误：fatal error C1083：Cannot open include file: 'config-win.h': No s uch file or directory。下面就是要解决这个错误

我找到了如下解决办法：
1.安装mysql-python之前, 请先安装setuptools. 
https://pypi.python.org/pypi/setuptools/7.0
![](https://i.imgur.com/5rQTAHV.png)

2.我安装完之后再去试试能不能安装mysqldb，然而还是不行还是原来的错误，接着找错误。

3.需要安装mysql connector 前往 
http://dev.mysql.com/downloads/connectorc/6.0.html#downloads  
根据python的版本下载32位或64位版本. 默认安装即可. 

4.windows可能你还需要安装一个VCforpython2.7. 去微软官网下载即可。

5.再次执行 python setup.py install 安装成功. 

![](https://i.imgur.com/1rG4szj.png)

> 下面开开心心的开启新一轮爬虫之旅吧！！！

> 摁钉 author by qinchong
