# 同步队列器

>AQS，AbstractQueuedSynchronizer 它是构建多或者其他同步组建的家畜框架，如ReentrantLock、ReentrantReadWriteLock、Semaphore等。
JUC的作者Doug Lea，期望它能够成为实现大部分同步需求的基础。它是JUC并发包中的核心基础组件。
AQS解决了实现同步器时候的大量细节，例如获取同步状态，FIFO同步队列。基于AQS构建同步器极大的减少实现工作，不必处理再多个位置上的竞争问题，
AQS主要是用的方式是继承，子类通过实现它的抽象方法类管理同步状态。

* 资源共享方式
    * Exclusive（独占，只有一个线程能执行，如ReentrantLock）
    * Share（共享，多个线程可同时执行，如Semaphore和CountDownLatch）

* 组成
    * Node节点，用于存放获取线程的节点，存在与     