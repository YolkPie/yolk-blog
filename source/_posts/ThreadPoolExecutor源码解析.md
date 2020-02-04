---
title: ThreadPoolExecutor源码解析
date: 2020-02-04 15:48:44
tags: 线程池
categories: Java
author: 王巨峰
keywords: ThreadPoolExecutor
description: ThreadPoolExecutor是实现其他线程池的核心。
cover: https://img10.360buyimg.com/imagetools/jfs/t1/90370/34/11521/107557/5e39232fE0443a650/a68f5e97a56d8d46.png
top_img: https://img11.360buyimg.com/imagetools/jfs/t1/103030/23/11663/54205/5e392306E702b79c4/4282caa4f87627b4.png
---
## 构造方法参数

1.构造方法就不在此赘述，重点关注构造方法种的参数。
<!-- more -->

| 参数名                   |                             作用                             |
| ------------------------ | :----------------------------------------------------------: |
| corePoolSize             |                        核心线程池大小                        |
| maximumPoolSize          |                        最大线程池大小                        |
| keepAliveTime            | 线程池中超过corePoolSize数目的空闲线程最大存活时间；可以allowCoreThreadTimeOut(true)使得核心线程有效时间 |
| TimeUnit                 |                    keepAliveTime时间单位                     |
| workQueue                |                         阻塞任务队列                         |
| threadFactory            |                         新建线程工厂                         |
| RejectedExecutionHandler | 当提交任务数超过maxmumPoolSize+workQueue之和时，任务会交给RejectedExecutionHandler来处理 |

2.重点理解：  corePoolSize  ，  maximumPoolSize ，workQueue这三者之间的联系

a.当线程池小于corePoolSize时，新提交任务将创建一个新线程执行任务，即使此时线程池中存在空闲线 程。     b.当线程池达到corePoolSize时，新提交任务将被放入workQueue中，等待线程池中任务调度执行 
c.当workQueue已满，且maximumPoolSize>corePoolSize时，新提交任务会创建新线程执行任务 
d.当提交任务数超过maximumPoolSize时，新提交任务由RejectedExecutionHandler处理 
e.当线程池中超过corePoolSize线程，空闲时间达到keepAliveTime时，关闭空闲线程 
f.当设置allowCoreThreadTimeOut(true)时，线程池中corePoolSize线程空闲时间达到keepAliveTime也将关闭 

## 核心变量

```java
// 注意，代码的注释都在代码之后。
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING,0))；
      //标志当前线程池的状态
private static final int COUNT_BITS = Integer.SIZE - 3;
     //   等于29(标记线程个数)
private static final int CAPACITY = (1 << COUNT_BITS) - 1;
     //  CAPACITY =  0001 1111 1111 1111 1111 1111 1111 1111   


//线程池的各种状态：
private static final int RUNNING = -1 << COUNT_BITS;
//用二进制表示RUNNING  1110  0000 0000 0000 0000 0000 0000 0000 运行态，可处理新任务并执行队列中的任务
private static final int SHUTDOWN =  0 << COUNT_BITS;
//用二进制表示SHUTDOWN 0000  0000 0000 0000 0000 0000 0000 0000 关闭态，不接受新任务，但处理队列中的任务
private static final int STOP   =  1 << COUNT_BITS;
//用二进制表示STOP     0010  0000 0000 0000 0000 0000 0000 0000 停止态，不接受新任务，不处理队列中任务，且打断运行中任务
private static final int TIDYING  =  2 << COUNT_BITS;
//用二进制表示TIDYING  0100  0000 0000 0000 0000 0000 0000 0000 整理态，所有任务已经结束，workerCount = 0 ，将执行terminated()方法
private static final int TERMINATED =  3 << COUNT_BITS
//用二进制表示TERMINATED 0110  0000 0000 0000 0000 0000 0000 0000  结束态，terminated() 方法已完成


private final BlockingQueue<Runnable> workQueue；
//任务缓存队列，用来存放等待执行的任务
private final ReentrantLock mainLock = new ReentrantLock();
//操作许多变量都需要这个锁
private final HashSet<Worker> workers = new HashSet<Worker>();
//存放工作集，需要获取mainLock才可以操作这个变量
private volatile boolean allowCoreThreadTimeOut;
//是否允许为核心线程设置存活时间
private int largestPoolSize;
//用来记录线程池中曾经出现过的最大线程数
private long completedTaskCount
//用来记录已经执行完毕的任务个数

```

