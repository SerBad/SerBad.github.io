---
title: Android使用ActivityResultContract
tags: Android
comments: false
date: 2021-02-23 14:58:03
---

从`AndroidX`的[Activity `1.2.0-alpha02`](https://developer.android.google.cn/jetpack/androidx/releases/activity#1.2.0-alpha02) 和 [Fragment `1.3.0-alpha02`](https://developer.android.google.cn/jetpack/androidx/releases/fragment#1.3.0-alpha02)开始，`startActivityForResult`被标注为弃用了，因为多了一种新的办法，这里做个记录。
<!--more-->
新建一个自己的契约类`ActivityResultContract`，继承自`ActivityResultContract`，写上自己要传入的参数和要返回的结果。
```Kotlin
class PersonalActivityResultContract : ActivityResultContract<String, UserInfo?>() {
    override fun createIntent(context: Context, input: String): Intent {
        val intent = Intent(context, HeaderPreviewActivity::class.java)
        intent.putExtra("url", input)
        return intent
    }

    override fun parseResult(resultCode: Int, intent: Intent?): UserInfo? {
        if (resultCode == Activity.RESULT_OK) {
            if (intent?.hasExtra("userInfo") == true) {
                if (intent.getSerializableExtra("userInfo") != null) {
                    return intent.getSerializableExtra("userInfo") as UserInfo
                }
            }
        }
        return null
    }
}
```
然后在`fragment`或者`activity`的`onCreate`中使用`registerForActivityResult`注册一下得到`mActivityResultLauncher`，需要注意的是，`fragment`和必须在`onCreate`及以前执行，`activity`必须在`onStart`及之前执行。

```Kotlin
    private var mActivityResultLauncher: ActivityResultLauncher<String>? = null
    override fun onCreate(savedInstanceState: Bundle?) {
        mActivityResultLauncher = registerForActivityResult(PersonalActivityResultContract()) {
            if (it != null) {
                //你自己要处理的逻辑
            }
        }

        super.onCreate(savedInstanceState)
    }
```
然后在要调用的地方
```Kotlin
mActivityResultLauncher?.launch("")
```

以上这样就可以了。这样写有个好处就是更好的封装，不用再自己写一大堆的`onActivityResult`代码，也方便解耦和复用。当然这里只是省掉了原来的`startActivityForResult`和`onActivityResult`，其他的地方照旧。

以上基本够用，当然你也可以定义自己的路由，只需要实现自己的`ActivityResultRegistry`就可以了。当然，你也可以在`launch`的地方传入你自己`ActivityOptionsCompat`。

如果不需要使用到`startActivityForResult`，还是用原来的`startActivity`即可。

参考链接：https://developer.android.google.cn/training/basics/intents/result