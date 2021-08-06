# 一、创建线程

## 1.实现多线程的两种方法

### （1）实现Runnable接口

```java
/**
 * 使用Runnable方式实现线程
 */
public class RunnableStyle implements Runnable{
    public static void main(String[] args) {
        Thread thread=new Thread(new RunnableStyle());
        thread.start();
    }

    @Override
    public void run() {
        System.out.println("Runnable");
    }
}

Runnable
```

### （2）继承Thread类

```java
/**
 * 用Thread方式实现线程
 */
public class ThreadStyle extends Thread{

    public static void main(String[] args) {
        new ThreadStyle().start();
    }

    @Override
    public void run()
    {
        System.out.println("Thread");
    }

}

Thread
```

### （3）两种方法对比

- 实现Runnable接口的方法更好
- 本质对比
  - Runnable：最终调用target.run()
  - Thread：重写run()

​		run()方法源码如下：

```java
private Runnable target;

@Override
public void run() {
    if (target != null) {
        target.run();
    }
}
```

##  2.同时使用两种方法

```java
/**
 * 同时使用Runnable与Thread
 */
public class BothRunnableThread {
    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("Runnable");
            }
        }){
            @Override
            public void run(){
                System.out.println("Thread");
            }
        }.start();
    }
}

Thread
```

​		准确地讲，创建线程只有一种方式那就是构造Thread类，而实现线程的执行单元有两种方式：

1. 实现Runnable接口的run方法，并把Runnable实例传给Thread类
2. 重写Thread的run方法（继承Thread类）

## 3.典型错误观点

### （1）线程池

```java
/**
 * 线程池创建线程
 * 源码中线程池也是使用new Thread创建线程
 */
public class ThreadPool5 {
    public static void main(String[] args) {
        ExecutorService executorService= Executors.newCachedThreadPool();
        for (int i = 0; i < 1000; i++) {
            executorService.submit(new Task(){});
        }
        //可理解为瞬间提交完1000个任务，1000个任务共同sleep500ms，而不是500*1000ms
    }
}
class Task implements Runnable{

    @Override
    public void run() {
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName());
    }
}
```

### （2）定时器

```java
/**
 * 定时器创建线程
 */
public class TimerTaskDemo {
    public static void main(String[] args) {
        Timer timer=new Timer();
        timer.scheduleAtFixedRate(new TimerTask() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName());
            }
        },1000,1000);
    }
}
```

### （3）匿名类

```java
/**
 * 匿名类创建线程
 */
public class AnonymousInnerClassDemo {
    public static void main(String[] args) {
        new Thread(){
            @Override
            public void run(){
                System.out.println(Thread.currentThread().getName());
            }
        }.start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName());
            }
        }).start();
    }
}
```

### （4）Lambda表达式

```java
/**
 * Lambda表达式创建线程
 */
public class LambdaDemo {
    public static void main(String[] args) {
        new Thread(()-> System.out.println(Thread.currentThread().getName())).start();
    }
}
```

# 二、启动线程

## 1.run和start方法对比

```java
/**
 * 对比start和run两种启动线程的方式
 */
public class StartAndRunMethod {
    public static void main(String[] args) {
        Runnable runnable = ()->{
            System.out.println(Thread.currentThread().getName());
        };
        runnable.run();

        new Thread(runnable).start();
    }
}

main
Thread-0
```

​		由此可见我们应该用start方法，因为run方法是由主线程调用的。

## 2.start()方法原理解读

```java
/**
 * 演示不能两次调用start方法，否则会报错
 */
public class CantStartTwice {
    public static void main(String[] args) {
        Thread thread = new Thread();
        thread.start();
        thread.start();
    }
}
```

​		start()方法源码如下：

```java
    public synchronized void start() {
        /**
         * A zero status value corresponds to state "NEW".
         */
        if (threadStatus != 0)
            throw new IllegalThreadStateException();
        group.add(this);

        boolean started = false;
        try {
            start0();
            started = true;
        } finally {
            try {
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
            }
        }
    }
    private native void start0();
```

## 2.run()方法原理解读

```java
private Runnable target;

@Override
public void run() {
    if (target != null) {
        target.run();
    }
}
```

​		可以看到run()方法就是普通的方法，所以线程调用run()方法和主线程调用普通方法没有区别。

# 三、中断线程

​		原理：使用interrupt来**通知**，而不是强制中断

## 1.停止线程的最佳实践

### （1）普通情况

```java
/**
 * run方法内没有sleep或wait方法时停止线程
 */
public class RightWayStopThreadWithoutSleep implements Runnable{
    @Override
    public void run(){
        int num=0;
        while(!Thread.currentThread().isInterrupted() && num<=Integer.MAX_VALUE/2)
        {
            if(num%10000==0)
                System.out.println(num+"是10000的倍数");
            num++;
        }
        System.out.println("任务结束");
    }

    public static void main(String[] args) throws InterruptedException {
        Thread thread=new Thread(new RightWayStopThreadWithoutSleep());
        thread.start();
        Thread.sleep(1000);
        thread.interrupt();
    }
}

...
488820000是10000的倍数
488830000是10000的倍数
任务结束
```

### （2）遇到阻塞

#### ①迭代完后阻塞阻塞

```java
/**
 * 带有sleep的中断线程的写法
 */
public class RightWayStopThreadWithSleep {
    public static void main(String[] args) throws InterruptedException {
        Runnable runnable = ()->{
            int num=0;
            try {
                while(num<=10000000 && !Thread.currentThread().isInterrupted()){
                    if(num%100==0){
                        System.out.println(num+"是100的倍数");
                    }
                    num++;
                }
                Thread.sleep(1000);
            }catch (InterruptedException e){
                System.out.println("线程被中断");
                e.printStackTrace();
            }
        };
        Thread thread = new Thread(runnable);
        thread.start();
        Thread.sleep(100);
        thread.interrupt();
    }
}
```

#### ②每次迭代后都阻塞

```java
/**
 * 在执行过程中，每次循环都会调用sleep或wait等方法，那么不需要每次迭代都检查是否已中断
 */
public class RightWayStopThreadWithSleepEveryLoop {
    public static void main(String[] args) throws InterruptedException {
        Runnable runnable = ()->{
            int num=0;
            try {
                while(num<=10000){
                    if(num%100==0){
                        System.out.println(num+"是100的倍数");
                    }
                    num++;
                    Thread.sleep(10);
                }
            }catch (InterruptedException e){
                System.out.println("线程被中断");
                e.printStackTrace();
            }
        };
        Thread thread = new Thread(runnable);
        thread.start();
        Thread.sleep(5000);
        thread.interrupt();
    }
}
```

#### ③错误示范（while里面放try/catch）

```java
/**
 * 若while里面放try/catch，会导致中断失效
 */
public class CantInterrupt {
    public static void main(String[] args) throws InterruptedException {
        Runnable runnable = ()->{
            int num=0;
            while(num<=10000 && !Thread.currentThread().isInterrupted()){
                //加上isInterrupted判断也无济于事，因为如果在sleep过程中被中断，sleep会清除interrupted标志位
                if(num%100==0){
                    System.out.println(num+"是100的倍数");
                }
                num++;
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };

        Thread thread = new Thread(runnable);
        thread.start();
        Thread.sleep(500);
        thread.interrupt();
    }
}
```

### （3）两种最佳实践

#### ①优先抛出

```java
/**
 * 最佳实践：catch了InterruptedException之后的优先选择：在方法签名中抛出异常,那么在run()中就会强制try/catch
 */
public class RightWayStopThreadInProd implements Runnable{
    @Override
    public void run() {//run()方法是顶层函数，必须对异常进行处理，无法继续通过throws递交给上层
        while(true&&!Thread.currentThread().isInterrupted()){
            System.out.println("go");
            try {
                throwInMethod();
            } catch (InterruptedException e) {
                //保存日志
                System.out.println("保存日志");
                e.printStackTrace();
                break;
            }
        }
    }
    /*private void throwInMethod(){
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();//这里没有将错误信息递交给上层的run()方法，而是直接以糟糕的方式处理掉
        }
    }*/
    private void throwInMethod() throws InterruptedException {
        Thread.sleep(2000);
    }

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new RightWayStopThreadInProd());
        thread.start();
        Thread.sleep(1000);
        thread.interrupt();
    }
}
```

​		这里要注意，run()方法是最后实现的逻辑代码，不能够抛出checked Exception。

```java
/**
 * run()无法抛出checked Exception，只能用try/catch
 */
public class RunThrowException {
    public void aVoid() throws Exception{
        throw new Exception();
    }

    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run(){//正确的异常处理
                try {
                    throw new Exception();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            /*public void run() throws Exception{//不被允许
                throw new Exception();
            }*/
        });
    }
}
```

#### ②恢复中断

```java
/**
 * 最佳实践2：在catch子语句中调用Thread.currentThread().interrupt()来恢复设置的中断状态，
 * 以便在后续的执行中，依然能够检查到刚才发生了中断
 * 回到刚才的RightWayStopThreadInProd补上中断，让它跳出
 */
public class RightWayStopThreadInProd2 implements Runnable{
    @Override
    public void run() {//run()方法是顶层函数，必须对异常进行处理，无法继续通过throws递交给上层
        while(true){
            if(Thread.currentThread().isInterrupted()){
                System.out.println("Interrupted,程序运行结束");
                break;
            }
            System.out.println("go");
            reInterrupt();
        }
    }
    private void reInterrupt(){
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();//底层代码若想处理异常，则必须继续抛出该异常
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new RightWayStopThreadInProd2());
        thread.start();
        Thread.sleep(1000);
        thread.interrupt();
    }
}

go
java.lang.InterruptedException: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at threadcoreknowledge.stopthread.RightWayStopThreadInProd2.reInterrupt(RightWayStopThreadInProd2.java:22)
	at threadcoreknowledge.stopthread.RightWayStopThreadInProd2.run(RightWayStopThreadInProd2.java:17)
	at java.lang.Thread.run(Thread.java:748)
Interrupted,程序运行结束
```

## 2.错误的停止线程方法

### （1）stop()、suspend()和resume()

```java
/**
 * 错误的停止方法：用stop()停止线程，会导致线程运行一半时突然停止，
 * 没办法完成一个基本单位的操作（一个连队），会造成脏数据（有的连队多/少领取装备）
 */
public class StopThread implements Runnable{
    @Override
    public void run() {
        //模拟指挥军队：一共5个连队，每个连队10人，以连队为单位,
        //发放武器弹药，叫到号的士兵前去领取
        for (int i = 0; i < 5; i++) {
            System.out.println("连队"+(i+1)+"开始领取武器");
            for (int j = 0; j < 10; j++) {
                System.out.println(j);
                try {
                    Thread.sleep(50);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("连队"+(i+1)+"已经领取完毕");
        }
    }

    public static void main(String[] args) {
        Thread thread = new Thread(new StopThread());
        thread.start();
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        thread.stop();
        //stop()本质上是不安全的，停止线程会导致它解锁已锁定的所有监听器Monitor
    }
}
```

### （2）volatile关键字

#### ①看似可行

```java
/**
 * 演示用volatile的局限：part1 看似可行
 */
public class WrongWayVolatile implements Runnable{
    private volatile boolean canceled = false;

    @Override
    public void run() {
        int num=0;
        try {
            while(num<=10000 && !canceled){
                if (num%100==0){
                    System.out.println(num+"是100的倍数");
                }
                num++;
                Thread.sleep(1);
            }
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        WrongWayVolatile r=new WrongWayVolatile();
        Thread thread = new Thread(r);
        thread.start();
        Thread.sleep(5000);
        r.canceled=true;
    }
}
```

#### ②阻塞时的局限性

