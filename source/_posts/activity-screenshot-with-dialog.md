---
title: 当前activity截图以及包含打开的dialog
date: 2016-08-28 14:08:51
tags: Android
comments: false
---
当前activity截图以及包含打开的dialog，没有找到官方提供的截图方法，当前方法是读取要截图的view的缓存然后绘制称为图片，要保存打开的dialog，如果需要背景，那就把背景绘制到一起，就可以了。
<!-- more -->
```java
		View mainView = getWindow().getDecorView();
		//也可以用下面的，效果等同于上面的效果
    //mainView= findViewById(android.R.id.content);
    mainView.setDrawingCacheEnabled(true);
    mainView.buildDrawingCache();
    //没找到可以截取状态栏的办法，如果有人知道，可以告知一下本人。上面获取到的bitmap是包含顶部状态栏的，所以需要去掉。
    int statusBarHeight = AppUtil.getStatusBarHeight(context);
    //virtualHight是底部虚拟按键的高度，如何获取，参见我的上一篇博客，这里不进行赘述了。
    Bitmap bitmap = Bitmap.createBitmap(mainView.getDrawingCache(),0,statusBarHeight, mainView.getWidth(),mainView.getHeight()－statusBarHeight-virtualHight);
    //如果需要同时保存打开的dialog的截图，可以这么做，如果不需要，上面的bitmap就是当前activity的截图了。
    View dialogView = dialog.getWindow().getDecorView();
    int location[] = new int[2];
    mainView.getLocationOnScreen(location);
    int location2[] = new int[2];
    dialogView.getLocationOnScreen(location2);
		dialogView.setDrawingCacheEnabled(true);
    dialogView.buildDrawingCache();
    Bitmap bitmap2 = Bitmap.createBitmap(dialogView.getDrawingCache(), 0, 0, dialogView.getWidth(), dialogView.getHeight());
    Canvas canvas = new Canvas(bitmap);
    canvas.drawBitmap(bitmap2, location2[0] - location[0], location2[1] - location[1], new Paint());
    mainView.destroyDrawingCache();
    dialogView.destroyDrawingCache();
```
下面是保存bitmap的方法
```java
		public void saveBitmap(Bitmap bitmap,String fileName){
		File path = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES);
	    File imageFile = new File(path, fileName+".png");
	    FileOutputStream fileOutPutStream = null;
	    try {
			fileOutPutStream = new FileOutputStream(imageFile);
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		}
		//第一个参数是图片的类型，有三种，第二个参数是保存的图片的质量。0～100
		bitmap.compress(Bitmap.CompressFormat.PNG, 100, fileOutPutStream);
		try {
			fileOutPutStream.flush();
			fileOutPutStream.close();
		} catch (IOException e) {
			e.printStackTrace();
		}
		//这是保存好的图片的绝对地址
        //Uri uri = Uri.parse("file://" + imageFile.getAbsolutePath());
	}
```
