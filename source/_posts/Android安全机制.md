title: Android安全机制
date: 2016/4/5 14:45:58
categories:
- Android
tags:
- 安全机制

---
Android系统是基于Linux内核开发的，因此，Android系统不仅保留和继承了Linux操作系统的安全机制，而且其系统架构的各个层次都有独特的安全特性。
<!-- more -->

# Linux内核安全机制

Android的Linux内核包含了强制访问控制机制和自主访问控制机制。强制访问控制机制由Linux安全模块来实现。自主访问控制机制通常由文件访问控制来实现，Linux文件系统的权限控制是由user、group、other与读(r) 、写(w) 、执行(x)的不同组合来实现的。这样，每个文件都有三个基本权限集，它们的组合可以容许、限制、拒绝用户、用户组和其他用户的访问。**通常，只有uid是“system”或“root”用户才拥有Android系统文件的访问权限，而应用程序只有通过申请Android权限才能实现对相应文件的访问**，也正因为此，Android使用内核层Linux的自主访问控制机制和运行时的Dalvik虚拟机来实现Android的“沙箱”机制。

# Android的“沙箱”机制
Android“沙箱”的本质是为了实现不同应用程序和进程之间的互相隔离，即在默认情况下，应用程序没有权限访问系统资源或其它应用程序的资源。每个APP和系统进程都被分配唯一并且固定的User Id，这个uid与内核层进程的uid对应。**每个APP在各自独立的Dalvik虚拟机中运行，拥有独立的地址空间和资源**。运行于Dalvik虚拟机中的进程必须依托内核层Linux进程而存在，因此**Android使用Dalvik虚拟机和Linux的文件访问控制来实现沙箱机制**，任何应用程序如果想要访问系统资源或者其它应用程序的资源必须在自己的manifest文件中进行声明权限或者共享uid。
Android中的数据分为system和data两个区，其中system是只读的，data是用来存放应用自己的数据，这样保证系统数据不会被随意改写。

# 应用权限机制
任何一个应用程序在使用Android受限资源（网络、电话、短信、蓝牙、通讯录、SdCard等）之前都必须以XML文件的形式事先向Android系统提出申请，等待Android系统批准后应用程序方可使用相应的资源，权限与Java的API是多对多的映射关系。

如何让两个app运行在同一个进程里？ 1. 两个app用相同的private key来签名。 2. 两个app的Manifest文件中添加android:sharedUserId 设置成相同的UID。