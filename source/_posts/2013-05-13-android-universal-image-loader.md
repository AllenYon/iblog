---
category: Android
date: 2013-05-13
layout: post
title: 介绍开源项目 Android-Universal-Image-Loader -Part 1
---

Github地址：[Android-Universal-Image-Loader](https://github.com/nostra13/Android-Universal-Image-Loader) 万能的图片加载器!  
作者：[nostra13](https://github.com/nostra13)

废话不多开始介绍

This project aims to provide a reusable instrument for asynchronous image loading, caching and displaying. It is originally based on Fedor Vlasov's project and has been vastly refactored and improved since then.

作者说他也是[Fedor Vlasov's project](https://github.com/thest1/LazyList)这个项目改写过来的，做到对图片的异步加载，缓存和显示。

##特性
- 多线程图片加载
- 尽可能多的配置选项（线程池，加载器，解析器，内存/磁盘缓存，显示参数等等）
- 图片可以缓存在内存中，或者设备文件目录下，或者SD卡中
- 可以监听加载进度
- 可以自定义显示每一张图片时都带不同参数
- 支持Widget

## Quick Setup
### 1.Include library
[下载连接 JAR](https://github.com/nostra13/Android-Universal-Image-Loader/raw/master/downloads/universal-image-loader-1.8.4-with-sources.jar)

### 2.Android Manifest

```
<manifest>
    <uses-permission android:name="android.permission.INTERNET" />
    <!-- Include next permission if you want to allow UIL to cache images on SD card -->
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    ...
    <application android:name="MyApplication">
        ...
    </application>
</manifest>
```
### 3. Application class
```
public class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        // Create global configuration and initialize ImageLoader with this configuration
        ImageLoaderConfiguration config = new ImageLoaderConfiguration.Builder(getApplicationContext())
            ...
            .build();
        ImageLoader.getInstance().init(config);
    }
}
```

## Configuration and Display Options

- Configuration(ImageLoaderConfiguration) 是相对于整个应用的配置
- Display Options(DisplayImageOptions)是针对每一个显示图片任务所提供的参数(ImageLoader.displayImage(…)).

### Configuration
所有的选项都是可选的，只选择你真正想制定的去配置。

```
// DON'T COPY THIS CODE TO YOUR PROJECT! This is just example of ALL options using.
File cacheDir = StorageUtils.getCacheDirectory(context);
ImageLoaderConfiguration config = new ImageLoaderConfiguration.Builder(context)
		//如果图片尺寸大于了这个参数，那么就会这按照这个参数对图片大小进行限制并缓存
        .memoryCacheExtraOptions(480, 800) // default=device screen dimensions
        .discCacheExtraOptions(480, 800, CompressFormat.JPEG, 75)
        .taskExecutor(AsyncTask.THREAD_POOL_EXECUTOR)
        .taskExecutorForCachedImages(AsyncTask.THREAD_POOL_EXECUTOR)
        .threadPoolSize(3) // default
        .threadPriority(Thread.NORM_PRIORITY - 1) // default
        .tasksProcessingOrder(QueueProcessingType.FIFO) // default
        .denyCacheImageMultipleSizesInMemory()
        .memoryCache(new LruMemoryCache(2 * 1024 * 1024))
        .memoryCacheSize(2 * 1024 * 1024)
        .discCache(new UnlimitedDiscCache(cacheDir)) // default
        .discCacheSize(50 * 1024 * 1024)
        .discCacheFileCount(100)
        .discCacheFileNameGenerator(new HashCodeFileNameGenerator()) // default
        .imageDownloader(new BaseImageDownloader(context)) // default
        .imageDecoder(new BaseImageDecoder()) // default
        .defaultDisplayImageOptions(DisplayImageOptions.createSimple()) // default
        .enableLogging()
        .build();
```
参数主要包括了这么几类配置
1. 线程池配置
2. 内存缓存配置
3. 磁盘缓存配置
4. 使用哪个图片下载器
5. 使用哪个图片解析器
实际上，不做任何配置也是ImageLoader也是可以使用的。
插一句，配置选项的确够丰富，但有多是没有必要的。

### Display Options

显示参数可以分别被每一个显示任务调用(ImageLoader.displayImage(…))

Note:**如果没有调用ImageLoader.displayImage(…),那么将使用配置选项中的默认显示参数(ImageLoaderConfiguration.defaultDisplayImageOptions(…))**

```
DisplayImageOptions options = new DisplayImageOptions.Builder()
        .showStubImage(R.drawable.ic_stub)  // 在显示真正的图片前，会加载这个资源
        .showImageForEmptyUri(R.drawable.ic_empty) //空的Url时
        .showImageOnFail(R.drawable.ic_error)
        .resetViewBeforeLoading() //
        .delayBeforeLoading(1000)     // 延长1000ms 加载图片  （想不出来用在什么场景下）
        .cacheInMemory()              
        .cacheOnDisc()               
        .preProcessor(...)            
        .postProcessor(...)           
        .extraForDownloader(...)      //可以向加载器携带一些参数
        .imageScaleType(ImageScaleType.IN_SAMPLE_POWER_OF_2) // default  
        .bitmapConfig(Bitmap.Config.ARGB_8888) // default
        .decodingOptions(...)
        .displayer(new SimpleBitmapDisplayer()) // default
        .handler(new Handler()) // default
        .build();
```
## 用法
### 可接收的URL
```
String imageUri = "http://site.com/image.png"; // from Web
String imageUri = "file:///mnt/sdcard/image.png"; // from SD card
String imageUri = "content://media/external/audio/albumart/13"; // from content provider
String imageUri = "assets://image.png"; // from assets
String imageUri = "drawable://" + R.drawable.image; // from drawables (only imag
es, non-9patch)
```
###例子
```
// Load image, decode it to Bitmap and display Bitmap in ImageView
imageLoader.displayImage(imageUri, imageView);
```
```
// Load image, decode it to Bitmap and return Bitmap to callback
imageLoader.loadImage(imageUri, new SimpleImageLoadingListener() {
    @Override
    public void onLoadingComplete(String imageUri, View view, Bitmap loadedImage) {
        // Do whatever you want with Bitmap
    }
});
```
### 完整版
```
// Load image, decode it to Bitmap and display Bitmap in ImageView
imageLoader.displayImage(imageUri, imageView, displayOptions, new ImageLoadingListener() {
    @Override
    public void onLoadingStarted(String imageUri, View view) {
        ...
    }
    @Override
    public void onLoadingFailed(String imageUri, View view, FailReason failReason) {
        ...
    }
    @Override
    public void onLoadingComplete(String imageUri, View view, Bitmap loadedImage) {
        ...
    }
    @Override
    public void onLoadingCancelled(String imageUri, View view) {
        ...
    }
});
```
```
ImageSize targetSize = new ImageSize(120, 80); // result Bitmap will be fit to this size
imageLoader.loadImage(imageUri, targetSize, displayOptions, new SimpleImageLoadingListener() {
    @Override
    public void onLoadingComplete(String imageUri, View view, Bitmap loadedImage) {
        // Do whatever you want with Bitmap
    }
});
```
###ImageLoader Helpers
一些可能会用到的帮助类和方法

```
ImageLoader |
            | - getMemoryCache()
            | - clearMemoryCache()
            | - getDiscCache()
            | - clearDiscCache()
            | - denyNetworkDownloads(boolean)
            | - handleSlowNetwork(boolean)
            | - pause()
            | - resume()
            | - stop()
            | - destroy()
            | - getLoadingUriForView(ImageView)
            | - cancelDisplayTask(ImageView)
MemoryCacheUtil |
                | - findCachedBitmapsForImageUri(...)
                | - findCacheKeysForImageUri(...)
                | - removeFromCache(...)
DiscCacheUtil |
              | - findInCache(...)
              | - removeFromCache(...)
StorageUtils |
             | - getCacheDirectory(Context)
             | - getIndividualCacheDirectory(Context)
             | - getOwnCacheDirectory(Context, String)
PauseOnScrollListener
```

## Useful Info

1.使用默认值时缓存无效。如果你想要加载图片的时候使用内存/磁盘中的缓存，那么你应该这样设置DisplayImageOptions:

```
// Create default options which will be used for every
//  displayImage(...) call if no options will be passed to this method
DisplayImageOptions defaultOptions = new DisplayImageOptions.Builder()
        ...
        .cacheInMemory()
        .cacheOnDisc()
        ...
        .build();
ImageLoaderConfiguration config = new ImageLoaderConfiguration.Builder(getApplicationContext())
        ...
        .defaultDisplayImageOptions(defaultOptions)
        ...
        .build();
ImageLoader.getInstance().init(config); // Do it on Application start
```
```
// Then later, when you want to display image
ImageLoader.getInstance().displayImage(imageUrl, imageView); // Default options will be used
```
或者这样:

```
DisplayImageOptions options = new DisplayImageOptions.Builder()
        ...
        .cacheInMemory()
        .cacheOnDisc()
        ...
        .build();
ImageLoader.getInstance().displayImage(imageUrl, imageView, options); // Incoming options will be used
```

2.如果你允许缓存在磁盘中，那么UIL将尝试缓存到(/sdcard/Android/data/[package_name]/cache)路径中。如果外部存储不可用的话，图片将缓存在设备文件系统下。为了支持在外部存储(SD card）中缓存,那么应该在AndroidManifest.xml中加上这个权限:

```
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```

3.UIL如果知道Bitmap应该有多大的尺寸放在确定了点ImageView中？它搜索以下几个定义的参数：

- 从ImageView测量准确的高和宽获得
- 从android:layout_width 和 android:layout_height 参数获得
- 从 android:maxWidth 和/或者 android:maxHeight 参数获得
- 从在configuration定义的maximun width 和/或者 height参数获得
- 从设备屏幕 width 和/或者 height 获得

If you often got OutOfMemoryError in your app using Universal Image Loader then try next (all of them or several):

- Reduce thread pool size in configuration (.threadPoolSize(...)). 1 - 5 is recommended.
- Use .bitmapConfig(Bitmap.Config.RGB_565) in display options. Bitmaps in RGB_565 consume 2 times less memory than in ARGB_8888.
- Use .memoryCache(new WeakMemoryCache()) in configuration or disable caching in memory at all in display options (don't call .cacheInMemory()).
- Use .imageScaleType(ImageScaleType.IN_SAMPLE_INT) in display options. Or try .imageScaleType(ImageScaleType.EXACTLY).
- Avoid using RoundedBitmapDisplayer. It creates new Bitmap object with ARGB_8888 config for displaying during work.

5.For memory cache configuration (ImageLoaderConfiguration.memoryCache(...)) you can use already prepared implementations.

- Cache using only strong references:
LruMemoryCache (Least recently used bitmap is deleted when cache size limit is exceeded) - Used by default for API >= 9
- Caches using weak and strong references:
UsingFreqLimitedMemoryCache (Least frequently used bitmap is deleted when cache size limit is exceeded)
LRULimitedMemoryCache (Least recently used bitmap is deleted when cache size limit is exceeded) - Used by default for API < 9
FIFOLimitedMemoryCache (FIFO rule is used for deletion when cache size limit is exceeded)
LargestLimitedMemoryCache (The largest bitmap is deleted when cache size limit is exceeded)
LimitedAgeMemoryCache (Decorator. Cached object is deleted when its age exceeds defined value)
- Cache using only weak references:
WeakMemoryCache (Unlimited cache)

6.For disc cache configuration (ImageLoaderConfiguration.discCache(...)) you can use already prepared implementations:

- UnlimitedDiscCache (The fastest cache, doesn't limit cache size) - Used by default
- TotalSizeLimitedDiscCache (Cache limited by total cache size. If cache size exceeds specified limit then file with the most oldest last usage date will be deleted)
- FileCountLimitedDiscCache (Cache limited by file count. If file count in cache directory exceeds specified limit then file with the most oldest last usage date will be deleted. Use it if your cached files are of about the same size.)
- LimitedAgeDiscCache (Size-unlimited cache with limited files' lifetime. If age of cached file exceeds defined limit then it will be deleted from cache.)
**NOTE: UnlimitedDiscCache is 30%-faster than other limited disc cache implementations.**

7.To display bitmap (DisplayImageOptions.displayer(...)) you can use already prepared implementations:
- RoundedBitmapDisplayer (Displays bitmap with rounded corners)
- FadeInBitmapDisplayer (Displays image with "fade in" animation)

8.To avoid list (grid, ...) scrolling lags you can use PauseOnScrollListener:

```
boolean pauseOnScroll = false; // or true
boolean pauseOnFling = true; // or false
PauseOnScrollListener listener = new PauseOnScrollListener(imageLoader, pauseOnScroll, pauseOnFling);
listView.setOnScrollListener(listener);
```
