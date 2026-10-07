# 概述

`LangGraph` 是一个用**状态 + 节点 + 边** (也就是图) 来编排 Agent 执行流程的框架。

- `LangChain` 集成了可被`LangGraph`编排的组件，以简化 LLM 应用程序的开发。

## 核心概念

| 概念       | 含义                             |
| ---------- | -------------------------------- |
| State      | 整个流程共享的数据               |
| Node       | 一个处理步骤，本质是 Python 函数 |
| Edge       | 决定节点执行顺序                 |
| START      | 流程入口节点，图的第一个节点     |
| END        | 流程结束节点，图的最后一个节点   |
| StateGraph | 一个图，可以添加节点与边         |

基本的执行流程是：

```mermaid
flowchart LR
    S["START"] --> A["节点 A"]
    A --> B["节点 B"]
    B --> E["END"]
```

## 使用示例

```python
# 1. 定义共享状态
class State(TypedDict):
    text: str
    length: int

# 2. 定义节点，本质是一个函数
def clean_text(state: State):
    cleaned = state["text"].strip()

    # 返回需要更新的状态字段
    return {"text": cleaned}

def calculate_length(state: State):
    return {"length": len(state["text"])}

# 3. 创建图
builder = StateGraph(State)

# 4. 注册节点
builder.add_node("clean_text", clean_text)
builder.add_node("calculate_length", calculate_length)

# 5. 连接节点
builder.add_edge(START, "clean_text")
builder.add_edge("clean_text", "calculate_length")
builder.add_edge("calculate_length", END)

# 6. 将定义好的图编译成可执行对象
graph = builder.compile()

# 7. 执行
result = graph.invoke({
    "text": "  hello langgraph  ",
    "length": 0,
})

```

# 图(`Graph`)

## 状态(`State`)

## 配置(`config`)

在调用图时通过`config`参数传入，是本次图运行时的配置参数。

常见的配置用途有：

<h4>指定会话</h4>

当图配置了`checkpointer`时，会以`thread_id`为`key`持久化不同会话的状态，并对相同`thread_id`的调用加载之前的会话状态。

```
config = {
    "configurable": {
        "thread_id": "conversation-001"
    }
}
```

## 常用 API

<h4>检查状态</h4>

获取 `config` 对应配置的图的 `state`。

```
graph.get_state(config)
```



# 节点(`Node`)

节点是一个函数(或可调用对象)，它接收图的`State`，并通过返回值更新`State`。

- 应禁止直接在节点中更新状态，而是通过返回值由`LangGraph`统一更新。

## 预构建节点



# 边(`Edge`)

边（`Edge`）用来规定当前节点执行完成后，路由到哪一个节点，即控制节点的执行顺序。

`LangGraph`中有两类边：

- **普通边：**固定地指定下一个节点
- **条件边(Conditional edge)：**条件边是一个函数，通常包含 `if` 语句。它接收当前图的`State`， 根据当前 `State` 动态选择下一个节点，返回一个字符串或字符串列表，指示接下来要调用哪个（或哪些）节点。

<h3>向图中添加边</h3>

对于普通边，直接使用

```
graph_builder.add_edge(source_node,end_node)
```

对于条件边，使用

```
graph_builder.add_conditional_edges(source_node,route_function,path_map)
```

- **`route_function`：**定义的条件边函数或可调用对象
- **`path_map`：**节点映射，其中`key`对应条件边返回的字符串，`value`为该字符串对应的节点名称。

## 预构建条件边

`LangGraph`提供了一些具备常用功能的条件边，只要 `State` 符合 `LangGraph` 的规定，可以直接使用预构建的条件边以简化代码。

# 检查点(`checkpointer`)

检查点是 `LangGraph` 提供的一种持久化机制，用于保存每一步的状态，为 `Agnet`提供任务记忆。

- `checkpointer`以`thread_id`为key保存状态
- 图在遍历每个节点时都会通过检查点保存 `State` 

<h3>为图添加检查点(<code>checkpointer</code>)</h3>

在编译图时通过`checkpointer`参数为其添加检查点。

```python
from langgraph.checkpoint.memory import MemorySaver

memory = MemorySaver()
graph = graph_builder.compile(checkpointer=memory)
```