```java
/**
 * 演示用volatile的局限：part2
 * 陷入阻塞时，volatile是无法停止线程的
 * 此例中，生产者的生产速度很快，消费者消费速度很慢
 * 所以阻塞队列满了以后，生产者会阻塞，等待消费者进一步消费
 */
public class WrongWayVolatileCantStop{
    public static void main(String[] args) throws InterruptedException {
        ArrayBlockingQueue storage = new ArrayBlockingQueue(10);

        Producer producer = new Producer(storage);
        Thread producerThread = new Thread(producer);
        producerThread.start();
        Thread.sleep(1000);

        Consumer consumer=new Consumer(storage);
        while(consumer.needMoreNums()){
            System.out.println(consumer.storage.take()+"被消费掉");
            Thread.sleep(100);
        }
        System.out.println("消费者不需要更多数据");

        //一旦消费者不需要更多数据，我们应该让生产者也停下来，以免不必要的浪费
        producer.canceled=true;
        System.out.println(producer.canceled);
    }
}

class Producer implements Runnable{
    public volatile boolean canceled=false;
    BlockingQueue storage;

    public Producer(BlockingQueue storage){
        this.storage=storage;
    }

    @Override
    public void run() {
        int num=0;
        try {
            while(num<=10000 && !canceled){
                if (num%100==0){
                    storage.put(num);
                    System.out.println(num+"是100的倍数，已被放入仓库中");
                }
                num++;
                //Thread.sleep(1);
            }
        }catch (InterruptedException e){
            e.printStackTrace();
        }finally {
            System.out.println("生产者结束运行");
        }
    }
}

class Consumer{
    BlockingQueue storage;

    public Consumer(BlockingQueue storage){
        this.storage=storage;
    }

    public boolean needMoreNums(){
        if(Math.random()>0.95){
            return false;
        }
        return true;
    }
}
```

#### ③用中断解决无限等待问题（②中的Thread.sleep(1)也可解决）

```java
/**
 * 用中断来修复刚才的无尽等待问题
 */
public class WrongWayVolatileFixed {
    public static void main(String[] args) throws InterruptedException {
        WrongWayVolatileFixed body=new WrongWayVolatileFixed();

        ArrayBlockingQueue storage = new ArrayBlockingQueue(10);

        Producer producer = body.new Producer(storage);
        Thread producerThread = new Thread(producer);
        producerThread.start();
        Thread.sleep(1000);

        Consumer consumer=body.new Consumer(storage);
        while(consumer.needMoreNums()){
            System.out.println(consumer.storage.take()+"被消费掉");
            Thread.sleep(100);
        }
        System.out.println("消费者不需要更多数据");
        producerThread.interrupt();

    }

    class Producer implements Runnable{
        BlockingQueue storage;

        public Producer(BlockingQueue storage){
            this.storage=storage;
        }

        @Override
        public void run() {
            int num=0;
            try {
                while(num<=10000 && !Thread.currentThread().isInterrupted()){
                    if (num%100==0){
                        storage.put(num);
                        System.out.println(num+"是100的倍数，已被放入仓库中");
                    }
                    num++;
                }
            }catch (InterruptedException e){
                e.printStackTrace();
            }finally {
                System.out.println("生产者结束运行");
            }
        }
    }

    class Consumer{
        BlockingQueue storage;

        public Consumer(BlockingQueue storage){
            this.storage=storage;
        }

        public boolean needMoreNums(){
            if(Math.random()>0.95){
                return false;
            }
            return true;
        }
    }
}
```

## 3.停止线程相关重要函数解析

### （1）static boolean interrupted()

​		源码如下：

```java
public static boolean interrupted() {
    return currentThread().isInterrupted(true);//true代表的是ClearInterrupted，即是否清除标志位
}
```

### （2）boolean isInterrupted()

```java
public boolean isInterrupted() {
    return isInterrupted(false);
}
```

### （3）实例分析

```java
/**
 * 注意Thread.interrupted()方法的目标对象是“当前线程”，而不管本方法来自于哪个对象
 */
public class RightWayInterrupted {
    public static void main(String[] args) throws InterruptedException {

        Thread threadOne = new Thread(new Runnable() {
            @Override
            public void run() {
                for (; ; ) {
                }
            }
        });

        // 启动线程
        threadOne.start();
        //设置中断标志
        threadOne.interrupt();
        
        //获取中断标志
        System.out.println("isInterrupted: " + threadOne.isInterrupted());
        //获取中断标志并重置
        System.out.println("isInterrupted: " + threadOne.interrupted());
        //获取中断标志并重置
        System.out.println("isInterrupted: " + Thread.interrupted());
        //获取中断标志
        System.out.println("isInterrupted: " + threadOne.isInterrupted());
        threadOne.join();
        System.out.println("Main thread is over.");
    }
}

isInterrupted: true
isInterrupted: false
isInterrupted: false
isInterrupted: true
```

## 4.如何处理不可中断的阻塞？（例如抢锁时ReentrantLock.lock()或者Socket I/O时无法响应中断,那应该怎么让该线程停止呢? ）

​		如果线程阻塞是由于调用了 wait()，sleep()或join()方法，你可以中断线程，通过抛出InterruptedException异常来唤醒该线程。
​		但是对于不能响应InterruptedException的阻塞，并没有一个通用的解决方案。
​		但是我们可以利用特定的其它的可以响应中断的方法，比如ReentrantLock.lockInterruptibly()、比如关闭套接字使线程立即返回等方法来达到目的。
答案有很多种，因为有很多原因会造成线程阻塞，所以针对不同情况，唤起的方法也不同。

# 四、线程状态

## 1.六种状态概览

![image-20210702194241901](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210702194241901.png)

## 2.New，Runnable，Terminated

```java
/**
 * 展示线程的New，Runnable，Terminated状态。即使是正在运行，也是Runnable状态，而非Running
 */
public class NewRunnableTerminated implements Runnable{
    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            System.out.println(i);
        }
    }

    public static void main(String[] args) {
        Thread thread = new Thread(new NewRunnableTerminated());

        //打印出NEW
        System.out.println(thread.getState());

        thread.start();
        //打印出Runnable
        System.out.println(thread.getState());
        try {
            Thread.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //打印出Runnable状态，即使是正在运行，也是Runnable而非Running
        System.out.println(thread.getState());

        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        //打印出Terminated
        System.out.println(thread.getState());
    }
}
```

## 3.Blocked，Waiting，TimedWaiting

```java
/**
 * 展示Blocked，Waiting，TimedWaiting
 */
public class BlockedWaitingTimedWaiting implements Runnable{

    public static void main(String[] args) throws InterruptedException {
        BlockedWaitingTimedWaiting runnable = new BlockedWaitingTimedWaiting();
        Thread thread1 = new Thread(runnable);
        Thread thread2 = new Thread(runnable);
        thread1.start();
        thread2.start();
        //防止程序运行过快，若不等待，线程的执行会快于I/O
        Thread.sleep(50);

        //打印出Timed_Waiting状态，因为正在执行Thread.sleep(1000);
        System.out.println(thread1.getState());
        //打印出Blocked状态，因为thread2想拿到syn()的锁却拿不到
        System.out.println(thread2.getState());

        try {
            Thread.sleep(1300);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        //打印出Waiting状态，因为已经执行wait方法
        System.out.println(thread1.getState());
    }

    @Override
    public void run() {
        syn();
    }

    private synchronized void syn(){
        try {
            Thread.sleep(1000);
            wait();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

TIMED_WAITING
BLOCKED
WAITING
```

## 4.转化的特殊情况

1. **如果发生异常，可以直接跳到终止TERMNATED状态**，不必再遵循路径，比如可以从WAITING直接到TERMINATED。
2. 从Object.wait()刚被唤醒时，**通常不能立刻抢到monitor锁，那就会从WAITING先进入BLOCKED状态**，抢到锁后再转换到RUNNABLE状态。

# 五、Thread与Object线程方法

## 1.wait、notify与notifyAll

### （1）展示wait与notify的作用

```java
/**
 * 展示wait和notify的基本用法
 * 1.研究代码的执行顺序
 * 2.证明wait是释放锁的
 */
public class Wait {
    public static Object object = new Object();

    static class Thread1 extends Thread{

        @Override
        public void run(){
            synchronized (object){
                System.out.println("线程"+Thread.currentThread().getName()+ "开始执行");
                try {
                    //当前调用object的线程会进入等待
                    //1.等待期间若遇到中断，会抛出异常
                    //2.wait会释放当前锁
                    System.out.println("线程"+Thread.currentThread().getName()+"释放了锁");
                    object.wait();
                    System.out.println("线程"+Thread.currentThread().getName()+"获取到了锁");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    static class Thread2 extends Thread{
        @Override
        public void run(){
            synchronized (object){
                object.notify();
                System.out.println("线程"+Thread.currentThread().getName()+"调用了notify");
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread1 thread1=new Thread1();
        Thread2 thread2=new Thread2();
        thread1.start();
        Thread.sleep(200);
        thread2.start();
    }
}

线程Thread-0开始执行
线程Thread-0释放了锁
线程Thread-1调用了notify
线程Thread-0获取到了锁
```

### （2）notify与notifyAll的区别

```java
/**
 * 3个线程，线程1和线程2被阻塞，线程3唤醒它们。notify,notifyAll。
 * 以及start先执行不代表线程先启动。
 */
public class WaitNotifyAll implements Runnable{

    private static final Object resourseA=new Object();

    @Override
    public void run() {
        synchronized (resourseA){
            System.out.println(Thread.currentThread().getName()+" got resourseA lock.");
            try {
                System.out.println(Thread.currentThread().getName()+" waits to start.");
                resourseA.wait();
                System.out.println(Thread.currentThread().getName()+" is waiting to end.");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Runnable r=new WaitNotifyAll();
        Thread thread0=new Thread(r);
        Thread thread1=new Thread(r);
        Thread thread2=new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (resourseA){
                    resourseA.notifyAll();
                    //resourseA.notify();
                    System.out.println(Thread.currentThread().getName()+" notified.");
                }
            }
        });
        thread0.start();
        thread1.start();
        //Thread.sleep(200);
        thread2.start();
    }
}

执行结果请自己执行，执行顺序不同，得到的结果也不同
```

### （3）wait只释放当前monitor

```java
/**
 * 证明wait只释放当前的自己的锁
 */
public class WaitNotifyReleaseOwnMonitor_AllWait {
    private static volatile Object resourceA = new Object();
    private static volatile Object resourceB = new Object();
    public static void main(String[] args) {
        Thread threadA = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (resourceA){
                    System.out.println("ThreadA got resourceA lock.");
                    synchronized (resourceB){
                        System.out.println("ThreadA got resourceB lock.");
                        try {
                            System.out.println("ThreadA releases resourceA lock.");
                            resourceA.wait();

                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
        });
        Thread threadB = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (resourceA){
                    System.out.println("ThreadB got resourceA lock.");
                    System.out.println("ThreadB tries to resourceB lock.");
                    synchronized (resourceB){
                        System.out.println("ThreadB got resourceB lock.");
                    }
                }
            }
        });
        threadA.start();
        threadB.start();
    }
}

ThreadA got resourceA lock.
ThreadA got resourceB lock.
ThreadA releases resourceA lock.
ThreadB got resourceA lock.
ThreadB tries to resourceB lock.
```

### （4）特点与性质

- 用必须先拥有monitor
- 只能唤醒其中一个
- 属于Object类
- 类似功能的Condition

## 2.生产者消费者模式

