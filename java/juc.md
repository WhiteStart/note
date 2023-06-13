#  1.进程与线程

## 1.1 进程与线程的区别

- 进程基本上相互独立，而线程存在于进程内，是进程的一个子集
- 进程拥有==共享的资源==，如内存空间等，供其内部的线程共享
- 进程间通信较为复杂 
  - 同一台计算机的进程通信称为IPC(Intel-porcess communication)
  - 不同计算机之间的进程通信，需要通过网络，并遵守共同的协议
- 线程通信相对简单，因为他们共享进程内的内存，例如多个线程可以访问同一个共享变量
- 线程更轻量，线程上下文切换成本一般比进程上下文切换低



## 1.2 并行与并发

单核cpu下，线程串行执行。操作系统中的任务调度器将cpu的时间片(windows约为15ms)分给不同的线程使用，cpu切换时间很短人无法察觉，即==微观串行，宏观并行==。

一般将这种==线程轮流使用CPU的做法称为并发==（concurrent）

![截屏2023-02-20 上午10.48.49](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-02-20 上午10.48.49.png)

多核cpu下，每个核都可以调度运行线程，称为并行

![截屏2023-02-20 上午10.50.16](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-02-20 上午10.50.16.png)

![截屏2023-02-20 上午10.50.42](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-02-20 上午10.50.42.png)

- 并发(concurrent)是同一时间应对(dealing with)多件事情的能力
- 并行(parallel)是同一件动手做(doing)多件事情的能力



## 1.3 应用

从方法调用的角度来讲

- 需要等待结果返回，才能运行的是==同步==
- 不需要等待结果返回，就能继续运行的是==异步==

同步在多线程中使多个线程步调一致

### 1）设计

多线程可以让方法执行变为异步的。如读取磁盘文件时，假设读取操作花费了5秒，如果没有线程调度，那代码就得暂停

### 2）结论

- 在项目中，视频文件转换格式等操作费时，这是开一个新县城处理视频转换，避免阻塞主线程
- tomcat的异步servlet也是类似的项目，让用户线程处理耗时较长的操作，避免阻塞tomcat的工作线程
- ui程序中，开线程进行其他操作，避免阻塞ui程序



## 2.线程的创建

### 1）Thread

```java
Thread t1 = new Thread(()-> log.debug("thread"), "t1");
t1.start();        
```

### 2）Runnable

```java
Runnable r = () -> log.debug("runnable");
Thread t2 = new Thread(r,"t2");
t2.start();
```

- Thread合并了线程和其任务

- Runnable更容易与线程池等高级API配合
- Runnable让任务类脱离了Thread继承体系

### 3）FutureTask配合Thread

FutureTask能够接收Callable类型的参数，用来处理==有返回结果==的情况

```java
FutureTask<Integer> task = new FutureTask<>(new Callable<Integer>() {
    @Override
    public Integer call() throws Exception {
      int sum = 0;
      for (int i = 1; i <= 100; i++) {
        sum += i;
      }
      return sum;
    }
});

Thread t = new Thread(task, "futureTask");
t.start();

// 获取执行结果
log.debug("{}", task.get());
```

### 4）线程池

#### 线程池的状态

|            |                                                              |
| ---------- | ------------------------------------------------------------ |
| RUNNING    | 接受新任务，处理队列中的任务                                 |
| SHUTDOWN   | 调用shutdown(),关闭接受新任务 但会处理完队列中的任务         |
| STOP       | 调用shutdownnow(),中断所有任务                               |
| TIDYING    | 线程池没有线程运行后，自动变为此状态，并调用terminated()方法，该方法是空方法，由程序员自定义 |
| TERMINATED | terminated()方法执行完后，变为TERMINATED状态                 |



# 2.线程运行的原理

## $\textcolor{orange}{2.1 线程的生命周期}$

- NEW:初始态，线程被创建但还没有调用start()
- RUNNABLE:运行态，线程被调用了start()等待运行的状态
- BLOCKED:阻塞态，需要等待锁释放
- WAITING:等待状态，表示该线程需要等待其他线程做出一些特定动作(通知或终端)
- TIME_WAITING:超时等待，可以在指定的时间后自行返回
- TERMINATED:终止状态，表示该线程已运行完毕



## 2.2 上下文切换

因为一些原因导致 CPU 不再执行当前的线程，转而执行另一个线程的代码

- cpu时间片用完
- 垃圾回收
- 有更高优先级的线程需要运行
- 现成自己调用了sleep、yield、wait、join、park、synchronized、lock等方法

当Context Switch发生时，需要操作系统保存当前状态，并恢复另一个线程的状态，Java中对应的概念就是程序计数器，它的作用是记住下一条jvm指令执行的地址。



- 状态包括程序计数器、虚拟机栈中每个栈帧的信息，如局部变量、操作数栈、返回地址等
- Context Switch频繁发生会影响性能



## 2.3 栈与栈帧

Java Virtual Machine Stacks ( Java虚拟机栈)

线程启动后，虚拟机为其分配一块栈内存

- 每个栈由多个栈帧组成，对应着每次方法调用时所占用的内存
- 每个线程只能有一个活动栈帧，对应着当前正在执行的方法



## 2.4 常用方法

| 方法名           |        | 功能说明                                             | 注意                                                         |
| ---------------- | ------ | ---------------------------------------------------- | ------------------------------------------------------------ |
| start()          |        | 启动一个新县城，在新的线程中运行run()方法            | start()方法只是让线程进入就绪，里面代码不一定立刻运行(cpu时间片没分到)。每个线程对象的start方法只能调用一次，如果调用了多次会出现IllegalThreadStateException |
| run()            |        | 新线程启动后会调用的方法                             | 如果在构造Thread对象时传递了Runnable参数，则线程启动后会调用Runnable中的run方法，否则默认不执行任何操作。但可以创建Thread的子类对象，来覆盖默认行为。 |
| join()           |        | 等待线程运行结束                                     |                                                              |
| join(long n)     |        | 等待线程运行结束，最多等待n毫秒                      |                                                              |
| getId()          |        | 获取线程长整形的id                                   | id唯一                                                       |
| getName()        |        | 获取线程名                                           |                                                              |
| setName()        |        | 修改线程名                                           |                                                              |
| getPriority(int) |        | 修改线程优先级                                       | java中规定线程优先级是1-10的整数，较大的优先级能提高该线程被CPU调度的几率 |
| getState()       |        | 获取线程状态                                         | NEW、RUNNABLE、BLOCKED、WAITING、TIMED_WAITING、TERMINATED   |
| isInterrupted()  |        | 判断当前线程是否被打断                               | 不会清除 打断标记                                            |
| isAlive()        |        | 线程是否存活(还没有运行完毕)                         |                                                              |
| interrupt()      |        | 打断线程                                             | 如果被打断线程正在sleep、wait、join会导致被打断的线程抛出InterruptedException，并清除`打断标记`;如果打断的正在运行的线程，则会设置`打断标记`;park的线程被打断也会设置`打断标记` |
| interrupted()    | static | 判断当前线程是否被打断                               | 会清除打 断标记                                              |
| currentThread()  | static | 获取当前正在执行的线程                               |                                                              |
| sleep(long n)    | static | 让当前线程休眠n毫秒，休眠时让出cpu的时间片给其他线程 |                                                              |
| yield()          | static | 提示线程调度器让出当前线程对cpu的使用                |                                                              |



## 2.5 sleep 与 yield

### sleep

1. 调用sleep会让当前线程从 `Running` 进入 `Timed Waiting` 状态
2. 其他线程可以使用interrupt方法打断正在睡眠的线程，这时 sleep 会抛出 InterruptedException
3. 睡眠结束后的线程未必立刻得到执行
4. TimeUnit的 sleep 代替Thread的 sleep 获取更好的可读性

### yield

1. 调用yield()或让当前线程从 `Running` 进入 `Runnable` 就绪状态，然后调度执行其他线程
2. 具体的实现依赖于操作系统的任务调度器

### 线程优先级

- 线程优先级会提示(hint)调度器优先调度该线程，但它仅仅是一个提示，调度器可以忽略它
- 如果cpu比较忙，那么优先级高的线程会获得更多的时间片，但 `cpu`闲时，优先级几乎没用



## 									案例-防止CPU占用100%

**sleep实现**

在没有利用 cpu 来计算时，不要让while(true)空转浪费 cpu，这时可以使用yield或者sleep来让出cpu的使用权给其他程序

```java
class TestCpu{
	public static void main(String[] args){
			while(true){
				try{
          Thread.sleep(1);
        } catch(Exception e){
          e.printStackTrace();
        }
			}
	}
}
```

- 可以用wait或条件变量达到类似的效果
- 不同的是，后两种都需要`加锁`，并且需要相应的唤醒操作，一般适用于需要进行同步的场景
- sleep适用于`无锁同步`场景



## 2.6 join方法详解

### 为什么需要join

```java
static int r = 0;
private static void test1() throws InterruptedException {
    log.debug("开始");
    Thread t1 = new Thread(()->{
      try {
        log.debug("---开始---");
        TimeUnit.SECONDS.sleep(1);
        log.debug("---结束---");
        r = 10;
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    }, "t1");
    t1.start();
    // t1.join();
    log.debug("结果为:{}",r);
    log.debug("结束");
}
```

分析

- 主线程和线程t1是并行执行的，t1线程需要1秒之后才能算出r=10
- 而主线程一开始就打印r的结果，r=0

解决方法

- 用sleep行不行？为什么？
- join(),加在start()之后



### 应用之同步(案例1)

- 需要等待结果返回，才能运行的就是同步
- 不需要等待结果返回，就能继续运行的是异步 

```java
		static int r1 = 0;
    static int r2 = 0;

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(1);
                r1 = 10;
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        Thread t2 = new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(2);
                r2 = 20;
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        t1.start();
        t2.start();
        long start = System.currentTimeMillis();
        log.debug("join begin");
        t2.join();
        log.debug("t2 join end");
        t1.join();
        log.debug("t1 join end");
        long end = System.currentTimeMillis();
        log.debug("r1:{},r2:{}.cost:{}", r1, r2, end - start);
    }
```

```
14:36:09.885 [main] DEBUG com.example.multithreading.Test7 - join begin
14:36:11.887 [main] DEBUG com.example.multithreading.Test7 - t2 join end
14:36:11.888 [main] DEBUG com.example.multithreading.Test7 - t1 join end
14:36:11.889 [main] DEBUG com.example.multithreading.Test7 - r1:10,r2:20.cost:2005
```

### 限时同步

```java
Thread t1 = new Thread(() -> {
    try {
      TimeUnit.SECONDS.sleep(2);
      r1 = 10;
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
});

long start = System.currentTimeMillis();
t1.start();
t1.join(1500);
// t1.join(3000);
long end = System.currentTimeMillis();
log.debug("r1:{},cost:{}", r1, end - start);
```

```java
14:45:36.044 [main] DEBUG com.example.multithreading.Test7 - r1:0,r2:0,cost:1504
```

- 如果等待之间超过1500ms，就不等了，继续执行
- 如果等待时间不足3000ms，也会直接执行

## 2.7 interrupt方法详解

### 打断sleep，wait，join的线程

阻塞

- 打断sleep的线程，会清空打断状态

```java
Thread t1 = new Thread(() -> {
    log.debug("sleep...");
    try {
      TimeUnit.SECONDS.sleep(5);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
}, "t1");

t1.start();
Thread.sleep(1000);
log.debug("interrupt");
t1.interrupt();
log.debug("打断标记:{}", t1.isInterrupted());
```

```java
14:53:11.872 [t1] DEBUG com.example.multithreading.Test8 - sleep...
14:53:12.871 [main] DEBUG com.example.multithreading.Test8 - interrupt
14:53:12.872 [main] DEBUG com.example.multithreading.Test8 - 打断标记:false
java.lang.InterruptedException: sleep interrupted
	at java.base/java.lang.Thread.sleep(Native Method)
	at java.base/java.lang.Thread.sleep(Thread.java:337)
	at java.base/java.util.concurrent.TimeUnit.sleep(TimeUnit.java:446)
	at com.example.multithreading.Test8.lambda$main$0(Test8.java:13)
	at java.base/java.lang.Thread.run(Thread.java:833)

```

### 打断正常的线程

```java
Thread t1 = new Thread(() -> {
    while (true){
        boolean isInterrupted = Thread.currentThread().isInterrupted();
        if(isInterrupted){
            // do sth...
            break;
        }
    }
}, "t1");

t1.start();
log.debug("interrupt");
t1.interrupt();
log.debug("打断标记:{}", t1.isInterrupted());
```

- 打断标记，线程可以自主进行一些操作再结束，或者不结束

## $\textcolor{green}{设计模式-两阶段终止模式}$

```java
@Slf4j(topic = "Test9")
public class Test9 {
    public static void main(String[] args) throws InterruptedException {
        TwoPhaseTermination termination = new TwoPhaseTermination();
        termination.start();

        Thread.sleep(3000);
        termination.stop();
    }
}

@Slf4j(topic = "Test9")
class TwoPhaseTermination{
    private Thread monitor;

    // 启动监控线程
    public void start(){
        monitor = new Thread(()->{
            while (true){
                Thread current = Thread.currentThread();
                // catch中重新将interrupt()设置为true，符合条件进入判断
                if(current.isInterrupted()){
                    log.debug("料理后事");
                    break;
                }
                try {
                    TimeUnit.SECONDS.sleep(1);
                    log.debug("执行监控记录");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                    // sleep会清除打断标记，因此在这里重新设置打断标记
                    current.interrupt();
                }
            }
        });

        monitor.start();
    }

    // 停止监控线程
    public void stop(){
        monitor.interrupt();
    }
}
```

## 2.8 守护线程(Daemon Thread)

线程分为用户线程和守护线程。用户线程就是`普通线程`，守护线程则是JVM的后台线程。守护线程通常用于执行一些辅助性的任务，如垃圾回收、内存管理等，并不影响主线程的运行，在程序结束后自动关闭。

