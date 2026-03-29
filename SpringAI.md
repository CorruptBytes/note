# 概述

**Spring AI** 是 **Spring 官方** 提供的 AI 应用开发框架，用于在 **Spring Boot** 生态中 **统一、简化** 大语言模型（LLM）及相关 AI 能力的集成。

## 相关依赖

<h3><code>SpringAI</code>版本管理依赖</h3>

```xml
<dependencyManagement>
	<dependencis>
		<dependency>
			<groupId>org.springframework.ai</groupId>
			<artifactId>spring-ai-bom</artifactId>
			<version>${spring-ai.version}</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>
```

- 功能类似`SpringBoot`父工程，管理相关依赖的版本，引入后无需再手动声明依赖版本。

<h3><code>ollama</code>起步依赖</h3>

```xml
<dependency>
	<groupId>org.springframework.ai</groupId>
	<artifactId>spring-ai-starter-model-ollama</artifactId>
</dependency>
```

**`ollama`模型配置**

```yaml
spring:
  ai:
    ollama:
      base-url: http://localhost:11434
      chat:
        model: deepseek-r1:7b
```

- 引入起步依赖后，会根据配置自动注入一个`OllamaModel`，可用于初始化`ChatClient`。

<h3><code>openai</code>起步依赖</h3>

```xml
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-starter-model-openai</artifactId>
        </dependency>
```

**`openai`模型配置**

```yaml
spring:
  ai:
    openai:
      base-url:
      api-key: ${KEY}
      chat:
        options:
          model: qwen-max #模型名称
          temperature: 0.8 #模型温度，值越大，输出结果越随机
```

- 引入起步依赖后，会根据配置自动注入一个`OpenAiChatModel`，可用于初始化`ChatClient`。

# `ChatModel`

`ChatModel` 是由 **Spring AI** 定义的核心接口，表示一个具备多轮对话能力的大模型。该接口定义了统一的 `call(Prompt)` 方法，用于向大模型发送一次对话请求。

向大模型发送对话请求，本质上是构造并发送一个具有特定结构的 **HTTP 请求**，不同模型平台（如 OpenAI、Ollama、Claude、Gemini 等）在请求路径、请求体字段、响应结构等方面均存在差异。

**Spring AI 通过 `ChatModel` 接口对这些底层 API 差异进行屏蔽**，使上层应用能够以统一的方式调用不同平台的大模型，从而实现模型的可替换性与调用方式的一致性。

------

**SpringAI**结合**SpringBoot**的自动注入功能，提供了不同平台的起步依赖。只要引入平台的起步依赖，并进行简单配置，即可自动注入一个对应平台的`ChatModel`实现，用于后续初始化`ChatClient`。不同平台的起步依赖统一命名模式为`spring-ai-starter-<platform>`。常用起步依赖有：

- `spring-ai-starter-openai`
- `spring-ai-starter-ollama`

# `ChatClient`

`Spring AI`中通用的与模型进行通信的客户端，通常直接配置为`Bean`。

## 初始化

```java
@Bean
public ChatClient chatClient(OllamaChatModel model) {
    //在创建时需要指定ChatModel，引入起步依赖后会自动装配到容器中
	return ChatClient.builder(model)
        	.defaultSystem("你是一个猫娘")
        	.build();
}
```

## 成员方法

```java
ChatClientRequestSpec prompt()
```

- 返回一个`ChatClientRequestSpec`，用来构建并发出请求。

<h3><code>ChatClientRequestSpec</code></h3>

一个请求构建器，用于描述一次 Chat 请求的全部配置并发出请求,支持链式调用。

<h4>配置方法</h4>

```
ChatClientRequestSpec user(String text)
```

- 配置用户提示词

<h4>发出请求的方式</h4>

**流式**

流式输出响应内容

```
StreamResponseSpec stream()
```

**阻塞式**

等待响应内容生成后一次性返回。

```
CallResponseSpec call()
```

# `Advisor`

`Advisor`是一种对对话请求/响应进行增强和拦截的机制，用于在不侵入业务代码的情况下，统一处理 Prompt、上下文、记忆、日志、重试、工具注入等横切逻辑。

