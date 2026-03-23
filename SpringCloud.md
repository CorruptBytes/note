# 概述

> SpringCloud是目前全球使用最广泛的微服务框架。它集成了各种微服务功能组件，并基于SpringBoot实现了组件的自动装配。

![image-20250827152718620](D:\笔记\图片\image-20250827152718620.png)

## 组成

- SpringCloud框架的核心是Spring 官方提供的`Spring Cloud Commons`模块,它定义了微服务开发需要遵循的公共接口和约定，提供服务注册、配置管理、负载均衡、断路器等公共抽象。
- 社区或厂商基于`Spring Cloud Commons`提供具体实现。因此，SpringCloud中的各组件均遵循相同的接口与规范，是可以自由插拔替换成不同的实现的。

**Spring Cloud = Spring 官方提供的公共规范（Commons） + 社区或厂商提供的具体实现组件。**

## 相关依赖

**`SpringCloud`版本管理依赖**

```xml
        <dependencies>
            <!-- springCloud -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
```

- 版本管理依赖，统一管理SpringCloud中各组件的版本

**Alibaba组件版本管理依赖**

```xml
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring-cloud-alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
```

- 版本管理依赖，统一管理`Spring Cloud Alibaba`实现的各组件的版本。

## 服务拆分

> 将单体项目中的不同服务拆分成一个个微服务是实现微服务架构的第一步

### 拆分原则

- **高内聚**：每个微服务的职责要尽量单一，包含的业务相互关联度高、完整度高。

- **低耦合**：每个微服务的功能要相对独立，尽量减少对其它微服务的依赖。
- 不同微服务，不要重复开发相同业务
- 微服务数据独立，不要访问其它微服务的数据库,每个微服务有自己独立的数据库
- 微服务可以将自己的业务暴露为接口，供其它微服务调用

### 拆分方式

- **纵向拆分**：按照业务模块来拆分
- **横向拆分**：抽取公共服务，提高复用性

**对于大型项目来说，通常是两种拆分方式配合使用**

## 项目结构

> 通常来说，微服务项目的结构通常有两种:

**独立Project**

> 每一个模块都是一个独立的项目，放在不同的仓库管理，这种情况下，各个项目完全独立，耦合程度最低

**Maven聚合**

> 各个微服务模块成为一个父工程的Module。父工程专门用来作依赖管理，适用于小型的微服务项目

# 远程调用

> 微服务中的各个模块在物理上相互独立，微服务内不能直接调用其他微服务的功能。因此微服务需要通过发送Http请求访问所依赖微服务暴露出的接口来调用其他微服务的功能。这称为远程调用。