```java
/**
 * 用wait/notify来实现
 */
public class ProducerConsumerModel {
    public static void main(String[] args) {
        EventStorage eventStorage=new EventStorage();
        Producer producer=new Producer(eventStorage);
        Consumer consumer=new Consumer(eventStorage);
        new Thread(producer).start();
        new Thread(consumer).start();
    }
}
class Producer implements Runnable{

    private EventStorage storage;

    public Producer(EventStorage storage){
        this.storage=storage;
    }

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            storage.put();
        }
    }
}

class Consumer implements Runnable{

    private EventStorage storage;

    public Consumer(EventStorage storage){
        this.storage=storage;
    }

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            storage.take();
        }
    }
}

class EventStorage{
    private int maxSize;
    private List<Date> storage;

    public EventStorage(){
        maxSize=10;
        storage = new ArrayList<>();
    }

    public synchronized void put(){
        while(storage.size()==maxSize){
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        storage.add(new Date());
        System.out.println("仓库里有"+storage.size()+"个产品。");
        notify();//虽然唤醒了另一个线程，但需要等待当前线程执行完毕
    }

    public synchronized void take(){
        while (storage.size()==0){
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println(storage.get(0)+"拿到,现在仓库还剩下"+(storage.size()-1)+"个商品。");
        storage.remove(0);
        notify();//虽然唤醒了另一个线程，但需要等待当前线程执行完毕
    }
}
```

## 3.两个线程交替打印0-100的奇偶数

### （1）synchronized关键字

```java
/**
 * 两个线程交替打印0~100的奇偶数，用synchronized关键字实现
 */
public class WaitNotifyPrintOddEvenSyn {

    private static int count;
    private static int sum;
    private static final Object lock = new Object();

    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                while(count<100){
                    synchronized (lock){
                        if((count&1)==0){
                            System.out.println(Thread.currentThread().getName()+":"+count);
                            count++;
                        }
                    }
                }
            }
        },"偶数").start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                while(count<100){
                    synchronized (lock){
                        if((count&1)==1){
                            System.out.println(Thread.currentThread().getName()+":"+count);
                            count++;
                        }
                    }
                }
            }
        },"奇数").start();
    }
}
```

### （2）wait和notify

```java
/**
 * 两个线程交替打印0~100的奇偶数，用wait和notify
 */
public class WaitNotifyPrintOddEvenWait {

    private static int count=0;
    private static final Object lock = new Object();


    static class TurningRunner implements Runnable{

        @Override
        public void run() {
            while(count<=100){
                //拿到锁之后，立刻打印
                synchronized (lock){
                    System.out.println(Thread.currentThread().getName()+":"+count++);
                    lock.notify();
                    if(count<=100){
                        try {
                            //若任务还未结束，则让出当前锁，并且自己休眠
                            lock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new Thread(new TurningRunner(),"偶数").start();
        Thread.sleep(100);
        new Thread(new TurningRunner(),"奇数").start();
    }
}
```

## 4.sleep方法

### （1）不释放锁

​		作用：只让线程在预期的时间执行，其他时候不要占用CPU资源。

​		sleep方法并不释放锁（包括synchronized和lock），与wait不同。

#### ①synchronized

```java
/**
 * 展示线程sleep时不释放synchronized的monitor，等sleep时间到了以后，正常结束后才释放锁
 */
public class SleepDontReleaseMonitor implements Runnable{

    public static void main(String[] args) {
        SleepDontReleaseMonitor sleepDontReleaseMonitor = new SleepDontReleaseMonitor();
        new Thread(sleepDontReleaseMonitor).start();
        new Thread(sleepDontReleaseMonitor).start();
    }

    @Override
    public void run() {
        syn();
    }

    private synchronized void syn(){
        System.out.println("线程"+Thread.currentThread().getName()+"获取到了monitor.");
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("线程"+Thread.currentThread().getName()+"退出了同步代码块.");
    }
}

线程Thread-0获取到了monitor.
线程Thread-0退出了同步代码块.
线程Thread-1获取到了monitor.
线程Thread-1退出了同步代码块.
```

#### ②lock

```java
/**
 * 演示sleep不释放lock(lock需要手动释放)
 */
public class SleepDontReleaseLock implements Runnable{
    private static final Lock lock=new ReentrantLock();

    @Override
    public void run() {
        lock.lock();
        System.out.println("线程"+Thread.currentThread().getName()+"获取到了锁.");
        try {
            Thread.sleep(5000);
            //观察sleep后是否会释放锁
            System.out.println("线程"+Thread.currentThread().getName()+"被唤醒.");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            lock.unlock();
            System.out.println("线程"+Thread.currentThread().getName()+"释放了锁.");
        }
    }

    public static void main(String[] args) throws InterruptedException {
        SleepDontReleaseLock sleepDontReleaseLock=new SleepDontReleaseLock();
        new Thread(sleepDontReleaseLock).start();
        Thread.sleep(1);
        new Thread(sleepDontReleaseLock).start();
    }
}

线程Thread-0获取到了锁.
//after 5s
线程Thread-0被唤醒.
线程Thread-0释放了锁.
线程Thread-1获取到了锁.
//after 5s
线程Thread-1被唤醒.
线程Thread-1释放了锁.
```

### （2）响应中断

- 抛出InterruptedException
- **清除中断状态**

```java
/**
 * 每隔1s输出当前时间，一段时间后被中断
 * Thread.sleep()
 * TimeUnit.SECONDS.sleep()
 */
public class SleepInterrupted implements Runnable{

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new SleepInterrupted());
        thread.start();
        Thread.sleep(6300);
        thread.interrupt();
    }

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println(new Date());
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                System.out.println("该线程被中断。");
                e.printStackTrace();
            }
        }
    }
}
```

## 5.join方法

​		作用：由于新线程的加入而等待新线程执行完再出发。比如main等待thread1执行完毕。

### （1）普通用法

```java
/**
 * 演示join，注意语句输出顺序，会变化
 */
public class Join {
    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName()+"执行完毕");
            }
        });
        Thread thread2 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName()+"执行完毕");
            }
        });
        thread1.start();
        thread2.start();
        System.out.println("开始等待子线程运行完毕");
        //若将join注释掉，则会在主线程执行完毕后执行子线程，并不会等待
        thread1.join();
        thread2.join();
        System.out.println("所有子线程执行完毕");
    }
}

开始等待子线程运行完毕
Thread-1执行完毕
Thread-0执行完毕
所有子线程执行完毕
```

### （2）join期间被中断

```java
/**
 * 演示join期间被中断的效果
 */
public class JoinInterrupt {
    public static void main(String[] args) throws InterruptedException {
        Thread mainThread = Thread.currentThread();
        Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    mainThread.interrupt();
                    Thread.sleep(5000);
                    System.out.println("Thread1 finished.");
                } catch (InterruptedException e) {
                    System.out.println("子线程中断");
                }
            }
        });
        thread1.start();
        System.out.println("等待子线程运行完毕");
        try {
            thread1.join();//主线程等待thread1这个子线程运行
        } catch (InterruptedException e) {
            System.out.println(Thread.currentThread().getName()+"主线程被中断");//子线程中执行的中断主线程的异常被捕获
            thread1.interrupt();//一旦主线程被中断，将中断传递给子线程，否则会出现主线程运行完毕，子线程仍在执行的问题
        }
        Thread.sleep(1000);
        System.out.println("子线程运行完毕");
    }
}

等待子线程运行完毕
main主线程被中断
子线程中断
子线程运行完毕
```

### （3）join原理

```java
/**
 * 通过分析join的原理，分析出join的代替写法
 */
public class JoinPrinciple {
    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                    System.out.println("Thread1 finished.");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName()+"执行完毕");
            }
        });
        thread1.start();
        System.out.println("开始等待子线程运行完毕");
        //thread1.join();//join底层是wait
        synchronized (thread1){
            thread1.wait();
        }
        System.out.println("所有子线程执行完毕");
    }
}
```

## 6.yield方法

​		作用：释放我的CPU时间片。

​		但是JVM不保证遵守yield原则。

​		yield后线程仍为Runnable状态，而不是像sleep那样陷入阻塞。

## 7.Thread.CurrentThread方法

```java
/**
 * 演示打印main,Thread-0,Thread-1
 */
public class CurrentThread implements Runnable{
    public static void main(String[] args) throws InterruptedException {
        new CurrentThread().run();
        new Thread(new CurrentThread()).start();
        new Thread(new CurrentThread()).start();
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName());
    }
}

main
Thread-0
Thread-1
```

# 六、线程属性

## 1.概览

- 编号ID
- 名称Name
- 是否是守护线程isDaemon：true为守护线程，false为用户线程
- 优先级Priority

## 2.线程ID

```java
/**
 * ID从1开始，JVM运行之后，我们自己创建的线程的ID早已不是2.
 */
public class Id {

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread();
        System.out.println("主线程的ID："+Thread.currentThread().getId());
        System.out.println("子线程的ID："+thread.getId());
    }
}

主线程的ID：1
子线程的ID：12
```

## 3.线程优先级

​		线程优先级共有10个级别（1-10），默认为5，并且子线程会继承父线程的优先级。

```java
/**
 * 演示线程优先级
 */
public class Priority {
    public static void main(String[] args) {
        Thread thread = new Thread();
        System.out.println("主线程的优先级："+Thread.currentThread().getPriority());
         System.out.println("子线程的优先级："+thread.getPriority());
    }
}

主线程的优先级：5
子线程的优先级：5
```

## 4.守护线程

​		作用：给用户线程提供服务。

​		守护线程的3个特性如下：

1. 线程类型默认继承自父线程。我们创建的线程都是用户线程，因为其父线程就是用户线程。
2. 被谁启动（一般而言守护线程都是JVM启动，JVM启动时会有一个非守护线程——main函数）
3. 不影响JVM退出。若仍有用户线程未结束运行，那么JVM不会退出；而若是只剩守护线程未结束运行，那么JVM仍会退出。

> 守护线程与普通线程整体无区别，**唯一区别**就是是否影响JVM的退出。
>
> 我们平时也**不应该**把线程设置为守护线程。

# 七、线程异常

## 1.Java异常体系图

![image-20210703225220581](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210703225220581.png)

​		异常类Exception可以分为运行时异常（RuntimeException）和非运行时异常。

​		 **Java异常又可以分为不受检查异常（Unchecked Exception）和检查异常（Checked Exception）。**

- Error：大多数由JVM生成并抛出，大多数错误与代码执行者所执行的操作无关，属于程序无法处 理的错误，无法通过catch进行捕获。
- Exception：程序本身可以处理的异常。 
- 检查异常：在正确的程序运行过程中，很容易出现的、情理可容的异常状况，在一定程度上这种异 常的发生是可以预测的，并且一旦发生该种异常，就必须采取某种方式进行处理。 
- 不受检查异常：包括 RuntimeException 及其子类和 Error 。

## 2.需要UncaughtException的理由

### （1）主线程可以轻松发现异常，子线程不行

```java
/**
 * 单线程：抛出，处理，有异常堆栈
 * 多线程：子线程发生异常，不会影响程序的正常运行
 */
public class ExceptionInChildThread implements Runnable{

    public static void main(String[] args) {
        new Thread(new ExceptionInChildThread()).start();
        for (int i = 0; i < 2000; i++) {
            System.out.println(i);
        }
    }

    @Override
    public void run() {
        throw new RuntimeException();
    }
}
```

### （2）子线程异常无法用传统方法捕获

```java
/**
 * 1.不加try/catch 时抛出4个异常，都带有线程名字
 * 2.加了try/catch，期望捕获到第一个线程的异常，线程234不应该运行，希望看到打印出Caught Exception
 * 3.执行时发现，根本没有Caught Exception，线程234仍然运行且抛出异常
 * 说明线程的异常不能用传统方法捕获
 */
public class CantCatchDirectly implements Runnable{

    public static void main(String[] args) throws InterruptedException {
        //trycatch捕获主线程的异常，而此时异常是在子线程中抛出，所以无法直接捕获
        try{
            new Thread(new CantCatchDirectly(),"MyThread-1").start();
            Thread.sleep(100);
            new Thread(new CantCatchDirectly(),"MyThread-2").start();
            Thread.sleep(100);
            new Thread(new CantCatchDirectly(),"MyThread-3").start();
            Thread.sleep(100);
            new Thread(new CantCatchDirectly(),"MyThread-4").start();
        }catch (RuntimeException e){
            System.out.println("Caught Exception by "+Thread.currentThread().getName());
        }
        /*new Thread(new CantCatchDirectly(),"MyThread-1").start();
        Thread.sleep(100);
        new Thread(new CantCatchDirectly(),"MyThread-2").start();
        Thread.sleep(100);
        new Thread(new CantCatchDirectly(),"MyThread-3").start();
        Thread.sleep(100);
        new Thread(new CantCatchDirectly(),"MyThread-4").start();*/

    }

    @Override
    public void run() {
        throw new RuntimeException();
    }
}
```

