# 倒计时计数器

CountDownLatch 为一个倒计时计数器，可以将线程固定在某几个程序运行结束后再运行，比如我们航天工程里面火箭发射，前置很多检查需要都都准备就绪才能发射。

```java 
public class CountDownLatchDemo {
    static CountDownLatch countDownLatch = new CountDownLatch(5);
    static ReentrantLock lock = new ReentrantLock();
    static AtomicInteger count = new AtomicInteger(1);

    public static void main(String[] args) throws Exception {
        ExecutorService service = Executors.newCachedThreadPool();
        Thread t = new Thread(() -> {
            try {
                lock.lock();
                Thread.sleep(100);
                System.out.println("火箭发射前设备状态检查" + count.getAndIncrement());
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
            countDownLatch.countDown();
        });
        for (int i = 0; i < 5; i++) {
            service.execute(t);
        }
        countDownLatch.await();
        System.out.println("设备状态检查执行完了，开始发射！");
    }
}

```

* 打印结果		
	* 火箭发射前设备状态检查1	
	* 火箭发射前设备状态检查2	
	* 火箭发射前设备状态检查3	
	* 火箭发射前设备状态检查4	
	* 火箭发射前设备状态检查5	
	* 设备状态检查执行完了，开始发射！