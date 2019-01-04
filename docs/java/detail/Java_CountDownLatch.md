## CountDownLatch用法

### 利用CountDownLatch等待所有任务完成再执行其他逻辑

```java
package com.ibingbo.test;

import java.util.concurrent.CountDownLatch;

/**
 * Created by bing on 2017/2/8.
 */
public class CountDownLatchTest {

    public static void main(String[] args) throws InterruptedException {
        //任务记数器为10
        CountDownLatch latch = new CountDownLatch(10);
        for (int i=0;i<10;i++) {
            new Thread(new Worker(latch,8000-i*100,i)).start();
        }
        //等待所有任务执行完成
        latch.await();
        System.out.println("all done");
    }

    static class Worker implements Runnable {
        //计数器
        private CountDownLatch latch;
        private int workTime;
        private int num;

        public Worker(CountDownLatch latch, int workTime,int num) {
            this.latch = latch;
            this.workTime = workTime;
            this.num = num;
        }

        @Override
        public void run() {
            try {
                Thread.sleep(workTime);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println(this.num+" done");
            //该任务执行完，计数器减一
            this.latch.countDown();
        }
    }

}
```

### 利用CountDownLatch实现多线程并发执行，即模拟并发请求

```java
package com.ibingbo.test;

import java.util.concurrent.CountDownLatch;

/**
 * Created by bing on 2017/2/16.
 */
public class CountDownLatchTest2 {

    static class WorkerExt implements Runnable {
        //启动所有线程闸门
        private  CountDownLatch startCdl;
        //所有连接都准备就绪控制器
        private  CountDownLatch doneCdl;

        public WorkerExt(CountDownLatch startCdl, CountDownLatch doneCdl) {
            this.startCdl = startCdl;
            this.doneCdl = doneCdl;
        }

        @Override
        public void run() {
            try {
                //启动器等待
                startCdl.await();
                System.out.println(Thread.currentThread().getName() + " has been working!!!!");
                Thread.sleep(30000);
                System.out.println(Thread.currentThread().getName() + " has been done!!!!");
                //就绪控制器计数开始计数
                this.doneCdl.countDown();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        //启动闸门值为1
        CountDownLatch startCdl = new CountDownLatch(1);
        //连接总数为100
        CountDownLatch doneCdl = new CountDownLatch(100);
        for (int i=0;i<100;i++) {
            new Thread(new WorkerExt(startCdl, doneCdl), "Thread" + i).start();
        }
        //记录所有连接的开始时间，纳秒
        long start = System.nanoTime();
        //所有线程准备就绪，开始闸门才开始
        startCdl.countDown();
        try {
            //等待所有线程准备就绪
            doneCdl.await();
            System.out.println("start running all worker....");

        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        long end = System.nanoTime();
        System.out.println("the task takes times(ms): " + (end - start) / 100000);
    }
}
```