### （3）不直接捕获则无人处理业务逻辑，合理捕获可以提高健壮性

## 3.两种解决方法

### （1）手动在每个run方法中进行try catch（不推荐）

```java
	@Override
    public void run() {
        try{
        	throw new RuntimeException();
        }catch (RuntimeException e){
            System.out.println("Caught Exception by "+Thread.currentThread().getName());
        }
    }
```

### （2）利用UncaughtExceptionHandler（推荐）

​		源码如下：

```java
public interface UncaughtExceptionHandler {
    void uncaughtException(Thread t, Throwable e);
}
```

## 4.自己实现

​		共有三种方法：

### （1）给程序统一设置

```java
/**
 * 自己的MyUncaughtExceptionHandler
 */
public class MyUncaughtExceptionHandler implements Thread.UncaughtExceptionHandler {
    private String name;
    public MyUncaughtExceptionHandler(String name){
        this.name=name;
    }

    @Override
    public void uncaughtException(Thread t, Throwable e) {
        Logger logger = Logger.getAnonymousLogger();
        logger.log(Level.WARNING,"线程异常，已终止"+t.getName(),e);
        System.out.println(name+"捕获了异常"+t.getName()+"异常"+e);
    }
}

/**
 * 使用刚才自己写的UncaughtExceptionHandler
 */
public class UseOwnUncaughtExceptionHandler implements Runnable{
    public static void main(String[] args) throws InterruptedException {
        Thread.setDefaultUncaughtExceptionHandler(new MyUncaughtExceptionHandler("捕获器-1"));

        new Thread(new UseOwnUncaughtExceptionHandler(),"MyThread-1").start();
        Thread.sleep(100);
        new Thread(new UseOwnUncaughtExceptionHandler(),"MyThread-2").start();
        Thread.sleep(100);
        new Thread(new UseOwnUncaughtExceptionHandler(),"MyThread-3").start();
        Thread.sleep(100);
        new Thread(new UseOwnUncaughtExceptionHandler(),"MyThread-4").start();

    }

    @Override
    public void run() {
        throw new RuntimeException();
    }
}
```

### （2）给每个线程单独设置

### （3）给线程池设置

# 八、线程安全

## 1.运行结果错误：a++多线程下出现消失的请求现象

### （1）运行结果出错

```java
/**
 * 线程安全问题第一种：运行结果出错
 * 演示计数不准确（减少），并找出具体出错的位置。
 */
public class MultiThreadsError implements Runnable{
    static MultiThreadsError instance = new MultiThreadsError();
    public int index=0;

    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(instance);
        Thread thread2 = new Thread(instance);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println("表面上结果是："+instance.index);
    }

    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            index++;
        }
    }
}

表面上结果是：12666
```

### （2）表面结果+出错次数<真正次数

```java
public class MultiThreadsError implements Runnable{
    static MultiThreadsError instance = new MultiThreadsError();
    static AtomicInteger realIndex = new AtomicInteger();
    static AtomicInteger wrongCount = new AtomicInteger();
    public int index=0;
    final boolean[] marked = new boolean[100000];

    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(instance);
        Thread thread2 = new Thread(instance);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println("表面上结果是："+instance.index);
        System.out.println("真正运行的次数是："+realIndex.get());
        System.out.println("错误次数是："+wrongCount.get());
    }

    @Override
    public void run() {
        marked[0] = true;
        for (int i = 0; i < 10000; i++) {
            index++;
            realIndex.incrementAndGet();
            if(marked[index]){
                wrongCount.incrementAndGet();
            }
            marked[index] = true;
        }
    }
}

表面上结果是：15332
真正运行的次数是：20000
错误次数是：3154
```

​		当多个线程都运行到for语句中的`if(marked[index])`时，若第一个线程判断`marked[index]==false`后还未执行`marked[index]=true`时，第二个线程去做if判断，就会导致漏掉出错次数。

### （3）表面结果+出错次数>真正次数

```java
public class MultiThreadsError implements Runnable{
    static MultiThreadsError instance = new MultiThreadsError();
    static AtomicInteger realIndex = new AtomicInteger();
    static AtomicInteger wrongCount = new AtomicInteger();
    public int index=0;
    final boolean[] marked = new boolean[100000];

    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(instance);
        Thread thread2 = new Thread(instance);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println("表面上结果是："+instance.index);
        System.out.println("真正运行的次数是："+realIndex.get());
        System.out.println("错误次数是："+wrongCount.get());
    }

    @Override
    public void run() {
        marked[0] = true;
        for (int i = 0; i < 10000; i++) {
            index++;//这里一个线程的index++会与另一个线程的marked[index]冲突
            realIndex.incrementAndGet();
            synchronized (instance){
                if(marked[index]){
                    wrongCount.incrementAndGet();
                }
                marked[index] = true;
            }
        }
    }
}

表面上结果是：19988
真正运行的次数是：20000
错误次数是：7431
```

### （4）最终完善1：把全部业务逻辑放在synchronized里

```java
public class MultiThreadsError implements Runnable{
    static MultiThreadsError instance = new MultiThreadsError();
    static AtomicInteger realIndex = new AtomicInteger();
    static AtomicInteger wrongCount = new AtomicInteger();
    public int index=0;
    final boolean[] marked = new boolean[100000];

    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(instance);
        Thread thread2 = new Thread(instance);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println("表面上结果是："+instance.index);
        System.out.println("真正运行的次数是："+realIndex.get());
        System.out.println("错误次数是："+wrongCount.get());
    }

    @Override
    public void run() {
        marked[0] = true;
        for (int i = 0; i < 10000; i++) {
            synchronized (instance){
                index++;
                realIndex.incrementAndGet();
                if(marked[index]){
                    wrongCount.incrementAndGet();
                }
                marked[index]=true;
            }
        }
    }
}

表面上结果是：20000
真正运行的次数是：20000
错误次数是：0
```

### （5）最终完善2：使用工具类CyclicBarrier

```java
public class MultiThreadsError implements Runnable{
    static MultiThreadsError instance = new MultiThreadsError();
    static AtomicInteger realIndex = new AtomicInteger();
    static AtomicInteger wrongCount = new AtomicInteger();
    static volatile CyclicBarrier cyclicBarrier1 = new CyclicBarrier(2);
    static volatile CyclicBarrier cyclicBarrier2 = new CyclicBarrier(2);
    public int index=0;
    final boolean[] marked = new boolean[100000];

    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(instance);
        Thread thread2 = new Thread(instance);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println("表面上结果是："+instance.index);
        System.out.println("真正运行的次数是："+realIndex.get());
        System.out.println("错误次数是："+wrongCount.get());
    }

    @Override
    public void run() {
        marked[0] = true;
        for (int i = 0; i < 10000; i++) {
            try {
                cyclicBarrier2.reset();
                cyclicBarrier1.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
            index++;
            try {
                cyclicBarrier1.reset();
                cyclicBarrier2.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
            realIndex.incrementAndGet();
            synchronized (instance){
                if(marked[index] && marked[index-1]){
                    wrongCount.incrementAndGet();
                }
                marked[index] = true;
            }
        }
    }
}

表面上结果是：19998
真正运行的次数是：20000
错误次数是：2
```

​		出错处的判断逻辑一定要改成`marked[index] && marked[index-1]`，不然每个偶数位均会出错。这是因为两个线程都到`index++`后才能突破屏障继续运行，而index初值为0，每次都会运行两个线程的`index++`，第一个线程运行是`marked[index]`为false，然后置为true，第二个线程在if语句处就会判定为出错。

## 2.死锁

```java
/**
 * 第二种线程安全问题：演示死锁
 */
public class MultiThreadsError2 implements Runnable{

    int flag=1;
    static Object o1 = new Object();
    static Object o2 = new Object();

    public static void main(String[] args) throws InterruptedException {
        MultiThreadsError2 r1 = new MultiThreadsError2();
        MultiThreadsError2 r2 = new MultiThreadsError2();
        r1.flag=1;
        r2.flag=0;
        Thread thread1 = new Thread(r1);
        Thread thread2 = new Thread(r2);
        thread1.start();
        thread2.start();
    }


    @Override
    public void run() {
        System.out.println("flag = "+flag);
        if(flag==1){
            synchronized (o1){
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (o2){
                    System.out.println("1");
                }
            }
        }

        if(flag==0){
            synchronized (o2){
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (o1){
                    System.out.println("0");
                }
            }
        }
    }
}

flag = 1
flag = 0
无尽等待
```

## 3.对象发布和初始化时的安全问题

​		发布：使对象可以让超过这个类范围之内的地方使用，例如public.

​		逸出：被发布到不该发布的地方，比如：

1. 方法返回一个private对象
2. 还未完成初始化就把对象提供给外界，比如：
   - 在构造函数中未初始化完毕就this赋值
   - 隐式逸出——注册监听事件
   - 构造函数中运行线程

### （1）逃逸private

```java
/**
 * 方法返回一个private对象
 */
public class MultiThreadsError3{

    private Map<String,String> states;

    public MultiThreadsError3(){
        states=new HashMap<>();
        states.put("1","周一");
        states.put("2","周二");
        states.put("3","周三");
        states.put("4","周四");
    }

    public void output(){
        System.out.println(states.get("1"));
    }

    public Map<String, String> getStates() {
        return states;//Error！
    }

    public static void main(String[] args) {
        MultiThreadsError3 multiThreadsError3 = new MultiThreadsError3();
        Map<String,String> state = multiThreadsError3.getStates();
        System.out.println(state.get("1"));
        state.remove("1");
        System.out.println(state.get("1"));
        multiThreadsError3.output();
    }
}

周一
null
null
```

​		可以看到此时我们的private对象因为被方法return出去而产生预期外的修改。

### （2）this赋值

```java
/**
 * 未初始化就进行this赋值
 */
public class MultiThreadsError4 {
    static Point point;

    public static void main(String[] args) throws InterruptedException {
        new PointMaker().start();
        //Thread.sleep(10);//x = 2,y = 0
        Thread.sleep(105);//x = 2,y = 4
        if(point!=null){
            System.out.println(point);
        }
    }
}

class Point{
    private final int x,y;
    public Point(int x,int y) throws InterruptedException {
        this.x = x;
        MultiThreadsError4.point = this;
        Thread.sleep(100);
        this.y = y;
    }

    @Override
    public String toString(){
        return "x = " + x + ",y = " + y;
    }
}

class PointMaker extends Thread{
    @Override
    public void run(){
        try {
            new Point(2,4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

### （3）注册监听器

```java
/**
 * 观察者模式
 */
public class MultiThreadsError5 {

    int count;

    public MultiThreadsError5(MySource source){
        source.registerListener(new EventListener() {
            @Override
            public void onEvent(Event e) {
                System.out.println("\n我得到的数字是："+count);
            }
        });
        for (int i = 0; i < 130; i++) {
            System.out.print(1);
        }
        count = 100;
    }

    public static void main(String[] args) {
        MySource source = new MySource();
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                source.eventCome(new Event() {
                });
            }
        }).start();
        MultiThreadsError5 multiThreadsError5 = new MultiThreadsError5(source);
    }

    static class MySource{

        private EventListener listener;

        void registerListener(EventListener eventListener){
            this.listener = eventListener;
        }

        void eventCome(Event e){
            if(listener!=null){
                listener.onEvent(e);
            }else{
                System.out.println("未初始化完毕");
            }
        }
    }
    interface EventListener{
        void onEvent(Event e);
    }
    interface Event{

    }
}

