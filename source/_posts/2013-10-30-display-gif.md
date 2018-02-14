---
category: Android
date: 2013-10-30
layout: post
title:  Android中显示动画的GIF-Movie类解决方案
---

首先看看GIF文件的存储结构和动画原理-->[[转]GIF图片的文件储存结构和动画原理](http://blog.sina.com.cn/s/blog_571296860100jvhz.html)

以及已知的两种解决方案-->[Android & how to use animated GIF](http://weavora.com/blog/2012/02/07/android-how-to-use-animated-gif/)

Github上还算可以的开源项目-->[gif-movie-view](https://github.com/sbakhtiarov/gif-movie-view)

但实际使用的时候,还是有些小问题,比如

我们的GIF不全是本地已经存在的,可能是从文件加载,或者是网络下载的,这时候在使用Movie.decodeStream(InputStream is)时会遇到

- [IOException on Movie.decodeFile() - android](http://stackoverflow.com/questions/18307266/ioexception-on-movie-decodefile-android)
- [IOException - Cannot load file](http://stackoverflow.com/questions/10240042/ioexception-cannot-load-file)

我的做法是统一使用Movie.decodeByteArray(byte[] b) 类方法,可以避免很多麻烦,下载 [***gif-movie-view***](https://github.com/sbakhtiarov/gif-movie-view)
 项目,并在GifMovieView中添加

```
  public void setMovieByteArray(byte[] bb) {
        mMovie = Movie.decodeByteArray(bb, 0, bb.length);
        requestLayout();
    }
```

假如我们显示GIF的时候是异步的,那么在还没有显示时,GifMovieView不占据任何空间,完成加载后整个界面会重绘,因为GifMovieView重载的onMeasure()函数是这样子的:

```
  @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        if (mMovie != null) {
            int movieWidth = mMovie.width();
            int movieHeight = mMovie.height();
			/*
             * Calculate horizontal scaling
			 */

            float scaleH = 1f;
            int measureModeWidth = MeasureSpec.getMode(widthMeasureSpec);
            if (measureModeWidth != MeasureSpec.UNSPECIFIED) {
                int maximumWidth = MeasureSpec.getSize(widthMeasureSpec);
                if (movieWidth > maximumWidth) {
                    scaleH = (float) movieWidth / (float) maximumWidth;
                }
            }
			/*
             * calculate vertical scaling
			 */

            float scaleW = 1f;
            int measureModeHeight = MeasureSpec.getMode(heightMeasureSpec);

            if (measureModeHeight != MeasureSpec.UNSPECIFIED) {
                int maximumHeight = MeasureSpec.getSize(heightMeasureSpec);
                if (movieHeight > maximumHeight) {
                    scaleW = (float) movieHeight / (float) maximumHeight;
                }
            }

			/*
             * calculate overall scale
			 */
            mScale = 1f / Math.max(scaleH, scaleW);

            mMeasuredMovieWidth = (int) (movieWidth * mScale);
            mMeasuredMovieHeight = (int) (movieHeight * mScale);

            setMeasuredDimension(mMeasuredMovieWidth, mMeasuredMovieHeight);

        } else {
            /*
             * No movie set, just set minimum available size.
			 */
			setMeasuredDimension(getSuggestedMinimumWidth(), getSuggestedMinimumHeight());
        }
    }
```

我们知道setMeasuredDimension(int width,int height);将指定View在父容器里占据的width,和height,假如你们的业务逻辑在得到GIF文件的URL同时,还能获得它的width和height,并且这里你不希望调用requestLayout()函数重绘,做了如下修改

```
setMeasuredDimension(mWidth,mHeight);  //mWidth, mHeight 是已知Gif的宽高
```

这个时候可能连显示也不正常了! 可能是因为父容器判断GifMovieView大小并没有发生改变,这样的话就不调用onLayout() 和 onDraw() 函数,也就没有开始GIF的动画循环,所以就是一片空白. 有待考证.

另外还有些注意点 比如API 11 之后不能使用硬件渲染:

```
  /**
	* Starting from HONEYCOMB have to turn off HW acceleration to draw
	* Movie on Canvas.
    */
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
    	setLayerType(View.LAYER_TYPE_SOFTWARE, null);
    }
```

###总结:
Movie的坑还是很多的,不像成品类库,相对于WebView来显示GIF的优势是轻量级,更易管理.
