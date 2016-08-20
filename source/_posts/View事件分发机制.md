title: View事件分发机制
date: 2016/3/11 20:46:25
categories:
- Android
- Android开发艺术探索笔记
tags:
- Android
- View
- 事件分发机制
---

# 介绍
点击事件的事件分发就是对MotionEvent事件的分发过程，当一个MotionEvent产生了以后，系统需要把这个事件传递给一个具体的View，而这个传递的过程就是分发的过程。
<!-- more -->

# 涉及到的三个方法

- dispatchTouchEvent：用来进行事件的分发，如果事件能够传递给当前View，那么此方法一定会被调用，返回结果受当前View的onTouchEvent和View的dispatchTouchEvent方法的影响，表示是否当消耗当前事件
- onInterceptTouchEvent：用来判断是否拦截某个事件，如果当前View拦截了某个事件，那么在同一个事件序列当中，此方法不会被再次调用，返回结果表示是否拦截当前事件；
- onTouchEvent：在dispatchTouchEvent方法中调用，用来处理点击事件，返回结果表示是否消耗当前事件，如果不消耗，则在同一个事件序列中，当前View无法再次接收到事件。

## 三个方法之间的关系

```java
public boolean dispatchTouchEvent(MotionEvent ev) { 
    boolean consume = false;
    if(onInterceptTouchEvent(ev)) { 
        consume = onTouchEvent(ev);
    } else { 
        consume = child.dispatchTouchEvent(ev); 
    }
    return consume; 
}
```

上面的伪代码很好的描述了三者之间的关系。如果当前View拦截事件，就交给自己的onTouchEvent去处理，否则就传给子View，直到事件被最终处理。

# 事件分发顺序
当一个点击事件产生后，它的传递过程如下：Activity -> Window -> View。如果View的onTouchEvent返回false，那么它的父容器onTouchEvent将会被调用，以此类推，最终将由Activity的onTouchEvent处理。

## Activity对事件的分发过程
**Activity -> Window -> DecorView。**

Windows是一个抽象类，可以控制**顶级View**的外观和行为策略，PhoneWindow是这个类的唯一个实现。
DecorView就是当前界面的底层容器，即setContentView所设置的View是它的一个子View。

## 顶级View对点击事件的分发过程

**ViewGroup -> dispatchTouchEvent -> onInterceptTouchEvent -> onTouch or onTouchEvent**

顶级View一般都是一个ViewGroup。拦截事件之后，如果ViewGroup设置了mOnTouchListener，则Listener里的onTouch方法会屏蔽掉onTouchEvent。如果onTouchEvent设置了mOnClickListener，则Listener里的onClick会被调用。如果ViewGroup没有拦截则传给子View直到整个事件分发完成。

##View对点击事件的处理过程
如果View设置了mOnTouchListener，则Listener里的onTouch方法会屏蔽掉onTouchEvent。如果onTouchEvent设置了mOnClickListener，则Listener里的onClick会被调用。
View没有onInterceptTouchEvent方法，一旦有点击事件传递给他，他就会处理。

注：上面只是描述了事件分发过程的原理，关于源码的分析请参考书本的相应章节。

欢迎转载，转载请注明出处[http://sparkyuan.github.io/](http://sparkyuan.github.io/)