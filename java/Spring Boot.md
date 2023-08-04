# 1.ApplicationContext与BeanFactory

## BeanFactory

### 1.1 BeanFactory是什么？

1. ApplicationContext的父接口
2. ==Spring的核心容器==，主要的ApplicationContext实现都【组合】了它的功能

```java
public static void main(String[] args) {
    ConfigurableApplicationContext context =  SpringApplication.run(DemoApplication.class, args);
    System.out.println(context.getBeanFactory());
}
```

### 1.2 BeanFactory能做什么？

- 表面上只有getBean()


- 实际上==控制反转，基本的依赖注入，直至Bean生命周期的各种功能，都由它的实现类提供==


```java
public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
    ConfigurableApplicationContext context =  SpringApplication.run(DemoApplication.class, args);
  	// DefaultSingletonBeanRegistry中存有所有的单例对象
    Field singletonObject = DefaultSingletonBeanRegistry.class.getDeclaredField("singletonObjects");
    singletonObject.setAccessible(true);
    ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
    Map<String, Object> map = (Map<String, Object>) singletonObject.get(beanFactory);
    map.entrySet().stream().filter(e->e.getKey().startsWith("component"))
                .forEach(e->{
                    System.out.println(e.getKey() + ":" + e.getValue());
                });
}
```

### 1.3ApplicationContext比BeanFactory多点什么

![截屏2022-07-12 下午3.38.40](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-07-12 下午3.38.40.png)

#### 1 MessageSource: 提供支持多种语言的能力

![截屏2022-07-12 下午4.01.18](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2022-07-12 下午4.01.18.png)

```java
// 读取翻译后的结果
System.out.println(context.getMessage("hi", null, Locale.CHINA));
System.out.println(context.getMessage("hi", null, Locale.ENGLISH));
```

#### 2 ResourcePatternResolver : 根据通配符获取一组资源

```java
Resource[] resource = context.getResources("classpath*:META-INF/spring.factories");
for (Resource r : resource) {
    System.out.println(r);
}

// 输出
URL [jar:file:/Users/huangminzhi/.m2/repository/org/springframework/boot/spring-boot/2.6.4/spring-boot-2.6.4.jar!/META-INF/spring.factories]
URL [jar:file:/Users/huangminzhi/.m2/repository/org/springframework/boot/spring-boot-autoconfigure/2.6.4/spring-boot-autoconfigure-2.6.4.jar!/META-INF/spring.factories]
URL [jar:file:/Users/huangminzhi/.m2/repository/org/springframework/spring-beans/5.3.16/spring-beans-5.3.16.jar!/META-INF/spring.factories]
URL [jar:file:/Users/huangminzhi/.m2/repository/com/baomidou/mybatis-plus-boot-starter/3.5.1/mybatis-plus-boot-starter-3.5.1.jar!/META-INF/spring.factories]
```

```java
System.out.println(context.getEnvironment().getProperty("java_home"));
System.out.println(context.getEnvironment().getProperty("server.port"));

// 输出
null
8080
```

#### 3 ApplicationEventPublisher: 发布事件对象

```java
@Component
@Slf4j
public class TestComponent {

    @Autowired
    private ApplicationEventPublisher context;

    public void register(){
        log.info("register");
        context.publishEvent(new UserRegisterEvent(this));
    }
}
```

```java
@Component
@Slf4j
public class TestComponent2 {

    @EventListener
    public void test(UserRegisterEvent e){
        log.info("{}", e);
        log.info("发送短信");
    }
}
```

```java
public class UserRegisterEvent extends ApplicationEvent {
    public UserRegisterEvent(Object source) {
        super(source);
    }
}
```

```java
@SpringBootApplication
public class DemoApplication{
    public static void main(String[] args) throws IOException {
        ConfigurableApplicationContext context =  SpringApplication.run(DemoApplication.class, args);
        context.getBean(TestComponent.class).register();
    }
```

```java
// 输出
2022-07-13 00:51:09.188  INFO 59423 --- [           main] com.example.demo.TestComponent           : register
2022-07-13 00:51:09.189  INFO 59423 --- [           main] com.example.demo.TestComponent2          : com.example.demo.UserRegisterEvent[source=com.example.demo.TestComponent@1c5c616f]
2022-07-13 00:51:09.189  INFO 59423 --- [           main] com.example.demo.TestComponent2          : 发送短信
```

#### 4 EnvironmentCapable: 处理环境信息



### 1.4 BeanFactory实现

