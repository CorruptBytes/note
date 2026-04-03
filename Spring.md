# 事务

## 事务失效

有时候会出现在方法上添加了`@Transactional`注解，但运行时事务没有开启。常见的事务失效原因有：

**1. 方法不是 `public` 修饰**

**原因**：Spring AOP 的`JDK`动态代理只能对 `public` 方法生效，非 `public` 方法无法被代理，因此事务不生效。

**解决方案**：将事务方法修改为 `public`。

```java
@Service
public class UserService {

    @Transactional
    public void saveUser() {
        // 事务方法必须是 public
    }
}
```

**2. 内部调用事务方法**

**原因**：内部调用使用的是原始对象 `this`，绕过 Spring 代理对象，事务不生效。

**解决方案**

- **拆分成两个类**

  将原版的`service`拆分为业务类和事务类，在事务类上添加`@Transactional`，由业务类调用事务类完成业务

  ```java
  @Service
  public class CouponTxService {
      @Transactional
      public void doReceive(Long userId, Long couponId) {
  
          int count = couponMapper.countByUser(userId, couponId);
          if (count > 0) {
              throw new RuntimeException("已领取");
          }
  
          couponMapper.insert(userId, couponId);
      }
  }
  @Service
  public class CouponService {
  
      @Autowired
      private CouponTxService couponTxService;
  
      public void receiveCoupon(Long userId, Long couponId) {
  
          synchronized (userId.toString().intern()) {
              couponTxService.doReceive(userId, couponId); // ✅ 外部调用
          }
      }
  }
  ```

- **注入自身代理对象调用。**

  ```java
  @Service
  public class UserService {
  
      @Autowired
      private UserService self;
  
      public void outerMethod() {
          self.saveUser(); // 通过代理对象调用事务方法
      }
      @Transactional
      public void saveUser() {
          // 事务方法
      }
  }
  ```

- **通过 AOP 上下文获得代理对象**

  1. 引入`Aspectj`依赖

     ```xml
     <dependency>
         <groupId>org.aspectj</groupId> 
         <artifactId>aspectjweaver</artifactId>
     </dependency>
     ```

  2. 开启AspectJ的AOP功能，并暴露代理对象

     ```java
     @EnableAspectJAutoProxy(exposeProxy = true)
     @SpringBootApplication
     public class PromotionApplication {
        //...
     }
     ```

  3. 使用代理对象调用事务方法

     ```java
     IUserCouponService serviceProxy = (IUserCouponService) AopContext.currentProxy();
     serviceProxy.sell();
     ```

**3. 异常被捕获**

**原因**：事务依赖异常回滚，如果事务方法内部捕获异常，代理对象无法感知到异常，就无法回滚。

**解决方案**：不要在事务方法内部捕获异常，或者捕获异常后手动标记回滚。

```java
@Transactional
public void saveUser() {
    try {
        // 业务逻辑
    } catch(Exception e) {
        TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
    }
}
```

**4. 回滚异常类型不匹配**

**原因**：默认情况下，Spring 事务只对 `RuntimeException` 及其子类异常进行回滚

**解决方案**：通过 `rollbackFor` 属性指定异常类型。

```java
@Transactional(rollbackFor = Exception.class)
public void saveUser() throws Exception {
    throw new Exception();
}
```

**5. 事务传播行为错误**

**原因**：事务方法嵌套调用时，传播行为设置不正确，可能导致事务未按预期生效。

**解决方案**：根据业务选择合适的传播属性，如 `REQUIRED`、`REQUIRES_NEW`。

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void newTransactionMethod() {
    // 新事务方法
}
```

**6. 当前类没有被 Spring 管理**

**原因**：类未被 Spring 托管，框架不会创建代理对象，事务自然不生效。

**解决方案**：将类交给 Spring 管理，例如使用 `@Service`、`@Component`，并通过依赖注入调用。

```java
@Service
public class UserService {

