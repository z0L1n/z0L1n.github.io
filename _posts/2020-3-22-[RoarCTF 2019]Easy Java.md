---
layout:     post
title:      [RoarCTF 2019]Easy Java
subtitle:   java源码泄露
date:       2020-3-22
author:     z0L1n
header-img: img/post-bg-ios9-web.jpg
catalog: 	 true
tags:
    - 源码泄露
    - 任意下载
---

#[RoarCTF 2019]Easy Java
##0x题解
###依照正常套路吧先来个题解：
- 1、首先我们可以看得的是一个初始页面带有登陆框还有个help链接，点了之后发现没有该文件，首先来个万能密码，以及sql注入简单检测一下，再进行过滤词筛选，发现全部都只返回一个结果！！！！！所以很可能这是假的！！！：
![1](https://github.com/z0L1n/z0L1n.github.io/blob/master/img/easyjava1.png "1")
- 2、回想起刚刚的help链接的url：/Download?filename=help.docx
似乎是下载的意思，但是我为什么没有下载？？？？？查了一下，get请求参数有限制，一般都使用post请求下载，所以抓包发post数据，成功下载help.docx文件，但是内容没有意义。
思考以及查看了表哥们的记录，发现知识点在于java源码泄露涉及到WEB-INF/web.xml泄露，通过这个下载点去下载java的源码文件，接着审计。
- 3、由于学习过java，复习了一下java文件结构：
 WEB-INF主要包含一下文件或目录：
/WEB-INF/web.xml：Web应用程序配置文件，描述了 servlet 和其他的应用组件配置及命名规则。
 /WEB-INF/classes/：含了站点所有用的 class 文件，包括 servlet class 和非servlet class，他们不能包含在 .jar文件中
 /WEB-INF/lib/：存放web应用需要的各种JAR文件，放置仅在这个应用中要求使用的jar文件,如数据库驱动jar文件
 /WEB-INF/src/：源码目录，按照包名结构放置各个java文件。
 /WEB-INF/database.properties：数据库配置文件
漏洞检测以及利用方法：通过找到web.xml文件，推断class文件的路径，最后直接class文件，在通过反编译class文件，得到网站源码 
构造payload：filename=/WEB-INF/web.xml：
![2](https://github.com/z0L1n/z0L1n.github.io/blob/master/img/easyjava2.png "2")

FlagController类里面包含flag信息，然后构造payload：filename=/WEB-INF/classes/com/wm/ctf/FlagController.class
下载后查看源码发现base64字符串解密后get到flag。
##0x思考与总结
>题目解完了，老惯例来个总结：在这题目中，由于下载点没有做相应的限制机制，导致源码泄露的问题出现，遇到java题目可以考虑源码泄露的利用。
get请求和post都能下载文件，但是get请求对参数有限制，一旦参数过多可能导致功能报错。
学习到的知识点：
1.javaweb的基本框架，以及WEB-INF这一安全机制目录，java源码泄露将导致严重的安全威胁
2.网站中的下载模块，需要限制机制，可以添加为一个漏洞测试点。