- 守护线程会在其他普通线程都停止运行后`自动关闭`
- 垃圾回收线程是一个`守护线程`
- Tomcat中的 `Acceptor` 和 `Poller` 线程都是守护线程，所以Tomcat接收到 shutdown命令后，不会等待他们处理完当前请求



## 2.9 五种状态

<font color=red size=4>从操作系统层面来讲</font>

![截屏2023-03-17 下午7.06.13](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-03-17 下午7.06.13.png)

- 【初始状态】仅是在语言层面创建了线程对象，还未与操作系统线程关联
- 【可运行状态】 (就绪状态)指该线程已经被创建(英语操作系统线程关联)，可以由 CPU 调度执行
- 【运行状态】指获取了 CPU 时间片运行中的状态
  - 当 CPU 时间片用完，会从【运行状态】转换至【可运行状态】，会导致线程的上下文切换
- 【阻塞状态】
  - 如果调用了阻塞API，如BIO读写文件，这时该线程不会用到 CPU，会导致线程上下文切换，进入【阻塞状态】
  - 等BIO操作完毕，会由操作系统唤醒阻塞的线程，转换至【可运行状态】
  - 与【可运行状态】的区别是，对【阻塞状态】的线程来说只要他们一直不唤醒，调度器就不会考虑他们
- 【终止状态】表示线程已经执行完毕，生命周期已结束，不会再转换为其他状态



## 2.10 六种状态

![截屏2023-03-17 下午10.29.50](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-03-17 下午10.29.50.png)

- NEW 线程刚被创建，但还没有调用start()方法
- RUNNABLE 当调用了start()方法之后，(JAVA API 层面的RUNNABLE状态涵盖了操作系统层面的【可运行状态】、【运行状态】、【阻塞状态】)，BIO导致的线程阻塞，在Java里无法区分，仍然认为可运行
- BLOCKED、WAITING、TIMED_WAITING 都是Java API层面对【阻塞状态】的细分
- TERMINATED 当前线程代码运行结束





## $\textcolor{red}{本章小结}$

- 线程创建
- 线程api：start、run、sleep、join、interrupt等
- 线程状态
- <font color=green>应用方面</font>
  - 异步调用：主线程执行期间，其他线程异步执行耗时操作
  - 提高效率：并行计算，缩短运算时间
  - 同步等待：join
  - 统筹规划：合理使用线程，得到最优效果
- <font color=blue>原理方面</font>
  - 线程运行流程：栈、栈帧、上下文切换、程序计数器
  - Thread两种创建方式的源码
- <font color=orange>设计模式</font>
  - 两阶段终止



# $\textcolor{red}{3.共享模型之管程}$

- 阻塞式解决方案：synchronized， Lock
- 非阻塞式的解决方案：原子变量



## 1.synchronized

- <font color=red>锁普通方法，等价与锁住this</font>

```java
public void increment() {
    synchronized (this) {
        count++;
    }
}

public synchronized void decrement() {
    count--;
}
```

- <font color=red>锁静态方法时，等价于锁住类</font>

```java
public static synchronized void test(){

}

public static void test2(){
    synchronized (Room.class){

    }
}
```

### 分析1

- 两个线程都锁的是n1对象，因此会阻塞

```java
synchronized void a(){
    try {
        Thread.sleep(2000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    log.debug("1");
}

synchronized void b(){
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    log.debug("2");
}

public static void main(String[] args) {
    Number n1 = new Number();
    new Thread(()->{
        log.debug("t1 begin");
        n1.a();
    }).start();

    new Thread(()->{
        log.debug("t2 begin");
        n1.b();
    }).start();
}
```

```java
22:33:43.416 [Thread-0] DEBUG com.example.multithreading.线程八锁.Number - t1 begin
22:33:43.416 [Thread-1] DEBUG com.example.multithreading.线程八锁.Number - t2 begin
22:33:45.423 [Thread-0] DEBUG com.example.multithreading.线程八锁.Number - 1
22:33:46.423 [Thread-1] DEBUG com.example.multithreading.线程八锁.Number - 2
```

### 分析2

- a()方法锁住的是n1对象，而d()方法锁住的是类对象，不互斥

```java
synchronized void a(){
    try {
        Thread.sleep(2000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    log.debug("1");
}

static synchronized void d(){
    try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    log.debug("4");
}

public static void main(String[] args) {
    Number n1 = new Number();
    new Thread(()->{
        log.debug("t1 begin");
        n1.a();
    }).start();

    new Thread(()->{
        log.debug("t4 begin");
        d();
    }).start();
}
```



## 2.变量的安全分析

### 成员变量和静态变量是否线程安全？

- 没有线程共享，线程安全
- 有线程共享
  - 只有读操作，线程安全
  - 有读写操作，则这段代码是临界区，需要考虑线程安全

### 局部变量是否线程安全？

- 安全
- <font color=red>局部变量引用的对象未必</font>
  - 没有逃离方法的作用范围，安全
  - 逃离方法的作用范围，不安全

```java
public class Threadsafe {
	  // 防止子类继承产生线程安全问题，可以使用final
    public void m1(int loop){
        List<String> list = new ArrayList<>();
        for (int i = 0; i < loop; i++) {
            m2(list);
            m3(list);
        }
    }

    // 修改成private可以提供安全
    public void m2(List<String> list){
        list.add("1");
    }

    // 修改成private可以提供安全
    public void m3(List<String> list){
        list.remove(0);
    }
}

class ThreadSafeSubClass extends Threadsafe{
    // 暴露
    // 重写了父类的public m3方法，因此对同一个list也有2个线程在操作，会产生线程安全问题
    @Override
    public void m3(List<String> list) {
        new Thread(()->{
            list.remove(0);
        }).start();
    }
}

class Test{
    public static void main(String[] args) {
        Threadsafe test = new Threadsafe();
        for (int i = 0; i < 2; i++) {
            new Thread(()->{
                test.m1(200);
            }, "thread" + (i+1)).start();
        }
    }
}

class Test2{
    public static void main(String[] args) {
        ThreadSafeSubClass test = new ThreadSafeSubClass();
        for (int i = 0; i < 2; i++) {
            new Thread(()->{
                test.m1(200);
            }, "thread" + (i+1)).start();
        }
    }
}
```

- <font color=red>访问修饰符private一定程度上可以防止线程不安全问题，避免了子类对它的继承</font>



## 3.常见的线程安全类

- String
- Integer
- StringBuffer
- Random
- Vector
- Hashtable
- java.util.concurrent包下的类

线程安全指的是，多个线程调用他们同一个实例的某个方法时，是线程安全的。

- 他们的每个方法是原子的
- <font color=red>但多个方法的组合不是原子的</font>



## 4.实例分析

### 例一

```java
public class UserServiceImpl implements UserService{
    
    private int count = 0;
    
    @Override
    public void incr() {
        // 线程不安全
        count++;
    }
}

public class MyServlet extends HttpServlet {

    private UserService userService = new UserServiceImpl();

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        userService.incr();
        super.doGet(req, resp);
    }
}
```

### 例二

- 此时应该使用@Around将start改为局部变量，保证线程安全

```java
@Aspect
@Component
public class MyAspect {
		private long start = 0L;
  
    @Before("exectuion(* *(..))"))
    public void before(){
        start = System.nanoTime();
    }
  
    @After("execution(* *(..))")
    public void after(){
        long end = System.nanoTime();
        System.out.println("cost time..." + (end - start));
    }
}
```

### 例三

```java
public class MyServlet extends HttpServlet {

    private UserService userService = new UserServiceImpl();

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        userService.incr();
        super.doGet(req, resp);
    }
}

public class UserServiceImpl implements UserService{
    // UserDaoImpl方法中有一个成员变量，可能会被修改，因此线程不安全
    // 如果UserDaoImpl中没有任何成员变量，那么可以线程安全
    UserDao userDao = new UserDaoImpl();
    public void update(){
        // 如果是局部变量 线程安全
        UserDao userDao = new UserDaoImpl();
    }
}

public class UserDaoImpl implements UserDao{
		private Connection = null;
  
    public void update(){
      	// do sth...
    }
}
```



## 5.Monitor 原理

### Java对象头

以32位虚拟机为例

- 普通对象

![截屏2023-04-06 下午4.10.48](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-04-06 下午4.10.48.png)

- 数组对象

![截屏2023-04-06 下午4.11.33](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-04-06 下午4.11.33.png)

- Mark Word

![截屏2023-04-06 下午4.50.03](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-04-06 下午4.50.03.png)

### Monitor锁

![0](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-04-06 下午4.50.22.png)

- 刚开始Monitor中Owner为null
- 当Thread-2执行synchronized(obj)就会将Monitor的所有者置为Thread-2，Monitor中只能有一个`Owner`
- 在Thread-2上锁的过程中，如果Thread-3，Thread-4，Thread-5也来执行synchronized(obj)，就会进入EntryList BLOKCED
- Thread-2执行完同步代码块内容后，唤醒`EmptyList`中等待的线程来竞争锁，竞争的时候是非公平的
- `WaitSet`中的线程是==之前获得过锁，但是不满足进入WAITING状态的线程==



## $\textcolor{orange}{6.synchronized原理进阶}$

### 1.轻量级锁

如果一个对象虽然有多线程访问，但多线程访问的时间是错开的(没有竞争)，那么可以使用轻量级锁来优化。

- 创建锁记录(Lock Record)对象，每个线程的栈帧都会包含一个锁记录的结构，内部可以存储锁定对象的Mark Word

![截屏2023-04-06 下午6.52.18](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-04-06 下午6.52.18.png)

- 让锁记录中Object reference指向锁对象，并<font color=red>尝试CAS替换Object的Mark Word，将Mark Word的值存入锁记录</font>

![截屏2023-04-06 下午6.54.16](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-04-06 下午6.54.16.png)

- 如果CAS替换成功，对象头中存储了锁记录地址和状态00，表示由该线程给对象加锁

![截屏2023-04-06 下午6.58.23](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-04-06 下午6.58.23.png)

### 2.锁升级

- 如果CAS失败
  - 如果是其他线程已经持有了该Object的轻量级锁，表明有竞争，进行锁升级
  - 如果是自己执行了synchronized锁重入，那么再添加一条Lock Record

![截屏2023-04-06 下午7.01.41](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-04-06 下午7.01.41.png)

- 当退出synchronized代码块(解锁)时，如果有取值为null的锁记录，表示有重入，这时重置锁记录，表示重入计数减一

![截屏2023-04-06 下午7.06.16](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-04-06 下午7.06.16.png)

- 当退出synchronized代码块(解锁时)，锁记录的值不为null，这时使用CAS将Mark Word的值恢复给对象头

  - 成功，则解锁成功
  - 失败，说明轻量级锁已升级为重量级锁，进入重量级锁的解锁流程

  ![截屏2023-04-06 下午10.07.11](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-04-06 下午10.07.11.png)

- Thread-0加了轻量级锁，这时Thread-1进入发现了已经是00(被加锁)，Thread-1加轻量级锁失败，进行锁升级
  - 为Object对象申请Monitor锁，让Object指向重量级锁的地址
  - 然后自己进入Monitor的EntryList BlOCKED

![截屏2023-04-06 下午10.45.05](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-04-06 下午10.45.05.png)

- 当Thread-0退出同步块解锁时，使用CAS将Mark Word的值恢复给对象头，失败。进入重量锁的解锁流程，按照Monitor地址找到Monitor对象，设置Owner为null，唤醒EntryList中BLOCKED线程

### 3.自旋优化

重量级锁竞争的时候，会进行自旋优化。如果当前线程自旋成功(即这时候持锁线程已经退出了同步块，释放了锁)，这时当前线程就可以避免阻塞。

![截屏2023-04-07 下午10.07.35](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-04-07 下午10.07.35.png)

如果自旋失败，也会阻塞

### 4.偏向锁

轻量级锁在没有竞争时(仅有自己一个线程)，每次重入仍然需要执行CAS操作。因此，当第一次CAS将线程ID设置到对象的Mark Word后，后续重入发现线程ID是自己的就表示没有竞争，不重新CAS。以后只要不发生竞争，这个对象就归该线程所有。

#### 偏向状态

| Mark Word(64 bits)                                           | State              |
| ------------------------------------------------------------ | ------------------ |
| unused: 25 \| hascode: 31 \|  unused: 1 \| age:4   \| biased_lock:0   \|  01 | Normal             |
| thread: 54   \| epoch:2        \|  unused: 1 \| age:4   \| biased_lock:1   \|  01 | Biased             |
| ptr_to_lock_record \| 62                                               \|  00 | Lightweight Locked |
| ptr_to_heavyweight_monitor:62                                \|  10 | Heavyweight Locked |
| \|   11                                                      | Marked for GC      |

一个对象创建时：

- 如果开启了偏向锁(默认)，那么对象创建后，markword值为0x05即最后3位为101，这时它的thread、epoch、age都是0
- 偏向锁默认是==延迟的==，不会在程序启动时立即生效，如果想避免延迟，可以加VM参数 `-XX:BiasedLockingStartupDelay=0`来禁用延迟
- 如果没有开启偏向锁，那么对象创建后，markword值为0x01即最后三位为001，这时它的hashcode、age都为0，第一次用到hashcode才会赋值

#### 偏向撤销

- 调用hashcode()

```
Cat c = new Cat();
c.hashcode() // 从Biased变成normal，因为Biased状态下没有空间存储31位的了
```

- 其他线程使用对象
- 调用wait/notify(只有重量级锁有)

#### 批量重偏向

如果对象被多个线程访问，但==没有竞争==，这时偏向了T1的对象仍有机会偏向T2，重偏向会重置对象的thread-id。

1. 偏向锁首先偏向了t1
2. 在t2中循环<font color=red>20</font>次后，重置偏向为t2

```java
public static void main(String[] args) {
        List<Object> list = new ArrayList<>();
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 30; i++) {
                Object instance = new Object();
                list.add(instance);
                synchronized (instance) {
                    log.debug("{}", instance);
                }
            }
            synchronized (list) {
                list.notify();
            }
        }, "t1");
        t1.start();

        new Thread(()->{
            synchronized (list){
                try {
                    list.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            log.debug("======>");
            for (int i = 0; i < 30; i++) {
                Object o = list.get(i);
                log.debug("{}", o);
            }
        }, "t2").start();
    }
```

#### 批量撤销

1. 当撤销偏向锁阈值超过<font color=red>40</font>次后，jvm会觉得确实偏向错了，会将整个类的所有对象都变为不可偏向的，新建的对象也是不可偏向的(001)

### 5.锁消除

编译器可以通过静态分析代码，找到那些不需要加锁的临界区域，并将其锁去除，从而提高程序的性能。



## 7.wait/notify

![截屏2023-04-08 下午4.21.51](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-04-08 下午4.21.51.png)

- Owner线程发现条件不满足，==调用wait方法，即可进入Waitset变为WAITING状态==
- BLOCKED和WAITING的线程都处于阻塞状态，不占用CPU时间
- BLOCKED线程会在Owner线程释放锁的时候环形
- WAITING线程会在Owner线程调用notify或notifyAll时环形，但唤醒后并不意味着立刻获得锁，仍然需要进入EntryList竞争

### API介绍

- obj.wait()让进入object监视器的线程到waitSet等待
- obj.notify()在object上正在waitSet等待的线程中挑选一个唤醒
- object.notifyAll()让object上正在waitSet等待的线程全部唤醒

<font color=red>线程之间进行写作的手段，都是Object对象的方法。必须获得对象的锁，才能调用这几个方法。</font>

```java
synchronized (lock) {
    try {
      lock.wait();
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
}
```

### API使用

- wait()与sleep()有什么区别？
  - sleep()是thread的方法，而wait是Object()方法
  - sleep()不需要强制和synchronized配合使用，但wait()需要
  - sleep()会持有锁，而wait()会释放锁

```java
synchronized(lock){
		while(条件不成立){
        lock.wait();
    }
    // do sth..
}

// 另一个线程
synchronized(lock){
    lock.notifyAll();
}
```

## $\textcolor{green}{同步模式之保护暂停}$

Guarded Suspension，用在一个线程等待另一个线程的执行结果

- 有一个结果需要从一个线程传递到另一个线程，让他们关联同一个GuardedObject
- 如果有结果不断从一个线程到另一个线程，那么可以使用消息队列
- JDK中，join的实现、future的视线，采用的是这个模式
- 因为要等待另一方的结果，因此归类到同步模式

![截屏2023-04-13 下午8.48.51](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-04-13 下午8.48.51.png)

```java
class GuardedObject {

    private Object response;

    public Object get(long timeout) {
        synchronized (this) {
            long begin = System.currentTimeMillis();
            long passedTime = 0;
            while (response == null) {
                long waitTime = timeout - passedTime;
                if(waitTime <= 0) {
                    break;
                }
                try {
                    this.wait(waitTime);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                passedTime = System.currentTimeMillis() - begin;
            }
        }
        return response;
    }

    public void complete(Object response) {
        synchronized (this) {
            this.response = response;
            this.notifyAll();
        }
    }
}
```

- join方法运用的就是保护暂停模式

![截屏2023-04-13 下午9.23.06](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-04-13 下午9.23.06.png)

- 保护暂停模式是一一对应的

![截屏2023-04-18 下午1.43.19](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-04-18 下午1.43.19.png)

```java
public class Test2 {
    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 3; i++) {
            new People().start();
        }
        Thread.sleep(1000);
        // 保护性暂停模式 必须是一一对应的关系
        for (Integer id : Mailboxes.getIds()) {
            new Postman(id, "内容" + id).start();
        }
    }
}

@Slf4j(topic = "c.people")
class People extends Thread {

    public void run() {
        GuardObject object = Mailboxes.create();
        log.debug("收信id:{}", object.getId());
        Object mail = object.get(5000);
        log.debug("收信id:{}, 内容:{}", object.getId(), mail);
    }
}

@Slf4j(topic = "c.postman")
class Postman extends Thread {
    private int id;
    private String mail;

    public Postman(int id, String mail) {
        this.id = id;
        this.mail = mail;
    }

    public void run() {
        GuardObject object = Mailboxes.getGuardObject(id);
        log.debug("送信id:{}, 内容:{}", id, mail);
        object.complete(mail);
    }
}

class Mailboxes {
    private static Map<Integer, GuardObject> box = new ConcurrentHashMap<>();

    private static int id = 1;

    public static synchronized int generateId() {
        return ++id;
    }

    public static GuardObject create() {
        GuardObject go = new GuardObject(generateId());
        box.put(go.getId(), go);
        return go;
    }

    public static Set<Integer> getIds() {
        return box.keySet();
    }

    public static GuardObject getGuardObject(int id) {
        return box.remove(id);
    }
}

class GuardObject {
    private int id;
    private Object response;

    public GuardObject(int id) {
        this.id = id;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public Object get(long timeout) {
        synchronized (this) {
            long begin = System.currentTimeMillis();
            long passedTime = 0;
            while (response == null) {
                long waitTime = timeout - passedTime;
                if (waitTime <= 0) {
                    break;
                }
                try {
                    this.wait(waitTime);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                passedTime = System.currentTimeMillis() - begin;
            }
        }
        return response;
    }

    public void complete(Object response) {
        synchronized (this) {
            this.response = response;
            this.notifyAll();
        }
    }
}
```

## $\textcolor{green}{异步模式之生产者/消费者}$

- 产生结果和消费结果的线程不需要一一对应
- 消费队列可以用来平衡生产和消费的线程资源
- 生产者仅负责产生数据
- 消费队列有容量限制，满不会再加入数据，空不会在消耗数据
- JDK中各种阻塞队列采用的就是生产/消费者模式

![截屏2023-04-18 下午2.08.33](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-04-18 下午2.08.33.png)

```java
@Slf4j(topic = "Test")
public class Test {
    public static void main(String[] args) {

        MessageQueue messageQueue = new MessageQueue(2);

        for (int i = 0; i < 3; i++) {
            int id = i;
            new Thread(() -> {
                messageQueue.put(new Message(id, "值" + id));
            }, "生产者" + i).start();
        }

        new Thread(() -> {
            while (true){
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                Message message = messageQueue.take();
                log.debug("获取消息{}",message);
            }

        }, "消费者").start();
    }
}

@Slf4j(topic = "MessageQueue")
class MessageQueue {

    private LinkedList<Message> list = new LinkedList<>();
    private int capacity;

    public MessageQueue(int capacity) {
        this.capacity = capacity;
    }

    // 获取消息
    public Message take() {
        synchronized (list) {
            while (list.isEmpty()) {
                try {
                    log.debug("队列为空，消费者等待");
                    list.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            Message message = list.removeFirst();
            list.notifyAll();
            return message;
        }
    }

    // 存入消息
    public void put(Message message) {
        synchronized (list) {
            while (list.size() == capacity) {
                try {
                    log.debug("队列已满，生产者等待");
                    list.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            list.addLast(message);
            log.debug("已生产消息{}", message);
            list.notifyAll();
        }

    }
}

@Slf4j(topic = "Message")
final class Message {
    private int id;
    private Object value;

    public Message(int id, Object value) {
        this.id = id;
        this.value = value;
    }

    @Override
    public String toString() {
        return "Message{" +
                "id=" + id +
                ", value=" + value +
                '}';
    }
}
```





## 8.Park & Unpark

### 基本使用

```java
// 暂停当前线程
LockSupport.park();

// 恢复运行
LockSupport.unpark(暂停线程对象);
```

### 特点

与Object的wait/notify相比

- wait，notify必须配合Object Monitor一起使用，park & unpark 不必
- park & unpark 是以线程为单位来【阻塞】和【唤醒】现成，而notify只能随机唤醒一个等待线程，notifyAll是唤醒所有等待线程，不【精确】
- park & unpark 可以先unpark



## 9.线程状态之重新理解

![截屏2023-04-18 下午2.30.10](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-04-18 下午2.30.10.png)

### 情况1 NEW --> RUNNABLE

- 当调用t.start()方法时，由NEW --> RUNNABLE



### 情况2 RUNNABLE <--> WAITING

- **t线程**用synchronized(obj)获取对象锁后

  - 调用obj.wait()方法时，**t线程** 从`RUNNABLE --> WAITING`

  - 调用obj.notify(),obj.notifyAll(),t.interrupt()时
    - <font color=red>竞争锁成功：`WAITING-->RUNNABLE`</font>
    - <font color=red>竞争锁失败：`WAITING-->BLOCKED`</font>



### 情况3 RUNNABLE <--> WAITING

- 当前线程调用t.join()方法，`RUNNABLE --> WAITING`
  - 当前线程在 t线程 对象的监视器上等待
- t线程运行结束，或调用了当前线程的interrupt()，`WAITING --> RUNNABLE`



### 情况4 RUNNABLE <--> WAITING

- 当前线程调用LockSupport.park()方法，会让当前线程从`RUNNABLE --> WAITING`
- 调用LockSupport.unpark(目标线程)或调用了线程的interrupt()，会让当前线程从`WAITING --> RUNNABLE`



### 情况5678 RUNNABLE <--> TIMED_WAITING

与WAITING的区别在于调用的方法带有超时时间，以及有一个额外的sleep(long time)



### 情况9 RUNNABLE <--> BLOCKED

- t线程用 synchronized(obj)获取了对象锁时如果竞争失败，RUNNABLE --> BLOCKED
- 持有obj锁线程的同步代码块执行完毕，会唤醒该对象上所有BLOCKED的线程重新竞争，如果t线程完成竞争，则BLOCKED-->RUNNABLE，其他失败的线程仍然BLOCKED



### 情况10 RUNNABLE <--> TERMINATED

当前线程所有代码运行完毕，进入TERMINATED



## 10.活跃性

### 死锁检查

```java
new Thread(()->{
    synchronized (A){
      System.out.println("A线程获取A");
      try {
        Thread.sleep(1000);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      synchronized (B){
        System.out.println("A线程获取B");
      }
    }
}, "A").start();

new Thread(()->{
    synchronized (B){
      System.out.println("B线程获取B");
      try {
        Thread.sleep(1000);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      synchronized (A){
        System.out.println("B线程获取A");
      }
    }
}, "B").start();
```

#### 方法一：

- 首先使用 jps 查看进程id
- 然后使用jstack id

```bash
Found one Java-level deadlock:
=============================
"A":
  waiting to lock monitor 0x00006000023f00d0 (object 0x000000070fa76c28, a java.lang.Object),
  which is held by "B"

"B":
  waiting to lock monitor 0x00006000023e40d0 (object 0x000000070fa76c18, a java.lang.Object),
  which is held by "A"

Java stack information for the threads listed above:
===================================================
"A":
        at com.example.multithreading.活跃性.TestDeadLock.lambda$main$0(TestDeadLock.java:18)
        - waiting to lock <0x000000070fa76c28> (a java.lang.Object)
        - locked <0x000000070fa76c18> (a java.lang.Object)
        at com.example.multithreading.活跃性.TestDeadLock$$Lambda$14/0x0000000800c01208.run(Unknown Source)
        at java.lang.Thread.run(java.base@17/Thread.java:833)
"B":
        at com.example.multithreading.活跃性.TestDeadLock.lambda$main$1(TestDeadLock.java:32)
        - waiting to lock <0x000000070fa76c18> (a java.lang.Object)
        - locked <0x000000070fa76c28> (a java.lang.Object)
        at com.example.multithreading.活跃性.TestDeadLock$$Lambda$15/0x0000000800c01428.run(Unknown Source)
        at java.lang.Thread.run(java.base@17/Thread.java:833)

Found 1 deadlock.

```

#### 方法二:

- 使用jconsole

![截屏2023-04-18 下午4.53.59](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-04-18 下午4.53.59.png)



### 活锁

线程之间互相改变了对方的结束条件，导致线程无法结束

```java
new Thread(()->{
    while (count > 0){
      try {
        Thread.sleep(200);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      count--;
      log.debug("count:{}", count);
    }
}, "t1").start();

new Thread(()->{
    while (count < 20){
      try {
        Thread.sleep(200);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      count++;
      log.debug("count:{}", count);
    }
}, "t2").start();
```



## $\textcolor{orange}{11.ReentrantLock}$

相对于synchronized，具备如下特点

- 可中断
- 可以设置超时时间
- 可以设置为公平锁
- 支持多个条件变量

与synchronized一样，都支持可重入

```java
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {

}finally {
  lock.unlock();
}
```

### 可中断

- lock.lockInterruptibly() 避免死等

```java
    private static ReentrantLock lock = new ReentrantLock();
    public static void main(String[] args) {
        Thread t1 = new Thread(()->{
            try {
                log.debug("尝试获取锁");
                lock.lockInterruptibly();
                try {
                    log.debug("获取到了锁");
                }finally {
                    lock.unlock();
                }
            } catch (InterruptedException e) {
                log.debug("没有获得锁，返回");
                e.printStackTrace();
            }
        }, "t1");

        lock.lock();
        t1.start();

        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        log.debug("打断t1");
        t1.interrupt();
    }
```

### 锁超时

- lock.tryLock(1, TimeUnit.SECONDS)设置超时时间，避免死等，类似自旋

```java
private static ReentrantLock lock = new ReentrantLock();
public static void main(String[] args) throws InterruptedException {
    Thread t1 = new Thread(()->{
      log.debug("尝试获取锁");
      try {
        if(!lock.tryLock(1, TimeUnit.SECONDS)){
          log.debug("获取不到锁");
          return;
        }
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      try {
        log.debug("获取到了锁");
      }finally {
        lock.unlock();
      }
    }, "t1");

    lock.lock();
    t1.start();
    Thread.sleep(500);
    lock.unlock();
}
```

### 公平锁

- flag为true时变为公平锁
- 本意是为了解决饥饿问题

```java
public ReentrantLock(boolean fair) {
  	sync = fair ? new FairSync() : new NonfairSync();
}
```



### 条件变量

synchronized中也有条件变量，即Monitor中的waitSet，当条件不满足时进入waitSet等待。

ReentrantLock的条件变量相对于synchronized的<font color=red>优势是支持多个条件变量</font>

- synchronized是不满足条件的线程均在同一个waitSet中等待
- ReentrantLock支持多间"休息室"，根据不同的条件分为多个，唤醒时也根据条件唤醒

使用流程

- await前需要获得锁
- await执行后，会释放锁，进入conditionObject等待
- await的线程被唤醒（或打断、或超时）重新竞争lock锁
- 竞争lock锁成功后，从await后继续执行



## $\textcolor{green}{同步模式之顺序控制}$

### 1.wait/notify

```java
public class TestWaitNotify {
    static final Object lock = new Object();
    static boolean b2 = false;
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(()->{
            synchronized (lock){
                while (!b2){
                    try {
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                log.debug("t1 run");
            }
        }, "t1");

        Thread t2 = new Thread(()->{
            synchronized (lock){
                log.debug("t2 run");
                b2 = true;
                lock.notifyAll();
            }
        }, "t2");

        t1.start();
        Thread.sleep(2000);
        t2.start();
    }
}
```



### 2.ReentrantLock

```java
public class TestReentrantLock {
    static ReentrantLock lock = new ReentrantLock();
    static boolean b2 = false;
    public static void main(String[] args) throws InterruptedException {
        Condition condition = lock.newCondition();
        Thread t1 = new Thread(()->{
            lock.lock();
            try {
                while (!b2){
                    condition.await();
                }
                log.debug("t1 run");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }, "t1");

        Thread t2 = new Thread(()->{
            log.debug("t2 run");
            lock.lock();
            try {
                b2 = true;
                condition.signal();
            }finally {
                lock.unlock();
            }
        }, "t2");

        t1.start();
        Thread.sleep(2000);
        t2.start();
    }
}
```



### 3.park/unpark

```java
public class TestParkUnPark {
    public static void main(String[] args) {
        Thread t1 = new Thread(()->{
            LockSupport.park();
            log.debug("1");
        }, "t1");
        t1.start();

        Thread t2 = new Thread(()->{
            log.debug("2");
            LockSupport.unpark(t1);
        }, "t2");
        t2.start();
    }
}
```



## $\textcolor{green}{交替输出}$

### 1.wait/notify

```java
public class TestWaitNotify {

    public static void main(String[] args) {
        WaitNotify wn = new WaitNotify(1, 5);
        new Thread(()->{
            wn.print("a", 1, 2);
        }, "t1").start();
        new Thread(()->{
            wn.print("b", 2, 3);
        }, "t2").start();
        new Thread(()->{
            wn.print("c", 3, 1);
        }, "t3").start();
    }
}

class WaitNotify{
    public void print(String content, int waitFlag, int nextFlag){
        for (int i = 0; i < loopNumber; i++) {
            synchronized (this){
                while (flag != waitFlag){
                    try {
                        this.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.print(content);
                flag = nextFlag;
                this.notifyAll();
            }
        }
    }

    private int flag;

    private int loopNumber;

    public WaitNotify(int flag, int loopNumber) {
        this.flag = flag;
        this.loopNumber = loopNumber;
    }
}
```

### 2.ReentrantLock

```java
public class TestReentrantLock {
    public static void main(String[] args) throws InterruptedException {
        AwaitSignal signal = new AwaitSignal(5);
        Condition a = signal.newCondition();
        Condition b = signal.newCondition();
        Condition c = signal.newCondition();
        new Thread(()->{
            signal.print("a", a, b);
        }).start();
        new Thread(()->{
            signal.print("b", b, c);
        }).start();
        new Thread(()->{
            signal.print("c", c, a);
        }).start();

        Thread.sleep(1000);
        signal.lock();
        try {
            System.out.println("---开始---");
            a.signal();
        }finally {
            signal.unlock();
        }
    }
}

class AwaitSignal extends ReentrantLock {
    private int loopNumber;

    public AwaitSignal(int loopNumber){
        this.loopNumber = loopNumber;
    }

    // 参数一 打印内容
    // 参数二 进入哪一件休息室等待
    // 参数三 下一间休息室
    public void print(String content, Condition current, Condition next){
        for (int i = 0; i < loopNumber; i++) {
            lock();
            try {
                current.await();
                System.out.print(content);
                next.signal();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                unlock();
            }
        }
    }
}
```



## $\textcolor{red}{本章小结}$

- 分析多线程访问共享资源时，哪些代码片段属于临界区
- 使用synchronized互斥解决临界区的线程安全问题
  - 掌握synchronized锁对象语法
  - 掌握synchronized加载成员方法和静态方法
  - 掌握wait/notify同步方法
- 使用lock互斥解决临界区的线程安全为题
  - 掌握lock的使用细节：==可打断、锁超时==、公平锁、==条件变量==
- 学会分析变量的线程安全性，掌握常见线程安全类的使用
- 了解线程活跃性问题：死锁、活锁、饥饿
- <font color=red>应用方面</font>
  - 互斥：使用synchronized或Lock达到共享资源互斥效果
  - 同步：使用wait/notify或Lock的条件变量来达到线程间通信效果
- <font color=blue>原理方面</font>
  - monitor、synchronized、wait/notify原理
  - synchronized进阶原理
  - park & unpark原理

- <font color=green>模式方面</font>
  - 同步模式之保护性暂停
  - 异步模式之生产者消费者
  - 同步模式之顺序控制



# 4.共享模式之内存

上一章的Monitor主要关注的是访问共享变量时，保证临界区代码的原子性

这一章学习共享变量在==多线程之间的【可见性】问题与多条指令执行时的【有序性】问题==



## 1.Java内存模型

JMM(Java Memory Model)，它定义了主存、工作内存抽象概念，底层对应着CPU寄存器、缓存、硬件内存、CPU指令优化等。

JMM体现在以下3个方面：

- 原子性：保证指令不会受到线程上下文切换的影响
- 可见性：保证指令不会受CPU缓存的影响
- 有序性：保证指令不会受CPU指令并行优化的影响



## 2.可见性

### 1.问题

- 线程t并不会因为主线程中run改为了false而停下来

```java
static boolean run = true;

public static void main(String[] args) throws InterruptedException {
    Thread t1 = new Thread(()->{
      while (run){
        // do sth.
      }
    });
    t1.start();

    Thread.sleep(1000);
    log.debug("停止t");
    run = false;
}
```

1.初始状态，t线程从主存中读取了run=true到工作内存中

![截屏2023-04-19 下午3.32.39](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-04-19 下午3.32.39.png)



2.t线程需要频繁的从主存中读取run的值，JIT编译器会将run的值缓存至==自己工作内存中的高速缓存中==，减少对主存中run的访问，提高效率

![截屏2023-04-19 下午3.35.21](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-04-19 下午3.35.21.png)

3.1秒之后，main线程修改了run的值，并同步至主存，而t线程是从自己工作内存的高速缓存中读取的这个变量的值，结果永远都是旧值

![截屏2023-04-19 下午3.38.07](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-04-19 下午3.38.07.png)

### 2.解决

- ==volatile==可以用来修饰成员变量和静态成员变量，避免线程从自己的工作缓存中查找变量的值，必须到主存中获取，线程操作volatile变量都是直接操作主存。

```java
static volatile boolean run = true;
```

- sychronized

```java
static volatile boolean run = true;

static Object lock = new Object();

public static void main(String[] args) throws InterruptedException {
    Thread t1 = new Thread(()->{
      while (true){
        synchronized (lock){
          if(!run){
            break;
          }
        }
      }
    });
    t1.start();

    Thread.sleep(1000);
    log.debug("停止t");
    synchronized (lock){
      run = false;
    }
}
```

### 3.可见性 VS 原子性

volatile保证的是在多个线程之间，一个线程对volatile变量的修改对另外一个线程的课件，但并不能保证原子性，仅适用于一个写线程，多个读线程的情况：

上例从字节码角度理解是这样的：

```
getstatic run //  线程 t 获取 run true
getstatic run //  线程 t 获取 run true
getstatic run //  线程 t 获取 run true
getstatic run //  线程 t 获取 run true
putstatic run //  线程 main 修改 run = false
getstatic run //  线程 t 获取 run true
```

线程安全的例子:两个线程，一个i++，一个i--，只能保证看到最新值，不能解决指令交错

```
// i初始为0
getstatic i 	// 线程 2 获取静态变量i的值，线程内i=0

getstatic i 	// 线程 1 获取静态变量i的值，线程内i=0
iconst_1    	// 线程 1 准备常量1
iadd					// 线程 1 自增，线程内i+=1
putstatic i 	// 线程 1 将修改后的值存入静态变量i，静态变量i=1

iconst_1		  // 线程 2 准备常量
isub					// 线程 2 自减 线程内 i-=1
putstatic i		// 线程 2 将修改后的值存入静态变量i=-1
```

`synchronized`语句块既可以保证代码块的原子性，也同时保证代码块内变量的可见性，但缺点是`synchronized`属于重量级操作，性能相对更低。



## $\textcolor{green}{终止模式之两阶段终止}$

可以使用volatile关键字改进第二章中的两阶段终止模式（监控的优雅退出）

```java
@Slf4j
public class Test {
    private Thread monitor;

    private volatile boolean stop = false;

    // 启动监控线程
    public void start(){
        monitor = new Thread(()->{
            while (true){
//                Thread currentThread = Thread.currentThread();
                if(stop){
                    log.debug("料理后事");
                    break;
                }
                try {
                    Thread.sleep(1000);
                    log.debug("执行监控记录");
                }catch (InterruptedException e){
                    // sleep出现异常后，会清除打断标记
                    // 需要重置打断标记
//                    currentThread.interrupt();
                }
            }
        }, "monitor");

        monitor.start();
    }

    // 停止监控线程
    public void stop(){
        stop = true;
        monitor.interrupt();
    }

    public static void main(String[] args) throws InterruptedException {
        Test twoPhaseTermination = new Test();
        twoPhaseTermination.start();

        Thread.sleep(3500);
        log.debug("停止监控");
        twoPhaseTermination.stop();
    }
}
```



## $\textcolor{green}{同步模式之balking}$

balking(犹豫)模式用在一个线程发现另一个线程或本线程已经做了某一件相同的事，那么本线程就无需再做了，直接结束返回

```java
// 启动监控线程
public void start(){
    synchronized (this){
      if(starting){
          return;
      }
      starting = true;
    }
    // do sth...
}
```



## 3.volatile

1. 可见性：当一个线程修改`volatile`变量的值时，其他线程可以立即看到这个变量的最新值。这是因为`volatile`变量的值被修改后，会立即刷新到主内存，而其他线程读取该变量时会直接从主内存中读取，而不是从缓存中读取。因此，使用`volatile`关键字可以<font color=red>避免多线程程序中的数据不一致问题</font>。
2. 有序性：使用`volatile`关键字可以保证程序执行的有序性，即对于`volatile`变量的<font color=red>读写操作不会被重排序</font>到其他指令之前或之后。这意味着，即使在多个线程之间执行，也能保证操作顺序的一致性。

`volatile`关键字并<font color=red>不能保证原子性</font>，也就是说，它不能保证复合操作（例如递增）的原子性。如果需要保证原子性，需要使用同步机制，如`synchronized`关键字或`Lock`接口。

`volatile`关键字是Java中<font color=red>保证多线程程序中共享变量的可见性和有序性</font>的一种机制，它可以确保共享变量的一致性，并且在一些场景下，使用`volatile`关键字可以提高程序的性能。

### 如何保证可见性与有序性

指令重排序

在不改变程序结果的前提下，这些指令的各个阶段可以通过**重排序**和**组合**来实现指令级并行。



volatile提存实现原理是内存屏障，Memory Barrier(Memory Fence)

- 对volatile变量的写指令后会加入写屏障
- 对volatile变量的读指令前会加入读屏障



- 写屏障（sfence）保证<font color=red>在该屏障之前</font>的，对共享变量的改动，都同步到主存中

- ```java
  public void act2(Result r){
  		num = 2;
  		ready = true; // ready是 volatile 赋值带写屏障
      // 写屏障 （不会将写屏障之前的代码指令重排到写屏障之后）
  }
  ```

- 读屏障(Lfence)保证<font color=red>在该屏障之后</font>，对共享变量的读取，加载的是主存中的最新数据

- ```java
  public void act1(Result r){
  		// 读屏障 （不会将读屏障之后的代码指令重排到写屏障之前）
      // ready 是 volatile 读取值，带读屏障
      if(ready){
        	r.r1 = num + num;
      }else{
          r.r1 = 1;
      }
  }
  ```

- 不能解决指令交错
  - 写屏障仅仅是保证之后的读能够读到最新的结果，但不能保证读跑到它前面
  - 而有序性的保证也只是保证了<font color=red>本线程内</font>相关代码不被重排序



## 4.happens-before

happens-before规定了对共享变量的写操作对其他线程的读操作可见，它是可见性与有序性的一套规则总结。抛开以下happens-before规则，JMM并不能保证一个线程对共享变量的写，对于其他线程对该共享变量的读可见。

- 线程解锁m之前对变量的写，对接下来要对m加锁的其他线程对该变量的读可见

```java
static int x;
static Object m = new Object();
public static void main(String[] args) {
    new Thread(()->{
      synchronized (m){
        x = 10;
      }
    }, "t1").start();

    new Thread(()->{
      synchronized (m){
        System.out.println(x);
      }
    }, "t2").start();
}
```

- 线程对volatile变量的写，对接下来其他线程对该变量的读可见

```java
static volatile int x;

public static void main(String[] args) throws InterruptedException {
    new Thread(()->{
      x = 10;
    }, "t1").start();

    new Thread(()->{
      System.out.println(x);
    }, "t2").start();
}
```

- 线程t1打断t2(interrupt)前对变量的写，对于其他线程得知t2被打断后对变量的读可见

```java
Thread t2 = new Thread(()->{
    while (true){
      if(Thread.currentThread().isInterrupted()){
        System.out.println(x);
        break;
      }
    }
}, "t2");
t2.start();

new Thread(()->{
    try {
      Thread.sleep(1000);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    x = 10;
    t2.interrupt();
}, "t1").start();

while (!t2.isInterrupted()){
  	Thread.yield();
}
System.out.println(x);
```

- 对变量默认值(0，false，null)的写，对其他线程对该变量的读可见
- 具有传递性，如果x hb -> y 并且 y hb -> z，那么有x hb -> z，配合volatile防止指令重排



## $\textcolor{orange}{5.线程安全单例练习}$

- **饿汉式**

```java
// Q1: 为什么加final？ 防止子类继承，并执行不当操作破坏单例
// Q2: 如果实现了序列化接口，还需要做什么来防止反序列化？ 实现readResolve()
public final class Singleton implements Serializable {
    // Q3: 设置私有能否防止反射创建新的实例？ 不能
    private Singleton() {

    }

    private static final Singleton instance = new Singleton();

    // Q4: 为什么提供静态方法而不是将 instance 设置为 public？
    // 1.提供更好的封装性
    // 2.创建单例时更好的控制
  	// 3.提供泛型支持
    public static Singleton getInstance(){
        return instance;
    }
  
    // A2 防止反序列化
    public Object readResolve(){
        return instance;
    }
}
```

- **枚举**

  - Q1: 枚举单例是如何限制实例个数的？
    - 是当前枚举类的一个静态成员变量

  - Q2: 枚举单例在创建时是否具有并发问题？
    - 没有，由JVM在类加载阶段完成

  - Q3: 枚举单例是否能被反射破坏？
    - ==不能==

  - Q4: 枚举单例是否能被反序列化破坏？
    - ==不能==

  - Q5: 枚举单例是饿汉式还是懒汉式？
    - 饿汉式

  - Q6: 枚举单例如果希望加入一些创建时的初始化逻辑该怎么做？
    - 通过构造方法等来初始化

```java
enum Singleton{
		INSTANCE;
}
```

- 懒汉式

```java
public final class Singleton implements Serializable {
    private Singleton() {

    }

    private static volatile Singleton instance;

    public static Singleton getInstance(){
        if(instance == null){
            synchronized (Singleton.class){
                if(instance == null){
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }

//    public Object readResolve(){
//        return instance;
//    }
}
```



## 本章小结

- 可见性-由JVM缓存优化引起
- 有序性-由JVM指令重排序优化引起
- happens-before原则
- <font color=blue>原理方面</font>
  - CPU指令并行
  - volatile
- <font color=green>模式方面</font>
  - 两阶段终止模式的volatile改进
  - 同步模式之balking



# 5.共享模型之无锁

## 1.无锁安全并发

```java
@Slf4j
public class Test {
    public static void main(String[] args) {
        Account a = new AccountCAS(new AtomicInteger(10000));
        Account.test(a);

    }
}

class AccountCAS implements Account{

    private AtomicInteger balance;

    public AccountCAS(AtomicInteger balance) {
        this.balance = balance;
    }

    @Override
    public int getBalance() {
        return balance.get();
    }

    @Override
    public void withdraw(Integer amount) {
        // 局部变量 存储在当前线程的工作内存中，不在主存
        while (true){
            // 获取余额的最新值
            int pre = balance.get();
            // 要修改的余额
            int next = pre - amount;
            // 真正修改
            if(balance.compareAndSet(pre, next)){
                break;
            }
        }
    }
}

interface Account{
    int getBalance();

    void withdraw(Integer amount);

    static void test(Account account){
        List<Thread> list = new ArrayList<>();
        for (int i = 0; i < 1; i++) {
            list.add(new Thread(()->{
                account.withdraw(10);
            }));
        }
        list.forEach(Thread::start);
        list.forEach(t->{
            try {
                t.join();
            }catch (InterruptedException e){
                e.printStackTrace();
            }
        });
        System.out.println(account.getBalance());
    }
}
```

### CAS与volatile

CAS必须借助volatile才能读取到共享变量的最新值来实现【比较并交换的效果】

```java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;

    /*
     * This class intended to be implemented using VarHandles, but there
     * are unresolved cyclic startup dependencies.
     */
    private static final Unsafe U = Unsafe.getUnsafe();
    private static final long VALUE
        = U.objectFieldOffset(AtomicInteger.class, "value");

    private volatile int value;
    ......
```

### 为什么无锁效率高

- 无锁情况下，即使重试失败，线程始终在高速运行。synchronized会让线程在没有获得锁的时候发生上下文切换，进入阻塞。

### CAS的特点

结合CAS和volatile可以实现无锁并发，适用于线程数少、多核CPU的场景。

- CAS基于乐观锁思想

- synchronized基于悲观锁
- CAS体现的是无锁并发、无阻塞并发
  - 没有使用synchronized，线程不会阻塞，提升效率
  - 如果竞争激烈，重试会频繁发生，效率受损



## 2.原子整数

JUC并发包提供了：

- AtomicBoolean
- AtomicInteger
- AtomicLong

```java
public static void main(String[] args) {
        AtomicInteger i = new AtomicInteger(0);
        i.getAndIncrement();
        i.incrementAndGet();

        i.updateAndGet(v -> v * 10);

        i.getAndAdd(5);

        System.out.println(updateAndGet(i, v -> v / 2));
    }

		// 模仿并自定义
    public static int updateAndGet(AtomicInteger i, IntUnaryOperator operator) {
        while (true) {
            int prev = i.get();
            int next = operator.applyAsInt(prev);
            if (i.compareAndSet(prev, next)) {
                return next;
            }
        }
    }
```



## 3.原子引用

- AtomicReference

```java
// 对于浮点数类型的修改
class DecimalAccountCAS implements  DecimalAccount{
    
    private AtomicReference<BigDecimal> balance;

    public DecimalAccountCAS(BigDecimal balance) {
        this.balance = new AtomicReference<>(balance);
    }

    @Override
    public BigDecimal getBalance() {
        return balance.get();
    }

    @Override
    public void withdraw(BigDecimal amount) {
        while (true){
            // BigDecimal是不可变类，是线程安全的但是多个操作的组合不一定是线程安全的
            BigDecimal prev = balance.get();
            BigDecimal next = prev.subtract(amount);
            if (balance.compareAndSet(prev, next)) {
                break;
            }
        }
    }
}
```

- AtomicMarkableReference

只是想知道是否修改过，true/false

- AtomicStampedReference(**增加版本号，追踪原子引用的整个变化过程，避免ABA问题**)

```java
public class TestABA {
    static AtomicStampedReference<String> ref;
    public static void main(String[] args) throws InterruptedException {
        ref = new AtomicStampedReference<>("A", 0);
        String prev = ref.getReference();
        int stamp = ref.getStamp();
        log.debug("{}", stamp);
        aba();
        Thread.sleep(1000);
        log.debug("{}", stamp);
        log.debug("change A->C {}", ref.compareAndSet(prev, "B", stamp, stamp + 1));
    }

    public static void aba() {
        new Thread(() -> {
            String prev = ref.getReference();
            int stamp = ref.getStamp();
            log.debug("{}", stamp);
            log.debug("change A->B {}", ref.compareAndSet(prev, "B", stamp, stamp + 1));
        }, "t1").start();

        new Thread(() -> {
            String prev = ref.getReference();
            int stamp = ref.getStamp();
            log.debug("{}", stamp);
            log.debug("change B->A {}", ref.compareAndSet(prev, "A", stamp, stamp + 1));
        }, "t2").start();
    }
}
```



## 4.原子数组

```java
		public static void main(String[] args) {
        demo(
                ()->new int[10],
                arr -> arr.length,
                (arr, index) -> arr[index]++,
                arr -> System.out.println(Arrays.toString(arr))
        );

        demo(
                ()->new AtomicIntegerArray(10),
                arr -> arr.length(),
                (arr, index) -> arr.getAndIncrement(index),
                arr -> System.out.println(arr)
        );
    }

    /*
     * supplier 提供者 无中生有 ()->结果
     * function 函数 一个参数一个结果 ()->结果 ，  BiFunction(参数1,参数2)->结果
     * consumer 消费者 一个参数，没有结果 (参数)->void    BiConsumer(参数1,参数2)->void
     */

    /**
     *
     * @param arraySupplier 提供数组，可以是线程(不)安全数组
     * @param lengthFun     获取数组长度的方法
     * @param putConsumer   自增方法，回传arr，index
     * @param printConsumer 打印数组的方法
     * @param <T>
     */
    private static <T> void demo(
            Supplier<T> arraySupplier,
            Function<T, Integer> lengthFun,
            BiConsumer<T, Integer> putConsumer,
            Consumer<T> printConsumer){
        List<Thread> list = new ArrayList<>();
        T arr = arraySupplier.get();
        int length = lengthFun.apply(arr);
        for (int i = 0; i < length; i++) {
            list.add(new Thread(()->{
                // 一个线程操作10000次
                for (int j = 0; j < 10000; j++) {
                    putConsumer.accept(arr, j % length);
                }
            }));
        }
        list.forEach(Thread::start);
        printConsumer.accept(arr);
    }
```



## 5.字段更新器

- 保护对象属性赋值的原子性

```java
public class Test {

    public static void main(String[] args) {
        Student student = new Student();
        AtomicReferenceFieldUpdater updater =
                AtomicReferenceFieldUpdater.newUpdater(Student.class, String.class, "name");
        System.out.println(updater.compareAndSet(student, null, "张三"));
        System.out.println(student);
    }
}

class Student{
    volatile String name;

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                '}';
    }
}
```



## 6.原子累加器

- 相比于Atomic，Adder的效率自增效率更高。因为在有竞争时，设置了多个累加单元，t0累加Cell[0],t1累加Cell[1]......最后将结果汇总。操作不同的Cell变量，减少CAS次数，提升性能

```java
		public static void main(String[] args) {
        for (int i = 0; i < 5; i++) {
            demo(
                    () -> new AtomicLong(0),
                    (adder) -> adder.getAndIncrement()
            );
        }


        for (int i = 0; i < 5; i++) {
            demo(
                    () -> new LongAdder(),
                    (adder) -> adder.increment()
            );
        }
    }

    public static <T> void demo(Supplier<T> supplier, Consumer<T> action){
        T adder = supplier.get();
        List<Thread> list = new ArrayList<>();
        for (int i = 0; i < 4; i++) {
            list.add(new Thread(()->{
                for (int j = 0; j < 500000; j++) {
                    action.accept(adder);
                }
            }));
        }
        long start = System.nanoTime();
        list.forEach(Thread::start);
        list.forEach(t -> {
            try {
                t.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        System.out.println(adder + ",cost: " + (System.nanoTime() - start) / 1000000);
    }
```



### LongAdder原理

LongAdder类的几个关键域

```java
// 累加单元数组，懒惰初始化
transient volatile Cell[] cells;
// 基础值，如果没有竞争，则用cas累加这个域
transient volatile long base;
// 在 cells 创建或扩容时，置为1，表示加锁
transient volatile int cellsBusy;
```

#### <font color=blue>原理之伪共享</font>

```java
@sun.misc.Contended
static final class Cell{
  	volatile long value;
    public Cell(long x){
      	value = x;
    }
  
  	// 最重要的方法，cas方式进行累加，prev表示旧值，next表示新值
    final boolean cas(long prev, long next){
      	return UNSAFE.compareAndSwapLong(this, valueOffset, prev, next);
    }
  	// ......
}
```

- 缓存与内存的速度比较

![截屏2023-05-04 上午10.46.26](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-04 上午10.46.26.png)

| 从cpu到 | 大约需要的时钟周期          |
| ------- | --------------------------- |
| 寄存器  | 1cycle(4GHz的CPU约为0.25ns) |
| L1      | 3~4cycle                    |
| L2      | 10~20cycle                  |
| L3      | 40~45cycle                  |
| 内存    | 120~240cycle                |

因为CPU与内存的速度差异很大，需要靠预读数据至缓存来提升效率

而缓存以缓存行为单位，每个缓存行对应着一块内存，一般是 64 byte（8个long）

缓存的加入会造成数据副本的产生，即同一份数据会缓存在不同核心的缓存行中

CPU要保证数据的一致性，如果某个CPU核心更改了数据，其他CPU核心对应的==整个缓存行必须失效==

![截屏2023-05-04 下午1.41.11](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-04 下午1.41.11.png)

Cell是数组形式，在内存中连续存储，一个Cell为24字节(16字节的对象头和8字节的value)，因此缓存行可以存在2个Cell对象。会出现以下问题：

- Core-0要修改Cell[0]
- Core-1要修改Cell[1]

无论谁修改成功，都会导致对方Core的缓存行失效。

@sun.misc.Contended用来解决这个问题，它使得使用此注解的对象或字段的前后个增加128字节大小的padding，从而让CPU将对象预读至缓存时占用不同的缓存行。这样，不会造成对方的缓存行失效。

![截屏2023-05-04 下午2.00.06](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-04 下午2.00.06.png)



## $\textcolor{orange}{7.Unsafe}$

### 基本介绍

位于sun.misc.Unsafe的单例实现

调用方法:

1. 从`getUnsafe`方法的使用限制条件出发，通过Java命令行命令`-Xbootclasspath/a`把调用Unsafe相关方法的类A所在jar包路径追加到默认的bootstrap路径中，使得A被引导类加载器加载，从而通过`Unsafe.getUnsafe`方法安全的获取Unsafe实例。

   ```bash
   java -Xbootclasspath/a: ${path}   // 其中path为调用Unsafe相关方法的类所在jar包路径 
   ```

2. 通过反射获取

