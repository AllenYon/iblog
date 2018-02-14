---
category: Android
date: 2013-06-19
layout: post
title: ViewPager TouchEvent流程源码分析
---

由于ViewPager是继承ViewGroup的，所以它需要重写`onInterceptTouchEvent(MotionEvent ev)`函数，来控制父控件与子控件触摸事件传递流程

[知识点： onInterceptTouchEvent和onTouchEvent调用时序](http://blog.csdn.net/ddna/article/details/5473293) 网友总结的知识点

[Google官方指南：Managing Touch Events in a ViewGroup](http://developer.android.com/training/gestures/viewgroup.html)

>首先吐槽下Google在触摸事件处理流程上的混乱，回调函数中逻辑都乱成浆糊了，ViewPager中仅onInterceptTouchEvent()和onTouchEvent()两个函数就长达270行，中间变量也多的发指！希望将来的能有所改善,类似iOS，清晰回调down，move，cancel等事件

首先分析public boolean onInterceptTouchEvent(MotionEvent ev)函数，顾名思义是拦截触摸事件的回调函数：

- 返回false: 表示父控件不拦截触摸事件，而将后续的所有事件传递给子控件
- 返回true:  表示父控件拦截了子控件接收触摸事件的机会，而将后续事件转交给了自己的onTouchEvent()函数去处理。

ViewPager使用了两个重要的变量

1. mIsBeingDragged：如果该变量为true，onInterceptionTouchEvent()会立刻返回true，表明"我现在准备要把整个父控件拖动起来了，子控件们都不可能接收到Click,Touch,DoubleClick等触摸事件了"
2. mIsUnableToDrag: 与mIsBeingDragged相对应，表示父控件的触摸手势被判断为不应该拖动，比如Y轴滑动的距离大于了TouchSlop-->[Google术语](http://developer.android.com/training/gestures/viewgroup.html),向X轴触摸滑动得角度不足等等情况（具体数据是 xDiff*0.5>yDiff ）.

 		// Nothing more to do here if we have decided whether or not we
        // are dragging.
        if (action != MotionEvent.ACTION_DOWN) {
            if (mIsBeingDragged) {
                if (DEBUG) Log.v(TAG, "Intercept returning true!");
                return true;
            }
            if (mIsUnableToDrag) {
                if (DEBUG) Log.v(TAG, "Intercept returning false!");
                return false;
            }
        }


如果用伪代码来写onInteceptTouchEvent()的话大概是这样的：

```
{
	if(action_cancel||action_up)
	return	false;

	if(!action_down)
		if (mIsBeingDragged)
			return true;
		if (mIsUnableToDrag)
			return false;
	switch(action){
		case move:
			if(可以滑动&&不是从屏幕边缘开始触摸滑动的){
				mIsUnableToDrag=ture;
				return false;
			}
			if(xDiff>mTouchSlop && xDiff*0.5>yDiff){
				mIsBeingDragged=true;
			}else if(yDiff>mTouchSlop){
				mIsUnableToDrag=true;
			}
			if(mIsBeingDragged){
				//拖动滑行x
				performDrag(x);
			}
			break；
		case down：
			初始化变量
			mIsUnableToDrag=false;

			//当ViewPager 正在左右页面切换滑动时，我们按下去之后  直接判断为开始滑动状态，
			//mCloseEnough默认值为2(dip),意思是Scroller可能已经快要结束它的动画了，必须要结束X坐标和当前X坐标相差大于2(dip)以上，才认为是还在滑动中
			mScroller.computeScrollOffset();
			if(mScrollState==SCROLL_STATE_SETTLING&&Math.abs(mScroller.getFinalX()-mScroller.getCurrX())>mCloseEnough){
				mIsBeingDragged=true;
			}else{
				mIsBeingDragged=false;
			}
			break；
	}


	return mIsBeingDragged;
}
```

好了，我们现在可以假设mIsBeingDragged==true,那么ViewPager就开始回调onTouchEvent(MotionEvent ev)()

```
public boolean onTouchEvent(MotionEvent ev) {
	if (ev.getAction() == MotionEvent.ACTION_DOWN && ev.getEdgeFlags() != 0) {
		//按下时触摸到的屏幕边缘，则返回false
		return false;
    }

    switch(action){
    	case down:
    		变量初始化
    		mIsBeingDragged=true;
    		break;
    	case move:
    		if(!mIsBeingDragged){
    			if(xDiff>mTouchSlop && xDiff > yDiff){
    				mIsBeingDragged=true;
    			}
    		}
    		//Not else! Note that mIsBeingDragged can be set above.
    		if(mIsBeingDragged){
    			 // 页面逻辑
    		}
    		break;
    	case up:
    		if(mIsBeingDragged){
    			 // 页面逻辑
    			 endDrag();
    		}
    		break;
    	case cancel:
    		if(mIsBeingDragged){
    			 // 页面逻辑
    			 endDrag();
    		}
    		break;
    }

    // 可以看出，除了一些特殊情况外，onTouchEvent() 是必定返回true的，页面逻辑里实现了拖动和甩动等效果。
    return true;
}
```

##总结
---
假如我们要定制一个ViewGroup，类似与ViewPager,需要做手势判断的父控件，我们就有可能需要重写OnInteceptTouchEvent()，在move事件里判断手势，并决定是否拦截，在onTouchEvent里对整个ViewGroup做手势动画。


##完整源码
---

 	<!-- java ->
    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        /*
         * This method JUST determines whether we want to intercept the motion.
         * If we return true, onMotionEvent will be called and we do the actual
         * scrolling there.
         */
        final int action = ev.getAction() & MotionEventCompat.ACTION_MASK;

        // Always take care of the touch gesture being complete.
        if (action == MotionEvent.ACTION_CANCEL || action == MotionEvent.ACTION_UP) {
            // Release the drag.
            if (DEBUG) Log.v(TAG, "Intercept done!");
            mIsBeingDragged = false;
            mIsUnableToDrag = false;
            mActivePointerId = INVALID_POINTER;
            if (mVelocityTracker != null) {
                mVelocityTracker.recycle();
                mVelocityTracker = null;
            }
            return false;
        }

        // Nothing more to do here if we have decided whether or not we
        // are dragging.
        if (action != MotionEvent.ACTION_DOWN) {
            if (mIsBeingDragged) {
                if (DEBUG) Log.v(TAG, "Intercept returning true!");
                return true;
            }
            if (mIsUnableToDrag) {
                if (DEBUG) Log.v(TAG, "Intercept returning false!");
                return false;
            }
        }

        switch (action) {
            case MotionEvent.ACTION_MOVE: {
                /*
                 * mIsBeingDragged == false, otherwise the shortcut would have caught it. Check
                 * whether the user has moved far enough from his original down touch.
                 */

                /*
                * Locally do absolute value. mLastMotionY is set to the y value
                * of the down event.
                */
                final int activePointerId = mActivePointerId;
                if (activePointerId == INVALID_POINTER) {
                    // If we don't have a valid id, the touch down wasn't on content.
                    break;
                }

                final int pointerIndex = MotionEventCompat.findPointerIndex(ev, activePointerId);
                final float x = MotionEventCompat.getX(ev, pointerIndex);
                final float dx = x - mLastMotionX;
                final float xDiff = Math.abs(dx);
                final float y = MotionEventCompat.getY(ev, pointerIndex);
                final float yDiff = Math.abs(y - mInitialMotionY);
                if (DEBUG)
                    Log.v(TAG, "Moved x to " + x + "," + y + " diff=" + xDiff + "," + yDiff);

                if (dx != 0 && !isGutterDrag(mLastMotionX, dx) &&
                        canScroll(this, false, (int) dx, (int) x, (int) y)) {
                    // Nested view has scrollable area under this point. Let it be handled there.
                    mLastMotionX = x;
                    mLastMotionY = y;
                    mIsUnableToDrag = true;
                    return false;
                }
                // x轴滑动大于 mTouchSlop  x轴滑动大于y轴滑动两倍
                if (xDiff > mTouchSlop && xDiff * 0.5f > yDiff) {
                    if (DEBUG) Log.v(TAG, "Starting drag!");
                    mIsBeingDragged = true;
                    setScrollState(SCROLL_STATE_DRAGGING);

                    // mLastMotionX 修正
                    mLastMotionX = dx > 0 ? mInitialMotionX + mTouchSlop :
                            mInitialMotionX - mTouchSlop;
                    mLastMotionY = y;
                    setScrollingCacheEnabled(true);
                } else if (yDiff > mTouchSlop) {
                    // The finger has moved enough in the vertical
                    // direction to be counted as a drag...  abort
                    // any attempt to drag horizontally, to work correctly
                    // with children that have scrolling containers.
                    if (DEBUG) Log.v(TAG, "Starting unable to drag!");
                    mIsUnableToDrag = true;
                }
                if (mIsBeingDragged) {
                    // Scroll to follow the motion event
                    if (performDrag(x)) {
                        ViewCompat.postInvalidateOnAnimation(this);
                    }
                }
                break;
            }

            case MotionEvent.ACTION_DOWN: {
                /*
                 * Remember location of down touch.
                 * ACTION_DOWN always refers to pointer index 0.
                 */
                mLastMotionX = mInitialMotionX = ev.getX();
                mLastMotionY = mInitialMotionY = ev.getY();
                mActivePointerId = MotionEventCompat.getPointerId(ev, 0);
                mIsUnableToDrag = false;

                mScroller.computeScrollOffset();
                if (mScrollState == SCROLL_STATE_SETTLING &&
                        Math.abs(mScroller.getFinalX() - mScroller.getCurrX()) > mCloseEnough) {
                    // Let the user 'catch' the pager as it animates.
                    mScroller.abortAnimation();
                    mPopulatePending = false;
                    populate();
                    mIsBeingDragged = true;
                    setScrollState(SCROLL_STATE_DRAGGING);
                } else {
                    completeScroll(false);
                    mIsBeingDragged = false;
                }

                if (DEBUG) Log.v(TAG, "Down at " + mLastMotionX + "," + mLastMotionY
                        + " mIsBeingDragged=" + mIsBeingDragged
                        + "mIsUnableToDrag=" + mIsUnableToDrag);
                break;
            }

            case MotionEventCompat.ACTION_POINTER_UP:
                onSecondaryPointerUp(ev);
                break;
        }

        if (mVelocityTracker == null) {
            mVelocityTracker = VelocityTracker.obtain();
        }
        mVelocityTracker.addMovement(ev);

        /*
         * The only time we want to intercept motion events is if we are in the
         * drag mode.
         */
        return mIsBeingDragged;
    }

    @Override
    public boolean onTouchEvent(MotionEvent ev) {
        if (mFakeDragging) {
            // A fake drag is in progress already, ignore this real one
            // but still eat the touch events.
            // (It is likely that the user is multi-touching the screen.)
            return true;
        }

        if (ev.getAction() == MotionEvent.ACTION_DOWN && ev.getEdgeFlags() != 0) {
            // Don't handle edge touches immediately -- they may actually belong to one of our
            // descendants.
            return false;
        }

        if (mAdapter == null || mAdapter.getCount() == 0) {
            // Nothing to present or scroll; nothing to touch.
            return false;
        }

        if (mVelocityTracker == null) {
            mVelocityTracker = VelocityTracker.obtain();
        }
        mVelocityTracker.addMovement(ev);

        final int action = ev.getAction();
        boolean needsInvalidate = false;

        switch (action & MotionEventCompat.ACTION_MASK) {
            case MotionEvent.ACTION_DOWN: {

                mScroller.abortAnimation();
                mPopulatePending = false;
                populate();
                mIsBeingDragged = true;
                setScrollState(SCROLL_STATE_DRAGGING);

                // Remember where the motion event started
                mLastMotionX = mInitialMotionX = ev.getX();
                mLastMotionY = mInitialMotionY = ev.getY();
                mActivePointerId = MotionEventCompat.getPointerId(ev, 0);
                break;
            }
            case MotionEvent.ACTION_MOVE:
                if (!mIsBeingDragged) {
                    final int pointerIndex = MotionEventCompat.findPointerIndex(ev, mActivePointerId);
                    final float x = MotionEventCompat.getX(ev, pointerIndex);
                    final float xDiff = Math.abs(x - mLastMotionX);
                    final float y = MotionEventCompat.getY(ev, pointerIndex);
                    final float yDiff = Math.abs(y - mLastMotionY);
                    if (DEBUG)
                        Log.v(TAG, "Moved x to " + x + "," + y + " diff=" + xDiff + "," + yDiff);
                    if (xDiff > mTouchSlop && xDiff > yDiff) {
                        if (DEBUG) Log.v(TAG, "Starting drag!");

                        mIsBeingDragged = true;
                        mLastMotionX = x - mInitialMotionX > 0 ? mInitialMotionX + mTouchSlop :
                                mInitialMotionX - mTouchSlop;
                        mLastMotionY = y;
                        setScrollState(SCROLL_STATE_DRAGGING);
                        setScrollingCacheEnabled(true);
                    }
                }
                // Not else! Note that mIsBeingDragged can be set above.
                if (mIsBeingDragged) {
                    // Scroll to follow the motion event
                    final int activePointerIndex = MotionEventCompat.findPointerIndex(
                            ev, mActivePointerId);
                    final float x = MotionEventCompat.getX(ev, activePointerIndex);
                    needsInvalidate |= performDrag(x);
                }
                break;
            case MotionEvent.ACTION_UP:
                if (mIsBeingDragged) {

                    final VelocityTracker velocityTracker = mVelocityTracker;
                    velocityTracker.computeCurrentVelocity(1000, mMaximumVelocity);
                    int initialVelocity = (int) VelocityTrackerCompat.getXVelocity(
                            velocityTracker, mActivePointerId);
                    mPopulatePending = true;
                    final int width = getWidth();
                    final int scrollX = getScrollX();
                    final ItemInfo ii = infoForCurrentScrollPosition();
                    final int currentPage = ii.position;
                    final float pageOffset = (((float) scrollX / width) - ii.offset) / ii.widthFactor;
                    final int activePointerIndex =
                            MotionEventCompat.findPointerIndex(ev, mActivePointerId);
                    final float x = MotionEventCompat.getX(ev, activePointerIndex);

                    final int totalDelta = (int) (x - mInitialMotionX);
                    int nextPage = determineTargetPage(currentPage, pageOffset, initialVelocity,
                            totalDelta);

                    //Link Add
                    if (nextPage == currentPage) {
                        if (mOnPopViewListener != null) {
                            mOnPopViewListener.onPoped();
                        }
                    }
                    setCurrentItemInternal(nextPage, true, true, initialVelocity);


                    mActivePointerId = INVALID_POINTER;
                    endDrag();
                    needsInvalidate = mLeftEdge.onRelease() | mRightEdge.onRelease();
                }
                break;
            case MotionEvent.ACTION_CANCEL:
                if (mIsBeingDragged) {
                    scrollToItem(mCurItem, true, 0, false);
                    mActivePointerId = INVALID_POINTER;
                    endDrag();
                    needsInvalidate = mLeftEdge.onRelease() | mRightEdge.onRelease();
                }
                break;
            case MotionEventCompat.ACTION_POINTER_DOWN: {
                final int index = MotionEventCompat.getActionIndex(ev);
                final float x = MotionEventCompat.getX(ev, index);
                mLastMotionX = x;
                mActivePointerId = MotionEventCompat.getPointerId(ev, index);
                break;
            }
            case MotionEventCompat.ACTION_POINTER_UP:
                onSecondaryPointerUp(ev);
                mLastMotionX = MotionEventCompat.getX(ev,
                        MotionEventCompat.findPointerIndex(ev, mActivePointerId));
                break;
        }
        if (needsInvalidate) {
            ViewCompat.postInvalidateOnAnimation(this);
        }
        return true;
    }

    private boolean performDrag(float x) {
        boolean needsInvalidate = false;

        final float deltaX = mLastMotionX - x;
        mLastMotionX = x;

        float oldScrollX = getScrollX();
        float scrollX = oldScrollX + deltaX;
        final int width = getWidth();

        float leftBound = width * mFirstOffset;

        // Link modify
        // float rightBound = width * mLastOffset;
        float rightBound = width * mCurItem;
        boolean leftAbsolute = true;
        boolean rightAbsolute = true;

        final ItemInfo firstItem = mItems.get(0);
        final ItemInfo lastItem = mItems.get(mItems.size() - 1);
        if (firstItem.position != 0) {
            leftAbsolute = false;
            leftBound = firstItem.offset * width;
        }

        // 注释
		// if (lastItem.position != mAdapter.getCount() - 1) {
		//		rightAbsolute = false;
		//	 	rightBound = lastItem.offset * width;
		//	}

        if (scrollX < leftBound) {
            if (leftAbsolute) {
                float over = leftBound - scrollX;
                needsInvalidate = mLeftEdge.onPull(Math.abs(over) / width);
            }
            scrollX = leftBound;
        } else if (scrollX > rightBound) {
            if (rightAbsolute) {
                float over = scrollX - rightBound;

                //Link Modify
				// needsInvalidate = mRightEdge.onPull(Math.abs(over) / width);
                needsInvalidate = true;
            }
            scrollX = rightBound;
        }
        mLastMotionX += scrollX - (int) scrollX;

        scrollTo((int) scrollX, getScrollY());
        pageScrolled((int) scrollX);

        return needsInvalidate;
    }