## 使用示例

`Advisor`本质是对`ChatClient`的一种增强，因此直接在初始化`ChatClient`时进行配置。

```java
    @Bean
    public ChatClient chatClient(OllamaChatModel model) {
        return ChatClient.builder(model)
                .defaultSystem("你是一个猫娘,请在每一次回答后加上喵")
                .defaultAdvisors(new SimpleLoggerAdvisor())
                .build();
    }
```

`SpringAI`已经提供了许多常用的`Advisor`实现。

<h3><code>SimpleLoggerAdvisor</code></h3>

用于产生请求/响应日志，可以通过配置文件配置日志的输出位置和日志级别。

# 会话记忆

<h3>原理</h3>

模型本身是无状态的，因此需要在每次请求时，将历史对话作为上下文的一部分一并发送给大模型，以实现对话连续性。在`openAI`制定的规范中，发送给模型的`message`包含三个字段：

```json
{
  "role": "user",
  "content": "你好，介绍一下你自己",
   "name":"optional"
}
```

- `role`：表示**这条消息的身份**，决定了模型如何理解这段内容。

  |        role         |                    含义                    |
  | :-----------------: | :----------------------------------------: |
  |      `system`       | **系统指令**，用于设定模型行为、规则、人格 |
  |       `user`        |                **用户输入**                |
  |     `assistant`     |              **模型历史回复**              |
  | `tool` / `function` |    工具或函数返回结果（新版本中更常见）    |

- `content`：表示**消息的具体内容**。

其中角色为`assistant`的消息，就是用来承载模型的历史回复。

<h3>实现示例</h3>

1. 注册`ChatMemroy`为`Bean`，并为`ChatClient`添加`MessageChatMemoryAdvisor`，它会使用`ChatMemory`存储历史消息，并在每次对话请求前后，自动将历史消息从 `ChatMemory` 中取出并注入到 `prompt` 中，同时在请求完成后把新的消息写回 `ChatMemory`

   ```java
   	@Bean
       public ChatMemory chatMemory() {
           return MessageWindowChatMemory.builder().build();
       }
       @Bean
       public ChatClient chatClient(OllamaChatModel model,ChatMemory chatMemory) {
           return ChatClient.builder(model)
                   .defaultSystem("你是一个猫娘,请在每一次回答后加上喵")
                   .defaultAdvisors(SimpleLoggerAdvisor.builder().build(), 
                           MessageChatMemoryAdvisor.builder(chatMemory).build())
                   .build();
       }
   ```

2. 在发送请求时通过将会话ID添加到`Advisor`上下文中以使`Advisor`可以访问到。

   ```java
   @RequestMapping(value = "/chat",produces = "text/html;charset=utf-8")
   public Flux<String> chat(String prompt,String chatId) {
        return chatClient.prompt().user(prompt).advisors(a -> a.param(ChatMemory.CONVERSATION_ID,chatId)).stream().content();
   }
   ```

   - 在请求完成后，`MessageChatMemoryAdvisor`会优先从上下文中键为`ChatMemory.CONVERSATION_ID`的键值对中获得会话ID，否则使用默认会话ID。

## `ChatMemory`

`SpringAI`定义的用于规范会话记忆存储的接口，通过该接口可以屏蔽会话记忆底层的存储介质。使得无论采用内存、数据库还是其他存储方式，只要实现了 `ChatMemory` 接口，便可以通过统一的方式存取会话记忆。主要包含三个方法:

```java
public interface ChatMemory {
    void add(String conversationId, List<Message> messages);
    List<Message> get(String conversationId);
    void clear(String conversationId);
}
```

- 通常直接将`ChatMemory`注册为一个`Bean`。

  ```
  @Bean
  public ChatMemory chatMemory() {
  	return new MessageWindowChatMemory();
  }
  ```

### `MessageWindowChatMemory`

**SpringAI**提供了一个基于内存的`ChatMemory`实现，底层通过一个`Map`在内存中存储对话记忆。

## `MessageChatMemoryAdvisor`

`Spring AI` 提供的一个内置 `Advisor`，用于在每次对话请求前后，自动将历史消息从 `ChatMemory` 中取出并注入到 `prompt` 中，同时在请求完成后把新的消息写回 `ChatMemory`，从而实现会话记忆的自动管理。