111111111111111111111111111111111111111111
我得到的数字是：0
1111111111111111111111111111111111111111111111111111111111111111111111111111111111111111
```

### （4）构造函数中新建线程

```java
/**
 * 构造函数中新建线程
 */
public class MultiThreadsError6 {
    private Map<String,String> states;

    public MultiThreadsError6(){
        new Thread(new Runnable() {
            @Override
            public void run() {
                states=new HashMap<>();
                states.put("1","周一");
                states.put("2","周二");
                states.put("3","周三");
                states.put("4","周四");
            }
        }).start();

    }


    public Map<String, String> getStates() {
        return states;
    }

    public static void main(String[] args) throws InterruptedException {
        MultiThreadsError6 multiThreadsError6 = new MultiThreadsError6();
        //Thread.sleep(1000);//后续可用工厂模式修复
        Map<String,String> state = multiThreadsError6.getStates();
        System.out.println(state.get("1"));

    }
}

Exception in thread "main" java.lang.NullPointerException
at background.MultiThreadsError6.main(MultiThreadsError6.java:35)
```

## 4.如何解决逸出

### （1）使用副本（详见逃逸private）

```java
/**
 * 方法返回一个private对象
 */
public class MultiThreadsError3{

    private Map<String,String> states;

    public MultiThreadsError3(){
        states=new HashMap<>();
        states.put("1","周一");
        states.put("2","周二");
        states.put("3","周三");
        states.put("4","周四");
    }

    public void output(){
        System.out.println(states.get("1"));
    }

    public Map<String, String> getStates() {
        return states;
    }

    public Map<String, String> getStatesImproved() {//用副本解决
        return new HashMap<>(states);
    }

    public static void main(String[] args) {
        MultiThreadsError3 multiThreadsError3 = new MultiThreadsError3();
        Map<String,String> state = multiThreadsError3.getStatesImproved();
        System.out.println(state.get("1"));
        state.remove("1");
        System.out.println(state.get("1"));
        multiThreadsError3.output();
    }
}

周一
null
周一
```

### （2）工厂模式

```java
/**
 * 用工厂模式修复初始化问题
 */
public class MultiThreadsError7 {
    int count;
    private EventListener listener;

    private MultiThreadsError7(MySource source){
        listener = new EventListener() {
            @Override
            public void onEvent(Event e) {
                System.out.println("\n我得到的数字是："+count);
            }
        };
        for (int i = 0; i < 230; i++) {
            System.out.print(1);
        }
        count = 100;
    }
    public  static MultiThreadsError7 getInstance(MySource source){
        MultiThreadsError7 safeListener = new MultiThreadsError7(source);
        source.registerListener(safeListener.listener);
        return safeListener;
    }

    public static void main(String[] args) {
        MultiThreadsError7.MySource source = new MultiThreadsError7.MySource();
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                source.eventCome(new Event() {
                });
            }
        }).start();
        MultiThreadsError7 multiThreadsError7 = new MultiThreadsError7(source);
    }

    static class MySource{

        private EventListener listener;

        void registerListener(EventListener eventListener){
            this.listener = eventListener;
        }

        void eventCome(Event e){
            if(listener!=null){
                listener.onEvent(e);
            }else{
                System.out.println("\n未初始化完毕");
            }
        }
    }
    interface EventListener{
        void onEvent(Event e);
    }
    interface Event{

    }
}

111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111
未初始化完毕
11111111111111111111111111111111111111111111111111111111111111111111111111111
```

## 5.多线程为什么会带来性能问题

- 调度：上下文切换

  - 上下文是指**某一时间点CPU寄存器和程序计数器的内容**。

    - 寄存器是CPU内部的数量较少但是速度很快的内存（与之对应的是CPU外部相对较慢的RAM主内存）。寄存器通过对常用值（通常是运算的中间值）的快速访问来提高计算机程序运行的速度。
  - 程序计数器是一个专用的寄存器，用于表明指令序列中 CPU正在执行的位置，存的值为正在执行的指令的位置或者下一个将要被执行的指令的位置，具体依赖于特定的系统。
    
  - 上下文切换可以认为是内核在CPU上对于进程（包括线程）进行以下的活动：

    - **挂起一个进程**，将这个进程在CPU中的状态（上下文）存储于内存中的某处

    - 在内存中检索下一个进程的上下文并将其**在CPU 的寄存器中恢复**

    - 跳转到程序计数器所指向的位置（即跳转到进程被中断时的代码行），以**恢复该进程。**

      上下文就是保护现场的内容，比如线程的状态、寄存器等，都是为了把线程切换回来再次执行所必需的内容。

  - 缓存开销：缓存失效

    程序有很大概率会访问刚才访问过的数据，所以CPU为了加快执行速度，会把常用到的数据缓存到CPU内，这样以后再用到该数据时，可以很快使用。
    但是现在上下文被切换了，CPU即将执行**不同的线程的不同的代码**，那么原本缓存的内容有极大概率也没有价值了。这就需要CPU重新缓存，这导致线程在被调度运行后，一开始的启动速度有点慢。
    为此，线程调度器为了避免太频繁的上下文切换带来的开销过大，往往会给每个被调度到的线程设置一个**“最小执行时间”**，等待时间必须至少为“最小执行时间”时才能切换上下文，这样就把上下文切换的最小阈值提高，减少上下文切换的次数，从而提高性能（但是当然，也降低了响应性）。

  - 何时会导致密集的上下文切换：抢锁、IO

- 协作：内存同步

  - 为了数据的正确性，同步手段往往会使用禁止编译器优化，使CPU内的缓存失效，从而减少了原本可以进行的优化。

# 九、Java内存模型——底层原理

## 1.从Java命令到CPU指令

1. 我们编写的Java代码，是`*.java文件`
2. 编译后（javac命令），从刚才的`*.java`文件会生成一个新的Java字节码文件（`*.class`）
3. **JVM**会执行刚才生成的字节码文件（`*.class`），**并把字节码文件转化为机器指令**
4. 机器指令可以直接在CPU上执行，也就是最终的程序执行

​		JVM实现会带来不同的“翻译”，不同的CPU平台的机器指令千差万别，无法保证并发安全的效果一致。由此急需一套转化过程的规范、原则，这就是我们为什么要学习底层原理。

## 2.JVM内存结构 VS Java内存模型 VS Java对象模型

### （1）概述

- JVM内存结构：与Java虚拟机的**运行时区域**有关。
- Java内存模型：与Java的**并发编程**有关。
- Java对象模型：与Java**对象在虚拟机中的表现形式**有关。

### （2）JVM内存结构

![image-20210705115656987](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210705115656987.png)

- 堆（Heap）：通过new或其他指令所创建的实例对象。堆的优势是在运行时动态的分配，无需事先告知编译器。
- 虚拟机栈，即Java栈（Java Stack）：保存了各个基本类型，以及对于对象的引用（对象的本身保存在堆中）。在编译初期就已经确定大小，运行时大小不会改变。
- 方法区（Method Area）：主要存储的是已经加载的**static静态变量、类信息以及常量信息**，还包含**永久引用**（比如static修饰的类）。
- 本地方法栈（Native）：主要存储和本地方法相关的内容，**本地方法主要指native方法**。
- 程序计数器（PC）：主要存储当前线程所执行到的**字节码的行号数**，还包括下一条需要执行的指令、分支、循环等。

### （3）Java对象模型

![image-20210705121139669](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210705121139669.png)

- 是Java对象自身的存储模型
- JVM会给这个类创建一个instanceKlass**（不是Class，就是为了与之区分）**，保存在**方法区**，用来在JVM层表示该Java类
- 当我们在Java代码中，new一个对象时，JVM会创建一个instanceOopDesc对象，这个对象中包含**对象头**以及**实例数据**

## 3.为什么需要JMM

​		Java Memory Model，即JMM，Java内存模型，是一组**规范**，需要各个JVM的实现来遵守JMM规范，以便于开发者可以**利用这些规范，更方便地开发多线程程序。**

​		JMM是**工具类和关键字的原理**，volatile、synchronized、Lock等的原理都是JMM.

​		最重要的3点内容：**重排序、可见性、原子性**。

## 4.重排序

### （1）重排序的前3种可能性

​		演示代码如下：

```java
/**
 * 演示重排序的现象
 * “并非一定出现，所以多次运行，直到达到某个条件才停止”，测试小概率事件
 */
public class OutOfOrderExecution {
    private static int x = 0,y = 0;
    private static int a = 0,b = 0;

    public static void main(String[] args) throws InterruptedException {
        int i=0;
        for(;;){
            i++;
            x = 0;
            y = 0;
            a = 0;
            b = 0;

            CountDownLatch latch = new CountDownLatch(1);

            Thread one = new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        latch.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    a = 1;
                    x = b;
                }
            });
            Thread two = new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        latch.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    b = 1;
                    y = a;
                }
            });
            one.start();
            two.start();
            Thread.sleep(100);
            latch.countDown();
            one.join();
            two.join();

            String result = "第" + i + "次（"+ x + "," + y + ")";
            if(x==1 && y==1){
                System.out.println(result);
                break;
            }else {
                System.out.println(result);
            }
        }
    }
}
```

​		一共有3种情况：

- a=1;x=b(0);b=1;y=a(1),最终结果是x=0,y=1;
- b=1;y=a(0);a=1;x=b(1),最终结果是x=1,y=0;
- b=1;a=1;x=b(1);y=a(1),最终结果是x=1,y=1;

### （2）真正发生重排序

​		会出现x=0;y=0的结果，这就是因为线程内部发生了重排序，其中一种代码执行顺序的可能为：y=a(0);a=1;x=b(0);b=1.

​		在线程1和线程2内部的两行代码的**实际执行顺序**和代码在**Java文件中的顺序**不一致，代码指令并不是严格按照代码语句顺序执行的，他们的顺序被改变了，这就是重排序。

### （3）重排序的好处：提高处理速度

#### ①对比重排序前后的**指令优化**：

![image-20210705174508515](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210705174508515.png)

​		重排序明显提高了处理速度。

#### ②重排序的三种情况：

- **编译器优化**：包括JVM，JIT编译器等
- **CPU指令重排**：就算编译器不发生重排，CPU也可能对指令进行重排
- **内存的“重排序”**：线程A的修改在线程B中却看不到，**引出可见性问题**

## 5.可见性

### （1）演示可见性问题

```java
/**
 * 演示可见性带来的问题
 */
public class FieldVisibility {
    int a = 1;
    int b = 2;

    public static void main(String[] args) {
        while(true){
            FieldVisibility test = new FieldVisibility();

            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    test.change();
                }
            }).start();

            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    test.print();
                }
            }).start();
        }
    }

    private void change(){
        a = 3;
        b = a;
    }

    private void print(){
        System.out.println("b = " + b + ";a = " + a);
    }
}

b = 3;a = 3
b = 2;a = 1
b = 2;a = 3
b = 3;a = 1
```

​		**b = 3;a = 1的出现就是因为可见性问题。change()已经运行完毕，但print出的a值仍然是原始值1.**

​		可以分析如下代码：

```java
public class FieldVisibility {
    int x = 0;

    private void writerThread(){
        x = 1;
    }

    private void readerThread(){
    	int r2 = x;
    }
}
```

![image-20210705181846842](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210705181846842.png)

​		此时可以使用volatile关键字解决问题：

```java
volatile int a = 1;
volatile int b = 2;

