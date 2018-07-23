---
title: Android上的生命周期监控
tags: Android
comments: false
date: 2018-07-23 13:36:11
---
做Android开发总是避免不了要处理各种生命周期，那么有没有什么优雅的方法呢？现在我就记录几种方法。
# Lifecycle  
`Lifecycle`是Google在[Jetpack](https://developer.android.google.cn/jetpack/)中提出的，在包[`android.arch.lifecycle`](https://developer.android.com/reference/android/arch/lifecycle/package-summary.html)下面，如果使用的是suppor包中的SupportActivity或者FragmentActivity已经继承了Lifecycle。使用的方法如下：
<!-- more -->
```kotlin
class myFragmentActivity(val fragmentActivity: FragmentActivity) : LifecycleObserver {
    init {
     //在这里注册一下
      fragmentActivity.lifecycle.addObserver(this)
    }
    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    fun onResume(){
        Timber.tag("xx").i("ON_RESUME")
    }
    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    fun onPause(){
        Timber.tag("xx").i("ON_PAUSE")
    }
    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    fun onStop(){
        Timber.tag("xx").i("ON_STOP")
    }
    @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
    fun onDestroy(){
        Timber.tag("xx").i("ON_DESTROY")
    }
}
```
这样就可以很方便去管理生命周期，也可以和原来的activity的方法解耦。也可以通过LifecycleOwner来管理生命周期
```kotlin
class MyActivity : Activity(), LifecycleOwner {
    lateinit var mLifecycleRegistry: LifecycleRegistry

    override fun onCreate(savedInstanceState: Bundle) {
        super.onCreate(savedInstanceState)

        mLifecycleRegistry = LifecycleRegistry(this)
        mLifecycleRegistry.markState(Lifecycle.State.CREATED)
    }

    override fun onStart() {
        super.onStart()
        mLifecycleRegistry.markState(Lifecycle.State.STARTED)
    }

    //这里得到的就是生命周期的结果
    override fun getLifecycle(): Lifecycle = mLifecycleRegistry
}
```
这里不详细赘述里面的细节，只是记录一下使用的方法，有关细节可以查看官方文档。

# FragmentLifecycleCallbacks
如果使用的是fragment，那么这里提供另一种获取生命周期的方法，通过`FragmentLifecycleCallbacks`可以获取到fragment的生命周期回调
```kotlin
  fragment.requireFragmentManager().registerFragmentLifecycleCallbacks(object : FragmentManager.FragmentLifecycleCallbacks() {
    override fun onFragmentResumed(fm: FragmentManager, f: Fragment) {
      Timber.tag("xx").i("onFragmentResumed")
    }

    override fun onFragmentPaused(fm: FragmentManager, f: Fragment) {
      Timber.tag("xx").i("onFragmentPaused")
    }

    override fun onFragmentStopped(fm: FragmentManager, f: Fragment) {
      Timber.tag("xx").i("onFragmentStopped")
      //fragment.requireFragmentManager().unregisterFragmentLifecycleCallbacks(this)
    }

    override fun onFragmentDestroyed(fm: FragmentManager, f: Fragment) {
      Timber.tag("xx").i("onFragmentDestroyed")
    }
}, true)
```
所以，通过为activity添加一个空白的fragment也是一种方法，然后fragment就可以获取到activity的生命周期了，Glide就是通过添加一个fragment来管理生命周期的。

# ActivityLifecycleCallbacks
如果要全局管理生命周期有没有什么办法呢？当然还是有的，可以设置全局的Application，然后在application里面注册监听，就可以处理所有的activity的生命周期了。记得及时释放掉。
```kotlin
registerActivityLifecycleCallbacks(object : Application.ActivityLifecycleCallbacks {
    override fun onActivityPaused(activity: Activity?) {
    }

    override fun onActivityResumed(activity: Activity?) {
    }

    override fun onActivityStarted(activity: Activity?) {
    }

    override fun onActivityDestroyed(activity: Activity?) {
    }

    override fun onActivitySaveInstanceState(activity: Activity?, outState: Bundle?) {
    }

    override fun onActivityStopped(activity: Activity?) {
    }

    override fun onActivityCreated(activity: Activity?, savedInstanceState: Bundle?) {
    }
})
```

就记录这么多的方法，然后基于上面的进行封装处理就可以了，如果你有更好的方法，也欢迎来一起讨论。