**提供者**/**消费者**

> 提供者为一次业务中，提供接口给其他微服务调用的微服务；消费者为一次业务中，调用其他微服务提供接口的微服务

- 提供者与消费者是相对而言的，在一次业务中，一个微服务可以既是消费者，又是提供者

------

## RestTemplate

> 由Spring提供的，专门用于微服务之间发送Restful风格的Http请求的类。

```java
//url即服务接口的url,后面的参数即反序列化类型
restTemplate.getForObject(url, User.class);
//url路径中可以使用{}占位符，占位符会匹配Map的键，并将值填入.
public <T> ResponseEntity<T> exchange(
	String url, // 请求路径
	HttpMethod method, // 请求方式
	@Nullable HttpEntity<?> requestEntity, // 请求实体，可以为空
 	Class<T> responseType, // 返回值类型
	Map<String, ?> uriVariables // 请求参数
)
//如果返回值为没有字节码的特殊类型或者集合,通过ParameterizedTypeReference对象的泛型指定。
ResponseEntity<List<ItemDTO>> response = restTemplate.exchange("http://localhost:8081/items?ids={ids}", HttpMethod.GET, null,
                new ParameterizedTypeReference<List<ItemDTO>>() {
                }, Map.of("ids", CollUtils.join(itemIds, ",")));
```

- 这种方式是与语言和技术无关的，只是利用其他微服务暴露的接口，发送Http请求获得信息。

## OpenFegin

> 一个声明式的http客户端，由Spring Cloud在Feign基础上做的增强版。根据接口+注解，自动生成 HTTP 客户端，调用其他微服务暴露出的API。

### 特性

- **声明式：**只需**声明**调用规则(通常由接口和注解控制)，无需写调用逻辑，底层HTTP请求的细节由框架自动完成。
- 学习成本低:基于**SpringMVC注解**声明Http请求
- 可插拔式替换

### 相关依赖

**OpenFeign起步依赖**

```xml
<!--OpenFeign-->
	<dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

**负载均衡**

```xml
<!--负载均衡-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

- 早期OpenFeign的负载均衡是依赖于Ribbon，现在已经改为loadbalancer

**ok-http**

```xml
<!--ok-http-->
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-okhttp</artifactId>
</dependency>
```

- 支持连接池的OpenFeign底层Http客户端

### 使用

1. 在启动类上添加`@EnableFeginClients`注解，启用OpenFeign

   ```java
   @EnableFeignClients
   @SpringBootApplication
   public class CartApplication {
       public static void main(String[] args) {
           SpringApplication.run(CartApplication.class,args);
       }
   }
   ```

2. 编写FeignClient接口，通过接口方法与注解声明Http请求信息

   ```java
   //FeignClient接口标记这是一个FeignClient，并声明了其代理的微服务名
   @FeignClient(value = "item-service")
   public interface ItemClient {
       @GetMapping("/items")
       List<ItemDTO> queryItemByIds(@RequestParam("ids") Collection<Long> ids);
   }
   ```

3. 框架根据接口自动生成实现类并注册为Bean，可以通过自动注入获得Bean

   ```java
   @Autowired
   private final ItemClient itemClient;
   itemClient.queryByIds(itemIds.stream().toList());
   ```

### 底层Http客户端

> OpenFeign发送HTTP请求依赖其他可替换的客户端模块，可按需选择：

- **HttpURLConnection**：由JDK提供，OpenFeign底层的默认实现，不支持连接池

 - **Apache HttpClient**：支持连接池
 - **OKHttp**：支持连接池

------

**更换Http客户端:**

```yml
feign:
  okhttp: #如果更换为httpClinet，则键为httpclient
    enabled: true
```

### 服务接口复用方式

> 对于大型项目而言，一个微服务可能被许多个微服务依赖，如果在每个微服务中都编写这个微服务的`Client`，代码复用性差,为了避免重复定义接口与实体类，可以将`Fegin Client`抽取出来，常见两种抽取方式:

**方式一:统一抽取公共依赖**

将所有微服务的`Feign Client`接口与相关实体类放在一个统一的 `common-api` 依赖中。 每个微服务只要引入该依赖，就能调用任意其他服务。

**优点**：

1. **集中管理**，所有接口、实体类放在一起，方便统一维护。
2. 依赖少，所有微服务只需要引入一个 `common-api` 包。
3. 避免不同依赖版本不一致的问题。

**缺点**：

1. 包含所有服务的接口，**体积大**，可能有很多服务用不到的类。
2. 服务间耦合度高，一个小服务的改动也可能导致整个 `common-api` 重新发布。
3. 不利于模块解耦，容易形成“大泥球依赖”。

**方式二:每个微服务单独抽取 API 依赖**
每个微服务单独维护自己的 `xxx-service-api` 模块，包含本服务的`Feign Client`与相关实体类。其他微服务需要调用时，只需引入对应服务的 `xxx-service-api`。

**优点**：

1. **低耦合**，只依赖所需的服务 API，清晰明了。
2. 便于服务独立升级，不会因为无关服务的改动影响自己。
3. 结构更符合 **领域驱动设计（DDD）** 的理念。

**缺点**：

1. 需要维护多个 `api` 模块，数量多时管理相对复杂。
2. 版本管理要求高，若 `service-api` 没有及时更新，可能出现调用方和提供方不匹配的问题。
3. 在多服务场景下，依赖配置稍显繁琐。

- 如果是 **小型系统**，微服务数量少，可以用 **方式一（统一依赖）**，减少模块和依赖管理成本。
- 如果是 **中大型系统**，微服务数量多，推荐 **方式二（分服务抽取 API）**，可以保持清晰的依赖关系，避免“大而全”的公共依赖。

**注意点**

- 如果api模块与微服务模块用到了相同的DTO,删除微服务模块的DTO,通过api模块引入这个类

### 配置

> OpenFeign的大部分配置均是在配置类上进行，通过`@EnableFeignClients`注解指定所加载的配置类使其生效。

```java
@EnableFeignClients(defaultConfiguration = DefaultFeignConfig.class)
```

- 配置类是一个普通类，内部通过定义方法声明Bean的方式进行配置。

#### 日志

> OpenFeign只会在FeignClient所在包的日志级别为**DEBUG**时，才会输出日志。而且其日志级别有4级：

- **NONE**：不记录任何日志信息，这是默认值。
- **BASIC**：仅记录请求的方法，URL以及响应状态码和执行时间
- **HEADERS**：在BASIC的基础上，额外记录了请求和响应的头信息
- **FULL**：记录所有请求和响应的明细，包括头信息、请求体、元数据。

**Feign默认的日志级别就是NONE**

------

**通过在配置类中声明Bean定义日志级别**

```java
public class DefaultFeignConfig {
    @Bean
    public Logger.Level fullFeignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```

#### 拦截器

> OpenFeign提供了拦截器接口，所有由OpenFeign发起的请求都会先调用拦截器处理请求然后再发出

```java
public interface RequestInterceptor {
    /**
    * Called for every request. Add data using methods on the supplied {@link RequestTemplate}.
    */
    void apply(RequestTemplate template);
}
```

**在配置类中声明Bean定义拦截器**

```java
public class DefaultFeignConfig {
    @Bean
    public RequestInterceptor userInfoRequestInterceptor() {
        return new RequestInterceptor() {
            @Override
            public void apply(RequestTemplate template) {
                Long userInfo = UserContext.getUser();
                if (userInfo != null) {
                    template.header("user-info",userInfo.toString());
                }
            }
        };
    }
}
```

### 注解

- 开启OpenFeign功能并进行相关配置

  ```
  @EnableFeignClients
  ```

  - `basePackages`:启动类默认扫描所在包及其子包,当Client位于范围之外时，通过`basePackages`参数指定扫描包
  - `clients`:扫描不到Client时，通过字节码指定Client。
  - `defaultConfiguration`:指定Feign配置类字节码文件

- 声明FeignClient

  ```
  @FeignClient()
  ```

  - `value`：指定服务名
  - `path`:声明请求的公共前缀,FeignClient不能使用@RequestMapping声明公共前缀
  - `fallbackFactory`:远程调用出现异常后进行处理的`fallbackFactroy`的字节码文件
  - `contextId`

# 服务治理

> 服务治理是微服务架构中的一项核心能力,主要用于 **管理、协调、优化各个微服务之间的关系和调用**。治理内容包括:

- **服务注册与发现**：服务上线时自动注册，调用方通过注册中心找到目标服务（不用写死 IP 和端口）。
- **负载均衡**：多个服务实例时，请求按策略分发，避免某台过载。
- **熔断与限流**：某个服务不可用时，快速失败或限流保护，避免雪崩。
- **监控与日志**：服务调用链追踪，错误率统计，便于定位问题。
- **配置管理**：动态下发配置，而不需要重启服务。
- **健康检查**:定时检查服务是否可用

可以说，**远程调用** 解决的是独立的不同服务之间的调用，而**服务治理** 解决的是 **“如何让这种调用更稳定、更高效、更可控”**。

------

## 注册中心

> 注册中心是微服务架构中实现服务治理的核心组件，核心职责是管理服务的注册与发现，一些注册中心也会额外提供 **健康检查、负载均衡、配置管理** 等服务治理相关的能力。

- 服务注册

  > 微服务在启动时，把自己的信息（如 **服务名、IP地址、端口、健康状态** 等） **登记到注册中心**,以便其他服务可以发现它。

- 服务发现

  > 服务消费者在需要调用某个服务时，通过 **注册中心拉取可用的服务实例地址列表**，无需硬编码服务的具体 IP 或端口。

**通过服务名而不是具体IP地址或端口调用服务**是注册中心的核心优势

# Nacos

> Nacos是由阿里巴巴研发的注册中心组件，已加入SpringColudAlibaba中。

## 安装部署

## 数据存储

> Nacos 需要一个数据源来持久化各种信息，包括服务注册信息、配置管理信息、命名空间及元数据，以及集群状态和元信息等。

### 单机部署

- 此时默认使用 **内嵌的 Derby 数据库**，不需要额外配置。

### 集群部署

- 内嵌数据库不支持多节点并发访问，因此 **集群模式下不适用**。

- 必须配置 **外部数据库（如 MySQL）** 来持久化服务注册信息和配置数据。
- 集群中的各个 Nacos 节点都会连接同一个外部数据库，从而保证 **数据一致性** 和 **服务可用性**。

------

1. 初始化数据库，创建相关表

   <iframe src="D:\笔记\resource\mysql-schema.sql" width="100%" height="250px"></iframe>

2. 通过加载环境变量的方式更改配置(也可挂载配置文件修改配置)

   <iframe src="resource\custom.env" width="100%" height=150px><iframe/>

   ```bash
   docker run -d \
   --name nacos \
   --env-file ./nacos/custom.env \
   -p 8848:8848 \
   -p 9848:9848 \
   -p 9849:9849 \
   --restart=always \
   nacos/nacos-server:v2.1.0-slim
   ```

## 健康检查

> Nacos将服务实例分为两种:临时实例与非临时实例,默认所有服务均是临时实例。对于不同的实例，Nacos会通过不同的方式进行健康检查。

- **临时实例:**临时实例会每隔一段时间向Nacos发送心跳来确保服务的健康，如果长时间未收到某个服务的心跳，注册中心会认为该服务不可用，将其从注册列表中剔除
- **非临时实例:**Nacos会主动请求询问非临时实例确保服务健康，如果服务不可用，Nacos不会剔除非临时实例，而是会等待其恢复

------

如果服务出现变更，Nacos会**主动向订阅了该服务的消费者推送消息**

## 服务注册

> 微服务启动时，将自己的相关信息注册到Nacos节点，以便其他服务可以发现它。

------

**服务注册无需编码，引入依赖并配置相关信息即可**

1. 引入nacos discovery依赖

   ```xml
   <!--nacos 服务注册发现-->
   <dependency>   
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
   </dependency>
   ```

2. 配置Nacos相关信息以及服务信息

   ```yaml
   spring:
     application:
       name: item-service # 服务名称
       cloud:
         nacos:
           server-addr: 192.168.150.101:8848 # nacos地址
   ```


## 服务发现

> 微服务消费者通过服务名向Nacos注册中心拉取可用的服务列表。

1. 引入nacos discovery依赖

   ```xml
   <!--nacos 服务注册发现-->
   <dependency>   
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
   </dependency>
   ```

2. 配置Nacos相关信息

   ```yaml
   spring:
     cloud:
       nacos:
         server-addr: 192.168.150.101:8848 # nacos注册中心
         discovery:
           namespace: f923fb34-cb0a-4c06-8fca-ad61ea61a3f0
           group: DEFAULT_GROUP
           ip: 192.168.150.101 #指定当前服务在注册到 Nacos 时使用的 IP 地址,默认会自动检测机器IP，但是在某些环境中自动检测的IP是无法对外访问的(Docker、虚拟机、容器环境),因此需要手动指定 IP
           register-enabled: false #是否注册此服务到Nacos中，默认为True
   ```

   

3. 通过DiscoveryClient类拉取服务实例,并根据负载均衡策略挑选一个实例发送请求

   ```java
   // 1.根据服务名称，拉取服务的实例列表
   List<ServiceInstance> instances = discoveryClient.getInstances("item-service");
   // 2.负载均衡，挑选一个实例
   ServiceInstance instance = instances.get(RandomUtil.randomInt(instances.size()));
   // 3.获取实例的IP和端口
   URI uri = instance.getUri();
   // 通过RestTemplate发送http请求
   ```

   - Nacos会自动注册一个DiscoveryClient类型的Bean，并完成配置，使用时可以直接注入。

## 配置管理

> Nacos不仅提供了注册中心的功能，还提供了配置中心的功能，主要实现了配置共享和配置热更新。解决了下列问题:

- 微服务重复配置过多，维护成本高
- 业务配置经常变动，每次修改都要重启服务
- 网关路由配置写死，如果变更需要重启网关

### 相关依赖

- Nacos-config

  ```xml
  <dependency>
  	<groupId>com.alibaba.cloud</groupId>
  	<artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
  </dependency>
  ```

- bootstrap

  ```xml
  <!--读取bootstrap文件-->
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-bootstrap</artifactId>
  </dependency>
  ```

  - SpringCloud拉取远程配置的时机早于`Application`配置文件被加载
  - 此时需要将Nacos相关信息等配置到`bootstrap`引导配置文件中
  - SpringBoot在`2.x`版本后取消了bootstrap配置文件，需要单独引入依赖

### 配置共享

> 有许多插件的配置对于所有微服务而言是相同的，这一类配置可以将其配置在Nacos中，通过Nacos实现共享配置。

1. 在Nacos服务端添加配置文件

   - 配置文件的`DataId`要带上后缀名
   - 配置文件中可以使用通配符

2. 在`bootstrap`文件中配置相关信息

   ```yml
   spring:
     application:
       name: cart-service # 服务名称
     profiles:
       active: dev
     cloud:
       nacos:
         server-addr: 192.168.150.101:8848 # nacos地址
         config:
         file-extension: yaml # 文件后缀名
         shared-configs: # 共享配置
           - data-id: shared-jdbc.yaml
           - data-id: shared-log.yaml
   ```

   - 除了在`bootstrap`文件中配置的`data-id`对应的配置文件,在服务启动时，会自动读取`data-id`为`[spring.application.name].[file-extesnion]`和`[spring.application.name]-[spring.active.profile].[file-extesnion]`的配置文件，如果不存在则忽略。

### 配置热更新

> 当配置文件更新后，无需重启即可使新配置文件生效

1. 热更新依赖于配置中心中特殊`data-id`的配置文件

   - 这个配置文件的`data-id`为`[spring.application.name]-[spring.active.profile].[file-extesnion]`,其中`profile`可选

2. 微服务需要依赖特定的注解读取热更新的配置属性

   - 方式一

     ```java
     @ConfigurationProperties(prefix = "hm.cart")
     public class CartProperties {
     	private int maxItems;
     }
     ```

   - 方式二

     ```java
     @RefreshScope
     public class CartProperties {
         @Value("${hm.cart.maxItems}")
         private int maxItems;
     }
     ```

# 网关

> 网络关口，负责请求的路由,转发,身份校验。在微服务架构中，网关也是一个微服务，通过将网关注册到注册中心，网关可以获取所有微服务的信息。这时，只需要知道网关的服务地址，无论是前端还是后端，都可以通过网关地址访问网关，再由网关路由转发到所需的微服务。无需再暴露所有微服务的地址。

------

**SpringCloud中提供了两种网关组件的实现:**

## **Spring Cloud Gateway**

> 由SpringCloud实现,基于WebFlux响应式编程,无需调优即可获得优异性能

### 快速使用

1. 创建单独的网关模块，将其注册到注册中心

### 相关依赖

- GateWay

  ```xml
  <!--网关-->
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-starter-gateway</artifactId>
          </dependency>
  ```

- 负载均衡

  ```xml
          <!--负载均衡-->
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-starter-loadbalancer</artifactId>
          </dependency>
  ```

  - 网关同样可以实现负载均衡，依赖于loadbalancer。

- nacos-discovery

  ```xml
          <!--nacos discovery-->
          <dependency>
              <groupId>com.alibaba.cloud</groupId>
              <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
          </dependency>
  ```

### 路由配置

> Gateway在配置文件中配置路由相关信息，无需编码

```yml
spring:
  cloud:
    gateway:
      routes: 
        - id: item # 路由规则id，自定义，唯一
        uri: lb://item-service # 路由目标微服务，lb代表负载均衡
        predicates: # 路由断言，判断请求是否符合规则，符合则路由到目标
          - Path=/items/** # 以请求路径做判断，以/items开头则符合
        - id: xx
          uri: lb://xx-service
          predicates:
          - Path=/xx/**
		filters:
		  - AddRequestHeader=Hello,World
	 default-filters:
	     - AddRequestHeader=Hello,World
```

**路由属性**

> 路由对应的Java类为RouteDefinition,具有属性:

- id：路由唯一标示

- uri：路由目标地址

- predicates：路由断言，判断请求是否符合当前路由。

  - 格式为:`断言类型=值`

  - 一个路由的多条断言之间是`AND`的关系，必须同时满足才能匹配，一条断言的多个值是`OR`的关系,如`Path=/items/**,/search/**`，只要有一个满足则断言成功。

- filters：路由过滤器，对请求或响应做特殊处理。

  - 格式为:`过滤器类型=过滤器所需参数`,多个参数由`,`分隔。

------

**断言**

> 处理断言的类为**RoutePredicateFactory**(路由断言工厂)，Spring提供了其12种实现:

|          类型          |            **说明**            |                           **示例**                           |
| :--------------------: | :----------------------------: | :----------------------------------------------------------: |
|         After          |      是某个时间点后的请求      | ` -  After=2037-01-20T17:42:47.789-07:00[America/Denver]  `  |
|         Before         |     是某个时间点之前的请求     | ` -  Before=2031-04-13T15:14:47.433+08:00[Asia/Shanghai]  `  |
|        Between         |    是某两个时间点之前的请求    | `-  Between=2037-01-20T17:42:47.789-07:00[America/Denver],  2037-01-21T17:42:47.789-07:00[America/Denver]  ` |
|         Cookie         |     请求必须包含某些cookie     |                `  - Cookie=chocolate, ch.p  `                |
|         Header         |     请求必须包含某些header     |               `  - Header=X-Request-Id, \d+  `               |
|          Host          | 请求必须是访问某个host（域名） |        `-  Host=**.somehost.org,**.anotherhost.org  `        |
|         Method         |     请求方式必须是指定方式     |                    `- Method=GET,POST  `                     |
|          Path          |    请求路径必须符合指定规则    |              `- Path=/red/{segment},/blue/**  `              |
|         Query          |    请求参数必须包含指定参数    |           ` - Query=name, Jack或者-  Query=name  `           |
|       RemoteAddr       |    请求者的ip必须是指定范围    |               ` - RemoteAddr=192.168.1.1/24  `               |
|         Weight         |            权重处理            |                   `  - Weight=group1, 2  `                   |
| XForwarded Remote Addr |     基于请求的来源IP做判断     |          `- XForwardedRemoteAddr=192.168.1.1/24  `           |

**过滤器**

> 过滤器对应的接口为`GatewayFilter`,与`GlobalFilter`Gateway提供了33种固定的路由过滤器实现,可以通过配置文件直接配置这些过滤器,同时Gateway还支持编码自定义过滤器。

- 路由过滤器

  > 作用于任意指定的路由；默认不生效，在配置文件进行配置后生效。路由过滤器由`GatewayFilterFactory`读取配置文件创建。Gateway实现了许多路由过滤器工厂。

  |       **名称**       |           **说明**           |                    **示例**                     |
  | :------------------: | :--------------------------: | :---------------------------------------------: |
  |   AddRequestHeader   |   给当前请求添加一个请求头   |     AddrequestHeader=headerName,headerValue     |
  | RemoveRequestHeader  |    移除请求中的一个请求头    |         RemoveRequestHeader=headerName          |
  |  AddResponseHeader   |  给响应结果中添加一个响应头  |    AddResponseHeader=headerName,headerValue     |
  | RemoveResponseHeader | 从响应结果中移除有一个响应头 |         RemoveResponseHeader=headerName         |
  |     RewritePath      |         请求路径重写         | RewritePath=/red/?(?<segment>.*),  /$\{segment} |
  |     StripPrefix      |   去除请求路径中的N段前缀    |     StripPrefix=1，则路径/a/b转发时只保留/b     |

  - 路由过滤器有两种配置方法:
    - 配置在`spring.cloud.gateway.default-filters`键，这里的配置全局有效
    - 配置在`spring.cloud.gateway.routes.filters`键，这里的配置只对当前路由生效

- 自定义过滤器

  - 自定义全局过滤器

    > 自定义过滤器需要实现两个接口:`GlobalFilter`与`Ordered`

    ```java
    @Component
    public class MyGlobalFilter implements GlobalFilter, Ordered {
        @Override
        //exchage是上下文对象，存储了请求响应的相关信息，chain为过滤器链，需要通过他将请求发送给下一个过滤器，同时需要将上下文对象传递下去。
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
            HttpHeaders headers = exchange.getRequest().getHeaders();
            return chain.filter(exchange);
        }
    
        @Override
        public int getOrder() {
            return 0;
        }
    }   
    ```

    - 需要将其注册为SpringBean，声明后自动生效，无需配置

  - 自定义路由过滤器

    > 路由过滤器均是由`AbstractGatewayFilterFactory`创建，继承此接口并实现其apply方法,apply方法返回一个`GatewayFilter`类型的过滤器。

    ```java
    @Component
    public class PrintAnyGatewayFilterFactory extends AbstractGatewayFilterFactory<Object> {
        @Override
        //可以通过config获得配置过滤器时的参数
        public GatewayFilter apply(Object config) {
            return new GatewayFilter() {
                @Override
                public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
                    System.out.println("PrintAny filter 执行了");
    				return chain.filter(exchange);
                }
            };
        }
    }
    //如果想要对路由过滤器的优先级进行配置，可以实现GatewayFilter与Order接口单独写一个类，或者使用OrderedGatewayFilter
    ```

    - 自定义路由过滤器的命名必须严格遵循`XXXAbstractGatewayFilterFactory`
    - 自定义路由过滤器与普通路由过滤器使用方法一样，在配置文件中通过`XXX=value`进行配置，如果过滤器无参数,则直接写`XXX`。

- NettyRoutingFilter

  > 由Gateway定义的用于将请求转发到路由指定微服务的过滤器，自动生效，位于过滤器链的最末端。
  
- 

### 登录校验

## **Netfilx** Zuul

> 由Netflix实现,基于Servlet的阻塞式编程,需要调优才能获得与SpringCloudGateway类似的性能。

# 服务保护

> 防止某个微服务出现故障，导致雪崩问题，从而使整个微服务集群停摆。

**常见方案:**

- **请求限流**

> 通过限流器限制访问微服务的请求的并发量，避免服务因流量激增出现故障。这也是流量整形的实现

- **线程隔离**

> 也叫做舱壁模式，模拟船舱隔板的防水原理。通过限定每个业务能使用的线程数量而将故障业务隔离，避免故障扩散

- **失败处理**

> 给业务编写一个调用失败时的处理的逻辑，称为fallback。当调用出现故障（比如无线程可用）时，按照失败处理逻辑执行业务并返回，而不是直接抛出异常。

- **服务熔断**

> 由**断路器**统计请求的异常比例或慢调用比例，如果超出阈值则会**熔断**该业务，则拦截该接口的请求。
>
> 熔断期间，所有请求快速失败，全都走fallback逻辑。

------

**SpringCloud中提供了两种用于服务保护的组件**

|              |                  **Sentinel**                  |         **Hystrix**          |
| ------------ | :--------------------------------------------: | :--------------------------: |
| **线程隔离** |                   信号量隔离                   |    线程池隔离/信号量隔离     |
| **熔断策略** |            基于慢调用比例或异常比例            |         基于异常比率         |
| **限流**     |             基于 QPS，支持流量整形             |          有限的支持          |
| **Fallback** |                      支持                      |             支持             |
| **控制台**   | 开箱即用，可配置规则、查看秒级监控、机器发现等 |            不完善            |
| **配置方式** |             基于控制台，重启后失效             | 基于注解或配置文件，永久生效 |

## Sentinel

> Sentinel是阿里巴巴开源的一款微服务流量控制组件。Sentinel提供了控制台(Dashboard),通过控制台无需编码即可实现推送规则、监控、管理机器信息等。

### 相关依赖

- Sentinel

  ```xml
  <!--sentinel-->
  <dependency>
      <groupId>com.alibaba.cloud</groupId> 
      <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
  </dependency>
  ```

### 快速使用

**安装Sentinel控制台**

1. 下载Sentinel的控制台jar包

   ```
   https://github.com/alibaba/Sentinel/releases
   ```

2. 运行jar包启动控制台

   ```
   java -Dserver.port=8090 -Dcsp.sentinel.dashboard.server=localhost:8090 -Dproject.name=sentinel-dashboard -jar sentinel-dashboard.jar
   ```

**引入Sentinel客户端依赖**

1. 引入依赖

   ```xml
   <!--sentinel-->
   <dependency>
       <groupId>com.alibaba.cloud</groupId> 
       <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
   </dependency>
   ```

2. 配置Sentinel控制台信息

   ```yml
   spring:
     cloud: 
       sentinel:
         transport:
           dashboard: localhost:8090
   ```

- 服务接口被调用后Sentinel控制台才能监控到服务

### 簇点链路

> 就是单机调用链路。是一次请求进入服务后经过的每一个被Sentinel监控的资源链。默认Sentinel会监控SpringMVC的每一个Endpoint（http接口）。限流、熔断等都是针对簇点链路中的**资源**设置的。而资源名默认就是接口的请求路径

------

**Restful风格的API很多时候请求路径相同，只是请求方式不同。因此需要修改配置，将请求方式+请求路径作为簇点资源名称**

```yml
spring:
  cloud:
    sentinel:
      transport:
        dashboard: localhost:8090
      http-method-specify: true # 开启请求方式前缀
```

- 配置FeignClient也作为Sentinel的簇点资源

  ```yml
  feign:
    sentinel:
      enabled: true
  ```

### 流量控制

> Sentinel可以根据`QPS`对每个簇点链路进行流量控制

### 线程隔离

> 线程隔离也属于流量控制，可以根据`并发线程数`对每个簇点链路进行线程隔离

### FallBack

> 当请求失败后，会调用FallBack逻辑，有两种配置方式:

**方式一：FallbackClass，无法对远程调用的异常做处理**

**方式二：FallbackFactory，可以对远程调用的异常做处理，通常都会选择这种**

- 想要对远程调用进行异常处理，首先需要将远程调用配置为簇点资源。

```java
public class UserClientFallbackFactory implements FallbackFactory<UserClient> {
@Override
    public UserClient create(Throwable throwable) {
        // 创建UserClient接口实现类，实现其中的方法，编写失败降级的处理逻辑
        return new UserClient() {
            @Override
            public User findById(Long id) {
                // 记录异常信息，可以返回空或抛出异常
                log.error("查询用户失败", throwable);
                return null;
            }
        };
    }
}

```

- 当调用FeignClient失败后，会再次调用`FallbackFactory`中重写的FeignClient
- 将FallbackFactory注册为Bean，然后在@FeignClient注解中进行配置。

```java
@Bean
public UserClientFallbackFactory userClientFallback(){
    return new UserClientFallbackFactory();
}
@FeignClient(value = "userservice", fallbackFactory = UserClientFallbackFactory.class)
public interface UserClient {   
    @GetMapping("/user/{id}") 
    User findById(@PathVariable("id") Long id);
}
```

### 服务熔断

> 熔断是解决雪崩问题的重要手段。思路是由**断路器**统计服务调用的异常比例、慢请求比例，如果超出阈值则会**熔断**该服务。即拦截访问该服务的一切请求；而当服务恢复时，断路器会放行访问该服务的请求。

![image-20250831115429384](D:\笔记\图片\image-20250831115429384.png)

- 断路器具有三种状态

# MQ

> MessageQueue,即消息队列。一种用于跨系统、跨服务传递消息的中间件，它以队列的形式暂存消息，实现不同系统之间的异步通信、削峰填谷和解耦。

**微服务同步调用**

> 调用方微服务发出请求后必须等待被调用方处理完并返回结果才能继续执行，同一时间业务中只有一个服务在工作，形成一个调用链。适合具有依赖性的业务。

**缺点:**

- 拓展性差,不符合开闭原则
- 性能下降
- 级联失败,可能引发雪崩问题

------

**微服务异步调用**

> 调用方微服务发出服务请求后**不会阻塞等待**结果返回，而是会继续执行自己的逻辑。异步调用有多种实现方式，常见使用消息通知模式实现。

**优点**

- 解除耦合，拓展性强
- 无需等待，性能好
- 故障隔离
- 缓存消息，流量削峰填谷

**缺点:**

- 不能立即得到调用结果，时效性差
- 不确定下游业务执行是否成功
- 业务安全依赖于Broker的可靠性

------

**消息通知模式**

> 一种典型的**异步通信模式**，通常由三个角色组成

- 消息发送者：投递消息的人，即微服务的调用者
- 消息接收者：接收和处理消息的人，即微服务的提供者
- 消息代理(Broker)：管理、暂存、转发消息，通常使用消息队列实现

**消息发送者**在完成自身业务后将消息发送给**消息代理**；**消息代理**负责存储、转发并可靠传递该消息；**消息接收者**从代理中获取消息并执行相应的处理逻辑

------

## 常见MQ

|            |        RabbitMQ         |            ActiveMQ            |  RocketMQ  |   Kafka    |
| :--------: | :---------------------: | :----------------------------: | :--------: | :--------: |
| 公司/社区  |         Rabbit          |             Apache             |    阿里    |   Apache   |
|  开发语言  |         Erlang          |              Java              |    Java    | Scala&Java |
|  协议支持  | AMQP，XMPP，SMTP，STOMP | OpenWire,STOMP，REST,XMPP,AMQP | 自定义协议 | 自定义协议 |
|   可用性   |           高            |              一般              |     高     |     高     |
| 单机吞吐量 |          一般           |               差               |     高     |   非常高   |
|  消息延迟  |         微秒级          |             毫秒级             |   毫秒级   |  毫秒以内  |
| 消息可靠性 |           高            |              一般              |     高     |    一般    |

## RabbitMQ

### 架构

![image-20250901162206753](D:\笔记\图片\image-20250901162206753.png)

**`virtual-host`**

> 虚拟主机，不同虚拟主机中的数据和组件是相互屏蔽的，起到数据隔离的作用

- 用户可以管理多个不同的虚拟主机
- 用户只可以操作具有权限的虚拟主机中的队列与交换机，对于其他虚拟主机，其队列与交换机是可见的，但是无法操作。

**`publisher`**

> 消息发送者

**`consumer`**

> 消息的消费者

**`queue`**

> 队列，存储消息

**`exchange`**

> 交换机，负责路由消息

- RabbitMQ中消息必须由交换机路由到队列中，不可以直接投递到队列中

- 交换机会立即路由消息，不会存储消息，即使没有队列与其绑定(此时消息会直接丢失)

### 安装部署

**通过Docker部署**

```bash
docker run \
 -e RABBITMQ_DEFAULT_USER=itheima \
 -e RABBITMQ_DEFAULT_PASS=123321 \
 -v mq-plugins:/plugins \
 --name mq \
 --hostname mq \
 -p 15672:15672 \
 -p 5672:5672 \
 --network hm-net\
 -d \
 rabbitmq:3.8-management
```

- 15672为控制台端口，5672为服务端口

## SpringAMQP

> 由Spring提供的基于AMQP协议定义的一套API规范，提供了模板操作和使用消息队列。基于SpringBoot提供了自动装配功能。

- SpringAMQP包含两部分，**spring-amqp** 提供了基于AMQP协议的基础抽象；**spring-rabbit** 是它的默认实现，基于 RabbitMQ。

**AMPQ**

> **A**dvanced **M**essage **Q**ueuing **P**rotocol，是用于在应用程序之间传递业务消息的开放标准。该协议与语言和平台无关。

### 相关依赖

**spring-amqp起步依赖**

```xml
<!--AMQP依赖，包含RabbitMQ-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

**jackson**

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```

### 使用

1. 配置MQ服务端信息

   ```yml
   spring:
     rabbitmq:
       host: 192.168.150.101 # 主机名
       port: 5672 # 端口
       virtual-host: /hmall # 虚拟主机
       username: hmall # 用户名
       password: 123 # 密码
   ```
   
2. 通过`RabbitTemplate`对象发送消息

   ```java
   @Autowired
   private RabbitTemplate rabbitTemplate;
   @Test
   public void testSimpleQueue() {
       // 队列名称
       String queueName = "simple.queue";
       // 消息
       String message = "hello, spring amqp!";
       // 发送消息
       rabbitTemplate.convertAndSend(queueName, message);
   }
   
   ```

   - RabbitTemplate在引入起步依赖后已经自动注入容器

3. SpringAMQP提供声明式的消息监听，只需要通过**注解**在方法上声明要监听的队列名称，SpringAMQP就会把消息传递给当前方法,并实现消息到Java对象的转换。

   ```java
   @Component
   public class SpringRabbitListener {
       @RabbitListener(queues = "simple.queue")
       public void listenSimpleQueueMessage(String msg) throws InterruptedException {
           log.info("spring 消费者接收到消息：【" + msg + "】");
       }
   }
   ```
   
------

**任务模型**

> Work Queues，消息队列的一种使用方式，即多个消费者绑定到一个队列，共同消费队列中的消息。

![image-20250901211401652](D:\笔记\图片\image-20250901211401652.png)

- 默认情况下，RabbitMQ的会将消息轮询投递给绑定在队列上的每一个消费者。这可以加快消息处理速度,但是并没有考虑到消费者是否已经处理完消息，可能出现消息堆积。通过设置消息预取数控制消费者可以堆积的消息数，实现能者多劳。

  ```yml
  spring:
    rabbitmq:
      listener:
        simple:
          prefetch: 1 # 每次只能获取一条消息，处理完成才能获取下一个消息
  ```


### 交换机

> 交换机接收发送者发送的消息，并将消息路由到与其绑定的队列。

- RabbitMQ中发送者只能将消息发送到交换机中，无法直接投递到队列中
- 所有队列都会绑定一个默认的交换机，队列的`routingkey`为其队列名

**根据路由方式不同，常见的交换机有三种:**

**Fanout:广播**

> 将接收到的消息路由到每一个与其绑定的队列中

**Direct:定向**

> 将接收到的消息根据规则路由到指定的队列，因此称为**定向**路由。

- 每一个队列在与交换机绑定时会设置一个`BingingKey`；发布者发送消息时，需要指定消息的`RoutingKey`，交换机将消息路由到`BindingKey`与消息`RoutingKey`一致的队列

**Topic:话题**

> `Topic`与`Direct`类似，但是提供了更灵活的路由规则。

- `routingkey`与`Bindingkey`均是由多个单词组合的字符串，单词之间以`.`分隔

- 此时`BindingKey`支持通配符:`#`：代指0个或多个任意单词,`*`：代指任意一个单词
- 交换机匹配与`BindingKey`规则一致的`routingkey`将其路由到对应的队列中。

### 声明队列/交换机

> RabbitMQ的控制台可以声明队列与交换机并进行绑定，但一般不推荐使用。更推荐使用SpringAMQP提供的API进行声明。

- 通过API声明队列/交换机时，如果队列或交换机不存在，则声明；如果存在，则跳过。

**SpringAMQP提供了两种方式声明队列/交换机:**

#### Bean声明

- Queue：用于声明队列，可以用工厂类QueueBuilder构建
- Exchange：用于声明交换机，可以用工厂类ExchangeBuilder构建
- Binding：用于声明队列和交换机的绑定关系，可以用工厂类BindingBuilder构建

```java
@Configuration
public class FanoutConfig {
    // 声明FanoutExchange交换机
    @Bean
    public FanoutExchange fanoutExchange(){
        return new FanoutExchange("hmall.fanout");
        //或
        //return ExchangeBuilder.fanoutExchange("hmall.fanout").build();
    }
    // 声明第1个队列
    @Bean
    public Queue fanoutQueue1(){
        return new Queue("fanout.queue1");
        //或
        //  return QueueBuilder.durable("fanout.queue2").build();
    }
    //绑定队列1和交换机,通过传参自动注入
    @Bean
    public Binding bindingQueue1(Queue fanoutQueue1, FanoutExchange fanoutExchange){
        return BindingBuilder.bind(fanoutQueue1).to(fanoutExchange);
    }
}
```

- durable表示持久队列，这种队列会被持久化到磁盘，防止消息丢失。

#### 注解声明

> `@RabbitListener`在声明一个消费者的同时，还可以在内部通过`bindings`属性声明队列，交换机以及绑定关系。

```java
@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "direct.queue1"),
    exchange = @Exchange(name = "itcast.direct", type = ExchangeTypes.DIRECT),
    key = {"red", "blue"}
))
public void listenDirectQueue1(String msg){
    System.out.println("消费者1接收到Direct消息：【"+msg+"】");
}
```

- 一般在消费者服务中声明队列与交换机，声明的同时会指定此方法为声明队列的消费者。

### 消息转换器

> Spring中将对象转换为消息对象或将消息对象转换为所需对象均是由org.springframework.amqp.support.converter.MessageConverter来处理的。而默认实现是SimpleMessageConverter，基于JDK的ObjectOutputStream完成序列化。

**默认的消息转换器存在下列问题:**

- JDK的序列化有安全风险

- JDK序列化的消息太大
- JDK序列化的消息可读性差

**推荐使用JSON序列化代替默认的JDK序列化**

- jackson依赖

  ```xml
  <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
  </dependency>
  ```

- 通过@Bean的方式注入`Jackson2JsonMessageConverter`

  ```java
  @Bean
  public MessageConverter messageConverter(){
  	return new Jackson2JsonMessageConverter();
  }
  
  ```


### 可靠性

> 增强MQ的可靠性可以从三个方面入手:①发送者②MQ③消费者

**发送者**

- 发送者重连

  > 发送者连接MQ时由于网络波动等不稳定因素失败，可以通过开启超时重连机制解决

  ```yml
  spring:
    rabbitmq:
      connection-timeout: 1s # 设置MQ的连接超时时间
      template:
        retry:
        enabled: true # 开启超时重试机制
        initial-interval: 1000ms # 失败后的初始等待时间
        multiplier: 1 # 失败后下次的等待时长倍数,initial-interval * multiplier
        max-attempts: 3 # 最大重试次数
  ```

  - 默认超时重连机制是关闭的

<h4>如何保证MQ消息顺序性</h4>

`topic` 或 `direct` 交换机通常会绑定多个队列。同一生产者向交换机发送的消息如果被路由到不同队列，由于跨队列消费，MQ 无法保证消息按照发送顺序被消费；实际上，只要存在跨队列消费，就无法保证顺序性。

在大多数业务场景中，并不要求消息全局有序，而只要求同一业务对象的消息保持有序，即局部有序。因此通常以业务 ID 作为路由依据，确保同一业务 ID 的消息进入同一个队列，并通过单线程或串行方式消费，从而保证顺序。

如果消费者采用线程池并行处理消息，可以使用多个阻塞队列按照ID接收消息，不同线程绑定一个阻塞队列，同一个ID的消息只会进入同一个阻塞队列，以保证同一ID的消息串行消费。

如果业务架构必须使用多队列并发消费，则无法在 MQ 层面保证强有序，只能在业务层通过状态机校验、幂等控制、重试或延迟消费等方式保证最终一致性。

<h4>如何保证消息可靠性</h4>

保证消息可靠性的核心是保证消息不丢失。

消息丢失的原因可能有以下几点：

- 生产者向MQ发送消息的过程中出现异常

  - 可以使用`ACK`机制，MQ收到消息后回复生产者一个`ACK`以确认接收到消息

- MQ本身服务出现异常

  - 开启MQ的持久化机制，将消息持久化到磁盘中。

  - 建立MQ集群，保证服务高可用。

- MQ将消息发送给消费者的过程中出现异常或消费者获得消息之后还未处理时出现异常

  - 使用`ACK`机制，消费者处理完消息后，向MQ返回一个`ACK`。

<h4>如何保证消息幂等性或者说如何避免消息重复消费</h4>

避免消息重复消费的核心是在消费者方对消息进行重复检查。

最通用的一个方案就是在消息体中携带一个唯一ID，消费者处理完消息前，将消息ID存储起来(如使用`Redis`的`SetNX`命令)，存储成功则说明是第一次消费。

对于新增操作，可以通过业务消息自带的唯一数据(如订单ID)，先查后插或者直接使用数据库唯一索引，避免消息重复导致新增重复数据。

对于更新操作，可以在消息体中携带版本号，消费者更新数据时对比版本号确认消息是否重复。


# 注册中心

## Eureka

> SPringCloud中的一个组件，是一种注册中心的实现

- eureka-server

  > 注册中心，初始化时，所有微服务均会在注册中心进行注册。作用是记录服务信息与心跳监控

- eureka-client

  > 注册中心的客户端。服务提供者与消费者均属于eureka-client，当消费者需要使用服务时，会到注册中心拉取服务信息，提供者会每30秒向注册中心发送一次心跳保证服务健康，可以正常提供服务，当某个微服务没有心跳时，会从注册列表中剔除。

![image-20250824120718398](D:\笔记\图片\image-20250824120718398.png)

### 搭建注册中心

> Eureka注册中心要单独实现成一个微服务项目

1. 引入依赖

   ```xml
   <dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId></dependency>
   
   ```

2. 编写启动类,添加@EnableEurekaServer注解

   ```
   @EnableEurekaServer
   @SpringBootApplication
   public class EurekaApplication {
       public static void main(String[] args) {
           SpringApplication.run(EurekaApplication.class,args);
       }
   }
   ```

3. 添加application.yml

   ```yml
   server:
     port: 10086 # 微服务端口
   spring:
     application:
       name: eurekeserver #微服务名字
   eureka:
     client:
       service-url:
         defaultZone: http://127.0.0.1:10086/eureka #配置eureka服务地址
   ```

> Eureka在启动时，会先将自己注册进服务

### 注册服务

1. 在被注册微服务中引入spring-cloud-starter-netflix-eureka-client依赖

   ```
   <dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   </dependency>
   
   ```

2. 配置微服务的相关信息

   ```yml
   spring:  
     application:
       name: userservice #微服务名字
   eureka:
     client:
       service-url:
         defaultZone:http://127.0.0.1:10086/eureka #eureka服务地址，要与注册中心地址一致
   ```

### 服务发现

> 即消费者拉取服务。

1. 此时访问服务不再需要服务的IP与端口，直接通过服务名访问服务

   ```
   String url = "http://userservice/user/1"
   ```

2. 通过RestTemplate类访问服务，如果想要实现负载均衡，可以在配置Bean时加入注解@LoadBalanced

   ```java
       @Bean
       @LoadBalanced
       public RestTemplate restTemplate() {
           return new RestTemplate();
       }
   ```

## Nacos

> 也是SpringCloud中用于实现注册中心的一个组件，由Alibaba开发。相比Eureka功能更加丰富

- 安装

  > 在官网下载安装包并解压

  ```
  nacos.io
  ```

- 启动

  > 相比Eureka通过专门创建一个微服务项目启动，Nacos通过命令启动。在nacos的`bin`目录下执行命令

  ```
  stratup.cmd -m standalone
  ```

  > `-m`表示启动模式，standalone表示单机模式
  >
  > 启动后需要登录，默认账号与密码均是nacos

### 服务注册

- 引入依赖

  1. 首先在父工程中引入spring-cloud-alibaba的版本管理依赖

     ```xml
     <dependency>    
     <groupId>com.alibaba.cloud</groupId>    
     <artifactId>spring-cloud-alibaba-dependencies</artifactId>   <version>2.2.6.RELEASE</version>    
     <type>pom</type>    
     <scope>import</scope>
     </dependency>
     ```

  2. 在客户端(需要被注册的微服务)项目中引入nacos客户端依赖

     ```xml
     <!-- nacos客户端依赖 -->
     <dependency>   
     <groupId>com.alibaba.cloud</groupId>    
     <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
     </dependency>
     
     ```

- 注册服务

  > 修改微服务的配置文件

  ```yml
  spring:  
  	cloud:    
  		nacos:
  			server-addr: localhost:8848 # nacos 服务端地址 
  ```

### 服务发现

> 同Eureka

### 健康检测

> 不同于Eureka，Nacos中的服务实例分为两种:非临时实例与临时实例，默认所有实例都是临时实例

- 临时实例

  > 临时实例会每隔一段时间向Nacos发送心跳来确保服务的健康，如果服务停止，会将其从注册列表中剔除

- 非临时实例

  > Nacos会主动请求询问非临时实例确保服务健康，如果服务停止，Nacos不会剔除非临时实例，而是会等待其恢复

```
spring:
	cloud:
		nacos:
			discovery:
				ephemeral: false #设置为非临时实例
```

- 主动推送

  > 对于消费者而言，除了定时拉取服务，如果服务出现变更，Nacos服务端会主动推送告知消费者。Eureka只有被动拉取的功能

> Nacos集群默认采用AP方式，当集群中存在非临时实例时，采用CP模式；Eureka采用AP方式

### 服务分级存储模型

> Nacos将服务分为3级，最顶层即每一个微服务，每个微服务可以有多个实例，在同一个机房内的多个微服务实例被称为一个集群。

![image-20250824165408096](D:\笔记\图片\image-20250824165408096.png)

> 调用服务时，应尽可能选择本地集群的服务，跨集群调用延迟较高，本地集群不可用时，再去访问其他集群

- 配置服务集群属性

  ```yml
  spring:  
  	cloud:    
  		nacos:      
  			server-addr: localhost:8848 # nacos 服务端地址      
  			discovery:        
  				cluster-name: HZ # 配置集群名称，也就是机房位置，例如：HZ，杭州
  ```

### 负载均衡

> Nacos默认也是使用Ribbon实现负载均衡

- 修改负载均衡策略

  > 默认Nacos采用轮询策略

  ```yml
  userservice:
    ribbon:
      NFLoadBalancerRuleClassName: com.alibaba.cloud.nacos.ribbon.NacosRule #优先同集群随机规则
  ```

- 权重设置

  > 可以配置实例的权重控制访问频率，权重为0时停止访问。

  配置权重无需编码，直接在Nacos的服务控制台调整

### 环境隔离

> Nacos中服务存储和数据存储的最外层都是一个名为namespace的东西，用来做最外层隔离.每个namespace都有唯一的ID。不同namespace下的服务不可见

![image-20250824171834323](D:\笔记\图片\image-20250824171834323.png)

- 修改命名空间

  > 默认情况下，所有服务都在一个名为public的默认空间内。可以在Nacos控制台新建命名空间，然后在服务配置文件中修改命名空间

  ```yml
  spring:
    application:
      name: orderservice
    cloud:
      nacos:
            discovery:
              server-addr: localhost:8848
              cluster-name: SD
              namespace: 900ac499-6a9b-48ed-9842-0025a8b9484c #命名空间ID
  ```

  

## Ribbon

> SpringCloud中的组件，实现了负载均衡的功能。Ribbon拦截了RestTemplate发送的请求，根据服务名称查询具体的服务地址，并根据策略进行负载均衡

### 负载均衡原理

![image-20250824134521088](D:\笔记\图片\image-20250824134521088.png)

![image-20250824153105306](D:\笔记\图片\image-20250824153105306.png)

> 当RestTemplate发送请求后，会被LoadBalancerInterceptor拦截，拦截器获取请求的主机名，将其交给RibbonLoadBalancerClient，然后RibbonLoadBalancerClient将主机名交给DynamicServerListLoadBalancer，这个类根据的IRule(负载均衡策略)选择某个服务，返回给RibbonLoadBanlancerClient，修改uri，然后发送请求

### 负载均衡策略

> Ribbon中通过IRule接口定义负载均衡的具体实现策略

![image-20250824154147848](D:\笔记\图片\image-20250824154147848.png)

|  **内置负载均衡规则类**   |                         **规则描述**                         |
| :-----------------------: | :----------------------------------------------------------: |
|      RoundRobinRule       | 简单轮询服务列表来选择服务器。它是Ribbon默认的负载均衡规则。 |
| AvailabilityFilteringRule | 对以下两种服务器进行忽略：   （1）在默认情况下，这台服务器如果3次连接失败，这台服务器就会被设置为“短路”状态。短路状态将持续30秒，如果再次连接失败，短路的持续时间就会几何级地增加。  （2）并发数过高的服务器。如果一个服务器的并发连接数过高，配置了AvailabilityFilteringRule规则的客户端也会将其忽略。并发连接数的上限，可以由客户端的<clientName>.<clientConfigNameSpace>.ActiveConnectionsLimit属性进行配置。 |
| WeightedResponseTimeRule  | 为每一个服务器赋予一个权重值。服务器响应时间越长，这个服务器的权重就越小。这个规则会随机选择服务器，这个权重值会影响服务器的选择。 |
|     ZoneAvoidanceRule     | 以区域可用的服务器为基础进行服务器的选择。使用Zone对服务器进行分类，这个Zone可以理解为一个机房、一个机架等。而后再对Zone内的多个服务做轮询。 |
|     BestAvailableRule     |       忽略那些短路的服务器，并选择并发数较低的服务器。       |
|        RandomRule         |                  随机选择一个可用的服务器。                  |
|         RetryRule         |                      重试机制的选择逻辑                      |

> 默认规则为`ZoneAvoidanceRule  `

- 修改负载均衡策略

  - 定义新的IRuleBean

    ```java
    @Bean
    public IRule randomRule(){
    	return new RandomRule();
    }
    ```

    > 配置全局有效

  - 通过配置文件修改负载均衡策略

    ```yml
    userservice: #提供者微服务名
    	ribbon:
    		NFLoadBalancerRuleClassName:com.netflix.loadbalancer.RandomRule #负载均衡规则
    ```

    > 只对对应的微服务有效

### 饥饿加载

> Ribbon默认采用懒加载，即第一次访问时才回去创建LoadBalanceClient，拉取服务列表，请求时间会很长。饥饿加载则会在项目启动时创建并拉取服务列表，降低第一次访问时长。

- 配置

  ```
  ribbon:
  	eager-load:
  		enabled: true #开启饥饿加载
  		clients: userservcie #根据服务名指定对哪个服务饥饿加载
  ```


# Nacos配置管理

> Nacos除了具有注册中心的功能，还是一个配置中心。

- 功能

  - 统一配置管理

    > 可以统一一个集群中的配置信息(如数据库信息等)

  - 配置热更新

    > 配置修改后，无需重启。

## 实现

> 启动一个配置管理服务，通过配置管理服务配合本地配置合并在一起，完成完整的配置。当配置更改时，配置管理服务会通知相关微服务完成配置的热更新

![image-20250824175822648](D:\笔记\图片\image-20250824175822648.png)

1. 在Nacos控制台新建配置

   - 配置ID通常为`服务名-profile.后缀名`,profile为环境,因为Nacos会根据配置文件中的这几项配置读取对应的文件
   - 配置内容通常是一些开关类的配置，或者说需要热更新的配置，不要将所有配置都写在Nacos配置中

2. 引入Nacos配置管理依赖

   ```xml
   <!--nacos配置管理客户端依赖-->
   <dependency>    
   <groupId>com.alibaba.cloud</groupId>    
   <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
   </dependency>
   ```

3. 在bootstrap.yml中配置nacos地址

   - bootstrap.yml是SpringBoot提供的引导文件，优先级比application.yml高，可以在读取application.yml之前读取

   ```yml
   spring:
     application:
       name: userservice #服务名称
     profiles:
       active: dev #激活的环境,dev指开发环境
     cloud:
       nacos:
         server-addr: localhost:8848 #Nacos地址
         config:
           file-extension: yaml #文件后缀名
   ```

### 配置热更新

> Nacos中的配置文件变更后，微服务无需重启就可以感知。

- 方式一:在@Value注入的变量所在类上添加注解@RefreshScope

  ```
  
  ```

- 方式二:使用@ConfigurationProperties注解

  ```
  @Component
  @ConfigurationProperties(prefix="pattern")
  public DatePattern {
  	private String dateformat;
  }
  ```

### 多环境配置共享

微服务启动时会从nacos读取多个配置文件：

- [spring.application.name]-[spring.profiles.active].yaml，例如：userservice-dev.yaml

- [spring.application.name].yaml，例如：userservice.yaml

无论profile如何变化，[spring.application.name].yaml这个文件一定会加载，因此多环境共享配置可以写入这个文件

多种配置的优先级：服务名-profile.yaml >服务名称.yaml > 本地配置

# Feign

> SpringCloud中用于发送Http请求的插件，是一个声明式的http客户端。Feign也是依赖于Ribbon实现了负载均衡

- 依赖坐标

  ```xml
   <dependency>
   	<groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
  ```

## 使用

1. 在启动类添加@EnableFeignClients注解开启Feign功能

   ```java
   @EnableFeignClients
   @SpringBootApplication
   public class OrderApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(OrderApplication.class, args);
       }
   }
   ```

2. 声明接口，这个接口声明了微服务的远程调用接口

   ```java
   @FeignClient("服务名")
   public interface UserClient {
   	@GetMapping("/user/{id}")
   	User findById(@PathVariable("id") Lond id)
   }
   ```

   > 接口信息依赖SpringMVC的注解来声明

3. 通过自动注入获得接口实现类，调用方法发送http请求

## 自定义配置

|          类型          |       作用       |                          说明                          |
| :--------------------: | :--------------: | :----------------------------------------------------: |
| **feign.Logger.Level** |   修改日志级别   |     包含四种不同的级别：NONE、BASIC、HEADERS、FULL     |
|  feign.codec.Decoder   | 响应结果的解析器 | http远程调用的结果做解析，例如解析json字符串为java对象 |
|  feign.codec.Encoder   |   请求参数编码   |          将请求参数编码，便于通过http请求发送          |
|    feign. Contract     |  支持的注解格式  |                 默认是SpringMVC的注解                  |
|     feign. Retryer     |   失败重试机制   | 请求失败的重试机制，默认是没有，不过会使用Ribbon的重试 |

- 方式一:配置文件配置

  ```yaml
  feign:
    client:   
      config:       
        default: # 这里用default就是全局配置，如果是写服务名称，则是针对某个微服务的配置           
          loggerLevel: FULL # 日志级别 
  ```

- 代码配置

  > 声明配置类，内部将配置声明为Bean

  ```java
  public class FeignClientConfiguration {
  @Bean   
  public Logger.Level feignLogLevel(){
  	return Logger.Level.BASIC;    
  	}
  }
  //如果是全局配置，则把它放到@EnableFeignClients这个注解中
  @EnableFeignClients(defaultConfiguration = FeignClientConfiguration.class) 
  //如果是局部配置，则把它放到@FeignClient这个注解中
  @FeignClient(value = "userservice", configuration = FeignClientConfiguration.class) 
  ```

## Feign底层Http客户端

> Feign底层默认使用URLConnection发送http请求

- URLConnection：默认实现，不支持连接池

- Apache HttpClient ：支持连接池

  ```
  <!--httpClient的依赖 -->
  <dependency>    
  <groupId>io.github.openfeign</groupId>   
  <artifactId>feign-httpclient</artifactId>
  </dependency>
  ```

  ```yaml
  feign: 
    client:
      config:
        default: # default全局的配置        
          loggerLevel: BASIC # 日志级别，BASIC就是基本的请求和响应信息 
    httpclient:
      enabled: true # 开启feign对HttpClient的支持
      max-connections: 200 # 最大的连接数
      max-connections-per-route: 50 # 每个路径的最大连接数
  ```

- OKHttp：支持连接池

# Gateway

> SpringCloud中作为网关的插件。在SpringCloud中，网关是一个独立的微服务

- 功能
  - 身份认证和权限校验
  - 服务路由、负载均衡
  - 请求限流

## 使用

1. 1.引入SpringCloudGateway的依赖和nacos的服务发现依赖

   ```xml
   <!--        Nacos服务发现-->
           <dependency>
               <groupId>com.alibaba.cloud</groupId>
               <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
   
           </dependency>
           <!--            gateway网关-->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-gateway</artifactId>
           </dependency>
   ```

2. 编写路由配置以及Nacos地址

   ```yml
   server:
     port: 10010 # 网关端口
   spring:
     application:
       name: gateway # 服务名称
     cloud:
       nacos:
         server-addr: localhost:8848 # nacos地址
       gateway:
         routes: # 网关路由配置
           - id: user-service # 路由id，自定义，只要唯一即可
             # uri: http://127.0.0.1:8081 # 路由的目标地址 http就是固定地址
             uri: lb://userservice # 路由的目标地址 lb就是负载均衡，后面跟服务名称
             predicates: # 路由断言，也就是判断请求是否符合路由规则的条件
               - Path=/user/** # 这个是按照路径匹配，只要以/user/开头就符合要求
   ```

## 路由断言工厂

> 在配置文件中，会配置路由断言规则，这些配置会被路由断言工厂(Route Predicate Factory)读取并处理，转变为路由的判断条件

Spring中提供了11种Predicate工厂

|  **名称**  |            **说明**            |                           **示例**                           |
| :--------: | :----------------------------: | :----------------------------------------------------------: |
|   After    |      是某个时间点后的请求      |    -  After=2037-01-20T17:42:47.789-07:00[America/Denver]    |
|   Before   |     是某个时间点之前的请求     |    -  Before=2031-04-13T15:14:47.433+08:00[Asia/Shanghai]    |
|  Between   |    是某两个时间点之前的请求    | -  Between=2037-01-20T17:42:47.789-07:00[America/Denver],  2037-01-21T17:42:47.789-07:00[America/Denver] |
|   Cookie   |     请求必须包含某些cookie     |                   - Cookie=chocolate, ch.p                   |
|   Header   |     请求必须包含某些header     |                  - Header=X-Request-Id, \d+                  |
|    Host    | 请求必须是访问某个host（域名） |          -  Host=**.somehost.org,**.anotherhost.org          |
|   Method   |     请求方式必须是指定方式     |                      - Method=GET,POST                       |
|    Path    |    请求路径必须符合指定规则    |                - Path=/red/{segment},/blue/**                |
|   Query    |    请求参数必须包含指定参数    |             - Query=name, Jack或者-  Query=name              |
| RemoteAddr |    请求者的ip必须是指定范围    |                 - RemoteAddr=192.168.1.1/24                  |
|   Weight   |            权重处理            |                                                              |

## 过滤器

> 由网关提供的一种过滤器，称为GatewayFilter。对进入网关的请求和微服务返回的响应做处理

![image-20250825095833850](D:\笔记\图片\image-20250825095833850.png)

**方式一：**通过配置文件配置

Spring提供了许多种过滤器工厂，可以通过配置文件进行配置

| **名称**             | **说明**                     |
| -------------------- | ---------------------------- |
| AddRequestHeader     | 给当前请求添加一个请求头     |
| RemoveRequestHeader  | 移除请求中的一个请求头       |
| AddResponseHeader    | 给响应结果中添加一个响应头   |
| RemoveResponseHeader | 从响应结果中移除有一个响应头 |
| RequestRateLimiter   | 限制请求的流量               |

```yaml
spring:
  cloud:
  	gateway:
  	  routes: # 网关路由配置
        - id: user-service
  	      uri: lb://userservice
  		  predicates:
  			- Path=/user/**          
  		  filters: # 过滤器
  			- AddRequestHeader=Truth, Itcast is freaking awesome! 局部过滤器，只对对应路由生效
	  default-filters:
	    - AddRequestHeader=hello,World #默认过滤器，全局生效
```

**方式二:**通过GlobalFilter编码实现

> 全局过滤器，可以根据需要自己写逻辑的过滤器。GlobalFilter是一个接口

```java
public interface GlobalFilter {
/**    
 *  处理当前请求，有必要的话通过{@link GatewayFilterChain}将请求交给下一个过滤器处理
 *
 * @param exchange 请求上下文，里面可以获取Request、Response等信息
 * @param chain 用来把请求委托给下一个过滤器 
 * @return {@code Mono<Void>} 返回标示当前过滤器业务结束
 */   
 Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain);
 }
