---
layout: post
title: "Android Toast 原理分析"
subtitle: "Android Toast Principle Analysis"
author: "beforenight"
header-img: "img/post-bg-net.jpg"
header-mask: 0.4
tags:
  - Android
  - Toast
  - Analysis
---

Android Toast 是我们日常开发中常用的View组件，下面分析一下Toast是如何运作的，开始之前先要知道所有的视图都是通过 WindowManager.addView(mView, mParams) 添加并显示到屏幕上的，这是一个IPC调用。

## Toast原型图
![Toast](/img/Toast原型图.png)

## Toast 的使用
```
Toast.makeText(MainActivity.this, "message", Toast.LENGTH_SHORT).show();
```
如果要在子线程中显示呢？直接使用上面的代码会报错：
> java.lang.RuntimeException: Can’t create handler inside thread that has not called Looper.prepare()

这也就意味着 Toast 必须在 Looper.prepare() 之后才能正常运行，看来 Toast 内部使用了Handler，在初始化Handler的时候没有在本线程拿到 Looper 所以会报以上错误，可以看一下 Handler 的初始化代码：
```
public Handler(Callback callback, boolean async) {
        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
}
```

那如果一定要在子线程中显示呢？
那么就在子线程中创建 Looper,再试试看：
```
new Thread() {
    @Override
    public void run() {
         super.run();
         Looper.prepare();
         Toast.makeText(MainActivity.this, "Hello World!", Toast.LENGTH_SHORT).show();
         Looper.loop();
    }

}.start();
```
正常运行。
然后，在这里抛出几个问题：
- 这里为什么一定要加上 Looper 呢？
- Toast 是如何一步步添加到屏幕上的？
- 同时显示多个Toast为什么会依次显示？
- 如果在循环体里显示100个Toast会发生什么？
下面原理篇将会一一解释。

## Toast 原理分析

## Toast 构建方法
```
public static Toast makeText(Context context, CharSequence text, @Duration int duration) {
    Toast result = new Toast(context);
    LayoutInflater inflate = (LayoutInflater)
    context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
    View v = inflate.inflate(com.android.internal.R.layout.transient_notification, null);
    TextView tv = (TextView)v.findViewById(com.android.internal.R.id.message);
    tv.setText(text);
    result.mNextView = v;
    result.mDuration = duration;
    return result;
}
```
设置了默认的布局，设置了布局里的文字，设置了显示时间，主要看 Toast 的构造方法。

Toast 构造方法
```
public Toast(Context context) {
   mContext = context;
   mTN = new TN();
   mTN.mY = context.getResources().getDimensionPixelSize(
                com.android.internal.R.dimen.toast_y_offset);
   mTN.mGravity = context.getResources().getInteger(
                com.android.internal.R.integer.config_toastDefaultGravity);
}
```

值得注意的是 **mTN = new TN()** ，事实上这个 TN 对象是一个 Binder 回调，当然本身也是Binder 对象，是提供给 NotificationManagerService 回调用的，实现的接口非常简单，对应了显示和隐藏 Toast ：
```
void hide() throws RemoteException;
void show() throws RemoteException;
```

大致可以猜一下 Toast 的显示流程，我们调用 Toast 的 show 方法，Toast 类内部通过NotificationManager 告诉系统我要显示这个 Toast，并且传递了自己的信息以及 mTN 对象， NotificationManagerService 内部处理完成后远程调用mTN对象的 show 方法，这样就回到了应用进程，然后通过 WindowManager的addView 方法显示出该 Toast。并且 NotificationManagerService 在显示回调的时候会发一个时长为 duration 的延时消息，在该消息到底之后回调mTN的hide方法隐藏 Toast。这里有两个远程调用，一个是应用调用 NotificationManager，一个是 NotificationManagerService 回调 NT 对象，前者系统是服务端，后者应用为服务端。

我们来根据源码验证一下：
前面看到，初始化Toast的时候初始化了TN对象，简单的看一下TN对象：

