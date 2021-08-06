# 一、线程池

## 1.概念

​		如果不使用线程池，每个任务都新开一个线程处理：

- 一个线程

  ```java
  /**
   * 一个线程
   */
  public class EveryTaskOneThread {
  
      public static void main(String[] args) {
          Thread thread = new Thread(new Task());
          thread.start();
      }
  
      static class Task implements Runnable{
  
          @Override
          public void run() {
              System.out.println("执行任务");
          }
      }
  }
  
  执行任务
  ```

- for循环创建线程

  ```java
  /**
   * for循环创建线程
   */
  public class ForLoop {
  
      public static void main(String[] args) {
          for (int i = 0; i < 1000; i++) {
              Thread thread = new Thread(new Task());
              thread.start();
          }
      }
  
      static class Task implements Runnable{
  
          @Override
          public void run() {
              System.out.println("执行任务");
          }
      }
  }
  ```

- 这样开销太大，我们希望有固定数量的线程去执行这1000个线程，这样就避免了**反复创建并销毁线程**所带来的**开销问题**，容易报OOM（Out Of Memory）异常。

### （1）为什么要使用线程池

- 问题1：**反复创建线程开销大**
- 问题2：过多的线程会**占用太多内存**

### （2）线程池的优点

- 加快**响应速度**
- 合理利用**CPU和内存**
- **统一管理**

### （3）线程池适合应用的场合

- 服务器接收到大量请求时，适用线程池技术是非常合适的。
- 在实际开发中，**若要创建5个以上线程**，那么就可以使用线程池来管理。

## 2.创建线程池

### （1）线程池构造函数的参数

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210707182302676.png" alt="image-20210707182302676" style="zoom:67%;" />

#### ①corePoolSize和maximumPoolSize

- 线程池可能会在核心线程数的基础上，额外增加一些线程，但是新增加的线程数有一个上限，这就是最大量`maximumPoolSize`。

  <img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210707182626586.png" alt="image-20210707182626586" style="zoom:67%;" />

- 添加线程规则如下：

  - `线程数 < corePoolSize`，**即使其他工作线程处于空闲状态，也会创建一个新线程来运行新任务**（因为目前线程数不够）。
  - 如果队列未满且`corePoolSize ≤ 线程数 < maximumPoolSize`，则**将任务放入队列**。
  - 如果队列已满且`线程数 < maximumPoolSize`,则**创建一个新线程来运行任务**。
  - 如果队列已满且`maximumPoolSize ≤ 线程数`，则**拒绝该任务**。

  <img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210707183356635.png" alt="image-20210707183356635" style="zoom:67%;" />

- 增减线程的特点：

  - 通过设置`corePoolSize`和`maximumPoolSize`相同，就可以**创建固定大小的线程池**。
  - **线程池希望保持较少的线程数**，并且只有在负载变得很大时才增加它。
  - 通过设置`maximumPoolSize`为很高的值，例如`Integer.MAX_VALUE`，可以**允许线程池容纳任意数量的并发任务**。
  - 只有在队列填满时才创建多于`corePoolSize`的线程。所以如果使用的是无界队列（例如`LinkedBlockingQueue`），那么线程数就不会超过`corePoolSize`。

#### ②keepAliveTime

​		如果线程池当前的线程数多于`corePoolSize`，那么如果多余的**线程空闲时间超过`keepAliveTime`，它们就会被终止。**

#### ③ThreadFactory用来创建线程

- 新的线程是由`ThreadFactory`创建的，默认使用`Executors.defaultThreadFactory()`，创建出的线程都是同一线程组，拥有同样的`NORM_PRIORITY`优先级且都是用户线程。若自己指定`ThreadFactory`，那么就可以改变线程名、线程组、优先级、是否是守护线程等。

#### ④workQueue

​		三种最常见的队列类型：

- 直接交换：`SynchronousQueue`，**做一个简单的中转，并没有容量**。
- 无界队列：`LinkedBlockingQueue`，**容量无限大**。
- 有界队列：`ArrayBlockingQueue`。

### （2）手动创建or自动创建

- 手动创建更好，这样可以让我们更加明确线程池的运行规则，避免资源耗尽的风险。
- 自动创建会带来风险

#### ①newFixedThreadPool

```java
/**
 * 演示newFixedThreadPool
 */
public class FixedThreadPoolTest {

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(4);
        for (int i = 0; i < 1000; i++) {
            executorService.execute(new Task());
        }

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

​		源码如下：

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

​		由于`LinkedBlockingQueue`是没有容量上限的，所以当请求数越来越多时，即请求堆积时，容易造成占用大量内存，**可能会导致OOM。**

```java
/**
 * 演示newFixedThreadPool发生OOM错误
 */
public class FixedThreadPoolOOM {

    private static ExecutorService executorService = Executors.newFixedThreadPool(1);

    public static void main(String[] args) {
        for (int i = 0; i < Integer.MAX_VALUE; i++) {
            executorService.execute(new SubThread());
        }
    }
}

class SubThread implements Runnable{

    @Override
    public void run() {
        try {
            Thread.sleep(1000000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

Exception in thread "main" java.lang.OutOfMemoryError: GC overhead limit exceeded
	at threadpool.FixedThreadPoolOOM.main(FixedThreadPoolOOM.java:15)
```

#### ②newSingleThreadExecutor

```java
/**
 * 演示newSingleThreadExecutor
 */
public class SingleThreaExecutor {
    
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newSingleThreadExecutor();
        for (int i = 0; i < 1000; i++) {
            executorService.execute(new Task());
        }
    }
}
```

​		源码如下：

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

​		可以看出，这里和刚才的`newFixedThreadPool`原理基本一致，只不过把线程数直接设置为1，所以也会导致同样的问题，即请求堆积时，可能会占用大量内存。

#### ③newCachedThreadPool

- 可缓存线程池
- 特点：无界线程池，具有自动回收多余线程的功能

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210707195558792.png" alt="image-20210707195558792" style="zoom:67%;" />

```java
/**
 * 演示newCachedThreadPool
 */
public class CachedThreadPool {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < 1000; i++) {
            executorService.execute(new Task());
        }
    }
}
```

​		源码如下：

```java
return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
```

​		这里的弊端在于第二个参数`maximumPoolSize`被设置为`Integer.MAX_VALUE`，这**可能会创建数量非常多的线程，甚至导致OOM。**

​		采用SynchronousQueue装等待的任务，这个阻塞队列没有存储空间，这意味着**只要有请求到来，就必须要找到一条工作线程处理他，如果当前没有空闲的线程，那么就会再创建一条新的线程。**

#### ④newScheduledThreadPool

- 支持**定时及周期性任务执行**的线程池

```java
/**
 * 演示newScheduledThreadPool
 */
public class ScheduledThreadPool {

    public static void main(String[] args) {
        ScheduledExecutorService threadPool = Executors.newScheduledThreadPool(10);
        //threadPool.schedule(new Task(),1, TimeUnit.SECONDS);
        threadPool.scheduleAtFixedRate(new Task(),1,3,TimeUnit.SECONDS);
    }
}
```

​		源码如下：

```java
public ScheduledThreadPoolExecutor(int corePoolSize) {
    //extends ThreadPoolExecutor
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
          new DelayedWorkQueue());
}
```

#### ⑤总结

- 正确的创建线程池方法需要**根据不同的业务场景，自己设置线程池参数**，比如内存大小、线程名字等。
- 线程池里的线程数量设定为多少比较合适：
  - CPU密集型（加密、计算hash等）：**最佳线程数为CPU核心数的1-2倍左右。**
  - 耗时IO型（读写数据库、文件、网络读写等）：最佳线程数一般会大于CPU核心数很多倍，**以JVM线程监控显示繁忙情况为依据，保证线程空闲可以衔接**上，参考Brain Goetz推荐的计算方法：$线程数=CPU核心数×(1+\frac{平均等待时间}{平均工作时间})$。

### （3）常见线程池特点

#### ①FixedThreadPool

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210709112012671.png" alt="image-20210709112012671" style="zoom:67%;" />

​		当请求数越来越多时，即请求堆积时，容易造成占用大量内存，**可能会导致OOM。**

#### ②CachedThreadPool

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210709112127964.png" alt="image-20210709112127964" style="zoom: 67%;" />

​		无界线程池，具有自动回收多余线程的功能

#### ③ScheduledThreadPool

​		支持定时及周期性任务执行的线程池。

#### ④SingleThreadExecutor

- 单线程的线程池：它只会用唯一的工作线程来执行任务
- 它的原理和`FixedThreadPool`一样，但此时的线程数量被设置为了1.

#### ⑤以上4种线程池的构造函数的参数

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210709113114937.png" alt="image-20210709113114937" style="zoom:67%;" />

#### ⑥阻塞队列分析

- `FixedThreadPool`和`SingleThreadPool`的Queue是`LinkedBlockingQueue`

  线程数量固定，无法继续增长，所以不得不使用无界的阻塞队列。

- `CachedThreadPool`使用的Queue是`SynchronousQueue`

  该线程池无界，所以不需要阻塞队列存储，直接交付给线程执行即可。

- `ScheduledThreadPool`使用的是延迟队列`DelayedWorkQueue`

  延迟队列可以把任务根据时间先后做一定的延迟。

#### ⑦JDK1.8：workStealingPool

- 若某个任务可以产生子任务，才符合该线程池的使用场景
- 每个子线程之间可以窃取其他线程的任务，帮助并行执行，提高效率，所以**子任务最好不要加锁**
- **不保证执行顺序**

## 3.停止线程池

### （1）shutdown

​		该方法不会立刻停止所有正在执行的线程，只是**会拒绝新申请加入的线程任务。**

### （2）isShutdown

​		线程是否已经shutdown。

### （3）isTerminated

​		线程是否全部运行完毕。

```java
/**
 * 演示关闭线程池shutdown、isShutdown、isTerminated
 */
public class ShutDown {

    public static void main(String[] args) throws InterruptedException {
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 1000; i++) {
            executorService.execute(new ShutDownTask());
        }
        Thread.sleep(100);
        System.out.println(executorService.isShutdown());//false
        executorService.shutdown();
        System.out.println(executorService.isShutdown());//true
        System.out.println(executorService.isTerminated());//false
        Thread.sleep(10000);
        System.out.println(executorService.isTerminated());//true

        //executorService.execute(new ShutDownTask());会报错
    }
}
class ShutDownTask implements Runnable{

    @Override
    public void run() {
        try {
            Thread.sleep(5);
            System.out.println(Thread.currentThread().getName());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

### （4）awaitTermination

​		等待一段时间，若在这段时间内线程执行完毕则返回true，否则返回false。所以该方法是测试某线程一段时间内能否停止的方法。

```java
/**
 * 演示关闭线程池shutdown、awaitTermination
 */
public class ShutDown {

    public static void main(String[] args) throws InterruptedException {
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 100; i++) {
            executorService.execute(new ShutDownTask());
        }
        executorService.shutdown();
        boolean b = executorService.awaitTermination(1L, TimeUnit.SECONDS);
        System.out.println(b);
    }
}
class ShutDownTask implements Runnable{

    @Override
    public void run() {
        try {
            Thread.sleep(90);
            System.out.println(Thread.currentThread().getName());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

### （5）shutdownNow

​		立刻把线程池关闭。用中断的信号去`interrupt`正在执行的线程，队列中的线程则直接以列表的形式返回。

```java
package threadpool;

import java.util.List;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

/**
 * 演示关闭线程池shutdown
 */
public class ShutDown {

    public static void main(String[] args) throws InterruptedException {
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 100; i++) {
            executorService.execute(new ShutDownTask());
        }
        Thread.sleep(200);
        List<Runnable> runnableList = executorService.shutdownNow();
        System.out.println(runnableList.size());//70
    }
}
class ShutDownTask implements Runnable{

    @Override
    public void run() {
        try {
            Thread.sleep(90);
            System.out.println(Thread.currentThread().getName());
        } catch (InterruptedException e) {
            System.out.println(Thread.currentThread().getName() + "被中断了");
        }
    }
}
```

## 4.任务太多，怎么拒绝

- 拒绝时机：
  - 当Executor关闭时，提交新任务会被拒绝。
  - 当Executor对最大线程和工作队列容量**使用有限边界并且已经饱和时**，提交新任务会被拒绝。

- 拒绝策略
  1. AbortPolicy：抛出异常
  2. DiscardPolicy：默默丢弃新提交的任务
  3. DiscardOldestPolicy：默默丢弃最老的任务
  4. CallerRunsPolicy：让提交任务的线程去执行，而不是本线程池执行

- 钩子方法：可以在每个任务执行前后做一些日志、统计处理

```java
/**
 * 演示每个任务的执行前后都可以放钩子函数
 */
public class PauseableThreadPool extends ThreadPoolExecutor {

    private boolean isPaused;
    private final ReentrantLock lock = new ReentrantLock();
    private Condition unpaused = lock.newCondition();

    public PauseableThreadPool(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue);
    }

    public PauseableThreadPool(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, threadFactory);
    }

    public PauseableThreadPool(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, RejectedExecutionHandler handler) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, handler);
    }

    public PauseableThreadPool(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, threadFactory, handler);
    }

    @Override
    protected void beforeExecute(Thread t, Runnable r) {
        super.beforeExecute(t, r);
        lock.lock();
        try{
            while(isPaused){
                unpaused.await();
            }
        }catch (InterruptedException e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }



    private void pause(){
        lock.lock();
        try{
            isPaused=true;
        }finally {
            lock.unlock();
        }
    }

    public void resume(){
        lock.lock();
        try{
            isPaused=false;
            unpaused.signalAll();
        }finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        PauseableThreadPool pauseableThreadPool = new PauseableThreadPool(10,20,10L,TimeUnit.SECONDS,new LinkedBlockingDeque<>());
        Runnable runnable = new Runnable(){

            @Override
            public void run() {
                System.out.println("我被执行");
                try{
                    Thread.sleep(10);
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
        };
        for (int i = 0; i < 10000; i++) {
            pauseableThreadPool.execute(runnable);
        }
        Thread.sleep(1500);
        pauseableThreadPool.pause();
        System.out.println("已经暂停！！！！！！！！！！！");
        Thread.sleep(1500);
        pauseableThreadPool.resume();
        System.out.println("已经恢复！！！！！！！！！！！");
    }
}
```

## 5.Executor家族辨析

- 线程池组成部分如下
  - 线程池管理器
  - 工作线程
  - 任务队列（支持并发）
  - 任务接口（Task）

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210716001216022.png" alt="image-20210716001216022" style="zoom:67%;" />

### （1）Executor家族

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210716001327647.png" alt="image-20210716001327647" style="zoom:50%;" />

#### ①Executor

​		源码如下：

```java
public interface Executor {
    void execute(Runnable command);
}
```

#### ②ExecutorService

​		源码如下：

```java
public interface ExecutorService extends Executor {
    void shutdown();
    List<Runnable> shutdownNow();
    boolean isShutdown();
    boolean isTerminated();
    boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;
    ....
}
```

#### ③Executors

​		源码如下：

```java
public class Executors {
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
    public static ExecutorService newWorkStealingPool(int parallelism) {
        return new ForkJoinPool
            (parallelism,
             ForkJoinPool.defaultForkJoinWorkerThreadFactory,
             null, true);
    }
    ...
}
```

### （2）线程池实现任务复用的原理

- 相同线程执行不同任务

- 源码如下：

  ```java
  /**
   * ThreadPoolExecutor
   */
      public void execute(Runnable command) {
          if (command == null)
              throw new NullPointerException();
          int c = ctl.get();//ctl记录了线程池状态与线程数
          if (workerCountOf(c) < corePoolSize) {
              if (addWorker(command, true))//若传入false,则是与maximumPoolSize比较；若传入true，则是与corePoolSize比较
                  return;
              c = ctl.get();
          }
          //线程数大于等于corePoolSize后
          if (isRunning(c) && workQueue.offer(command)) {//检查线程池是否再运行，是的话就将任务放入工作队列中
              int recheck = ctl.get();
              if (!isRunning(recheck) && remove(command))//如果线程突然中断就移除该任务
                  reject(command);
              else if (workerCountOf(recheck) == 0)//线程由于异常减少到0，则需要再创建新线程
                  addWorker(null, false);
          }
          else if (!addWorker(command, false))
              reject(command);
      }
  
  
  final void runWorker(Worker w) {
      Thread wt = Thread.currentThread();
      Runnable task = w.firstTask;
      w.firstTask = null;
      w.unlock(); // allow interrupts
      boolean completedAbruptly = true;
      try {
          while (task != null || (task = getTask()) != null) {//整个线程不会停止，周而复始的取下一个任务
              w.lock();
  			...
              try{
                  task.run();
              }
              ...
          }
          completedAbruptly = false;
      } finally {
          processWorkerExit(w, completedAbruptly);
      }
  }
  ```

## 6.线程池状态

- RUNNING：接受新任务并处理排队任务。

- SHUTDOWN：不接受新任务，但处理排队任务。

- STOP：不接受新任务也不处理排队任务，并中断正在进行的任务。

- TIDYING：中文是整洁，即所有任务均已终止，workerCount为0时，线程会转换到TIDYING状态，并将运行terminate()钩子方法。

- TERMINATED：terminate()运行完成。

  ```java
      // runState is stored in the high-order bits
      private static final int RUNNING    = -1 << COUNT_BITS;
      private static final int SHUTDOWN   =  0 << COUNT_BITS;
      private static final int STOP       =  1 << COUNT_BITS;
      private static final int TIDYING    =  2 << COUNT_BITS;
      private static final int TERMINATED =  3 << COUNT_BITS;
  ```

# 二、ThreadLocal

## 1.两大使用场景

- 场景1：每个线程需要一个独享的对象（通常是**工具类**，典型需要使用的类有SimpleDateFormat和Random）
- 场景2：**每个线程内**需要保存全局变量（例如在拦截器中获取用户信息），可以让不同方法直接使用，**避免参数传递**的麻烦。

### （1）场景1

- 每个Thread内有自己的实例副本，**不共享**

​		SimpleDateFormat的进化之路如下：

#### ①2个线程分别用自己的SimpleDateFormat

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210716232454752.png" alt="image-20210716232454752" style="zoom:50%;" />

```java
/**
 * 两个线程打印日期
 */
public class ThreadLocalNormalUsage00 {
    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                String date = new ThreadLocalNormalUsage00().date(10);
                System.out.println(date);
            }
        }).start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                String date = new ThreadLocalNormalUsage00().date(1077);
                System.out.println(date);
            }
        }).start();
    }

    public String date(int seconds){
        //参数单位为毫秒，从1970.01.01 00:00:00 GMT计时,中国时区从00:08:00开始
        Date date = new Date(1000*seconds);
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        return simpleDateFormat.format(date);
    }
}
```

#### ②10个线程分别用自己的SimpleDateFormat

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210716233319489.png" alt="image-20210716233319489" style="zoom:50%;" />

```c++
/**
 * 十个线程分别用自己的SimpleDateFormat
 */