    @Transactional
    public void saveUser() {
        // 事务方法
    }
}
```

**7 . 多线程导致事务上下文丢失**

**原因**：事务是绑定在当前线程的`ThreadLocal`

**解决方案**：不要在事务中使用多线程



# IOC

> Inversion Of Control，即控制反转。指把对象依赖的控制权从对象本身反转到外部容器，由容器负责创建和注入依赖，从而降低耦合度，提高可维护性

**控制翻转的核心是反转对象对依赖的控制权，Spring通过IOC容器和DI实现控制反转**

## DI

> Dependence Inject，依赖注入，是实现控制反转的一种具体方式。

通过依赖注入，类只需要声明自己所依赖的对象，IOC容器会在运行时将这些依赖注入到对象中，而无需在类内部通过`new`关键字创建依赖对象，使对象与依赖之间大幅度解耦。

**常用的依赖注入方式有三种：**

- **构造器注入：**通过构造函数传递依赖对象，保证对象初始化时依赖已就绪

  ```
  
  ```

- **Setter方法注入：**通过Setter方法设置依赖，灵活性高，但依赖可能未完全初始化

  ```java
  public class PaymentService{
  	private PaymentGateway gateway;
  	@Autowired
  	public void setGateway(PaymentGateway gateway){
          this.gateway = gateway;
      }
  ```

- **字段注入：**在依赖字段上加`@Autowired`或`@Resource`注解，代码简洁但隐藏了依赖关系。

## IOC容器

# AOP

> `AOP`(Aspect Oriented Programming)，是一种补充面向对象编程的编程思想，将**横切关注点**(与业务无关、但被多个模块共同使用的功能)从业务代码中分离出来，通过“切面”的方式统一管理。

**简而言之，AOP就是在不修改业务代码的前提下，对方法进行统一增强。**

<h3>AOP相关概念</h3>

**目标对象**

被增强方法所在的对象

**代理对象**

对目标方法进行增强后的对象，由动态代理生成，这是客户端实际调用的对象

**连接点(Joinpoint)**

程序执行过程中**可以被增强的点**

**切点(Pointcut)**

切点定义了哪些连接点需要被增强，通常由切点表达式描述

- 切点本质是一组规则，筛选哪些连接点需要被增强。

**通知(Advice)**

通知封装了增强部分的代码逻辑，定义了在目标方法执行的**什么时机**执行什么样的增强操作。

**切面(Aspect)**

切面是对横切关注点的模块化封装，由切点（Pointcut）和通知（Advice）组成，用于描述对哪些目标方法在什么时机执行什么样的增强操作

**织入(Weaving)**

将切面中定义的通知，按照切点规则，应用到目标方法的执行过程中的过程。

## Spring AOP

**相关坐标**

```xml
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.7</version>
        </dependency>
```

```xml
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>7.0.1</version>
        </dependency>
```

### XML配置AOP

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/context/spring-aop.xsd">

    <bean id="userService" class="com.aop.service.UserServiceImpl"/>
     
    <bean id="myAdvice" class="com.aop.advice.MyAdvice"/>
    <aop:config>
        <!--配置切点表达式-->
        <aop:pointcut id="myPointcut" expression="execution(void com.aop.service.UserServiceImpl.show1())"/>
        <!--配置切面-->
        <aop:aspect ref="myAdvice">
            <aop:before method="before" pointcut-ref="myPointcut"/>
        </aop:aspect>
    </aop:config>
</beans>
```

<h4>通知</h4>

通知的具体增强代码封装在类方法中，称为通知方法，这种类被称为通知类，通过`<aop:aspect ref="beanId">`指定一类切面的通知类。

`AspectJ`根据通知的执行时机将通知分为5种，通过XML标签指定不同类型的通知。

| 通知名称 |         配置方式         |                         执行时机                         |
| -------- | :----------------------: | :------------------------------------------------------: |
| 前置通知 |      `<aop:before>`      |                   目标方法执行之前执行                   |
| 后置通知 | `<aop:after-returning>`  |      目标方法执行之后执行，目标方法异常时，不再执行      |
| 环绕通知 |      `<aop:around>`      | 目标方法执行前后执行，目标方法异常时，环绕后方法不再执行 |
| 异常通知 | `<  aop:after-throwing>` |                  目标方法抛出异常时执行                  |
| 最终通知 |      `<aop:after>`       |           不管目标方法是否有异常，最终都会执行           |

- 标签内部使用`method="方法名"`属性指定通知方法，使用`pointcut`或`pointcut-ref`指定切点表达式。

**环绕通知**是其他类型通知实现的基础，其通知方法书写与其他通知不同

```java
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        Object result = joinPoint.proceed(); //执行原方法。
        return result;
    }
```

通知方法在被调用时，Spring可以为其传递一些必要的参数，通过声明方法参数可以自动接收。

|       参数类型        |                             作用                             |
| :-------------------: | :----------------------------------------------------------: |
|      `JoinPoint`      | 连接点对象，任何通知都可使用，可以获得当前目标对象、目标方法参数等信息 |
| `ProceedingJoinPoint` | `JoinPoint`子类对象，只用于环绕通知方法中，通过`ProceedingJoinPoint.proceed()`执行原方法 |
|      `Throwable`      | 异常对象，使用在异常通知中，需要在配置文件中指出异常对象名称 |

<h4>切点表达式</h4>

切点表达式有两种配置方式:

- 通过`<aop:pointcut id="" expression=""/>`标签定义切点表达式,在配置切面时通过`point-ref="切点id"`属性引用切点表达式。这种切点表达式可以应用于多个切面中
- 在配置切面时`<aop:before method="before" pointcut=""/>`直接通过`pointcut`属性定义切点表达式。这种方式只在当前切面中生效。

**切点表达式语法**

```
execution([访问修饰符] 返回值类型 包名.类名.方法名(参数类型))
```

- 访问修饰符可以省略
- 返回值类型、某一级包名、类名、方法名、可以使用`*`通配符表示任意
- `..`表示匹配该包及其子包下的包或类,`.`表示匹配该包下的包或类
- 参数列表中可以使用`..`表示任意参数

<h4>切面</h4>

切面的配置方式有两种：

通过 `<aop:aspect ref="通知类">`标签指定通知类，内部可以根据此通知类配置具有不同通知类型的多个切面

```xml
        <aop:aspect ref="myAdvice">
            <aop:before method="before" pointcut-ref="myPointcut"/>
        </aop:aspect>
```

通知类实现Spring提供的`Advice`接口的子类，接口定义了通知的类型。直接通过`<aop:advisor/>`标签配置一个切面。

```xml
<aop:advisor advice-ref="myAdvice" pointcut-ref="myPointcut"/>
```

- 此时不需要指定通知类型，通知类型已经由`myAdvice`实现的接口定义。

### 注解配置AOP

- 注解配置AOP首先要开启AOP自动代理，通过`<aop:aspectj-autoproxy/>`标签开启或者`@EnableAspectjAutoProxy`开启

1. 在通知类上加入`@Aspect`注解表示在此类中声明切面

   ```
   @Component
   @Aspect
   public class MyAdvice {
   
   }
   ```

2. 在通知方法上使用注解配置通知类型，并在注解的`value`参数中配置切点表达式

   ```java
   @Component
   @Aspect
   public class MyAdvice {
       @Before("execution(* com.aop.service..*.*(..))")
       public void before() {
           System.out.println("before...前置增强");
       }
       public void after() {
           System.out.println("after...后置增强");
       }
       //也可以统一配置切点表达式，通过MyAdvice.myPointcut()这个切点表达式引入
           @Pointcut()
       public void myPointcut() {
   
       }
   
   }
   ```

Spring提供了五种注解声明不同的通知类型：

**@After**

**@Before**

**@Around**

**`@AfterThrowing`**

**`@AfterReturning`**

<h4>Spring中AOP实现原理</h4>

Spring AOP 的实现依赖于动态代理。

**Spring AOP支持两种动态代理：**

- **基于JDK的动态代理：**默认情况下，优先选择JDK动态代理生成代理对象(底层实现为`JdkDynamicAopProxy`)。这要求被代理的类实现一个或多个接口。代理对象只能对目标对象的接口方法进行增强。
- **基于CGLIB的动态代理：**当被代理的类没有实现任何接口且没有被`final`修饰时，Spring会使用CGLIB库生成一个被代理类的子类对象作为代理对象(底层使用`CglibAopProxy`)。CGLIB（`CodeGeneration Library`）是一个第三方代码生成库，通过继承方式实现动态代理。
  - 可以在通过`<aop:config proxy-target-class="true">`属性强制使用CGLIB动态代理。 

## `JoinPoint`

`JoinPoint`（连接点）表示程序执行过程中一个可以被 AOP 拦截的位置,它封装了当前被拦截方法调用时的上下文信息。

-  `JoinPoint` 只提供访问能力，并不能控制方法的执行流程。

<h3>常用方法</h3>

**获取方法参数**

```
Object[] getArgs()
```

**获取签名对象**

```
Signature getSignature()
```

- `SpringAOP`只支持方法级别的连接点，因此签名对象通常为一个`MethodSignature`实例，因此可强转`Signature`对象为方法签名对象，以访问方法的相关信息。

**获取被代理的真实对象**

```
Object getTarget()
```

**获取代理对象**

```
Object getThis()
```

**打印调用信息**

```
String toShortString()
String toLongString()
```



# Bean

## 生命周期

一个 Bean 从被 Spring 容器创建开始，到被容器销毁为止，所经历的一整套完整过程。

<h3>实例化</h3>

Spring通过反射创建`Bean`对象

- 通过反射调用构造方法，构造器参数注入属性在这里执行

<h3>属性注入</h3>

通过反射进行`Setter`方法注入或字段注入(`@Autowired`,`@Resource`)

<h3><code>Aware</code> 接口回调</h3>

这一步让 Bean 感知 Spring 容器本身。如果 Bean 实现了这些函数式接口，Spring 会回调：

```java
BeanNameAware
BeanFactoryAware
ApplicationContextAware
```

<h3><code>BeanPostProcessor</code>前置处理</h3>

初始化前置处理阶段Spring 会依次调用**所有注册的 `BeanPostProcessor`** 的 `postProcessBeforeInitialization()` 方法,所有符合条件的 Bean 都会经过该方法处理。

- 当一个` BeanPostProcessor` 本身被注册为 Bean 时，它不会调用自己处理自己。

- 自定义的`BeanPostProcessor`(自定义逻辑，比如打印日志、字段加工等)
- Spring 内置的 BPP(实现 AOP、事务增强等功能)

<h3>初始化</h3>

初始化阶段，Spring 会调用 Bean 的初始化方法，常用的初始化方法配置方式有（按执行顺序）：

1. 使用 `@PostConstruct` 注解

   ```
   @PostConstruct
   public void init() {
       // 初始化逻辑，例如打开资源、初始化缓存等
   }
   ```

2. 实现 `InitializingBean` 接口

   ```
   public class MyBean implements InitializingBean {
       @Override
       public void afterPropertiesSet() {
           // 初始化逻辑
       }
   }
   ```

3. 通过 Java Config 配置 `init-method`

   ```
   @Bean(initMethod = "customInit")
   public MyBean myBean() {
       return new MyBean();
   }
   
   public class MyBean {
       public void customInit() {
           // 自定义初始化逻辑
       }
   }
   
   ```

<h3><code>BeanPostProcessor</code>后置处理</h3>

初始化后置处理阶段，依次调用所有已注册的`BeanPostProcessor`的`postProcessAfterInitialization()`方法，调用顺序与初始化前置处理一致。

-  AOP 代理对象就在这里生成

<h3>使用</h3>

Bean 已完全初始化，可以被使用

<h3>销毁</h3>

当容器关闭时，Spring 会销毁由容器管理的 **单例 Bean**，即调用 Bean 的销毁方法以释放资源和清理状态。

- **只有单例 Bean 会被容器自动销毁**，而原型 Bean 的销毁需要由用户手动处理。
- `Bean`的内存释放由`GC`负责，Spring 不直接释放内存。

常用的 Bean 销毁方法配置方式：

1. 使用注解 `@PreDestroy`

   ```
   @PreDestroy
   public void cleanUp() { ... }
   ```

2. 实现 `DisposableBean` 接口的 `destroy()` 方法

   ```
   @Override
   public void destroy() { ... }
   ```

3. 在 Java Config 中通过 `destroyMethod` 属性指定销毁方法

   ```
   @Bean(destroyMethod = "destroy")
   public MyBean myBean() { ... }
   ```

Spring提供了多种方式控制Bean的生命周期

**在XML文件中通过`init-method`和`destroy-method`属性来指定Bean初始化后和销毁前需要调用的方法**

```xml
<bean id-"myBean" class="com.example.MyBeanClass"init-method-"init" destroy-method-"destroy"/>
public class MyBeanClass｛
	public void init(){
		初炲化逻辑
	}
	public void destroy() {
		诮毀逻辑
	}
}
```

**实现`InitializingBean`和`DisposableBean`接口**

```java
public class MyBeanClassimplements InitializingBean,DisposableBean
	@Override
	public void afterPropertiesSet() throws Exception {
    	∥初始化逻辑
	}

	@Override
	public void destroy() throws Exception {
    	∥销毀逻辑
	}
}
```

**使用`@PostConstruct`和`@PreDestroy`注解**

```java
public class MyBeanClass｛
	@PostConstruct
	public void init(){
		∥初始化逻辑
    }
	@PreDestroy
	public void destroy(){
		∥销毀逻辑
    }
}
```

**使用@Bean注解的`initMethod`和`destroyMethod`属性**

```java
@Configuration
public class AppConfig{
	@Bean(initMethod="init",destroyMethod="destroy")
	public MyBeanClass myBean() {
        return new MyBeanClass():
    }
}
```

## 作用域

> Bean的作用域(Scope)定义了Bean的生命周期和可见性，如`Bean`如何被创建，如何被销毁以及是否可以被多个用户共享

**Spring支持7种作用域以应对不同的应用场景**

- `Singleton`(单例)：默认作用域，Spring容器中只会创建一个Bean实例，并在容器的整个生命周期中共享该实例。
- `Prototype`(原型)：每次从容器中获取该Bean时都会创建一个新实例，适用于状态非常瞬时的Bean。
- `Request`(请求)：每个HTTP请求都会创建一个新的 Bean实例。仅在`Spring Web`应用程序中有效，适用于Web应用中需求局部性的Bean。
- `Session`(会话)：Session范围内只会创建一个Bean实例。该Bean实例在用户会话范围内共享，仅在
  `Spring Web`应用程序中有效，适用于与用户会话相关的Bean。
- `Application`：当前 `ServletContext` 中只存在一个Bean实例。仅在`Spring Web`应用程序中有效，该`Bean`实例在整个`ServletContext`范围内共享，适用于应用程序范围内共享的Bean。
- `WebSocket`（Web套接字）：在`WebSocket`范围内只存在一个Bean实例。仅在支持`WebSocket`的应用程序中有效，该Bean实例在`WebSocket`会话范围内共享，适用于WebSocket会话范围内共享的Bean。
- `Customscopes`（自定义作用域）：Spring允许开发者定义自定义的作用域，通过实现Scope接口来创建新的 Bean作用域。

**配置Bean作用域**

- 在`Bean`配置文件中通过`<bean`标签的`scope`属性配置

  ```xml
  <bean id-"myBean" class="com.example.MyBeanClass" scope-"singleton"/>
  ```

- 在Spring Boot应用中，通过`@Scope`注解指定Bean的作用域

  ```java
  @Bean
  @Scope("prototype")
  public MyBeanClass myBean()
  return new MyBeanClass();
  ```


# `SpringTask`

> `Spring`提供API快速编写单机定时任务。

在底层由`TaskScheduler`实现，本质是**线程池**+**时间触发器**

1. 首先需要通过`@EnableScheduling`注解开启定时任务功能

   ```java
   @SpringBootApplication
   @EnableScheduling
   public class Application {
       public static void main(String[] args) {
           SpringApplication.run(Application.class, args);
       }
   }
   
   ```

2. 一般会创建一个类统一管理一类定时任务，这个类必须注册到IOC容器中

   ```java
   @Component
   public class DemoJob {
   
       @Scheduled(fixedRate = 5000)
       public void run() {
           System.out.println("定时任务执行：" + LocalDateTime.now());
       }
   }
   ```

**Spring默认所有定时任务共用一个线程，如果一个任务卡死，则会导致任务全部阻塞。在实际使用时，通常配合线程池使用。**

```java
@Configuration
@EnableScheduling
public class ScheduleConfig implements SchedulingConfigurer {

    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        scheduler.setPoolSize(10);
        scheduler.setThreadNamePrefix("schedule-");
        scheduler.initialize();
        taskRegistrar.setTaskScheduler(scheduler);
    }
}
```

<h4>Spring定时任务缺陷</h4>

- 声明定时任务的方法不能有参数。
- 声明定时任务的方法 不能是 `static`修饰
- 声明定时任务的方法如果抛出异常会直接导致任务执行线程停止，因此需要在内部处理异常。
- 不适用于分布式，只是单机定时任务。
- 在声明定时任务的方法上加`@Transactional`，事务不会生效。

## `@Scheduled`

加在方法上，将方法配置为定时任务。

**`@Scheduled`**提供了三种方式配置定时任务的执行频率。

- `@Scheduled(fixedRate = 5000)`:以固定频率执行任务，每`xxx`毫秒执行一次。
  - 以上一次任务**开始时间**为基准，如果任务执行较慢，可能出现任务并发执行的情况。
- `@Scheduled(fixedDelay = 5000)`:以固定延迟执行任务，任务执行完毕后`xxx`毫秒后再次执行
  - 以上一次**执行完成**为基准,不可能出现任务并发执行。
- `@Scheduled(cron = "0 0 2 * * ?")`：通过`corn`表达式配置任务执行周期
  - `corn`表达式格式:`秒 分 时 日 月 周`



# 小知识

## 循环依赖

> 循环依赖指两个类互相依赖对方，如`A`类中有`B`属性，`B`类中有`A`属性,从而形成了一个依赖闭环。

**循环依赖问题在Spring中主要有四种情况：**

- 通过构造方法进行依赖注入时产生的循环依赖问题。
- 通过setter方法进行依赖注入且是在多例（原型）模式下产生的循环依赖问题。
- 通过setter方法进行依赖注入且是在单例模式下产生的循环依赖问题。
- 通过字段进行依赖注入且是在单例模式下产生的循环依赖问题

**Spring可以解决后两种情况产生的循环依赖问题,其他两种会在程序启动后产生异常**

------

SpringBoot2.6+默认禁止循环依赖，即使Spring可以解决此循环依赖问题，仍然会报错。

可以通过修改配置文件开启循环依赖

```properties
spring.main.allow-circular-references=true
```

------

Spring 在`DefaultSingletonBeanRegistry`类中维护了三个重要的缓存(Map)，称为三级缓存

- `singletonObjects`(一级缓存)：存放的是完全初始化好的、可用的 Bean实例，`getBean()`方法最终返回的就是这里面的Bean。此时Bean已实例化、属性已填充、初始化方法已执行、AOP代理（如果需要）也已生成。
- `earlysingletonObjects`(二级缓存)：存放的是提前暴露的Bean的原始对象引用或早期代理对象引用，专门用来处理循环依赖。当一个Bean还在创建过程中（尚未完成属性填充和初始化)，但它的引用需要被注入到另一个Bean时，就暂时放在这里。此时Bean已实例化（调用了构造函数），但属性尚未填充，初始化方法尚未执行，它可能是一个原始对象，也可能是一个为了解决AOP代理问题而提前生成的代理对象。
- `singletonFactories`(三级缓存)：存放的是 Bean 的`ObjectFactory`工厂对象。在`Spring`通过反射创建`Bean`之后，会将`Bean`对应的`ObjectFactory`对象放入三级缓存，通过它的`ObjectFacotry`方法可以得到早期`Bean`。

