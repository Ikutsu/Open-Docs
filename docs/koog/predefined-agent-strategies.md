# 预定义智能体策略

为了简化智能体实现，Koog 为常见的智能体用例提供了预定义的智能体策略。
以下是可用的预定义策略：

- [聊天智能体策略](#chat-agent-strategy)
- [ReAct 策略](#react-strategy)

## 聊天智能体策略

聊天智能体策略旨在执行聊天交互过程。
它编排不同阶段、节点和工具之间的交互，以处理用户输入、执行工具并以聊天式方式提供响应。

### 概述

聊天智能体策略实现了一种模式，其中智能体：

1. 接收用户输入
2. 使用 LLM 处理输入
3. 要么调用工具，要么提供直接响应
4. 处理工具结果并继续对话
5. 如果 LLM 尝试用纯文本而不是工具进行响应，则提供反馈

这种方法创建了一个会话式界面，智能体可以在其中使用工具来满足用户请求。

### 设置与依赖项

Koog 中聊天智能体策略的实现是通过 `chatAgentStrategy` 函数完成的。要使该函数在您的智能体代码中可用，请添加以下依赖项导入：

```
ai.koog.agents.ext.agent.chatAgentStrategy
```

要使用该策略，请按照以下模式创建 AI 智能体：

<!--- INCLUDE
import ai.koog.agents.core.agent.AIAgent
import ai.koog.prompt.executor.llms.all.simpleOpenAIExecutor
import ai.koog.agents.ext.agent.chatAgentStrategy
import ai.koog.agents.core.tools.ToolRegistry
import ai.koog.prompt.executor.clients.openai.OpenAIModels

val apiKey = System.getenv("OPENAI_API_KEY") ?: error("Please set OPENAI_API_KEY environment variable")
val promptExecutor = simpleOpenAIExecutor(apiKey)
val toolRegistry = ToolRegistry.EMPTY
val model =  OpenAIModels.Reasoning.O4Mini
-->
```kotlin
val chatAgent = AIAgent(
    executor = promptExecutor,
    toolRegistry = toolRegistry,
    llmModel = model,
    // 将 chatAgentStrategy 设置为智能体策略
    strategy = chatAgentStrategy()
)
```
<!--- KNIT example-predefined-strategies-01.kt -->

### 何时使用聊天智能体策略

聊天智能体策略特别适用于：

- 构建需要使用工具的会话式智能体
- 创建可以根据用户请求执行操作的助手
- 实现需要访问外部系统或数据的聊天机器人
- 您希望强制使用工具而非纯文本响应的场景

### 示例

以下是一个 AI 智能体的代码示例，它实现了预定义的聊天智能体策略 (`chatAgentStrategy`) 以及智能体可能使用的工具：

<!--- INCLUDE
import ai.koog.agents.core.agent.AIAgent
import ai.koog.prompt.executor.llms.all.simpleOpenAIExecutor
import ai.koog.agents.ext.agent.chatAgentStrategy
import ai.koog.agents.core.tools.ToolRegistry
import ai.koog.prompt.executor.clients.openai.OpenAIModels
import ai.koog.agents.ext.tool.AskUser
import ai.koog.agents.ext.tool.SayToUser

typealias searchTool = AskUser
typealias weatherTool = SayToUser

val apiKey = System.getenv("OPENAI_API_KEY") ?: error("Please set OPENAI_API_KEY environment variable")
val promptExecutor = simpleOpenAIExecutor(apiKey)
val toolRegistry = ToolRegistry.EMPTY
val model =  OpenAIModels.Reasoning.O4Mini
-->
```kotlin
val chatAgent = AIAgent(
    executor = promptExecutor,
    llmModel = model,
    // 将 chatAgentStrategy 用作智能体策略
    strategy = chatAgentStrategy(),
    // 添加智能体可使用的工具
    toolRegistry = ToolRegistry {
        tool(searchTool)
        tool(weatherTool)
    }
)

suspend fun main() {
    // 使用用户查询运行智能体
    val result = chatAgent.run("What's the weather like today and should I bring an umbrella?")
}
```
<!--- KNIT example-predefined-strategies-02.kt -->

## ReAct 策略

ReAct（推理与行动）策略是一种 AI 智能体策略，它在推理和执行阶段之间交替进行，以动态处理任务并从大语言模型（LLM）请求输出。

### 概述

ReAct 策略实现了一种模式，其中智能体：

1. 推理当前状态并规划后续步骤
2. 根据推理采取行动
3. 观察这些行动的结果
4. 重复循环

这种方法结合了推理（逐步思考问题）和行动（执行工具以收集信息或执行操作）的优势。

### 流程图

以下是 ReAct 策略的流程图：

![Koog 流程图](img/koog-react-diagram-light.png#only-light)
![Koog 流程图](img/koog-react-diagram-dark.png#only-dark)

### 设置与依赖项

Koog 中 ReAct 策略的实现是通过 `reActStrategy` 函数完成的。要使该函数在您的智能体代码中可用，请添加以下依赖项导入：

```
ai.koog.agents.ext.agent.reActStrategy
```
<!--- KNIT example-predefined-strategies-03.kt -->

要使用该策略，请按照以下模式创建 AI 智能体：

<!--- INCLUDE
import ai.koog.agents.core.agent.AIAgent
import ai.koog.prompt.executor.llms.all.simpleOpenAIExecutor
import ai.koog.agents.ext.agent.reActStrategy
import ai.koog.agents.core.tools.ToolRegistry
import ai.koog.prompt.executor.clients.openai.OpenAIModels

val apiKey = System.getenv("OPENAI_API_KEY") ?: error("Please set OPENAI_API_KEY environment variable")
val promptExecutor = simpleOpenAIExecutor(apiKey)
val toolRegistry = ToolRegistry.EMPTY
val model =  OpenAIModels.Reasoning.O4Mini
-->
```kotlin hl_lines="5-10"
val reActAgent = AIAgent(
    executor = promptExecutor,
    toolRegistry = toolRegistry,
    llmModel = model,
    // 将 reActStrategy 设置为智能体策略
    strategy = reActStrategy(
        // 设置可选参数值
        reasoningInterval = 1,
        name = "react_agent"
    )
)
```
<!--- KNIT example-predefined-strategies-04.kt -->

### 参数

`reActStrategy` 函数接受以下参数：

| 参数                | 类型   | 默认值   | 描述                                                                |
|---------------------|--------|----------|---------------------------------------------------------------------|
| `reasoningInterval` | Int    | 1        | 指定推理步骤的间隔。必须大于 0。                                      |
| `name`              | String | `re_act` | 策略的名称。                                                        |

### 示例用例

以下是一个 ReAct 策略如何与一个简单的银行智能体协作的示例：

#### 1. 用户输入

用户发送初始提示。例如，这可以是一个问题，如 `How much did I spend last month?`。

#### 2. 推理

智能体通过接收用户输入和推理提示执行初始推理。推理可能如下所示：

```text
I need to follow these steps:
1. Get all transactions from last month
2. Filter out deposits (positive amounts)
3. Calculate total spending
```

#### 3. 行动与执行，阶段 1

根据智能体在上一步中定义的行动项，它运行一个工具来获取上个月的所有交易。

在这种情况下，要运行的工具是 `get_transactions`，以及与获取上个月所有交易的请求相匹配的已定义 `startDate` 和 `endDate` 实参：

```text
{tool: "get_transactions", args: {startDate: "2025-05-19", endDate: "2025-06-18"}}
```

该工具返回的结果可能如下所示：

```text
[
  {date: "2025-05-25", amount: -100.00, description: "Grocery Store"},
  {date: "2025-05-31", amount: +1000.00, description: "Salary Deposit"},
  {date: "2025-06-10", amount: -500.00, description: "Rent Payment"},
  {date: "2025-06-13", amount: -200.00, description: "Utilities"}
]
```

#### 4. 推理

通过工具返回的结果，智能体再次执行推理，以确定其流程中的后续步骤：

```text
I have the transactions. Now I need to:
1. Remove the salary deposit of +1000.00
2. Sum up the remaining transactions
```

#### 5. 行动与执行，阶段 2

根据之前的推理步骤，智能体调用 `calculate_sum` 工具，该工具汇总作为工具实参提供的金额。由于推理也得出了从交易中移除正金额的行动点，因此作为工具实参提供的金额只有负数：

```text
{tool: "calculate_sum", args: {amounts: [-100.00, -500.00, -200.00]}}
```

该工具返回最终结果：

```text
-800.00
```

#### 6. 最终响应

智能体返回包含计算总和的最终响应（助手消息）：

```text
您上个月在杂货、房租和水电费上花费了 $800.00。
```

### 何时使用 ReAct 策略

ReAct 策略特别适用于：

- 需要多步推理的复杂任务
- 智能体需要在提供最终答案之前收集信息的场景
- 有益于分解为更小步骤的问题
- 需要分析性思维和工具使用的任务

### 示例

以下是一个 AI 智能体的代码示例，它实现了预定义的 ReAct 策略 (`reActStrategy`) 以及智能体可能使用的工具：

<!--- INCLUDE
import ai.koog.agents.core.agent.AIAgent
import ai.koog.prompt.executor.llms.all.simpleOpenAIExecutor
import ai.koog.agents.ext.agent.reActStrategy
import ai.koog.agents.core.tools.ToolRegistry
import ai.koog.prompt.executor.clients.openai.OpenAIModels
import ai.koog.agents.ext.tool.AskUser
import ai.koog.agents.ext.tool.SayToUser

typealias Input = String

typealias getTransactions = AskUser
typealias calculateSum = SayToUser

val apiKey = System.getenv("OPENAI_API_KEY") ?: error("Please set OPENAI_API_KEY environment variable")
val promptExecutor = simpleOpenAIExecutor(apiKey)
val toolRegistry = ToolRegistry.EMPTY
val model =  OpenAIModels.Reasoning.O4Mini
-->
```kotlin
val bankingAgent = AIAgent(
    executor = promptExecutor,
    llmModel = model,
    // 将 reActStrategy 用作智能体策略
    strategy = reActStrategy(
        reasoningInterval = 1,
        name = "banking_agent"
    ),
    // 添加智能体可使用的工具
    toolRegistry = ToolRegistry {
        tool(getTransactions)
        tool(calculateSum)
    }
)

suspend fun main() {
    // 使用用户查询运行智能体
    val result = bankingAgent.run("How much did I spend last month?")
}
```
<!--- KNIT example-predefined-strategies-05.kt -->