运行结果最终如下：
b = 3;a = 3
b = 2;a = 1
b = 2;a = 3
```

![image-20210705181903881](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210705181903881.png)

![image-20210705181947677](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210705181947677.png)

### （2）出现可见性问题的原因

![image-20210705182644820](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210705182644820.png)

​		CPU有多级缓存，导致读的数据过期。

- 高速缓存的**容量**比主内存小，但是**速度**仅次于寄存器，所以在CPU和主内存之间就多了Cache层
- 线程间的对于共享变量的**可见性问题不是直接由多核引起的，而是由多缓存引起的。**
- 如果所有核心都**共用一个缓存**，那么**就不存在内存可见性问题。**
- 每个核心都会将自己需要的数据**读到独占缓存中**，数据修改后也是**写入到缓存中**，然后**等待刷入到主存中**。所以会导致**有些核心读取的值是一个过期的值**。

## 6.主内存和本地内存

​		Java作为高级语言，屏蔽了底层细节，用JMM定义了一套读写内存数据的**规范**，虽然我们**不再需要关心一级缓存和二级缓存的问题**，但是JMM**抽象了主内存和本地内存的概念。**

​		这里说的本地内存**并不是真的一块给每个线程分配的内存**，而是JMM的一个抽象，是对于寄存器、一级缓存、二级缓存等的抽象。

![主内存和工作内存1](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/主内存和工作内存1.png)

![主内存和工作内存2](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/主内存和工作内存2.png)

​		JMM有以下规定（本地内存即工作内存）：

- **所有的变量**都存储在**主内存**中，同时**每个线程也有自己独立的工作内存**，**工作内存中的变量内容是主内容中的拷贝**。
- 线程**不能直接读写主内存中的变量**，而是**只能操作自己工作内存中的变量，然后再同步到主内存**。
- **主内存是多个线程共享的，但线程间不共享工作内存**，如果**线程间需要通信，必须借助主内存中转来完成。**

​		总结：**所有的共享变量存在于主内存中**，每个**线程有自己的本地内存**，而且**线程读写共享数据也是通过本地内存交换**，这才导致了**可见性问题**。

## 7.Happens-Before原则

​		两种解释：

- happens-before规则是用来解决**可见性**问题的：在时间上，**动作A发生在动作B之前，B保证能看见A**，这就是happens-before。
- 两个操作可以用happens-before来确定它们的执行顺序：**如果一个操作happens-before于另一个操作，那么我们说第一个操作对于第二个操作是可见的。**

​		如果**两个操作不具备happens-before，那么JVM是可以根据需要自由排序的**，但是如果具备happens-before （比如新建线程时，例如thread.start() happens before run方法里面的语句)，那么JVM也不能改变它们之间的顺序让happens-before失效。

​		happens-before的原则如下（如果我们分别有操作x和操作 y，我们用hb(x, y)来表示 x happens-before y）：

### （1）单线程原则

![image-20210705214934023](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210705214934023.png)

​		单个线程中的每个操作都happens-before该程序顺序中稍后出现的该线程中的每个操作。

​		如果操作x和操作y是同一个线程的两个操作，并且在代码执行上x先于y出现，那么有 hb(x,y).

​		注意: x happens-before y，**并不是说x操作一定要在y操作之前被执行，而是说x的执行结果对于y 是可见的，只要满足可见性，发生了重排序也是可以的。**

### （2）※锁操作（synchronized和lock）

​		如果A是对锁的解锁unlock，而B是对同一个锁的加锁lock，那么hb(A，B)。

![image-20210705215359826](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210705215359826.png)

![image-20210705215334130](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210705215334130.png)

### （3）※volatile变量

​		假设A是对volatile变量的写操作，B是对这个变量的读操作，那么hb(A，B)。

![image-20210705215519326](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210705215519326.png)

### （4）线程启动

​		新建线程时，thread.start()一定happens-before run方法里面的语句。

![image-20210705215608888](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210705215608888.png)

### （5）线程join

​		我们知道join可以让线程之间等待，假设线程A通过调用threadA. start()生成一个新线程B，然后调用threadA.join()。线程A将在join()期间会等待，直到线程B的run方法完成。**在join方法返回后，线程A中的所有后续操作都将看到线程B的run方法中执行的所有操作**，也就是happens-before。

​		线程的终止也类似，isAlive返回false一定在线程所有代码执行完之前。

![image-20210705220113710](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210705220113710.png)

### （6）传递性

​		如果hb(A,B)而且hb(B,C)，那么可以推出hb(A,C)

### （7）中断

​		一个线程被其他线程interrupt，那么检测中断（isInterrupted）或者抛出InterruptedException时一定能看到。

### （8）构造方法

​		对象构造方法的最后一行指令happens-before于finalize()方法的第一行指令。

### （9）工具类的Happens-Before原则

1. 线程安全的容器get一定能看到在此之前的put等存入动作
2. CountDownLatch
3. Semaphore
4. Future
5. 线程池
6. CyclicBarrier

## 8.volatile关键字

### （1）概述

​		volatile是一种**同步机制**，比synchronized或者Lock相关类更**轻量**，因为使用**volatile并不会发生上下文切换等开销很大的行为**。

​		如果一个变量被修饰成volatile，那么JVM就知道了这个变量可能**会被并发修改**。

​		但是开销小，能力也小，虽说volatile是用来同步的保证线程安全的，但是volatile做不到synchronized那样的**原子保护**，volatile仅在**很有限的场景**下才能发挥作用。

### （2）不适用于a++

```java
/**
 * 不适用于volatile的场景
 */
public class NoVolatile implements Runnable{

    volatile int a;
    AtomicInteger realA = new AtomicInteger();

    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            a++;
            realA.incrementAndGet();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Runnable r = new NoVolatile();
        Thread thread1 = new Thread(r);
        Thread thread2 = new Thread(r);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println(((NoVolatile)r).a);
        System.out.println(((NoVolatile)r).realA.get());
    }
}

18102
20000
```

### （3）适用场合1：boolean flag

​		若一个共享变量自始至终**只被各个线程赋值**，而没有其他的操作，那么就可以用volatile来代替synchronized或代替原子变量，**因为赋值自身是有原子性的（a++没有），而volatile有保证了可见性，所以就足以保证线程安全。**

```java
，/**
 * volatile使用的情况1
 */
public class UseVolatile1 implements Runnable{
    volatile boolean done = false;
    AtomicInteger realA = new AtomicInteger();

    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            setDone();
            realA.incrementAndGet();
        }
    }

    private void setDone(){
        done = true;
    }

    public static void main(String[] args) throws InterruptedException {
        Runnable r = new UseVolatile1();
        Thread thread1 = new Thread(r);
        Thread thread2 = new Thread(r);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println(((UseVolatile1)r).done);
        System.out.println(((UseVolatile1)r).realA.get());
    }
}

true
20000
```

​		**关键在于：setDone并不取决于done之前的状态，而不是因为类型是boolean。**反例如下：

```java
/**
 * volatile不适用的情况2
 */
public class NoVolatile2 implements Runnable{
    volatile boolean done = false;
    AtomicInteger realA = new AtomicInteger();

    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            flipDone();
            realA.incrementAndGet();
        }
    }

    private void flipDone(){
        done = !done;
    }

    public static void main(String[] args) throws InterruptedException {
        Runnable r = new NoVolatile2();
        Thread thread1 = new Thread(r);
        Thread thread2 = new Thread(r);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println(((NoVolatile2)r).done);
        System.out.println(((NoVolatile2)r).realA.get());
    }
}

true
20000
或
false
20000
```

### （4）适用场合2：作为刷新之前变量的触发器

```java
/**
 * volatile使用的情况2
 */
public class FieldVisibility {
    int a = 1;
    int c = 1;
    int d = 1;
    volatile int b = 2;

    public static void main(String[] args) {
        while(true){
            FieldVisibility test = new FieldVisibility();

            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    test.change();
                }
            }).start();

            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    test.print();
                }
            }).start();
        }
    }

    private void change(){
        a = 7;
        c = 3;
        d = 4;
        b = 0;
    }

    private void print(){
        if(b==0){
            System.out.println("a = " + a + ";b = " + b + ";c = " + c + ";d = " + d);
        }
    }
}
a = 7;b = 0;c = 3;d = 4
```

​		**用了`volatile int b`后，可以保证读取b后，之前的所有变量均可见，**所以无需将a、b、c设置为volatile变量。

### （5）volatile的两点作用

1. **可见性**：读一个volatile变量之前，需要先使相应的**本地缓存失效**，这样就**必须到主内存读取最新值**，写一个**volatile属性会立即刷入到主内存。**
2. **禁止指令重排序优化**：解决单例双重锁乱序问题。

### （6）volatile与synchronized

​		volatile可以看做是轻量版的synchronized：**如果一个共享变量自始至终只被各个线程赋值，而没有其他的操作**，那么就**可以用volatile来代替synchronized或者代替原子变量**，因为**赋值自身是有原子性的（a++没有），而volatile又保证了可见性，所以就足以保证线程安全。**

### （7）小结

- volatile修饰符适用于以下两种场景：
  - 第一种是**某个属性被多个线程共享，其中有一个线程修改了此属性，其他线程可以立即得到修改后的值**，且对**该属性的修改不依赖该属性的状态**。比如boolean flag。
  - 第二种是作为触发器，实现**轻量级同步**。

- volatile属性的读写都是**无锁**的，不能替代synchronized，因为没有提供**原子性**和**互斥性**。因为无锁，不需要获取锁和释放锁，因此是低成本的。
- volatile**只能作用于属性**，而不能作用于方法或一部分代码块上，我们用volatile修饰属性，这样**编译器就不会对这个属性做指令重排序。**
- volatile使得**long和double的赋值是原子的。**
- volatile 提供了**可见性**，任何一个线程对其的修改将**立马对其他线程可见**。**volatile属性不会被线程缓存，始终从主存中读取**。
- volatile 提供了**happens-before 保证**，**对 volatile变量v的写入happens-before 所有其他线程后续对v的读操作。**

## 9.保证可见性的措施

​		除了**volatile**可以让变量保证可见性外，**synchronized、Lock、并发集合、Thread.join()和Thread.start()**等都可以保证可见性，具体见happens-before原则的规定。

## 10.synchronized可见性的正确理解

- synchronized不仅保证了**原子性**，还保证了**可见性**，例如典型的a++。
- synchronized不仅让被保护的代码安全，还保证了happens-before原则

![image-20210706110027458](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210706110027458.png)

## 11.原子性

​		原子性：一系列的操作，**要么全部执行成功，要么全部不执行，不会出现执行一半的情况，是不可分割的。**

### （1）Java中的原子操作

1. 除long和double之外的**基本类型**（int，byte，boolean，short，char，float）的赋值操作。
2. 所有引用**reference的赋值操作**，与实现是32位还是64位无关。
3. java.concurrent.**Atomic**.*包中所有类的原子操作。

### （2）long和double的原子性

​		long和 double，它们的值需要占用64位的内存空间，Java编程语言规范中提到，对于64 位的值的写入，**可以分为两个32位的操作进行写入**。本来一个整体的赋值操作，被拆分为低 32位赋值和高32位赋值两个操作，中间如果发生了其他线程对于这个值的读操作，必然就会读到一个奇怪的值。

​		这个时候我们要使用volatile 关键字进行控制了，**JMM规定了对于volatile long和 volatiledouble，JVM需要保证写入操作的原子性。**

​		所以**在32位上的JVM上，long和 double的操作不是原子的，但是在64位的JVM上是原子的。**实际的商用Java虚拟机不会出现出错情况。

### （3）原子操作+原子操作!=原子操作

​		简单地把原子操作组合在一起，并不能保证整体依然具有原子性。

​		**全同步的HashMap也不完全安全**。

## 12.单例模式8种写法

### （1）单例模式的作用

- 节省内存和计算
- 保证结果正确
- 方便管理

### （2）适用场景

- **无状态的工具类**：比如日志工具类，不管是在哪里使用，我们需要的只是它帮我们记录日志信息，除此之外，并不需要在它的实例对象上存储任何状态，这时候我们就**只需要一个实例对象即可**。
- **全局信息类**：比如我们在一个类上记录网站的访问次数，我们不希望有的访问被记录在对象A上，有的却记录在对象B上，这时候我们就让这个类成为单例。

### （3）饿汉式（静态常量）[可用]

```java
/**
 * 饿汉式（静态常量）（可用）
 */
