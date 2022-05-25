# Java并发的那些事儿(三)之JDK并发包



重入锁、Condition 条件、信号量、CountDownLatch、CyclicBarrier、LockSupport



### 重入锁

> 管理同步资源的实现类，可用于替换 synchronized关键字。

使用 ReentrantLock 类实现。

#### 使用方式

```java
for(int j=0;j<10000;j++) {
    lock.lock();        // 加锁保护
    try {
        i++;
    }finally {
		lock.unlock();
    }
}
```



创建：ReentrantLock lock = new ReentrantLock();

```java
	 public ReentrantLock() {
        sync = new NonfairSync();
    }

    /**
     * fair=true ,表示公平锁；锁的获取按照时间的先后顺序
     */
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```



加锁：lock.lock();

解锁：lock.unLock();

锁中断：lock.lockInterruptibly() ，使用该方法加锁表示这个锁是可以被中断的



ReentrantLock 被称为重入锁，是因为这个锁是可以被同一个线程反复进入的。

```java
lock.lock();
lock.lock();
try {

}finally {
	lock.unLock();	// 解锁的个数需与加锁个数一致，否则该线程将一直持有该锁资源
	lock.unLock();
}
```



锁申请等待限时：lock1.tryLock(5, TimeUnit.MINUTES)，表示尝试5分钟去获取锁，如果成功获取到锁，则返回true，否则返回false	



#### ReentrantLock  与synchronized的区别

1、ReentrantLock  是Java实现类，依赖于API；synchronized 是关键字，由虚拟机实现；

2、ReentrantLock  对锁的添加与释放可控，synchronized 不可控；

3、ReentrantLock  存在公平锁；

4、ReentrantLock  等待锁可中断，同时提供了限时获取锁的方法；



### Condition 条件

> 配合ReentrantLock  重入锁的接口类，用于控制线程的暂停与执行，类比Object.wait() 与 Object.notify() 方法

#### 使用方式

```java
public class ConditionDemo implements Runnable {

    public static ReentrantLock lock = new ReentrantLock();
    public static Condition condition = lock.newCondition();

    @Override
    public void run() {
        try {
            lock.lock();
            System.out.println("目标线程获取到锁");
            condition.await();      // 使线程等待，同时释放锁，能被中断
//            condition.awaitUninterruptibly();   // 与上面的方式基本一致，只是不能被中断
            System.out.println("目标执行方法结束");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ConditionDemo demo = new ConditionDemo();

        Thread thread = new Thread(demo);
        thread.start();

        Thread.sleep(1000); // 保证目标线程先拿到锁
        lock.lock();
        condition.signal();         // 唤醒线程，系统会从当前Condition对象的等待队列中唤醒一个线程
        System.out.println("主线程获取该锁");
        lock.unlock();

    }
}

```



#### 常用方法



使用`condition.await();` 和  ` condition.signal(); ` 时，都需要先获取相关的锁。



 在JDK内部常被用于堵塞队列，可参考 `ArrayBlockingQueue`





### 信号量 Semaphore

> 信号量可以说是锁的扩展。信号量可以指定多个线程同时访问某一个资源。

​	使用场景一般是需要使用多线程做处理时，但场景有限制同时工作的线程数，使用信号量可以做流量控制。例如文件读取。

​	在很多情况下，可能有多个线程需要访问数目很少的资源。假想在服务器上运行着若干个回答客户端请求的线程。这些线程需要连接到同一数据库，但任一时刻只能获得一定数目的数据库连接。这时候使用信号量就可以大大提高效率和性能了。

#### Demo

```java
public class SemaphoreDemo implements Runnable {

    final Semaphore semaphore = new Semaphore(5,true);   // 5个许可，即同时允许5个线程访问同一资源

    @Override
    public void run() {
        try {
            semaphore.acquire();    // 尝试获取许可

            Thread.sleep(2000); // 模拟耗时

            System.out.println(Thread.currentThread().getId() + "-done");

            semaphore.release();    // 释放许可

        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        SemaphoreDemo semaphoreDemo = new SemaphoreDemo();

       ExecutorService service = Executors.newFixedThreadPool(20);
       for(int i=0;i<20;i++) {  // 提交20个任务，每次最多会有5个线程进入
           service.submit(semaphoreDemo);
       }
    }
}
```





### 倒计时器：CountDownLatch

> 多线程控制工具类。通常用来控制线程等待，直到等待倒计时结束，再开始执行。



#### 使用方式

​	设定一个count 倒计时，构建CountDownLatch 对象，每完成一个任务时调用 CountDownLatch对象的countDown() 方法，完成count 个任务后，才会执行 await() 后面的逻辑。



#### Demo

```java
public class CountDownLatchDemo implements Runnable{

    private static final int count = 10;

    private static CountDownLatch countDownLatch = new CountDownLatch(count);  // 10个线程执行完即可


    @Override
    public void run() {
        System.out.println(Thread.currentThread().getId() + "准备完毕");
        countDownLatch.countDown();
    }

    public static void main(String[] args) {

        CountDownLatchDemo demo = new CountDownLatchDemo();

        ExecutorService exec = Executors.newFixedThreadPool(count);
        for (int i = 0; i <= count; i++) {
            exec.submit(demo);
        }

        try {
            countDownLatch.await();	// 等待检查
            System.out.println("准备工作完毕，开始执行");
            exec.shutdown();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }
}

```



运行结果：

