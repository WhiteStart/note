# 数据基本类型与包装类型

| 数据类型 | 取值范围     | 占用字节                |
| -------- | ------------ | ----------------------- |
| byte     | -128~127     | 1                       |
| short    | -32768~32767 | 2                       |
| int      | -2^31~2^31-1 | 4                       |
| float    |              | 4                       |
| long     |              | 8                       |
| double   |              | 8                       |
| char     | 0~2^16-1     | 2                       |
| boolean  |              | 没有明确定义，依赖于jvm |

## 1.1基本类型与包装类型的区别

- 成员变量包装类型不赋值就是 null ，而基本类型有默认值且不是 null。

- 包装类型可用于泛型，而基本类型不可以。

- 基本数据类型的局部变量存放在 Java 虚拟机栈中的局部变量表中，基本数据类型的成员变量（未被 static 修饰 ）存放在 Java 虚拟机的堆中。包装类型属于对象类型，我们知道几乎所有对象实例都存在于堆中。

- 相比于对象类型， 基本数据类型占用的空间非常小。

## 1.2 包装类型的缓存机制(缓冲池)

```java
// Integer设置了缓存类,Byte,Short,Long,Character也有
// jvm默认设置high为127，如果在-128~127范围内直接调用缓冲池中的，用于提高性能
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
      int h = 127;
      String integerCacheHighPropValue =
        sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
      if (integerCacheHighPropValue != null) {
        try {
          int i = parseInt(integerCacheHighPropValue);
          i = Math.max(i, 127);
          h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
        } catch( NumberFormatException nfe) {
          // If the property cannot be parsed into an int, ignore it.
        }
      }
      high = h;

      cache = new Integer[(high - low) + 1];
      int j = low;
      for(int k = 0; k < cache.length; k++)
        cache[k] = new Integer(j++);

      // range [-128, 127] must be interned (JLS7 5.1.7)
      assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}

public static Integer valueOf(int i) {
    if(i >= -128 && i <= IntegerCache.high)
      return IntegerCache.cache[i + 128];
    else
      return new Integer(i);
}
```

```java
// i1,i2在缓冲池的默认范围内
Integer i1 = 100;
Integer i2 = 100;
Integer i3 = 200;
Integer i4 = 200;

// ==比较地址
// .equals()先比较地址 不相同再比较数值
System.out.println(i1==i2); // true
System.out.println(i3==i4); // false
```

```java
Integer d1 = new Integer(2022);
int d2 = 2022;
// int类型与Integer类型比较时, 先将Integer拆箱, 再比较值
System.out.println(d1==d2); // true
```

## 1.3装/拆箱

- 装箱 将数据类型装箱称数据对象
  -  值类型 => 引用类型

```java
// Integer i = Integer.valueOf(10)
Integer i = 10;
```

- 拆箱 将数据对象拆箱称成数据类型
  - 引用类型 => 值类型 

```java
// int n = Number.intValue(i)
int n = i;
```

- 实例

```java
List<Integer> list = new ArrayList<>();
// 自动装箱
list.add(3);
// 自动拆箱
int x = list.get(0);
```

# String

## 1.1 版本的区别

```
Java 8 中，String 内部使用 char 数组存储数据。

Java 9 之后，String 类的实现改用 byte 数组存储字符串，同时使用 coder 来标识使用了哪种编码。
```

## 1.2 不可变的好处

```java
1.可以缓存 hash 值
2.String Pool 的需要。如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。
3.安全性。String 经常作为参数，String 不可变性可以保证参数不可变。如网络传输
4.线程安全
```

## 1.3 字符串的存储

```java
// s1存储在堆上的字符串常量池中
String s1 = "hello";
// jvm检查堆中是否有s2的对象值，如果有就引用
String s2 = "hello";
// 在堆中普通区域创建S3
String s3 = new String("hello");
// 在堆中普通区域创建S4
String s4 = new String ("hello");

// true
System.out.println(s1 == s2);
// false
System.out.println(s1 == s3);
// false
System.out.println(s3 == s4);

// 目的: 最小化在堆上存储具有重复值的字符串对象所造成的冗余和内存浪费/过度使用
```

