### 创建线程的三种方式
##### 继承Thread类创建线程类
1. 定义Thread的子类，并重写该类的run方法，该run方法的方法体就代表了线程要完成的任务。因此把run方法称为执行体
2. 创建Thread子类的实例，即创建了线程对象
3. 调用线程对象的start方法来启动该线程
```
public class MyThread extends Thread{
    @Override
    public void run() {
        super.run();
        System.out.println(getName()+" 线程已执行 =================== ");
    }
}
    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + "  : " + i);
            if (i == 20) {
                new MyThread().start();
                new MyThread().start();
            }
        }
    }
```

##### 通过Runnable接口创建线程类
1. 定义Runnable接口的实现类，并重写该接口的run方法
2. 创建Runnable实现类的实例，并依次实例作为Thread的target来创建Thread对象，该Thread对象才是真正的线程对象
3. 调用线程对象的start方法启动线程
```
 public static class MyRunnable implements Runnable{
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() +"  ===== 执行");
        }
    }

public static void main(String[] args) {
        MyRunnable runnable = new MyRunnable();
        new Thread(runnable).start();
        new Thread(runnable).start();
    }
```

##### 通过Callable和Future创建线程
1. 创建Callable接口的实现类，并实现call方法，该方法将作为线程执行体，并且有返回值
2. 创建Callable实现类的实例，使用FutureTask类来包装Callable对象，该FutureTask对象封装了该Callable对象call方法的返回值
3.使用FutureTask对象作为Thread对象的target创建并开启线程
4. FutureTask对象的get方法来获得子线程执行结束后的返回值，调用get方法会阻塞线程
```
 public static class MyCallable implements Callable<String> {

        @Override
        public String call() {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return Thread.currentThread().getName() +" 执行完毕";
        }
    }
    public static void main(String[] args) {
        MyCallable callable = new MyCallable();
        FutureTask<String> futureTask = new FutureTask<>(callable);
        new Thread(futureTask,"有返回值的子线程").start();

        try {
            System.out.println("子线程的返回值 = "  + futureTask.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
```

##### 三种方式对比
采用Runnable、Callable接口的方式创建多线程时：
优势：
1. 线程类只是实现了Runnable接口或Callable接口，还可以继承其他类
2. 多线程可以共享一个target对象，所以非常适合多个线程处理同一份资源的情况，从而可以将CPU、代码和数据分开，形成清晰的模型，较好的体现了面向对象的思想
劣势：变成稍微复杂，如果要访问当前线程，则必须使用Thread.currentThread()方法

使用继承Thread类的方式创建多线程时
优势是：编写简单，如果要访问当前线程，则无需使用Thread.currentThread()方法，直接使用this即可获得当前线程
劣势是：线程类已经继承了Thread类，所以不能再继承其他类

### 线程池
作用：
1. 降低系统资源消耗，通过重用已存在的线程，降低线程创建和销毁的操作
2. 提高系统响应速度，当有任务到达时，无需等待新线程的创建便能立即执行
3. 方便线程并发数的管控，线程若是无限制的创建，不仅会额外消耗大量系统资源，更是占用过多资源而阻塞系统或者oom等状况，从而降低系统的稳定性。线程池能有效管控线程，统一分配、调优，提供资源使用率
4. 更强大的功能，线程池提供了定时、定期以及可控线程数等功能，使用方便简单

##### ThreadPoolExecutor
构造方法中个参数的含义：
```
   
    public static class ThreadPool extends ThreadPoolExecutor{

        /**
         *
         * @param corePoolSize  核心线程数
         * @param maximumPoolSize  最大线程数
         * @param keepAliveTime   空闲线程存活时间 allowCoreThreadTimeOut属性为true是，空闲核心线程也会被回收
         * @param timeUnit      时间单位
         * @param blockingQueue 等待执行任务的阻塞队列
         * @param threadFactory    创建线程的工厂类
         * @param rejectedExecutionHandler   当任务队列已满并且线程池中的活动线程已经达到所限定的最大值或者是无法成功执行任务，
         *                                   这时候ThreadPoolExecutor会调用RejectedExecutionHandler中的rejectedExecution方法。
         *                                   在线程池中它默认是AbortPolicy，在无法处理新任务时抛出RejectedExecutionException异常。
         *                                   一共有四个可选值：CallerRunsPolicy：只用调用者所在线程来运行任务。
         *                                   AbortPolicy：直接抛出RejectedExecutionException异常。
         *                                   DiscardPolicy：丢弃掉该任务，不进行处理。
         *                                   DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前任务。
         *
         *
         */
        public ThreadPool(int corePoolSize
                , int maximumPoolSize
                , long keepAliveTime
                , TimeUnit timeUnit
                , BlockingQueue<Runnable> blockingQueue
                , ThreadFactory threadFactory
                , RejectedExecutionHandler rejectedExecutionHandler) {
            super(corePoolSize
                    , maximumPoolSize
                    , keepAliveTime
                    , timeUnit
                    , blockingQueue
                    , threadFactory
                    , rejectedExecutionHandler);
        }
    }
```