循环依赖解决的前提就是提前暴漏未属性注入完成的`Bean`。

当循环依赖间进行属性注入时，`Spring`会查询缓存，最终可以在三级缓存找到依赖的工厂对象，通过工厂对象获得早期`Bean`并将其放入二级缓存，然后在三级缓存中移除工厂对象，直接注入这个早期`Bean`即可。

**为什么要使用三级缓存，而不是二级缓存**

使用三级缓存的核心原因是为了正确处理需要AOP代理的Bean。三级缓存中缓存了能返回对象早期引用的`ObjectFactory`。当发生循环依赖时，需要提前获得未完全初始化的`Bean`，Spring会调用`ObjectFactory.getObject`方法，在方法中会判断：如果这个`Bean`需要代理，就提前生成一个代理对象放入二级缓存并返回；如果不需要被代理，就返回原始对象并放入二级缓存。相比于二级缓存，三级缓存提供了是否提前生成代理对象的动态决策能力，维持了`Bean`生命周期的完整性，又在循环依赖时特殊处理。



## Spring单例如何实现

Spring的单例并不是JVM级别的单例，而是IOC容器的单例。一个JVM中可以有多个IOC容器，每个容器各自维护一份单例Bean。

IOC容器中通过`Map`维护了一个单例池缓存，`key`为`beanName`,`value`为Bean实例。

当容器启动并调用`getBean`方法时，会先检查缓存：

- 如果缓存中存在对应`Bean`，直接返回缓存对象
- 如果缓存中不存在，则创建`Bean`，并将`Bean`放入缓存。

因此只有第一次获取`Bean`时才会创建并放进容器缓存，之后每次用的都是缓存里的同一个对象。