```java
				DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        // bean 的定义(class, scope, 初始化, 销毁)
        AbstractBeanDefinition singleton = BeanDefinitionBuilder
          .genericBeanDefinition(Config.class)
          .setScope("singleton")
          .getBeanDefinition();
        beanFactory.registerBeanDefinition("config", singleton);

        // 给 BeanFactory 添加一些常用的[后处理器 => 添加一些扩展功能]
        AnnotationConfigUtils.registerAnnotationConfigProcessors(beanFactory);

        // BeanFactory 后处理器
        beanFactory.getBeansOfType(BeanFactoryPostProcessor.class)
                .values()
                .forEach(beanFactoryPostProcessor -> {
                    beanFactoryPostProcessor.postProcessBeanFactory(beanFactory);
                });

        // Bean 后处理器，针对 bean 的生命周期的各个阶段提供扩展，例如@Autowired @Resource...
        beanFactory.getBeansOfType(BeanPostProcessor.class)
                .values()
                .forEach(beanFactory::addBeanPostProcessor);

        for (String name:beanFactory.getBeanDefinitionNames()) {
            System.out.println(name);
        }

        // 不调用此方法，所有单例对象会在使用到的时候再创建
        beanFactory.preInstantiateSingletons(); // 准备好所有单例
        System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>");
        System.out.println(beanFactory.getBean(Bean1.class).getBean2());
}
```

- BeanFactory不会主动做的事
  1. 不会主动调用 BeanFactory 后处理器
  2. 不会主动添加 Bean 后处理器
  3. 不会主动初始化单例
  4. 不会解析BeanFactory 还不会解析${}与#{}
-  bean 后处理器会有排序的逻



## $\textcolor{orange}{ApplicationContext}$

### Application的创建方式

有4种，记录了最重要的一种。

==**基于java配置类来实现，用于 web 环境**==

1. 配置tomcat服务器(必要)

2. 创建servlet对象(必要)

3. 将servlet对象注册到tomcat服务器中(必要)

   创建控制器展示效果

```java
		@Configuration
    static class WebConfig{
        @Bean
        // 产生tomcat容器
        public ServletWebServerFactory servletWebServerFactory(){
            return new TomcatServletWebServerFactory();
        }

        @Bean
        // 创建Servlet对象
        public DispatcherServlet dispatcherServlet(){
            return new DispatcherServlet();
        }

        @Bean
        // 注册servlet到tomcat服务器
        public DispatcherServletRegistrationBean registrationBean(DispatcherServlet dispatcherServlet){
            return new DispatcherServletRegistrationBean(
                    dispatcherServlet,
                    "/"
            );
        }

        @Bean("/hello")
        public Controller controller1(){
            return new Controller() {
                @Override
                public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
                    response.getWriter().print("hello123");
                    return null;
                }
            };
        }
```



### 从这种创建方式中学到了什么？

1. 内嵌tomcat的的工作方式即是如此
2. Spring Web程序的入口都是通过`return new DispatcherServletRegistrationBean(dispatcherServlet,"/");`先匹配到"/"通配符，然后由DispatcherServlet进行分发到其他控制器
3. 通过上述3个Bean就可以做一个Web应用服务器





# $\textcolor{orange}{2.Bean}$

## 2.1 什么是Spring Bean？

Bean代指被IOC容器所管理的对象



## 2.2 将一个类声明为Bean的注解有哪些？

- @Component
  - 通用的注解，可标注任务类为`Spring`组件。如果一个`Bean`不知道属于哪层，可以使用@`Component`注解标注。
- @Repository
  - 对应持久层即Dao层，用于数据库相关操作
- @Service
  - 对应服务层，主要设计一些复杂的逻辑，需要用到Dao层
- @Controller
  - 对应控制层，主要用于接受用户请求并调佣service层返回数据给前端页面



## 2.3 Bean的作用域有哪些?

- Singlton
  - IOC容器中只有唯一的bean实例。Spring中的bean默认都是单例的。
- prototype
  - 每次获取都会创建一个新的bean实例。调用getBean()两次，会得到不同的Bean实例。
- 还有4种Web应用可用的Bean 暂时不管了



## $\textcolor{red}{2.4 Bean的生命周期}$

- 声明一个bean

```java
@Component
@Slf4j
public class LifeCycleBean {
    public LifeCycleBean() {
        log.info("构造函数");
    }

    @Autowired
    public void autowire(@Value("${JAVA_HOME}") String home){
        log.info("依赖注入:{}", home);
    }

    @PostConstruct
    public void init(){
        log.info("初始化");
    }

    @PreDestroy
    public void destroy(){
        log.info("销毁");
    }

}
```

