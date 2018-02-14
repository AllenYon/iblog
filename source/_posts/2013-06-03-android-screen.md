---
category: Android
date: 2013-06-03
layout: post
title: Android屏幕分辨率占有率分析及应用
---

Android屏幕分辨率各种各样，碎片化严重，开发者苦不堪言，转身看看iOS开发同学，只要适配2个尺寸的屏幕就好了，= =！

但我们总是能克服的对不，假如我们现在有这么一个需求：
###我们将满屏显示一张网络图片，希望它不要失真或者被裁剪！

怎么做？先看看[友盟的统计](http://www.umindex.com/#android_resolution):

<table>
    <tr>
        <td>排名</td>
        <td>分辨率</td>
        <td>高宽比</td>
        <td>占有率</td>
    </tr>
     <tr>
        <td>1</td>
        <td>800*480</td>
        <td>1.66</td>
        <td>37.7%</td>
    </tr>
     <tr>
        <td>2</td>
        <td>480*320</td>
        <td>1.5</td>
        <td>25.4%</td>
    </tr>
     <tr>
        <td>3</td>
        <td>854*480</td>
        <td>1.779</td>
        <td>9.5%</td>
    </tr>
      <tr>
        <td>4</td>
        <td>1280*720</td>
        <td>1.777 </td>
        <td>5.6%</td>
    </tr>
      <tr>
        <td>5</td>
        <td>960*540 </td>
        <td>1.777</td>
        <td>5.6%</td>
    </tr>
      <tr>
        <td>6</td>
        <td>320*240</td>
        <td>1.333</td>
        <td>3.1%</td>
    </tr>
    <tr>
        <td>7</td>
        <td>1280*800</td>
        <td>1.6</td>
        <td>2.5%</td>
    </tr>
    <tr>
        <td>8</td>
        <td>1024*600</td>
        <td>1.706</td>
        <td>1.3%</td>
    </tr>
    <tr>
        <td>9</td>
        <td>960*640</td>
        <td>1.5</td>
        <td>0.9%</td>
    </tr>
    <tr>
        <td>10</td>
        <td>1024*768</td>
        <td>1.3</td>
        <td>0.5%</td>
    </tr>
    <tr>
        <td>11</td>
        <td>other</td>
        <td>?</td>
        <td>4.7%</td>
    </tr>
</table>

让我们合并下高宽比约等项，再排名：

<table>
    <tr>
        <td>排名</td>
        <td>高宽比</td>
        <td>占有率</td>
    </tr>
     <tr>
        <td>1</td>
        <td>1.6~1.66</td>
        <td>2.5%+37.7%=40.2%</td>
    </tr>
     <tr>
        <td>2</td>
        <td>1.5</td>
        <td>25.4%+0.9%=26.3%</td>
    </tr>
     <tr>
        <td>3</td>
        <td>1.706~1.779</td>
        <td>1.3%+8.8%+5.6%+9.5%=25.2%</td>
    </tr>
     <tr>
        <td>4</td>
        <td>1.3~1.333</td>
        <td>0.5%~3.1%=3.6%</td>
    </tr>
     <tr>
        <td>5</td>
        <td>?</td>
        <td>4.7%</td>
    </tr>
</table>

可以看出前三项占了91%!!!,况且1.333的屏幕是320*240这种小屏幕，好古董的屏幕，而且未知项中可能还包含前三项中的可能，所以如果我们忽略一些细节的话，最终情况是这样的：

<table>
    <tr>
        <td>排名</td>
        <td>分辨率</td>
        <td>高宽比</td>
        <td>占有率</td>
    </tr>
     <tr>
        <td>1</td>
        <td>800*480</td>
        <td>1.66</td>
        <td>40.2%</td>
    </tr>
     <tr>
        <td>2</td>
        <td>480*320</td>
        <td>1.5</td>
        <td>26.3%</td>
    </tr>
     <tr>
        <td>3</td>
        <td>1280*720</td>
        <td>1.777</td>
        <td>25.2%</td>
    </tr>
</table>


有什么用呢?

###结论：假如这张网络请求下来的图片是设计师给出的，就请设计师对应着表3给出3套图，程序中我们动态算出手机屏幕的高宽比，根据高宽比对应选出最合适的图片，将scaleType设置为fitXY，这样就能做到最大程度的不失真了。

给一个写好的枚举类

```
enum AspectRatio {
    SCALE_15, SCALE_166, SCALE_177;

    public static AspectRatio getAspectRatio(float h2w) {
        if (h2w <= 1.5f) {
            return AspectRatio.SCALE_15;
        } else if (h2w > 1.5 && h2w <= 1.67) {
           return AspectRatio.SCALE_166;
        } else if (h2w > 1.67) {
            return AspectRatio.SCALE_177;
        } else {
             return AspectRatio.SCALE_166;
        }
    }
}
```
