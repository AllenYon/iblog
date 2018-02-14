---
category: Android
date: 2013-05-10
layout: post
title: ACTION_MASK 是用来干嘛的？
---

我们经常可以看到这样的代码

```
int action=event.getAction();
switch(action&MotionEvent.ACTION_MASK){
    // balabala
}
```

从字面上理解，即为动作与上动作掩码.看下这些常量的值

```
ACTION_MASK             0x000000ff
ACTION_DOWN             0x00000000
ACTION_UP               0x00000001  
ACTION_MOVE             0x00000002
ACTION_POINTER_DOWN     0x00000005
ACTION_POINTER_UP       0x00000006
ACTION_POINTER_1_DOWN   0x00000005            
ACTION_POINTER_1_UP     0x00000006   
ACTION_POINTER_2_DOWN   0x00000105   
ACTION_POINTER_2_UP     0x00000106
ACTION_POINTER_3_DOWN   0x00000205           
ACTION_POINTER_3_UP     0x00000206
```

我们发现单指DOWN/UP分别为 0x005和0x006,
而双值和三指DOWN/UP分别为 0x105和0x106,0x205和0x206

假设当前触摸动作为ACTION_POINTER_2_DOWN时，

```
int action=0x105;
int maskAction=0x0ff&0x105; //  maskAction=0x005;
```

如此，触摸动作含义改变为ACTION_POINTER_DOWN

总结，Android在很多需要性能的地方都采用了这种传入int类型，再加掩码操作。如onMeasure。
