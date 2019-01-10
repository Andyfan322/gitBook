# 线程池工厂

>Executors面对开发人员的创建线程池的工厂类，它和线程执行器ExecutorService关系很紧密。

![](/Users/huangfan/Desktop/Executors方法.png)

可以看到，提供了很多静态的方法，我们主要关注以下方法：

* newFixedThreadPool(int nThreads) 固定线程数量的线程池
* newScheduledThreadPool(int corePoolSize) 拥有至少corePoolSize数量的线程池，可以延迟或者固定频率执行
* newCachedThreadPool() 线程不固定线程池，常用于多且小的异步任务使用
* newSingleThreadExecutor() 串行按照提交顺序执行任务，的单线程化线程池
* newWorkStealingPool() 任务窃取线程池，JDK1.8以后加的，充分利用现有的cpu资源创建线程池,一个线程维护一个任务队列，常用在任务执行时间差别较大的业务

```java 
public class ExecutorsDemo {

    private static ExecutorService service = Executors.newCachedThreadPool();
    private static ExecutorService service2 = Executors.newFixedThreadPool(3);
    private static ExecutorService service3 = Executors.newScheduledThreadPool(3);
    private static ExecutorService service4 = Executors.newSingleThreadExecutor();
    private static ExecutorService service5 = Executors.newWorkStealingPool();

    public static void main(String[] args) throws Exception {
        addTask("newCachedThreadPool", service);
        addTask("newFixedThreadPool", service2);
        addTask("newScheduledThreadPool", service3);
        addTask("newSingleThreadExecutor", service4);
        addTask("newWorkStealingPool", service5);
    }

    private static void addTask(String type, ExecutorService service) throws Exception {
        for (int i = 0; i < 10; i++) {
            service.execute(() -> System.out.println(type + "," + Thread.currentThread().getName()));
        }
        service.shutdown();
        service.awaitTermination(5, TimeUnit.MILLISECONDS);//等待任务完成再关闭主main线程
    }
}
```

> **注意：**虽然service.shutdown()是关闭线程，此时已经提交的线程会被执行完毕，但是打印需要时间，所以再最后加了一句 service.awaitTermination(5, TimeUnit.MILLISECONDS)，目的是为了看到打印消息。

* ScheduledExecutorService
 
  ![](/Users/huangfan/Desktop/ScheduleExecutorServicel类.png)

这里面我介绍下ScheduledExecutorService接口里面的4个方法

```java
public interface ScheduledExecutorService extends ExecutorService {

public <V> ScheduledFuture<V> schedule(Callable<V> callable,long delay, TimeUnit unit); 

public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,long initialDelay,long period,TimeUnit unit);

public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay,long delay,TimeUnit unit);

}
```







  