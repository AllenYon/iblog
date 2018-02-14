---
category: Android
date: 2013-05-14
layout: post
title: Lazy ListView--细化ListView加载图片策略
---

使用ListView控件来加载图片时，我们一般会使用异步加载的方式。但是如果我们滑动过快的话，可能会超出AsyncTask最大128个线程的限制，然后报异常，FC。于是我们可以使用```setOnScrollListener(…)```监听当ScrollState变为```SCROLL_STAT_IDLE```时我们再触发图片加载，[Android-Universal-Image-Loader](https://github.com/nostra13/Android-Universal-Image-Loader) 有实现这样的逻辑。

但是我将细化这个加载策略

1. ScrollState==SCROLL_STATE_TOUCH_SCROLL（我们手指触摸屏幕，拖动ListView滑动） 时，我们和往常一样创建加载任务，并且显示图片，我们定义为普通加载模式。
2. ScrollState==SCROLL_STATE_FLING （ListView在滑动，但是我们手指已经离开了屏幕，ListView出去 Fling 状态）时，图片停止加载。我们定义为延迟加载模式。
3. ScrollState==SCROLL_STATE_IDEL，这时候ListView滑动停止，我们根据不同的加载模式，有不同的策略。
4. 在普通加载模式下，图片逐张被加载显示出来。
5. 在延迟加载模式下，LazyAdapter中会回调onLazyLoad 并带有两个参数：
	1. firstVisibleItem
	2. lastVisibleItem

Adapter需要根据这两个变量来创建加载图片任务，并显示。我们需要继承LazyAdapter类。

```
public abstract class LazyAdapter extends BaseAdapter {
	...
    public abstract void onLazyLoad(int first, int last);
    ...
}
```
LazyListView 中监听ScrollState是这样实现的：


```
public class LazyListView extends ListView implements AbsListView.OnScrollListener {
	. . .
    private int mFirstCount;
    private int mVisibleCount;
    private LazyAdapter mLazyAdapter;
    @Override
    public void onScrollStateChanged(AbsListView view, int scrollState) {
        // TODO Auto-generated method stub
        if (scrollState == SCROLL_STATE_IDLE) {
            mLazyAdapter.onLazyLoad(mFirstCount, mVisibleCount + mFirstCount);
        }
    }
    @Override
    public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {
        // TODO Auto-generated method stub
        mFirstCount = firstVisibleItem;
        mVisibleCount = visibleItemCount;
    }
}
```
接着我们需要LazyAapater维持两个变量```isFirstLoad```和```mCurrentScrollState```
一个用来标记是否第一次加载，一个用来记录当前的ScrollState。在调用```notifyDataSetChanged()```时，还需要重新标记为第一次加载状态，这是因为它不会触发ScrollState改变。

所以完整版的LazyAdapter是这样的：

```
public abstract class LazyAdapter extends BaseAdapter {
    private boolean isFirstLoad;
    private int mCurrentScrollState;
    public void setFirstLoad(boolean isFirstLoad) {
        this.isFirstLoad = isFirstLoad;
    }
    public boolean isFirstLoad() {
        return isFirstLoad;
    }
    public void resetFirstLoad() {
        this.isFirstLoad = true;
    }
    public void setScrollState(int state) {
        this.mCurrentScrollState = state;
    }
    public boolean isTouchScroll() {
        return mCurrentScrollState == AbsListView.OnScrollListener.SCROLL_STATE_TOUCH_SCROLL;
    }
    @Override
    public void notifyDataSetChanged() {
        // TODO Auto-generated method stub
        resetFirstLoad();
        super.notifyDataSetChanged();
    }
    public abstract void onLazyLoad(int first, int last);
}
```

在LazyListView调用```setAdapter()```时，我们需要类型检测下，并且将LazyAdapter设为第一次加载状态.
然后完成版的LazyListView是这样的:

```
public class LazyListView extends ListView implements AbsListView.OnScrollListener {
    public LazyListView(Context context) {
        super(context);
        setOnScrollListener(this);
    }
    public LazyListView(Context context, AttributeSet attrs) {
        super(context, attrs);
        setOnScrollListener(this);
    }
    public LazyListView(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
        setOnScrollListener(this);
    }
    private int mFirstCount;
    private int mVisibleCount;
    private LazyAdapter mLazyAdapter;
    @Override
    public void setAdapter(ListAdapter adapter) {
        if (adapter instanceof LazyAdapter) {
            mLazyAdapter = (LazyAdapter) adapter;
            mLazyAdapter.setFirstLoad(true);
            super.setAdapter(adapter);
        } else {
            throw new IllegalArgumentException("The adapter must be LazyAdapter");
        }
    }
    @Override
    public void onScrollStateChanged(AbsListView view, int scrollState) {
        // TODO Auto-generated method stub
        if (scrollState == SCROLL_STATE_IDLE) {
            mLazyAdapter.onLazyLoad(mFirstCount, mVisibleCount + mFirstCount);
        }
        if (mLazyAdapter.isFirstLoad()) {
            if (scrollState == SCROLL_STATE_TOUCH_SCROLL) {
                mLazyAdapter.setFirstLoad(false);
            }
        }
        mLazyAdapter.setScrollState(scrollState);
    }
    @Override
    public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {
        mFirstCount = firstVisibleItem;
        mVisibleCount = visibleItemCount;
    }
}
```

接着看看我们如何使用：

```
public class DemoAdapter extends LazyAdapter {
    private SparseArray<LKImageView> mLazyImgs;
    @Override
    public void onLazyLoad(int first, int last) {
        //ToDo
        for (int i = first - 2; i < last + 2; i++) {      
            LKImageView img = mLazyImgs.get(i);
            if (img != null) {
            	DisplayOptions options=…
                img.displayWithMemory(options);
            }
        }
        mLazyImgs.clear();
    }
    @Override
    public View getView(int position, View cv, ViewGroup viewGroup) {
      	…
       	Holder holder=…
       	DisplayOptions options=…
       	if(isFirstLoad()){
       		holder.mImg.display(options);
       	}else{
       		if(isTouchScroll()){
       			holder.mImg.display(options);
       		}else{
       			if(holder.mImg.tryDisplayWithMemory(options)){
       				mLazyImgs.append(position,holder.mImg);
       			}
       		}
       	}
       	. . .
        return view;  //ToDo
    }
}
```

LKImageView 是继承自ImageView类，并扩展了很多附加功能的控件，可以实现内存/磁盘缓存，显示功能==>Link-ImageLoader
现在我们只要知道：

- ```public void display(DisplayOptions options);``` 异步加载图片的方法。
- ```public boolean tryDisplayWithMemory(DisplayOptions options); ```尝试从内存中同步加载图片的方法

在onLazyLoad()中 ```for (int i = first - 2; i < last + 2; i++)```
分别-2和+2，可以做到不仅仅加载当前屏幕的可见范围，还可以多加载几个，不做调整也是没有关系的。

**并且```mLazyImgs.clear()```在结束是必须要调用的，不然会有问题。**

现在就去尝试下吧，你可以下载我们的这个应用

<img src="https://lh4.ggpht.com/EUP4NTyHIMdWZsMgYX5w2kgIE3e3JE5Ud_Yx7tkeYOj8AALgSS2TUYoW92V-dQvVyVM=w124" height=50>[壁纸控](https://play.google.com/store/apps/details?id=com.huaban.wallpaper&feature=search_result#?t=W251bGwsMSwxLDEsImNvbS5odWFiYW4ud2FsbHBhcGVyIl0.) ，里面使用了这套加载策略。
