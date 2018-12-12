# 线程池

线程池和数据库的连接吃鬼脸很像，我们不必要重复的创建线程，这是耗性能的也是不明智的选择。JUC提供了一整套Executor框架。是的线程的控制得到了很好支持。

* Executor执行器	
	 * 下属接口有：ExecutorService, ScheduledExecutorService
	 * 实现类：AbstractExecutorService, ThreadPoolExecutor, ScheduledThreadPoolExecutor
  
![](http://47.95.12.0:3389/ftp/Executor 类结构.png)


* ExecutorService
   这是一个很重要的接口，关键的实现类有ThreadPoolExecutor
   
   ![](http://47.95.12.0:3389/ftp/ExecutorService类机构.png)
   
   * execute(Runnable r) 接受一个Runnable，以异步的方式运行，没有返回结果
   * submit(Runnable r) 接受一个Runnable,以异步的方式运行，返回结果Future，用来判断任务是否执行完毕
   * submit(Callable c) 接受一个Callable，以异步方式运行，返回Future，Callable返回值从这里获取
   * inVokeAny(Collection<? extends Callable<T>>) 返回Callable某一个对象的结果，且不固定
   * invokeAll(Collection<? extends Callable<T>>) 返回所有的Callable对象

```java 
public class ExecutorServiceDemo {
    private static ExecutorService service = Executors.newFixedThreadPool(10);
    private static Runnable r = () -> System.out.println("我是 Runnable 线程" + Thread.currentThread().getName());
    private static Callable c = () -> "我是Callable 线程" + Thread.currentThread().getName();
    static CountDownLatch countDownLatch = new CountDownLatch(10);

    public static void main(String[] args) throws Exception {
        List<Callable<String>> callableList = new ArrayList<>();
        countDownLatch.await();
        for (int i = 0; i < 10; i++) {
//            Future<?> future = service.submit(r);
//            Future<?> future2 = service.submit(c);
            callableList.add(c);
            countDownLatch.countDown();
//            System.out.println(future.isDone());
//            System.out.println(future2.get());
        }
        List<Future<String>> futures = service.invokeAll(callableList);
        for (int i = 0; i < 10; i++) {
            System.out.println(futures.get(i).isDone());
        }
        service.shutdown();
    }
}
```