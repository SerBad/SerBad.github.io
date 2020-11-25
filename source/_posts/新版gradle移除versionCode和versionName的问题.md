---
title: 新版gradle移除versionCode和versionName的问题
tags: Android
comments: false
date: 2020-11-25 18:09:26
---

从``com.android.tools.build:gradle:4.1.0``开始，``build.gradle``文件正式移除了``versionName``和``versionCode``，参照[链接](https://developer.android.google.cn/studio/releases/gradle-plugin#4-1-0)。

<!--more-->
如果依然需要``BuildConfig.VERSION_NAME``的话，可以使用如下方式
```gradle
buildConfigField "int", 'VERSION_CODE', String.valueOf(1)
buildConfigField 'String', 'VERSION_NAME', "\"" + "1.0.0" + "\""
```
但是仅仅这么做不太够，因为这样之后并不能让``AndroidManifest.xml``带上版本号，导致最后打的包里面是没有版本号的。所以还需要在``AndroidManifest.xml``加上版本号。
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:versionCode="1"
    android:versionName="1.0.0">

</manifest>
```
当然也可以舍弃``BuildConfig.VERSION_NAME``的方式，用以下代码来读取版本号。

```kotlin
 try {
            val manager = this.packageManager
            val info = manager.getPackageInfo(this.packageName, 0)
            info.versionName
            if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.P) {
                L.i("info.versionName ${info.versionName} ${info.longVersionCode} ")
            }else{
                L.i("info.versionName ${info.versionName} ${info.versionCode} ")
            }
        } catch (e: Exception) {
            e.printStackTrace()

        }
```
