---

layout: post
title: IOS平台上使用jrtplib库
category: 技术

tags: [IOS, Xcode, jrtplib ]

keywords: ios;mac;Xcode;jrtplib;

description:
---

最近在开发一个IOS版本app，因其通信协议部分采用了RTP/RTCP协议，所以需要将jrtplib库移植到IOS平台下。而google（表问我为何不用百度）了下，发现关于如何在IOS平台（或者说Xcode）下编译jrtplib库的文章很少，所以就有了这边文章，希望能帮到后来者。

## Xcode下如何生成静态库

IOS平台下编译成静态库和Android平台下有很大的不同，Android平台主要通过```NDK```工具包：

>The Native Development Kit (NDK) is a set of tools that allow you to leverage C and C++ code in your Android apps. You can use it either to build from your own source code, or to take advantage of existing prebuilt libraries.

而且可以通过```Android.mk```的build file来组织C/C++源码文件，```Android.mk```语法和```MakeFile```文件很相似。

IOS平台下编译成静态库则相对来说比较简单：通过创建特定的Xcode项目（Cocoa Touch Static Library）、导入源码文件并点击编译即可。

这里分享一个很详细的教程[《Xcode7中创建静态库》](http://www.jianshu.com/p/656ba8094d1d)。

## ios-jrtplib库

注意，如果希望jrtplib库能在Xcode下顺利编译通过的话，还需要对源码进行小小的修改（主要是recvfrom函数调用方式的兼容性问题），所以这里直接给出一个自己修改之后**可直接使用的IOS版jrtplib库**：[ios-jrtplib](https://github.com/hadesmo/ios-jrtplib)。