##### ThreadPoolExecutor的使用方式
```

    public static void main(String[] args) {
       ThreadPool pool = new ThreadPool(5,10,30,TimeUnit.SECONDS,new LinkedBlockingDeque<Runnable>());

       //execute方式，无返回值，无法判断任务是否以执行完毕
       pool.execute(new Runnable() {
           @Override
           public void run() {

           }
       });

       // submit 通过 future.get()获取返回值,该方法会阻塞直到任务完成
      Future<String> future= pool.submit(new Callable<String>() {
          @Override
          public String call() throws Exception {
              return "submit方式 的返回值";
          }
      }
      );
        try {
            String result = future.get();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
```

##### 线程池执行流程
1. 如果在线程池中的线程数量没有达到核心的线程数量，这时候就会启动一个核心线程来执行任务
2. 如果线程池中的线程数量已经超过核心线程数，这个时候任务就会被插入到任务队列中排队等待执行
3. 由于任务队列已满，无法将任务插入到任务队列中，这个时候如果线程池中的线程数量没有达到线程池所设定的最大值，那么这个时候就会立即启动一个非核心线程来执行任务
4. 如果线程池中的数量达到了所规定的最大值，那么就会拒绝执行此任务，这时候就会调用RejectedExecutionHandler中的rejectedExecution方法来通知调用者

### 四种线程池类
##### newFixedThreadPool
1. 线程数量固定的线程池
2. 最大线程数就是我们设置的核心线程数
3. 核心线程不回被回收，直到关闭线程池
4. 可以快速响应外界请求
5. 不存在超时机制
6. 任务队列的大小无限制，使用LinkedBlockingQueue

##### newCachedThreadPool
1. 没有核心线程
2. 线程池中的最大线程数为int最大值
3. 空闲线程超时时间为60s
4. 使用SynchronousQueue，没有任务队列
5. 有任务就会直接创建一个新线程执行任务

##### newScheduledThreadPool
1. 固定核心线程
2. 最大线程数为int最大值
3. 超时时间为0s，即线程空闲会立刻被回收
4. 可定时或者周期性执行任务

##### newSingleThreadExecutor
1. 只有一个核心线程
2. 最大线程数为1
3. 在任务队列中依次等候执行
4. 不需要处理线程同步问题

### 死锁
一般来说，要出现死锁问题需要满足以下条件：
1. 互斥条件：一个资源每次只能被一个线程使用
2. 请求与保持条件： 一个线程因请求资源而阻塞时，对已获得的资源保持不放
3. 不剥夺条件：线程已获得的资源，在为使用完成之前，不能强行剥夺
4. 循环等待条件：若干线程之间形成一种头尾相接的循环等待资源关系

##### 静态的锁顺序死锁
a和b两个方法都需要获得A锁和B锁。一个线程执行a方法且已获得了A锁，在等待B锁；另一个线程执行了b方法且已经获得了B锁，在等待A锁。这种状态，就是发生了静态的锁顺序死锁
```
//可能发生静态锁顺序死锁的代码
class StaticLockOrderDeadLock {
    private final Object lockA = new Object();
    private final Object lockB = new Object();

    public void a() {
        synchronized (lockA) {
            synchronized (lockB) {
                System.out.println("function a");
            }
        }
    }

    public void b() {
        synchronized (lockB) {
            synchronized (lockA) {
                System.out.println("function b");
            }
        }
    }
}
```
**解决方法：所有需要多个锁的线程，都要以相同的顺序来获得锁**
```
//正确的代码
class StaticLockOrderDeadLock {
    private final Object lockA = new Object();
    private final Object lockB = new Object();

    public void a() {
        synchronized (lockA) {
            synchronized (lockB) {
                System.out.println("function a");
            }
        }
    }

    public void b() {
        synchronized (lockA) {
            synchronized (lockB) {
                System.out.println("function b");
            }
        }
    }
}
```