# 提示词工程

提示词工程(`Prompt engineering`)是一个通过设计和优化提示词，使语言模型能够稳定、可靠地产生期望输出的过程。

<h2>提示词编写原则</h2>

- **明确模型角色：**必须显式指定模型的角色与职责，而不能由模型自行推断其身份，以避免行为发散并提升输出一致性。
- **输出格式优先：**在描述任务内容之前，应优先定义输出格式与约束条件。输出格式的不稳定将直接导致结果无法在工程系统中使用。
- **明确禁止：**明确不能做什么，比说明应该做什么更有助于约束模型行为。
- **单一职责：**一个 `Prompt` 应只解决一类明确的问题。对于复杂任务，应拆分为多个子 Prompt 分步完成，以降低复杂度和不确定性。
- **失败定义：**必须明确模型在无法完成任务时的行为,以避免编造答案，提升系统可靠性。
- **限制输出边界：**通过限制输出长度、结构或字段数量，降低模型输出的随机性，从而提升结果的可预测性。
- **结构优于语气：**使用结构化、规则化的指令描述需求，而非依赖自然语言或语气表达。结构化输入更易被模型稳定遵循。
- **配置化：**Prompt 应作为配置资源进行管理，而非临时文本。应支持版本控制、复用与迭代，以适配长期工程演进。

## `Prompt`分层结构

```
System Prompt
  ├── Role（角色）
  ├── Skills（技能）
  ├── Rules（规则）
  ├── Output Format（输出格式）
  └── Constraints（约束）
```

<h3><code>Skill</code></h3>

`Skill`就是一段可复用的描述并约束模型能力的 Prompt 模块，以限制模型自由发挥，增强可控性。

<h4>工程实践</h4>

在现有工程实践中，会在与模型交互的服务端配置多套`skill`，每一套`skill`都是一个单独的文件，适用于一类场景。在文件中，会有一个简短的`metadata`模块描述该`skill`。服务端在与模型交互前，会收集所有的`metadata`信息组合放到系统提示词中与用户提示词一起发到模型。其中收集`metadata`的过程称为`Discovery`。

大模型在收到请求后，会理解用户语义并判断与哪个`skill`有关，这个过程称为`Activation`。如果有`skill`符合，会先生成一个特殊的回复，要求客户端读取该`skill`文件的完整内容，然后大模型就可以根据该`skill`生成完整的符合要求的回复。



# `Function Calling`

`SpringAI`通过`Advisor`简化了`Function Calling`的开发。开发者只需要定义并配置`Tool`后，`SpringAI`会自动在发送请求时携带`Tool`信息并在解析响应后调用`Tool`。

## `Tool`定义

`SpringAI`提供了一系列注解声明`Tool`相关信息，开发者只需要在编写`Tool`时为其加入注解描述相关信息，SpringAI会扫描注解生成文档说明。

- 通常将一类`Tool`作为方法封装到一个工具类中。

<h3><code>@Tool</code></h3>

添加在方法上，描述方法相关信息。

```java
@Tool(String name, String description,boolean returnDirect)
```

|                        |                            |             |
| :--------------------: | :------------------------: | :---------: |
|     `String name`      |      工具名称，非必须      |             |
|  `String description`  |     描述工具作用，必须     |             |
| `boolean returnDirect` | 方法结果是否直接返回给前端 | 默认`false` |

<h3><code>@ToolParam</code></h3>

添加在方法参数或类字段上,描述参数或字段含义

```java
@ToolParam(boolean required,String description)
```

## `Tool`配置

`Tool`相关信息是在每次对话请求时携带在`HTTP`请求体中的，因此需要在初始化`ChatClient`中配置`Tool`。

```java
ChatClient.Builder ChatClient.defaultTools(Object... toolObjects)
```

# RAG

**RAG**(`Retrieval-Augmented Generation`，检索增强生成)，是一种在文本生成过程中显式引入外部知识检索机制，使模型能够基于检索到的文档进行回答，从而提升知识密集型任务回答准确性的架构。

