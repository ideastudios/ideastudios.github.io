---
layout:     post
title:      "Android Handler机制"
subtitle:   "安卓中Handler的机制"
#iframe:     "http://huangxuan.me/pwa-in-my-pov/"
date:       2016-08-25
author:     "ideastudio"
tags:
    - Hanlder
    - Looper
    - Message
---




# Android的消息处理有三个核心类：Looper,Handler和Message

## 1.  消息类 Message

android.os.Message的主要功能是进行消息的封装，同时可以指定消息的操作形式，Message类定义的变量和常用方法如下:  
* public int what：变量，用于定义此Message属于何种操作  
* public Object obj：变量，用于定义此Message传递的信息数据，通过它传递信息  
* public int arg1：变量，传递一些整型数据时使用  
* public int arg2：变量，传递一些整型数据时使用  
* public Handler getTarget()：普通方法，取得操作此消息的Handler对象  


在整个消息处理机制中，message又叫task，封装了任务携带的信息和处理该任务的handler,虽然Handler有默认的构造方法，但首选的方法是通过Message.obtain()获取Message对象，这样在大多数情况下，避免了创建新对象而分配内存

## 2. 消息通道：Looper

在使用Handler处理Message时，需要Looper（通道）来完成。在应用启动的过程中，系统会自动在主线程调用Looper.prepareMainLooper()产生一个Looper对象，而在其他线程中如果要使用则必须调用Looper.prepare()方法，去实例化一个Looper，一个线程有且只能有一个Looper对象，在线程中传使用Looper的典型实例如一下：

    class LooperThread extends Thread {
        public Handler mHandler;

        public void run() {
            Looper.prepare();//在当前线程中初始化一个Looper对象

            mHandler = new Handler() {//初始化一个Handler
                public void handleMessage(Message msg) {
                    // process incoming messages here
                    //收到消息后应做的处理
                }
            };
            //这里创建Message 然后发送消息
            Looper.loop();
            //一定要到最后才是调用Looper.loop() 因为里面有一个for(;;)循环 从MessageQueue中获取Message对象，然后再分发 交给Handler处理
        }
    }

Lopper的prepare()方法代码

    public static void prepare() {
        prepare(true);
    }

    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
          //一个线程仅能创建一个Looper对象
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }


Lopper的构造方法 在prepare()方法中有调用


    private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);//创建了一个MessageQueue实例用来存放Message
        mThread = Thread.currentThread();//获取当前线程的引用
    }

Lopper的loop()方法,调用该方法后，Looper正式开始工作，不断从MessageQueue中取出队头的消息执行

    public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        for (;;) {
          //死循环 所以调用loop()方法后，之后的代码都不会执行
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }

            msg.target.dispatchMessage(msg);
            //消息的分发 msg.target其实就是发送消息的Handler对象

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycleUnchecked();
        }
    }

## 3. 消息操作类：Handler

Message封装了所有的消息，而这些消息的操作需要android.os.Handler类完成，Handler构造方法以及重要成员变量如下:

    public class Handler {
    final MessageQueue mQueue;
    final Looper mLooper;//关联的Looper
    final Callback mCallback;
    public Handler(Callback callback, boolean async) {
        if (FIND_POTENTIAL_LEAKS) {
            final Class<? extends Handler> klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                    (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }

        mLooper = Looper.myLooper();获取该线程关联的Looper对象
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
                //如果在创建handler之前没有调用Looper.prepare()则抛出异常
        }
        mQueue = mLooper.mQueue; // 将Looer初始化时创建的MessageQueue 关联起来
        mCallback = callback;
        mAsynchronous = async;
    }
    }

Looper.loop()方法中，循环取出MessageQueue中的消息，并进行分发，而Message相对应的dispatchMessage方法为

    public void dispatchMessage(Message msg) {
        if (msg.callback != null) {
        //msg.callback 是一个Runnable对象 由postRunnable()传入，并将其包装为一条Message
            handleCallback(msg);
            // 上一条语句的实现为调用Runnable的run()方法 message.callback.run();
        } else {
          //直接调用handler的handleMessage()方法
            if (mCallback != null) {
                //如果创建Handler时有传入callback 则调用callback的handleCallback()方法，这种情况下不需要自己新建一个类继承Handler
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }

