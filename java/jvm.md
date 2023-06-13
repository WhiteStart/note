# 内存结构

## 1.程序计数器

**Program Counter Register**

- 作用：记住下一条jvm指令的执行地址
- 特点：
  - 线程私有
  - 不会出现内存溢出

## 2.Java虚拟机栈

Java Virtual Machine Stack

- 每个线程运行时所需要的内存，称为虚拟机栈
- 每个栈由多个栈帧(**Frame**)组成，对应着每次方法调用时所占用的内存
- 每个线程只能有一个**活动栈帧**(位于顶部的栈帧)，对应着当前正在执行的方法 

```java
public class Demo1 {
    public static void main(String[] args) {
        method1();
    }

    private static void method1(){
        method2(1, 2);
    }

    private static int method2(int a,int b){
        int c = a + b;
        return c;
    }
}
```

![截屏2022-08-29 上午9.47.59](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-08-29 上午9.47.59.png)

### 问题辨析

1.垃圾回收是否需要涉及栈内存？

在每次方法调用后，对应栈帧会弹出栈，不需要进行垃圾回收。

2.栈内存的分配越大越好吗？

不是。当内存一定时，每个线程占用的栈内存越大，可执行的线程数量就越小。

3.方法内的局部变量是否线程安全？

- 全局变量y 线程不安全
- 局部变量x 线程安全

```java
static int y = 0;
private static void m1(){
    int x = 0;
    for (int i = 0; i < 3000; i++) {
        x++;
        y++;
    }
    System.out.println("x:" + x + ",y:" + y);
}

public static void main(String[] args) {
    Runnable r = Demo2::m1;

    Thread thread = new Thread(r, "t1");
    Thread thread2 = new Thread(r, "t2");

    thread.start();
    thread2.start();
}
```

- 方法内部局部变量没有逃离方法的作用访问范围，线程安全。
- 局部变量引用了对象，并逃离方法的作用范围，线程不安全。

```java
// 线程安全
private static void m1(){
    StringBuilder sb = new StringBuilder();
    sb.append(1);
    sb.append(2);
    sb.append(3);
    System.out.println(sb);
}

// 线程不安全
private static void m2(StringBuilder sb){
    sb.append(1);
    sb.append(2);
    sb.append(3);
    System.out.println(sb);
}

// 返回了StringBuilder 可能会被其他线程拿到 因此线程不安全
private static StringBuilder m3(){
    StringBuilder sb = new StringBuilder();
    sb.append(1);
    sb.append(2);
    sb.append(3);
    return sb;
}
```



### 内存溢出

1. 栈帧过多导致栈内存溢出

![截屏2022-08-29 下午2.16.55](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-08-29 下午2.16.55.png)

2. 栈帧过大导致栈内存溢出

![截屏2022-08-29 下午2.17.24](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-08-29 下午2.17.24.png)

3. 递归无限循环导致内存溢出

4. 两个类之间循环引用

```java
@Data
class Emp{
    private String name;
    //加入注解忽略一部分，避免循环调用
    @JsonIgnore
    private Dept dept;
}

@Data
class Dept{
    private String name;
    private List<Emp> emps;
}
```

```java
public static void main(String[] args) throws JsonProcessingException {
    Dept d = new Dept();
    d.setName("Market");

    Emp e1 = new Emp();
    e1.setName("zhang");
    e1.setDept(d);

    Emp e2 = new Emp();
    e2.setName("li");
    e2.setDept(d);

    d.setEmps(Arrays.asList(e1, e2));

    // {name: 'Market', emps: [{name:'zhang', dept:{name:'', emps:[]}}]}
    ObjectMapper mapper = new ObjectMapper();, 
    System.out.println(mapper.writeValueAsString(d));
}
```



### 运行诊断

==top：查看所有进程情况==

#### 案例1：cpu占用过高

定位

1. top定位CPU占用高的进程

2. ps命令进一步定位是哪个线程引起的cpu占用过高

- ps H -eo pid,tid,%cpu  ：查看所有线程的pid,tid,cpu
- ==ps H -eo pid,tid,%cpu | grep {进程id}== ：查看制定线程的pid,tid,cpu

3. jstack {进程id}

- 根据线程id找到有问题的线程，进一步定位到问题代码的源码行号

#### 案例2：程序运行迟迟没有结果

程序可能产生了死锁