```

```java
@Order(-1) //数值越小优先级越高
@Component
public class AuthorizeFilter implements GlobalFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 1.获取请求参数
        MultiValueMap<String, String> params = exchange.getRequest().getQueryParams();        // 2.获取authorization参数
        String auth = params.getFirst("authorization");
        // 3.校验        
        if ("admin".equals(auth)) {            
            // 放行
            return chain.filter(exchange);
        }
        // 4.拦截
        // 4.1.禁止访问        
        exchange.getResponse().setStatusCode(HttpStatus.FORBIDDEN);        
        // 4.2.结束处理
        return exchange.getResponse().setComplete();    
    }
}

```

- 每一个过滤器都必须指定一个int类型的order值，order值越小，优先级越高，执行顺序越靠前。
- GlobalFilter通过实现Ordered接口，或者添加@Order注解来指定order值，由我们自己指定
- 路由过滤器和defaultFilter的order由Spring指定，默认是按照声明顺序从1递增。
- 当过滤器的order值一样时，会按照 defaultFilter > 路由过滤器 > GlobalFilter的顺序执行。

## 跨域

跨域：域名不一致就是跨域，主要包括：

- 域名不同： www.taobao.com 和 www.taobao.org 和 www.jd.com 和 miaosha.jd.com
- 域名相同，端口不同：localhost:8080和localhost8081

跨域问题：浏览器禁止请求的发起者与服务端发生跨域ajax请求，请求被浏览器拦截的问题

解决方案：CORS

```yaml
spring:
  cloud:
    gateway:
      globalcors: # 全局的跨域处理
        add-to-simple-url-handler-mapping: true # 解决 options 请求被拦截问题
        corsConfigurations:
          '[/**]':
            allowedOrigins:  # 允许哪些网站的跨域请求
              - "http://localhost:8090"
              - "http://www.leyou.com"
            allowedMethods:  # 允许的跨域 ajax 请求方式
              - "GET"
              - "POST"
              - "DELETE"
              - "PUT"
              - "OPTIONS"
            allowedHeaders: "*"   # 允许在请求中携带的头信息
            allowCredentials: true # 是否允许携带 cookie
            maxAge: 360000 # 这次跨域检测的有效期