![截屏2022-06-09 下午4.05.01](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-06-09 下午4.05.01.png)

### 2.4 intern方法

- 当一个字符串调用 intern() 方法时，如果 常量池 中已经存在一个字符串和该字符串值相等（使用 equals() 方法进行确定），那么就会返回 常量池 中字符串的引用；否则，就会在 常量池 中添加该字符串，并返回这个字符串的引用。

==简单理解为将当前字符串入池(StringTable)==

```java
// 堆中创建s3引用指向的对象"11"，常量池中创建对象"1"
String s3 = new String("1") + new String("1");
// 常量池中不存在"11"，存储堆中"11"的引用，指向s3引用的对象
s3.intern();
// 常量池中存在指向s3引用对象的一个引用
String s4 = "11";
// s4引用与s3引用指向相同
System.out.println(s3 == s4);
```

```java
// 堆中创建s引用指向的对象"1"，常量池中创建对象"1"
String s = new String("1");
// 常量池中存在"1"，因此没有存储堆中"11"引用
// 相当于没有赋值，没有调用
s.intern();
// 生成一个s2的引用指向常量池中的"1"对象
String s2 = "1";
System.out.println(s == s2);
```

```java
String a = "a" + new String("b");
String b = new String("ab");
String c = "a"+"b";
String e = "b";
String f = "a" + e;

// false
System.out.println(b.intern() == a);
// true
System.out.println(b.intern() == c);
// false
// 字符串相加，都是静态字符串的结果会加到常量池中，含有变量(f)则不会
System.out.println(b.intern() == f);
```

### 2.5 StringBuffer与StringBuilder

```
操作少量的数据: 适用String
单线程操作字符串缓冲区下操作大量数据: 适用StringBuilder
多线程操作字符串缓冲区下操作大量数据: 适用StringBuffer
```



# 关键字

## final

```java
// 数据：声明数据为常量，可以是编译时常量，也可以是在运行时被初始化后不能被改变的常量。
// 对于基本类型，final 使数值不变；
// 对于引用类型，final 使引用不变，也就不能引用其它对象，但是被引用的对象本身是可以修改的。

// 方法：声明方法不能被子类重写。
// 类:声明类不允许被继承。
// private 方法隐式地被指定为 final，如果在子类中定义的方法和基类中的一个 // private 方法签名相同，此时子类的方法不是重写基类方法，而是在子类中定义了一个新的方法
```

```java
String a = "hello2"; 　  
String b = "hello";       
String c = b + 2;       
// false
System.out.println((a == c));
```

```java
String a = "hello2";   　
final String b = "hello";       
String c = b + 2;       
// true
// 被final修饰的变量会在class常量池中保存一个副本
System.out.println((a == c));
```

## static

==成员变量与静态变量的区别==

|              |             成员变量             |           静态变量           |
| :----------: | :------------------------------: | :--------------------------: |
|   生命周期   |  随对象的创建(销毁)而存在(消失)  | 随类的加载(消失)而存在(消失) |
|   调用方式   |         只能通过对象调用         |     可以被对象或类名调用     |
| 数据存储位置 | 堆内存的对象中，是对象的特有数据 |  方法区(共享数据区)的静态区  |

```java
public class A {
    // 实例变量 每创建一个类实例就会产生一个实例变量，与该实例同生共死
    private int x;         
    // 静态变量 类的所有实例共享一个静态变量，可通过类名访问，在内存中只存在一份
    private static int y;  
}
```

- 静态方法在类加载的时候就存在了，它不依赖于任何实例，所以静态方法必须有实现。
- 静态方法只能访问所属类的静态字段和静态方法，方法中不能有 this 和 super 关键字，这两个关键字与具体对象关联
- 静态语句块在类初始化时运行一次。

```java
public class OuterClass {

    class InnerClass {
    }

    static class StaticInnerClass {
    }

    public static void main(String[] args) {
        // InnerClass innerClass = new InnerClass(); // 'OuterClass.this' cannot be referenced from a static context
        OuterClass outerClass = new OuterClass();
        InnerClass innerClass = outerClass.new InnerClass();
        StaticInnerClass staticInnerClass = new StaticInnerClass();
    }
}

```

### 类的初始化顺序