public class ThreadLocalNormalUsage01 {
    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 10; i++) {
            int finalI = i;
            new Thread(new Runnable() {
                @Override
                public void run() {
                    String date = new ThreadLocalNormalUsage01().date(finalI);
                    System.out.println(date);
                }
            }).start();
            Thread.sleep(100);
        }
    }

    public String date(int seconds){
        //参数单位为毫秒，从1970.01.01 00:00:00 GMT计时,中国时区从00:08:00开始
        Date date = new Date(1000*seconds);
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        return simpleDateFormat.format(date);
    }
}
```

​		但是但需求变成1000个时，那么必然要使用线程池，否则创建、销毁带来的开销非常大。

#### ③1000个线程分别用自己的SimpleDateFormat

```java
/**
 * 1000个打印日期的任务，用线程池来执行
 */
public class ThreadLocalNormalUsage02 {

    public static ExecutorService threadPool = Executors.newFixedThreadPool(10);

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 1000; i++) {
            int finalI = i;
            threadPool.submit(new Runnable() {
                @Override
                public void run() {
                    String date = new ThreadLocalNormalUsage02().date(finalI);
                    System.out.println(date);
                }
            });
        }
        threadPool.shutdown();
    }

    public String date(int seconds){
        //参数单位为毫秒，从1970.01.01 00:00:00 GMT计时,中国时区从00:08:00开始
        Date date = new Date(1000*seconds);
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        return simpleDateFormat.format(date);
    }
}
```

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210716234756103.png" alt="image-20210716234756103" style="zoom:50%;" />

​		创建、销毁1000次dateformat对象，浪费资源。

#### ④所有线程共用一个SimpleDateFormat对象

```java
/**
 * 只创建一次SimpleDateFormat对象
 */
public class ThreadLocalNormalUsage03 {

    public static ExecutorService threadPool = Executors.newFixedThreadPool(10);
    public static SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 1000; i++) {
            int finalI = i;
            threadPool.submit(new Runnable() {
                @Override
                public void run() {
                    String date = new ThreadLocalNormalUsage03().date(finalI);
                    System.out.println(date);
                }
            });
        }
        threadPool.shutdown();
    }

    public String date(int seconds){
        //参数单位为毫秒，从1970.01.01 00:00:00 GMT计时,中国时区从00:08:00开始
        Date date = new Date(1000*seconds);
        return simpleDateFormat.format(date);
    }
}
1970-01-01 08:16:34
1970-01-01 08:16:33
1970-01-01 08:16:32
1970-01-01 08:16:32
```

​		我们可以看到有一些时间是相同的，这与之前的效果不一致，这是因为所有线程共用一个SimpleDateFormat对象，而且该对象本身并不是线程安全的，所以发生了线程安全问题。

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210716235437433.png" alt="image-20210716235437433" style="zoom:50%;" />

#### ⑤加锁避免线程安全问题

```java
/**
 * 加锁避免线程安全问题
 */
public class ThreadLocalNormalUsage04 {

    public static ExecutorService threadPool = Executors.newFixedThreadPool(10);
    public static SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 1000; i++) {
            int finalI = i;
            threadPool.submit(new Runnable() {
                @Override
                public void run() {
                    String date = new ThreadLocalNormalUsage04().date(finalI);
                    System.out.println(date);
                }
            });
        }
        threadPool.shutdown();
    }

    public  String date(int seconds){
        //参数单位为毫秒，从1970.01.01 00:00:00 GMT计时,中国时区从00:08:00开始
        Date date = new Date(1000*seconds);
        String s = null;
        synchronized (ThreadLocalNormalUsage04.class){
            s = simpleDateFormat.format(date);
        }
        return s;
    }
}
```

#### ⑥更好的解决方法是使用ThreadLocal

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210717000920411.png" alt="image-20210717000920411" style="zoom:50%;" />

```java
/**
 * 利用ThreadLocal，给每个线程分配自己的DateFormat对象，保证了线程安全，高效利用内存
 */
public class ThreadLocalNormalUsage05 {

    public static ExecutorService threadPool = Executors.newFixedThreadPool(10);

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 1000; i++) {
            int finalI = i;
            threadPool.submit(new Runnable() {
                @Override
                public void run() {
                    String date = new ThreadLocalNormalUsage05().date(finalI);
                    System.out.println(date);
                }
            });
        }
        threadPool.shutdown();
    }

    public String date(int seconds){
        //参数单位为毫秒，从1970.01.01 00:00:00 GMT计时,中国时区从00:08:00开始
        Date date = new Date(1000*seconds);
        //SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        //去获取对象而不是创建对象，在每个线程中只有1份，所以最终只有10份
        //这里也与我们的线程池是newFixedThreadPool有关，本例它只会创建10个线程，是固定的
        //后面的线程接收到新的Task后，去调用get()时都不会调用initialValue方法
        SimpleDateFormat simpleDateFormat = ThreadSafeFormatter.dateFormatThreadLocal2.get();
        return simpleDateFormat.format(date);
    }
}

class ThreadSafeFormatter{
    public static ThreadLocal<SimpleDateFormat> dateFormatThreadLocal = new ThreadLocal<SimpleDateFormat>(){
        @Override
        protected SimpleDateFormat initialValue(){
            return new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        }
    };

    public static ThreadLocal<SimpleDateFormat> dateFormatThreadLocal2 = ThreadLocal.withInitial(()->new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));

}
```

### （2）场景2

- 每个线程内需要保存**全局变量**（例如在拦截器中获取用户信息），**可以让不同方法直接使用，避免参数传递的麻烦。**
- 一个比较繁琐的解决方案是把user作为参数层层传递，从service-1()传到service-2()，再传到service-3()...这样会导致**代码冗余且不易维护。**

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210723112659128.png" alt="image-20210723112659128" style="zoom:50%;" />

- 为此可以用ThreadLocal保存一些业务内容（用户信息等），**这些信息在同一个线程内相同，但不同的线程使用的业务内容不同**。

- 在此基础上可以演进，使用UserMap

  <img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210723113104357.png" alt="image-20210723113104357" style="zoom:50%;" />

- 当多线程同时工作时，我们需要保证线程安全，可以用synchronized，也可以用ConcurrentHashMap，但是都会**影响性能**

  <img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210723113225848.png" alt="image-20210723113225848" style="zoom:50%;" />

- 更好的方法是使用ThreadLocal，这样无需synchronized，不会影响性能，不需要层层传递参数，就可以**满足保存当前线程对应的用户信息的需求**。
- **在线程周期内**，都通过这个静态ThreadLocal实例的get()方法取得自己set过的对象，**避免了将对象作为参数传递的麻烦**。

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210723113513148.png" alt="image-20210723113513148" style="zoom:50%;" />

```java
/**
 * 演示ThreadLocal的用法2：避免传递参数的麻烦
 */
public class ThreadLocalNormalUsage06 {
    public static void main(String[] args) {
        new Service1().process();
    }
}

class Service1 {
    public void process() {
        User user = new User("Uranus");
        UserContextHolder.holder.set(user);
        new Service2().process();
    }
}

class Service2 {
    public void process() {
        User user = UserContextHolder.holder.get();
        System.out.println("Service2:" + user.name);
        new Service3().process();
    }
}

class Service3 {
    public void process() {
        User user = UserContextHolder.holder.get();
        System.out.println("Service3:" + user.name);
    }
}

class UserContextHolder {
    public static ThreadLocal<User> holder = new ThreadLocal<>();

}

class User {
    String name;

    public User(String name) {
        this.name = name;
    }
}
```

## 2.2个作用与4个好处

### （1）作用

1. 让某个需要用到的对象在**线程间隔离（每个线程都有自己的独立的对象）**
2. 任何方法中都可以**轻松获得**该对象

### （2）好处

- 达到**线程安全**
- **无需加锁，提高效率**
- 更高效地**利用内存、节省开销**
- **免去传参**的繁琐

## 3.ThreadLocal原理

​		每个Thread对象中都持有一个ThreadLocalMap对象，且可以拥有多个ThreadLocal。

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210723174206991.png" alt="image-20210723174206991" style="zoom: 67%;" />

## 4.主要方法

### （1）T initialValue()

​		初始化。

- 该方法会返回当前线程对应的“初始值”，这是一个**延迟加载**的方法，只有在**调用get时才会触发**。

- 当线程第一次使用get方法访问变量时，将调用此方法。除非线程先前调用了set方法，在这种情况下，不会为本线程调用initialValue方法。

  这正对应了ThreadLocal的两种经典用法

- 通常**每个线程最多调用一次此方法**，但如果在**调用了remove()后再调用get()，则可以再次调用此方法**。

  场景1中的例子里，后面的线程去调用get()时都不会调用initialValue方法，只是去获取对象而非创建，所以最终只有10份。

- 若不重写该方法，最终会返回null。一般使用匿名内部类的方法**重写initialValue**方法，以便在后续使用中可以初始化副本对象。

### （2）void set(T t)

​		为这个线程设置一个新值。

### （3）T get()

​		得到这个线程对应的value。如果是首次调用get()，则会调用initialValue来得到这个值。

### （4）void remove()

​		删除对应这个线程的值。

```java
/**
 * 演示ThreadLocal的get,set,remove，initialValue方法
 */
public class ThreadLocalNormalUsage07 {
    public static void main(String[] args) {
        new Service01().process();
    }
}

class Service01 {
    public void process() {
        User1 user1 = new User1("Uranus!");
        UserContextHolder1.holder.set(user1);
        new Service02().process();
    }
}

class Service02 {
    public void process() {
        User1 user1 = UserContextHolder1.holder.get();
        System.out.println("Service2:" + user1.name);
        UserContextHolder1.holder.remove();
        user1 = new User1("Uranus?");
        UserContextHolder1.holder.set(user1);
        new Service03().process();
    }
}

class Service03 {
    public void process() {
        User1 user1 = UserContextHolder1.holder.get();
        System.out.println("Service3:" + user1.name);
    }
}

class UserContextHolder1 {
    public static ThreadLocal<User1> holder = new ThreadLocal<>();

}

class User1 {
    String name;

    public User1(String name) {
        this.name = name;
    }
}

Service2:Uranus!
Service3:Uranus?
```

## 5.源码分析

### （1）get()

​		get方法先取出当前线程的**ThreadLocalMap**,然后调用**map.getEntry**方法，把本ThreadLocal的引用**作为参数传入**，取出map中属于本ThreadLocal的value。

```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocal.ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocal.ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}

private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocal.ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}
```

### （2）set()

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocal.ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

> ThreadLocalMap以及map中的key和value是存放在Thread中的，而不是ThreadLocal中。

### （3）initialValue()

​		没有默认实现，如果我们要用initialValue方法，需要自己实现，通常是匿名内部类方式。

```
protected T initialValue() {
    return null;
}
```

### （4）remove()

```java
public void remove() {
    ThreadLocal.ThreadLocalMap m = getMap(Thread.currentThread());
    if (m != null)
        m.remove(this);
}
```

> 只会删掉调用remove()的ThreadLocal的记录，而不会删掉整个THreadLocalMap的记录。

## 6.ThreadLocalMap

​		ThreadLocalMap**是ThreadLocal类中的静态类**，是Thread的成员变量，也就是Thread.threadLocals

```java
ThreadLocal.ThreadLocalMap threadLocals = null;
```

​		ThreadLocalMap里面最重要的一个键值对数组Entry[] table,可以认为是一个map，键值对：

- **键：这个ThreadLocal本身**
- **值：实际需要的成员变量**，比如user或者SimpleDateFormat对象

> ThreadLocalMap这里采用的是线性探测法，也就是如果发生冲突，**继续找下一个空位置**，而不是用链表拉链。

## 7.注意点

### （1）内存泄漏

- 定义：某个对象不再有用，但是占用的内存无法被回收。

- Key的泄漏：ThreadLocalMap中的Entry继承自**WeakReference**，是**弱引用**。

  弱引用的特点是，如果这个对象**只被弱引用关联**，那么**这个对象就可以被回收。**

  所以弱引用不会阻止GC，因为这个弱引用的机制。

- Value的泄漏：ThreadLocalMap的每个Entry都是一个对**key的弱引用**，同时每个Entry都包含了一个对**value的强引用**。

  正常情况下，线程终止后，**保存在ThreadLocal里的value会被垃圾回收**，因为没有任何强引用了。

  但是如果线程不终止（比如线程池中线程需要保持很久，**线程池实质上是一直在用同一个线程执行任务**），那么key对应的value就不能被回收，因为有以下的调用链：

  > Thread→ThreadLocalMap→Entry(key为null)→Value

  Value和Thread之间还存在这个强引用链路，所以**导致value无法回收**，就可能出现OOM。

  JDK已经考虑了这个问题，所以在set,remove,rehash方法中会**扫描key为null的Entry，并把对应的value设置为null**，这样value对象就可以被回收。

  但是如果一个ThreadLocal不被使用，以上方法也不会被调用，同时线程又不停止，那么调用链就一直存在，就会导致Value的内存泄漏。

```java
public WeakReference(T referent) {
    super(referent);
}

static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);//继承自WeakReference，弱引用
        value = v;//强引用
    }
}

```

- 如何避免内存泄漏（阿里规约）：调用**remove**方法，就会删除对应的Entry对象，可以避免内存泄漏。所以**使用完ThreadLocal后，应该调用remove方法。**

### （2）空指针异常

```java
/**
 * 演示ThreadLocal空指针异常
 */
public class ThreadLocalNPE {
    ThreadLocal<Long> longThreadLocal = new ThreadLocal<Long>();

    public void set() {
        longThreadLocal.set(Thread.currentThread().getId());
    }

    public Long get() {
        return longThreadLocal.get();
    }

    //未set就get，会报空指针异常，这是因为long会装箱拆箱，尝试把null转化成基本类型，而Long不会
    /*public long get() {
        return longThreadLocal.get();
    }*/

