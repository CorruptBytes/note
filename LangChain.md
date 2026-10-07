# 概述

## 相关包

<h3><code>langchain</code></h3>

<h3><code>langchain-[platform]</code></h3>

`LangChain`用于适配各平台模型的依赖包

## 使用示例

# `Agent`

`Agent`是一个可以在循环中调用工具，直到完成指定任务为止的系统。

![](./图片/core_agent_loop.jpeg)

## 创建`Agent`

`langchain`中可以通过`create_agent`快速创建一个可高度自定义的`agent`。

```python
create_agent(
    model: str | BaseChatModel,
    tools: Sequence[BaseTool | Callable[..., Any] | dict[str, Any]] | None = None,
    *,
    system_prompt: str | SystemMessage | None = None,
    middleware: Sequence[AgentMiddleware[StateT_co, ContextT]] = (),
    response_format: ResponseFormat[ResponseT] | type[ResponseT] | dict[str, Any] | None = None,
    state_schema: type[AgentState[ResponseT]] | None = None,
    context_schema: type[ContextT] | None = None,
    checkpointer: Checkpointer | None = None,
    store: BaseStore | None = None,
    interrupt_before: list[str] | None = None,
    interrupt_after: list[str] | None = None,
    debug: bool = False,
    name: str | None = None,
    cache: BaseCache[Any] | None = None,
    transformers: Sequence[TransformerFactory] | None = None,
) -> CompiledStateGraph[
    AgentState[ResponseT], ContextT, _InputAgentState, _OutputAgentState[ResponseT]
]
```

- `create_agent`使用`LangGraph`构建了一个基于图的智能体运行时，通过图定义`Agent`如何处理信息，该`Agent`遵循`React`模式。

## 核心参数

<h4><code>model</code>(模型)</h4>

<h4><code>tools</code>(工具)</h4>

<h4><code>system_prompt</code>(系统提示词)</h4>

<h4><code>state_schema</code></h4>

`Langchain`使用`AgentState`存储`Agent`的状态，可以通过`state_schema`参数扩展`AgentState`中的字段。

```python
class CustomAgentState(AgentState):
    user_id: str
    preferences: dict

agent = create_agent(
    "gpt-5.5",
    tools=[get_user_info],
    state_schema=CustomAgentState,
    checkpointer=InMemorySaver(),
)
```



## 调用

```
result = agent.invoke(
    {"messages": [{"role": "user", "content": "旧金山天气如何？"}]}
)
```

## 结构化输出

`create_agent`通过`response_format`参数设置结构化输出策略，从而使`agent`返回特定格式的输出。Langchain提供多种策略让`agent`返回结构化输出：

- `ToolStrategy`：适用于支持工具调用的模型，将输出结构包装为一个工具，让模型通过`Tool_Calling`返回结构化输出。
- `ProiderStrategy`：适用于提供商原生支持的结构化输出的模型，直接返回结构化输出。

```
class WeatherResponse(BaseModel):
    temperature: int
    condition: str

agent = create_agent(
    model="gpt-5",
    response_format=ProviderStrategy(WeatherResponse)
)
```



<h4><code>ToolStrategy</code></h4>

<h4><code>ProviderStrategy</code></h4>

# `Model`

`Model`即`LLM`的封装。`langchain`中的`Model`为不同平台的`LLM`提供了统一的调用接口。

`Model`通常有两种使用方式：

- 在`Agent`中使用：作为`Agent`的推理引擎。
- 单独使用：直接被调用以完成一些简单的任务，如文本生成。

## 创建`Model`

`Langchain`中提供了多种方式初始化一个`Model`，最简单的方式是调用`init_chat_model`方法，它可以以相同的方式创建不同平台的`Model`。

```python
model = init_chat_model("gpt-5.4")
```

还可以直接创建不同平台对应的`Model Class`:

```python
model = ChatOpenAI(model="gpt-5.4")
model = ChatAnthropic(model="claude-sonnet-4-6")
```

## 核心方法

**`invoke`**

调用一次模型，完全生成响应后返回输出消息。

```python
invoke(
        self,
        input: LanguageModelInput,
        config: RunnableConfig | None = None,
        *,
        stop: list[str] | None = None,
        **kwargs: Any,
    ) -> AIMessage
```

**`stream`**

调用一次模型，但是实时地流式输出返回消息。

```

```

**`batch`**

一次性向模型发送一批请求，以提高调用效率。

```

```

## 核心参数

模型的全部参数取决于其提供商和具体模型，但所有模型都具有以下标准参数:

<h4>model</h4>

目标模型的标识符名称，`string`类型，必需。可以使用`:`一起声明提供商和模型，如`open:o1:`。

<h4>api_key</h4>

