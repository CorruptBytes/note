# MQ

MessageQueue,即消息队列。一种用于跨系统、跨服务传递消息的中间件，它以队列的形式暂存消息，实现不同系统之间的异步通信、削峰填谷和解耦。

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

<h4>如何保证消息可靠性或者说如何保证消息不丢失</h4>

消息的可靠性要从三个阶段保证：生产者、MQ、消费者。

- 对于生产者端，可能存在生产者向MQ发送消息时丢失。

  可以使用发送确认机制，MQ收到消息后回复生产者一个`ACK`以确认接收到消息，并在生产者端添加失败重试机制，保证消息发送成功。

- 对于MQ端，MQ服务可能出现异常导致宕机。

  可以开启MQ的持久化机制，将消息持久化到磁盘中，防止宕机丢失。同时建立MQ集群，保证服务高可用。

- 对于消费者端，可能存在MQ发送消息给消费者时丢失或消费者获得消息后业务处理出现异常

  可以使用手动ACK，在业务处理成功后再确认。如果失败，则触发重试机制，消息会重新发送。

MQ 通常保证“至少一次投递（At-Least-Once)，因此还需要保证消息的幂等性，防止重复消费；

同时也可以使用死信队列，防止异常消息多次重试。

<h4>如何保证消息幂等性或者说如何避免消息重复消费</h4>

避免消息重复消费的核心是在消费者方对消息进行重复检查。

最通用的一个方案就是在消息体中携带一个唯一ID，消费者处理完消息前，将消息ID存储起来(如使用`Redis`的`SetNX`命令)，存储成功则说明是第一次消费。

对于新增操作，可以通过业务消息自带的唯一数据(如订单ID)，先查后插或者直接使用数据库唯一索引，避免消息重复导致新增重复数据。

对于更新操作，可以在消息体中携带版本号，消费者更新数据时对比版本号确认消息是否重复。

# `RocketMQ`

`RocketMQ`是由`Alibaba`开源的高性能，高可靠消息队列，已捐赠给`Apache`基金会。

- `RocketMQ`借鉴了`Kafaka`的设计与架构思想，并在此基础上进行了简化与调整。

<h3>特点</h3>

- **高性能：**吞吐量高(单机支持百万级 QPS )，延迟低。
- **高可用：**`Broker`支持主从复制，自动故障转移
- **多种消息类型：**支持普通消息，顺序消息(可保证顺序消费)，延时消息，事务消息，批量消息。
- **多种消费模式：**支持集群消费(负载均衡)和广播消费(每个消费者都消费)。

<h3>消息模型</h3>

`RocketMQ` 基于 `Pub/Sub` 模型实现。通过 `Topic`（主题）对消息进行逻辑分组，生产者在发送消息时需要指定消息所属的 `Topic`。每个 `Topic` 可以关联多个 `MessageQueue`，`MessageQueue` 是消息的实际存储与传递单元。一个 `Topic` 通过划分多个 `MessageQueue` 来提升消息的并行处理能力与负载均衡能力，其中每个 `MessageQueue` 内的消息是有序的。消费者订阅 `Topic` 后，会从该 `Topic` 下的多个 `Queue` 中拉取消息进行消费。

## 架构

`RocketMQ`的工作架构为：

<h4>生产者(<code>Producer</code>)</h4>

负责发送消息，支持同步 / 异步 /单向发送。
- **同步：** 发送消息后阻塞等待`Broker`响应成功后再向下执行。适用于可靠性要求高，但吞吐量较低的场景。
- **异步：** 消息由专门的 IO 线程发送，当前线程不会阻塞，并通过回调的方式异步处理`Broker`响应。异步发送适用于吞吐量高但对可靠性有要求的场景，防止消息发送阻塞响应。
- **单向：** 只负责发送消息，完全不等待任何返回。单向发送适用于不要求可靠性的场景，可以处理的吞吐量是最高的。

<h4>消费者(<code>Consumer</code>)</h4>

消费者负责消费消息。在`RocketMQ`中，多个消费者可以组成消费者组(`ConsumerGroup`)

消费者组支持两种消费模式：集群模式(负载均衡)，广播模式(广播消费)。

集群模式下负载均衡，一个消费者组中的所有消费者共享消费进度，一个`Queue`可以关联一个消费者，同一时刻一个 `Queue` 只能被一个消费者消费，每一个消费者组在 `Broker` 端维护自己独立的消费进度；广播模式下消息以广播方式分发，每个消费者都会独立消费该 `Topic `下的所有 `Queue`，各自在本地独立维护一份消费进度。

- 无论是集群模式，还是广播模式，不同消费者组都是相互独立，互不影响的。

消费者支持推(push)和拉(pull)两种方式获取消息，推为`Broker`主动推送消息，拉为消费者主动拉取。默认采用拉模式

<h4><code>Broker</code></h4>

`RocketMQ`的核心组件，负责消息存储，消息转发以及消息进度管理。