```java
父类（静态变量、静态语句块）
子类（静态变量、静态语句块）
父类（实例变量、普通语句块）
父类（构造函数）
子类（实例变量、普通语句块）
子类（构造函数）
```



## default

- 使用default关键字，可以==在接口中定义具体的方法==
- 方便在给接口添加新方法时，不影响已有实现

```java
// 接口中可以有default方法与static方法
public interface A {
    default void defaultMethod(){
        System.out.println("defaultMethod");
    }

    void common();
}
```

```java
public class Test implements A{
    @Override
    public void common() {
        System.out.println("common");
    }

    public static void main(String[] args) {
        Test test = new Test();
        test.common();
        test.defaultMethod();
    }
}
```



# 深拷贝与浅拷贝

- 浅拷贝：对于基本类型赋值不变，对于引用类型则会得到相同子对象的另一个引用(即原对象和克隆的对象仍然会共享一些信息)。而如**String**这样的**不可变类**则也不会改变。

  ![截屏2022-07-23 上午12.04.15](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-07-23 上午12.04.15.png)

```java
Student A = new Student("A", new Subject("subject"), 18);
// 得到了相同子对象的另一个引用
Student copy = A;
A.setAge(20);
System.out.println(copy);
```

调用clone方法前必须先:

- 实现 Cloneable 接口
- 重新定义 clone 方法，并指定 public 访问修饰符

```java
@Data()
public class Student implements Cloneable{
    private String name;
    private Subject subject;
    private int age;

    public Student(String name, Subject subject, int age) {
        this.name = name;
        this.subject = subject;
        this.age = age;
    }

    // 重写clone方法，并将返回值改为Student
    @Override
    public Student clone() throws CloneNotSupportedException {
        return (Student) super.clone();
    }
}

@Data
public class Subject{
    private String name;

    public Subject(String name) {
        this.name = name;
    }
}
```

```java
Student A = new Student("A", new Subject(""), 18);
Student light = A.clone();
System.out.println(A);
System.out.println(light);
// A => Student(name=A, subject=Subject(name=subject), age=18)
// light => Student(name=A, subject=Subject(name=subject), age=18)

A.setName("newA");
A.getSubject().setName("123");
A.setAge(20);
System.out.println(A);
System.out.println(light);
// A => Student(name=newA, subject=Subject(name=123), age=20)
// light => Student(name=A, subject=Subject(name=123), age=18)
```

1. **基本类型age，不可变类型String不受影响**
2. **引用类型Subject改变**

![截屏2022-07-23 上午12.28.13](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-07-23 上午12.28.13.png)

- 深拷贝:通常子对象都是可以变的。因此如果想要拷贝的内容无关联，必须建立深拷贝来同时克隆所有子对象。

  ```java
  @Override
  public Student clone() throws CloneNotSupportedException {
      Student clone = (Student) super.clone();
      // ？ Non-static method 'clone()' cannot be referenced from a static context
      clone.subject = Subject.clone();
      return clone;
  }
  ```



# 重写与重载

## 1.重写

- 子类方法的访问权限必须>=父类
- 子类方法的返回类型必须是父类方法返回类型或子类型
- 子类方法抛出的异常类型必须是父类抛出异常类型或子类型

## 2.重载

- 存在于同一个类中。方法名相同，但参数类型，个数，顺序至少一个不同。
- ==返回值不同，其它都相同不算是重载==

```java
public class Test{
		public void run(){}
    public void run(String a){}
    public void run(int a,int b){}
}
```

```java
// 重载方法的优先级，char->int->long->float->double ->Character -> Serializable -> Object
public static void sayHello(int arg) {
  	System.out.println("this is int " +arg);
}

public static void sayHello(char arg) {
  	System.out.println("this is char " +arg);
}

public static void main(String[] args) {
    // => this is char a,将char换为long，则输出this is int 97
  	sayHello('a'); 
}
```

# 多态

```java
// 一个对象具有多种状态，表现为父类的引用指向子类的实例
// 多态的必要条件 1.继承 2.重写 3.向上转型
class Cat extends Animal{
    public void eat(){
        System.out.println("我吃鱼");
    }
}

class Dog extends Animal{
    public void eat(){
        System.out.println("我吃骨头");
    }

    public void run(){
        System.out.println("我会跑");
    }
}
```

