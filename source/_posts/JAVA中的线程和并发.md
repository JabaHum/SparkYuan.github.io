title: Java中的线程和并发
date: 2017/01/10 13:40:13
categories:
- Thread


---
线程调度和并发控制在Java知识体系中占有重要地位，本文主要讲解Java这方面的一些必须要知道的知识点。
<!-- more -->

# Thread
程序是静态的代码，进程是进程一次动态执行过程，对应了从代码加载、执行到执行完毕一个完整过程。同一程序可以被多次加载到系统的不同内存区域执行，形成不同进程。一个进程执行时可以产生多个线程。浏览器，下载的同时可以播放音乐。

## 实现方法 

```java
class SimpleThread extends Thread{
public void run(){}
}
...
//Thread.start()之后，即进入Runnable状态，当进入Running状态时，即调用run()
new SimpleThread("ThreadOne").start();

```

```java
class Hello implements Runnable{
public void run(){}
}
...
Thread t1 = new Thread(new Hello());//任何实现Runnable接口的对象都可以作为target参数。
t1.start();
```

4)	Daemon 
A "daemon" thread is intended to provide a general service in the background as long as the program is running, but is not part of the essence of the program. Thus, when all of the non- daemon threads complete, the program is terminated, killing all daemon threads in the process. 
守护线程的好处就是你不需要关心它的结束问题。例如你在你的应用程序运行的时候希望播放背景音乐，如果将这个播放背景音乐的线程设定为非守护线程，那么在用户请求退出的时候，你不仅要退出主线程，还要通知播放背景音乐的线程退出；如果设定为守护线程则不需要了。
5)	线程基本控制方法
i.	sleep() 把CPU让给优先级比其低的线程。
ii.	yield() 如果其他线程与当前线程相同优先级并是可运行的，把调用该方法的线程放入Runnable线程池，并允许其他线程运行。如果没有符合的线程，则什么都不做。
iii.	t.join()使当前线程等待直到线程t结束，该线程恢复到Runnable状态。
# 资源共享
常见的读后写，写后读造成的数据不一致问题。
## 解决方法
### synchronized method
To control access to a shared resource, you first put it inside an object. Then any method that uses the resource can be made synchronized. If a task is in a call to one of the synchronized methods, all other tasks are blocked from entering any of the synchronized methods of that object until the first task returns from its call.

``` java
synchronized void f() { /* ... */ }
synchronized void g() { /* ... */ }
```
 - if f( ) is called for an object by one task, a different task cannot call f( ) or g( ) for the same object until f( ) is completed and releases the lock.
 - Note that it’s especially important to make fields private when working with concurrency; otherwise the synchronized keyword cannot prevent another task from accessing a field directly, and thus producing collisions.
 - There’s also a single lock per class (as part of the Class object for the class), so that synchronized static methods can lock each other out from simultaneous access of static data on a class-wide basis.

 ### Lock objects

 ``` java
 public class MutexEvenGenerator extends IntGenerator {
  private int currentEvenValue = 0;
  private Lock lock = new ReentrantLock();
  public int next() {
    lock.lock();
    try {
      ++currentEvenValue;
      Thread.yield(); // Cause failure faster
      ++currentEvenValue;
      return currentEvenValue;
} finally {
      lock.unlock();
    }
}

 public static void main(String[] args) {
    EvenChecker.test(new MutexEvenGenerator());
  }
}
```
Lock对象提供了粒度更细的锁机制。
- unlock要放在finally里
- 在使用synchronized不能获得锁或者需求是当获得对象锁若干次之后就不再获得该对象锁，这个时候用Lock
7)	线程同步，对象锁
i.	排他锁，synchronized(this){临界区}
ii.	对某一个对象加锁，即使线程被强占，也不会破坏数据。
iii.	对象锁返还
1.	Synchronized()执行完
2.	语句块中出现异常
3.	调用wait()，此线程释放对象锁，放入wait pool中，sleep()不行
iv.	synchronized保护的共享数据必须是私有的
v.	public synchronized void push()
vi.	同一线程可以多次获得同一对象锁
vii.	线程间的交互
1.	线程调用x.wait()，线程放入x的wait pool，并该线程释放x的锁。当线程调用x.notify()时，x wait pool中的一个线程移入lock pool中等待x的锁，一旦获得便可运行。
2.	当线程需要共享数据改变时，调用wait()，暂时释放对象锁，允许其他对象修改，修改完成后调用notify()，通知等待的线程重新占有锁并运行。
3.	使用某类资源的线程称为消费者，产生或释放同类资源的线程成为生产者。
