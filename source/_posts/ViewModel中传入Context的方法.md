---
title: ViewModel中传入Context的方法
tags: Android
comments: false
date: 2021-02-19 11:15:23
---

``ViewModel``使用的越来越多了，严格来说，官方并不建议你在``ViewModel``中添加``Context``的引用。同时，``ViewModel``的构造方法是没有任何参数的，有的时候会很不灵活。以下记录两种方法。
<!--more-->
#1.通过kotlin的拓展函数
```kotlin

fun <T : ViewModelProvider, V : ViewModel> T.get(
    key: String,
    modelClass: Class<V>,
    context: FragmentActivity
): V {
    val model = get(key, modelClass)
    if (model is TestViewModel) {
        model.addContext(context)
    }
    return model
}

fun <T : ViewModelProvider, V : ViewModel> T.get(
    key: String,
    modelClass: Class<V>,
    context: Context
): V {
    val model = get(key, modelClass)
    if (model is TestViewModel) {
        model.addContext(context)
    }
    return model
}

fun <T : ViewModelProvider, V : ViewModel> T.get(
    modelClass: Class<V>,
    context: FragmentActivity
): V {
    val model = get(modelClass)
    if (model is TestViewModel) {
        model.addContext(context)
    }
    return model
}

fun <T : ViewModelProvider, V : ViewModel> T.get(
    modelClass: Class<V>,
    context: Context
): V {
    val model = get(modelClass)
    if (model is TestViewModel) {
        model.addContext(context)
    }
    return model
}
```
在``TestViewModel``中添加如下的方法
```kotlin
class TestViewModel : ViewModel() {
    protected lateinit var context: Context
    open fun addContext(context: FragmentActivity) {
        this.context = context
    }

    open fun addContext(context: Context) {
        this.context = context
    }
}
```

使用方法
```kotlin
val viewModel = ViewModelProvider(this).get(TestViewModel::class.java, this)
```

#2.通过自定义ViewModelProvider.Factory

```kotlin
class CoreViewModelFactory(private val context: Context) : ViewModelProvider.Factory {
    override fun <T : ViewModel?> create(modelClass: Class<T>): T {
        try {
            for (constructor in modelClass.constructors) {
                if (arrayOf(Context::class.java).contentEquals(constructor.parameterTypes)) {
                    return (constructor as Constructor<T>).newInstance(context)
                }
            }
            return modelClass.newInstance()
        } catch (e: InstantiationException) {
            throw RuntimeException("Cannot create an instance of $modelClass", e)
        } catch (e: IllegalAccessException) {
            throw RuntimeException("Cannot create an instance of $modelClass", e)
        }
    }
}
```
关于这一块，仔细阅读ViewModelProvider的代码，会发现，里面同样提供了两三种的Factory。针对可以直接拥有``context``的``AndroidViewModel``，提供了``ViewModelProvider.AndroidViewModelFactory``，只是在引用的时候，不要再自己添加一遍了。

以下是你的``TestViewModel``
```kotlin
class TestViewModel(private val context: Context) : ViewModel() {
    init {
        L.i(" context $context ")
    }
}
```

使用方法
```kotlin
val viewModel = ViewModelProvider(this, CoreViewModelFactory(this)).get(TestViewModel::class.java)
```

以上两种方法也可以用来帮助你自定义一些你要传入的参数。