## 1. 向上转型

- 成员变量：==编译看左边，执行看左边==
- 成员方法：==编译看左边，执行看右边==

成员方法有重写，而成员变量没有

```java
public class Animal {
    public void eat(){
        System.out.println("animal eating...");
    }

    public static void main(String[] args) {
        Animal animal = new Dog(); // 向上转型，子类单独定义的方法会丢失，即animal.run()报错
        animal.eat(); //我吃骨头
    }
}
```

**向上转型的好处**

- 减少重复代码
- 提高系统扩展性

## 2. 向下转型

- 向下转型的前提是父类对象指向的是子类对象(即向下转型之前，需先向上转型)
- 向下转型只能转化为本类对象



# 泛型

```java
public interface Test<T> {
    T next();
}

public class A{
    public <T> T run(T a){
        return a;
    }
}
```

```java
public class HttpResponse<T> {
    private boolean success;
    private String errMsg;
    private Integer errCode;
    private T data;
    
    // 泛型方法
    <T> HttpResponse<T> success(T data);
  
    HttpResponse<Void> failure();
}
```

## 原始类型

```java
// 原始类型用第一个限定的类型变量来替换， 如果没有给限定类型就用object替换
public class Interval<T extends Comparable & Serializable> implements Serializable{
  private T lower;
  private T upper;
  
  public Interval(T first, T second){
    ......
  }
}

// 原始类型
public class Interval implements Serializable{
  private Comparable lower;
  private Comparable upper;
  ......
}

// 若改成Serializable & Comparable,则原始类型会变成Serializable
// 为了提高效率，应该将标签接口(没有方法的接口)放在列表的末尾
```

## 类型擦除

```java
// 类型擦除与多态的冲突和解决办法
class Pair<T> {  

    private T value;  

    public T getValue() {  
        return value;  
    }  

    public void setValue(T value) {  
        this.value = value;  
    }  
}

// 子类继承父类，但是编译后多态类型被擦除，因为Pair中的类型都是Object，而子类的类型却是Date
// jvm中运用桥方法解决了这个冲突
class DateInter extends Pair<Date> {  

    @Override
    public void setValue(Date value) {  
        super.setValue(value);  
    }  

    @Override
    public Date getValue() {  
        return super.getValue();  
    }  
}
```

```java
// 静态方法和静态变量不可以使用泛型类所声明的泛型类型参数
public class Test2<T> {    
    public static T one;   //编译错误    
    public static  T show(T one){ //编译错误    
        return null;    
    }    
}

public static <T> T show(T one){ //这是正确的    
    return null;    
}   
```

## ？和 T 的区别

```java
public <T> void test(){
    List<T> list = new ArrayList<>();
    List<?> list2 = new ArrayList<>();
  
    T t = operate();
  
    // 不可以
    ? t = operate();
}
// T 是一个 确定 的类型，通常用于泛型类和泛型方法的定义，？是一个 不确定 的类型，通常用于泛型方法的调用代码和形参，不能用于定义类和泛型方法。
```

### 1.通过 T 来确保泛型参数的一致性

```java
// first与second的参数T是一样的
public <T extends Number> void test(List<T> first, List<T> second);

// first与second的参数可以不一样
public <? extends Number> void test2(List<?> first, List<?> second);
```

### 2.类型参数可以多重限定而通配符不行

```java
public <T extends Number & Serializable> T test(){}
```

### 3.通配符可以使用超类限定而类型参数不行

```java
T extends A
  
? extends A
? extends B
```

## Class<T>和Class<?>的区别

```java
public class Test{
// 可以
public Class<?> clazz;
// 不可以，因为 T 需要指定类型
public Class<T> clazzT;
}
```

```java
public class Test<T> {
    public Class<?> clazz;
    // 不会报错
    public Class<T> clazzT;
}
```

# 序列化

- **序列化**： 将数据结构或对象转换成二进制字节流的过程
- **反序列化**：将在序列化过程中所生成的二进制字节流的过程转换成数据结构或者对象的过程

==序列化保存的是对象的状态，静态变量属于类的状态，不会改变==