```

# MQ

> MessageQueue，消息队列，字面意思存放消息的队列，也就是事件驱动模式下的Broker。市面上有许多MQ的实现

|            |        RabbitMQ         |            ActiveMQ            | RocketMQ   | Kafka      |
| :--------: | :---------------------: | :----------------------------: | ---------- | ---------- |
| 公司/社区  |         Rabbit          |             Apache             | 阿里       | Apache     |
|  开发语言  |         Erlang          |              Java              | Java       | Scala&Java |
|  协议支持  | AMQP，XMPP，SMTP，STOMP | OpenWire,STOMP，REST,XMPP,AMQP | 自定义协议 | 自定义协议 |
|   可用性   |           高            |              一般              | 高         | 高         |
| 单机吞吐量 |          一般           |               差               | 高         | 非常高     |
|  消息延迟  |         微秒级          |             毫秒级             | 毫秒级     | 毫秒以内   |
| 消息可靠性 |           高            |              一般              | 高         | 一般       |

## 微服务调用方式

> 微服务的调用方式可以分为同步调用与异步调用

- 同步通讯

  > 同步通讯是阻塞式的，发送后，需要等待对方回应才能进行下一步，过程中不能做任何事情

  - 时效性较强

  - 耦合度高

    > 每次加入新的需求，都要修改原来的代码

  - 性能下降

    > 调用者需要等待服务提供者响应，如果调用链过长则响应时间等于每次调用的时间之和

  - 资源浪费

    > 调用链中的每个服务在等待响应过程中，不能释放请求占用的资源，高并发场景下会极度浪费系统资源

  - 级联失败

    > 如果服务提供者出现问题，所有调用方都会跟着出问题，如同多米诺骨牌一样，迅速导致整个微服务群故障

- 异步通讯

  > 异步通讯是非阻塞式的，发送后，可以直接进行其他事务，等待接收方回应后再进行处理。常用实现就是事件驱动模式

  ****

  在服务之间加入一个Broker，当事件来临时，消费者服务通知Broker，Broker再通知提供者服务。提供者服务需要在Broker完成注册才能被通知

  ![image-20250825194459473](D:\笔记\图片\image-20250825194459473.png)

  - 服务解耦

  - 性能提升，吞吐量提高

  - 服务没有强依赖，不担心级联失败问题

  - 流量削峰

  - •依赖于Broker的可靠性、安全性、吞吐能力

    •架构复杂了，业务没有明显的流程线，不好追踪管理

## RabbitMQ

> 基于Erlang语言开发的开源消息通信中间件

- 官网

  ```
  https://www.rabbitmq.com/
  ```

![image-20250826100155013](D:\笔记\图片\image-20250826100155013.png)

- channel:操作MQ的工具
- exchange:路由消息到队列中
- queue:缓存消息
- virtual host:虚拟主机,是对queue、exchange等资源的逻辑分组

### 安装与部署

#### 单机部署

1. 通过docker拉取镜像

   ```
   docker pull rabbitmq
   ```

2. 启动容器

   ```bash
   docker run \
    -e RABBITMQ_DEFAULT_USER=itcast \
    -e RABBITMQ_DEFAULT_PASS=123321 \
    --name mq \
    --hostname mq1 \
    -p 15672:15672 \
    -p 5672:5672 \
    -d \
    rabbitmq:3-management
   ```

   - 15672端口为控制台端口
   - 5672为应用端口

### 消息模型

> 根据MQ的结构，可将其分为三类:

- 基本消息队列（BasicQueue）

  ![image-20250826101010630](D:\笔记\图片\image-20250826101010630.png)

- 工作消息队列（WorkQueue）

  ![image-20250826101015679](D:\笔记\图片\image-20250826101015679.png)

- 发布订阅（Publish、Subscribe),又根据交换机类型不同分为三种：

  - Fanout Exchange：广播

    ![image-20250826101025155](D:\笔记\图片\image-20250826101025155.png)

  - Direct Exchange：路由

    ![image-20250826101031536](D:\笔记\图片\image-20250826101031536.png)

  - Topic Exchange：主题

    ![image-20250826101038409](D:\笔记\图片\image-20250826101038409.png)

## SpringAMQP

> SpringAMQP是基于AMQP协议定义的一套API规范，提供了模板来发送和接收消息。包含两部分，其中spring-amqp是基础抽象，spring-rabbit是底层的默认实现。

- AMQP

  > Advanced Message Queuing Protocol，高级消息队列协议，是用于在应用程序之间传递业务消息的开放标准。该协议与语言和平台无关，更符合微服务中独
  > 立性的要求。

- 依赖坐标

  ```xml
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-amqp</artifactId>
          </dependency>
  ```

- 配置

  ```yml
  spring:
    rabbitmq:
      host: 172.31.85.64
      port: 5672
      virtual-host: /
      username: root
      password: 1234 
  ```

  > 这是RabbitMQ的连接信息

### RabbitTemplate

> SpringAMQP中发送消息的模板，通过RabbitMQ实现。引入依赖后，可以自动注入RabbitTemplate对象

```java
@Autowired
private RabbitTemplate rabbitTemplate;
rabbitTemplate.convertAndSend(queueName,message);
```

### 接收消息

> 每一个方法都是一个消费者

```java
@Component
public class SpringRabbitListener {
    @RabbitListener(queues = "simple.queue")
    public void listenSimpleQueue(String message) {
        System.out.println(message);
    }
}
```

- 消息预取

  > 当队列中突然涌入大量消息后，监听者会预先取出一部分消息暂存，之后慢慢的处理，而不是让他们在队列中堆积

  ```
  spring:
    rabbitmq:
      listener:
        simple:
          prefetch: 1 #配置消息预取上限
  ```

  > 当多个消费者能力不一时，消息预取太大会导致速度快的消费者资源浪费。

## Work Queue

> 工作队列,用于单个消费者处理消息的能力无法匹配消息到来的速度，导致消息在队列中堆积

![image-20250826101015679](D:\笔记\图片\image-20250826101015679.png)

## 发布订阅

> 发布订阅模式允许将同一消息发送给多个消费者。实现方式是加入了exchange（交换机）。

![image-20250826111335977](D:\笔记\图片\image-20250826111335977.png)

![image-20250826111507863](D:\笔记\图片\image-20250826111507863.png)

### Fanout Exchange

> Fanout Exchange 会将接收到的消息广播到每一个跟其绑定的queue

```java
@Configuration
public class FanoutConfig {
// 声明FanoutExchange交换机   
@Bean    
public FanoutExchange fanoutExchange(){
	return new FanoutExchange("itcast.fanout");
	}
// 声明第1个队列
@Bean
public Queue fanoutQueue1(){
	return new Queue("fanout.queue1");
	}
//绑定队列1和交换机   
@Bean    
public Binding bindingQueue1(Queue fanoutQueue1, FanoutExchange fanoutExchange){
	return BindingBuilder.bind(fanoutQueue1).to(fanoutExchange);
	}
}