| 状态       |                                                              |
| ---------- | ------------------------------------------------------------ |
| RUNNING    | 运行态，可处理新任务并执行队列中的任务                       |
| SHUTDOW    | 关闭态，不接受新任务，但处理队列中的任务                     |
| STOP       | 停止态，不接受新任务，不处理队列中任务，且打断运行中任务     |
| TIDYING    | 整理态，所有任务已经结束，workerCount = 0，将执行terminated()方法 |
| TERMINATED | 结束态，terminated() 方法已完成                              |

**状态转换：**

*RUNNING -> SHUTDOWN*：手动调用shutdown方法，或者ThreadPoolExecutor要被GC回收的时候调用finalize方法，finalize方法内部也会调用shutdown方法

*(RUNNING or SHUTDOWN) -> STOP*：调用shutdownNow方法

*SHUTDOWN -> TIDYING*：当队列和线程池都为空的时候

*STOP -> TIDYING*：当线程池为空的时候

*TIDYING -> TERMINATED*：terminated方法调用完成之后

**ThreadPoolExecutor演示状态图**

![ThreadPoolExecutor执行状态图](executor.png)

> 摘自：https://www.jianshu.com/p/f8a73cb0983a

## 核心方法

### runStateOf(int c)  计算线程池当前的状态

```java
/**
 * 这个方法是计算出线程池的状态，
 * 由于 CAPACITY =  0001 1111 1111 1111 1111 1111 1111 1111 ，在进行取反运算后 
 * 变为11100000000000000000000000000000，再于参数做 & 操作,会将低29位全部置为0,而   
 * 高三位还是保持 111 不变，就可以判断出线程池当前处于什么状态
 */
private static int runStateOf(int c) {
    return c & ~CAPACITY;
}

```

### workerCountOf(int c) 计算线程数量

```java
/**
 * 这个方法是计算当前线程池种线程的数量
 * CAPACITY =  0001 1111 1111 1111 1111 1111 1111 1111
 * 参数 c 与 CAPACITY 进行 & 运算后，会保留 c 低29位的值，从而知道线程数量
 */
private static int workerCountOf(int c) {
    return c & CAPACITY;
}
```

### ctlOf(int rs, int wc)  

```java
 /**
  * 此方法是为了在本ThreadPoolExecutor种只是为了将“线程状态”和“线程数量” 放在一个变量中
  * 比如 ctlOf(RUNNING, 0)
  */
private static int workerCountOf(int c) {
    return c & CAPACITY;
}
```

### execute(Runnable command)  execute 是开启ThreadPoolExecutor工作的方法。

```java
public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();

        int c = ctl.get();
        // 通过位运算得出当然线程池中的worker数量与构造参数corePoolSize进行比较
        if (workerCountOf(c) < corePoolSize) {
        // 如果小于corePoolSize，则直接新增一个worker，并把当然用户提交的任务               command作为参数，如果成功则返回。
            if (addWorker(command, true))
                return;
         // 如果失败，则获取最新的线程池数据
            c = ctl.get();
        }
         //如果线程池仍在运行，则把任务放到阻塞队列中等待执行。
        if (isRunning(c) && workQueue.offer(command)) {
         //这里的recheck思路是为了处理并发问题,再次检查线程池的状态是否为运行态
            int recheck = ctl.get();
         //当任务成功放入队列时，如果recheck发现线程池已经不再运行了则从队列中把任务             删除
            if (!isRunning(recheck) && remove(command))
                // 删除成功以后，会调用构造参数传入的拒绝策略。
                reject(command);
                // 如果worker的数量为0（此时队列中可能有任务没有执行），则新建一个                  worker（由于此时新建woker的目的是执行队列中堆积的任务，
                // 因此入参没有执行任务，请接着看 addWorker 方法）。
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        //如果前面的新增woker，放入队列都失败，则会继续新增worker此时线程池的状态是woker数量达到corePoolSize，阻塞队列任务已满
        //只能基于maximumPoolSize参数新建woker
        else if (!addWorker(command, false))
            reject(command);
    }
```

​   其实从以上execute 可以看出，再执行该方法时，会根据线程池的状态等进行不用的操作 a,直接创建线程池  b，添加到阻塞队列  c.  基于maximumPoolSize创建work