<h4>向量模型(<code>Embedding Model</code>)</h4>

本质也是一种通过训练而来AI模型，可以接收文本输入，将文本向量化，保证内容越相似的文本，在向量空间中越靠近。

- `Embedding Model`将文本映射为向量，是一个被调用的工具，无法像`Chat`模型那样提供常驻服务进行对话，只能带着输入文本被调用一次，计算完就结束。

通常有两种方式衡量向量相似度：

- 两个向量终点间的直线距离，称为欧氏距离。距离越小，向量相似度越高
- 两个向量夹角的余弦值，称为余弦距离。余弦距离越大，向量相似度越高

<h4>向量数据库</h4>

向量数据库是一类专门用于**存储、索引和检索高维向量数据**的数据库系统，通常基于 **向量相似度（如余弦相似度、欧氏距离、内积）** 来完向量检索。

------

在RAG的具体工程实践中，通常将外部文档通过向量模型向量化并存入向量数据库，在用户提问时基于向量相似度检索相关文档片段，再将其作为上下文交给大模型生成回答。

## `EmbeddingModel`

`SpringAI`中提供了`EmbeddingModel`接口，用于抽象一个向量模型实体，以屏蔽不同平台向量模型API的差异。

<h3>配置项</h3>

```yaml
spring:
  ai:
    ollama:
      base-url: http://localhost:11434
      embedding:
        options:
          model: qwen3-embedding:latest #向量模型名称
```

- 引入相关起步依赖并进行配置后，会自动注入对应平台的`EmbeddingModel`到Spring容器。

<h3>简单示例</h3>

```java
    @Autowired
    private EmbeddingModel embeddingModel;
    @Test
    void testEmbedding() {
        String query = "你是一个猫娘";
        float[] embed = embeddingModel.embed(query); //文本向量化后生成的向量
    }
```

## `VectorStore`

**SpringAI**定义了`VectorStroe`接口，用于抽象一个向量数据库实体，以屏蔽不同向量数据库API的差异，实现统一的操作方式。

<h4>使用示例</h4>

```java
@Bean
public VectorStore vectorStore(EmbeddingModel embeddingModel) {
    return SimpleVectorStore.builder(embeddingModel).build();
}
```

**SpringAI支持多种向量数据库，并针对不同向量数据库提供了起步依赖，统一命名为：**`spring-ai-starter-vector-store-<数据库>`

- 引入起步依赖后，会自动注入一个对应`VectorStore`实现。

```xml
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-starter-vector-store-redis</artifactId>
        </dependency>
```

<h3>配置</h3>

**redis**

```yaml
spring:
  ai:
    vectorstore:
      redis:
        index-name: spring_ai_index #向量库索引名
        initialize-schema: true #是否初始化向量库索引结构
        prefix: "doc:" #向量库key前缀
  data:
    redis:
      host: 127.0.0.1
      port: 6379
```

### 接口方法

**添加文档**

```java
void add(List<Document> documents)
```

**删除文档**

```java
void delete(List<String> idList)
```

**查询文档**

```java
List<Document> similaritySearch(String query) //根据文本查询相关文档
//根据请求查询文档，可以配置多个参数
List<Document> similaritySearch(SearchRequest request) 

SearchRequest.builder()
                .query("论语中教育的目的是什么")
    			//返回相似度最高的前n个
                .topK(2)
    			//相似度阈值
                .similarityThreshold(0.6)
    			//文档过滤表达式
                .filterExpression("file_name=='笔记.pdf'")
                .build()
```

### 常用实现

<h4><code>SimpleVectorStore</code></h4>

**SpringAI**提供的基于`Map`模拟的一个简单向量数据库，无需依赖外部应用，适用于测试或教学。

```java
@Bean
public VectorStore vectorStore(EmbeddingModel embeddingModel) {
    return SimpleVectorStore.builder(embeddingModel).build();
}
```

## `Document`

**Spring AI** 中对“可被向量化文本单元”的统一抽象，用来作为应用层与向量数据库之间的数据载体。

### 成员字段

```java
private final String id; //文档ID
private final Media media; // 多模态内容
private final String text; // 纯文本内容
private final Map<String, Object> metadata; //文档的元数据，也就是文档的基本信息
```

