---
category: Android
date: 2013-05-14
layout: post
title: Java并发之CountDownLatch
---
书摘：

CountDownLatch被用来同步一个或多个任务，强制他们等待由其他任务执行的一组操作完成。

你可以向CountDownLatch对象设置一个初始计数值，任务在这个对象上调用wati()的方法都将阻塞，知道这个计数值达到0。其他任务在结束其工作时，可以在该对象上调用countDown()来减少这个计数值。CountDownLatch被设计为只触发一次，计数值不能被重置。如果你需要能够重置计数值的版本，则可以使用[CyclicBarrier](http://linkyan.com/4000/2013/05/cyclicbarrier/)。

调用countDown()的任务在产生这个调用时并没有被阻塞，只有对await()的调用会被阻塞，直到计数值达到0。

CountDownLatch的典型用法是将一个程序分为n个互相独立的可解决任务，并创建值为0的CountDownLatch。当每个任务完成是，都会在这个锁存器上调用countDown()。等待问题被解决的任务在这个锁存器上调用await()，将他们自己拦住，知道锁存器计数结束。下面演示这种技术的一个框架示例：


```
class TaskProtion implements Runnable {
    private static int counter = 0;
    private final int id = counter++;
    private static Random rand = new Random(47);
    private final CountDownLatch latch;

    TaskProtion(CountDownLatch latch) {
        this.latch = latch;
    }

    @Override
    public void run() {
        //ToDo
        try {
            dowork();
            latch.countDown();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public void dowork() throws InterruptedException {
        TimeUnit.MILLISECONDS.sleep(rand.nextInt(2000));
        System.out.println(this + "completed");
    }

    @Override
    public String toString() {
        return String.format("%1$-3d", id);
    }
}

class WaitingTask implements Runnable {
    private static int counter = 0;
    private final int id = counter++;
    private final CountDownLatch latch;

    WaitingTask(CountDownLatch latch) {
        this.latch = latch;
    }

    @Override
    public void run() {
        //ToDo
        try {
            latch.await();
            System.out.println("Latch barrier passed for " + this);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    @Override
    public String toString() {
        return String.format("WaitingTask %1$-3d ", id);
    }
}

public class CountDownLatchDemo {
    static final int SIZE = 100;

    public static void main(String[] args) throws Exception {
        ExecutorService exec = Executors.newCachedThreadPool();
        CountDownLatch latch = new CountDownLatch(SIZE);

        for (int i = 0; i < 10; i++) {
            exec.execute(new WaitingTask(latch));
        }
        for (int i = 0; i < SIZE; i++) {
            exec.execute(new TaskProtion(latch));
        }

        System.out.println("Launched all tasks ");
        exec.shutdown();

    }
}
```

TaskPortion将随即地休眠一段时间，以模拟这部分工作的完成，而WaitingTask表示系统中，必须等待的部分，它要等待到问题的初始部分完成为止。所有任务都使用了在main()中定义的同一个单一的CountDownLatch。

[链接:Java并发之CyclicBarrier](http://linkyan.com/2013/05/cyclicbarrier/)