```java
new Thread(()->{
    synchronized (a){
        try {
            Thread.sleep(2000);
        }catch (InterruptedException e){
            e.printStackTrace();
        }
        synchronized (b){
            System.out.println("I get a and b");
        }
    }
}).start();
Thread.sleep(1000);
new Thread(()->{
    synchronized (b){
        synchronized (a){
            System.out.println("I get a and b");
        }
    }
}).start();
```

定位

1. top查看进程id
2. jstack {线程id}

![截屏2022-08-29 下午4.58.25](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-08-29 下午4.58.25.png)



## 3.本地方法栈

Native Method Stack

- Java调用非java代码的接口

![截屏2022-08-31 下午2.36.15](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-08-31 下午2.36.15.png)

## 4.堆

Heap

- ==通过new关键字，创建的对象都会使用堆内存==

特点

- ==线程共享，堆中的对象都需要考虑线程安全的问题==
- 有==垃圾回收机制==



### 内存溢出

```java
public static void main(String[] args) {
    int i = 0;
    try{
        List<String> list = new ArrayList<>();
        String a = "hello";
        while (true){
            list.add(a);
            a = a + a;
            i++;
        }
    }catch (Throwable e){
        e.printStackTrace();
        System.out.println(i);
    }
}
```

### 堆内存诊断

1.jps工具

- 查看当前系统中有哪些java进程

2.jhsdb jmap工具

- 查看堆内存占用情况

3.jconsole工具

- 图形界面多功能监测工具，可以连续监测



### 案例分析

- 垃圾回收后，内存占用仍然很高
  - 使用==jvisualvm==(==堆转储dump==，进一步对堆的内存信息进行分析)

```java
public class Demo2 {
    public static void main(String[] args) {
        List<Student> students = new ArrayList<>();
        for (int i = 0; i < 200; i++) {
            students.add(new Student());
        }
        try {
            Thread.sleep(100000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class Student{
    private byte[] big = new byte[1024 * 1024];
}
```

![截屏2022-09-01 上午11.23.37](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-09-01 上午11.23.37.png)



## 5.方法区

![截屏2022-09-01 下午3.32.53](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-09-01 下午3.32.53.png)

### 内存溢出

- 1.8以后==元空间==会导致内存溢出
  - ==-XX:MaxMetaspaceSize=8m==

场景

- spring
- mybatis

```java
/**
 * 演示元空间内存溢出
 * -XX:MaxMetaspaceSize=10m
 */
public class Demo extends ClassLoader {
    public static void main(String[] args) {
        int j = 0;
        try {
            Demo test = new Demo();
            for (int i = 0; i < 10000; i++, j++) {
                // 生成类的二进制字节码
                ClassWriter cw = new ClassWriter(0);
                // 版本号，public， 类名， 包名， 父类， 接口
                cw.visit(Opcodes.V1_8, Opcodes.ACC_PUBLIC, "Class" + i, null, "java/lang/Object", null);

                // 返回 byte[]数组
                byte[] code = cw.toByteArray();
                test.defineClass("Class" + i, code, 0, code.length);
            }
        }finally {
            System.out.println(j);
        }
    }
}
```

### 常量池

一张表，虚拟机指令根据这张常量表找到要执行的类名、方法名、参数类型、字面量等信息

- javac Demo.java
- javap -v Demo.class

```shell
Classfile /Users/huangminzhi/Desktop/笔记/java/多线程/src/main/java/com/example/juc/jvm/方法区/常量池/Demo.class
  Last modified 2022-9-1; size 453 bytes
  MD5 checksum 8604f653d607c7dd8d30f522e1fc2450
  Compiled from "Demo.java"
public class com.example.juc.jvm.方法区.常量池.Demo
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #6.#15         // java/lang/Object."<init>":()V
   #2 = Fieldref           #16.#17        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #18            // hello world
   #4 = Methodref          #19.#20        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = Class              #21            // com/example/juc/jvm/方法区/常量池/Demo
   #6 = Class              #22            // java/lang/Object
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               main
  #12 = Utf8               ([Ljava/lang/String;)V
  #13 = Utf8               SourceFile
  #14 = Utf8               Demo.java
  #15 = NameAndType        #7:#8          // "<init>":()V
  #16 = Class              #23            // java/lang/System
  #17 = NameAndType        #24:#25        // out:Ljava/io/PrintStream;
  #18 = Utf8               hello world
  #19 = Class              #26            // java/io/PrintStream
  #20 = NameAndType        #27:#28        // println:(Ljava/lang/String;)V
  #21 = Utf8               com/example/juc/jvm/方法区/常量池/Demo
  #22 = Utf8               java/lang/Object
  #23 = Utf8               java/lang/System
  #24 = Utf8               out
  #25 = Utf8               Ljava/io/PrintStream;
  #26 = Utf8               java/io/PrintStream
  #27 = Utf8               println
  #28 = Utf8               (Ljava/lang/String;)V
{
  public com.example.juc.jvm.方法区.常量池.Demo();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 4: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String hello world
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 6: 0
        line 7: 8
}
SourceFile: "Demo.java"
```

### 运行时常量池

- 常量池是*.class文件中的，==当该类被加载时，它的常量池信息就会放入运行时常量池==，并把里面的符号地址变为真实地址

```java
// 二进制字节码(类基本信息，常量池，类方法定义，包含了虚拟机指令)
public class Demo {
    public static void main(String[] args) {
        // StringTable []
        // 常量池中的信息，都会被加载到运行时常量池中，这时 a b ab 都是常量池中的符号，还没有变为 java 字符串对象
        // ldc #2 会把 a 符号变为 "a" 字符串对象 => StringTable ["a"]
        // ldc #3 ...                        => StringTable ["a" "b"]
        //                                   => StringTable ["a" "b" "ab"]
        String s1 = "a";
        String s2 = "b";
        String s3 = "ab";

        // s1 s2 是两个变量
        // 运行期间 new StringBuilder().append("a").append("b").toString() => new String("ab")
        String s4 = s1 + s2;

        // javac 在 编译期间 的优化，结果已经确定为ab
        String s5 = "a" + "b";
        String s6 = s4.intern();

        System.out.println(s3 == s4);
        System.out.println(s3 == s5);
        System.out.println(s4 == s6);
    }
}
```

### StringTable

- 常量池中的字符串仅是符号，第一次用到时才变为对象
- 利用串池的机制，来避免重复创建字符串对象
- 字符串变量拼接的原理是StringBuilder(1.8)
- 字符串常量拼接的原理是==编译期优化==
- 可以使用intern方法，主动将串池中还没有的字符串对象放入串池

```java
String s = new String("a") + new String("b");

// 将这个字符串尝试放入串池，如果池中有则返回池中的，没有则将该字符串放入再返回
s.intern();
```



#### 垃圾回收

- 配置vmOptions
  - -Xmx10m -XX:+PrintStringTableStatistics -XX:+PrintGCDetails -verbose:gc

```java
public static void main(String[] args) {
    int i = 0;
    try {
        
    }catch (Throwable e){
        e.printStackTrace();
    }finally {
        System.out.println(i);
    }
}
```

- 控制台输出的StringTable信息

```shell
StringTable statistics:
Number of buckets       :     60013 =    480104 bytes, avg   8.000
Number of entries       :       883 =     21192 bytes, avg  24.000
Number of literals      :       883 =     59552 bytes, avg  67.443
Total footprint         :           =    560848 bytes
Average bucket size     :     0.015
Variance of bucket size :     0.015
Std. dev. of bucket size:     0.121
Maximum bucket size     :         2
```

- 增加对象使StringTable溢出

```java
public static void main(String[] args) {
    int i = 0;
    try {
        for (int j = 0; j < 10000; j++) {
            String.valueOf(j).intern();
            i++;
        }
    }catch (Throwable e){
        e.printStackTrace();
    }finally {
        System.out.println(i);
    }
}
```

- StringTable因内存不足进行垃圾回收

```shell
[GC (Allocation Failure) [PSYoungGen: 2048K->496K(2560K)] 2048K->610K(9728K), 0.0021969 secs] [Times: user=0.00 sys=0.00, real=0.00 secs
...
StringTable statistics:
Number of buckets       :     60013 =    480104 bytes, avg   8.000
Number of entries       :      8770 =    210480 bytes, avg  24.000
Number of literals      :      8770 =    438208 bytes, avg  49.967
Total footprint         :           =   1128792 bytes
Average bucket size     :     0.146
Variance of bucket size :     0.162
Std. dev. of bucket size:     0.402
Maximum bucket size     :         3
```

#### 性能调优

1. ==调整-XX:StringTableSize=桶个数==

