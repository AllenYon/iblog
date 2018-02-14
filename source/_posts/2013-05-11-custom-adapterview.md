---
category: Android
date: 2013-05-11
layout: post
title: 创建属于你自己的3D ListView - Part 1(译)

---
原文连接 [Making your own 3D list - Part 1](http://developer.sonymobile.com/2010/05/20/android-tutorial-making-your-own-3d-list-part-1/)

本文是教你如何在安卓应用中做出很酷效果的ListView的教材第一章，我叫Anders Ericson，主要从事UI工作，在Timescape应用中有个就是我做的，你可以在Xperia X10 mini 和 Xperia X10 mini pro中找到.当用户尝试使用一个App时，UI是他们第一个关注的东西,所以我决定做一个教材，使得其他任何安卓开发者可以创建属于他们自己的ListView,类似与在Timescape中的3D感觉和物理特性(dynamics).

在教材的第一章，我们将创建一个很基础的list，然后在接下去的两章中，越来越多的功能和特性将添加进来。我同时也会教你如何使用list的基本构造，然后改变它使得无论如何工作的都使你的应用做到最好。下面那个链接是第一章的源码，已经为你准备好构建在如Eclipse这样的IDE中。同时不要忘了下载Sony Ericsson Tutorials 应用从Android Market中，你可以尝试使用同样的应用在教材的每一步。我很期待看到你的评论和问题。

Android中标准的ListView已经支持了很多东西，并且包括了绝大多数的你可以想到的用户事件。但是listview看起来太平淡了，当你想继承它并且做很多事情时，都发现做不到，然后无疾而终。标准ListView的另一个缺点是缺乏好的物理特性（和改变它的能力）。因此，如果你要你的UI看起来不那么普通的话，你要做的仅仅是实现你自己的View.

因为在一篇文章中写完会有太多的代码，所以我准备分为3部分。第一部分（就是这章）先创建一个基本的List，有太多的东西要去包括，但我想让我们更关注它们在后面几章.第二章，我们将看到listview的外貌改变和一些类3D图像算法。最后一章，我们将改变list的行为并且添加一些物理动力学进去，一些非常能提升外观和感觉的东西。

虽然这里用到的技术和在X10 Mini上用到的一样，当这篇教材的目的不是仅仅拷贝一个看起来很特殊的list，而是教你实现自己的listview。我很确定你肯定有很多关于你的listview该看起来怎么样，它的行为和它用来干嘛的主意。

### Hello AdapterView

当我们瞄准一个list（能显示其他view）,我们需要继承ViewGroup,更合适的是AdapterView。（原因，或者更多的原因我们不从AbsListView继承，是因为它不允许我们在list上做有活力的效果）。那么，让我们从开始建立安卓项目并且创建一个继承自AdapterView<Adapter> 的 MyListView 开始吧，AdapterView有四个抽象函数我们需要去实现:

- getAdapter()
- setAdapter()
- getSelectedView()
- setSelection()

getAdapter()和setAdapter()直接实现，其他两个现在先仅仅抛出异常.

```
public class MyListView extends AdapterView {
    /** The adapter with all the data */
    private Adapter mAdapter;
    public MyListView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
    @Override
    public void setAdapter(Adapter adapter) {
        mAdapter = adapter;
        removeAllViewsInLayout();
        requestLayout();
    }
    @Override
    public Adapter getAdapter() {
        return mAdapter;
    }
    @Override
    public void setSelection(int position) {
        throw new UnsupportedOperationException("Not supported");
    }
    @Override
    public View getSelectedView() {
        throw new UnsupportedOperationException("Not supported");
    }
}
```
这里唯一值得注意的是setAdapter方法。当我们获得一个新的Adapter我们移除之前所有的视图，然后请求一个布局，并且按照adapter放置视图。如果我们现在创建一个activity和带有假数据的adapter，然后使用新视图，我们将得不到任何东西，在屏幕上。这是因为如果我们要在屏幕上得到一些东西，我们需要重写onLayout()方法。

#### 显示我们的第一个视图

在[onLayout( )](http://developer.android.com/reference/android/view/View.html#onLayout(boolean,%20int,%20int,%20int,%20int) )中，我们从adpater中获取视图，并且添加他们作为一个子视图。

```
@Override
protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
    super.onLayout(changed, left, top, right, bottom);
    // if we don't have an adapter, we don't need to do anything
    if (mAdapter == null) {
        return;
    }
    if (getChildCount() == 0) {
        int position = 0;
        int bottomEdge = 0;
        while (bottomEdge &lt; getHeight() &amp;&amp; position &lt; mAdapter.getCount()) {
            View newBottomChild = mAdapter.getView(position, null, this);
            addAndMeasureChild(newBottomChild);
            bottomEdge += newBottomChild.getMeasuredHeight();
            position++;
        }
    }
    positionItems();
}
```
So，What happends hers? 首先执行父调用和空值检查，然后开始真正有用的代码。如果我们还没有添加任何子节点，那么我们就开始那么做。这个while循环直到我们添加足够多的视图覆盖整个屏幕为止。当我们从adapter获得一个视图，我们就把它添加为一个子节点，然后我们需要测量(measure)它，为的是得到它的正确尺寸。当我们添加完所有的视图，我们把它们放置到正确的位置.


```
/**
 * Adds a view as a child view and takes care of measuring it
 *
 * @param child The view to add
 */
private void addAndMeasureChild(View child) {
    LayoutParams params = child.getLayoutParams();
    if (params == null) {
        params = new LayoutParams(LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT);
    }
    addViewInLayout(child, -1, params, true);
    int itemWidth = getWidth();
    child.measure(MeasureSpec.EXACTLY | itemWidth, MeasureSpec.UNSPECIFIED);
}
/**
 * Positions the children at the &quot;correct&quot; positions
 */
private void positionItems() {
    int top = 0;
    for (int index = 0; index < getChildCount(); index++) {
        View child = getChildAt(index);
        int width = child.getMeasuredWidth();
        int height = child.getMeasuredHeight();
        int left = (getWidth() - width) / 2;
        child.layout(left, top, left + width, top + height);
        top += height;
    }
}

```
这些代码比较简单，自解释，所以我不准备过多描述。虽然我在测量子视图时走了点捷径，但是这些代码在大部分情况下是工作良好的。positioItems()从top（0)开始，然后布局这些子视图，一个挨着一个，没有任何padding。值得注意的是，我们忽略了list可能有的padding.

### 滑动
如果现在运行这些代码，我们在屏幕上得到一些东西了。然而，这一点交互的感觉也没有（interactive）。当我们触摸屏幕时它不会滑动，我们也不能点击任何item.要让触摸生效，我们需要重写 onTouchEvent()。

仅仅使其滑动的触摸逻辑是十分简单的，当我们得到一个按下事件，我们保存下按下事件的位置，和list的位置。我们将使用第一个item的top作为list的位置(position)，当我们获得一个移动事件，我们计算与按下事件的距离，然后根据开始位置和当前位置的距离重新安置list。

```
@Override
public boolean onTouchEvent(MotionEvent event) {
    if (getChildCount() == 0) {
        return false;
    }
    switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN:
            mTouchStartY = (int)event.getY();
            mListTopStart = getChildAt(0).getTop();
            break;
        case MotionEvent.ACTION_MOVE:
            int scrolledDistance = (int)event.getY() - mTouchStartY;
            mListTop = mListTopStart + scrolledDistance;
            requestLayout();
            break;
        default:
            break;
    }
    return true;
}
```
现在list的位置已经由mListTop确定下来了，无论何时它改变了，我实际上只需要请求重新布局就好了。我们之前实现的positionItem()总是从0开始布局。现在我们需要改变它，以至于它能从mListTop开始。

如果我们现在就去尝试滑动，它会表现的不错，但我们还是能发现一些明显的问题在我们的list中。首先，滑动没有限制，这样我们可以把所有的item滑出屏幕外。我们需要几种限制检查方式去阻止我们那样做。第二，如果我们往下滑，我们只能看到之前已经显示了的item。没有新的item显示出来，即使adapter包含了更多的item。我们现在就修复第二个问题，然后把第一个问题放到下一章再说。

#### 处理所有的item
之所以在我们滑动式没有新的item出现，原因在于onLayout()这段代码。这段代码只会在没有一个视图未被添加的时候才会添加视图。**(The code there only adds views if no views haven't already been added.)** list组件的必要条件之一就是无论它是有10个item还是10,000个item，它都应该工作正常。记住了这点，我们就不能简单地在开始的时候把所有的item从adapter拿出，然后全部添加为子视图，我们需要确定我们能高效地处理这些视图.为了做到高效，我们只需要持有list可见那部分的子视图。如果我们能维持一小块视图的缓存，最好不过了，我们能让adpater重用这些而不是反复的从xml中inflate。

处理这些问题的地方在onLayout()。新版本的onLayout()看起来像是这样的：

```
@Override
protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
    super.onLayout(changed, left, top, right, bottom);
    // if we don't have an adapter, we don't need to do anything
    if (mAdapter == null) {
        return;
    }
    if (getChildCount() == 0) {
        mLastItemPosition = -1;
        fillListDown(mListTop);
    } else {
        int offset = mListTop + mListTopOffset - getChildTop(getChildAt(0));
        removeNonVisibleViews(offset);
        fillList(offset);
    }
    positionItems();
    invalidate();
}
```
fillListDown()和之前的while循环或多或少是同一个东西。也添加了一个方法做同样的事情，只是它从顶部开始添加，叫做fillListUp()。他们都叫做fillList()。removeNonVisibleViews()把超出可见范围（超出顶部和底部）的视图移除掉。有两个变量被添加进来，为了跟踪视图，确保与adpater连接时在正确的位置：mFirstItemPosition和mLastPosition。它们是当前可见视图的在adpater中第一个和最后一个位置。每当我们移除或者添加一个视图，它都会更新。当滑动中的list滑到第一个可见item的顶部时，我们也需要更新list的位置，每当我们在顶部添加一个新视图或者移除顶部视图时。

为了弥补positionItems()将上下移动list的事实，我们需要让removeVisibleViews()和fillList()知道到底list移动了多少。这就是偏移变量(offset variable)。否则我们可能不会移除那些执行positionItems()时离开可见区域的items,或者我们可能忘了去添加将变为可见状态的items。当mListTop被定义为第一个item的顶部，即使它不可见，我们也需要跟踪当前第一个可见item到之前第一个item位置的距离。

如果你之前有实现过一个adapter，你应该知道检测并且使用convertView参数，代替每次从xml中inflat一个新的视图，这种方式来提升性能。现在我们正实现它的另一面，那就是我们将调用getView()而不是实现它，然后我们需要确定我们让adapter重用views。我们需要的是可以重用视图的缓存。标准的ListView支持不同种类的视图，但是现在我们假设所有的item-views是一样的。我们将只使用LinkedList来作为缓存容器.每当我们移除一个子视图时(在 removeNonVisibleViews())我们添加一个到缓存中，每当我们向adapter调用getView是（在fillListDown（）和fillListUp（））时，我们返回一个缓存视图（如果有的话）作为convertView。

###点击和长按

想让list变的有用的，那么就需要让在list中的所有item可以被点击。AdapterView实现了设置OnItemClickListener和OnItemLongClickLstener的方法，我们需要确认在合适的时间调用这些监听器。为了支持点击item的视图，我们需要做三件事情
1. 检测一个点击时间
2. 找出被点击的item
3. 调用监听器（如果设置了），携带正确的参数
那么从顶部开始，并且实现一个点击检测吧。

Android提供的[GestureDestector](http://developer.android.com/reference/android/view/GestureDetector.html)类可以使用，但是我们事实上我不建议使用。其中一个原因是我发现它相当不可靠，特别是长按手势和甩手势。另一个原因是如果你委托手势检测到另一类的话，你可能无法追踪触摸状态，并且想要知道更多关于触摸状态的信息。

首先，定义一些触摸状态

```
/** User is not touching the list */
private static final int TOUCH_STATE_RESTING = 0;
/** User is touching the list and right now it's still a "click" */
private static final int TOUCH_STATE_CLICK = 1;
/** User is scrolling the list */
private static final int TOUCH_STATE_SCROLL = 2;
```
我们事前已经重写过了OnTouchEvent()，现在我们将添加一些代码来处理新的状态。

```
@Override
 public boolean onTouchEvent(final MotionEvent event) {
     if (getChildCount() == 0) {
         return false;
     }
     switch (event.getAction()) {
         case MotionEvent.ACTION_DOWN:
             startTouch(event);
             break;
         case MotionEvent.ACTION_MOVE:
             if (mTouchState == TOUCH_STATE_CLICK) {
                 startScrollIfNeeded(event);
             }
             if (mTouchState == TOUCH_STATE_SCROLL) {
                 scrollList((int)event.getY() - mTouchStartY);
             }
             break;
         case MotionEvent.ACTION_UP:
             if (mTouchState == TOUCH_STATE_CLICK) {
                 clickChildAt((int)event.getX(), (int)event.getY());
             }
             endTouch();
             break;
         default:
             endTouch();
             break;
     }
     return true;
 }
```
在添加代码前，这段的大部分是相同的。处理按下事件的代码被重构为一个方法,startTouch()。同时我们将状态改变为TOUCH_STATE_CLICK。现在我们不知道用户是要点击或者滑动视图，但是直到我们识别为一个滑动，我们就处理它为一个点击。

滑动的识别是在startScrollIfNeeded()中处理的，在滑动事件函数（move events）中被调用的那个。比较当前触摸坐标和按下事件的坐标，如果用户手指滑动的距离超出了临界值，那么就改变状态为TOUCH_STATE_SCROLL。我使用10px的临界值，效果还不错。你也可以使用ViewConfiguraion类然后调用getScaledTouchSlop()获取的值作为临界值。

If we are scrolling,then it's the same code as without the additions for click/state,though it's now in a separate method.(眼花了).list的顶部被修改，布局被要求重新布置list。

为了支持点击事件，我们也需要处理好ACTION_UP事件，我们还需要确认我们把ACTION_CANCEL和ACTION_OUTSIDE事件区分开来。除了按下和移动事件，其他所有的事件我们都需要重置触摸状态为TOUCH_STATE_RESTING,已经在endTouch()中调用了，但是唯独ACTION_UP我们需要调用点击监听器。当然，我们只能调用点击监听器，如果我们是在点击状态而不是滑动状态。

```
private void clickChildAt(final int x, final int y) {
    final int index = getContainingChildIndex(x, y);
    if (index != INVALID_INDEX) {
        final View itemView = getChildAt(index);
        final int position = mFirstItemPosition + index;
        final long id = mAdapter.getItemId(position);
        performItemClick(itemView, position, id);
    }
}
private int getContainingChildIndex(final int x, final int y) {
    if (mRect == null) {
        mRect = new Rect();
    }
    for (int index = 0; index &lt; getChildCount(); index++) {
        getChildAt(index).getHitRect(mRect);
        if (mRect.contains(x, y)) {
            return index;
        }
    }
    return INVALID_INDEX;
}
```
clickChildAt() is responsible for calling the listener (if any) with the position for the child at the specified coordinates. To find the correct view clickChildAt() uses getContainingChildIndex() which loops through the child views and for each view checks if the coordinates given are contained within the hit-rect of the view or not.

When we have click handling in place, adding a check for long press is quite simple. A convenient way of checking for a long press is to create a Runnable that calls the long press listener. Then whenever we get a down event we post this Runnable with a delay on the view). Whenever we get an up event or when we switch to scrolling, we know it’s not going to be a long press any more so then we simply remove the Runnable by calling [removeCallbacks()](http://developer.android.com/reference/android/view/View.html#removeCallbacks(java.lang.Runnable)). How long to check for a long-press is up to you and the specific view you are implementing, but if it’s not something special, it’s a good idea to use the same delay as the rest of the system. Use [ViewConfiguration.getLongPressTimeout()](http://developer.android.com/reference/android/view/ViewConfiguration.html#getLongPressTimeout()) to get it.

In order to be able to scroll the list even if the child views respond to touch events, you need to intercept touch events when you want to start scrolling. This is done by overriding [onInterceptTouchEvent()](http://developer.android.com/reference/android/view/ViewGroup.html#onInterceptTouchEvent(android.view.MotionEvent))  which lets us monitor all the touch events passed to our children and lets us, if we want to, intercept the touch events at any point.

```
@Override
public boolean onInterceptTouchEvent(final MotionEvent event) {
    switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN:
            startTouch(event);
            return false;
        case MotionEvent.ACTION_MOVE:
            return startScrollIfNeeded(event);
        default:
            endTouch();
            return false;
    }
}
```

The implementation of onInterceptTouchEvent() looks quite like onTouchEvent(). If we get a down event, we save the position of the list then return false to let the motion event pass to wherever it’s going. When we get move events, we check if we’ve moved far enough for this to be counted as a scroll move. If we have moved enough, we set the state to scrolling and then we return true to intercept future events.

####To be continued…
What we’ve made so far is a very simple list. We handle views efficiently and a user can scroll it and click and long press items. If we stop here however, we could just as well have used ListView (though we’ve learned a bit about implementing a view group). In the next part of this tutorial, we will take a look at canvas transformations and give the list a more 3D look and after that we will look into the dynamics of the list like bounce and fling effects.

[[Download] 3D List sample project – Part 1 (31kb)](http://developer.sonymobile.com/downloads/code-example-module/3d-list-sample-project-part-1/)