    public static void main(String[] args) {
        ThreadLocalNPE threadLocalNPE = new ThreadLocalNPE();
        System.out.println(threadLocalNPE.get());

        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                threadLocalNPE.set();
                System.out.println(threadLocalNPE.get());
            }
        });
        thread.start();
    }
}
```

### （3）共享对象

​		如果在每个线程中ThreadLocal.set()进去的对象本来就是**多线程共享**的，比如static对象，**那么多个线程的ThreadLocal.get()取得的还是这个共享对象本身**，还是存在并发访问问题。

​		而且如果真的需要共享对象，比如static对象，也根本用不到ThreadLocal，直接使用即可。

### （4）不要强行使用

​		如果可以不使用ThreadLocal就解决问题，那么不要强行使用。

- 例如在任务数很少的时候，在局部变量中新建对象就可以解决问题，那么就不需要使用到ThreadLocal

# 三、Java锁

## 1.Lock

### （1）简介、地位与作用

- 锁是一种工具，用于控制对**共享资源**的访问
- Lock和synchronized，这两个是最常见的锁，它们都可以达到**线程安全**的目的，但是在使用上和功能上又有较大不同
- **Lock并不是用来代替synchronized的**，而是当使用synchronized不合适或不足以满足要求时，**来提供高级功能**
- Lock接口最常见的实现类是ReentrantLock
- 通常情况下，Lock只允许一个线程访问共享资源。不过有时候一些特殊的实现也可允许**并发访问**，比如ReadWriteLock里面的**ReadLock**。

> 为什么需要Lock？为什么synchronized不够用？
>
> - 效率低：锁释放的情况少（要么执行完毕，要么抛出异常），试图获得锁时**不能设定超时、不能中断一个正在试图获得锁的线程。**
> - 不过灵活：加锁和释放的时机单一，每个锁仅有单一的条件（某个对象），可能是不够用的
> - 无法知道是否**成功获取到锁**

### （2）Lock主要方法介绍

​		在Lock中声明了4个方法获取锁：

- lock()

  - 最普通的获取锁，如果锁已被其他线程获取，则进行等待。

  - lock不会像synchronized一样在**异常时自动释放锁**。

  - 因此最佳实践是，在**finally中释放锁**，以保证发生异常时锁一定被释放。

  - lock()方法不能被中断，这会带来很大的隐患：一旦陷入死锁，lock()就会陷入永久等待。

    ```java
    /**
     * lock不会像synchronized一样在异常时自动释放锁
     * 因此最佳实践是，在finally中释放锁，以保证发生异常时锁一定被释放。
     */
    public class MustUnlock {
        private static Lock lock = new ReentrantLock();
    
        public static void main(String[] args) {
            lock.lock();
            try {
                //获取本锁保护的资源
                System.out.println(Thread.currentThread().getName() + "开始执行任务");
            } finally {
                lock.unlock();
                System.out.println(Thread.currentThread().getName() + "已释放锁");
            }
        }
    }
    
    main开始执行任务
    main已释放锁
    ```

- tryLock():

  - 用来**尝试获取锁**，如果当前锁没有被其他线程占用，则获取成功，返回true，否则返回false。
  - 相比于lock，这样的方法显然功能更为强大，我们可以**根据能否获取到锁决定后续程序的行为**。
  - 该方法会**立即返回**，即使拿不到锁时也不会一直等待。

- tryLock(long time,TimeUnit unit):等待一段时间，**超时后放弃**。

  ```java
  /**
   * 用tryLock避免死锁
   */
  public class TryLockDeadLock implements Runnable {
  
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
              if (flag == 1) {
                  try {
                      if (lock1.tryLock(800, TimeUnit.MILLISECONDS)) {
                          //加上try/finally保证获取到的lock1一定会被释放
                          try {
                              System.out.println("线程1获取到锁1");
                              Thread.sleep(new Random().nextInt(1000));
                              if (lock2.tryLock(800, TimeUnit.MILLISECONDS)) {
                                  try {
                                      System.out.println("线程1获取到锁2");
                                      break;
                                  } finally {
                                      lock2.unlock();
                                  }
                              } else {
                                  System.out.println("线程1获取锁2失败，已重试");
                              }
                          } finally {
                              lock1.unlock();
                              Thread.sleep(new Random().nextInt(1000));
                          }
                      } else {
                          System.out.println("线程1获取锁1失败，已重试");
                      }
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
  
              if (flag == 0) {
                  try {
                      if (lock2.tryLock(800, TimeUnit.MILLISECONDS)) {
                          try {
                              System.out.println("线程2获取到锁1");
                              Thread.sleep(new Random().nextInt(1000));
                              if (lock1.tryLock(800, TimeUnit.MILLISECONDS)) {
                                  try {
                                      System.out.println("线程2获取到锁2");
                                      break;
                                  } finally {
                                      lock1.unlock();
                                  }
                              } else {
                                  System.out.println("线程2获取锁2失败，已重试");
                              }
                          } finally {
                              lock2.unlock();
                              Thread.sleep(new Random().nextInt(1000));
                          }
                      } else {
                          System.out.println("线程2获取锁1失败，已重试");
                      }
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
          }
      }
  }
  
  线程1获取到锁1
  线程2获取到锁1
  线程1获取锁2失败，已重试
  线程2获取到锁2
  线程1获取到锁1
  线程1获取到锁2
  ```

- lockInterruptibly()：相当于把tryLock(long time,TimeUnit unit)的超时时间设置为无限。**等待锁的过程中，线程可以被中断。**

  ```java
  /**
   * 演示lockInterruptibly()方法
   */
  public class LockInterruptibly implements Runnable{
      private Lock lock = new ReentrantLock();
  
      public static void main(String[] args) {
          LockInterruptibly lockInterruptibly = new LockInterruptibly();
          Thread thread0 = new Thread(lockInterruptibly);
          Thread thread1 = new Thread(lockInterruptibly);
          thread0.start();
          thread1.start();
  
          try {
              Thread.sleep(1000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
          thread0.interrupt();
          //thread1.interrupt();
      }
  
      @Override
      public void run() {
          System.out.println(Thread.currentThread().getName()+"尝试获取锁");
          try {
              lock.lockInterruptibly();
              try {
                  System.out.println(Thread.currentThread().getName()+"获取到了锁");
                  Thread.sleep(3000);
              }catch (InterruptedException e){
                  System.out.println(Thread.currentThread().getName()+"睡眠期间被中断");
              } finally {
                  lock.unlock();
                  System.out.println(Thread.currentThread().getName()+"释放了锁");
              }
          } catch (InterruptedException e) {
              System.out.println(Thread.currentThread().getName()+"等锁期间被中断");
          }
      }
  }
  
  无打断情况：
  Thread-0尝试获取锁
  Thread-1尝试获取锁
  Thread-0获取到了锁
  Thread-0释放了锁
  Thread-1获取到了锁
  Thread-1释放了锁
  
  thread0.interrupt()：
  Thread-0尝试获取锁
  Thread-1尝试获取锁
  Thread-0获取到了锁
  Thread-0睡眠期间被中断
  Thread-0释放了锁
  Thread-1获取到了锁
  Thread-1释放了锁
      
  thread1.interrupt()：
  Thread-0尝试获取锁
  Thread-1尝试获取锁
  Thread-0获取到了锁
  Thread-1等锁期间被中断
  Thread-0释放了锁
  ```

### （3）可见性保证

​		Lock的加解锁和synchronized有同样的**内存语义**，也就是说，

## 2.锁的分类

- 这些分类是从各种不同角度出发去看的。

- 这些分类并不是互斥的，多种类型可以并存。

  <img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210725173743622.png" alt="image-20210725173743622" style="zoom: 80%;" />

### （1）乐观锁和悲观锁

- 悲观锁又称互斥同步锁，乐观锁又称非互斥同步锁
- 互斥同步锁的劣势：
  - 阻塞和唤醒带来的性能劣势
  - 可能陷入**永久阻塞**
  - **优先级翻转**：优先级低的线程拿到锁之后，即使优先级很高的线程来获取锁，只要优先级低的线程不释放锁，优先级高的线程就拿不到锁。

#### ①悲观锁（互斥同步锁）

- **每次获取并修改数据时，把数据锁住**，防止别的线程争抢。
- Java中悲观锁的实现就是synchronized和Lock相关类。
- 数据库中的悲观锁比如`select for update`

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210725181805298.png" alt="image-20210725181805298" style="zoom: 50%;" />

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210725181819764.png" alt="image-20210725181819764" style="zoom:50%;" />

#### ②乐观锁（非互斥同步锁）

- 认为自己处理操作时不会有其他线程抢占资源，所以**不会锁住被操作对象**。
- 在更新时，去对比在该线程修改的期间有没有被其他线程改变过：如果**没被改变过**，则正常修改数据；如果被改变过，则不能进行更新数据过程，可以放弃、报错、重试等。
- 乐观锁的典型例子就是**原子类、并发容器等，git和数据库中用version控制也是**。
- 乐观锁的实现一般是**利用CAS算法**。

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210725191423493.png" alt="image-20210725191423493" style="zoom:50%;" />

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210725191431247.png" alt="image-20210725191431247" style="zoom:50%;" />

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210725191437500.png" alt="image-20210725191437500" style="zoom:50%;" />

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210725191443781.png" alt="image-20210725191443781" style="zoom:50%;" />

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210725191449864.png" alt="image-20210725191449864" style="zoom:50%;" />

#### ③开销对比

- 悲观锁的原始开销高于乐观锁，但是**一劳永逸**，就算临界区持锁时间越来越长，也不会对互斥锁的开销造成影响。
- 相反，乐观锁一开始开销比悲观锁小，但如果自旋很长时间或者不停重试，**那么消耗的资源也会越来越多**。

#### ④适用场景

- 悲观锁：**适合并发写入多的情况，适合临界区持锁时间较长的情况**，悲观锁可以避免大量的无用自旋等消耗
- 乐观锁：**适合并发写入少，大部分是读取的场景**，不加锁能让读取性能大幅提高。

### （2）可重入锁和非可重入锁

#### ①演示预定电影院座位

```java
/**
 * 演示多线程预定电影院座位
 */
public class CinemaBookSeat {
    private static ReentrantLock lock = new ReentrantLock();

    private static void bookSeat() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "开始预定座位");
            Thread.sleep(1000);
            System.out.println(Thread.currentThread().getName() + "完成预定座位");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        new Thread(()->bookSeat()).start();
        new Thread(()->bookSeat()).start();
        new Thread(()->bookSeat()).start();
        new Thread(()->bookSeat()).start();
    }
}
```

#### ②演示打印字符串

```java
/**
 * 演示reentrantLock被打断的情况
 */
public class LockDemo {

    public static void main(String[] args) {
        new LockDemo().init();
    }

    private void init(){
        final Outputer outputer = new Outputer();
        new Thread(new Runnable() {
            @Override
            public void run() {
                while (true){
                    try {
                        Thread.sleep(5);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    outputer.output("ok");
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                while (true){
                    try {
                        Thread.sleep(5);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    outputer.output("uranus");
                }
            }
        }).start();
    }

    static class Outputer{
        Lock lock = new ReentrantLock();

        public void output(String name){
            int len = name.length();
            lock.lock();
            try {
                for (int i = 0; i < len; i++) {
                    System.out.print(name.charAt(i));
                }
                System.out.println("");
            }finally {
                lock.unlock();
            }
        }
    }
}

使用锁：
ok
ok
uranus
uranus
ok
uranus
    
不使用锁：
uranus
uraok
nus
okuranus
```

#### ③可重入性质

- 定义：指的是以线程为单位，**当一个线程获取对象锁之后，这个线程可以再次获取本对象上的锁，而其他的线程是不可以的**。
- 优点：**避免死锁，提升封装性**。

​		演示可重入锁：

```java
/**
 * 演示可重入锁基本用法
 */
public class GetHoldCount {
    private static ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) {
        System.out.println(lock.getHoldCount());
        lock.lock();
        System.out.println(lock.getHoldCount());
        lock.lock();
        System.out.println(lock.getHoldCount());
        lock.lock();
        System.out.println(lock.getHoldCount());
        lock.unlock();
        System.out.println(lock.getHoldCount());
        lock.unlock();
        System.out.println(lock.getHoldCount());
        lock.unlock();
        System.out.println(lock.getHoldCount());
    }
}

0
1
2
3
2
1
0
```

​		演示递归获取可重入锁：

```java
/**
 * 演示递归获取可重入锁
 */
public class RecursionDemo {

    private static ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) {
        accessResource();
    }

    private static void accessResource(){
        lock.lock();
        System.out.println("已经对资源进行处理");
        try{
            if(lock.getHoldCount()<5){
                System.out.println(lock.getHoldCount());
                accessResource();
                System.out.println(lock.getHoldCount());
            }
        }finally {
            lock.unlock();
        }
    }
}

已经对资源进行处理
1
已经对资源进行处理
2
已经对资源进行处理
3
已经对资源进行处理
4
已经对资源进行处理
4
3
2
1
```

#### ④可重入原理

- 实现原理实现是通过**为每个锁关联一个请求计数器和一个占有它的线程**。当**计数为0时，认为锁是未被占有的**；线程请求一个未被占有的锁时，JVM将记录锁的占有者，并且将请求计数器置为1 。
- 如果同一个线程再次请求这个锁，计数器将递增；
- 每次占用线程退出同步块，计数器值将递减。**直到计数器为0,锁被释放**。

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210725224509080.png" alt="image-20210725224509080" style="zoom:67%;" />

​		PS：acquires通常为1.

#### ⑤其他方法

- isHeldByCurrentThread可以判断锁是否被当前线程持有
- getQueueLength可以返回当前正在等待这把锁的队列有多长

### （3）公平锁和非公平锁

- 公平的定义：公平指的是按照线程请求的顺序分配锁，非公平指的是不完全按照请求的顺序分配锁，**并非乱序**，在一定情况下可以插队。

  非公平不提倡插队，只是在合适的时机可以插队。

- 非公平锁的出现可以**避免唤醒带来的空档期，可以提高效率**。

#### ①公平的情况（以ReentrantLock为例）

​		ReentrantLock本身是非公平锁，如果在创建ReentrantLock对象时，参数填写为true，那么这就是公平锁。

​		假设线程1234按序调用lock()：

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210725233237459.png" alt="image-20210725233237459" style="zoom:50%;" />

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210725233257582.png" alt="image-20210725233257582" style="zoom:50%;" />

#### ②不公平的情况（以ReentrantLock为例）

​		如果在线程1释放锁时，线程5恰好去执行lock()：

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210725233327909.png" alt="image-20210725233327909" style="zoom:50%;" />

#### ③代码演示

```java
/**
 * 演示公平锁与不公平锁
 */
public class FairLock {

    public static void main(String[] args) throws InterruptedException {
        PrintQueue printQueue = new PrintQueue();
        Thread thread[] = new Thread[10];

        for (int i = 0; i < 10; i++) {
            thread[i] = new Thread(new Job(printQueue));
        }

        for (int i = 0; i < 10; i++) {
            thread[i].start();
            Thread.sleep(100);
        }
    }
}

class Job implements Runnable {
    PrintQueue printQueue;

    public Job(PrintQueue printQueue) {
        this.printQueue = printQueue;
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + "开始打印");
        printQueue.printJob(new Object());
        System.out.println(Thread.currentThread().getName() + "打印完毕");
    }
}

class PrintQueue {
    private Lock queueLock = new ReentrantLock(true);

    public void printJob(Object document) {
        queueLock.lock();
        try {
            int duration = new Random().nextInt(2) + 1;
            System.out.println(Thread.currentThread().getName() + "正在打印，需要" + duration + "秒");
            Thread.sleep(duration * 1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            queueLock.unlock();
        }

        queueLock.lock();
        try {
            int duration = new Random().nextInt(2) + 1;
            System.out.println(Thread.currentThread().getName() + "正在打印，需要" + duration + "秒");
            Thread.sleep(duration * 1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            queueLock.unlock();
        }
    }
}

公平情况：
Thread-0开始打印
Thread-0正在打印，需要2秒
Thread-1开始打印
Thread-2开始打印
Thread-3开始打印
Thread-4开始打印
Thread-5开始打印
Thread-6开始打印
Thread-7开始打印
Thread-8开始打印
Thread-9开始打印
Thread-1正在打印，需要2秒
Thread-2正在打印，需要2秒
Thread-3正在打印，需要2秒
Thread-4正在打印，需要1秒
Thread-5正在打印，需要1秒
Thread-6正在打印，需要1秒
Thread-7正在打印，需要1秒
Thread-8正在打印，需要1秒
Thread-9正在打印，需要2秒
Thread-0正在打印，需要2秒
Thread-0打印完毕
Thread-1正在打印，需要1秒
Thread-1打印完毕
Thread-2正在打印，需要1秒
Thread-2打印完毕
Thread-3正在打印，需要2秒
Thread-3打印完毕
Thread-4正在打印，需要1秒
Thread-4打印完毕
Thread-5正在打印，需要2秒
Thread-5打印完毕
Thread-6正在打印，需要1秒
Thread-6打印完毕
Thread-7正在打印，需要2秒
Thread-7打印完毕
Thread-8正在打印，需要2秒
Thread-8打印完毕
Thread-9正在打印，需要2秒
Thread-9打印完毕

不公平情况：
Thread-0开始打印
Thread-0正在打印，需要2秒
Thread-1开始打印
Thread-2开始打印
Thread-3开始打印
Thread-4开始打印
Thread-5开始打印
Thread-6开始打印
Thread-7开始打印
Thread-8开始打印
Thread-9开始打印
Thread-0正在打印，需要1秒
Thread-0打印完毕
Thread-1正在打印，需要2秒
Thread-1正在打印，需要2秒
Thread-1打印完毕
Thread-2正在打印，需要1秒
Thread-2正在打印，需要1秒
Thread-2打印完毕
Thread-3正在打印，需要2秒
Thread-3正在打印，需要2秒
Thread-3打印完毕
Thread-4正在打印，需要2秒
Thread-4正在打印，需要1秒
Thread-4打印完毕
Thread-5正在打印，需要1秒
Thread-5正在打印，需要2秒
Thread-5打印完毕
Thread-6正在打印，需要1秒
Thread-6正在打印，需要2秒
Thread-6打印完毕
Thread-7正在打印，需要2秒
Thread-7正在打印，需要2秒
Thread-7打印完毕
Thread-8正在打印，需要1秒
Thread-8正在打印，需要2秒
Thread-8打印完毕
Thread-9正在打印，需要1秒
Thread-9正在打印，需要2秒
Thread-9打印完毕
```

#### ④特例

- tryLock()方法不遵守设定的公平的规则。例如**当有线程执行tryLock时，一旦有线程释放锁，那么这个正在tryLock的线程就可以立即获取锁**，即使在它之前已经有其它线程在等待队列里了。

#### ⑤对比与原理

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210726000552261.png" alt="image-20210726000552261" style="zoom:50%;" />

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210726000652224.png" alt="image-20210726000652224" style="zoom:50%;" />

### （3）共享锁和排它锁

- 排它锁，又称独占锁、独享锁、写锁，可读可写，一个线程获取到排它锁后，其它线程不可以继续获取排它锁，**保证了线程安全，比如synchronized.**
- 共享锁，又称为读锁，获取共享锁后，**可以查看但无法修改和删除数据，其他线程此时也可以获取共享锁**，也可以查看但无法修改和删除数据。
- 共享锁和排它锁的典型是**读写锁ReentrantReadWriteLock**，其中读锁是共享锁，写锁是排它锁。
- 读写锁的作用
  - 在没有读写锁之前，我们假设使用ReentrantLock，那么我们虽然保证了线程安全，**但也浪费了一定资源：多个读操作同时进行，并没有线程安全问题。**
  - 在读的地方使用读锁，写的地方使用写锁，灵活控制，如果**没有写锁的情况下，读是无阻塞的**，提高了程序的执行效率。
- 读写锁的规则
  - 多个线程只申请读锁，都可以申请到
  - 如果一个线程已经占用了读锁，则此时如果其他线程要申请写锁，则申请写锁的线程会一直等待读锁释放。
  - 如果一个线程已经占用了写锁，则此时如果其他线程要申请写锁或读锁，则申请的线程会一直等待写锁释放。
  - 总结：**要么一个或多个线程同时有读锁，要么一个线程有写锁**，二者不会同时出现。

#### ①演示预定与查看电影院座位

```java
/**
 * 演示读写锁基本用法
 */
public class CinemaReadWrite {
    private static ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock();

    private static ReentrantReadWriteLock.ReadLock readLock = reentrantReadWriteLock.readLock();
    private static ReentrantReadWriteLock.WriteLock writeLock = reentrantReadWriteLock.writeLock();

    public static void main(String[] args) {
        new Thread(()->read(),"Thread1").start();
        new Thread(() -> read(), "Thread2").start();
        new Thread(() -> write(), "Thread3").start();
        new Thread(() -> write(), "Thread4").start();

    }

    private static void read() {
        readLock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "得到读锁，正在读取");
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println(Thread.currentThread().getName() + "释放读锁");
            readLock.unlock();
        }
    }

    private static void write() {
        writeLock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "得到写锁，正在写入");
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println(Thread.currentThread().getName() + "释放写锁");
            writeLock.unlock();
        }
    }
}
```

#### ②读锁插队策略

- 公平锁：不允许插队

- 非公平：假设线程2和线程4正在同时读取，线程3想要写入，拿不到锁，于是进入等待队列，线程5不在队列里，现在过来想要读取，此时有两种策略。

  - 策略1（不适用）：读可以插队，效率高，但是**容易造成饥饿**。

    <img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210726123235551.png" alt="image-20210726123235551" style="zoom:67%;" />

  - 策略2（适用）：避免饥饿。

    <img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210726123712049.png" alt="image-20210726123712049" style="zoom:67%;" />

  - **总结：写锁可以随时插队；读锁仅在等待队列头结点不是想获取写锁的线程时可以插队。**

```java
static final class FairSync extends ReentrantReadWriteLock.Sync {
    private static final long serialVersionUID = -2274990926593161451L;
    final boolean writerShouldBlock() {
        return hasQueuedPredecessors();
    }
    final boolean readerShouldBlock() {
        return hasQueuedPredecessors();
    }
}

static final class NonfairSync extends ReentrantReadWriteLock.Sync {
    private static final long serialVersionUID = -8159625535654395037L;
    final boolean writerShouldBlock() {
        return false; // writers can always barge
    }
    final boolean readerShouldBlock() {
        return apparentlyFirstQueuedIsExclusive();
    }
}
```

#### ③代码演示读锁不插队

```java
/**
 * 演示非公平情况下读锁不插队
 */