```java
public class Test Implements Serializable{
		private String name;
    // 阻止实例中那些用此关键字修饰的的变量序列化；当对象被反序列化时，被 transient 修饰的变量值不会被持久化和恢复
    pirvate transient String password;
}

// 1.transient 只能修饰变量，不能修饰类和方法。
// 2.transient 修饰的变量，在反序列化后变量值将会被置成类型的默认值。例如，如果是修饰 int 类型，那么反序列后结果就是 0。
// 3.static 变量因为不属于任何对象(Object)，所以无论有没有 transient 关键字修饰，均不会被序列化。


// 思考用途
1.传给前端时给不需要的字段加上？避免写一个新的model
```



# 内部类

**广泛意义上的内部类一般来说包括这四种：成员内部类、局部内部类、匿名内部类和静态内部类**

- ==内部类可以访问该(外部)类的所有变量或方法==

- ==对同包中的其他类隐藏==
-  方便编写线程代码

## 1.成员内部类

- 可以==看做外部类的一个成员==，即可以用private、public等修饰

### 1.1 静态成员内部类

- static修饰的类那它一定是内部类(创建不需要依赖于外围类的对象)

- 不能使用任何外围类的非static成员变量和方法
- 静态内部类内允许有static属性、方法

```java
// 静态内部类实现单例
public class Singleton {
    private Singleton(){

    }

    public static class Holder{
        private static final Singleton instance = new Singleton();
      
      	public void t1(){
            System.out.println("可以定义非静态方法");
        }

        public static void t2(){
            System.out.println("可以定义静态方法");
        }
    }

    public static Singleton getInstance(){
        return Holder.instance;
    }
}
```

### 1.2 非静态成员内部类

- 当内部类有与外部类同名的变量或方法时，默认访问内部类的，如果想调用外部类的需要外部类.this.sth

```java
public class OuterClass {
    private double radius;
    public static int count = 1;
    public String same = "out";

    public OuterClass(double radius) {
        this.radius = radius;
    }

    private InnerClass getInnerInstance(){
        return new InnerClass();
    }

    class InnerClass{

        public InnerClass() {}

        // 可以直接访问外部类的所有属性和方法
        public void draw(){
            System.out.println(radius);
            System.out.println(count);
        }
      
        public void same(){
            System.out.println("内外恰好同名的成员变量或方法:" + same);
            System.out.println("调用外部同名的方法:" + OuterClass.this.same);
        }
    }
}
```

- 成员内部类依附外部类而存在，==要创建该类的对象，前提要有外部类对象==
- 外部类可以访问内部类的所有，但需要==先创建内部类的实例==

```java
OuterClass out = new OuterClass(2);
// 必须先有外部类，才能创建内部类的实例
InnerClass in = out.getInnerInstance();
InnerClass in2 = out.new InnerClass();
```

- 成员内部类中的static方法

```java
// 报错 Static declarations in inner classes are not supported at language level '8'
// 非静态成员内部类要依赖外部类，不能有static变量
public static String test = "test";

// *** static final同时修饰的(基本类型或String类型的字段)存放在常量池中，不涉及类的加载。 ***
public static final int b = 2;
```



## 2.局部内部类???

- 定义在一个方法或者一个作用域里面的类
- 和成员内部类的区别在于**局部内部类的访问仅限于方法内或者该作用域内**。
- 类似方法中的局部变量，不能用修饰符修饰

```java
public static void test() {
    // 局部内部类
    class InnerClass {
        public void say(String str) {
            System.out.println(str);
        }
    }
  	InnerClass innerClass = new InnerClass();
    innerClass.say("局部内部类");
}
```

## 3.匿名内部类

- 匿名内部类用于继承其他类或是实现接口，并不需要增加额外的方法，只是对继承方法的实现或是重写
- 匿名内部类在编译的时候由系统自动起名为Outter$1.class



# lambda表达式

## 1.函数式接口

```java
// 函数式接口
@FunctionalInterface
public interface ConsumerInterface<T> {
    // 没有具体的功能，可以自己根据需求实现的灵活接口
    void consumerInterface(T t);
}
```

