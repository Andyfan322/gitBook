# 循环栅栏
CyclicBarrier 和CountDownLatch整体的作用差不多，都是协调线程在某个点触发以后的任务，但是CountDownLatch 没有循环的功能。比如：部队里面士兵需要区演练，首先是集合再做演练任务。看看下面的例子

```java
public class CyclicBarrierDemo {
    public static class Soldier implements Runnable {
        private String soldier;
        private final CyclicBarrier cyclic;

        public Soldier(String soldier, CyclicBarrier cyclic) {
            this.soldier = soldier;
            this.cyclic = cyclic;
        }

        @Override
        public void run() {
            try {
                cyclic.await();
                doWork();
                cyclic.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
        void doWork() {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(soldier + "任务完成");
        }
    }


    public static class BarrierRun implements Runnable {
        boolean flag;
        int N;

        public BarrierRun(boolean flag, int n) {
            this.flag = flag;
            N = n;
        }
        @Override
        public void run() {
            if (flag) {
                System.out.println("司令：[士兵)" + N + "个，任务完成！]");
            } else {
                System.out.println("司令：[士兵)" + N + "个，集合完毕！]");
                flag = true;
            }
        }
    }


    public static void main(String[] args) {
        final int N = 10;
        Thread[] allSoldier = new Thread[N];
        boolean flag = false;
        CyclicBarrier cyclicBarrier = new CyclicBarrier(N, new BarrierRun(flag, N));
        System.out.println("集合队伍！");
        for (int i = 0; i < N; i++) {
            System.out.println("士兵" + i + "报道！");
            allSoldier[i] = new Thread(new Soldier("士兵" + i, cyclicBarrier));
            allSoldier[i].start();
        }
    }
}
```
* 打印结果	
集合队伍！	
士兵0报道！	
士兵1报道！	
士兵2报道！	
士兵3报道！	
士兵4报道！	
士兵5报道！	
士兵6报道！	
士兵7报道！	
士兵8报道！	
士兵9报道！	
司令：[士兵)10个，集合完毕！]		
士兵9任务完成		
士兵7任务完成		
士兵6任务完成		
士兵0任务完成		
士兵8任务完成		
士兵2任务完成		
士兵4任务完成		
士兵3任务完成		
士兵1任务完成		
士兵5任务完成		
司令：[士兵)10个，任务完成！]	