public class NonfairBargeDemo {
    private static ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock(false);

    private static ReentrantReadWriteLock.ReadLock readLock = reentrantReadWriteLock.readLock();
    private static ReentrantReadWriteLock.WriteLock writeLock = reentrantReadWriteLock.writeLock();

    public static void main(String[] args) {
        new Thread(() -> write(), "Thread1").start();
        new Thread(() -> read(), "Thread2").start();
        new Thread(() -> read(), "Thread3").start();
        new Thread(() -> write(), "Thread4").start();
        new Thread(() -> read(), "Thread5").start();
    }

    private static void read() {
        System.out.println(Thread.currentThread().getName() + "开始尝试获取读锁");
        readLock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "得到读锁，正在读取");
            Thread.sleep(20);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println(Thread.currentThread().getName() + "释放读锁");
            readLock.unlock();
        }
    }

    private static void write() {
        System.out.println(Thread.currentThread().getName() + "开始尝试获取写锁");
        writeLock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "得到写锁，正在写入");
            Thread.sleep(40);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println(Thread.currentThread().getName() + "释放写锁");
            writeLock.unlock();
        }
    }
}

Thread1开始尝试获取写锁
Thread2开始尝试获取读锁
Thread1得到写锁，正在写入
Thread3开始尝试获取读锁
Thread4开始尝试获取写锁
Thread5开始尝试获取读锁
Thread1释放写锁
Thread2得到读锁，正在读取
Thread3得到读锁，正在读取
Thread3释放读锁
Thread2释放读锁
Thread4得到写锁，正在写入
Thread4释放写锁
Thread5得到读锁，正在读取
Thread5释放读锁
```

#### ④代码演示**读锁实际上可以插队**

```java
/**
 * 演示非公平情况下读锁实际上可以插队
 */
public class NonfairBargeDemo {
    private static ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock(false);

    private static ReentrantReadWriteLock.ReadLock readLock = reentrantReadWriteLock.readLock();
    private static ReentrantReadWriteLock.WriteLock writeLock = reentrantReadWriteLock.writeLock();

    public static void main(String[] args) {
        new Thread(() -> write(), "Thread1").start();
        new Thread(() -> read(), "Thread2").start();
        new Thread(() -> read(), "Thread3").start();
        new Thread(() -> write(), "Thread4").start();
        new Thread(() -> read(), "Thread5").start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                Thread thread[] = new Thread[1000];
                for (int i = 0; i < 1000; i++) {
                    thread[i] = new Thread(() -> read(), "子线程创建的Thread" + i);
                }
                for (int i = 0; i < 1000; i++) {
                    thread[i].start();
                }
            }
        }).start();

    }

    private static void read() {
        System.out.println(Thread.currentThread().getName() + "开始尝试获取读锁");
        readLock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "得到读锁，正在读取");
            Thread.sleep(20);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println(Thread.currentThread().getName() + "释放读锁");
            readLock.unlock();
        }
    }

    private static void write() {
        System.out.println(Thread.currentThread().getName() + "开始尝试获取写锁");
        writeLock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "得到写锁，正在写入");
            Thread.sleep(40);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println(Thread.currentThread().getName() + "释放写锁");
            writeLock.unlock();
        }
    }
}

Thread2得到读锁，正在读取
子线程创建的Thread927得到读锁，正在读取
Thread3得到读锁，正在读取
```

​		对比公平锁：

```java
/**
 * 演示公平情况下的运行结果以作对比
 */
public class NonfairBargeDemo {
    private static ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock(true);

    private static ReentrantReadWriteLock.ReadLock readLock = reentrantReadWriteLock.readLock();
    private static ReentrantReadWriteLock.WriteLock writeLock = reentrantReadWriteLock.writeLock();

    public static void main(String[] args) {
        new Thread(() -> write(), "Thread1").start();
        new Thread(() -> read(), "Thread2").start();
        new Thread(() -> read(), "Thread3").start();
        new Thread(() -> write(), "Thread4").start();
        new Thread(() -> read(), "Thread5").start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                Thread thread[] = new Thread[1000];
                for (int i = 0; i < 1000; i++) {
                    thread[i] = new Thread(() -> read(), "子线程创建的Thread" + i);
                }
                for (int i = 0; i < 1000; i++) {
                    thread[i].start();
                }
            }
        }).start();

    }

    private static void read() {
        System.out.println(Thread.currentThread().getName() + "开始尝试获取读锁");
        readLock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "得到读锁，正在读取");
            Thread.sleep(20);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println(Thread.currentThread().getName() + "释放读锁");
            readLock.unlock();
        }
    }

    private static void write() {
        System.out.println(Thread.currentThread().getName() + "开始尝试获取写锁");
        writeLock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "得到写锁，正在写入");
            Thread.sleep(40);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println(Thread.currentThread().getName() + "释放写锁");
            writeLock.unlock();
        }
    }
}

Thread1释放写锁
子线程创建的Thread863开始尝试获取读锁
子线程创建的Thread864开始尝试获取读锁
子线程创建的Thread865开始尝试获取读锁
Thread2得到读锁，正在读取
子线程创建的Thread866开始尝试获取读锁
子线程创建的Thread867开始尝试获取读锁
Thread3得到读锁，正在读取
子线程创建的Thread999开始尝试获取读锁
子线程创建的Thread998开始尝试获取读锁
Thread3释放读锁
Thread2释放读锁
Thread4得到写锁，正在写入
Thread4释放写锁
Thread5得到读锁，正在读取
子线程创建的Thread0得到读锁，正在读取
子线程创建的Thread1得到读锁，正在读取
```

#### ⑤锁的升降级

​		支持锁的降级，不支持升级。

```java
/**
 * 演示锁的升降级
 */
public class Upgrading {
    private static ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock();

    private static ReentrantReadWriteLock.ReadLock readLock = reentrantReadWriteLock.readLock();
    private static ReentrantReadWriteLock.WriteLock writeLock = reentrantReadWriteLock.writeLock();

    public static void main(String[] args) throws InterruptedException {
        System.out.println("演示降级是可以的");
        Thread thread1 = new Thread(()->writeDowngrading(),"Thread1");
        thread1.start();
        thread1.join();

        System.out.println("------------------");
        System.out.println("演示升级是不可以的");
        Thread thread2 = new Thread(()->readUpgrading(),"Thread2");
        thread2.start();
    }

    private static void readUpgrading() {
        readLock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "得到读锁，正在读取");
            Thread.sleep(1000);
            System.out.println(Thread.currentThread().getName() + "升级会带来阻塞");
            writeLock.lock();
            System.out.println(Thread.currentThread().getName() + "获取到了写锁，升级成功");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println(Thread.currentThread().getName() + "释放读锁");
            readLock.unlock();
        }
    }

    private static void writeDowngrading() {
        writeLock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "得到写锁，正在写入");
            Thread.sleep(1000);
            readLock.lock();
            System.out.println(Thread.currentThread().getName()+"在不释放写锁的情况下直接获取读锁，降级成功");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println(Thread.currentThread().getName()+"释放读锁");
            readLock.unlock();
            System.out.println(Thread.currentThread().getName() + "释放写锁");
            writeLock.unlock();
        }
    }
}
```

- 不支持锁的升级
  - 若有两个线程A，B都是读锁，此时A想升级，但此时B仍在读，产生冲突；
  - 若有两个线程A，B都是读锁，A、B均想升级，会产生冲突。
  - 如果能设计一种锁，保证每次**只有一个线程升级**且其他的线程都必须释放，那么也是可以实现锁的升级的。
  - **ReentrantLock不支持锁的升级。**

### （4）自旋锁和阻塞锁

#### ①定义

- **阻塞或唤醒一个Java线程需要操作系统切换CPU状态来完成**，这种状态转换需要耗费处理器时间。
- 如果同步代码块中的内容过于简单，**状态切换消耗的时间有可能比用户代码执行的时间还要长。**
- 许多场景中，同步资源的锁定时间很短，**为了这一小段时间去切换线程，线程挂起和恢复现场的花费可能会让系统得不偿失。**
- AtomicInteger的实现：**自旋锁的实现原理是CAS**，AtomicInteger中调用unsafe进行自增操作的源码中的do-while循环就是一个自旋操作。如果修改过程中遇到其他线程竞争导致没修改成功，就**在while里死循环，直至修改成功**。

#### ②实现自己的自旋锁

```java
/**
 * 实现自己的自旋锁
 */
public class SpinLock {
    private AtomicReference<Thread> sign = new AtomicReference<>();

    public static void main(String[] args) {
        SpinLock spinLock = new SpinLock();
        Runnable r = new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + "开始尝试获取自旋锁");
                spinLock.lock();
                System.out.println(Thread.currentThread().getName() + "成功获取自旋锁");
                try {
                    Thread.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println(Thread.currentThread().getName() + "释放自旋锁");
                    spinLock.unlock();
                }
            }
        };
        Thread thread1 = new Thread(r);
        Thread thread2 = new Thread(r);
        thread1.start();
        thread2.start();
    }

    public void lock() {
        Thread current = Thread.currentThread();
        while (!sign.compareAndSet(null, current)) {
            System.out.println(Thread.currentThread().getName() + "自旋获取失败，再次尝试");
        }
    }

    public void unlock() {
        Thread current = Thread.currentThread();
        sign.compareAndSet(current, null);
    }
}

Thread-0开始尝试获取自旋锁
Thread-1开始尝试获取自旋锁
Thread-0成功获取自旋锁
Thread-1自旋获取失败，再次尝试
Thread-1自旋获取失败，再次尝试
Thread-0释放自旋锁
Thread-1自旋获取失败，再次尝试
Thread-1成功获取自旋锁
Thread-1释放自旋锁
```

#### ③适用场景

- 自旋锁一般用于多核的服务器，在**并发度不是特别高**的情况下，比阻塞锁的效率高。
- 自旋锁适用于临界区比较短小的情况，否则**如果临界区很大**（线程一旦拿到锁，需要很长时间才能释放），那也是不合适的。

### （5）可中断锁

- Java中的synchronized就不是可中断锁，而Lock是可中断锁，因为tryLock(time)和lockInterruptibly都能响应中断。
- 如果某一线程A正在执行锁中的代码，另一线程B正在等待获取该锁，可能由于等待时间过长，线程B不想等待了，想先处理其他事情，我们就可以中断它，这就是可中断锁。

### （6）锁优化

#### ①JVM对锁的优化

- 自旋锁和自适应（自旋一定时间后仍失败的话，进入阻塞状态）
- 锁消除
- 锁的粒度粗化

#### ②优化锁和提高并发性能

- 缩小同步代码块
- 尽量不要锁住方法
- 减少锁的次数
- 避免人为制造大量使用锁的场景
- 锁中尽量不要再包含锁
- 选择合适的锁类型或合适的工具类

# 四、原子类

- 定义：不可分割，即一个操作是**不可中断**的，即便是多线程的情况下也可以保证
- java.util.concurrent.**atomic**
- 原子类的作用与锁类似，是为了保证**并发情况下线程安全**。不过原子类相比锁具有一定的优势：
  - **粒度更细：原子变量可以把竞争范围缩小到变量级别**，这是我们可以获得的最细粒度的情况。
  - 效率更高：通常，**使用原子类的效率会比使用锁的效率更高**，**除了高度竞争**的情况。

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210727182001103.png" alt="image-20210727182001103" style="zoom:50%;" />

## 1.AtomicInteger

### （1）常用方法

- public final int get():获取当前值
- public final int getAndSet(int newValue):获取当前值，并设置新的值
- public final int getAndIncrement():获取当前的值，并自增1
- public final int getAndDecrement():获取当前的值，并自减1
- public final int getAndAdd(int delta):获取当前的值，并加上预期的值
- boolean **compareAndSet**(int expect,int update):如果当前的值等于预期值，则以原子方式将该值设置为输入值update

### （2）代码演示

```java
/**
 * 演示AtomicInteger的基本用法，对比非原子类的线程安全问题，使用了原子类后，不需要加锁也可以保证线程安全
 */
public class AtomicIntegerDemo1 implements Runnable {
    private static final AtomicInteger atomicInteger = new AtomicInteger();
    private static volatile int basicCount = 0;

    public static void main(String[] args) throws InterruptedException {
        AtomicIntegerDemo1 r = new AtomicIntegerDemo1();
        Thread t1 = new Thread(r);
        Thread t2 = new Thread(r);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println("原子类的结果：" + atomicInteger.get());
        System.out.println("非原子类的结果：" + basicCount);
    }

    public void incrementAtomic() {
        atomicInteger.getAndIncrement();
    }

    /*public synchronized void incrementBasic() {
        basicCount++;
    }*/
    
    public void incrementBasic() {
        basicCount++;
    }
    
    

    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            incrementAtomic();
            incrementBasic();
        }
    }
}

原子类的结果：20000
非原子类的结果：18491
```

## 2.AtomicArray

```java
/**
 * 演示原子数组的使用方法
 */
public class AtomicArrayDemo {
    public static void main(String[] args) throws InterruptedException {
        AtomicIntegerArray atomicIntegerArray = new AtomicIntegerArray(1000);
        Thread[] threadsIncrementer = new Thread[100];
        Thread[] threadsDecrementer = new Thread[100];
        Incrementer incrementer = new Incrementer(atomicIntegerArray);
        Decrementer decrementer = new Decrementer(atomicIntegerArray);

        for (int i = 0; i < 100; i++) {
            threadsIncrementer[i] = new Thread(incrementer);
            threadsDecrementer[i] = new Thread(decrementer);
            threadsIncrementer[i].start();
            threadsDecrementer[i].start();
        }

        for (int i = 0; i < 100; i++) {
            threadsIncrementer[i].join();
            threadsDecrementer[i].join();
        }

        for (int i = 0; i < atomicIntegerArray.length(); i++) {
            if (atomicIntegerArray.get(i) != 0) {
                System.out.println("发现非0值" + i);
            }
        }
        System.out.println("运行结束");
    }
}

class Incrementer implements Runnable {
    private AtomicIntegerArray array;

    public Incrementer(AtomicIntegerArray array) {
        this.array = array;
    }

    @Override
    public void run() {
        for (int i = 0; i < array.length(); i++) {
            array.getAndIncrement(i);
        }
    }
}


class Decrementer implements Runnable {
    private AtomicIntegerArray array;

    public Decrementer(AtomicIntegerArray array) {
        this.array = array;
    }

    @Override
    public void run() {
        for (int i = 0; i < array.length(); i++) {
            array.getAndDecrement(i);
        }
    }
}

运行结束
```

## 3.AtomicReference

​		AtomicReference类的作用**和AtomicInteger并没有本质区别**，AtomicReference可以**让一个对象保证原子性**。当然，AtomicReference的功能更强，因为一个对象里可以包含很多属性。

## 4.AtomicIntegerFieldUpdater

- 把普通变量升级为具有原子功能的变量
- 使用场景：**偶尔**需要一个原子get-set操作

```java
/**
 * 演示AtomicIntegerFieldUpdaterDemo的用法
 */
public class AtomicIntegerFieldUpdaterDemo implements Runnable {
    static Candidate tom;
    static Candidate peter;
    public static AtomicIntegerFieldUpdater<Candidate> scoreUpdater
            = AtomicIntegerFieldUpdater.newUpdater(Candidate.class, "score");

    public static void main(String[] args) throws InterruptedException {
        tom = new Candidate();
        peter = new Candidate();
        AtomicIntegerFieldUpdaterDemo r = new AtomicIntegerFieldUpdaterDemo();
        Thread t1 = new Thread(r);
        Thread t2 = new Thread(r);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println("普通变量" + peter.score);
        System.out.println("升级变量" + tom.score);
    }

    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            peter.score++;
            scoreUpdater.getAndIncrement(tom);
        }
    }

    public static class Candidate {
        volatile int score;
        //volatile static int score;
    }
}

普通变量19766
升级变量20000
```

> 不支持静态变量的升级

## 5.Adder累加器

- 是Java8引入的新类。
- 高并发下**LongAdder比AtomicLong效率高**，不过**本质是空间换时间**。
- 竞争激烈的时候，LongAdder把不同线程对应到不同Cell上进行修改，降低了冲突的概率，是**多段锁**的概念，提高了并发性。
- 这里演示多线程情况下AtomicLong的性能，有16个线程对同一个AtomicLong累加。
- AtomicLong由于竞争很激烈，**每一次加法，都要flush和refresh**，导致狠耗费资源。

### （1）代码演示

​		AtomicLongDemo：

```java
/**
 * 演示高并发场景下，LongAdder比AtomicLong性能好
 */
public class AtomicLongDemo {
    public static void main(String[] args) {
        AtomicLong counter = new AtomicLong(0);
        ExecutorService service = Executors.newFixedThreadPool(16);
        long start = System.currentTimeMillis();

        for (int i = 0; i < 10000; i++) {
            service.submit(new Task(counter));
        }
        service.shutdown();
        while ((!service.isTerminated())) {

        }
        long end = System.currentTimeMillis();

        System.out.println("AtomicLong耗时：" + (end - start) / 1000.0 + "s");
        System.out.println(counter.get());

    }

    private static class Task implements Runnable {
        private AtomicLong counter;

        public Task(AtomicLong counter) {
            this.counter = counter;
        }

        @Override
        public void run() {
            for (int i = 0; i < 10000; i++) {
                counter.incrementAndGet();
            }
        }
    }
}

AtomicLong耗时：1.54s
100000000
```

​		LongAdderDemo：

```java
/**
 * 演示高并发场景下，LongAdder比AtomicLong性能好
 */
public class LongAdderDemo {
    public static void main(String[] args) {
        LongAdder counter = new LongAdder();
        ExecutorService service = Executors.newFixedThreadPool(16);
        long start = System.currentTimeMillis();

        for (int i = 0; i < 10000; i++) {
            service.submit(new LongAdderDemo.Task(counter));
        }
        service.shutdown();
        while ((!service.isTerminated())) {

        }
        long end = System.currentTimeMillis();

        System.out.println("LongAdder耗时：" + (end - start) / 1000.0 + "s");
        System.out.println(counter.sum());

    }

    private static class Task implements Runnable {
        private LongAdder counter;

        public Task(LongAdder counter) {
            this.counter = counter;
        }

        @Override
        public void run() {
            for (int i = 0; i < 10000; i++) {
                counter.increment();
            }
        }
    }
}

