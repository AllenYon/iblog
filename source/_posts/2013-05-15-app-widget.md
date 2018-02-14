---
category: Android
date: 2013-05-15
layout: post
title: Android开发者指南之App Widgets (译)
---
App Widgets是一种能嵌入到其他应用程序(如Home screen)的小型应用程序界面，并且能接收定时的更新。这些视图被称为用户界面中的窗口小部件，需要App Widget provider才能发布.能持有其他App Widgets的应用组件叫做App Widget host.下面显示了一张Music App Widget的图。

![music app widget](http://developer.android.com/images/appwidgets/appwidget.png)

本文档将描述如何使用App Widget provider发布一个App Widget。

> ####Widget Design
> 更多关于如何设计你的app widget,你可以阅读这篇 [窗口小部件设计指导](http://developer.android.com/design/patterns/widgets.html).

##基础
---
创建一个App Widget ，你需要以下条件:

1. AppWidgetProviderInfo 对象。用来描述App Widget的元数据，比如App Widget的布局，更新频率，AppWidgetProvider类。也可以使用xml定义。
2. AppWidgetProvider 类实例。定义了一些基本方法，需要你去实现这些接口，是基于广播(broadcast)事件的。通过它，你可以接收到什么时候App Widget被更新了，可使用，不可使用和被删除了等状态。
3. 界面布局。定义了为App Widget初始化的布局，定义在XML中。

此外，你可以实现一个App Widget配置Activity。这个可选的Activity的主要作用是，当用户在启动器(launches)中添加了你的App Widget时允许他在创建的时候就修改App Widget的设置。

以下各节描述了如何设置每个组件

##在Manifest中声明一个App Widget
---
首先，在你应用程序的AndroidManifest.xml文件中声明AppWidgetProvider类，像这样:

```
<receiver android:name="ExampleAppWidgetProvider" >
    <intent-filter>
        <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
    </intent-filter>
    <meta-data android:name="android.appwidget.provider"
               android:resource="@xml/example_appwidget_info" />
</receiver>
```

`<receiver>`元素需要android:name属性，用来指定App Widget所使用的AppWidgetProvider。

`<intent-filter>`元素必须包含一个带有android:name属性的`<action>`元素。这个属性指定了AppWidgetProvider接收`ACTION_APPWIDGET_UPDATE`广播。这是你必须要显示声明的广播。AppWidgetManager会自动发生其他所有的App Widget广播到AppWidgetProvider中，如果有必要的话。

`<meta-data>`元素指定了AppWidgetProviderInfo的来源并且要求有以下属性：
- android:name-指定元数据名称。使用`android.appwidget.provider`作为AppWidgetProviderInfo的描述来识别数据
- android:resource-指定AppWidgetProviderInfo的本地来源

##添加AppWidgetProviderInfo 元数据
---
AppWidgetProviderInfo定义了一个App Widget必要的特性（qualities），例如最小的布局尺寸，初始布局资源，多久更新一次App Widget，创建时调用的配置Activity。可以使用单个`<appwidget-provider>`元素的XML资源来定义AppWidgetProviderInfo对象，然后保持到项目的`res/xml/`文件夹下。

例如：

```
<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
    android:minWidth="294dp"
    android:minHeight="72dp"
    android:updatePeriodMillis="86400000"
    android:previewImage="@drawable/preview"
    android:initialLayout="@layout/example_appwidget"
    android:configure="com.example.android.ExampleAppWidgetConfigure"
    android:resizeMode="horizontal|vertical"
    android:widgetCategory="home_screen|keyguard"
    android:initialKeyguardLayout="@layout/example_keyguard">
</appwidget-provider>
```

关于`<appwidget-provider>`属性的一些总结:

- `miniWidth`和`minHeight`
- `minResizeWidth`和`minResizeHeight`
- `updatePeriodMillis`
- `initialLayout`
- `configure`
- `perviewImage`
- `autoAdvanceViewId`
- `resizeMode`
- `widgetCategory`
- `initialKeyguardLayout`

关于更多`<appwidget-provider>可以接收的属性的相关信息，可以查看[AppWidgetProviderInfo](http://developer.android.com/reference/android/appwidget/AppWidgetProviderInfo.html)类。

## 创建App Widget布局
---

你必须将你的App Widget初始化布局定义在XML中，并且将它保存在项目的`res/layout/`目录下。你可以使用下面列表列出的视图对象来设计你的App Widget,但是在你设计你的App Widget前，请先阅读并理解[App Widget Design Guidelines](http://developer.android.com/guide/practices/ui_guidelines/widget_design.html)。

创建一个App Widget布局是很简单的，如果你很了解[Layouts](http://developer.android.com/guide/topics/ui/declaring-layout.html)的话。但是你应该意识到App Widget是基于[RemoteViews](http://developer.android.com/reference/android/widget/RemoteViews.html)的，它不支持所有类型的布局和控件(view widget)。

一个RemoteViews对象(consequently 一个App Widget)可以支持以下的布局类:

- FrameLayout
- LinearLayout
- RelativeLayout
- GridLayout

和以下控件类:

- AnalogClock
- Button
- Chronometer
- ImageButton
- ImageView
- ProgressBar
- TextView
- ViewFlipper
- ListView
- GridView
- StackView
- AdapterViewFlipper

这些类的派生类(Descendants)是不支持的。

RemoteViews 也支持 ViewStub,一种不可见，不占大小的视图，你可以在运行时延迟`inflate`布局资源

### 为App Widget添加margins

## 使用AppWidgetProvider类
---
###接收App Widget广播意图 (broadcast intents)

## 创建一个App Widget配置Activity
---

###从配置Activity更新App Widget


## 设置预览图片
---

##使App Widget在锁屏上可用
---

###改变大小指导

##使用带有集合的App Widget
---
Android 3.0引进了带有集合的App Widget。这种App Widget使用RemoteViewService去显示那些支持远程数据的集合,例如从content provider。由RemoteViewsService提供的数据展示在App Widget中使用了以下视图类型中的一种,我们称之为"collections views"

- ListView:A view that shows items in a vertically scrolling list. For an example, see the Gmail app widget.
- GridView:A view that shows items in two-dimensional scrolling grid. For an example, see the Bookmarks app widget.
- StackView:A stacked card view (kind of like a rolodex), where the user can flick the front card up/down to see the previous/next card, respectively. Examples include the YouTube and Books app widgets.
- AdapterViewFlipper:An adapter-backed simple ViewAnimator that animates between two or more views. Only one child is shown at a time.

正如上文所述,这些collections views显示支持远程数据的集合。这意味着他们使用Adpater将接口和数据
绑定一起。Adapter绑定将特殊items从一些集合的数据绑定到特殊的视图对象上。因为这些collection views被adapters支持，所以Android框架必须包含额外的体系结构去支持他们在app widget中使用。在app widget的上下文中，adapter被代替为RemoteViewsFactory,仅仅是简单的对Adapter接口进行了薄封装。当请求在集合中一个具体的item，RemoteViewsFactory会为集合创建并返回RemoteViews对象作为Item。为了能在你的app widget中使用collection view，你必须实现RemoteViewService和RemoteViewsFactory。

RemoteViewsService是一种允许一个远程adapter请求RemoteViews对象的service。
RemoteViewsFactory是一种为collections view（如ListView,GridView等）和视图中潜在数据做适配的接口。下面这段模板代码,你可以用实现service和interface，从StackView Widget sample中提取出来的:

```
public class StackWidgetService extends RemoteViewsService {
    @Override
    public RemoteViewsFactory onGetViewFactory(Intent intent) {
        return new StackRemoteViewsFactory(this.getApplicationContext(), intent);
    }
}
class StackRemoteViewsFactory implements RemoteViewsService.RemoteViewsFactory {
//... include adapter-like methods here. See the StackView Widget sample.
}
```

### 例子应用

本节的代码片段取材至StackView Widget sample:

![sample](http://developer.android.com/images/appwidgets/StackWidget.png)

例子有十个栈视图组成，从"0!"显示到"9!"。这个例子app widget有以下主要行为：

- 用户可以垂直方向甩动在app widget的顶部View，来显示上一个或者下一个view。这是内建在StackView的行为。
- Without any user interaction, the app widget automatically advances through its views in sequence, like a slide show. This is due to the setting android:autoAdvanceViewId="@id/stack_view" in the res/xml/stackwidgetinfo.xml file. This setting applies to the view ID, which in this case is the view ID of the stack view.
- If the user touches the top view, the app widget displays the Toast message "Touched view n," where n is the index (position) of the touched view. For more discussion of how this is implemented, see Adding behavior to individual items.

### 实现带有集合的App Widget
To implement an App Widget with collections, you follow the same basic steps you would use to implement any app widget. The following sections describe the additional steps you need to perform to implement an App Widget with collections.

### 带有集合的App Widget的清单
In addition to the requirements listed in Declaring an App Widget in the Manifest, to make it possible for App Widgets with collections to bind to your RemoteViewsService, you must declare the service in your manifest file with the permission BIND_REMOTEVIEWS. This prevents other applications from freely accessing your app widget's data. For example, when creating an App Widget that uses RemoteViewsService to populate a collection view, the manifest entry may look like this:

```
<service android:name="MyWidgetService"
...
android:permission="android.permission.BIND_REMOTEVIEWS" />
```
The line `android:name="MyWidgetService"` refers to your subclass of RemoteViewsService.


### 带有集合的App Widget的布局
The main requirement for your app widget layout XML file is that it include one of the collection views: ListView, GridView, StackView, or AdapterViewFlipper. Here is the widget_layout.xml for the StackView Widget sample:

```
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <StackView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/stack_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:loopViews="true" />
    <TextView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/empty_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:background="@drawable/widget_item_background"
        android:textColor="#ffffff"
        android:textStyle="bold"
        android:text="@string/empty_view_text"
        android:textSize="20sp" />
</FrameLayout>
```
Note that empty views must be siblings of the collection view for which the empty view represents empty state.

In addition to the layout file for your entire app widget, you must create another layout file that defines the layout for each item in the collection (for example, a layout for each book in a collection of books). For example, the StackView Widget sample only has one layout file, widget_item.xml, since all items use the same layout. But the WeatherListWidget sample has two layout files: dark_widget_item.xml and light_widget_item.xml.

### 带有集合的App Widget的AppWidgetProvider

As with a regular app widget, the bulk of your code in your AppWidgetProvider subclass typically goes in onUpdate(). The major difference in your implementation for onUpdate() when creating an app widget with collections is that you must call setRemoteAdapter(). This tells the collection view where to get its data. The RemoteViewsService can then return your implementation of RemoteViewsFactory, and the widget can serve up the appropriate data. When you call this method, you must pass an intent that points to your implementation of RemoteViewsService and the App Widget ID that specifies the app widget to update.

For example, here's how the StackView Widget sample implements the onUpdate() callback method to set the RemoteViewsService as the remote adapter for the app widget collection:

```
public void onUpdate(Context context, AppWidgetManager appWidgetManager,
int[] appWidgetIds) {
    // update each of the app widgets with the remote adapter
    for (int i = 0; i < appWidgetIds.length; ++i) {     
        // Set up the intent that starts the StackViewService, which will
        // provide the views for this collection.
        Intent intent = new Intent(context, StackWidgetService.class);
        // Add the app widget ID to the intent extras.
        intent.putExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, appWidgetIds[i]);
        intent.setData(Uri.parse(intent.toUri(Intent.URI_INTENT_SCHEME)));
        // Instantiate the RemoteViews object for the App Widget layout.
        RemoteViews rv = new RemoteViews(context.getPackageName(), R.layout.widget_layout);
        // Set up the RemoteViews object to use a RemoteViews adapter.
        // This adapter connects
        // to a RemoteViewsService  through the specified intent.
        // This is how you populate the data.
        rv.setRemoteAdapter(appWidgetIds[i], R.id.stack_view, intent);       
        // The empty view is displayed when the collection has no items.
        // It should be in the same layout used to instantiate the RemoteViews
        // object above.
        rv.setEmptyView(R.id.stack_view, R.id.empty_view);
        //
        // Do additional processing specific to this app widget...
        //     
        appWidgetManager.updateAppWidget(appWidgetIds[i], rv);   
    }
    super.onUpdate(context, appWidgetManager, appWidgetIds);
}
```

### RemoteViewsService类

As described above, your RemoteViewsService subclass provides the RemoteViewsFactory used to populate the remote collection view.

Specifically, you need to perform these steps:

1. Subclass RemoteViewsService. RemoteViewsService is the service through which a remote adapter can request RemoteViews.
2. In your RemoteViewsService subclass, include a class that implements the RemoteViewsFactory interface. RemoteViewsFactory is an interface for an adapter between a remote collection view (such as ListView, GridView, and so on) and the underlying data for that view. Your implementation is responsible for making a RemoteViews object for each item in the data set. This interface is a thin wrapper around Adapter.

The primary contents of the RemoteViewsService implementation is its RemoteViewsFactory, described below.

### RemoteViewsFactory接口

Your custom class that implements the RemoteViewsFactory interface provides the app widget with the data for the items in its collection. To do this, it combines your app widget item XML layout file with a source of data. This source of data could be anything from a database to a simple array. In the StackView Widget sample, the data source is an array of WidgetItems. The RemoteViewsFactory functions as an adapter to glue the data to the remote collection view.

The two most important methods you need to implement for your RemoteViewsFactory subclass are onCreate() and getViewAt() .

The system calls onCreate() when creating your factory for the first time. This is where you set up any connections and/or cursors to your data source. For example, the StackView Widget sample uses onCreate() to initialize an array of WidgetItem objects. When your app widget is active, the system accesses these objects using their index position in the array and the text they contain is displayed

Here is an excerpt from the StackView Widget sample's RemoteViewsFactory implementation that shows portions of the onCreate() method:

```
class StackRemoteViewsFactory implements
RemoteViewsService.RemoteViewsFactory {
    private static final int mCount = 10;
    private List<WidgetItem> mWidgetItems = new ArrayList<WidgetItem>();
    private Context mContext;
    private int mAppWidgetId;
    public StackRemoteViewsFactory(Context context, Intent intent) {
        mContext = context;
        mAppWidgetId = intent.getIntExtra(AppWidgetManager.EXTRA_APPWIDGET_ID,
                AppWidgetManager.INVALID_APPWIDGET_ID);
    }
    public void onCreate() {
        // In onCreate() you setup any connections / cursors to your data source. Heavy lifting,
        // for example downloading or creating content etc, should be deferred to onDataSetChanged()
        // or getViewAt(). Taking more than 20 seconds in this call will result in an ANR.
        for (int i = 0; i < mCount; i++) {
            mWidgetItems.add(new WidgetItem(i + "!"));
        }
        ...
    }
...
```

The RemoteViewsFactory method getViewAt() returns a RemoteViews object corresponding to the data at the specified position in the data set. Here is an excerpt from the StackView Widget sample's RemoteViewsFactory implementation:

```
public RemoteViews getViewAt(int position) {
    // Construct a remote views item based on the app widget item XML file,
    // and set the text based on the position.
    RemoteViews rv = new RemoteViews(mContext.getPackageName(), R.layout.widget_item);
    rv.setTextViewText(R.id.widget_item, mWidgetItems.get(position).text);
    ...
    // Return the remote views object.
    return rv;
}
```

### 为个别Item添加行为
The above sections show you how to bind your data to your app widget collection. But what if you want to add dynamic behavior to the individual items in your collection view?

As described in Using the AppWidgetProvider Class, you normally use setOnClickPendingIntent() to set an object's click behavior—such as to cause a button to launch an Activity. But this approach is not allowed for child views in an individual collection item (to clarify, you could use setOnClickPendingIntent() to set up a global button in the Gmail app widget that launches the app, for example, but not on the individual list items). Instead, to add click behavior to individual items in a collection, you use setOnClickFillInIntent(). This entails setting up up a pending intent template for your collection view, and then setting a fill-in intent on each item in the collection via your RemoteViewsFactory.

This section uses the StackView Widget sample to describe how to add behavior to individual items. In the StackView Widget sample, if the user touches the top view, the app widget displays the Toast message "Touched view n," where n is the index (position) of the touched view. This is how it works:

- The StackWidgetProvider (an AppWidgetProvider subclass) creates a pending intent that has a custom action called TOAST_ACTION.
- When the user touches a view, the intent is fired and it broadcasts TOAST_ACTION.
- This broadcast is intercepted by the StackWidgetProvider's onReceive() method, and the app widget displays the Toast message for the touched view. The data for the collection items is provided by the RemoteViewsFactory, via the RemoteViewsService.

>Note: The StackView Widget sample uses a broadcast, but typically an app widget would simply launch an activity in a scenario like this one.

#### 设置待定意图模板(pending intent template)
StackWidgetProvider（AppWidgetProvider子类）设置了一个待定意图.集合中具体的Item不能单独设置属于他们自己的待定意图。(注：性能原因) 取而代之的，集合作为一个整体设置一个待定意图模板，然后具体的Item设置填充意图(fill-in intent)来创建在item-by-item
The StackWidgetProvider (AppWidgetProvider subclass) sets up a pending intent. Individuals items of a collection cannot set up their own pending intents. Instead, the collection as a whole sets up a pending intent template, and the individual items set a fill-in intent to create unique behavior on an item-by-item basis.

This class also receives the broadcast that is sent when the user touches a view. It processes this event in its onReceive() method. If the intent's action is TOAST_ACTION, the app widget displays a Toast message for the current view.


```
public class StackWidgetProvider extends AppWidgetProvider {
    public static final String TOAST_ACTION = "com.example.android.stackwidget.TOAST_ACTION";
    public static final String EXTRA_ITEM = "com.example.android.stackwidget.EXTRA_ITEM";
    ...
    // Called when the BroadcastReceiver receives an Intent broadcast.
    // Checks to see whether the intent's action is TOAST_ACTION. If it is, the app widget
    // displays a Toast message for the current item.
    @Override
    public void onReceive(Context context, Intent intent) {
        AppWidgetManager mgr = AppWidgetManager.getInstance(context);
        if (intent.getAction().equals(TOAST_ACTION)) {
            int appWidgetId = intent.getIntExtra(AppWidgetManager.EXTRA_APPWIDGET_ID,
                AppWidgetManager.INVALID_APPWIDGET_ID);
            int viewIndex = intent.getIntExtra(EXTRA_ITEM, 0);
            Toast.makeText(context, "Touched view " + viewIndex, Toast.LENGTH_SHORT).show();
        }
        super.onReceive(context, intent);
    }
    @Override
    public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
        // update each of the app widgets with the remote adapter
        for (int i = 0; i < appWidgetIds.length; ++i) {
            // Sets up the intent that points to the StackViewService that will
            // provide the views for this collection.
            Intent intent = new Intent(context, StackWidgetService.class);
            intent.putExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, appWidgetIds[i]);
            // When intents are compared, the extras are ignored, so we need to embed the extras
            // into the data so that the extras will not be ignored.
            intent.setData(Uri.parse(intent.toUri(Intent.URI_INTENT_SCHEME)));
            RemoteViews rv = new RemoteViews(context.getPackageName(), R.layout.widget_layout);
            rv.setRemoteAdapter(appWidgetIds[i], R.id.stack_view, intent);
            // The empty view is displayed when the collection has no items. It should be a sibling
            // of the collection view.
            rv.setEmptyView(R.id.stack_view, R.id.empty_view);
            // This section makes it possible for items to have individualized behavior.
            // It does this by setting up a pending intent template. Individuals items of a collection
            // cannot set up their own pending intents. Instead, the collection as a whole sets
            // up a pending intent template, and the individual items set a fillInIntent
            // to create unique behavior on an item-by-item basis.
            Intent toastIntent = new Intent(context, StackWidgetProvider.class);
            // Set the action for the intent.
            // When the user touches a particular view, it will have the effect of
            // broadcasting TOAST_ACTION.
            toastIntent.setAction(StackWidgetProvider.TOAST_ACTION);
            toastIntent.putExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, appWidgetIds[i]);
            intent.setData(Uri.parse(intent.toUri(Intent.URI_INTENT_SCHEME)));
            PendingIntent toastPendingIntent = PendingIntent.getBroadcast(context, 0, toastIntent,
                PendingIntent.FLAG_UPDATE_CURRENT);
            rv.setPendingIntentTemplate(R.id.stack_view, toastPendingIntent);
            appWidgetManager.updateAppWidget(appWidgetIds[i], rv);
        }
    super.onUpdate(context, appWidgetManager, appWidgetIds);
    }
}
```

#### 设置填充意图(fill-in intent)
你的RemoteViewsFactory必须为每一个在集合中的item设置一个填充意图(fill-in intent)。这样才可能做到区分给定的Item单独的点击动作。填充意图然后与待定意图模板(PendingItent template)结合一起，确定了当item被点击时将执行的最终意图。

```
public class StackWidgetService extends RemoteViewsService {
    @Override
    public RemoteViewsFactory onGetViewFactory(Intent intent) {
        return new StackRemoteViewsFactory(this.getApplicationContext(), intent);
    }
}
class StackRemoteViewsFactory implements RemoteViewsService.RemoteViewsFactory {
    private static final int mCount = 10;
    private List<WidgetItem> mWidgetItems = new ArrayList<WidgetItem>();
    private Context mContext;
    private int mAppWidgetId;
    public StackRemoteViewsFactory(Context context, Intent intent) {
        mContext = context;
        mAppWidgetId = intent.getIntExtra(AppWidgetManager.EXTRA_APPWIDGET_ID,
                AppWidgetManager.INVALID_APPWIDGET_ID);
    }
    // 初始化数据
        public void onCreate() {
            // 在 onCreate()你可以建立任何连接/游标 到你的源数据中.Heavy lifting.
            // 如下载或者创建内容等，应该延迟到onDataSetChanged()方法中
            // 或者 getViewAt(), 在这个回调中耗时超过20秒，将会引起ANR.
            for (int i = 0; i < mCount; i++) {
                mWidgetItems.add(new WidgetItem(i + "!"));
            }
           ...
        }
        ...
        // Given the position (index) of a WidgetItem in the array, use the item's text value in
        // combination with the app widget item XML file to construct a RemoteViews object.
        public RemoteViews getViewAt(int position) {
            // position will always range from 0 to getCount() - 1.
            // Construct a RemoteViews item based on the app widget item XML file, and set the
            // text based on the position.
            RemoteViews rv = new RemoteViews(mContext.getPackageName(), R.layout.widget_item);
            rv.setTextViewText(R.id.widget_item, mWidgetItems.get(position).text);
            // Next, set a fill-intent, which will be used to fill in the pending intent template
            // that is set on the collection view in StackWidgetProvider.
            Bundle extras = new Bundle();
            extras.putInt(StackWidgetProvider.EXTRA_ITEM, position);
            Intent fillInIntent = new Intent();
            fillInIntent.putExtras(extras);
            // Make it possible to distinguish the individual on-click
            // action of a given item
            rv.setOnClickFillInIntent(R.id.widget_item, fillInIntent);
            ...
            // Return the RemoteViews object.
            return rv;
        }
    ...
    }
```

## 保持集合数据最新

下面的流程图发生在一个使用了集合的App Widget更新数据时。它展示了App Widget代码与RemoteViewsFactory的交互，以及你如何触发更新。

![fresh data](http://developer.android.com/images/appwidgets/appwidget_collections.png)

使用了集合的App Widgets的其中一个特性就是可以为用户提供最新的内容。例如，Android 3.0 Gmail app widget，可以为用户提供他们收件箱的一个快照。为了做到这一点，你需要能触发你的RemoteViewsFactory和集合view,去获得并且显示新数据。你可以通过使用AppWidgetManager调用notifyAppWidgetViewDataChanged()来实现这一点。然后在你的RemoteViewsFactory的OnDataChanged()获得一个回调结果，一个你有机会可以获取任何新数据的回调接口。注意，你可以在onDataSetChanged()回调方法中执行密集的同步操作(processing-intensive operations synchronously)。回调方法在元数据或者视图数据从RemoteViewsFactory那里获取之前以及被执行完成，所以你可以放心。此外，你也可以在getViewAt()方法内执行密集的同步操作。**如果这个回调函数执行了很长时间，那么加载视图(RemoteViewsFactory中指定的getLoadingView()方法)将被显示在集合视图中正确的位置，直到它返回结果.**
