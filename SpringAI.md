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

<h3>召回率(<code>Recall</code>)</h3>

召回率用于衡量知识库中有多少相关信息被检索到了。

<p align="center">Recall = 检索到的相关文档数/所有相关文档数</p>

**如何提高召回率**

- `TopK`取大一些
- 降低向量检索的相似度阈值
- **`Hybrid Search`：**结合向量检索与`BM25`(关键词检索)
- `Query `改写:将原始问题改写为更标准的表达后扩展成多种表达

**精确率**

<h4>向量模型(<code>Embedding Model</code>)</h4>

本质也是一种通过训练而来AI模型，可以接收文本输入，将文本向量化，保证内容越相似的文本，在向量空间中越靠近。

- `Embedding Model`将文本映射为向量，是一个被调用的工具，无法像`Chat`模型那样提供常驻服务进行对话，只能带着输入文本被调用一次，计算完就结束。

通常有两种方式衡量向量相似度：

- 两个向量终点间的直线距离，称为欧氏距离。距离越小，向量相似度越高
- 两个向量夹角的余弦值，称为余弦距离。余弦距离越大，向量相似度越高

**向量维度**

向量维度的大小决定了向量化的语义表达能力。向量维度是向量模型在训练前确定的一个参数，一旦模型训练完成，输出维度和权重矩阵尺寸就已经固定，无法更改。每一个向量模型都有自己的向量维度，常见的向量模型与其向量维度有：

| 模型                  | 维度      |
| --------------------- | --------- |
| BERT                  | 768       |
| sentence-transformers | 384 / 768 |
| OpenAI embedding      | 1536      |
| 一些大模型            | 3072      |

按照直觉来说，向量维度应该为2的整数幂，但是大多数向量模型的向量维度并不是2的整数幂。这是因为：

向量维度本质是向量模型的一个结构参数，取决于`Transformer hidden size`，是否做拼接，`pooling`方式等因素，这些值是通过实验在表达能力、训练稳定性与计算成本之间权衡得到的最优解，而不是由计算机底层的位运算或内存对齐规则所主导。同时现代 GPU 和线性代数库（如 BLAS、CUDA）已经能够很好地对任意维度进行向量化计算与分块优化，使得维度不再需要严格遵循 2 的整数次幂约束。

现在，很多新的向量模型在模型输出前添加了一个可以动态调整的投影层，可以通过调整投影层维度调整模型输出向量维度，从而使模型支持不同程度的语义表达，以适配不同的任务场景。

<h4>向量数据库</h4>

向量数据库是一类专门用于**存储、索引和检索高维向量数据**的数据库系统，通常基于 **向量相似度（如余弦相似度、欧氏距离、内积）** 来完向量检索。

------

在RAG的具体工程实践中，通常将外部文档通过向量模型向量化并存入向量数据库，向量化的本质是把语义压缩到一个向量空间。在用户提问时基于向量相似度检索相关文档片段，再将其作为上下文交给大模型生成回答。

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

## 文档分块(`Chunking`)

在`RAG`中，文档分块是影响召回率、相关性和上下文利用效率的核心环节，核心是将长文档切分成刚好适合检索和拼接的小片段。

<h3>常见分块方式</h3>

<h4>固定长度分块</h4>

按照固定`token`/字符数切分文档，不考虑语义边界。

**特点**

- 实现简单
- 易于控制上下文大小
- 可能切断语义

<h4>滑动窗口分块</h4>

同样按照固定`token`/字符数切分文档，但分块之间增加重叠区域，以减少语义断裂。

**特点**

- 保留上下文连续性，提升召回质量
- 数据冗余，存储和向量成本更高

<h4>语义结构分块</h4>

利用文本自带的逻辑结构，如段落，标题，`Markdown`结构,`HTML`标签等，对文档进行分块。

**特点**

- 语义完整性强
- 分块长度不均匀

<h4>语义分块</h4>

使用用 NLP / embedding 模型判断语义边界，从而进行分块

<h4>LLM驱动分块</h4>

让大模型自己判断如何分块。

<h4>递归分块</h4>

递归分块是在语义结构分块的基础上，引入分块大小约束。首先按照最大语义结构（如章节、段落）进行切分；若某个分块超过设定的 chunk 大小限制，则继续在更细粒度的结构（如段落、句子）上递归拆分，直到所有分块满足 token 或字符长度约束为止。	

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

## `Agent`

Agent 是一个能够感知环境，根据用户输入和当前状态进行决策，进而主动调用工具或服务执行行动，从而持续推进目标达成的智能体。这些供`Agent`调用的预设服务或工具称为`Agent Tools`。

<h4><code>Agent</code>的三个核心要素</h4>

**感知（`Perception`）**

获取环境信息，包括用户输入，环境状态，外部数据等。

**决策(`Decision`)**

基于感知到的信息进行分析与推理，确定下一步的行动策略，例如意图识别、任务规划、工具选择等。

**行动(`Action`)**

根据决策结果执行具体操作，如调用外部工具、更新环境状态或生成并返回结果。

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

## `MCP`

`MCP`(模型上下文协议，`Model Context Protocol`)是一种用于规范大模型与外部系统之间交互的通信协议，通过定义如何向模型提供**上下文信息（Context）**，如何描述和调用**外部工具（Tools）**，如何组织和返回**执行结果（Results）**，从而让模型能够**安全、可控、可扩展地与外部世界交互**。

`MCP`的核心目标是为`LLM`提供稳定统一的上下文供给服务，上下文包括`Tool`(可执行能力)，`Resource`(外部资源，如文档，知识库)，`Prompt`(可复用提示词模板)，并进一步规范模型如何发现、选择并调用这些能力，从而实现对外部系统能力的标准化接入。



<h3>核心规范</h3>

**标准化工具描述**

所有外部工具采用统一格式描述

```
{
  "name": "get_weather",
  "description": "获取天气",
  "parameters": {
    "city": "string"
  }
}
```

**标准化调用方式**

模型输出结构化数据，由程序解析结构化数据并调用

```
{
  "tool": "get_weather",
  "args": {
    "city": "Beijing"
  }
}
```

**统一工具返回结果**

工具返回的结果必须是结构化数据。

```
{
  "temperature": 25,
  "unit": "C",
  "weather": "sunny"
}
```

**标准化上下文传递**

规定工具结果会作为一条`role`为`tool`的消息添加到上下文。

```
[
  { "role": "user", "content": "北京天气怎么样？" },
  { "role": "assistant", "tool_call": "get_weather" },
  { "role": "tool", "content": { "temperature": 25, "unit": "C" } }
]
```

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

# Workflow

`Workflow`(工作流)是对业务过程的抽象建模，用于按照预定义的规则和顺序组织多个任务的执行与流转，并对流程状态进行管理。

与传统业务流程相比，Workflow 的核心区别在于将流程从具体业务代码中抽离出来进行配置化建模，由流程引擎负责解释执行，从而使流程不再作为代码逻辑的一部分，提升了系统的可维护性、可扩展性以及可管理性。

一个完整的`workflow`通常包含以下部分：

| 组件            | 含义             |
| --------------- | ---------------- |
| Task（任务）    | 一个具体步骤     |
| Flow（流转）    | 任务之间的关系   |
| Rule（规则）    | 条件判断         |
| State（状态）   | 当前执行到哪一步 |
| Actor（执行者） | 谁来执行         |