### `DocumentReader`

`DocumentReader` 是 Spring AI 提供的用于从文本或文件中读取内容，并生成 `Document` 对象的统一接口。

<h4>使用示例</h4>

```java
public void test(Resource resource) {
    //1.创建DocumentReader
    PagePdfDocumentReader reader = new PagePdfDocumentReader(resource,
            PdfDocumentReaderConfig.builder()
                    //PDF文本格式化器，这里使用默认的
               .withPageExtractedTextFormatter(ExtractedTextFormatter.defaults())
                    //PDF拆分方式，这里采用每页拆分为一个文档
                    .withPagesPerDocument(1)
                    .build());
    //2.读取PDF
    List<Document> documents = reader.read();
    //3.将文档存入向量数据库
    vectorStore.add(documents);
}
```

Spring AI 提供了多种 `DocumentReader` 实现，以支持对多种文本与文件格式的解析：

- **`JsonReader`：**内置

- **`TextReader`：**内置

- **`PagePdfDocumentReader`：**

  ```xml
  <dependency>
      <groupId>org.springframework.ai</groupId>
      <artifactId>spring-ai-pdf-document-reader</artifactId>
  </dependency>
  ```

## `QuestionAnswerAdvisor`

基于**向量化**的RAG架构，在每次发送请求前都需要将用户提示词向量化后据此检索向量数据库并将检索结果添加到对话请求中，**SpringAI**提供了`QuestionAnswerAdvisor`简化了这一流程。

<h4>相关依赖</h4>

```xml
<dependency>
   <groupId>org.springframework.ai</groupId>
   <artifactId>spring-ai-advisors-vector-store</artifactId>
</dependency>
```

<h4>使用示例</h4>

```java
    @Bean ChatClient pdfChatClient(OllamaChatModel chatModel,VectorStore vectorStore) {
        return ChatClient.builder(chatModel)
                .defaultAdvisors(
                        QuestionAnswerAdvisor.builder(vectorStore)
                                .searchRequest(SearchRequest.builder().build())
                                .build()).build();
    }
pdfChatClient.prompt(prompt).advisors(
  a -> a.param(QuestionAnswerAdvisor.FILTER_EXPRESSION,"file_name='hello.pdf'"))
    .stream()
    .content;
```

- 有的检索参数是需要由用户请求参数确认的(如过滤表达式)，**`QuestionAnswerAdvisor`**会在检索前检查`Advisor`上下文有没有键为`QuestionAnswerAdvisor.FILTER_EXPRESSION`的键值对，有则添加其内容为过滤表达式。

# 多模态

模态（`Modalities`）是信息或数据的表现形式，如文本、图像、音频、视频等。多模态（`Multimodal`）是指系统能够对来自不同模态的数据进行理解、处理，并在必要时生成不同模态输出的能力。

- 多模态是AI模型的能力，与**SpringAI**无关,**SpringAI只是封装了发送多模态请求的接口**。

**`ChatClient`提供了多模态输入的接口**

```java
chatClient.prompt().user(p -> p.text("你好").media(media)).call().content()
```



# 小知识

## AI应用开发技术架构

## 纯Prompt问答

> 利用大模型的推理能力，通过Prompt提问完成业务

## Agent + Function Calling

> AI拆解任务，调用业务端提供的接口实现复杂业务

- 特征

  > 将应用端业务能力与AI大模型推理能力结合，简化复杂业务功能开发

## Fine-tuning

> 针对特有业务场景对基础大模型做数据训练和微调，以满足特定场景的需求

- 特征

## NLP

Natural Language Processing ,自然语言处理，用于理解和处理人类语言

## LLM

> *Large Language Models,大型语言模型*

## GPT

- G

  即Generative，生成式，根据上文预测之后应该出现的文本，从而形成连贯的文本输出

- P

  > *即Pre-trained，预训练，通过大规模的文本数据进行预训练，让大模型可以理解人类的语言的语法，词性*

- *T*

  > *Transformer，深度学习的一种神经网络模型，多数AIGC模型都依赖于此*

## *Transformer*