- 6个常见的阶段

```java
@Component
@Slf4j
public class MyBeanPostProcessor implements InstantiationAwareBeanPostProcessor, DestructionAwareBeanPostProcessor {

    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        if(beanName.equals("lifeCycleBean")){
            log.info("<<<<<< 实例化执行之前，返回的对象会替换掉原来的 bean");
        }
        return null;
    }

    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        if(beanName.equals("lifeCycleBean")){
            log.info("<<<<<< 实例化执行之后，如果返回false会跳过依赖注入阶段");
//            return false;
        }
        return true;
    }

    @Override
    public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) throws BeansException {
        if(beanName.equals("lifeCycleBean")){
            log.info("<<<<< 依赖注入执行阶段，如@Autowired、@Value、@Resource");
        }
        return pvs;
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if(beanName.equals("lifeCycleBean")){
            log.info("<<<<< 初始化执行之前，这里返回的对象会替换掉原本的 bean， 如@PostConstruct、@ConfigurationProperties");
        }
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if(beanName.equals("lifeCycleBean")){
            log.info("<<<<< 初始化执行之后，这里返回的对象会替换掉原来的 bean， 如代理增强");
        }
        return bean;
    }

    @Override
    public void postProcessBeforeDestruction(Object o, String s) throws BeansException {
        if(s.equals("lifeCycleBean")){
            log.info("<<<< 销毁前执行 如@PreDestroy");
        }
    }
}
```

![截屏2023-03-16 上午1.02.21](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-03-16 上午1.02.21.png)

### 设计模式: 模版方法

```java
public static void main(String[] args) {
    MyBeanFactory factory = new MyBeanFactory();
    factory.addBeanPostProcessor(bean -> System.out.println("解析@Autowired"));
    factory.addBeanPostProcessor(bean -> System.out.println("解析@Resource"));
    factory.getBean();
}

// 模版方法 Template Method Pattern
static class MyBeanFactory{
    public Object getBean(){
        Object bean = new Object();
        System.out.println("构造" + bean);
        System.out.println("依赖注入" + bean);
        for(BeanPostProcessor processor: processors){
            processor.inject(bean);
        }
        System.out.println("初始化" + bean);
        return bean;
    }
    private List<BeanPostProcessor> processors = new ArrayList<>();

    public void addBeanPostProcessor(BeanPostProcessor processor){
        processors.add(processor);
    }
}

interface BeanPostProcessor{
    // 对依赖注入阶段扩展
    void inject(Object bean);
}
```





## 2.5 单例Bean的线程安全问题

当多个线程操作同一个对象的时候，存在资源竞争。

解决办法:

1. 在Bean中尽量避免定义可变的成员变量
2. 在类中定义一个`ThreadLocal`成员变量，将需要的可变成员变量保存在`ThreadLocal`中(推荐)

大部分Bean实际都是无状态(没有实例变量)的(Dao,Service)层，这种情况下，Bean线程安全



# $\textcolor{orange}{3.AOP}$

```java
public class Animal {
    private String height;
    private double weight;

    public void eat() {
        long start = System.currentTimeMillis();
        System.out.println("I can eat...");
        System.out.println("执行时长：" + (System.currentTimeMillis() - start)/1000f + "s");
    }

    public void run() {
        long start = System.currentTimeMillis();
        System.out.println("I can run...");
        System.out.println("执行时长：" + (System.currentTimeMillis() - start)/1000f + "s");
    }
}
```

这部分重复的代码，一般统称为 `横切逻辑代码`。

![截屏2023-03-07 上午10.53.57](/Users/huangminzhi/Library/Application Support/typora-user-images/截屏2023-03-07 上午10.53.57.png)

`横切逻辑代码`存在的问题:

- 代码重复
- 横切逻辑与业务代码混在一起，不便维护

- `Apsect oriented programming`面向切面编程，用于处理横切逻辑代码的重复问题。

- Spring AOP基于<font color=red size=4>动态代理</font>，如果要代理的对象，实现了某个接口，那么Spring AOP会使用`JDK Proxy`，去创建代理对象。对于没有实现接口的对象，则使用`Cglib`生成一个被代理对象的子类来作为代理。

## 动态代理

Java提供了两种动态代理的方式

1. 基于接口的动态代理(常用)
2. 基于类的动态代理

在JDK中，提供了一个java.lang.reflect包，该包中的Proxy类可以用来创建动态代理对象。

### 1) 基于接口的动态代理

创建流程：