##### 动态的锁顺序死锁
动态的锁顺序死锁是指两个线程调用同一个方法时，传入的参数颠倒造成的死锁。如下代码，一个线程调用了transferMoney方法并传入参数accountA,accountB；另一个线程调用了transferMoney方法并传入参数accountB,accountA。此时就可能发生在静态的锁顺序死锁中存在的问题，即：第一个线程获得了accountA锁并等待accountB锁，第二个线程获得了accountB锁并等待accountA锁。
```
//可能发生动态锁顺序死锁的代码
class DynamicLockOrderDeadLock {
    public void transefMoney(Account fromAccount, Account toAccount, Double amount) {
        synchronized (fromAccount) {
            synchronized (toAccount) {
                //...
                fromAccount.minus(amount);
                toAccount.add(amount);
                //...
            }
        }
    }
}
```
**动态的锁顺序死锁解决方案如下：使用System.identifyHashCode来定义锁的顺序。确保所有的线程都以相同的顺序获得锁。**
```
//正确的代码
class DynamicLockOrderDeadLock {
    private final Object myLock = new Object();

    public void transefMoney(final Account fromAccount, final Account toAccount, final Double amount) {
        class Helper {
            public void transfer() {
                //...
                fromAccount.minus(amount);
                toAccount.add(amount);
                //...
            }
        }
        int fromHash = System.identityHashCode(fromAccount);
        int toHash = System.identityHashCode(toAccount);

        if (fromHash < toHash) {
            synchronized (fromAccount) {
                synchronized (toAccount) {
                    new Helper().transfer();
                }
            }
        } else if (fromHash > toHash) {
            synchronized (toAccount) {
                synchronized (fromAccount) {
                    new Helper().transfer();
                }
            }
        } else {
            synchronized (myLock) {
                synchronized (fromAccount) {
                    synchronized (toAccount) {
                        new Helper().transfer();
                    }
                }
            }
        }

    }
}
```

##### 协作对象之间发生的死锁
有时，死锁并不会那么明显，比如两个相互协作的类之间的死锁，比如下面的代码：一个线程调用了Taxi对象的setLocation方法，另一个线程调用了Dispatcher对象的getImage方法。此时可能会发生，第一个线程持有Taxi对象锁并等待Dispatcher对象锁，另一个线程持有Dispatcher对象锁并等待Taxi对象锁。
```
//可能发生死锁
class Taxi {
    private Point location, destination;
    private final Dispatcher dispatcher;

    public Taxi(Dispatcher dispatcher) {
        this.dispatcher = dispatcher;
    }

    public synchronized Point getLocation() {
        return location;
    }

    public synchronized void setLocation(Point location) {
        this.location = location;
        if (location.equals(destination))
            dispatcher.notifyAvailable(this);//外部调用方法，可能等待Dispatcher对象锁
    }
}

class Dispatcher {
    private final Set<Taxi> taxis;
    private final Set<Taxi> availableTaxis;

    public Dispatcher() {
        taxis = new HashSet<Taxi>();
        availableTaxis = new HashSet<Taxi>();
    }

    public synchronized void notifyAvailable(Taxi taxi) {
        availableTaxis.add(taxi);
    }

    public synchronized Image getImage() {
        Image image = new Image();
        for (Taxi t : taxis)
            image.drawMarker(t.getLocation());//外部调用方法，可能等待Taxi对象锁
        return image;
    }
}
```
上面的代码中， 我们在持有锁的情况下调用了外部的方法，这是非常危险的（可能发生死锁）。为了避免这种危险的情况发生， 我们使用开放调用。如果调用某个外部方法时不需要持有锁，我们称之为开放调用。