- 为保证高可用，支持主从集群。`Master`负责读写,`Slave`负责备份数据。当主节点宕机时，可以自动故障转移。
- 为提高吞吐量，可以部署`Broker`集群，一个`Topic`下的多个队列可以分布在不同的`Broker`节点中，同时`Producer` 会在多个 `Queue` 之间轮询发送，以保证负载均衡。

**`CommitLog`**

`CommitLog` 是 `Broker` 用来存储所有消息的物理文件

- 所有 `Topic` 的消息都会顺序写入同一个 `CommitLog`

`Queue`中只存储消息在 `CommitLog` 中的物理偏移量以及一些辅助信息，而不存储消息本体。

<h4><code>NameServer</code></h4>

一个轻量级注册中心，管理`Broker`状态，为`Producer`/`Consumer`提供路由。

- `NameServer`无状态，可以部署独立的多个实例形成集群。

<h4><code>Proxy</code></h4>

`Proxy`是`RocketMQ5.0`引入的新的核心组件，它是一个**网关层**。通过`Proxy`，客户端不再直连 `Broker / NameServer`，而是统一通过 `Proxy` 访问消息系统。

`Proxy`统一了客户端访问入口，支持协议转换，负载均衡，安全控制。

- `Proxy`无状态，支持多实例部署。

## 消息类型

`RocketMQ`支持普通消息，顺序消息(可保证顺序消费)，延时消息，事务消息(解决分布式事务问题)等多种消息类型。

**延时消息**

生产者将消息发送后，`Broker`会先将消息存储到内部的延迟队列中。等待延迟时间到达后，再将消息重新投递到对应`Topic`中。

**批量消息**

将多条消息合并成一个消息，一次性发送出去。减少了网络IO，可以提升吞吐量。

- 批量消息的最大消息大小受`Broker`中配置的影响(默认是4MB)，考虑吞吐量与因素延迟，重试成本，内存压力等因素的平衡，建议不超过1MB
- 批量消息中的所有消息必须属于同一个`Topic`。
- 批量消息中的单条消息不可以是延迟消息，事务消息。

<h3>事务消息</h3>

事务消息用来解决本地事务与消息发送的一致性问题，避免出现本地事务失败，而消息发送成功或本地事务成功，而消息发送失败的情况。

<h4>原理</h4>

事务消息基于两阶段提交的思想实现。

1. `Producer`会先向`Broker`发送一条半消息。半消息存储在一个`RocketMQ`中一个特定的`Topic`中，对消费者不可见。
2. `Producer`端执行本地事务，并在事务执行完成后将事务的执行结果作为消息发送到`Broker`。
3. 根据本地事务的结果：如果成功，将半消息转移到目标`Topic`，消息变为可消费；如果失败，将半消息删除。

为了防止`Producer`出现异常，无法向`Broker`发送事务执行结果，`Broker`会主动进行事务回查，也就是主动询问`Producer`事务的状态。

<h4>事务消息状态</h4>

事务消息具有三种状态：

|        状态        |       含义       |
| :----------------: | :--------------: |
|  `COMMIT_MESSAGE`  | 提交，消息可消费 |
| `ROLLBACK_MESSAGE` |  回滚，消息删除  |
|      `UNKNOW`      |  未知，等待回查  |

最初消息为`UNKONW`状态，当本地事务执行完成后，会将执行结果发送到`Broker`，根据执行结果改变事务消息的状态并执行对应操作。

## 消息过滤

`RocketMQ`支持多种方式在 Topic 内进一步细分消息，从而帮助消费者在`Topic`下进行更精细的消息过滤。

### `Tag`

生产者在发送消息时为其指定`Tag`，消费者订阅`Topic`时，可以订阅指定`Tag`的消息，且可以通过`tag1 || tag2`的形式同时订阅多个`Tag`。

- 生产者不能控制指定`Tag`的消息进入指定的队列，只能负载均衡分配。

**原理**

`Broker`会把`Tag`转换为一个hash值(`TagCode`)。`Consumer`在拉取消息时,`Broker`会根据`TagCode`进行快速过滤。

### `SQL92`

生产者在发送消息时可以为消息设置自定义属性(key-value 形式，值为字符串)。消费者在订阅时，可以通过 `SQL92` 标准的条件表达式基于属性对消息进行筛选过滤，`Broker` 会执行表达式筛选，只将符合条件的消息返回给消费者。

- 消息过滤是在`Broker`端进行的。

## 消息持久化

## 安装部署

<h3><code>RocketMQ</code>安装</h3>

`RocketMQ`本身包含两个核心组件：`NameServer`和`Broker`，这两个组件可以独立部署启动。

**拉取官方镜像**

```
docker pull apache/rocketmq
```

**创建网络**

```
docker network create rmq-net
```

**启动`NameServer`**

使用`RocketMQ`中`bin`目录下的`mqnamesrv.sh`命令启动`NameServer`服务