2. ==考虑将字符串对象是否入池==

- 读取一个有13万个数字的txt文件，并打印StringTable信息
  - -XX:PrintStringTableStatistics

```java
try(BufferedReader reader = new BufferedReader(new InputStreamReader(new FileInputStream("1.txt"), StandardCharsets.UTF_8))) {
    String line = null;
    long start = System.nanoTime();
    while (true){
        line = reader.readLine();
        if(line == null) break;
        line.intern();
    }
    System.out.println("cos:" + (System.nanoTime()-start)/1000000);
}
```

```java
cos:73
StringTable statistics:
// 类似哈希表 桶的个数
Number of buckets       :     60013 =    480104 bytes, avg   8.000
Number of entries       :    132447 =   3178728 bytes, avg  24.000
Number of literals      :    132447 =   7574552 bytes, avg  57.189
Total footprint         :           =  11233384 bytes
// 平均每个桶的size
Average bucket size     :     2.207
Variance of bucket size :     1.746
Std. dev. of bucket size:     1.321
Maximum bucket size     :         8
```

- -XX:StringTableSize=1009

```
cos:285
StringTable statistics:
Number of buckets       :      1009 =      8072 bytes, avg   8.000
Number of entries       :    132447 =   3178728 bytes, avg  24.000
Number of literals      :    132447 =   7574552 bytes, avg  57.189
Total footprint         :           =  10761352 bytes
Average bucket size     :   131.266
Variance of bucket size :    28.149
Std. dev. of bucket size:     5.306
Maximum bucket size     :       143
```

- 将桶大小减少至1009，时间cos明显增加



## 6.直接内存

- 常见于==NIO操作==时，用于数据缓冲区
- 分配回收成本较高，但读写性能高
- ==不受JVM内存回收管理==

#### 基本使用

- 未使用直接内存时，磁盘文件的读入需要经过两块缓冲区

![截屏2022-09-02 下午10.29.50](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-09-02 下午10.29.50.png)

- 使用直接内存

![截屏2022-09-02 下午10.34.33](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-09-02 下午10.34.33.png)

#### 内存溢出

```java
private static final int _100Mb = 1024 * 1024 * 100;

public static void main(String[] args) {
    List<ByteBuffer> list = new ArrayList<>();
    int i = 0;
    try {
        while (true){
            // 分配直接内存 
            ByteBuffer byteBuffer = ByteBuffer.allocateDirect(_100Mb);
            list.add(byteBuffer);
            i++;
        }
    }finally {
        System.out.println(i);
    }
}
```

![截屏2022-09-03 下午4.06.54](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-09-03 下午4.06.54.png)

#### 内存释放

直接内存的释放==不是有jvm负责==的，而是通过底层的==Unsafe类==完成释放。

- 使用Unsafe对象完成直接内存的分配回收，并且回收需要主动调用freeMemory方法
- ByteBuffer的实现类内部，使用了Cleaner(虚引用)来检测ByteBuffer对象，一旦ByteBuffer对象被垃圾回收，那么就会有ReferenceHandler线程通过Cleaner的clean方法调用freeMemory来释放直接内存



# 垃圾回收

## 1.如何判断对象可以回收

### 1.1 引用计数法

- 每当A对象被引用，就令其引用计数+1，当引用计数为0时，进行垃圾回收
- 弊端
  - 当A与B循环引用时，造成内存泄露

![截屏2022-09-03 下午4.25.16](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-09-03 下午4.25.16.png)

### 1.2 可达性分析算法

- java虚拟机中的垃圾回收器采用可达性分析来探索所有存活的对象
- 扫描堆中的对象，看是否能够沿着GC Root对象为起点的引用链找到该对象，==找不到则进行第一次标记并判断该对象是否覆盖finalize()方法或调用过finalize()方法,如果没有，jvm将判定对象已经死亡，并回收==。(类似葡萄串，掉下去的葡萄是可回收的)
  - System Class
  - Native Stack
  - Thread
  - Busy Monitor

finalize()方法效率低下，不推荐使用

### 1.3 引用

- 强引用
- 软引用
- 弱引用
- 虚引用
- 终结器引用

![截屏2022-09-05 上午10.58.24](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-09-05 上午10.58.24.png)

#### 1.强引用

对于A1对象，当B和C的强引用都消失时，A1对象才会被回收

#### 2.软引用和弱引用

对于A2对象，B的强引用消失且发生垃圾回收==发现内存不够时==，软引用的对象A2才会被回收

对于A3对象，当B的强引用消失且发生垃圾回收，弱引用的对象A3会被直接回收

![截屏2022-09-05 上午11.09.05](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-09-05 上午11.09.05.png)

若分配了引用对象，软/弱引用指向的对象被回收后，会进入引用队列，进一步处理以释放内存

#### 3.虚引用

虚引用的对象ByteBuffer被垃圾回收时，虚引用本身会进入引用队列。引用队列查找该虚引用，调用Unsafe.freeMemory方法来清除不归jvm管辖的直接内存，避免内存泄漏。

  ![截屏2022-09-05 上午11.18.36](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-09-05 上午11.18.36.png)

![截屏2022-09-05 上午11.20.00](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-09-05 上午11.20.00.png)

#### 4.终结器引用

- 用以实现对象的finalize()方法，也可以称为终结器引用
- 无需手动编码，其内部配合引用队列使用
- 在GC时，终结器引用入队。由Finalizer线程通过终结器引用找到被引用对象并调用它的finalize()方法，==第二次GC时才能回收被引用对象==



#### 5.总结与案例

1. 强引用

- 只有所有GC Roots对象都不通过`强引用`引用该对象，该对象才能被垃圾回收

2. 软引用

- 仅有软引用引用该对象时，在垃圾回收后，内存仍不足时会再次垃圾回收，回收软引用对象
- 可以配合引用队列来释放软引用自身

```java
public static void soft() {
    // list --> SoftReference --> byte[]

    // 引用队列
    ReferenceQueue<byte[]> queue = new ReferenceQueue<>();

    List<SoftReference<byte[]>> list = new ArrayList<>();
    for (int i = 0; i < 5; i++) {
        // 关联了引用队列，当软引用关联的byte[]被回收时，软引用自己会加入到queue中
        SoftReference<byte[]> ref = new SoftReference<>(new byte[_4MB], queue);
        System.out.println(ref.get());
        list.add(ref);
        System.out.println(list.size());
    }

    // 从队列中获取无用的软引用对象并移除
    Reference<? extends byte[]> poll = queue.poll();
    while (poll != null){
        list.remove(poll);
        poll = queue.poll();
    }

    System.out.println("================");
    for (SoftReference<byte[]> ref : list) {
        System.out.println(ref.get());
    }
}
```

3. 弱引用

- 仅有弱引用引用 该对象时，在垃圾回收时，无论内存是否充足，都会回收弱引用对象
- 可以配合引用队列来释放引用自身

4. 虚引用

- 必须配合引用队列使用，主要配合ByteBuffer使用，被引用对象回收时，会将虚引用入队，由Reference Handler线程调用虚引用相关方法释放直接内存

5. 终结器引用

- 无需手动编码，内部配合引用队列使用。在垃圾回收时，终结器引用入队(被引用对象暂时没有被回收)，再由Finalizer线程通过终结器引用找到被引用对象并调用它的finalize方法，==第二次GC==时才能被回收的对象



## 2.STW

stop the world，实在垃圾回收算法执行过程中，需要将jvm冻结的一种状态。==在STW状态下，JAVA所有线程都是停止的(除GC线程)。native方法可以执行，但不能与jvm交==互。==GC算法优化的重点就在于减STW，也是GC优化的重点==。



## 3.垃圾回收算法

### 3.1 标记清除

标记不与GC Root链接的内存，直接清除

- 优点：垃圾清除速度快

- 缺点：内存空间不连续，产生内存碎片难以利用

![截屏2022-09-06 下午2.00.57](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-09-06 下午2.00.57.png)

### 3.2 标记整理

标记不与GC Root链接的内存清除后进行整理，使之连续

- 优点：避免了内存碎片
- 缺点：整理时需耗费时间，降低性能

![截屏2022-09-06 下午2.06.52](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-09-06 下午2.06.52.png)  

### 3.3 复制

将FROM区中的需要保留的内存复制到TO区，清除FROM区的垃圾，然后交换两区

- 优点：避免了内存碎片，同时速率快
- 缺点：需要双倍的内存空间

![截屏2022-09-06 下午2.12.56](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-09-06 下午2.12.56.png)

![截屏2022-09-06 下午2.10.52](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-09-06 下午2.10.52.png)

![截屏2022-09-06 下午2.15.02](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-09-06 下午2.15.02.png)

## 4.分代垃圾回收

![截屏2022-09-06 下午2.32.00](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-09-06 下午2.32.00.png)

- 对象首先分配在伊甸园区域
- 新生代空间不足时，触发minor gc，伊甸园和from存活的对象使用copy复制到to中，存活的对象年龄加一，交换from区和to区
- minor gc会引发stop the world，暂停其他用户的线程，等垃圾回收结束，用户线程才恢复运行
- 当对象寿命超过阈值时，会晋升至老年代，最大寿命是15(4bit)
- 当老年代空间不足时，先尝试minor gc，如果之后空间仍然不足，那么触发full gc，STW时间更长



#### 3.1 相关VM参数

|       含义       | 参数                                                         |
| :--------------: | :----------------------------------------------------------- |
|    堆初始大小    | -Xms                                                         |
|    堆最大大小    | -Xmx或-XX:HeapMaxSize=size                                   |
|    新生代大小    | -Xmn或-XX:NewSize=size -XX:MaxNewSize=size                   |
| 幸存者比例(动态) | -XX:InitialSurvivorRatio=ratio 和 -XX:+UseAdaptiveSizePolicy |
|    幸存者比例    | -XX:SurvivorRatio=ratio                                      |
|     晋升阈值     | -XX:MaxTenuringThreshold=threshold                           |
|     晋升详情     | -XX:+PrintTenuringDistribution                               |
|      GC详情      | -XX:PrintGCDetails -verbose:gc                               |
| FullGC前MinorGC  | -XX:+ScavengeBeforeFullGC                                    |

```java
Heap
 def new generation   total 9216K, used 2068K [0x00000007bec00000, 0x00000007bf600000, 0x00000007bf600000)
  eden space 8192K,  25% used [0x00000007bec00000, 0x00000007bee05290, 0x00000007bf400000)
  from space 1024K,   0% used [0x00000007bf400000, 0x00000007bf400000, 0x00000007bf500000)
  to   space 1024K,   0% used [0x00000007bf500000, 0x00000007bf500000, 0x00000007bf600000)
 tenured generation   total 10240K, used 0K [0x00000007bf600000, 0x00000007c0000000, 0x00000007c0000000)
   the space 10240K,   0% used [0x00000007bf600000, 0x00000007bf600000, 0x00000007bf600200, 0x00000007c0000000)
 Metaspace       used 2985K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 327K, capacity 388K, committed 512K, reserved 1048576K
```



## 5.垃圾回收器

### 1.串行

- 标记整理算法

- 单线程
- 堆内存较小，适合个人电脑

| -XX:+UseSerialGC     (Serial + SerialOld) | 打开串行回收 |
| ----------------------------------------- | ------------ |

![截屏2022-09-07 上午8.24.09](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-09-07 上午8.24.09.png)

### 2.吞吐量优先

- 标记整理算法

- 多线程
- 堆内存较大，多核CPU
- 单位时间内STW时间最短

| -XX:+UseParallelGC ~ -XX:+UseParallelOldGC | 1.8中默认开启，开启一个命令则另一个自动开启        |
| ------------------------------------------ | -------------------------------------------------- |
| -XX:+UseAdaptiveSizePolicy                 | 动态调整Eden区大小，晋升阈值                       |
| -XX:GCTimeRatio=ratio                      | 调整垃圾回收时间与总时间的占比 1/(1+ratio)，一般19 |
| -XX:MaxGCPauseMillis=ms                    | 最大暂停毫秒数，默认200ms                          |
| -XX:ParallelGCThreads=n                    | 并行进行垃圾回收的线程数目                         |

![截屏2022-09-07 上午8.29.16](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-09-07 上午8.29.16.png)

### 3.响应时间优先

- 多线程

- 堆内存较大，多核CPU

- 尽可能让STW单次时间最短

- 内存碎片过多会导致并发失败，==退化为SerialOld==,致使垃圾回收时间变长

  - -XX:+UseConcMarkSweepGC(==老年代==) ~ -XX:+UseParNewGC(新生代)   ~ SerialOld(并发失败CMS退化为串行垃圾回收)
  - -XX:ParallGCThreads=n(并行线程数，一般等于核心数) ~ -XX:ConcGCThreads=threads(并发线程数，一般为核数的1/4)
  - -XX:CMSInitiatingOccupancyFraction=percent(老年代内存占比percent时就进行垃圾回收)(并发清理可能会产生垃圾，下次才能回收，导致累积)
  - -XX:+CMSScavengeBeforeRemark(在重新标记前，先对新生代进行一次回收)

  