```java
public class Test {
    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
        Field theUnsafe = Unsafe.class.getDeclaredField("theUnsafe");
        theUnsafe.setAccessible(true);
        Unsafe unsafe = (Unsafe) theUnsafe.get(null);

        System.out.println(unsafe);

        // 1.获取域的偏移地址
        long id = unsafe.objectFieldOffset(Teacher.class.getDeclaredField("id"));
        long name = unsafe.objectFieldOffset(Teacher.class.getDeclaredField("name"));

        Teacher t = new Teacher();
        unsafe.compareAndSwapInt(t, id, 0, 1);
        unsafe.compareAndSwapObject(t, name, null, "teacher");

        System.out.println(t);
    }
}

@Data
class Teacher{
    volatile int id;
    volatile String name;
}
```

### 功能介绍

![截屏2023-05-04 下午2.55.27](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-04 下午2.55.27.png)

#### 内存操作

主要包括堆外内存的分配、拷贝。释放、给定地址值操作等方法

```java
//分配内存, 相当于C++的malloc函数
public native long allocateMemory(long bytes);
//扩充内存
public native long reallocateMemory(long address, long bytes);
//释放内存
public native void freeMemory(long address);
//在给定的内存块中设置值
public native void setMemory(Object o, long offset, long bytes, byte value);
//内存拷贝
public native void copyMemory(Object srcBase, long srcOffset, Object destBase, long destOffset, long bytes);
//获取给定地址值，忽略修饰限定符的访问限制。与此类似操作还有: getInt，getDouble，getLong，getChar等
public native Object getObject(Object o, long offset);
//为给定地址设置值，忽略修饰限定符的访问限制，与此类似操作还有: putInt,putDouble，putLong，putChar等
public native void putObject(Object o, long offset, Object x);
//获取给定地址的byte类型的值（当且仅当该内存地址为allocateMemory分配时，此方法结果为确定的）
public native byte getByte(long address);
//为给定地址设置byte类型的值（当且仅当该内存地址为allocateMemory分配时，此方法结果才是确定的）
public native void putByte(long address, byte x);
```

通常，我们在Java中创建的对象都处于堆内内存（heap）中，堆内内存是由JVM所管控的Java进程内存，并且它们遵循JVM的内存管理机制，JVM会采用垃圾回收机制统一管理堆内存。与之相对的是堆外内存，存在于JVM管控之外的内存区域，==java中对堆外内存的操作，依赖于Unsafe提供的操作堆外内存的native方法。==

**使用堆外内存的原因**

- **对垃圾回收停顿的改善**。由于堆外内存是直接受操作系统管理而不是JVM，所以当我们使用堆外内存时，即可<font color=red>保持较小的堆内内存规模。从而在GC时减少回收停顿对于应用的影响。</font>
- **提升程序I/O操作的性能**。通常在I/O通信过程中，会存在堆内内存到堆外内存的数据拷贝操作，对于需要频繁进行内存间数据拷贝且生命周期较短的暂存数据，都建议存储到堆外内存。

**典型应用**

DirectByteBuffer是Java用于实现堆外内存的一个重要类，通常用在通信过程中做缓冲池，如在Netty、MINA等NIO框架中应用广泛。DirectByteBuffer对于堆外内存的创建、使用、销毁等逻辑均由Unsafe提供的堆外内存API来实现。

```java
DirectByteBuffer(int cap) {                   // package-private
				// ......
        try {
            // 分配内存
            base = UNSAFE.allocateMemory(size);
        } catch (OutOfMemoryError x) {
            Bits.unreserveMemory(size, cap);
            throw x;
        }
        // 初始化
        UNSAFE.setMemory(base, size, (byte) 0);
        
  		  // .......
  
        // 垃圾回收，释放堆外内存
        cleaner = Cleaner.create(this, new Deallocator(base, size, cap));
    }
```

Cleaner继承自虚引用PhantomReference（==众所周知，无法通过虚引用获取与之关联的对象实例，且当对象仅被虚引用引用时，在任何发生GC的时候，其均可被回收==），通常PhantomReference与引用队列ReferenceQueue结合使用，可以实现虚引用关联对象被垃圾回收时能够进行系统通知、资源清理等功能。如下图所示，当某个被Cleaner引用的对象将被回收时，JVM垃圾收集器会将此对象的引用放入到对象引用中的pending链表中，等待Reference-Handler进行相关处理。其中，Reference-Handler为一个拥有最高优先级的守护线程，会循环不断的处理pending链表中的对象引用，执行Cleaner的clean方法进行相关清理工作。

![截屏2023-05-04 下午3.28.08](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-04 下午3.28.08.png)

所以当DirectByteBuffer仅被Cleaner引用（即为虚引用）时，其可以在任意GC时段被回收。当DirectByteBuffer实例对象被回收时，在Reference-Handler线程操作中，会调用Cleaner的clean方法根据创建Cleaner时传入的Deallocator来进行堆外内存的释放。



#### CAS相关

```java
/**
	*  CAS
  * @param o         包含要修改field的对象
  * @param offset    对象中某field的偏移量
  * @param expected  期望值
  * @param update    更新值
  * @return          true | false
  */
public final native boolean compareAndSwapObject(Object o, long offset,  Object expected, Object update);

public final native boolean compareAndSwapInt(Object o, long offset, int expected,int update);
  
public final native boolean compareAndSwapLong(Object o, long offset, long expected, long update);
```

**典型应用**

CAS在==java.util.concurrent.atomic相关类、Java AQS、CurrentHashMap==等实现上有非常广泛的应用。



#### 线程调度

```java
//取消阻塞线程
public native void unpark(Object thread);
//阻塞线程
public native void park(boolean isAbsolute, long time);
```

**典型应用**

Java锁和同步器框架的核心类AbstractQueuedSynchronizer，就是通过调用`LockSupport.park()`和`LockSupport.unpark()`实现线程的阻塞和唤醒的，而LockSupport的park、unpark方法实际是调用Unsafe的park、unpark方式来实现。



------

参考自https://tech.meituan.com/2019/02/14/talk-about-java-magic-class-unsafe.html，还有部分没有记录



## 本章小结

- CAS与volatile
- API
  - 原子整数
  - 原子引用
  - 原子数组
  - 字段更新器
  - 原子累加器
- Unsafe



# 6.共享模型之不可变

String类是不可变的，在调用substring()等方法时，创建的是一个新的对象(保护性拷贝)，由此引发了对象创建过多的问题。

## $\textcolor{green}{享元模式(Flyweight  pattern)}$

当需要重用数量有限的同一类对象时

### 体现

#### 1.包装类

在JDK中Boolean，Byte，Short，Integer，Long，Character等包装类提供了valueOf方法，例如Long的valueOf会缓存-128~127之间的Long对象，在这个范围内会重用对象，大于这个范围，才会新建Long对象。

```java
public static Long valueOf(long l){
  	final int offset = 128;
    if(l >= -128 && l <= 127){
        return LongCache.cache[(int)l + offset];
    }
    return new Long(l);
}
```

```java
private static class LongCache {
    private LongCache() {}

    static final Long[] cache;
    static Long[] archivedCache;

    static {
      int size = -(-128) + 127 + 1;

      // Load and use the archived cache if it exists
      CDS.initializeFromArchive(LongCache.class);
      if (archivedCache == null || archivedCache.length != size) {
        Long[] c = new Long[size];
        long value = -128;
        for(int i = 0; i < size; i++) {
          c[i] = new Long(value++);
        }
        archivedCache = c;
      }
      cache = archivedCache;
    }
}
```

------

Tips:

Byte,Short,Long缓存的范围都是-128~127

Character缓存的范围是0~127

Integer的默认范围是-128~127，最小值不能变，但最大值可以调整虚拟机参数

`-Djava.lang.Integer.IntegerCache.high`来改变

Boolean缓存了TRUE和FALSE

#### 2.String 串池

#### 3. BigDecimal BigInteger

### DIY

一个线上应用商城，QPS达到数千，如果每次都重新创建和关闭数据库连接，性能影响很大。这时应预先创建好一批连接，放入连接池。一次请求到达后，从连接池获取链接，使用完毕后再还回连接池，既节约了连接的创建和关闭时间，也实现了连接的重用，不至于让连接压垮数据库。

```java
@Slf4j
public class Pool {
    private int poolSize;

    private Connection[] connections;

    // 0 空闲
    // 1 繁忙
    private AtomicIntegerArray states;

    public Pool(int poolSize) {
        this.poolSize = poolSize;
        this.connections = new Connection[poolSize];
        this.states = new AtomicIntegerArray(poolSize);
        for (int i = 0; i < poolSize; i++) {
            connections[i] = new MockConnection();
        }
    }

    public Connection borrow() {
        while (true) {
            // 获取空闲连接
            for (int i = 0; i < poolSize; i++) {
                if (states.get(i) == 0) {
                    if (states.compareAndSet(i, 0, 1)) {
                        log.debug("borrow,{}", connections[i]);
                        return connections[i];
                    }
                }
            }
            // 如果没有空闲连接，进入连接池等待
            synchronized (this) {
                try {
                    log.debug("wait...");
                    this.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public void free(Connection c) {
        for (int i = 0; i < poolSize; i++) {
            if (connections[i] == c) {
                states.set(i, 0);
                synchronized (this) {
                    log.debug("free {}", c);
                    this.notifyAll();
                }
                break;
            }
        }
    }

    @Override
    public String toString() {
        return ", connections=" + Arrays.toString(connections) +
                ", states=" + states +
                '}';
    }
}
```

```java
public class MockConnection implements Connection{

}
```

```java
public static void main(String[] args) {
    Pool pool = new Pool(2);
    for (int i = 0; i < 5; i++) {
      new Thread(()->{
        Connection borrow = pool.borrow();
        try {
          Thread.sleep(new Random().nextInt(1000));
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
        pool.free(borrow);
      }).start();
    }
}
```



## final原理

```java
public class FinalTest {
    final int a = 10;
}
```

字节码

```bash
0: aload_0
1: invokespecial #1
4: aload_0
5: bipush
7: putfield      #2
<-- 写屏障
10: return
```

final变量的赋值也会通过putfield指令完成，同样在这条指令之后也会加入写屏障，保证其他线程读到它的值时不会出现为 0 的情况



## 无状态

没有任何成员变量的类是线程安全的，因为成员变量保存的数据也可以称为状态信息，因此没有成员变量就称之为【无状态】





# $\textcolor{red}{7.自定义线程池}$

![截屏2023-03-19 下午8.15.19](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-03-19 下午8.15.19.png)

## 1.DIY

### 定义拒绝策略

```java
@FunctionalInterface
interface RejectPolicy<T>{
    void reject(BlockingQueue<T> queue, T task);
}
```

### 定义阻塞队列

```java
@Slf4j
class BlockingQueue<T> {
    // 1.任务队列
    private Deque<T> queue = new ArrayDeque<>();

    // 2.锁
    private ReentrantLock lock = new ReentrantLock();

    // 3.生产者条件变量
    private Condition fullWaitSet = lock.newCondition();

    // 4.消费者条件变量
    private Condition emptyWaitSet = lock.newCondition();

    // 5.容量
    private int capacity;

    public BlockingQueue(int capacity) {
        this.capacity = capacity;
    }

    // 带超时时间的阻塞获取
    public T poll(long timeout, TimeUnit unit) {
        lock.lock();
        try {
            // 将超时时间统一转换为纳秒
            long nanos = unit.toNanos(timeout);
            while (queue.isEmpty()) {
                try {
                    // 当没到nanos就被唤醒时，awaitNanos()返回的是剩余时间
                    if (nanos <= 0) {
                        return null;
                    }
                    nanos = emptyWaitSet.awaitNanos(nanos);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            T t = queue.removeFirst();
            fullWaitSet.signal();
            return t;
        } finally {
            lock.unlock();
        }
    }

    // 阻塞获取
    public T poll() {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                try {
                    emptyWaitSet.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            T t = queue.removeFirst();
            // 有元素被取出，提醒fullWaitSet可以添加
            fullWaitSet.signal();
            return t;
        } finally {
            lock.unlock();
        }
    }

    // 带超时时间的阻塞添加
    public boolean put(T task, long timeout, TimeUnit unit) {
        lock.lock();
        try {
            long nanos = unit.toNanos(timeout);
            while (queue.size() == capacity) {
                try {
                    log.debug("【等待】任务队列{}", task);
                    if (nanos <= 0) {
                        log.debug("【等待】超时，结束任务{}", task);
                        return false;
                    }
                    nanos = fullWaitSet.awaitNanos(nanos);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            log.debug("【加入】任务队列{}", task);
            queue.addLast(task);
            emptyWaitSet.signal();
            return true;
        } finally {
            lock.unlock();
        }
    }

    // 阻塞添加
    public void put(T task) {
        lock.lock();
        try {
            while (queue.size() == capacity) {
                try {
                    log.debug("【等待】任务队列{}", task);
                    fullWaitSet.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            log.debug("【加入】任务队列{}", task);
            queue.addLast(task);
            // 有元素加入，提醒emptyWaitSet可以取出
            emptyWaitSet.signal();
        } finally {
            lock.unlock();
        }
    }

    private int getSize() {
        lock.lock();
        try {
            return queue.size();
        } finally {
            lock.unlock();
        }
    }

    public void strategicMode(RejectPolicy<T> rejectPolicy, T task) {
        lock.lock();
        try {
            // 队列是否已满
            if(queue.size() == capacity){
                rejectPolicy.reject(this, task);
            }else {
                log.debug("【加入】任务队列,{}", task);
                queue.addLast(task);
                emptyWaitSet.signal();
            }
        }finally {
            lock.unlock();
        }
    }
}
```

### 构建线程池