### addWorker(Runnable firstTask, boolean core)

```java
 private boolean addWorker(Runnable firstTask, boolean core) {
        //CAS 操作校验线程池的状态
        retry:
        for (; ; ) {
            int c = ctl.get();
            int rs = runStateOf(c);

            // Check if queue empty only if necessary.
            //检查线程池 是否是 SHUTDOWN 状态 ,如果 “是SHUTDOWN 状态” && “不是SHUTDOWN or  firstTask不是null or  workQueue不是空” 就不能添加
            if (rs >= SHUTDOWN &&
                    !(rs == SHUTDOWN &&
                            firstTask == null &&
                            !workQueue.isEmpty()))
                return false;

            for (; ; ) {
                int wc = workerCountOf(c);
                //校验线程数量
                if (wc >= CAPACITY ||
                        wc >= (core ? corePoolSize : maximumPoolSize))
                    return false;
                //线程数加1  CAS 操作 ，操作成功跳出循环
                if (compareAndIncrementWorkerCount(c))
                    break retry; //线程池加 1 成功，跳出循环
                 //加1 失败，获取最新的线程池个数，判断线程池状态........
                c = ctl.get(); 
                //判断当前线程池状态是不是之前获取过的线程池状态，
                //如果不是，说明又别的地方已经将线程池数量、状态改变了，要从retry开始操作
                //如果是，说明不用再次判断线程池状态，继续当前循环即可
                if (runStateOf(c) != rs)
                    continue retry;
                // else CAS failed due to workerCount change; retry inner loop
            }
        }

        boolean workerStarted = false;
        boolean workerAdded = false;
        ThreadPoolExecutor.Worker w = null;
        try {
            //通过线程池构造函数参数threadFactory生成的woker对象
            //注意 Work 对象其实实现了 Runnable 它本身就是一个线程，Work 对象会 firstTask保              存我们的自定义线程
            w = new ThreadPoolExecutor.Worker(firstTask);           
            //这个变量t就是代表woker线程，不是用户提交的线程任务firstTask！！！
            final Thread t = w.thread;
            if (t != null) {
                final ReentrantLock mainLock = this.mainLock;
                 // 加锁,对线程池状态进行判断
                mainLock.lock();
                try {
                    //获取线程池状态
                    int rs = runStateOf(ctl.get()); 
                    if (rs < SHUTDOWN ||
                            (rs == SHUTDOWN && firstTask == null)) {
                        //新建的work不能是存活，因为还没运行
                        if (t.isAlive()) // precheck that t is startable
                            throw new IllegalThreadStateException();
                        //把新建的woker线程放入集合保存 ->  HashSet
                        workers.add(w);           
                        int s = workers.size();
                        //为了实时监控 线程池中的线程数量
                        if (s > largestPoolSize)  
                            largestPoolSize = s;
                        workerAdded = true;
                    }
                } finally {
                    mainLock.unlock();
                }
                //添加成功，开始执行
                if (workerAdded) {
                    t.start();
                    workerStarted = true;
                }
            }
        } finally {
            //添加失败
            if (!workerStarted)
                addWorkerFailed(w);
        }
        return workerStarted;
    }
```

​   在 addWork方法种，我们可以看到，ThreadPoolExecutor为了更高效的运行，只在必要的地方加锁，而其他操作采用CAS 操作来解决并发问题，关于CAS操作，请浏览本文最后一节。

### Worker类

​   程池中的每一个线程被封装成一个Worker对象，ThreadPool维护的其实就是一组Worker对象，看一下Worker的定义：

```java
  private final class Worker
            extends AbstractQueuedSynchronizer
            implements Runnable {

        private static final long serialVersionUID = 6138294804551838833L;

        final Thread thread;
      
        Runnable firstTask;


        volatile long completedTasks;

        Worker(Runnable firstTask) {
            setState(-1); // inhibit interrupts until runWorker
            this.firstTask = firstTask;
            this.thread = getThreadFactory().newThread(this);
        }


        public void run() {
            runWorker(this);
        }

        // Lock methods
        //
        // The value 0 represents the unlocked state.
        // The value 1 represents the locked state.

        protected boolean isHeldExclusively() {
            return getState() != 0;
        }

        protected boolean tryAcquire(int unused) {
            if (compareAndSetState(0, 1)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        protected boolean tryRelease(int unused) {
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }

        public void lock() {
            acquire(1);
        }

        public boolean tryLock() {
            return tryAcquire(1);
        }

        public void unlock() {
            release(1);
        }

        public boolean isLocked() {
            return isHeldExclusively();
        }

        void interruptIfStarted() {
            Thread t;
            if (getState() >= 0 && (t = thread) != null && !t.isInterrupted()) {
                try {
                    t.interrupt();
                } catch (SecurityException ignore) {
                }
            }
        }
    }
```