```
private static class TN extends ITransientNotification.Stub {
        final Runnable mShow = new Runnable() {
            @Override
            public void run() {
                handleShow();
            }
        };

        final Runnable mHide = new Runnable() {
            @Override
            public void run() {
                handleHide();
                // Don't do this in handleHide() because it is also invoked by handleShow()
                mNextView = null;
            }
        };

        private final WindowManager.LayoutParams mParams = new WindowManager.LayoutParams();
        final Handler mHandler = new Handler();    

        int mGravity;
        int mX, mY;
        float mHorizontalMargin;
        float mVerticalMargin;

        View mView;
        View mNextView;

        WindowManager mWM;

        TN() {
            // XXX This should be changed to use a Dialog, with a Theme.Toast
            // defined that sets up the layout params appropriately.
            final WindowManager.LayoutParams params = mParams;
            params.height = WindowManager.LayoutParams.WRAP_CONTENT;
            params.width = WindowManager.LayoutParams.WRAP_CONTENT;
            params.format = PixelFormat.TRANSLUCENT;
            params.windowAnimations = com.android.internal.R.style.Animation_Toast;
            params.type = WindowManager.LayoutParams.TYPE_TOAST;
            params.setTitle("Toast");
            params.flags = WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON
                    | WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE
                    | WindowManager.LayoutParams.FLAG_NOT_TOUCHABLE;
        }

        /**
         * schedule handleShow into the right thread
         */
        @Override
        public void show() {
            if (localLOGV) Log.v(TAG, "SHOW: " + this);
            mHandler.post(mShow);
        }

        /**
         * schedule handleHide into the right thread
         */
        @Override
        public void hide() {
            if (localLOGV) Log.v(TAG, "HIDE: " + this);
            mHandler.post(mHide);
        }

        public void handleShow() {
	    //略
        }

        private void trySendAccessibilityEvent() {
        //略
        }        

        public void handleHide() {
        //略
        }
}
```

可以看到构造方法里初始化了WindowManager.LayoutParams，用于后面WindowManager的add方法显示View。成员变量里初始化了Handler，这也就解释了，为什么在子线程中初始化Toast会出错。那么这里的show方法为什么不直接显示呢，而是通过Handler转发到创建Toast的线程再显示？原因在于TN是一个Binder类，并且是服务端，所以运行在Binder线程池中，所以用Handler转发到创建Toast的线程。

现在看回到Toast的show方法：
```
public void show() {
    if (mNextView == null) {
        throw new RuntimeException("setView must have been called");
    }

    INotificationManager service = getService();
    String pkg = mContext.getOpPackageName();
    TN tn = mTN;
    tn.mNextView = mNextView;

    try {
        service.enqueueToast(pkg, tn, mDuration);
    } catch (RemoteException e) {
      // Empty
    }
}
```

如上所述，并没有直接显示，而是调用了NotificationManager的enqueueToast，并且带上了包名，tn对象，显示时长。从名字可以看出这是一个入队列的操作，也就解释了同时调用多个Toast的时候会依次显示。
那么接下来看NotificationManagerService的enqueueToast方法：

```
public void enqueueToast(String pkg, ITransientNotification callback, int duration){
            //省略部分无关代码

            synchronized (mToastQueue) {
                int callingPid = Binder.getCallingPid();
                long callingId = Binder.clearCallingIdentity();
                try {
                    ToastRecord record;
                    int index = indexOfToastLocked(pkg, callback);
                    // If it's already in the queue, we update it in place, we don't
                    // move it to the end of the queue.
                    if (index >= 0) {
                        record = mToastQueue.get(index);
                        record.update(duration);
                    } else {
                        // Limit the number of toasts that any given package except the android
                        // package can enqueue.  Prevents DOS attacks and deals with leaks.
                        if (!isSystemToast) {
                            int count = 0;
                            final int N = mToastQueue.size();
                            for (int i=0; i= MAX_PACKAGE_NOTIFICATIONS) {
                                         Slog.e(TAG, "Package has already posted " + count
                                                + " toasts. Not showing more. Package=" + pkg);
                                         return;
                                     }
                                 }
                            }
                        }

                        record = new ToastRecord(callingPid, pkg, callback, duration);
                        mToastQueue.add(record);
                        index = mToastQueue.size() - 1;
                        keepProcessAliveLocked(callingPid);
                    }
                    // If it's at index 0, it's the current toast.  It doesn't matter if it's
                    // new or just been updated.  Call back and tell it to show itself.
                    // If the callback fails, this will remove it from the list, so don't
                    // assume that it's valid after this.
                    if (index == 0) {
                        showNextToastLocked();
                    }
                } finally {
                    Binder.restoreCallingIdentity(callingId);
                }
            }
}
```