![截屏2022-09-07 上午8.46.22](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-09-07 上午8.46.22.png)

- 初始标记只标记根对象
- 并发标记时其他线程可能产生干扰，于是重新标记
- 初始标记和重新标记产生STW



### 4.G1垃圾回收器

![截屏2022-09-07 下午1.57.12](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-09-07 下午1.57.12.png)

#### 目标

- 在延迟可控的情况下，尽可能高的吞吐量

#### 分区Region

- <font color=red>整体上标记整理算法，region之间是复制算法</font>

- 使用G1收集器时，它将整个Java堆划分成约2048个大小相同的独立Region块，每个Region块大小根据堆空间的实际大小而定，整体被控制在1MB到32MB之间，且为2的N次幂，即1MB,2MB, 4MB, 8MB, 1 6MB, 32MB。可以通过-XX :G1HeapRegionSize设定。==所有的Region大小相同，且在JVM生命周期内不会被改变==。

- 虽然还保留有新生代和老年代的概念，但新生代和老年代不再是物理隔离的了,它们都是一部分Region (不需要连续)的集合。==通过Region的动态分配方式实现逻辑上的连续==。
- G1新增的H块，主要用于存储大对象，如果超过1.5region,就放到H。

#### 适用场景

- 同时注重吞吐量(Throughput)和低延迟(Low latency)，默认暂停目标:200ms
- 超大堆内存，会将堆划分为多个大小相等的Region

- 在下面的情况时，使用G1可能比CMS好:
  - ①超过50%的Java堆被活动数据占用
  - ②对象分配频率或年代提升频率变化很大
  - ③GC停顿时间过长(长于0.5至1秒)

#### 相关VM参数

-XX:+UseG1GC

==-XX:G1HeapRegionSize=size==

==-XX:MaxGCPauseMillis=time==

#### 回收阶段

![截屏2022-09-07 上午10.24.44](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-09-07 上午10.24.44.png)

##### 1.Young Collection

- 新生代垃圾回收使用==复制算法==将幸存对象放如幸存区，会STW

![截屏2022-09-07 上午10.26.13](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-09-07 上午10.26.13.png)

![截屏2022-09-07 上午10.25.14](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-09-07 上午10.25.41.png)

- 多次回收后仍然存在的将会晋升到老年代(红色箭头表示复制算法)

![截屏2022-09-07 上午11.00.55](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-09-07 上午11.00.55.png)

##### 2.Young Collection + CM

- 在Young GC时会进行GC Root的初始标记
- 老年==代占用堆空间比例达到阈值时，进行并发标记==(不会STW)，由JVM参数决定

-XX:InitiatingHeapOccupancyPercent=percent(默认45%)

​       // 初始化堆占用比

##### 3.Mixed Collection

会对E、S、O进行全面垃圾回收

- 最终标记(Remark)会STW
- 拷贝存活(Evacuation)会STW

-XX:MaxGCPauseMillis=ms

在有限的暂停时间内，优先回收价值最大的Region(内存占用多)

![截屏2022-09-07 上午11.10.27](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-09-07 上午11.10.27.png)

​																红色箭头表示复制算法

#### Full GC

- SerialGC
  - 新生代内存不足发生的垃圾手机 - minor gc
  - 老年代内存不足发生的垃圾收集 - full gc
- ParallelGC
  - 新生代内存不足发生的垃圾收集 - minor gc
  - 老年代内存不足发生的垃圾收集 - full gc
- CMS
  - 新生代内存不足发生的垃圾收集 - minor gc
  - 老年代内存不足

- G1
  - 新生代内存不足发生的垃圾收集 - minor gc
  - 老年代内存不足
    - 老年代内存占比达到45%，触发并发标记与混合收集
    - 若回收速度超过了垃圾产生速度，仍处于并发垃圾收集
    - 若回收速度低于垃圾产生速度，并发收集失败退化为串行，full gc

#### JDK 8u20字符串去重

- 优点：节省大量内存
- 缺点：略微多占用了cpu时间，新生代回收时间略微增加

-XX:+UseStringDeduplication

