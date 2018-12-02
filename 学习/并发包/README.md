# 并发包

 * 一、可重入锁 ReentrantLock		 
 
 >线程基础里面我介绍了线程保持同步，使用的是synchronized 关键字配合wait和notify来使用，JDK5 以后提供的并发包里ReentrantLock 完全可以替代他们。
 *  演示个累加例子
   
	```java 
	public class ReentryLock {
	    static int sum = 0;
	
	    public static void main(String[] args) {
	        ReentrantLock reentrantLock = new ReentrantLock();
	        Runnable r = () -> {
	            for (int i = 0; i < 100; i++) {
	                reentrantLock.lock();
	                sum++;
	                reentrantLock.unlock();
	            }
	        };
	        Thread t1 = new Thread(r);
	        Thread t2 = new Thread(r);
	        t1.start();
	        t2.start();
	        try {
	            t1.join();
	            t2.join();
	        } catch (InterruptedException e) {
	            e.printStackTrace();
	        }
	        System.out.println(sum);
	    }
	 }   
	```  
	* 运行结果：200     
 * 关键方法/功能
    * 可以中断响应
    * 锁申请时等待限时
    * 公平锁
 * 类比		
  - **可以中断：**synchronized在等待锁时候，要没等待，要么获得锁只从代码，而ReentryLock可以优雅的中断等待。小明周末打电话给好朋友小刚取打篮球，小刚说自己有点事需要等下，好了给小明电话，于是小明等待小刚的电话，等待过程中小明突然接到女朋友电话，她买好了电影票，电影还有半个小时开场了，这下肯定去陪女朋友，不然单身一辈子了，于是立马电话和小刚说自己有急事不去打篮球了。相反小刚手头事一时半会忙不完，也可以电话给小明，今天不能去打篮球了。这就是中断的好处。下面看看这个简答的例子：
    
 ```java 
 public class ReentryLockInter {
    static ReentrantLock lock = new ReentrantLock();//小明
    static ReentrantLock lock2 = new ReentrantLock();//小刚
    static boolean isInterrupt = false;
    static String friend ;

    private static void setInterrupt(boolean isInterrupt,String friend) {
        ReentryLockInter.isInterrupt = isInterrupt;
        ReentryLockInter.friend = friend;
    }

    public static void main(String[] args) {
        Runnable r = () -> {
            if (isInterrupt) {
                try {
                    lock.lockInterruptibly();
                    System.out.println(Thread.currentThread().getName() + "突然有急事，如是打电话给"+friend+"，今天有事，不能去打篮球了");
                    lock2.lockInterruptibly();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    if(lock.isHeldByCurrentThread())lock.unlock();
                    if(lock2.isHeldByCurrentThread())lock2.unlock();
                }
            }
        };
        Thread t1 = new Thread(r);
        t1.setName("小明");
        setInterrupt(true,"小刚");
        Thread t2 = new Thread(r);
        t2.setName("小刚");
        setInterrupt(true,"小明");
        t1.start();
        t2.start();
    }
}
```
 	* 打印结果		
   小刚突然有急事，如是打电话给小明，今天有事，不能去打篮球了   
   小明突然有急事，如是打电话给小明，今天有事，不能去打篮球了
   
- **锁申请时等待限时**：这个功能可以很好的·防止死锁的产生。还是拿小明周末约小刚打篮球的事情，这次小明女朋友想要他下午5点去陪她看电影，而目前是早上8点，还有好几个小时，于是小明可以等小刚一段时间忙好，相约去打篮球。
   
 ```java
public class ReentryLockTest3 {
    static ReentrantLock lock = new ReentrantLock();//小明
    static ReentrantLock lock2 = new ReentrantLock();//小刚
    static int time = 0;
    static String friend;

    private static void setInterrupt(int time, String friend) {
        ReentryLockTest3.time = time;
        ReentryLockTest3.friend = friend;
    }

    public static void main(String[] args) {
        Runnable r = () -> {
            try {
                if (lock.tryLock(time, TimeUnit.HOURS)) {
                    System.out.println(Thread.currentThread().getName() + "等" + friend + "一起去打篮球");
                } else {
                    lock.lockInterruptibly();
                    lock2.lockInterruptibly();
                    System.out.println(Thread.currentThread().getName() + "打电话给" + friend + "说，等你太久了，今天不去打篮球了");
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                if (lock.isHeldByCurrentThread()) lock.unlock();
                if (lock2.isHeldByCurrentThread()) lock2.unlock();
            }
        };
        Thread t1 = new Thread(r, "小明");
        setInterrupt(5, "小刚");
        t1.start();
        Thread t2 = new Thread(r, "小明");
        setInterrupt(0, "小刚");
        t2.start();
    }
}
```  
* 打印结果		
	小明等小刚一起去打篮球		
	小明打电话给小刚说，等你太久了，今天不去打篮球了   
	
- **公平锁：**小明因为女朋友，取消了和小刚约一起去打篮球，来到电影院，今天电影院人真多，原来是一个当红电影明星来宣传电影，需要排队进场，如是他和女朋友去一起排队，大家都是先来先到顺序进场。想想如果此时有个人搞特权，在队伍的最后面被安排先进场了，大家肯定不爽。这就是公平和非公平锁。公平锁实现成本较高，因为它的实现是维护一个有序队列

```java
public class ReentryLockTest4 {
    static ReentrantLock fairLock = new ReentrantLock(true);//默认不写为false

    static Runnable enteringCinema = () -> {
        try {
            fairLock.lock();
            System.out.println(Thread.currentThread().getName() + "进场");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (fairLock.isHeldByCurrentThread()) fairLock.unlock();
        }
    };

    public static void main(String[] args) {
        Thread t1 = new Thread(enteringCinema, "路人甲");
        Thread t2 = new Thread(enteringCinema, "小明");
        Thread t3 = new Thread(enteringCinema, "小明女朋友");
        Thread t4 = new Thread(enteringCinema, "路人乙");
        t1.start();
        t2.start();
        t3.start();
        t4.start();
    }
}
```

* 打印结果		
	路人甲进场	
	小明进场	
	小明女朋友进场		
	路人乙进场	
	
* Condition，ReentrantLock好帮手

```java
public class ConditionDemo {
    static ReentrantLock lock = new ReentrantLock();
    static Condition condition = lock.newCondition();


    static Runnable r = () -> {
        lock.lock();
        try {
            System.out.println("执行condition.await()方法，暂停方法");
            condition.await();
            System.out.println("恢复执行");//end
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }

    };

    public static void main(String[] args) {
        Thread t1 = new Thread(r);
        t1.start();
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        lock.lock();
        condition.signal();
        lock.unlock();//若这个被注释，则end执行不到，得到的和suspend效果一样，线程被长久挂起，很危险
        System.out.println("执行condition.signal()方法");

    }
}
```	
	
   