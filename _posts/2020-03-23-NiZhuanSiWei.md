---
layout:     post                    # 使用的布局（不需要改）
title:      刷题日记-[ZJCTF 2019]NiZhuanSiWei               # 标题 
subtitle:   简单的一道web文件包含、伪协议利用
date:       2020-03-23              # 时间
author:     z0L1n                      # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - 文件包含
    - php伪协议
    - 反序列化
---



## 0x题解：
>先说说题解：
首先打开页面，呈现出来的是一串php代码，我们一起来看看：

![](https://github.com/z0L1n/pic/blob/master/2002/pic/007IMTbqgy1gd45ydnz25j30mn0d1756.jpg?raw=true)

**信息收集**
- 1、url可以包含三个get参数：text、file、password
- 2、要满足第一个条件：
`file_get_contents($text,'r')==="welcome to the zjctf"`
构造`text=data://text/plain;base64,d2VsY29tZSB0byB0aGUgempjdGY=`,可以让`file_get_contents`函数获取text内容，满足条件
- 3、file参数不能含有关键字‘flag’
- 4、文件包含file，其中后面有提示，我们可以利用文件包含漏洞来获取useless.php源码看看情况
- 5、password参数的内容需要经过反序列化，然后输出，自然而然涉及到类的序列化，很有可能useless.php里面的内容和序列化有关。
接下来开始尝试构造url：/?text=data://text/plain;base64,d2VsY29tZSB0byB0aGUgempjdGY=&file=php://filter/read=convert.base64-encode/resource=useless.php

![](https://github.com/z0L1n/pic/blob/master/2002/pic/007IMTbqgy1gd45yf830rj30sv052mx7.jpg?raw=true)

获取到一串base64代码，解密得：
![](https://github.com/z0L1n/pic/blob/master/2002/pic/007IMTbqgy1gd45yh5u6dj30jp0d6aag.jpg?raw=true)

这里涉及到类Flag，含有属性file，以及方法输出file的文件内容，题目提醒flag.php,结合上面的password反序列化后，输出password结构，我们可以将$file=flag.php,然后序列化构造后，传递给password后，反序列化时能获取flag.php的内容：
构造payload：
`?text=data://text/plain;base64,d2VsY29tZSB0byB0aGUgempjdGY=&file=php://filter/read=convert.base64-encode/resource=useless.php&password=O:4:%22Flag%22:1:{s:4:%22file%22;s:8:%22flag.php%22;}`

！！！此时发现页面和第一次构造的内容相当，究竟是什么问题呢，在文件包含时已经输出可能导致后续代码未执行就退出了，所以=把file后面的参数修改成file=useless.php进行尝试：

![](https://github.com/z0L1n/pic/blob/master/2002/pic/007IMTbqgy1gd45ykjgmhj30ct08dt8n.jpg?raw=true)

该页面没有什么，神仙右键查看源码看一看：
![](https://github.com/z0L1n/pic/blob/master/2002/pic/007IMTbqgy1gd45yn4nvmj30k008wjru.jpg?raw=true)
get到flag题目完成。


##0x知识点反顾与思考：

>老规矩，来个知识点总结以及思考：
- 1、这道题主要简单考察了代码审计，文件包含与php伪协议的结合使用，还有简单考察了反序列化的知识点。
- 2、在见到初始页面时，简单的代码审计没有什么难度，见到include函数倒是反应出可能存在文件包含利用，将参数file的内容获取为经过伪协议处理过的useless.php源码，然后文件包含的利用时，则包含了这一串字符串，一旦获取即可拿到源码，序列化的利用是比较简单的在本地去构造需要的参数内容然后序列化输出赋值给password就行。
- 3、在这次题目中加深了序列化与反序列化的知识，对于php的伪协议利用更加熟悉了，顺便对代码审查能力简单检验了一下。