LongAdder耗时：0.122s
100000000
```

### （2）LongAdder带来的改进与原理

- 在内部，LongAdder与AtomicLong的实现原理不同

  - AtomicLong：每一次加法都需要做同步，所以高并发时会导致冲突比较多，降低效率。

  - LongAdder：**每个线程会有一个自己的计数器，仅用来在自己线程内计数**，这样就不会和其他线程的计数器干扰。

    在加和的过程中，**无需同步，也不需要flush和refresh**。

- LongAdder引入了分段累加的概念，内部有一个base变量和一个Cell[]数组共同参与计数。

  - base变量：竞争不激烈，直接累加到该变量上。
  - Cell[]数组：竞争激烈，各个线程分散累加到自己的槽Cell[i]中。

- sum源码

  ```java
  public long sum() {
      Cell[] as = cells;
      Cell a;
      long sum = base;
      if (as != null) {
          for (int i = 0; i < as.length; ++i) {
              if ((a = as[i]) != null)
                  sum += a.value;
          }
      }
      return sum;
  }
  ```

### （3）对比

- 低竞争下，二者具有相似的特征。高竞争下，LongAdder的预期吞吐量要高得多，但要消耗更多的空间。
- LongAdder适合的场景是统计求和计数的场景，而且LongAdder基本只提供了add方法，而AtomicLong还具有CAS方法。

## 6.Accumulator累加器

​		Accumulator和Adder非常相似，其本质是一个**更通用版本**的Adder。

```java
/**
 * 演示LongAccumulator的用法
 */
public class LongAccumulatorDemo {
    public static void main(String[] args) {
        LongAccumulator accumulator = new LongAccumulator((x,y)-> Math.max(x,y),0);//x
        ExecutorService executorService = Executors.newFixedThreadPool(8);
        IntStream.range(1,10).forEach(i->executorService.submit(()->accumulator.accumulate(i)));//并行计算

        executorService.shutdown();
        while (!executorService.isTerminated()){

        }
        System.out.println(accumulator.getThenReset());
    }
}

9
```

​		使用场景：

- 需要大量计算且**需要并行计算**；
- **不关注计算顺序**。

# 五、CAS

- 一定是并发场景
- 认为**V的值应该是A，若是的话就把它改成B，如果不是A（说明被别人修改过了），那么就不进行修改**，避免多人同时修改导致出错。
- CAS有三个操作数：内存值V，预期值A，要修改的值B，**当且仅当内存值V与预期值A相同时，才将内存值修改为B**，否则什么都不做，最后返回现在的V值。
- CPU的**特殊指令，CPU保证其原子性**

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210728012011159.png" alt="image-20210728012011159" style="zoom:50%;" />

## 1.CAS等价代码

```java
/**
 * 模拟CAS操作，等价代码
 */
public class SimulatedCAS {
    private volatile int value;

    public synchronized int compareAndSwap(int expectedValue, int newValue) {
        int oldValue = value;
        if (oldValue == expectedValue) {
            value = newValue;
        }
        return oldValue;
    }
}
```

## 2.案例演示

​		两个线程竞争，其中一个落败。

```java
/**
 * 两个线程竞争，其中一个落败
 */
public class TwoThreadsCompetition implements Runnable{
    private volatile int value;

    public static void main(String[] args) throws InterruptedException {
        TwoThreadsCompetition r = new TwoThreadsCompetition();
        r.value=0;
        Thread t1 = new Thread(r,"Thread-1");
        Thread t2 = new Thread(r,"Thread-2");
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(r.value);
    }

    public synchronized int compareAndSwap(int expectedValue, int newValue) {
        int oldValue = value;
        if (oldValue == expectedValue) {
            value = newValue;
        }
        return oldValue;
    }

    @Override
    public void run() {
        compareAndSwap(0,1);
    }
}
```

## 3.应用场景

- 乐观锁
- 并发容器
- 数据库版本号
- 原子类

### （1）分析Java中如何利用CAS实现原子操作

- AtomicInteger加载**Unsafe**工具，用来**直接操作内存数据**

- 用Unsafe来实现底层操作

- 用volatile修饰value字段，保证可见性

  ```java
  static {
      try {
          valueOffset = unsafe.objectFieldOffset
              (AtomicInteger.class.getDeclaredField("value"));
      } catch (Exception ex) { throw new Error(ex); }
  }
  
  private volatile int value;
  ```

- getAndAddInt方法分析

  ```java
  public final int getAndAdd(int delta) {
      return unsafe.getAndAddInt(this, valueOffset, delta);
  }
  
  public final int getAndAddInt(Object var1, long var2, int var4) {
      int var5;
      do {
          var5 = this.getIntVolatile(var1, var2);
      } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
  
      return var5;
  }
  
  public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
  ```

### （2）Unsafe类

- Unsafe是CAS的核心类。Java无法直接访问底层操作系统，而是通过native方法访问。尽管如此，JVM还是开了后门，JDK中有一个Unsafe类，它提供了**硬件级别的原子操作**。

- valueOffset表示的是变量值在内存中的偏移地址，因为Unsafe就是根据内存偏移地址获取数据原值的，这样我们就可以通过Unsafe实现CAS。

  ```java
  valueOffset = unsafe.objectFieldOffset//详见上文代码
  ```

## 4.缺点

- ABA问题：可用版本号改善。
- **自旋时间过长**

# 六、Final

## 1.不变性

- 如果对象在**被创建后，状态就不能被修改**，那么它就是不可变的。
- **具有不变性的对象一定是线程安全的**，我们不需要对其采取任何额外的安全措施，也能保证线程安全。

## 2.作用

- **防止类被继承，方法被重写，变量被修改**。
- 天生是**线程安全**的，不需要额外的同步开销。

## 3.用法

### （1）修饰变量

​		被final修饰的对象，值不能被更改。**如果变量是对象，那么对象的引用不能变，但是对象自身的内容依然可以变化**。

- final instance variable(类中的final属性)

  1. 在声明变量的**等号右边**直接赋值
  2. **构造函数**中复制
  3. 类的**初始代码块**中赋值（不常用）

  如果不采用第一种赋值方法，就必须在第2、3中挑一个来赋值，而不能不赋值，这是final语法规定的。

  ```java
  /**
   * 演示final变量
   */
  public class FinalVariableDemo {
      private final int a;
  
      //private final int a = 6;
      
      /*public FinalVariableDemo(int a) {
          this.a = a;
      }*/
      
      {
          a=7;
      }
  }
  ```

- final static variable(类中的static final属性)

  1. 在声明变量的**等号右边**直接赋值
  2. **static初始代码块赋值**，不能用普通的初始代码块

  ```java
  /**
   * 演示final变量
   */
  public class FinalVariableDemo {
      private static final int a;
  
      //private static final int a = 5;
      
      static {
          a = 5;
      }
  }
  ```

- final local variable(方法中的final属性)

  变量是在方法里的，不存在构造函数，也不存在初始代码块。它**不规定赋值时机**，只要求在**使用前必须赋值**，这和方法中的非final变量的要求是一致的。

### （2）修饰方法

- **构造方法**不允许final修饰。
- **不可被重写**，即使子类有同样名字的方法，也不是重写。
- 引申：**static方法不能被重写**。

```java
public class FinalMethodDemo {
    public void drink() {

    }

    public final void eat() {

    }

    public static void sleep() {

    }
}

class SubClass extends FinalMethodDemo {

    @Override
    public void drink() {
        super.drink();
    }

    //不允许
    /*public final void eat(){
        
    }*/

    public static void sleep() {

    }
}
```

### （3）修饰类

- 不可被继承。
- 例如典型的**String类**就是final的。

## 4.对象不变性

### （1）变量声明在类中

- 不变性并不意味着，简单地用final修饰就是不可变

  - 对于**基本数据类型**，被final修饰后就具有不变性
  - 对于**对象类型**，需该对象保证自身被创建后，状态永远不会变才可以

- 如何利用final实现对象不可变

  - **把所有属性都声明为final的想法是错误的**

  - 一个属性是对象类型的不可变对象的**正确例子**

    ```java
    /**
     * 有一个对象属性，但是整体不可变，其他类无法修改set里面的数据
     */
    public class ImmutableDemo {
        private final Set<String> students = new HashSet<>();
    
        public ImmutableDemo(){
            students.add("a");
            students.add("b");
            students.add("c");
        }
    
        public boolean isStudent(String name){
            return students.contains(name);
        }
    }
    ```

- 满足以下条件，对象才是不可变的：

  - 对象**创建后，其状态就不能修改**
  - **所有属性都是final**修饰的
  - 对象创建过程中**没有发生逸出**

### （2）变量声明在方法中——栈封闭

​		在方法里新建的局部变量，实际上是存储在每个线程私有的栈空间，每个栈的空间是不能被其他线程所访问到的，所以不会有线程安全问题。

```java
/**
 * 演示栈封闭的两种情况：基本变量和对象
 * 先演示线程争抢带来错误结果，然后把变量放到方法内，就会产生正确结果
 */
public class StackConfinement implements Runnable {
    int index = 0;

    public static void main(String[] args) throws InterruptedException {
        StackConfinement r = new StackConfinement();
        Thread thread1 = new Thread(r);
        Thread thread2 = new Thread(r);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println(r.index);
    }

    public void inThread() {
        int neverGoOut = 0;
        for (int i = 0; i < 10000; i++) {
            neverGoOut++;
        }
        System.out.println("栈内保护的数字是线程安全的：" + neverGoOut);
    }

    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            index++;
        }
        inThread();
    }
}
```

## 5.变量存放区域

```java
public class FinalStringDemo {
    public static void main(String[] args) {
        String a = "Uranus1";//常量池
        final String b = "Uranus";//一开始就可以确定b的值，无需编译时确定
        String c = "Uranus";//运行时才确定
        String d = b + 1;//常量池
        String e = c + 1;//运行时才确定，堆
        System.out.println((a == d));
        System.out.println((a == e));
        System.out.println(a.equals(d));
        System.out.println(a.equals(e));
    }
}

true
false
true
true
```

```java
public class FinalStringDemo2 {
    public static void main(String[] args) {
        String a = "Uranus1";
        final String b = get();//编译器无法确认b的值，不会进行优化，所以c是在运行中生成的
        String c = b + 1;//运行时才确定，堆
        System.out.println((a == c));
    }

    private static String get() {
        return "Uranus";
    }
}

false
```

# 七、并发容器

## 1.概览

- ConcurrentHashMap：线程安全的HashMap
- CopyOnWriteArrayList：线程安全的List
- BlockingQueue：这是一个接口，表示阻塞队列，非常适合用于作为数据共享的通道
- ConcurrentLinkedQueue：搞笑的非阻塞并发队列，使用链表实现，可以看作一个线程安全的LinkedList
- ConcurrentSkipListMap：是一个Map，使用跳表的数据结构进行快速查找

## 2.演进过程

- Vector和Hashtable

- HashMap和ArrayList

  虽然这两个类不是线程安全的，但是可以用

  - Collections.synchronizedList(new ArrayList<E>())
  - Collections.synchronizedMap(new HashMap<K,V>())

  使之变成线程安全的。

- ConcurrentHashMap和CopyOnWriteArrayList

  - 取代同步的HashMap与ArrayList
  - 绝大多数情况下，ConcurrentHashMap和CopyOnWriteArrayList的性能都要更好

## 3.ConcurrentHashMap

### （1）Map

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210728192154400.png" alt="image-20210728192154400" style="zoom:50%;" />

- HashMap：具体见[Java容器](Markdown/Java容器)
- Hashtable
- LinkedHashMap：**可以保持键值对的插入顺序**，顺序为插入顺序或者最近最少使用（LRU）顺序。
- TreeMap：可排序

```java
/**
 * 演示Map的基本用法
 */
public class MapDemo {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();
        System.out.println(map.isEmpty());//true
        map.put("Uranus", 16);
        map.put("gyz", 26);
        System.out.println(map.keySet());//[Uranus, gyz]
        System.out.println(map.values());//[16, 26]
        System.out.println(map.get("Uranus"));//16
        System.out.println(map.get("1"));//null
        System.out.println(map.size());//2
        System.out.println(map.containsKey("Uranus"));//true
        map.remove("Uranus");
        System.out.println(map.containsKey("Uranus"));//false
    }
}
```

> 1. 为什么不用Collections.synchronizedMap()?
>
>    因为这是通过一个锁保证不同线程之间的并发访问，但是synchronized在并发量高的时候性能并不理想。
>
> 2. 为什么HashMap是线程不安全的？
>
>    - 同时put**碰撞**导致数据丢失
>    - 同时put**扩容**导致数据丢失
>    - **死循环造成的CPU100%**——主要存在与Java7中，这是因为**多线程同时扩容时，会造成链表的死循环，即互相指向对方**。

### （2）HashMap1.7和1.8的结构图

#### ①结构

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210728201136925.png" alt="image-20210728201136925" style="zoom:50%;" />

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210728201204063.png" alt="image-20210728201204063" style="zoom:50%;" />

#### ②关于并发的特点

- **非线程安全**
- 迭代时不允许修改内容
- 只读的并发是安全的
- 如果一定要把HashMap用在并发环境，用Collections.synchronizedMap(new HashMap());

### （3）ConcurrentHashMap1.7和1.8的结构图

<img src="C:\Users\Ursnus\AppData\Roaming\Typora\typora-user-images\image-20210728203146493.png" alt="image-20210728203146493" style="zoom:50%;" />

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210728203103501.png" alt="image-20210728203103501" style="zoom:50%;" />

- ConcurrnetHashMap 由很多个**分段锁（ Segment） 组合，每个分段锁维护着几个桶（HashEntry）**，多个线程可以同时访问不同分段锁上的桶，从而使其并发度更高（并发度就是 Segment 的个数），这是因为**每个Segment独立上ReentrantLock锁**。
- 每一个 Segment 是一个类似于 HashMap 的结构，所以**每一个Segment的内部可以进行扩容**，依然是**数组和链表组成的拉链法**。
-  Segment 的个数一旦**初始化就不能改变**，默认 Segment 的个数是 **16 个**，你也可以认为 ConcurrentHashMap **默认支持最多 16 个线程并发**。

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210728203656188.png" alt="image-20210728203656188" style="zoom:50%;" />

- **JDK 1.8 使用了 CAS 操作来支持更高的并发度，在 CAS 操作失败时使用内置锁 synchronized。**结构也由 Java7 中的 **Segment 数组 + HashEntry 数组 + 链表** 进化成了 **Node 数组 + 链表 / 红黑树**，Node 是类似于一个 HashEntry 的结构。它的冲突再达到一定大小时会转化成红黑树，在冲突小于一定数量时又退回链表。并且 JDK 1.8 的实现也在链表过长时会转换为红黑树。

> ​		为什么链表长度超过8时要转为红黑树？
>
> - 首先默认为链表而不是红黑树，是因为链表占用的内存更少。
>
> - 其次链表想要达到长度为8也是很难的，这也是为了避免极端情况。
>
>   以下为相应链表长度的概率：
>
> ```java
> 0:    0.60653066
> 1:    0.30326533
> 2:    0.07581633
> 3:    0.01263606
> 4:    0.00157952
> 5:    0.00015795
> 6:    0.00001316
> 7:    0.00000094
> 8:    0.00000006
> ```

### （4）ConcurrentHashMap也会发生线程安全问题

```java
/**
 * 组合操作并不保证线程安全
 */
public class OptionsNotSafe implements Runnable {
    private static ConcurrentHashMap<String, Integer> scores = new ConcurrentHashMap<String, Integer>();

    public static void main(String[] args) throws InterruptedException {
        scores.put("Uranus", 0);
        Thread t1 = new Thread(new OptionsNotSafe());
        Thread t2 = new Thread(new OptionsNotSafe());
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(scores);
    }

    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            Integer score = scores.get("Uranus");
            int newScore = score + 1;
            scores.put("Uranus", newScore);
        }
    }
}

{Uranus=11100}
```

​		ConcurrentHashMap提供了**replace方法**保证更换值的原子性：

```java
/**
 * 组合操作并不保证线程安全
 */
public class OptionsNotSafe implements Runnable {
    private static ConcurrentHashMap<String, Integer> scores = new ConcurrentHashMap<String, Integer>();

    public static void main(String[] args) throws InterruptedException {
        scores.put("Uranus", 0);
        Thread t1 = new Thread(new OptionsNotSafe());
        Thread t2 = new Thread(new OptionsNotSafe());
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(scores);
    }

    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            boolean b = false;
            while (!b){
                Integer score = scores.get("Uranus");
                int newScore = score + 1;
                b = scores.replace("Uranus",score,newScore);
            }
        }
    }
}

{Uranus=20000}
```

## 4.CopyOnWriteArrayList

### （1）概览

- 代替Vector和SynchronizedList，就和ConcurrentHashMap代替SynchronizedMap原因一样
- Vector和SynchronizedList的锁粒度太大，**并发效率相对比较低**，并且**迭代时无法编辑**。
- Copy-On-Write并发容器还包括CopyOnWriteArraySet，用来替代同步Set
- 使用场景
  - **读操作可以尽可能地快**，而**写即使慢一些**也没有太大关系
  - **读多写少**：黑名单，每日更新；监听器，迭代操作远多于修改操作。

### （2）读写规则

- 回顾读写锁：读读共享，其他均互斥。
- 读写锁规则的升级：**读取是完全不用加锁的，且写入也不会阻塞读取操作**，只有**写入和写入**间需要进行同步等待，这是因为**add方法使用了ReentrantLock**.

```java
/**
 * 演示CopyOnWriteArrayList可以在迭代过程中修改数组内容，但是ArrayList不可以
 */
