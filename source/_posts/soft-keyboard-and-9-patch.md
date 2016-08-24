---
title: 获取软键盘高度和生成9-patchh
date: 2016-08-24 23:30:49
tags: Android
comments: true
---

最近在做一个类似聊天的功能，所以需要获取到软键盘的高度来控制界面的显示，同时有一些手机上面有虚拟按键，在界面布局的时候需要注意。下面的代码就是一段获取手机软键盘高度的办法。
<!-- more -->
```java
	Rect r = new Rect();
	getWindow().getDecorView().getWindowVisibleDisplayFrame(r);
	DisplayMetrics metrics = new DisplayMetrics();
	getWindowManager().getDefaultDisplay().getMetrics(metrics);
	//屏幕的高度，不包含虚拟按键
	int screenHeight = metrics.heightPixels;
	//屏幕真实的高度，包含虚拟按键
	getWindowManager().getDefaultDisplay().getRealMetrics(metrics);
	//虚拟按键的高度
	int virtualHight = metrics.heightPixels - screenHeight;
	int statusBarHeight=getStatusBarHeight(context);
	// 在不显示软键盘时，heightDiff等于状态栏的高度
	// 在显示软键盘时，heightDiff会变大，等于软键盘加状态栏的高度。
	// 所以heightDiff大于状态栏高度时表示软键盘出现了，
	// 这时可算出软键盘的高度，即heightDiff减去状态栏的高度
	if (heightDiff > statusBarHeight) {
		keyboardHeight = heightDiff - statusBarHeight;
	}

	if (isShowKeyboard) {
		// 如果软键盘是弹出的状态，并且heightDiff小于等于状态栏高度，
		if (heightDiff <= statusBarHeight) {
			// 说明这时软键盘已经收起
			isShowKeyboard=false;
		}
	} else {
        // 如果软键盘是收起的状态，并且heightDiff大于状态栏高度，
        if (heightDiff > statusBarHeight) {
         // 说明这时软键盘已经弹出
         isShowKeyboard=true;
		}
	}
```
下面是获取手机状态栏高度的办法
```java
	// 获取手机状态栏高度
    public static int getStatusBarHeight(Context context) {
        Class<?> c = null;
        Object obj = null;
        Field field = null;
        int x = 0, statusBarHeight = 0;
        try {
            c = Class.forName("com.android.internal.R$dimen");
            obj = c.newInstance();
            field = c.getField("status_bar_height");
            x=Integer.parseInt(field.get(obj).toString());
            statusBarHeight = context.getResources().getDimensionPixelSize(x);
        } catch (Exception e1) {
            e1.printStackTrace();
        }
        return statusBarHeight;
    }
```
对于聊天框，一开始想的是自定义控件的办法，后来在网上找到了**9-patch**，可以自动拉伸图片，需要用到的是**.png**的图片，可以找美工切好图，在Android studio上，右键选择图片，可以自动生成**xx.9.png**的文件，如下图：
![xx.9.png](http://img.blog.csdn.net/20160824174835941)

在```sdk\tools\``` 下面有**draw9patch** 命令，直接打开拉入图片也能生成**9-Patch**，如下图：
![draw9patch](http://img.blog.csdn.net/20160824204210485)

下面就是最终效果了，需要注意的是，生成的**xx.9.png**文件需保存在 **drawable**的目录下面，然后就可以直接调用了。
在工具上，有边线可以直接拖动，多试验几次找到最合适自己的，如图所示黑色阴影是要显示的区域的边距，两条绿色区域的交叉处就是最后会被拉伸的地方，右边就是最后被拉伸的效果图。
![效果](http://img.blog.csdn.net/20160824174900410)