> *一种神经网络模型，在2017年由谷歌发表的论文《Attention Is All You Need》中首次提出，其最重要的特点是注意力机制*

## *大模型应用*

> *基于大模型的推理，分析，生成能力，结合传统编程能力，开发出的应用，如可以具有上下文的大模型，通过传统编程能力将之前的文本缓存，然后一起提交给大模型生成后续对话*

## Prompt

提示词，可分为`User prompt`和`System Prompt`。

`User prompt`就是用户输入给AI的话

`System prompt`是由开发者或者用户设定的用于描述AI的角色，性格，背景知识，语气等特点的提示词。

当用户发送用户提示词时，系统会自动把`System Prompt`一起输入给 AI模型，以使对话更加自然。

## Agent

Agent 是一个能够感知环境，根据用户输入和当前状态进行决策，并主动调用预设的服务或工具，持续推进目标达成的智能体。这些供`Agent`调用的预设服务或工具称为`Agent Tools`。

<h4>原理</h4>

AI 模型根据当前环境和用户输入，输出系统约定的结构化文本，系统解析文本并调用预设的服务或工具，执行后再将结果返回给用户。

AI模型本质是一个概率模型，即使通过`System Prompt`中预设了输出的文本格式，还是有可能返回格式错误的内容。有多种解决方案：

- `Agent`发现返回格式不对时，自动进行重试。

- `Function Calling`:核心思想是统一格式，规范描述。Function Calling 本质上是一套模型与系统之间的结构化调用协议，用来替代在 Prompt 中用自然语言约定工具调用和返回格式。

  传统方式中，通过自然语言在`System Prompt`中描述预设的服务与工具以及返回的格式；`Function Calling`对这些描述进行了标准化并将其从`System prompt`中剥离出来，将每一个`Agent Tools`用一个`json`对象定义，放在一个单独的字段中。

  ```json
  {	#工具名
      "name": "list_files",
   	"desc": "列出目录",
   	"params":{
   		"path": "str"
  	}
  }
  ```

  在运行时，这些工具定义会作为独立的结构化信息提供给模型，而不再混杂在自然语言 Prompt 中，降低了模型输出的不确定性，也提升了 Agent 系统在多工具场景下的稳定性和可维护性。

在Function Calling中，工具定义的 JSON 结构以及模型返回结构，都是在模型训练阶段结合 API 协议约定好的，不同模型或不同厂商的具体字段和格式可能不同。

Function Calling 的本质，就是在模型训练阶段教会模型一套“工具调用协议”， 当模型判断需要调用工具时，就会严格按照该协议，生成固定结构的内容（通常是 JSON），供外部程序确定性解析并执行。

## MCP

模型上下文协议(`Model Context Protocol`)是一个通信协议，用来规范`Agent`和`Tools`之间怎么交互。`Agent`接收到AI模型的输出后，需要解析输出表调用对应的`Agent Tools`,最简单的做法是`Agent`与`Agent Tools`写在同一个程序中，在使用时直接函数调用即可。但是这将`Agent`与`Agent Tools`强耦合，`Agent Tools`也无法复用。因此，可用将`Agent Tools`变成服务，统一托管，所有`Agent`均可以来调用。

在 MCP 中，提供工具和资源的一方称为 *MCP Server*，发起调用的一方称为 *MCP Client*。
 MCP 作为通信协议，规定了 MCP Server 与 MCP Client 之间的交互方式，以及 MCP Server 必须暴露的标准接口（例如：查询可用工具列表、获取工具定义、执行工具调用等）。

## `Agentic AI`



# 部署

> *模型部署有两种方式，云部署与本地部署*

## 云部署

将模型部署在云平台的服务器上

- *优点*
  1. *前期成本低*
  2. *部署维护简单*
  3. *弹性扩展*
  4. *全球访问*
- *缺点*
  1. *数据隐私存在问题*
  2. *需要依赖网络才能访问云服务*
  3. *长期成本高*

### 开放API

> *使用已经部署好的服务，访问其API获取服务*

- *优点*
  1. 前期成本极低
  2. 无需部署
  3. *无需维护*
  4. *全球访问*