Worker类继承了AQS，并实现了Runnable接口，注意其中的firstTask和thread属性：firstTask用它来保存传入的任务；thread是在调用构造方法时通过ThreadFactory来创建的线程，是用来处理任务的线程。

在调用构造方法时，需要把任务传入，这里通过`getThreadFactory().newThread(this);`来新建一个线程，newThread方法传入的参数是this，因为Worker本身继承了Runnable接口，也就是一个线程，所以一个Worker对象在启动的时候会调用Worker类中的run方法。

Worker继承了AQS，使用AQS来实现独占锁的功能。为什么不使用ReentrantLock来实现呢？可以看到tryAcquire方法，它是不允许重入的，而ReentrantLock是允许重入的：

1. lock方法一旦获取了独占锁，表示当前线程正在执行任务中，这时就不能中断该线程。

2. 如果该线程现在不是独占锁的状态，也就是空闲的状态，说明它没有在处理任务，这时可以对该线程进行中断；

3. 线程池在执行shutdown方法或tryTerminate方法时会调用interruptIdleWorkers方法来中断空闲的线程，interruptIdleWorkers方法会使用tryLock方法来判断线程池中的线程是否是空闲状态；

4. 之所以设置为不可重入，是因为不希望任务在调用像setCorePoolSize这样的线程池控制方法时重新获取锁。如果使用ReentrantLock，它是可重入的，这样如果在任务中调用了如setCorePoolSize这类线程池控制的方法，会中断正在运行的线程。下面是setCorePoolSize方法，interruptIdleWorkers请接着往下看

   ```java
    public void setCorePoolSize(int corePoolSize) {
           if (corePoolSize < 0)
               throw new IllegalArgumentException();
           int delta = corePoolSize - this.corePoolSize;
           this.corePoolSize = corePoolSize;
           if (workerCountOf(ctl.get()) > corePoolSize)
               //中断线程
               interruptIdleWorkers();
           else if (delta > 0) {
               int k = Math.min(delta, workQueue.size());
               while (k-- > 0 && addWorker(null, true)) {
                   if (workQueue.isEmpty())
                       break;
               }
           }
       }
   ```

所以，Worker继承自AQS，用于判断线程是否空闲以及是否可以被中断。

​   在构造方法中`setState(-1);`，把state变量设置为-1，为什么这么做呢？是因为AQS中默认的state是0，如果刚创建了一个Worker对象，还没有执行任务时，这时就不应该被中断，看一下tryAquire方法：

```java
   public boolean tryLock() {
            return tryAcquire(1);
     }
  protected boolean tryAcquire(int unused) {
            if (compareAndSetState(0, 1)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
    
```

### runWork ()

