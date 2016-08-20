title: IntentFilter匹配规则详解
date: 2016/2/6 14:51:51
categories:
- Android
- Android开发艺术探索笔记
tags:
- IntentFilter
- 匹配规则
---
启动Activity的方式分为两种，显示和隐式调用。显示调用很简单，直接指明要启动的Activity就可以了，这里主要介绍一下隐式调用。隐式调用需要Intent能够匹配目标组件的IntentFilter中所设置的过滤信息。**只有一个Intent同时匹配action，category和data才算匹配成功。**
<!-- more -->

# 示例

```
<intent-filter>
    <action android:name="com.sparkyuan.c" />
    <action android:name="com.sparkyuan.d" />

    <category android:name="com.sparkyuan.c" />
    <category android:name="com.sparkyuan.d" />
    <category android:name="android.intent.category.DEFAULT" />

    <data android:mimeType="text/plain" />
</intent-filter>`
```

# action匹配规则
action是一个字符串，系统预定义了一些action，我们也可以自己定义action。
添加方法

```java
intent.setAction("com.sparkyuan.a");
```
注：

- 一个intent-filter中可以有多个action
- intent中的action与intent-filter中有一个相同即可
- action区分大小写；

# category匹配规则
添加方法

```java
intent.addCategory("com.sparkyuan.d");
```
注：

- intent中可以不存在category，但如果存在就必须匹配intent-filter其中一个
- 系统在startActivity或者startActivityForResult的时候默认为Intent加上一个android.intent.category.DEAFAULT，所以必须在intent-filter中加上android.intent.category.DEFAULT这个category

# data匹配规则
## data语法
```
<data android:scheme="http"
      android:host="www.baidu.com"
      android:port="80"
      android:path="string"
      android:pathPattern="string"
      android:pathPrefix="string"
      android:mimeType="text/plain" />
```

## 组成
data由两部分组成，mimeType和URI。
URI格式如下：
````
<scheme>://<host>:<port>[<path>|<pathPrefix>|<pathPattern>]
//示例
content://com.example.project:200/folder/subfolder/etc 
````
##匹配规则
匹配规则与action类似，只要有一个data匹配就可以。

注：
```
<data  android:mimeType="text/plain" />
```
虽然没有指定URI，但是URI有默认值，**默认值为content和file**，所以intent的URI部分必须为content或者file才可以。
下面的方法可以匹配他
 ```
 intent.setDataAndType(Uri.parse("file://abc"),"text/plain");
 ```
# 最后
 
intent-filter的规则对于Service和BroadcastReceiver是一样的，但是对于Service建议尽量使用显示方法来启动。
在使用隐式Intent时可以先对是否有相应的Activity做出判断，以防出错。采用PackageManager的resolveActivity方法或者Intent的resolveActivity，如果匹配不到就返回null。

``` java
contex.getPackageManager().resolveActivity(intent, PackageManager.MATCH_DEFAULT_ONLY);
intent.resolveActivity(getPackageManager());
```
 **MATCH_DEFAULT_ONLY**的目的是去除那些category中不含DEFAULT的Activity。