```java
@Slf4j(topic = "ThreadPool")
class ThreadPool {

    private int coreSize;

//    private int maxSize;

    private long timeout;

    private TimeUnit timeUnit;

    private BlockingQueue<Runnable> taskQueue;

    private HashSet<Worker> workers = new HashSet();

    private RejectPolicy<Runnable> rejectPolicy;

    public ThreadPool(int coreSize, int queueCapacity, RejectPolicy<Runnable> rejectPolicy) {
        this.coreSize = coreSize;
        this.taskQueue = new BlockingQueue<>(queueCapacity);
        this.rejectPolicy = rejectPolicy;
    }

    public ThreadPool(int coreSize, long timeout, TimeUnit timeUnit, int queueCapacity, RejectPolicy<Runnable> rejectPolicy) {
        this.coreSize = coreSize;
        this.timeout = timeout;
        this.timeUnit = timeUnit;
        this.taskQueue = new BlockingQueue<>(queueCapacity);
        this.rejectPolicy = rejectPolicy;
    }

    public void execute(Runnable task) {
        // 任务数没超过核心线程数，直接交给 worker 对象执行
        synchronized (workers) {
            if (workers.size() < coreSize) {
                Worker worker = new Worker(task);
                log.debug("【新增】worker:{}", worker);
                workers.add(worker);
                worker.start();
            } else { // 任务数没超过核心线程数
                // 1) 死等
                // 2) 超时等待
                // 3) 放弃执行当前任务
                // 4) 调用者抛出异常
                // 5) 调用者自己执行
                taskQueue.strategicMode(rejectPolicy, task);
            }
        }
    }

    class Worker extends Thread {
        private Runnable task;

        public Worker(Runnable task) {
            this.task = task;
        }

        public void run() {
            // 执行任务
            // 1) task 不为空 执行任务
            // 2) task 执行完毕，从任务队列中获取
            while (task != null || (task = taskQueue.poll(timeout, timeUnit)) != null) {
                try {
                    log.debug("正在执行{}...", task);
                    task.run();
                } catch (Exception e) {

                } finally {
                    task = null;
                }
            }
            synchronized (workers) {
                log.debug("【移除】worker:{}", this);
                workers.remove(this);
            }
        }
    }
}
```



## 2.ThreadPoolExecutor

### 2.1 线程池状态

ThreadPoolExecutor使用int的高3位来表示线程池状态，低29位表示线程数量

| 状态名     | 高3位 | 接收新任务 | 处理阻塞队列任务 | 说明                                      |
| ---------- | ----- | ---------- | ---------------- | ----------------------------------------- |
| RUNNING    | 111   | Y          | Y                |                                           |
| SHUTDOWN   | 000   | N          | Y                | 不会接收新任务，但会处理阻塞队列剩余任务  |
| STOP       | 001   | N          | N                | 会中断正在执行的任务，并抛弃阻塞队列任务  |
| TIDYING    | 010   | -          | -                | 任务权执行完毕，活动线程位0，即将进入终结 |
| TERMINATED | 001   | -          | -                | 终结状态                                  |

从数字上比较，TERMINATED>TIDYING>STOP>SHUTDOWN>RUNNING

这些信息存储在一个原子变量ctl中，目的是将==线程池状态==与==线程个数==合二为一，这样就可以用一次cas原子操作进行赋值

### 2.2构造方法

```java
public ThreadPoolExecutor(
  int corePoolSize, // 核心线程数
  int maximumPoolSize,  // 最大线程数
  long keepAliveTime,   // 生存时间--针对救急线程
  TimeUnit unit,				// 时间单位
  BlockingQueue<Runnable> workQueue,  // 阻塞队列
  ThreadFactory threadFactory,				// 线程工厂--起名
  RejectedExectionHandler handler			// 拒绝策略
)
```

- 线程池中开始没有线程，当一个任务提交给线程池后，线程池会创建一个新线程来执行任务。
- 当线程数达到corePoolSize并没有线程空闲，这时再加入任务，新加的任务会被加入workQueue队列排队，直到有空闲的线程。
- 如果选择是的有界队列，当任务超过了队列大小时，会创建 maximumPoolSize - corePoolSize数目的线程来救急。
- 如果线程达到 maximumPoolSize，那么会执行拒绝策略。jdk提供了4种实现，其他框架也有时限
  - AbortPolicy 让调用者抛出RejectedExecutionException异常，默认策略
  - CallerRunsPolicy 让调用者自己运行
  - DiscardPolicy 放弃本次任务
  - DiscardOldestPolicy 放弃队列中最早的任务，本任务取而代之
  - Dubbo的实现：在抛出RejectedExecutionException异常之前会记录日志，并dump线程栈信息，方便定位问题
  - Netty的实现：创建一个新线程来执行任务
  - ActiveMQ的实现：带超时等待(60s)尝试放入队列
  - PinPoint的实现：拒绝策略链，逐一尝试策略链中每种拒绝策略
- 当高峰过去后，超过corePoolSize的救急线程如果一段时间没有任务，就会根据(KeepAliveTime,Unit)结束

![截屏2023-05-08 下午4.57.21](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-08 下午4.57.21.png)



### 2.3 newFixedThreadPool(固定大小线程池)

- 创建

```java
ExecutorService executorService = Executors.newFixedThreadPool(2, new ThreadFactory() {
    private AtomicInteger t = new AtomicInteger(1);

    @Override // 自定义线程名
    public Thread newThread(Runnable r) {
      return new Thread(r, "test" + t.getAndIncrement());
    }
});
```

- 源码

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
}
```

特点

- 核心线程数=最大线程数（没有急救线程被创建），因此也无需超时时间
- 阻塞队列是无界的，可以放任意数量的任务

**适用场景**

<font color=red>适用于任务量已知，相对耗时的任务</font>



### 2.4 newCachedThreadPool

- 创建

```java
ExecutorService service = Executors.newCachedThreadPool()
```

- 源码

```java
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

特点

- 核心线程数是 0 ，最大线程数是Integer.MAX_VALUE，救急线程的空闲生存时间是 60s，表示
  - 全部线程都是救急线程
  - 线程可以无限创建
- 队列采用 Synchronous 实现特点是，没有容量。一定要有线程来，才会创建。

**使用场景**

整个线程池表现为线程数跟根据任务量不断增长，没有上限。当任务执行完毕，空闲1分钟后释放线程。

<font color=red>适合任务数比较密集，但每个任务执行较短的情况</font>



### 2.5 newSingleThreadExecutor

- 源码

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
      	// 适配器模式
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

**使用场景**

希望多个任务排队执行，线程数固定位1，任务多余一个时，会放入无界队列排队。任务执行完毕，唯一的线程也不会被释放。

区别：

- 自己创建的单线程串行执行任务，如果任务失败而终止，那么没有任何补救措施，而线程池还会新建一个线程，保证池的正常工作
- `Executors.newSingleThreadExecutor()`线程个数始终为1，不能修改
  - `FinalizableDelegatedExecutorService`应用的是装饰器模式，保证了只对外暴露了`ExecutorService`接口，因此不能调用`ThreadPoolExecutor`中特有的方法
- `Executors.newFixedThreadPool(1)`初始为1时，以后还可以修改
  - 对外暴露的是 `ThreadPoolExecutors` 对象，可以墙砖后调用`setCorePoolSize()`等方法进行修改



### 2.6 提交任务

```java
// 执行任务
void execute(Runnable command);

// 提交任务 task，用返回值 Future 获得任务执行结果
<T> Future<T> submit(Callable<T> task);

// 提交 tasks 中所有任务
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException;

// 提交 tasks 中所有任务,带超时时间
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException;

// 提交 tasks 中所有任务，哪个任务先执行完毕就返回哪个的结果，其余任务取消
<T> List<Future<T>> invokeAny(Collection<? extends Callable<T>> tasks) throws InterruptedException, ExecutionException;

// 提交 tasks 中所有任务，哪个任务先执行完毕就返回哪个的结果，其余任务取消，带超时时间
<T> List<Future<T>> invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException, ExecutionException;
```



### 2.7 关闭线程池

#### shutdown

```java
/*
	- 不会接收新任务
	- 但已提交的任务会执行完
	- 此方法不会阻塞调用线程的执行
*/
void shutdown();
```



#### shutdownNow

```java
/*
	- 不会接收新任务
	- 会将队列中的任务返回
	- 并用 interrupt 中断正在执行的线程
*/
List<Runnable> shutdownNow();
```



#### 其他方法

```java
// 不在 RUNNING 状态的线程池，此方法就返回 true
boolean isShutdown();

// 线程池状态是否 TERMIANTED
boolean isTerminated();

// 调用 shutdown 后，由于调用线程并不会等待所有任务运行结束，因此如果它想在线程池 TERMIANTED 后做一些事情，可以用此方法等待
boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;
```



### $\textcolor{green}{异步模式之工作线程}$

让有限的工作线程(Worker Thread)来轮流异步处理无限多的任务。也可以归类为分工模式，典型就是线程池，也体现了经典设计中的享元模式。



**饥饿现象**

固定大小线程池会有饥饿现象

- 两个工人是同一个线程池中的两个线程
- 为客人点餐和到后厨做菜，这是两个阶段的工作
  - 客人点餐：必须先点完餐，才能做、上菜，在此期间工人必须等待
  - 后厨做菜：
- 如果同时来了2个客人，A和B都去处理点餐了，就没人做饭了，导致死锁

<font color=red>不同任务类型应该使用不同的线程池，这样能够避免饥饿，提升效率</font>

### 2.8 创建多少线程池合适

- 过小导致程序不能充分利用系统资源、容易导致饥饿
- 过大会导致更多的线程上下文切换，占用内存更多



#### 1. CPU密集型计算

通常采用 `cpu核数+1`能够实现最优的CPU利用率，<font color=red>+1是保证当线程由于页确实故障(操作系统)或其他原因导致暂停</font>时，额外的这个线程就能顶上去，保证 CPU 时钟周期不被浪费



#### 2. I/O密集型运算

CPU不总是处于繁忙。当执行业务计算时，这时候会使用CPU资源，但当你执行 IO 操作时、远程 RPC 调用时，包括进行数据库操作时，CPU就闲下来了，可以使用多线程提高利用率。

经验公式：

<font color=red>线城数 = 核数 * 期望 CPU 利用率 * 总时间(CPU计算时间+等待时间) / CPU 计算时间</font>

eg:

------

4核CPU计算时间是50%，其他等待时间是50%，期望cpu利用率100%，套用公式

4 * 100% * 100% / 50% = 8

------

4核CPU计算时间是10%，其他等待时间是90%，期望cpu利用率100%，套用公式

4 * 100% * 100% / 10% = 40



### 2.9 任务调度线程池

```java
ScheduledExecutorService pool = Executors.newScheduledThreadPool(1);
```

```java
pool.schedule(()->{
    log.debug("t1");
    try {
      Thread.sleep(2000);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
}, 1, TimeUnit.SECONDS); // 延迟1秒开始执行
```

```java
pool.scheduleAtFixedRate(()->{
    log.debug("t2");
    try {
      Thread.sleep(2000);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
}, 2, 1, TimeUnit.SECONDS); // 延迟2秒开始执行，每隔1秒执行一次，但如果执行的时间x > 1,则每隔x秒执行

17:22:17.749 [pool-1-thread-1] DEBUG com.example.multithreading.线程池.任务调度线程池.Test - t2
17:22:19.753 [pool-1-thread-1] DEBUG com.example.multithreading.线程池.任务调度线程池.Test - t2
17:22:21.757 [pool-1-thread-1] DEBUG com.example.multithreading.线程池.任务调度线程池.Test - t2
```

```java
pool.scheduleWithFixedDelay(()->{
    log.debug("3");
    try {
      Thread.sleep(2000);
    }catch (InterruptedException e){
      e.printStackTrace();
    }
}, 2, 1, TimeUnit.SECONDS);  // 延迟2秒开始执行，每隔执行时间 x + 1执行一次

17:20:14.275 [pool-1-thread-1] DEBUG com.example.multithreading.线程池.任务调度线程池.Test - 3
17:20:17.282 [pool-1-thread-1] DEBUG com.example.multithreading.线程池.任务调度线程池.Test - 3
17:20:20.286 [pool-1-thread-1] DEBUG com.example.multithreading.线程池.任务调度线程池.Test - 3
```



## 3. Tomcat线程池

![截屏2023-05-10 下午5.30.12](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-10 下午5.30.12.png)

- LimitLatch 用来限流，可以控制最大链接个数，类似JUC中的Semaphore
- Acceptor只负责【接收新的socket链接】
- Poller只负责监听socketChannel是否有【可读的I/O事件】
- 一旦可读，封装一个任务对象(socketProcessor)，提交给Executor线程池处理
- Executor线程池中的工作线程最终负责【处理请求】



## 4. Fork/Join

Fork/Join是JDK 1.7 加入的新的线程池的实现， 它体现的是一种分治思想，适用于能够进行任务拆分的`cpu密集型`运算。

任务拆分，即将一个大任务拆分为算法上相同的小任务，直至不能拆分可以直接求解

Fork/Join在分支的基础上加入了多线程，可以把每个任务的分解和合并交给不同的线程来完成，进一步提升运算效率

Fork/Join默认创建与cpu核心数大小相同的线程池



# 8.AQS

## 1.概念

AbstractQueuedSynchronizer，阻塞式锁和相关的同步器工具的框架

### 特点

- 用state属性来表示资源的状态（分为独占式和共享式），子类需要定义如何维护这个状态，控制如何获取锁和释放锁
  - getState - 获取state状态
  - setState - 设置state状态
  - compareAndSetState - cas设置 state 状态
  - 独占模式是只有一个线程能够访问资源，而共享模式可以允许多个线程访问资源
- 提供了基于FIFO的等待队列，类似Monitor的EntryList
- 条件变量来实现等待、唤醒机制，支持多个条件变量，类似Monitor的WaitSet



### 核心思想

- 如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。
- 如果被请求的共享资源被占用，那么就需要一套【线程阻塞等待以及被唤醒时锁分配的机制】，这个机制AQS是用`CLH`队列锁实现的，即将暂时获取不到锁的线程加入到队列中。

