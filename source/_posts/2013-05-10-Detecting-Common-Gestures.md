---
category: Android
date: 2013-05-10
layout: post
title: "检测常见的手势 （译文）"
---

一个触摸手势发生在一个用户将一个或多个手指放在触摸屏上，然后你的应用将这些触摸事件(pattern)翻译为一个具体的手势。有两个相应的阶段去作手势检测:

- 收集关于触摸事件的数据.
- 解释(interperting)这些数据，看它是否符合你的应用程序所支持的手势的标准(criteria).

### 支持包/类

本课程中的示例使用了GestureDetectorCompat 和MotionEventCompat 类.这些类在支持包中.当可能需要在Android 1.6或者更高的系统中使用时，你应该使用支持包。注意，MotionEventCompat <em>不是</em> MotionEvent类的替代版。相反，它提供了多种静态方法 ，为了接收已经结合事件的期望动作，传递你的MotionEvent对象。

### 收集数据

当用户将一个或多个手指放在屏幕上，在可接收触摸事件的视图上触发回调函数 onTouchEvent().对于每个触摸事件序列(位置，压力，大小，另一个手指的添加等),最终确定为一个手势，onTouchEvent()会多次调用(fired)

当用户第一次接触屏幕时，手势开始，系统追踪用户手指的位置，手势也跟着继续，直到捕获用户的手指离开屏幕时结束。始终相互作用，传递(delivered)到onTouchEvent()的MotionEvent提供了每个交互的细节。你的应用可以使用这些提供了MotionEvent的数据去决定是否一个手势动作发生了值得去关注。

### 在Activity或者View中捕获触摸事件

重载onTouchEvent()回调函数，以拦截触摸事件在Activity或者View

以下片段使用了getActionMasked()去提取用户从eventParameter执行的动作。它提供给你原始数据，由你决定是否一个手势是你关心发生的。

```
public class MainActivity extends Activity {
// This example shows an Activity, but you would use the same approach if
// you were subclassing a View.
@Override
public boolean onTouchEvent(MotionEvent event){
    int action = MotionEventCompat.getActionMasked(event);
    switch(action) {
        case (MotionEvent.ACTION_DOWN) :
            Log.d(DEBUG_TAG,"Action was DOWN");
            return true;
        case (MotionEvent.ACTION_MOVE) :
            Log.d(DEBUG_TAG,"Action was MOVE");
            return true;
        case (MotionEvent.ACTION_UP) :
            Log.d(DEBUG_TAG,"Action was UP");
            return true;
        case (MotionEvent.ACTION_CANCEL) :
            Log.d(DEBUG_TAG,"Action was CANCEL");
            return true;
        case (MotionEvent.ACTION_OUTSIDE) :
            Log.d(DEBUG_TAG,"Movement occurred outside bounds " +
                    "of current screen element");
            return true;      
        default :
            return super.onTouchEvent(event);
    }      
}
```

然后你可以做你自己的处理在这些事件上，去决定一个手势是否发生。当你需要做一个定制的手势时，这是必须要做的过程。然而，如果你的应用使用常用的手势如双击，长按，扫等等，你可以使用GestureDetector类的优势。GestureDetector将事情简化，它不需要你自己处理个别的触摸时间就可以检测常见的手势.见讨论[Detect Gestures](http://developer.android.com/training/gestures/detector.html#detect)

###在单个View中捕获触摸事件###

作为OnTouchEvent()的代替，你可以使用setOnTouchListener()方法将一个View.OnTouchListener依附(attach)到任何View对象上去。它使得我们无需子类化一个已经存在的View，而可以去监听触摸事件。例如：


```
View myView = findViewById(R.id.my_view);
myView.setOnTouchListener(new OnTouchListener() {
    public boolean onTouch(View v, MotionEvent event) {
        // ... Respond to touch events      
        return true;
    }
});
```

如果你创建了一个监听器，在处理ACTION_DOWN事件后返回 false ，你需要注意了。如果你这么做了，这个监听器将接收不到随后的ACTION_MOVE和ACTION_UP 事件（String of events)。这是因为ACTION_DOWN是所有触摸事件的触发点。

如果你创建了一个定制View，你可以重载onTouchEvent()，如以上讨论的。

###检测手势###


Android提供了GestureDetector类来检测通用手势，它支持了一些手势在onDown(),onLongPress(),onFling()等方法中。你可以结合OnTouchEvent()使用GestureDetector，如刚才讨论的。

###检测所有支持的手势###

当你实例化一个GestureDetectorCompat对象时，其中一个参数需要实现GestureDetector.OnGestureListener接口。GestureDetector.OnGestureListener通知用户何时一个特定的触摸事件发生。为了使你的GestureDetector对象可以接收事件，你需要重载View或者Activity的OnTouchEvent()方法，然后沿着所有被观察的事件传递到检测实例。