```java
 final void runWorker(ThreadPoolExecutorBak.Worker w) {
        Thread wt = Thread.currentThread();
        Runnable task = w.firstTask;
        w.firstTask = null;
        w.unlock(); // allow interrupts
        boolean completedAbruptly = true;
        try {
            //如果task不为null 或者 从队列中取出的任务不为null
            //task != null 成立时，直接运行，不会从队列中拿任务; 只有再不成立时 才回去队列里               拿任务
            //当 task 传null 意味着建立工作线程执行队列中的自定义线程
            while (task != null || (task = getTask()) != null) {
                w.lock();
                //interrupt() 中断线程；interrupted()和isInterrupted()判断线程是否中断
                //Thread.interrupted() 测试当前线程是否被中断，并清除中断位
                //wt.isInterrupted() 测试wt线程是否被中断，不清楚状态位，直接返回是否被中断
                //如果（当前线程池的状态已经成为 STOP 状态 || (当前线程已终端 && 线程池成为                          STOP）） && wt线程不是中断状态
                //其实这个判断可以这么理解:
                //1，如果当前线程池已经是 STOP状态，那么 
                //   (runStateAtLeast(ctl.get(), STOP) ||(Thread.interrupted() &&                        runStateAtLeast(ctl.get(), STOP))肯定会返回true,将会直接中断wt
                //2, 如果当前线程池不是STOP状态，例如：running 那么 runStateAtLeast会返回                        false, Thread.interrupted()会清除中断位，如果 wt还是中断状态，将会                       直接中断wt
                if ((runStateAtLeast(ctl.get(), STOP) ||
                        (Thread.interrupted() &&
                                runStateAtLeast(ctl.get(), STOP))) &&
                        !wt.isInterrupted())
                    wt.interrupt();
                try {
                    // 无实现
                    beforeExecute(wt, task);
                    Throwable thrown = null;
                    try {
                        //用户提交的线程启动
                        task.run();
                    } catch (RuntimeException x) {
                        thrown = x;
                        throw x;
                    } catch (Error x) {
                        thrown = x;
                        throw x;
                    } catch (Throwable x) {
                        thrown = x;
                        throw new Error(x);
                    } finally {
                        //无实现
                        afterExecute(task, thrown);
                    }
                } finally {
                    task = null;
                    w.completedTasks++;
                    w.unlock();
                }
            }
            completedAbruptly = false;
        } finally {
            //如果异常终止 completedAbruptly = true
            processWorkerExit(w, completedAbruptly);
        }
    }

```

### getTask()

```java
private Runnable getTask() {
    boolean timedOut = false; // Did the last poll() time out?
    //TODO
    for (; ; ) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.
        //在必要的条件下检查任务队列是否为empty
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            decrementWorkerCount();
            return null;
        }

        int wc = workerCountOf(c);

        // Are workers subject to culling?
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;
        // 减少线程池中线程数量，返回null
        if ((wc > maximumPoolSize || (timed && timedOut))
                && (wc > 1 || workQueue.isEmpty())) {
            if (compareAndDecrementWorkerCount(c))
                return null;
            continue;
        }

        try {
            //TODO 获取任务并返回，如果是定时等待任务，则使用poll方法，否则使用take方法
            Runnable r = timed ?
                    workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                    workQueue.take();
            if (r != null)
                return r;
            timedOut = true;
        } catch (InterruptedException retry) {
            timedOut = false;
        }
    }
}
```

​   

###  getActiveCount（） 方法

```java
public int getActiveCount() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        int n = 0;
        //只是判断工作线程是否上锁
        for (ThreadPoolExecutorBak.Worker w : workers)
            if (w.isLocked())
                ++n;
        return n;
    } finally {
        mainLock.unlock();
    }
}
 public boolean isLocked() {
       return isHeldExclusively();
 }
 protected boolean isHeldExclusively() {
       return getState() != 0;
 public void unlock() {
            release(1);
 }
 protected boolean tryRelease(int unused) {
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
  }
 public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
 }
```

​   之前一直在疑惑 ThreadPoolExecutor.getActiveCount()为什么不准确，在work开始运行时，会执行一次unlock，也就是说会将 state 设置为0 ，而线程还未开始运行，所以当调用w.isLocked 时，只是判断 state != 0 ,那这是 isHeldExclusively 肯定返回false ,而  getActiveCount的值就不是最新，最准确的值。

### shutdown() 与 shutdownNow()

