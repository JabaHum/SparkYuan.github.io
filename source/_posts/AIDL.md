title: 属性动画
categories:
- Android
- Android开发艺术探索笔记
tags:
- AIDL
- IPC
---
Android Interface Definition Language (AIDL)， Android接口定义语言。系统中的进程之间不能共享内存，因此，需要提供一些机制在不同进程之间进行数据通信Interprocess communication (IPC)。AIDL就是解决这个问题的。
<!-- more -->

#介绍
Android Interface Definition Language (AIDL)， Android接口定义语言。系统中的进程之间不能共享内存，因此，需要提供一些机制在不同进程之间进行数据通信Interprocess communication (IPC)。AIDL就是解决这个问题的。
阅读本文需要了解Service的相关知识，关于Service的讲解请参考[http://blog.csdn.net/l664675249/article/details/48899323](http://blog.csdn.net/l664675249/article/details/48899323)

#创建.aidl文件
aidl是用Java语法编写的，后缀为.aidl的文件。

- 每一个aidl文件必须定义一个接口，在这个接口里声明方法
- 在aidl里不能有static属性（field）
- aidl支持基本的数据类型，当你需要使用额外的数据类型时需要把它们import进来，即使它们跟这个文件在同一个包中。


##示例

```java
// IRemoteService.aidl
package com.example.android;

// Declare any non-default types here with import statements

/** Example service interface */
interface IRemoteService {
    /** Request the process ID of this service, to do evil things with it. */
    int getPid();

    /** Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
    void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat,
            double aDouble, String aString);
}
```

把aidl文件存在src/目录下，当你build项目的时候，SDK工具会在gen/目录下生成一个与.aidl文件名字相同的.java文件。
#实现接口
生成的IRemoteService.java如下

```java
public interface IRemoteService extends android.os.IInterface
{
/** Local-side IPC implementation stub class. */
public static abstract class Stub extends android.os.Binder implements com.spark.meizi.IRemoteService
{
private static final java.lang.String DESCRIPTOR = "com.spark.meizi.IRemoteService";
/** Construct the stub at attach it to the interface. */
public Stub()
{
this.attachInterface(this, DESCRIPTOR);
}
/**
 * Cast an IBinder object into an com.spark.meizi.IRemoteService interface,
 * generating a proxy if needed.
 */
public static com.spark.meizi.IRemoteService asInterface(android.os.IBinder obj)
{
if ((obj==null)) {
return null;
}
android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
if (((iin!=null)&&(iin instanceof com.spark.meizi.IRemoteService))) {
return ((com.spark.meizi.IRemoteService)iin);
}
return new com.spark.meizi.IRemoteService.Stub.Proxy(obj);
}
@Override public android.os.IBinder asBinder()
{
return this;
}
@Override public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException
{
switch (code)
{
case INTERFACE_TRANSACTION:
{
reply.writeString(DESCRIPTOR);
return true;
}
case TRANSACTION_basicTypes:
{
data.enforceInterface(DESCRIPTOR);
int _arg0;
_arg0 = data.readInt();
long _arg1;
_arg1 = data.readLong();
boolean _arg2;
_arg2 = (0!=data.readInt());
float _arg3;
_arg3 = data.readFloat();
double _arg4;
_arg4 = data.readDouble();
java.lang.String _arg5;
_arg5 = data.readString();
this.basicTypes(_arg0, _arg1, _arg2, _arg3, _arg4, _arg5);
reply.writeNoException();
return true;
}
}
return super.onTransact(code, data, reply, flags);
}
private static class Proxy implements com.spark.meizi.IRemoteService
{
private android.os.IBinder mRemote;
Proxy(android.os.IBinder remote)
{
mRemote = remote;
}
@Override public android.os.IBinder asBinder()
{
return mRemote;
}
public java.lang.String getInterfaceDescriptor()
{
return DESCRIPTOR;
}
/**
     * Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
@Override public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, java.lang.String aString) throws android.os.RemoteException
{
android.os.Parcel _data = android.os.Parcel.obtain();
android.os.Parcel _reply = android.os.Parcel.obtain();
try {
_data.writeInterfaceToken(DESCRIPTOR);
_data.writeInt(anInt);
_data.writeLong(aLong);
_data.writeInt(((aBoolean)?(1):(0)));
_data.writeFloat(aFloat);
_data.writeDouble(aDouble);
_data.writeString(aString);
mRemote.transact(Stub.TRANSACTION_basicTypes, _data, _reply, 0);
_reply.readException();
}
finally {
_reply.recycle();
_data.recycle();
}
}
}
static final int TRANSACTION_basicTypes = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
}
/**
     * Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, java.lang.String aString) throws android.os.RemoteException;
}
```
在这个文件中有一个**内部类Stub**，这是父接口的一个抽象实现，并声明了aidl中的所有方法。为了实现由aidl生成的接口，我们需要继承Stub并实现从aidl继承过来的方法。下面是一个使用匿名类的例子

```java
private final IRemoteService.Stub mBinder = new IRemoteService.Stub() {
    public int getPid(){
        return Process.myPid();
    }
    public void basicTypes(int anInt, long aLong, boolean aBoolean,
        float aFloat, double aDouble, String aString) {
        // Does nothing
    }
};
```
这样mBinder就是一个Stub的实例了，下一步就是如何在client端使用，与service端产生交互了。
注：

- 不能保证请求是在主线程被执行的，所以从构建到使用要考虑Service线程的安全性
- 默认情况下，请求是同步的，所以尽量不要在主线程中发出请求
- 所有的异常都不会返回给请求者（Caller）

#在Client中使用接口
当你已经实现你的Service之后，你需把它暴露在Client中使Client可以绑定它。继承Service并实现onBind()方法，来返回一个实现了Stub的实例。下面就是一个把IRemoteService暴露给Client的例子：

```java
public class RemoteService extends Service {
    @Override
    public void onCreate() {
        super.onCreate();
    }

    @Override
    public IBinder onBind(Intent intent) {
        // Return the interface
        return mBinder;
    }

    private final IRemoteService.Stub mBinder = new IRemoteService.Stub() {
        public int getPid(){
            return Process.myPid();
        }
        public void basicTypes(int anInt, long aLong, boolean aBoolean,
            float aFloat, double aDouble, String aString) {
            // Does nothing
        }
    };
}

```
现在client（比如是一个Activity）可以调用bindService()来连接这个Service，通过onServiceConnected()来接收Service中 onBind() 返回的mBinder，最后使用YourServiceInterface.Stub.asInterface(service)来把返回的mBinder转换成YourServiceInterface类型。例子如下：

```java
IRemoteService mIRemoteService;
private ServiceConnection mConnection = new ServiceConnection() {
    // Called when the connection with the service is established
    public void onServiceConnected(ComponentName className, IBinder service) {
        // Following the example above for an AIDL interface,
        // this gets an instance of the IRemoteInterface, which we can use to call on the service
        mIRemoteService = IRemoteService.Stub.asInterface(service);
    }

    // Called when the connection with the service disconnects unexpectedly
    public void onServiceDisconnected(ComponentName className) {
        Log.e(TAG, "Service has unexpectedly disconnected");
        mIRemoteService = null;
    }
};
```
注：
如果Service和Client在两个不同的Application中，Client的Application的src/目录下必须也有对应的.aidl文件。
#一个Client的例子
```java
public static class Binding extends Activity {
    /** The primary interface we will be calling on the service. */
    IRemoteService mService = null;
    /** Another interface we use on the service. */
    ISecondary mSecondaryService = null;

    Button mKillButton;
    TextView mCallbackText;

    private boolean mIsBound;

    /**
     * Standard initialization of this activity.  Set up the UI, then wait
     * for the user to poke it before doing anything.
     */
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.remote_service_binding);

        // Watch for button clicks.
        Button button = (Button)findViewById(R.id.bind);
        button.setOnClickListener(mBindListener);
        button = (Button)findViewById(R.id.unbind);
        button.setOnClickListener(mUnbindListener);
        mKillButton = (Button)findViewById(R.id.kill);
        mKillButton.setOnClickListener(mKillListener);
        mKillButton.setEnabled(false);

        mCallbackText = (TextView)findViewById(R.id.callback);
        mCallbackText.setText("Not attached.");
    }

    /**
     * Class for interacting with the main interface of the service.
     */
    private ServiceConnection mConnection = new ServiceConnection() {
        public void onServiceConnected(ComponentName className,
                IBinder service) {
            // This is called when the connection with the service has been
            // established, giving us the service object we can use to
            // interact with the service.  We are communicating with our
            // service through an IDL interface, so get a client-side
            // representation of that from the raw service object.
            mService = IRemoteService.Stub.asInterface(service);
            mKillButton.setEnabled(true);
            mCallbackText.setText("Attached.");

            // We want to monitor the service for as long as we are
            // connected to it.
            try {
                mService.registerCallback(mCallback);
            } catch (RemoteException e) {
                // In this case the service has crashed before we could even
                // do anything with it; we can count on soon being
                // disconnected (and then reconnected if it can be restarted)
                // so there is no need to do anything here.
            }

            // As part of the sample, tell the user what happened.
            Toast.makeText(Binding.this, R.string.remote_service_connected,
                    Toast.LENGTH_SHORT).show();
        }

        public void onServiceDisconnected(ComponentName className) {
            // This is called when the connection with the service has been
            // unexpectedly disconnected -- that is, its process crashed.
            mService = null;
            mKillButton.setEnabled(false);
            mCallbackText.setText("Disconnected.");

            // As part of the sample, tell the user what happened.
            Toast.makeText(Binding.this, R.string.remote_service_disconnected,
                    Toast.LENGTH_SHORT).show();
        }
    };

    /**
     * Class for interacting with the secondary interface of the service.
     */
    private ServiceConnection mSecondaryConnection = new ServiceConnection() {
        public void onServiceConnected(ComponentName className,
                IBinder service) {
            // Connecting to a secondary interface is the same as any
            // other interface.
            mSecondaryService = ISecondary.Stub.asInterface(service);
            mKillButton.setEnabled(true);
        }

        public void onServiceDisconnected(ComponentName className) {
            mSecondaryService = null;
            mKillButton.setEnabled(false);
        }
    };

    private OnClickListener mBindListener = new OnClickListener() {
        public void onClick(View v) {
            // Establish a couple connections with the service, binding
            // by interface names.  This allows other applications to be
            // installed that replace the remote service by implementing
            // the same interface.
            Intent intent = new Intent(Binding.this, RemoteService.class);
            intent.setAction(IRemoteService.class.getName());
            bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
            intent.setAction(ISecondary.class.getName());
            bindService(intent, mSecondaryConnection, Context.BIND_AUTO_CREATE);
            mIsBound = true;
            mCallbackText.setText("Binding.");
        }
    };

    private OnClickListener mUnbindListener = new OnClickListener() {
        public void onClick(View v) {
            if (mIsBound) {
                // If we have received the service, and hence registered with
                // it, then now is the time to unregister.
                if (mService != null) {
                    try {
                        mService.unregisterCallback(mCallback);
                    } catch (RemoteException e) {
                        // There is nothing special we need to do if the service
                        // has crashed.
                    }
                }

                // Detach our existing connection.
                unbindService(mConnection);
                unbindService(mSecondaryConnection);
                mKillButton.setEnabled(false);
                mIsBound = false;
                mCallbackText.setText("Unbinding.");
            }
        }
    };

    private OnClickListener mKillListener = new OnClickListener() {
        public void onClick(View v) {
            // To kill the process hosting our service, we need to know its
            // PID.  Conveniently our service has a call that will return
            // to us that information.
            if (mSecondaryService != null) {
                try {
                    int pid = mSecondaryService.getPid();
                    // Note that, though this API allows us to request to
                    // kill any process based on its PID, the kernel will
                    // still impose standard restrictions on which PIDs you
                    // are actually able to kill.  Typically this means only
                    // the process running your application and any additional
                    // processes created by that app as shown here; packages
                    // sharing a common UID will also be able to kill each
                    // other's processes.
                    Process.killProcess(pid);
                    mCallbackText.setText("Killed service process.");
                } catch (RemoteException ex) {
                    // Recover gracefully from the process hosting the
                    // server dying.
                    // Just for purposes of the sample, put up a notification.
                    Toast.makeText(Binding.this,
                            R.string.remote_call_failed,
                            Toast.LENGTH_SHORT).show();
                }
            }
        }
    };

    // ----------------------------------------------------------------------
    // Code showing how to deal with callbacks.
    // ----------------------------------------------------------------------

    /**
     * This implementation is used to receive callbacks from the remote
     * service.
     */
    private IRemoteServiceCallback mCallback = new IRemoteServiceCallback.Stub() {
        /**
         * This is called by the remote service regularly to tell us about
         * new values.  Note that IPC calls are dispatched through a thread
         * pool running in each process, so the code executing here will
         * NOT be running in our main thread like most other things -- so,
         * to update the UI, we need to use a Handler to hop over there.
         */
        public void valueChanged(int value) {
            mHandler.sendMessage(mHandler.obtainMessage(BUMP_MSG, value, 0));
        }
    };

    private static final int BUMP_MSG = 1;

    private Handler mHandler = new Handler() {
        @Override public void handleMessage(Message msg) {
            switch (msg.what) {
                case BUMP_MSG:
                    mCallbackText.setText("Received from service: " + msg.arg1);
                    break;
                default:
                    super.handleMessage(msg);
            }
        }

    };
}
```