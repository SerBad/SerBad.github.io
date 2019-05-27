---
title: Android关于裁剪图片透明区域的算法
tags: Android
comments: false
date: 2019-05-27 18:49:53
---
最近项目中遇到这么一个需求，需要裁剪掉图片的透明区域。找了很久，最后确定，只能通过自己读取Bitmap的像素点来读取图片的边界来裁剪。下面记录一下过程。
<!-- more -->
原图如下![](https://upload-images.jianshu.io/upload_images/8433-8e884a696f70b1fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# PorterDuffXfermode
最开始想的是使用PorterDuffXfermode来处理，因为这种方式其实很快的，但是，虽然这种方式可以用来处理图片，但是无法满足获取图片边界的需求。
代码如下：
```java
    public static Bitmap cutStickerBitmap(String name, Context context, Bitmap bitmap) {
        Calendar calendar = Calendar.getInstance();
        Bitmap bit = Bitmap.createBitmap(bitmap.getWidth(), bitmap.getHeight(), Bitmap.Config.ARGB_8888);
        bit.eraseColor(Color.WHITE);
        Canvas canvas = new Canvas(bit);

        Paint paint = new Paint();
        paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.DST_IN));

        canvas.drawBitmap(bitmap, 0, 0, paint);
        Paint paintRect = new Paint();
        paintRect.setColor(Color.RED);
        paintRect.setAntiAlias(true);
        paintRect.setStyle(Paint.Style.STROKE);
        paintRect.setStrokeWidth(4);
        Rect rect = new Rect();
        //画一个框表示边界
        rect.set(0, 0, bitmap.getWidth(), bitmap.getHeight());
        canvas.drawRect(rect, paintRect);
        Log.i("xx", " 运行时间 cutStickerBitmap " + (Calendar.getInstance().getTimeInMillis() - calendar.getTimeInMillis()));
        return bit;
    }
```
代码处理的时间，可以看到时间很快，可惜不符合需求：
![](https://upload-images.jianshu.io/upload_images/8433-cc2aeb6b5781d17d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
处理的结果如下： ![](https://upload-images.jianshu.io/upload_images/8433-9c08301a7e609efd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 自己写算法读像素
关于这个算法，其实，我写过两个版本。 自己写算法的话，需要自己评估你自己的需求和结果。一开始，为了防止图形是不连续的，我采取的思路是从图片四周来获取边界。思路如图：
![](https://upload-images.jianshu.io/upload_images/8433-7aa7fbf20e14fd18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这么做的确是可以保证百分百正确的，但是这么做有一个问题，那就是因为每次得到的图片分辨率不一样，在有的时候处理一次需要10s以上，这实在是太慢了。

后来仔细观察之后发现，其实不需要那么严格的，因为得到的所有的图形都是连续的，所以新的算法就是从中间开始处理的。
 ![](https://upload-images.jianshu.io/upload_images/8433-70e1bb6ae6123328.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
沿着中线，先向右边获取边界，再沿着中线从左边获取边界，同时为了加快速度，采取的一次读两个像素的方式，如果要求严谨可以改成每次读一个像素，算法代码如下。
```java
    //处理贴纸
    public static Bitmap cutStickerBitmap(Bitmap bitmap) {
        Calendar calendar = Calendar.getInstance();
        int top = 0;
        int left = 0;
        int right = bitmap.getWidth() - 1;
        int bottom = 0;

        for (int w = bitmap.getWidth() / 2; w < bitmap.getWidth(); w += 2) {
            int h = 0;
            for (h = 0; h < bitmap.getHeight(); h+=2) {
                int color = bitmap.getPixel(w, h);
                if (color != 0) {
                    if (top == 0) {
                        top = h;
                    }
                    top = Math.min(top, h);
                    break;
                }
            }

            if (h >= bitmap.getHeight() - 1) {
                right = w;
                break;
            }
        }

        for (int w = bitmap.getWidth() / 2 - 1; w >= 0; w -= 2) {
            int h = 0;
            for (h = 0; h < bitmap.getHeight(); h+=2) {
                int color = bitmap.getPixel(w, h);
                if (color != 0) {
                    if (top == 0) {
                        top = h;
                    }
                    top = Math.min(top, h);
                    break;
                }
            }

            if (h >= bitmap.getHeight() - 1) {
                left = w;
                break;
            }
        }

        for (int w = left; w <= right; w++) {
            for (int h = bitmap.getHeight() - 1; h >= top; h--) {
                int color = bitmap.getPixel(w, h);
                if (color != 0) {
                    bottom = Math.max(bottom, h);
                    break;
                }
            }
        }

        //对边界值做一次判断，保证严谨
        int x = (int) Math.max(left  , 0f);
        int y = (int) Math.max(top  , 0f);
        int w = (int) Math.min(right - left  , bitmap.getWidth() - x);
        int h = (int) Math.min(bottom - top  , bitmap.getHeight() - y);
        if (x + w > bitmap.getWidth()) {
            x = 0;
            w = bitmap.getWidth();
        }

        if (y + h > bitmap.getHeight()) {
            y = 0;
            h = bitmap.getHeight();
        }
        Log.i("xx", " 运行时间 cutStickerBitmap " + (Calendar.getInstance().getTimeInMillis() - calendar.getTimeInMillis()));

        return Bitmap.createBitmap(bitmap, x, y, w, h);
    }
```
代码处理的时间如下：![](https://upload-images.jianshu.io/upload_images/8433-537a941a7f34266c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
