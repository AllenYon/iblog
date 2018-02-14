---
category: Android
date: 2013-06-19
layout: post
title:  Google官方指南-动画滚动手势
---

source:[http://developer.android.com/training/gestures/scroll.html](http://developer.android.com/training/gestures/scroll.html)

在Android中，通常我们使用ScrollView类来实现滑动。任何标准的布局都可能超出容器本身的边界，需要嵌套在由框架管理的可滑动视图的ScrollView中去展示。只有在需要特定的场景中，我们才可能需要去实现一个自定的Scroller。本章节描述了这么一个场景，显示一个使用了滑动器，响应触摸手势的滑动效果。

你可以使用Scroller（如 Scroller 或者 OverScroller）去收集生产一个滑动动画所需要响应的触摸事件的数据。它们很类似，但是OverScroller包含指示用户甩动手势已经到达内容边界的方法。InteractiveChart示例使用了EdgeEffect类(准确地说是 EdgeEffectCompat类)去显示发光的效果，当用户到达内容的边界。

>>>注意：我们建议你使用OverScroller而不是Scroller去实现滑动动画。OverScroller为旧的设备提供了最好的向后兼容性。


同样提醒你通常只有你要自己实现滑动时才使用scrollers。ScrollView和HorizontalScrollView已经为你准备好一切，如果你把布局嵌套其中。

Scroller被用于随时间推移动画滚动，使用平台标准的滚动物理（摩擦，速度等）。Scroller其实本身不绘制任何东西。Scrollers随时间推移跟踪滚动偏移，但是他们不会自动地将你的视图移动到那些位置。而是应该由你去获取坐标并且按照一定的速率，以使得滚动动画看起来比较平滑。

##理解滚动术语
---

"Scrolling"在不同的Android中不同的上下文有着不同的含义。

- 滚动是移动视图窗口(viewport--也就是你正看到的内容的窗口)的一般过程。当同时在x和y轴滚动式，称为平移(panning)。在样品应用中提供了这样一个类，InteractiveChart,说明了两种类型的滚动,拖拽和甩动。
- 拖动是一种滚动--发生在用户将他的手指在触摸屏上拖动。简单的拖动经常是通过重写GestureDetector.OnGestureListener类的onScroll()函数实现的。更多关于拖动的讨论，查看Dragging and Scaling.

甩是一种滚动发生在用户拖动并快速抬起他的手指。在用户抬起他的手指后，你通常要保持滚动(移动视口),但是减速滚动直到视口停止移动。甩动作可以通过重写GestureDetector.OnGestureListener类的onFling()实现，并且使用一个scroller类。这就是本主题要将的使用情况。

通常使用scroller对象关联甩手势，但他们可以被用在几乎任何你想要UI去反馈触摸事件显示滚动的上下文中。例如，你可以重写onTouchEvent()直接处理触摸事件，并且产生一个滚动效果或者一个吸附页面（snapping to page）动画响应那些触摸事件


##实现基于触摸的滚动
---

本段描述了如何使用scroller类，下面展示的片段来自InteractiveChart。它使用了GestureDetector，并且重写GestureDetector.SimpleOnGestureListener中的onFling()。使用了OverScroller跟踪甩手势。如果用户在甩手势之后达到了内容的边界，应用会显示一个发光效果。

>>>Note: The InteractiveChart sample app displays a chart that you can zoom, pan, scroll, and so on. In the following snippet, mContentRect represents the rectangle coordinates within the view that the chart will be drawn into. At any given time, a subset of the total chart domain and range are drawn into this rectangular area. mCurrentViewport represents the portion of the chart that is currently visible in the screen. Because pixel offsets are generally treated as integers, mContentRect is of the type Rect. Because the graph domain and range are decimal/float values, mCurrentViewport is of the type RectF.


The first part of the snippet shows the implementation of onFling():

```
// The current viewport. This rectangle represents the currently visible
// chart domain and range. The viewport is the part of the app that the
// user manipulates via touch gestures.
private RectF mCurrentViewport =
        new RectF(AXIS_X_MIN, AXIS_Y_MIN, AXIS_X_MAX, AXIS_Y_MAX);

// The current destination rectangle (in pixel coordinates) into which the
// chart data should be drawn.
private Rect mContentRect;

private OverScroller mScroller;
private RectF mScrollerStartViewport;
...
private final GestureDetector.SimpleOnGestureListener mGestureListener
        = new GestureDetector.SimpleOnGestureListener() {
    @Override
    public boolean onDown(MotionEvent e) {
        // Initiates the decay phase of any active edge effects.
        releaseEdgeEffects();
        mScrollerStartViewport.set(mCurrentViewport);
        // Aborts any active scroll animations and invalidates.
        mScroller.forceFinished(true);
        ViewCompat.postInvalidateOnAnimation(InteractiveLineGraphView.this);
        return true;
    }
    ...
    @Override
    public boolean onFling(MotionEvent e1, MotionEvent e2,
            float velocityX, float velocityY) {
        fling((int) -velocityX, (int) -velocityY);
        return true;
    }
};

private void fling(int velocityX, int velocityY) {
    // Initiates the decay phase of any active edge effects.
    releaseEdgeEffects();
    // Flings use math in pixels (as opposed to math based on the viewport).
    Point surfaceSize = computeScrollSurfaceSize();
    mScrollerStartViewport.set(mCurrentViewport);
    int startX = (int) (surfaceSize.x * (mScrollerStartViewport.left -
            AXIS_X_MIN) / (
            AXIS_X_MAX - AXIS_X_MIN));
    int startY = (int) (surfaceSize.y * (AXIS_Y_MAX -
            mScrollerStartViewport.bottom) / (
            AXIS_Y_MAX - AXIS_Y_MIN));
    // Before flinging, aborts the current animation.
    mScroller.forceFinished(true);
    // Begins the animation
    mScroller.fling(
            // Current scroll position
            startX,
            startY,
            velocityX,
            velocityY,
            /*
             * Minimum and maximum scroll positions. The minimum scroll
             * position is generally zero and the maximum scroll position
             * is generally the content size less the screen size. So if the
             * content width is 1000 pixels and the screen width is 200  
             * pixels, the maximum scroll offset should be 800 pixels.
             */
            0, surfaceSize.x - mContentRect.width(),
            0, surfaceSize.y - mContentRect.height(),
            // The edges of the content. This comes into play when using
            // the EdgeEffect class to draw "glow" overlays.
            mContentRect.width() / 2,
            mContentRect.height() / 2);
    // Invalidates to trigger computeScroll()
    ViewCompat.postInvalidateOnAnimation(this);
}
```

当onFling()调用postInvalidateOnAniamtion(),会触发computeScroll()去更新x,y的值。当一个子视图使用scroller动画显示一个滚动时，通常是必须要做的，就像是在例子中一样。


Most views pass the scroller object's x and y position directly to scrollTo(). The following implementation of computeScroll() takes a different approach—it calls computeScrollOffset() to get the current location of x and y. When the criteria for displaying an overscroll "glow" edge effect are met (the display is zoomed in, x or y is out of bounds, and the app isn't already showing an overscroll), the code sets up the overscroll glow effect and calls postInvalidateOnAnimation() to trigger an invalidate on the view:

```
// Edge effect / overscroll tracking objects.
private EdgeEffectCompat mEdgeEffectTop;
private EdgeEffectCompat mEdgeEffectBottom;
private EdgeEffectCompat mEdgeEffectLeft;
private EdgeEffectCompat mEdgeEffectRight;

private boolean mEdgeEffectTopActive;
private boolean mEdgeEffectBottomActive;
private boolean mEdgeEffectLeftActive;
private boolean mEdgeEffectRightActive;

@Override
public void computeScroll() {
    super.computeScroll();

    boolean needsInvalidate = false;

    // The scroller isn't finished, meaning a fling or programmatic pan
    // operation is currently active.
    if (mScroller.computeScrollOffset()) {
        Point surfaceSize = computeScrollSurfaceSize();
        int currX = mScroller.getCurrX();
        int currY = mScroller.getCurrY();

        boolean canScrollX = (mCurrentViewport.left > AXIS_X_MIN
                || mCurrentViewport.right < AXIS_X_MAX);
        boolean canScrollY = (mCurrentViewport.top > AXIS_Y_MIN
                || mCurrentViewport.bottom < AXIS_Y_MAX);

        /*          
         * If you are zoomed in and currX or currY is
         * outside of bounds and you're not already
         * showing overscroll, then render the overscroll
         * glow edge effect.
         */
        if (canScrollX
                && currX < 0
                && mEdgeEffectLeft.isFinished()
                && !mEdgeEffectLeftActive) {
            mEdgeEffectLeft.onAbsorb((int)
                    OverScrollerCompat.getCurrVelocity(mScroller));
            mEdgeEffectLeftActive = true;
            needsInvalidate = true;
        } else if (canScrollX
                && currX > (surfaceSize.x - mContentRect.width())
                && mEdgeEffectRight.isFinished()
                && !mEdgeEffectRightActive) {
            mEdgeEffectRight.onAbsorb((int)
                    OverScrollerCompat.getCurrVelocity(mScroller));
            mEdgeEffectRightActive = true;
            needsInvalidate = true;
        }

        if (canScrollY
                && currY < 0
                && mEdgeEffectTop.isFinished()
                && !mEdgeEffectTopActive) {
            mEdgeEffectTop.onAbsorb((int)
                    OverScrollerCompat.getCurrVelocity(mScroller));
            mEdgeEffectTopActive = true;
            needsInvalidate = true;
        } else if (canScrollY
                && currY > (surfaceSize.y - mContentRect.height())
                && mEdgeEffectBottom.isFinished()
                && !mEdgeEffectBottomActive) {
            mEdgeEffectBottom.onAbsorb((int)
                    OverScrollerCompat.getCurrVelocity(mScroller));
            mEdgeEffectBottomActive = true;
            needsInvalidate = true;
        }
        ...
    }
```