```java
     public void shutdown() {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            //检查安全权限
            checkShutdownAccess();
            //更改线程池状态
            advanceRunState(SHUTDOWN);
            //这里只是尝试中断线程
            interruptIdleWorkers();
            onShutdown(); // hook for ScheduledThreakPoolExecutorBak
        } finally {
            mainLock.unlock();
        }
        tryTerminate();
    }
    private void advanceRunState(int targetState) {
        for (; ; ) {
            int c = ctl.get();
            //runStateAtLeast c >= targetState
            //如果线程池已经达到 targetState 状态，那么直接返回，如果还没达到，将c 改为 targetState状态
            if (runStateAtLeast(c, targetState) ||
                    ctl.compareAndSet(c, ctlOf(targetState, workerCountOf(c))))
                break;
        }
    }



//shutdownNow
   public List<Runnable> shutdownNow() {
        List<Runnable> tasks;
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            checkShutdownAccess();
            //和 shutdown() 处理方式一样，传参不一样而已
            advanceRunState(STOP);
            //中断所有的 work线程
            interruptWorkers();
            //取出阻塞队列中的work
            tasks = drainQueue();
        } finally {
            mainLock.unlock();
        }
        tryTerminate();
        return tasks;
  }
  //中断线程
  private void interruptWorkers() {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            for (ThreadPoolExecutorBak.Worker w : workers)
                w.interruptIfStarted();
        } finally {
            mainLock.unlock();
        }
  }
 // 中断线程，无论线程是否处于空闲状态
  void interruptIfStarted() {
            Thread t;
            if (getState() >= 0 && (t = thread) != null && !t.isInterrupted()) {
                try {
                    t.interrupt();
                } catch (SecurityException ignore) {
                }
            }
        }
  //拿出阻塞队列中的work  
   private List<Runnable> drainQueue() {
        BlockingQueue<Runnable> q = workQueue;
        ArrayList<Runnable> taskList = new ArrayList<Runnable>();
        // drainTo():
        // 一次性从BlockingQueue获取所有可用的数据对象（还可以指定获取数据的个数），
        //通过该方法，可以提升获取数据效率；不需要多次分批加锁或释放锁。
        q.drainTo(taskList);
        //如果还有接着从 workQueue 中拿出
        if (!q.isEmpty()) {
            for (Runnable r : q.toArray(new Runnable[0])) {
                if (q.remove(r))
                    taskList.add(r);
            }
        }
        return taskList;
    }
```

​   上面程序的解释，可以明显看出shutdownNow 和 shutdown的区别。shutdown只会将线程池状态修改，也会中断线程，但是会的等所有正在运行的线程运行完成后中断但；shutdownNow 会直接结束正在运行的线程和清除队列中的线程。中断方法接着往下看。

### tryTerminate()

```java 
// 尝试终止函数
final void tryTerminate() {
    for (; ; ) {
        // 以下状态直接返回：
        // 1.线程池还处于RUNNING状态
        // 2.SHUTDOWN状态但是任务队列非空
        // 3.runState >= TIDYING 线程池已经停止了或在停止了
        int c = ctl.get();
        if (isRunning(c) ||
                runStateAtLeast(c, TIDYING) ||
                (runStateOf(c) == SHUTDOWN && !workQueue.isEmpty()))
            return;
        //  workerCount不为0 则还不能停止线程池,而且这时线程都处于空闲等待的状态
        //  需要中断让线程“醒”过来，醒过来的线程才能继续处理shutdown的信号。
        if (workerCountOf(c) != 0) { // Eligible to terminate
            //runWoker方法中w.unlock就是为了可以被中断,getTask方法也处理了中断。
            //ONLY_ONE:这里只需要中断1个线程去处理shutdown信号就可以了。
            //这里问什么传 ONLY_ONE ？
            interruptIdleWorkers(ONLY_ONE);
            return;
        }

        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            //进入TIDYING状态
            if (ctl.compareAndSet(c, ctlOf(TIDYING, 0))) {
                try {
                    terminated();
                } finally {
                    ctl.set(ctlOf(TERMINATED, 0));
                    //继续awaitTermination
                    termination.signalAll();
                }
                return;
            }
        } finally {
            mainLock.unlock();
        }
        // else retry on failed CAS
    }
}
```

​   tryTerminate这个方法会在多处调用，比如processWorkerExit、addWorkerFailed、shutdown、shutdownNow、remove、purge 中调用，为什么会多处调用？ 在这个方法中调用了 interruptIdleWorkers方法，请接着往下看。

### interruptIdleWorkers(boolean onlyOne) 和 interruptIdleWorkers()

```java
//shutdown方法就是调用该下面的方法，去中断空闲线程，而不是中断所有线程
private void interruptIdleWorkers() {
    interruptIdleWorkers(false);
} 
private void interruptIdleWorkers(boolean onlyOne) {
        // 在这里为什么要获取mainLock锁，因为workers是HashSet 是线程不安全的，
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            for (ThreadPoolExecutorBak.Worker w : workers) {
                Thread t = w.thread;
                // 由于Work实现的是AQS是不可重入的，所以只能等正在运行的线程主动释放锁
                // 如果线程可以被中断 并且 可以获取锁 ，那么就执行中断
                if (!t.isInterrupted() && w.tryLock()) {
                    try {
                        t.interrupt();
                    } catch (SecurityException ignore) {
                    } finally {
                        w.unlock();
                    }
                }
                if (onlyOne)
                    break;
            }
        } finally {
            mainLock.unlock();
        }
    }
```

