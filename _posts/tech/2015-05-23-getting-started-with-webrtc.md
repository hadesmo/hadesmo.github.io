---
layout: post
title: Licode(一)：入门介绍
category: 技术                                                                                                                                                
tags: [webRTC, Licode]
keywords: webRTC,Licode
description: 
---

搭建一个基于webrtc的网络会议系统是加入新公司后的第一个项目，google一番之后无意中发现了licode这一开源项目，研究了下，这货针对网络会议或类似场景给出了基于webtrc的完整解决方案，弄的我相当感激涕零。同时，发现网上关于licode的文章少之又少，故想分享下自己关于licode的一些心得，尽绵薄之力避免后来者继续掉落坑中。

# 什么是webrtc？

> WebRTC（Web Real-Time Communication）是一个开源项目（2010年5月，Google以6820万美元收购VoIP软件开发商Global IP Solutions的GIPS引擎，并改为名为“WebRTC”），旨在让Web开发者能够基于Web浏览器轻易快捷开发出丰富的实时多媒体应用，而无需下载安装任何插件，Web开发者也无需关注多媒体的数字信号处理过程，只需编写简单的Javascript程序即可实现。W3C等组织正在制定Javascript 标准API。同时，Google也希望和致力于让WebRTC的技术成为HTML5标准之一。

可以通过[《Getting Started with WebRTC》](http://www.html5rocks.com/en/tutorials/webrtc/basics/?redirect_from_locale=zh)对webtrc有个更清晰的认识，由于网络上已经有很多涉及webrtc技术的文章，这里就不再详细讨论了，更多相关资料：

* webRTC中文社区： <http://www.webrtcbbs.com/forum.php>
* webRTC源代码（Note：需要翻墙，方可下载）：
git clone <https://chromium.googlesource.com/external/webrtc>

# 什么是Licode？

[Licode](http://lynckia.com/licode/index.html)是基于webRTC技术之上的开源项目，通过更便捷（easy，fast and scalable）的接口你可以快速搭建出基于webRTC技术的网络视频会议系统，或者与此类似的系统。你可以通过[Try it!](http://chotis2.dit.upm.es/)对Licode有个更为直观的认识。Licode的GitHub地址：<https://github.com/ging/licode>

![image](/public/upload/img/2015-05-23-getting-started-with-webrtc.md/licode.png)

# 初识Licode架构

官方给出的Licode架构如下图：

![image](/public/upload/img/2015-05-23-getting-started-with-webrtc.md/licode-architecture.png)

**Note：因为有动画效果，点击这里查看更多细节<http://lynckia.com/licode/architecture.html>**

Licode由四个模块组成：

* Erizo：基于webRTC针对视频会议场景的一对多组件，官方叫法为：MCU(Multipoint Control Unit)
* Erizo API：Erozo的NodeJs版本
* Erizo Controller：负责管理（manage）视频会议sessions
* Nuve：负责管理（manage）服务器资源（会议房间、与会用户、加入凭证等）

# Mac X下搭建Licode测试环境

通过官方文档<http://lynckia.com/licode/install.html>是无法把Licode安装到Mac Yosimite上的，github上的安装脚本是针对Mac Mountain Lion的，google了很久，终于发现了一种方法 --- 通过在mac上安装虚拟机的方式（虚拟机上在运行Ubuntu12.04 LTS）完美解决之，不容易啊：

> At this point I’m afraid the Mac OS building scripts are very outdated. To test Licode I suggest you to use our Vagrantfile located in extras/vagrant
You can find more information about vagrant in https://www.vagrantup.com
>
> We haven’t ruled out updating the scripts for Mac, but at this point we are focused on other issues. Sorry about that.

Vagrant的介绍、安装和基本的用法，这里不赘述了，直接参考下这里吧：[Vagrant的介绍](https://github.com/astaxie/Go-in-Action/blob/master/ebook/zh/01.1.md)，需要安装的box名称为：

```
# Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "precise32" 
  # The url from where the 'config.vm.box' box will be fetched if it
  # doesn't already exist on the user's system.
  config.vm.box_url = "http://files.vagrantup.com/precise32.box"
```

Vagrant初始化成功后，会在当前目录下生成Vagrantfile文件，直接将Licode项目$ROOT/extras/vagrant/下的两个文件：Vagrantfile和bootstrap.sh复制到当前目录下，然后执行vagrant up命令，耐心等待编译完成，然后执行以下命令：

```
# This step will initialize all Licode components.
./licode/scripts/initLicode.sh
```	

```
# This step will initialize all Licode components.
./licode/scripts/initBasicExample.sh
```	

执行成功之后，通过chrome浏览器 connect to "localhost:3001" and test your basic videoconference example：

![image](/public/upload/img/2015-05-23-getting-started-with-webrtc.md/licode-test.png)

# 写在后面

这是Licode源码分析的第一篇，后续的文章将陆续对Licode的组件进行分析，希望自己能坚持下去，在此过程中如果能帮忙到后来者，将是我莫大的荣幸。

参考资料：

* WebRTC: 无客户端也能实时通信：<http://cube.qq.com/?p=105>
* webRTC维基百科：<http://zh.wikipedia.org/zh-cn/WebRTC>
* webRTC入门指南（html5rocks版）：<http://www.webrtcbbs.com/forum.php?mod=viewthread&tid=11&extra=page%3D1>
* Getting Started with WebRTC：<http://www.html5rocks.com/en/tutorials/webrtc/basics/?redirect_from_locale=zh>
* Licode官网：<http://lynckia.com/licode/index.html>
* Vagrant的介绍：<https://github.com/astaxie/Go-in-Action/blob/master/ebook/zh/01.1.md>
