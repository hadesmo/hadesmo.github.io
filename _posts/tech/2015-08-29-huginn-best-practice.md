---

layout: post
title: huginn：最佳实践
category: 技术

tags: [huginn, IFTTT ]
keywords: huginn;IFTTT

description:
---

## huginn是什么

[huginn](https://github.com/cantino/huginn)是一个功能上类似于IFTTT的开源项目，和IFTTT不同的是，huginn运行在自己搭建的VPS上，作者将其称为“hackable Yahoo”。

huginn项目的官方描述如下：

> Huginn is a system for building agents that perform automated tasks for you online. They can read the web, watch for events, and take actions on your behalf. Huginn's Agents create and consume events, propagating them along a directed graph.

另外，值得一提的是，huginn名字的由来：

> two ravens named Huginn and Muninn sit on Odin's shoulders. The ravens tell Odin everything they see and hear. Odin sends Huginn and Muninn out at dawn, and the birds fly all over the world before returning at dinner-time. As a result, Odin is kept informed of many events.

## huginn可以做什么

huginn能用来做什么，在huginn的[github主页](https://github.com/cantino/huginn)上已有很详细的说明，就不再赘述
这里分享下自己用huginn做了哪些事情吧，仅供参考：

- 用于漫画/动画追番：当有最新一期时，发送slack消息
- 违章查询：每天定时发送关于违章记录的slack消息
- 下雨提醒：当明天会下雨时，发送带伞提醒
- 微信公众号有我感兴趣主题(基于关键字)时，发送slack消息

![image](/public/upload/img/2015-08-29-huginn-best-practice.md/use-huginn.png)

各位可以大开脑洞，看看huginn还能帮助自己做哪些事情，个人觉得huginn+智能硬件的OPEN API很值得想象。

## huginn开发环境部署

在本机上部署huginn开发环境，过程很简单，照做如下步骤即可：

- 在github上fork下huginn项目，clone fork出来的huginn项目至本地：*git clone https://github.com/yourname/huginn*

- 将higunn中央仓库设置为远端仓库：*git remote add upstream https://github.com/cantino/huginn.git*

- 在项目根目录下执行命令：*cp .env.example .env*，然后编辑.env文件，将*APP_SECRET_TOKEN*变量设置为6-8位随机字符(字母+数字)

- 因为huginn项目基于ruby on rails，所以请先在本机上安装ruby on rails，网上已有很多教程，自行google之

- 在项目根目录下执行：*bundle*安装依赖库，huginn依赖的库较多，安装过程有些久

- 执行如下命令：*bundle exec rake db:create, bundle exec rake db:migrate, and then bundle exec rake db:seed*

- 执行：*bundle exec foreman start*，然后在浏览器中输入：*http://localhost:3000/*，即可看到登录界面
- 初始化用户名和密码分别是：*admin，password*，enjoy it

 > Note:
 > 1、开发环境下，邮件发送默认是关闭的，需要修改*.env*的配置项*SEND_EMAIL_IN_DEVELOPMENT*设置为*ture*
 > 2、huginn支持slack消息推送，个人推荐优先考虑slack推送，其次才是邮件推送
 > 3、可以到[ApiStorce](http://apistore.baidu.com/)搜索国内能提供API


## huginn部署至公网

huginn可以部署到Heroku，不过貌似有限制，所以不推荐，感兴趣的可以尝试，这里主要讲如何在VPS上部署huginn：

huginn的部署之前我是参考了这片文章《[Dploying huginn on any server or virtualbox using chef solo and or vagrant](https://github.com/cantino/huginn/wiki/Deploying-huginn-on-any-server-or-virtualbox-using-chef-solo-and-or-vagrant)》，期间采坑无数，足足折腾了两个晚上才最终搞定，苦不堪言。

不过，在写作本文时，已有人重新梳理了huginn的部署，大致看了下，部署过程简化了许多，也更加可控：《[Installation guide for Ubuntu/Debian](https://github.com/cantino/huginn/tree/master/doc/manual)》，而且还列出了默认配置huginn所需的[最小硬件需求](https://github.com/cantino/huginn/blob/master/doc/manual/requirements.md)，如需降低硬件需求，可以看下这篇文章《[Running Huginn on minimal systems with low RAM & CPU e.g. Raspberry Pi](https://github.com/cantino/huginn/wiki/Running-Huginn-on-minimal-systems-with-low-RAM-&-CPU-e.g.-Raspberry-Pi)》。

个人是在一台较低端的VPS（1 Core CPU/512M RAM/512M Swap/10G HDD）上运行hugin的，从部署完成到现在，半个多月了，运行良好。


## 写在最后

huginn上已经有相当丰富的agent可以使用，在使用agent的过程中，有一点感触很深：
每种类型的agent的功能都很强大，只要熟悉了配置规则，几乎仅依靠简单的配置就能实现你想要任何复杂功能
除了实现基本功能外，无需修改代码既能提供非常好的扩展性，这点很值得学习

后续，打算研究下huginn的源代码，学习其是如何组织代码的

参考资料：

- huginn主页：https://github.com/cantino/huginn
