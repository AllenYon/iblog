---
category: Android
date: 2013-05-31
layout: post
title: Android框架设计之ExceptionCenter
---

##What's up?
---
我们可能常常遇到这样的问题：

1. http请求异常很多种，我们或者将他们归类包装，或者将他们一一抛到上层去处理，通常我们的应用会有很多层，API层,Cache层，业务层，界面层等等，层次越多需要将异常抛的层就越多，最终到达界面了再根据不同的异常渲染不同的页面效果。
2. 某些异常不单单是提示用户就可以了的，还需要处理特定的业务或者与用户进行交互的，比如某用户被禁用了我们除了提示还需要将用户登出，清理数据等。
3. 除了去处理异常，我们还需要对得到的异常分析，保存等定制操作，在哪里做比较合适，比较优雅？

## So,How to solvoe it?
---
我们可以将需要在界面渲染的异常，跟踪保存的异常等统统扔到 ExceptionCenter中，通过`public boolean      fileter(Exception e)`函数过滤每一个异常，进行匹配。如果返回`true` 则在`public void handleException(Context ctx,Exception e)`函数中进行对应的逻辑处理，大概如此：

```
public abstract class LKException implements Serializable {

    public abstract boolean filter(Exception e);

    public abstract void handleException(Context ctx, Exception e);

    public abstract void handleTargetException(Context ctx);

}

```

`implements Serializable`是为了能在Intent中传递。

`ExceptionCenter`采用继承`BroadcastReceiver`异步来处理事件，大致是这样的：

```
public class ExceptionCenter extends BroadcastReceiver {

    public static final String ACTION_EXCEPTION = "com.huaban.exceptioncenter";
    public static final String EXTRA_EXCEPTION = "exception";
    public static final String EXTRA_LKEXCEPTION = "lkexception";

    private ExceptionCenterFactory mFactory;

    List<Class<? extends LKException>> mClazz = new ArrayList<Class<? extends LKException>>();

    public ExceptionCenter(ExceptionCenterFactory factory) {
        super();
        factory.onAddExceptionList(mClazz);
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        //ToDo
        Exception exception = (Exception) intent.getSerializableExtra(EXTRA_EXCEPTION);
        LKException lkException = (LKException) intent.getSerializableExtra(EXTRA_LKEXCEPTION);
        if (lkException != null) {
            lkException.handleTargetException(context);
        } else if (exception != null) {
            try {
                for (Class<? extends LKException> item : mClazz) {
                    LKException lk = item.newInstance();
                    if (lk.filter(exception)) {
                        lk.handleException(context, exception);
                    }
                }
            } catch (InstantiationException e) {
                e.printStackTrace();
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            }
        }
    }
}
```

可以看到`onReceive()`函数中 处理异常有两种模式，一种是不需要进行过滤判断的，直接进行handle，一种是通过循环执行`lk.filter(exception)`判断是否需要进行处理。这是考虑到了开发者可以在ExceptionCenter先在外部进行判断再传进来处理，也可以在内部进行判断同时处理。

同时提供了几个类函数方便使用:

```
private static BroadcastReceiver mBr;

public static void start(Context ctx, ExceptionCenterFactory factory) {
	mBr = new ExceptionCenter(factory);
    IntentFilter intentFilter = new IntentFilter(ACTION_EXCEPTION);
    LocalBroadcastManager.getInstance(ctx).registerReceiver(mBr, intentFilter);
}

public static void stop(Context ctx) {
	LocalBroadcastManager.getInstance(ctx).unregisterReceiver(mBr);
    mBr = null;
}

public static void recevieLKException(Context ctx, LKException exception) {
    Intent intent = new Intent(ACTION_EXCEPTION);
    intent.putExtra(EXTRA_LKEXCEPTION, exception);
    LocalBroadcastManager.getInstance(ctx).sendBroadcast(intent);
}

public static void recevieException(Context ctx, Exception exception) {
    Intent intent = new Intent(ACTION_EXCEPTION);
    intent.putExtra(EXTRA_EXCEPTION, exception);
    LocalBroadcastManager.getInstance(ctx).sendBroadcast(intent);
}
```

`start(Context ctx,ExceptionCenterFactroy factory)`在你的MainActivity的onCreate()中使用，

`stop(context ctx)`在onDestory()中使用，因为使用了support.v4包中的[LocalBroadcatManager](http://developer.android.com/reference/android/support/v4/content/LocalBroadcastManager.html)类，所以这种方式比正常的BroadcastReceiver更加效率,Google原话：


>If you don't need to send broadcasts across applications, consider using
this class with {@link android.support.v4.content.LocalBroadcastManager} instead
of the more general facilities described below.  This will give you a much
more efficient implementation (no cross-process communication needed) and allow
you to avoid thinking about any security issues related to other applications
being able to receive or send your broadcasts.


`ExceptionCenterFactory`只有一个抽象函数

```
public abstract class ExceptionCenterFactory {
    public abstract void onAddExceptionList(List<Class<? extends LKException>> filterList);
}
```
开发者将需要过滤处理的类添加到队列中。

类似这中实现：

```
public class WallpaperExceptionFactory extends ExceptionCenterFactory {
    @Override
    public void onAddExceptionList(List<Class<? extends LKException>> exceptionList) {
        //ToDo
        exceptionList.add(AccountDisableException.class);
        exceptionList.add(LKJSONException.class);
    }
}
```

好的 来个简单的处理异常的类：

```
public class AccountDisableException extends LKException {
    private static final String ACCOUNT_DISABLE = "account_disable";

    private String mMessage;

    public AccountDisableException() {
    }

    public AccountDisableException(String msg) {
        this.mMessage = msg;

    }

    @Override
    public boolean filter(Exception e) {
        if (e.getMessage().equals("account_disabled")) {
            return true;
        }
        return false;  //ToDo
    }

    @Override
    public void handleException(Context ctx, Exception e) {
        //ToDo
        Toast.makeText(ctx, "account disable " + e.getMessage(), Toast.LENGTH_SHORT).show();

    }

    @Override
    public void handleTargetException(Context ctx) {
        //ToDo
        Toast.makeText(ctx, "account disable target" + mMessage, Toast.LENGTH_SHORT).show();
    }
}
```

由于我们传进了`Context ctx`参数，实际上我们可以做任何有关Android的事情了。就看你怎么去做了。

## Last word
---
多稚嫩的框架，这只是alpha版而已。慢慢来，先这么看看
