# 信号量

Semaphore 和操作系统的信号量很类似，但是这里它也可以作为锁使用。即可以作为锁与协调线程作用

* 锁

```java
public class SemaphoreDemo {
    static Semaphore semaphore = new Semaphore(1,true);//一个信号量且是公平的，默认非公平
    static Runnable r = () -> {
        try {
            System.out.println(Thread.currentThread().getName() + "等待");
            semaphore.acquire();
            System.out.println(Thread.currentThread().getName()+"进入");
            semaphore.release();
            System.out.println(Thread.currentThread().getName() + "释放");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    };

    public static void main(String[] args) {
        Thread t1 = new Thread(r,"1");
        Thread t2 = new Thread(r,"2");
        Thread t3 = new Thread(r,"3");
        t1.start();
        t2.start();
        t3.start();
    }
}
```

打印结果	
2等待
1等待
3等待
2进入
2释放
1进入
1释放
3进入
3释放

> 可以看到首先3个线程同时到来临界区边界，但是同时只有一个线程能进入临界区。

* 协调线程

```java
public class SemaohoreDemo2 {
    public static void main(String[] args) {
        ExecutorService pool = Executors.newCachedThreadPool();
        final Semaphore semaphore = new Semaphore(3, true);
        for (int i = 0; i < 6; i++) {
            Runnable runnable = () -> {
                try {
                    semaphore.acquire();//获取信号灯许可
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                try {
                    Thread.sleep(2000);//为了打印，进入临界区总的线程数量
                    System.out.println(Thread.currentThread().getName()+"进入临界区");
                    Thread.sleep(4000);//模拟业务逻辑处理
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                System.out.println(Thread.currentThread().getName()+"释放信号");
                semaphore.release();//释放信号灯

            };
            pool.execute(runnable);

        }
        pool.shutdown();
    }
}
```

打印结果	
pool-1-thread-3进入临界区	
pool-1-thread-2进入临界区	
pool-1-thread-1进入临界区	
pool-1-thread-3释放信号		
pool-1-thread-1释放信号	
pool-1-thread-2释放信号	
pool-1-thread-4进入临界区	
pool-1-thread-5进入临界区	
pool-1-thread-6进入临界区	
pool-1-thread-4释放信号	
pool-1-thread-5释放信号	
pool-1-thread-6释放信号	

> 从结果可以看到有3个线程同时进入了临界区。