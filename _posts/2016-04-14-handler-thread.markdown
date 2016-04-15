---
layout: post
title:  "Handler Thread"
date:   2016-04-14 12:00:00
categories: Android
permalink: /archivers/handlerthread
comments: True
---

If you have been an Android developer for sometime, no doubt you follow the gospel - move heavy duty processing stuff to a non-ui thread. This is because if you do heavy operations such as networking, image manipulation, the UI will be frozen, possibly leading to an ANR message...argggh.
You will need to perform the heavy operations on some other thread and then communicate the result to the UI thread.

However, if you do not deal with user accounts and still want to use the Sync adapter, this post will show you how to create one. I will assume you have not 
incorporated an account authenticator. So, I will show the stub implementations

**Background processing options**

The android performance videos mention the following four options for performing background operations:

1. **Async Task** - Most famous and used by the majority. However it has it's limitations, you cannot reuse it. need to create a new one. Good for one off operations.

2. **HandlerThread** - Least used, but the topic of today's blog post. Many moving parts, but excellent choice to run repeated actions.

3. **ThreadPoolExecutor** - Plain old java, non android artifact. Best for performing a giant operation split into many multiple mini operations.

4. **Service** - Probably the second most famous background processing artifact. Main differentiator is that it could allow you to run a background operation even if the app is killed.

With that behind us or above us. let us move on to learning HandlerThread.

**Looper, Handler and Message Queue**

To explain HandlerThread, I will give a rudimentary idea about a few different concepts:

*  **Handler** - The handler object actually puts the message, runnable in the message queue.

*  **Message Queue** - A simple queue that holds message, runnables. These message,runnables are sent by the handler.

*  **Looper** - The looper loops through the message queue and processes the messages, runnables on the respective handler thread. How does it know which thread to process on - each message, runnable on generation is associated with a handler and it is this handler that puts the runnable in the message queue.

So the following block diagram explains how all these three elements work together.  Going from left to right, the handler puts the messages, runnables in the message queue

![Making of a HandlerThread](/images/handler.png)

**Create a HandlerThread**

First, you will create the class that extends the HandlerThread:

```java
private class MyWorkerThread extends HandlerThread {
        private Handler mWorkerHandler;

        public MyWorkerThread(String name){
            super(name);
        }

        public void postTask(Runnable task){
            mWorkerHandler.post(task);
        }

        public void prepareHandler(){
            mWorkerHandler = new Handler(getLooper());
        }
    }
``` 

**HandlerThread in action**

Now, imagine you have a runnable called task. Inorder to run it on the HandlerThread, create an instance of the HandlerThread first, start it like any normal thread and prepare it to so it gets a handle of the looper:

```java
mWorkerThread = new MyWorkerThread("workerThread");
mWorkerThread.start();
mWorkerThread.prepareHandler();
```

Now to run the task runnable on the HandlerThread, post it on the HandlerThread instance as follows:

```java
mWorkerThread.postTask(theTask());    
```

You can communicate back information from the HandlerThread to the main UI thread by using the runOnUiThread method.

The following is a good blog explaining in handlers in more detail:

* [https://blog.nikitaog.me/2014/10/11/android-looper-handler-handlerthread-i/](https://blog.nikitaog.me/2014/10/11/android-looper-handler-handlerthread-i/)


