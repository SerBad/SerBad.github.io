---
title: Android共享元素
date: 2018-06-22 14:10:28
tags: Android
comments: false
---
在这里记录一下android共享元素的方法，踩了满多坑的，其实蛮简单的，Android共享是从Android5.0开始提供的，之前的版本我没有去碰兼容，但是其实是有解决方法的。这里只是记录一下踩过的坑，不对共享元素做深入的分析。

<!--more-->
首先从页面A跳转到页面B，那么页面A只需要调用
```kotlin
ActivityCompat.startActivity(context, intent, ActivityOptionsCompat.makeSceneTransitionAnimation(context, view, "zoomImageView").toBundle())
```
需要注意的是，需要传入要共享的view和共享view的transitionName的。然后在页面B对要共享的view设置一样的transitionName就好了。

踩过坑，如果使用的是fresco的话，在页面被销毁的情况下，需要让view保持在页面上。
```kotlin
ActivityCompat.setExitSharedElementCallback(context, object : SharedElementCallback() {
                    override fun onSharedElementEnd(sharedElementNames: List<String>?,
                                                    sharedElements: MutableList<View>?, sharedElementSnapshots: MutableList<View>?) {
                        super.onSharedElementEnd(sharedElementNames, sharedElements, sharedElementSnapshots)
                        for (view in sharedElements!!) {
                            if (view is SimpleDraweeView) {
                                view.show()
                            }
                        }
                    }
                })
```
如果还是不行的话，在页面B的onCreate添加如下代码就可以了，话说fresco还是蛮反人类的，各种奇怪的问题。
```kotlin
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
         window.sharedElementEnterTransition = AutoTransition() // 进入
         window.sharedElementReturnTransition = AutoTransition()// 返回
         ActivityCompat.setEnterSharedElementCallback(this, object : SharedElementCallback() {
             override fun onSharedElementEnd(sharedElementNames: List<String>?,
                                             sharedElements: List<View>?, sharedElementSnapshots: List<View>?) {
                 for (view in sharedElements!!) {
                     if (view is PhotoDraweeView) {
                         (view as PhotoDraweeView).setScale(1f, true)
                     }
                 }
             }
         })
     }
```
要想在结束的时候依然能够有共享元素的动画效果，需要调用的是
``` kotlin  
finishAfterTransition()
```

如果需要在页面B退出的时候，回调结果像`startActivityForResult`一样的话，需要的在`finishAfterTransition`方法之后，去调用`setResult`，在页面B的代码如下
```kotlin
override fun finishAfterTransition() {
      val resultData = Intent()
      resultData.putExtra(CURRENT_ITEM,“要回调的值”)
      setResult(Activity.RESULT_OK, resultData)
      super.finishAfterTransition()
   }
```
在页面A接受的方法是，需要注意的是，如果是fragment的话，是没有`onActivityReenter`方法的，不过你可以选择自定义一个接口，然后在下面的代码调用它
```kotlin
override fun onActivityReenter(resultCode: Int, data: Intent) {
     super.onActivityReenter(resultCode, data)
     val list = supportFragmentManager.fragments
     list.forEach {
         if (it is MyFragment) {
             it.onReenter(resultCode, data)
         }
     }
 }

//MyFragment中定义的方法
 open fun onReenter(resultCode: Int, data: Intent) {
   }
```

差不多就是这些吧，这些就能满足基本的共享元素动画了。需要注意的是，一定要设置正确的transitionName。
贴一个我觉得做的不错的
- https://github.com/kitek/android-gallery.git
- https://github.com/ongakuer/PhotoDraweeView

对于我，因为第二页使用的是ViewPager，导致因为ViewPgaer的缓存问题每次返回的item都不对，所以我直接把transitionName设置到ViewPager上就好了，虽然这不是最佳的解决方案，但是也解决问题了。
