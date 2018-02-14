---
layout: post
category: Android
date: 2013-06-07
title: Android ActionBar源码解析之FragmentTab
---

在Android3.0以上以及android.support.v4包中包含了ActionBar这个控件，十分好用的一个模块就是它的[ActionBar.Tab](http://developer.android.com/reference/android/app/ActionBar.Tab.html).**但是**，它将布局与Fragment的管理耦合在了一起，假如我们想定制的话，还是需要花费一点功夫的，所以我将它们解耦，简化了一下，单独拿了出来。点击查看：[Github：Android-FragmentTab](https://github.com/LinkYan/Android-FragmentTab)

首先让我们看看FragmentTab的实现，不是很复杂：

```
public class FragmentTab {
    FragmentManager mFragmentManager;
    HashMap<String, TabItem> mTabItems = new HashMap<String, TabItem>();
    TabItem mSelectedTabItem;

    public FragmentTab(FragmentManager mFragmentManager) {
        this.mFragmentManager = mFragmentManager;
    }

    public void addTabItem(TabItem item) {
        mTabItems.put(item.getTag(), item);
    }

    public void selectTab(String tag) {
        selectTab(mTabItems.get(tag));
    }

    public void selectTab(TabItem tabItem) {

        final FragmentTransaction trans = mFragmentManager.beginTransaction()
                .disallowAddToBackStack();

        if (mSelectedTabItem == tabItem) {
            if (mSelectedTabItem != null) {
                mSelectedTabItem.onTabReselected(mSelectedTabItem, trans);
            }
        } else {
            if (mSelectedTabItem != null) {
                mSelectedTabItem.onTabUnselected(mSelectedTabItem, trans);
            }
            mSelectedTabItem = tabItem;
            if (mSelectedTabItem != null) {
                mSelectedTabItem.onTabSelected(mSelectedTabItem, trans);
            }
        }

        if (!trans.isEmpty()) {
            trans.commit();
        }

    }

}
```

可以看出我们实际上只有两个方法可以调用`addTabItem(TabItem item)`和`selectTab(TabItem item)` ，一个添加tab和一个选择tab。

TabItem是一个抽象类,构造函数中的`Class<T> mClass`是Fragment的类，以及它的参数`Bundle mArgs`:

```
public abstract class TabItem<T> implements TabListener {
    protected final Context mContext;
    protected final String mTag;
    protected final Class<T> mClass;
    protected final Bundle mArgs;
    protected Fragment mFragment;

    public TabItem(Context ctx, String mTag, Class<T> mClass) {
        this(ctx, mTag, mClass, null);
    }

    public TabItem(Context ctx, String mTag, Class<T> mClass, Bundle mArgs) {
        this.mContext = ctx;
        this.mTag = mTag;
        this.mClass = mClass;
        this.mArgs = mArgs;
    }

    public String getTag() {
        return mTag;
    }
}
```

TabListener接口定义了3个方法，tab选中时，tab从选中切换到未被选中时，以及再次被选中时。

```
public interface TabListener {
    public void onTabSelected(TabItem tab, FragmentTransaction ft);

    public void onTabUnselected(TabItem tab, FragmentTransaction ft);

    public void onTabReselected(TabItem tab, FragmentTransaction ft);
}
```

最后就是TabItem的实现了，默认行为与ActionBar.Tab是一致的，更多使用帮助：[Github：Android-FragmentTab](https://github.com/LinkYan/Android-FragmentTab)

```
public class TabItemImpl<T> extends TabItem<T> implements TabListener {


    public TabItemImpl(Context ctx, String mTag, Class<T> mClass) {
        super(ctx, mTag, mClass);
    }

    public TabItemImpl(Context ctx, String mTag, Class<T> mClass, Bundle mArgs) {
        super(ctx, mTag, mClass, mArgs);
    }

    @Override
    public void onTabSelected(TabItem tab, FragmentTransaction ft) {
        if (mFragment == null) {
            mFragment = Fragment.instantiate(mContext, mClass.getName(), mArgs);
            ft.add(R.id.layout_contain, mFragment, mTag);
        } else {
            ft.attach(mFragment);
        }
    }

    @Override
    public void onTabUnselected(TabItem tab, FragmentTransaction ft) {
        if (mFragment != null) {
            ft.detach(mFragment);
        }
    }

    @Override
    public void onTabReselected(TabItem tab, FragmentTransaction ft) {
        //ToDo
    }
}
```
