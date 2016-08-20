title: Activity在异常情况下的生命周期
date: 2016/2/5 16:46:25
categories:
- Android
- Android开发艺术探索笔记
tags:
- Activity

---
关于Activity正常情况下的生命周期讲解的文章已经很多了，本文讲解一下Activity在异常情况下的生命周期。
<!-- more -->

# 情况1：资源相关的系统配置发生改变
资源相关的系统配置发生改变，举个栗子。当前Activity处于竖屏状态的时候突然转成横屏，系统配置发生了改变，Activity就会销毁并且重建，其onPause, onStop, onDestory均会被调用。因为实在异常情况下终止的，所以系统会调用**onSaveInstanceState**来保存当前Activity状态。这个方法是在onStop之前，与onPause没有固定的时序关系。当Activity重建的时候系统会把onSaveInstanceState所保存的Bundle作为对象传递给onRestoreInstanceState和onCreate方法。

**注：**

- View的源码中每个View都有onSaveInstanceState和onRestoreInstanceState这两个方法。
- 接收位置可以是onRestoreInstanceState和onCreate方法，区别是：onRestoreInstanceState如果被调用，参数Bundle一定是有值的，在onCreate中需要判断参数是否为null。
- onSaveInstanceState只有在Activity即将销毁并有机会重新显示时才会调用，正常销毁的Activity生命周期中不会调用，**比如：旋转屏幕，按Home键，启动新Activity等。**

# 情况2：资源内存不足导致低优先级Activity被杀死
## Activity优先级
1. 前台Activity——正在和用户交互的Activity，优先级最高
2. 可见但非前台Activity——Activity中弹出的对话框导致Activity可见但无法交互
3. 后台Activity——已经被暂停的Activity，优先级最低

系统内存不足是，会按照以上顺序杀死Activity，并通过onSaveInstanceState和onRestoreInstanceState这两个方法来存储和恢复数据。
## 不让Activity重新创建的方法
系统配置有很多内容，当某项改变时，我们不想让Activity重新创建可以在AndroidMainfest中给Activity指定configChanges属性。比如
```html
 android:configChanges="orientation"
```
configChanges属性非常多，[具体可参考官方文档](http://developer.android.com/intl/zh-cn/guide/topics/manifest/activity-element.html#config)
常用的有locale, orientation和keyboardHidden这三个。