```java
String s1 = new String("hello");  
String s2 = new String("hello");
```

- 将所有新分配的字符串放入一个队列
- 当新生代回收时，G1并发检查是否有字符串重复
- 如果他们值一样，让他们引用同一个char[]
- 与String.intern()不同：
  - String.intern()关注的是字符串对象
  - 而字符串去重关注的是char[]
  - 在jvm内部，使用了不同的字符串表

#### JDK 8u40并发标记类卸载

所有对象经过并发标记后，就能知道哪些类不再被使用。当一个类加载器的所有类都不再使用，则卸载它加载的所有类

-XX:+ClassUnloadingWithConcurrentMark(默认启用)





- CMS

  CMS（Concurrent Mark Sweep）收集器是一种以==获取最短回收停顿时间为目标==的收集器。它非常符合在注重用户体验的应用上使用。

  CMS（Concurrent Mark Sweep）收集器是 HotSpot 虚拟机第一款真正意义上的并发收集器，它第一次实现了让垃圾收集线程与用户线程（基本上）同时工作。

  - **初始标记：** 暂停所有的其他线程，并记录下直接与 root 相连的对象，速度很快 ；
  - **并发标记：** 同时开启 GC 和用户线程，用一个闭包结构去记录可达对象。但在这个阶段结束，这个闭包结构并不能保证包含当前所有的可达对象。因为用户线程可能会不断的更新引用域，所以 GC 线程无法保证可达性分析的实时性。所以这个算法里会跟踪记录这些发生引用更新的地方。
  - **重新标记：** 重新标记阶段就是为了修正并发标记期间因为用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段的时间稍长，远远比并发标记阶段时间短
  - **并发清除：** 开启用户线程，同时 GC 线程开始对未标记的区域做清扫。

  ![截屏2023-03-24 下午9.30.41](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-03-24 下午9.30.41.png)

  - <font color=red>优点</font>
    - 并发收集
    - 低停顿
  - <font color=red>缺点</font>
    - CPU资源敏感
    - 无法处理浮动垃圾
    - 标记-清除算法会导致大量内存碎片

- G1

  G1 (Garbage-First) 是一款面向服务器的垃圾收集器,主要针对配备多颗处理器及大容量内存的机器. 以极==高概率满足 GC 停顿时间要求的同时,还具备高吞吐量性能特征==。

  - **并行与并发**：==G1 能充分利用 CPU、多核环境下的硬件优势==，使用多个 CPU（CPU 或者 CPU 核心）来缩短 Stop-The-World 停顿时间。部分其他收集器原本需要停顿 Java 线程执行的 GC 动作，G1 收集器仍然可以通过并发的方式让 java 程序继续执行。

  - **分代收集**：虽然 G1 可以不需要其他收集器配合就能独立管理整个 GC 堆，但是还是保留了分代的概念。

  - **空间整合**：与 CMS 的“标记-清除”算法不同，G1 从整体来看是基于“标记-整理”算法实现的收集器；从局部上来看是基于“标记-复制”算法实现的。

  - **==可预测的停顿==**：这是 G1 相对于 CMS 的另一个大优势，降低停顿时间是 G1 和 CMS 共同的关注点，但 G1 除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为 M 毫秒的时间片段内。

  

  G1 收集器的运作大致分为以下几个步骤：

  - **初始标记**
  - **并发标记**
  - **最终标记**
  - **筛选回收**

  **G1 收集器在后台维护了一个优先列表，每次根据允许的收集时间，优先选择回收价值最大的 Region(这也就是它的名字 Garbage-First 的由来)** 。这种使用 Region 划分内存空间以及有优先级的区域回收方式，保证了 G1 收集器在有限时间内可以尽可能高的收集效率（把内存化整为零）。

  

### CMS与G1的区别简述

|                    | CMS                                                          | G1                                                           |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 算法               | **标记-清除**。”**低停顿**“垃圾回收器。                      | **标记-整理**。”**低延迟**“垃圾回收器。                      |
| 内存分配与回收策略 | 空闲列表：快速分配内存，当一个对象被回收时，内存不会马上被回收，而是加入到一个空闲列表中，等待下次分配时使用。可以减少内存碎片，但会增加分配时间 | 局部回收：即在每次垃圾回收时只清理一部分内存。它会根据堆内存中各个Region的使用情况来选择需要回收的区域，以此最小化每次垃圾回收的影响范围。 |