```java
public class Main<T> {
    private List<T> list;

    // 参数是一个函数式接口
    public void traverse(ConsumerInterface<T> function){
        // 遍历
        for(T t:list){
            // 根据函数式接口的功能运行
            function.consumerInterface(t);
        }
    }

    public static void main(String[] args) {
        Main<String> m = new Main<>();
        // lambda表达式，传入了一个函数式接口
        m.traverse(s -> System.out.println(s+1));
    }
}
```

## 2.使用lambda的优势

- 使用lambda表达式后，可以简化某些*匿名内部类*（`Anonymous Classes`）的写法，上下文会自动推断类型。

```java
list.sort(new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                return o1.length() - o2.length();
            }
        });

list.sort((o1, o2) -> o1.length() - o2.length());
list.sort(Comparator.comparingInt(String::length));
```

## 3.应用举例

- List

```java
List<String> list = new ArrayList<>(Arrays.asList("I","love","you","to"));

list.forEach(s -> System.out.println(s+1));

list.removeIf(s -> s.length() > 2);

list.replaceAll(str -> {
    if(str.length() > 3){
        return str.toUpperCase(Locale.ROOT);
    }
    return str;
});

```

- Map



# 集合

## 1.List

### 1.1 ArrayList

- Arrays.asList("I","love","you","too")生成的ArrayList来自其私有内部类，并不是从import java.util.ArrayList中产生的，因此不能进行增删操作。

- ArrayList是线程不安全的，`CopyOnWriteArrayList`线程安全。
  - 在添加新元素时，`CopyOnWriteArrayList`会复制一个新的数组并在此进行(加锁并发)写操作，读操作在原数组上进行。
  - `CopyOnWriteArrayList`适合读多写少的场景，但比占内存，且读到的不是最新数据，不适合实时性很高的场景。

### 1.2 LinkedList

```java
transient int size = 0;

    /**
     * Pointer to first node.
     * Invariant: (first == null && last == null) ||
     *            (first.prev == null && first.item != null)
     */
    transient Node<E> first;

    /**
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
    transient Node<E> last;
```

### 1.3 ArrayList与LinkedList区别

```
1.ArrayList基于动态数组，LinkedList基于双向链表

2.查找元素时
	使用for循环，ArrayList快很多
	使用迭代器，差不多
	
3.增加元素时
	增加头部元素，LinkedList快很多
	增加中间元素，ArrayList快很多
	增加尾部元素，ArrayList略快
	
4.删除元素时
	删除头部元素，LinkedList快很多
	删除中间元素，ArrayList快很多
	删除尾部元素，ArrayList略快
```

## 2.Set

### 2.1 add方法

```java
// Returns: true if this set did not already contain the specified element
// 返回值：当set中没有包含add的元素时返回真
public boolean add(E e) {
        return map.put(e, PRESENT)==null;
}
```

### 2.2 comparable和comparator区别

- `comparable` 接口实际上是出自`java.lang`包 它有一个 `compareTo(Object obj)`方法用来排序
- `comparator`接口实际上是出自 java.util 包它有一个`compare(Object obj1, Object obj2)`方法用来排序

```java
Collections.sort(arrayList, new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
      return o2.compareTo(o1);
    }
});

/*
对于一些普通的数据类型（比如 String, Integer, Double…），它们默认实现了Comparable 接口，实现了 compareTo 方法，我们可以直接使用。

而对于一些自定义类，它们可能在不同情况下需要实现不同的比较策略，我们可以新创建 Comparator 接口，然后使用特定的 Comparator 实现进行比较。
*/
```

```
1、什么是无序性？
无序性不等于随机性 ，无序性是指存储的数据在底层数组中并非按照数组索引的顺序添加 ，而是根据数据的哈希值决定的。

2、什么是不可重复性？
不可重复性是指添加的元素按照 equals()判断时 ，返回 false，需要同时重写 equals()方法和 HashCode()方法。
```

### 2.3 HashSet、LinkedHashSet 和 TreeSet 三者的异同

