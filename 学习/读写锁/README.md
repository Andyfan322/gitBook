# 读写锁

ReadWriteLock 读写锁，提供了很好的读写分离，读读之间时并行，读写/写写之间串行，先看个例子

```java
public class ReadWriteLockDemo {
    final static ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
    final static Lock readLock = readWriteLock.readLock();
    final static Lock writeLock = readWriteLock.writeLock();

    public void read(Lock lock) throws Exception {
        Thread read = new Thread(() -> {
            lock.lock();
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        });
        read.start();
    }

    public void write(Lock lock) throws Exception {
        Thread write = new Thread(() -> {
            lock.lock();
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        });
        write.start();
    }

    public static void main(String[] args) throws Exception {
        ReadWriteLockDemo demo = new ReadWriteLockDemo();
        for (int i = 0; i < 10; i++) {
            demo.read(readLock);
            demo.write(writeLock);

        }
    }
}
```
ReadWriteLock 与ReentrantLock 之间作用不同，ReentrantLock任何读写之间都是串行，在读多写少的场景读写锁可以发挥很大的作用。