public class CopyOnWriteArrayListDemo1 {
    public static void main(String[] args) {
        //ArrayList<String> list = new ArrayList<>();//会报错
        CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();

        list.add("1");
        list.add("2");
        list.add("3");
        list.add("4");
        list.add("5");

        Iterator<String> iterator = list.iterator();
        while (iterator.hasNext()) {
            System.out.println("list is" + list);
            String next = iterator.next();
            System.out.println(next);

            if (next.equals("2")) {
                list.remove("5");
            }
            if (next.equals("3")) {
                list.add("6");
            }
        }
    }
}
```

### （3）实现原理

- **写操作在一个复制的数组**上进行，读操作还是在**原始数组**中进行，**读写分离，互不影响**。
- **写操作需要加锁，防止并发写入时导致写入数据丢失**。
- 写操作结束之后需要把**原始数组指向新的复制数组**。

### （4）缺点

- **数据一致性**问题：CopyOnWrite容器**只能保证最终数据一致性，不能保证数据实时一致性**。
- **内存占用**问题：CopyOnWrite的写是**复制机制**，所以**进行写操作时，内存里会同时驻扎两个对象的内存**。

## 5.并发队列

- 用队列可以**在线程间**传递数据：生产者消费者模式、银行转账。
- 考虑锁等线程安全问题的重任从“我们”转移到了“队列”上。

### （1）并发队列简介

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210729002756096.png" alt="image-20210729002756096" style="zoom: 67%;" />

### （2）阻塞队列BlockingQueue

- 阻塞队列是具有阻塞功能的队列。通常，阻塞队列的一端是给生产者放数据用，另一端给消费者拿数据用。阻塞队列是**线程安全**的，所以生产者和消费者都可以是多线程的。

  <img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210729003311376.png" alt="image-20210729003311376" style="zoom:50%;" />

- 阻塞功能：最有特色的两个带有阻塞功能的方法如下：

  - take()方法：获取并移除队列的头结点，一旦**执行take时，队列里无数据，则阻塞**，直到队列里有数据；

  - put()方法：插入元素，但如果队列已满，就因为无法继续插入而阻塞，直到队列有空闲空间。

    <img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210729003530890.png" alt="image-20210729003530890" style="zoom:50%;" />

    <img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210729003548866.png" alt="image-20210729003548866" style="zoom:50%;" />

- 是否有界：这是一个非常重要的属性，无界队列意味着里面可以容纳非常多（Integer.MAX_VALUE,约为$2^{31}$，可以近似认为是无界容量。

- 阻塞队列和**线程池**的关系：阻塞队列是线程池的重要组成部分。

### （3）BlockingQueue主要方法

- put,take：若队列满或队列空，会阻塞。
- add,remove,element：add和remove在队列满或队列空时**不会阻塞，而是抛出异常**；element返回队列头元素，若队列为空，也是**抛出异常**。
- offer,poll,peek：offer在队列满时会返回bool值false；poll在**队列不为空时会取出元素并删除，队列为空时会返回null**；peek和poll的区别就在于**取出元素后不会删除。**

### （4）ArrayBlockingQueue

- 有界：创建时需要指定容量；
- 公平：可以指定是否需要保证公平，如果想保证公平的话，那么等待了最长时间的线程会被优先处理，不会这同时会带来一定的性能损耗。

```java
public class ArrayBlockingQueueDemo {


    public static void main(String[] args) {
        final ArrayBlockingQueue<String> queue = new ArrayBlockingQueue<>(3);
        Interviewer r1 = new Interviewer(queue);
        HR r2 = new HR(queue);
        new Thread(r1).start();
        new Thread(r2).start();
    }
}

class Interviewer implements Runnable {
    BlockingQueue<String> queue;

    public Interviewer(BlockingQueue queue) {
        this.queue = queue;
    }

    @Override
    public void run() {
        System.out.println("10个候选人已到齐");
        for (int i = 0; i < 10; i++) {
            String candidate = "Candidate" + i;
            try {
                queue.put(candidate);
                Thread.sleep(1);//延缓打印，防止队列满时仍打印安排完毕
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(candidate + "已经安排完毕");
        }
        try {
            queue.put("stop");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class HR implements Runnable {
    BlockingQueue<String> queue;

    public HR(BlockingQueue queue) {
        this.queue = queue;
    }

    @Override
    public void run() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        String msg;
        try {
            while (!(msg = queue.take()).equals("stop")) {
                System.out.println("目前轮到" + msg);
                Thread.sleep(1);//延缓打印，防止还未安排就打印轮到XX
            }
            System.out.println("流程已结束");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

10个候选人已到齐
Candidate0已经安排完毕
Candidate1已经安排完毕
Candidate2已经安排完毕
目前轮到Candidate0
Candidate3已经安排完毕
目前轮到Candidate1
目前轮到Candidate2
Candidate4已经安排完毕
Candidate5已经安排完毕
目前轮到Candidate3
目前轮到Candidate4
Candidate6已经安排完毕
目前轮到Candidate5
Candidate7已经安排完毕
目前轮到Candidate6
Candidate8已经安排完毕
目前轮到Candidate7
Candidate9已经安排完毕
目前轮到Candidate8
目前轮到Candidate9
流程已结束
```

### （5）LinkedBlockingQueue

- 无界：容量为Integer.MAX_VALUE
- 内部结构：Node，两把锁

### （6）PriorityBlockingQueue

- 支持优先级
- 自然顺序（不是先进先出，可以自己制定compareTo排序规则）
- 无界队列（扩容）
- PriorityQueue的线程安全版本

### （7）SynchronousQueue

- **容量为0**而不是1，因为它不需要持有元素，所要做的只有直接传递。
- 效率很高。
- 没有peek等函数，也没有iterate相关方法。
- 是一个极好的用来直接直接传递的并发数据结构。
- SynchronousQueue是线程池**Executors.newCachedThreadPool()使用的阻塞队列**。

### （8）DelayQueue

- 延迟队列，根据延迟时间排序
- 无界队列
- 元素需要实现Delayed接口，规定排序规则

### （9）非阻塞并发队列

- 并发包中的非阻塞队列只有ConcurrentLinkedQueue，使用**链表**作为其数据结构，**使用CAS非阻塞算法实现线程安全**，适合用在对**性能要求较高**的并发场景。

# 八、控制并发流程

## 1.概览

- 控制并发流程的工具类，作用就是帮助我们更容易地让线程之间合作，让线程之间相互配合，以此满足业务逻辑
- 比如**让线程A等待线程B执行完毕后再执行**等合作策略

<img src="C:\Users\Ursnus\AppData\Roaming\Typora\typora-user-images\image-20210729153320911.png" alt="image-20210729153320911" style="zoom:50%;" />

## 2.CountDownLatch

### （1）流程

​		**倒数结束之前，一直处于等待状态，直到倒计时结束，此线程才继续工作**。

### （2）主要方法

- CountDownLatch(int count):仅有一个构造函数，参数count为需要倒数的数值。
- await():调用await()方法的线程会被挂起，一直等待，直到count为0才继续执行。
- countDown():将count值减1，直到为0时，等待的线程会被唤起。

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210729154106836.png" alt="image-20210729154106836" style="zoom: 67%;" />

### （3）两个典型用法

#### ①用法1：一个线程等待多个线程都执行完毕，再继续自己的工作

```java
/**
 * 工厂中，质检，5个工人检查，所有人都认为通过时才通过
 */
public class CountDownLatchDemo1 {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(5);
        ExecutorService service = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 5; i++) {
            final int no = i + 1;
            Runnable r = new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep((long) (Math.random() * 10000));
                        System.out.println("No." + no + "完成了检查");
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }finally {
                        latch.countDown();
                    }
                }
            };
            service.submit(r);
        }
        System.out.println("等待五人检查完。。。");
        latch.await();
        System.out.println("所有人都完成了工作，进入下一个环节");
        service.shutdown();
    }
}

等待五人检查完。。。
No.3完成了检查
No.1完成了检查
No.2完成了检查
No.5完成了检查
No.4完成了检查
所有人都完成了工作，进入下一个环节
```

#### ②用法2：多个线程等待某一个线程的信号，同时开始执行

```java
/**
 * 模拟100米跑步，5名选手贮备完毕，只等裁判员一声令下，所有人同时开始跑步
 */
public class CountDownLatchDemo2 {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(1);
        ExecutorService service = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 5; i++) {
            final int no = i + 1;
            Runnable r = new Runnable() {
                @Override
                public void run() {
                    System.out.println("No." + no + "准备完毕，等待发令");
                    try {
                        latch.await();
                        System.out.println("No." + no + "开始跑步");
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            };
            service.submit(r);
        }
        //裁判员检查场地
        Thread.sleep(5000);
        System.out.println("发令枪响，比赛开始！");
        latch.countDown();
        service.shutdown();
    }
}

No.2准备完毕，等待发令
No.4准备完毕，等待发令
No.3准备完毕，等待发令
No.1准备完毕，等待发令
No.5准备完毕，等待发令
发令枪响，比赛开始！
No.4开始跑步
No.3开始跑步
No.2开始跑步
No.5开始跑步
No.1开始跑步
```

#### ③用法1和用法2结合

```java
/**
 * 模拟100米跑步，5名选手贮备完毕，只等裁判员一声令下，所有人同时开始跑步.
 * 当所有人都到终点后，比赛结束
 */
public class CountDownLatchDemo1And2 {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch begin = new CountDownLatch(1);
        CountDownLatch end = new CountDownLatch(5);
        ExecutorService service = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 5; i++) {
            final int no = i + 1;
            Runnable r = new Runnable() {
                @Override
                public void run() {
                    System.out.println("No." + no + "准备完毕，等待发令");
                    try {
                        begin.await();
                        System.out.println("No." + no + "开始跑步");
                        Thread.sleep((long)(Math.random()*10000));
                        System.out.println("No." + no + "跑到终点了");
                        end.countDown();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            };
            service.submit(r);
        }
        //裁判员检查场地
        Thread.sleep(5000);
        System.out.println("\n发令枪响，比赛开始！\n");
        begin.countDown();

        end.await();
        System.out.println("\n比赛结束！");
        service.shutdown();
    }
}

No.2准备完毕，等待发令
No.4准备完毕，等待发令
No.1准备完毕，等待发令
No.3准备完毕，等待发令
No.5准备完毕，等待发令

发令枪响，比赛开始！

No.2开始跑步
No.4开始跑步
No.3开始跑步
No.1开始跑步
No.5开始跑步
No.1跑到终点了
No.4跑到终点了
No.5跑到终点了
No.2跑到终点了
No.3跑到终点了

比赛结束！
```

#### ④注意事项

- 可以多等多
- CountDownLatch不能重用。如果需要重新计算，可以创建新的CountDownLatch实例或使用CyclicBarrier.

## 3.Semaphore信号量

### （1）概览

- Semaphore可以用来**限制或管理数量有限的资源**的使用情况。
- 信号量的作用是维护一个“许可证”的计数，线程可以“获取”许可证，那信号量剩余的许可证就减一，线程也可以“释放”一个许可证，那信号量剩余的许可证就加1，当信号量所拥有的许可证数量为0，那么下一个还想要获取许可证的线程就必须等待，直到有另外的线程释放了许可证。

### （2）使用流程

- 初始化Semaphore并**指定许可证的数量**。
- 想执行一个任务之前要调用**acquire()**或acquireUninterruptibly()方法。
- 任务执行结束后，调用**release()**来释放许可证。

### （3）主要方法

- new Semaphore(int permits,boolean fair)：这里可以设置是否使用**公平**策略，如果传入true，那么Semaphore会把之前等待的线程放到FIFO队列里，以便于有了新的许可证后，可用分发给之前等待时间最长的线程。
- acquire()：申请许可证，可被打断。
- acquireUninterruptibly()：申请许可证，不可被打断。
- tryAcquire()：看看目前是否有空闲的许可证，有的话就去获取，没有的话也不会陷入阻塞。
- tryAcquire(timeout)：和tryAcquire()一样，但是多了一个超时时间。
- release()：释放许可证。

```java
/**
 * 演示Semaphore用法
 */
public class SemaphoreDemo {
    static Semaphore semaphore = new Semaphore(3, true);

    public static void main(String[] args) {
        ExecutorService service = Executors.newFixedThreadPool(50);
        for (int i = 0; i < 100; i++) {
            service.submit(new Task());
        }
        service.shutdown();
    }

    static class Task implements Runnable {

        @Override
        public void run() {
            try {
                semaphore.acquire();//一次拿到1个许可证
				//semaphore.acquire(3);//一次拿到3个许可证
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "获取到许可证");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }finally {
                System.out.println(Thread.currentThread().getName() + "释放了许可证");
                semaphore.release();//一次释放1个许可证
                //semaphore.release(3);//一次释放3个许可证
            }
        }
    }
}
```

> 1. 获取和释放的数量必须一致
> 2. 注意初始化Semaphore时设置**公平性**，一般设置为true更合理
> 3. 并不是必须有获取许可证的线程释放那个许可证，**获取和释放许可证对线程并无要求，可以是A获取，由B释放**。
> 4. 信号量的作用，除了控制临界区最多同时有N个线程访问外，另一个作用是实现“条件等待”，例如线程1需要在线程2完成准备工作后才能开始工作，那么就由线程1acquire()，线程2完成任务后release()，相当于是**轻量级的CountDownLatch**。

## 4.Condition条件对象

### （1）作用

- 线程1需要等待某个条件时，它就去执行condition.await()方法，一旦执行了该方法，线程就会进入阻塞状态。
- 此时线程2去执行对应的条件，直到这个条件达成的时候，线程2就去执行**condition.signal()**方法，这时JVM就会从被阻塞的线程里找那些等待该condition的线程。线程1收到可执行信号时，它的线程状态就会变成**Runnable**可执行状态。

- signalAll()会唤起所有正在等待的线程。
- signal()是公平的，只会唤起那个等待时间最长的线程。

### （2）基本用法

```java
/**
 * 演示Condition的基本用法
 */
public class ConditionDemo1 {
    private ReentrantLock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    public static void main(String[] args) throws InterruptedException {
        ConditionDemo1 conditionDemo1 = new ConditionDemo1();
        //conditionDemo1.method1();//会阻塞
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                    conditionDemo1.method2();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
        conditionDemo1.method1();
    }

    void method1() throws InterruptedException {
        lock.lock();
        try {
            System.out.println("条件不满足，开始等待");
            condition.await();
            System.out.println("条件满足，开始执行");
        }finally {
            lock.unlock();
        }
    }

    void method2(){
        lock.lock();
        try {
            System.out.println("准备工作完成，唤醒其他线程");
            condition.signal();
        }finally {
            lock.unlock();
        }
    }
}

条件不满足，开始等待
准备工作完成，唤醒其他线程
条件满足，开始执行
```

### （3）用Condition实现生产者消费者模式（交替工作）

```java
/**
 * 演示用Condition实现生产者消费者模式
 */
public class ConditionDemo2 {
    private int queueSize = 10;
    private PriorityQueue<Integer> queue = new PriorityQueue<>(queueSize);
    private ReentrantLock lock = new ReentrantLock();
    private Condition notFull = lock.newCondition();
    private Condition notEmpty = lock.newCondition();

    public static void main(String[] args) {
        ConditionDemo2 conditionDemo2 = new ConditionDemo2();
        Producer producer = conditionDemo2.new Producer();
        Consumer consumer = conditionDemo2.new Consumer();
        producer.start();
        consumer.start();
    }

    class Consumer extends Thread {

        @Override
        public void run() {
            consume();
        }

