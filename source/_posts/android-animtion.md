---
title: android动画
date: 2016-04-14 21:45:03
update: 2016-06-02 20:44:09
tags: Android
comments: false
---

关于android的动画效果一共有三种
> + View Animtion 即 tweened animation 补间动画
> + Drawable Animation 即 frame-by-frame animation 帧动画
> + Property Animation 属性动画

  <!--more-->

 关于这几种动画一直没有搞清楚，最近折腾好久才发现自己搞错了

 前两种动画效果，网上的教程很多，只放一篇吧
 <http://www.cnblogs.com/feisky/archive/2010/01/11/1644482.html>

 推荐一个Android官网最新的文档的网站，面翻墙，国内人做的镜像
 <http://www.loverobots.cn/android-api/reference/android/view/animation/package-summary.html>

Property Animation的动画找到了一篇很详细的介绍的
> * [Android属性动画完全解析(上)，初识属性动画的基本用法](http://blog.csdn.net/guolin_blog/article/details/43536355)
* [Android属性动画完全解析(中)，ValueAnimator和ObjectAnimator的高级用法](http://blog.csdn.net/guolin_blog/article/details/43816093)
* [Android属性动画完全解析(下)，Interpolator和ViewPropertyAnimator的用法](http://blog.csdn.net/guolin_blog/article/details/44171115)

特此记录下来，免得再犯错

附上`ObjectAnimator.ofFloat(Object target, String propertyName, float... values);`中第二个参数[**propertyName**](http://developer.android.com/guide/topics/graphics/prop-animation.html)的值

> * ***translationX*** and ***translationY***  :
These properties control where the View is located as a delta from its left and top coordinates which are set by its layout container.
* ***rotation***, ***rotationX***, and ***rotationY***  :
These properties control the rotation in 2D (rotation property) and 3D around the pivot point.
* ***scaleX*** and ***scaleY***  :
These properties control the 2D scaling of a View around its pivot point.
* ***pivotX*** and ***pivotY***  :
These properties control the location of the pivot point, around which the rotation and scaling transforms occur. By default, the pivot point is located at the center of the object.
* ***x*** and ***y***  :
These are simple utility properties to describe the final location of the View in its container, as a sum of the left and top values and translationX and translationY values.
* ***alpha***  :
Represents the alpha transparency on the View. This value is 1 (opaque) by default, with a value of 0 representing full transparency (not visible).

##update:
 关于上面那个值，最近发现自己错了，一直懒得改这个地方。其实那个只要你在``target`` 里面你定义一个``setXxYy(float f)`` 这个 ``xxYy`` 就可以用在 ``propertyName`` 。很简单的，还支持：
 -  **public static ObjectAnimator ofArgb (Object target, String propertyName, int... values)**
 -  **public static ObjectAnimator ofInt (Object target, String propertyName, int... values)**
 -  **public static ObjectAnimator ofMultiFloat (Object target, String propertyName, float[][] values)**

还有一个是``android.animation.ValueAnimator``  ，其实``ObjectAnimator`` 也是继承自``ValueAnimator`` 的。
- **public static ValueAnimator ofArgb (int... values)**
-  **public static ValueAnimator ofFloat (float... values)**
-  **public static ValueAnimator ofInt (int... values)**

以上还有很多方法都是提供，值的变化的，感兴趣的可以去查看文档。