### processWorkerExit(ThreadPoolExecutorBak.Worker w, boolean completedAbruptly) 方法

```java 
// 异常情况下 completedAbruptly = true
private void processWorkerExit(ThreadPoolExecutorBak.Worker w, boolean completedAbruptly) {
    // completedAbruptly true 线程数 -1
    if (completedAbruptly) // If abrupt, then workerCount wasn't adjusted
        decrementWorkerCount();

    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        // 统计完成的work数量
        completedTaskCount += w.completedTasks;
        // 移除work
        workers.remove(w);
    } finally {
        mainLock.unlock();
    }
    //尝试终止线程池，
    tryTerminate();

    int c = ctl.get();
    if (runStateLessThan(c, STOP)) {
        // 出现异常 completedAbruptly = true / 正常情况下 completedAbruptly = false
        if (!completedAbruptly) {
              //如果允许核心线程超时，队列也不为空，相当于不会建立新的work去执行队列中的自定义线程，而是用当前的这些线程去执行，
                //如果不允许核心线程超时，当前线程池数量大于核心线程数 也会直接返回，也不会创建新的work去执行队列中的自定义线程
            int min = allowCoreThreadTimeOut ? 0 : corePoolSize;
            if (min == 0 && !workQueue.isEmpty())
                min = 1;
            if (workerCountOf(c) >= min)
                return; // replacement not needed
        }
        // 如果出现异常，将会新建一个 work ,去执行阻塞队列中的 work
        addWorker(null, false);
    }
}
```

​   processWorkerExit方法是在runWork方法中的 finally 中调用，也就是说，执行线程不管结果如何都会调用该方法，该方法就是统计线程执行完成数量、维护works、尝试终止线程池、以及如果线程执行异常，会创建工作线程去执行队列中的用户自定义线程，其实也可以认为是“收尾工作”

​   为什么在shutdown 和 shutdownNow中都会采用不同的方式中断线程，因为在这两个方法中，已经将线程池的状态改SHUTDOWN，而这时如果workQueue为空，那肯定有线程在getTask方法中阻塞，而用户已经无法提交线程到线程池，这时线程池就无法关闭。

​   这也可以解释shutdown,shutdownNow 的不同，因为 shutdownNow 不会关心线程处于什么状态就直接中断，而shutdown会等所有线程执行完成而再进行中断。

## 附录 ##

### 1. CAS 

锁机制存在以下问题：

（1）在多线程竞争下，加锁、释放锁会导致比较多的上下文切换和调度延时，引起性能问题。

（2）一个线程持有锁会导致其它所有需要此锁的线程挂起。

（3）如果一个优先级高的线程等待一个优先级低的线程释放锁会导致优先级倒置，引起性能风险。

​   **独占锁是一种悲观锁，synchronized就是一种独占锁，会导致其它所有需要锁的线程挂起，等待持有锁的线程释放锁。而另一个更加有效的锁就是乐观锁。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止**

​   **乐观锁用的机制就是CAS**，**Compare and Swap**

​   CAS有3个操作数，内存值V，旧的预期值A，要修改的新值B。当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做。

​        在Java中可以使用锁和循环CAS来进行原子操作，自旋CAS的基本思路就是循环进行CAS操作，直到成功为止。在JDK1.5之后，JDK的并发包里提供了一些类来支持原子操作，如AtomicBoolean,AtomicInteger,AtomicLong都是用原子的方式来更新指定类型的值。例如AtomicInteger的用法如下：

```java
 AtomicInteger atomicInteger=new AtomicInteger(0);
 int i=atomicInteger.get();
 atomicInteger.compareAndSet(i,i++);
```

**CAS还存在以下三个问题：**

（1）ABA问题。如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时发现它的值没有发生变        化，但实际上却发生了。ABA的解决思路就是使用版本号，在变量前面追加版本号，那么A——B——A 就变成了1A——2B——3A。 
（2）循环时间长开销大 

（3）只能保证一个共性变量的原子操作。也就是多个共享变量操作时，循环CAS就无法保证操作的原子性了。但从JDK1.5开始，JDK提供了AtomicReference类来保证引用对象之间的原子性，就可以把多个变量放在一个对象里来进行CAS操作。