```java
//返回同步状态的当前值
protected final int getState() {  
        return state;
}
 // 设置同步状态的值
protected final void setState(int newState) { 
        state = newState;
}
//原子地(CAS操作)将同步状态值设置为给定值update如果当前同步状态的值等于expect(期望值)
protected final boolean compareAndSetState(int expect, int update) {
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```



### 资源共享方式

- Exclusive(独占)：只有一个线程能执行，如ReentrantLock
  - 公平锁：按照入队顺序排序，先到先得
  - 非公平锁：自由竞争

- share(共享)：多个线程可同时执行，如Semaphore、CountDownLatch



自定义同步器在实现时只需要实现【共享资源 state 的获取与释放方式即可】，具体线程等待队列的维护(如获取资源失败入队/唤醒出队等)，AQS已经在上层已经帮我们实现好了(==也是AQS为什么是抽象类而不是接口的原因==)。



### 模版方法的应用

默认情况下，每个方法都抛出 UnsupportedOperationException。 这些方法的实现必须是内部线程安全的，并且通常应该简短而不是阻塞。AQS类中的其他方法都是final ，所以无法被其他类使用，只有这几个方法可以被其他类使用。

```java
//该线程是否正在独占资源。只有用到condition才需要去实现它。
boolean isHeldExclusively()
  
//独占方式。尝试获取资源，成功则返回true，失败则返回false。  
boolean tryAcquire(int)
  
//独占方式。尝试释放资源，成功则返回true，失败则返回false。
boolean tryRelease(int)

//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
int tryAcquireShared(int)

//共享方式。尝试释放资源，成功则返回true，失败则返回false。
boolean tryReleaseShared(int)
```

AbstractQueuedSynchronizer类底层的数据结构是使用`CLH(Craig,Landin,and Hagersten)队列`是一个虚拟的双向队列(虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系)

![截屏2023-05-11 下午7.17.50](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-11 下午7.17.50.png)









获取锁的姿势

```java
// 如果获取锁失败
if(!tryAcquire(arg)){
  	// 入队，可以选择阻塞当前线程
}
```

释放锁的姿势

```java
// 如果释放锁成功
if(tryRelease(arg)){
		// 让阻塞线程恢复运行
}
```



## 2.ReentrantLock

![截屏2023-05-10 下午10.23.04](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-10 下午10.23.04.png)

### 1.非公平锁的实现原理

```java
public ReentrantLock(){
  	sync = new NonfairSync();
}
```

```java
// jdk1.8 源码
final void lock() {
    if (compareAndSetState(0, 1))
      setExclusiveOwnerThread(Thread.currentThread());
    else
      acquire(1);
}
```

Nonfair继承自AQS

没有竞争时

![截屏2023-05-10 下午10.24.15](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-10 下午10.24.15.png)

第一个竞争出现时

![截屏2023-05-20 下午4.02.38](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-20 下午4.02.38.png)

Thread-1执行了

1.CAS尝试将state从 0 改为 1 ，结果失败

2.进入tryAcquire逻辑，这是 state = 1，仍然失败

3.接下来进入 addWaiter 逻辑，构造 Node 队列

- 黄色三角表示WaitStatus状态，0 为默认正常状态
- Node的创建是懒惰的
- 第一个Node成为Dummy(哑元)或哨兵，用来站位，并不关联线程

![截屏2023-05-20 下午4.09.31](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-20 下午4.09.31.png)

当前线程进入`acquireQueued`逻辑

1.`acquireQueued`会在一个死循环中不断尝试获取锁，失败后进入`park`阻塞

2.如果自己是紧邻着head(排第二位)，那么再次`tryAcquire()`尝试获取锁，这时state仍然是1，失败

3.进入 `shouldParkAfterFailedAcquire` 逻辑，将前驱 node ，即 head 的 waitStaus改为-1，这次返回false![截屏2023-05-20 下午4.14.41](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-20 下午4.14.41.png)

4.`shouldParkAfterFailedAcquire` 执行完毕回到 `acquireQueued`，再次reyAcquire 尝试获取锁，这是state仍为1，失败

5.当再次进入 `shouldParkAfterFailedAcquire` 时，前驱 node 的 waitStatus为-1，这次返回true

6.进入 `parkAndCheckInterrupt`，Thread-1 park![截屏2023-05-20 下午4.23.26](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-20 下午4.23.26.png)

再次有多个线程经历上述竞争失败过程

![截屏2023-05-20 下午4.22.34](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-20 下午4.22.34.png)

Thread-0释放锁，进入tryRealse()流程，如果成功

- 设置exclusiveOwnerThread为null
- state = 0

![截屏2023-05-20 下午4.22.21](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-20 下午4.22.21.png)

当前队列不为 null，并且head的 waitStatus = -1，进入 `unparkSuccessor` 流程

找到队列中离 head 最近的一个 Node (没取消的)，`unpark()` 恢复其运行，图中的Thread-1，如果是非公平锁，也可能由别的线程竞争成功



### 2.可重入原理

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            // 如果已经获得了锁，线程还是当前线程，表示发生了重入
            else if (current == getExclusiveOwnerThread()) {
              	// state++
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }

        protected final boolean tryRelease(int releases) {
            // state--
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            // 支持锁重入，只有state=0时，才释放成功
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
  			// ...
}
```



## 3.semaphore

### 概念

信号量，用来限制能同时访问共享资源的线程上限

```java
public static void main(String[] args) {
    Semaphore s = new Semaphore(3);
    for (int i = 0; i < 5; i++) {
      int j = i;
      new Thread(()->{
        try {
          s.acquire();
          log.debug("线程{}，获取一个信号量" + Thread.currentThread().getName());
          try {
            Thread.sleep(1000 + j * 1000);
          }catch (InterruptedException e){
            e.printStackTrace();
          }
        }catch (InterruptedException e){
          e.printStackTrace();
        }finally {
          s.release();
          log.debug("释放1个信号量");
        }
      }).start();
    }
}
```

### 应用

- 使用 Semaphore 限流，在访问高峰期时，让请求线程阻塞，高峰期过去再释放许可，只适合限制单机线程数量，并且仅是==限制线程数，而不是限制资源数(==例如连接数，Tomcat LimitLatch)
- 用 Semaphore 实现简单连接池，对比【享元模式】下的实现(wait/notify)，性能和可读性更好。

```java
// 改进数据库连接池
public class Pool {
    private int size;

    private Connection[] connections;

    // 0 空闲
    // 1 繁忙
    private AtomicIntegerArray states;

    private Semaphore semaphore;

    public Pool(int size) {
        this.size = size;
        semaphore = new Semaphore(size);
        connections = new Connection[size];
        this.states = new AtomicIntegerArray(size);
        for (int i = 0; i < size; i++) {
            connections[i] = new MockConnection();
        }
    }

    public Connection borrow() {
        try {
            semaphore.acquire();
        }catch (InterruptedException e){
            e.printStackTrace();
        }
        for (int i = 0; i < size; i++) {
            if(states.get(i) == 0){
                if (states.compareAndSet(i, 0, 1)) {
                    log.debug("borrow,{}", connections[i]);
                    return connections[i];
                }
            }
        }
        return null;
    }

    public void free(Connection c) {
        for (int i = 0; i < size; i++) {
            if (connections[i] == c) {
                states.set(i, 0);
                semaphore.release();
                log.debug("free{}", connections);
                break;
            }
        }
    }
}
```



### 原理

![截屏2023-05-11 下午4.04.37](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-11 下午4.04.37.png)

假设Thread-1,Thread-3,Thread-4竞争成功，那么其余2个线程就会进入AQS队列park阻塞

![截屏2023-05-11 下午4.16.52](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-11 下午4.16.52.png)

Thread-4释放了permits，状态如下

![截屏2023-05-11 下午4.17.25](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-11 下午4.17.25.png)



## 4.CountDownLatch

### 概念

用来进行线程同步协作，等待所有线程完成倒计

```java
public class CountDownLatchTest {
    public static void main(String[] args) {
        CountDownLatch countDownLatch = new CountDownLatch(1);

        ExecutorService service = Executors.newFixedThreadPool(2);

        service.execute(()->{
            log.debug("begin");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            countDownLatch.countDown();
            log.debug("end...{}", countDownLatch.getCount());
        });

        service.execute(()->{
            try {
                log.debug("wait");
                countDownLatch.await();
                log.debug("run");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
    }
}
```

### **应用**

#### 1.王者荣耀的加载界面

```java
public static void main(String[] args) throws InterruptedException {
    ExecutorService service = Executors.newFixedThreadPool(10);
    CountDownLatch latch = new CountDownLatch(10);
    Random r = new Random();
    String[] strs = new String[10];
    for (int i = 0; i < 10; i++) {
      int cur = i;
      service.execute(()->{
        for (int j = 0; j <= 100; j++) {
          try {
            Thread.sleep(r.nextInt(100));
          } catch (InterruptedException e) {
            e.printStackTrace();
          }
          strs[cur] = j + "%";
          // "/r" 后一次打印覆盖掉前一次打印
          System.out.print("\r" + Arrays.toString(strs));
        }
        latch.countDown();
      });
    }

    latch.await();
    log.debug("游戏开始");
    service.shutdown();
}
```

#### 2.等待多个远程调用结束





## 5.CyclicBarrier

### 概念

循环栅栏，用来进行线程协作，等待线程满足某个技术。构造时设置【计数个数】，每个线程直行到某个需要【同步】的时刻调用await()方法进行等待，当等待的线程数满足【计数个数】时，继续执行

相对于CountDownLatch，CyclicBarrier的计数器置为0后会变为初始值，并且多了一个汇总方法

```java
public static void main(String[] args) {
    // 线程池的大小最好与barrier大小相同
    // 加入线程池大小为3，那么运行的顺序是 t1 t1 t2，即2次t1完成了一个task finish
  	ExecutorService service = Executors.newFixedThreadPool(2);
    CyclicBarrier barrier = new CyclicBarrier(2, ()->{
      log.debug("task finish");
    });

    for (int i = 0; i < 3; i++) {
      service.execute(()->{
        try {
          log.debug("t1开始");
          Thread.sleep(1000);
          barrier.await();
        } catch (InterruptedException | BrokenBarrierException e) {
          e.printStackTrace();
        }
      });

      service.execute(()->{
        try {
          log.debug("t2开始");
          Thread.sleep(2000);
          barrier.await();
        } catch (InterruptedException | BrokenBarrierException e) {
          e.printStackTrace();
        }
      });
    }
}
```



# 9.线程安全的集合

![截屏2023-05-15 下午2.56.10](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-05-15 下午2.56.10.png)

- Blocking 大部分实现基于锁，并提供阻塞方法

- CopyOnWrite 之类的容器修改开销相对较重

- Concurrent 类型的容器

  - 内部多使用 cas 操作，提高吞吐量

  - 弱一致性

    - 遍历时弱一致性。例如，当利用迭代器遍历时，如果容器发生修改，迭代器仍然可以继续进行遍历，这是内容是旧的
    - 求大小弱一致性，size操作未必是100%准确的
    - 读取弱一致性

    遍历时如果发生了修改，对于非安全容器来讲，使用 fail-fast 机制也就是让遍历立刻失败，抛出 ConcurrentModificationException，不再继续遍历



## ConcurrentHashMap

### 文件读写小练习

```java
private static void extracted() {
    int length = ALPHA.length();
    int count = 200;
    List<String> list = new ArrayList<>(length * count);
    for (int i = 0; i < length; i++) {
      char c = ALPHA.charAt(i);
      for (int j = 0; j < count; j++) {
        list.add(String.valueOf(c));
      }
    }
    Collections.shuffle(list);
    for (int i = 0; i < 26; i++) {
      try (PrintWriter out = new PrintWriter(
        new OutputStreamWriter(
          new FileOutputStream("/tmp/" + (i + 1) + ".txt")
        )
      )) {
        String collect = list.subList(i * count, (i + 1) * count)
          .stream()
          .collect(Collectors.joining("\n"));
        out.println(collect);
      } catch (Exception e) {
        e.printStackTrace();
      }
    }
}
    
private static List<String> readFromFile(int id) {
    String path = "/tmp/" + id + ".txt";
    List<String> ans = new ArrayList<>();
    try (BufferedReader reader = new BufferedReader(new FileReader(path))) {
      String line;
      while ((line = reader.readLine()) != null) {
        ans.add(line);
      }
    } catch (Exception e) {
      e.printStackTrace();
    }

    return ans;
}
```

### computeIfAbsent的应用

```java
private static <V> void demo(Supplier<Map<String, V>> supplier, BiConsumer<Map<String, V>, List<String>> consumer) {
    Map<String, V> countMap = supplier.get();
    List<Thread> ts = new ArrayList<>();
    for (int i = 1; i <= 26; i++) {
      int idx = i;
      Thread thread = new Thread(() -> {
        List<String> words = readFromFile(idx);
        consumer.accept(countMap, words);
      });
      ts.add(thread);
    }

    ts.forEach(t -> t.start());
    ts.forEach(t -> {
      try {
        t.join();
      } catch (Exception e) {
        e.printStackTrace();
      }
    });

    System.out.println(countMap);
}
```

#### 导致不安全的原因

虽然 `ConcurrentHashMap` 是线程安全的，但线程安全的指的是其中的每个方法单独都具备原子性，单独使用时线程安全的，如果多个方法一起调用，那么就无法保证原子性

```java
demo(
    () -> new ConcurrentHashMap<String, Integer>(),
    (map, words) -> {
      for (String word : words) {
        // get和put方法分别是安全的，但是一起用并不线程安全
        Integer count = map.get(word);
        int newValue = count == null ? 1 : count + 1;
        map.put(word, newValue);
      }
    }
);
```

#### computeIfAbsent配合LongAdder保证安全

```java
demo(
    () -> new ConcurrentHashMap<String, LongAdder>(),
    (map, words) -> {
      for (String word : words) {
        // 如果缺少一个key，则计算生成一个value，然后将 k v 放入 map
        LongAdder value = map.computeIfAbsent(word, key -> new LongAdder());
        value.increment();
      }
    }
);
```