- *缺点*
  1. *数据隐私存在危险*
  2. *网络依赖*
  3. *长期成本高*
  4. *定制限制*

## 本地部署

将模型部署到本地服务器上

- *优点*
  1. *数据安全*
  2. *不依赖外部网络*
  3. *长期成本低*
  4. *可以高度定制模型*
- *缺点*
  1. *初始成本高*
  2. *维护复杂*
  3. *部署周期长*

### `ollama`

一个模型部署的工具，不仅可以部署大型语言模型，还可以用于管理大型语言模型。底层基于`Docker`实现。

- 通过`OllamaSetup.exe /DIR=安装目录`的方式更改默认安装目录。
- `Ollama`保存模型的位置为`OLLAMA_MODELS`环境变量声明的位置。

<h4>使用示例</h4>

`Ollama`启动后，会在后台运行一个服务，可以通过命令行与`Ollama`进行交互，从而下载，部署，管理并使用模型。

1. 下载并启动模型

   ```bash
   ollama run deepseek-r1:8b
   ```

   - 如果本地没有下载过此模型，则会自动下载并启动。

2. 启动后会自动进入与模型对话的控制台，可以在控制台中与模型交互。

#### 命令

`Ollama`的命令与`docker`非常相似。

**查看帮助信息**

```
ollama --help
```

**查看`ollama`版本**

```
ollama -v
```

**运行模型**

```
ollama run <model>
```

- 如果本地没有此模型，则会自动下载模型后启动。

**查看正在运行的模型信息**

```
ollama ps
```

**查看本地存在的模型**

```
ollama list
```

#### `Modelfile`

类似`Docker`中的`Dockerfile`，通过`Modelfile`可以根据现有模型派生一个新模型

## 调用模型API

现在大模型的API接口规范基本遵循openAI的规范，示例如下

```python
//初始化OpenAI客户端
client = OpenAI(api_key="<DeepSeek API Key>",base_url="http://api.deepseek.com")
//发送http请求到大模型
response = client.chat.completions.create(
		model = "deepseek-r1",
    	//role字段为角色
		messages=[{"role":"system","content":"你好!"}],
		stream=False)
//获取返回结果

print(response.choices[0].message.content)
```

- 本质上调用模型的API就是向部署了模型的服务器发送携带特定格式数据的http请求

## `SSE`

`SSE`意为服务器发送事件(`Server-Sent Events`)，是一种基于`HTTP`协议实现的服务器推送机制，可以实现服务器向浏览器的单向实时事件流传输。

在`SSE`通信中，客户端发起一次 HTTP 请求后，服务端通过保持响应未结束的方式形成`HTTP`长连接。服务端在此期间以 `text/event-stream` 格式持续向客户端分块推送数据，客户端可以实时接收这些数据。

- `SSE`底层通常基于 HTTP/1.1 的分块传输实现流式数据发送，它在`HTTP chunked`的基础上定义了一套标准的数据格式，浏览器根据此格式，可以解析出一条条由服务端定义的消息(或者说事件)，并以此作为回调的单位，而不是不可控的HTTP分块。

<h3>特点</h3>

- **单向通信：**服务端向客户端持续推送
- **基于 HTTP：**无需协议升级，直接使用 HTTP 建立连接
- **自动重连：**浏览器在连接断开后会自动重新建立连接
- **数据格式简单：**基于 `text/event-stream` 的文本流格式
- **浏览器原生支持：**通过 `EventSource` API 直接使用

<h3>实现</h3>

<h4>Spring</h4>

**`SseEmitter`**

底层基于`Servlet`，一个线程处理一个连接，不适合高并发场景。

```java
@GetMapping("/sse")
public SseEmitter sse() {
    SseEmitter emitter = new SseEmitter();
    new Thread(() -> {
        try {
            for (int i = 0; i < 5; i++) {
                emitter.send("message " + i);
                Thread.sleep(1000);
            }
            emitter.complete();
        } catch (Exception e) {
            emitter.completeWithError(e);
        }
    }).start();

    return emitter;
}
```

- `Controller`返回`emitter`后，不会关闭`HTTP`响应，知道`emitter`执行`comptlete`或`completeWithError`后响应才会关闭。