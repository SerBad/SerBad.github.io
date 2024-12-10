---
title: Messenger传输大的数据
tags: Android
comments: false
categories: Android
date: 2021-10-09 14:59:47
---
``Messenger``作为跨进程，是很常用的方法，轻便，已经基于AIDL做了很多的封装了，但是这个方法只能传输比较小的数据，如果要传输大一些的数据咋办呢？可以使用``Bundle.putBinder ``，我这里做个记录：
<!--more-->
首先创建一个aidl，``GetLargeOne.aidl``
```aidl
// GetLargeOne.aidl

// Declare any non-default types here with import statements

//为了解决传输数据量很大的时候处理的情况
interface GetLargeOne {
    /**
     * Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
    String getOne();
}
```
在发送端的使用方式：
```kotlin
        val data = Bundle()
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR2) {
            data.putBinder("one", object : GetLargeOne.Stub() {
                override fun getOne(): String {
                    return "要传送的数据"
                }
            })
        }
```
在接收端的使用方式：
```kotlin
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR2) {
           val string =  GetLargeOne.Stub.asInterface(data?.getBinder("one")).one
            //string，就是上面传送过来的数据
        }
```
注意，如果是跨进程，必须使用的是``GetLargeOne.Stub.asInterface``来处理，否者会报错。

因为aidl不支持泛型，所以需要什么类型，你就给自己定义一个什么类型就好了。以上代码的重点，其实就是``GetLargeOne.Stub.asInterface``，我也是试了一段时间，才幡然醒悟的，因为很多资料，都是直接奔着使用aidl的方式去传输去了，如果是使用``Messenger``在这里做的绑定，其实也可以这样写的。

以上的发送端，和接收端，并不局限于你在哪个进程，只在于，你是谁发送，谁接收。
 
参考：https://developer.android.com/guide/components/aidl?hl=zh-cn#Implement

