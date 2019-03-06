---
title: 关于SharedPreferences的一次问题
date: 2016-06-02 20:50:22
tags: Android
comments: false
---
SharedPreferences可以用来保存一些很简单的数据，对应的就是一个key-value。但是最近遇到一个问题就是在多进程的时候，没有办法同步保存数据。看到一个解决办法是``Context.MODE_MULTI_PROCESS`` 来进行多线程访问。在官网在介绍这个方法的时候是这样子的：
<!--more-->
```
This constant was deprecated in API level 23.
MODE_MULTI_PROCESS does not work reliably in some versions of Android, and furthermore does not provide any mechanism for reconciling concurrent modifications across processes.
Applications should not attempt to use it.
Instead, they should use an explicit cross-process data management approach such as ContentProvider.

SharedPreference loading flag: when set, the file on disk will be checked for modification even if the shared preferences instance is already loaded in this process.
This behavior is sometimes desired in cases where the application has multiple processes, all writing to the same SharedPreferences file.
Generally there are better forms of communication between processes, though.

This was the legacy (but undocumented) behavior in and before Gingerbread (Android 2.3) and this flag is implied when targetting such releases.
For applications targetting SDK versions greater than Android 2.3, this flag must be explicitly set if desired.
```

在6.0以上这个方法已经被弃用了，最低支持2.3。而且最好使用``ContentProvider``。
一定要用的话，要尽力避免同时去操作SharedPreferencesde。其实也找到一些开源的库，可是觉得没有必要，所以还是算了。

放一些链接：
 -  <https://github.com/android-cn/android-discuss/issues/135>
 -  <http://zmywly8866.github.io/2015/09/09/sharedpreferences-in-multiprocess.html>