解决协作对象之间发生的死锁：需要使用开放调用，即避免在持有锁的情况下调用外部的方法。
```
//正确的代码
class Taxi {
    private Point location, destination;
    private final Dispatcher dispatcher;

    public Taxi(Dispatcher dispatcher) {
        this.dispatcher = dispatcher;
    }

    public synchronized Point getLocation() {
        return location;
    }

    public void setLocation(Point location) {
        boolean flag = false;
        synchronized (this) {
            this.location = location;
            flag = location.equals(destination);
        }
        if (flag)
            dispatcher.notifyAvailable(this);//使用开放调用
    }
}

class Dispatcher {
    private final Set<Taxi> taxis;
    private final Set<Taxi> availableTaxis;

    public Dispatcher() {
        taxis = new HashSet<Taxi>();
        availableTaxis = new HashSet<Taxi>();
    }

    public synchronized void notifyAvailable(Taxi taxi) {
        availableTaxis.add(taxi);
    }

    public Image getImage() {
        Set<Taxi> copy;
        synchronized (this) {
            copy = new HashSet<Taxi>(taxis);
        }
        Image image = new Image();
        for (Taxi t : copy)
            image.drawMarker(t.getLocation());//使用开放调用
        return image;
    }
}
```

 ### synchronized和ReentrantLock
1. 实现同步的基础：Java中每个对象都可以作为锁。当线程试图访问同步代码时，必须先获得对象锁，退出或者抛出异常时必须释放锁
2.表现形式为：代码块同步和方法同步

##### 使用场景
1. 方法同步
```
public synchronized void get()
```
锁住的是对象，类的其中一个实例，当该对象(仅仅是这一个对象)在不同线程中执行这个同步方法时，线程之间会形成互斥。达到同步效果，但如果不同线程同时对该类的不同对象执行这个同步方法时，则线程之间不回形成互斥，因为它们拥有的是不同的锁
2. 代码块同步
```
synchronized(this){
}
```
描述同上
3. 方法同步2
```
public synchronized static void get()
```
锁住的是该类，当所有该类的对象在不同线程中调用这个static同步方法时，线程之间会形成互斥，达到同步效果
4. 代码块同步2
```
synchronized(A.class){}
```
描述同3
5. 代码块同步3
```
synchronized(o){}
```
这里的o可以是任何一个Object对象或者数组，并不一定是它本身对象或者类，谁拥有o这个锁，谁就能够操作该代码块

##### ReentrantLock锁
一个可重入的互斥锁，它具有与使用synchronized方法和语句所访问的隐式监视器锁相同的一些基本行为和语以，但功能更强大
1. Lock接口
Lock，锁对象。在Java中锁是用来控制多个线程访问共享资源的方式，一般来说，一个锁能够防止多个线程同时访问共享资源（但有的锁可以允许多个线程并发访问共享资源，比如读写锁，后面我们会分析）。在Lock接口出现之前，Java程序是靠synchronized关键字（后面分析）实现锁功能的，而JAVA SE5.0之后并发包中新增了Lock接口用来实现锁的功能，它提供了与synchronized关键字类似的同步功能，只是在使用时需要显式地获取和释放锁，缺点就是缺少像synchronized那样隐式获取释放锁的便捷性，但是却拥有了锁获取与释放的可操作性，可中断的获取锁以及超时获取锁等多种synchronized关键字所不具备的同步特性。

 Lock接口的主要方法：

    void lock(): 执行此方法时，如果锁处于空闲状态，当前线程将获取到锁。相反，如果锁已经被其他线程持有，将禁用当前线程，直到当前线程获取到锁。 
    boolean tryLock()： 如果锁可用，则获取锁，并立即返回true，否则返回false. 该方法和lock()的区别在于，tryLock()只是"试图"获取锁，如果锁不可用，不会导致当前线程被禁用，当前线程仍然继续往下执行代码。而lock()方法则是一定要获取到锁，如果锁不可用，就一直等待，在未获得锁之前,当前线程并不继续向下执行. 通常采用如下的代码形式调用tryLock()方法： 
    void unlock()： 执行此方法时，当前线程将释放持有的锁. 锁只能由持有者释放，如果线程并不持有锁，却执行该方法，可能导致异常的发生. 
    Condition newCondition()： 条件对象，获取等待通知组件。该组件和当前的锁绑定，当前线程只有获取了锁，才能调用该组件的await()方法，而调用后，当前线程将缩放锁。

2.ReentrantLock的使用
```
ReentrantLock lock = new ReentrantLock(); //参数默认false，不公平锁
.....................
lock.lock(); //如果被其它资源锁定，会在此等待锁释放，达到暂停的效果
try {
    //操作
} finally {
    lock.unlock();  //释放锁
}
```