        private void consume() {
            while (true) {
                lock.lock();
                try {
                    while (queue.size() == 0) {
                        System.out.println("队列空，等待数据");
                        try {
                            notEmpty.await();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    queue.poll();
                    notFull.signal();
                    System.out.println("从队列里取走了一个数据，队列剩余" + queue.size() + "个元素");
                } finally {
                    lock.unlock();
                }
            }
        }
    }

    class Producer extends Thread {

        @Override
        public void run() {
            produce();
        }

        private void produce() {
            while (true) {
                lock.lock();
                try {
                    while (queue.size() == queueSize) {
                        System.out.println("队列满，等待空余");
                        try {
                            notFull.await();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    queue.offer(1);
                    notEmpty.signal();
                    System.out.println("从队列里取添加了一个数据，队列剩余" + queue.size() + "个元素");
                } finally {
                    lock.unlock();
                }
            }
        }
    }
}
```

### （4）注意事项

- 如果说Lock用来代替synchronized，那么Condition就是用来代替Object.wait/notify的，所以用法和性质几乎都一样。
- await方法会**自动释放持有的Lock锁**，不需要自己事先手动释放。
- 调用await时**必须持有锁**，否则会抛出异常。

## 5.CyclicBarrier循环栅栏

### （1）概览

- CyclicBarrier循环栅栏和CountDownLatch很类似，都能阻塞一组线程
- 当有大量线程相互配合，分别计算不同任务，且需要最后**统一汇总**时，可以使用CyclicBarrier。它可以构造一个集结点，某一个线程执行完毕，就会到集结点等待，直到所有线程都到了集结点，那么该栅栏就会被撤销，所有线程再统一出发，继续执行剩下的任务。

### （2）代码演示

```java
/**
 * 演示CyclicBarrier的用法
 */
public class CyclicBarrierDemo {
    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(5, new Runnable() {

            @Override
            public void run() {
                System.out.println("这一波人都到场了，大家统一出发！");
            }
        });
        //可重用
        for (int i = 0; i < 10; i++) {
            new Thread(new Task(i, cyclicBarrier)).start();
        }
    }

    static class Task implements Runnable {
        private int id;
        private CyclicBarrier cyclicBarrier;

        public Task(int id) {
            this.id = id;
        }

        public Task(int id, CyclicBarrier cyclicBarrier) {
            this.id = id;
            this.cyclicBarrier = cyclicBarrier;
        }

        @Override
        public void run() {
            System.out.println("线程" + id + "现在前往集合地点");
            try {
                Thread.sleep((long) (Math.random() * 10000));
                System.out.println("线程" + id + "到了集合地点，开始等待其他人到达");
                cyclicBarrier.await();
                System.out.println("线程" + id + "出发了");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }
}

线程0现在前往集合地点
线程3现在前往集合地点
线程2现在前往集合地点
线程1现在前往集合地点
线程5现在前往集合地点
线程4现在前往集合地点
线程6现在前往集合地点
线程7现在前往集合地点
线程8现在前往集合地点
线程9现在前往集合地点
线程1到了集合地点，开始等待其他人到达
线程6到了集合地点，开始等待其他人到达
线程2到了集合地点，开始等待其他人到达
线程8到了集合地点，开始等待其他人到达
线程4到了集合地点，开始等待其他人到达
这一波人都到场了，大家统一出发！
线程4出发了
线程1出发了
线程6出发了
线程2出发了
线程8出发了
线程5到了集合地点，开始等待其他人到达
线程0到了集合地点，开始等待其他人到达
线程7到了集合地点，开始等待其他人到达
线程3到了集合地点，开始等待其他人到达
线程9到了集合地点，开始等待其他人到达
这一波人都到场了，大家统一出发！
线程9出发了
线程5出发了
线程0出发了
线程3出发了
线程7出发了
```

### （3）CyclicBarrier与CountDownLatch区别

- 作用不同：CyclicBarrier要等待固定数量的线程，CountDownLatch只需等待数字为0.

  也就是说，CyclicBarrier**用于线程**，CountDownLatch**用于事件**。

- 可重用性不同：CyclicBarrier可以重用，CountDownLatch不可以。

- CyclicBarrier在构造函数中可以放入Runnable，以便满足条件后立即运行。

# 九、AQS

​		AQS全称为AbstractQueuedSynchronizer。

## 1.Semaphore和AQS的关系

​		Semaphore内部有一个Sync类，Sync类继承了AQS。

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210729201801598.png" alt="image-20210729201801598" style="zoom:50%;" />

## 2.AQS的比喻

- 比喻：群面，单面。
  - Semaphore：一个人面试完后，后一个人才可以进来。
  - CountDownLatch：群面，等待所有人到齐。
  - 这些同步工作类，要做的就只是写下自己的“要人规则”。是来一个面一个，还是凑齐10人一起面试。
  - 剩下的**招呼面试者的脏活累活交给AQS来做**。
- 安排就坐、叫号、先来后到等HR的工作就是AQS做的工作。
- 面试官不会去关心两个面试者是不是号码冲突了，也不会去管面试者是否需要一个地方休息等问题，这些都交给HR去做。
- 如果没有AQS，就需要每个协作工具自己实现：
  - 同步状态的原子性管理
  - 线程的阻塞和解除阻塞
  - 队列的管理

## 3.AQS的作用、重要性与地位

- AQS是一个用于构建锁、同步器、协作工具类的工具类（框架）。有了AQS以后，更多的协作工具类都可以很方便的写出来。
- 一句话总结：**有了AQS，构建线程协作类就容易多了**。

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210729203352419.png" alt="image-20210729203352419" style="zoom:67%;" />

## 4.AQS的三要素

​		AQS最核心的就是三大部分：

- state
- 控制线程抢锁和配合的**FIFO队列**
- 期望协作工具类去实现的**获取/释放**等重要方法

### （1）state状态

​		这里的state的具体含义，会根据具体实现类的不同而不同，比如：

- 在Semaphore里，它表示“**剩余的许可证的数量**”，
- 在CountDownLatch里，它表示“**还需要倒数的数量**”，
- 在ReentrantLock里，state用来表示“锁”的占有情况，包括**可重入计数。**当**state为0时，标识该Lock不被任何线程所占有**。

​		state是**volatile修饰**的，会被并发的修改，所以所有修改state的方法都需要**保证线程安全**，比如getState、setState以及compareAndSetState操作来读取和更新这个状态。这些方法都依赖于**j.u.c.atomic**包的支持。

### （2）FIFO队列

​		这个队列用来**存放“等待的线程”**，AQS就是排队管理器，**AQS会维护一个等待的线程队列，把线程都放到这个队列里**。

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210729205001598.png" alt="image-20210729205001598" style="zoom:50%;" />

### （3）期望协作工具类去实现的**获取/释放**等重要方法

- 这里的获取和释放方法，是利用AQS的协作工具类里最重要的方法，是由协作类自己去实现的，并且**含义各不相同**。

- 获取方法
  - 获取操作会依赖state变量，经常会阻塞；
  - 在Semaphore中，获取就是acquire方法，作用是获取一个许可证；
  - 在CountDownLatch中，获取就是await方法，作用是“等待，直到倒数结束”。
- 释放方法
  - 释放操作不会阻塞
  - 在Semaphore中，释放就是release方法，作用是释放一个许可证；
  - CountDownLatch中，释放就是CountDown方法，作用是倒数一个数。

## 5.用AQS实现一个自己的Latch

​		AQS用法：

1. 写一个类，想好**协作的逻辑**，实现**获取/释放方法**。
2. 内部写一个**Sync类继承AbstractQueuedSynchronizer。**
3. 根据是否独占来**重写tryAcquire/tryRelease或tryAcquireShared(int acquires)和tryReleaseShared(int releases)等方法**，在之前写的获取/释放方法中**调用AQS的acquire/release或Shared方法**

```java
/**
 * 用AQS实现一个自己的Latch
 */
public class OneShotLatch {
    private final Sync sync = new Sync();

    public void signal() {
        sync.releaseShared(0);
    }

    public void await() {
        sync.acquireShared(0);
    }

    private class Sync extends AbstractQueuedSynchronizer {
        @Override
        protected int tryAcquireShared(int arg) {
            return (getState() == 1) ? 1 : -1;
        }

        @Override
        protected boolean tryReleaseShared(int arg) {
            setState(1);
            return true;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        OneShotLatch oneShotLatch = new OneShotLatch();
        for (int i = 0; i < 10; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    System.out.println(Thread.currentThread().getName() + "尝试获取latch，获取失败则等待");
                    oneShotLatch.await();
                    System.out.println(Thread.currentThread().getName() + "继续运行");
                }
            }).start();
        }
        Thread.sleep(5000);
        oneShotLatch.signal();

        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + "尝试获取latch，获取失败则等待");
                oneShotLatch.await();
                System.out.println(Thread.currentThread().getName() + "继续运行");
            }
        }).start();
    }
}

Thread-0尝试获取latch，获取失败则等待
Thread-5尝试获取latch，获取失败则等待
Thread-6尝试获取latch，获取失败则等待
Thread-9尝试获取latch，获取失败则等待
Thread-4尝试获取latch，获取失败则等待
Thread-3尝试获取latch，获取失败则等待
Thread-2尝试获取latch，获取失败则等待
Thread-1尝试获取latch，获取失败则等待
Thread-8尝试获取latch，获取失败则等待
Thread-7尝试获取latch，获取失败则等待
Thread-0继续运行
Thread-4继续运行
Thread-2继续运行
Thread-9继续运行
Thread-6继续运行
Thread-5继续运行
Thread-8继续运行
Thread-7继续运行
Thread-1继续运行
Thread-3继续运行
Thread-10尝试获取latch，获取失败则等待
Thread-10继续运行
```

```java
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);//放入阻塞队列中
}

public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();//把所有线程唤醒
        return true;
    }
    return false;
}
```

# 十、Future和Callable

## 1.Runnable的缺陷

- 不能返回一个返回值

- 不能抛出checked Exception

  我们最终是由主线程执行run方法且编写相关处理异常的代码。假如run方法能抛出异常，则由Thread类接受并处理相关异常，但是这些类不由我们编写，所以我们无法决定如何处理异常，由此可见，为了能处理异常，我们应该在main方法中处理异常。所以run方法不能抛出异常，这样直接在main函数中try/catch即可。

```java
public interface Runnable {
    public abstract void run();
}
```

## 2.Callable接口

- 类似于Runnable，被其它线程执行的任务
- 实现**call方法**
- **有返回值**

```java
public interface Callable<V> {
    V call() throws Exception;
}
```

## 3.Future类

### （1）用法一：Callable与Future的关系

- 我们可以用**Future.get()**来获取Callable接口返回的执行结果，可以通过**Future.isDone()**来判断任务是否已经执行完毕，以及取消该任务、限时获取任务结果等。
- 在call()未执行完毕之前，调用get()的线程（假定此时是主线程）**会被阻塞**，直到call()方法返回了结果后，此时future.get()才会得到该结果，然后主线程切换到Runnable状态。
- 所以Future是一个**存储器，存储了call()这个任务的结果**，而这个任务的执行时间是无法提前确定的，因为这完全取决于call()方法执行的情况。

### （2）主要方法

#### ①get()：获取结果

​		get()的行为取决于Callable任务的状态，只有以下这5种情况：

1. 任务正常完成：get()立即返回结果。
2. 任务**尚未完成**（未开始或进行中）：get将**阻塞**并直到任务完成。
3. 任务执行中抛出Exception：get()方法会抛出ExecutionException，这里的抛出异常，是call()执行时产生的异常。无论call()执行时抛出的异常类型是什么，**最后get方法抛出的异常类型都是ExecutionException。**
4. 任务被取消：get方法会抛出CancellationException
5. 任务超时：get方法有一个重载方法，是传入一个延迟时间的，如果时间到了还没有获得结果，get()就会抛出TimeoutException。

#### ②get(long timeout,TimeUnit unit)：有超时的获取

- 超时的需求很常见
- 用该方法时，如果call()在规定时间内完成了任务，那么就会正常获取到返回值；而如果在指定时间内没有计算出结果，那么就会**抛出TimeoutException。**
- 超时后不需要再获取返回值，**则任务需要取消**。

#### ③cancel()：取消任务的执行

- 如果这个任务还没有开始执行，任务会被正常的取消，未来也不会被执行，方法返回true。
- 如果任务已完成，或者已取消，那么cancel()会执行失败，方法返回false。
- 如果任务已经开始执行，那么这个取消方法不会直接取消该任务，而是根据我们填的参数mayInterruptIfRunning做判断。
  - true使用于任务能够处理interrupt
  - false仅用于避免启动尚未启动的任务，适用于：
    - 未能处理interrupt的任务
    - 不清楚任务是否支持取消
    - 需要等待已经开始的任务执行完成

#### ④isDone()：判断线程是否执行完毕

​		只会判断线程是否执行完毕，**不会判断线程是否成功执行**。

#### ⑤isCancelled()：判断线程是否被取消

### （3）代码演示

#### ①线程池的submit方法返回Future对象

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210730160812120.png" alt="image-20210730160812120" style="zoom:50%;" />

​		首先，我们要给线程池提交我们的任务，提交时线程会**立刻返回**给我们一个**空的Future容器**（placeholder）。当线程的任务一旦执行完毕，也就是我们可以获取结果时，线程池便会把该**结果填入到之前给我们的那个Future去**（而**不是创建一个新的Future**），我们此时便可以**从该Future中获得任务执行的结果**。

```java
/**
 * 演示一个Future的使用方法
 */
