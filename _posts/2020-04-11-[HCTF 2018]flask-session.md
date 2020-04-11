---
layout:     post                    # 使用的布局（不需要改）
title:      python-flask-session安全漏洞               # 标题 
subtitle:   flask-session仅签名的安全漏洞
date:       2020-04-11              # 时间
author:     z0L1n                      # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - 基础
---

## 0x00：flask session问题：
涉及到python安全问题，flask是基于python的非常轻量的web框架，sessio直接储存在客户端中，同时session的内容可以通过抓包查看，截取的http请求头
的cookie字段中的session就是我们所需要的，其中敏感数据加密后存储在session中，该session仅进行了签名，缺少数据防篡改实现，这便很容易存在安全漏洞
，这里拓展一点点，Django默认将session储存在数据库中。
![]()

## 0x01:由flask-session解密学习：
由于在ctf中遇到过一题，需要解密flask-session然后重新构造session达到伪造管理员身份的目的，先贴出解密代码：
```
data = payload.split(".")[0]
data = base64_decode(data)
if compressed:
   data = zlib.decompress(data)
```