在访问模型时用于身份认证的密钥，`string`类型，可选。通常使用环境变量传递。

<h4>temperature</h4>

模型温度，决定模型输出的随机性，`number`类型，可选。温度越高模型输出越随机。

<h4>max_tokens</h4>

限制模型在一次响应中输出的最大`token`数，`number`类型，可选。

<h4>timeout</h4>

等待模型响应的最大超时时间，单位为秒，`number`类型，可选。

<h4>max_retries</h4>

请求失败的最大重试次数，`number`类型，可选，默认为6。

```
model = init_chat_model(
    "claude-sonnet-4-6",
    # Kwargs passed to the model:
    temperature=0.7,
    timeout=30,
    max_tokens=1000,
    max_retries=6,  # Default; increase for unreliable networks
)
```

## 工具调用

可以使用`bind_tools`方法为模型绑定`tool`。

```python
from langchain.tools import tool

@tool
def get_weather(location: str) -> str:
    """Get the weather at a location."""
    return f"It's sunny in {location}."


model_with_tools = model.bind_tools([get_weather])

s
```

当模型绑定工具后，其响应会包含请求调用工具的消息。对于单独使用的模型，需要手动调用工具并将调用结果返回给模型用于后续推理。对于`agent`，它可以在循环中自动处理工具调用消息。

# `Memory`

## 短期记忆

`LangChain` 将短期记忆视为 Agent 状态的一部分进行统一管理，并通过 `Checkpointer` 以 `Thread` 为隔离单元持久化存储，从而保证不同会话之间的状态互不影响。

### `checkPoniter`

`Langchain`提供了不同类型的`checkPointer`，它们的主要区别是持久化方式不同:

- `InMemorySaver`：基于内存实现的`checkPointer`

  ```
  agent = create_agent(
      model="ollama:devstral-2",
      tools=[get_user_info],
      checkpointer=InMemorySaver(),
  )
  
  ```

- `PostgresSaver`：基于`Postgres`数据库实现的`checkPointer`

  ```
  pip install langgraph-checkpoint-postgres
  ```

  ```
  DB_URI = "postgresql://postgres:postgres@localhost:5432/postgres?sslmode=disable"
  with PostgresSaver.from_conn_string(DB_URI) as checkpointer:
      checkpointer.setup() # auto create tables in PostgreSQL
      agent = create_agent(
          "gpt-5.5",
          tools=[get_user_info],
          checkpointer=checkpointer,
      )
  ```

  



# `Middleware`

中间件（Middleware）提供了一种能够更精细控制 Agent 内部行为的机制，它提供在`Agent`的关键行为执行前或执行后的钩子。

<h4>使用示例</h4>

通过在`create_agent`方法中传递`middleware`参数以配置中间件。

```
agent = create_agent(
    model="gpt-5.4",
    tools=[...],
    middleware=[
        SummarizationMiddleware(...),
        HumanInTheLoopMiddleware(...)
    ],
)
```

## 内置`Middleware`

`Langchain`提供了许多内置的`Middleware`，大致可分为提供商无关和提供商定制两类。

<h3>提供商无关的<code>Middleware</code></h3>

这些中间件可以应用于任何提供商提供的 LLM 。

|      Middleware      |                  Description                   |
| :------------------: | :--------------------------------------------: |
|    Summarization     |   在接近 token 限制时自动对对话历史进行摘要    |
|  Human-in-the-loop   |       在工具调用前暂停执行以等待人工审批       |
|   Model call limit   |         限制模型调用次数以防止成本过高         |
|   Tool call limit    |       通过限制工具调用次数来控制工具执行       |
|    Model fallback    |        当主模型失败时自动切换到备用模型        |
|    PII detection     |        检测并处理个人可识别信息（PII）         |
|      To-do list      |         为智能体提供任务规划与跟踪能力         |
|  LLM tool selector   |      在调用主模型前使用 LLM 选择相关工具       |
|      Tool retry      |        对失败的工具调用进行指数退避重试        |
|     Model retry      |        对失败的模型调用进行指数退避重试        |
|  LLM tool emulator   |      使用 LLM 模拟工具执行，用于测试目的       |
|   Context editing    |   通过裁剪或清理工具使用记录来管理对话上下文   |
| Provider tool search | 将工具交由服务商端进行搜索，在需要时再动态暴露 |
|      Shell tool      |   为智能体提供可执行命令的持久化 shell 会话    |
|     File search      |    提供基于文件系统的 Glob 和 Grep 搜索工具    |
|      Filesystem      | 为智能体提供文件系统，用于存储上下文和长期记忆 |
|       Subagent       |             支持生成并调用子智能体             |

<h3>提供商定制的<code>Middleware</code></h3>

这些`Middleware`是为特定的提供商优化的。