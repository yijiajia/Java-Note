# Java定时任务实现的几种方式

* 死循环定时调度法
* Timer类；
* ScheduledExecutorService
* Spring Task；
* Quartz
* 分布式定时调度中间件（XXL-JOB / ElasticJob）

## 死循环定时调度法

> 利用异步线程通过内部一直循环实现自动重复执行，利用sleep()休眠来实现定时。

```java
/**
 * 死循环定时调度法
 */
public class DeadCycleTimer implements Runnable{

    @Override
    public void run() {
        while (true) {
            System.out.println("定时任务开始");
            try {
                Thread.sleep(getSleepTime());   // 每天凌晨0点开始
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("do somethings");
            System.out.println("定时任务结束");
        }
    }

    private long getSleepTime() {
        LocalDateTime endDay = LocalDateTime.of(LocalDate.now(), LocalTime.MAX);
        return endDay.toEpochSecond(ZoneOffset.ofHours(8));
    }

    public static void main(String[] args) {
        // 启动运行
        DeadCycleTimer deadCycleTimer = new DeadCycleTimer();
        Thread thread = new Thread(deadCycleTimer);
        thread.start();
    }
}
```

缺点：`DeadCycleTimer`一直占用的cpu，其睡眠的时候是在无意义的消耗CPU；



## Timer 定时器

> 利用JDK自带的Timer实现，任务类通过继承TimerTask实现任务方法。

```java
public class TimerDemo extends TimerTask{

    @Override
    public void run() {
        System.out.println("定时任务开始");
        // do somethings
        try {
            Thread.sleep(1000); // 模拟运行
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("定时任务结束");
    }

    public static void main(String[] args) {
        /**
         * 每天执行一次
         */
        Timer timer = new Timer();
        timer.scheduleAtFixedRate(new TimerDemo(),1000, TimeUtils.getTomorrowSecond());
    }
}
```



缺点：`Timer`内部维护着任务队列，队列的实现跟最小根堆实现类似；

* 当提交的任务中若有抛出异常的，则整个定时器里的所有任务都会失效。
* 调度任务的线程只有一个，一旦有任务出现堵塞，那么就会影响到其它任务的执行。

## ScheduledExecutorService接口

> 利用JUC下的ScheduledExecutorService接口实现定时任务



```java
/**
 * ScheduledExecutorService类
 */
public class ExecutorsTask {

    /**
     * 指定时间一天执行一次
     */
    static class OneDayTaskAppointTime implements Runnable {

        @Override
        public void run() {
            System.out.println("定时任务开始；" + DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss").format(LocalDateTime.now()));
            // do somethings
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("定时任务结束");
        }
    }

    static class OtherTask implements Runnable {
        @Override
        public void run() {
            System.out.println("OtherTask" + DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss").format(LocalDateTime.now()));
            // do somethings
            System.out.println("OtherTask");
        }
    }


    public static void main(String[] args) {
        /**
         * 单线程提交多个任务，会交替执行
         */
        ScheduledExecutorService singleService = Executors.newSingleThreadScheduledExecutor();
        singleService.scheduleWithFixedDelay(new OneDayTaskAppointTime(),0,5, TimeUnit.SECONDS);
        singleService.scheduleWithFixedDelay(new OtherTask(),0,5, TimeUnit.SECONDS);

        /**
         * 多线程提交多个任务
         */
        ScheduledExecutorService service = Executors.newScheduledThreadPool(2);
        service.scheduleWithFixedDelay(new OneDayTaskAppointTime(),0,5, TimeUnit.SECONDS);
        service.scheduleWithFixedDelay(new OtherTask(),0,5, TimeUnit.SECONDS);

    }
}
```



与Timer的比较

|     Timer      |         ScheduledThreadPoolExecutor          |
| :------------: | :------------------------------------------: |
|   单线程阻塞   |              多线程任务互不影响              |
| 异常时任务停止 | 依赖于线程池，单个任务出现异常不影响其他任务 |



## Spring Task



