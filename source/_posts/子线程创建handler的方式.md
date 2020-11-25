---
title: 子线程创建handler的方式
tags: Android
comments: false
date: 2020-11-25 18:07:45
---
子线程创建handler的方式，在这里做个记录
<!--more-->
```
Thread(object : Runnable {
    override fun run() {
        Looper.prepare()
        //Looper.myLooper()这里也可以不写，Handler会自动获取当前线程的looper
        //也可以用Looper.getMainLooper()获取主线程的looper
        //Looper.getMainLooper()在整个程序进程中是单例的
        //进程在ActivityThread中调用Looper.prepareMainLooper()初始化Looper.getMainLooper()
        //通过设置AndroidManifest.xml中的process，可以让产生多个进程，让Looper.getMainLooper()不一样
        val handler = Handler(Looper.myLooper())
        handler.post {
            L.i("ss ${Looper.myLooper()?.toString()}")
        }
        Looper.loop()
    }
}).start()
```