- `HashSet`、`LinkedHashSet` 和 `TreeSet` 都是 `Set` 接口的实现类，都能保证元素唯一，并且都不是线程安全的。
- `HashSet`、`LinkedHashSet` 和 `TreeSet` 的主要区别在于底层数据结构不同。`HashSet` 的底层数据结构是哈希表（基于 `HashMap` 实现）。`LinkedHashSet` 的底层数据结构是链表和哈希表，元素的插入和取出顺序满足 FIFO。`TreeSet` 底层数据结构是红黑树，元素是有序的，排序的方式有自然排序和定制排序。
- 底层数据结构不同又导致这三者的应用场景不同。`HashSet` 用于不需要保证元素插入和取出顺序的场景，`LinkedHashSet` 用于保证元素的插入和取出顺序满足 FIFO 的场景，`TreeSet` 用于支持对元素自定义排序规则的场景。

## 3.Map

### 3.1 HashMap

```
当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间。
```

![截屏2022-06-13 下午3.38.03](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-06-13 下午3.38.03.png)

#### 3.1.1 遍历

##### 1.lambda表达式遍历

```java
Map<Integer, Integer> map = new HashMap<>();
map.put(1, 10);
map.put(2, 20);
map.forEach((key, value) -> System.out.println(key + ":" + value));
```

##### 2.foreach循环遍历

```java
// 遍历获取键和值
for(Map.Entry<Integer, Integer> entry: map.entrySet()){
  	System.out.println(entry.getKey());
  	System.out.println(entry.getValue());
}
```

```java
// 获取键
for(Integer key: map.keySet()){
  	System.out.println(key);
}
// 获取值
for(Integer value: map.values()){
  	System.out.println(value);
}
```

#### 3.1.3 HashSet和HashMap的区别

HashSet` 底层就是基于 `HashMap` 实现的。（`HashSet` 的源码非常非常少，因为除了 `clone()`、`writeObject()`、`readObject()`是 `HashSet` 自己不得不实现之外，其他方法都是直接调用 `HashMap` 中的方法。

| `HashMap`                              | `HashSet`                                                    |
| -------------------------------------- | ------------------------------------------------------------ |
| 实现了 `Map` 接口                      | 实现 `Set` 接口                                              |
| 存储键值对                             | 仅存储对象                                                   |
| 调用 `put()`向 map 中添加元素          | 调用 `add()`方法向 `Set` 中添加元素                          |
| `HashMap` 使用键（Key）计算 `hashcode` | `HashSet` 使用成员对象来计算 `hashcode` 值，对于两个对象来说 `hashcode` 可能相同，所以`equals()`方法用来判断对象的相等性 |

#### 3.1.4 ConcurrentHashMap 和 Hashtable 的区别

`ConcurrentHashMap` 和 `Hashtable` 的区别主要体现在实现线程安全的方式上不同。

- **底层数据结构：**`ConcurrentHashMap`JDK1.8 采用的数据结构跟 `HashMap1.8` 的结构一样，数组+链表/红黑二叉树。`Hashtable` 和 JDK1.8 之前的 `HashMap` 的底层数据结构类似都是采用 **数组+链表** 的形式。
- **实现线程安全的方式（重要）：**

![截屏2022-06-13 下午4.22.40](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-06-13 下午4.22.40.png)

`ConcurrentHashMap` 取消了 `Segment` 分段锁，采用 

CAS (CompareAndSwap，是一种无锁算法。在不使用锁（没有线程被阻塞）的情况下实现多线程之间的变量同步。java.util.concurrent包中的原子类就是通过CAS来实现了乐观锁)

和 `synchronized` 来保证并发安全。数据结构跟 HashMap1.8 的结构类似，数组+链表/红黑二叉树。Java 8 在链表长度超过一定阈值（8）时将链表（寻址时间复杂度为 O(N)）转换为红黑树（寻址时间复杂度为 O(log(N))）

`synchronized` 只锁定当前链表或红黑二叉树的首节点，这样只要 hash 不冲突，就不会产生并发，效率又提升 N 倍。







# ---

编译时期可以检查代码的错误和语法，确保代码的正确性。

运行时期将Java代码转换为可执行代码并执行程序的过程。



可变长参数与List的区别

- 可变长参数将元素封装到数组中直接访问，List需要创建然后逐个添加元素。
- 可变长参数简介方便，适用于数量较少的情况。而List适用于需要进行大量操作、对参数进行curd等场景。
