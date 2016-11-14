---
title: 关于错误INSTALL_FAILED_NO_MATCHING_ABIS
date: 2016-06-02 20:44:09
tags: android
comments: true
---
故事的起因就是因为在往genymotion安装的应用的时候，出现了`INSTALL_FAILED_NO_MATCHING_ABIS`。
![错误](http://img.blog.csdn.net/20160428170835800)

<!--more-->

因为android平台的多样性，针对不同的CPU架构于是就有了[**ABI**](http://developer.android.com/ndk/guides/abis.html)（Application Binary Interface,）。

目前android支持的ABI有：`armeabi`、`armeabi-v7a`、`arm64-v8a`、`x86`、`x86_64`、`mips`、`mips64`。

在打包的时候可以使用[`Splits`](http://tools.android.com/tech-docs/new-build-system/user-guide/apk-splits)生成不同架构的apk。在`build.gradle`配置的方法就是：
```gradle
android {
  ...
  splits {
    abi {
      enable true
      reset()
      include 'armeabi','armeabi-v7a','arm64-v8a','x86','x86_64','mips','mips64'
      universalApk true
    }
  }
}
```
效果就是：
![这里写图片描述](http://img.blog.csdn.net/20160428170922441)

通过这种方法就可以针对不同的CPU架构来分开打包了。

使用[`Splits`](http://tools.android.com/tech-docs/new-build-system/user-guide/apk-splits) 还可以针对不同的分辨率来打包：
```gradle
android {
  ...
  splits {
    density {
      enable true
      exclude "ldpi", "tvdpi", "xxxhdpi"
      compatibleScreens 'small', 'normal', 'large', 'xlarge'
    }
  }
```

因为都是一些学习总结的文章，还是贴一篇其他人写的不错的文章来参考：
1. <http://blog.chengyunfeng.com/?p=889&utm_source=tuicool&utm_medium=referral>
2.  <http://www.jianshu.com/p/cb05698a1968>