这里的mToastQueue是一个ArrayList，用于保存所有的ToastRecord，其中ToastRecord保存了callingPid（应用的PID），pkg（包名），callback(TN对象)，duration（显示时长）
大致流程如下：
1. 判断当前Toast是否已经存在，如果已经存在就更新显示时长。
2. 如果不存在，遍历mToastQueue，统计相同包名的ToastRecord数量，如果大于MAX_PACKAGE_NOTIFICATIONS（50）则不继续添加。如注释所述防止DOS攻击。最后添加进mToastQueue
3. 如果当前是第一个，调用showNextToastLocked()开始依次显示mToastQueue中的Toast

继续看showNextToastLocked方法：
```
void showNextToastLocked() {
        ToastRecord record = mToastQueue.get(0);
        while (record != null) {
            if (DBG) Slog.d(TAG, "Show pkg=" + record.pkg + " callback=" + record.callback);
            try {
                record.callback.show();
                scheduleTimeoutLocked(record);
                return;
            } catch (RemoteException e) {
                Slog.w(TAG, "Object died trying to show notification " + record.callback
                        + " in package " + record.pkg);
                // remove it from the list and let the process die
                int index = mToastQueue.indexOf(record);
                if (index >= 0) {
                    mToastQueue.remove(index);
                }
                keepProcessAliveLocked(record.pid);
                if (mToastQueue.size() > 0) {
                    record = mToastQueue.get(0);
                } else {
                    record = null;
                }
            }
        }
}
```

主要就是循环取出mToastQueue中的ToastRecord 元素，并且调用record.callback.show();也就是回调了TN对象的show方法回答应用进程。可以看到接下来还调用了scheduleTimeoutLocked(record);

```
private void scheduleTimeoutLocked(ToastRecord r){
        mHandler.removeCallbacksAndMessages(r);
        Message m = Message.obtain(mHandler, MESSAGE_TIMEOUT, r);
        long delay = r.duration == Toast.LENGTH_LONG ? LONG_DELAY : SHORT_DELAY;
        mHandler.sendMessageDelayed(m, delay);
}
```

根据duration 发送了一次延时消息，在消息事件里调用record.callback.hide();隐藏Toast。
不管是显示还是隐藏，最终都回调到了应用进程的TN对象中，那么现在回看TN对象的show方法，上面说了，TN对象的show方法会通过Handler调用handleShow方法：

```
public void handleShow() {
            if (localLOGV) Log.v(TAG, "HANDLE SHOW: " + this + " mView=" + mView
                    + " mNextView=" + mNextView);
            if (mView != mNextView) {
                // remove the old view if necessary
                handleHide();
                mView = mNextView;
                Context context = mView.getContext().getApplicationContext();
                String packageName = mView.getContext().getOpPackageName();
                if (context == null) {
                    context = mView.getContext();
                }
                mWM = (WindowManager)context.getSystemService(Context.WINDOW_SERVICE);
                // We can resolve the Gravity here by using the Locale for getting
                // the layout direction
                final Configuration config = mView.getContext().getResources().getConfiguration();
                final int gravity = Gravity.getAbsoluteGravity(mGravity, config.getLayoutDirection());
                mParams.gravity = gravity;
                if ((gravity & Gravity.HORIZONTAL_GRAVITY_MASK) == Gravity.FILL_HORIZONTAL) {
                    mParams.horizontalWeight = 1.0f;
                }
                if ((gravity & Gravity.VERTICAL_GRAVITY_MASK) == Gravity.FILL_VERTICAL) {
                    mParams.verticalWeight = 1.0f;
                }
                mParams.x = mX;
                mParams.y = mY;
                mParams.verticalMargin = mVerticalMargin;
                mParams.horizontalMargin = mHorizontalMargin;
                mParams.packageName = packageName;
                if (mView.getParent() != null) {
                    if (localLOGV) Log.v(TAG, "REMOVE! " + mView + " in " + this);
                    mWM.removeView(mView);
                }
                if (localLOGV) Log.v(TAG, "ADD! " + mView + " in " + this);
                mWM.addView(mView, mParams);
                trySendAccessibilityEvent();
            }
        }
```

前面都是在配置显示参数WindowManager.LayoutParams,
最后显示该Toast对应的View：mWM.addView(mView, mParams);绕了一大圈最终显示出 Toast了。

## 总结
最后，再来看看 Toast 设计原型图

![Toast](/img/Toast原型图.png)
