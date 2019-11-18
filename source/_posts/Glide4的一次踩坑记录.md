---
title: Glide4的一次踩坑记录
tags: Android
comments: false
date: 2019-03-06 15:15:45
---
如果你用图片加载库直接加载图片，并不需要做任何处理，那么，其实大多数时候你用任何加载库并没有太大的区别。一旦你需要处理各种各样的图片的时候，你就会遇到各种各样的问题了。这里记录一个关于Glide的问题，一下都是Glide4.9版本。
<!--more-->
在Glide中加载图片，可以很简单，类似于这样就可以了：
```java
   Glide.with(context)
      .load(url)
      .into(imageView);
```
但是，当你在加载图片之前对图片做处理，我们可以怎么做呢？比如我这样的需求，第一次显示的时候加载左边，点击之后加载右边，显然直接用Glide加载是不行的。下面介绍一些踩过的坑。
![](/images/icon_spine_unselected_1.png)

# 1. Transformation 变换
通过内置的一些变换类，可以达到一些处理效果，[详情参考](https://muyangmin.github.io/glide-docs-cn/doc/transformations.html#%E5%86%85%E7%BD%AE%E7%B1%BB%E5%9E%8B)
```java
   Glide.with(context)
      .load(url)
      .transform(new CircleCrop())
      .into(imageView);
```
当然也可以通过自定义一些手段来达到目的，[详情参考](https://muyangmin.github.io/glide-docs-cn/doc/transformations.html#%E5%AE%9A%E5%88%B6%E5%8F%98%E6%8D%A2)
```java
import android.graphics.Bitmap;
import android.support.annotation.NonNull;

import com.bumptech.glide.load.engine.bitmap_recycle.BitmapPool;
import com.bumptech.glide.load.resource.bitmap.BitmapTransformation;

import java.security.MessageDigest;

/**
 * Created by JayChou on 2019/3/5.
 */
public class GlideSelectIcon extends BitmapTransformation {
    private static final String ID = "yourpackagename.GlideSelectIcon";
    private static final byte[] ID_BYTES = ID.getBytes(CHARSET);

    private boolean isSelect = false;

    @Override
    protected Bitmap transform(@NonNull BitmapPool pool, @NonNull Bitmap toTransform, int outWidth, int outHeight) {
        Bitmap bitmap;
        if (isSelect) {
            bitmap = Bitmap.createBitmap(toTransform, toTransform.getWidth() / 2, 0, toTransform.getWidth() / 2, toTransform.getHeight()); //对图片的高度的一半进行裁剪
        } else {
            bitmap = Bitmap.createBitmap(toTransform, 0, 0, toTransform.getWidth() / 2, toTransform.getHeight()); //对图片的高度的一半进行裁剪
        }
        pool.put(bitmap);
        return bitmap;
    }

    public boolean isSelect() {
        return isSelect;
    }

    public void setSelect(boolean select) {
        isSelect = select;
    }

    @Override
    public boolean equals(Object o) {
        return o instanceof GlideSelectIcon;
    }

    @Override
    public int hashCode() {
        return ID_BYTES.hashCode();
    }

    @Override
    public void updateDiskCacheKey(@NonNull MessageDigest messageDigest) {
        messageDigest.update(ID_BYTES);
    }
}
```
```java
   Glide.with(context)
      .load(url)
      .transform(new GlideSelectIcon())
      .into(imageView);
```
需要注意的是，上面的方法是会直接处理掉图片的，`updateDiskCacheKey` 和`hashCode ` 和 `equals `都只是帮你处理缓存key的。如果你需要自己处理缓存，那就更新这几个地方就可以了，如果你只需要缓存原图可以用`.diskCacheStrategy(DiskCacheStrategy.RESOURCE)`的策略。

这里有一个坑需要注意的是，这货如果你不是更新缓存的话，那么就不会走`BitmapTransformation.transform() `这个方法，也就是说只有第一次加载和更新缓存的时候才会走这个方法，所以这是一次性处理的效果。但是每次都更新缓存的话又太费流量了，还有啥办法呢？
# 2. [Target 变换](https://muyangmin.github.io/glide-docs-cn/doc/targets.html#%E5%85%B3%E4%BA%8Etarget)
我这边的需求恰好是需要每次都更新一边图片，但是我又不想每次都刷新缓存，那么怎么办呢？就可以使用加载后的图片处理方法`ViewTarget`。

```java
import android.graphics.Bitmap;
import android.graphics.Canvas;
import android.graphics.PixelFormat;
import android.graphics.drawable.Drawable;
import android.support.annotation.Nullable;
import android.widget.ImageView;

import com.bumptech.glide.Glide;
import com.bumptech.glide.request.target.DrawableImageViewTarget;

/**
 * Created by JayChou on 2019/3/6.
 */
public class GlideSelectDrawableImageViewTarget extends DrawableImageViewTarget {

    private boolean isSelect = false;

    public GlideSelectDrawableImageViewTarget(ImageView view, boolean isSelect) {
        super(view);
        Glide.with(view).clear(view);
        this.isSelect = isSelect;
    }

    @Override
    protected void setResource(@Nullable Drawable resource) {
        if (resource != null) {
            Bitmap toTransform = drawableToBitmap(resource);
            Bitmap bitmap;
            if (isSelect) {
                bitmap = Bitmap.createBitmap(toTransform, toTransform.getWidth() / 2, 0, toTransform.getWidth() / 2, toTransform.getHeight()); //对图片的高度的一半进行裁剪
            } else {
                bitmap = Bitmap.createBitmap(toTransform, 0, 0, toTransform.getWidth() / 2, toTransform.getHeight()); //对图片的高度的一半进行裁剪
            }
            view.setImageBitmap(bitmap);
        }
    }

    public Bitmap drawableToBitmap(Drawable drawable) {
        // 取 drawable 的长宽
        int w = drawable.getIntrinsicWidth();
        int h = drawable.getIntrinsicHeight();

        // 取 drawable 的颜色格式
        Bitmap.Config config = drawable.getOpacity() != PixelFormat.OPAQUE ? Bitmap.Config.ARGB_8888
                : Bitmap.Config.RGB_565;
        // 建立对应 bitmap
        Bitmap bitmap = Bitmap.createBitmap(w, h, config);
        // 建立对应 bitmap 的画布
        Canvas canvas = new Canvas(bitmap);
        drawable.setBounds(0, 0, w, h);
        // 把 drawable 内容画到画布中
        drawable.draw(canvas);
        return bitmap;
    }
}

```
然后添加到Glide中就可以了
```java
   Glide.with(context)
      .load(url)
      .into(new GlideSelectDrawableImageViewTarget(imageView,true));
```

这里需要说明的地方是：
我因为用的是`DrawableImageViewTarget`，所以自己把`drawable`转成了`Bitmap`来处理的，你也可以直接处理`drawable`。

我每次都调用了一下`Glide.with(view).clear(view);`，是因为Glide在添加 `ViewTarget`之后，会针对当前的imageView做引用复用，所以除非你手动清理一遍ImagView的引用，否者每次新设置的`ViewTarget`都不会起作用，只会复用上一次的`ViewTarget`。

这里踩了很长时间的坑，一直很奇怪的是`setResource`中的`isSelect`一直没有变化，排查了很久才发现这里，关于这一段的说明在[这里](https://muyangmin.github.io/glide-docs-cn/doc/targets.html#%E5%8F%96%E6%B6%88%E5%92%8C%E9%87%8D%E7%94%A8)。
![](/images/glide官方文档截图Target.png)

---
第一个是因为缓存的问题卡住了，第二个是因为对象引用卡住了。以上就是关于这次的Glide踩坑记录，论官方文档全看完并理解的重要性。