public class Singleton1 {
    private final static Singleton1 INSTANCE = new Singleton1();

    private Singleton1(){

    }

    public static Singleton1 getInstance(){
        return INSTANCE;
    }
}
```

### （4）饿汉式（静态代码块）[可用]

```java
/**
 * 饿汉式（静态代码块）（可用）
 */
public class Singleton2 {
    private final static Singleton2 INSTANCE;

    static {
        INSTANCE = new Singleton2();
    }
    
    private Singleton2(){

    }

    public static Singleton2 getInstance(){
        return INSTANCE;
    }
}
```

### （5）懒汉式（线程不安全）[不可用]

```java
/**
 * 懒汉式（线程不安全）[不可用]
 */
public class Singleton3 {
    private static Singleton3 INSTANCE;
    
    private Singleton3(){
        
    }
    
    public static Singleton3 getInstance(){
        //两个线程同时到达这里时，可能会多次创建实例，就不符合单例要求
        if(INSTANCE == null){
            INSTANCE = new Singleton3();
        }
        return INSTANCE;
    }
}
```

### （6）懒汉式（线程安全，同步方法）[不推荐用，效率低]

```java
/**
 * 懒汉式（线程安全，同步方法）[不推荐用]
 */
public class Singleton4 {
    private static Singleton4 INSTANCE;

    private Singleton4(){

    }

    public synchronized static Singleton4 getInstance(){
        if(INSTANCE == null){
            INSTANCE = new Singleton4();
        }
        return INSTANCE;
    }
}
```

### （7）懒汉式（线程不安全，同步代码块）[不可用]

```java
/**
 * 懒汉式（线程不安全，同步代码块）[不可用]
 */
public class Singleton5 {
    private static Singleton5 INSTANCE;

    private Singleton5(){

    }

    public static Singleton5 getInstance(){
        if(INSTANCE == null){
            synchronized (Singleton5.class){
                INSTANCE = new Singleton5();
            }
        }
        return INSTANCE;
    }
}
```

### （8）※双重检查，也称双重锁[推荐面试用]

```java
/**
 * 双重检查，也称双重锁[推荐面试用]
 */
public class Singleton6 {
    private volatile static Singleton6 INSTANCE;

    private Singleton6(){

    }

    public static Singleton6 getInstance(){
        if(INSTANCE == null){
            synchronized (Singleton6.class){
                if(INSTANCE == null){
                    INSTANCE = new Singleton6();
                }
            }
        }
        return INSTANCE;
    }
}
```

​		优点：线程安全；延迟加载；效率较高

- 为什么要double-check
  - **线程安全**
  - **单check不行，会多次创建实例**
  - **synchronized方法不行，要考虑性能问题**
- 为什么要用volatile
  - **新建对象实际上有3个步骤，并非原子操作，很可能产生重排序（以下为正常顺序）：**
    1. 创建空对象
    2. 调用构造方法
    3. 将实例赋值给引用
  - **重排序会带来NPE（空指针异常）**
  - **防止重排序**

### （9）静态内部类[可用]

```java
/**
 * 静态内部类[可用]
 */
public class Singleton7 {

    private Singleton7(){

    }

    private static class SingletonInstance{
        //JVM类的加载的性质保证即使多个线程同时访问，也不会创建多个实例
        private static final Singleton7 INSTANCE = new Singleton7();
    }
    
    public static Singleton7 getInstance(){
        return SingletonInstance.INSTANCE;
    }
}
```

### （10）※枚举（生产实践中最佳的单例模式写法）[推荐用]

```java
/**
 * 枚举（生产实践中最佳的单例模式写法）[推荐用]
 */
public enum Singleton8 {
    INSTANCE;
    
    //任意方法
    public void whatever(){
        
    }
}
```

### （11）不同写法对比

- 饿汉：简单，但是没有lazy loading，即需要时加载
- 懒汉：有线程安全问题
- 静态内部类：可用
- ※双重检查：面试用
- ※枚举：最好
  - **写法简单**
  - **线程安全有保障**
  - **避免反序列化破坏单例**

### （12）各种写法的适用场合

- **最好的方法是利用枚举，因为还可以防止反序列化重新创建新的对象**;
- **非线程同步的方法不能使用**;
- 如果程序一开始要**加载的资源太多，那么就应该使用懒加载**;
- 饿汉式如果是**对象的创建需要配置文件就不适用**。
- 懒加载虽然好，但是静态内部类这种方式会**引入编程复杂性**，大部分情况还是推荐枚举，简单。

# 十、死锁

## 1.死锁的影响

​		死锁的影响在不同系统中是不一样的，这取决于系统对死锁的处理能力。

- 数据库：检测并放弃事务
- JVM：无法自动处理

## 2.必然死锁的例子

```java
/**
 * 必定发生死锁的情况
 */
public class MustDeadLock implements Runnable{
    int flag = 1;

    static Object o1 = new Object();
    static Object o2 = new Object();

    public static void main(String[] args) {
        MustDeadLock r1 = new MustDeadLock();
        MustDeadLock r2 = new MustDeadLock();
        r1.flag = 1;
        r2.flag = 0;
        Thread t1 = new Thread(r1);
        Thread t2 = new Thread(r2);
        t1.start();
        t2.start();
    }

    @Override
    public void run() {
        System.out.println("flag = " + flag);

        if(flag == 1){
            synchronized (o1){
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (o2){
                    System.out.println("线程1成功拿到两把锁");
                }
            }
        }
        if(flag == 0){
            synchronized (o2){
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (o1){
                    System.out.println("线程2成功拿到两把锁");
                }
            }
        }
    }
}

flag = 1
flag = 0
陷入阻塞
```

## 3.转账发生死锁

```java
/**
 * 转账时遇到死锁，一旦打开注释，便会发生死锁
 */
public class TransferMoney implements Runnable{
    int flag = 1;
    static Account a = new Account(500);
    static Account b = new Account(500);

    public static void main(String[] args) throws InterruptedException {
        TransferMoney r1 = new TransferMoney();
        TransferMoney r2 = new TransferMoney();
        r1.flag = 1;
        r2.flag = 0;
        Thread t1 = new Thread(r1);
        Thread t2 = new Thread(r2);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println("a的余额为：" + a.balance);
        System.out.println("b的余额为：" + b.balance);
    }

    @Override
    public void run() {
        if(flag == 1){
            transferMoney(a,b,100);
        }
        if(flag == 0){
            transferMoney(b,a,200);
        }
    }
    public static void transferMoney(Account from,Account to,int amount){
        synchronized (from){
            /*try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }*/
            synchronized (to){
                if(from.balance - amount < 0){
                    System.out.println("余额不足，转账失败");
                }
                from.balance -= amount;
                to.balance += amount;
                System.out.println("成功转账" + amount + "元");
            }
        }
    }

    static class Account{
        int balance;

        public Account(int balance) {
            this.balance = balance;
        }
    }
}

成功转账100元
成功转账200元
a的余额为：600
b的余额为：400
```

## 4.多人转账仍会发生死锁

```java
/**
 * 多人同时转账，依然很危险
 */
public class MultiTransferMoney {

    private  static final int NUM_ACCOUNTS = 500;
    private  static final int NUM_MONEY = 1000;
    private  static final int NUM_ITERATIONS = 1000000;
    private  static final int NUM_THREADS = 20;

    public static void main(String[] args) {
        Random rnd = new Random();
        Account[] accounts = new Account[NUM_ACCOUNTS];
        for (int i = 0; i < accounts.length; i++) {
            accounts[i] = new Account(NUM_MONEY);
        }

        class TransferThread extends Thread{

            @Override
            public void run(){
                for (int i = 0; i < NUM_ITERATIONS; i++) {
                    int fromAcct = rnd.nextInt(NUM_ACCOUNTS);
                    int toAcct = rnd.nextInt(NUM_ACCOUNTS);
                    int amount = rnd.nextInt(NUM_MONEY);
                    TransferMoney.transferMoney(accounts[fromAcct],accounts[toAcct],amount);
                }
            }
        }

        for (int i = 0; i < NUM_THREADS; i++) {
            new TransferThread().start();
        }
    }
}
```

## 5.死锁的四个必要条件

- 互斥
- 请求并保持
- 不剥夺
- 循环等待

## 6.定位死锁

### （1）jstack

![image-20210707105254172](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210707105254172.png)

![image-20210707105314762](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210707105314762.png)

![image-20210707105351062](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210707105351062.png)

### （2）ThreadMXBean

```java
/**
 * 用ThreadMXBean检测死锁
 */
public class ThreadMXBeanDetection implements Runnable{
    int flag = 1;

    static Object o1 = new Object();
    static Object o2 = new Object();

    public static void main(String[] args) throws InterruptedException {
        ThreadMXBeanDetection r1 = new ThreadMXBeanDetection();
        ThreadMXBeanDetection r2 = new ThreadMXBeanDetection();
        r1.flag = 1;
        r2.flag = 0;
        Thread t1 = new Thread(r1);
        Thread t2 = new Thread(r2);
        t1.start();
        t2.start();
        Thread.sleep(1000);

        ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
        long[] deadlockedThreads = threadMXBean.findDeadlockedThreads();
        if(deadlockedThreads != null && deadlockedThreads.length > 0){
            for (int i = 0; i < deadlockedThreads.length; i++) {
                ThreadInfo threadInfo =  threadMXBean.getThreadInfo(deadlockedThreads[i]);
                System.out.println("发现死锁：" + threadInfo.getThreadName());
            }
        }
    }

    @Override
    public void run() {
        System.out.println("flag = " + flag);

        if(flag == 1){
            synchronized (o1){
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (o2){
                    System.out.println("线程1成功拿到两把锁");
                }
            }
        }
        if(flag == 0){
            synchronized (o2){
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (o1){
                    System.out.println("线程2成功拿到两把锁");
                }
            }
        }
    }
}

flag = 0
flag = 1
发现死锁：Thread-1
发现死锁：Thread-0
```

## 7.死锁修复策略

- 避免策略（哲学家就餐的换手方案、转账换序方案）
- 检测与恢复策略（一段时间检测是否有死锁，如果有就剥夺某一个资源，来打开死锁）
- 鸵鸟策略：鸵鸟这种动物在遇到危险的时候，通常就会把头埋在地上，这样一来它就看不到危险了。而鸵鸟策略的意思就是说，如果我们发生死锁的概率极其低，那么我们就直接忽略它，直到死锁发生的时候，再人工修复。

### （1）避免策略——转账换序方案

​		如果我们能避免相反的获取锁的顺序，就可以避免死锁的发生，所以我们从这个角度出发，去避免死锁。

​		其实我们在转账时只要能获取到两把锁就可以，**而不在乎这两把锁的获取顺序**，无论是先获取哪把锁或者后获取哪把锁，只要最终获取到了两把锁，就可以开始转账。**我们使用HashCode的值来决定顺序**，从而保证了线程安全。

```java
/**
 * 转账时遇到死锁，一旦打开注释，便会发生死锁
 */
public class TransferMoney implements Runnable{
    int flag = 1;
    static Account a = new Account(500);
    static Account b = new Account(500);
    static Object lock = new Object();

    public static void main(String[] args) throws InterruptedException {
        TransferMoney r1 = new TransferMoney();
        TransferMoney r2 = new TransferMoney();
        r1.flag = 1;
        r2.flag = 0;
        Thread t1 = new Thread(r1);
        Thread t2 = new Thread(r2);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println("a的余额为：" + a.balance);
        System.out.println("b的余额为：" + b.balance);
    }

