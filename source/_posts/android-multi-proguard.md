---
title: Android多模块混淆的问题
tags: Android
comments: false
date: 2019-01-21 18:49:53
---
Android在多模块或者组件化的时候，关于混淆的管理，一般常见的做法就是两条。
<!-- more -->
-  把所有的混淆规则规则都放在`app`模块下面，由`app`统一管理。这样就会有一个问题，就是到会导致混淆规则的冗余。
- 由`module`管理自己的混淆规则，这样的话需要你对自己的模块有一个很好的管理。

这里就是记录下，由`module`的处理混淆的方法，[参看官方文档](https://developer.android.com/studio/projects/android-library#Considerations)。管理子module的方法，本质上就是管理aar的方法，是通用的。在module中添加：
```
    release {
        consumerProguardFiles   'proguard-rules.pro'
    }
```
这样就可以了，需要注意的是，
1. 多模块或者组件化混淆，只要app模块开了混淆，子模块无论是否打开混淆都是默认开启的。只是通过上面的方法，子模块可以自定义混淆的规则。
2. 子模块的混淆规则是无法影响app模块的的。所以建议，在子模块里尽量只放和子模块相关的混淆规则，一些公有的混淆方式请放在app或者公有的模块中。
