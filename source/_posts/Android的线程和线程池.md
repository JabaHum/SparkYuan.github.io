title: Android的线程和线程池
date: 2016/3/25 14:02:51
categories:
- Android
- Android开发艺术探索笔记
tags:
- AsyncTask
- HandlerThread
- IntentService
- Thread

---
在Java中默认情况下一个进程只有一个线程，也就是主线程，其他线程都是子线程，也叫工作线程。Android中的主线程主要处理和界面相关的事情，而子线程则往往用于执行耗时操作。线程的创建和销毁的开销较大，所以如果一个进程要频繁地创建和销毁线程的话，都会采用线程池的方式。
<!-- more -->

# Android中线程的形态

- 传统的Thread
- AsyncTask
- HandlerThread
- IntentService

## 各种线程形态的比较
![Threads](/images/threads.png)


## 传统的Thread

这是Java本身就支持的类，自定义化程度高，但是所有的功能都需要自己维护。

## AsyncTask
AsyncTask常用于可以在几秒钟完成的后台任务，关于AsyncTask的讲解可以看这一篇文章[http://sparkyuan.me/2016/03/23/AsyncTask源码剖析(API 23)/ ](http://sparkyuan.me/2016/03/23/AsyncTask%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90(API%2023)/)/)
讲解了AsyncTask的基本用法和源码分析。

## HandlerThread
HandlerThread继承了Thread，是一种可以使用Handler的Thread，它的实现就是在run方法中通过Looper.prepare()来创建消息队列，并通过Looper.loop()来开启消息循环，这样在实际的使用中就允许在HandlerThread中创建Handler了。外界可以通过Handler的消息方式通知HandlerThread执行一个具体的任务。
HandlerThread的一个应用场景就是用在IntentService中。HandlerThread的run方法是一个无限循环，因此当明确不需要再使用HandlerThread的时候，可以通过它的quit或者quitSafely方法来终止线程的执行，这是一个良好的编程习惯。

## IntentService
IntentService是一个特殊的Service，它继承自Service并且是个抽象类，要使用它就要创建它的子类。与AsyncTask不同的是，IntentService用于需要长时间执行的任务，因为他是Service，所以他的优先级比单纯的线程高很多。
IntentService的onCreate方法中会创建HandlerThread，并使用HandlerThread的Looper来构造一个Handler对象ServiceHandler，这样通过ServiceHandler对象发送的消息最终都会在HandlerThread中执行。IntentService会将Intent封装到Message中，通过ServiceHandler发送出去，在ServiceHandler的handleMessage方法中会调用IntentService的抽象方法onHandleIntent，所以IntentService的子类都要是实现这个方法。

### 一个例子

```java
public class LocalIntentService extends IntentService {
    private static final String TAG = "LocalIntentService";

    public LocalIntentService() {
        super(TAG);
    }

    @Override
    protected void onHandleIntent(Intent intent) {
        String action = intent.getStringExtra("task_action");
        Log.d(TAG, "receive task :" +  action);
        SystemClock.sleep(3000);
        if ("com.ryg.action.TASK1".equals(action)) {
            Log.d(TAG, "handle task: " + action);
        }
    }

    @Override
    public void onDestroy() {
        Log.d(TAG, "service destroyed.");
        super.onDestroy();
    }
}
```

```java
 Intent service = new Intent(this, LocalIntentService.class);
        service.putExtra("task_action", "com.sparkyuan.com");
        startService(service);
        
```

# Android中的线程池
## 使用线程池的优点

1. 重用线程，避免线程的创建和销毁带来的性能开销；
2. 能有效控制线程池的最大并发数，避免大量的线程之间因互相抢占系统资源而导致的阻塞现象；
3. 能够对线程进行简单的管理，并提供定时执行以及指定间隔循环执行等功能。

## ThreadPoolExecutor
Executor只是一个接口，真正的线程池是ThreadPoolExecutor。ThreadPoolExecutor提供了一系列参数来配置线程池，通过不同的参数可以创建不同的线程池，Android的线程池都是通过Executors提供的工厂方法得到的。
```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue)
                              ThreadFactory threadFactory
```

1. corePoolSize：核心线程数，默认情况下，核心线程会在线程中一直存活；
2. maximumPoolSize：最大线程数，当活动线程数达到这个数值后，后续的任务将会被阻塞；
3. keepAliveTime：非核心线程闲置时的超时时长，超过这个时长，闲置的非核心线程就会被回收；
4. unit：用于指定keepAliveTime参数的时间单位，有TimeUnit.MILLISECONDS、TimeUnit.SECONDS、TimeUnit.MINUTES等；
5. workQueue：任务队列，通过线程池的execute方法提交的Runnable对象会存储在这个参数中；
6. threadFactory：线程工厂，为线程池提供创建新线程的功能。它是一个接口，它只有一个方法Thread newThread(Runnable r)；
7. RejectedExecutionHandler：当线程池无法执行新任务时，可能是由于任务队列已满或者是无法成功执行任务，这个时候就会调用这个Handler的rejectedExecution方法来通知调用者，默认情况下，rejectedExecution会直接抛出一个rejectedExecutionException。

## ThreadPoolExecutor执行任务规则

1. 如果线程池中的线程数未达到核心线程的数量，那么会直接启动一个核心线程来执行任务
2. 如果线程池中的线程数量已经达到或者超过核心线程的数量，那么任务会被插入到任务队列中排队等待执行
3. 如果在步骤2中无法将任务插入到的任务队列中，可能是任务队列已满，这个时候如果线程数量没有达到规定的最大值，那么会立刻启动非核心线程来执行这个任务
4. 如果步骤3中线程数量已经达到线程池规定的最大值，那么就拒绝执行此任务，ThreadPoolExecutor会调用RejectedExecutionHandler的rejectedExecution方法来通知调用者。

## AsyncTask中的THREAD_POOL_EXECUTOR线程池的配置情况
1. corePoolSize=CPU核心数+1；
2. maximumPoolSize=2倍的CPU核心数+1；
3. 核心线程无超时机制，非核心线程在闲置时间的超时时间为1s；
4. 任务队列的容量为128。

## 线程池分类
1. FixedThreadPool：线程数量固定的线程池，它只有核心线程；
2. CachedThreadPool：线程数量不固定的线程池，它只有非核心线程；
3. ScheduledThreadPool：核心线程数量固定，非核心线程数量没有限制的线程池，主要用于执行定时任务和具有固定周期的任务；
4. SingleThreadPool：只有一个核心线程的线程池，确保了所有的任务都在同一个线程中按顺序执行。

### 示例

```java
private void runThreadPool() {
        Runnable command = new Runnable() {
            @Override
            public void run() {
                SystemClock.sleep(2000);
            }
        };

        ExecutorService fixedThreadPool = Executors.newFixedThreadPool(4);
        fixedThreadPool.execute(command);
        
        ExecutorService cachedThreadPool = Executors.newCachedThreadPool();
        cachedThreadPool.execute(command);
        
        ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(4);
        // 2000ms后执行command
        scheduledThreadPool.schedule(command, 2000, TimeUnit.MILLISECONDS);
        // 延迟10ms后，每隔1000ms执行一次command
        scheduledThreadPool.scheduleAtFixedRate(command, 10, 1000, TimeUnit.MILLISECONDS);

        ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();
        singleThreadExecutor.execute(command);
    }
    
```