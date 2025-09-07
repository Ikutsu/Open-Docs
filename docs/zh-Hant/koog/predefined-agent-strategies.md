# 預定義代理策略

為簡化代理實作，Koog 提供預定義的代理策略，以應對常見的代理使用案例。
以下是可用的預定義策略：

- [聊天代理策略](#chat-agent-strategy)
- [ReAct 策略](#react-strategy)

## 聊天代理策略

聊天代理策略旨在執行聊天互動流程。
它協調不同階段、節點和工具之間的互動，以處理使用者輸入、執行工具，並以類似聊天的模式提供回應。

### 概述

聊天代理策略實作了代理遵循的模式：

1. 接收使用者輸入
2. 使用 LLM 處理輸入
3. 呼叫工具或提供直接回應
4. 處理工具結果並繼續對話
5. 如果 LLM 嘗試以純文字回應而非使用工具，則提供回饋

這種方法建立了一個對話式介面，代理可以在其中使用工具來滿足使用者請求。

### 設定與依賴項

Koog 中聊天代理策略的實作是透過 `chatAgentStrategy` 函數完成的。要在你的代理程式碼中使用該函數，請添加以下依賴項匯入：

```
ai.koog.agents.ext.agent.chatAgentStrategy
```

要使用該策略，請按照以下模式建立一個 AI 代理：

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
    // Set chatAgentStrategy as the agent strategy
    strategy = chatAgentStrategy()
)
```
<!--- KNIT example-predefined-strategies-01.kt -->

### 何時使用聊天代理策略

聊天代理策略特別適用於：

- 建立需要使用工具的對話式代理
- 建立可以根據使用者請求執行動作的助理
- 實作需要存取外部系統或資料的聊天機器人
- 你希望強制使用工具而非純文字回應的場景

### 範例

這是一個 AI 代理的程式碼範例，它實作了預定義的聊天代理策略 (`chatAgentStrategy`) 以及代理可能使用的工具：

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
    // Use chatAgentStrategy as the agent strategy
    strategy = chatAgentStrategy(),
    // Add tools the agent can use
    toolRegistry = ToolRegistry {
        tool(searchTool)
        tool(weatherTool)
    }
)

suspend fun main() { 
    // Run the agent with a user query
    val result = chatAgent.run("What's the weather like today and should I bring an umbrella?")
}
```
<!--- KNIT example-predefined-strategies-02.kt -->

## ReAct 策略

ReAct（推理與行動）策略是一種 AI 代理策略，它在推理和執行階段之間交替，以動態處理任務並向大型語言模型 (LLM) 請求輸出。

### 概述

ReAct 策略實作了代理遵循的模式：

1. 推理當前狀態並規劃下一步
2. 根據該推理採取行動
3. 觀察這些行動的結果
4. 重複循環

這種方法結合了推理（逐步思考問題）和行動（執行工具以收集資訊或執行操作）的優勢。

### 流程圖

以下是 ReAct 策略的流程圖：

![Koog flow diagram](img/koog-react-diagram-light.png#only-light)
![Koog flow diagram](img/koog-react-diagram-dark.png#only-dark)

### 設定與依賴項

Koog 中 ReAct 策略的實作是透過 `reActStrategy` 函數完成的。要在你的代理程式碼中使用該函數，請添加以下依賴項匯入：

```
ai.koog.agents.ext.agent.reActStrategy
```
<!--- KNIT example-predefined-strategies-03.kt -->

要使用該策略，請按照以下模式建立一個 AI 代理：

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
    // Set reActStrategy as the agent strategy
    strategy = reActStrategy(
        // Set optional parameter values
        reasoningInterval = 1,
        name = "react_agent"
    )
)
```
<!--- KNIT example-predefined-strategies-04.kt -->

### 參數

`reActStrategy` 函數接受以下參數：

| 參數                | 類型   | 預設值   | 描述                                                           |
|---------------------|--------|----------|----------------------------------------------------------------|
| `reasoningInterval` | Int    | 1        | 指定推理步驟的間隔。必須大於 0。                               |
| `name`              | String | `re_act` | 策略的名稱。                                                   |

### 使用案例範例

以下是一個範例，說明 ReAct 策略如何與一個簡單的銀行代理協同工作：

#### 1. 使用者輸入

使用者發送初始提示。例如，這可能是一個問題，像是「上個月我花了多少錢？」。

#### 2. 推理

代理透過接收使用者輸入和推理提示來執行初始推理。推理可能如下所示：

```text
我需要遵循這些步驟：
1. 取得上個月的所有交易
2. 過濾掉存款（正數金額）
3. 計算總支出
```

#### 3. 行動與執行，階段 1

根據代理在上一步中定義的行動項目，它運行一個工具來取得上個月的所有交易。

在此情況下，要運行的工具是 `get_transactions`，以及符合取得上個月所有交易請求的已定義 `startDate` 和 `endDate` 引數：

```text
{tool: "get_transactions", args: {startDate: "2025-05-19", endDate: "2025-06-18"}}
```

該工具返回的結果可能如下所示：

```text
[
  {date: "2025-05-25", amount: -100.00, description: "Grocery Store"},
  {date: "2025-05-31", amount: +1000.00, description: "Salary Deposit"},
  {date: "2025-06-10", amount: -500.00, description: "Rent Payment"},
  {date: "2025-06-13", amount: -200.00, description: "Utilities"}
]
```

#### 4. 推理

有了工具返回的結果，代理再次執行推理以確定其流程中的下一步：

```text
我已經有交易記錄了。現在我需要：
1. 移除 +1000.00 的薪資存款
2. 加總剩餘的交易
```

#### 5. 行動與執行，階段 2

根據先前的推理步驟，代理呼叫 `calculate_sum` 工具，該工具將作為工具引數提供的金額加總。由於推理也導致了從交易中移除正數金額的行動點，因此作為工具引數提供的金額只有負數。

```text
{tool: "calculate_sum", args: {amounts: [-100.00, -500.00, -200.00]}}
```

該工具返回最終結果：

```text
-800.00
```

#### 6. 最終回應

代理返回包含計算總和的最終回應（助理訊息）：

```text
您上個月在雜貨、租金和水電費上花費了 $800.00。
```

### 何時使用 ReAct 策略

ReAct 策略特別適用於：

- 需要多步驟推理的複雜任務
- 代理需要在提供最終答案之前收集資訊的場景
- 受益於分解為更小的步驟的問題
- 需要分析性思維和工具使用的任務

### 範例

這是一個 AI 代理的程式碼範例，它實作了預定義的 ReAct 策略 (`reActStrategy`) 以及代理可能使用的工具：

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
    // Use reActStrategy as the agent strategy
    strategy = reActStrategy(
        reasoningInterval = 1,
        name = "banking_agent"
    ),
    // Add tools the agent can use
    toolRegistry = ToolRegistry {
        tool(getTransactions)
        tool(calculateSum)
    }
)

suspend fun main() { 
    // Run the agent with a user query
    val result = bankingAgent.run("How much did I spend last month?")
}
```
<!--- KNIT example-predefined-strategies-05.kt -->