```

### DirectExchange

> Direct Exchange 会将接收到的消息根据规则路由到指定的Queue，因此称为路由模式（routes）。

- 每一个Queue都与Exchange设置一个BindingKey
- 发布者发送消息时，指定消息的RoutingKey
- Exchange将消息路由到BindingKey与消息RoutingKey一致的队列'

```java
@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "direct.queue1"),
    exchange = @Exchange(name = "itcast.direct", type = ExchangeTypes.DIRECT),
    key = {"red", "blue"}))
public void listenDirectQueue1(String msg){
    System.out.println("消费者1接收到Direct消息：【"+msg+"】");
}
@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "direct.queue2"),
    exchange = @Exchange(name = "itcast.direct", type = ExchangeTypes.DIRECT),
    key = {"red", "yellow"}))
public void listenDirectQueue2(String msg){
    System.out.println("消费者2接收到Direct消息：【"+msg+"】 ");
}

```

### TopicExchange

> TopicExchange与DirectExchange类似，区别在于routingKey必须是多个单词的列表，并且以 **.** 分割。
>
> Queue与Exchange指定BindingKey时可以使用通配符：
>
> \#：代指0个或多个单词
>
> *：代指一个单词

### 消息转换器

> AMQP中的消息发送与接收时的消息对象需要进行序列化和反序列化，是由org.springframework.amqp.support.converter.MessageConverter来处理的。而默认实现是SimpleMessageConverter，基于JDK的ObjectOutputStream完成序列化。但是ObjectOutputStream可读性差，一般需要需要定义一个MessageConverter 类型的Bean来覆盖默认消息转换器。推荐用JSON方式序列化

```xml
<dependency>
<groupId>com.fasterxml.jackson.core</groupId>
<artifactId>jackson-databind</artifactId>
</dependency>
```

```java
@Bean
public MessageConverter jsonMessageConverter(){
return new Jackson2JsonMessageConverter(); 
}
```

# ES

> elasticsearch，一款非常强大的开源搜索引擎，可以帮助我们从海量数据中快速找到需要的内容。ES基于Lucene开发，支持分布式，可水平扩展，提供Restful接口，可被任何语言调用

- Lucene

  > Lucene是一个Java语言的搜索引擎类库，是Apache公司的顶级项目，具有易扩展，高性能（基于倒排索引）的优势。但是只限于Java语言开发，学习曲线陡峭且不支持水平扩展。倒排索引是Lucene的核心

- 文档(document):ES面向文档存储，将每一条数据称为一个文档(document)，ES中文档会被序列化为json格式后存储。

- 索引（index）：相同类型的文档的集合，又称为索引库，存放数据的地方

- 映射（mapping）：索引中文档的字段约束信息，类似表的结构约束

## ES与Mysql对比

| MySQL  | Elasticsearch |                           **说明**                           |
| :----: | :-----------: | :----------------------------------------------------------: |
| Table  |     Index     |      索引(index)，就是文档的集合，类似数据库的表(table)      |
|  Row   |   Document    | 文档（Document），就是一条条的数据，类似数据库中的行（Row），文档都是JSON格式 |
| Column |     Field     | 字段（Field），就是JSON文档中的字段，类似数据库中的列（Column） |
| Schema |    Mapping    | Mapping（映射）是索引中文档的约束，例如字段类型约束。类似数据库的表结构（Schema） |
|  SQL   |      DSL      | DSL是elasticsearch提供的JSON风格的请求语句，用来操作elasticsearch，实现CRUD |

- 擅长事务类型操作，可以确保数据的安全和一致性

- 擅长海量数据的搜索、分析、计算

- ES与MYSQL在业务中是功能互补的，而不是相互替代的

  ![image-20250826154626560](D:\笔记\图片\image-20250826154626560.png)

## 倒排索引

> 传统数据库(如Mysql)采用正向索引，而ES采用倒排索引，大大提升了搜索速度。倒排索引更适合文字等复杂搜索场景。

- 文档的内容按照语义分隔成多个词语，每个词语称为词条(term)。

![image-20250826152942249](D:\笔记\图片\image-20250826152942249.png)

- 正向索引基于文档id创建索引。查询词条时必须先找到文档，而后判断是否包含词条
- 倒排索引对文档字段内容分词，对词条创建索引，并记录具有该词条的文档id以及词条在文档中的位置。查询时先根据词条查询到文档id，而后获取到文档。
- 可以说正向索引是在文档中匹配词条，而倒排索引是通过词条寻找文档。

## 安装

#### 单机部署

> 部署es的同时，还需要部署kibana。因为kibana提供了一个elasticsearch的可视化界面，可以轻松的发送DSL请求

1. 创建docker网络，帮助es和kibana容器互联

   ```
   docker network create es-net
   ```

2. 拉取es镜像

   ```
   docker pull elasticsearch:7.12.1
   ```

3. 部署es容器

   ```bash
   docker run -d \
   	--name es \
       -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
       -e "discovery.type=single-node" \
       -v es-data:/usr/share/elasticsearch/data \
       -v es-plugins:/usr/share/elasticsearch/plugins \
       --privileged \
       --network es-net \
       -p 9200:9200 \
       -p 9300:9300 \
   elasticsearch:7.12.1
   ```

   - 9200端口是控制台端口，9300为应用端口

4. 拉取kibana镜像并部署容器

   > kibana版本要与es版本保持一致

   ```
   dcoker pull kibana:7.12.1
   
   docker run -d \
   --name kibana \
   -e ELASTICSEARCH_HOSTS=http://es:9200 \
   --network=es-net \
   -p 5601:5601  \
   kibana:7.12.1
   ```

   - 5601为kibana的控制台端口

## 分词器(analyzer)

> ES在创建倒排索引时，需要先对文档内容进行分词生成词条;在搜索时,也需要对用户输入内容分词以在倒排索引中查询。但ES默认的分词规则对中文处理不好，无法理解中文词语，只会逐字分词。

### IK分词器

> IK分词器是专门用于ES分词的插件，增加了对中文的分词处理

#### 安装

1. GitHub仓库下载压缩包

   ```
   https://github.com/infinilabs/analysis-ik
   ```

2. 将压缩包解压后放进es容器的`/usr/share/elasticsearch/plugins`目录中，然后重启容器

### 分词模式

> IK分词器包含两种分词模式:

- `ik_smart`:最少切分
- `ik_max_word`:最细切分,会在最少切分的基础上，再次尝试分词是否还可以分词

### 字典

> 分词器底层依赖一个记录了所有词条的字典，因为是固定的，所以在时效性和一些专有名词上可能不尽人意。可以人为的对字典进行修改。

- 扩展

  > ik分词器目录下的`config`目录下的`IkAnalyzer.cfg.xml`文件，可以增加字典文件

  ```xml
  <!--用户可以在这里配置自己的扩展字典 -->
  <entry key="ext_dict">my.dic</entry>
  <!--用户可以在这里配置自己的扩展停止词字典-->
  <entry key="ext_stopwords">stopword.dic</entry>
  ```

## Mapping

> Mapping 是索引库的属性，不仅定义了索引库中字段的规则与约束，还定义了索引全局规则。

**字段约束**

- type:字段数据类型，常见的简单类型有：
  - 字符串：text（可分词的文本）、keyword（精确值，例如：品牌、国家、ip地址）
  - 数值：long、integer、short、byte、double、float、
  - 布尔：boolean
  - 日期：date
  - 对象：object
  - ES中没有数组，但是允许一个字段具有多个值
- index:是否创建倒排索引,默认为true
- analyzer:使用哪种分词器
- properties:该字段的子字段,字段嵌套时使用

```
{
  "mappings": {
    "properties": {
      "字段名":{
        "type": "text",
        "analyzer": "ik_smart"
      },
      "字段名2":{
        "type": "keyword",
        "index": "false"
      },
      "字段名3":{
        "properties": {
          "子字段": {
            "type": "keyword"
          }
        }
      },
      // ...略
    }
  }
} 
```

- 在mappings属性下的properties字段定义字段约束.

## DSL

> DSL(Domain Specific Language)是elasticsearch提供的JSON风格的请求语句，用来操作elasticsearch

### 索引库相关

- 创建索引库

  ```json
  PUT /索引库名称
  {
    "mappings": {
      "properties": {
        "字段名":{
          "type": "text",
          "analyzer": "ik_smart"
        },
        "字段名2":{
          "type": "keyword",
          "index": "false"
        },
        "字段名3":{
          "properties": {
            "子字段": {
              "type": "keyword"
            }
          }
        },
        // ...略
      }
    }
  } 
  ```

  - 索引库名称必须全部小写

- 查看索引库

  ```
  GET /索引库名
  ```

- 删除索引库

  ```
  DELETE /索引库名
  ```

- 增加索引库字段

  ```
  PUT /索引库/_mapping
  {
  	"properties":{
  		"新字段名":{
  			"type":"integer"
  		}
  	}
  }
  ```
  
  - 索引库中的字段是禁止修改的,但是可以增加新的字段
  
- 查看索引库中文档数量

  ```
  GET /索引库/_count
  ```

### 文档相关

- 插入文档

  ```json
  POST /索引库名/_doc/文档id
  {
      "字段1": "值1",
      "字段2": "值2",
      "字段3": {
          "子属性1": "值3",
          "子属性2": "值4"
      },
      // ...
  }
  ```

- 查询文档

  ```
  GET /索引库名/_doc/文档id 
  ```

- 删除文档

  ```
  DELETE /索引库名/_doc/文档id 
  ```

- 修改文档

  **方式一:**全量修改，删除旧文档，添加新文档

  ```
  PUT /索引库名/_doc/文档id
  {
      "字段1": "值1",
      "字段2": "值2",
      // ... 略
  }
  ```

  - 如果文档ID不存在，则不会删除文档，且会根据ID新增文档

  **方式二:**增量修改，修改指定字段值

  ```
  POST /索引库名/_update/文档id
  {
      "doc": {
           "字段名": "新的值",
      }
  }
  ```

**批量处理**

> 通过一次请求完成多次文档操作

```
POST /_bulk
{ "index" : { "_index" : "索引库名", "_id" : "1" } }
{ "字段1" : "值1", "字段2" : "值2" }  //新增操作
{ "index" : { "_index" : "索引库名", "_id" : "1" } }
{ "字段1" : "值1", "字段2" : "值2" }
{ "index" : { "_index" : "索引库名", "_id" : "1" } }
{ "字段1" : "值1", "字段2" : "值2" }
{ "delete" : { "_index" : "test", "_id" : "2" } }  //删除操作
{ "update" : {"_id" : "1", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} } //更新操作
```

### 查询

> DSL可以以JSON格式定义查询条件,同时还可以在查询后对结果进行排序，分页，高亮，聚合等处理。DSL查询可以分为两大类:①叶子查询(Leaf query clauses)②复合查询（Compound query clauses）

对于不同类型的字段，ES会使用不同的查询方式:对于具有倒排索引的字段，使用倒排索引查询；对于无倒排索引的字段，使用正向索引

**查询结果**

> 查询结果也是一个JSON格式的字符串

```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 10000,
      "relation" : "gte"
    },
    "max_score" : 1.0,
    "hits":[
        {数据一},
        {数据二}
    ]
  }
}
```

|      字段      |      含义      |      |
| :------------: | :------------: | ---- |
|      took      |    查询耗时    |      |
|    time_out    |    是否超时    |      |
|   hits.total   |  查询条数相关  |      |
| hits.max_score |     相关度     |      |
|   hits.hits    | 真正的查询结果 |      |

------

- 查询语句

```json
GET /indexName/_search
{
  "query": {
    "查询类型": {
      "查询条件": "条件值"
    }
  },
  "from": 0, // 分页开始的位置，即从第几条开始查(索引从0开始)
  "size": 10, //  查询的条数
  "sort": [
  	{
  		"FIELD":"desc", // 简写，排序方式有ASC与DESC
  	},
  	{
  		"FIELD":{
  			"order":"desc" //完整写法
  		}
  	}
  ],
    "highlight": {
    "fields": { // 指定要高亮的字段
      "FIELD": {
        "pre_tags": "<em>",  // 高亮的前置标签
        "post_tags": "</em>" // 高亮的后置标签
      }
    }
  },
    "aggs": { // 定义聚合
    "cateAgg": { //给聚合起个名字
      "terms": { // 聚合的类型，按照品牌值聚合，所以选择term
        "field": "category", // 参与聚合的字段
        "size": 20 // 希望获取的聚合结果数量
      }
    }
    }   
}
```

- 如果不写查询条件则默认查询所有数据

| 查询类型  |         说明         |              |
| :-------: | :------------------: | :----------: |
| match_all | 查询索引库的所有数据 | 查询条件为空 |
|           |                      |              |
|           |                      |              |

**叶子查询**

> 一般是在特定的字段里查询特定值，属于简单查询，很少单独使用。

- **全文检索（full text）**查询：利用分词器对用户输入内容分词，然后去词条列表中匹配。例如：

  |             |                                                             |                                                              |
  | :---------: | :---------------------------------------------------------: | :----------------------------------------------------------: |
  |    match    | 对输入的TEXT进行分词，然后根据FIELD字段倒排索引检索词条查询 |                  "match": {"FIELD": "TEXT"}                  |
  | multi_match |                      同时查询多个字段                       | "multi_match": {  "query": "TEXT","fields": ["FIELD1", " FIELD12"] } |
  |             |                                                             |                                                              |

- **精确查询**：不对用户输入内容分词，直接精确匹配，一般是查找keyword、数值、日期、布尔等类型。例如：

  | 查询类型 |                                       |                                                       |
  | :------: | ------------------------------------- | :---------------------------------------------------: |
  |   ids    | 根据id精确匹配文档，可以给定多个id    | "query": { "ids": {"values": ["3995645","663300"]} }  |
  |  range   | 查询字段值在gte~lte之间的文档         | "range": { "FIELD": {    "gte": 10,  "lte": 20   }  } |
  |   term   | 根据VALUE不分词直接查询带有VALUE的doc |        "term": {"FIELD": {"value": "VALUE"} }         |

-   **地理（geo）查询**：用于搜索地理位置，搜索方式很多。例如

  |                  |      |      |
  | :--------------: | ---- | ---- |
  | geo_bounding_box |      |      |
  |   geo_distance   |      |      |
  |                  |      |      |

**复合查询**

> 以逻辑方式组合多个叶子查询或者更改叶子查询的行为方式,可分为两类:

- 基于逻辑运算组合叶子查询，实现组合条件

  > 称为布尔(bool)查询

  |          |                                  |      |
  | :------: | :------------------------------: | ---- |
  |   must   |   必须匹配每个子查询，类似“与”   |      |
  |  should  |    选择性匹配子查询，类似“或”    |      |
  | must_not | 必须不匹配，不参与算分，类似“非” |      |
  |  filter  |       必须匹配，不参与算分       |      |

  ```json
  {
      "query": {
      	"bool": {
        		"must": [
          				{"match": {"name": "手机"}}
        				],
          	"should": [
  					]
      }
  }
  
  ```

- 基于某种算法修改查询时的文档相关性算分，从而改变文档排名

  |                |      |      |
  | :------------: | ---- | ---- |
  | function_score |      |      |
  |    dis_max     |      |      |
  |                |      |      |

**排序**

> 默认情况下根据相关度(_score)来排序，也可以指定字段排序，可以排序的字段类型有:`keyword`、数值类型、地理坐标类型、日期类型等

**分页**

> 默认情况下只返回查询结果的前十条数据，如果需要查询更多数据需要修改分页参数,通过`from`与`size`指定。

```json
{
  "query": {
    "match_all": {}
  },
  "from": 0, // 分页开始的位置，即从第几条开始查(索引从0开始)
  "size": 10, //  查询的条数
  "sort": [
    {"price": "asc"}
  ]
}

```

**高亮**

> 查询时在词条对应位置加入标签，以配合前端使词条高亮

```json
{
  "query": {
    "match": {
      "FIELD": "TEXT"
    }
  },
  "highlight": {
    "fields": { // 指定要高亮的字段
      "FIELD": {
        "pre_tags": "<em>",  // 高亮的前置标签
        "post_tags": "</em>" // 高亮的后置标签
      }
    }
  }
}
```

- 当标签为`<em></em>`时，可以省略`pre_tags`与`post_tags`属性，默认即为`<em></em>`

**聚合**

> 对文档数据的统计，分析，以及运算称为聚合，常见聚合有三类:

- 桶（Bucket）聚合：用来对文档做分组

  - TermAggregation：按照文档字段值分组
  - Date Histogram：按照日期阶梯分组，例如一周为一组，或者一月为一组
  
- 度量（Metric）聚合：用以计算一些值，比如：最大值、最小值、平均值等

  - Avg：求平均值
  - Max：求最大值
  
  - Min：求最小值
  - Stats：同时求max、min、avg、sum等
  
- 管道（pipeline）聚合：其它聚合的结果为基础做聚合

------

**参与聚合的字段只能是Keyword,数值，日期，布尔等类型的字段**

```json
"size": 0,  // 设置size为0，结果中不包含文档，只包含聚合结果
"aggs": { // 定义聚合
    "cateAgg": { //给聚合起个名字
      "terms": { // 聚合的类型，按照品牌值聚合，所以选择term
        "field": "category", // 参与聚合的字段
        "size": 20 // 希望获取的聚合结果数量
      }
    }
}
```

- 通过设置size为0则只返回聚合不返回具体数据

**doc结构**

```json
{
  "_index" : "items",
  "_type" : "_doc",
  "_id" : "317578",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "id" : "317578",
    "name" : "RIMOWA 21寸托运箱拉杆箱 SALSA AIR系列果绿色 820.70.36.4",
  }
}
```

|         |              |      |
| :-----: | :----------: | ---- |
| _index  |  所属索引库  |      |
|  _type  | 查询结果类型 |      |
|   _id   |              |      |
| _source |    源数据    |      |



## JavaRestClient

> 由ES官方提供的各种语言操作ES的客户端，本质就是组装DSL语句，通过http请求发送给ES。ES提供了两种客户端

### 相关依赖

- es-rest-high-level-client

  ```xml
  <dependency>
      <groupId>org.elasticsearch.client</groupId>
      <artifactId>elasticsearch-rest-high-level-client</artifactId>
  </dependency>
  ```

  - es8.x以前的Java客户端

### 快速使用

1. 创建客户端对象

   ```java
   RestHighLevelClient client = new RestHighLevelClient(RestClient.builder(
       HttpHost.create("http://192.168.150.101:9200")
   ));
   ```

2. 创建request对象，request对象封装了操作的逻辑

   ```
   // 1.创建Request对象
   CreateIndexRequest request = new CreateIndexRequest("items");
   // 2.设置请求参数，MAPPING_TEMPLATE是静态常量字符串，内容是JSON格式请求体
   request.source(MAPPING_TEMPLATE, XContentType.JSON);
   ```

   - `Request`支持链式编程

3. 通过客户端对象发送请求

   ```
   client.indices().create(request, RequestOptions.DEFAULT);
   ```

   - 需要用不同的方法发送不同的请求

### 索引库操作API

**获得索引库操作对象**

> 通过客户端的indices()方法可以获得调用所有索引库相关操作的的对象。

```
client.indices()
```

- 创建索引库

  ```java
  // 1.创建Request对象
  CreateIndexRequest request = new CreateIndexRequest("items");
  // 2.设置请求参数，MAPPING_TEMPLATE是静态常量字符串，内容是JSON格式请求体
  request.source(MAPPING_TEMPLATE, XContentType.JSON);
  // 3.发起请求
  client.indices().create(request, RequestOptions.DEFAULT);
  ```

- 删除索引库

  ```java
  // 1.创建Request对象 
  DeleteIndexRequest request = new DeleteIndexRequest("indexName");
  // 2.发起请求
  client.indices().delete(request, RequestOptions.DEFAULT);
  ```

- 查看索引库信息

  ```java
  // 1.创建Request对象
  GetIndexRequest request = new GetIndexRequest("indexName");
  // 2.发起请求 
  client.indices().get(request, RequestOptions.DEFAULT);
  ```

- 查看索引库是否存在

  ```java
  // 1.创建Request对象
  GetIndexRequest request = new GetIndexRequest("indexName");
  // 2.发起请求 
  client.indices().exists(request, RequestOptions.DEFAULT);
  ```

### 文档操作API

- 新增文档

  ```java
  // 1.创建request对象 
  IndexRequest request = new IndexRequest("indexName").id("1");
  // 2.准备JSON文档
  request.source("{\"name\": \"Jack\", \"age\": 21}", XContentType.JSON);
  // 3.发送请求
  client.index(request, RequestOptions.DEFAULT);
  ```

- 删除文档

  ```java
  // 1.创建request对象
  DeleteRequest request = new DeleteRequest("indexName", "1");
  // 2.删除文档 
  client.delete(request, RequestOptions.DEFAULT);
  ```

- 通过ID查询文档

  ```java
  // 1.创建request对象
  GetRequest request = new GetRequest("indexName", "1");
  // 2.发送请求，得到结果
  GetResponse response = client.get(request, RequestOptions.DEFAULT);
  // 3.解析结果 
  String json = response.getSourceAsString();
  ```

  - 一个完整的文档除了字段信息还包含其他内容，其中source属性保存了字段信息

- #### **修改文档**

  **全量更新**

  > 再次写入id一样的文档，就会删除旧文档，添加新文档。与新增的JavaAPI一致。

  **局部更新**

  > 只更新指定部分字段

  ```java
  // 1.创建request对象
  UpdateRequest request = new UpdateRequest("indexName", "1");
  // 2.准备参数，每2个参数为一对 key value
  request.doc(
  	"age", 18,
      "name", "Rose"
  );
  // 3.更新文档
  client.update(request, RequestOptions.DEFAULT);
  ```

### 批量处理API

```java
// 1.创建Bulk请求
BulkRequest request = new BulkRequest(); 
// 2.添加要批量提交的请求：这里添加了两个新增文档的请求
request.add(new IndexRequest("indexName")
            .id("101").source("json source", XContentType.JSON));
