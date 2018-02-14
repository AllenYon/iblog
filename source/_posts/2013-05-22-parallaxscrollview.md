---
category: Android
date: 2013-05-22
layout: post
title: Android中制作有景深视差的ScrollView
---

<img src="http://pic.yupoo.com/wsyanligang_v/CSz7cQTh/UZ5XO.png" width=330 />

首先我们需要创建一个类继承ViewGroup，用来包含前景(ForgroundView)和后景(BackgourdView)。

```
public class ParallaxScrollView extends ViewGroup{}
```

接着是一个继承自ScrollView的类，每当`onScrollChanged()`触发时，都反馈给ParallaxScrollView。

```
public class ObservableScrollView extends ScrollView{
	private ScrollCallbacks mCallbacks;
	@Override
    protected void onScrollChanged(int l, int t, int oldl, int oldt) {
        super.onScrollChanged(l, t, oldl, oldt);
        if (mCallbacks != null) {
            mCallbacks.onScrollChanged(l, t, oldl, oldt);
        }
    }
    static interface ScrollCallbacks {
        public void onScrollChanged(int l, int t, int oldl, int oldt);
    }
}
```

然后如何使用呢？我们需要在xml中这样定义

```
<couk.jenxsol.parallaxscrollview.views.ParallaxScrollView>
	<!--Background-->
	<ImageView
	    android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:scaleType="fitXY"
        android:src="..."/>
	<!--Foreground-->
	<couk.jenxsol.parallaxscrollview.views.ObservableScrollView>
		<LinearLayout>
		</LinearLayout>
	<couk.jenxsol.parallaxscrollview.views.ObservableScrollView/>
</couk.jenxsol.parallaxscrollview.views.ParallaxScrollView>
```

做到前后景视差，就需要当ScrollView滑动时，前景和后景滑动的距离不同，例如：前景正常向上滑动了10px,但是后景这时候只向上滑动3px，我们将这个变量定义为`mScrollDiff`,那么`mScrollDiff`等于0.3f。

`ScrollView`的高度可以分为两部分，一个是`ScrollContentHeight`，ScrollView中包含的LinearLayout的高度，通常这个内容是很长的，一个是`ScollViewHeight`，通常就是我们屏幕的高再减去状态栏或者标题栏，即可见范围。以上我们称为前景。

后景就是我们的背景了，我们需要在每当前景滚动时，就动态改变后景的布局。

那我们就一步步解析如果做到前后景视差的ScrollView吧，首先在ParallaxScrollView完成布局填充后，我们完成一些初始化工作:

```
protected void onFinishInflate() {
    super.onFinishInflate();
    if (getChildCount() > 2) {
        throw new IllegalStateException("ParallaxScrollView can host only two direct children");
    }
    organiseViews();
}
```

作者考虑到了如果开发者没有按照使用说明来编写xml的话的一些意外情况，我们现在暂时忽略。

```
private void organiseViews() {
    if (getChildCount() <= 0)
        return;
   // 将第一个子元素作为背景，第二个子元素作为前景
       final View background = getChildAt(0);
       final View foreground = getChildAt(1);
       organiseBackgroundView(background);
       organiseForgroundView(foreground);
    }
private void organiseBackgroundView(final View background) {
    mBackground = background;
}
private void organiseForgroundView(final View forground) {
	…
    mScrollView = (ObservableScrollView) forground;
	…
    if (mScrollView != null) {
        mScrollView.setLayoutParams(forground.getLayoutParams());
        mScrollView.setCallbacks(mScrollCallbacks);
        mScrollView.setFillViewport(true);
    }
}
```

每当ObservaleScrollView触发`onScrollChanged()`时，ParallaxScrollView都调用
`requestLayout()`

```
private final ScrollCallbacks mScrollCallbacks = new ScrollCallbacks() {
   @Override
   public void onScrollChanged(int l, int t, int oldl, int oldt) {
       requestLayout();
    }
};
```

接着就是最重要的`onMeasure()`和`onLayout()`函数了,他们分别计算出前景和后景的宽和高，以及每次调用`requestLayout()`时,`onLayout()`函数将视图放置在正确的位置上。首先看看`onMeasure()`函数做了什么事情：

```
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    if (mScrollView != null) {
      	measureChild(mScrollView,
       		MeasureSpec.makeMeasureSpec(MeasureSpec.getSize(widthMeasureSpec), MeasureSpec.AT_MOST),
           MeasureSpec.makeMeasureSpec(MeasureSpec.getSize(heightMeasureSpec), MeasureSpec.AT_MOST));
        mScrollContentHeight = mScrollView.getChildAt(0).getMeasuredHeight();
        mScrollViewHeight = mScrollView.getMeasuredHeight();
    }
    if (mBackground != null) {
        int minHeight = 0;
        minHeight = (int) (mScrollViewHeight +
                mParallaxOffset * (mScrollContentHeight - mScrollViewHeight));
        minHeight = Math.max(minHeight, MeasureSpec.getSize(heightMeasureSpec));
        measureChild(mBackground,
            MeasureSpec.makeMeasureSpec(MeasureSpec.getSize(widthMeasureSpec), MeasureSpec.EXACTLY),
            MeasureSpec.makeMeasureSpec(minHeight, MeasureSpec.EXACTLY));
        mBackgroundRight = getLeft() + mBackground.getMeasuredWidth();
        mBackgroundBottom = getTop() + mBackground.getMeasuredHeight();
        mScrollDiff = (float) (mBackground.getMeasuredHeight() - mScrollViewHeight)/(float) (mScrollContentHeight - mScrollViewHeight);
    }
}
```
通过调用`measureChild()`函数，我们就可以从`getMeasureHeight()`函数得到被计算之后的视图的高度。计算背景的minHeight时，使用了`Math.max()`进行了一次比较，这是因为有可能我们提供的背景原图的高度比拉伸后的背景还要高，这时候就需要选一个高度数值比较大的minHeight了。

`mParallaxOffset`是对背景图片拉伸比例的变量，默认值为0.2f。`mScrollDiff`就是前景和后景的滑动比了。

```
protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
    final int parentLeft = getPaddingLeft();
    final int parentRight = right - left - getPaddingRight();
    final int parentTop = getPaddingTop();
    final int parentBottom = bottom - top - getPaddingBottom();
    if (mScrollView != null && mScrollView.getVisibility() != GONE) {
    	final int width = mScrollView.getMeasuredWidth();
        final int height = mScrollView.getMeasuredHeight();
        int childLeft=0;
        int childTop=0;
        //暂时省略gravity参数
        mScrollView.layout(childLeft, childTop, childLeft + width, childTop + height);
    }
    if (mBackground != null) {
        final int scrollYCenterOffset = -mScrollView.getScrollY();
        final int offset = (int) (scrollYCenterOffset * mScrollDiff);
        mBackground.layout(getLeft(), offset, mBackgroundRight, offset + mBackgroundBottom);
    }
}
```

至此所有要点都介绍完毕，你可以从[Github:ParallaxScrollView](https://github.com/chrisjenx/ParallaxScrollView)这个项目下载到全部详细的源码。
