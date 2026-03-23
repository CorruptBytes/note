# 概述

> SPringBoot是基于Spring框架(这里指Spring Framework框架)的子项目，其设计目的是用来简化Spirng应用程序的构建与开发。

## 特点

- 起步依赖

> 通过maven的依赖继承，springboot提供了一些整合了spring开发所需依赖的起步依赖，引入这一行依赖，就可以使用相关功能开发所需的一切组件

```xml
<!--SpringBootWeb开发的起步依赖-->
<dependency> 
	<groupId>org.springframework.boot</groupId>   				<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

- 自动配置

> 遵循约定大于配置的原则，Springboot能根据项目依赖自动配置大部分东西，如在boot程序启动后，一些bean对象会自动注入到ioc容器

- 内嵌服务器

> 内嵌Tomcat，Jetty等服务器，无需自己部署war包

- 外部化配置

> 配置文件不会一起打入war包，当需要修改配置时，只需要修改外部配置文件，无需重新编译打包，重新启动即可

- 不需要xml配置

> SpringBoot摒弃了xml文件配置，使用properties或yml文件配置

## Boot基础工程

> SpringBoot工程打包方式为jar包，因为不需要自己部署，spring boot自动部署

![SpringBoot工程结构](D:\笔记\图片\SpringBoot工程结构.png)

- 基础pom文件

> 所有boot工程都会继承一个父工程，用于管理起步依赖的版本

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <!--所有boot工程的父工程，用于管理起步依赖的版本，因此spring boot的起步依赖在引入时不需要写版本号-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.4.4</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.study</groupId>
    <artifactId>springboot-start</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot-start</name>
    <description>springboot-start</description>
    <url/>
    <licenses>
        <license/>
    </licenses>
    <developers>
        <developer/>
    </developers>
    <scm>
        <connection/>
        <developerConnection/>
        <tag/>
        <url/>
    </scm>
    <properties>
        <java.version>17</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

- 基础启动类

> springboot应用启动依赖于application启动类，也叫引导类。运行这个类，springboot程序就启动了

```java
	@SpringBootApplication
public class SpringbootStartApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootStartApplication.class, args);
    }

}
```

# Boot配置文件

> Spring Boot 提供了多种配置文件用于**集中管理应用的可变参数**,比如:

- 服务端口、日志路径、数据库连接
- 激活不同的环境（开发/测试/生产）
- 第三方组件参数（如 RabbitMQ、Nacos、Redis）

实现了**配置与代码分离**，便于不同环境的部署和管理。

------

## 主配置文件

> 主配置文件在任何情况下都会被优先加载。SpringBoot的应用主配置文件为`application.后缀名`，支持三种后缀:

- yml
- yaml
- properties

**优先级为:**properties > yml > yaml

------

`application`配置文件**无需显式声明或加载**，Spring Boot 启动时就会去查找它，并把里面的配置注入到应用上下文中,这也是SpringBoot约定大于配置的体现

- properties


> 采用键值对的方式进行配置，不同层级通过`.`来分隔

```
键层级1.键层级2=值
```

-  yml/yaml（常用）

> 这是SpringBoot最常用的配置格式

- 可以使用占位符`${配置键名}`在配置文件中引用其他配置值,多级键名用`.`分隔

## 环境配置文件

> 又叫做Profile配置文件,用于定义特定环境的配置,比如开发环境（`dev`）、测试环境（`test`）、生产环境（`prod`）。名称为`application-{环境名}.后缀`。

**激活方式:**

可以在主配置文件中配置激活环境，也可通过JVM参数或命令行参数配置。

```
spring:
  profiles:
    active: dev
```

- Spring Boot 会先加载 `application`文件，再加载对应的 profile 文件，Profile 文件的配置会覆盖 `application` 中的同名配置。
- 配置文件中的占位符是延迟解析的,会在 **整个环境配置加载完毕后** 再解析。因此可以在主配置文件中通过占位符引用环境配置文件的键,从而实现特定环境配置


## 配置信息读取

> 配置文件中的配置信息分为两种:第三方技术配置信息和自定义配置信息

### 第三方配置信息

> 引入对应起步依赖后，只要根据技术文档编写对应配置，起步依赖会自动读取并进行配置

### 自定义配置信息

> 有一些数据不适合写在java代码中，此时就可以写在配置文件中，这些信息就可以写在配置文件中，通过一些方法读取他,如发送邮件时发件人的相关信息

```yml
email:
  user: 804686559@qq.com
  code: dfasgaga
  host: smtp.qq.com
  auth: true
