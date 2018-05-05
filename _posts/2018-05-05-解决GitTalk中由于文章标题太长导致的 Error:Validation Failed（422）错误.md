---
layout:     post
title: 解决GitTalk中由于文章标题太长导致的 Error:Validation Failed（422）错误
subtitle: bug修复
date:       2018-05-05
author:     Los-GTI
header-img: img/ms1.jpg
catalog: true
tags:
    - bug
---

#### 前言

之前gittalk评论都没有什么问题，突然有一天我发布了一篇新博客的时候出现了`Error:Validation Failed`错误，这样就很蛋疼啊，虽然我的博客也没几个人看，主要还是记录自己的学习过程为主，但是作为一个强迫症患者且作为一个
合格的程序员，有了bug还是要解决的，所以就有了这篇文章！

#### 解决办法

出现问题之后我就去github上搜索有没有人出现了类似的错误，发现果然有人已经出现了这个错误，看了一圈我大体知道了发生错误的原因：

**Gittalk默认的配置`id:location.href`,id会被保存为issue的一个label，用来关联文章和issue。之前GitHub没有对label的长度做限制，所以创建的一些issue的label长度超过50是可以的；但是github最近对label加了限制，限制label的长度的上限是50个字符，所以文章名比较长的都会生成label失败，也就没办法评论了！**

当然既然有人发现了这个问题，就会有大神提供了一些解决方案，下面我讲一下我知道的能解决问题的方案以及解决了我的问题的方案：

- 第一个方案：当然就是刚刚我说的把所有的标题都改短，这也不算解决办法吧，充其量算个规避问题的方案；

- 第二个方案：文章名经过URL编码后转MD5，然后再生成label，这也就不会超过长度了，具体解决步骤我甩个链接[转MD5解决办法](https://priesttomb.github.io/%E6%97%A5%E5%B8%B8/2018/02/12/%E5%A4%84%E7%90%86Gitalk%E4%B8%AD%E7%94%B1%E4%BA%8E%E6%96%87%E7%AB%A0URL%E8%BF%87%E9%95%BF%E5%AF%BC%E8%87%B4%E7%9A%84Validation-Failed(422)/)，但是我安照这种办法并没有解决我的问题，不知道是什么情况，但是确实有人用这种方法解决了问题，如果这种办法不行的话你可以接着试一下第三种方案！接着说一句用这种方法可能出现的问题：`进行 md5 加密确保唯一性和长度不超过50，但是如果修改了 id 的配置，那之前创建的 issue 就无法与之前的文章关联了，因为之前文章对应的 label 是没有经过 md5 加密的值，也就无法找到对应的 issue 了，所以，如果修改了 id 则需要把之前创建的 issue 的 label 都更新才行。`
- 第三个方案：将id设置为文章的时间，这样的话长度保证在50个字符之内，完美解决了这个问题,下面我上一下我修改的地方，但是关于这个id在哪里修改，大家要根据自己使用的主题，找到id配置的位置进行修改；
```
					<div id="gitalk-container"></div>
                    <script type="text/javascript">
                    var gitalk = new Gitalk({
                    clientID: 'xxx',
                    clientSecret: 'xxx',
                    repo: 'xxx',
                    owner: 'xxx',
                    admin: xxx,
                    id: '{{ page.date }}',
                    });
                    gitalk.render('gitalk-container');
                    </script>
```

至此问题完美解决，希望能帮助你哦！当然可能还有一些更好的解决办法，这里我所叙述的仅代表我的个人观点和踩坑记录！