[![btBQQs.png](https://s4.ax1x.com/2022/03/03/btBQQs.png)](https://imgtu.com/i/btBQQs)



ps： 如果提交的线程数不足10个，那主线程将一直等待，直到倒计时结束。



#### 与 join 方法的区别

CounDownLatch 与join类似，都可实现线程等待。

1. 使用粒度不同。join 使用起来更方便，而`CountDownLatch`的粒度会更细。为什么要说`CountDownLatch`粒度更细呢。因为`CountDownLatch` 可以让当前线程只等待执行一部分的情况下就继续执行了，而`join` 必须等待线程完全执行完毕才可以。  （`countDownLatch.countDown()` 的方法可在线程中任一位置调用）
2. 原理不同。`join` 底层是使用`Object.wait(0) `方法进行等待的，当线程执行完毕后会调用`notifyAll `方法唤醒线程。而`CountDownLatch` 依靠初始化时的count变量来计算，每当执行一次 ` countDown` 方法表示 count变量减1，`await()` 方法就是去判断这个变量值是否为0，如果是则表示所有操作已完成，否则继续等待。



### 循环栅栏：CyclicBarrier

> CyclicBarrier是另一种多线程并发控制工具。与CounDownLatch 相似，也可以实现线程间的计数等待。但CyclicBarrier可以实现更复杂的功能。栅栏可以理解为障碍，负责堵塞线程。解除线程堵塞的关键在于倒计时结束，当倒计时结束时即会执行障碍任务。



使用场景：通常将一个问题拆分成一系列相互独立的子问题。



#### 使用方法

1. 构建 CyclicBarrier 对象，分配count 倒计时数  以及 障碍任务；
   * 障碍任务是每一次调用 CyclicBarrier 对象的await() 方法后执行线程任务；
2. 调用 CyclicBarrier 对象的await() 方法堵塞，直到满足count 个线程才会继续执行。
   * 在满足count后，会执行构建 CyclicBarrier 对象时设定的 障碍任务；
3. 下一次使用 wait() 方法会继续从0开始堵塞等待，直到满足count个线程，重复前面的动作；
4. 如果await 的调用超时，或者await 堵塞的线程被打断，那么栅栏就被认为打破了，所有堵塞的await 调用都将终止并抛出 BrokenBarrierException.



#### 构造方法

```java
 /**
 * @param parties，倒计时数，当线程数满足这个数量时，会执行 barrierAction参数的线程
 * @param barrierAction，满足await条件时执行的任务线程
 **/
 public CyclicBarrier(int parties, Runnable barrierAction) {
        if (parties <= 0) throw new IllegalArgumentException();
        this.parties = parties;
        this.count = parties;
        this.barrierCommand = barrierAction;
    }
```



```
await();	// 倒计时等待的方法，倒计时统计的是执行的线程数。
```





#### Demo

```java
public class CyclicBarrierDemo {

    public static class Solider implements Runnable {

        private String name;
        private final CyclicBarrier cyclicBarrier;

        public Solider(String name, CyclicBarrier cyclicBarrier) {
            this.name = name;
            this.cyclicBarrier = cyclicBarrier;
        }

        @Override
        public void run() {

            try {
                cyclicBarrier.await();

                // doWork
                Thread.sleep(1000);
                System.out.println(name + "任务完成！");

                cyclicBarrier.await();

            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }


        }
    }


    public static class BarrierTask implements Runnable {

        boolean flag;
        int n;

        public BarrierTask(boolean flag, int n) {
            this.flag = flag;
            this.n = n;
        }

        @Override
        public void run() {
            if(flag) {
                System.out.println("司令：[士兵" + n + "个，任务完毕！]");
            }else {
                flag = true;
                System.out.println("司令：[士兵" + n + "个，集合完毕！]");
            }
        }
    }

    public static void main(String[] args) {

        Thread[] allSolider = new Thread[10];
        CyclicBarrier barrier = new CyclicBarrier(10,new BarrierTask(false,10));
        System.out.println("集合队伍！");
        for (int i = 0; i < 10; i++) {
            System.out.println("士兵" + i + "报道");
            allSolider[i] = new Thread(new Solider("士兵" + i,barrier));
            allSolider[i].start();
        }

    }

}

```



运行结果

```
集合队伍！
士兵0报道
.....省略
士兵9报道
司令：[士兵10个，集合完毕！]
士兵2任务完成！
.....省略
士兵4任务完成！
司令：[士兵10个，任务完毕！]
```





### 线程堵塞工具类：LockSupport

> 线程堵塞工具，可以在线程内任意位置让线程堵塞。与Thread.suspend()类似可以让线程暂停挂起，不同的是使用LockSupport 不需要先获取锁，也不用担心 Thread.resume() 唤醒方法发生在暂停方法之前从而导致线程一直堵塞的问题。



#### 常用方法

方法为静态方法

```java
LockSupport.park();	// 在当前线程堵塞
LockSupport.unpark(t1); // 唤醒t1线程
```



#### Demo

```java
public class LockSupportDemo {

    public static Object object = new Object();

    static ChangeObjectThread t1 = new ChangeObjectThread("t1");
    static ChangeObjectThread t2 = new ChangeObjectThread("t2");

    public static class ChangeObjectThread extends Thread {

        public ChangeObjectThread(String name) {
          super.setName(name);
        }

        @Override
        public void run() {
            synchronized (object) {
                System.out.println("in:" + getName());
                LockSupport.park(); // 挂起线程
                System.out.println(getName() +  "：end;");
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        t1.start();
        Thread.sleep(1000);
        t2.start();
        LockSupport.unpark(t1); // 唤醒t1线程
        LockSupport.unpark(t2);
        t1.join();
        t2.join();
        System.out.println("end");
    }


}
```









TODO



synchronized的优化、CAS、Threadlocal、 AQS原理