```

 - 获得配置文件中的信息，需要在对应变量(或者方法形参上)上使用@SpringBoot提供的注解@Value("${键名}")来使用，${}为el表达式，键名通过`.`连接不同层级。

```java
@Value("${student.age}")
private int age;
```

> 注意:
>
> - @value注解不可以用在局部变量上

 - 在实体类上加入@ConfigurationProperties(prefix = "前缀")，此时成员变量不需要加入@value注解，且当通过依赖注入获得这个bean时，配置信息会被自动注入

```java
@ConfigurationProperties(prefix = "email")
@Component
public class email {
    public String user;
    public String code;
    public String host;
    public boolean auth;
}
```

> 注意：
>
> - 这个实体类必须被注册为bean。
> - 前缀必须全部使用小写
> - 实体类的成员变量名与配置文件中的键名保持一致

# Bean相关配置

## Bean扫描

> Boot中的Bean扫描依赖于@SpringBootApplication注解，这是一个组合注解，其中一个子注解为@ComponentScan，用于bean扫描
>
> 如果需要修改扫描包的范围，可以在启动类再手动加入@ComponentScan注解

## Bean配置

## Bean注册条件

# 静态资源映射

> SpringBoot会自动将`resources`目录中约定的特定名字的文件夹映射为静态资源访问路径而无需对其编写`Controller`。

|        物理路径（项目中存放的位置）        |  URL访问路径   | 历史来源              | 推荐用途                          |
| :----------------------------------------: | :------------: | --------------------- | --------------------------------- |
|       `src/main/resources/static/**`       | `/` + 文件路径 | Spring Boot 官方默认  | 存放前端静态文件：JS、CSS、图片等 |
|       `src/main/resources/public/**`       | `/` + 文件路径 | 传统 Java Web 目录    | 兼容旧项目的静态资源迁移          |
|     `src/main/resources/resources/**`      | `/` + 文件路径 | Spring 旧项目默认支持 | 一般不使用，兼容性目录            |
| `src/main/resources/META-INF/resources/**` | `/` + 文件路径 | Servlet 规范          | 第三方依赖（JAR）提供静态页面     |

# 注解

## `Bean`定义相关

**`@Import`**

```
@Import(Class<?>[] value)
```

- 在**不依赖组件扫描（`@ComponentScan`）**的前提下，**显式地将指定类注册为 Spring 容器中的 Bean**。
- `@Import`在导入类时，不关心该类是否已经被声明为`Bean`,即类上是否具有`@Component`等声明`Bean`的注解。

通常会使用`@Import`导入以下四种类：

- 普通`Bean`
- 配置类：配置类中定义的所有`Bean`也会被导入
  - 配置类不是必须加入`@Configuration`注解，如果加了，则会对该类进行`CGLIB`代理，保证内部`Bean`为单例。
- `ImportSelector`的实现类，`ImportSelector`接口的方法要求返回多个类的全限定名组成的数组，这些类均会被注册为`Bean`。一般用于加载配置文件中的类
- `ImportBeanDefinitionRegistrar`的实现类，实现这个接口的`registerBeanDefinitions`的方法，并通过参数`BeanDefinitionRegistry`注册`Bean`，这些`Bean`均会被注册到IOC容器中。

Spring Boot 的自动配置机制以及绝大多数 `@EnableXXX` 功能启用注解，本质上都是通过 `@Import` 将相关配置类或注册器引入 Spring 容器中实现的

## 配置相关

- **@ConfigurationProperties**

  根据公共前缀`prefix`与类属性名将对应的配置属性注入到类属性

  ```
  @ConfigurationProperties(prefix="")
  ```

  - 使用在类上
  - 配置属性可以来自于配置文件等任何SpringBoot支持的配置方式
  - 通常配合`@EnableConfigurationProperties`或`@Component`注解使用

## 功能启用相关

@EnableConfigurationProperties

```java
@EnableConfigurationProperties(xxxProperties.class)
//参数说明
```

- 启用带有`@ConfigurationProperties` 注解类的属性绑定功能,并将该类注册为Spring容器中的Bean

**`@EnableAsync`**

```java
@SpringBootApplication
@EnableAsync
public class Application {
}

@Service
public class EmailService {

    @Async
    public void sendEmail() {
        System.out.println("发送邮件线程：" + Thread.currentThread().getName());
    }
}
```

- **开启 Spring 的异步方法调用能力**,允许你把一个方法标记为**异步执行**，调用方**立即返回**，真正的逻辑由**线程池**在后台执行。底层基于AOP代理+线程池实现。

- 默认情况下，会使用`SimpleAsyncTaskExecutor`执行`@Async`标记的方法

  - **不复用线程**
  - 无队列
  - 高并发下可能 **OOM**

- 因此通常会自定义线程池添加到IOC容器中，并通过`@Aysnc`注解指定需要使用的线程池。

  ```java
      @Bean("myAsyncExecutor")
      public Executor asyncExecutor() {
          ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
          executor.setCorePoolSize(5);
          executor.setMaxPoolSize(10);
          executor.setQueueCapacity(100);
          executor.setThreadNamePrefix("async-");
          executor.initialize();
          return executor;
      }
      @Async("myAsyncExecutor")
  	public void task() {}
  ```

**`@EnableConfigurationProperties`**

```
@EnableConfigurationProperties(Class<?>[] value)
```

- 不依赖组件扫描，将 `value` 中标注了 `@ConfigurationProperties` 的类注册为 Spring Bean，
  并通过配置属性绑定机制，将配置文件中的属性绑定到对应的属性类实例上。
- 通常用于自动配置功能中，通过本地文件配置外部自动配置导入的Bean

# 小知识

## 手动创建Boot工程

> 分为三步:创建Maven工程，继承父工程并引入起步依赖，提供起步类

1. 创建maven工程，骨架选择quickstart

![image-20250414210009677](D:\笔记\图片\image-20250414210009677.png)

2. 继承父工程，并引入起步依赖，需要指定父工程的版本号

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.4.4</version>
  </parent>
  <groupId>com.study</groupId>
  <artifactId>springboot-manual</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>springboot-manual</name>
  <url>http://maven.apache.org</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
  </dependencies>
</project>

```

3. 编写启动类，启动类一般为当前工程名+Application，使用驼峰命名

```java
//添加注解，表示这是一个启动类
@SpringBootApplication
public class SpringBootManualApplication
{
    public static void main( String[] args )
    {
		//调用启动方法，传入启动类字节码文件与main方法的ars参数
        SpringApplication.run(SpringBootManualApplication.class,args);
    }
}

```

4. 创建resources目录，在目录中创建并编写配置文件，配置文件一般命名为application.properties

## SpringBoot配置项官方说明

```http
https://docs.spring.io/spring-boot/appendix/application-properties/index.html#appendix.application-properties
```

- 常用配置

|           配置项           |         说明         | 默认值 |
| :------------------------: | :------------------: | :----: |
|        server.port         | 修改服务器启动端口号 |  8080  |
| server.sevlet.context-path |  修改服务器虚拟目录  |        |
|                            |                      |        |

## 自动配置和`@EnableXXX`注解

`SpringBoot`中提供了许多`@EnableXXX`类型的注解，它们通常用于动态的开启某些功能，其底层原理是使用`@Import`注解导入功能的配置类，实现`Bean`的动态加载。

<h3>@EnableXXX的实现原理</h3>

默认情况下,`@SpringApplication`注解只会扫描注解所在包及其子包。如果想要获得外部`Jar`包中定义的`Bean`，需要手动在启动类上加入`ComponentScan`重新定义扫描范围(这会覆盖`@SpringApplication`默认定义的扫描范围)。

`Spring`提供了`@Import(类名.class)`注解，这个注解在配置类上使用，用于将指定类注册为 Spring 容器中的 Bean。如果这个类是被`@Configuration`定义的配置类，其内部的Bean也会被注入Spring容器。

Spring和第三方`Jar`包基于`@Import`注解封装了`@EnableXXX`注解，当使用`@EnableXXX`注解时，其内部的`@Import`注解会将所需的`Bean`导入到IOC容器中，从而开启相关功能。

<h3>自动配置原理</h3>

`SpringBoot`自动配置功能通过`@EnableAutoConfiguration`注解启用，这是一个特殊的功能启用注解。这个注解中的`@Import`向IOC容器中注册了`AutoConfigurationImportSelector`类型的`Bean`。

`AutoConfigurationImportSelector`会执行以下工作：

1. 在 **Spring Boot 应用启动的配置类解析阶段**，`AutoConfigurationImportSelector` 扫描并聚合类路径上
   所有 jar 包中的 `META-INF/spring.factories` 文件，获取声明的自动配置类全限定名列表。

   ```properties
   # Auto Configure
   org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
   org.mybatis.spring.boot.autoconfigure.MybatisAutoConfiguration
   ```

2. `AutoConfigurationImportSelector` 对候选自动配置类进行排除和排序处理，并将需要导入的自动配置类全类名返回给 Spring。

3. Spring 随后将这些自动配置类作为 **配置类** 进行解析，并结合 `@ConditionalOnClass`、`@ConditionalOnBean`、`@ConditionalOnProperty` 等条件注解，决定自动配置类是否生效。

4. 符合条件的自动配置类会被注册为 `BeanDefinition`， 其内部声明的 `Bean` 最终在容器刷新阶段实例化并注入到**Spring Boot** 的 IOC 容器中。

**`@EnableAutoConfiguration`被整合到了`@SpringApplication`注解中，因此自动配置功能是默认启用的**

### 自定义Starter起步依赖

**起步依赖（Starter）的主要功能是：依赖聚合与自动配置。**依赖聚合用于将实现某一功能所需的全部依赖统一整合，使应用只需通过引入一个 `Starter` 依赖，即可一次性获得完整的功能支持。除了依赖聚合，Starter 还需要**依赖一个自动配置模块**， 该模块基于`Spring Boot`的自动配置机制， 在满足条件的情况下将相关组件自动注册到 **Spring Boot** 的 IOC 容器中，从而实现“开箱即用”。

**自定义`Starter`的核心是编写自动配置模块**

1. 在自动配置模块中编写自动配置类

   ```java
   @Configuration
   @EnableConfigurationProperties(MyFeatureProperties.class)
   @ConditionalOnClass(MyFeatureService.class)
   @ConditionalOnProperty(prefix = "my.feature", name = "enabled", havingValue = "true", matchIfMissing = true)
   public class MyFeatureAutoConfiguration {
   
       @Bean
       @ConditionalOnMissingBean
       public MyFeatureService myFeatureService() {
           return new MyFeatureService();
       }
   }
   ```

2. 在自动配置模块的`META-INF/spring.factories`文件下配置需要被加载的自动配置类列表

   ```properties
   org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
   com.xxx.autoconfigure.MyFeatureAutoConfiguration
   ```

   - 从SpringBoot2.7开始，`spring.factories` 被标记为 **已弃用**，取而代之的是`META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`,Spring3.0开始，`spring.factories`被完全移除。

     ```
     #一行一个自动配置类的全类名
     com.xxx.autoconfigure.MyFeatureAutoConfiguration
     ```



> YAML（YAML Ain't Markup Language），一种数据序列化格式。

```语法
1. 小写敏感
2. 属性层级关系使用多行描述，每行结尾使用冒号结束
3. 使用缩进表示层级关系，同层级左侧对齐，只允许使用空格（不允许使用Tab键）
4. 空格的个数并不重要，只要保证同层级的左侧对齐即可。
5. 属性值前面添加空格（属性名与属性值之间使用冒号+空格作为分隔）
6. # 表示注释
7. 数组数据在数据书写位置的下方使用减号作为数据开始符号，每行书写一个数据，减号与数据间空格分隔

server:
  port: 10086
  servlet:
    context-path: /start

```

```优点
1. 容易阅读
2. 容易与脚本语言交互
3. 以数据为核心，重数据轻格式
```

## Boot整合mybatis

1. 引入boot整合mybatis的依赖和数据库驱动依赖

```xml
 <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>3.0.4</version>
        </dependency>

        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>
```

2. 配置数据源信息

```yml
spring:
  datasource:
    username: root
    password: 1234
    url: jdbc:mysql://localhost:3306/emp
    driver-class-name: com.mysql.cj.jdbc.Driver
```

3. 在需要的mapper类上加上@mapper注解，这个注解将标注的类声明为映射类并将其注册到Spring容器中交给Spring管理

## Spring中的注解

- @ComponentScan(basepackages="包名")

> 加在配置类或启动类上的注解，用于bean扫描，当不指定参数时，默认扫描当前类所在包及其子包

- 自定义Bean注册

| 注解        | 说明                 | 位置                                         |
| ----------- | -------------------- | -------------------------------------------- |
| @Component  | 声明bean的基础注解   | 不属于以下三类时,用此注解                    |
| @Controller | @Component的衍生注解 | 标注在控制器类上                             |
| @Service    | @Component的衍生注解 | 标注在业务类上                               |
| @Repository | @Component的衍生注解 | 标注在数据访问类上(由于与mybatis整合,用的少) |

- 第三方Bean注册

> 通常情况下，注册Bean需要修改被注册类的源码，当无法修改其源码时，可以通过@Bean和@Import注解配合注册这个Bean

1. @Bean注解为方法注解，此时，Spring会将这个方法的返回值注入IOC容器管理。一般会单独创建一个配置类用于管理第三方Bean，配置类要加入@configuration声明为配置类

```java
@Bean
public Country country() {
     return new Country();
}
```

> 注意:
>
> - Bean注解可以配置Bean的名字，如果不指定，则默认为方法名
> - 如果在Bean声明的方法中需要使用其他Bean，只需要作为形参传入即可，Spring会自动注入

2. @Import用于导入配置类，当然，如果配置类可以被扫描到时，不需要导入。@Import还可以导入ImportSelector接口的实现类，这个接口规定了selectImports方法，这个方法返回一个字符串数组，这个数组内部为想要导入的配置类的全类名

```
public class CommonImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[]{};
    }
}
```

- 注册条件注解

> SPirngBOot提供的设置注册生效条件的注解@Conditional,由于@conditional注解使用起来比较麻烦，一般使用其衍生注解

![image-20250415171253343](D:\笔记\图片\image-20250415171253343.png)

## 启动类的main方法

```
SpringApplication.run(SpringbootRegisterApplication.class, args);
此方法会返回一个context容器对象，如果需要使用，可以通过变量接收这个对象
```



## 跨域

> 跨域请求可能浏览器拦截

**同源策略（Same-Origin Policy）**

> 只有请求的 **协议（protocol）+ 域名（host）+ 端口（port）** 都相同，
>  才被认为是“同源”。否则就是跨域

| 请求类型                       | 是否受同源策略限制 | 是否需要 CORS       |
| ------------------------------ | ------------------ | ------------------- |
| AJAX（XMLHttpRequest / fetch） | ✅ 是               | ✅ 需要后端开启 CORS |
| `<script src="...">`           | ❌ 否               | ❌ 不需要            |
| `<img src="...">`              | ❌ 否               | ❌ 不需要            |
| `<link href="...">`（CSS）     | ❌ 否               | ❌ 不需要            |
| `<iframe src="...">`           | ✅ 是（部分受限）   | 可能需要            |
| `<video src="...">`            | ❌ 否（播放可跨域） | ❌ 不需要            |

- 图片、脚本、样式表这类 **静态资源请求** 被认为是 “**无害的跨域请求**”，浏览器允许跨域加载它们

跨域是由于浏览器的同源策略导致的，因此跨域异常只会在前端发生。

### 跨域问题的解决方式

<h4><code>JSONP</code></h4>

这是一种原始的跨域解决方案，实现简单，且对于各种浏览器的兼容性很好。但只能发送`GET`请求，且存在安全性问题，因此已经基本废弃。

**原理**

`JSONP`利用了`<script src="URL">`不受同源策略限制，可以加载任意域名JS脚本的漏洞，绕过了同源策略。

```javascript
<script src="http://other-domain.com/data?callback=handle"></script>
<script>
  function handle(data) {
    console.log(data); // 拿到跨域返回的数据
  }
</script>
```

当服务器接收到请求后，会直接返回调用`callback`函数的JS代码并填入返回参数：

```javascript
handle({ name: "Tom", age: 18 });
```

在前端，jQuery 提供了封装好的 JSONP 支持，可以快速发起跨域请求；在后端，使用 Jackson 的 `MappingJacksonValue` 配合 Spring Boot，可以自动将原本的 JSON 数据包装成 `callback(JSON)` 格式，从而与前端 JSONP 请求配合，实现跨域数据交互。

<h4><code>CORS</code></h4>

全称为跨源资源共享(`Cross-Origin Resource Sharing`)，是浏览器的一种跨域资源访问机制，通过服务器设置响应头，告诉浏览器哪些跨域请求是被允许的，从而在保证安全的前提下实现跨域 AJAX 通信。

**原理**

`CORS`不需要前端提供特殊支持，前端只需正常发送`AJAX`请求，服务器通过添加响应头声明“允许哪些跨域请求”，常用响应头有：

|                 头                 |                    说明                     |
| :--------------------------------: | :-----------------------------------------: |
|   `Access-Control-Allow-Origin`    |   指定允许的请求源（可以是 `*` 表示所有）   |
|   `Access-Control-Allow-Methods`   | 允许的 HTTP 方法（GET、POST、PUT、DELETE…） |
|   `Access-Control-Allow-Headers`   |                允许的请求头                 |
| `Access-Control-Allow-Credentials` |       是否允许发送 Cookie / 认证信息        |

对于类型为`GET`/`POST`且没有自定义头的请求：

```
浏览器发送 AJAX 请求
       ↓
服务器响应，并加上 Access-Control-Allow-Origin
       ↓
浏览器检测是否允许
       ↓
允许 → JS 可以拿到响应数据
```

对于使用了自定义头或非 GET/POST请求

```
浏览器先发 OPTIONS 请求（预检）
       ↓
服务器返回允许的源、方法、头
       ↓
浏览器确认后，再发送实际请求
```

**实现**

`SpringBoot`提供了`@CrossOrigin` 注解，可以添加在`Controller`类或内部方法上，表示允许跨域。

```
@RestController
@RequestMapping("/api")
public class UserController {

    @CrossOrigin(origins = "http://localhost:3000") // 允许前端地址跨域
    @GetMapping("/user")
    public User getUser() {
        return new User("Tom", 18);
    }
}
```

- `origins`：允许跨域的源，可以是单个 URL 或 `*`（允许所有）

  `@CrossOrigin` 还支持：

  - `methods`：允许的 HTTP 方法（GET, POST…）
  - `allowedHeaders`：允许的请求头
  - `allowCredentials`：是否允许发送 Cookie

也可以使用`WebMvcConfigurer`进行全局配置

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")            // 匹配所有接口
                .allowedOrigins("http://localhost:3000") // 允许的源
                .allowedMethods("GET","POST","PUT","DELETE") // 允许的方法
                .allowedHeaders("*")            // 允许请求头
                .allowCredentials(true);        // 允许发送 Cookie
    }
}
```

**优点**

- 实现简单，前端无需提供任何支持，仅需服务端和浏览器提供支持就可以实现跨域

**缺点**

- 需要浏览器支持，IE10以下并不支持CORS。

<h4><code>Nginx</code>反向代理</h4>

通过 Nginx 反向代理，前端静态资源和 AJAX 请求都走同一个域名和端口(即Nginx服务器)，`AJAX`请求到达后会转发到后端，浏览器视角下满足同源策略，从而安全、透明地实现跨域访问后端服务。

| 类型     |  请求目标  |          实际请求          |
| -------- | :--------: | :------------------------: |
| 正向代理 | 目标服务器 | 代理帮客户端请求目标服务器 |
| 反向代理 | 代理服务器 | 代理帮客户端请求后端服务器 |

- 正向代理和反向代理的本质是主动使用代理的一方不同。对于正向代理，是客户端主动使用代理访问目标服务器；对于反向代理，是 服务端主动通过代理服务器接收客户端请求并转发到真实后端服务器，从而隐藏真实后端。
- 正向代理典型用途有VPN，网络爬虫；反向代理典型用途有负载均衡，避免跨域，CDN加速。