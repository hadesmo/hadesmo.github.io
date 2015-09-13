---

layout: post
title: 使用toolchian编译protobuf库
category: 技术

tags: [protobuf, ndk, toolchian, android ]
keywords: protobuf;ndk;toolchian;android

description:
---

最近接手了一个android ndk项目，需要在android平台下使用C++版本的protobuf库，然却发现网上关于如何使用toolchain工具交叉编译protobuf库的文章是少之又少，所以这里分享下自己的经验吧，希望能帮助到后来者。

## 写在前面

在ndk的官方文档上，有一段话想特别贴出来：

> The NDK is not appropriate for most novice Android programmers, and has little value for many types of Android apps. It is often not worth the additional complexity it inevitably brings to the development process. However, it can be useful in cases in which you need to:
>
> - Squeeze extra performance out of a device for computationally intensive applications like games or physics simulations.
> - Reuse your own or other developers' C or C++ libraries.

以ndk方式进行android客户端开发所带来的性能提升，往往低于预期；而因此带来的复杂度却又常常超乎预料，而且，网上关于ndk的文章都较少，被坑的几率较大，所以希望大家在实际的项目中关于是否要使用ndk要慎之又慎。

## 编译ndk项目

如果你对于如何编译ndk项目，不是很熟悉的话，请先仔细阅读下这篇文章：[《Building Your Project》](https://developer.android.com/ndk/guides/build.html)，这里对于这些基础知识不会进行过多介绍。

## 交叉编译protobuf 2.5 库

- 下载protobuf库：```wget http://protobuf.googlecode.com/files/protobuf-2.5.0.tar.gz```

- 把protobuf库放置到ndk sources目录下并解压：```cp protobuf-2.5.0.tar.gz $NDK/sources;cd $NDK/sources;tar fxz protobuf-2.5.0.tar.gz;rm protobuf-2.5.0.tar.gz;cd -```

- protobuf库主目录下创建build脚本：```touch build_android.sh```，并写入如下内容：

```
#!/bin/bash
export NDK=YOUR_NDK_PATH/android-ndk-r10e
export SYSROOT=$NDK/platforms/android-15/arch-arm/

export TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.9/prebuilt/darwin-x86_64/

export PATH=$PATH:$TOOLCHAIN/bin

export CC="arm-linux-androideabi-gcc --sysroot $SYSROOT"
export CXX="arm-linux-androideabi-g++ --sysroot $SYSROOT"
export CXXSTL=$NDK/sources/cxx-stl/gnu-libstdc++/4.8

function build_one
{
    mkdir build

    ./configure --prefix=$(pwd)/build \
    --host=arm-linux-androideabi \
    --with-sysroot=$SYSROOT \
    --enable-static \
    --disable-shared \
    --enable-cross-compile \
    --with-protoc=protoc LIBS="-lc" \
    CFLAGS="-march=armv7-a" \
    CXXFLAGS="-march=armv7-a -I$CXXSTL/include -I$CXXSTL/libs/armeabi-v7a/include -L$CXXSTL/libs/armeabi-v7a/ -lgnustl_static"

    make clean
    make
    make install
}

CPU=arm
PREFIX=$(pwd)/android/$CPU
ADDI_CFLAGS="-marm"
build_one

# Inspect the library architecture specific information
# arm-linux-androideabi-readelf -A build/lib/libprotobuf-lite.a
```

注意：CXXSTL变量的值，需要根据你下载的ndk版本进行相应的调整，也尝试过使用portstl库，但发现编译不通过，所以使用了gnustl库

- 执行编译脚本```./build_android.sh```，编译时间可能较长请耐心等待

- ```ls -R build```查看a库文件是由已经成功生成

```
build/lib:
libprotobuf-lite.a  libprotobuf-lite.la libprotobuf.a       libprotobuf.la      libprotoc.a         libprotoc.la
```

- 在build目录下创建```Android.mk```文件：

```
LOCAL_PATH:= $(call my-dir)

 include $(CLEAR_VARS)
LOCAL_MODULE:= libprotobuf
LOCAL_SRC_FILES:= lib/libprotobuf.a
LOCAL_EXPORT_C_INCLUDES := $(LOCAL_PATH)/include
include $(PREBUILT_STATIC_LIBRARY)

 include $(CLEAR_VARS)
LOCAL_MODULE:= libprotoc
LOCAL_SRC_FILES:= lib/libprotoc.a
LOCAL_EXPORT_C_INCLUDES := $(LOCAL_PATH)/include
include $(PREBUILT_STATIC_LIBRARY)

 include $(CLEAR_VARS)
LOCAL_MODULE:= libprotobuf-lite
LOCAL_SRC_FILES:= lib/libprotobuf-lite.a
LOCAL_EXPORT_C_INCLUDES := $(LOCAL_PATH)/include
include $(PREBUILT_STATIC_LIBRARY)
```

- 修改需要使用protobuf库的NDK项目```Application.mk```文件：

```
APP_PLATFORM            :=  android-21

APP_ABI                 :=  armeabi

APP_STL                 :=  gnustl_static

APP_CPPFLAGS            += -frtti
APP_CPPFLAGS            += -fexceptions
```

- 现在可以在ndk项目中正常使用protobuf库了，enjoy it!

参考资料：

- [https://developer.android.com/ndk/guides/build.html](https://developer.android.com/ndk/guides/build.html)
- [How to build protocol buffer by Android NDK](http://stackoverflow.com/questions/7144008/how-to-build-protocol-buffer-by-android-ndk)
- [Build Google Protobuf library for android development](https://gist.github.com/helayzhang/9034454)