##### 重入锁
当一个线程得到一个对象后，再次请求该对象锁时是可以再次得到该对象的锁的
具体概念就是：自己可以再次获取自己的内部锁
Java里面的内置锁(synchronized)和lock(ReentrantLock)都是可重入的

##### 公平锁
CPU在调度线程的时候是在等待队列里随机挑选一个线程，由于这种随机性所以是无法保证线程先到先得的(synchronized控制的锁就是这种非公平锁)。但是这样就会产生饥饿现象，即有些线程(优先级较低的线程)可能永远也无法获取CPU的执行权，优先级高的线程会不断的抢占它的资源。那么如何解决饥饿问题呢？这就需要公平锁了，公平锁可以保证线程按照时间的先后顺序执行，避免饥饿线程的产生。但公平锁的效率比较低，因为要实现顺序执行，需要维护一个有序队列

ReentrantLock便是一种公平锁，通过在构造方法中传入true就是公平锁，传入false就是非公平锁。

###### synchronized和ReentrantLock的比较
- 区别：
1. Lock是一个接口，而synchronized是Java中的关键字，synchronized是内置的语言实现
2. synchronized在发生异常时，会自动释放线程占有的锁，因此不回导致死锁现象发生；而Lock在发生异常时，如果没有主动通过unLock()去释放锁，则很可能造成死锁现象，因此使用Lock时需要在finally块中释放锁
3. Lock可以让等待锁的线程响应中断，而synchronized却不行，使用synchronized时，等待的线程会一直等待下去，不能够响应中断
4. 通过Lock可以知道有没有成功获取锁，二synchronized却无法办到
5. Lock可以提高多个线程进行读操作的效率

##### 两者在锁的相关概念上区别：

1)可中断锁

顾名思义，就是可以响应中断的锁。

在Java中，synchronized就不是可中断锁，而Lock是可中断锁。如果某一线程A正在执行锁中的代码，另一线程B正在等待获取该锁，可能由于等待时间过长，线程B不想等待了，想先处理其他事情，我们可以让它中断自己或者在别的线程中中断它，这种就是可中断锁。

lockInterruptibly()的用法体现了Lock的可中断性。

2)公平锁

公平锁即尽量以请求锁的顺序来获取锁。比如同是有多个线程在等待一个锁，当这个锁被释放时，等待时间最久的线程（最先请求的线程）会获得该锁（并不是绝对的，大体上是这种顺序），这种就是公平锁。

非公平锁即无法保证锁的获取是按照请求锁的顺序进行的。这样就可能导致某个或者一些线程永远获取不到锁。

在Java中，synchronized就是非公平锁，它无法保证等待的线程获取锁的顺序。ReentrantLock可以设置成公平锁。

3)读写锁

读写锁将对一个资源（比如文件）的访问分成了2个锁，一个读锁和一个写锁。

正因为有了读写锁，才使得多个线程之间的读操作可以并发进行，不需要同步，而写操作需要同步进行，提高了效率。

ReadWriteLock就是读写锁，它是一个接口，ReentrantReadWriteLock实现了这个接口。

可以通过readLock()获取读锁，通过writeLock()获取写锁。

4)绑定多个条件

一个ReentrantLock对象可以同时绑定多个Condition对象，而在synchronized中，锁对象的wait()和notify()或notifyAll()方法可以实现一个隐含的条件，如果要和多余一个条件关联的时候，就不得不额外地添加一个锁，而ReentrantLock则无须这么做，只需要多次调用new Condition()方法即可。

3.性能比较

在性能上来说，如果竞争资源不激烈，两者的性能是差不多的，而 当竞争资源非常激烈时（即有大量线程同时竞争），此时ReentrantLock的性能要远远优于synchronized 。所以说，在具体使用时要根据适当情况选择。

在JDK1.5中，synchronized是性能低效的。因为这是一个重量级操作，它对性能最大的影响是阻塞的是实现，挂起线程和恢复线程的操作都需要转入内核态中完成，这些操作给系统的并发性带来了很大的压力。相比之下使用Java提供的ReentrankLock对象，性能更高一些。到了JDK1.6，发生了变化，对synchronize加入了很多优化措施，有自适应自旋，锁消除，锁粗化，轻量级锁，偏向锁等等。导致在JDK1.6上synchronize的性能并不比Lock差。官方也表示，他们也更支持synchronize，在未来的版本中还有优化余地，所以还是提倡在synchronized能实现需求的情况下，优先考虑使用synchronized来进行同步。