1. 创建一个实现了InvocationHandler接口的类，重写invoke()方法。
2. 调用Proxy类的静态方法newProxyInstance()，传入代理对象的类加载器、代理对象所实现的接口以及InvocationHandler对象。
3. 使用返回的代理对象进行操作，代理对象的方法调用会被转发到InvocationHandler的invode()方法中进行处理。

```java
public interface UserService {
    void addUser(String name, String password);
}
```

```java
@Service
public class UserServiceImpl implements UserService{

    @Override
    public void addUser(String name, String password) {
        System.out.println("add user:" + name);
    }
}
```

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class UserServiceProxy implements InvocationHandler {

    private Object target;

    public UserServiceProxy(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("before invoke:" + method.getName());
        Object result = method.invoke(target, args);
        System.out.println("after invoke:" + method.getName());
        return result;
    }

    public static void main(String[] args) {
        UserService userService = new UserServiceImpl();
      
        UserService proxy = (UserService) Proxy.newProxyInstance(
          			// jdk.internal.loader.ClassLoaders$AppClassLoader@251a69d7
                userService.getClass().getClassLoader(),
          			// [interface com.example.test.service.动态代理测试.UserService]
                userService.getClass().getInterfaces(),
                new UserServiceProxy(userService));
        proxy.addUser("John", "123456");
    }
}
```

```java
before invoke:addUser
add user:John
after invoke:addUser
```

### 2) 基于类的动态代理

创建流程:

- 首先创建了一个UserService实例和一个UserServiceProxy实例。
- 使用Enhancer类创建代理对象，设置被代理类为UserService，设置回调为UserServiceProxy。
- 通过代理对象调用被代理类的方法，代理对象会自动调用UserServiceProxy中的intercept方法，在方法执行前后添加日志记录。

```
@Service
public class UserService {

    public void addUser(String name, String password) {
        System.out.println("add user:" + name);
    }

    public void deleteUser(String username) {
        System.out.println("delete user: " + username);
    }
}
```

```java
import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;
import java.lang.reflect.Method;

public class UserServiceProxy implements MethodInterceptor {

    private Object target;

    public UserServiceProxy(Object target) {
        this.target = target;
    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("before intercept:" + method.getName());
        Object result = method.invoke(target, args);
        System.out.println("after intercept:" + method.getName());
        return result;
    }

    public static void main(String[] args) {
        UserService userService = new UserService();
        UserServiceProxy proxy = new UserServiceProxy(userService);
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(UserService.class);
        enhancer.setCallback(proxy);
        UserService userServiceProxy = (UserService) enhancer.create();
        userServiceProxy.addUser("user", "123456");
        userServiceProxy.deleteUser("user");
    }
}
```



## AOP术语

|       术语        | 含义                                                         |
| :---------------: | :----------------------------------------------------------- |
|   Target(目标)    | 被通知的对象                                                 |
|    Proxy(代理)    | 向目标对象应用通知之后创建的代理对象                         |
| joinPoint(连接点) | 目标对象的所属类种，定义的所有方法均为连接点                 |
| PointCut(切入点)  | 被切面拦截/增强的连接点(切入点一定是链接点，连接点不一定是切入点) |
|   Advice(通知)    | 增加的逻辑/代码，业绩拦截到目标对象的连接点之后要做的事情    |
|   Aspect(切面)    | PointCut+Advice                                              |
|   Weaving(织入)   | 将通知应用到目标对象，进而生成代理对象的过程动作             |



### @Before,@After,@Around

- `@After`注解用于标记一个方法，该方法将在连接点之后执行。连接点是程序执行中的特定点，例如方法的调用或异常的处理。`@After`注解的方法将立即在连接点之后执行。

- 相比之下，`@Before`注解用于标记一个方法，在连接点之前执行。因此，`@Before`注解的方法将在连接点之前执行，而`@After`注解的方法将在连接点之后执行。

- `@After`和`@Before`注解经常一起在AOP中使用，定义一个在连接点之前和/或之后执行某些操作的切面。

```java
  	@Before("execution(* com.example.UserService.getUserById(Long)) && args(userId)")
    public void logStart(Long userId) {
        logger.info("getUserById method starts with userId: {}", userId);
    }

    @After("execution(* com.example.UserService.getUserById(Long)) && args(userId)")
    public void logEnd(Long userId) {
        logger.info("getUserById method ends with userId: {}", userId);
    }

    @AfterReturning(pointcut = "execution(* com.example.UserService.getUserById(Long))", returning = "result")
    public void logTime(JoinPoint joinPoint, Object result) {
        long elapsedTime = System.currentTimeMillis() - startTime.get();
        logger.info("getUserById method takes {} ms with result: {}", elapsedTime, result);
    }
