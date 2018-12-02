# 线程基础
> 线程是什么，创建方式，这里不不做介绍。

* stop方法（Deprecated）

   stop()会放弃持有的线程的锁，假设在为多个变量赋值的时候，刚好赋值到中间，执行了stop方法，很容易导致数据的不一致，此方法已经被废弃，尽量不要用，下面演示个demo
   
``` java 
public class ThreadTest {
    static Integer i, j;

    static Runnable r1 = () -> {
        while (true) {
                i = new Random().nextInt();
                j = i;
        }
    };

    static Runnable r2 = () -> {
        while (true) {
            if (i != j) {
                System.out.println("i:" + i + ",j:" + j);
            } else {
                System.out.println("00000");
            }
        }
    };

    public static void main(String[] args) {
        Thread read = new Thread(r2);
        read.start();
        while (true) {
            Thread write = new Thread(r1);
            write.start();
            try {
                Thread.sleep(25);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            write.stop();//Deprecated
        }
    }
}
```

 * 打印结果  

	00000  
	00000  
	i:2071039910,j:1508352969  
	i:-1148719051,j:-743674023  
	00000	
	00000	
	
* 线程中断来替代stop	

 >上面代码读取的不一致，核心原因是一以为线程突然停止，那么有没有一种优雅的停止呢，JDK为我们提供了更优雅的方式，中断。我们将上述代码稍做改变，数据的一致性立马可以得到保证。
  
 ```java
 public class ThreadTest {
    static Integer i, j;

    static Runnable r1 = () -> {
        while (true) {
            if(Thread.currentThread().isInterrupted()) {//判断是否有中断标志
                i = new Random().nextInt();
                j = i;
                break;
            }
        }
    };

    static Runnable r2 = () -> {
        while (true) {
            if (i != j) {
                System.out.println("i:" + i + ",j:" + j);
            } else {
                System.out.println("00000");
            }
        }
    };

    public static void main(String[] args) {
        Thread read = new Thread(r2);
        read.start();
        while (true) {
            Thread write = new Thread(r1);
            write.start();
            try {
                Thread.sleep(25);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            write.interrupt();// 设置中断标志
        }
    }
}
```

* 结果	
00000	
00000	
···

**注意点：若是被设置了中断的线程使用了sleep方法，则运行时可能会抛出中断异常且中断标志会被清空，这里贴出异常，代码就不写出来了。**

```java
java.lang.InterruptedException: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at com.andyfan.thread.ThreadTest.lambda$static$0(ThreadTest.java:20)
	at com.andyfan.thread.ThreadTest$$Lambda$1/558638686.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:745)
```

* 等待、唤醒 wait/notify		

> wait和notify时Object里面定义的方法。

* 工作原理：obj.wait()，线程进入等待队列，obj.notify() 则从等待队列里面随机选择一个线程继续执行等待前代码，意味着唤醒是非公平的。
* wait和notify的调用必须在synchronized 代码块里，这也意味着必须要获得对象的监视器（锁）。
* wait方法被调用后会主动释放监视器（锁），这个是sleep方法重要区分点。

```java
public class ThreadTest2 {
    final static Object object = new Object();

    static Runnable r1 = () -> {
        synchronized (object) {
            long start = 0l;
            try {
                System.out.println("wait方法被执行了");
                start = System.currentTimeMillis();
                object.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println();
            System.out.println("wait方法释放后被执行了,等待了notify方法释放锁" + (System.currentTimeMillis() - start)/1000  + "秒");
        }
    };

    static Runnable r2 = () -> {
        synchronized (object) {
            object.notify();
            System.out.println("notify方法被执行了");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("notify休眠1秒后");
        }
    };

    public static void main(String[] args) {
        Thread t1 = new Thread(r1);
        Thread t2 = new Thread(r2);
        t1.start();
        try {
            Thread.sleep(20);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        t2.start();
    }
}
```

* 打印结果	
	wait方法被执行了	
	notify方法被执行了		
	notify休眠1秒后		
	wait方法释放后被执行了,等待了notify方法释放锁1秒

* suspend 和resume (Deprecated）
  > suspend线程会被挂起，它是不释放监视器（锁）的，必须等到resume 才会释放锁。这2个方法也被废弃了，因为若resume在suspend前执行，会导致线程永远别挂起，自己和其他线程由于获取不到监视器根本没办法正常工作，下面演示个被一直被挂起例子


```java 
public class ThreadTest3 {
    final static Object object = new Object();

    static Runnable r = () -> {
        synchronized (object) {
            String name = Thread.currentThread().getName();
            System.out.println(name);
            Thread.currentThread().suspend();//Deprecated
            System.out.println(name + "被刮挂起后执行。。。");
        }
    };

    public static void main(String[] args) {
        Thread t1 = new Thread(r);
        Thread t2 = new Thread(r);
        t1.start();
        t2.start();
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        t1.resume();//Deprecated
        t2.resume();//在t2的suspend前执行，导致t2一直被挂起！
        try {
            t1.join();
            t2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

* 用jstack -l pid 命令 查看堆栈信息,可以看到线程t2一直处于Runnable状态	

```java
"Thread-1" #11 prio=5 os_prio=31 tid=0x00007fdbd703f800 nid=0x5903 runnable [0x000070000aa79000]
   java.lang.Thread.State: RUNNABLE
	at java.lang.Thread.suspend0(Native Method)
	at java.lang.Thread.suspend(Thread.java:1029)
	at com.andyfan.thread.ThreadTest3.lambda$static$0(ThreadTest3.java:14)
	- locked <0x000000076abf9f58> (a java.lang.Object)
	at com.andyfan.thread.ThreadTest3$$Lambda$1/558638686.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:745)
```	

* join和yield	
    * join 方法其实内部调用的是wait方法，JDK实现代码块：
   ```
       if (millis == 0) {
            while (isAlive()) {
                wait(0);
            }
        }
    ```
        
    * yield（谦让） 个人觉得这个方法使用需要勇气,有点鸡肋，因为它执行了，表明是让出当前CPU时间片，让和它同等优先级的线程优先执行，但是它持有的锁又不释放，共有资源还会再接下来时间竞争，且时间还不固定。
    
```java 
public class ThreadTest4 {
    static int j = 0;

    public static void main(String[] args) throws Exception {
        Thread thread = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                j = i;
            }
        });
        thread.start();
        thread.join();//main的输出，等待线程执行完再执行,不然输出可能是0或者很小的值
        System.out.println(j);
    }
}
```

