---
layout:     post                    # 使用的布局（不需要改）
title:      回顾burpsuite学习笔记               # 标题 
subtitle:   整理一下自己的学习文档
date:       2020-03-28              # 时间
author:     z0L1n                      # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - web漏洞
---

## 0x00 burpsuite介绍：
> burpsuite是用于攻击web应用程序的集成平台，我们可以手动或者自动的测试web应用程序的活动，在2.0以后的模块基本为以下几个：
- 1、targe（目录）显示目标目录结构的一个功能
- 2、proxy（代理）拦截http/s的请求，作为一个中间人，允许拦截查看修改在两个方向上的原始数据流
- 3、intruder（入侵）经常用于对web应用的自动化攻击，枚举标识符进行信息收集以及fuzzing技术探测漏洞
- 4、repeater（中继）手动去触发单独的http请求并能够分析应用响应
- 5、sequencer（会话）用来分析会话令牌以及重要数据的随机性
- 6、decoder（解码器）对数据进行智能解码编码
- 7、comparer（对比）通过一些相关请求和响应得到数据的可视化差异
- 8、extender（拓展）加载一些burpsuite的拓展

## 0x01 功能解析和学习：
- 1、proxy功能：

![1](https://wx2.sinaimg.cn/mw690/007IMTbqgy1gd9zrdf8a8j318g0smgrq.jpg)
>
首先我们可以看到代理监听，一般来说监听我们浏览器的8080端口，用burpsuite配合火狐使用，设置好火狐的代理即可在intercept模块里面开启侦听拦截。
如果是https请求，利用`import/export CA certificate`去加载证书就可以了，burpsuite下载之后，会生成自己的CA certificate，只需要导入就可以了。
![2](https://wx2.sinaimg.cn/mw690/007IMTbqgy1gd9zrjh6bej30ns0bg40g.jpg)
>
在拦截到请求包之后，我们有三个选项，分别是`forward`、`drop`、`action`，点击`forward`可以让请求前进，发送到服务器，点击`drop`可以丢弃该请求包，
如果我们对请求包有其他操作可以右键或者点击`action`进行。该模块内容比较掌握。

- 2、intruder功能：

![3](https://wx2.sinaimg.cn/mw690/007IMTbqgy1gd9zro4l0vj31770iitc3.jpg)
>
在target模块是设置我们的host以及port，选择是否为https。
在position模块，我们需要添加标识符，为payload模块作准备，一般来说，标识符的位置就是我们
稍后枚举payload的地方，通过枚举来完成fuzzing探测。
position中有个选项：attack type，里面有四种模式：
- ①sniper：对变量依次进行破解，多个标记依次进行
- ②battering ram：对变量同时进行破解，多个标记同时进行
- ③pitchfork：每个变量对应一个字典，取对应的字典项，该模式需要至少2个变量
- ④cluster bomb：每个变量对应一个字典，并且进行交替破解，尝试各种组合。
我们经常使用的是sniper模式，其他模式还是比较少用，cluster bomb适用于用户名和登陆密码的破解。
![4](https://wx1.sinaimg.cn/mw690/007IMTbqgy1gd9zrtchmuj316j0pgn1d.jpg)

**payloads：**
- 在这个模块中，我们首先可以设定的是payloadset，数量类型、
- 然后对于payload type：我经常使用的是simple list简单字典，numbers或者dates生成相应类型的payload，对于其他类型，简单学习一下
比如 runtime file：运行文件、custom iterator：自定义迭代器、brute forcer：暴力破解、recursive grep：递归查询
`payload options：`这个模块的内容会随着payload type的变更而改变，在使用过程中去学习
`payload processing：`此模块可以添加规则对生成的payload进行编码、加密、截取等操作
`options：`在此模块中设置我们发送请求的线程、重传次数、以及超时重试等，此外如果我们对于响应中的内容有查找提取的需求，可以在grep里面去设置，
另外对于重定向的处理，在redirections里面可以设置。

![5](https://wx4.sinaimg.cn/mw690/007IMTbqgy1gd9zs39g1pj314h0op77s.jpg)

- 3、repeater模块：

![6](https://wx1.sinaimg.cn/mw690/007IMTbqgy1gd9zsnla02j317e0kln19.jpg)

>我们拦截的请求发送到该模块的时候就可以手动修改的去触发单独的http请求探测漏洞，并分析响应，使用很简单。
go：当我们微调好之后，点击go就可以发出请求，也可以点击cancel取消。页面中<>这两个箭头的功能是返回上一次和下一个操作，在最底部我们可以搜索条件，可以用
正则表达式，底部右边显示匹配结果数。
在raw、params、headers、hex、render中我们可以相应的看纯文本、参数、请求响应头、十六进制数据以及对应的页面，使用简单，非常方便的就可以找到我们
想要的数据。
- 4、sequence模块：

![7](https://wx4.sinaimg.cn/mw690/007IMTbqgy1gd9zst8z67j31650pt425.jpg)

>该模块是随机数分析工具，在分析过程中可能会对系统造成不可预测的影响。接下来看一下该如何使用：
我们可以转发带有token或者参数的抓取历史到该模块下，然后点击config选中token或者参数的值然后进行抓取，也就是点击start live capture，当数据量足够大时
我们可以立即分析，也可以把获取的数据保存下来，在manual load模块中加载分析，分析时我么可以在 analysis options中进行一些设定，因为日常比较少用，就先
了解熟悉一下使用思路。

- 5、decoder模块：

![8](https://wx3.sinaimg.cn/mw690/007IMTbqgy1gd9zsxpeq7j317o0n6gnz.jpg)

>此模块功能比较简单：两个内容框分别为输入域和输出域，在右边侧栏为编码解码模块，根据实际需要我们可以选择编码、解码、散列这三个选项，基本编码选项支持
html/base64/ascii/gzip/进制转换，以及SHA/SHA-224/SHA-256/md5等，此功能一个特色就是，我们可以对同一数据进行多次的编码解码的转换。

- 6、comparer模块：

![8](https://wx1.sinaimg.cn/mw690/007IMTbqgy1gd9zt1medrj317f0m73zs.jpg)

>该模块主要提供一个可视化的差异对比，对比分析两个数据之间的区别、例如分析用户登陆成功与失败时，服务端反馈结果的差异，还有利用intruder攻击时，不同payload的不同响应差别
等，我们的数据加载可以从其他模块发过来，或者直接粘贴和文件加载然后进行分析，一般来说分析会让我们更快知道哪里的差异。


## 0x02 burp使用心得：
对于web应用来说，用的非常多的就是它了，非常好的一个工具，基本能满足我们的目前需求了，在专业版上面，还有漏洞检测功能，
更大程度上为我们的测试提供了便利，最主要的还是拦截和攻击模块，经常使用。一般就是抓包，微调测试参数值或者直接上intruder进行枚举攻击测试。