```
docker run -d \
--name rmqnamesrv \
--network rmq-net \
-p 9876:9876 \
apache/rocketmq \
sh mqnamesrv
```

**启动`Broker`**

使用`RocketMQ`中`bin`目录下的`mqborker.sh`命令启动`Broker`服务。 

```
docker run -d \
--name rmqbroker \
--network rmq-net \
-p 10911:10911 -p 10909:10909 \
-e "NAMESRV_ADDR=rmqnamesrv:9876" \
apache/rocketmq \
sh mqbroker -n rmqnamesrv:9876
```

- 启动后，可以使用`RocketMQ`的`bin`目录下的`tools.sh`进行快速测试

  ```sh
  #发送消息，默认发送1000条
      sh tools.sh org.apache.rocketmq.example.quickstart.Producer
  #接收消息，会一直等待并接收消息
  sh tools.sh org.apache.rocketmq.example.quickstart.Consumer
  ```

<h3><code>RocketMQ Dashboard</code>安装</h3>

`RocketMQ`内部没有提供可视化管理工具，需要使用外部控制台查看其运行情况。`RocketMQ Dashboard`是`Apache RocketMQ` 官方提供的一个基于`Web`服务的可视化管理工具

**拉取镜像**

```
docker pull apacherocketmq/rocketmq-dashboard
```

**启动容器**

```
docker run -d \
  --name rmqdashboard \
  --network rmq-net \
  -p 8080:8082 \
  -e "JAVA_OPTS=-Drocketmq.namesrv.addr=rmqnamesrv:9876" \
  apacherocketmq/rocketmq-dashboard
```

- `NameServer`地址通过`Java`选项配置，如果在同一个`Docker`网络中，可以直接使用容器名。

## Java客户端

**`Mvaen`依赖坐标**

```xml
        <dependency>
            <groupId>org.apache.rocketmq</groupId>
            <artifactId>rocketmq-client</artifactId>
            <version>5.1.0</version>
        </dependency>
```

### 生产者

<h3><code>DefaultMQProducer</code></h3>

默认的生产者实现。

```java
DefaultMQProducer producer = new DefaultMQProducer("test");
producer.setNamesrvAddr("172.31.85.64:9876");
//start方法会进行大量初始化配置，必须在发送消息前调用
producer.start();
producer.send(new Message("study","你好".getBytes(StandardCharsets.UTF_8)));
producer.shutdown();
```

<h4>常用属性</h4>

**所属生产者组**

```
private String producerGroup;
```

<h3>常用方法</h3>

**同步发送消息**

```
SendResult send(final Message msg);
SendResult send(final Message msg, final long timeout);
```

**异步发送消息**

通过回调函对`Broker`响应进行处理。

```
void send(final Message msg, final SendCallback sendCallback);
```

**单向发送消息**

```java
void sendOneway(final Message msg)
```

**发送顺序消息**

在`MesageQueueSelector`中根据`arg`选择一个`Queue`，保证消息发送到一个队列，进而保证有序性。

```
SendResult send(final Message msg, final MessageQueueSelector selector, final Object arg)
```

- 使用顺序消息时，消费者端也要配合进行顺序消费

<h3><code>TransactionMQProducer</code></h3>



### 消费者

<h3><code>DefaultMQPushConsumer</code></h3>

使用推模式获取消息的消费者。

```

```

<h3><code>DefaultMQPullConsumer</code></h3>

默认的拉模式消费者实现，目前已经废弃，推荐使用`DefaultLitePullConsumer`替代。



```java
//通过为Message设置延时属性发送延时消息
void setDelayTimeLevel(int level)
```

### 消息

<h4><code>Message</code></h4>

在`RocketMQ`中使用`Message`对象封装一条消息。

**延迟消息**

通过为`Message`设置`delayTimeLevel`属性或将其变为延迟消息。

```
void setDelayTimeLevel(int level)
```

**批量消息**

将多条`Message`组装到一个`Collection`中使用生产者的`send`发送，即为批量消息。

```
SendResult send(final Collection<Message> msgs)
```

### 消息过滤

### `Tag`

**生产者指定消息`Tag`**

```
Message message = new Message();
message.setTags("tag1");
```

**消费者订阅指定`Tag`**

```java
void subscribe(final String topic, final String subExpression)

DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("group1");
consumer.subscribe("topic","tag1 || tag2");
//或者使用MessageSelector
consumer.subscribe("topic", MessageSelector.byTag("tag1 || tag2"));
```

- `||`表示同时订阅多个`Tag`的消息

### `SQL92`

**指定消息属性**

```java
Message message = new Message();
message.putUserProperty("user","xiaoliu");
```

**消费者设置SQL表达式**

```java
void subscribe(final String topic, final MessageSelector selector)

consumer.subscribe("topic", MessageSelector.bySql("user = xiaoliu"));
```



- 只有推模式的消费者客户端可以使用`SQL`过滤，拉模式无法使用。



## `SpringBoot`整合`RocketMQ`

**起步依赖**

```
<dependency>

```