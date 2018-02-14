---
category: Android
date: 2013-05-31
layout: post
title: Java并发之CyclicBarrier
---
书摘：

`CyclicBarrier`适用于这样的情况：你希望创建一组任务,他们并行地执行工作，然后在进行下一个步骤之前等待，直至所有任务都完成(看起来有些像join())。 它使得所有的并行任务都将在栅栏处列队，因此可以一致地向钱移动。这非常像CountDownLatch,只是CountDownLatch是只触发一次的事件，而CyclicBarrier可以多次重用。

刚开始接触计算机时开始，我就对仿真着了迷，而并发是使仿真成为可能的一个关键因素。记得我最开始编写的一个程序就是一个仿真：一个用BASIC编写的（由于文件名的限制而命名为HOSRAC.BAS的赛马游戏）。下面是那个程序的面向对象的多线程版本，其中使用了`CyclicBarrier`：

```
package cn.link.example;

import java.util.ArrayList;
import java.util.List;
import java.util.Random;
import java.util.concurrent.*;

class Horse implements Runnable {
    private static int counter = 0;
    private final int id = counter++;
    private int strides = 0;
    private static Random rand = new Random(47);
    private static CyclicBarrier barrier;

    public Horse(CyclicBarrier b) {
        barrier = b;
    }

    public synchronized int getStrides() {
        return strides;
    }

    @Override
    public void run() {
        //ToDo
        while (!Thread.interrupted()) {
            synchronized (this) {
                strides += rand.nextInt(3);
            }
            try {
                barrier.await();
            } catch (InterruptedException e) {
                e.printStackTrace();  //To change body of catch statement use File | Settings | File Templates.
            } catch (BrokenBarrierException e) {
                e.printStackTrace();  //To change body of catch statement use File | Settings | File Templates.
            }

        }
    }

    @Override
    public String toString() {
        return "Horse " + id + " ";
    }

    public String tracks() {
        StringBuilder s = new StringBuilder();
        for (int i = 0; i < getStrides(); i++) {
            s.append("*");
        }
        s.append(id);
        return s.toString();
    }
}

public class HorseRace {
    static final int FINISH_LINE = 80;
    private List<Horse> horses = new ArrayList<Horse>();
    private ExecutorService exec = Executors.newCachedThreadPool();
    private CyclicBarrier barrier;

    public HorseRace(int nHorses, final int pause) {
        barrier = new CyclicBarrier(nHorses, new Runnable() {
            @Override
            public void run() {
                //ToDo
                StringBuilder s = new StringBuilder();
                for (int i = 0; i < FINISH_LINE; i++) {
                    s.append("=");
                }
                System.out.println(s);
                for (Horse horse : horses) {
                    System.out.println(horse.tracks());
                }

                for (Horse horse : horses) {
                    if (horse.getStrides() >= FINISH_LINE) {
                        System.out.println(horse + "won!");
                        exec.shutdownNow();
                        return;
                    }
                }

                try {
                    TimeUnit.MILLISECONDS.sleep(pause);
                } catch (InterruptedException e) {
                    e.printStackTrace();  //To change body of catch statement use File | Settings | File Templates.
                }

            }
        });

        for (int i = 0; i < nHorses; i++) {
            Horse horse = new Horse(barrier);
            horses.add(horse);
            exec.execute(horse);
        }
    }

    public static void main(String[] args) {
        int nHorses = 7;
        int pause = 200;

//        if (args.length > 0) {
//            int n = new Integer(args[0]);
//            nHorses = n > 0 ? n : nHorses;
//        }
        new HorseRace(nHorses, pause);

    }
}
```

可以向CyclicBarrier提供一个“栅栏动作”，他是一个Runnable,当计数值到达0时自动执行--这是CyclicBarrier和CountDownLatch之间的另一个区别。这里，栅栏动作是作为匿名内部类创建的，它被提交给了CyclicBarrier的构造器。

我试图让每匹马都打印自己，但是之后的显示顺序取决于任务管理器。CyclicBarrier使得每匹马都要执行为了向前移动所必需执行的所有工作，然后必须在栅栏处等待其他所有的马都准备完毕。当所有的马都向前移动时，CyclicBarrier将自动调用Runnable栅栏动作任务，按顺序显示马和终点线的位置。

一旦所有的任务都越过了栅栏，它就会自动的为下一回合的比赛做好准备。

为了展示这个非常简单的动画效果，你需要将控制台视窗的尺寸调整为小到只有马时，才会展示出来。

[链接:Java并发之CountDownLatch](http://linkyan.com/2013/05/countdownlatch/)