public class OneFuture {
    public static void main(String[] args) {
        ExecutorService service = Executors.newFixedThreadPool(10);
        Future<Integer> future = service.submit(new CallableTask());
        try {
            System.out.println(future.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
        service.shutdown();
    }

    static class CallableTask implements Callable<Integer>{

        @Override
        public Integer call() throws Exception {
            Thread.sleep(3000);
            return new Random().nextInt();
        }
    }
}

-176835188
```

#### ②Callable的Lambda表达式形式

```java
/**
 * 演示一个Future的使用方法,lambda形式
 */
public class OneFutureLambda {
    public static void main(String[] args) {
        ExecutorService service = Executors.newFixedThreadPool(10);
        Callable<Integer> callable = ()->{
            Thread.sleep(3000);
            return new Random().nextInt();
        };
        
        /*Callable<Integer> callable = new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                Thread.sleep(3000);
                return new Random().nextInt();
            }
        };*/
        
        Future<Integer> future = service.submit(callable);
        try {
            System.out.println(future.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
        service.shutdown();
    }
}
```

#### ③多个任务，用Future数组获取结果

```java
/**
 * 演示批量提交任务时，用List来批量接收结果
 */
public class MultiFutures {
    public static void main(String[] args) {
        ExecutorService service = Executors.newFixedThreadPool(5);
        ArrayList<Future> futures = new ArrayList<>();

        Callable<Integer> callable = () -> {
            Thread.sleep(3000);
            return new Random().nextInt();
        };

        for (int i = 0; i < 20; i++) {
            Future<Integer> future = service.submit(callable);
            futures.add(future);
        }
		//Thread.sleep(7000);
        for (int i = 0; i < 20; i++) {
            Future<Integer> f = futures.get(i);
            try {
                Integer integer = f.get();
                System.out.println("第" + (i + 1) + "个线程的结果为：" + integer);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
        }
        service.shutdown();
    }
}

第1个线程的结果为：1086620811
第2个线程的结果为：2049370056
第3个线程的结果为：-1435038242
第4个线程的结果为：-860900543
第5个线程的结果为：-1504765008
第6个线程的结果为：1417712235
第7个线程的结果为：-700006441
第8个线程的结果为：-1064005325
第9个线程的结果为：-1032986341
第10个线程的结果为：-1436018787
第11个线程的结果为：-1155180185
第12个线程的结果为：1617605817
第13个线程的结果为：-516575586
第14个线程的结果为：969387006
第15个线程的结果为：1699465185
第16个线程的结果为：202486537
第17个线程的结果为：844911663
第18个线程的结果为：-199200140
第19个线程的结果为：-1301144890
第20个线程的结果为：-49929546
```

#### ④任务执行过程中抛出Exception和isDone展示

```java
/**
 * 演示get方法过程中抛出异常，for循环为了演示Exception的抛出时机
 * 并不是一产生异常就抛出，直到我们get执行时，才会抛出。
 */
public class GetException {
    public static void main(String[] args) {
        ExecutorService service = Executors.newFixedThreadPool(20);
        Callable<Integer> callable = () -> {
            Thread.sleep(1000);
            throw new IllegalArgumentException("Callable抛出IllegalArgumentException异常");
        };

        Future<Integer> future = service.submit(callable);

        try {
            System.out.println(future.isDone());
            
            for (int i = 0; i < 5; i++) {
                System.out.println((i+1));
                Thread.sleep(500);//并不是一产生异常就抛出，直到我们get执行时，才会抛出
            }
            
            System.out.println(future.isDone());
            future.get();
        } catch (InterruptedException e) {
            e.printStackTrace();
            System.out.println("Callable抛出InterruptedException异常");
        } catch (ExecutionException e) {
            e.printStackTrace();
            System.out.println("Callable抛出ExecutionException异常");
        }
    }
}

false
1
2
3
4
5
true
Callable抛出ExecutionException异常
```

#### ⑤获取任务超时

```java
/**
 * 演示get的超时方法，需要注意超时后的处理，调用future.cancel()。
 * 演示cancel传入true和false的区别，代表是否中断正在执行的任务
 */
public class Timeout {
    private static final Ad DEFAULT_AD = new Ad("无网络时的默认广告");
    private static final ExecutorService service = Executors.newFixedThreadPool(10);

    static class Ad {
        String name;

        public Ad(String name) {
            this.name = name;
        }

        @Override
        public String toString() {
            return "Ad{" +
                    "name='" + name + '\'' +
                    '}';
        }
    }

    static class FetchAdTask implements Callable<Ad> {

        @Override
        public Ad call() throws Exception {
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                System.out.println("sleep期间被中断");
                return new Ad("被中断时的默认广告");
            }
            return new Ad("HITwh");
        }
    }

    public void printAd() {
        Future<Ad> f = service.submit(new FetchAdTask());
        Ad ad;
        try {
            //ad = f.get(4000, TimeUnit.MILLISECONDS);
            ad = f.get(2000, TimeUnit.MILLISECONDS);
        } catch (InterruptedException e) {
            ad = new Ad("被中断时的默认广告");
        } catch (ExecutionException e) {
            ad = new Ad("异常时的默认广告");
        } catch (TimeoutException e) {
            ad = new Ad("超时的默认广告");
            System.out.println("超时，未获取到广告");
            boolean cancel = f.cancel(true);
            System.out.println("cancel结果：" + cancel);
        }
        service.shutdown();
        System.out.println(ad);
    }

    public static void main(String[] args) {
        Timeout timeout = new Timeout();
        timeout.printAd();
    }
}

超时，未获取到广告
cancel结果：true
sleep期间被中断
Ad{name='超时的默认广告'}//并非被中断时的默认广告，此时ad已经被赋值为超时的默认广告
```

### （4）用法二：用FutureTask创建Future

- 用FutureTask获取Future和任务的结果
- FutureTask是一种**包装器**，可以把Callable转化成**Future和Runnable**，它**同时实现二者的接口**。
- 把Callable实例当做参数，生成FutureTask对象，然后把这个对象当做一个Runnable对象，用线程池或另起线程去执行这个Runnable对象，最后通过FutureTask获取刚才执行的结果。

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210730173155045.png" alt="image-20210730173155045" style="zoom:50%;" />

```java
/**
 * 演示FutureTask的用法
 */
public class FutureTaskDemo {
    public static void main(String[] args) {
        Callable<Integer> callable = () -> {
            System.out.println("子线程正在计算");
            Thread.sleep(3000);
            int sum = 0;
            for (int i = 0; i <= 100; i++) {
                sum += i;
            }
            return sum;
        };

        FutureTask<Integer> integerFutureTask = new FutureTask<>(callable);
        /*Thread thread = new Thread(integerFutureTask);
        thread.start();*/

        ExecutorService service = Executors.newFixedThreadPool(2);
        service.submit(integerFutureTask);

        try {
            System.out.println("Task运行结果：" + integerFutureTask.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
        service.shutdown();
    }
}

子线程正在计算
Task运行结果：5050
```

### （5）Future的注意点

- 当for循环**批量获取**future的结果时，容易发生一部分线程很慢的情况，get方法调用时应使用timeout限制。
- future的**生命周期不能后退**，一旦完成任务，它就永久停在了“已完成”的状态，不能重头再来。

![image-20210730174150351](http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210730174150351.png)

# 十一、实战项目：高性能缓存

## 1.最简单的缓存——HashMap

```java
/**
 * 最简单的缓存形式：HashMap
 */
public class Cache1 {
    private final HashMap<String, Integer> cache = new HashMap<>();

    public static void main(String[] args) throws InterruptedException {
        Cache1 cache1 = new Cache1();
        System.out.println("开始计算");
        Integer result;
        result = cache1.computer("13");
        System.out.println("第1次计算结果" + result);
        result = cache1.computer("13");
        System.out.println("第2次计算结果" + result);
    }

    public Integer computer(String userId) throws InterruptedException {
        Integer result = cache.get(userId);
        //先检查HashMap里有没有保存过之前的计算结果
        if (result == null) {
            //如果缓存中找不到，那么需要现在计算一下结果，并且保存到HashMap中
            result = doCompute(userId);
            cache.put(userId, result);
        }
        return result;
    }

    private Integer doCompute(String userId) throws InterruptedException {
        Thread.sleep(2000);
        return new Integer(userId);
    }
}

开始计算
第1次计算结果13
第2次计算结果13
```

## 2.保证并发安全

### （1）synchronized

- 性能差
- 代码复用性差

### （2）装饰者模式解耦

​		我们假设ExpensiveFunction类是耗时计算的实现类，实现了Computable接口，但是**其本身不具备缓存功能**。

​		Computable：

```java
/**
 * 有一个计算函数computer，用来代表耗时计算
 * 每个计算器都要实现这个接口，这样就可以无侵入实现缓存功能
 */
public interface Computable<A,V> {
    V compute(A arg) throws Exception;
}
```

​		ExpensiveFunction：

```java
/**
 * 耗时计算的实现类，实现了computable接口，本身不具备缓存能力，无需考虑缓存的事情
 */
public class ExpensiveFunction implements Computable<String,Integer>{

    @Override
    public Integer compute(String arg) throws Exception {
        Thread.sleep(2000);
        
        //return new Integer(arg);
        return Integer.valueOf(arg);
    }
}
```

​		Cache2：

```java
/**
 * 用装饰者模式，给计算器自动添加缓存功能
 */
public class Cache2<A, V> implements Computable<A, V> {
    private final Map<A, V> cache = new HashMap<>();

    private final Computable<A, V> computable;

    public Cache2(Computable<A, V> computable) {
        this.computable = computable;
    }

    @Override
    public synchronized V compute(A arg) throws Exception {
        System.out.println("进入缓存机制");
        V result = cache.get(arg);
        if (result == null) {
            result = computable.compute(arg);//装饰者模式的体现
            cache.put(arg, result);
        }
        return result;
    }

    public static void main(String[] args) throws Exception {
        Cache2<String, Integer> expensiveComputer = new Cache2(new ExpensiveFunction());
        Integer result = null;
        result = expensiveComputer.compute("22");
        System.out.println("第1次运算结果" + result);
        result = expensiveComputer.compute("22");
        System.out.println("第2次运算结果" + result);
    }
}

进入缓存机制
第1次运算结果22
进入缓存机制
第2次运算结果22
```

### （3）减少synchronized的粒度——仍然不安全

```java
/**
 * 减少synchronized的粒度，试图提高性能，但是依然在读写并发情况下不安全
 */
public class Cache4<A, V> implements Computable<A, V> {
    private final Map<A, V> cache = new HashMap<>();

    private final Computable<A, V> computable;

    public Cache4(Computable<A, V> computable) {
        this.computable = computable;
    }

    @Override
    public V compute(A arg) throws Exception {
        System.out.println("进入缓存机制");
        V result = cache.get(arg);
        if (result == null) {
            result = computable.compute(arg);//装饰者模式的体现
            //仍存在线程安全问题
            synchronized (this){
                cache.put(arg, result);
            }
        }
        return result;
    }

    public static void main(String[] args) throws Exception {
        Cache4<String, Integer> expensiveComputer = new Cache4(new ExpensiveFunction());
        Integer result = null;
        result = expensiveComputer.compute("22");
        System.out.println("第1次运算结果" + result);
        result = expensiveComputer.compute("22");
        System.out.println("第2次运算结果" + result);
    }
}
```

### （4）用ConcurrentHashMap保证并发安全

​		缺点：

- 在计算完成前，另一个要求计算相同值的请求到来，会导致**计算两遍**，这与缓存想避免多次计算的初衷相反，不可取。

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210730195911132.png" alt="image-20210730195911132" style="zoom: 67%;" />

```java
/**
 * 用并发集合ConcurrentHashMap
 */
public class Cache5<A, V> implements Computable<A, V> {
    private final Map<A, V> cache = new ConcurrentHashMap<>();

    private final Computable<A, V> computable;

    public Cache5(Computable<A, V> computable) {
        this.computable = computable;
    }

    @Override
    public V compute(A arg) throws Exception {
        System.out.println("进入缓存机制");
        V result = cache.get(arg);
        if (result == null) {
            result = computable.compute(arg);//装饰者模式的体现
            cache.put(arg, result);
        }
        return result;
    }

    public static void main(String[] args) throws Exception {
        Cache5<String, Integer> expensiveComputer = new Cache5(new ExpensiveFunction());
        Integer result = null;
        result = expensiveComputer.compute("22");
        System.out.println("第1次运算结果" + result);
        result = expensiveComputer.compute("22");
        System.out.println("第2次运算结果" + result);
    }
}
```

## 3.避免重复计算——Future和Callable

​		**第二个请求会因为cache已经成功put(arg, ft)，直接执行f.get()，此时因为ft.run()的计算过程还未计算完毕，所以第二个请求会阻塞到f.get()直至计算完毕**

```java
/**
 * 利用Future避免重复计算
 */
public class Cache7<A, V> implements Computable<A, V> {
    private final Map<A, Future<V>> cache = new ConcurrentHashMap<>();

    private final Computable<A, V> computable;

    public Cache7(Computable<A, V> computable) {
        this.computable = computable;
    }

    @Override
    public V compute(A arg) throws Exception {
        Future<V> f = cache.get(arg);
        if (f == null) {
            Callable<V> callable = new Callable<V>() {
                @Override
                public V call() throws Exception {
                    return computable.compute(arg);
                }
            };
            FutureTask<V> ft = new FutureTask<>(callable);
            f = ft;
            cache.put(arg, ft);//避免了重复计算
            System.out.println("从FutureTask调用计算函数");
            ft.run();//执行Callable
        }
        //第二个请求会因为cache已经成功put(arg, ft)，直接执行f.get()
        //此时因为ft.run()的计算过程还未计算完毕，所以第二个请求会阻塞到这里，直至计算完毕
        return f.get();
    }

    public static void main(String[] args) throws Exception {
        Cache7<String, Integer> expensiveComputer = new Cache7(new ExpensiveFunction());

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Integer result = expensiveComputer.compute("33");
                    System.out.println(Thread.currentThread().getName() + "计算结果：" + result);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Integer result = expensiveComputer.compute("34");
                    System.out.println(Thread.currentThread().getName() + "计算结果：" + result);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Integer result = expensiveComputer.compute("33");
                    System.out.println(Thread.currentThread().getName() + "计算结果：" + result);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
}

从FutureTask调用计算函数
从FutureTask调用计算函数
Thread-2计算结果：33
Thread-0计算结果：33
Thread-1计算结果：34
```



## 4.依然存在重复计算的可能——用原子操作putIfAbsent

- 如果有两个同时计算33的线程，同时调用cache.get方法，那么返回的结果都为null，后面还是会**创建两个任务去计算相同的值**。

<img src="http://uranus-picture.oss-cn-beijing.aliyuncs.com/img/image-20210730203254236.png" alt="image-20210730203254236" style="zoom:67%;" />

```java
public V compute(A arg) throws Exception {
    Future<V> f = cache.get(arg);
    if (f == null) {
        Callable<V> callable = new Callable<V>() {
            @Override
            public V call() throws Exception {
                return computable.compute(arg);
            }
        };
        FutureTask<V> ft = new FutureTask<>(callable);
        f = cache.putIfAbsent(arg, ft);//避免了重复计算
        if (f == null) {
            f = ft;
            System.out.println("从FutureTask调用计算函数");
        	ft.run();//执行Callable
        }
    }
    return f.get();
}

default V putIfAbsent(K key, V value) {
    V v = get(key);
    if (v == null) {
        v = put(key, value);
    }

    return v;
}
```

## 5.计算中抛出异常

### （1）添加异常

​		MayFail：

```java
/**
 * 耗时计算的实现类，有概率计算失败
 */
public class MayFail implements Computable<String,Integer>{

    @Override
    public Integer compute(String arg) throws Exception {
        double random = Math.random();
        if(random>0.5){
            throw new IOException("读取文件出错");
        }
        Thread.sleep(3000);

        //return new Integer(arg);
        return Integer.valueOf(arg);
    }
}
```

​		Cache8：

```java
/**
 * 抛出异常
 */
public class Cache8<A, V> implements Computable<A, V> {
    private final Map<A, Future<V>> cache = new ConcurrentHashMap<>();

    private final Computable<A, V> computable;

    public Cache8(Computable<A, V> computable) {
        this.computable = computable;
    }

    @Override
    public V compute(A arg) throws Exception {
        Future<V> f = cache.get(arg);
        if (f == null) {
            Callable<V> callable = new Callable<V>() {
                @Override
                public V call() throws Exception {
                    return computable.compute(arg);
                }
            };
            FutureTask<V> ft = new FutureTask<>(callable);
            f = cache.putIfAbsent(arg, ft);//避免了重复计算
            if (f == null) {
                f = ft;
                System.out.println("从FutureTask调用计算函数");
                ft.run();//执行Callable
            }
        }
        return f.get();
    }

    public static void main(String[] args) throws Exception {
        Cache8<String, Integer> expensiveComputer = new Cache8(new MayFail());

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Integer result = expensiveComputer.compute("33");
                    System.out.println(Thread.currentThread().getName() + "计算结果：" + result);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Integer result = expensiveComputer.compute("34");
                    System.out.println(Thread.currentThread().getName() + "计算结果：" + result);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Integer result = expensiveComputer.compute("33");
                    System.out.println(Thread.currentThread().getName() + "计算结果：" + result);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
}

从FutureTask调用计算函数
从FutureTask调用计算函数
java.util.concurrent.ExecutionException: java.io.IOException: 读取文件出错
	at java.util.concurrent.FutureTask.report(FutureTask.java:122)
	at java.util.concurrent.FutureTask.get(FutureTask.java:192)
	at cache.Cache8.compute(Cache8.java:43)
	at cache.Cache8$3.run(Cache8.java:65)
	at java.lang.Thread.run(Thread.java:748)
Caused by: java.io.IOException: 读取文件出错
	at cache.computable.MayFail.compute(MayFail.java:14)
	at cache.computable.MayFail.compute(MayFail.java:8)
	at cache.Cache8$1.call(Cache8.java:32)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at cache.Cache8.compute(Cache8.java:40)
	... 2 more
Thread-2计算结果：33
Thread-0计算结果：33
```

### （2）正确的异常处理逻辑

- CancellationException和InterruptedException是人为取消造成的，那么我们应该立即终止任务；
- 如果是计算错误，我们应该重试，直至计算正确。
- 这里我们加上while(true)保证计算出错不会影响我们的逻辑。
  - 如果计算错误，进入下一个循环重新计算，直至计算成功；
  - 如果是人为取消，就抛出异常然后结束运行。

## 6.缓存污染问题

```java
/**
 * 解决缓存污染问题
 */
public class Cache8<A, V> implements Computable<A, V> {
    private final Map<A, Future<V>> cache = new ConcurrentHashMap<>();

    private final Computable<A, V> computable;

    public Cache8(Computable<A, V> computable) {
        this.computable = computable;
    }

    @Override
    public V compute(A arg) throws InterruptedException, ExecutionException {
        while (true) {
            Future<V> f = cache.get(arg);
            if (f == null) {
                Callable<V> callable = new Callable<V>() {
                    @Override
                    public V call() throws Exception {
                        return computable.compute(arg);
                    }
                };
                FutureTask<V> ft = new FutureTask<>(callable);
                f = cache.putIfAbsent(arg, ft);//避免了重复计算
                if (f == null) {
                    f = ft;
                    System.out.println("从FutureTask调用计算函数");
                    ft.run();//执行Callable
                }
            }
            try {
                return f.get();
            } catch (InterruptedException e) {
                System.out.println("任务被中断");
                cache.remove(arg);//解决缓存污染
                throw e;
            } catch (ExecutionException e) {
                System.out.println(Thread.currentThread().getName() + "计算错误，需要重试");
                cache.remove(arg);//解决缓存污染
            } catch (CancellationException e) {
                System.out.println("任务被取消");
                cache.remove(arg);//解决缓存污染
                throw e;
            }
        }
    }

    public static void main(String[] args) throws Exception {
        Cache8<String, Integer> expensiveComputer = new Cache8(new MayFail());

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Integer result = expensiveComputer.compute("33");
                    System.out.println(Thread.currentThread().getName() + "计算结果：" + result);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Integer result = expensiveComputer.compute("34");
                    System.out.println(Thread.currentThread().getName() + "计算结果：" + result);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Integer result = expensiveComputer.compute("33");
                    System.out.println(Thread.currentThread().getName() + "计算结果：" + result);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

        /*Future<Integer> future = expensiveComputer.cache.get("33");
        future.cancel(true);*/
    }
}
```

## 7.缓存过期功能和随机性

- 为每个结果指定**过期时间**，并定期扫描过期的元素。
- 如果同时国企，那么同时都拿不到缓存，导致打爆MySQL和CPU，造成缓存雪崩，缓存击穿等高并发下的缓存问题。
- 将缓存的过期时间设置为随机。

```java
/**
 * 出于安全性考虑，缓存需要设置有效期，到期自动失效
 * 否则容易造成缓存不一致问题
 */
public class Cache9<A, V> implements Computable<A, V> {
    private final Map<A, Future<V>> cache = new ConcurrentHashMap<>();

    private final Computable<A, V> computable;

    public final static ScheduledExecutorService executor = Executors.newScheduledThreadPool(5);

    public Cache9(Computable<A, V> computable) {
        this.computable = computable;
    }

    @Override
    public V compute(A arg) throws InterruptedException, ExecutionException {
        while (true) {
            Future<V> f = cache.get(arg);
            if (f == null) {
                Callable<V> callable = new Callable<V>() {
                    @Override
                    public V call() throws Exception {
                        return computable.compute(arg);
                    }
                };
                FutureTask<V> ft = new FutureTask<>(callable);
                f = cache.putIfAbsent(arg, ft);//避免了重复计算
                if (f == null) {
                    f = ft;
                    System.out.println("从FutureTask调用计算函数");
                    ft.run();//执行Callable
                }
            }
            try {
                return f.get();
            } catch (InterruptedException e) {
                System.out.println("任务被中断");
                cache.remove(arg);
                throw e;
            } catch (ExecutionException e) {
                System.out.println(Thread.currentThread().getName() + "计算错误，需要重试");
                cache.remove(arg);
            } catch (CancellationException e) {
                System.out.println(Thread.currentThread().getName() + "任务被取消");
                cache.remove(arg);
                throw e;
            }
        }
    }

    public V compute(A arg,long expire) throws ExecutionException, InterruptedException {
        if (expire>0){
            executor.schedule(new Runnable() {
                @Override
                public void run() {
                    expire(arg);
                }
            },expire,TimeUnit.MILLISECONDS);
        }
        return compute(arg);
    }

    public synchronized void expire(A key){
        Future<V> future = cache.get(key);
        if(future != null){
            if(!future.isDone()){
                System.out.println("Future任务被取消");
                future.cancel(true);
            }
            System.out.println("过期时间到，缓存被清除");
            cache.remove(key);
        }
    }

    public V computeRandomExpire(A arg) throws ExecutionException, InterruptedException {
        long randomExpire = (long) (Math.random()*10000);
        return compute(arg,randomExpire);
    }
    
    public static void main(String[] args) throws Exception {
        Cache9<String, Integer> expensiveComputer = new Cache9(new MayFail());

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Integer result = expensiveComputer.compute("33",4000L);
                    System.out.println(Thread.currentThread().getName() + "计算结果：" + result);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Integer result = expensiveComputer.compute("34");
                    System.out.println(Thread.currentThread().getName() + "计算结果：" + result);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Integer result = expensiveComputer.compute("33");
                    System.out.println(Thread.currentThread().getName() + "计算结果：" + result);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

        Thread.sleep(6000);
        Integer result = expensiveComputer.compute("33");
        System.out.println(Thread.currentThread().getName() + "计算结果：" + result);
        executor.shutdown();
    }
}
```

## 8.用线程池测试缓存性能

```java
public class Cache10 {
    static Cache9<String, Integer> expensiveComputer = new Cache9<>(new ExpensiveFunction());

    public static void main(String[] args) {
        ExecutorService service = Executors.newFixedThreadPool(10000);
        long start = System.currentTimeMillis();
        for (int i = 0; i < 10000; i++) {
            service.submit(() -> {
                Integer result = null;
                try {
                    result = expensiveComputer.compute("44");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (ExecutionException e) {
                    e.printStackTrace();
                }
                System.out.println(result);
            });
        }
        
        service.shutdown();
        while (!service.isTerminated()) {

        }
        long end = System.currentTimeMillis();
        System.out.println("总耗时：" + (end - start) + "毫秒");
    }
}

总耗时：2505毫秒
```

## 9.用CountDownLatch实现压测

```java
public class Cache10 {
    static Cache9<String, Integer> expensiveComputer = new Cache9<>(new ExpensiveFunction());
    public static CountDownLatch latch = new CountDownLatch(1);

    public static void main(String[] args) throws InterruptedException {
        ExecutorService service = Executors.newFixedThreadPool(10000);
        long start = System.currentTimeMillis();
        for (int i = 0; i < 10000; i++) {
            service.submit(() -> {
                Integer result = null;
                try {
                    System.out.println(Thread.currentThread().getName() + "开始等待");
                    latch.await();
                    System.out.println(Thread.currentThread().getName() + "被放行");
                    result = expensiveComputer.compute("44");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (ExecutionException e) {
                    e.printStackTrace();
                }
                System.out.println(result);
            });
        }

        Thread.sleep(5000);
        latch.countDown();
        service.shutdown();
        long end = System.currentTimeMillis();
        //System.out.println("总耗时：" + (end - start) + "毫秒");
    }
}
```

## 10.用ThreadLocal确认时间的统一性

```java
public class Cache10 {
    static Cache9<String, Integer> expensiveComputer = new Cache9<>(new ExpensiveFunction());
    public static CountDownLatch latch = new CountDownLatch(1);

    public static void main(String[] args) throws InterruptedException {
        ExecutorService service = Executors.newFixedThreadPool(1000);
        long start = System.currentTimeMillis();
        for (int i = 0; i < 1000; i++) {
            service.submit(() -> {
                Integer result = null;
                try {
                    System.out.println(Thread.currentThread().getName() + "开始等待");
                    latch.await();
                    SimpleDateFormat simpleDateFormat = ThreadSafeFormatter.dateFormatThreadLocal.get();
                    String time = simpleDateFormat.format(new Date());
                    System.out.println(Thread.currentThread().getName() + "：" + time + "被放行");
                    result = expensiveComputer.compute("44");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (ExecutionException e) {
                    e.printStackTrace();
                }
                System.out.println(result);
            });
        }

        Thread.sleep(5000);
        latch.countDown();
        service.shutdown();
        long end = System.currentTimeMillis();
        //System.out.println("总耗时：" + (end - start) + "毫秒");
    }
}

class ThreadSafeFormatter {
    public static ThreadLocal<SimpleDateFormat> dateFormatThreadLocal = new ThreadLocal<SimpleDateFormat>() {
        @Override
        protected SimpleDateFormat initialValue() {
            return new SimpleDateFormat("mm:ss");
        }
    };

    public static ThreadLocal<SimpleDateFormat> dateFormatThreadLocal2 = ThreadLocal.withInitial(() -> new SimpleDateFormat("mm:ss"));
}
```