request.add(new IndexRequest(" indexName ")
            .id("102").source("json source2", XContentType.JSON));
// 3.发起bulk请求
client.bulk(request, RequestOptions.DEFAULT);
```

### 查询API

```java
// 1.准备Request
SearchRequest request = new SearchRequest("indexName");
// 2.组织DSL参数 
request.source()
	.query(QueryBuilders.matchAllQuery());
// 3.发送请求，得到响应结果
SearchResponse response = client.search(request, RequestOptions.DEFAULT);
// ...解析响应结果
```

- RestClient通过`QueryBuilder`类来构建查询条件，可以通过`QueryBuilders`工具类获得`QueryBulider`对象。
- 获得`request.source()`请求体源对象后，可以通过`query()`方法构建查询条件,`size()`与`from()`方法构造分页条件,通过`sort()`方法构造排序条件,通过`highlighter()`方法构造高亮条件

```java
request.source().from(0).size(5);
// 价格排序
request.source().sort("price", SortOrder.ASC);

request.source().highlighter(
    SearchSourceBuilder.highlight()
    .field("name")
    .preTags("<em>")
    .postTags("</em>")
);

```

# XXL-Job

> XXL-Job 是一款分布式任务调度平台，用于统一管理和执行定时任务。

## 架构

> `XXL-Job`分为两部分，一部分是调度中心，是一个独立的服务，用于调度任务执行以及日志管理。另一部分是执行器，负责执行任务。

<img src="D:\笔记\图片\image-20251119162542236.png" alt="image-20251119162542236" style="zoom:50%;" />

## 部署

> 调度中心是一个独立的服务，因此需要单独部署，可以使用docker容器化部署。

```
docker run \
-e PARAMS="--spring.datasource.url=jdbc:mysql://192.168.150.101:3306/xxl_job?Unicode=true&characterEncoding=UTF-8 \
--spring.datasource.username=root \
--spring.datasource.password=123" \
--restart=always \
-p 8880:8080 \
-v xxl-job-admin-applogs:/data/applogs \
--name xxl-job-admin \
-d \
xuxueli/xxl-job-admin:2.3.0
```

## 相关依赖

**`xxl-job`**

```xml
<dependency>
    <groupId>com.xuxueli</groupId>
    <artifactId>xxl-job-core</artifactId>
</dependency>
```

## 快速使用

1. 部署调度中心服务

2. 引入`xxl-job`依赖

3. 配置执行器

   ```java
   @Bean
   public XxlJobSpringExecutor xxlJobExecutor() {
       XxlJobSpringExecutor xxlJobSpringExecutor = new XxlJobSpringExecutor();
       xxlJobSpringExecutor.setAdminAddresses(adminAddresses);
       xxlJobSpringExecutor.setAppname(appname);
       xxlJobSpringExecutor.setIp(ip);
       xxlJobSpringExecutor.setPort(port);
       xxlJobSpringExecutor.setAccessToken(accessToken);
       xxlJobSpringExecutor.setLogPath(logPath);
       xxlJobSpringExecutor.setLogRetentionDays(logRetentionDays);
       return xxlJobSpringExecutor;
   }
   ```

   - 执行器在项目启动后会自动注册到调度中心

4. 编写任务

   ```java
   @Slf4j
   @Component
   public class CreateTableHandler {
       @XxlJob("createTableJob")
       public void createPointsBoardTableBySeason(){
           log.debug("开始执行创建历史榜单表的任务");
       }
   }
   
   ```

5. 在调度中心为执行器添加任务。

# 分布式事务

在分布式系统中，如果一个业务需要多个服务合作完成，而每一个服务都有自己的事务，多个事务必须同时成功或失败，这样的事务就是分布式事务。

**分支事务**

分布式事务中每个服务的事务称为分支事务

**全局事务**

所有分支事务组合在一起形成的事务称为全局事务

## `Seata`

一个开源的分布式事务解决方案。

### 原理

`Seata`的分布式事务由三个组件完成：

- **TC (Transaction Coordinator) -** **事务协调者：**维护全局和分支事务的状态，协调全局事务提交或回滚。
- **TM (Transaction Manager) -** **事务管理器：**定义全局事务的范围，开启全局事务、提交或回滚全局事务。
- **RM (Resource Manager) -** **资源管理器：**管理分支事务，与TC交互以注册分支事务和报告分支事务的状态

![image-20260129212548801](D:\笔记\图片\image-20260129212548801.png)

### 相关依赖

**Seata客户端**

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
</dependency>
```

- 添加到需要分布式事务的各个微服务中，为其提供`RM`,`TM`等组件；
- `TC`需要与各个微服务的`RM`,`TM`组件交互以维护全局和分支事务的状态，这要求`TC`可以被所有微服务访问，因此`TC`被设计为一个独立的服务，需要单独部署。


<h4><code>Seata</code>报错</h4>

JDK16+的模块化机制（Jigsaw Project）限制了对 JDK 内部类和方法的反射访问,导致反射失败，抛出异常。加入虚拟机选项`--add-opens java.base/java.lang=ALL-UNNAMED`强行打开模块边界

### 快速使用

<h4>1.部署TC服务</h4>

TC是一个基于`Spring Boot`开发的独立服务，需要单独部署。在启动时，Seata会被注册到注册中心。

