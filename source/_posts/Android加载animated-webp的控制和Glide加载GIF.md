---
title: Android加载animated-webp的控制和Glide加载GIF
tags: Android
comments: false
date: 2019-03-08 13:21:43
---
因为动态的webp使用的也越来越多了，所以这里记录一种加载处理的办法。目前常用的Android图片加载库，只有`fresco`是可以直接加载animated webp的。那么如何处理呢？记录一下，不然坑都白踩了。本质上webp和gif都是一组图片组成的连续图片，如果要单独解析每一帧怎么办呢。
<!-- more -->
# Android支持
如果要拿到webp的第一帧，在Android以上是可以直接使用如下这种方法，默认取的是第一帧，但是，这种方法只有在8.0及以上才有用。
```java
  Bitmap  bitmap =  BitmapFactory.decodeFile(filename);
```

# fresco控制animated webp
添加`fresco`支持webp的方法是：
```gradle
  implementation 'com.facebook.fresco:fresco:1.13.0'
  implementation 'com.facebook.fresco:webpsupport:1.13.0'
//implementation 'com.facebook.fresco:animated-gif:1.13.0'
```
添加之后就可以直接加载webp了，也可以设置自动开始播放，如下：
```java
  ImageRequestBuilder imageRequestBuilder = ImageRequestBuilder.newBuilderWithSource(uri);
  PipelineDraweeControllerBuilder builder = Fresco.newDraweeControllerBuilder();
  builder.setImageRequest(imageRequestBuilder.build());
  builder.setAutoPlayAnimations(true);
  builder.setControllerListener(new BaseControllerListener<ImageInfo>() {
    @Override
    public void onFinalImageSet(String id, ImageInfo imageInfo, Animatable animatable) {
    //加载完成的监听
    }
  });
  simpleDraweeView.setController(builder.build());
```
到这一步都很简单，但是要加载webp的第一帧咋办呢？
先来一种最简单的，类似于`BitmapFactory.decodeFile()`，`fresco`中也有一个方法，同样的，这种方法也是仅支持8.0及以上
```java
  Bitmap bitmap = WebpSupportStatus
                  .loadWebpBitmapFactoryIfExists()
                  .decodeFile(filename, new BitmapFactory.Options());
```
那么还有什么办法呢？还可以使用下面这种方法。这种方法是用的是`fresco`的订阅者，可以处理加载之后的数据，可以拿到webp的每一帧，然后做你想要的处理，[官网链接](https://frescolib.org/docs/datasources-datasubscribers.html)。
```java
  ImageDecodeOptionsBuilder decodeOptionsBuilder = new ImageDecodeOptionsBuilder();
  decodeOptionsBuilder.setDecodeAllFrames(true);
  decodeOptionsBuilder.setBitmapConfig(Bitmap.Config.ARGB_8888);
  ImagePipelineFactory.getInstance()
          .getImagePipeline()
          .fetchDecodedImage(ImageRequestBuilder
                          .newBuilderWithSource(Uri.fromFile(new File(filename)))
                          .setImageDecodeOptions(new ImageDecodeOptions(decodeOptionsBuilder))
                          .build()
                  , context)
          .subscribe(new BaseDataSubscriber<CloseableReference<CloseableImage>>() {
              @Override
              protected void onNewResultImpl(DataSource<CloseableReference<CloseableImage>> dataSource) {
                  if (dataSource.getResult().get() instanceof CloseableAnimatedImage) {
                      CloseableAnimatedImage animatedImage = (CloseableAnimatedImage) dataSource.getResult().get();
                      if (animatedImage.getImage().getFrameCount() > 0) {
                          Bitmap frameBitmap = Bitmap.createBitmap(animatedImage.getWidth(), animatedImage.getHeight(), Bitmap.Config.ARGB_8888);
                          //添加透明通道
                          frameBitmap.eraseColor(Color.TRANSPARENT);
                          frameBitmap.setHasAlpha(true);
                          animatedImage.getImage().getFrame(0).renderFrame(animatedImage.getWidth(), animatedImage.getHeight(), frameBitmap);
                          try {
                              //保存成文件
                              File file = new File(filename);
                              File outFile = new File(file.getParent(), file.getName() + ".png");
                              FileOutputStream out = new FileOutputStream(outFile);
                              frameBitmap.compress(Bitmap.CompressFormat.PNG, 100, out);
                              out.flush();
                              out.close();

                          } catch (FileNotFoundException e) {
                              e.printStackTrace();
                          } catch (IOException e) {
                              e.printStackTrace();
                          }
                      }
                  }

              }

              @Override
              protected void onFailureImpl(DataSource<CloseableReference<CloseableImage>> dataSource) {

              }
          }, CallerThreadExecutor.getInstance());
```
需要注意的地方有三个：
1. 下面这种方法默认解出来的bitmap是不带透明通道的，需要你处理一遍。
2. 第二个是，这种方法拿到的bitmap离开这个方法之后就有可能会被回收掉，如果你要长久使用的话，建议自己再copy一遍，我为了节省解压的时间，我直接保存成为一张png的文件，然后去使用，你也可以直接copy出来bitmap去使用的。
3. 这种方法可以解压出来每一帧，选择你需要帧数来处理就可以了。

# Glide4控制GIF
关于GIF，目前支持的主流的加载库只有`Glide`和`fresco`，`Picasso`并不默认支持。

在这里顺便记录一下Glide4.0加载Gif的并控制播放的方式。在Glide3中`GlideDrawableImageViewTarget `已经被废弃了，可以使用`GifDrawable`来控制，里面有控制GIF的方法，也可以拿到GIF的每一帧来处理，定义好的接口方法就在那里，不离不弃，看下接口就知道了。
```java
public void setStartPlayGif(final boolean startPlayGif) {
    Glide.with(context)
        .asGif()
        .load("gifFile")
        .into(new ImageViewTarget<GifDrawable>(imageView) {
            @Override
            protected void setResource(@Nullable GifDrawable resource) {
                if (resource != null) {
                    if(startPlayGif){
                        //resource.setLoopCount(1);
                        view.setImageDrawable(resource);
                    } else {
                        view.setImageBitmap(resource.getFirstFrame());
                    }
                }
            }
        });
}
```