Here is the section of the code that performs the actual zoom:

```
// Custom object that is functionally similar to Scroller
Zoomer mZoomer;
private PointF mZoomFocalPoint = new PointF();
...

// If a zoom is in progress (either programmatically or via double
// touch), performs the zoom.
if (mZoomer.computeZoom()) {
    float newWidth = (1f - mZoomer.getCurrZoom()) *
            mScrollerStartViewport.width();
    float newHeight = (1f - mZoomer.getCurrZoom()) *
            mScrollerStartViewport.height();
    float pointWithinViewportX = (mZoomFocalPoint.x -
            mScrollerStartViewport.left)
            / mScrollerStartViewport.width();
    float pointWithinViewportY = (mZoomFocalPoint.y -
            mScrollerStartViewport.top)
            / mScrollerStartViewport.height();
    mCurrentViewport.set(
            mZoomFocalPoint.x - newWidth * pointWithinViewportX,
            mZoomFocalPoint.y - newHeight * pointWithinViewportY,
            mZoomFocalPoint.x + newWidth * (1 - pointWithinViewportX),
            mZoomFocalPoint.y + newHeight * (1 - pointWithinViewportY));
    constrainViewport();
    needsInvalidate = true;
}
if (needsInvalidate) {
    ViewCompat.postInvalidateOnAnimation(this);
}
```

This is the computeScrollSurfaceSize() method that's called in the above snippet. It computes the current scrollable surface size, in pixels. For example, if the entire chart area is visible, this is simply the current size of mContentRect. If the chart is zoomed in 200% in both directions, the returned size will be twice as large horizontally and vertically.

```
private Point computeScrollSurfaceSize() {
    return new Point(
            (int) (mContentRect.width() * (AXIS_X_MAX - AXIS_X_MIN)
                    / mCurrentViewport.width()),
            (int) (mContentRect.height() * (AXIS_Y_MAX - AXIS_Y_MIN)
                    / mCurrentViewport.height()));
}
```

For another example of scroller usage, see the source code for the ViewPager class. It scrolls in response to flings, and uses scrolling to implement the "snapping to page" animation.