1. 初始化数据库

   TC会维护全局和分支事务的状态，因此需要存储它们的相关信息。`Setata`支持多种存储模式，考虑数据安全和持久化的需要，一般选择基于数据库存储。

   <iframe height="50%" src="D:\笔记\resource\seata-tc.sql"/>

   - 除了存储数据，TC还可以通过数据库表实现锁以保证线程安全。

2. 准备配置文件,后续要将此目录挂载到docker容器中  **[Seata配置文件目录](D:\笔记\resource\seata)**

3. 创建容器

   ```bash
   docker run --name seata \
   -p 8099:8099 \
   -p 7099:7099 \
   -e SEATA_IP=172.31.85.64 \
   -v ./seata:/seata-server/resources \
   --privileged=true \
   --network hm-net \
   -d \
   seataio/seata-server:1.5.2
   ```

   - Seata依赖于mysql与nacos,必须将其连接在同一个docker网络中
   - 7099为控制台端口，8099为服务端口

<h4>2.在微服务中集成Seata客户端并配置</h4>

**客户端依赖**

```
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
</dependency>
```

**基础配置**

```yaml
  seata:
     registry: # 注册中心的配置，微服务根据这些信息去注册中心获取TC服务地址
       type: nacos # 注册中心类型 nacos
       nacos:
         server-addr: 192.168.150.101:8848
         namespace: "" #空表示public命名空间
         group: DEFAULT_GROUP      
         application: seata-server # seata服务名称
         username: nacos
         password: nacos
     tx-service-group: hmall # 事务组名称
     service:
       vgroup-mapping: # 事务组与tc集群的映射关系,集群即注册中心中的集群
         hmall: "default"
  	 data-source-proxy-mode: XA # 开启数据源代理的XA模式
```

- 全局事务的每一个参与者都需要进行`Seata`配置，推荐放在注册中心中共享配置。

`Seata`提供了解决分布式事务的不同模式，可以根据不同场景选择合适的模式。使用频率最高的两种模式为**XA模式** 和 **AT模式**

- 如果对数据一致性要求高，则选择`XA`模式，如果对性能要求高，则选择`AT`模式

<h3>XA模式</h3>

XA 规范 是 `X/Open` 组织定义的分布式事务处理（DTP，Distributed Transaction Processing）标准，描述了全局的TM与局部的RM之间的接口，几乎所有主流的关系型数据库都对 XA 规范 提供了支持。

**优点**

- 事务的强一致性，满足ACID原则。
- 常用数据库都支持，实现简单，并且没有代码侵入

**缺点**

- 一阶段所有分支事务均不提交，会持续锁定数据库资源，等待二阶段结束才释放，性能较差
- 依赖关系型数据库实现事务

<h4>Seata中 XA 模式原理</h4>

称为二阶段提交模式

![image-20250831165003954](D:\笔记\图片\image-20250831165003954.png)

<h4>XA模式使用</h4>

1. 在配置文件中开启XA模式

   ```yml
   seata:
     data-source-proxy-mode: XA # 开启数据源代理的XA模式
   ```
   
2. 在发起全局事务的入口方法添加@GlobalTransactional注解，并在各个分支事务方法中添加`@Transactional`注解开启事务。

   ```java
   @GlobalTransactional
   public Long createOrder(OrderFormDTO order) {
       // 创建订单 ... 略
       // 清理购物车 ...略 
       // 扣减库存 ...略
       return order.getId();
   }
   ```

<h3>AT模式</h3>

AT模式同样是分阶段提交的事务模型，但弥补了XA模型中资源锁定时间过长的缺陷。

**优点**

- 分支事务执行完毕直接提交，锁定资源时间短

**缺点**

- 分支事务执行完毕直接提交，如果最终全局事务回滚，会导致短暂的数据不一致，只保证最终一致性。

<h4>原理</h4>

![image-20250831170220392](D:\笔记\图片\image-20250831170220392.png)

<h4>AT模式使用</h4>

1. 创建数据库表用来保存`undo log`，需要在分支事务对应的每一个微服务中创建。	

   ```sql
   CREATE TABLE IF NOT EXISTS `undo_log`
   (
       `branch_id`     BIGINT       NOT NULL COMMENT '分支事务id',
       `xid`           VARCHAR(128) NOT NULL COMMENT '全局事务id',
       `context`       VARCHAR(128) NOT NULL COMMENT 'undo_log context,such as serialization',
       `rollback_info` LONGBLOB     NOT NULL COMMENT 'rollback info',
       `log_status`    INT(11)      NOT NULL COMMENT '0:normal status,1:defense status',
       `log_created`   DATETIME(6)  NOT NULL COMMENT 'create datetime',
       `log_modified`  DATETIME(6)  NOT NULL COMMENT 'modify datetime',
       UNIQUE KEY `ux_undo_log` (`xid`, `branch_id`)
   ) ENGINE = InnoDB
     AUTO_INCREMENT = 1
     DEFAULT CHARSET = utf8mb4 COMMENT ='AT transaction mode undo table';
   ```

2. 开启AT模式

   ```yml
   seata:
     data-source-proxy-mode: AT # 开启数据源代理的AT模式，如果不进行配置，默认为AT模式
   ```

3. 给发起全局事务的入口方法添加`@GlobalTransactional`注解，并在各个分支事务方法中添加`@Transactional`注解开启事务。

   ```java
   @GlobalTransactional
   public Long createOrder(OrderFormDTO order) {
       // 创建订单 ... 略
       // 清理购物车 ...略 
       // 扣减库存 ...略
       return order.getId();
   }
   ```


**XA与AT对比**

- XA模式一阶段不提交事务，锁定资源；AT模式一阶段直接提交，不锁定资源。
- XA模式依赖数据库机制实现回滚；AT模式利用数据快照实现数据回滚。
- XA模式强一致；AT模式最终一致

# gRPC

`gRPC`是由 Google 开源的一个 **高性能 RPC 框架**，允许 **客户端直接调用远程服务器上的方法**，就像调用本地方法一样。

<h3>特点</h3>

- **高性能：**基于`Http/2`，支持多路复用。使用 `Protobuf`作为默认序列化协议,这是一种高效的二进制序列化协议。
- **强类型接口定义：**接口定义通过`.proto`文件，文件编译后自动生成代码。客户端/服务端都遵循同一份接口，减少出错
- **跨语言支持：**官方支持 Java、Go、Python、C++、C#、Node.js 等语言。编写`.proto`文件后，可直接生成不同语言的服务端和客户端
- **支持四种通信方式：**单次请求-单次响应（Unary RPC）,服务器流式响应（Server streaming）,客户端流式请求（Client streaming）,双向流（Bidirectional streaming）。

## Protocol Buffers

简称为`Protobuf`，是一种高效的二进制序列化协议。目前`protobuf`有两个版本：`proto2`和`proto3`，主流应用的是`proto3`。

`Protobuf`提供了一套接口定义语言（IDL，Interface Definition Language)，用于定义数据结构和服务接口(`.proto`)，从而实现跨语言的接口契约与数据通信。

- 为了提高效率和减少冗余，`protobuf`采用非自描述的二进制编码格式，传输数据时不携带数据的完整结构信息。因此需要通过`IDL`定义接口和数据结构，并基于该定义生成代码，将结构元数据固化到程序中。


<h4><code>gRPC</code>版本管理依赖</h4>

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-bom</artifactId>
            <version>1.59.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

- 引入后，再引入`gRPC`相关依赖无需显示指定版本号

<h4><code>grpc-nettyshaded</code></h4>

```xml
   <dependency>
        <groupId>io.grpc</groupId>
        <artifactId>grpc-netty-shaded</artifactId>
        <version>1.79.0</version>
    </dependency>
```

- 网络传输实现（基于 Netty），负责 HTTP/2 通信。

<h4><code>grpc-protobuf</code></h4>

```xml
   <dependency>
        <groupId>io.grpc</groupId>
        <artifactId>grpc-protobuf</artifactId>
        <version>1.79.0</version>
    </dependency>
```

- 提供 Protobuf 的序列化和生成的消息类支持。

<h4><code>grpc-stub</code></h4>

```xml
    <dependency>
        <groupId>io.grpc</groupId>
        <artifactId>grpc-stub</artifactId>
        <version>1.79.0</version>
    </dependency>
```

- 提供客户端和服务端调用的 Stub 类，封装 RPC 调用逻辑。

<h4><code>protc</code></h4>

```xml
    <build>
        <extensions>
            <extension>
                <groupId>kr.motd.maven</groupId>
                <artifactId>os-maven-plugin</artifactId>
                <version>1.7.1</version>
            </extension>
        </extensions>
        <plugins>
            <plugin>
                <groupId>org.xolstice.maven.plugins</groupId>
                <artifactId>protobuf-maven-plugin</artifactId>
                <version>0.6.1</version>
                <configuration>
                    <protocArtifact>com.google.protobuf:protoc:3.25.8:exe:${os.detected.classifier}</protocArtifact>
                    <pluginId>grpc-java</pluginId>
                    <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.79.0:exe:${os.detected.classifier}</pluginArtifact>
                </configuration>
                <executions>
                    <execution>
                        <goals>
<!--                            用于生成消息-->
                            <goal>compile</goal>
<!--                            用于生成服务接口-->
                            <goal>compile-custom</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

<h4>protoc</h4>

`.proto`需要经过编译生成对应的代码,`protoc`为官方的编译器，是一个命令行工具，可在`github`上进行安装。

```bash
protoc [OPTION] PROTO_FILES

protoc --java_out=out_dir proto_file
```

也可使用`Maven`插件进行`proto`文件的编译。

```xml
<build>
  <extensions>
    <extension>
      <groupId>kr.motd.maven</groupId>
      <artifactId>os-maven-plugin</artifactId>
      <version>1.7.1</version>
    </extension>
  </extensions>
  <plugins>
    <plugin>
      <groupId>org.xolstice.maven.plugins</groupId>
      <artifactId>protobuf-maven-plugin</artifactId>
      <version>0.6.1</version>
      <configuration>
        <protocArtifact>com.google.protobuf:protoc:3.25.8:exe:${os.detected.classifier}</protocArtifact>
        <pluginId>grpc-java</pluginId>
        <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.79.0:exe:${os.detected.classifier}</pluginArtifact>
          <!-- 设置输出目录,默认为target目录-->
          <outputDirectory>${basedir}/src/main/java</outputDirectory>
          <!--                    不要每次生成前都清空输出目录-->
                    <clearOutputDirectory>false</clearOutputDirectory>
      </configuration>
      <executions>
        <execution>
          <goals>
            <goal>compile</goal>
            <goal>compile-custom</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

- 执行 `mvn compile` 时，自动把 `.proto` 文件生成为 Java 代码

- `.proto`文件存放在`src/main/proto`下
- 默认`protoc`只提供生成`message`的能力，`service`对应的代码是由插件生成的，每一个`service`都是一个单独的Java文件，类名为`XXXGrpc`。

<h3><code>.proto</code>语法</h3>

- 在`Maven`工程模板中，一般`.proto`文件统一放置在`src/main/proto`下。

```protobuf
syntax = "proto3"; //设置协议版本

//设置，不同目标编程语言设置不同
option java_multiple_files = false; ////是否一个消息对应一个Java文件

option java_package = "com.example"; //生成的类放在哪个包中


option java_outer_classname="UserService";  //外部类名字，protobuf会生成多个不同功能的类，这些类会封装在统一的外部类中

//逻辑包，非必须，不指定则为java_package
package xxx;

//导入其他.proto的文件
import "xxx/UserService.proto";

//消息定义
message HelloRequest {
	string msg = 1;
}
message HelloResponse {
	string response = 1;
}

//服务定义
service HelloService {
	rpc hello(HelloRequest) returns(HelloResponse){}
}
```

<h4>注释</h4>

- 使用 `//` 符号添加单行注释。
- 使用 `/* */` 符号添加多行注释。

<h4>标量类型</h4>

| Proto  Type |  C++ Type   | Java/Kotlin Type |
| :---------: | :---------: | :--------------: |
|   double    |   double    |      double      |
|    float    |    float    |      float       |
|    int32    |   int32_t   |       int        |
|    int64    |   int64_t   |       long       |
|   uint32    |  uint32_t   |       int        |
|   uint64    |  uint64_t   |       long       |
|   sint32    |   int32_t   |       int        |
|   sint64    |   int64_t   |       long       |
|   fixed32   |  uint32_t   |       int        |
|   fixed64   |  uint64_t   |       long       |
|  sfixed32   |   int32_t   |       int        |
|  sfixed64   |   int64_t   |       long       |
|    bool     |    bool     |     boolean      |
|   string    | std::string |      String      |
|    bytes    | std::string |    ByteString    |

<h4>枚举</h4>

```
enum 枚举名 {
	SPRING = 0;
	SUMMER = 1;
}
```

- 枚举值必须从0开始。

<h4>消息(<code>message</code>)</h4>

`message`定义了传输的数据结构，类似结构体。

```protobuf
//字段声明格式: [keyword...] type fieldName = num;
message LoginRequest {
	string username = 1; //值为字段在消息中的编号
	string password = 2;
	int32 age = 3;

}
```

- 编号从1开始，最大值为`2 ^ 29 - 1`，其中19000 - 19999不能使用，为`protobuf`保留为自己使用的编号。

- 一般请求消息命名为`XXXRequest`，响应消息命名为`XXXResponse`。

- 消息可以嵌套定义(类似内部类)，也可在消息内使用其他消息作为字段。

  ```protobuf
  message Response {
  	message Result {
  	} 
  	Result result = 1;
  }
  //在外部使用嵌套消息，与内部类使用类似
  message AAA {
  	Response.Result result = 1;
  }
  ```

**关键字**

- `singular`：默认关键字，标识该字段只能有一个值

- `repeated`：该字段为一个集合字段，可能存在多个值(等价于Java中的`List`)

- `oneof`：声明一个互斥字段组，一组字段中，同一时刻只能有一个字段被设置。

  ```protobuf
  message LoginRequest {
    oneof login_type {
      string phone = 1;
      string email = 2;
      string username = 3;
    }
  }
  ```

<h4>服务</h4>

```
service 服务名 {
	rpc 服务接口名(形参类型...) returns (返回值类型){}
}
```

服务通信方式




<h3>使用示例</h3>

1. 编写`.proto`文件定义服务接口和消息格式

   ```protobuf
   syntax = "proto3";
   
   package demo;
   
   service UserService {
       rpc GetUser(UserRequest) returns (UserResponse);
   }
   
   message UserRequest {
       int32 id = 1;
   }
   
   message UserResponse {
       int32 id = 1;
       string name = 2;
   }
   ```

2. 使用`protoc`生成客户端和服务端代码

   ```
   protoc --java_out=./gen --grpc-java_out=./gen user.proto
   ```

3. 在服务端实现定义的`RPC`方法

## `Grpc`类

`.proto` 文件中的每个 `service` 在编译后都会生成一个 `XXXGrpc` 类。该类是 **gRPC Java 框架在客户端和服务端交互中的核心入口类**。`XXXGrpc` 类内部封装了多个功能不同的静态内部类，用于承载 **RPC 方法描述、客户端 Stub（同步 / 异步 / Future）、以及服务端实现基类**

<h3><code>XXXImplBase</code></h3>

**服务端开发时需要继承的基类**，服务端业务层继承并重写其方法以实现 RPC 方法的具体业务逻辑。

```java
public class HelloServiceImpl extends HelloServiceGrpc.HelloServiceImplBase {
    @Override
    //为适应不同传输方式,gRPC将响应封装为StreamObserver
    public void hello(HelloProto.HelloRequest request, StreamObserver<HelloProto.HelloResponse> responseObserver) {
        String name = request.getName();
        System.out.println(name);
        HelloProto.HelloResponse response = HelloProto.HelloResponse.newBuilder().setResult("Hello,World").build();
        //将响应返回
        responseObserver.onNext(response);
        //通知客户端响应已结束
        responseObserver.onCompleted();
    }
}
```

<h3><code>XXXStub</code></h3>

`Stub`是`gRPC`自动生成的用于客户端与服务端交互的代理类,`gRPC`中提供了多种`Stub`：

- `Stub`：异步，通过监听处理
- `BlockingStub`：阻塞
- `FutureStub`：同步异步均支持，只能用于 **一元RPC**

<h4><code>FutureStub</code></h4>

只能用于`Unary RPC`通信方式，调用后返回一个`Future`对象，`ListenableFuture`支持使用同步和异步两种方式对响应结果进行处理。

**同步**

```
   ListenableFuture<HelloProto.HelloResponse> future = stub.hello(HelloProto.HelloRequest.newBuilder().setName("小刘").build());
        HelloProto.HelloResponse helloResponse = future.get(); //如果响应未完成，会阻塞
```

