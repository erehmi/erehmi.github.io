---
date: "2015-02-08 23:26:00"
description: ""
layout: post
permalink: android-case-gallery
categories:
  - blog
  - android
  - case
title: Android开发案例 - 图库
---

本文不涉及UI方面的内容, 如果您是希望了解UI方面的访客, 请跳过此文. 
本文将要详细介绍如何实现流畅加载本地图库. 像平时用得比较多应用, 如微信(见下图),
微博等应用, 都实现了图库功能, 其中主要功能包括:
-   默认显示所有图片
-   按目录显示图片
另外, 界面要素包括:
-   图片缩略图
-   图片目录列表以及目录中包含的图片数
![](<http://erehmi.github.io/assets/image/wechat-gallery.png>)

**讨论: **[在Android上,
如何实现流畅加载本地照片的相册? ](<http://segmentfault.com/q/1010000002542272/a-1020000002543547>)
 
**知识要点**

-   *ContentProvider* - 数据存取接口

-   *CursorLoader* - Cursor异步加载器

-   [Android Universal Image Loader](<https://github.com/nostra13/Android-Universal-Image-Loader>)

p.s. 实现图库的难点就在于, 如何快速的查询出图片以及目录信息, 貌似 Android
没有直接提供这样的接口,
我们只可以用 *android.provider.MediaStore.Images.Media* 和 *android.provider.MediaStore.Images.Thumbnails*.
我们虽然能使用 *Thumbnails* 查询出缩略图信息和图片ID,
但是它没有提供图片的详细信息, 另外,
如果用于保存缩略图的信息或者目录被(意外或者人为)删除了,
那使用 *Thumbnails* 基本上就没有什么意义了. 因此,
我们在这里使用 *Media* 来查询图片以及目录. 


**实现代码**
**\> 定义**
{% highlight java %}
static final Uri CONTENT_URI = Media.EXTERNAL_CONTENT_URI;
static final String SORT_ORDER = Media.DATE_MODIFIED + " DESC";
{% endhighlight %}

在这里, 我们只查询sdcard上的图片, 并按照修改时间倒序排列.

**\> 查询图片**
代码略, 直接使用 *CursorLoader* 按 **SORT\_ORDER **顺序加载图片,
如果指定了目录, 则设置 **Media.BUCKET\_ID + "=?"** 的查询条件

**\> 查询目录**
和查询图片一样, 也使用CursorLoader来加载数据, 只不过设置的参数不同而已,
如下:
{% highlight java %}
// 方法一
static final String[] PROJECTION_D = {
                            "DISTINCT " + Media.BUCKET_ID, 
                            Media.BUCKET_DISPLAY_NAME,
                            "COUNT(*) AS " + Media._COUNT};
// 方法二
static final String[] PROJECTION = {
                            Media.BUCKET_ID,
                            Media.BUCKET_DISPLAY_NAME,
                            Media._ID,
                            "COUNT(*) AS " + Media._COUNT};
static final String SELECTION = "1=1) GROUP BY (" + Media.BUCKET_ID;
static final String[] SELECTION_ARGS = null;
{% endhighlight %}

 　　需要说明的是, 如果对应的目录不需要显示首张图片的缩略图, 那么可以使用方法一,
否则使用方法二(*PROJECTION* 要与 *SELETION* 配合使用, 而方法一不需要).
其他参数对应设置到CursorLoader即可.


**优化策略**
　　最后, 需要提到的一点就是, 我们使用 UIL(即, [Android Universal Image
Loader](<https://github.com/nostra13/Android-Universal-Image-Loader>)) 加载图片,
但是每次加载都是我们查询出来的原图, 按照github上演示代码的全局初始设定,
加载到三五屏, 第一屏的图片就已经不在内存缓存里,
如果重新滚动到第一屏要显示第一屏图片的话, 就还得从原图读取, 用户体验就很差了.
而我们看到Android相册, 微信相册等应用, 加载很流畅. 通过观察这些应用缓存,
能看到只要浏览很多未被浏览的图片, 那缓存就会变大. 因此,
可以想到的可实现的方式就是:

-   利用UIL缓存原图为较小尺寸的缩略图, 比如320px或者96px


到这里, 按此方法实现的图库的性能以及用户体验基本和Android相册差不多了.
此方法唯一不足的是, 需要应用自己缓存缩略图, 可以假想下,
如果手机上已安装的80%的应用都有这样的一个功能页面,
那岂不是每个应用都要自己生成一套缩略图? 


**END. \>\> SEE MORE:**
[http://erehmi.github.io/](<**http://erehmi.github.io/**>)
