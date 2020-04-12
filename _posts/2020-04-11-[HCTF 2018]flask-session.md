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

## 0x01:flask-session解密以及篡改：  
由于在ctf中遇到过一题，需要解密flask-session然后重新构造session关键内容的值达到伪造管理员身份的目的，session加密后的内容构造分为三部分分别以.分隔，第一部分为session信息，第二部分为时间戳，第三部分为session信息和时间戳以及secret_key的签名验证：   
在这里我们会用到github上的一个脚本flask-session-cookie-manager：   
核心代码：  
```
app = MockApp(secret_key)
s = si.get_signing_serializer(app)
s.dumps(session_cookie_structure)
如果是没有secret_key的，直接base64解密或者zlib解压字符串即可，一般来说为了安全性，secret_key设置的一般都挺多。
```

**Hctf admin：  
1、在首页中发现，一个提示，猜测需要admin账户：  
![](https://github.com/z0L1n/pic/blob/master/2002/flask-admin/1.png?raw=true)  
接下来还是正常测试各个页面，注册登录后在修改密码的页面发现另外一个提示：  
![](https://github.com/z0L1n/pic/blob/master/2002/flask-admin/2.png?raw=true)  
这就是github代码泄露，我们去下载后进行审计，由于是python写的flask，直接查看route相关的，数据库文件以及配置文件这些重要的：  
![](https://github.com/z0L1n/pic/blob/master/2002/flask-admin/3.png?raw=true)  
数据库文件中查看到两条信息，但是无法破解密码，另寻其他出路     
![](https://github.com/z0L1n/pic/blob/master/2002/flask-admin/4.png?raw=true)  
配置文件中发现secret_key，flask sesssion的构造需要使用，这个时候我们可以篡改session得到admin账户身份。    
![](https://github.com/z0L1n/pic/blob/master/2002/flask-admin/5.png?raw=true)  
同时在index.html中发现题目flag获取信息：  
接下来我们就利用原身份，在index页面利用burp截取请求，然后篡改访问index页面：  
![](https://github.com/z0L1n/pic/blob/master/2002/flask-admin/6.png?raw=true)  
![](https://github.com/z0L1n/pic/blob/master/2002/flask-admin/7.png?raw=true)  
解密后内容，可以看出session泄露了很多内容，一旦我们使用session加密时用的secure_key，就可以篡改成功了。把name的值改为admin后，使用脚本加密session内容：  
![](https://github.com/z0L1n/pic/blob/master/2002/flask-admin/8.png?raw=true)  
替换原session，由响应可以看到，flask session安全漏洞利用成功。  
![](https://github.com/z0L1n/pic/blob/master/2002/flask-admin/9.png?raw=true)  