**异步**

```java
//直接对Future添加监听器        
ListenableFuture<HelloProto.HelloResponse> future = stub.hello(HelloProto.HelloRequest.newBuilder().setName("小刘").build());
        //注册监听器，监听器会在Future完成时被调用
        future.addListener(() -> {
            try {
                HelloProto.HelloResponse response = future.get();
            } catch (Exception ignore){}
        }, Executors.newSingleThreadExecutor());
//使用由gRPC提供的Futures工具类
Futures.addCallback(future, new FutureCallback<HelloProto.HelloResponse>() {
            @Override
            public void onSuccess(HelloProto.HelloResponse result) {             
            }
            @Override
            public void onFailure(Throwable t) {
            }
        },Executors.newSingleThreadExecutor());
    }
```

## `SpringBoot`整合`gRPC`

### 客户端

```
<dependency>
	<groupId>org.springframework.grpc</groupId>
	<artifactId>spring-grpc-spring-boot-starter</artifactId>
	<exclusions>
		<exclusion>
			<groupId>io.grpc</groupId>
			<artifactId>grpc-netty</artifactId>
		</exclusion>
	</exclusions>
</dependency>
<dependency>
	<groupId>io.grpc</groupId>
	<artifactId>grpc-netty-shaded</artifactId>
</dependency>
```

<h4>使用示例</h4>

1. 在配置文件中配置

   ```
   spring:
     application:
       name: grpc-service
     grpc:
       client:
         default-channel:
           address: localhost:9090
           negotiation-type: plaintext
   ```

2.  

### 服务端

<h4>相关依赖</h4>

```xml
<dependency>
	<groupId>org.springframework.grpc</groupId>
	<artifactId>spring-grpc-spring-boot-starter</artifactId>
	<exclusions>
		<exclusion>
			<groupId>io.grpc</groupId>
			<artifactId>grpc-netty</artifactId>
		</exclusion>
	</exclusions>
</dependency>
<dependency>
	<groupId>io.grpc</groupId>
	<artifactId>grpc-netty-shaded</artifactId>
</dependency>
```

<h4>使用示例</h4>

`SpringBoot`主要对`gRPC`中的服务注册与启动进行了整合

1. 编写`gRPCService`，在类上添加`@GrpcService`注解，声明这是一个`gRPC`服务类。

2. 在配置文件中对`gRPC`进行相关配置

   ```yaml
   spring:
     application:
       name: grpc-service
     grpc:
       server:
         address: localhost:8080
   ```

   

## 服务发布

实现`XXXImplBase`后，服务端业务就编写好了，接下来需要将服务发布，供他人使用。

```java
    public static void main(String[] args) throws IOException, InterruptedException {
        ServerBuilder serverBuilder = ServerBuilder.forPort(8080);
        Server server = serverBuilder.build();
        server.start();
        server.awaitTermination();
    }
```

## 客户端使用服务

`.proto`编译后得到的`HelloServiceImpl`中封装了不同通信方式的客户端代理类(`XXXStub`)。

```java
        ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost",8080).usePlaintext().build();
        HelloServiceGrpc.HelloServiceBlockingStub stub = HelloServiceGrpc.newBlockingStub(channel);
        HelloProto.HelloResponse response = stub.hello(HelloProto.HelloRequest.newBuilder().setName("小刘").build());
        System.out.println(response.getResult());
```

## gRPC的四种通信方式

<h3>一元RPC(<code>Unary RPC</code>)</h3>

请求响应的形式，单次请求，单次响应。

<h3>服务器流式RPC(<code>Server streaming RPC</code>)</h3>

客户端发送一次请求，服务端可以流式的返回多个响应。

**`.proto`服务声明语法**

```protobuf
rpc XXXService(XXXRequest) returns(stream XXXResponse)
```

**服务端编写**

```java
    @Override
    public void serverStream(HelloProto.HelloRequest request, StreamObserver<HelloProto.HelloResponse> responseObserver) {
        String name = request.getName();
        System.out.println(name);
        for (int i = 0; i < 10; i++) {
            HelloProto.HelloResponse response = HelloProto.HelloResponse.newBuilder().setResult("当前时间:" + LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy:MM:dd hh:mm:ss"))).build();
            responseObserver.onNext(response);
            try {
                Thread.sleep(1000);
            } catch (Exception ignore) {
                
            }
        }
        responseObserver.onCompleted();
    }
```

**客户端调用**

- 此时服务端的返回会被封装为一个迭代器

```java
//客户端同步处理响应，客户端会阻塞，直到服务端完全响应完毕        
ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost", 8080).usePlaintext().build();
        HelloServiceGrpc.HelloServiceBlockingStub stub = HelloServiceGrpc.newBlockingStub(channel);
        Iterator<HelloProto.HelloResponse> response = stub.serverStream(HelloProto.HelloRequest.newBuilder().setName("小刘").build());
        while (response.hasNext()) {
            HelloProto.HelloResponse next = response.next();
            System.out.println(next.getResult());
        }
//客户端异步处理响应，基于观察者模式，客户端会继续向下执行，服务端每返回一个响应调用一次函数进行处理
        ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost", 8080).usePlaintext().build();
        HelloServiceGrpc.HelloServiceStub stub = HelloServiceGrpc.newStub(channel);
        stub.serverStream(HelloProto.HelloRequest.newBuilder().setName("小刘").build(),
                new StreamObserver<HelloProto.HelloResponse>() {
                    @Override
                    public void onNext(HelloProto.HelloResponse helloResponse) {
                        System.out.println(helloResponse.getResult());
                    }

                    @Override
                    public void onError(Throwable throwable) {

                    }

                    @Override
                    public void onCompleted() {

                    }
                });
```

<h3>客户端流式RPC(<code>Client streaming RPC</code>)</h3>

客户端发送多次请求，浏览器等待所有请求发送完成后一次性响应。

**`.proto`语法**

```
rpc clientStream(stream HelloRequest) returns(HelloResponse){}
```

**服务端**

```java
//基于观察者模式，每当一个请求到来时，调用对应的方法处理    
public StreamObserver<HelloProto.HelloRequest> clientStream(StreamObserver<HelloProto.HelloResponse> responseObserver) {
        return new StreamObserver<HelloProto.HelloRequest>() {
            @Override
            public void onNext(HelloProto.HelloRequest helloRequest) {
            }
            @Override
            public void onError(Throwable throwable) {
            }
            @Override
            public void onCompleted() {

            }
        };
    }
```

**客户端**

```java
        ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost", 8080).usePlaintext().build();
        HelloServiceGrpc.HelloServiceStub stub = HelloServiceGrpc.newStub(channel);
		//
        StreamObserver<HelloProto.HelloRequest> request = stub.clientStream(new StreamObserver<HelloProto.HelloResponse>() {
            @Override
            public void onNext(HelloProto.HelloResponse helloResponse) {

            }
            @Override
            public void onError(Throwable throwable) {

            }
            @Override
            public void onCompleted() {
                System.out.println("响应结束");
            }
        });
        for (int i = 0; i < 10; i++) {
            request.onNext(HelloProto.HelloRequest.newBuilder().setName("小刘").build());
        }
        request.onCompleted();
```

- `gRPC`中发送数据所使用的`onNext`方法是非阻塞的，它会将发送数据写入 gRPC 内部队列，之后会由`Netty EventLoop` 异步发送，如果数据还未从`EventLoop`写入`TCP`缓冲区，会存在数据丢失问题，解决方案有两个：

  - 使用同步工具，将方法变为阻塞式，保证接收到响应后再向下执行

    ```java
    CountDownLatch latch = new CountDownLatch(1);
    
    StreamObserver<Request> requestObserver =
            asyncStub.upload(new StreamObserver<Response>() {
    
                @Override
                public void onNext(Response value) {}
    
                @Override
                public void onError(Throwable t) {
                    latch.countDown();
                }
    
                @Override
                public void onCompleted() {
                    latch.countDown();
                }
            });
    
    requestObserver.onNext(req1);
    requestObserver.onNext(req2);
    
    requestObserver.onCompleted();
    
    latch.await();  // 等待RPC结束
    ```

  - 关闭`channel`：`shutdown`采用优雅关闭，**不会阻塞当前线程**，会 **先把还没发送完的数据 flush 出去，再关闭连接**。

    ```java
    requestObserver.onCompleted();
    
    channel.shutdown();
    channel.awaitTermination(5, TimeUnit.SECONDS);
    ```

<h3>双向流RPC(<code>Bi-directional streaming</code>)</h3>

客户端可以发送多次请求，浏览器可以返回多个响应。

**`.proto`**

```
rpc biStream(stream HelloRequest) returns(stream HelloResponse){}
```

**服务端**

```
    public StreamObserver<HelloProto.HelloRequest> biStream(StreamObserver<HelloProto.HelloResponse> responseObserver) {
        return new StreamObserver<HelloProto.HelloRequest>() {
            @Override
            public void onNext(HelloProto.HelloRequest helloRequest) {              
            }
            @Override
            public void onError(Throwable throwable) {
            }
            @Override
            public void onCompleted() {
                responseObserver.onCompleted();
            }
        };
    }
```

**客户端**

```java
    public static void main(String[] args) {
        ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost",8080).usePlaintext().build();
        HelloServiceGrpc.HelloServiceStub stub = HelloServiceGrpc.newStub(channel);
        StreamObserver<HelloProto.HelloRequest> requestObserver = stub.biStream(new StreamObserver<HelloProto.HelloResponse>() {
            @Override
            public void onNext(HelloProto.HelloResponse helloResponse) {
            }

            @Override
            public void onError(Throwable throwable) {
            }

            @Override
            public void onCompleted() {
                System.out.println("响应完成");
            }
        });
        requestObserver.onCompleted();
    }
```



# 小知识

## 微服务架构

一种分布式架构风格，把系统按业务领域拆分成多个单一职责的独立服务，每个服务独立部署、独立扩展，通过网络通信协作组合形成复杂的大型应用。

- 单一职责：微服务拆分粒度小，每一个服务都对应唯一的业务能力，做到单一职责，避免重复业务开发
- 面向服务：微服务对外暴露业务接口
- 自治：团队独立、技术独立、数据独立、部署独立
- 隔离性强：服务调用做好隔离、容错、降级，避免出现级联问题

![image-20250823111654427](D:\笔记\图片\image-20250823111654427.png)

![image-20250823111924349](D:\笔记\图片\image-20250823111924349.png)

## 单体架构

将系统的所有功能模块打包成一个应用，统一部署，一起运行。

- 优点
  - 架构简单
  - 部署成本低

- 缺点
  - 耦合度高
  - 系统可用性差
  - 系统发布效率低
  - 团队协作成本高

## 分布式架构

系统由多个运行在在不同节点上的组件组成，这些组件通过网络通信协作完成整体功能。

## Spring Boot 设置端口的几种方式

| 方式       | 示例                 | 优先级               |
| ---------- | -------------------- | -------------------- |
| 配置文件   | `application.yml`    | 默认                 |
| 系统属性   | `-Dserver.port=9090` | 高于配置文件         |
| 命令行参数 | `--server.port=7070` | 最高，临时修改最方便 |

Spring Boot 支持从多个地方读取配置，优先级如下：

1. **命令行参数**(程序实参)（比如 `--server.port=9090`）
2. **Java 系统属性**（就是 `-Dserver.port=9090`）
3. **环境变量**
4. **`application.yml` / `application.properties`**
5. **Spring Boot 默认值**（默认 8080）

- -Dkey=value

  > Java的系统属性,是一种JVM参数

  ```
  java -Dserver.port=8080 -jar app.jar 
  ```

  > `-D` 必须在 `java` 命令和 `-jar app.jar` 之间

  - 定义方式：`-Dkey=value`
  - JVM 会自动解析，存放在 `System.getProperties()`
  - 可在程序中通过 `System.getProperty("key")` 访问
  - 整个 JVM 进程全局可见，可以被任意类访问

- 程序参数

  > Java启动时直接传入Main函数args数组的字符串参数,可以通过访问args数组获取它们,又称为命令行参数

  ```
  java -jar app.jar --server.port=9090 arg1
  ```

  > SpringBoot会对args数组中形为`--key=value`字符串进行解析，加载成配置属性

**JVM区分程序参数和虚拟机选项只根据位置，而不是格式。在 主类名之前的都是 JVM 选项在 主类名之后的都是程序参数**

## 适合的微服务技术栈版本

| 组件                 | 版本           | 说明                                        |
| -------------------- | -------------- | ------------------------------------------- |
| Spring Boot          | 2.3.12.RELEASE | 与 Hoxton.SR12 兼容                         |
| Spring Cloud         | Hoxton.SR12    | 对应 Spring Boot 2.3.x                      |
| Spring Cloud Alibaba | 2.2.6.RELEASE  | 与 Hoxton.SR12 完全兼容                     |
| Nacos Server         | 1.4.2          | 推荐 1.x 系列，2.x 可能存在客户端不兼容问题 |
| MyBatis / MySQL      | 任意           | 与 Spring Boot 2.3.x 兼容即可               |

## Nacos集群部署

如果只是单机运行，Nacos 默认会把配置存在内嵌数据库（Derby）里，或者直接存放在内存中。

但在 **集群模式** 下，如果不配置一个公共的数据库，那么不同节点的配置数据就不一致。

所以，Nacos 集群通常都会接一个 **MySQL 数据库**，用来保存配置中心的数据。这样无论你访问哪个节点，拿到的配置内容都是一致的。

## ELK

> elastic stack。elasticsearch结合kibana、Logstash、Beats的一套技术栈。被广泛应用在日志数据分析、实时监控等领域。

## SpringCloud启动流程

![image-20250830140559819](D:\笔记\图片\image-20250830140559819.png)

- bootstrap.yml为引导配置文件，专门用于读取Nacos远程配置的文件
- 需要引入`bootstrap`依赖框架才会读取这个配置文件

## 雪崩

> 微服务调用链路中的某个微服务出现故障，引起整个链路中的所有微服务都不可用。

**解决思路：**

- 保证代码的健壮性
- 保证网络畅通
- 能应对较高的并发需求

## QPS

> 每秒钟请求次数

### DevOps

> 称为持续集成。Dev(Develop)代表开发，Ops(Operation)代表运维，是一种为了打破开发与运维之间阻隔而产生的技术，旨在让开发与运维之间无缝衔接。可以实现自动构建，发布，测试，从而可以更加频繁的小模块化的向主干提交分支，防止主干与分支存在较大差异，从而快速发现错误，解决错误。

DevOps的实现依赖于三个组件，Jenkins借助Docker实现快速部署，Gogs实现代码托管。当分支提交代码到Gogs时，Gogs会向Jenkins发出请求通知。Jenkins会根据请求开启任务。

### Jenkins

> 持续集成的核心

### Gogs

> 简易搭建的自主Git服务，可以快速搭建一个代码托管的私服

## Swagger

> 

# 高并发优化方案

宏观角度来看，对高并发场景的优化方向有三个:

- **提高单机并发：**尽可能减小业务接口的**RT**(ResponseTime),提升单机并发能力。
- **水平扩展：**	将热点服务水平扩展，做好负载均衡，将并发请求分流到不同服务器上
- **服务保护：**	做好服务熔断、降级保护措施，提高服务的可用性

<h3>数据库相关业务优化方案</h3>

对于数据库相关业务而言，对响应时间影响最大的就是对数据库的操作。因此数据库相关业务的主要优化方向就是加快数据库的读写操作。

对于读多写少的业务，通常采用以下两个优化方案：

- 优化代码和SQL
- 添加缓存

对于写多读少的业务，通常从有以下优化方案：

- 优化代码及SQL
- 变同步写为异步写
- 合并写请求

<h4>异步写</h4>

接收请求后，先不处理业务，而是发送MQ消息后直接返回用户结果；后续通过消息监听器监听MQ消息，处理业务。

**优点**

- 大大减少响应时间
- 利用MQ暂存消息，起到流量削峰整形作用
- 降低写频率，减轻数据库并发压力

**缺点**

- 依赖MQ可靠性
- 没有减少数据库写次数，仅仅是降低频率

**适用于业务复杂， 业务链较长的业务**

<h4>合并写请求</h4>

当接收写请求时，先将写数据缓存。通过定时任务定期处理缓存中积攒的数据，将他们一次性写入数据库

**优点**

- 大大减少了响应时间
- 降低了数据库写频率和写次数，减轻数据库并发压力

**缺点**

- 实现复杂
- 依赖缓存系统可靠性
- 不支持事务和复杂业务

**适用于写频率较高、写业务相对简单的场景**

## 云原生

云原生(`CloudNative`)是一套用于在云环境中**构建和运行应用程序的理念和技术体系**，核心特征包括容器化、微服务、持续交付和`DevOps`和弹性伸缩。
