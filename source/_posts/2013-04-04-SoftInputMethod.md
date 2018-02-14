---
category: Android
date: 2013-04-04
layout: post
title: 显示隐藏软键盘--InputMethodManager源码
---

InputMethodManager主要是用来管理IME的相关操作的，如显示,隐藏,开关等,我们常会用到这么三个方法

- public void toggleSoftInputFromWindow(android.os.IBinder windowToken, int showFlags, int hideFlags)
- public boolean hideSoftInputFromWindow(android.os.IBinder windowToken, int flags)
- public void showSoftInputFromInputMethod(android.os.IBinder token, int flags)   

showFlags,hideFlags是我们可以传入的参数常量，让我们看看文档里对它们的定义:

    /**
     * Flag for {@link #showSoftInput} to indicate that this is an implicit
     * request to show the input window, not as the result of a direct request
     * by the user.  The window may not be shown in this case.
     */
    public static final int SHOW_IMPLICIT = 0x0001;

    /**
     * Flag for {@link #showSoftInput} to indicate that the user has forced
     * the input method open (such as by long-pressing menu) so it should
     * not be closed until they explicitly do so.
     */
    public static final int SHOW_FORCED = 0x0002;


    /**
     * Flag for {@link #hideSoftInputFromWindow} to indicate that the soft
     * input window should only be hidden if it was not explicitly shown
     * by the user.
     */
    public static final int HIDE_IMPLICIT_ONLY = 0x0001;

    /**
     * Flag for {@link #hideSoftInputFromWindow} to indicate that the soft
     * input window should normally be hidden, unless it was originally
     * shown with {@link #SHOW_FORCED}.
     */
    public static final int HIDE_NOT_ALWAYS = 0x0002;

可以看出SHOW_IMPLICIT(隐形显示)和HIDE_IMPLICIT_ONLY(隐形隐藏)是相对应的，

1. 假如用户显示时设置flag为SHOW_FORCED(强制显示)，那么当隐藏是设置HIDE_IMPLICIT_ONLY时，软键盘并不会隐藏。
