---
category: Android
date: 2013-05-10
layout: post
title: 当ViewPager嵌套在ScrollView/ListView里时，手势冲突如何处理？
---

有时我们需要将ViewPager嵌套在其他已经含有手势动作的ViewGroup里,如ScrollView,ListView时，会造成手势冲突，如表现为ViewPager向左划时，不小心向上移动了一点距离，ViewPager立刻回弹到原始位置。

主要问题出在ScrollView/ListView作为ViewPager的ParentView，会先接受到触摸信息，而且他们对上下滑动是会做出拦截动作，并接管触摸信息的向下传递，导致ViewPager滑动异常。
先看一种[解决方式](http://justwyy.iteye.com/blog/1567390):

```
public class ScrollViewExtend extends ScrollView {  
    private float xDistance, yDistance, xLast, yLast;  
    public ScrollViewExtend(Context context, AttributeSet attrs) {  
        super(context, attrs);  
    }  
    @Override  
    public boolean onInterceptTouchEvent(MotionEvent ev) {  
        switch (ev.getAction()) {  
            case MotionEvent.ACTION_DOWN:  
                xDistance = yDistance = 0f;  
                xLast = ev.getX();  
                yLast = ev.getY();  
                break;  
            case MotionEvent.ACTION_MOVE:  
                final float curX = ev.getX();  
                final float curY = ev.getY();             
                xDistance += Math.abs(curX - xLast);  
                yDistance += Math.abs(curY - yLast);  
                xLast = curX;  
                yLast = curY;  
                if(xDistance > yDistance){  
                    return false;  
                }    
        }  
        return super.onInterceptTouchEvent(ev);  
    }  
}   
```

这种方式的确可以解决这个问题，但是其实Google已经提供了一个函数来解决ParentView与ChildView手势冲突的问题。

```
public void requestDisallowInterceptTouchEvent(boolean disallowIntercept)
```

由ViewPager在OnTouch/onInterceptTouchEvent，dispatchTouchEvent中调用即可。