```

- `@Around`注解可以控制目标方法的执行，并且可以在方法执行前后进行一些处理。在`@Around`注解中，需要使用`ProceedingJoinPoint`对象来手动调用目标方法，并可以在目标方法执行前后进行一些操作，例如记录日志、计算方法执行时间等等。

```java
		@Around("execution(* com.example.UserService.getUserById(Long))")
    public Object logMethod(ProceedingJoinPoint joinPoint) throws Throwable {
        String methodName = joinPoint.getSignature().getName();
        logger.info("Executing method: {}", methodName);

        long start = System.currentTimeMillis();

        Object result = joinPoint.proceed();

        long elapsedTime = System.currentTimeMillis() - start;
        logger.info("Executed method: {} with result: {} in {} ms", methodName, result, elapsedTime);

        return result;
    }
```



### Spring AOP 与 AspectJ AOP有什么区别？

- Spring AOP 属于`运行时增强`，而ApsectJ 属于`编译时增强`。Spring AOP 基于动态代理，AspcetJ 基于字节码操作

- Spring AOP 已经集成了 AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ 相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单，

- 如果我们的切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择 AspectJ ，它比 Spring AOP 快很多。



# 4.注解

### 1.ConfigurationProperties 和 Value

| Feature           | `@ConfigurationProperties` | `@Value` |
| :---------------- | :------------------------- | :------- |
| Relaxed binding   | Yes                        | Limited  |
| Meta-data support | Yes                        | No       |
| `SpEL` evaluation | No                         | Yes      |

- Relaxed binding：外部属性与Java对象之间的灵活绑定机制

  - my.main-project.person.first-name【Kebab case, which is recommended for use in `.properties` and YAML files.】

  - 大小写忽略

  - 下划线_与破折号-互相转换等

- Meta-data support：
  - 支持属性的
    - 类型转换
    - 默认值
    - 校验
  - 支持分组和继承

```java
@ConfigurationProperties(prefix = "myapp")
public class MyAppProperties {
    // Common properties
    private String commonProperty;

    // Database module properties
    private String databaseUrl;
    private String databaseUsername;
    private String databasePassword;

    // Cache module properties
    private boolean cacheEnabled;
    private int cacheMaxEntries;

    // getters and setters
}

@ConfigurationProperties(prefix = "myapp.database")
public class DatabaseProperties extends MyAppProperties {
    // Additional properties specific to the database module
    // ...
}

@ConfigurationProperties(prefix = "myapp.cache")
public class CacheProperties extends MyAppProperties {
    // Additional properties specific to the cache module
    // ...
}
```

- SpEL（Spring Expression language)
  - 支持动态计算和操作对象的属性、方法调用、算术运算、逻辑判断等。





# 依赖注入

Field Injection存在的问题

- 潜在的空指针风险

  - ```java
    @Service
    public class EmailService {
    
        @Autowired
        private EmailValidator emailValidator;
    }
    
    public void process(String email) {
        if(!emailValidator.isValid(email)){
            throw new IllegalArgumentException(INVALID_EMAIL);
        }
        // ...
    }
    
    EmailService emailService = new EmailService();
    emailService.process("test@baeldung.com");
    ```

- 不可变性

  - 字段注入无法创建不可变类
  - 不可变类的优势：线程安全、易于缓存、易于测试和调试

- 单一职责原则

  - 使用字段注入后，当前类还需要管理字段注入的依赖关系，不易于理解和维护

- 单元测试



# 自动装配原理

简单理解: 通过注解或者一些简单的配置就能在Spring Boot的帮助下实现某块功能。

- @SpringBootApplication中已包含@EnableAutoConfiguration

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
// 启动SpringBoot的自动配置
@SpringBootConfiguration
// 扫描@Component (@Service,@Controller)注解的 bean,注解默认会扫描启动类所在的包下所有的类 ，可以自定义不扫描某些 bean。如下图所示，容器中将排除TypeExcludeFilter和AutoConfigurationExcludeFilter。
@ComponentScan
// 允许在上下文中注册额外的 bean 或导入其他配置类
@EnableAutoConfiguration
public @interface SpringBootApplication {
	
}

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration //实际上它也是一个配置类
public @interface SpringBootConfiguration {
}
```

- @EnableAutoConfiguration

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage //作用：将main包下的所有组件注册到容器中
@Import({AutoConfigurationImportSelector.class}) //加载自动装配类 xxxAutoconfiguration
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```



