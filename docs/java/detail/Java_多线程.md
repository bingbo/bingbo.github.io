### 多线程优点

* 发挥多核CPU的优势，提高处理效率
* 防止单线程的阻塞问题
* 拆分小任务，便于建模

### 创建线程方法

* 继承Thread
* 实现Runnable
* 通过Callable和Future/FutureTask可以获取异步执行的结果

### volatile关键字

* valaitle关键字保证了其在多线程噗噗家人可见性，即每次读取到volatile变量，一定是最新的数据
* volatile不保证其原子性，并发性

### 线程安全

线程安全有以下几个级别：

1. 不可变

    像String、Integer、Long这些，都是final类型的类，任何一个线程都改变不了它们的值，要改变除非新创建一个，因此这些不可变对象不需要任何同步手段就可以直接在多线程环境下使用

2. 绝对线程安全

    不管运行时环境如何，调用者都不需要额外的同步措施。要做到这一点通常需要付出许多额外的代价，Java中标注自己是线程安全的类，实际上绝大多数都不是线程安全的，不过绝对线程安全的类，Java中也有，比方说CopyOnWriteArrayList、CopyOnWriteArraySet

3. 相对线程安全

    相对线程安全也就是我们通常意义上所说的线程安全，像Vector这种，add、remove方法都是原子操作，不会被打断，但也仅限于此，如果有个线程在遍历某个Vector、有个线程同时在add这个Vector，99%的情况下都会出现ConcurrentModificationException，也就是fail-fast机制。

4. 线程非安全

    这个就没什么好说的了，ArrayList、LinkedList、HashMap等都是线程非安全的类

### Java中如何获取到线程dump文件

死循环、死锁、阻塞、页面打开慢等问题，打线程dump是最好的解决问题的途径。所谓线程dump也就是线程堆栈，获取到线程堆栈有两步：

1. 获取到线程的pid，可以通过使用jps命令，在Linux环境下还可以使用ps -ef | grep java
1. 打印线程堆栈，可以通过使用jstack pid命令，在Linux环境下还可以使用kill -3 pid

另外提一点，Thread类提供了一个getStackTrace()方法也可以用于获取线程堆栈。这是一个实例方法，因此此方法是和具体线程实例绑定的，每次获取获取到的是具体某个线程当前运行的堆栈

### Thread.sleep(0)的作用是什么

这个问题和上面那个问题是相关的，我就连在一起了。由于Java采用抢占式的线程调度算法，因此可能会出现某条线程常常获取到CPU控制权的情况，为了让某些优先级比较低的线程也能获取到CPU控制权，可以使用Thread.sleep(0)手动触发一次操作系统分配时间片的操作，这也是平衡CPU控制权的一种操作

### synchronized和ReentrantLock的区别

synchronized是和if、else、for、while一样的关键字，ReentrantLock是类，这是二者的本质区别。既然ReentrantLock是类，那么它就提供了比synchronized更多更灵活的特性，可以被继承、可以有方法、可以有各种各样的类变量，ReentrantLock比synchronized的扩展性体现在几点上：

1. ReentrantLock可以对获取锁的等待时间进行设置，这样就避免了死锁
1. ReentrantLock可以获取各种锁的信息
1. ReentrantLock可以灵活地实现多路通知



## 通过wait和notify实现多个生产者与消费者

```java
package com.ibingbo.test.thread;

import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;

/**
 * Created by bing on 2017/3/1.
 */
public class MoreProducerAndComsumer {

    public static void main(String[] args) throws InterruptedException {
        List<String> list = new ArrayList<>();
        Object lock = new Object();
        //先初始化多个消费者线程
        for (int i=0;i<10;i++) {
            new Thread(new TConsumer(lock,list, i)).start();
        }

        //开始发送生产消息
        for (int j=0;j<100;j++) {
            new Thread(new TProducer(lock, list,j)).start();
        }

        Thread ft = new Thread(new FirstThread());
        Thread st = new Thread(new SecondThread());
        ft.start();
        ft.join(); //两个线程顺序执行，第一个执行完才执行第二个,join()有使线程排队运行的作用
        st.start();

    }

    /**
     * 模拟生产者
     */
    private static class TProducer implements Runnable {
        private Object lock;
        private int i;
        private List<String> list;

        public TProducer(Object lock,List<String> list,int i) {
            super();
            this.lock = lock;
            this.i=i;
            this.list = list;
        }

        @Override
        public void run() {
            System.out.println(this.i+" produced ...");
            //获取对象锁，添加消息，并通知
            synchronized (lock) {
                list.add(i + " message ");
                lock.notify();
            }

        }
    }

    /**
     * 模拟消费者
     */
    private static class TConsumer implements Runnable {
        private Object lock;
        private int i;
        private List<String> list;

        public TConsumer(Object lock, List<String> list, int i) {
            super();
            this.lock = lock;
            this.i = i;
            this.list = list;
        }

        @Override
        public void run() {
            try {
                //循环获取锁并进行消费消息
                while (true) {
                    synchronized (lock) {
                        //如果没有消息，则等待
                        if (list.isEmpty()) {
                            lock.wait();
                        }
                        String msg = list.remove(0);
                        System.out.println(this.i + " comsume .. " + msg);
                    }
                    Thread.sleep(1000);
                }

            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    static class FirstThread implements Runnable {
        @Override
        public void run() {

            System.out.println("****************");
            for (int i=0;i<10;i++) {
                System.out.println("first running...");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("****************");

        }
    }
    static class SecondThread implements Runnable {
        @Override
        public void run() {
            System.out.println("---------------");
            for (int i=0;i<10;i++) {
                System.out.println("second running...");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("---------------");

        }
    }
}
```