接下来的片段中，一个返回true的特定on&lt;TouchEvent&gt;意味着你已经处理了触摸时间。返回false则通过视图栈向下传递事件，直到触摸被成功处理。

执行下面的代码，感受下当你与屏幕互动(interact)时，动作是如何被触发的，以及每个触摸事件中MotionEvent的内容。你将会了解即使是单个交互动作有多少数据生产。

```
public class MainActivity extends Activity implements
        GestureDetector.OnGestureListener,
        GestureDetector.OnDoubleTapListener{
    private static final String DEBUG_TAG = "Gestures";
    private GestureDetectorCompat mDetector;
    // Called when the activity is first created.
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        // Instantiate the gesture detector with the
        // application context and an implementation of
        // GestureDetector.OnGestureListener
        mDetector = new GestureDetectorCompat(this,this);
        // Set the gesture detector as the double tap
        // listener.
        mDetector.setOnDoubleTapListener(this);
    }
    @Override
    public boolean onTouchEvent(MotionEvent event){
        this.mDetector.onTouchEvent(event);
        // Be sure to call the superclass implementation
        return super.onTouchEvent(event);
    }
    @Override
    public boolean onDown(MotionEvent event) {
        Log.d(DEBUG_TAG,"onDown: " + event.toString());
        return true;
    }
    @Override
    public boolean onFling(MotionEvent event1, MotionEvent event2,
            float velocityX, float velocityY) {
        Log.d(DEBUG_TAG, "onFling: " + event1.toString()+event2.toString());
        return true;
    }
    @Override
    public void onLongPress(MotionEvent event) {
        Log.d(DEBUG_TAG, "onLongPress: " + event.toString());
    }
    @Override
    public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX,
            float distanceY) {
        Log.d(DEBUG_TAG, "onScroll: " + e1.toString()+e2.toString());
        return true;
    }
    @Override
    public void onShowPress(MotionEvent event) {
        Log.d(DEBUG_TAG, "onShowPress: " + event.toString());
    }
    @Override
    public boolean onSingleTapUp(MotionEvent event) {
        Log.d(DEBUG_TAG, "onSingleTapUp: " + event.toString());
        return true;
    }
    @Override
    public boolean onDoubleTap(MotionEvent event) {
        Log.d(DEBUG_TAG, "onDoubleTap: " + event.toString());
        return true;
    }
    @Override
    public boolean onDoubleTapEvent(MotionEvent event) {
        Log.d(DEBUG_TAG, "onDoubleTapEvent: " + event.toString());
        return true;
    }
    @Override
    public boolean onSingleTapConfirmed(MotionEvent event) {
        Log.d(DEBUG_TAG, "onSingleTapConfirmed: " + event.toString());
        return true;
    }
}
```

### 检测一个被支持手势的子集


如果你只是想要处理小部分的手势，你可以继承GestureDetector.SimpleOnGestureListener ，而不是GestureDetector.OnGestureListener接口。


GestureDetector.SimpleOnGestureListener 提供所有On&lt;TouchEvent&gt;方法返回false的实现.因此你可以只重载你所关心的方法。例如，接下来的片段创建了一个继承GestureDetector.SimpleOnGestureListener类，重载了OnFling()和OnDown();


无论你是否使用GestureDetector.OnGestureListener，这是一个最佳实践(best practice)去实现一个返回true的 onDown()方法。因为所有的手势开始与一个onDown()信息。如果你在onDown()返回false，GestureDetector.SimpleOnGestureListener只做默认动作，系统假定你要忽视手势的剩余部分(rest)，然后GestureDetector.OnGestureListener的其他方法将永远不会被调用。这可能是在你应用中的一个潜在异常(potential)。除非你真的是要忽略手势，你才应该返回false.

```
public class MainActivity extends Activity {
    private GestureDetectorCompat mDetector;
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mDetector = new GestureDetectorCompat(this, new MyGestureListener());
    }
    @Override
    public boolean onTouchEvent(MotionEvent event){
        this.mDetector.onTouchEvent(event);
        return super.onTouchEvent(event);
    }
    class MyGestureListener extends GestureDetector.SimpleOnGestureListener {
        private static final String DEBUG_TAG = "Gestures";
        @Override
        public boolean onDown(MotionEvent event) {
            Log.d(DEBUG_TAG,"onDown: " + event.toString());
            return true;
        }
        @Override
        public boolean onFling(MotionEvent event1, MotionEvent event2,
                float velocityX, float velocityY) {
            Log.d(DEBUG_TAG, "onFling: " + event1.toString()+event2.toString());
            return true;
        }
    }
}
```
