# 线程阻塞

LockSupport 相比较suspend会造成永久被挂起，它是安全的，即使unpark()方法再在前面执行也无所谓，因为它的底层使用的信号量，为每一个线程准备了许可，如果许可可用，则park惠消费它，如果不可以不可用则阻塞等待，而unpark是将一个许可变成可用，但是和信号量有些许区别，它是不可以累加的，每次最多有一个许可。

```java
public class LockSupportDemo {
    static Runnable r = () -> {
        try {
            Thread.sleep(10);
            System.out.println(Thread.currentThread().getName() + "执行");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        LockSupport.park();
    };

    public static void main(String[] args) throws Exception {
        Thread t1 = new Thread(r, "1");
        Thread t2 = new Thread(r, "2");
        t1.start();
        t2.start();
        LockSupport.unpark(t1);
        LockSupport.unpark(t2);
        t1.join();
        t2.join();
    }
}

```


