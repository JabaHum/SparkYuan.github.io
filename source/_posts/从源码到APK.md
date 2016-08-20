title: 从源码到apk——apk打包过程
date: 2016/4/1 16:12:53
categories:
- Android
tags:
- apk

---
Android程序是怎么从源码变成可以安装使用的apk的
<!-- more -->

# 流程

## 官方版
![build](/images/build.png)

## 详细版
![build-detail](/images/android_build_process_detail.png)

上面就是一个关于构建过程的一个典型的流程图。

- aapt（Android Asset Packaging Tool）给你的Activity提供所需的资源文件,如 AndroidManifest.xml，XML文件,并编译它们。同时产生R.java文件，使你可以在java代码中引用这些资源。
- aidl工具把.aidl接口转换成Java接口。
- 你所有的Java代码,包括 R.java和 .aidl文件,由Java编译器和编译输出.class文件。
- dex工具把.class文件转换成Dalvik字节文件，第三方的类和.class也被转换成.dex文件
- 所有无法编译的资源（比如图片），编译好的资源文件和.dex都被送到apkbuilder工具中，生成最后的.apk
- 生成.apk时必须制定是debug还是release，release还要提供相应的key
- 如果选择release版本，还需要使用zipalign工具对apk对齐。齐处理即使得所有资源文件距离文件起始偏移为4字节的整数倍，这样通过内存映射访问apk文件时处理速度更快。

# 输出
生成的apk在app/build/outputs/apk/目录下，命名规则 app-<flavor>-<buildtype>.apk，例如，app-demo-debug.apk.

