---
title: Bitmap压缩策略
date: 2018-3-31
categories: android
author: Sunny
tags:
    - android
    - Bitmap
cover_picture: https://upload-images.jianshu.io/upload_images/5231076-c573b3176fab0f29.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
---

![Bitmap压缩策略.png](https://upload-images.jianshu.io/upload_images/5231076-c573b3176fab0f29.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1.Bitmap高效加载的核心思想

原因：Bitmap不能高效加载的原因是由于内存浪费可能导致的OOM。如果我们使用ImageView来显示图片，ImageView的大小没有原始图片那么大，此时把原图加载进内存就有相当一部分内存被浪费，如果图片比较多或者图片占用的内存没有得到及时释放，就会导致OOM。

解决方案：按一定的采样率将图片缩小后再加载进内存。

### 2.加载Bitmap的方式

BitmapFactory类提供了八种方法：

decodeFile(String)，decodeFile(String,Options)

decodeResource(Resources,int)，decodeResource(Resource,int,Options)

decodeStream(InputStream)，decodeStream(InputStream,Rect,Options)

decodeByteArray(byte[],int,int)，decodeByteArray(byte[],int,int,Options)

### 3.BitmapFactory.Options参数

①inSampleSize参数

采样率，Bitmap会根据这个值去算实际需要的Bitmap的大小，该值如果小于1会按1来算，1表示取原图的大小，大于1，表示图片的宽高取1/n(n表示inSampleSize值)，官方文档给出的inSampleSize取值为2的指数，即1,2,4,8等，如果不是2的指数系统会自动向下取整，取一个最接近2的指数的整数来代替，比如现在inSampleSize的值为3，那么系统实际上会按照inSampleSize = 2来计算图片的缩放比例；还需要注意一点的是当宽高的缩放比例不一样的时候，通常是取小的，这样不会导致图片显示不完整或者因为拉伸造成图片的失真。

举例：现在有一张200 x 400的原始图片，需要在一个100 x 100的ImageView上显示，那么宽的缩放比为2，高的缩放比是4，这时候应该取inSampleSize值为2，取小的值，这样实际获得的图片大小为100 x 200，在100 x 100上就能显示出来了，即使高多了一点也没关系，如果按照4的比例来取得话，那么实际获得的图片大小就是50 x 100，宽要比ImageView的宽小一半，这样显示的时候就不能铺满，如果硬要铺满会造成图片宽度被拉伸，从而失真。

②inJustDecodeBounds参数

一般情况下，我们获取图片的原始大小需要先加载图片，这样原始图片还是被加载到了内存中，这违背了高效加载Bitmap的初衷；设置inJustDecodeBounds = true就能在不加载图片的情况下获取到原始图片的大小，然后根据图片原始大小和inSampleSize来计算出实际需要的图片大小，然后再设置inJustDecodeBounds = false就能按照实际需要的图片大小去加载图片到内存。

### 4.高效加载Bitmap的流程

①new一个BitmapFactory.Options对象，设置Options中字段inJustDecodeBounds = true，并加载图片；

②从BitmapFactory.Options中获取图片的原始宽高，宽高分别对应于outWidth和outHeight字段；

③根据采样率的规则和目标View的大小计算出所需inSampleSize值；

④把inJustDecodeBounds设置为false，再根据inSampleSize加载图片。

### 5.Bitmap高效加载代码实现

```java
public static Bitmap decodeSampleBitmapByResource(Resources resources, @IdRes int resId, int reqWidth, int reqHeight) {
    //new一个Options对象并设置inJustDecodeBounds = true
    BitmapFactory.Options op = new BitmapFactory.Options();
    op.inJustDecodeBounds = true;
    //加载图片
    BitmapFactory.decodeResource(resources, resId, op);
    //依据图片的原始大小、目标View的大小以及获取inSampleSize值得规则计算inSampleSize的值
    op.inSampleSize = calculateInSample(op, reqWidth, reqHeight);
    //设置inJustDecodeBounds = false
    op.inJustDecodeBounds = false;
    //再根据inSampleSize加载图片
    return BitmapFactory.decodeResource(resources, resId, op);
}

public static int calculateInSample(BitmapFactory.Options options, int reqWidth, int reqHeight) {
    final int orWidth = options.outWidth;
    final int orHeight = options.outHeight;
    int inSample = 1;
    if (orWidth > reqWidth || orHeight > reqHeight) {
        int halfWidth = orWidth / 2;
        int halfHeight = orHeight / 2;
        while (halfWidth / inSample >= reqWidth && halfHeight / inSample >= reqHeight) {
            inSample *= 2;
        }
    }
    return inSample;
}
```