## 通过ReentrantLock的Condition模拟打印机的输入与打印

```java
/**
     * 模拟打印机服务
     */
    static class Service {
        private ReentrantLock lock = new ReentrantLock();
        private Condition condition = lock.newCondition();
        //模拟队列
        private List<String> queue = new ArrayList<>();

        /**
         * 输入处理
         * @param value
         */
        public void input(String value) {
            try {
                //先获取锁
                lock.lock();
                //判断队列不为空时等待打印完再输入
                while (!queue.isEmpty()) {
                    condition.await();

                }
                queue.add(value);
                System.out.println("input " + value);
                System.out.println("there are " + lock.getWaitQueueLength(condition) + " thread wait condition ..");
                //通知唤醒打印服务
                condition.signalAll();
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.unlock();

            }
        }

        /**
         * 输出打印处理
         */
        public void print() {
            try {
                //获取锁对象
                lock.lock();
                //判断队列为空时，等待输入
                while (queue.isEmpty()) {
                    condition.await();
                }
                String value = queue.remove(0);
                System.out.println("print " + value);
                System.out.println("there are " + lock.getWaitQueueLength(condition) + " thread wait condition ..");
                //打印完，通知唤醒输入线程输入
                condition.signalAll();
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }
    }

    /**
     * 模拟用户输入操作线程
     */
    static class InputTask implements Runnable {
        private Service service;

        public InputTask(Service service) {
            this.service = service;
        }

        @Override
        public void run() {
            for (int i = 0; i < Integer.MAX_VALUE; i++) {
                service.input("message " + new Random().nextInt());
                //模拟输入耗时
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    /**
     * 模拟打印机打印服务线程
     */
    static class PrintTask implements Runnable {
        private Service service;

        public PrintTask(Service service) {
            this.service = service;
        }

        @Override
        public void run() {
            for (int i = 0; i < Integer.MAX_VALUE; i++) {
                service.print();
            }
        }
    }

    public static void main(String[] args) {
        Service service = new Service();
        new Thread(new InputTask(service)).start();
        new Thread(new PrintTask(service)).start();

        for (int i=0;i<10;i++) {
            new Thread(new InputTask(service)).start();
            new Thread(new PrintTask(service)).start();
        }
    }
```

## 分别通过synchronized和ReentrantLock及Thread和Runable等方式实现多线程并发读写一个List

```java
public static void main(String[] args) {
       
        ReentrantLock lock = new ReentrantLock();
        List<Integer> list = new ArrayList<>();
        for (int i = 0; i < 500; i++) {
            list.add(i);
        }
        for (int i = 0; i < 5; i++) {
            //new ConsumeThread(list).start();
            new Thread(new ConsumeThread2(list,lock)).start();
        }
    }

    /**
     * 继承thread的方式
     */
    static class ConsumeThread extends Thread {
        private List<Integer> list;

        public ConsumeThread(List<Integer> list) {
            this.list = list;
        }

        @Override
        public void run() {
            //循环处理数据
            while (true) {
                //先锁定list进行处理
                synchronized (list) {
                    if (!list.isEmpty()) {
                        int val = list.remove(0);
                        System.out.println("thread " + Thread.currentThread().getId() + " take number " + val);
                    } else {
                        break;
                    }
                }
                //模拟处理耗时
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        }



    }

    /**
     * 通过实现runnable接口的方式
     */
    static class ConsumeThread2 implements Runnable {
        private List<Integer> linkedList;
        private ReentrantLock lock;

        ConsumeThread2(List linkedList, ReentrantLock lock) {
            this.linkedList = linkedList;
            this.lock = lock;
        }
        @Override
        public void run() {
            //循环处理list
            while (true) {
                //先获取锁，读写list数据
                try {
                    lock.lock();
                    if (!linkedList.isEmpty()) {
                        int val = linkedList.remove(0);
                        System.out.println("thread " + Thread.currentThread().getId() + " take number " + val);
                    } else {
                        break;
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }finally {
                    //最后释放锁
                    lock.unlock();
                }
                //模仿处理耗时
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

            }
        }
    }

```