    @Override
    public void run() {
        if(flag == 1){
            transferMoney(a,b,100);
        }
        if(flag == 0){
            transferMoney(b,a,200);
        }
    }
    public static void transferMoney(Account from,Account to,int amount){
        class Helper{

            public void transfer(){
                if(from.balance - amount < 0){
                    System.out.println("余额不足，转账失败");
                }
                from.balance -= amount;
                to.balance += amount;
                System.out.println("成功转账" + amount + "元");
            }

        }
        int fromHash = System.identityHashCode(from);
        int toHash = System.identityHashCode(to);
        if(fromHash < toHash)
        {
            synchronized (from){
                synchronized (to){
                    new Helper().transfer();
                }
            }
        }else if(fromHash > toHash) {
            synchronized (to){
                synchronized (from){
                    new Helper().transfer();
                }
            }
        }else{
            //公平竞争lock锁
            synchronized (lock){
                synchronized (to){
                    synchronized (from){
                        new Helper().transfer();
                    }
                }
            }
        }
    }

    static class Account{
        int balance;

        public Account(int balance) {
            this.balance = balance;
        }
    }
}

成功转账100元
成功转账200元
a的余额为：600
b的余额为：400
```

### （2）哲学家就餐问题演示

```java
/**
 * 演示哲学家就餐问题导致的死锁
 */
public class DiningPhilosophers {

    public static class Philosopher implements Runnable{
        private Object leftChopstick;
        private Object rightChopstick;

        public Philosopher(Object leftChopstick, Object rightChopstick) {
            this.leftChopstick = leftChopstick;
            this.rightChopstick = rightChopstick;
        }

        @Override
        public void run() {
            try{
                while(true){
                    doAction("Thingking");
                    synchronized (leftChopstick){
                        doAction("Picked up left chopstick");
                        synchronized (rightChopstick){
                            doAction("Picked up right chopstick and to eating");
                            doAction("Put down right chopstick");
                        }
                        doAction("Put down left chopstick");
                    }
                }
            }catch(InterruptedException e){
                e.printStackTrace();
            }
        }
        private void doAction(String action) throws InterruptedException {
            System.out.println(Thread.currentThread().getName() + " " + action);
            Thread.sleep((long)Math.random() * 10);//随机休眠[0,10)秒
        }
    }

    public static void main(String[] args) {
        Philosopher[] philosophers = new Philosopher[5];
        Object[] chopsticks = new Object[philosophers.length];
        for (int i = 0; i < chopsticks.length; i++) {
            chopsticks[i] = new Object();
        }
        for (int i = 0; i < philosophers.length; i++) {
            Object leftChopstick = chopsticks[i];
            Object rightChopstick = chopsticks[(i + 1) % chopsticks.length];
            philosophers[i] = new Philosopher(leftChopstick,rightChopstick);
            new Thread(philosophers[i],"哲学家" + (i + 1) + "号").start();
        }
    }
}
```

### （3）解决哲学家死锁的4种方法

#### ①服务员检查（避免策略）

​		每次哲学家要拿起左边的筷子时去询问服务员是否可以拿起，若拿起后陷入死锁，则禁止拿起，等不会发生死锁时在允许拿起。

#### ②改变一个哲学家拿叉子的顺序（避免策略）

​		破坏环路，即循环等待。

```java
/**
 * 演示哲学家就餐问题导致的死锁
 */
public class DiningPhilosophers {

    public static class Philosopher implements Runnable{
        private Object leftChopstick;
        private Object rightChopstick;

        public Philosopher(Object leftChopstick, Object rightChopstick) {
            this.leftChopstick = leftChopstick;
            this.rightChopstick = rightChopstick;
        }

        @Override
        public void run() {
            try{
                while(true){
                    doAction("Thingking");
                    synchronized (leftChopstick){
                        doAction("Picked up left chopstick");
                        synchronized (rightChopstick){
                            doAction("Picked up right chopstick and to eating");
                            doAction("Put down right chopstick");
                        }
                        doAction("Put down left chopstick");
                    }
                }
            }catch(InterruptedException e){
                e.printStackTrace();
            }
        }
        private void doAction(String action) throws InterruptedException {
            System.out.println(Thread.currentThread().getName() + " " + action);
            Thread.sleep((long)Math.random() * 10);//随机休眠[0,10)秒
        }
    }

    public static void main(String[] args) {
        Philosopher[] philosophers = new Philosopher[5];
        Object[] chopsticks = new Object[philosophers.length];
        for (int i = 0; i < chopsticks.length; i++) {
            chopsticks[i] = new Object();
        }
        for (int i = 0; i < philosophers.length; i++) {
            Object leftChopstick = chopsticks[i];
            Object rightChopstick = chopsticks[(i + 1) % chopsticks.length];
            if(i == philosophers.length - 1){
                philosophers[i] = new Philosopher(rightChopstick,leftChopstick);
            }else{
                philosophers[i] = new Philosopher(leftChopstick,rightChopstick);
            }
            new Thread(philosophers[i],"哲学家" + (i + 1) + "号").start();
        }
    }
}
```



#### ③餐票（避免策略）

​		总共只给四张餐票，必须有餐票才可以吃。

#### ④领导调节（检测与恢复策略）

​		若发生死锁，则强制剥夺一个哲学家的筷子资源。

### （4）死锁检测与恢复策略

#### ①检测算法

- 允许发生死锁
- 每次调用锁都记录
- 定期检查“**锁的调用链路图**”中是否存在环路
- 一旦发生死锁，就用死锁恢复机制进行恢复

#### ②恢复方法1：进程终止

​		逐个终止线程，直到死锁清除

​		终止顺序：

- 优先级（是前台交互还是后台处理）
- 已占用资源与还需要的资源
- 已经运行时间

#### ③恢复方法2：资源抢占

​		把已经分发出去的锁收回来，让线程回退几步，这样就不用结束整个线程，成本比较低。

​		缺点：可能同一个线程一直被抢占，造成饥饿

## 8.实际工程中避免死锁的8个tips

1. 设置超时时间（Lock的**tryLock**）

   ```java
   /**
    * 用tryLock避免死锁
    */
   public class TryLockDeadLock implements Runnable{
   
       int flag = 1;
       static Lock lock1 = new ReentrantLock();
       static Lock lock2 = new ReentrantLock();
   
       public static void main(String[] args) {
           TryLockDeadLock r1 = new TryLockDeadLock();
           TryLockDeadLock r2 = new TryLockDeadLock();
           r1.flag = 1;
           r2.flag = 0;
           new Thread(r1).start();
           new Thread(r2).start();
       }
   
       @Override
       public void run() {
           for (int i = 0; i < 100; i++) {
               if(flag == 1){
                   try {
                       if(lock1.tryLock(800, TimeUnit.MILLISECONDS)){
                           Thread.sleep(new Random().nextInt(1000));
                           if(lock2.tryLock(800,TimeUnit.MILLISECONDS)){
                               System.out.println("线程1成功获取到了两把锁");
                               lock2.unlock();
                               lock1.unlock();
                               break;
                           }else {
                               System.out.println("线程1获取锁2失败，已重试");
                           }
                       }else {
                           System.out.println("线程1获取锁1失败，已重试");
                           lock1.unlock();
                           Thread.sleep(new Random().nextInt(1000));
                       }
                   } catch (InterruptedException e) {
                       e.printStackTrace();
                   }
               }
               if(flag == 0){
                   try {
                       if(lock2.tryLock(3000, TimeUnit.MILLISECONDS)){
                           Thread.sleep(new Random().nextInt(1000));
                           if(lock1.tryLock(3000,TimeUnit.MILLISECONDS)){
                               System.out.println("线程2成功获取到了两把锁");
                               lock1.unlock();
                               lock2.unlock();
                               break;
                           }else {
                               System.out.println("线程2获取锁2失败，已重试");
                           }
                       }else {
                           System.out.println("线程2获取锁1失败，已重试");
                           lock2.unlock();
                           Thread.sleep(new Random().nextInt(1000));
                       }
                   } catch (InterruptedException e) {
                       e.printStackTrace();
                   }
               }
           }
       }
   }
   
   线程2获取到了锁2
   线程1获取到了锁1
   线程1获取锁2失败，已重试
   线程2成功获取到了两把锁
   线程1获取到了锁1
   线程1成功获取到了两把锁
   ```

2. 多使用**并发类**而不是自己设计锁，少用同步集合（如synchronized），同步集合的可拓展性相对差，效率低。

3. 尽量降低锁的使用**粒度**：多用几个不同的锁而不是一个锁

4. 如果能使用**同步代码块**，就不适用同步方法：自己指定对象

5. 给线程起有意义的**名字**

6. 避免**锁的嵌套**：MustDeadLock类

7. 分配资源前先看能不能**收回来**：银行家算法

8. 尽量不要几个功能用同一把锁：**专锁专用**

## 9.其他活性故障

### （1）活锁

#### ①概念

- 死锁：每个哲学家都拿着左边的餐叉，**永远都在等**右边的餐叉

- 活锁：同时拿起左边的餐叉，等待一段时间后同时放下手中的餐叉，过一段时间**又同时**拿起这些餐叉。

  定义：虽然线程没有阻塞，也**始终在运行**，但**程序得不到进展**，因为线程始终重复做同样的事，**有死锁与资源耗尽问题的风险**，而**死锁时是不消耗CPU资源**的。

#### ②代码演示

```java
/**
 * 描述：     演示活锁问题
 */
public class LiveLock {

    static class Spoon {

        private Diner owner;

        public Spoon(Diner owner) {
            this.owner = owner;
        }

        public Diner getOwner() {
            return owner;
        }

        public void setOwner(Diner owner) {
            this.owner = owner;
        }

        public synchronized void use() {
            System.out.printf("%s吃完了!", owner.name);


        }
    }

    static class Diner {

        private String name;
        private boolean isHungry;

        public Diner(String name) {
            this.name = name;
            isHungry = true;
        }

        public void eatWith(Spoon spoon, Diner spouse) {
            while (isHungry) {
                if (spoon.owner != this) {
                    try {
                        Thread.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    continue;
                }
                if (spouse.isHungry) {
                    System.out.println(name + ": 亲爱的" + spouse.name + "你先吃吧");
                    spoon.setOwner(spouse);
                    continue;
                }

                spoon.use();
                isHungry = false;
                System.out.println(name + ": 我吃完了");
                spoon.setOwner(spouse);

            }
        }
    }


    public static void main(String[] args) {
        Diner husband = new Diner("牛郎");
        Diner wife = new Diner("织女");

        Spoon spoon = new Spoon(husband);

        new Thread(new Runnable() {
            @Override
            public void run() {
                husband.eatWith(spoon, wife);
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                wife.eatWith(spoon, husband);
            }
        }).start();
    }
}
```

#### ③如何解决活锁问题

​		典型例子：**以太网的指数退避算法**。

​		我们以此为参照，加入随机因素。

```java
Random random = new Random();
if (spouse.isHungry && random.nextInt(10) < 9) {
	System.out.println(name + ": 亲爱的" + spouse.name + "你先吃吧");
	spoon.setOwner(spouse);
    continue;
}

牛郎: 亲爱的织女你先吃吧
织女: 亲爱的牛郎你先吃吧
牛郎: 亲爱的织女你先吃吧
织女吃完了!织女: 我吃完了
牛郎吃完了!牛郎: 我吃完了
```

#### ④工程中的活锁实例：消息队列

​		策略：消息如果处理失败，就放在队列开头重试，但可能由于依赖服务出问题，处理该消息一直失败，程序没有阻塞，但程序无法继续。

​		解决：**放到队列尾部或重试限制。**

### （2）饥饿

​		当线程需要某些资源时（主要为CPU），但是**始终获取不到。**

​		例如如果我们把线程的**优先级设置得过于低**（Java中优先级分为1到10），或者有某线**程持有锁同时又无限循环从而不释放锁**，或者**某程序始终占用某文件的写锁**，这些情况**都可能导致“饥饿”的发生**。

​		饥饿可能会导致响应性差:比如，我们的浏览器有一个线程负责处理前台响应（打开收藏夹等动作），另外的后台线程负责下载图片和文件、计算渲染等。在这种情况下，如果后台线程把CPU资源都占用了，那么前台线程将无法得到很好地执行，这会导致用户的体验很差。